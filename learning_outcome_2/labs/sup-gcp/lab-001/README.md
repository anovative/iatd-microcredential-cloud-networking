## IATD Microcredential Cloud Networking: Supplementary Lab - GCP VPC Peering and Cloud Router

**Objective:** This lab focuses on implementing advanced routing configurations in Google Cloud Platform using VPC Network Peering and Cloud Router. You'll learn to optimize traffic flow between VPC networks and implement dynamic routing using Cloud Router.

**Estimated Time:** 90-120 minutes

**Prerequisites:**
1. **GCP Account:** An active Google Cloud Platform account with billing enabled
2. **gcloud CLI:** Installed and configured with appropriate credentials
3. **Basic Understanding:** 
   - GCP VPC networking concepts
   - Cloud Router and BGP routing
   - Basic networking principles
   - VPC peering concepts

### Resource Naming Convention

* **VPC Networks:**
  - Primary VPC: `iatd-labs-gcp-primary-vpc`
  - Secondary VPC: `iatd-labs-gcp-secondary-vpc`
* **Subnets:**
  - Primary Public: `iatd-labs-gcp-primary-public`
  - Primary Private: `iatd-labs-gcp-primary-private`
  - Secondary Public: `iatd-labs-gcp-secondary-public`
  - Secondary Private: `iatd-labs-gcp-secondary-private`
* **Cloud Routers:**
  - Primary Router: `iatd-labs-gcp-primary-router`
  - Secondary Router: `iatd-labs-gcp-secondary-router`
* **VM Instances:**
  - Primary VM: `iatd-labs-gcp-primary-vm`
  - Secondary VM: `iatd-labs-gcp-secondary-vm`

### Part 1: Setting Up IAM and Service Accounts (CloudShell)

1. **Create Service Account:**
   ```bash
   # Create service account for VMs
   gcloud iam service-accounts create iatd-labs-gcp-sa \
     --description="Service account for IATD Labs VMs" \
     --display-name="IATD Labs Service Account"

   # Grant necessary permissions
   gcloud projects add-iam-policy-binding ${PROJECT_ID} \
     --member="serviceAccount:iatd-labs-gcp-sa@${PROJECT_ID}.iam.gserviceaccount.com" \
     --role="roles/compute.networkViewer"
   ```

   Expected Output:
   ```
   Created service account [iatd-labs-gcp-sa].
   Updated IAM policy for project [PROJECT_ID].
   ```

2. **Enable Required APIs:**
   ```bash
   # Enable necessary APIs
   gcloud services enable \
     compute.googleapis.com \
     servicenetworking.googleapis.com \
     cloudresourcemanager.googleapis.com
   ```

### Part 2: Creating VPC Networks and Subnets (Portal and CloudShell)

1. **Create Primary VPC Network:**
   ```bash
   gcloud compute networks create iatd-labs-gcp-primary-vpc \
     --subnet-mode=custom \
     --bgp-routing-mode=regional \
     --description="Primary VPC for IATD Labs"
   ```

   Expected Output:
   ```
   Created [https://www.googleapis.com/compute/v1/projects/YOUR_PROJECT/global/networks/iatd-labs-gcp-primary-vpc].
   NAME                        SUBNET_MODE  BGP_ROUTING_MODE  IPV4_RANGE  GATEWAY_IPV4
   iatd-labs-gcp-primary-vpc  CUSTOM       REGIONAL
   ```

2. **Create Secondary VPC Network:**
   ```bash
   gcloud compute networks create iatd-labs-gcp-secondary-vpc \
     --subnet-mode=custom \
     --bgp-routing-mode=regional \
     --description="Secondary VPC for IATD Labs"
   ```

3. **Create Subnets in Primary VPC:**
   ```bash
   # Create public subnet
   gcloud compute networks subnets create iatd-labs-gcp-primary-public \
     --network=iatd-labs-gcp-primary-vpc \
     --region=us-west1 \
     --range=172.16.1.0/24 \
     --description="Primary VPC Public Subnet"

   # Create private subnet
   gcloud compute networks subnets create iatd-labs-gcp-primary-private \
     --network=iatd-labs-gcp-primary-vpc \
     --region=us-west1 \
     --range=172.16.2.0/24 \
     --description="Primary VPC Private Subnet"
   ```

4. **Create Subnets in Secondary VPC:**
   ```bash
   # Create public subnet
   gcloud compute networks subnets create iatd-labs-gcp-secondary-public \
     --network=iatd-labs-gcp-secondary-vpc \
     --region=us-west1 \
     --range=172.17.1.0/24 \
     --description="Secondary VPC Public Subnet"

   # Create private subnet
   gcloud compute networks subnets create iatd-labs-gcp-secondary-private \
     --network=iatd-labs-gcp-secondary-vpc \
     --region=us-west1 \
     --range=172.17.2.0/24 \
     --description="Secondary VPC Private Subnet"
   ```

### Part 2: Setting Up VPC Network Peering

1. **Create VPC Peering from Primary to Secondary:**
   ```bash
   gcloud compute networks peerings create primary-to-secondary \
     --network=iatd-labs-gcp-primary-vpc \
     --peer-network=iatd-labs-gcp-secondary-vpc \
     --auto-create-routes
   ```

   Expected Output:
   ```
   Created [https://www.googleapis.com/compute/v1/projects/YOUR_PROJECT/global/networks/peerings/primary-to-secondary].
   NAME                  NETWORK                    PEER_NETWORK
   primary-to-secondary  iatd-labs-gcp-primary-vpc  iatd-labs-gcp-secondary-vpc
   ```

2. **Create VPC Peering from Secondary to Primary:**
   ```bash
   gcloud compute networks peerings create secondary-to-primary \
     --network=iatd-labs-gcp-secondary-vpc \
     --peer-network=iatd-labs-gcp-primary-vpc \
     --auto-create-routes
   ```

### Part 3: Configuring Cloud Routers (Mixed)

1. **Create Cloud Router for Primary VPC:**
   ```bash
   gcloud compute routers create iatd-labs-gcp-primary-router \
     --network=iatd-labs-gcp-primary-vpc \
     --region=us-west1 \
     --asn=65001
   ```

   Expected Output:
   ```
   Created [https://www.googleapis.com/compute/v1/projects/YOUR_PROJECT/regions/us-west1/routers/iatd-labs-gcp-primary-router].
   NAME                          REGION    NETWORK
   iatd-labs-gcp-primary-router  us-west1  iatd-labs-gcp-primary-vpc
   ```

2. **Create Cloud Router for Secondary VPC:**
   ```bash
   gcloud compute routers create iatd-labs-gcp-secondary-router \
     --network=iatd-labs-gcp-secondary-vpc \
     --region=us-west1 \
     --asn=65002
   ```

### Part 4: Security Configuration

1. **Configure Firewall Rules:**
   ```bash
   # Create firewall rule for internal traffic
   gcloud compute firewall-rules create iatd-labs-gcp-allow-internal \
     --network=iatd-labs-gcp-primary-vpc \
     --allow=tcp,udp,icmp \
     --source-ranges=172.16.0.0/16,172.17.0.0/16 \
     --description="Allow internal traffic between VPCs"

   # Create firewall rule for OS Login
   gcloud compute firewall-rules create iatd-labs-gcp-allow-oslogin \
     --network=iatd-labs-gcp-primary-vpc \
     --allow=tcp:22 \
     --source-ranges=35.235.240.0/20 \
     --target-tags=allow-oslogin \
     --description="Allow OS Login IAP traffic"
   ```

2. **Enable OS Login:**
   ```bash
   # Enable OS Login at project level
   gcloud compute project-info add-metadata \
     --metadata enable-oslogin=TRUE
   ```

### Part 5: Creating and Testing VM Instances

1. **Create VM in Primary VPC:**
   ```bash
   gcloud compute instances create iatd-labs-gcp-primary-vm \
     --zone=us-west1-a \
     --machine-type=e2-micro \
     --subnet=iatd-labs-gcp-primary-private \
     --service-account=iatd-labs-gcp-sa@${PROJECT_ID}.iam.gserviceaccount.com \
     --scopes=cloud-platform \
     --image-family=debian-11 \
     --image-project=debian-cloud \
     --tags=allow-oslogin \
     --shielded-secure-boot \
     --shielded-vtpm \
     --shielded-integrity-monitoring
   ```

2. **Create VM in Secondary VPC:**
   ```bash
   gcloud compute instances create iatd-labs-gcp-secondary-vm \
     --zone=us-west1-a \
     --machine-type=e2-micro \
     --subnet=iatd-labs-gcp-secondary-private \
     --service-account=iatd-labs-gcp-sa@${PROJECT_ID}.iam.gserviceaccount.com \
     --scopes=cloud-platform \
     --image-family=debian-11 \
     --image-project=debian-cloud \
     --tags=allow-oslogin \
     --shielded-secure-boot \
     --shielded-vtpm \
     --shielded-integrity-monitoring
   ```

3. **Test Connectivity:**
   ```bash
   # SSH into primary VM
   gcloud compute ssh iatd-labs-gcp-primary-vm --zone=us-west1-a

   # Ping secondary VM
   ping -c 4 <SECONDARY_VM_PRIVATE_IP>
   ```

   Expected Output:
   ```
   PING 172.17.2.2 (172.17.2.2) 56(84) bytes of data.
   64 bytes from 172.17.2.2: icmp_seq=1 ttl=64 time=1.23 ms
   64 bytes from 172.17.2.2: icmp_seq=2 ttl=64 time=1.15 ms
   64 bytes from 172.17.2.2: icmp_seq=3 ttl=64 time=1.18 ms
   64 bytes from 172.17.2.2: icmp_seq=4 ttl=64 time=1.20 ms
   ```

### Part 5: Verifying Routes and Connectivity

1. **List Effective Routes for Primary VM:**
   ```bash
   gcloud compute routes list \
     --filter="network=iatd-labs-gcp-primary-vpc" \
     --format="table(name,network,destRange,nextHopType,priority)"
   ```

   Expected Output:
   ```
   NAME                 NETWORK                    DEST_RANGE    NEXT_HOP_TYPE  PRIORITY
   peering-route-1     iatd-labs-gcp-primary-vpc  172.17.0.0/16 peering        1000
   default-route       iatd-labs-gcp-primary-vpc  0.0.0.0/0     default-internet-gateway  1000
   subnet-route-1      iatd-labs-gcp-primary-vpc  172.16.0.0/16 subnet         1000
   ```

### Troubleshooting Guide

1. **VPC Network Peering Issues:**
   * Verify peering state is 'ACTIVE' on both sides
   * Check for overlapping IP ranges
   * Ensure proper IAM permissions
   * Validate route exchange settings

2. **Cloud Router Problems:**
   * Confirm ASN numbers are unique
   * Check BGP session status
   * Verify route advertisement settings
   * Monitor BGP route learning

3. **VM Instance Connectivity:**
   * Check firewall rules are correctly configured
   * Verify OS Login is working
   * Ensure service account permissions
   * Validate network tags

4. **Common Mistakes:**
   * Missing firewall rules
   * Incorrect service account configuration
   * Wrong network tags
   * Misconfigured IAM roles
   * Overlapping IP ranges

5. **gcloud CLI Issues:**
   * Verify gcloud configuration
   * Check project and region settings
   * Validate authentication
   * Ensure API services are enabled

### Post-Lab: Cleanup

To avoid unnecessary costs, clean up the resources using any of these methods:

1. **Google Cloud Console:**
   * Navigate to VPC Networks
   * Delete resources in this order:
     1. VM instances
     2. Firewall rules
     3. VPC peering connections
     4. Cloud Routers
     5. Subnets
     6. VPC networks
   * Navigate to IAM & Admin
   * Delete resources in this order:
     1. Service account IAM bindings
     2. Service account

2. **gcloud CLI:**
   ```bash
   # Delete VM instances
   gcloud compute instances delete iatd-labs-gcp-primary-vm --zone=us-west1-a --quiet
   gcloud compute instances delete iatd-labs-gcp-secondary-vm --zone=us-west1-a --quiet

   # Delete firewall rules
   gcloud compute firewall-rules delete iatd-labs-gcp-allow-internal --quiet
   gcloud compute firewall-rules delete iatd-labs-gcp-allow-oslogin --quiet

   # Delete VPC peering
   gcloud compute networks peerings delete primary-to-secondary --network=iatd-labs-gcp-primary-vpc --quiet
   gcloud compute networks peerings delete secondary-to-primary --network=iatd-labs-gcp-secondary-vpc --quiet

   # Delete Cloud Routers
   gcloud compute routers delete iatd-labs-gcp-primary-router --region=us-west1 --quiet
   gcloud compute routers delete iatd-labs-gcp-secondary-router --region=us-west1 --quiet

   # Delete VPC networks (this will also delete associated subnets)
   gcloud compute networks delete iatd-labs-gcp-primary-vpc --quiet
   gcloud compute networks delete iatd-labs-gcp-secondary-vpc --quiet

   # Clean up IAM resources
   gcloud projects remove-iam-policy-binding ${PROJECT_ID} \
     --member="serviceAccount:iatd-labs-gcp-sa@${PROJECT_ID}.iam.gserviceaccount.com" \
     --role="roles/compute.networkViewer"

   gcloud iam service-accounts delete \
     iatd-labs-gcp-sa@${PROJECT_ID}.iam.gserviceaccount.com --quiet
   ```

### Post-Lab Summary

#### Key Concepts Learned
1. **VPC Network Architecture:**
   * VPC network design and segmentation
   * Subnet creation and IP addressing
   * Network topology planning
   * Private network connectivity

2. **VPC Network Peering:**
   * Peer-to-peer network connections
   * Route exchange configuration
   * Network isolation principles
   * Cross-VPC communication

3. **Cloud Router Implementation:**
   * Dynamic route advertisement
   * BGP protocol configuration
   * ASN assignment strategy
   * Route propagation verification

4. **Security Configuration:**
   * IAM roles and permissions
   * Service account management
   * Firewall rule implementation
   * OS Login configuration

#### Lab Achievements
* Created secure VPC networks with proper segmentation
* Implemented VPC Network Peering for cross-network connectivity
* Configured Cloud Routers for dynamic route exchange
* Set up secure VM instances with proper IAM and networking
* Established secure communication between VPCs
* Implemented security best practices

#### Next Steps
1. **Advanced Networking:**
   * Explore Cloud NAT configuration
   * Implement load balancing
   * Study hybrid connectivity options
   * Learn about Cloud CDN

2. **Security Enhancements:**
   * Implement Cloud Armor
   * Configure VPC Service Controls
   * Study Identity-Aware Proxy
   * Explore Cloud KMS

3. **Monitoring and Operations:**
   * Set up Cloud Monitoring
   * Configure VPC Flow Logs
   * Implement Cloud Trace
   * Study Cloud Audit Logs

### Additional Resources

#### Documentation
* [VPC Network Peering Overview](https://cloud.google.com/vpc/docs/vpc-peering)
* [Cloud Router Concepts](https://cloud.google.com/network-connectivity/docs/router/concepts/overview)
* [VPC Design Best Practices](https://cloud.google.com/vpc/docs/vpc-design-best-practices)
* [Security Best Practices](https://cloud.google.com/vpc/docs/vpc-security)

#### Tutorials and Guides
* [Advanced VPC Network Peering](https://cloud.google.com/vpc/docs/using-vpc-peering)
* [Cloud Router How-to Guides](https://cloud.google.com/network-connectivity/docs/router/how-to)
* [IAM Best Practices](https://cloud.google.com/iam/docs/best-practices)
* [Network Security Tutorials](https://cloud.google.com/vpc/docs/using-security-policies)

#### Reference Architecture
* [Enterprise Network Patterns](https://cloud.google.com/architecture/networking-patterns)
* [Security Reference Blueprints](https://cloud.google.com/architecture/security-foundations)
* [Hybrid Connectivity Patterns](https://cloud.google.com/architecture/hybrid-and-multi-cloud-patterns)

**Congratulations!** You've successfully implemented advanced routing in Google Cloud Platform using VPC Network Peering and Cloud Router. This lab has demonstrated key concepts in GCP networking that parallel the routing and traffic optimization scenarios covered in the Azure labs.
