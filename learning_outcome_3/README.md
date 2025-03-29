# IATD Microcredential: Cloud Networking
## Learning Outcome 3: Network Security and Access Control

This directory contains hands-on lab exercises for Learning Outcome 3 of the IATD Microcredential Cloud Networking course. These labs focus on Azure network security configurations, providing practical experience with Network Security Groups (NSGs), Application Security Groups (ASGs), and advanced security policies.

## Learning Outcome Overview

Learning Outcome 3 focuses on mastering Azure network security concepts and implementations. Through these labs, you will gain hands-on experience with:
- Configuring Network Security Groups (NSGs) and understanding their behavior
- Implementing Application Security Groups (ASGs) for logical grouping of resources
- Managing security rule prioritization and conflict resolution
- Implementing service tags for simplified security management
- Creating and enforcing organizational security policies
- Troubleshooting network security configurations
- Using diagnostic tools for security verification

## Prerequisites

Before starting these labs, you should have: 

1. An active Azure subscription ([https://azure.microsoft.com/free/](https://azure.microsoft.com/free/))
2. Completed Learning Outcome 1 and 2 labs
3. Understanding of:
   - Virtual networks and subnets
   - IP addressing (using 172.16.0.0/16 range)
   - Basic networking concepts
   - Azure portal, CLI, and PowerShell basics

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

These labs provide comprehensive coverage of Azure network security concepts and implementations. Through hands-on exercises, you'll learn to configure and manage various aspects of Azure network security:

### [Lab 1: Network Security Groups (NSGs) and Application Security Groups (ASGs) Fundamentals](/learning_outcome_3/labs/lab-001)
**Objective:** This lab guides you through the process of creating and configuring Network Security Groups (NSGs) and Application Security Groups (ASGs) in Azure. You'll learn how to implement layered security, logically group virtual machines, and troubleshoot NSG configurations.  
**Estimated Time:** 60-75 minutes

### [Lab 2: Advanced NSG Rule Management, Service Tags, and Security Policies](/learning_outcome_3/labs/lab-002)
**Objective:** This lab builds upon the fundamentals of NSGs by diving into more advanced rule management techniques, focusing on optimizing rule configuration with service tags and implementing security policies that align with organizational compliance requirements.  
**Estimated Time:** 75-90 minutes

### Supplementary Labs (Multi-Cloud)

These supplementary labs demonstrate how network security concepts covered in the Azure labs can be applied in other major cloud platforms:

### [AWS Supplementary Lab: Security Groups and Network ACLs](/learning_outcome_3/labs/sup-aws/lab-001)
**Objective:** This lab focuses on implementing network security in AWS using Security Groups and Network ACLs. You'll learn how these security mechanisms compare to Azure's NSGs and ASGs, and how to implement layered security in AWS environments.  
**Estimated Time:** 90-120 minutes

### [GCP Supplementary Lab: Firewall Rules and VPC Service Controls](/learning_outcome_3/labs/sup-gcp/lab-001)
**Objective:** This lab focuses on implementing network security in Google Cloud Platform using Firewall Rules and VPC Service Controls. You'll learn how these security mechanisms compare to Azure's NSGs and ASGs, and how to implement layered security in GCP environments.  
**Estimated Time:** 90-120 minutes

## Troubleshooting Guide

If you encounter issues during the labs, here are some common problems and their solutions:

1. **NSG Rule Issues:**
   - Verify rule priorities (lower numbers have higher priority)
   - Check source and destination settings
   - Ensure protocol and port settings match your requirements
   - Verify NSG is correctly associated with subnet or NIC

2. **ASG Issues:**
   - Confirm VMs are correctly associated with ASGs
   - Verify NSG rules correctly reference the ASGs
   - Check that ASGs are in the same region as the resources they're applied to

3. **Service Tag Issues:**
   - Ensure you're using the correct service tag for your scenario
   - Verify the service tag is correctly specified in the rule

4. **Connectivity Issues:**
   - Use Network Watcher's IP flow verify tool to check if traffic is allowed/denied
   - Check effective security rules on the network interface
   - Verify VM is running and network interfaces are properly configured

## Additional Resources

### Azure Network Security
- [Azure Network Security Groups Documentation](https://docs.microsoft.com/en-us/azure/virtual-network/network-security-groups-overview)
- [Application Security Groups Overview](https://docs.microsoft.com/en-us/azure/virtual-network/application-security-groups)
- [Service Tags Overview](https://docs.microsoft.com/en-us/azure/virtual-network/service-tags-overview)

### Monitoring and Troubleshooting
- [Network Watcher](https://docs.microsoft.com/en-us/azure/network-watcher/)
- [Troubleshoot NSGs using the Azure portal](https://docs.microsoft.com/en-us/azure/virtual-network/diagnose-network-traffic-filter-problem)
- [Verify NSG flow logs](https://docs.microsoft.com/en-us/azure/network-watcher/network-watcher-nsg-flow-logging-portal)

### Security Best Practices
- [Azure Network Security Best Practices](https://docs.microsoft.com/en-us/azure/security/fundamentals/network-best-practices)
- [Security Control: Network Security](https://docs.microsoft.com/en-us/azure/security/benchmarks/security-controls-v2-network-security)
- [Zero Trust Network Security](https://docs.microsoft.com/en-us/azure/security/fundamentals/zero-trust-network-security)