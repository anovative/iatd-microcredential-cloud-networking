# IATD Microcredential: Cloud Networking
## Learning Outcome 1: Design and Implement Core Network Infrastructure

This directory contains hands-on lab exercises for Learning Outcome 1 of the IATD Microcredential Cloud Networking course. These labs are designed to provide practical experience with designing and implementing core network infrastructure in Azure, as well as supplementary labs for AWS and GCP.

## Learning Outcome Overview

Learning Outcome 1 focuses on teaching essential skills for designing and implementing core network infrastructure across multiple cloud platforms. Through these labs, you will gain hands-on experience with virtual networks, subnets, IP addressing, VPN connections, disaster recovery, and more.

## Prerequisites

Before starting these labs, you should have: 

1. An active Azure subscription
2. Basic understanding of networking concepts
3. Familiarity with cloud computing fundamentals
4. AWS account with appropriate permissions (for supplementary lab)
5. GCP account with appropriate permissions (for supplementary lab)

## Lab Structure

Each lab follows a consistent structure:
- Clear objectives and learning outcomes
- Estimated time for completion
- Prerequisites specific to the lab
- Step-by-step instructions with screenshots and code samples
- Verification steps to confirm successful completion
- Cleanup instructions to avoid unnecessary charges

## Labs

### Core Labs (Azure)

These labs provide a comprehensive foundation in Azure networking:

### [Lab 1: Virtual Network Creation](/learning_outcome_1/labs/lab-001)
**Objective:** This lab guides you through the process of creating an Azure Virtual Network (VNet), the core building block for your private network. You'll learn to create VNets using the Azure Portal, Azure PowerShell, Azure CLI, and ARM templates.  
**Estimated Time:** 45 - 60 minutes

### [Lab 2: Working with Subnets](/learning_outcome_1/labs/lab-002)
**Objective:** This lab will guide you through creating subnets in Azure using the Azure Portal, Azure PowerShell and Azure CLI. You'll also learn to associate Network Security Groups (NSGs) and Route Tables to control network traffic, and configure service endpoints to securely access Azure services.  
**Estimated Time:** 45 minutes - 1 hour

### [Lab 3: Creating a Dual-Stack Virtual Network (VNet) and Virtual Machine](/learning_outcome_1/labs/lab-003)
**Objective:** This lab guides you through creating an Azure Virtual Network (VNet) with both IPv4 and IPv6 address spaces. You'll then deploy a Windows virtual machine (VM) into this dual-stack VNet, assigning both IPv4 and IPv6 addresses to the VM. This lab strategically uses the Azure Portal, Azure CLI, and PowerShell to expose you to each method.  
**Estimated Time:** 45 minutes - 1 hour

### [Lab 4: Setting up a Site-to-Site VPN Between Azure and AWS](/learning_outcome_1/labs/lab-004)
**Objective:** In this lab, you will establish a secure Site-to-Site VPN connection between an Azure Virtual Network (VNet) and an AWS Virtual Private Cloud (VPC). This will enable network traffic to flow securely between your cloud environments.  
**Estimated Time:** 60 - 90 minutes

### [Lab 5: Assigning IP Addresses to Virtual Machines in Azure](/learning_outcome_1/labs/lab-005)
**Objective:** In this lab, you will learn how to configure both static private and public IP addresses for Azure Virtual Machines. You'll also explore dynamic IP allocation.  
**Estimated Time:** 45 - 60 minutes

### [Lab 6: Creating Virtual Machines with Public/Private IPs and NAT Gateway Configuration](/learning_outcome_1/labs/lab-006)
**Objective:** In this lab, you will learn how to create Azure Virtual Machines (VMs) with either a public IP address or only a private IP address. You will then configure a NAT Gateway to provide outbound internet access for VMs with only private IPs.  
**Estimated Time:** 45 - 60 minutes

### [Lab 7: Using Azure Site Recovery to Replicate Virtual Machines to a Secondary Region](/learning_outcome_1/labs/lab-007)
**Objective:** In this lab, you will learn how to use Azure Site Recovery to replicate Azure Virtual Machines (VMs) to a secondary Azure region. You will then perform a test failover to simulate a disaster recovery scenario.  
**Estimated Time:** 60 - 75 minutes

### [Lab 8: Creating and Configuring an Azure DNS Zone](/learning_outcome_1/labs/lab-008)
**Objective:** In this lab, you will learn how to create an Azure DNS zone, add various types of DNS records, and delegate the zone to Azure DNS.  
**Estimated Time:** 60 - 75 minutes

### [Lab 9: Implementing Private DNS Zones in Azure](/learning_outcome_1/labs/lab-009)
**Objective:** In this lab, you will learn how to create a private DNS zone in Azure, link it to a virtual network, add DNS records, and verify name resolution from a virtual machine within the network.  
**Estimated Time:** 45-60 minutes

### [Capstone Lab: Multi-Cloud Networking Practice](/learning_outcome_1/labs/lab-capstone)
**Objective:** This capstone lab provides hands-on practice with multi-cloud networking concepts across Azure, AWS, and GCP. Through simple, guided exercises, you'll reinforce your understanding of basic networking concepts in each cloud platform.  
**Estimated Time:** 120-180 minutes

### Supplementary Labs (AWS & GCP)

These supplementary labs extend your networking knowledge to other major cloud platforms:

### AWS Supplementary Labs

#### [Lab 1: VPC Fundamentals](/learning_outcome_1/labs/sup-aws/lab-001)
**Objective:** This lab guides you through creating and configuring Amazon Virtual Private Cloud (VPC), implementing public and private subnets, configuring Internet and NAT Gateways, setting up security groups and routing, and launching EC2 instances in different subnet tiers.  
**Estimated Time:** 2-3 hours

### GCP Supplementary Labs

#### [Lab 1: VPC Fundamentals](/learning_outcome_1/labs/sup-gcp/lab-001)
**Objective:** This lab guides you through creating and configuring Google Cloud VPC Network, implementing subnet architecture, configuring firewall rules and Cloud NAT, deploying VM instances with different network access levels, and testing connectivity and routing.  
**Estimated Time:** 2-3 hours

## Resource Naming Conventions

Throughout these labs, resources follow a consistent naming convention:
- Resource groups: `iatd_labs_XX_rg` (where XX is the lab number)
- Virtual networks: `iatd_labs_XX_vnet`
- Subnets: `iatd_labs_XX_subnet`
- Virtual machines: `iatd_labs_XX_vm`

## Best Practices

1. **Clean up resources** after completing each lab to avoid unnecessary charges
2. **Follow the naming conventions** to easily identify and manage resources
3. **Read through the entire lab** before starting to understand the full workflow
4. **Take notes** during the labs to reinforce learning
5. **Experiment** beyond the lab instructions to deepen your understanding

## Troubleshooting

If you encounter issues during the labs:
1. Double-check that you've followed all steps in the correct order
2. Verify that resource names match exactly as specified
3. Ensure your Azure subscription has sufficient permissions and quota
4. Check the Azure status page for any service outages
5. Consult the Azure documentation for additional guidance

## Additional Resources

- [Microsoft Azure Documentation](https://docs.microsoft.com/en-us/azure/)
- [Azure Networking Documentation](https://docs.microsoft.com/en-us/azure/networking/)
- [Azure Architecture Center](https://docs.microsoft.com/en-us/azure/architecture/)
- [Microsoft Learn - Azure Networking](https://docs.microsoft.com/en-us/learn/paths/azure-networking/)
- [AWS Documentation](https://docs.aws.amazon.com/)
- [GCP Documentation](https://cloud.google.com/docs)
