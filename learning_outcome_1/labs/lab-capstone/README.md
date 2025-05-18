# Multi-Cloud Network Infrastructure Capstone Lab

## Overview
In this capstone lab, you'll gain hands-on experience with enterprise-grade networking across major cloud platforms: Microsoft Azure, Google Cloud Platform (GCP), and Amazon Web Services (AWS). Through structured exercises, you'll implement and configure networking components while applying industry best practices.

## Learning Objectives
After completing this capstone lab, you'll be able to:
- Design and implement cloud-native networking solutions
- Configure secure network architectures across multiple cloud providers
- Apply industry-standard networking practices and security controls
- Demonstrate proficiency with cloud networking tools and services

## Prerequisites
### Required Access
You'll need active accounts for:
- Microsoft Azure Subscription
- Amazon Web Services (AWS) Account
- Google Cloud Platform (GCP) Account

### Technical Requirements
Before starting, ensure you have:
- Working knowledge of cloud platform consoles
- Basic understanding of networking concepts
- Familiarity with cloud CLI tools
- Completed the prerequisite networking labs

## Lab Conventions

- **Naming Conventions:** All resources created in this lab will be prefixed with `iatd_capstone_` to ensure easy identification and cleanup.
- **IP Address Range:** The `172.16.0.0/16` range will be used (as per standardization) for all cloud environments.
- **Location:** Choose consistent regions within each cloud provider.

## Exercise Framework
The exercises are structured according to complexity levels to help you progress:
- **Foundation Level**: Core concepts and single-cloud implementations
- **Professional Level**: Advanced solutions and cross-cloud architectures

## Laboratory Exercises

### Exercise 1: Enterprise Network Design in Azure
 **Foundation Level** | Estimated Time: 45 minutes

**Context:**
You'll create a secure, multi-tier network architecture that mirrors real-world enterprise environments. This exercise will help you understand how to properly architect virtual networks for business applications.

**Prerequisites:** 
- Lab-001: Virtual Network Creation
- Lab-002: Network Security Groups

**What You'll Learn:**
- How to implement enterprise-grade virtual networks
- Ways to configure network segmentation
- Methods to deploy effective security controls

**Key Concepts You'll Use:**
- **Virtual Networks (VNets)**: Azure's networking foundation that provides isolation, segmentation, and secure communication
- **Subnets**: Network segments that help you organize and secure resources
- **Network Security Groups (NSGs)**: Virtual firewalls that let you control inbound and outbound traffic

**Your Implementation Steps:**
1. Virtual Network Configuration
   - Network Name: `practice-vnet`
   - Address Space: `10.0.0.0/16`
   - Region: Select based on your latency requirements
   
   *Why these settings?* The /16 address space gives you 65,534 available IP addresses, allowing room for future growth. Choose a region close to your users for better performance.

2. Network Segmentation
   - Web Tier: `10.0.1.0/24` (251 available IPs)
   - Database Tier: `10.0.2.0/24` (251 available IPs)
   
   *Design Consideration:* You're creating separate tiers to provide security isolation and clear traffic boundaries.

3. Security Implementation
   - Configure Network Security Groups (NSGs)
     * Web tier: Allow inbound HTTP/HTTPS (ports 80/443)
     * Database tier: Allow only web tier traffic on port 3306
   - Implement tier-based access controls
   - Enable database connectivity (port 3306)
   - Apply security best practices
   
   *Security Note:* You're following the principle of least privilege, allowing only necessary traffic.

4. Validation Procedures
   - Use Network Watcher to verify your setup
   - Validate your security rules
   - Test connectivity between your tiers
   
   *Verification Tip:* Use Network Watcher's IP flow verify feature to troubleshoot any connectivity issues you encounter.

**Technical Guidance:**
- Start with Azure Portal for your initial configuration
  * The visual interface helps you understand component relationships
  * Great for learning and initial setup
- Move to Azure CLI for automation
  * Enables you to automate future deployments
  * Essential for enterprise scenarios
- Use Network Watcher for troubleshooting
  * Helps you capture and analyze network traffic
  * Verifies your security rules are working
- Keep documentation of your changes
  * Track all your modifications
  * Important for compliance and auditing

**Common Pitfalls to Avoid:**
1. Not planning your IP address space properly
2. Overlooking your NSG rule priorities
3. Forgetting to document your security rule changes
4. Not testing connectivity between your tiers

**Your Success Checklist:**
✓ You've created a virtual network with proper address space
✓ Your subnets are configured with appropriate sizing
✓ You've implemented NSGs with secure rules
✓ You can demonstrate successful connectivity between tiers
✓ You've documented all your changes

### Exercise 2: GCP Network Architecture
 **Foundation Level** | Estimated Time: 45 minutes

**Context:**
You'll explore GCP's unique networking model, which differs from other cloud providers by offering global VPCs and automatic subnet routing. This exercise walks you through creating your own secure, scalable network infrastructure in GCP.

**Prerequisites:**
- Lab-004: Network Security
- Lab-006: Application Gateway

**Key Concepts You'll Use:**
- **VPC Networks**: Global resources that let you span multiple regions
- **Subnets**: Regional resources for organizing your resources
- **Firewall Rules**: Global resources that help you control traffic flow
- **Cloud Router**: Enables your dynamic route advertisement

**What You'll Learn:**
- How to design VPC networks following GCP's global approach
- Ways to implement GCP's networking model effectively
- Methods to configure enterprise security controls
- Understanding of GCP-specific networking features

**Your Implementation Steps:**
1. VPC Configuration
   - Network Name: `practice-vpc`
   - Network Type: Custom Mode
   - Region: Based on your compliance requirements
   
   *Why Custom Mode?* This gives you full control over subnet creation and IP ranges, essential for your enterprise environment.

2. Subnet Architecture
   - Subnet Name: `practice-subnet`
   - CIDR Range: `192.168.1.0/24`
   - Private Google Access: Enabled
   
   *Design Note:* Private Google Access lets your VM instances without external IPs access Google APIs and services.

3. Security Controls
   - Set up your firewall rules:
     * SSH (TCP 22): For your administrative access
     * HTTP (TCP 80): For your web traffic
     * HTTPS (TCP 443): For your secure web communication
   - Configure your rule priorities
   
   *Security Best Practice:* Always follow the principle of least privilege when setting up your firewall rules.

4. Architecture Validation
   - Verify your subnet deployment
   - Check your firewall configurations
   - Test your network connectivity
   
   *Testing Tip:* Use Cloud Shell to verify your connectivity and test your firewall rules.

**Technical Guidance:**
- Start with GCP Console for visual configuration
  * Helps you understand the global nature of VPC
  * Gives you clear visibility of your network components
- Progress to gcloud CLI for automation
  * Lets you create repeatable deployments
  * Enables your infrastructure as code journey
- Follow security best practices
  * Use your service accounts appropriately
  * Implement your IAM roles properly
- Maintain your configuration documentation
  * Track all your network changes
  * Document your design decisions

**Common Pitfalls to Avoid:**
1. Forgetting to enable your Private Google Access
2. Misconfiguring your firewall rule priorities
3. Not planning for your future subnet expansion
4. Overlooking your logging and monitoring setup

**Your Success Checklist:**
✓ You've configured your VPC network properly
✓ Your subnet is created with appropriate CIDR range
✓ You've implemented firewall rules securely
✓ Your Private Google Access is enabled
✓ You've verified your network connectivity

### Exercise 3: AWS Enterprise Network Design
 **Professional Level** | Estimated Time: 60 minutes

**Context:**
You'll work with Amazon's robust networking framework through Amazon VPC. In this exercise, you'll create a production-ready network architecture following AWS best practices for security and scalability.

**Prerequisites:**
- Lab-003: VPC Configuration
- Lab-005: Network Access Control

**Key Concepts You'll Use:**
- **VPC**: Your own isolated network in the AWS cloud
- **Subnets**: Your AZ-specific network segments
- **Route Tables**: Your traffic flow controllers
- **NACLs vs Security Groups**: Your stateless vs stateful firewalls

**What You'll Learn:**
- How to design enterprise VPC architecture
- Ways to implement multi-tier network segmentation
- Methods to deploy AWS security controls
- Techniques for configuring high-availability networking

**Your Implementation Steps:**
1. VPC Architecture
   - VPC Name: `practice-vpc`
   - CIDR Block: `172.16.0.0/16`
   - Enable DNS Resolution
   
   *Design Consideration:* Your /16 CIDR provides ample IP space for multiple tiers and future growth.

2. Network Segmentation
   - Public Tier: `172.16.1.0/24`
     * For your internet-facing resources
     * Where you'll place load balancers and bastion hosts
   - Private Tier: `172.16.2.0/24`
     * For your application and database servers
     * Where you'll host internal services
   - Multi-AZ Distribution
   
   *High Availability:* You're deploying across multiple AZs for resilience.

3. Connectivity Configuration
   - Deploy your Internet Gateway
     * Enables internet access for your public subnets
   - Configure your NAT Gateway
     * Allows your private instances to access internet
   - Set up your route tables
     * Controls traffic flow between your tiers
   - Associate your network tiers
   
   *Routing Tip:* Keep your public and private route tables separate for better security control.

4. Security Implementation
   - Configure your security groups
     * Your stateful firewall rules
     * Your application-specific access
   - Implement your NACLs
     * Your subnet-level security
     * Your additional security layer
   - Validate your network isolation
   
   *Security Note:* You're using both NACLs and security groups for defense in depth.

**Technical Guidance:**
- Start with VPC Wizard for initial architecture
  * Helps you streamline basic setup
  * Ensures you follow best practices
- Enable your high availability features
  * Set up multi-AZ deployment
  * Configure redundant NAT gateways
- Implement your security controls
  * Layer your security measures
  * Follow your principle of least privilege
- Monitor your resource utilization
  * Set up your VPC Flow Logs
  * Enable your CloudWatch metrics

**Common Pitfalls to Avoid:**
1. Not planning your high availability strategy
2. Overlooking the stateless nature of your NACLs
3. Making your subnets too small
4. Forgetting to enable your VPC Flow Logs

**Your Success Checklist:**
✓ You've created your VPC with appropriate CIDR
✓ You've implemented multi-AZ architecture
✓ You've configured security controls properly
✓ You've enabled high availability features
✓ You've set up monitoring and logging

## Resource Management

### Cleanup Procedures
To avoid unnecessary charges, make sure you clean up your resources after completing the exercises:

1. Microsoft Azure
   - Remove your resource groups
   - Verify your resource deletion
   - Check for any orphaned resources

2. Google Cloud Platform
   - Delete your network resources
   - Remove your firewall configurations
   - Verify your instance termination

3. Amazon Web Services
   - Terminate your compute instances
   - Remove your network components
   - Verify your elastic IP release

## Learning Outcomes
After completing these exercises, you'll have:
- Mastered cloud network architecture principles
- Gained practical security implementation experience
- Developed enterprise solution deployment skills
- Built expertise with cloud networking tools

## Professional Development
As you work through this lab, remember to:
- Document your implementation decisions
- Follow cloud provider best practices
- Consider high availability in your designs
- Implement security at every layer

For additional guidance, refer to the official documentation of each cloud provider and leverage enterprise architecture frameworks.
