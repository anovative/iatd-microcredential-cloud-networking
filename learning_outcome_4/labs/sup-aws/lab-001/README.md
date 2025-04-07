## IATD Microcredential Cloud Networking: Supplementary Lab - AWS Transit Gateway and PrivateLink

**Objective:** This lab focuses on implementing secure network connectivity in AWS using Transit Gateway and AWS PrivateLink. You'll learn how these services compare to Azure's Private Link and Virtual Network concepts, while implementing secure network isolation patterns.

**Estimated Time:** 90-120 minutes

**Prerequisites:**
1. **AWS Account:** An active AWS account with appropriate permissions
2. **AWS CLI:** Installed and configured with appropriate credentials
3. **Basic Understanding:** 
   - AWS VPC networking concepts
   - Security Groups and NACLs
   - Basic networking principles
   - EC2 instance management

### Resource Naming Convention

* **VPCs:**
  - Hub VPC: `iatd-labs-hub-vpc`
  - Spoke VPC 1: `iatd-labs-spoke1-vpc`
  - Spoke VPC 2: `iatd-labs-spoke2-vpc`
* **Subnets:**
  - Hub Subnet: `iatd-labs-hub-subnet`
  - Spoke1 App Subnet: `iatd-labs-spoke1-app-subnet`
  - Spoke2 App Subnet: `iatd-labs-spoke2-app-subnet`
* **Security Groups:**
  - Hub SG: `iatd-labs-hub-sg`
  - Spoke1 SG: `iatd-labs-spoke1-sg`
  - Spoke2 SG: `iatd-labs-spoke2-sg`
* **Transit Gateway:**
  - TGW: `iatd-labs-tgw`
* **EC2 Instances:**
  - Hub Server: `iatd-labs-hub-server`
  - Spoke1 Server: `iatd-labs-spoke1-server`
  - Spoke2 Server: `iatd-labs-spoke2-server`

### Let's Get Started!

### Part 1: Setting Up the Hub VPC (AWS CLI)

1. **Create the Hub VPC:**
   ```bash
   # Set environment variables
   export AWS_REGION="us-east-1"
   export HUB_VPC_CIDR="172.16.0.0/16"
   export HUB_SUBNET_CIDR="172.16.1.0/24"

   # Create VPC
   HUB_VPC_ID=$(aws ec2 create-vpc \
     --cidr-block $HUB_VPC_CIDR \
     --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=iatd-labs-hub-vpc}]' \
     --query 'Vpc.VpcId' \
     --output text)

   # Create subnet
   HUB_SUBNET_ID=$(aws ec2 create-subnet \
     --vpc-id $HUB_VPC_ID \
     --cidr-block $HUB_SUBNET_CIDR \
     --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=iatd-labs-hub-subnet}]' \
     --query 'Subnet.SubnetId' \
     --output text)
   ```

   Expected Output:
   ```
   vpc-0123456789abcdef0
   subnet-0123456789abcdef0
   ```

2. **Create Security Group for Hub VPC:**
   ```bash
   # Create security group
   HUB_SG_ID=$(aws ec2 create-security-group \
     --group-name iatd-labs-hub-sg \
     --description "Security group for hub VPC" \
     --vpc-id $HUB_VPC_ID \
     --query 'GroupId' \
     --output text)

   # Add inbound rules
   aws ec2 authorize-security-group-ingress \
     --group-id $HUB_SG_ID \
     --protocol tcp \
     --port 22 \
     --cidr 0.0.0.0/0
   ```

   Expected Output:
   ```
   sg-0123456789abcdef0
   {
       "Return": true,
       "SecurityGroupRules": [...]
   }
   ```

### Part 2: Creating Spoke VPCs

1. **Create Spoke VPC 1:**
   ```bash
   # Set environment variables for Spoke1
   export SPOKE1_VPC_CIDR="172.17.0.0/16"
   export SPOKE1_SUBNET_CIDR="172.17.1.0/24"

   # Create VPC
   SPOKE1_VPC_ID=$(aws ec2 create-vpc \
     --cidr-block $SPOKE1_VPC_CIDR \
     --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=iatd-labs-spoke1-vpc}]' \
     --query 'Vpc.VpcId' \
     --output text)

   # Create subnet
   SPOKE1_SUBNET_ID=$(aws ec2 create-subnet \
     --vpc-id $SPOKE1_VPC_ID \
     --cidr-block $SPOKE1_SUBNET_CIDR \
     --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=iatd-labs-spoke1-app-subnet}]' \
     --query 'Subnet.SubnetId' \
     --output text)
   ```

   Expected Output:
   ```
   vpc-0123456789abcdef1
   subnet-0123456789abcdef1
   ```

2. **Create Spoke VPC 2:**
   ```bash
   # Set environment variables for Spoke2
   export SPOKE2_VPC_CIDR="172.18.0.0/16"
   export SPOKE2_SUBNET_CIDR="172.18.1.0/24"

   # Create VPC
   SPOKE2_VPC_ID=$(aws ec2 create-vpc \
     --cidr-block $SPOKE2_VPC_CIDR \
     --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=iatd-labs-spoke2-vpc}]' \
     --query 'Vpc.VpcId' \
     --output text)

   # Create subnet
   SPOKE2_SUBNET_ID=$(aws ec2 create-subnet \
     --vpc-id $SPOKE2_VPC_ID \
     --cidr-block $SPOKE2_SUBNET_CIDR \
     --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=iatd-labs-spoke2-app-subnet}]' \
     --query 'Subnet.SubnetId' \
     --output text)
   ```

   Expected Output:
   ```
   vpc-0123456789abcdef2
   subnet-0123456789abcdef2
   ```

### Part 3: Setting Up Transit Gateway

1. **Create Transit Gateway:**
   ```bash
   # Create Transit Gateway
   TGW_ID=$(aws ec2 create-transit-gateway \
     --description "IATD Labs Transit Gateway" \
     --tag-specifications 'ResourceType=transit-gateway,Tags=[{Key=Name,Value=iatd-labs-tgw}]' \
     --query 'TransitGateway.TransitGatewayId' \
     --output text)

   # Wait for Transit Gateway to be available
   aws ec2 wait transit-gateway-available --transit-gateway-ids $TGW_ID
   ```

   Expected Output:
   ```
   tgw-0123456789abcdef0
   ```

2. **Attach VPCs to Transit Gateway:**
   ```bash
   # Attach Hub VPC
   aws ec2 create-transit-gateway-vpc-attachment \
     --transit-gateway-id $TGW_ID \
     --vpc-id $HUB_VPC_ID \
     --subnet-ids $HUB_SUBNET_ID \
     --tag-specifications 'ResourceType=transit-gateway-attachment,Tags=[{Key=Name,Value=iatd-labs-hub-attachment}]'

   # Attach Spoke VPCs
   aws ec2 create-transit-gateway-vpc-attachment \
     --transit-gateway-id $TGW_ID \
     --vpc-id $SPOKE1_VPC_ID \
     --subnet-ids $SPOKE1_SUBNET_ID \
     --tag-specifications 'ResourceType=transit-gateway-attachment,Tags=[{Key=Name,Value=iatd-labs-spoke1-attachment}]'

   aws ec2 create-transit-gateway-vpc-attachment \
     --transit-gateway-id $TGW_ID \
     --vpc-id $SPOKE2_VPC_ID \
     --subnet-ids $SPOKE2_SUBNET_ID \
     --tag-specifications 'ResourceType=transit-gateway-attachment,Tags=[{Key=Name,Value=iatd-labs-spoke2-attachment}]'
   ```

### Part 4: Implementing AWS PrivateLink

1. **Create VPC Endpoint Service in Hub VPC:**
   ```bash
   # Create Network Load Balancer
   NLB_ARN=$(aws elbv2 create-load-balancer \
     --name iatd-labs-hub-nlb \
     --type network \
     --subnet-mappings SubnetId=$HUB_SUBNET_ID \
     --query 'LoadBalancers[0].LoadBalancerArn' \
     --output text)

   # Create VPC Endpoint Service
   ENDPOINT_SERVICE_ID=$(aws ec2 create-vpc-endpoint-service-configuration \
     --network-load-balancer-arns $NLB_ARN \
     --acceptance-required false \
     --query 'ServiceConfiguration.ServiceId' \
     --output text)
   ```

2. **Create VPC Endpoints in Spoke VPCs:**
   ```bash
   # Create endpoint in Spoke1
   aws ec2 create-vpc-endpoint \
     --vpc-id $SPOKE1_VPC_ID \
     --service-name com.amazonaws.vpce.$AWS_REGION.$ENDPOINT_SERVICE_ID \
     --vpc-endpoint-type Interface \
     --subnet-ids $SPOKE1_SUBNET_ID \
     --security-group-ids $SPOKE1_SG_ID

   # Create endpoint in Spoke2
   aws ec2 create-vpc-endpoint \
     --vpc-id $SPOKE2_VPC_ID \
     --service-name com.amazonaws.vpce.$AWS_REGION.$ENDPOINT_SERVICE_ID \
     --vpc-endpoint-type Interface \
     --subnet-ids $SPOKE2_SUBNET_ID \
     --security-group-ids $SPOKE2_SG_ID
   ```

### Verification Steps

1. **Verify Transit Gateway Configuration:**
   ```bash
   # List Transit Gateway attachments
   aws ec2 describe-transit-gateway-attachments \
     --filters Name=transit-gateway-id,Values=$TGW_ID
   ```

2. **Verify VPC Endpoint Connectivity:**
   ```bash
   # List VPC endpoints
   aws ec2 describe-vpc-endpoints \
     --filters Name=vpc-id,Values=$SPOKE1_VPC_ID,$SPOKE2_VPC_ID
   ```

### Cleanup Instructions

1. **Delete VPC Endpoints:**
   ```bash
   # Get endpoint IDs
   ENDPOINTS=$(aws ec2 describe-vpc-endpoints \
     --filters Name=vpc-id,Values=$SPOKE1_VPC_ID,$SPOKE2_VPC_ID \
     --query 'VpcEndpoints[*].VpcEndpointId' \
     --output text)

   # Delete endpoints
   for ENDPOINT in $ENDPOINTS; do
     aws ec2 delete-vpc-endpoints --vpc-endpoint-ids $ENDPOINT
   done
   ```

2. **Delete Transit Gateway Attachments:**
   ```bash
   # Get attachment IDs
   ATTACHMENTS=$(aws ec2 describe-transit-gateway-attachments \
     --filters Name=transit-gateway-id,Values=$TGW_ID \
     --query 'TransitGatewayAttachments[*].TransitGatewayAttachmentId' \
     --output text)

   # Delete attachments
   for ATTACHMENT in $ATTACHMENTS; do
     aws ec2 delete-transit-gateway-attachment --transit-gateway-attachment-id $ATTACHMENT
   done
   ```

3. **Delete Transit Gateway:**
   ```bash
   aws ec2 delete-transit-gateway --transit-gateway-id $TGW_ID
   ```

4. **Delete VPCs and Resources:**
   ```bash
   # Delete Hub VPC resources
   aws ec2 delete-vpc --vpc-id $HUB_VPC_ID

   # Delete Spoke VPCs
   aws ec2 delete-vpc --vpc-id $SPOKE1_VPC_ID
   aws ec2 delete-vpc --vpc-id $SPOKE2_VPC_ID
   ```

5. **Verify Environment Variables are Unset:**
   ```bash
   unset AWS_REGION HUB_VPC_CIDR HUB_SUBNET_CIDR SPOKE1_VPC_CIDR SPOKE1_SUBNET_CIDR SPOKE2_VPC_CIDR SPOKE2_SUBNET_CIDR
   unset HUB_VPC_ID HUB_SUBNET_ID SPOKE1_VPC_ID SPOKE1_SUBNET_ID SPOKE2_VPC_ID SPOKE2_SUBNET_ID
   unset TGW_ID NLB_ARN ENDPOINT_SERVICE_ID
   ```

### Additional Resources

1. [AWS Transit Gateway Documentation](https://docs.aws.amazon.com/vpc/latest/tgw/what-is-transit-gateway.html)
2. [AWS PrivateLink Documentation](https://docs.aws.amazon.com/vpc/latest/privatelink/what-is-privatelink.html)
3. [AWS VPC Documentation](https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html)
