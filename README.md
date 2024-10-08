
# AWS VPC with Load Balancer and Private EC2 Instances

## Project Overview

In this project, we will build a secure Virtual Private Cloud (VPC) with a public and private subnet, set up a couple of EC2 instances in a private subnet, and configure a Classic Load Balancer to route traffic. This setup is commonly used in multi-tier web applications, where the frontend is hosted on a public subnet and the backend on private subnets. Understanding this architecture is essential for anyone working with AWS infrastructure because it’s a foundational pattern for high availability and security.

The purpose of this guide is to not just walk through the steps but also to discuss the *why* behind each configuration. Let's dive in!


![chrome_Oa17PJQEx7](https://github.com/user-attachments/assets/ade2e62b-707d-4140-adde-e65a85fab598)


## Table of Contents
- [Prerequisites](#prerequisites)
- [Architecture](#architecture)
- [Project Components](#project-components)
- [Step-by-Step Guide](#step-by-step-guide)
  - [1. Create a VPC](#1-create-a-vpc)
  - [2. Create Public and Private Subnets](#2-create-public-and-private-subnets)
  - [3. Configure Security Groups](#3-configure-security-groups)
  - [4. Launch EC2 Instances in Private Subnet](#4-launch-ec2-instances-in-private-subnet)
  - [5. Create a Classic Load Balancer](#5-create-a-classic-load-balancer)
  - [6. Test the Setup](#6-test-the-setup)
- [Cleanup](#cleanup)
- [License](#license)

## Prerequisites

Before starting, ensure you have the following:

- An [AWS account](https://aws.amazon.com/free/) (Free Tier eligible).
- Basic knowledge of AWS services, including VPC, EC2, and Load Balancers.
- AWS CLI and/or AWS Management Console access.
- SSH client (optional, if connecting to EC2 instances directly).

## Architecture

The architecture consists of the following components:

- **VPC**: Custom VPC with both public and private subnets.
- **Subnets**: 
  - **Public Subnet**: Contains the Load Balancer and NAT Gateway.
  - **Private Subnet**: Contains EC2 instances.
- **Load Balancer**: Classic Load Balancer that distributes traffic to EC2 instances in the private subnet.
- **Security Groups**: Defined for EC2 instances and Load Balancer to control traffic.
- **NAT Gateway**: Enables outbound internet access for EC2 instances in private subnets.

## Project Components

- **VPC**: `10.0.0.0/16`
- **Public Subnet**: `10.0.1.0/24`
- **Private Subnet**: `10.0.2.0/24`
- **Load Balancer**: HTTP Listener on Port 80
- **EC2 Instances**: Amazon Linux 2 AMI, `t2.micro`

## Step-by-Step Guide

### 1. Create a VPC
The first step is to create a Virtual Private Cloud (VPC). Think of a VPC as your own private data center in the cloud. This is where you define the networking rules, subnets, and IP ranges for your infrastructure.

1. **Navigate to the VPC Console**: Open the Amazon VPC console at [VPC Console](https://console.aws.amazon.com/vpc/).
2. **Create the VPC**:
   - **Click on** `Your VPCs` and then `Create VPC`.
   - **Set the name** (e.g., `Project2-VPC`).
   - **IPv4 CIDR Block**: Use `10.0.0.0/16` or `192.168.0.0/16` (choose any non-overlapping range).
   - **Tenancy**: Set to `Default` unless you have specific compliance needs for dedicated hardware.
   
3. **Why these choices?**  
   - CIDR Block defines the range of IP addresses available. Using a /16 block gives a large range, which is perfect for complex infrastructures.
   - Default tenancy is more cost-effective unless you need dedicated hardware for compliance.


### 2. Create Public and Private Subnets
Subnets are segments within your VPC, used to isolate resources and control traffic flow. We need one public and two private subnets for high availability.

1. **Create the Public Subnet**:
   - **Navigate to** `Subnets` in the VPC console.
   - Click on `Create Subnet`.
   - **Name**: `Public-Subnet`.
   - **VPC**: Select the VPC created earlier.
   - **CIDR Block**: Use `10.0.1.0/24`.
   - **Availability Zone**: Choose the first zone available.

2. **Create the Private Subnet**:
   - Repeat the steps above to create two private subnets, e.g., `Private-Subnet-A` and `Private-Subnet-B`.
   - Use CIDR blocks like `10.0.2.0/24` and `10.0.3.0/24` respectively.
   
3. **Why multiple subnets?**  
   The idea is to have multiple subnets across different availability zones to ensure that your application can handle failures in a single zone.


### 3. Configure Security Groups
Security groups are like virtual firewalls for your EC2 instances. We’ll create a security group that allows HTTP (port 80) and SSH (port 22) traffic.

1. **Create a Security Group**:
   - **Go to** `Security Groups` in the VPC console.
   - **Create Security Group**.
   - **Name**: `Project2-SG`.
   - **Description**: Allow HTTP and SSH traffic.
   - **VPC**: Select your VPC.

2. **Inbound Rules**:
   - Add rules for:
     - **Type**: HTTP, **Port**: 80, **Source**: Anywhere (0.0.0.0/0).
     - **Type**: SSH, **Port**: 22, **Source**: Anywhere.

3. **Why these rules?**  
   HTTP is for accessing your web server, and SSH is for securely connecting to your instances. We’re using "Anywhere" for simplicity, but in production, always limit SSH access to specific IPs.


### 4. Launch EC2 Instances in Private Subnet
You need a key pair to securely connect to your instances via SSH.

1. **Navigate to** `Key Pairs` in the EC2 console.
2. **Create Key Pair** and save the `.pem` file securely.

### 4.2 Launch EC2 Instances
1. **Launch an Instance**:
   - **AMI**: Choose Amazon Linux 2.
   - **Instance Type**: t2.micro.
   - **Number of Instances**: 2.
   - **Network**: Select your VPC.
   - **Subnet**: Choose `Private-Subnet-A` for the first instance and `Private-Subnet-B` for the second.

2. **User Data Script**:
   Add this script to install Apache automatically:

   ```bash
   #!/bin/bash
   sudo yum update -y
   sudo yum install httpd -y
   sudo systemctl start httpd
   sudo systemctl enable httpd
   echo "Hello from EC2 Instance $(hostname -f)" > /var/www/html/index.html
   ```

3. **Security Group**:
   Use the security group created earlier (`Project2-SG`).

4. **Key Pair**:
   Select the key pair you created earlier and launch.

5. Repeat the same steps to launch a **second EC2 instance** in the same private subnet.

### 5. Creating a NAT Gateway
A NAT Gateway allows instances in private subnets to connect to the internet without being exposed publicly.

1. **Navigate to** `NAT Gateways` in the VPC console.
2. **Create NAT Gateway**:
   - **Name**: `Project2-NAT`.
   - **Subnet**: Choose your `Public-Subnet`.
   - **Elastic IP**: Allocate a new Elastic IP.

3. **Why a NAT Gateway?**  
   This allows our private EC2 instances to download packages (like Apache) without exposing them to the internet.

### 6. Setting up the Internet Gateway
1. **Create an Internet Gateway** in the VPC console.
2. Attach it to your VPC.

### 7. Configuring Route Tables
1. **Create Route Tables** for both public and private subnets.
2. **Public Route Table**:
   - Add a route to `0.0.0.0/0` and target your Internet Gateway.
3. **Private Route Table**:
   - Add a route to `0.0.0.0/0` and target your NAT Gateway.

### 8. Create a Classic Load Balancer
**Open the EC2 Console**.
2. **Create a Classic Load Balancer**:
   - **VPC**: Select your VPC.
   - **Subnets**: Associate with `Public-Subnet`.
   - **Instances**: Register your private instances.

3. **Security Group**: Choose `Project2-SG`.
4. **Why use a Classic Load Balancer?**  
   A Classic Load Balancer can handle both HTTP and TCP traffic, making it ideal for load balancing web applications.


### 9. Test the Setup
1. Get the DNS name of the Load Balancer.
2. Open it in your browser. You should see "Welcome to Project 2!" displayed.

And that’s it! You’ve successfully set up a VPC with high availability using private subnets and a Classic Load Balancer. This setup is a common pattern for highly available web applications in AWS.

## Cleanup

To avoid unexpected charges, delete the resources created in this project:

1. Terminate EC2 instances.
2. Delete the Load Balancer.
3. Delete the VPC and associated subnets.

## License
This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.
