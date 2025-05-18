## IATD Microcredential Cloud Networking: AWS Lab 1 - Elastic Load Balancing with AWS Application Load Balancer

**Objective:** This lab guides you through implementing load balancing in AWS using the Application Load Balancer (ALB). You'll learn how to distribute traffic across multiple EC2 instances, configure health checks, and implement path-based routing.

**Estimated Time:** 75 - 90 minutes

**Prerequisites:**

1.  **AWS Account:** Requires an active AWS account. A free tier account is available at [https://aws.amazon.com/free/](https://aws.amazon.com/free/).
2.  **Basic understanding of AWS Networking:** Familiarity with VPCs, Subnets, and EC2 instances.

### Lab Conventions

*   **AWS Management Console:** Most interactions will occur via the AWS Management Console.
*   **Naming Conventions:** Resource names follow the convention `iatd-aws-lb-*`.
*   **IP Address Range:** The `172.16.0.0/16` range will be used (as per standardization).
*   **Region:** Choose a consistent AWS region (e.g., us-east-1).

#### Resource Naming

*   **VPC:** `iatd-aws-lb-vpc`
*   **Public Subnet 1:** `iatd-aws-lb-subnet-public-1`
*   **Public Subnet 2:** `iatd-aws-lb-subnet-public-2`
*   **Security Group - ALB:** `iatd-aws-lb-sg-alb`
*   **Security Group - EC2:** `iatd-aws-lb-sg-ec2`
*   **EC2 Instance 1:** `iatd-aws-lb-instance-1`
*   **EC2 Instance 2:** `iatd-aws-lb-instance-2`
*   **Application Load Balancer:** `iatd-aws-lb-alb`
*   **Target Group:** `iatd-aws-lb-target-group`

### Part 1: Setting up the VPC and Subnets

1.  **Sign in to the AWS Management Console:** Browse to [https://aws.amazon.com/console/](https://aws.amazon.com/console/) and sign in.

2.  **Create a VPC:**
    *   Search for "VPC" in the AWS Management Console and select **VPC**.
    *   Click **Create VPC**.
    *   Select **VPC and more** to create a VPC with subnets, route tables, and internet gateway.
    *   Configure the VPC:
        *   **Name tag:** `iatd-aws-lb-vpc`
        *   **IPv4 CIDR block:** `172.16.0.0/16`
        *   **Number of Availability Zones:** `2`
        *   **Number of public subnets:** `2`
        *   **Number of private subnets:** `0` (We don't need private subnets for this lab)
    *   Click **Create VPC**.

    Expected output:
    ```
    VPC created successfully with the following components:
    - VPC with CIDR 172.16.0.0/16
    - 2 public subnets in different availability zones
    - Internet Gateway
    - Route tables configured for internet access
    ```

### Part 2: Creating EC2 Instances

1.  **Create Security Group for EC2 Instances:**
    *   Navigate to **Security Groups** in the VPC dashboard.
    *   Click **Create security group**.
    *   Configure the security group:
        *   **Security group name:** `iatd-aws-lb-sg-ec2`
        *   **Description:** `Security group for EC2 instances behind ALB`
        *   **VPC:** Select `iatd-aws-lb-vpc`
        *   Add inbound rules:
            *   Type: HTTP, Source: Custom, Value: `172.16.0.0/16` (Allow traffic from within the VPC)
            *   Type: SSH, Source: My IP (for management access)
    *   Click **Create security group**.

2.  **Launch EC2 Instance 1:**
    *   Search for "EC2" in the AWS Management Console and select **EC2**.
    *   Click **Launch instance**.
    *   Configure the instance:
        *   **Name:** `iatd-aws-lb-instance-1`
        *   **Amazon Machine Image (AMI):** Amazon Linux 2023
        *   **Instance type:** t2.micro (Free tier eligible)
        *   **Key pair:** Create or select an existing key pair
        *   **Network settings:**
            *   **VPC:** `iatd-aws-lb-vpc`
            *   **Subnet:** Select the first public subnet
            *   **Security group:** Select `iatd-aws-lb-sg-ec2`
        *   **Advanced details:** Add the following user data script to install and configure a web server:

            ```bash
            #!/bin/bash
            yum update -y
            yum install -y httpd
            systemctl start httpd
            systemctl enable httpd
            echo "<html><body><h1>This is Server 1</h1></body></html>" > /var/www/html/index.html
            ```
    *   Click **Launch instance**.

3.  **Launch EC2 Instance 2:**
    *   Repeat the process above, but with the following changes:
        *   **Name:** `iatd-aws-lb-instance-2`
        *   **Subnet:** Select the second public subnet
        *   Modify the user data script to identify it as Server 2:

            ```bash
            #!/bin/bash
            yum update -y
            yum install -y httpd
            systemctl start httpd
            systemctl enable httpd
            echo "<html><body><h1>This is Server 2</h1></body></html>" > /var/www/html/index.html
            ```
    *   Click **Launch instance**.

    Expected output:
    ```
    Two EC2 instances launched successfully:
    - Instance 1 in the first public subnet with web server configured
    - Instance 2 in the second public subnet with web server configured
    ```

### Part 3: Creating the Application Load Balancer

1.  **Create Security Group for ALB:**
    *   Navigate to **Security Groups** in the VPC dashboard.
    *   Click **Create security group**.
    *   Configure the security group:
        *   **Security group name:** `iatd-aws-lb-sg-alb`
        *   **Description:** `Security group for Application Load Balancer`
        *   **VPC:** Select `iatd-aws-lb-vpc`
        *   Add inbound rules:
            *   Type: HTTP, Source: Anywhere-IPv4
            *   Type: HTTPS, Source: Anywhere-IPv4 (Optional, if you want to configure SSL)
    *   Click **Create security group**.

2.  **Create Target Group:**
    *   Navigate to **Target Groups** in the EC2 dashboard.
    *   Click **Create target group**.
    *   Configure the target group:
        *   **Choose a target type:** Instances
        *   **Target group name:** `iatd-aws-lb-target-group`
        *   **Protocol:** HTTP
        *   **Port:** 80
        *   **VPC:** Select `iatd-aws-lb-vpc`
        *   **Health check settings:**
            *   **Protocol:** HTTP
            *   **Path:** `/`
            *   **Advanced health check settings:** Use default values
    *   Click **Next**.
    *   Register targets:
        *   Select both EC2 instances (`iatd-aws-lb-instance-1` and `iatd-aws-lb-instance-2`)
        *   Click **Include as pending below**
        *   Click **Create target group**.

3.  **Create Application Load Balancer:**
    *   Navigate to **Load Balancers** in the EC2 dashboard.
    *   Click **Create load balancer**.
    *   Select **Application Load Balancer**.
    *   Configure the load balancer:
        *   **Load balancer name:** `iatd-aws-lb-alb`
        *   **Scheme:** Internet-facing
        *   **IP address type:** IPv4
        *   **Network mapping:**
            *   **VPC:** Select `iatd-aws-lb-vpc`
            *   **Mappings:** Select both public subnets
        *   **Security groups:** Select `iatd-aws-lb-sg-alb`
        *   **Listeners and routing:**
            *   **Protocol:** HTTP
            *   **Port:** 80
            *   **Default action:** Forward to `iatd-aws-lb-target-group`
    *   Click **Create load balancer**.

    Expected output:
    ```
    Application Load Balancer created successfully:
    - Internet-facing load balancer with HTTP listener on port 80
    - Configured to forward traffic to the target group containing both EC2 instances
    - Health checks configured to monitor the health of the instances
    ```

### Part 4: Testing the Load Balancer

1.  **Verify Target Health:**
    *   Navigate to **Target Groups** in the EC2 dashboard.
    *   Select `iatd-aws-lb-target-group`.
    *   In the **Targets** tab, verify that both instances are showing as **Healthy**.

2.  **Test Load Balancing:**
    *   Navigate to **Load Balancers** in the EC2 dashboard.
    *   Select `iatd-aws-lb-alb`.
    *   Copy the **DNS name** of the load balancer.
    *   Open a web browser and navigate to the DNS name.
    *   Refresh the page multiple times. You should see the content alternating between "This is Server 1" and "This is Server 2", demonstrating that the load balancer is distributing traffic across both instances.

    Expected output:
    ```
    - Browser shows either "This is Server 1" or "This is Server 2" when accessing the load balancer DNS name
    - Refreshing the page multiple times shows alternating content, confirming load balancing is working
    ```

### Part 5: Implementing Path-Based Routing (Optional)

1.  **Modify EC2 Instance 1 for Path-Based Routing:**
    *   Connect to `iatd-aws-lb-instance-1` via SSH.
    *   Create a directory for images and add a sample page:

        ```bash
        sudo mkdir -p /var/www/html/images
        echo "<html><body><h1>This is the Images Section</h1></body></html>" | sudo tee /var/www/html/images/index.html
        ```

2.  **Modify EC2 Instance 2 for Path-Based Routing:**
    *   Connect to `iatd-aws-lb-instance-2` via SSH.
    *   Create a directory for videos and add a sample page:

        ```bash
        sudo mkdir -p /var/www/html/videos
        echo "<html><body><h1>This is the Videos Section</h1></body></html>" | sudo tee /var/www/html/videos/index.html
        ```

3.  **Create Additional Target Groups:**
    *   Create a target group for images:
        *   **Target group name:** `iatd-aws-lb-target-group-images`
        *   Register only `iatd-aws-lb-instance-1`
    *   Create a target group for videos:
        *   **Target group name:** `iatd-aws-lb-target-group-videos`
        *   Register only `iatd-aws-lb-instance-2`

4.  **Add Path-Based Rules to ALB:**
    *   Navigate to **Load Balancers** in the EC2 dashboard.
    *   Select `iatd-aws-lb-alb`.
    *   Select the **Listeners** tab.
    *   Select the HTTP:80 listener and click **View/edit rules**.
    *   Click **Add rules**.
    *   Add a rule for images:
        *   **IF (Condition):** Path is `/images*`
        *   **THEN (Action):** Forward to `iatd-aws-lb-target-group-images`
    *   Add a rule for videos:
        *   **IF (Condition):** Path is `/videos*`
        *   **THEN (Action):** Forward to `iatd-aws-lb-target-group-videos`

5.  **Test Path-Based Routing:**
    *   Access the load balancer DNS name with the following paths:
        *   `/images` - Should show "This is the Images Section"
        *   `/videos` - Should show "This is the Videos Section"

    Expected outcome:
    ```
    - When accessing /images path: You should see "This is the Images Section"
    - When accessing /videos path: You should see "This is the Videos Section"
    - The same load balancer DNS name routes to different instances based on the URL path
    ```

### Post-Lab: Cleanup

To avoid incurring unnecessary costs, it's essential to clean up the resources created during this lab.

1.  **Delete the Application Load Balancer:**
    *   Navigate to **Load Balancers** in the EC2 dashboard.
    *   Select `iatd-aws-lb-alb`.
    *   Click **Actions** > **Delete**.
    *   Confirm deletion.

2.  **Delete Target Groups:**
    *   Navigate to **Target Groups** in the EC2 dashboard.
    *   Select all target groups created in this lab.
    *   Click **Actions** > **Delete**.
    *   Confirm deletion.

3.  **Terminate EC2 Instances:**
    *   Navigate to **Instances** in the EC2 dashboard.
    *   Select both instances (`iatd-aws-lb-instance-1` and `iatd-aws-lb-instance-2`).
    *   Click **Instance state** > **Terminate instance**.
    *   Confirm termination.

4.  **Delete Security Groups:**
    *   Navigate to **Security Groups** in the VPC dashboard.
    *   Delete `iatd-aws-lb-sg-alb` and `iatd-aws-lb-sg-ec2`.

5.  **Delete VPC and Associated Resources:**
    *   Navigate to **Your VPCs** in the VPC dashboard.
    *   Select `iatd-aws-lb-vpc`.
    *   Click **Actions** > **Delete VPC**.
    *   This will delete the VPC and all associated resources (subnets, route tables, internet gateway).
    *   Confirm deletion.

**Learning Outcomes:**

*   Successfully created and configured an AWS Application Load Balancer.
*   Successfully distributed traffic across multiple EC2 instances in different availability zones.
*   Successfully implemented and tested path-based routing.
*   Understood how to configure health checks and security groups for load balancing in AWS.
*   Learned about high availability and fault tolerance through multi-AZ deployment.
