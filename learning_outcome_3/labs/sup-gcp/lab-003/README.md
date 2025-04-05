## IATD Microcredential Cloud Networking: GCP Supplementary Lab 3 - Monitoring and Logging in Google Cloud

**Objective:** This lab introduces you to the fundamental concepts of monitoring and logging in Google Cloud. You will learn about Cloud Monitoring, Cloud Logging, and other GCP monitoring services to monitor resources, analyze logs, establish performance baselines, and set alert thresholds.

**Estimated Time:** 75 - 90 minutes

**Prerequisites:**

1.  **Google Cloud Account:** Requires an active Google Cloud account. A free tier account is available at [https://cloud.google.com/free/](https://cloud.google.com/free/).
2.  **Basic knowledge of GCP Networking:** Familiarity with VPC Networks, Firewall Rules, and Compute Engine instances.

**Let's get started!**

### Lab Conventions

*   **Google Cloud Shell:** All CLI interactions will occur within Google Cloud Shell.
*   **Google Cloud Console:** The Google Cloud Console will be used for visualization and configuration tasks.
*   **Naming Conventions:** Resource names follow the convention `iatd-labs-gcp-03-*`.
*   **IP Address Range:** The `172.16.x.x` range will be used.
*   **Region:** Choose a consistent GCP region (e.g., us-central1).

#### Resource Naming

*   **VPC Network:** `iatd-labs-gcp-03-vpc`
*   **Subnet:** `iatd-labs-gcp-03-subnet`
*   **VM Instance:** `iatd-labs-gcp-03-vm`
*   **Firewall Rule:** `iatd-labs-gcp-03-fw`
*   **Monitoring Dashboard:** `iatd-labs-gcp-03-dashboard`
*   **Alert Policy:** `iatd-labs-gcp-03-alert`
*   **Log Sink:** `iatd-labs-gcp-03-sink`

### Part 1: Setting up a Monitored Environment

1.  **Set Environment Variables:**

    ```bash
    PROJECT_ID=$(gcloud config get-value project)
    REGION="us-central1"
    ZONE="us-central1-a"
    VPC_NAME="iatd-labs-gcp-03-vpc"
    SUBNET_NAME="iatd-labs-gcp-03-subnet"
    SUBNET_RANGE="172.16.0.0/24"
    VM_NAME="iatd-labs-gcp-03-vm"
    FW_NAME="iatd-labs-gcp-03-fw"
    ```

2.  **Create a VPC Network and Subnet:**

    ```bash
    # Create VPC network
    gcloud compute networks create $VPC_NAME \
        --subnet-mode=custom
    
    # Create subnet
    gcloud compute networks subnets create $SUBNET_NAME \
        --network=$VPC_NAME \
        --region=$REGION \
        --range=$SUBNET_RANGE
    ```

    **Expected Output for network creation:**
    ```
    Created [https://www.googleapis.com/compute/v1/projects/your-project-id/global/networks/iatd-labs-gcp-03-vpc].
    NAME                 SUBNET_MODE  BGP_ROUTING_MODE  IPV4_RANGE  GATEWAY_IPV4
    iatd-labs-gcp-03-vpc  CUSTOM       REGIONAL
    ```

    **Expected Output for subnet creation:**
    ```
    Created [https://www.googleapis.com/compute/v1/projects/your-project-id/regions/us-central1/subnetworks/iatd-labs-gcp-03-subnet].
    NAME                   REGION       NETWORK              RANGE
    iatd-labs-gcp-03-subnet  us-central1  iatd-labs-gcp-03-vpc  172.16.0.0/24
    ```

3.  **Create Firewall Rules:**

    ```bash
    # Create firewall rule to allow SSH, HTTP, and ICMP
    gcloud compute firewall-rules create $FW_NAME \
        --network=$VPC_NAME \
        --allow=tcp:22,tcp:80,icmp \
        --source-ranges=0.0.0.0/0
    ```

    **Expected Output:**
    ```
    Created [https://www.googleapis.com/compute/v1/projects/your-project-id/global/firewalls/iatd-labs-gcp-03-fw].
    NAME                NETWORK              DIRECTION  PRIORITY  ALLOW                 DENY  DISABLED
    iatd-labs-gcp-03-fw  iatd-labs-gcp-03-vpc  INGRESS    1000      tcp:22,tcp:80,icmp         False
    ```

4.  **Create a Compute Engine VM Instance:**

    ```bash
    # Create VM instance
    gcloud compute instances create $VM_NAME \
        --zone=$ZONE \
        --machine-type=e2-micro \
        --subnet=$SUBNET_NAME \
        --image-family=debian-11 \
        --image-project=debian-cloud \
        --metadata=startup-script='#!/bin/bash
        apt-get update
        apt-get install -y nginx
        systemctl start nginx
        systemctl enable nginx'
    ```

    **Expected Output:**
    ```
    Created [https://www.googleapis.com/compute/v1/projects/your-project-id/zones/us-central1-a/instances/iatd-labs-gcp-03-vm].
    NAME                ZONE           MACHINE_TYPE  PREEMPTIBLE  INTERNAL_IP  EXTERNAL_IP    STATUS
    iatd-labs-gcp-03-vm  us-central1-a  e2-micro                  172.16.0.2   34.x.x.x        RUNNING
    ```

5.  **Get the External IP of the VM Instance:**

    ```bash
    # Get external IP
    EXTERNAL_IP=$(gcloud compute instances describe $VM_NAME \
        --zone=$ZONE \
        --format='get(networkInterfaces[0].accessConfigs[0].natIP)')
    
    echo "VM Instance External IP: $EXTERNAL_IP"
    ```

    **Expected Output:**
    ```
    VM Instance External IP: 34.x.x.x
    ```

6.  **Test the Web Server:**
    *   Open a web browser and navigate to the External IP address of the VM instance. You should see the Nginx welcome page.

### Part 2: Setting up Google Cloud Monitoring Services

1.  **Enable Monitoring API (if not already enabled):**

    ```bash
    gcloud services enable monitoring.googleapis.com
    ```

    **Expected Output:**
    ```
    Operation "operations/acf.p2-PROJECT_NUMBER-OPERATION_ID" finished successfully.
    ```

2.  **Create a Monitoring Dashboard (Console):**
    *   Navigate to the Cloud Monitoring console in the Google Cloud Console.
    *   Select **Dashboards** from the left navigation pane.
    *   Click **Create Dashboard**.
    *   Enter `iatd-labs-gcp-03-dashboard` as the dashboard name.
    *   Click **Save**.
    *   Add widgets to monitor CPU, memory, disk, and network metrics for your VM instance.

3.  **Install the Cloud Monitoring Agent on the VM Instance:**

    ```bash
    # SSH into the VM
    gcloud compute ssh $VM_NAME --zone=$ZONE --command="\
    curl -sSO https://dl.google.com/cloudagents/add-monitoring-agent-repo.sh && \
    sudo bash add-monitoring-agent-repo.sh && \
    sudo apt-get update && \
    sudo apt-get install -y stackdriver-agent && \
    sudo service stackdriver-agent start"
    ```

    **Expected Output:**
    ```
    OK
    ...
    The following NEW packages will be installed:
      stackdriver-agent
    ...
    Starting stackdriver-agent: [  OK  ]
    ```

4.  **Install the Cloud Logging Agent on the VM Instance:**

    ```bash
    # SSH into the VM
    gcloud compute ssh $VM_NAME --zone=$ZONE --command="\
    curl -sSO https://dl.google.com/cloudagents/add-logging-agent-repo.sh && \
    sudo bash add-logging-agent-repo.sh && \
    sudo apt-get update && \
    sudo apt-get install -y google-fluentd && \
    sudo apt-get install -y google-fluentd-catch-all-config && \
    sudo service google-fluentd start"
    ```

    **Expected Output:**
    ```
    OK
    ...
    The following NEW packages will be installed:
      google-fluentd google-fluentd-catch-all-config
    ...
    Starting google-fluentd: [  OK  ]
    ```

5.  **Configure Nginx Logs for Cloud Logging:**

    ```bash
    # SSH into the VM
    gcloud compute ssh $VM_NAME --zone=$ZONE --command="\
    sudo tee /etc/google-fluentd/config.d/nginx.conf > /dev/null << 'EOF'
    <source>
      @type tail
      format nginx
      path /var/log/nginx/access.log
      pos_file /var/lib/google-fluentd/pos/nginx-access.pos
      tag nginx-access
    </source>
    
    <source>
      @type tail
      format none
      path /var/log/nginx/error.log
      pos_file /var/lib/google-fluentd/pos/nginx-error.pos
      tag nginx-error
    </source>
    EOF
    
    sudo service google-fluentd restart"
    ```

    **Expected Output:**
    ```
    Stopping google-fluentd: [  OK  ]
    Starting google-fluentd: [  OK  ]
    ```

### Part 3: Monitoring Types and Log Analysis

1.  **Active Monitoring with Cloud Monitoring Alerts:**
    *   **Concept:** Actively monitoring metrics and triggering alerts when thresholds are exceeded.
    *   **Implementation:** Create a Cloud Monitoring alert policy to monitor CPU utilization of your VM instance.
        *   Navigate to the Cloud Monitoring console in the Google Cloud Console.
        *   Select **Alerting** from the left navigation pane.
        *   Click **Create Policy**.
        *   Select **Metric** as the condition type.
        *   Configure the alert to trigger when the VM's CPU usage exceeds 80%.
        *   Name the alert policy `iatd-labs-gcp-03-alert`.
        *   Click **Save**.

2.  **Passive Monitoring with Cloud Logging:**
    *   **Concept:** Collecting and analyzing logs without actively probing the system.
    *   **Implementation:** Navigate to the Cloud Logging console in the Google Cloud Console and explore the Nginx access and error logs collected by the Cloud Logging agent.
        *   Select **Logs Explorer** from the left navigation pane.
        *   Use the following query to filter Nginx access logs:

            ```
            resource.type="gce_instance"
            logName="projects/your-project-id/logs/nginx-access"
            ```

3.  **Synthetic Monitoring with Cloud Monitoring Uptime Checks:**
    *   **Concept:** Simulating user behavior to test application performance and availability.
    *   **Implementation:** Create a Cloud Monitoring uptime check to monitor the availability of your web server.
        *   Navigate to the Cloud Monitoring console in the Google Cloud Console.
        *   Select **Uptime Checks** from the left navigation pane.
        *   Click **Create Uptime Check**.
        *   Configure the check to monitor the HTTP endpoint of your VM instance.
        *   Name the uptime check `iatd-labs-gcp-03-uptime`.
        *   Click **Save**.

4.  **Flow-Based Monitoring with VPC Flow Logs:**
    *   **Concept:** Analyzing network traffic flow data to identify patterns and anomalies.
    *   **Implementation:** Enable VPC Flow Logs for your VPC network.

        ```bash
        # Enable VPC Flow Logs
        gcloud compute networks subnets update $SUBNET_NAME \
            --region=$REGION \
            --enable-flow-logs
        ```

        **Expected Output:**
        ```
        Updated [https://www.googleapis.com/compute/v1/projects/your-project-id/regions/us-central1/subnetworks/iatd-labs-gcp-03-subnet].
        ```

5.  **Log Analysis with Cloud Logging Query Language:**
    *   Navigate to the Cloud Logging console in the Google Cloud Console.
    *   Select **Logs Explorer** from the left navigation pane.
    *   Run the following query to analyze Nginx access logs:

        ```
        resource.type="gce_instance"
        logName="projects/your-project-id/logs/nginx-access"
        | parse jsonPayload.message as "* * * [*] \"* * *\" * *"
        | fields severity, httpRequest.requestMethod, httpRequest.status, httpRequest.latency
        ```

### Part 4: Performance Baselines and Alert Thresholds

1.  **Establishing Performance Baselines:**
    *   **Concept:** Determining the normal operating parameters of a system.
    *   **Implementation:** Monitor the VM instance's CPU, memory, disk, and network usage over a period of time to establish a baseline.
        *   Navigate to the Cloud Monitoring console in the Google Cloud Console.
        *   Select **Metrics Explorer** from the left navigation pane.
        *   Explore the Compute Engine metrics for your instance.
        *   Observe the patterns over time to establish a baseline.

2.  **Setting Alert Thresholds:**
    *   **Concept:** Defining thresholds that trigger alerts when exceeded.
    *   **Implementation:** Create a Cloud Monitoring alert policy to notify you when the VM instance's disk usage exceeds 80%.
        *   Navigate to the Cloud Monitoring console in the Google Cloud Console.
        *   Select **Alerting** from the left navigation pane.
        *   Click **Create Policy**.
        *   Select **Metric** as the condition type.
        *   Configure the alert to trigger when the VM's disk usage exceeds 80%.
        *   Name the alert policy `iatd-labs-gcp-03-disk-alert`.
        *   Click **Save**.

3.  **Static vs. Dynamic Thresholds:**
    *   **Concept:**
        *   **Static Thresholds:** Fixed values that trigger alerts (e.g., CPU > 80%).
        *   **Dynamic Thresholds:** Automatically adjust based on historical data and seasonality.
    *   **Implementation:** Discuss the pros and cons of each approach and when to use them.

### Part 5: Cloud Audit Logging

1.  **Enable Cloud Audit Logging:**

    ```bash
    # Enable Cloud Audit Logging
    gcloud logging sinks create iatd-labs-gcp-03-sink \
        storage.googleapis.com/iatd-labs-gcp-03-audit-logs \
        --log-filter='logName:"projects/'$PROJECT_ID'/logs/cloudaudit.googleapis.com%2Factivity"'
    ```

    **Note:** This command assumes you have already created a Cloud Storage bucket named `iatd-labs-gcp-03-audit-logs`. If not, you'll need to create it first.

2.  **Explore Cloud Audit Logs:**
    *   Navigate to the Cloud Logging console in the Google Cloud Console.
    *   Select **Logs Explorer** from the left navigation pane.
    *   Use the following query to filter Cloud Audit Logs:

        ```
        logName="projects/your-project-id/logs/cloudaudit.googleapis.com%2Factivity"
        ```

### Post-Lab: Cleanup

To avoid incurring unnecessary costs, it's essential to clean up the resources created during this lab.

1.  **Delete Alert Policies:**

    ```bash
    # Delete alert policies
    gcloud alpha monitoring policies delete projects/$PROJECT_ID/alertPolicies/iatd-labs-gcp-03-alert
    gcloud alpha monitoring policies delete projects/$PROJECT_ID/alertPolicies/iatd-labs-gcp-03-disk-alert
    ```

2.  **Delete Uptime Check:**

    ```bash
    # Delete uptime check
    gcloud alpha monitoring uptime-check-configs delete projects/$PROJECT_ID/uptimeCheckConfigs/iatd-labs-gcp-03-uptime
    ```

3.  **Delete Log Sink:**

    ```bash
    # Delete log sink
    gcloud logging sinks delete iatd-labs-gcp-03-sink
    ```

4.  **Delete VM Instance:**

    ```bash
    # Delete VM instance
    gcloud compute instances delete $VM_NAME \
        --zone=$ZONE \
        --quiet
    ```

    **Expected Output:**
    ```
    Deleted [https://www.googleapis.com/compute/v1/projects/your-project-id/zones/us-central1-a/instances/iatd-labs-gcp-03-vm].
    ```

5.  **Delete Firewall Rule:**

    ```bash
    # Delete firewall rule
    gcloud compute firewall-rules delete $FW_NAME --quiet
    ```

    **Expected Output:**
    ```
    Deleted [https://www.googleapis.com/compute/v1/projects/your-project-id/global/firewalls/iatd-labs-gcp-03-fw].
    ```

6.  **Delete Subnet:**

    ```bash
    # Delete subnet
    gcloud compute networks subnets delete $SUBNET_NAME \
        --region=$REGION \
        --quiet
    ```

    **Expected Output:**
    ```
    Deleted [https://www.googleapis.com/compute/v1/projects/your-project-id/regions/us-central1/subnetworks/iatd-labs-gcp-03-subnet].
    ```

7.  **Delete VPC Network:**

    ```bash
    # Delete VPC network
    gcloud compute networks delete $VPC_NAME --quiet
    ```

    **Expected Output:**
    ```
    Deleted [https://www.googleapis.com/compute/v1/projects/your-project-id/global/networks/iatd-labs-gcp-03-vpc].
    ```

8.  **Delete Cloud Storage Bucket (if created):**

    ```bash
    # Delete Cloud Storage bucket
    gsutil rm -r gs://iatd-labs-gcp-03-audit-logs
    ```

9.  **Verify Deletion:**
    *   Confirm that all resources have been deleted by checking the Google Cloud Console or using the gcloud CLI.

**Outcomes to be Achieved:**

*   Understand different monitoring types in Google Cloud (Cloud Monitoring metrics, logs, uptime checks).
*   Analyze logs using Cloud Logging Query Language.
*   Establish performance baselines and set alert thresholds using Cloud Monitoring Alert Policies.
*   Configure and use core Google Cloud monitoring services (Cloud Monitoring, Cloud Logging).
*   Implement VPC Flow Logs for network traffic analysis.

**Congratulations!** You have successfully completed this lab on monitoring and logging in Google Cloud. You've learned about Cloud Monitoring, Cloud Logging, and other GCP monitoring services to monitor resources, analyze logs, establish performance baselines, and set alert thresholds.
