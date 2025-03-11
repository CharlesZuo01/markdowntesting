# AWS-Networking-Test 

## Table of Contents 
- [Introduction](#introduction)
- [Prerequisites](#prerequisites)
- [AWS-Configuration](#AWS-configuration)
- [OpenVPN-Configuration](#OpenVPN-Configuration)
- [Usage](#Usage)
- [Testing](#Testing)

# Introduction
This creates an OpenVPN instance and a Ubuntu instance in us-east-1 and the AWS network infrastructure to support it. Up to two devices can establish a client VPN connection to the OpenVPN instance and ping the Ubuntu instance.

# Prerequisites
[AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html#getting-started-install-instructions)

#### Configuring AWS CLI

In the AWS console create a new AWS user and assign it to a group that has the permissions that you need. In this case, we are going to give it full administrator access

![0](https://github.com/user-attachments/assets/879e5448-8dc8-416a-8b40-287f2f4f5e78)

![0](https://github.com/user-attachments/assets/4d3c0f36-2d46-4099-9202-e5079a25da68)

After creating this user, select command CLI as the use case. We need this for terraform to work

![0](https://github.com/user-attachments/assets/d068e674-a5e2-4a84-bbd4-a5472c65c616)

Create an access key and copy paste your access key and secret access key 

![0](https://github.com/user-attachments/assets/3740fb4f-9fc2-4227-8a07-8d16e7077077)

In the CLI, run the AWS configure command and enter your access key and secret access key accordingly, along with the other inputs. In this example, we enter our keys and set our default region to us-east-1

![Untitled](https://github.com/user-attachments/assets/911d3f76-3c90-4acb-ae61-e2975b72ec6d)

[Terraform v5.33+](https://developer.hashicorp.com/terraform/tutorials/aws-get-started/install-cli)

[OpenVPN Client](https://openvpn.net/client/)

# AWS-Configuration
#### **network.tf**

network.tf creates a VPC 10.24.24.0/24 and two subnets in us-east-1a and us-east-1b with CIDRs 10.24.24.0/25 and 10.24.24.128/25 respectively. These are the subnets that the OpenVPN and Ubuntu instances are hosted in.

network.tf also creates an AWS Internet Gateway that provides internet access for the instances and configures the appropriate network routing tables ands to allow connectivity between the instances.


####  **outputs.tf**

This outputs the public IP address of the OpenVPN instance. AWS auto assigns a public IP when you use `associate_public_ip_address = true` in terraform. This output allows us to grab the public IP which will be used later for OpenVPN configuration


####  **providers.tf**

basic provider set up. See https://registry.terraform.io/providers/hashicorp/aws/latest/docs for more information


####  **security.groups.tf**

Creates open wide open security groups to allow full access between services. This terraform configuration has an open security group, but this can be modified for more granular access


####  **vpn-ec2.tf**

This creates the OpenVPN instance and references an SSH key that is not included in this repo. Please see https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/key_pair for more information but managing aws keys with terraform still has much to be desired 

####  **service-ec2.tf**

This creates an ubuntu instance that acts as our service. We will ping this instance to verify connectivity to servers across the VPN

# Diagram

<img width="725" alt="Screenshot 2025-03-06 at 9 39 21 PM" src="https://github.com/user-attachments/assets/ff7f7722-53a0-4306-be34-f29b3b4d4484" />

This show the AWS infrastructure. A user connects to the OpenVPN through the instances. Traffic is routed through the IGW to the OpenVPN instance and traffic to AWS infrastructure is tunneled through the VPN
# OpenVPN-Configuration

# Usage

# Testing


