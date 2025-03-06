# IATD Microcredential: Cloud Networking
## Learning Outcome 2: Design and Implement Advanced Network Infrastructure

This directory contains hands-on lab exercises for Learning Outcome 2 of the IATD Microcredential Cloud Networking course. These labs focus on Azure routing configurations, providing practical experience with User-Defined Routes (UDRs), system routes, and network traffic control.

## Learning Outcome Overview

Learning Outcome 2 focuses on mastering Azure routing concepts and implementations. Through these labs, you will gain hands-on experience with:
- Configuring User-Defined Routes (UDRs)
- Understanding Azure system routes and their behavior
- Implementing Network Virtual Appliances (NVAs) for traffic inspection
- Managing traffic flow using different next hop types
- Associating and verifying route tables with subnets

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

2. **NVA Issues:**
   - Confirm IP forwarding is enabled on the NVA's network interface
   - Verify IP forwarding is enabled in the NVA's OS
   - Check NSG rules allow required traffic
   - Ensure NVA is properly configured to handle routed traffic

3. **Connectivity Testing:**
   - Use `ping` and `traceroute` to verify traffic flow
   - Leverage Azure Network Watcher for detailed diagnostics
   - Check if VMs can reach their intended destinations
   - Verify DNS resolution is working correctly

4. **Common Mistakes:**
   - Route table not associated with subnet
   - Incorrect address prefixes in routes
   - Wrong next hop IP address
   - Missing or incorrect NSG rules
   - IP forwarding not enabled

## Additional Resources

- [Microsoft Azure Documentation](https://docs.microsoft.com/en-us/azure/)
- [Azure Networking Documentation](https://docs.microsoft.com/en-us/azure/networking/)
- [Azure Architecture Center](https://docs.microsoft.com/en-us/azure/architecture/)
- [Microsoft Learn - Azure Networking](https://docs.microsoft.com/en-us/learn/paths/azure-networking/)
- [Azure Virtual Network Documentation](https://docs.microsoft.com/en-us/azure/virtual-network/)
- [User-Defined Routes Overview](https://docs.microsoft.com/en-us/azure/virtual-network/virtual-networks-udr-overview)
- [Network Virtual Appliances](https://docs.microsoft.com/en-us/azure/virtual-network/virtual-networks-udr-overview#user-defined)
- [Network Watcher](https://docs.microsoft.com/en-us/azure/network-watcher/)
