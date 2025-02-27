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

### Part 2: AWS Network Infrastructure

**What you'll learn:**
- VPC design principles and implementation
- Public and private subnet configuration
- Internet and NAT Gateway setup
- Using both AWS Console and CLI for network setup

1. **Create VPC and Subnets (AWS Console):**
   * Navigate to AWS Console → VPC Dashboard
   * Click "Create VPC"
   * Choose "VPC and More" option
   * Configure:
     - Name tag: `iatd_capstone_vpc_aws`
     - IPv4 CIDR: 172.16.0.0/16
     - Number of AZs: 2
     - Public subnets: Yes
     - Private subnets: Yes
     - NAT gateways: 1 per AZ
     - VPC endpoints: S3 Gateway

2. **Configure Route Tables (AWS CLI):**
   ```bash
   # Create and configure route table for private subnet
   route_table_id=$(aws ec2 create-route-table \
     --vpc-id $vpc_id \
     --tag-specifications 'ResourceType=route-table,Tags=[{Key=Name,Value=iatd_capstone_rt_private}]' \
     --query 'RouteTable.RouteTableId' \
     --output text)

   # Add route through NAT Gateway
   aws ec2 create-route \
     --route-table-id $route_table_id \
     --destination-cidr-block 0.0.0.0/0 \
     --nat-gateway-id $nat_gateway_id
   ```

#### Part 2 Exercises: AWS Network Implementation

**Exercise 2.1: Multi-Region AWS Setup**
**Scenario:** Extend the AWS infrastructure to support a disaster recovery (DR) region.

**Hints:**
- Use different CIDR ranges for each region to avoid overlap
- Consider using Transit Gateway for simplified connectivity
- Enable route propagation where needed
- Remember to update security groups in both regions
- Use AWS Systems Manager for cross-region management

**Tasks:**
1. Create a new VPC in a different region
2. Set up VPC peering between regions
3. Configure route tables for cross-region communication
4. Implement a basic application that spans both regions

**Success Criteria:**
- Successfully establish VPC peering
- Demonstrate cross-region communication
- Show failover capabilities
- Document the DR process

**Exercise 2.2: VPC Endpoint Configuration**
**Scenario:** Implement private access to AWS services using VPC endpoints.

**Hints:**
- Start with Gateway endpoints (S3, DynamoDB) as they're free
- Use Interface endpoints for other AWS services
- Remember to update route tables for Gateway endpoints
- Check DNS settings when using Interface endpoints
- Review endpoint policies to ensure least privilege

**Tasks:**
1. Create VPC endpoints for essential AWS services (S3, DynamoDB)
2. Configure endpoint policies
3. Update route tables for endpoint access
4. Test access to services through endpoints

**Success Criteria:**
- Working VPC endpoints
- Secure service access
- Proper route configuration
- Successful access testing

### Part 3: GCP Network Infrastructure

**What you'll learn:**
- GCP VPC architecture and design
- Subnet creation and management
- Cloud Router and NAT configuration
- Using both GCP Console and gcloud CLI

1. **Create VPC Network (GCP Console):**
   * Navigate to GCP Console → VPC Networks
   * Click "Create VPC Network"
   * Configure:
     - Name: `iatd_capstone_vpc_gcp`
     - Subnet creation mode: Custom
     - New subnet:
       * Name: `iatd_capstone_subnet_gcp_public`
       * Region: us-central1
       * IP range: 192.168.1.0/24
     - Add another subnet:
       * Name: `iatd_capstone_subnet_gcp_private`
       * Region: us-central1
       * IP range: 192.168.2.0/24

2. **Configure Cloud Router and NAT (gcloud CLI):**
   ```bash
   # Create Cloud Router
   gcloud compute routers create iatd_capstone_router_gcp \
     --network=iatd_capstone_vpc_gcp \
     --region=us-central1

   # Configure Cloud NAT
   gcloud compute routers nats create iatd_capstone_nat_gcp \
     --router=iatd_capstone_router_gcp \
     --router-region=us-central1 \
     --nat-all-subnet-ip-ranges \
     --auto-allocate-nat-external-ips
   ```

#### Part 3 Exercises: GCP Network Security

**Exercise 3.1: Network Security Implementation**
**Scenario:** Implement a secure network architecture for a three-tier application.

**Hints:**
- Use hierarchical firewall rules for better management
- Consider using network tags for service identification
- Implement proper egress rules for each tier
- Use Cloud Armor preview mode before enforcement
- Remember to enable flow logs for troubleshooting

**Tasks:**
1. Create separate subnets for web, app, and database tiers
2. Configure appropriate firewall rules for each tier
3. Set up Cloud NAT for outbound internet access
4. Implement Cloud Armor for DDoS protection

**Success Criteria:**
- Proper network segmentation
- Secure communication between tiers
- Working outbound internet access
- Successfully block unauthorized access

**Exercise 3.2: VPC Service Controls**
**Scenario:** Implement VPC Service Controls to secure cloud resources.

**Hints:**
- Start with a dry-run perimeter to assess impact
- Use access levels to define trusted identities
- Consider using bridge perimeters for shared services
- Test thoroughly before enforcing policies
- Monitor VPC Service Controls logs for issues

**Tasks:**
1. Create a service perimeter
2. Configure access levels
3. Define resource policies
4. Test service isolation

**Success Criteria:**
- Working service perimeter
- Proper access controls
- Resource protection
- Successful isolation testing

### Part 4: Cross-Cloud Connectivity

**What you'll learn:**
- Setting up secure cross-cloud communication
- VPN Gateway configuration
- Transit Gateway implementation
- Understanding hybrid cloud networking

1. **Azure VPN Gateway Setup (Mixed Approach):**
   * Using Azure Portal:
     - Navigate to Virtual Network Gateway
     - Click "Create"
     - Configure:
       * Name: `iatd_capstone_vpn_azure`
       * Gateway type: VPN
       * SKU: VpnGw1
       * Virtual network: iatd_capstone_vnet_azure
       * Public IP: Create new

   * Using Azure CLI for connection:
   ```powershell
   # Create Local Network Gateway
   az network local-gateway create \
     --resource-group iatd_capstone_rg \
     --name "AWS-Connection" \
     --gateway-ip-address $aws_vpn_public_ip \
     --local-address-prefixes 172.16.0.0/16
   ```

#### Part 4 Exercises: Multi-Cloud Integration

**Exercise 4.1: Cross-Cloud Application**
**Scenario:** Create a hybrid application that spans all three cloud providers.

**Hints:**
- Use managed identities/roles where possible
- Consider latency when designing the architecture
- Implement retry logic for cross-cloud calls
- Use secure methods for sharing credentials
- Monitor costs across all cloud providers

**Tasks:**
1. Set up a simple web application that:
   - Stores data in Azure SQL Database
   - Uses AWS S3 for file storage
   - Utilizes GCP Cloud Functions for processing
2. Configure cross-cloud networking
3. Implement proper security measures
4. Monitor cross-cloud latency

**Success Criteria:**
- Application works across all clouds
- Secure data transmission
- Acceptable latency metrics
- Working monitoring solution

**Exercise 4.2: Network Performance Optimization**
**Scenario:** Optimize network performance across the multi-cloud infrastructure.

**Hints:**
- Use appropriate VM sizes for network performance
- Consider using accelerated networking where available
- Monitor bandwidth usage and latency
- Use content delivery networks where appropriate
- Test from different regions to establish baseline

**Tasks:**
1. Baseline current performance
2. Identify bottlenecks
3. Implement performance improvements
4. Document optimization process

**Success Criteria:**
- Improved network performance
- Reduced latency
- Optimized routing
- Performance monitoring dashboard

### Part 5: Security Implementation

**What you'll learn:**
- Multi-cloud security best practices
- Network security group configuration
- Firewall rules management
- Security monitoring and logging

1. **Azure Network Security Groups (Portal):**
   * Navigate to NSGs in Azure Portal
   * Create new NSG: `iatd_capstone_nsg_app`
   * Add inbound security rules:
     - Allow HTTPS (443)
     - Allow SSH from admin IP
     - Deny all other inbound traffic

2. **AWS Security Groups (CLI):**
   ```bash
   # Create and configure security group
   aws ec2 create-security-group \
     --group-name iatd_capstone_sg_web \
     --description "Web Security Group" \
     --vpc-id $vpc_id

   # Add inbound rules
   aws ec2 authorize-security-group-ingress \
     --group-id $sg_id \
     --protocol tcp \
     --port 443 \
     --cidr 0.0.0.0/0
   ```

3. **GCP Firewall Rules (Console):**
   * Navigate to VPC Network → Firewall
   * Create firewall rules:
     - Name: `iatd_capstone_fw_allow_internal`
     - Network: iatd_capstone_vpc_gcp
     - Priority: 1000
     - Direction: Ingress
     - Action: Allow
     - Targets: Specified target tags
     - Source filters: IP ranges
     - Protocols/ports: Specified protocols and ports

#### Part 5 Exercises: Multi-Cloud Security

**Exercise 5.1: Security Audit**
**Scenario:** Conduct a security audit of the multi-cloud infrastructure.

**Hints:**
- Use cloud-native security assessment tools
- Compare configurations against CIS benchmarks
- Review service endpoints and exposed ports
- Check for unused or misconfigured resources
- Document all security findings systematically

**Tasks:**
1. Review and document:
   - Network security groups
   - Firewall rules
   - VPN configurations
   - Access controls
2. Identify potential vulnerabilities
3. Implement security best practices
4. Create a security compliance report

**Success Criteria:**
- Complete security audit report
- Implemented security improvements
- Updated security documentation
- Compliance validation

**Exercise 5.2: Security Monitoring**
**Scenario:** Implement comprehensive security monitoring for the multi-cloud infrastructure.

**Hints:**
- Use log analytics for centralized monitoring
- Set up meaningful alert thresholds
- Consider using SIEM solutions for correlation
- Create runbooks for common security events
- Test alert notifications end-to-end

**Tasks:**
1. Set up monitoring in each cloud:
   - Azure Network Watcher
   - AWS CloudWatch
   - GCP Network Intelligence Center
2. Create a consolidated dashboard
3. Configure alerts for:
   - High latency
   - Failed connections
   - Security violations
   - Cost thresholds

**Success Criteria:**
- Working monitoring dashboard
- Proper alert configuration
- Incident response documentation
- Cost optimization recommendations

### Additional Capstone Challenges

#### Challenge 1: Infrastructure as Code

**Hints:**
- Start with simpler resources and build up
- Use variables for reusable values
- Implement proper state management
- Consider using modules for common patterns
- Test deployments in a separate environment

**Scenario:** Convert the manual infrastructure setup to Infrastructure as Code (IaC).

**Tasks:**
1. Create templates for:
   - Azure ARM templates
   - AWS CloudFormation
   - GCP Deployment Manager
2. Implement proper variable management
3. Set up CI/CD pipeline for infrastructure
4. Create testing procedures

**Success Criteria:**
- Working IaC templates
- Successful automated deployment
- Proper version control
- Documentation for future updates

#### Challenge 2: Disaster Recovery

**Hints:**
- Document dependencies between services
- Test DR procedures regularly
- Automate where possible
- Consider cost implications of different DR strategies
- Maintain up-to-date recovery documentation

**Scenario:** Implement a comprehensive DR strategy across all clouds.

**Tasks:**
1. Document critical infrastructure components
2. Create DR procedures for each component
3. Implement automated failover where possible
4. Test the DR plan

**Success Criteria:**
- Documented DR plan
- Successful failover testing
- Recovery time objectives (RTO) met
- Recovery point objectives (RPO) met

#### Challenge 3: Cost Optimization

**Hints:**
- Set up budget alerts
- Review costs daily during implementation
- Use cost calculators before deployment
- Clean up unused resources promptly

**Scenario:** Optimize costs while maintaining performance.

**Tasks:**
1. Review current resource usage
2. Identify cost optimization opportunities
3. Implement automated scaling
4. Create cost allocation tags

**Success Criteria:**
- Reduced infrastructure costs
- Maintained performance levels
- Working auto-scaling
- Clear cost allocation

### General Tips for All Exercises

**Planning Phase:**
- Review relevant documentation before starting
- Create a backup plan for critical changes
- Use a test environment when possible
- Document your intended approach

**Implementation Phase:**
- Make incremental changes
- Test each change before moving forward
- Keep track of all modifications
- Use version control for configurations

**Testing Phase:**
- Start with basic connectivity tests
- Progress to more complex scenarios
- Test failure scenarios
- Validate security controls

**Documentation Phase:**
- Use screenshots for visual reference
- Include command outputs
- Document any issues encountered
- Note specific configuration values

**Troubleshooting Tips:**
- Check cloud provider status pages
- Review service limits and quotas
- Use monitoring tools effectively
- Keep track of error messages

**Cost Management Tips:**
- Set up budget alerts
- Review costs daily during implementation
- Use cost calculators before deployment
- Clean up unused resources promptly

### Exercise Submission Guidelines
1. **Documentation Requirements:**
   - Solution description
   - Configuration screenshots
   - Scripts and templates
   - Lessons learned

2. **Testing Evidence:**
   - Test results
   - Performance metrics
   - Security validation
   - Cost analysis

3. **Best Practices:**
   - Follow cloud provider guidelines
   - Implement security best practices
   - Use proper naming conventions
   - Maintain clear documentation

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

### Practice Exercises

After completing each part of the lab, complete these exercises to reinforce your learning:

#### Exercise 1: Azure Network Troubleshooting
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

#### Exercise 2: Multi-Region AWS Setup
**Scenario:** Extend the AWS infrastructure to support a disaster recovery (DR) region.

**Hints:**
- Use different CIDR ranges for each region to avoid overlap
- Consider using Transit Gateway for simplified connectivity
- Enable route propagation where needed
- Remember to update security groups in both regions
- Use AWS Systems Manager for cross-region management

**Tasks:**
1. Create a new VPC in a different region
2. Set up VPC peering between regions
3. Configure route tables for cross-region communication
4. Implement a basic application that spans both regions

**Success Criteria:**
- Successfully establish VPC peering
- Demonstrate cross-region communication
- Show failover capabilities
- Document the DR process

#### Exercise 3: GCP Network Security
**Scenario:** Implement a secure network architecture for a three-tier application.

**Hints:**
- Use hierarchical firewall rules for better management
- Consider using network tags for service identification
- Implement proper egress rules for each tier
- Use Cloud Armor preview mode before enforcement
- Remember to enable flow logs for troubleshooting

**Tasks:**
1. Create separate subnets for web, app, and database tiers
2. Configure appropriate firewall rules for each tier
3. Set up Cloud NAT for outbound internet access
4. Implement Cloud Armor for DDoS protection

**Success Criteria:**
- Proper network segmentation
- Secure communication between tiers
- Working outbound internet access
- Successfully block unauthorized access

#### Exercise 4: Cross-Cloud Integration
**Scenario:** Create a hybrid application that spans all three cloud providers.

**Hints:**
- Use managed identities/roles where possible
- Consider latency when designing the architecture
- Implement retry logic for cross-cloud calls
- Use secure methods for sharing credentials
- Monitor costs across all cloud providers

**Tasks:**
1. Set up a simple web application that:
   - Stores data in Azure SQL Database
   - Uses AWS S3 for file storage
   - Utilizes GCP Cloud Functions for processing
2. Configure cross-cloud networking
3. Implement proper security measures
4. Monitor cross-cloud latency

**Success Criteria:**
- Application works across all clouds
- Secure data transmission
- Acceptable latency metrics
- Working monitoring solution

#### Exercise 5: Security Audit
**Scenario:** Conduct a security audit of the multi-cloud infrastructure.

**Hints:**
- Use cloud-native security assessment tools
- Compare configurations against CIS benchmarks
- Review service endpoints and exposed ports
- Check for unused or misconfigured resources
- Document all security findings systematically

**Tasks:**
1. Review and document:
   - Network security groups
   - Firewall rules
   - VPN configurations
   - Access controls
2. Identify potential vulnerabilities
3. Implement security best practices
4. Create a security compliance report

**Success Criteria:**
- Complete security audit report
- Implemented security improvements
- Updated security documentation
- Compliance validation

#### Exercise 6: Disaster Recovery Plan
**Scenario:** Create and test a disaster recovery plan for the multi-cloud infrastructure.

**Hints:**
- Document dependencies between services
- Test DR procedures regularly
- Automate where possible
- Consider cost implications of different DR strategies
- Maintain up-to-date recovery documentation

**Tasks:**
1. Document critical infrastructure components
2. Create DR procedures for each component
3. Implement automated failover where possible
4. Test the DR plan

**Success Criteria:**
- Documented DR plan
- Successful failover testing
- Recovery time objectives (RTO) met
- Recovery point objectives (RPO) met

#### Exercise 7: Cost Optimization
**Scenario:** Optimize the cost of the multi-cloud infrastructure while maintaining performance.

**Hints:**
- Set up budget alerts
- Review costs daily during implementation
- Use cost calculators before deployment
- Clean up unused resources promptly

**Tasks:**
1. Review current resource usage
2. Identify cost optimization opportunities
3. Implement automated scaling
4. Create cost allocation tags

**Success Criteria:**
- Reduced infrastructure costs
- Maintained performance levels
- Working auto-scaling
- Clear cost allocation

#### Exercise 8: Infrastructure as Code Challenge
**Scenario:** Convert the manual infrastructure setup to Infrastructure as Code (IaC).

**Hints:**
- Start with simpler resources and build up
- Use variables for reusable values
- Implement proper state management
- Consider using modules for common patterns
- Test deployments in a separate environment

**Tasks:**
1. Create templates for:
   - Azure ARM templates
   - AWS CloudFormation
   - GCP Deployment Manager
2. Implement proper variable management
3. Set up CI/CD pipeline for infrastructure
4. Create testing procedures

**Success Criteria:**
- Working IaC templates
- Successful automated deployment
- Proper version control
- Documentation for future updates

#### Exercise 9: Performance Tuning
**Scenario:** Optimize network performance across the multi-cloud infrastructure.

**Hints:**
- Use appropriate VM sizes for network performance
- Consider using accelerated networking where available
- Monitor bandwidth usage and latency
- Use content delivery networks where appropriate
- Test from different regions to establish baseline

**Tasks:**
1. Baseline current performance
2. Identify bottlenecks
3. Implement performance improvements
4. Document optimization process

**Success Criteria:**
- Improved network performance
- Reduced latency
- Optimized routing
- Performance monitoring dashboard

#### Exercise 10: Network Monitoring Challenge
**Scenario:** Implement comprehensive monitoring for the multi-cloud infrastructure.

**Hints:**
- Use log analytics for centralized monitoring
- Set up meaningful alert thresholds
- Consider using SIEM solutions for correlation
- Create runbooks for common security events
- Test alert notifications end-to-end

**Tasks:**
1. Set up monitoring in each cloud:
   - Azure Network Watcher
   - AWS CloudWatch
   - GCP Network Intelligence Center
2. Create a consolidated dashboard
3. Configure alerts for:
   - High latency
   - Failed connections
   - Security violations
   - Cost thresholds

**Success Criteria:**
- Working monitoring dashboard
- Proper alert configuration
- Incident response documentation
- Cost optimization recommendations

### Tips for Completing Exercises
1. **Documentation:** Keep detailed notes of your configurations and findings
2. **Testing:** Always verify your changes in a methodical way
3. **Security:** Consider security implications of every change
4. **Monitoring:** Use monitoring tools to validate your changes
5. **Cost:** Keep track of any resources you create
6. **Cleanup:** Remember to remove or stop resources when done

### Exercise Submission
For each exercise, prepare:
1. A brief report documenting your solution
2. Screenshots of key configurations
3. Any scripts or templates created
4. Lessons learned and challenges faced

### Cleanup Instructions

1. **Azure Cleanup:**
   ```powershell
   # Remove Resource Group and all resources
   Remove-AzResourceGroup -Name $rgName -Force
   ```

2. **AWS Cleanup:**
   ```bash
   # Delete VPC and associated resources
   aws ec2 delete-vpc --vpc-id $vpc_id
   ```

3. **GCP Cleanup:**
   ```bash
   # Delete VPC and associated resources
   gcloud compute networks delete iatd_capstone_vpc_gcp
   ```

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

### Practice Exercises

After completing each part of the lab, complete these exercises to reinforce your learning:

#### Exercise 1: Azure Network Troubleshooting
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

#### Exercise 2: Multi-Region AWS Setup
**Scenario:** Extend the AWS infrastructure to support a disaster recovery (DR) region.

**Hints:**
- Use different CIDR ranges for each region to avoid overlap
- Consider using Transit Gateway for simplified connectivity
- Enable route propagation where needed
- Remember to update security groups in both regions
- Use AWS Systems Manager for cross-region management

**Tasks:**
1. Create a new VPC in a different region
2. Set up VPC peering between regions
3. Configure route tables for cross-region communication
4. Implement a basic application that spans both regions

**Success Criteria:**
- Successfully establish VPC peering
- Demonstrate cross-region communication
- Show failover capabilities
- Document the DR process

#### Exercise 3: GCP Network Security
**Scenario:** Implement a secure network architecture for a three-tier application.

**Hints:**
- Use hierarchical firewall rules for better management
- Consider using network tags for service identification
- Implement proper egress rules for each tier
- Use Cloud Armor preview mode before enforcement
- Remember to enable flow logs for troubleshooting

**Tasks:**
1. Create separate subnets for web, app, and database tiers
2. Configure appropriate firewall rules for each tier
3. Set up Cloud NAT for outbound internet access
4. Implement Cloud Armor for DDoS protection

**Success Criteria:**
- Proper network segmentation
- Secure communication between tiers
- Working outbound internet access
- Successfully block unauthorized access

#### Exercise 4: Cross-Cloud Integration
**Scenario:** Create a hybrid application that spans all three cloud providers.

**Hints:**
- Use managed identities/roles where possible
- Consider latency when designing the architecture
- Implement retry logic for cross-cloud calls
- Use secure methods for sharing credentials
- Monitor costs across all cloud providers

**Tasks:**
1. Set up a simple web application that:
   - Stores data in Azure SQL Database
   - Uses AWS S3 for file storage
   - Utilizes GCP Cloud Functions for processing
2. Configure cross-cloud networking
3. Implement proper security measures
4. Monitor cross-cloud latency

**Success Criteria:**
- Application works across all clouds
- Secure data transmission
- Acceptable latency metrics
- Working monitoring solution

#### Exercise 5: Security Audit
**Scenario:** Conduct a security audit of the multi-cloud infrastructure.

**Hints:**
- Use cloud-native security assessment tools
- Compare configurations against CIS benchmarks
- Review service endpoints and exposed ports
- Check for unused or misconfigured resources
- Document all security findings systematically

**Tasks:**
1. Review and document:
   - Network security groups
   - Firewall rules
   - VPN configurations
   - Access controls
2. Identify potential vulnerabilities
3. Implement security best practices
4. Create a security compliance report

**Success Criteria:**
- Complete security audit report
- Implemented security improvements
- Updated security documentation
- Compliance validation

#### Exercise 6: Disaster Recovery Plan
**Scenario:** Create and test a disaster recovery plan for the multi-cloud infrastructure.

**Hints:**
- Document dependencies between services
- Test DR procedures regularly
- Automate where possible
- Consider cost implications of different DR strategies
- Maintain up-to-date recovery documentation

**Tasks:**
1. Document critical infrastructure components
2. Create DR procedures for each component
3. Implement automated failover where possible
4. Test the DR plan

**Success Criteria:**
- Documented DR plan
- Successful failover testing
- Recovery time objectives (RTO) met
- Recovery point objectives (RPO) met

#### Exercise 7: Cost Optimization
**Scenario:** Optimize the cost of the multi-cloud infrastructure while maintaining performance.

**Hints:**
- Set up budget alerts
- Review costs daily during implementation
- Use cost calculators before deployment
- Clean up unused resources promptly

**Tasks:**
1. Review current resource usage
2. Identify cost optimization opportunities
3. Implement automated scaling
4. Create cost allocation tags

**Success Criteria:**
- Reduced infrastructure costs
- Maintained performance levels
- Working auto-scaling
- Clear cost allocation

#### Exercise 8: Infrastructure as Code Challenge
**Scenario:** Convert the manual infrastructure setup to Infrastructure as Code (IaC).

**Hints:**
- Start with simpler resources and build up
- Use variables for reusable values
- Implement proper state management
- Consider using modules for common patterns
- Test deployments in a separate environment

**Tasks:**
1. Create templates for:
   - Azure ARM templates
   - AWS CloudFormation
   - GCP Deployment Manager
2. Implement proper variable management
3. Set up CI/CD pipeline for infrastructure
4. Create testing procedures

**Success Criteria:**
- Working IaC templates
- Successful automated deployment
- Proper version control
- Documentation for future updates

#### Exercise 9: Performance Tuning
**Scenario:** Optimize network performance across the multi-cloud infrastructure.

**Hints:**
- Use appropriate VM sizes for network performance
- Consider using accelerated networking where available
- Monitor bandwidth usage and latency
- Use content delivery networks where appropriate
- Test from different regions to establish baseline

**Tasks:**
1. Baseline current performance
2. Identify bottlenecks
3. Implement performance improvements
4. Document optimization process

**Success Criteria:**
- Improved network performance
- Reduced latency
- Optimized routing
- Performance monitoring dashboard

#### Exercise 10: Network Monitoring Challenge
**Scenario:** Implement comprehensive monitoring for the multi-cloud infrastructure.

**Hints:**
- Use log analytics for centralized monitoring
- Set up meaningful alert thresholds
- Consider using SIEM solutions for correlation
- Create runbooks for common security events
- Test alert notifications end-to-end

**Tasks:**
1. Set up monitoring in each cloud:
   - Azure Network Watcher
   - AWS CloudWatch
   - GCP Network Intelligence Center
2. Create a consolidated dashboard
3. Configure alerts for:
   - High latency
   - Failed connections
   - Security violations
   - Cost thresholds

**Success Criteria:**
- Working monitoring dashboard
- Proper alert configuration
- Incident response documentation
- Cost optimization recommendations

### Tips for Completing Exercises
1. **Documentation:** Keep detailed notes of your configurations and findings
2. **Testing:** Always verify your changes in a methodical way
3. **Security:** Consider security implications of every change
4. **Monitoring:** Use monitoring tools to validate your changes
5. **Cost:** Keep track of any resources you create
6. **Cleanup:** Remember to remove or stop resources when done

### Exercise Submission
For each exercise, prepare:
1. A brief report documenting your solution
2. Screenshots of key configurations
3. Any scripts or templates created
4. Lessons learned and challenges faced

### Cleanup Instructions

1. **Azure Cleanup:**
   ```powershell
   # Remove Resource Group and all resources
   Remove-AzResourceGroup -Name $rgName -Force
   ```

2. **AWS Cleanup:**
   ```bash
   # Delete VPC and associated resources
   aws ec2 delete-vpc --vpc-id $vpc_id
   ```

3. **GCP Cleanup:**
   ```bash
   # Delete VPC and associated resources
   gcloud compute networks delete iatd_capstone_vpc_gcp
   ```

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

```
