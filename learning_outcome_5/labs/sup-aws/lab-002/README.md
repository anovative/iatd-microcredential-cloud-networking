## IATD Microcredential Cloud Networking: AWS Lab 2 - Hybrid Connectivity with AWS Site-to-Site VPN and Transit Gateway

**Objective:** This lab guides you through implementing hybrid connectivity solutions in AWS. You'll learn how to set up a Site-to-Site VPN connection between your on-premises network (simulated in AWS) and AWS VPCs, and how to use AWS Transit Gateway to simplify connectivity between multiple VPCs.

**Estimated Time:** 90 - 120 minutes

**Prerequisites:**

1.  **AWS Account:** Requires an active AWS account. A free tier account is available at [https://aws.amazon.com/free/](https://aws.amazon.com/free/).
2.  **Basic understanding of AWS Networking:** Familiarity with VPCs, Subnets, Route Tables, and VPN concepts.

### Lab Conventions

*   **AWS Management Console:** Most interactions will occur via the AWS Management Console.
*   **Naming Conventions:** Resource names follow the convention `iatd-aws-hybrid-*`.
*   **IP Address Range:** The `172.16.0.0/16` range will be used (as per standardization).
*   **Region:** Choose a consistent AWS region (e.g., us-east-1).

#### Resource Naming

*   **On-premises VPC:** `iatd-aws-hybrid-onprem-vpc`
*   **AWS VPC 1:** `iatd-aws-hybrid-vpc-1`
*   **AWS VPC 2:** `iatd-aws-hybrid-vpc-2`
*   **Transit Gateway:** `iatd-aws-hybrid-tgw`
*   **Customer Gateway:** `iatd-aws-hybrid-cgw`
*   **VPN Gateway:** `iatd-aws-hybrid-vpn`
*   **Site-to-Site VPN Connection:** `iatd-aws-hybrid-s2s-vpn`
*   **EC2 Instance - On-premises:** `iatd-aws-hybrid-onprem-instance`
*   **EC2 Instance - VPC 1:** `iatd-aws-hybrid-vpc1-instance`
*   **EC2 Instance - VPC 2:** `iatd-aws-hybrid-vpc2-instance`

### Part 1: Setting up the VPCs

1.  **Sign in to the AWS Management Console:** Browse to [https://aws.amazon.com/console/](https://aws.amazon.com/console/) and sign in.

2.  **Create the On-premises VPC:**
    *   Search for "VPC" in the AWS Management Console and select **VPC**.
    *   Click **Create VPC**.
    *   Select **VPC and more** to create a VPC with subnets, route tables, and internet gateway.
    *   Configure the VPC:
        *   **Name tag:** `iatd-aws-hybrid-onprem-vpc`
        *   **IPv4 CIDR block:** `172.16.0.0/16`
        *   **Number of Availability Zones:** `1`
        *   **Number of public subnets:** `1`
        *   **Number of private subnets:** `0`
    *   Click **Create VPC**.

3.  **Create AWS VPC 1:**
    *   Click **Create VPC** again.
    *   Select **VPC and more**.
    *   Configure the VPC:
        *   **Name tag:** `iatd-aws-hybrid-vpc-1`
        *   **IPv4 CIDR block:** `172.17.0.0/16`
        *   **Number of Availability Zones:** `1`
        *   **Number of public subnets:** `1`
        *   **Number of private subnets:** `0`
    *   Click **Create VPC**.

4.  **Create AWS VPC 2:**
    *   Click **Create VPC** again.
    *   Select **VPC and more**.
    *   Configure the VPC:
        *   **Name tag:** `iatd-aws-hybrid-vpc-2`
        *   **IPv4 CIDR block:** `172.18.0.0/16`
        *   **Number of Availability Zones:** `1`
        *   **Number of public subnets:** `1`
        *   **Number of private subnets:** `0`
    *   Click **Create VPC**.

    Expected output:
    ```
    Three VPCs created successfully:
    - On-premises VPC with CIDR 172.16.0.0/16
    - AWS VPC 1 with CIDR 172.17.0.0/16
    - AWS VPC 2 with CIDR 172.18.0.0/16
    Each VPC has one public subnet, an internet gateway, and route tables configured for internet access
    ```

### Part 2: Creating EC2 Instances

1.  **Create Security Group for EC2 Instances:**
    *   Navigate to **Security Groups** in the VPC dashboard.
    *   Click **Create security group**.
    *   Configure the security group:
        *   **Security group name:** `iatd-aws-hybrid-sg-ec2`
        *   **Description:** `Security group for EC2 instances`
        *   **VPC:** Select `iatd-aws-hybrid-onprem-vpc`
        *   Add inbound rules:
            *   Type: All ICMP - IPv4, Source: `172.16.0.0/16`
            *   Type: All ICMP - IPv4, Source: `172.17.0.0/16`
            *   Type: All ICMP - IPv4, Source: `172.18.0.0/16`
            *   Type: SSH, Source: My IP (for management access)
    *   Click **Create security group**.
    *   Repeat this process for the other two VPCs, adjusting the VPC selection accordingly.

2.  **Launch EC2 Instance in On-premises VPC:**
    *   Search for "EC2" in the AWS Management Console and select **EC2**.
    *   Click **Launch instance**.
    *   Configure the instance:
        *   **Name:** `iatd-aws-hybrid-onprem-instance`
        *   **Amazon Machine Image (AMI):** Amazon Linux 2023
        *   **Instance type:** t2.micro (Free tier eligible)
        *   **Key pair:** Create or select an existing key pair
        *   **Network settings:**
            *   **VPC:** `iatd-aws-hybrid-onprem-vpc`
            *   **Subnet:** Select the public subnet
            *   **Auto-assign public IP:** Enable
            *   **Security group:** Select the security group created for this VPC
    *   Click **Launch instance**.

3.  **Launch EC2 Instance in AWS VPC 1:**
    *   Repeat the process above, but with the following changes:
        *   **Name:** `iatd-aws-hybrid-vpc1-instance`
        *   **VPC:** `iatd-aws-hybrid-vpc-1`
        *   **Security group:** Select the security group created for this VPC

4.  **Launch EC2 Instance in AWS VPC 2:**
    *   Repeat the process again, but with the following changes:
        *   **Name:** `iatd-aws-hybrid-vpc2-instance`
        *   **VPC:** `iatd-aws-hybrid-vpc-2`
        *   **Security group:** Select the security group created for this VPC

    Expected output:
    ```
    Three EC2 instances launched successfully:
    - One instance in the On-premises VPC
    - One instance in AWS VPC 1
    - One instance in AWS VPC 2
    Each instance has a public IP for SSH access and is configured to allow ICMP traffic from all three VPC CIDR ranges
    ```

### Part 3: Setting up AWS Transit Gateway

1.  **Create Transit Gateway:**
    *   Navigate to **Transit Gateways** in the VPC dashboard.
    *   Click **Create Transit Gateway**.
    *   Configure the transit gateway:
        *   **Name tag:** `iatd-aws-hybrid-tgw`
        *   **Description:** `Transit Gateway for hybrid connectivity lab`
        *   **Amazon side ASN:** `64512` (default)
        *   **DNS support:** Enabled
        *   **VPN ECMP support:** Enabled
        *   **Default route table association:** Enabled
        *   **Default route table propagation:** Enabled
    *   Click **Create Transit Gateway**.
    *   Wait for the transit gateway to be in the **available** state before proceeding.

2.  **Create Transit Gateway Attachments:**
    *   Navigate to **Transit Gateway Attachments** in the VPC dashboard.
    *   Click **Create Transit Gateway Attachment**.
    *   Configure the attachment for AWS VPC 1:
        *   **Transit Gateway ID:** Select `iatd-aws-hybrid-tgw`
        *   **Attachment type:** VPC
        *   **Attachment name tag:** `iatd-aws-hybrid-tgw-vpc1-attachment`
        *   **VPC ID:** Select `iatd-aws-hybrid-vpc-1`
        *   **Subnet IDs:** Select the public subnet in the VPC
    *   Click **Create Transit Gateway Attachment**.
    *   Repeat this process for AWS VPC 2, adjusting the attachment name and VPC selection accordingly.

3.  **Update Route Tables:**
    *   Navigate to **Route Tables** in the VPC dashboard.
    *   Select the route table associated with the public subnet in AWS VPC 1.
    *   Click the **Routes** tab.
    *   Click **Edit routes**.
    *   Click **Add route**.
    *   Configure the route:
        *   **Destination:** `172.18.0.0/16` (CIDR of AWS VPC 2)
        *   **Target:** Transit Gateway, select `iatd-aws-hybrid-tgw`
    *   Click **Save changes**.
    *   Repeat this process for the route table in AWS VPC 2, but set the destination to `172.17.0.0/16` (CIDR of AWS VPC 1).

    Expected output:
    ```
    Transit Gateway and attachments created successfully:
    - Transit Gateway is in the available state
    - VPC 1 and VPC 2 are attached to the Transit Gateway
    - Route tables are updated to route traffic between VPCs through the Transit Gateway
    ```

### Part 4: Setting up Site-to-Site VPN

1.  **Configure the On-premises VPC as a Customer Gateway:**
    *   Note the public IP address of the EC2 instance in the On-premises VPC. This will simulate your on-premises router.
    *   Navigate to **Customer Gateways** in the VPC dashboard.
    *   Click **Create Customer Gateway**.
    *   Configure the customer gateway:
        *   **Name:** `iatd-aws-hybrid-cgw`
        *   **IP address:** Enter the public IP of your On-premises EC2 instance
        *   **BGP ASN:** `65000` (or any valid ASN)
    *   Click **Create Customer Gateway**.

2.  **Create a Virtual Private Gateway (Optional - if not using Transit Gateway for VPN):**
    *   Navigate to **Virtual Private Gateways** in the VPC dashboard.
    *   Click **Create Virtual Private Gateway**.
    *   Configure the virtual private gateway:
        *   **Name tag:** `iatd-aws-hybrid-vpn`
        *   **ASN:** Amazon default ASN
    *   Click **Create Virtual Private Gateway**.
    *   After creation, select the VPG and click **Actions** > **Attach to VPC**.
    *   Select AWS VPC 1 and click **Attach**.

3.  **Create Site-to-Site VPN Connection using Transit Gateway:**
    *   Navigate to **Site-to-Site VPN Connections** in the VPC dashboard.
    *   Click **Create VPN Connection**.
    *   Configure the VPN connection:
        *   **Name tag:** `iatd-aws-hybrid-s2s-vpn`
        *   **Target gateway type:** Transit Gateway
        *   **Transit Gateway:** Select `iatd-aws-hybrid-tgw`
        *   **Customer gateway:** Existing, select `iatd-aws-hybrid-cgw`
        *   **Routing options:** Static
        *   **Static IP prefixes:** `172.16.0.0/16` (CIDR of On-premises VPC)
    *   Click **Create VPN Connection**.

4.  **Update Route Tables:**
    *   Navigate to **Route Tables** in the VPC dashboard.
    *   Select the route tables for both AWS VPC 1 and AWS VPC 2.
    *   Add a route to the On-premises VPC CIDR (`172.16.0.0/16`) via the Transit Gateway.
    *   Select the route table for the On-premises VPC.
    *   Add routes to both AWS VPC CIDRs (`172.17.0.0/16` and `172.18.0.0/16`) via the VPN connection.

    Expected output:
    ```
    Site-to-Site VPN connection created successfully:
    - Customer Gateway represents the On-premises router
    - VPN connection is established through the Transit Gateway
    - Route tables are updated to route traffic between On-premises and AWS VPCs
    ```

### Part 5: Testing Connectivity

1.  **Test Connectivity between AWS VPCs:**
    *   Connect to the EC2 instance in AWS VPC 1 via SSH.
    *   Ping the private IP address of the EC2 instance in AWS VPC 2:

        ```bash
        ping <private-ip-of-vpc2-instance>
        ```

    *   The ping should be successful, indicating that traffic is flowing through the Transit Gateway.

2.  **Simulate On-premises to AWS Connectivity:**
    *   In a real environment, you would configure your on-premises router with the VPN configuration details provided by AWS.
    *   For this lab, we're simulating the on-premises environment, so we'll focus on the AWS side of the configuration.
    *   The VPN connection status should show as "Available" in the AWS console, but the tunnel status may show as "DOWN" since we're not actually configuring a real VPN device.

    Expected outcome:
    ```
    - Successful ping between instances in AWS VPC 1 and AWS VPC 2, demonstrating Transit Gateway connectivity
    - VPN connection shows as "Available" in the AWS console
    - In a real environment, you would see successful connectivity between on-premises and AWS resources
    ```

### Part 6: Advanced Transit Gateway Features (Optional)

1.  **Transit Gateway Route Tables:**
    *   Navigate to **Transit Gateway Route Tables** in the VPC dashboard.
    *   Explore the default route table and see how routes are propagated from attached VPCs.
    *   You can create additional route tables for more complex routing scenarios.

2.  **Transit Gateway Network Manager:**
    *   Explore the Transit Gateway Network Manager, which provides a global view of your network across regions and accounts.

### Post-Lab: Cleanup

To avoid incurring unnecessary costs, it's essential to clean up the resources created during this lab.

1.  **Delete Site-to-Site VPN Connection:**
    *   Navigate to **Site-to-Site VPN Connections** in the VPC dashboard.
    *   Select `iatd-aws-hybrid-s2s-vpn`.
    *   Click **Actions** > **Delete VPN Connection**.
    *   Confirm deletion.

2.  **Delete Customer Gateway:**
    *   Navigate to **Customer Gateways** in the VPC dashboard.
    *   Select `iatd-aws-hybrid-cgw`.
    *   Click **Actions** > **Delete Customer Gateway**.
    *   Confirm deletion.

3.  **Delete Transit Gateway Attachments:**
    *   Navigate to **Transit Gateway Attachments** in the VPC dashboard.
    *   Select all attachments associated with `iatd-aws-hybrid-tgw`.
    *   Click **Actions** > **Delete Transit Gateway Attachment**.
    *   Confirm deletion.
    *   Wait for the attachments to be deleted before proceeding.

4.  **Delete Transit Gateway:**
    *   Navigate to **Transit Gateways** in the VPC dashboard.
    *   Select `iatd-aws-hybrid-tgw`.
    *   Click **Actions** > **Delete Transit Gateway**.
    *   Confirm deletion.

5.  **Terminate EC2 Instances:**
    *   Navigate to **Instances** in the EC2 dashboard.
    *   Select all instances created in this lab.
    *   Click **Instance state** > **Terminate instance**.
    *   Confirm termination.

6.  **Delete VPCs and Associated Resources:**
    *   Navigate to **Your VPCs** in the VPC dashboard.
    *   For each VPC created in this lab, click **Actions** > **Delete VPC**.
    *   This will delete the VPC and all associated resources (subnets, route tables, internet gateway).
    *   Confirm deletion.

**Learning Outcomes:**

*   Successfully created and configured AWS Transit Gateway for connecting multiple VPCs.
*   Successfully set up a simulated Site-to-Site VPN connection between on-premises and AWS environments.
*   Understood the key components of hybrid connectivity in AWS, including Customer Gateways, VPN Connections, and Transit Gateway.
*   Learned how to configure route tables for traffic flow between VPCs and on-premises networks.
*   Understood the benefits of using Transit Gateway for simplifying network architecture and reducing connection complexity.
