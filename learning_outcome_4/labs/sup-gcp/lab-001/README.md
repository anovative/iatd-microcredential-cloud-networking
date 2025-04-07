## IATD Microcredential Cloud Networking: Supplementary Lab - GCP Private Service Connect and VPC Service Controls

**Objective:** This lab focuses on implementing secure network connectivity in Google Cloud Platform using Private Service Connect and VPC Service Controls. You'll learn how these services compare to Azure's Private Link and Network Security concepts, while implementing secure network isolation patterns.

**Estimated Time:** 90-120 minutes

**Prerequisites:**
1. **GCP Account:** An active Google Cloud Platform account with billing enabled
2. **gcloud CLI:** Installed and configured with appropriate credentials
3. **Basic Understanding:** 
   - GCP VPC networking concepts
   - VPC Service Controls
   - Basic networking principles
   - GCP VM instance management

### Resource Naming Convention

* **VPC Networks:**
  - Producer VPC: `iatd-labs-producer-vpc`
  - Consumer VPC: `iatd-labs-consumer-vpc`
* **Subnets:**
  - Producer Subnet: `iatd-labs-producer-subnet`
  - Consumer Subnet: `iatd-labs-consumer-subnet`
* **Firewall Rules:**
  - Allow Internal: `iatd-labs-allow-internal`
  - Allow Health Checks: `iatd-labs-allow-health-checks`
* **Service Perimeter:**
  - Security Perimeter: `iatd-labs-service-perimeter`
* **VM Instances:**
  - Producer Server: `iatd-labs-producer-vm`
  - Consumer Server: `iatd-labs-consumer-vm`

### Let's Get Started!

### Part 1: Setting Up the Producer VPC (gcloud CLI)

1. **Create the Producer VPC:**
   ```bash
   # Set environment variables
   export PROJECT_ID=$(gcloud config get-value project)
   export REGION="us-central1"
   export PRODUCER_VPC_CIDR="172.16.0.0/16"
   export PRODUCER_SUBNET_CIDR="172.16.1.0/24"

   # Create VPC
   gcloud compute networks create iatd-labs-producer-vpc \
     --subnet-mode=custom \
     --bgp-routing-mode=regional

   # Create subnet
   gcloud compute networks subnets create iatd-labs-producer-subnet \
     --network=iatd-labs-producer-vpc \
     --region=$REGION \
     --range=$PRODUCER_SUBNET_CIDR
   ```

   Expected Output:
   ```
   NAME                     SUBNET_MODE  BGP_ROUTING_MODE  IPV4_RANGE  GATEWAY_IPV4
   iatd-labs-producer-vpc  CUSTOM       REGIONAL
   
   NAME                       REGION       RANGE            STACK_TYPE  IPV6_ACCESS_TYPE
   iatd-labs-producer-subnet  us-central1  172.16.1.0/24   IPV4_ONLY
   ```

2. **Create Firewall Rules:**
   ```bash
   # Allow internal traffic
   gcloud compute firewall-rules create iatd-labs-allow-internal \
     --network=iatd-labs-producer-vpc \
     --allow=tcp,udp,icmp \
     --source-ranges=$PRODUCER_VPC_CIDR

   # Allow health checks
   gcloud compute firewall-rules create iatd-labs-allow-health-checks \
     --network=iatd-labs-producer-vpc \
     --allow=tcp:80,tcp:443 \
     --source-ranges=130.211.0.0/22,35.191.0.0/16
   ```

### Part 2: Setting Up the Consumer VPC

1. **Create Consumer VPC:**
   ```bash
   # Set environment variables
   export CONSUMER_VPC_CIDR="172.17.0.0/16"
   export CONSUMER_SUBNET_CIDR="172.17.1.0/24"

   # Create VPC
   gcloud compute networks create iatd-labs-consumer-vpc \
     --subnet-mode=custom \
     --bgp-routing-mode=regional

   # Create subnet
   gcloud compute networks subnets create iatd-labs-consumer-subnet \
     --network=iatd-labs-consumer-vpc \
     --region=$REGION \
     --range=$CONSUMER_SUBNET_CIDR
   ```

### Part 3: Implementing Private Service Connect

1. **Create Service Attachment in Producer VPC:**
   ```bash
   # Create target pool
   gcloud compute target-pools create iatd-labs-target-pool \
     --region=$REGION

   # Create forwarding rule
   gcloud compute forwarding-rules create iatd-labs-fwd-rule \
     --region=$REGION \
     --target-pool=iatd-labs-target-pool \
     --load-balancing-scheme=INTERNAL

   # Create service attachment
   gcloud compute service-attachments create iatd-labs-service-attachment \
     --region=$REGION \
     --producer-forwarding-rule=iatd-labs-fwd-rule \
     --connection-preference=ACCEPT_AUTOMATIC
   ```

2. **Create Private Service Connect Endpoint in Consumer VPC:**
   ```bash
   # Get service attachment URI
   export SERVICE_ATTACHMENT_URI=$(gcloud compute service-attachments describe iatd-labs-service-attachment \
     --region=$REGION \
     --format='get(selfLink)')

   # Create PSC endpoint
   gcloud compute forwarding-rules create iatd-labs-psc-endpoint \
     --region=$REGION \
     --network=iatd-labs-consumer-vpc \
     --subnet=iatd-labs-consumer-subnet \
     --target-service-attachment=$SERVICE_ATTACHMENT_URI
   ```

### Part 4: Setting Up VPC Service Controls

1. **Enable Required APIs:**
   ```bash
   # Enable Security Command Center API
   gcloud services enable securitycenter.googleapis.com

   # Enable Access Context Manager API
   gcloud services enable accesscontextmanager.googleapis.com
   ```

2. **Create Service Perimeter:**
   ```bash
   # Create access policy
   ACCESS_POLICY=$(gcloud access-context-manager policies create \
     --title="IATD Labs Policy" \
     --format="get(name)")

   # Create service perimeter
   gcloud access-context-manager service-perimeters create iatd-labs-service-perimeter \
     --policy=$ACCESS_POLICY \
     --title="IATD Labs Service Perimeter" \
     --resources="projects/$PROJECT_ID" \
     --restricted-services="compute.googleapis.com"
   ```

### Verification Steps

1. **Verify Private Service Connect:**
   ```bash
   # List service attachments
   gcloud compute service-attachments list \
     --region=$REGION

   # List PSC endpoints
   gcloud compute forwarding-rules list \
     --filter="name:iatd-labs-psc-endpoint" \
     --region=$REGION
   ```

2. **Verify VPC Service Controls:**
   ```bash
   # List service perimeters
   gcloud access-context-manager service-perimeters list \
     --policy=$ACCESS_POLICY
   ```

### Cleanup Instructions

1. **Delete Private Service Connect Resources:**
   ```bash
   # Delete PSC endpoint
   gcloud compute forwarding-rules delete iatd-labs-psc-endpoint \
     --region=$REGION --quiet

   # Delete service attachment
   gcloud compute service-attachments delete iatd-labs-service-attachment \
     --region=$REGION --quiet

   # Delete forwarding rule
   gcloud compute forwarding-rules delete iatd-labs-fwd-rule \
     --region=$REGION --quiet

   # Delete target pool
   gcloud compute target-pools delete iatd-labs-target-pool \
     --region=$REGION --quiet
   ```

2. **Delete VPC Service Controls:**
   ```bash
   # Delete service perimeter
   gcloud access-context-manager service-perimeters delete iatd-labs-service-perimeter \
     --policy=$ACCESS_POLICY --quiet

   # Delete access policy
   gcloud access-context-manager policies delete $ACCESS_POLICY --quiet
   ```

3. **Delete Network Resources:**
   ```bash
   # Delete firewall rules
   gcloud compute firewall-rules delete iatd-labs-allow-internal --quiet
   gcloud compute firewall-rules delete iatd-labs-allow-health-checks --quiet

   # Delete producer VPC and subnet
   gcloud compute networks subnets delete iatd-labs-producer-subnet \
     --region=$REGION --quiet
   gcloud compute networks delete iatd-labs-producer-vpc --quiet

   # Delete consumer VPC and subnet
   gcloud compute networks subnets delete iatd-labs-consumer-subnet \
     --region=$REGION --quiet
   gcloud compute networks delete iatd-labs-consumer-vpc --quiet
   ```

4. **Verify Environment Variables are Unset:**
   ```bash
   unset PROJECT_ID REGION
   unset PRODUCER_VPC_CIDR PRODUCER_SUBNET_CIDR
   unset CONSUMER_VPC_CIDR CONSUMER_SUBNET_CIDR
   unset SERVICE_ATTACHMENT_URI ACCESS_POLICY
   ```

### Additional Resources

1. [Private Service Connect Documentation](https://cloud.google.com/vpc/docs/private-service-connect)
2. [VPC Service Controls Documentation](https://cloud.google.com/vpc-service-controls/docs)
3. [GCP VPC Documentation](https://cloud.google.com/vpc/docs/vpc)
