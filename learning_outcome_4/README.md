# IATD Microcredential: Cloud Networking
## Learning Outcome 4: Service Integration and Network Design

This directory contains hands-on lab exercises for Learning Outcome 4 of the IATD Microcredential Cloud Networking course. These labs focus on designing and implementing secure service integration solutions in cloud environments, with emphasis on communication patterns, network integration components, and secure access configurations.

## Learning Outcome Overview

Learning Outcome 4 focuses on mastering service integration and network design concepts. Through these labs, you will gain hands-on experience with:
- Designing appropriate communication patterns (sync vs async)
- Selecting and configuring network integration components
- Implementing service discovery mechanisms
- Securing service integrations
- Configuring service access using private endpoints
- Managing authentication and authorization
- Implementing monitoring and logging for integrated services
- Troubleshooting integration issues
- Optimizing network design for service communication

## Prerequisites

Before starting these labs, you should have: 

1. An active Azure subscription ([https://azure.microsoft.com/free/](https://azure.microsoft.com/free/))
2. Completed Learning Outcomes 1, 2, and 3 labs
3. Understanding of:
   - Virtual networks and subnets
   - IP addressing (using 172.16.0.0/16 range)
   - Network security concepts
   - Azure portal, CLI, and PowerShell basics
   - Basic service integration patterns

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

These labs provide comprehensive coverage of service integration and network design concepts in Azure. Through hands-on exercises, you'll learn to design, implement, and secure service integrations:

### Service Access and Communication Fundamentals

### [Lab 1: Securing Azure Service Access with Service Endpoints](/learning_outcome_4/labs/lab-001)
**Objective:** This lab guides you through configuring Azure Service Endpoints. You'll learn how to limit network access to Azure services to only resources within your Virtual Network, enhancing security and control.  
**Estimated Time:** 60-75 minutes

### [Lab 2: Private Communication within an Azure Virtual Network](/learning_outcome_4/labs/lab-002)
**Objective:** This lab demonstrates how resources within an Azure Virtual Network can communicate privately with each other, isolated from the public internet. You'll create a VNet with multiple subnets and verify private communication.  
**Estimated Time:** 60-75 minutes

### [Lab 3: Virtual Network Service Tags and Integration](/learning_outcome_4/labs/lab-003)
**Objective:** This lab explores Azure Virtual Network service tags and integration patterns. You'll learn how to use service tags for simplified network security rules and integrate Azure services with your VNet.  
**Estimated Time:** 75-90 minutes

### Service Integration Patterns

### [Lab 4: Implementing Azure Service Bus Integration](/learning_outcome_4/labs/lab-004)
**Objective:** This lab focuses on implementing asynchronous messaging patterns using Azure Service Bus. You'll learn about topics, subscriptions, and message handling patterns.  
**Estimated Time:** 75-90 minutes

### [Lab 5: API Management Integration Fundamentals](/learning_outcome_4/labs/lab-005)
**Objective:** This lab introduces Azure API Management as a service integration component. You'll learn how to expose, secure, and manage APIs using APIM.  
**Estimated Time:** 75-90 minutes

### [Lab 6: Advanced API Management and Security](/learning_outcome_4/labs/lab-006)
**Objective:** This lab builds upon APIM fundamentals with advanced topics like OAuth2, JWT validation, and rate limiting. You'll implement comprehensive API security patterns.  
**Estimated Time:** 90-120 minutes

### Service Discovery and Networking

### [Lab 7: Azure DNS and Service Discovery](/learning_outcome_4/labs/lab-007)
**Objective:** This lab explores service discovery patterns using Azure DNS. You'll learn about private DNS zones, custom domains, and DNS-based service discovery.  
**Estimated Time:** 75-90 minutes

### [Lab 8: Load Balancing and Traffic Management](/learning_outcome_4/labs/lab-008)
**Objective:** This lab focuses on implementing load balancing and traffic management patterns. You'll work with Azure Load Balancer and Traffic Manager for optimal service routing.  
**Estimated Time:** 90-120 minutes

### [Lab 9: Network Integration with Azure Front Door](/learning_outcome_4/labs/lab-009)
**Objective:** This lab demonstrates global service integration using Azure Front Door. You'll implement global load balancing, SSL offloading, and WAF policies.  
**Estimated Time:** 90-120 minutes

### Advanced Service Integration

### [Lab 10: Azure Private Endpoints - Configuration and Secure Access Patterns](/learning_outcome_4/labs/lab-010)
**Objective:** This lab guides you through configuring Private Endpoints for various Azure services. You'll learn how to implement network isolation, explore different connectivity options, and examine typical access patterns.  
**Estimated Time:** 90-120 minutes

### [Lab 11: Designing and Configuring Secure Service Integration Solutions](/learning_outcome_4/labs/lab-011)
**Objective:** This lab focuses on designing and implementing secure service integration solutions in Azure. You'll learn how to choose appropriate communication patterns, select network integration components, implement service discovery, and secure integrations.  
**Estimated Time:** 90-120 minutes

### [Capstone Lab: Multi-Cloud Service Integration and Network Design](/learning_outcome_4/labs/lab-capstone)
**Objective:** This comprehensive capstone lab integrates concepts from all previous labs, challenging you to implement enterprise-grade service integration solutions across Azure, AWS, and GCP. You'll design secure connectivity patterns, implement private endpoints, and create hub-and-spoke architectures with hybrid connectivity.  
**Estimated Time:** 180-240 minutes

### Supplementary Labs (Multi-Cloud)

These supplementary labs demonstrate how service integration and network design concepts covered in the Azure labs can be applied in other major cloud platforms:

### AWS Supplementary Labs

### [AWS Lab 1: Transit Gateway and PrivateLink](/learning_outcome_4/labs/sup-aws/lab-001)
**Objective:** This lab focuses on implementing secure network connectivity in AWS using Transit Gateway and AWS PrivateLink. You'll learn how these services compare to Azure's Private Link and Virtual Network concepts, while implementing secure network isolation patterns.  
**Estimated Time:** 90-120 minutes

### GCP Supplementary Labs

### [GCP Lab 1: Private Service Connect and VPC Service Controls](/learning_outcome_4/labs/sup-gcp/lab-001)
**Objective:** This lab focuses on implementing secure network connectivity in Google Cloud Platform using Private Service Connect and VPC Service Controls. You'll learn how these services compare to Azure's Private Link and Network Security concepts, while implementing secure network isolation patterns.  
**Estimated Time:** 90-120 minutes

## Additional Resources

### Azure Documentation
- [Azure Private Link](https://docs.microsoft.com/azure/private-link/)
- [Azure API Management](https://docs.microsoft.com/azure/api-management/)
- [Azure Service Bus](https://docs.microsoft.com/azure/service-bus-messaging/)
- [Azure App Service](https://docs.microsoft.com/azure/app-service/)
- [Azure Functions](https://docs.microsoft.com/azure/azure-functions/)
- [Azure Key Vault](https://docs.microsoft.com/azure/key-vault/)

### Best Practices
- [Azure Architecture Center - Integration patterns](https://docs.microsoft.com/azure/architecture/patterns/category/integration)
- [Azure Well-Architected Framework - Integration](https://docs.microsoft.com/azure/architecture/framework/integration/)
- [Security best practices for Azure integration services](https://docs.microsoft.com/azure/architecture/framework/security/design-integration)
- [Network security considerations for Azure integration services](https://docs.microsoft.com/azure/architecture/guide/technology-choices/load-balancing-overview)
