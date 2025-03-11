# AWS-Networking-Test 

## Table of Contents 
- [Introduction](#introduction)
- [Prerequisites](#prerequisites)
- [AWS-CLI](#AWS-CLI)
- [AWS-Configuration](#AWS-configuration)
- [Diagram](#Diagram)
- [OpenVPN-Configuration](#OpenVPN-Configuration)
- [Usage](#Usage)
- [Testing](#Testing)

# Introduction
This creates an OpenVPN instance and a Ubuntu instance in us-east-1 and the AWS network infrastructure to support it. Up to two devices can establish a client VPN connection to the OpenVPN instance and ping the Ubuntu instance.

# Prerequisites
[AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html#getting-started-install-instructions)

[Terraform v5.33+](https://developer.hashicorp.com/terraform/tutorials/aws-get-started/install-cli)

[OpenVPN Client](https://openvpn.net/client/)


# AWS-CLI

In the AWS console create a new AWS user and assign it to a group that has the permissions that you need. In this case, we are going to give it full administrator access

![0](https://github.com/user-attachments/assets/879e5448-8dc8-416a-8b40-287f2f4f5e78)

![0](https://github.com/user-attachments/assets/4d3c0f36-2d46-4099-9202-e5079a25da68)

After creating this user, select command CLI as the use case. We need this for terraform to work

![0](https://github.com/user-attachments/assets/d068e674-a5e2-4a84-bbd4-a5472c65c616)

Create an access key and copy paste your access key and secret access key 

![0](https://github.com/user-attachments/assets/3740fb4f-9fc2-4227-8a07-8d16e7077077)

In the CLI, run the AWS configure command and enter your access key and secret access key accordingly, along with the other inputs. In this example, we enter our keys and set our default region to us-east-1

![Untitled](https://github.com/user-attachments/assets/911d3f76-3c90-4acb-ae61-e2975b72ec6d)

# AWS-Configuration

This section explains the terraform configuration. References to Terraform resources can be found [here](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/vpc). For terraform state, we are using a local tfstate file. More information about this later in the USAGE section

#### **network.tf**

network.tf creates a VPC 10.24.24.0/24 and two subnets in us-east-1a and us-east-1b with CIDRs 10.24.24.0/25 and 10.24.24.128/25 respectively. These are the subnets that the OpenVPN and Ubuntu instances are hosted in.

network.tf also creates an AWS Internet Gateway that provides internet access for the instances and configures the appropriate network routing tables ands to allow connectivity between the instances.


####  **outputs.tf**

This outputs the public IP address of the OpenVPN instance. AWS auto assigns a public IP when you use `associate_public_ip_address = true` in terraform. This output allows us to grab the public IP which will be used later for OpenVPN configuration


####  **providers.tf**

This sets up our AWS provider configuration. See [here](https://registry.terraform.io/providers/hashicorp/aws/latest/docs) for more information on provider and backend set up

####  **security.groups.tf**

Creates open wide open security groups to allow full access between services. This terraform configuration has an open security group, but this can be modified for more granular access


####  **vpn-ec2.tf**

This creates the OpenVPN instance and references an the SSH key that was manually created. Please see this [link](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/key_pair) for more information on managing AWS key pairs with terraform, but the gist of it is that it is easier to reference an existing key pair with Terraform than it is to create one and then manage it.

####  **service-ec2.tf**

This creates an ubuntu instance that acts as our service. We will ping this instance to verify connectivity to servers across the VPN

Create the service by running the following commands
```
terraform plan
terraform apply 
```
Then type yes when asked if you want to apply

# Diagram

This show the AWS infrastructure. A user connects to the OpenVPN through the instances. Traffic is routed through the IGW to the OpenVPN instance and traffic to AWS infrastructure is tunneled through the VPN. The 10.24.24.0/24 network route is installed in the user device local routing table, and all traffic to this CIDR is tunneled through the VPN. The Ubuntu server can be pinged through VPN

![0](https://github.com/user-attachments/assets/eeb11e6b-904c-4e3b-9acc-2db7e5cc8dda)

# OpenVPN-Configuration



# Usage

# Testing


