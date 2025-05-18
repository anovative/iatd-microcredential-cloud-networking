## IATD Microcredential Cloud Networking: GCP Lab 1 - Load Balancing with Google Cloud Load Balancer

**Objective:** This lab guides you through implementing load balancing in Google Cloud Platform (GCP) using the HTTP(S) Load Balancer. You'll learn how to distribute traffic across multiple compute instances in different regions, configure health checks, and implement URL-based routing.

**Estimated Time:** 75 - 90 minutes

**Prerequisites:**

1.  **GCP Account:** Requires an active GCP account. A free tier account is available at [https://cloud.google.com/free/](https://cloud.google.com/free/).
2.  **Basic understanding of GCP Networking:** Familiarity with VPC networks, subnets, and Compute Engine instances.

### Lab Conventions

*   **Google Cloud Console:** Most interactions will occur via the Google Cloud Console.
*   **Naming Conventions:** Resource names follow the convention `iatd-gcp-lb-*`.
*   **IP Address Range:** The `172.16.0.0/16` range will be used (as per standardization).
*   **Region:** Resources will be deployed across multiple regions (us-central1 and us-east1).

#### Resource Naming

*   **VPC Network:** `iatd-gcp-lb-vpc`
*   **Subnet - Region 1:** `iatd-gcp-lb-subnet-central`
*   **Subnet - Region 2:** `iatd-gcp-lb-subnet-east`
*   **Firewall Rule:** `iatd-gcp-lb-fw-allow-http`
*   **Instance Template - Region 1:** `iatd-gcp-lb-template-central`
*   **Instance Template - Region 2:** `iatd-gcp-lb-template-east`
*   **Instance Group - Region 1:** `iatd-gcp-lb-ig-central`
*   **Instance Group - Region 2:** `iatd-gcp-lb-ig-east`
*   **Health Check:** `iatd-gcp-lb-health-check`
*   **Backend Service:** `iatd-gcp-lb-backend`
*   **URL Map:** `iatd-gcp-lb-url-map`
*   **HTTP Proxy:** `iatd-gcp-lb-http-proxy`
*   **Global Forwarding Rule:** `iatd-gcp-lb-forwarding-rule`

### Part 1: Setting up the VPC Network and Subnets

1.  **Sign in to the Google Cloud Console:** Browse to [https://console.cloud.google.com/](https://console.cloud.google.com/) and sign in.

2.  **Create a VPC Network:**
    *   Navigate to **VPC network** > **VPC networks**.
    *   Click **Create VPC Network**.
    *   Configure the VPC network:
        *   **Name:** `iatd-gcp-lb-vpc`
        *   **Description:** `VPC for load balancing lab`
        *   **Subnet creation mode:** Custom
        *   Add the first subnet:
            *   **Name:** `iatd-gcp-lb-subnet-central`
            *   **Region:** us-central1
            *   **IP address range:** `172.16.1.0/24`
        *   Add the second subnet:
            *   **Name:** `iatd-gcp-lb-subnet-east`
            *   **Region:** us-east1
            *   **IP address range:** `172.16.2.0/24`
    *   Click **Create**.

3.  **Create Firewall Rules:**
    *   Navigate to **VPC network** > **Firewall**.
    *   Click **Create Firewall Rule**.
    *   Configure the firewall rule:
        *   **Name:** `iatd-gcp-lb-fw-allow-http`
        *   **Network:** `iatd-gcp-lb-vpc`
        *   **Priority:** 1000
        *   **Direction of traffic:** Ingress
        *   **Action on match:** Allow
        *   **Targets:** All instances in the network
        *   **Source filter:** IP ranges
        *   **Source IP ranges:** `0.0.0.0/0`
        *   **Protocols and ports:** Specified protocols and ports
            *   Check TCP and enter port 80
    *   Click **Create**.

    Expected output:
    ```
    VPC network created successfully with the following components:
    - VPC network with custom subnets in two regions
    - Firewall rule allowing HTTP traffic from any source
    ```

### Part 2: Creating Instance Templates and Managed Instance Groups

1.  **Create Instance Template for Region 1:**
    *   Navigate to **Compute Engine** > **Instance templates**.
    *   Click **Create instance template**.
    *   Configure the instance template:
        *   **Name:** `iatd-gcp-lb-template-central`
        *   **Machine configuration:** e2-micro (2 vCPU, 1 GB memory)
        *   **Boot disk:** Debian GNU/Linux 11 (bullseye)
        *   **Service account:** Default compute service account
        *   **Access scopes:** Allow default access
        *   **Networking:**
            *   **Network:** `iatd-gcp-lb-vpc`
            *   **Subnet:** `iatd-gcp-lb-subnet-central`
        *   **Management:**
            *   **Startup script:**
                ```bash
                #! /bin/bash
                apt-get update
                apt-get install -y apache2
                cat <<EOF > /var/www/html/index.html
                <html><body><h1>Hello from Central Region!</h1>
                <p>Server: $(hostname)</p>
                <p>Region: us-central1</p>
                </body></html>
                EOF
                systemctl restart apache2
                ```
    *   Click **Create**.

2.  **Create Instance Template for Region 2:**
    *   Click **Create instance template** again.
    *   Configure the instance template similarly to the first one, but with these changes:
        *   **Name:** `iatd-gcp-lb-template-east`
        *   **Networking:**
            *   **Subnet:** `iatd-gcp-lb-subnet-east`
        *   **Startup script:** Update the HTML to show "Hello from East Region!" and "Region: us-east1"
    *   Click **Create**.

3.  **Create Managed Instance Group for Region 1:**
    *   Navigate to **Compute Engine** > **Instance groups**.
    *   Click **Create instance group**.
    *   Configure the instance group:
        *   **Name:** `iatd-gcp-lb-ig-central`
        *   **Instance template:** `iatd-gcp-lb-template-central`
        *   **Location:** Multiple zones
        *   **Region:** us-central1
        *   **Autoscaling:** Off
        *   **Number of instances:** 2
    *   Click **Create**.

4.  **Create Managed Instance Group for Region 2:**
    *   Click **Create instance group** again.
    *   Configure the instance group similarly to the first one, but with these changes:
        *   **Name:** `iatd-gcp-lb-ig-east`
        *   **Instance template:** `iatd-gcp-lb-template-east`
        *   **Region:** us-east1
    *   Click **Create**.

    Expected output:
    ```
    Instance templates and managed instance groups created successfully:
    - Two instance templates, one for each region
    - Two managed instance groups with 2 instances each
    - Each instance runs a web server with region-specific content
    ```

### Part 3: Setting up the HTTP(S) Load Balancer

1.  **Create Health Check:**
    *   Navigate to **Compute Engine** > **Health checks**.
    *   Click **Create health check**.
    *   Configure the health check:
        *   **Name:** `iatd-gcp-lb-health-check`
        *   **Protocol:** HTTP
        *   **Port:** 80
        *   **Request path:** `/`
        *   **Check interval:** 5 seconds
        *   **Timeout:** 5 seconds
        *   **Healthy threshold:** 2
        *   **Unhealthy threshold:** 2
    *   Click **Create**.

2.  **Create Backend Service:**
    *   Navigate to **Network services** > **Load balancing**.
    *   Click **Create load balancer**.
    *   Select **HTTP(S) Load Balancing** and click **Start configuration**.
    *   Select **From Internet to my VMs** and click **Continue**.
    *   Set the name to `iatd-gcp-lb-http`.
    *   In the **Backend configuration** section, click **Backend services** > **Create a backend service**.
    *   Configure the backend service:
        *   **Name:** `iatd-gcp-lb-backend`
        *   **Backend type:** Instance group
        *   **Protocol:** HTTP
        *   **Named port:** http
        *   **Timeout:** 30 seconds
        *   **Health check:** `iatd-gcp-lb-health-check`
        *   **Session affinity:** None
        *   **Add backend:**
            *   **Instance group:** `iatd-gcp-lb-ig-central`
            *   **Port numbers:** 80
            *   **Balancing mode:** Utilization
            *   **Maximum backend utilization:** 80%
            *   **Capacity scaler:** 1.0
        *   **Add another backend:**
            *   **Instance group:** `iatd-gcp-lb-ig-east`
            *   **Port numbers:** 80
            *   **Balancing mode:** Utilization
            *   **Maximum backend utilization:** 80%
            *   **Capacity scaler:** 1.0
        *   **Enable Cloud CDN:** No
    *   Click **Create**.

3.  **Configure Frontend:**
    *   In the **Frontend configuration** section, click **Add frontend IP and port**.
    *   Configure the frontend:
        *   **Name:** `iatd-gcp-lb-frontend`
        *   **Protocol:** HTTP
        *   **IP version:** IPv4
        *   **IP address:** Create IP address
            *   **Name:** `iatd-gcp-lb-ip`
            *   **Static IP address:** Let me choose
            *   Click **Reserve**
        *   **Port:** 80
    *   Click **Done**.

4.  **Review and Finalize:**
    *   Review your configuration.
    *   Click **Create**.

    Expected output:
    ```
    HTTP(S) Load Balancer created successfully:
    - Global load balancer with HTTP frontend
    - Backend service with instance groups in two regions
    - Health check configured to monitor instance health
    ```

### Part 4: Testing the Load Balancer

1.  **Wait for the Load Balancer to be Provisioned:**
    *   It may take a few minutes for the load balancer to be fully provisioned.
    *   Navigate to **Network services** > **Load balancing**.
    *   Wait for the load balancer to show a green checkmark indicating it's healthy.

2.  **Test Load Balancing:**
    *   Note the IP address of the load balancer from the **Frontend** section.
    *   Open a web browser and navigate to `http://<load-balancer-ip>`.
    *   Refresh the page multiple times. You should see the content alternating between "Hello from Central Region!" and "Hello from East Region!", demonstrating that the load balancer is distributing traffic across both regions.

    Expected output:
    ```
    - Browser shows either "Hello from Central Region!" or "Hello from East Region!" when accessing the load balancer IP
    - Refreshing the page multiple times shows alternating content, confirming global load balancing is working
    - Each page shows the hostname of the specific instance serving the request
    ```

### Part 5: Implementing URL-Based Routing (Optional)

1.  **Modify Instance Templates for URL-Based Routing:**
    *   Create a new instance template for images:
        *   **Name:** `iatd-gcp-lb-template-images`
        *   **Startup script:**
            ```bash
            #! /bin/bash
            apt-get update
            apt-get install -y apache2
            mkdir -p /var/www/html/images
            cat <<EOF > /var/www/html/images/index.html
            <html><body><h1>Images Section</h1>
            <p>Server: $(hostname)</p>
            </body></html>
            EOF
            systemctl restart apache2
            ```
    *   Create a new instance template for videos:
        *   **Name:** `iatd-gcp-lb-template-videos`
        *   **Startup script:**
            ```bash
            #! /bin/bash
            apt-get update
            apt-get install -y apache2
            mkdir -p /var/www/html/videos
            cat <<EOF > /var/www/html/videos/index.html
            <html><body><h1>Videos Section</h1>
            <p>Server: $(hostname)</p>
            </body></html>
            EOF
            systemctl restart apache2
            ```

2.  **Create Instance Groups for URL-Based Routing:**
    *   Create a managed instance group for images:
        *   **Name:** `iatd-gcp-lb-ig-images`
        *   **Instance template:** `iatd-gcp-lb-template-images`
    *   Create a managed instance group for videos:
        *   **Name:** `iatd-gcp-lb-ig-videos`
        *   **Instance template:** `iatd-gcp-lb-template-videos`

3.  **Create Backend Services for URL-Based Routing:**
    *   Create a backend service for images:
        *   **Name:** `iatd-gcp-lb-backend-images`
        *   **Backend:** `iatd-gcp-lb-ig-images`
    *   Create a backend service for videos:
        *   **Name:** `iatd-gcp-lb-backend-videos`
        *   **Backend:** `iatd-gcp-lb-ig-videos`

4.  **Update URL Map for Path-Based Routing:**
    *   Navigate to **Network services** > **Load balancing**.
    *   Select your load balancer.
    *   Click **Edit**.
    *   In the **Host and path rules** section, add the following rules:
        *   **Hosts:** Any host
        *   **Paths:** `/images/*`
        *   **Backend:** `iatd-gcp-lb-backend-images`
    *   Add another rule:
        *   **Hosts:** Any host
        *   **Paths:** `/videos/*`
        *   **Backend:** `iatd-gcp-lb-backend-videos`
    *   Click **Update**.

5.  **Test URL-Based Routing:**
    *   Access the load balancer IP with the following paths:
        *   `/images` - Should show "Images Section"
        *   `/videos` - Should show "Videos Section"

    Expected outcome:
    ```
    - When accessing /images path: You should see "Images Section"
    - When accessing /videos path: You should see "Videos Section"
    - The same load balancer IP routes to different backend services based on the URL path
    ```

### Post-Lab: Cleanup

To avoid incurring unnecessary costs, it's essential to clean up the resources created during this lab.

1.  **Delete Load Balancer:**
    *   Navigate to **Network services** > **Load balancing**.
    *   Select your load balancer.
    *   Click **Delete**.
    *   Confirm deletion.

2.  **Delete Health Check:**
    *   Navigate to **Compute Engine** > **Health checks**.
    *   Select `iatd-gcp-lb-health-check`.
    *   Click **Delete**.
    *   Confirm deletion.

3.  **Delete Instance Groups:**
    *   Navigate to **Compute Engine** > **Instance groups**.
    *   Select all instance groups created in this lab.
    *   Click **Delete**.
    *   Confirm deletion.

4.  **Delete Instance Templates:**
    *   Navigate to **Compute Engine** > **Instance templates**.
    *   Select all instance templates created in this lab.
    *   Click **Delete**.
    *   Confirm deletion.

5.  **Delete Firewall Rules:**
    *   Navigate to **VPC network** > **Firewall**.
    *   Select `iatd-gcp-lb-fw-allow-http`.
    *   Click **Delete**.
    *   Confirm deletion.

6.  **Delete VPC Network:**
    *   Navigate to **VPC network** > **VPC networks**.
    *   Select `iatd-gcp-lb-vpc`.
    *   Click **Delete**.
    *   Confirm deletion.

**Learning Outcomes:**

*   Successfully created and configured a global HTTP(S) Load Balancer in Google Cloud Platform.
*   Successfully distributed traffic across multiple compute instances in different regions.
*   Successfully implemented and tested URL-based routing.
*   Understood how to configure health checks and backend services for load balancing in GCP.
*   Learned about global load balancing and multi-region deployment for high availability and low latency.
