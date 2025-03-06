## IATD Microcredential Cloud Networking: Supplementary Lab - AWS VPC Fundamentals

**Objective:** In this lab, you will learn how to create and configure basic networking components in AWS using both the AWS Management Console (portal) and AWS CloudShell. This supplementary lab provides hands-on experience with AWS networking fundamentals to complement your Azure knowledge.

**Estimated Time:** 60 minutes

**Prerequisites:**

1.  **AWS Account:** You need an AWS account with appropriate permissions
2.  **AWS Management Console Access:** Access to AWS Management Console
3.  **Basic Understanding:** Familiarity with basic networking concepts

**Let's get started!**

### Resource Naming Convention

All resources created in this lab will follow this naming pattern:
- VPC: `iatd_labs_aws_vpc`
- Subnets: `iatd_labs_aws_subnet_[public/private]`
- Security Groups: `iatd_labs_aws_sg_[purpose]`
- Route Tables: `iatd_labs_aws_rt_[public/private]`
- EC2 Instances: `iatd_labs_aws_vm_[public/private]`

### IP Addressing

We'll use the following IP address ranges:
- VPC CIDR: 172.16.0.0/16
- Public Subnet: 172.16.1.0/24
- Private Subnet: 172.16.2.0/24

### Part 1: Creating a VPC and Subnets (Portal)

In this part, you will use the AWS Management Console to create the VPC and subnets.

1.  **Create a VPC:**
    * Navigate to VPC service in AWS Console
    * Click "Create VPC"
    * Enter the following details:
      - Name: `iatd_labs_aws_vpc`
      - IPv4 CIDR: 172.16.0.0/16
      - Click "Create VPC"

2.  **Create Subnets:**
    * In the VPC Dashboard, click "Subnets" → "Create subnet"
    * Create public subnet:
      - Name: `iatd_labs_aws_subnet_public`
      - VPC: Select `iatd_labs_aws_vpc`
      - Availability Zone: Select first AZ
      - IPv4 CIDR: 172.16.1.0/24
    * Create private subnet:
      - Name: `iatd_labs_aws_subnet_private`
      - VPC: Select `iatd_labs_aws_vpc`
      - Availability Zone: Select first AZ
      - IPv4 CIDR: 172.16.2.0/24

### Part 2: Configuring Internet Access (CloudShell)

Launch AWS CloudShell and run the following commands:

1.  **Create and Attach Internet Gateway:**
    ```bash
    # Create Internet Gateway
    IGW_ID=$(aws ec2 create-internet-gateway \
      --tag-specifications 'ResourceType=internet-gateway,Tags=[{Key=Name,Value=iatd_labs_aws_igw}]' \
      --query 'InternetGateway.InternetGatewayId' \
      --output text)

    # Get VPC ID
    VPC_ID=$(aws ec2 describe-vpcs \
      --filters "Name=tag:Name,Values=iatd_labs_aws_vpc" \
      --query 'Vpcs[0].VpcId' \
      --output text)

    # Attach IGW to VPC
    aws ec2 attach-internet-gateway \
      --vpc-id $VPC_ID \
      --internet-gateway-id $IGW_ID
    ```

2.  **Create NAT Gateway (Portal):**
    * Navigate to VPC Dashboard → NAT Gateways
    * Click "Create NAT Gateway"
    * Configure:
      - Name: `iatd_labs_aws_nat`
      - Subnet: Select public subnet
      - Connectivity type: Public
      - Click "Create NAT Gateway"

### Part 3: Configuring Route Tables (Mixed)

1.  **Create Route Tables (CloudShell):**
    ```bash
    # Create public route table
    PUBLIC_RT_ID=$(aws ec2 create-route-table \
      --vpc-id $VPC_ID \
      --tag-specifications 'ResourceType=route-table,Tags=[{Key=Name,Value=iatd_labs_aws_rt_public}]' \
      --query 'RouteTable.RouteTableId' \
      --output text)

    # Create private route table
    PRIVATE_RT_ID=$(aws ec2 create-route-table \
      --vpc-id $VPC_ID \
      --tag-specifications 'ResourceType=route-table,Tags=[{Key=Name,Value=iatd_labs_aws_rt_private}]' \
      --query 'RouteTable.RouteTableId' \
      --output text)
    ```

2.  **Configure Routes (Portal):**
    * Navigate to Route Tables in VPC Dashboard
    * For public route table:
      - Add route: 0.0.0.0/0 → Internet Gateway
    * For private route table:
      - Add route: 0.0.0.0/0 → NAT Gateway

### Part 4: Creating Security Groups (Portal)

1.  **Create Security Groups:**
    * Navigate to Security Groups in VPC Dashboard
    * Create security group:
      - Name: `iatd_labs_aws_sg_basic`
      - Description: "Basic security group for IATD AWS lab"
      - VPC: Select `iatd_labs_aws_vpc`
    * Add inbound rules:
      - SSH (22): Your IP
      - ICMP: VPC CIDR

### Part 5: Launching EC2 Instances (CloudShell)

1.  **Launch EC2 Instances:**
    ```bash
    # Get subnet IDs
    PUBLIC_SUBNET_ID=$(aws ec2 describe-subnets \
      --filters "Name=tag:Name,Values=iatd_labs_aws_subnet_public" \
      --query 'Subnets[0].SubnetId' \
      --output text)

    PRIVATE_SUBNET_ID=$(aws ec2 describe-subnets \
      --filters "Name=tag:Name,Values=iatd_labs_aws_subnet_private" \
      --query 'Subnets[0].SubnetId' \
      --output text)

    # Get latest Amazon Linux 2023 AMI ID
    AMI_ID=$(aws ec2 describe-images \
      --owners amazon \
      --filters "Name=name,Values=al2023-ami-2023.*-x86_64" \
      --query 'sort_by(Images, &CreationDate)[-1].ImageId' \
      --output text)

    # Launch instances
    aws ec2 run-instances \
      --image-id $AMI_ID \
      --instance-type t2.micro \
      --subnet-id $PUBLIC_SUBNET_ID \
      --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=iatd_labs_aws_vm_public}]'

    aws ec2 run-instances \
      --image-id $AMI_ID \
      --instance-type t2.micro \
      --subnet-id $PRIVATE_SUBNET_ID \
      --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=iatd_labs_aws_vm_private}]'
    ```

### Part 6: Testing the Configuration

1.  **Test Connectivity:**
    * Connect to public instance using Session Manager
    * Test internet connectivity
    * Ping private instance
    * Connect to private instance using Session Manager
    * Test outbound internet access through NAT Gateway

### Cleanup

Clean up resources using both Portal and CloudShell:

1.  **Using Portal:**
    * Terminate EC2 instances
    * Delete NAT Gateway
    * Release Elastic IP

2.  **Using CloudShell:**
    ```bash
    # Delete route tables
    aws ec2 delete-route-table --route-table-id $PUBLIC_RT_ID
    aws ec2 delete-route-table --route-table-id $PRIVATE_RT_ID

    # Detach and delete IGW
    aws ec2 detach-internet-gateway --internet-gateway-id $IGW_ID --vpc-id $VPC_ID
    aws ec2 delete-internet-gateway --internet-gateway-id $IGW_ID

    # Delete security groups
    aws ec2 delete-security-group --group-id $SECURITY_GROUP_ID
    ```

3.  **Final Cleanup (Portal):**
    * Delete subnets
    * Delete VPC

### Key Takeaways

*   **Mixed Interface Usage:** Experience with both AWS Console and CloudShell
*   **VPC Architecture:** Understanding AWS VPC components and their relationships
*   **Network Segmentation:** Implementing public and private subnets
*   **Internet Access:** Configuring internet and NAT gateways
*   **Resource Organization:** Consistent naming and tagging practices

**Congratulations!** You have successfully completed the AWS VPC fundamentals lab using both the AWS Management Console and CloudShell.
