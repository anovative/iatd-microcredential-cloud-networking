## IATD Microcredential Cloud Networking: Lab 3 - Deploying AWS VPCs using CloudFormation Templates

**Objective:** This lab will guide you through creating and deploying an Amazon Virtual Private Cloud (VPC) with public and private subnets using AWS CloudFormation templates. You'll learn how to define infrastructure as code, enabling repeatable and consistent deployments across different AWS regions.

**Estimated Time:** 60 - 75 minutes

**Prerequisites:**

1.  **AWS Account:** Requires an active AWS account. A free tier account is available at [https://aws.amazon.com/free/](https://aws.amazon.com/free/).
2.  **Basic understanding of AWS Networking:** Familiarity with VPCs, Subnets, Route Tables, Internet Gateways, and NAT Gateways.
3.  **Basic understanding of CloudFormation Templates:** Familiarity with the structure of CloudFormation templates is helpful.

### Lab Conventions

*   **AWS Management Console:** Most interactions will occur via the AWS Management Console.
*   **Naming Conventions:** Resource names follow the convention `iatd-aws-lab-09-*`.
*   **Region:** Choose a consistent AWS region (e.g., us-east-1).

#### Resource Naming

*   **VPC:** `iatd-aws-lab-09-vpc`
*   **Public Subnet 1:** `iatd-aws-lab-09-subnet-public-1`
*   **Public Subnet 2:** `iatd-aws-lab-09-subnet-public-2`
*   **Private Subnet 1:** `iatd-aws-lab-09-subnet-private-1`
*   **Private Subnet 2:** `iatd-aws-lab-09-subnet-private-2`
*   **Internet Gateway:** `iatd-aws-lab-09-igw`
*   **NAT Gateway:** `iatd-aws-lab-09-natgw`
*   **CloudFormation Template File:** `vpc_template.yaml`

### Part 1: Creating the CloudFormation Template

1.  **Sign in to the AWS Management Console:** Browse to [https://aws.amazon.com/console/](https://aws.amazon.com/console/) and sign in.

2.  **Create CloudFormation Template File:**

    *   Create a file named `vpc_template.yaml` on your local machine or in CloudShell.
    *   Paste in the following YAML:

        ```yaml
        AWSTemplateFormatVersion: "2010-09-09"
        Description: Creates a VPC with public and private subnets

        Parameters:
          VpcName:
            Type: String
            Default: iatd-aws-lab-09-vpc
            Description: Name of the VPC
          VpcCidr:
            Type: String
            Default: 10.0.0.0/16
            Description: CIDR block for the VPC
          PublicSubnet1Cidr:
            Type: String
            Default: 10.0.1.0/24
            Description: CIDR block for the Public Subnet 1
          PublicSubnet2Cidr:
            Type: String
            Default: 10.0.2.0/24
            Description: CIDR block for the Public Subnet 2
          PrivateSubnet1Cidr:
            Type: String
            Default: 10.0.101.0/24
            Description: CIDR block for the Private Subnet 1
          PrivateSubnet2Cidr:
            Type: String
            Default: 10.0.102.0/24
            Description: CIDR block for the Private Subnet 2
          AvailabilityZone1:
            Type: String
            Default: us-east-1a
            Description: Availability Zone for Subnet 1
          AvailabilityZone2:
            Type: String
            Default: us-east-1b
            Description: Availability Zone for Subnet 2

        Resources:
          VPC:
            Type: AWS::EC2::VPC
            Properties:
              CidrBlock: !Ref VpcCidr
              EnableDnsSupport: true
              EnableDnsHostnames: true
              Tags:
                - Key: Name
                  Value: !Ref VpcName

          InternetGateway:
            Type: AWS::EC2::InternetGateway
            Properties:
              Tags:
                - Key: Name
                  Value: !Sub "${VpcName}-igw"

          VPCGatewayAttachment:
            Type: AWS::EC2::VPCGatewayAttachment
            Properties:
              VpcId: !Ref VPC
              InternetGatewayId: !Ref InternetGateway

          PublicSubnet1:
            Type: AWS::EC2::Subnet
            Properties:
              VpcId: !Ref VPC
              CidrBlock: !Ref PublicSubnet1Cidr
              MapPublicIpOnLaunch: true
              AvailabilityZone: !Ref AvailabilityZone1
              Tags:
                - Key: Name
                  Value: !Sub "${VpcName}-public-subnet-1"

          PublicSubnet2:
            Type: AWS::EC2::Subnet
            Properties:
              VpcId: !Ref VPC
              CidrBlock: !Ref PublicSubnet2Cidr
              MapPublicIpOnLaunch: true
              AvailabilityZone: !Ref AvailabilityZone2
              Tags:
                - Key: Name
                  Value: !Sub "${VpcName}-public-subnet-2"

          PrivateSubnet1:
            Type: AWS::EC2::Subnet
            Properties:
              VpcId: !Ref VPC
              CidrBlock: !Ref PrivateSubnet1Cidr
              AvailabilityZone: !Ref AvailabilityZone1
              Tags:
                - Key: Name
                  Value: !Sub "${VpcName}-private-subnet-1"

          PrivateSubnet2:
            Type: AWS::EC2::Subnet
            Properties:
              VpcId: !Ref VPC
              CidrBlock: !Ref PrivateSubnet2Cidr
              AvailabilityZone: !Ref AvailabilityZone2
              Tags:
                - Key: Name
                  Value: !Sub "${VpcName}-private-subnet-2"

          PublicRouteTable:
            Type: AWS::EC2::RouteTable
            Properties:
              VpcId: !Ref VPC
              Tags:
                - Key: Name
                  Value: !Sub "${VpcName}-public-rt"

          PrivateRouteTable:
            Type: AWS::EC2::RouteTable
            Properties:
              VpcId: !Ref VPC
              Tags:
                - Key: Name
                  Value: !Sub "${VpcName}-private-rt"

          PublicRoute:
            Type: AWS::EC2::Route
            Properties:
              RouteTableId: !Ref PublicRouteTable
              DestinationCidrBlock: 0.0.0.0/0
              GatewayId: !Ref InternetGateway

          SubnetRouteTableAssociationPublic1:
            Type: AWS::EC2::SubnetRouteTableAssociation
            Properties:
              SubnetId: !Ref PublicSubnet1
              RouteTableId: !Ref PublicRouteTable

          SubnetRouteTableAssociationPublic2:
            Type: AWS::EC2::SubnetRouteTableAssociation
            Properties:
              SubnetId: !Ref PublicSubnet2
              RouteTableId: !Ref PublicRouteTable

          SubnetRouteTableAssociationPrivate1:
            Type: AWS::EC2::SubnetRouteTableAssociation
            Properties:
              SubnetId: !Ref PrivateSubnet1
              RouteTableId: !Ref PrivateRouteTable

          SubnetRouteTableAssociationPrivate2:
            Type: AWS::EC2::SubnetRouteTableAssociation
            Properties:
              SubnetId: !Ref PrivateSubnet2
              RouteTableId: !Ref PrivateRouteTable

        Outputs:
          VpcId:
            Description: The ID of the VPC
            Value: !Ref VPC
          PublicSubnet1Id:
            Description: The ID of the Public Subnet 1
            Value: !Ref PublicSubnet1
          PublicSubnet2Id:
            Description: The ID of the Public Subnet 2
            Value: !Ref PublicSubnet2
          PrivateSubnet1Id:
            Description: The ID of the Private Subnet 1
            Value: !Ref PrivateSubnet1
          PrivateSubnet2Id:
            Description: The ID of the Private Subnet 2
            Value: !Ref PrivateSubnet2
        ```

    *   Save the file.

### Part 2: Deploying the CloudFormation Template

1.  **Navigate to CloudFormation:**
    *   Search for "CloudFormation" in the AWS Management Console and select **CloudFormation**.

2.  **Create Stack:**
    *   Click **Create stack** and select **With new resources (standard)**.
    *   For "Prepare template", choose **Template is ready**.
    *   For "Template source", choose **Upload a template file**.
    *   Click **Choose file** and select the `vpc_template.yaml` file you created.
    *   Click **Next**.
    *   Specify stack details:
        *   Stack name: `iatd-aws-lab-09-vpc-stack`
        *   Parameters: Review the parameters and adjust the default values as needed (e.g., VPC CIDR, Subnet CIDRs, Availability Zones).
    *   Click **Next**.
    *   Configure stack options (optional): You can leave these options at their default values.
    *   Click **Next**.
    *   Review the stack configuration and check the box acknowledging that CloudFormation might create IAM resources.
    *   Click **Create stack**.

### Part 3: Verifying the Deployment

1.  **Navigate to VPCs:**
    *   Search for "VPC" in the AWS Management Console and select **VPC**.
    *   Verify that the `iatd-aws-lab-09-vpc` VPC has been created.

2.  **Check Subnets:**
    *   In the VPC dashboard, select **Subnets**.
    *   Verify that the following subnets have been created with the correct CIDR blocks and Availability Zones:
        *   `iatd-aws-lab-09-subnet-public-1`
        *   `iatd-aws-lab-09-subnet-public-2`
        *   `iatd-aws-lab-09-subnet-private-1`
        *   `iatd-aws-lab-09-subnet-private-2`

3.  **Check Route Tables:**
    *   In the VPC dashboard, select **Route Tables**.
    *   Verify that the `iatd-aws-lab-09-vpc-public-rt` and `iatd-aws-lab-09-vpc-private-rt` route tables have been created.

### Part 4: Reusing the CloudFormation Template in Another Region

1.  **Change Region:**
    *   In the AWS Management Console, switch to a different AWS region (e.g., us-west-2).

2.  **Create Stack in New Region:**
    *   Repeat the process from Part 2 to create a new CloudFormation stack using the same `vpc_template.yaml` file.

3.  **Verify Deployment in New Region:**
    *   Repeat the verification steps from Part 3 to ensure that the VPC, subnets, and route tables have been created correctly in the new region.

### Post-Lab: Cleanup

To avoid incurring unnecessary costs, it's essential to clean up the resources created during this lab.

1.  **Delete CloudFormation Stacks:**
    *   In the CloudFormation dashboard, select the `iatd-aws-lab-09-vpc-stack` stack.
    *   Click **Delete**.
    *   Repeat this process for any other stacks you created in different regions.

**Learning Outcomes:**

*   Successfully created a CloudFormation template to define an AWS VPC with public and private subnets.
*   Successfully deployed the CloudFormation template to create the VPC in your AWS account.
*   Successfully verified the deployment using the AWS Management Console.
*   Understood the benefits of using CloudFormation templates for infrastructure as code and repeatable deployments.
*   Learned how to reuse a CloudFormation template to deploy the same VPC configuration in another AWS region.
