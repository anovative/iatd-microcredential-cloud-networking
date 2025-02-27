## IATD Microcredential Cloud Networking: Capstone Lab - Multi-Cloud Network Infrastructure

**Objective:** In this comprehensive capstone lab, you will practice and reinforce your multi-cloud networking skills by implementing infrastructure spanning Azure, AWS, and GCP. This hands-on lab will help you apply concepts learned from previous labs in a practical, real-world scenario.

**Estimated Time:** 120-180 minutes

### Challenge Level System
Each exercise is categorized by difficulty to help you progress systematically:
- **Basic**: Fundamental concepts and single-cloud implementations
- **Intermediate**: Multi-component solutions and cross-cloud basics
- **Advanced**: Complex scenarios and full multi-cloud implementations

**Prerequisites:**

1. **Cloud Platform Access:**
   - Active Azure subscription
   - AWS account with appropriate permissions
   - GCP account with appropriate permissions
2. **Tools and Knowledge:**
   - Azure Cloud Shell
   - AWS CloudShell
   - GCP Cloud Shell
   - Understanding of previous labs (001-009)
   - Basic understanding of multi-cloud networking concepts

### Resource Naming Convention

All resources will follow the prefix `iatd_capstone_` with appropriate suffixes for each cloud platform:

**Azure Resources:**
- Resource Group: `iatd_capstone_rg`
- VNet: `iatd_capstone_vnet_azure`
- Subnets: `iatd_capstone_subnet_[purpose]`
- NSG: `iatd_capstone_nsg_[purpose]`
- Private DNS Zone: `capstone.azure.internal`
- VPN Gateway: `iatd_capstone_vpn_azure`

**AWS Resources:**
- VPC: `iatd_capstone_vpc_aws`
- Subnets: `iatd_capstone_subnet_aws_[purpose]`
- Security Groups: `iatd_capstone_sg_[purpose]`
- Transit Gateway: `iatd_capstone_tgw_aws`

**GCP Resources:**
- VPC: `iatd_capstone_vpc_gcp`
- Subnets: `iatd_capstone_subnet_gcp_[purpose]`
- Firewall Rules: `iatd_capstone_fw_[purpose]`
- Cloud Router: `iatd_capstone_router_gcp`

### IP Address Allocation

To avoid IP conflicts across clouds:
- Azure VNet: 10.0.0.0/16
  - Application Subnet: 10.0.1.0/24
  - Database Subnet: 10.0.2.0/24
  - Gateway Subnet: 10.0.3.0/24
- AWS VPC: 172.16.0.0/16
  - Public Subnet: 172.16.1.0/24
  - Private Subnet: 172.16.2.0/24
- GCP VPC: 192.168.0.0/16
  - Public Subnet: 192.168.1.0/24
  - Private Subnet: 192.168.2.0/24

### Part 1: Azure Network Infrastructure

**What you'll learn:**
- How to create a well-structured Virtual Network with multiple subnets
- Implementation of Private DNS for internal name resolution
- Proper network segmentation for different application tiers
- Both Azure CLI and Portal approaches for network management

1. **Create Base Infrastructure (Azure Portal):**
   * Navigate to the Azure Portal (https://portal.azure.com)
   * Click "Create a resource" → "Resource group"
   * Fill in the details:
     - Resource group name: `iatd_capstone_rg`
     - Region: Your preferred region
     - Click "Review + create" → "Create"
   
   * Once created, navigate to "Virtual networks" → "Create"
   * Basic tab:
     - Resource group: `iatd_capstone_rg`
     - Name: `iatd_capstone_vnet_azure`
     - Region: Same as resource group
   * IP Addresses tab:
     - IPv4 address space: 10.0.0.0/16
     - Add subnets:
       * Name: ApplicationSubnet
       * Address range: 10.0.1.0/24
       * Name: DatabaseSubnet
       * Address range: 10.0.2.0/24

2. **Configure Private DNS Zone (Azure CLI):**
   ```powershell
   # Create Private DNS Zone
   az network private-dns zone create \
     --resource-group iatd_capstone_rg \
     --name "capstone.azure.internal"

   # Link to VNet
   az network private-dns link vnet create \
     --resource-group iatd_capstone_rg \
     --zone-name "capstone.azure.internal" \
     --name "VNetLink" \
     --virtual-network "iatd_capstone_vnet_azure" \
     --registration-enabled true
   ```

#### Part 1 Exercises: Azure Network Foundation

**Exercise 1.1: Network Troubleshooting**
**Scenario:** A development team reports they cannot access the database subnet from the application subnet.

**Hints:**
- Use Network Watcher's IP flow verify to check connectivity
- Review NSG flow logs for denied traffic
- Check effective security rules for both subnets
- Don't forget to verify route table associations
- Use Test-NetConnection or curl to test connectivity

**Tasks:**
1. Review and validate the NSG rules between subnets
2. Check if the Private DNS zone is resolving names correctly
3. Verify the route tables are configured properly
4. Document your findings and solution

**Success Criteria:**
- Identify the root cause of the connectivity issue
- Implement the fix using both Portal and CLI
- Verify connectivity between subnets
- Create a troubleshooting runbook for future reference

**Exercise 1.2: Private DNS Zone Management**
**Scenario:** Implement a hierarchical DNS structure for different environments (dev, test, prod).

**Hints:**
- Consider using a naming convention like dev.internal, test.internal, prod.internal
- Remember to enable auto-registration where appropriate
- Use conditional forwarding for cross-environment resolution
- Test using nslookup from a VM in each subnet

**Tasks:**
1. Create additional Private DNS zones for each environment
2. Configure DNS record sets for key services
3. Implement proper DNS resolution between environments
4. Test DNS resolution from different subnets

**Success Criteria:**
- Working DNS resolution across environments
- Proper zone linking
- Documented DNS hierarchy
- Successful resolution testing

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

**What You'll Practice:**
- VPC architecture
- Public/Private subnet design
- Route table configuration

**Step-by-Step Tasks:**
1. Create a VPC:
   - Name: `practice-vpc`
   - CIDR: `172.16.0.0/16`
   - Enable DNS hostnames

2. Create Subnets:
   - Public subnet: `172.16.1.0/24`
     * Enable auto-assign public IP
   - Private subnet: `172.16.2.0/24`
     * Disable auto-assign public IP

3. Configure Routing:
   - Create an Internet Gateway
   - Create route tables for each subnet
   - Add appropriate routes

4. Set Up Security:
   - Create security groups
   - Configure basic inbound/outbound rules
   - Test connectivity

**Helpful Tips:**
- Draw your network design first
- Use AWS VPC Wizard for guidance
- Test connectivity after each step
- Document your configuration choices

### Learning Path

Follow this sequence for the best learning experience:

1. Start with Azure (Exercise 1):
   - Familiar Azure Portal interface
   - Clear visualization of networking concepts
   - Easy-to-use troubleshooting tools

2. Move to GCP (Exercise 2):
   - Similar concepts, different implementation
   - Simpler firewall rule structure
   - Good transition step

3. Finish with AWS (Exercise 3):
   - More complex networking model
   - Builds on previous learning
   - Great for understanding differences

**Key Tips for Success:**
- Take breaks between exercises
- Review related labs if needed
- Document what you learn
- Don't rush - focus on understanding

### Resource Cleanup

 Important: Clean up resources after practice to avoid charges!

1. Azure Cleanup:
   - Delete the practice-vnet resource group
   - Verify all resources are removed
   - Check Azure Portal for any remaining items

2. GCP Cleanup:
   - Delete the practice-vpc network
   - Remove all firewall rules
   - Verify in GCP Console

3. AWS Cleanup:
   - Delete the practice-vpc
   - Remove security groups
   - Check AWS Console for complete cleanup

### Key Learning Outcomes

After completing these exercises, you will:
1. Understand basic networking in three major clouds
2. Know how to create and secure network resources
3. Appreciate the differences between cloud providers
4. Have hands-on experience with cloud networking tools

Remember: These are practice exercises - feel free to experiment and learn from any mistakes!

### Additional Challenges

1. Implement load balancing across clouds
2. Set up cross-cloud monitoring
3. Create automated deployment scripts
4. Implement disaster recovery scenarios
5. Add container networking components

### Resource Cleanup

After completing your practice:
1. Delete all Azure resources
2. Remove AWS resources
3. Clean up GCP resources

This ensures you don't incur unnecessary charges.

### Key Learning Outcomes

After completing this capstone lab, you will have:
1. Designed and implemented a multi-cloud network infrastructure
2. Created secure and isolated network segments across clouds
3. Established cross-cloud connectivity
4. Implemented proper security controls
5. Gained practical experience with three major cloud providers
6. Applied concepts from all previous labs in a real-world scenario

### Additional Challenges

1. Implement load balancing across clouds
2. Set up cross-cloud monitoring
3. Create automated deployment scripts
4. Implement disaster recovery scenarios
5. Add container networking components
