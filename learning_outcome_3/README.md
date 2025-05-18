# IATD Microcredential: Cloud Networking
## Learning Outcome 3: Network Security and Access Control

This directory contains hands-on lab exercises for Learning Outcome 3 of the IATD Microcredential Cloud Networking course. These labs focus on Azure network security configurations, providing practical experience with Network Security Groups (NSGs), Application Security Groups (ASGs), DDoS protection, and monitoring and logging in cloud environments.

## Learning Outcome Overview

Learning Outcome 3 focuses on mastering cloud network security concepts and implementations. Through these labs, you will gain hands-on experience with:
- Configuring Network Security Groups (NSGs) and understanding their behavior
- Implementing Application Security Groups (ASGs) for logical grouping of resources
- Managing security rule prioritization and conflict resolution
- Implementing service tags for simplified security management
- Creating and enforcing organizational security policies
- Implementing DDoS protection strategies
- Setting up monitoring and logging for network resources
- Establishing performance baselines and alert thresholds
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

### [Lab 3: DDoS Mitigation in Azure - Fundamentals](/learning_outcome_3/labs/lab-003)
**Objective:** This lab introduces you to the fundamental concepts of DDoS attacks and how to mitigate them using Azure services. You will learn about different types of DDoS attacks, common mitigation techniques, and how to configure basic DDoS protection in Azure.  
**Estimated Time:** 75-90 minutes

### [Lab 4: DDoS Mitigation in Azure - Advanced](/learning_outcome_3/labs/lab-004)
**Objective:** This lab builds upon the fundamentals by exploring more advanced DDoS mitigation techniques using Azure's built-in DDoS protection services and Web Application Firewall (WAF).  
**Estimated Time:** 75-90 minutes

### [Lab 5: Monitoring and Logging in Azure - Fundamentals](/learning_outcome_3/labs/lab-005)
**Objective:** This lab introduces you to the fundamental concepts of monitoring and logging in Azure. You will learn about different monitoring types, log analysis techniques, performance baselines, alert thresholds, and core Azure monitoring services.  
**Estimated Time:** 75-90 minutes

### [Lab 6: Advanced Monitoring and Logging in Azure](/learning_outcome_3/labs/lab-006)
**Objective:** This lab builds upon the fundamentals by exploring more advanced monitoring and logging techniques in Azure. You will learn how to use Azure Network Watcher, NSG Flow Logs, Azure Resource Graph, and Azure Service Health.  
**Estimated Time:** 75-90 minutes

### [Capstone Lab: Multi-Cloud Network Security and Access Control](/learning_outcome_3/labs/lab-capstone)
**Objective:** This comprehensive capstone lab integrates concepts from all previous labs, challenging you to implement enterprise-grade security solutions across Azure, AWS, and GCP. You'll design multi-layered security architectures, implement DDoS protection, and create comprehensive monitoring solutions.  
**Estimated Time:** 180-240 minutes

### Supplementary Labs (Multi-Cloud)

These supplementary labs demonstrate how network security concepts covered in the Azure labs can be applied in other major cloud platforms:

### AWS Supplementary Labs

### [AWS Lab 1: Security Groups and Network ACLs](/learning_outcome_3/labs/sup-aws/lab-001)
**Objective:** This lab focuses on implementing network security in AWS using Security Groups and Network ACLs. You'll learn how these security mechanisms compare to Azure's NSGs and ASGs, and how to implement layered security in AWS environments.  
**Estimated Time:** 90-120 minutes

### [AWS Lab 2: Transit Gateway and Advanced Routing](/learning_outcome_3/labs/sup-aws/lab-002)
**Objective:** This lab explores advanced networking concepts in AWS using Transit Gateway. You'll learn how to connect multiple VPCs, manage route tables, and optimize traffic flow across your AWS network infrastructure.  
**Estimated Time:** 90-120 minutes

### [AWS Lab 3: Monitoring and Logging in AWS](/learning_outcome_3/labs/sup-aws/lab-003)
**Objective:** This lab introduces you to the fundamental concepts of monitoring and logging in AWS. You will learn about CloudWatch, CloudTrail, and AWS Config to monitor resources, analyze logs, establish performance baselines, and set alert thresholds.  
**Estimated Time:** 75-90 minutes

### GCP Supplementary Labs

### [GCP Lab 1: Firewall Rules and VPC Service Controls](/learning_outcome_3/labs/sup-gcp/lab-001)
**Objective:** This lab focuses on implementing network security in Google Cloud Platform using Firewall Rules and VPC Service Controls. You'll learn how these security mechanisms compare to Azure's NSGs and ASGs, and how to implement layered security in GCP environments.  
**Estimated Time:** 90-120 minutes

### [GCP Lab 2: VPC Network Peering and Cloud Router](/learning_outcome_3/labs/sup-gcp/lab-002)
**Objective:** This lab explores VPC Network Peering and Cloud Router in Google Cloud Platform. You'll learn how to connect VPC networks, implement dynamic routing with Cloud Router, and manage traffic flow between networks.  
**Estimated Time:** 90-120 minutes

### [GCP Lab 3: Monitoring and Logging in Google Cloud](/learning_outcome_3/labs/sup-gcp/lab-003)
**Objective:** This lab introduces you to the fundamental concepts of monitoring and logging in Google Cloud. You will learn about Cloud Monitoring, Cloud Logging, and other GCP monitoring services to monitor resources, analyze logs, establish performance baselines, and set alert thresholds.  
**Estimated Time:** 75-90 minutes

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
- [Azure DDoS Protection](https://docs.microsoft.com/en-us/azure/ddos-protection/ddos-protection-overview)

### Monitoring and Logging
- [Azure Monitor Overview](https://docs.microsoft.com/en-us/azure/azure-monitor/overview)
- [Network Watcher](https://docs.microsoft.com/en-us/azure/network-watcher/)
- [Log Analytics Workspace](https://docs.microsoft.com/en-us/azure/azure-monitor/logs/log-analytics-workspace-overview)
- [Troubleshoot NSGs using the Azure portal](https://docs.microsoft.com/en-us/azure/virtual-network/diagnose-network-traffic-filter-problem)
- [Verify NSG flow logs](https://docs.microsoft.com/en-us/azure/network-watcher/network-watcher-nsg-flow-logging-portal)

### AWS Resources
- [AWS Security Groups](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_SecurityGroups.html)
- [AWS Network ACLs](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-network-acls.html)
- [AWS CloudWatch](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/WhatIsCloudWatch.html)
- [AWS CloudTrail](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-user-guide.html)

### GCP Resources
- [GCP Firewall Rules](https://cloud.google.com/vpc/docs/firewalls)
- [GCP VPC Service Controls](https://cloud.google.com/vpc-service-controls/docs/overview)
- [Cloud Monitoring](https://cloud.google.com/monitoring/docs)
- [Cloud Logging](https://cloud.google.com/logging/docs)

### Security Best Practices
- [Azure Network Security Best Practices](https://docs.microsoft.com/en-us/azure/security/fundamentals/network-best-practices)
- [Security Control: Network Security](https://docs.microsoft.com/en-us/azure/security/benchmarks/security-controls-v2-network-security)
- [Zero Trust Network Security](https://docs.microsoft.com/en-us/azure/security/fundamentals/zero-trust-network-security)