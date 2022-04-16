# Project Title

Recruitment task

## Description

Please prepare the Virtual machine described below.
Task Description:

Virtual machine with installed Linux OS from prferably - Debian, Ubuntu with the following
software installed:
- php7.4
- apache2
- simple website ("Hello World") behind apache2

VM should be prepared and configured using IaaC tools (preferably Vagrant/Terraform or
Packer and Ansible) - we should be able to recreate this VM with the same configuration in
our environment by just using configuration files prepared by the candidate.

The project should have a simple README explaining how to set up the VM from scratch.

## Getting Started

The code was written in HCL and used in Terraform.
In the next steps, I will describe the dependencies and what a given element means.

* Step-by-step bullets

Here we enter the provider which is 'aws' and the region which is described in the variables, this solution is used when we have a large number of virtual machines on which we do not need to write separately 'ami', but we make changes globally.


```
provider "aws" {
    region = var.region_webserver
}

```

The next section is resources where we define the 'security group', its name 'AWS_SG', description and communication inputs and outputs specified on ports 80 and 22 using the TCP protoco.

```
resource "aws_security_group" "AWS_SG" {
    name = "AWS_SG"
    description = "Securty group for Webserver instance"

    ingress {
        from_port = 80
        protocol = "TCP"
        to_port = 80
        cidr_blocks = ["0.0.0.0/0"]
    }

    ingress {
        from_port = 22
        protocol = "TCP"
        to_port = 22
        cidr_blocks = ["89.64.40.140/32"]
    }

    egress {
        from_port = 0
        protocol = "-1"
        to_port = 0
        cidr_blocks = ["0.0.0.0/0"]
    }
}
```

The last element of the template are resources that describe the name of the virtual machine. 'Instance_type' refers to variables, and so does 'ami', the name of the key we use to bind to aws. Tags on which we will be able to find the server and security group to which we assign the machine easier.



```
resource "aws_instance" "Web_Ubuntu" {
        instance_type = var.type_webserver
        ami = var.ami_webserver
        key_name = "key_pair" 
        tags = {
            name = "Web_Ubuntu"
        }
        security_groups = ["${aws_security_group.AWS_SG.name}"]
        
        
```
The part of the template that is not static in the template and specifies the input of additional data by the user.
The last element is to call End Of File to the machine itself using commands.
```
        user_data = <<-EOF
      #!/bin/sh # first line in bash script - calling 
      sudo apt-get update # update all packages in linux 
      sudo apt install -y php7.4  # install php7.4
      sudo apt install -y apache2 # install apache2
      sudo systemctl status apache2 # chceck status of apache2
      sudo systemctl start apache2 # start apache 
      sudo chown -R $USER:$USER /var/www/html
      sudo echo "<html><body><h1>Hello World</h1></body></html>" > /var/www/html/index.html
      EOF
}

```
### Executing program



```
provider "aws" {
    region = var.region_webserver
}

resource "aws_security_group" "AWS_SG" {
    name = "AWS_SG"
    description = "Securty group for Webserver instance"

    ingress {
        from_port = 80
        protocol = "TCP"
        to_port = 80
        cidr_blocks = ["0.0.0.0/0"]
    }

    ingress {
        from_port = 22
        protocol = "TCP"
        to_port = 22
        cidr_blocks = ["89.64.40.140/32"]
    }

    egress {
        from_port = 0
        protocol = "-1"
        to_port = 0
        cidr_blocks = ["0.0.0.0/0"]
    }
}

resource "aws_instance" "Web_Ubuntu" {
        instance_type = var.type_webserver
        ami = var.ami_webserver
        key_name = "key_pair" 
        tags = {
            name = "Web_Ubuntu"
        }
        security_groups = ["${aws_security_group.AWS_SG.name}"]
        user_data = <<-EOF
      #!/bin/sh
      sudo apt-get update
      sudo apt install -y php7.4 
      sudo apt install -y apache2
      sudo systemctl status apache2
      sudo systemctl start apache2
      sudo chown -R $USER:$USER /var/www/html
      sudo echo "<html><body><h1>Hello World</h1></body></html>" > /var/www/html/index.html
      EOF
}

```
Variables that can be changed globally
```
variable "region_webserver" {default = "eu-central-1"}
variable "type_webserver" {default = "t2.micro"}
variable "ami_webserver" {default = "ami-0d527b8c289b4af7f"}

```
the result displayed in the terminal

```
output "publicip" {
    value = "${aws_instance.Web_Ubuntu.public_ip}"
}

```



## Authors

Contributors names and contact info

ex. Maciej Znalezniak
ex. [@gmail.com]

## Version History

* 1.0
    * Various bug fixes and optimizations
    * See [commit change]() or See [release history]()



