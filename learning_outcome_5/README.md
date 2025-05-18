# Learning Outcome 5: Load Balancing and Hybrid Connectivity

## Overview

This learning outcome focuses on implementing load balancing solutions and hybrid connectivity in cloud environments. You will learn how to configure and manage various load balancing services in Azure, including Azure Load Balancer, Application Gateway, and Azure Front Door. Additionally, you will explore hybrid connectivity options such as Site-to-Site VPNs and ExpressRoute.

## Labs

### Azure Labs

#### Load Balancing

1. [Lab 001: Load Balancing Fundamentals and Azure Load Balancer Basics](labs/lab-001/README.md)
   - Introduction to load balancing concepts and Azure Load Balancer
   - Setting up a basic load balancer with backend pools and health probes

2. [Lab 002: Configuring and Managing Azure Load Balancer](labs/lab-002/README.md)
   - Practical aspects of configuring Azure Load Balancer
   - Creating and managing load balancer components

3. [Lab 003: Advanced Load Balancing with Application Gateway - Part 1](labs/lab-003/README.md)
   - Introduction to Azure Application Gateway
   - Configuring SSL offloading and certificate management

4. [Lab 004: Advanced Load Balancing with Application Gateway - Part 2](labs/lab-004/README.md)
   - Implementing path-based routing
   - Configuring traffic distribution rules

5. [Lab 005: Global Load Balancing with Azure Front Door](labs/lab-005/README.md)
   - Configuring global load balancing
   - Distributing traffic across multiple regions

#### Hybrid Connectivity

6. [Lab 006: Hybrid Connectivity with VPN Gateway and ExpressRoute](labs/lab-006/README.md)
   - Understanding hybrid connectivity models
   - Configuring Site-to-Site VPN connections
   - Learning about ExpressRoute benefits and considerations

7. [Capstone Lab: Multi-Cloud Load Balancing and Hybrid Connectivity](labs/lab-capstone/README.md)
   - Implementing multi-tier load balancing architectures
   - Configuring global load balancing across regions
   - Creating hybrid connectivity solutions across cloud providers
   - Designing resilient and high-performance multi-cloud solutions

### AWS Supplementary Labs

1. [AWS Lab 001: Elastic Load Balancing with AWS Application Load Balancer](labs/sup-aws/lab-001/README.md)
   - Implementing load balancing in AWS using Application Load Balancer (ALB)
   - Distributing traffic across multiple EC2 instances in different availability zones
   - Configuring path-based routing

2. [AWS Lab 002: Hybrid Connectivity with AWS Site-to-Site VPN and Transit Gateway](labs/sup-aws/lab-002/README.md)
   - Setting up Site-to-Site VPN connections between on-premises and AWS VPCs
   - Using AWS Transit Gateway to simplify connectivity between multiple VPCs
   - Understanding hybrid connectivity options in AWS

### GCP Supplementary Labs

1. [GCP Lab 001: Load Balancing with Google Cloud Load Balancer](labs/sup-gcp/lab-001/README.md)
   - Implementing global load balancing in Google Cloud Platform
   - Distributing traffic across multiple regions
   - Configuring URL-based routing

2. [GCP Lab 002: Hybrid Connectivity with Cloud VPN and Cloud Interconnect](labs/sup-gcp/lab-002/README.md)
   - Setting up Cloud VPN for secure connections between on-premises and Google Cloud
   - Understanding Cloud Interconnect options for dedicated connectivity
   - Comparing different hybrid connectivity solutions in GCP

## Learning Objectives

By completing these labs, you will be able to:

- Implement and manage Azure Load Balancer for Layer 4 load balancing
- Configure Azure Application Gateway for Layer 7 load balancing
- Implement global load balancing with Azure Front Door
- Understand and configure hybrid connectivity solutions
- Apply best practices for load balancing and hybrid networking
