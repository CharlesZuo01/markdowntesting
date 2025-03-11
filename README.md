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


#### Creating your key pair

We are going to manually create a key pair in the AWS console because this is more efficient than creating a key pair with your local device and then uploading this key pair into AWS through terraform. Go to EC2 -> Key Pairs and click Create key pair

![Untitled](https://github.com/user-attachments/assets/05467e9b-2ccf-4737-8583-d9f57432d448)

Enter a name for your key pair and save the format as pem. The name of this key pair will be referened in the vpn-ec2.tf configuration file.

![Untitled](https://github.com/user-attachments/assets/c0091169-20f7-451f-b3e2-123aa3b36d49)


#### **network.tf**

network.tf creates a VPC 10.24.24.0/24 and two subnets in us-east-1a and us-east-1b with CIDRs 10.24.24.0/25 and 10.24.24.128/25 respectively. These are the subnets that the OpenVPN and Ubuntu instances are hosted in.

network.tf also creates an AWS Internet Gateway that provides internet access for the instances and configures the appropriate network routing tables ands to allow connectivity between the instances.


####  **outputs.tf**

This outputs the URL for the openvpn web admin login. This URL includes the OpenVPN instance public IP address which we will use to SSH to the instance using the SSH key we created earlier. This outputs.tf file also outputs the private IP address of the Ubuntu server, which we will ping to verify our connectivity to internal resources.

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

After your OpenVPN instance is built, run `terraform output` to get the public IP address of your VPN instance. Using the key pair created earlier, SSH to the instance

```
PS F:\switchet\CharlesZuo01-AWS-Networking-Test> terraform output
access_vpn_url = "https://44.211.41.123:943/admin"
PS F:\switchet\CharlesZuo01-AWS-Networking-Test> ssh -i C:\Users\User\Downloads\openvpn.pem root@44.211.41.123      
```

Finish the wizard setup. All default options can be entered. Go to the OpenVPN web console https://44.211.41.123:943/admin in your browser. See example

```
Please enter 'yes' to indicate your agreement [no]: yes

Once you provide a few initial configuration settings,
OpenVPN Access Server can be configured by accessing
its Admin Web UI using your Web browser.

Will this be the primary Access Server node?
(enter 'no' to configure as a backup or standby node)
> Press ENTER for default [yes]:

Please specify the network interface and IP address to be
used by the Admin Web UI:
(1) all interfaces: 0.0.0.0
(2) eth0: 10.24.24.67
Please enter the option number from the list above (1-2).
> Press Enter for default [1]: 2

Please specify the port number for the Admin Web UI.
> Press ENTER for default [943]:

Please specify the TCP port number for the OpenVPN Daemon
> Press ENTER for default [443]:

Should client traffic be routed by default through the VPN?
> Press ENTER for default [no]:

Should client DNS traffic be routed by default through the VPN?
> Press ENTER for default [no]:

Use local authentication via internal DB?
> Press ENTER for default [yes]:

Private subnets detected: ['10.24.24.0/24']

Should private subnets be accessible to clients by default?
> Press ENTER for EC2 default [yes]:

To initially login to the Admin Web UI, you must use a
username and password that successfully authenticates you
with the host UNIX system (you can later modify the settings
so that RADIUS or LDAP is used for authentication instead).

You can login to the Admin Web UI as "openvpn" or specify
a different user account to use for this purpose.

Do you wish to login to the Admin UI as "openvpn"?
> Press ENTER for default [yes]:

> Please specify your Activation key (or leave blank to specify later):

```
After initialization, the openvpn instance will disconnect you. Log back in and set a password

```
PS C:\Users\User\Downloads> ssh -i .\openvpn.pem  openvpnas@54.152.152.154
Welcome to OpenVPN Access Server Appliance 2.8.5

  System information as of Tue Mar 11 06:00:30 UTC 2025

  System load:  0.12              Users logged in:      0
  Usage of /:   16.9% of 7.69GB   IP address for eth0:  10.24.24.67
  Memory usage: 41%               IP address for as0t0: 172.27.224.1
  Swap usage:   0%                IP address for as0t1: 172.27.232.1
  Processes:    105

30 packages can be updated.
19 updates are security updates.


To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

openvpnas@ip-10-24-24-67:~$ sudo passwd openvpn
Enter new UNIX password:
Retype new UNIX password:
passwd: password updated successfully
```

Login to the web console using the creds openvpn/[password you created above]

Go to user management and create a new user. Under Access Control section, add the CIDR that your service is hosted in

![Untitled](https://github.com/user-attachments/assets/0bbb6a11-612e-494a-bfbb-120d5f555b86)

Open the OpenVPN client, create a new profile with the IP address of your instance, and connect to the instance using the new user you created.


![Untitled](https://github.com/user-attachments/assets/7692716c-6bba-4e24-b3a6-aca500023507)


# Testing
Run `terraform output` to get the IP address of the ubuntu server
Ping the ubuntu server when connected to VPN 

```
PS F:\switchet\CharlesZuo01-AWS-Networking-Test> terraform output
access_vpn_url = "https://54.152.152.154:943/admin"
service_private_ip = "10.24.24.221"
PS F:\switchet\CharlesZuo01-AWS-Networking-Test> ping 10.24.24.221

Pinging 10.24.24.221 with 32 bytes of data:
Reply from 10.24.24.221: bytes=32 time=77ms TTL=126
Reply from 10.24.24.221: bytes=32 time=77ms TTL=126
Reply from 10.24.24.221: bytes=32 time=76ms TTL=126
Reply from 10.24.24.221: bytes=32 time=76ms TTL=126
```
