## IATD Microcredential Cloud Networking: Supplementary Lab - AWS Transit Gateway and Advanced Routing

**Objective:** This lab focuses on implementing advanced routing configurations in AWS using Route Tables and Transit Gateway. You'll learn to optimize traffic flow between VPCs and implement centralized network management using AWS Transit Gateway.

**Estimated Time:** 90-120 minutes

**Prerequisites:**
1. **AWS Account:** An active AWS account with administrative access
2. **AWS CLI:** Installed and configured with appropriate credentials
3. **Basic Understanding:** 
   - Amazon VPC concepts
   - Route tables and routing
   - Basic networking principles
   - AWS Transit Gateway concepts

### Resource Naming Convention

* **VPCs:**
  - Main VPC: `iatd_labs_aws_main_vpc`
  - Secondary VPC: `iatd_labs_aws_secondary_vpc`
* **Subnets:**
  - Main Public: `iatd_labs_aws_main_public`
  - Main Private: `iatd_labs_aws_main_private`
  - Secondary Public: `iatd_labs_aws_secondary_public`
  - Secondary Private: `iatd_labs_aws_secondary_private`
* **Route Tables:**
  - Main Public: `iatd_labs_aws_main_public_rt`
  - Main Private: `iatd_labs_aws_main_private_rt`
  - Secondary Public: `iatd_labs_aws_secondary_public_rt`
  - Secondary Private: `iatd_labs_aws_secondary_private_rt`
* **Transit Gateway:** `iatd_labs_aws_tgw`
* **EC2 Instances:**
  - Main VPC Instance: `iatd_labs_aws_main_instance`
  - Secondary VPC Instance: `iatd_labs_aws_secondary_instance`

### Part 1: Creating VPCs and Subnets (Portal and CloudShell)

1. **Create Main VPC:**
   ```bash
   aws ec2 create-vpc \
     --cidr-block 172.16.0.0/16 \
     --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=iatd_labs_aws_main_vpc}]'
   ```

   Expected Output:
   ```json
   {
       "Vpc": {
           "CidrBlock": "172.16.0.0/16",
           "VpcId": "vpc-0123456789abcdef0",
           "Tags": [
               {
                   "Key": "Name",
                   "Value": "iatd_labs_aws_main_vpc"
               }
           ]
       }
   }
   ```

2. **Create Secondary VPC:**
   ```bash
   aws ec2 create-vpc \
     --cidr-block 172.17.0.0/16 \
     --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=iatd_labs_aws_secondary_vpc}]'
   ```

3. **Create Subnets in Main VPC:**
   ```bash
   # Store the Main VPC ID in a variable
   MAIN_VPC_ID=$(aws ec2 describe-vpcs \
     --filters "Name=tag:Name,Values=iatd_labs_aws_main_vpc" \
     --query "Vpcs[0].VpcId" \
     --output text)
   
   # Create public subnet
   aws ec2 create-subnet \
     --vpc-id $MAIN_VPC_ID \
     --cidr-block 172.16.1.0/24 \
     --availability-zone us-west-2a \
     --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=iatd_labs_aws_main_public}]'
   ```

   Expected Output:
   ```json
   {
       "Subnet": {
           "AvailabilityZone": "us-west-2a",
           "AvailabilityZoneId": "usw2-az1",
           "CidrBlock": "172.16.1.0/24",
           "SubnetId": "subnet-0123456789abcdef0",
           "VpcId": "vpc-0123456789abcdef0",
           "Tags": [
               {
                   "Key": "Name",
                   "Value": "iatd_labs_aws_main_public"
               }
           ]
       }
   }
   ```

   ```bash
   # Create private subnet
   aws ec2 create-subnet \
     --vpc-id $MAIN_VPC_ID \
     --cidr-block 172.16.2.0/24 \
     --availability-zone us-west-2b \
     --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=iatd_labs_aws_main_private}]'
   ```

   Expected Output:
   ```json
   {
       "Subnet": {
           "AvailabilityZone": "us-west-2b",
           "AvailabilityZoneId": "usw2-az2",
           "CidrBlock": "172.16.2.0/24",
           "SubnetId": "subnet-0123456789abcdef1",
           "VpcId": "vpc-0123456789abcdef0",
           "Tags": [
               {
                   "Key": "Name",
                   "Value": "iatd_labs_aws_main_private"
               }
           ]
       }
   }
   ```

4. **Create Subnets in Secondary VPC:**
   ```bash
   # Store the Secondary VPC ID in a variable
   SECONDARY_VPC_ID=$(aws ec2 describe-vpcs \
     --filters "Name=tag:Name,Values=iatd_labs_aws_secondary_vpc" \
     --query "Vpcs[0].VpcId" \
     --output text)
   
   # Create public subnet
   aws ec2 create-subnet \
     --vpc-id $SECONDARY_VPC_ID \
     --cidr-block 172.17.1.0/24 \
     --availability-zone us-west-2a \
     --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=iatd_labs_aws_secondary_public}]'
   ```

   Expected Output:
   ```json
   {
       "Subnet": {
           "AvailabilityZone": "us-west-2a",
           "AvailabilityZoneId": "usw2-az1",
           "CidrBlock": "172.17.1.0/24",
           "SubnetId": "subnet-0123456789abcdef2",
           "VpcId": "vpc-0123456789abcdef1",
           "Tags": [
               {
                   "Key": "Name",
                   "Value": "iatd_labs_aws_secondary_public"
               }
           ]
       }
   }
   ```

   ```bash
   # Create private subnet
   aws ec2 create-subnet \
     --vpc-id $SECONDARY_VPC_ID \
     --cidr-block 172.17.2.0/24 \
     --availability-zone us-west-2b \
     --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=iatd_labs_aws_secondary_private}]'
   ```

   Expected Output:
   ```json
   {
       "Subnet": {
           "AvailabilityZone": "us-west-2b",
           "AvailabilityZoneId": "usw2-az2",
           "CidrBlock": "172.17.2.0/24",
           "SubnetId": "subnet-0123456789abcdef3",
           "VpcId": "vpc-0123456789abcdef1",
           "Tags": [
               {
                   "Key": "Name",
                   "Value": "iatd_labs_aws_secondary_private"
               }
           ]
       }
   }
   ```

### Part 2: Creating and Configuring Transit Gateway (CloudShell)

1. **Create Transit Gateway:**
   ```bash
   aws ec2 create-transit-gateway \
     --description "IATD Labs Transit Gateway" \
     --tag-specifications 'ResourceType=transit-gateway,Tags=[{Key=Name,Value=iatd_labs_aws_tgw}]'
   ```

   Expected Output:
   ```json
   {
       "TransitGateway": {
           "TransitGatewayId": "tgw-0123456789abcdef0",
           "State": "pending",
           "Tags": [
               {
                   "Key": "Name",
                   "Value": "iatd_labs_aws_tgw"
               }
           ]
       }
   }
   ```

2. **Create Transit Gateway Attachments:**
   ```bash
   # Store the Transit Gateway ID in a variable
   TGW_ID=$(aws ec2 describe-transit-gateways \
     --filters "Name=tag:Name,Values=iatd_labs_aws_tgw" \
     --query "TransitGateways[0].TransitGatewayId" \
     --output text)
   
   # Store the Main VPC private subnet ID in a variable
   MAIN_PRIVATE_SUBNET_ID=$(aws ec2 describe-subnets \
     --filters "Name=tag:Name,Values=iatd_labs_aws_main_private" \
     --query "Subnets[0].SubnetId" \
     --output text)
   
   # Attach Main VPC
   aws ec2 create-transit-gateway-vpc-attachment \
     --transit-gateway-id $TGW_ID \
     --vpc-id $MAIN_VPC_ID \
     --subnet-ids $MAIN_PRIVATE_SUBNET_ID
   ```

   Expected Output:
   ```json
   {
       "TransitGatewayVpcAttachment": {
           "TransitGatewayAttachmentId": "tgw-attach-0123456789abcdef0",
           "TransitGatewayId": "tgw-0123456789abcdef0",
           "VpcId": "vpc-0123456789abcdef0",
           "VpcOwnerId": "123456789012",
           "State": "pending",
           "SubnetIds": [
               "subnet-0123456789abcdef1"
           ],
           "CreationTime": "2023-01-01T00:00:00.000Z"
       }
   }
   ```

   ```bash
   # Store the Secondary VPC private subnet ID in a variable
   SECONDARY_PRIVATE_SUBNET_ID=$(aws ec2 describe-subnets \
     --filters "Name=tag:Name,Values=iatd_labs_aws_secondary_private" \
     --query "Subnets[0].SubnetId" \
     --output text)
   
   # Attach Secondary VPC
   aws ec2 create-transit-gateway-vpc-attachment \
     --transit-gateway-id $TGW_ID \
     --vpc-id $SECONDARY_VPC_ID \
     --subnet-ids $SECONDARY_PRIVATE_SUBNET_ID
   ```

   Expected Output:
   ```json
   {
       "TransitGatewayVpcAttachment": {
           "TransitGatewayAttachmentId": "tgw-attach-0123456789abcdef1",
           "TransitGatewayId": "tgw-0123456789abcdef0",
           "VpcId": "vpc-0123456789abcdef1",
           "VpcOwnerId": "123456789012",
           "State": "pending",
           "SubnetIds": [
               "subnet-0123456789abcdef3"
           ],
           "CreationTime": "2023-01-01T00:00:00.000Z"
       }
   }
   ```

### Part 3: Configuring Route Tables (Mixed)

1. **Create Route Tables:**
   ```bash
   # Create route table for Main VPC private subnet
   aws ec2 create-route-table \
     --vpc-id $MAIN_VPC_ID \
     --tag-specifications 'ResourceType=route-table,Tags=[{Key=Name,Value=iatd_labs_aws_main_private_rt}]'
   ```

   Expected Output:
   ```json
   {
       "RouteTable": {
           "RouteTableId": "rtb-0123456789abcdef0",
           "VpcId": "vpc-0123456789abcdef0",
           "Routes": [
               {
                   "DestinationCidrBlock": "172.16.0.0/16",
                   "GatewayId": "local",
                   "Origin": "CreateRouteTable",
                   "State": "active"
               }
           ],
           "Tags": [
               {
                   "Key": "Name",
                   "Value": "iatd_labs_aws_main_private_rt"
               }
           ]
       }
   }
   ```

   ```bash
   # Store the Main VPC route table ID in a variable
   MAIN_RT_ID=$(aws ec2 describe-route-tables \
     --filters "Name=tag:Name,Values=iatd_labs_aws_main_private_rt" \
     --query "RouteTables[0].RouteTableId" \
     --output text)
   
   # Create route table for Secondary VPC private subnet
   aws ec2 create-route-table \
     --vpc-id $SECONDARY_VPC_ID \
     --tag-specifications 'ResourceType=route-table,Tags=[{Key=Name,Value=iatd_labs_aws_secondary_private_rt}]'
   ```

   Expected Output:
   ```json
   {
       "RouteTable": {
           "RouteTableId": "rtb-0123456789abcdef1",
           "VpcId": "vpc-0123456789abcdef1",
           "Routes": [
               {
                   "DestinationCidrBlock": "172.17.0.0/16",
                   "GatewayId": "local",
                   "Origin": "CreateRouteTable",
                   "State": "active"
               }
           ],
           "Tags": [
               {
                   "Key": "Name",
                   "Value": "iatd_labs_aws_secondary_private_rt"
               }
           ]
       }
   }
   ```

   ```bash
   # Store the Secondary VPC route table ID in a variable
   SECONDARY_RT_ID=$(aws ec2 describe-route-tables \
     --filters "Name=tag:Name,Values=iatd_labs_aws_secondary_private_rt" \
     --query "RouteTables[0].RouteTableId" \
     --output text)
   ```

2. **Add Routes:**
   ```bash
   # Add route in Main VPC to Secondary VPC via Transit Gateway
   aws ec2 create-route \
     --route-table-id $MAIN_RT_ID \
     --destination-cidr-block 172.17.0.0/16 \
     --transit-gateway-id $TGW_ID
   ```

   Expected Output:
   ```json
   {
       "Return": true
   }
   ```

   ```bash
   # Add route in Secondary VPC to Main VPC via Transit Gateway
   aws ec2 create-route \
     --route-table-id $SECONDARY_RT_ID \
     --destination-cidr-block 172.16.0.0/16 \
     --transit-gateway-id $TGW_ID
   ```

   Expected Output:
   ```json
   {
       "Return": true
   }
   ```

### Part 4: Security Configuration

1. **Create IAM Role for EC2 Instances:**
   ```bash
   # Create IAM role
   aws iam create-role \
     --role-name iatd_labs_aws_ec2_role \
     --assume-role-policy-document '{"Version":"2012-10-17","Statement":[{"Effect":"Allow","Principal":{"Service":"ec2.amazonaws.com"},"Action":"sts:AssumeRole"}]}'

   # Attach AWSSystemsManagerManagedInstanceCore policy
   aws iam attach-role-policy \
     --role-name iatd_labs_aws_ec2_role \
     --policy-arn arn:aws:iam::aws:policy/AWSSystemsManagerManagedInstanceCore

   # Create instance profile and add role
   aws iam create-instance-profile \
     --instance-profile-name iatd_labs_aws_ec2_profile

   aws iam add-role-to-instance-profile \
     --role-name iatd_labs_aws_ec2_role \
     --instance-profile-name iatd_labs_aws_ec2_profile
   ```

2. **Configure Security Groups:**
   ```bash
   # Create security group for instances
   aws ec2 create-security-group \
     --group-name iatd_labs_aws_sg \
     --description "Security group for IATD Labs instances" \
     --vpc-id $MAIN_VPC_ID

   # Allow ICMP traffic between instances
   aws ec2 authorize-security-group-ingress \
     --group-id <SECURITY_GROUP_ID> \
     --protocol icmp \
     --port -1 \
     --source-group <SECURITY_GROUP_ID>
   ```

### Part 5: Testing and Verification

1. **Launch Test EC2 Instances:**
   ```bash
   # Launch instance in Main VPC
   aws ec2 run-instances \
     --image-id ami-0123456789abcdef0 \
     --instance-type t2.micro \
     --subnet-id <MAIN_PRIVATE_SUBNET_ID> \
     --security-group-ids <SECURITY_GROUP_ID> \
     --iam-instance-profile Name=iatd_labs_aws_ec2_profile \
     --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=iatd_labs_aws_main_instance}]'

   # Launch instance in Secondary VPC
   aws ec2 run-instances \
     --image-id ami-0123456789abcdef0 \
     --instance-type t2.micro \
     --subnet-id <SECONDARY_PRIVATE_SUBNET_ID> \
     --security-group-ids <SECURITY_GROUP_ID> \
     --iam-instance-profile Name=iatd_labs_aws_ec2_profile \
     --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=iatd_labs_aws_secondary_instance}]'
   ```

2. **Verify Connectivity:**
   ```bash
   # Test connectivity from Main VPC instance to Secondary VPC instance
   aws ssm send-command \
     --document-name "AWS-RunShellScript" \
     --targets "Key=instanceids,Values=<MAIN_INSTANCE_ID>" \
     --parameters "commands=['ping -c 4 <SECONDARY_INSTANCE_PRIVATE_IP>']"
   ```

   Expected Output:
   ```json
   {
       "Command": {
           "CommandId": "01234567-89ab-cdef-0123-456789abcdef",
           "Status": "Success",
           "ResponseCode": 0,
           "Output": "PING 172.17.2.10 (172.17.2.10) 56(84) bytes of data.\n64 bytes from 172.17.2.10: icmp_seq=1 ttl=254 time=1.23 ms\n64 bytes from 172.17.2.10: icmp_seq=2 ttl=254 time=1.15 ms\n64 bytes from 172.17.2.10: icmp_seq=3 ttl=254 time=1.18 ms\n64 bytes from 172.17.2.10: icmp_seq=4 ttl=254 time=1.20 ms"
       }
   }
   ```

3. **Verify Route Tables:**
   ```bash
   # Check effective routes for Main VPC instance
   aws ec2 describe-route-tables \
     --route-table-ids <MAIN_RT_ID> \
     --query 'RouteTables[].Routes[]'
   ```

   Expected Output:
   ```json
   [
       {
           "DestinationCidrBlock": "172.16.0.0/16",
           "GatewayId": "local",
           "Origin": "CreateRouteTable",
           "State": "active"
       },
       {
           "DestinationCidrBlock": "172.17.0.0/16",
           "TransitGatewayId": "tgw-0123456789abcdef0",
           "Origin": "CreateRoute",
           "State": "active"
       }
   ]
   ```

4. **Verify Transit Gateway Attachments:**
   ```bash
   aws ec2 describe-transit-gateway-attachments \
     --filters "Name=transit-gateway-id,Values=<TGW_ID>"
   ```

   Expected Output:
   ```json
   {
       "TransitGatewayAttachments": [
           {
               "TransitGatewayAttachmentId": "tgw-attach-0123456789abcdef0",
               "TransitGatewayId": "tgw-0123456789abcdef0",
               "VpcId": "vpc-0123456789abcdef0",
               "State": "available"
           },
           {
               "TransitGatewayAttachmentId": "tgw-attach-0123456789abcdef1",
               "TransitGatewayId": "tgw-0123456789abcdef0",
               "VpcId": "vpc-0123456789abcdef1",
               "State": "available"
           }
       ]
   }
   ```

### Post-Lab: Cleanup

To avoid unnecessary costs, clean up the resources using any of these methods:

1. **AWS Management Console:**
   * Navigate to the EC2 Dashboard
   * Delete resources in this order:
     1. EC2 instances
     2. Transit Gateway attachments
     3. Transit Gateway
     4. Security groups
     5. Route tables
     6. Subnets
     7. VPCs
   * Navigate to IAM
   * Delete resources in this order:
     1. Remove role from instance profile
     2. Delete instance profile
     3. Detach policies from role
     4. Delete role

2. **AWS CLI:**
   ```bash
   # Store resource IDs in variables
   MAIN_INSTANCE_ID=$(aws ec2 describe-instances \
     --filters "Name=tag:Name,Values=iatd_labs_aws_main_vm" \
     --query "Reservations[0].Instances[0].InstanceId" \
     --output text)
   
   SECONDARY_INSTANCE_ID=$(aws ec2 describe-instances \
     --filters "Name=tag:Name,Values=iatd_labs_aws_secondary_vm" \
     --query "Reservations[0].Instances[0].InstanceId" \
     --output text)
   
   MAIN_TGW_ATTACHMENT_ID=$(aws ec2 describe-transit-gateway-attachments \
     --filters "Name=vpc-id,Values=$MAIN_VPC_ID" \
     --query "TransitGatewayAttachments[0].TransitGatewayAttachmentId" \
     --output text)
   
   SECONDARY_TGW_ATTACHMENT_ID=$(aws ec2 describe-transit-gateway-attachments \
     --filters "Name=vpc-id,Values=$SECONDARY_VPC_ID" \
     --query "TransitGatewayAttachments[0].TransitGatewayAttachmentId" \
     --output text)
   
   SECURITY_GROUP_ID=$(aws ec2 describe-security-groups \
     --filters "Name=group-name,Values=iatd_labs_aws_sg" \
     --query "SecurityGroups[0].GroupId" \
     --output text)
   
   # Terminate EC2 instances
   aws ec2 terminate-instances --instance-ids $MAIN_INSTANCE_ID $SECONDARY_INSTANCE_ID

   # Wait for instances to terminate
   aws ec2 wait instance-terminated --instance-ids $MAIN_INSTANCE_ID $SECONDARY_INSTANCE_ID

   # Delete Transit Gateway attachments
   aws ec2 delete-transit-gateway-vpc-attachment --transit-gateway-attachment-id $MAIN_TGW_ATTACHMENT_ID
   aws ec2 delete-transit-gateway-vpc-attachment --transit-gateway-attachment-id $SECONDARY_TGW_ATTACHMENT_ID

   # Wait for attachments to delete
   aws ec2 wait transit-gateway-attachment-deleted --transit-gateway-attachment-ids $MAIN_TGW_ATTACHMENT_ID $SECONDARY_TGW_ATTACHMENT_ID

   # Delete Transit Gateway
   aws ec2 delete-transit-gateway --transit-gateway-id $TGW_ID

   # Delete security group
   aws ec2 delete-security-group --group-id $SECURITY_GROUP_ID

   # Delete route tables
   aws ec2 delete-route-table --route-table-id $MAIN_RT_ID
   aws ec2 delete-route-table --route-table-id $SECONDARY_RT_ID

   # Delete subnets
   aws ec2 delete-subnet --subnet-id $MAIN_PRIVATE_SUBNET_ID
   aws ec2 delete-subnet --subnet-id $MAIN_PUBLIC_SUBNET_ID
   aws ec2 delete-subnet --subnet-id $SECONDARY_PRIVATE_SUBNET_ID
   aws ec2 delete-subnet --subnet-id $SECONDARY_PUBLIC_SUBNET_ID

   # Delete VPCs
   aws ec2 delete-vpc --vpc-id $MAIN_VPC_ID
   aws ec2 delete-vpc --vpc-id $SECONDARY_VPC_ID

   # Clean up IAM resources
   aws iam remove-role-from-instance-profile \
     --instance-profile-name iatd_labs_aws_ec2_profile \
     --role-name iatd_labs_aws_ec2_role

   aws iam delete-instance-profile \
     --instance-profile-name iatd_labs_aws_ec2_profile

   aws iam detach-role-policy \
     --role-name iatd_labs_aws_ec2_role \
     --policy-arn arn:aws:iam::aws:policy/AWSSystemsManagerManagedInstanceCore

   aws iam delete-role \
     --role-name iatd_labs_aws_ec2_role
   ```

   Expected Output (for terminate-instances):
   ```json
   {
       "TerminatingInstances": [
           {
               "CurrentState": {
                   "Code": 32,
                   "Name": "shutting-down"
               },
               "InstanceId": "i-0123456789abcdef0",
               "PreviousState": {
                   "Code": 16,
                   "Name": "running"
               }
           },
           {
               "CurrentState": {
                   "Code": 32,
                   "Name": "shutting-down"
               },
               "InstanceId": "i-0123456789abcdef1",
               "PreviousState": {
                   "Code": 16,
                   "Name": "running"
               }
           }
       ]
   }
   ```

### Post-Lab Summary

**Key Takeaways:**
* AWS Transit Gateway provides centralized routing between VPCs
* Route tables control traffic flow within and between VPCs
* AWS CLI provides powerful automation capabilities
* Transit Gateway simplifies network architecture

**Next Steps:**
* Explore Transit Gateway across multiple regions
* Implement more complex routing scenarios
* Study Transit Gateway Network Manager
* Learn about VPC sharing and AWS RAM

**Additional Resources:**
* [AWS Transit Gateway Documentation](https://docs.aws.amazon.com/vpc/latest/tgw/what-is-transit-gateway.html)
* [VPC Route Tables](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Route_Tables.html)
* [AWS Networking Best Practices](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-network-security.html)
* [Transit Gateway Design Patterns](https://docs.aws.amazon.com/vpc/latest/tgw/tgw-best-practices.html)

**Congratulations!** You've successfully implemented advanced routing in AWS using Transit Gateway and route tables. This lab has demonstrated key concepts in AWS networking that parallel the routing and traffic optimization scenarios covered in the Azure labs.

### Troubleshooting Guide

1. **Transit Gateway Issues:**
   * Verify Transit Gateway state is 'available'
   * Check Transit Gateway attachments are 'available'
   * Ensure route tables are properly associated
   * Verify route propagation settings

2. **VPC and Subnet Issues:**
   * Confirm CIDR blocks don't overlap
   * Check subnet associations with route tables
   * Verify Internet Gateway is attached (for public subnets)
   * Ensure proper NACL and Security Group rules

3. **EC2 Instance Connectivity:**
   * Verify Security Groups allow ICMP traffic
   * Check Systems Manager Agent is running
   * Ensure instances have proper IAM roles
   * Validate route table entries

4. **Common Mistakes:**
   * Missing or incorrect route table entries
   * Security Groups blocking traffic
   * Incorrect CIDR ranges
   * Transit Gateway attachment in wrong subnet
   * Missing IAM permissions

5. **AWS CLI Issues:**
   * Verify AWS CLI configuration
   * Check IAM permissions
   * Ensure correct region is set
   * Validate resource IDs

### Post-Lab: Cleanup

To avoid unnecessary costs, clean up the resources using any of these methods:

1. **AWS Management Console:**
   * Navigate to the VPC Dashboard
   * Delete resources in this order:
     1. EC2 instances
     2. Transit Gateway attachments
     3. Transit Gateway
     4. Route tables
     5. Subnets
     6. VPCs

2. **AWS CLI:**
   ```bash
   # Terminate EC2 instances
   aws ec2 terminate-instances --instance-ids $MAIN_INSTANCE_ID $SECONDARY_INSTANCE_ID

   # Delete Transit Gateway attachments
   aws ec2 delete-transit-gateway-vpc-attachment --transit-gateway-attachment-id $MAIN_TGW_ATTACHMENT_ID
   aws ec2 delete-transit-gateway-vpc-attachment --transit-gateway-attachment-id $SECONDARY_TGW_ATTACHMENT_ID

   # Delete Transit Gateway
   aws ec2 delete-transit-gateway --transit-gateway-id $TGW_ID

   # Delete VPCs (this will also delete associated resources)
   aws ec2 delete-vpc --vpc-id $MAIN_VPC_ID
   aws ec2 delete-vpc --vpc-id $SECONDARY_VPC_ID

   # Clean up IAM resources
   aws iam remove-role-from-instance-profile \
     --instance-profile-name iatd_labs_aws_ec2_profile \
     --role-name iatd_labs_aws_ec2_role

   aws iam delete-instance-profile \
     --instance-profile-name iatd_labs_aws_ec2_profile

   aws iam detach-role-policy \
     --role-name iatd_labs_aws_ec2_role \
     --policy-arn arn:aws:iam::aws:policy/AWSSystemsManagerManagedInstanceCore

   aws iam delete-role \
     --role-name iatd_labs_aws_ec2_role
   ```

### Post-Lab Summary

**Key Takeaways:**
* AWS Transit Gateway provides centralized routing between VPCs
* Route tables control traffic flow within and between VPCs
* AWS CLI provides powerful automation capabilities
* Transit Gateway simplifies network architecture

**Next Steps:**
* Explore Transit Gateway across multiple regions
* Implement more complex routing scenarios
* Study Transit Gateway Network Manager
* Learn about VPC sharing and AWS RAM

**Additional Resources:**
* [AWS Transit Gateway Documentation](https://docs.aws.amazon.com/vpc/latest/tgw/what-is-transit-gateway.html)
* [VPC Route Tables](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Route_Tables.html)
* [AWS Networking Best Practices](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-network-security.html)
* [Transit Gateway Design Patterns](https://docs.aws.amazon.com/vpc/latest/tgw/tgw-best-practices.html)

**Congratulations!** You've successfully implemented advanced routing in AWS using Transit Gateway and route tables. This lab has demonstrated key concepts in AWS networking that parallel the routing and traffic optimization scenarios covered in the Azure labs.
