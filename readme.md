# Setting Up a Private EC2 Instance Accessible via a Bastion Host in AWS

This guide will walk you through creating an Amazon EC2 instance with a private IP address (no public IP) and setting up a bastion host to connect to it securely.

## Table of Contents

- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Step-by-Step Guide](#step-by-step-guide)
  - [1. Create a VPC](#1-create-a-vpc)
  - [2. Create Subnets](#2-create-subnets)
  - [3. Set Up an Internet Gateway](#3-set-up-an-internet-gateway)
  - [4. Configure Route Tables](#4-configure-route-tables)
  - [5. Create Security Groups](#5-create-security-groups)
  - [6. Launch the Bastion Host](#6-launch-the-bastion-host)
  - [7. Launch the Private EC2 Instance](#7-launch-the-private-ec2-instance)
  - [8. Connect to the Private EC2 Instance via Bastion Host](#8-connect-to-the-private-ec2-instance-via-bastion-host)
- [Notes](#notes)
- [Troubleshooting](#troubleshooting)
- [Conclusion](#conclusion)

---

## Overview

We will:

1. **Create a Virtual Private Cloud (VPC)**
2. **Create Public and Private Subnets**
3. **Set Up an Internet Gateway**
4. **Configure Route Tables**
5. **Create Security Groups**
6. **Launch a Bastion Host in the Public Subnet**
7. **Launch a Private EC2 Instance in the Private Subnet**
8. **Connect to the Private EC2 Instance via the Bastion Host**

---

## Prerequisites

- **AWS Account** with appropriate permissions
- **SSH Key Pair** for EC2 instances
- Basic understanding of AWS services (EC2, VPC, Security Groups)

---

## Step-by-Step Guide

### 1. Create a VPC

1. **Navigate to the VPC Dashboard**:
   - Sign in to the AWS Management Console.
   - Go to **Services** > **VPC**.

2. **Create a New VPC**:
   - Click on **Your VPCs** > **Create VPC**.
   - **Name tag**: `MyVPC`
   - **IPv4 CIDR block**: `10.0.0.0/16`
   - Leave other settings as default.
   - Click **Create VPC**.

### 2. Create Subnets

#### Public Subnet

1. **Navigate to Subnets**:
   - Click on **Subnets** > **Create subnet**.

2. **Configure the Public Subnet**:
   - **Name tag**: `PublicSubnet`
   - **VPC**: Select `MyVPC`
   - **Availability Zone**: Choose any (e.g., `us-east-1a`)
   - **IPv4 CIDR block**: `10.0.1.0/24`
   - Click **Create subnet**.

#### Private Subnet

1. **Create Another Subnet**:
   - Click on **Create subnet**.

2. **Configure the Private Subnet**:
   - **Name tag**: `PrivateSubnet`
   - **VPC**: Select `MyVPC`
   - **Availability Zone**: Same or different (e.g., `us-east-1b`)
   - **IPv4 CIDR block**: `10.0.2.0/24`
   - Click **Create subnet**.

### 3. Set Up an Internet Gateway

1. **Create an Internet Gateway**:
   - Click on **Internet Gateways** > **Create internet gateway**.
   - **Name tag**: `MyInternetGateway`
   - Click **Create internet gateway**.

2. **Attach the Internet Gateway to the VPC**:
   - Select `MyInternetGateway`.
   - Click **Actions** > **Attach to VPC**.
   - Select `MyVPC` and click **Attach internet gateway**.

### 4. Configure Route Tables

#### Public Route Table

1. **Create a Route Table**:
   - Click on **Route Tables** > **Create route table**.
   - **Name tag**: `PublicRouteTable`
   - **VPC**: Select `MyVPC`
   - Click **Create route table**.

2. **Edit Routes**:
   - Select `PublicRouteTable`.
   - Click on the **Routes** tab > **Edit routes**.
   - Click **Add route**:
     - **Destination**: `0.0.0.0/0`
     - **Target**: Select `MyInternetGateway`
   - Click **Save routes**.

3. **Associate with Public Subnet**:
   - Click on the **Subnet associations** tab > **Edit subnet associations**.
   - Select `PublicSubnet`.
   - Click **Save associations**.

#### Private Route Table

1. **Create Another Route Table**:
   - Click on **Create route table**.
   - **Name tag**: `PrivateRouteTable`
   - **VPC**: Select `MyVPC`
   - Click **Create route table**.

2. **Associate with Private Subnet**:
   - Select `PrivateRouteTable`.
   - Click on the **Subnet associations** tab > **Edit subnet associations**.
   - Select `PrivateSubnet`.
   - Click **Save associations**.

### 5. Create Security Groups

#### Bastion Host Security Group

1. **Navigate to EC2 Dashboard**:
   - Go to **Services** > **EC2**.

2. **Create a Security Group**:
   - Click on **Security Groups** > **Create security group**.
   - **Security group name**: `BastionSecurityGroup`
   - **Description**: `Allows SSH access from trusted IPs`
   - **VPC**: Select `MyVPC`

3. **Configure Inbound Rules**:
   - Click **Add Rule**:
     - **Type**: `SSH`
     - **Protocol**: `TCP`
     - **Port Range**: `22`
     - **Source**: `Your IP` (or specify an IP range)
   - Click **Create security group**.

#### Private EC2 Instance Security Group

1. **Create Another Security Group**:
   - Click on **Create security group**.
   - **Security group name**: `PrivateInstanceSecurityGroup`
   - **Description**: `Allows SSH from Bastion Host`
   - **VPC**: Select `MyVPC`

2. **Configure Inbound Rules**:
   - Click **Add Rule**:
     - **Type**: `SSH`
     - **Protocol**: `TCP`
     - **Port Range**: `22`
     - **Source**: Select `BastionSecurityGroup` (start typing its name and select from the dropdown)
   - Click **Create security group**.

### 6. Launch the Bastion Host

1. **Launch an EC2 Instance**:
   - Click on **Instances** > **Launch instances**.

2. **Configure the Bastion Host**:
   - **Name**: `BastionHost`
   - **AMI**: Choose **Amazon Linux 2 AMI**
   - **Instance Type**: `t2.micro` (or as needed)
   - **Key Pair**: Select or create a key pair
   - **Network Settings**:
     - **VPC**: Select `MyVPC`
     - **Subnet**: Select `PublicSubnet`
     - **Auto-assign Public IP**: Enable
     - **Firewall (security groups)**: Select `BastionSecurityGroup`
   - Click **Launch instance**.

### 7. Launch the Private EC2 Instance

1. **Launch Another EC2 Instance**:
   - Click on **Launch instances**.

2. **Configure the Private Instance**:
   - **Name**: `PrivateInstance`
   - **AMI**: Choose **Amazon Linux 2 AMI**
   - **Instance Type**: `t2.micro`
   - **Key Pair**: Use the same key pair (optional)
   - **Network Settings**:
     - **VPC**: Select `MyVPC`
     - **Subnet**: Select `PrivateSubnet`
     - **Auto-assign Public IP**: Disable
     - **Firewall (security groups)**: Select `PrivateInstanceSecurityGroup`
   - Click **Launch instance**.

### 8. Connect to the Private EC2 Instance via Bastion Host

#### SSH into the Bastion Host

1. **Retrieve Bastion Host's Public IP**:
   - In the EC2 console, select `BastionHost`.
   - Note the **Public IPv4 address**.

2. **Connect via SSH**:
   - Open your terminal.
   - Run:

     ```bash
     ssh -i /path/to/your/key.pem ec2-user@<BastionHost_Public_IP>
     ```

#### SSH from Bastion Host to Private Instance

1. **Copy Key Pair to Bastion Host**:
   - From your local machine, copy the key pair to the bastion host:

     ```bash
     scp -i /path/to/your/key.pem /path/to/your/key.pem ec2-user@<BastionHost_Public_IP>:/home/ec2-user/
     ```

2. **Set Permissions on Bastion Host**:
   - On the bastion host, run:

     ```bash
     chmod 600 /home/ec2-user/key.pem
     ```

3. **Retrieve Private Instance's Private IP**:
   - In the EC2 console, select `PrivateInstance`.
   - Note the **Private IPv4 address**.

4. **Connect to Private Instance**:
   - From the bastion host, run:

     ```bash
     ssh -i /home/ec2-user/key.pem ec2-user@<PrivateInstance_Private_IP>
     ```

   - You should now be connected to the private EC2 instance.

---

## Notes

- **Security Best Practices**:
  - Restrict SSH access to specific IP addresses.
  - Regularly update and patch your instances.
  - Consider using AWS Systems Manager Session Manager for secure and auditable access without the need for SSH keys.

- **NAT Gateway (Optional)**:
  - If your private instance needs internet access for updates or downloads, set up a NAT Gateway in the public subnet.
  - Update the private route table to direct internet traffic through the NAT Gateway.

## Troubleshooting

- **Cannot SSH into Bastion Host**:
  - Verify that the bastion host's security group allows SSH from your IP address.
  - Ensure the key pair matches and permissions are set (`chmod 400 key.pem`).

- **Cannot SSH from Bastion Host to Private Instance**:
  - Check that the private instance's security group allows SSH from the bastion host's security group.
  - Confirm that the key pair file on the bastion host has the correct permissions (`chmod 600 key.pem`).

- **Permission Denied Errors**:
  - Ensure you are using the correct username (`ec2-user` for Amazon Linux).
  - Confirm that the key pair is associated with the private instance.

## Conclusion

You have successfully set up a private EC2 instance and a bastion host for secure access. This architecture enhances security by keeping your resources private while allowing necessary connectivity.

---
