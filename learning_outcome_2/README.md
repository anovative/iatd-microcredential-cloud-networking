# IATD Microcredential: Cloud Networking
## Learning Outcome 2: Design and Implement Advanced Network Infrastructure

This directory contains hands-on lab exercises for Learning Outcome 2 of the IATD Microcredential Cloud Networking course. These labs focus on Azure routing configurations, providing practical experience with User-Defined Routes (UDRs), system routes, and network traffic control.

## Learning Outcome Overview

Learning Outcome 2 focuses on mastering Azure routing concepts and implementations. Through these labs, you will gain hands-on experience with:
- Configuring User-Defined Routes (UDRs) and understanding their behavior
- Understanding Azure system routes and route prioritization
- Implementing Network Virtual Appliances (NVAs) for traffic inspection and optimization
- Managing traffic flow using different next hop types
- Associating and verifying route tables with subnets
- Configuring and managing Virtual Network peering
- Understanding route propagation across peered networks
- Implementing cross-region routing solutions
- Optimizing network traffic using NVAs
- Handling overlapping routes and route precedence

## Prerequisites

Before starting these labs, you should have: 

1. An active Azure subscription ([https://azure.microsoft.com/free/](https://azure.microsoft.com/free/))
2. Completed Learning Outcome 1 labs
3. Understanding of:
   - Virtual networks and subnets
   - IP addressing (using 172.16.0.0/16 range)
   - Basic routing concepts
   - Network Virtual Appliances
   - VNET Peering

## Lab Structure

Each lab follows a consistent structure:
- Clear objectives and expected outcomes
- Detailed prerequisites specific to the lab
- Step-by-step instructions with expected outputs
- Verification steps with specific expected outcomes
- Comprehensive cleanup instructions to avoid unnecessary charges
- Azure CLI and PowerShell examples in Azure Cloud Shell

## Labs

### Core Labs (Azure)

These labs provide comprehensive coverage of Azure routing concepts and implementations. Through hands-on exercises, you'll learn to configure and manage various aspects of Azure routing, from basic UDRs to complex network traffic control:

### [Lab 1: User-Defined Routes for Traffic Inspection](/learning_outcome_2/labs/lab-001)
**Objective:** Configure a User-Defined Route (UDR) to route traffic from a subnet to a Network Virtual Appliance (NVA) for inspection. You'll learn to create and configure NVAs, set up UDRs, and verify traffic inspection.  
**Estimated Time:** 60-90 minutes

### [Lab 2: Examining Azure System Routes](/learning_outcome_2/labs/lab-002)
**Objective:** Explore and understand the system routes automatically configured in an Azure virtual network. You'll investigate default system routes, understand their hierarchy, and learn how they facilitate network communication.  
**Estimated Time:** 60-75 minutes

### [Lab 3: Configuring Different Next Hop Types](/learning_outcome_2/labs/lab-003)
**Objective:** Configure User Defined Routes (UDRs) with different next hop types to control traffic flow within and out of an Azure Virtual Network. You'll work with virtual appliances, peered VNets, and internet gateways.  
**Estimated Time:** 75-90 minutes

### [Lab 4: Associating Route Tables with Subnets](/learning_outcome_2/labs/lab-004)
**Objective:** Learn to associate route tables with subnets and verify effective routes to control network traffic within Azure virtual networks. You'll use Azure CLI and PowerShell to manage route tables and verify routing behavior.  
**Estimated Time:** 60-75 minutes

### [Lab 5: Route Propagation with Virtual Network Peering](/learning_outcome_2/labs/lab-005)
**Objective:** Understand how routes are propagated across peered virtual networks in Azure. You'll configure VNet peering, observe route propagation behavior, and learn to control route propagation settings.  
**Estimated Time:** 60-90 minutes

### [Lab 6: Route Prioritization with Overlapping Routes](/learning_outcome_2/labs/lab-006)
**Objective:** Learn how Azure handles overlapping routes and route prioritization. You'll create scenarios with multiple overlapping routes, understand route precedence, and verify effective routing behavior.  
**Estimated Time:** 75-90 minutes

### [Lab 7: Cross-Region Routing with Virtual Network Peering](/learning_outcome_2/labs/lab-007)
**Objective:** Implement and test routing between virtual networks in different Azure regions using global VNet peering. You'll learn about cross-region connectivity, latency considerations, and route propagation across regions.  
**Estimated Time:** 90-120 minutes

### [Lab 8: Optimizing Traffic with Network Virtual Appliance](/learning_outcome_2/labs/lab-008)
**Objective:** Deploy a Network Virtual Appliance (NVA) to optimize traffic between subnets. You'll configure traffic routing through the NVA, enable IP forwarding, and understand NVA-based network optimization scenarios.  
**Estimated Time:** 60-90 minutes

### [Capstone Lab: Multi-Cloud Advanced Routing and Traffic Optimization](/learning_outcome_2/labs/lab-capstone)
**Objective:** Apply advanced routing concepts and traffic optimization techniques across Azure, AWS, and GCP in a comprehensive capstone project. You'll implement enterprise-grade routing solutions, configure traffic optimization using cloud-native tools, and demonstrate proficiency with advanced networking features.  
**Estimated Time:** 120-180 minutes

### Supplementary Labs (AWS and GCP)

These supplementary labs extend your learning to other major cloud providers, demonstrating similar routing and traffic optimization concepts in AWS and GCP environments:

### AWS Supplementary Labs

#### [Lab 1: Transit Gateway and Advanced Routing](/learning_outcome_2/labs/sup-aws/lab-001)
**Objective:** Implement advanced routing configurations in AWS using Transit Gateway. You'll learn to connect multiple VPCs, configure route tables, and optimize traffic flow using AWS-native networking services.  
**Estimated Time:** 90-120 minutes

### GCP Supplementary Labs

#### [Lab 1: VPC Network Peering and Cloud Router](/learning_outcome_2/labs/sup-gcp/lab-001)
**Objective:** Configure VPC Network Peering and Cloud Router in Google Cloud Platform. You'll implement dynamic routing between VPC networks and optimize traffic flow using GCP's networking features.  
**Estimated Time:** 90-120 minutes

## Resource Naming Conventions

Throughout these labs, resources follow a consistent naming convention using the IP range 172.16.0.0/16:
- Resource groups: `iatd_labs_XX_rg` (where XX is the lab number)
- Virtual networks: `iatd_labs_XX_vnet` (with additional suffixes like `_main` or `_peered` when needed)
- Subnets: Purpose-based names (e.g., `protected-subnet`, `nva-subnet`, `backend-subnet`)
- Route tables: `iatd_labs_XX_rt` or `iatd_labs_XX_rt_protected`
- Virtual machines: `iatd_labs_XX_vm` with role suffix (e.g., `_nva`, `_protected`)
- Network interfaces: `iatd_labs_XX_vm_nic` or `iatd_labs_XX_nva_nic`
- Public IPs: `iatd_labs_XX_vm_pip` or `iatd_labs_XX_nva_pip`

## Best Practices

1. **Clean up resources** after completing each lab to avoid unnecessary charges
2. **Follow the naming conventions** to easily identify and manage resources
3. **Read through the entire lab** before starting to understand the full workflow
4. **Take notes** during the labs to reinforce learning
5. **Experiment** beyond the lab instructions to deepen your understanding
6. **Verify routing configurations** using:
   - `az network nic show-effective-route-table`
   - Network connectivity tests (`ping`, `traceroute`)
   - Azure Network Watcher
7. **Document route tables** and their associations for each subnet

## Troubleshooting

If you encounter routing or connectivity issues during the labs:

1. **Route Table Issues:**
   - Verify route table is correctly associated with the subnet
   - Check effective routes using `az network nic show-effective-route-table`
   - Ensure route prefixes don't conflict with system routes
   - Verify next hop types and IP addresses are correct
   - Check route prioritization when using overlapping routes

2. **NVA Issues:**
   - Confirm IP forwarding is enabled on the NVA's network interface
   - Verify IP forwarding is enabled in the NVA's OS
   - Check NSG rules allow required traffic
   - Ensure NVA is properly configured to handle routed traffic
   - Monitor NVA performance and capacity

3. **VNet Peering Issues:**
   - Verify peering status is 'Connected' on both VNets
   - Check 'Allow Gateway Transit' and 'Use Remote Gateway' settings
   - Ensure non-overlapping address spaces between peered VNets
   - Validate route propagation settings
   - For global peering, verify regional connectivity

4. **Cross-Region Routing:**
   - Consider latency between regions
   - Verify global VNet peering status
   - Check regional dependencies and failover configurations
   - Monitor bandwidth and performance metrics
   - Ensure consistent route tables across regions

5. **Connectivity Testing:**
   - Use `ping` and `traceroute` to verify traffic flow
   - Leverage Azure Network Watcher for detailed diagnostics
   - Check if VMs can reach their intended destinations
   - Verify DNS resolution is working correctly
   - Test connectivity across peered networks

6. **Common Mistakes:**
   - Route table not associated with subnet
   - Incorrect address prefixes in routes
   - Wrong next hop IP address
   - Missing or incorrect NSG rules
   - IP forwarding not enabled
   - Overlapping IP address ranges
   - Incorrect peering configurations
   - Not accounting for cross-region latency

## Additional Resources

### Core Documentation
- [Microsoft Azure Documentation](https://docs.microsoft.com/en-us/azure/)
- [Azure Networking Documentation](https://docs.microsoft.com/en-us/azure/networking/)
- [Azure Architecture Center](https://docs.microsoft.com/en-us/azure/architecture/)
- [Microsoft Learn - Azure Networking](https://docs.microsoft.com/en-us/learn/paths/azure-networking/)

### Virtual Networks and Routing
- [Azure Virtual Network Documentation](https://docs.microsoft.com/en-us/azure/virtual-network/)
- [User-Defined Routes Overview](https://docs.microsoft.com/en-us/azure/virtual-network/virtual-networks-udr-overview)
- [Route Tables and Route Prioritization](https://docs.microsoft.com/en-us/azure/virtual-network/virtual-networks-udr-overview#how-azure-selects-a-route)
- [System Routes in Azure](https://docs.microsoft.com/en-us/azure/virtual-network/virtual-networks-udr-overview#system-routes)

### VNet Peering and Cross-Region Connectivity
- [Virtual Network Peering](https://docs.microsoft.com/en-us/azure/virtual-network/virtual-network-peering-overview)
- [Global VNet Peering](https://docs.microsoft.com/en-us/azure/virtual-network/virtual-network-peering-overview#cross-region)
- [Route Propagation in Peered Networks](https://docs.microsoft.com/en-us/azure/virtual-network/virtual-network-peering-overview#routing)
- [Cross-Region Connectivity Best Practices](https://docs.microsoft.com/en-us/azure/architecture/reference-architectures/hybrid-networking/)

### Network Virtual Appliances
- [Network Virtual Appliances Overview](https://docs.microsoft.com/en-us/azure/virtual-network/virtual-networks-udr-overview#user-defined)
- [NVA High Availability](https://docs.microsoft.com/en-us/azure/architecture/reference-architectures/dmz/nva-ha)
- [NVA Performance Considerations](https://docs.microsoft.com/en-us/azure/architecture/reference-architectures/dmz/nva-ha#nva-performance-considerations)

### AWS Networking Resources
- [AWS Transit Gateway Documentation](https://docs.aws.amazon.com/vpc/latest/tgw/)
- [VPC Peering in AWS](https://docs.aws.amazon.com/vpc/latest/peering/)
- [AWS Route Tables and Routes](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Route_Tables.html)
- [AWS Network ACLs](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-network-acls.html)
- [AWS VPC Connectivity Options](https://docs.aws.amazon.com/whitepapers/latest/aws-vpc-connectivity-options/)

### GCP Networking Resources
- [GCP VPC Network Overview](https://cloud.google.com/vpc/docs/vpc)
- [Cloud Router Documentation](https://cloud.google.com/network-connectivity/docs/router)
- [VPC Network Peering](https://cloud.google.com/vpc/docs/vpc-peering)
- [GCP Routes Overview](https://cloud.google.com/vpc/docs/routes)
- [Traffic Director and Load Balancing](https://cloud.google.com/traffic-director/docs)mz/nva-ha)
- [NVA Performance Optimization](https://docs.microsoft.com/en-us/azure/architecture/reference-architectures/hybrid-networking/network-virtual-appliances)
- [IP Forwarding Configuration](https://docs.microsoft.com/en-us/azure/virtual-network/virtual-networks-udr-overview#ip-forwarding)

### Monitoring and Troubleshooting
- [Network Watcher](https://docs.microsoft.com/en-us/azure/network-watcher/)
- [Network Performance Monitor](https://docs.microsoft.com/en-us/azure/network-watcher/network-watcher-monitoring-overview)
- [Connection Monitor](https://docs.microsoft.com/en-us/azure/network-watcher/connection-monitor-overview)
- [Traffic Analytics](https://docs.microsoft.com/en-us/azure/network-watcher/traffic-analytics)
