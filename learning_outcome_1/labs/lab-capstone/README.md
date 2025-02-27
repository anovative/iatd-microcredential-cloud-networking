## IATD Microcredential Cloud Networking: Capstone Lab - Multi-Cloud Network Infrastructure

**Objective:** In this comprehensive capstone lab, you will practice and reinforce your multi-cloud networking skills by implementing infrastructure spanning Azure, AWS, and GCP. This hands-on lab will help you apply concepts learned from previous labs in a practical, real-world scenario.

**Estimated Time:** 120-180 minutes

### Prerequisites
Before starting this lab, ensure you have:
1. Active subscriptions for:
   - Microsoft Azure
   - Amazon Web Services (AWS)
   - Google Cloud Platform (GCP)
2. Basic familiarity with cloud consoles and CLIs
3. Completion of previous networking labs (recommended)

### Challenge Level System
Each exercise is categorized by difficulty to help you progress systematically:
- **Basic**: Fundamental concepts and single-cloud implementations
- **Intermediate**: Multi-component solutions and cross-cloud basics

### Practice Exercises

These exercises are designed to help you practice cloud networking concepts at your own pace. Each exercise builds upon skills from previous labs.

#### Exercise 1: Azure Virtual Network Basics
**Challenge Level: Basic**

**Related Labs:** 
- Lab-001: Virtual Network Creation
- Lab-002: Network Security Groups

**What You'll Practice:**
- Creating and configuring a Virtual Network
- Understanding subnet design
- Implementing basic network security

**Step-by-Step Tasks:**
1. Create a Virtual Network:
   - Name: `practice-vnet`
   - Address space: `10.0.0.0/16`
   - Region: Your choice

2. Create two subnets:
   - Web subnet: `10.0.1.0/24`
   - Database subnet: `10.0.2.0/24`

3. Set up Network Security:
   - Create NSG for web subnet
   - Create NSG for database subnet
   - Allow web → database on port 3306
   - Block other subnet traffic

4. Verify Your Setup:
   - Use Network Watcher
   - Check effective security rules
   - Test connectivity between subnets

**Helpful Tips:**
- Start with Azure Portal for visual learning
- Try repeating the tasks using Azure CLI
- Use Network Watcher to understand traffic flow
- Don't worry about making mistakes - this is for practice!

#### Exercise 2: GCP Network Foundation
**Challenge Level: Basic**

**Related Labs:**
- Lab-004: Network Security
- Lab-006: Application Gateway

**What You'll Practice:**
- Creating a VPC network
- Understanding GCP's networking model
- Basic firewall configuration

**Step-by-Step Tasks:**
1. Create a VPC:
   - Name: `practice-vpc`
   - Subnet mode: Custom
   - Region: Your choice

2. Create a subnet:
   - Name: `practice-subnet`
   - IP range: `192.168.1.0/24`
   - Private Google access: Enabled

3. Configure Firewall Rules:
   - Allow SSH (port 22)
   - Allow HTTP (port 80)
   - Allow HTTPS (port 443)
   - Set appropriate priority numbers

4. Verify Your Setup:
   - Check subnet configuration
   - Verify firewall rules
   - Use Cloud Shell to test connectivity

**Helpful Tips:**
- Use GCP Console to visualize the network
- Practice with gcloud commands
- Pay attention to firewall rule priorities
- Take screenshots of your configurations

#### Exercise 3: AWS VPC Design
**Challenge Level: Intermediate**

**Related Labs:**
- Lab-003: VPC Configuration
- Lab-005: Network Access Control

**What You'll Practice:**
- VPC creation and configuration
- Public and private subnet design
- Route table management
- Security group implementation

**Step-by-Step Tasks:**
1. Create a VPC:
   - Name: `practice-vpc`
   - CIDR block: `172.16.0.0/16`
   - Enable DNS hostnames

2. Create Subnets:
   - Public subnet: `172.16.1.0/24`
   - Private subnet: `172.16.2.0/24`
   - Choose different AZs for each

3. Configure Internet Access:
   - Create Internet Gateway
   - Create NAT Gateway
   - Configure route tables
   - Associate subnets

4. Implement Security:
   - Create security groups
   - Configure NACLs
   - Test connectivity

**Helpful Tips:**
- Use the VPC Wizard for initial setup
- Remember to enable auto-assign public IPs for public subnet
- Test connectivity using EC2 instances
- Clean up resources when done to avoid charges

### Cleanup Instructions
After completing the exercises, remember to delete all created resources to avoid unnecessary charges:

1. Azure:
   - Delete resource groups
   - Verify no orphaned resources

2. GCP:
   - Delete VPC networks
   - Remove firewall rules
   - Check for running instances

3. AWS:
   - Terminate EC2 instances
   - Delete VPC and associated resources
   - Check for elastic IPs

### Learning Outcomes
After completing these exercises, you will:
- Understand basic networking concepts in Azure, GCP, and AWS
- Be able to create and configure network resources
- Know how to implement basic network security
- Have hands-on experience with cloud networking tools

Remember, these are practice exercises - feel free to experiment and learn from any mistakes. If you get stuck, refer back to the previous labs or consult the cloud provider's documentation.
