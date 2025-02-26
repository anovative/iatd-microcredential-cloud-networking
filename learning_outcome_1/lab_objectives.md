# Learning Outcome 1: Design and Implement Core Network Infrastructure
## Lab Objectives Summary

This document provides a concise summary of all lab objectives for Learning Outcome 1 in a format suitable for inclusion in course documentation.

### Lab 1: Virtual Network Creation
- Create Azure Virtual Networks (VNets) using multiple methods:
  - Azure Portal
  - Azure PowerShell
  - Azure CLI
  - ARM templates
- Understand the core building blocks of private networks in Azure
- Configure address spaces and basic network settings

### Lab 2: Working with Subnets
- Create and configure subnets within Azure Virtual Networks
- Associate Network Security Groups (NSGs) with subnets
- Configure Route Tables to control network traffic
- Implement service endpoints for secure access to Azure services
- Apply subnet delegation for Azure services

### Lab 3: Creating a Dual-Stack Virtual Network and Virtual Machine
- Create an Azure Virtual Network with both IPv4 and IPv6 address spaces
- Deploy a Windows virtual machine into a dual-stack VNet
- Configure both IPv4 and IPv6 addresses for the virtual machine
- Implement and test dual-stack connectivity
- Use multiple deployment methods (Portal, CLI, PowerShell)

### Lab 4: Setting up a Site-to-Site VPN Between Azure and AWS
- Establish a secure Site-to-Site VPN connection between cloud environments
- Configure an Azure Virtual Network Gateway
- Set up an AWS Virtual Private Gateway
- Create and configure connection objects in both clouds
- Test and verify cross-cloud connectivity
- Troubleshoot common VPN connection issues

### Lab 5: Assigning IP Addresses to Virtual Machines in Azure
- Configure static private IP addresses for Azure VMs
- Assign and manage public IP addresses
- Implement dynamic IP allocation
- Understand IP address assignment methods and their use cases
- Verify IP configurations using various tools

### Lab 6: Creating Virtual Machines with Public/Private IPs and NAT Gateway
- Create VMs with public IP addresses
- Deploy VMs with only private IP addresses
- Configure a NAT Gateway to provide outbound internet access
- Implement network address translation for private resources
- Test and verify outbound connectivity through NAT Gateway

### Lab 7: Using Azure Site Recovery to Replicate Virtual Machines
- Configure Azure Site Recovery for VM replication
- Set up replication to a secondary Azure region
- Create recovery plans for organized failover
- Perform test failovers to validate disaster recovery
- Understand disaster recovery concepts and best practices

### Lab 8: Creating and Configuring an Azure DNS Zone
- Create an Azure DNS zone
- Add various types of DNS records (A, CNAME, MX, TXT)
- Delegate a domain to Azure DNS
- Configure DNS record sets with appropriate TTL values
- Test and verify DNS resolution

### Lab 9: Implementing Private DNS Zones in Azure
- Create a private DNS zone in Azure
- Link private DNS zones to virtual networks
- Add DNS records for internal name resolution
- Configure and verify name resolution from VMs
- Understand private DNS concepts and use cases

## Summary of Skills Developed

Through these labs, students will develop the following skills:

1. **Azure Virtual Network Design and Implementation**
   - Address space planning
   - Subnet configuration
   - IP addressing (IPv4 and IPv6)
   - Network security implementation

2. **Cross-Cloud Connectivity**
   - Site-to-Site VPN configuration
   - Hybrid network architecture

3. **Name Resolution**
   - Public DNS configuration
   - Private DNS implementation
   - DNS record management

4. **IP Address Management**
   - Static vs. dynamic allocation
   - Public vs. private addressing
   - NAT implementation

5. **Disaster Recovery**
   - VM replication
   - Failover testing and implementation
   - Recovery planning

6. **Azure Administration Methods**
   - Portal-based management
   - PowerShell automation
   - Azure CLI scripting
   - ARM template deployment
