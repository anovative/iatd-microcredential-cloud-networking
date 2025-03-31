## IATD Microcredential Cloud Networking: Supplementary Lab - AWS Security Groups and Network ACLs

**Objective:** This lab focuses on implementing network security in AWS using Security Groups and Network ACLs. You'll learn how these security mechanisms compare to Azure's NSGs and ASGs, and how to implement layered security in AWS environments.

**Estimated Time:** 90-120 minutes

**Prerequisites:**
1. **AWS Account:** An active AWS account with administrative access
2. **AWS CLI:** Installed and configured with appropriate credentials
3. **Basic Understanding:** 
   - Amazon VPC concepts
   - Security Groups and Network ACLs
   - Basic networking principles
   - EC2 instance management

### Resource Naming Convention

* **VPCs:**
  - Main VPC: `iatd_labs_aws_sec_vpc`
* **Subnets:**
  - Web Subnet: `iatd_labs_aws_web_subnet`
  - App Subnet: `iatd_labs_aws_app_subnet`
* **Security Groups:**
  - Web Tier: `iatd_labs_aws_web_sg`
  - App Tier: `iatd_labs_aws_app_sg`
* **Network ACLs:**
  - Web Tier: `iatd_labs_aws_web_nacl`
  - App Tier: `iatd_labs_aws_app_nacl`
* **EC2 Instances:**
  - Web Server: `iatd_labs_aws_web_instance`
  - App Server: `iatd_labs_aws_app_instance`

### Part 1: Creating VPC and Subnets (Portal and CloudShell)

1. **Create VPC:**
   ```bash
   aws ec2 create-vpc \
     --cidr-block 172.16.0.0/16 \
     --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=iatd_labs_aws_sec_vpc}]'
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
                   "Value": "iatd_labs_aws_sec_vpc"
               }
           ]
       }
   }
   ```

2. **Create Subnets:**
   ```bash
   # Store the VPC ID in a variable
   VPC_ID=$(aws ec2 describe-vpcs \
     --filters "Name=tag:Name,Values=iatd_labs_aws_sec_vpc" \
     --query "Vpcs[0].VpcId" \
     --output text)
   
   # Create web subnet
   aws ec2 create-subnet \
     --vpc-id $VPC_ID \
     --cidr-block 172.16.1.0/24 \
     --availability-zone us-west-2a \
     --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=iatd_labs_aws_web_subnet}]'
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
                   "Value": "iatd_labs_aws_web_subnet"
               }
           ]
       }
   }
   ```

   ```bash
   # Create app subnet
   aws ec2 create-subnet \
     --vpc-id $VPC_ID \
     --cidr-block 172.16.2.0/24 \
     --availability-zone us-west-2b \
     --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=iatd_labs_aws_app_subnet}]'
   ```

### Part 2: Creating and Configuring Security Groups (CloudShell)

1. **Create Web Tier Security Group:**
   ```bash
   aws ec2 create-security-group \
     --group-name iatd_labs_aws_web_sg \
     --description "Security group for web tier" \
     --vpc-id $VPC_ID \
     --tag-specifications 'ResourceType=security-group,Tags=[{Key=Name,Value=iatd_labs_aws_web_sg}]'
   ```

   Expected Output:
   ```json
   {
       "GroupId": "sg-0123456789abcdef0"
   }
   ```

   ```bash
   # Store the security group ID
   WEB_SG_ID=$(aws ec2 describe-security-groups \
     --filters "Name=tag:Name,Values=iatd_labs_aws_web_sg" \
     --query "SecurityGroups[0].GroupId" \
     --output text)
   
   # Add inbound rules for HTTP and SSH
   aws ec2 authorize-security-group-ingress \
     --group-id $WEB_SG_ID \
     --protocol tcp \
     --port 80 \
     --cidr 0.0.0.0/0
   
   aws ec2 authorize-security-group-ingress \
     --group-id $WEB_SG_ID \
     --protocol tcp \
     --port 22 \
     --cidr 0.0.0.0/0
   ```

2. **Create App Tier Security Group:**
   ```bash
   aws ec2 create-security-group \
     --group-name iatd_labs_aws_app_sg \
     --description "Security group for app tier" \
     --vpc-id $VPC_ID \
     --tag-specifications 'ResourceType=security-group,Tags=[{Key=Name,Value=iatd_labs_aws_app_sg}]'
   ```

   ```bash
   # Store the security group ID
   APP_SG_ID=$(aws ec2 describe-security-groups \
     --filters "Name=tag:Name,Values=iatd_labs_aws_app_sg" \
     --query "SecurityGroups[0].GroupId" \
     --output text)
   
   # Add inbound rule to allow traffic only from web tier on port 8080
   aws ec2 authorize-security-group-ingress \
     --group-id $APP_SG_ID \
     --protocol tcp \
     --port 8080 \
     --source-group $WEB_SG_ID
   
   # Add SSH access
   aws ec2 authorize-security-group-ingress \
     --group-id $APP_SG_ID \
     --protocol tcp \
     --port 22 \
     --cidr 0.0.0.0/0
   ```

### Part 3: Creating and Configuring Network ACLs (CloudShell)

1. **Create Web Tier Network ACL:**
   ```bash
   aws ec2 create-network-acl \
     --vpc-id $VPC_ID \
     --tag-specifications 'ResourceType=network-acl,Tags=[{Key=Name,Value=iatd_labs_aws_web_nacl}]'
   ```

   Expected Output:
   ```json
   {
       "NetworkAcl": {
           "NetworkAclId": "acl-0123456789abcdef0",
           "VpcId": "vpc-0123456789abcdef0",
           "Tags": [
               {
                   "Key": "Name",
                   "Value": "iatd_labs_aws_web_nacl"
               }
           ]
       }
   }
   ```

   ```bash
   # Store the NACL ID
   WEB_NACL_ID=$(aws ec2 describe-network-acls \
     --filters "Name=tag:Name,Values=iatd_labs_aws_web_nacl" \
     --query "NetworkAcls[0].NetworkAclId" \
     --output text)
   
   # Add inbound rules
   aws ec2 create-network-acl-entry \
     --network-acl-id $WEB_NACL_ID \
     --ingress \
     --rule-number 100 \
     --protocol tcp \
     --port-range From=80,To=80 \
     --cidr-block 0.0.0.0/0 \
     --rule-action allow
   
   aws ec2 create-network-acl-entry \
     --network-acl-id $WEB_NACL_ID \
     --ingress \
     --rule-number 110 \
     --protocol tcp \
     --port-range From=22,To=22 \
     --cidr-block 0.0.0.0/0 \
     --rule-action allow
   
   # Add outbound rules
   aws ec2 create-network-acl-entry \
     --network-acl-id $WEB_NACL_ID \
     --egress \
     --rule-number 100 \
     --protocol tcp \
     --port-range From=1024,To=65535 \
     --cidr-block 0.0.0.0/0 \
     --rule-action allow
   ```

2. **Create App Tier Network ACL:**
   ```bash
   aws ec2 create-network-acl \
     --vpc-id $VPC_ID \
     --tag-specifications 'ResourceType=network-acl,Tags=[{Key=Name,Value=iatd_labs_aws_app_nacl}]'
   ```

   ```bash
   # Store the NACL ID
   APP_NACL_ID=$(aws ec2 describe-network-acls \
     --filters "Name=tag:Name,Values=iatd_labs_aws_app_nacl" \
     --query "NetworkAcls[0].NetworkAclId" \
     --output text)
   
   # Add inbound rules
   aws ec2 create-network-acl-entry \
     --network-acl-id $APP_NACL_ID \
     --ingress \
     --rule-number 100 \
     --protocol tcp \
     --port-range From=8080,To=8080 \
     --cidr-block 172.16.1.0/24 \
     --rule-action allow
   
   aws ec2 create-network-acl-entry \
     --network-acl-id $APP_NACL_ID \
     --ingress \
     --rule-number 110 \
     --protocol tcp \
     --port-range From=22,To=22 \
     --cidr-block 0.0.0.0/0 \
     --rule-action allow
   
   # Add outbound rules
   aws ec2 create-network-acl-entry \
     --network-acl-id $APP_NACL_ID \
     --egress \
     --rule-number 100 \
     --protocol tcp \
     --port-range From=1024,To=65535 \
     --cidr-block 0.0.0.0/0 \
     --rule-action allow
   ```

### Part 4: Testing and Verification

1. **Launch EC2 Instances:**
   ```bash
   # Get subnet IDs
   WEB_SUBNET_ID=$(aws ec2 describe-subnets \
     --filters "Name=tag:Name,Values=iatd_labs_aws_web_subnet" \
     --query "Subnets[0].SubnetId" \
     --output text)
   
   APP_SUBNET_ID=$(aws ec2 describe-subnets \
     --filters "Name=tag:Name,Values=iatd_labs_aws_app_subnet" \
     --query "Subnets[0].SubnetId" \
     --output text)
   
   # Launch web instance
   aws ec2 run-instances \
     --image-id ami-0c55b159cbfafe1f0 \
     --instance-type t2.micro \
     --subnet-id $WEB_SUBNET_ID \
     --security-group-ids $WEB_SG_ID \
     --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=iatd_labs_aws_web_instance}]' \
     --user-data file://web-user-data.sh
   ```

   ```bash
   # Launch app instance
   aws ec2 run-instances \
     --image-id ami-0c55b159cbfafe1f0 \
     --instance-type t2.micro \
     --subnet-id $APP_SUBNET_ID \
     --security-group-ids $APP_SG_ID \
     --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=iatd_labs_aws_app_instance}]' \
     --user-data file://app-user-data.sh
   ```

2. **Verify Security Group Rules:**
   ```bash
   # Verify web security group
   aws ec2 describe-security-groups \
     --group-ids $WEB_SG_ID
   
   # Verify app security group
   aws ec2 describe-security-groups \
     --group-ids $APP_SG_ID
   ```

3. **Test Connectivity:**
   ```bash
   # Get instance IPs
   WEB_INSTANCE_IP=$(aws ec2 describe-instances \
     --filters "Name=tag:Name,Values=iatd_labs_aws_web_instance" \
     --query "Reservations[0].Instances[0].PublicIpAddress" \
     --output text)
   
   APP_INSTANCE_IP=$(aws ec2 describe-instances \
     --filters "Name=tag:Name,Values=iatd_labs_aws_app_instance" \
     --query "Reservations[0].Instances[0].PrivateIpAddress" \
     --output text)
   
   # Test HTTP access to web instance
   curl http://$WEB_INSTANCE_IP
   
   # SSH to web instance and test connection to app instance
   ssh -i your-key.pem ec2-user@$WEB_INSTANCE_IP
   curl http://$APP_INSTANCE_IP:8080
   ```

### Post-Lab: Cleanup

1. **Terminate EC2 Instances:**
   ```bash
   # Get instance IDs
   WEB_INSTANCE_ID=$(aws ec2 describe-instances \
     --filters "Name=tag:Name,Values=iatd_labs_aws_web_instance" \
     --query "Reservations[0].Instances[0].InstanceId" \
     --output text)
   
   APP_INSTANCE_ID=$(aws ec2 describe-instances \
     --filters "Name=tag:Name,Values=iatd_labs_aws_app_instance" \
     --query "Reservations[0].Instances[0].InstanceId" \
     --output text)
   
   # Terminate instances
   aws ec2 terminate-instances \
     --instance-ids $WEB_INSTANCE_ID $APP_INSTANCE_ID
   ```

2. **Delete Security Groups:**
   ```bash
   aws ec2 delete-security-group --group-id $WEB_SG_ID
   aws ec2 delete-security-group --group-id $APP_SG_ID
   ```

3. **Delete Network ACLs:**
   ```bash
   aws ec2 delete-network-acl --network-acl-id $WEB_NACL_ID
   aws ec2 delete-network-acl --network-acl-id $APP_NACL_ID
   ```

4. **Delete Subnets:**
   ```bash
   aws ec2 delete-subnet --subnet-id $WEB_SUBNET_ID
   aws ec2 delete-subnet --subnet-id $APP_SUBNET_ID
   ```

5. **Delete VPC:**
   ```bash
   aws ec2 delete-vpc --vpc-id $VPC_ID
   ```

**Congratulations!** You've successfully implemented network security in AWS using Security Groups and Network ACLs. This lab has demonstrated key concepts in AWS network security that parallel the NSG and ASG implementations covered in the Azure labs.

### Troubleshooting Guide

1. **Security Group Issues:**
   - Verify inbound and outbound rules are correctly configured
   - Check that security groups are associated with the correct instances
   - Remember that security groups are stateful (return traffic is automatically allowed)

2. **Network ACL Issues:**
   - Verify both inbound and outbound rules (NACLs are stateless)
   - Check rule numbers and precedence (lower numbers are evaluated first)
   - Ensure NACLs are associated with the correct subnets

3. **Connectivity Issues:**
   - Check instance status (running, stopped, etc.)
   - Verify network interfaces are correctly configured
   - Test connectivity using appropriate tools (ping, telnet, curl)

4. **Common Mistakes:**
   - Forgetting to allow outbound traffic in Network ACLs
   - Misconfiguring CIDR blocks in security rules
   - Using incorrect port numbers
   - Not allowing ephemeral ports for return traffic in NACLs
