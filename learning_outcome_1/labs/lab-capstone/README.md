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

## Exercise Framework
The exercises are structured according to complexity levels to help you progress:
- **Foundation Level**: Core concepts and single-cloud implementations
- **Professional Level**: Advanced solutions and cross-cloud architectures

## Laboratory Exercises

### Exercise 1: Enterprise Network Design in Azure
 **Foundation Level** | Estimated Time: 45 minutes

**Context:**
In modern enterprise environments, a well-architected virtual network is crucial for application security and performance. This exercise simulates a real-world scenario where you'll create a secure, multi-tier network architecture for a business application.

**Prerequisites:** 
- Lab-001: Virtual Network Creation
- Lab-002: Network Security Groups

**Learning Objectives:**
- Implement enterprise-grade virtual networks
- Configure network segmentation
- Deploy security controls

**Key Concepts:**
- **Virtual Networks (VNets)**: Azure's networking foundation that provides isolation, segmentation, and secure communication
- **Subnets**: Network segments that help organize and secure resources
- **Network Security Groups (NSGs)**: Virtual firewalls that control inbound and outbound traffic

**Implementation Steps:**
1. Virtual Network Configuration
   - Network Name: `practice-vnet`
   - Address Space: `10.0.0.0/16`
   - Region: Select based on latency requirements
   
   *Why these settings?* The /16 address space provides 65,534 available IP addresses, allowing room for future growth. Choose a region close to your users for better performance.

2. Network Segmentation
   - Web Tier: `10.0.1.0/24` (251 available IPs)
   - Database Tier: `10.0.2.0/24` (251 available IPs)
   
   *Design Consideration:* Separate tiers provide security isolation and clear traffic boundaries.

3. Security Implementation
   - Configure Network Security Groups (NSGs)
     * Web tier: Allow inbound HTTP/HTTPS (ports 80/443)
     * Database tier: Allow only web tier traffic on port 3306
   - Implement tier-based access controls
   - Enable database connectivity (port 3306)
   - Apply security best practices
   
   *Security Note:* Following the principle of least privilege, each tier only allows necessary traffic.

4. Validation Procedures
   - Utilize Network Watcher
   - Validate security rules
   - Perform connectivity testing
   
   *Verification Tip:* Use Network Watcher's IP flow verify feature to troubleshoot connectivity issues.

**Technical Guidance:**
- Leverage Azure Portal for initial configuration
  * Visual interface helps understand relationships between components
  * Great for learning and initial setup
- Implement infrastructure as code using Azure CLI
  * Enables automation and repeatability
  * Essential for enterprise deployments
- Utilize Network Watcher for troubleshooting
  * Provides packet capture capabilities
  * Helps verify security rules
- Document configuration changes
  * Keep track of all modifications
  * Important for compliance and auditing

**Common Pitfalls to Avoid:**
1. Not planning IP address space properly
2. Overlooking NSG rule priorities
3. Forgetting to document security rule changes
4. Not testing connectivity between tiers

**Success Criteria:**
 Virtual network created with proper address space
 Subnets configured with appropriate sizing
 NSGs implemented with secure rules
 Successful connectivity between tiers
 All changes documented

### Exercise 2: GCP Network Architecture
 **Foundation Level** | Estimated Time: 45 minutes

**Context:**
Google Cloud Platform's networking model differs from other cloud providers, offering unique features like global VPCs and automatic subnet routing. This exercise walks you through creating a secure, scalable network infrastructure in GCP.

**Prerequisites:**
- Lab-004: Network Security
- Lab-006: Application Gateway

**Key Concepts:**
- **VPC Networks**: Global resources that can span multiple regions
- **Subnets**: Regional resources for resource organization
- **Firewall Rules**: Global resources that control traffic flow
- **Cloud Router**: Enables dynamic route advertisement

**Learning Objectives:**
- Design VPC networks following GCP's global approach
- Implement GCP's networking model effectively
- Configure enterprise security controls
- Understand GCP-specific networking features

**Implementation Steps:**
1. VPC Configuration
   - Network Name: `practice-vpc`
   - Network Type: Custom Mode
   - Region: Based on compliance requirements
   
   *Why Custom Mode?* Provides full control over subnet creation and IP ranges, essential for enterprise environments.

2. Subnet Architecture
   - Subnet Name: `practice-subnet`
   - CIDR Range: `192.168.1.0/24`
   - Private Google Access: Enabled
   
   *Design Note:* Private Google Access allows VM instances without external IPs to access Google APIs and services.

3. Security Controls
   - Implement firewall rules:
     * SSH (TCP 22): Allow administrative access
     * HTTP (TCP 80): Enable web traffic
     * HTTPS (TCP 443): Secure web communication
   - Configure rule priorities
   
   *Security Best Practice:* Always use the principle of least privilege when configuring firewall rules.

4. Architecture Validation
   - Verify subnet deployment
   - Validate firewall configurations
   - Test network connectivity
   
   *Testing Tip:* Use Cloud Shell to verify connectivity and test firewall rules.

**Technical Guidance:**
- Utilize GCP Console for visual configuration
  * Helps understand the global nature of VPC
  * Provides clear visibility of network components
- Implement using gcloud CLI for automation
  * Essential for repeatable deployments
  * Enables infrastructure as code
- Follow security best practices
  * Use service accounts appropriately
  * Implement proper IAM roles
- Maintain configuration documentation
  * Track all network changes
  * Document design decisions

**Common Pitfalls to Avoid:**
1. Forgetting to enable Private Google Access
2. Misconfiguring firewall rule priorities
3. Not planning for future subnet expansion
4. Overlooking logging and monitoring setup

**Success Criteria:**
 VPC network properly configured
 Subnet created with appropriate CIDR range
 Firewall rules implemented securely
 Private Google Access enabled
 Network connectivity verified

### Exercise 3: AWS Enterprise Network Design
 **Professional Level** | Estimated Time: 60 minutes

**Context:**
Amazon Web Services provides a robust networking framework through Amazon VPC. This exercise focuses on creating a production-ready network architecture that follows AWS best practices for security and scalability.

**Prerequisites:**
- Lab-003: VPC Configuration
- Lab-005: Network Access Control

**Key Concepts:**
- **VPC**: Your isolated network in the AWS cloud
- **Subnets**: AZ-specific network segments
- **Route Tables**: Control traffic flow between subnets
- **NACLs vs Security Groups**: Stateless vs stateful firewalls

**Learning Objectives:**
- Design enterprise VPC architecture
- Implement multi-tier network segmentation
- Deploy AWS security controls
- Configure high-availability networking

**Implementation Steps:**
1. VPC Architecture
   - VPC Name: `practice-vpc`
   - CIDR Block: `172.16.0.0/16`
   - Enable DNS Resolution
   
   *Design Consideration:* The /16 CIDR provides ample IP space for multiple tiers and future growth.

2. Network Segmentation
   - Public Tier: `172.16.1.0/24`
     * Internet-facing resources
     * Load balancers and bastion hosts
   - Private Tier: `172.16.2.0/24`
     * Application and database servers
     * Internal services
   - Multi-AZ Distribution
   
   *High Availability:* Deploy across multiple AZs for resilience.

3. Connectivity Configuration
   - Deploy Internet Gateway
     * Enable internet access for public subnets
   - Configure NAT Gateway
     * Allow private instances to access internet
   - Implement route tables
     * Control traffic flow between tiers
   - Associate network tiers
   
   *Routing Tip:* Keep public and private route tables separate for better security control.

4. Security Implementation
   - Configure security groups
     * Stateful firewall rules
     * Application-specific access
   - Implement NACLs
     * Subnet-level security
     * Additional security layer
   - Validate network isolation
   
   *Security Note:* Use both NACLs and security groups for defense in depth.

**Technical Guidance:**
- Utilize VPC Wizard for initial architecture
  * Streamlines basic setup
  * Ensures best practices
- Enable high availability features
  * Multi-AZ deployment
  * Redundant NAT gateways
- Implement proper security controls
  * Layer security measures
  * Follow principle of least privilege
- Monitor resource utilization
  * Set up VPC Flow Logs
  * Enable CloudWatch metrics

**Common Pitfalls to Avoid:**
1. Not planning for high availability
2. Overlooking NACL stateless nature
3. Insufficient subnet sizing
4. Forgetting to enable VPC Flow Logs

**Success Criteria:**
 VPC created with appropriate CIDR
 Multi-AZ architecture implemented
 Security controls properly configured
 High availability features enabled
 Monitoring and logging configured

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
