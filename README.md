
# AWS VPC with Load Balancer and Private EC2 Instances

## Project Overview

This project demonstrates how to create a high-availability architecture in AWS using a Virtual Private Cloud (VPC) with public and private subnets. It sets up a Classic Load Balancer in a public subnet to distribute traffic to EC2 instances located in private subnets. The EC2 instances host a basic Apache web server.

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
1. Go to the **VPC Dashboard** in the AWS Management Console.
2. Click on **Create VPC** and configure as follows:
   - **Name Tag**: `Project2-VPC`
   - **IPv4 CIDR Block**: `10.0.0.0/16`
3. Click **Create VPC**.

### 2. Create Public and Private Subnets
1. **Public Subnet**:
   - **Subnet Name**: `Public-Subnet`
   - **VPC ID**: Select `Project2-VPC`
   - **Availability Zone**: `us-west-2a`
   - **CIDR Block**: `10.0.1.0/24`
   - Enable **Auto-assign public IPv4 address**.
2. **Private Subnet**:
   - **Subnet Name**: `Private-Subnet`
   - **VPC ID**: Select `Project2-VPC`
   - **Availability Zone**: `us-west-2a`
   - **CIDR Block**: `10.0.2.0/24`

### 3. Configure Security Groups
1. Go to **EC2 Dashboard** > **Security Groups** > **Create Security Group**.
2. Create a security group called `Project2-SG` with the following rules:
   - **Inbound Rules**:
     - **Type**: SSH | **Protocol**: TCP | **Port Range**: 22 | **Source**: My IP
     - **Type**: HTTP | **Protocol**: TCP | **Port Range**: 80 | **Source**: 0.0.0.0/0
3. Attach the security group to your EC2 instances.

### 4. Launch EC2 Instances in Private Subnet
1. Go to **EC2 Dashboard** and **Launch Instance**.
2. Choose the **Amazon Linux 2 AMI** and select **t2.micro**.
3. Configure the instance as follows:
   - **Network**: Select `Project2-VPC`
   - **Subnet**: Select `Private-Subnet`
4. Add the following **User Data** script to install Apache automatically:

   ```bash
   #!/bin/bash
   sudo yum update -y
   sudo yum install httpd -y
   sudo systemctl start httpd
   sudo systemctl enable httpd
   echo "Hello from EC2 Instance $(hostname -f)" > /var/www/html/index.html
   ```

5. Repeat the same steps to launch a **second EC2 instance** in the same private subnet.

### 5. Create a Classic Load Balancer
1. Go to **EC2 Dashboard** > **Load Balancers** > **Create Load Balancer**.
2. Select **Classic Load Balancer**.
3. Configure as follows:
   - **Name**: `Project2-LB`
   - **VPC**: Select `Project2-VPC`
   - **Subnet**: Select `Public-Subnet`
   - **Listeners**: HTTP, Port 80
4. Add both EC2 instances to the target group.
5. Configure Health Check for `/index.html`.

### 6. Test the Setup
1. Copy the **DNS Name** of the Load Balancer.
2. Open the DNS Name in your web browser.
3. You should see the custom message "Hello from EC2 Instance" from one of the private EC2 instances.

## Cleanup

To avoid unexpected charges, delete the resources created in this project:

1. Terminate EC2 instances.
2. Delete the Load Balancer.
3. Delete the VPC and associated subnets.

## License
This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.
