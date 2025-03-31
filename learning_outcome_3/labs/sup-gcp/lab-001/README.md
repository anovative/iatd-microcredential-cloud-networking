## IATD Microcredential Cloud Networking: Supplementary Lab - GCP Firewall Rules and VPC Service Controls

**Objective:** This lab focuses on implementing network security in Google Cloud Platform using Firewall Rules and VPC Service Controls. You'll learn how these security mechanisms compare to Azure's NSGs and ASGs, and how to implement layered security in GCP environments.

**Estimated Time:** 90-120 minutes

**Prerequisites:**
1. **GCP Account:** An active Google Cloud Platform account with billing enabled
2. **gcloud CLI:** Installed and configured with appropriate credentials
3. **Basic Understanding:** 
   - GCP VPC networking concepts
   - Firewall rules and security policies
   - Basic networking principles
   - GCP VM instance management

### Resource Naming Convention

* **VPC Networks:**
  - Main VPC: `iatd-labs-gcp-sec-vpc`
* **Subnets:**
  - Web Subnet: `iatd-labs-gcp-web-subnet`
  - App Subnet: `iatd-labs-gcp-app-subnet`
* **Firewall Rules:**
  - Allow Web Traffic: `iatd-labs-gcp-allow-web`
  - Allow Internal Traffic: `iatd-labs-gcp-allow-internal`
  - Allow SSH: `iatd-labs-gcp-allow-ssh`
* **VM Instances:**
  - Web Server: `iatd-labs-gcp-web-vm`
  - App Server: `iatd-labs-gcp-app-vm`
* **Service Perimeter:**
  - Security Perimeter: `iatd-labs-gcp-service-perimeter`

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
     accesscontextmanager.googleapis.com
   ```

### Part 2: Creating VPC Network and Subnets (CloudShell)

1. **Create VPC Network:**
   ```bash
   gcloud compute networks create iatd-labs-gcp-sec-vpc \
     --subnet-mode=custom \
     --description="VPC for security lab"
   ```

   Expected Output:
   ```
   Created [https://www.googleapis.com/compute/v1/projects/YOUR_PROJECT/global/networks/iatd-labs-gcp-sec-vpc].
   NAME                   SUBNET_MODE  BGP_ROUTING_MODE  IPV4_RANGE  GATEWAY_IPV4
   iatd-labs-gcp-sec-vpc  CUSTOM       REGIONAL
   ```

2. **Create Subnets:**
   ```bash
   # Create web subnet
   gcloud compute networks subnets create iatd-labs-gcp-web-subnet \
     --network=iatd-labs-gcp-sec-vpc \
     --region=us-west1 \
     --range=172.16.1.0/24 \
     --description="Web tier subnet"

   # Create app subnet
   gcloud compute networks subnets create iatd-labs-gcp-app-subnet \
     --network=iatd-labs-gcp-sec-vpc \
     --region=us-west1 \
     --range=172.16.2.0/24 \
     --description="App tier subnet"
   ```

   Expected Output (for the first command):
   ```
   Created [https://www.googleapis.com/compute/v1/projects/YOUR_PROJECT/regions/us-west1/subnetworks/iatd-labs-gcp-web-subnet].
   NAME                      REGION    NETWORK                RANGE
   iatd-labs-gcp-web-subnet  us-west1  iatd-labs-gcp-sec-vpc  172.16.1.0/24
   ```

### Part 3: Creating Firewall Rules (CloudShell)

1. **Create Firewall Rule for Web Traffic:**
   ```bash
   gcloud compute firewall-rules create iatd-labs-gcp-allow-web \
     --direction=INGRESS \
     --priority=1000 \
     --network=iatd-labs-gcp-sec-vpc \
     --action=ALLOW \
     --rules=tcp:80,tcp:443 \
     --source-ranges=0.0.0.0/0 \
     --target-tags=web-server \
     --description="Allow HTTP and HTTPS traffic from anywhere"
   ```

   Expected Output:
   ```
   Creating firewall...⠏
   Created [https://www.googleapis.com/compute/v1/projects/YOUR_PROJECT/global/firewalls/iatd-labs-gcp-allow-web].
   NAME                    NETWORK                DIRECTION  PRIORITY  ALLOW        DENY  DISABLED
   iatd-labs-gcp-allow-web  iatd-labs-gcp-sec-vpc  INGRESS    1000      tcp:80,443         False
   ```

2. **Create Firewall Rule for Internal Traffic:**
   ```bash
   gcloud compute firewall-rules create iatd-labs-gcp-allow-internal \
     --direction=INGRESS \
     --priority=1000 \
     --network=iatd-labs-gcp-sec-vpc \
     --action=ALLOW \
     --rules=tcp:8080 \
     --source-tags=web-server \
     --target-tags=app-server \
     --description="Allow traffic from web to app tier"
   ```

3. **Create Firewall Rule for SSH Access:**
   ```bash
   gcloud compute firewall-rules create iatd-labs-gcp-allow-ssh \
     --direction=INGRESS \
     --priority=1000 \
     --network=iatd-labs-gcp-sec-vpc \
     --action=ALLOW \
     --rules=tcp:22 \
     --source-ranges=0.0.0.0/0 \
     --description="Allow SSH from anywhere"
   ```

### Part 4: Creating VM Instances (CloudShell)

1. **Create Web Server VM:**
   ```bash
   gcloud compute instances create iatd-labs-gcp-web-vm \
     --zone=us-west1-a \
     --machine-type=e2-micro \
     --subnet=iatd-labs-gcp-web-subnet \
     --tags=web-server \
     --image-family=debian-10 \
     --image-project=debian-cloud \
     --metadata=startup-script='#! /bin/bash
apt-get update
apt-get install -y apache2
cat <<EOF > /var/www/html/index.html
<html><body><h1>Hello from Web Server</h1></body></html>
EOF' \
     --service-account=iatd-labs-gcp-sa@${PROJECT_ID}.iam.gserviceaccount.com
   ```

   Expected Output:
   ```
   Created [https://www.googleapis.com/compute/v1/projects/YOUR_PROJECT/zones/us-west1-a/instances/iatd-labs-gcp-web-vm].
   NAME                 ZONE        MACHINE_TYPE  PREEMPTIBLE  INTERNAL_IP  EXTERNAL_IP    STATUS
   iatd-labs-gcp-web-vm  us-west1-a  e2-micro                  172.16.1.2    34.82.156.210  RUNNING
   ```

2. **Create App Server VM:**
   ```bash
   gcloud compute instances create iatd-labs-gcp-app-vm \
     --zone=us-west1-a \
     --machine-type=e2-micro \
     --subnet=iatd-labs-gcp-app-subnet \
     --tags=app-server \
     --image-family=debian-10 \
     --image-project=debian-cloud \
     --metadata=startup-script='#! /bin/bash
apt-get update
apt-get install -y apache2
sed -i \'s/Listen 80/Listen 8080/g\' /etc/apache2/ports.conf
sed -i \'s/:80/:8080/g\' /etc/apache2/sites-enabled/000-default.conf
cat <<EOF > /var/www/html/index.html
<html><body><h1>Hello from App Server</h1></body></html>
EOF
systemctl restart apache2' \
     --service-account=iatd-labs-gcp-sa@${PROJECT_ID}.iam.gserviceaccount.com \
     --no-address
   ```

### Part 5: Setting Up VPC Service Controls (Portal)

1. **Create Access Policy (if not already created):**
   * Navigate to **Security > VPC Service Controls** in the GCP Console.
   * Click **Create** to create a new access policy if you don't have one.
   * Enter a name like `iatd-labs-access-policy`.
   * Click **Create Policy**.

2. **Create Service Perimeter:**
   * In the VPC Service Controls page, click **New Perimeter**.
   * Enter `iatd-labs-gcp-service-perimeter` as the name.
   * Select your project.
   * Under **Services**, select the following services:
     * Compute Engine API
     * Cloud Storage
   * Under **VPC Accessible Services**, select **Enable VPC Accessible Services** and choose the same services.
   * Under **Access Levels**, you can create a new access level to restrict access based on IP ranges.
   * Click **Create Perimeter**.

### Part 6: Testing and Verification

1. **Test Web Server Access:**
   ```bash
   # Get the external IP of the web server
   WEB_VM_IP=$(gcloud compute instances describe iatd-labs-gcp-web-vm \
     --zone=us-west1-a \
     --format='get(networkInterfaces[0].accessConfigs[0].natIP)')
   
   # Test HTTP access
   curl http://$WEB_VM_IP
   ```

   Expected Output:
   ```html
   <html><body><h1>Hello from Web Server</h1></body></html>
   ```

2. **Test Internal Communication:**
   ```bash
   # SSH to the web server
   gcloud compute ssh iatd-labs-gcp-web-vm --zone=us-west1-a
   
   # From the web server, get the internal IP of the app server
   APP_VM_IP=$(gcloud compute instances describe iatd-labs-gcp-app-vm \
     --zone=us-west1-a \
     --format='get(networkInterfaces[0].networkIP)')
   
   # Test connection to app server
   curl http://$APP_VM_IP:8080
   ```

   Expected Output:
   ```html
   <html><body><h1>Hello from App Server</h1></body></html>
   ```

3. **Verify Firewall Rules:**
   ```bash
   # List all firewall rules for the VPC
   gcloud compute firewall-rules list \
     --filter="network:iatd-labs-gcp-sec-vpc"
   ```

4. **Test VPC Service Controls:**
   * Try to access a protected service from outside the perimeter.
   * Verify that access is denied as expected.

### Post-Lab: Cleanup

1. **Delete VM Instances:**
   ```bash
   gcloud compute instances delete iatd-labs-gcp-web-vm iatd-labs-gcp-app-vm \
     --zone=us-west1-a \
     --quiet
   ```

2. **Delete Firewall Rules:**
   ```bash
   gcloud compute firewall-rules delete iatd-labs-gcp-allow-web iatd-labs-gcp-allow-internal iatd-labs-gcp-allow-ssh \
     --quiet
   ```

3. **Delete Subnets:**
   ```bash
   gcloud compute networks subnets delete iatd-labs-gcp-web-subnet iatd-labs-gcp-app-subnet \
     --region=us-west1 \
     --quiet
   ```

4. **Delete VPC Network:**
   ```bash
   gcloud compute networks delete iatd-labs-gcp-sec-vpc \
     --quiet
   ```

5. **Delete Service Perimeter:**
   * Navigate to **Security > VPC Service Controls** in the GCP Console.
   * Select your perimeter and click **Delete**.

6. **Delete Service Account:**
   ```bash
   gcloud iam service-accounts delete iatd-labs-gcp-sa@${PROJECT_ID}.iam.gserviceaccount.com \
     --quiet
   ```

**Congratulations!** You've successfully implemented network security in Google Cloud Platform using Firewall Rules and VPC Service Controls. This lab has demonstrated key concepts in GCP network security that parallel the NSG and ASG implementations covered in the Azure labs.

### Troubleshooting Guide

1. **Firewall Rule Issues:**
   - Verify rule priority (lower numbers have higher priority)
   - Check source and target tags are correctly applied
   - Ensure IP ranges are correctly specified
   - Verify the rules are associated with the correct VPC network

2. **VM Connectivity Issues:**
   - Check that VMs have the correct network tags
   - Verify firewall rules are correctly configured
   - Ensure VMs are in the correct subnets
   - Check that service accounts have appropriate permissions

3. **VPC Service Controls Issues:**
   - Verify the perimeter is correctly configured
   - Check that the right services are included
   - Ensure your project is correctly added to the perimeter
   - Test access from both inside and outside the perimeter

4. **Common Mistakes:**
   - Forgetting to add network tags to VMs
   - Misconfiguring firewall rule priorities
   - Using incorrect IP ranges
   - Not granting sufficient permissions to service accounts