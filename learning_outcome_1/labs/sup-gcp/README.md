## IATD Microcredential Cloud Networking: Supplementary Lab - GCP VPC Fundamentals

**Objective:** In this lab, you will learn how to create and configure basic networking components in Google Cloud Platform using both the Google Cloud Console (portal) and Cloud Shell. This supplementary lab provides hands-on experience with GCP networking fundamentals to complement your Azure knowledge.

**Estimated Time:** 60 minutes

**Prerequisites:**

1.  **GCP Account:** You need a Google Cloud Platform account with appropriate permissions
2.  **Google Cloud Console Access:** Access to Google Cloud Console
3.  **Basic Understanding:** Familiarity with basic networking concepts

**Let's get started!**

### Resource Naming Convention

All resources created in this lab will follow this naming pattern:
- VPC: `iatd_labs_gcp_vpc`
- Subnets: `iatd_labs_gcp_subnet_[public/private]`
- Firewall Rules: `iatd_labs_gcp_fw_[purpose]`
- Cloud Router: `iatd_labs_gcp_router`
- Cloud NAT: `iatd_labs_gcp_nat`
- VM Instances: `iatd_labs_gcp_vm_[public/private]`

### IP Addressing

We'll use the following IP address ranges:
- VPC CIDR: 192.168.0.0/16
- Public Subnet: 192.168.1.0/24
- Private Subnet: 192.168.2.0/24

### Part 1: Creating a VPC Network (Portal)

1.  **Create a Custom VPC Network:**
    * Navigate to VPC networks in Google Cloud Console
    * Click "Create VPC Network"
    * Configure:
      - Name: `iatd_labs_gcp_vpc`
      - Subnet creation mode: Custom
      - Click "Add Subnet"
        * Name: `iatd_labs_gcp_subnet_public`
        * Region: us-central1
        * IP range: 192.168.1.0/24
      - Click "Add Subnet" again
        * Name: `iatd_labs_gcp_subnet_private`
        * Region: us-central1
        * IP range: 192.168.2.0/24
      - Click "Create"

### Part 2: Configuring Firewall Rules (CloudShell)

1.  **Create Basic Firewall Rules:**
    ```bash
    # Allow SSH access to public instances
    gcloud compute firewall-rules create iatd_labs_gcp_fw_ssh \
      --network=iatd_labs_gcp_vpc \
      --allow=tcp:22 \
      --source-ranges=0.0.0.0/0 \
      --description="Allow SSH access"

    # Allow internal communication
    gcloud compute firewall-rules create iatd_labs_gcp_fw_internal \
      --network=iatd_labs_gcp_vpc \
      --allow=tcp,udp,icmp \
      --source-ranges=192.168.0.0/16 \
      --description="Allow internal communication"
    ```

### Part 3: Setting up Cloud NAT (Mixed)

1.  **Create Cloud Router (Portal):**
    * Navigate to Cloud Routers in Google Cloud Console
    * Click "Create Router"
    * Configure:
      - Name: `iatd_labs_gcp_router`
      - Network: `iatd_labs_gcp_vpc`
      - Region: us-central1
      - Click "Create"

2.  **Configure Cloud NAT (CloudShell):**
    ```bash
    gcloud compute routers nats create iatd_labs_gcp_nat \
      --router=iatd_labs_gcp_router \
      --router-region=us-central1 \
      --nat-all-subnet-ip-ranges \
      --auto-allocate-nat-external-ips
    ```

### Part 4: Creating VM Instances (Mixed)

1.  **Create Public VM Instance (Portal):**
    * Navigate to Compute Engine → VM instances
    * Click "Create Instance"
    * Configure:
      - Name: `iatd_labs_gcp_vm_public`
      - Region: us-central1
      - Machine type: e2-micro
      - Boot disk:
        * Click "Change"
        * Operating System: Debian
        * Version: Latest Debian version (currently Debian 12)
        * Click "Select"
      - Networking:
        * Network: `iatd_labs_gcp_vpc`
        * Subnet: `iatd_labs_gcp_subnet_public`
      - Click "Create"

2.  **Create Private VM Instance (CloudShell):**
    ```bash
    # Get the latest Debian image
    LATEST_DEBIAN=$(gcloud compute images list \
      --filter="family=debian-12" \
      --format="get(name)" \
      --sort-by=~creationTimestamp \
      --limit=1)

    gcloud compute instances create iatd_labs_gcp_vm_private \
      --zone=us-central1-a \
      --machine-type=e2-micro \
      --subnet=iatd_labs_gcp_subnet_private \
      --image=$LATEST_DEBIAN \
      --image-project=debian-cloud \
      --no-address \
      --network=iatd_labs_gcp_vpc
    ```

### Part 5: Testing Connectivity

1.  **Test Public VM Access:**
    * Connect to public VM using Cloud Shell
    ```bash
    gcloud compute ssh iatd_labs_gcp_vm_public --zone=us-central1-a
    ```
    * Test internet connectivity:
    ```bash
    ping -c 4 google.com
    ```

2.  **Test Private VM Access:**
    * Connect to private VM using Cloud Shell
    ```bash
    gcloud compute ssh iatd_labs_gcp_vm_private --zone=us-central1-a
    ```
    * Test outbound internet access through Cloud NAT:
    ```bash
    ping -c 4 google.com
    ```

### Cleanup

Clean up resources using both Portal and CloudShell:

1.  **Using CloudShell:**
    ```bash
    # Delete VM instances
    gcloud compute instances delete iatd_labs_gcp_vm_public --zone=us-central1-a --quiet
    gcloud compute instances delete iatd_labs_gcp_vm_private --zone=us-central1-a --quiet

    # Delete Cloud NAT and router
    gcloud compute routers nats delete iatd_labs_gcp_nat \
      --router=iatd_labs_gcp_router \
      --router-region=us-central1 --quiet
    gcloud compute routers delete iatd_labs_gcp_router \
      --region=us-central1 --quiet

    # Delete firewall rules
    gcloud compute firewall-rules delete iatd_labs_gcp_fw_ssh --quiet
    gcloud compute firewall-rules delete iatd_labs_gcp_fw_internal --quiet
    ```

2.  **Using Portal:**
    * Navigate to VPC networks
    * Select `iatd_labs_gcp_vpc`
    * Click "Delete"
    * Confirm deletion

### Key Takeaways

*   **Mixed Interface Usage:** Experience with both Google Cloud Console and Cloud Shell
*   **VPC Architecture:** Understanding GCP VPC components and their relationships
*   **Network Segmentation:** Implementing public and private subnets
*   **Internet Access:** Configuring Cloud NAT for private instances
*   **Resource Organization:** Consistent naming and tagging practices

**Congratulations!** You have successfully completed the GCP VPC fundamentals lab using both the Google Cloud Console and Cloud Shell.
