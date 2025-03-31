## IATD Microcredential Cloud Networking: Supplementary Lab - GCP DDoS Mitigation Fundamentals

**Objective:** This lab introduces you to DDoS mitigation techniques using Google Cloud Armor and Google Cloud Load Balancing. You'll learn how these security mechanisms compare to Azure's DDoS Protection and how to implement layered security in GCP environments.

**Estimated Time:** 90-120 minutes

**Prerequisites:**
1. **GCP Account:** An active Google Cloud Platform account with billing enabled
2. **gcloud CLI:** Installed and configured with appropriate credentials
3. **Basic Understanding:** 
   - GCP VPC networking concepts
   - Cloud Armor and Load Balancing
   - Basic networking principles
   - GCP VM instance management

### Resource Naming Convention

* **VPC Network:** `iatd-labs-gcp-ddos-vpc`
* **Subnet:** `iatd-labs-gcp-ddos-subnet`
* **Compute Engine Instance:** `iatd-labs-gcp-ddos-vm`
* **Firewall Rule:** `iatd-labs-gcp-ddos-fw`
* **Google Cloud Armor Policy:** `iatd-labs-gcp-ddos-armor`
* **Google Cloud Load Balancer:** `iatd-labs-gcp-ddos-lb`

### Part 1: Setting Up the Environment (CloudShell)

1. **Set Variables:**
   ```bash
   # Set project ID
   PROJECT_ID=$(gcloud config get-value project)
   REGION="us-central1"
   ZONE="us-central1-a"
   
   # Resource names
   VPC_NAME="iatd-labs-gcp-ddos-vpc"
   SUBNET_NAME="iatd-labs-gcp-ddos-subnet"
   VM_NAME="iatd-labs-gcp-ddos-vm"
   FW_NAME="iatd-labs-gcp-ddos-fw"
   ARMOR_POLICY="iatd-labs-gcp-ddos-armor"
   LB_NAME="iatd-labs-gcp-ddos-lb"
   ```

2. **Create a VPC Network and Subnet:**
   ```bash
   # Create VPC
   gcloud compute networks create $VPC_NAME \
     --subnet-mode=custom
   
   # Create subnet
   gcloud compute networks subnets create $SUBNET_NAME \
     --network=$VPC_NAME \
     --region=$REGION \
     --range=172.16.0.0/24
   ```

   **Expected Output:**
   ```
   Created [https://www.googleapis.com/compute/v1/projects/PROJECT_ID/global/networks/iatd-labs-gcp-ddos-vpc].
   NAME                  SUBNET_MODE  BGP_ROUTING_MODE  IPV4_RANGE  GATEWAY_IPV4
   iatd-labs-gcp-ddos-vpc  CUSTOM       REGIONAL
   
   Created [https://www.googleapis.com/compute/v1/projects/PROJECT_ID/regions/us-central1/subnetworks/iatd-labs-gcp-ddos-subnet].
   NAME                     REGION       NETWORK                 RANGE
   iatd-labs-gcp-ddos-subnet  us-central1  iatd-labs-gcp-ddos-vpc  172.16.0.0/24
   ```

3. **Create a Firewall Rule:**
   ```bash
   # Create firewall rule for HTTP and SSH
   gcloud compute firewall-rules create $FW_NAME \
     --network=$VPC_NAME \
     --allow=tcp:22,tcp:80,tcp:443 \
     --source-ranges=0.0.0.0/0 \
     --description="Allow HTTP, HTTPS and SSH from anywhere"
   ```

   **Expected Output:**
   ```
   Created [https://www.googleapis.com/compute/v1/projects/PROJECT_ID/global/firewalls/iatd-labs-gcp-ddos-fw].
   NAME                 NETWORK                 DIRECTION  PRIORITY  ALLOW        DENY  DISABLED
   iatd-labs-gcp-ddos-fw  iatd-labs-gcp-ddos-vpc  INGRESS    1000      tcp:22,80,443        False
   ```

### Part 2: Setting up a Target Compute Engine Instance

1. **Create a Compute Engine Instance:**
   ```bash
   gcloud compute instances create $VM_NAME \
     --zone=$ZONE \
     --machine-type=e2-micro \
     --subnet=$SUBNET_NAME \
     --image-family=debian-11 \
     --image-project=debian-cloud \
     --tags=http-server,https-server \
     --metadata=startup-script='#! /bin/bash
       apt-get update
       apt-get install -y apache2
       echo "<html><body><h1>Hello from GCP VM</h1></body></html>" > /var/www/html/index.html
       systemctl start apache2
       systemctl enable apache2'
   ```

   **Expected Output:**
   ```
   Created [https://www.googleapis.com/compute/v1/projects/PROJECT_ID/zones/us-central1-a/instances/iatd-labs-gcp-ddos-vm].
   NAME                  ZONE           MACHINE_TYPE  PREEMPTIBLE  INTERNAL_IP  EXTERNAL_IP    STATUS
   iatd-labs-gcp-ddos-vm  us-central1-a  e2-micro                  172.16.0.2   34.123.456.789  RUNNING
   ```

2. **Get the External IP Address of the VM:**
   ```bash
   VM_IP=$(gcloud compute instances describe $VM_NAME \
     --zone=$ZONE \
     --format="get(networkInterfaces[0].accessConfigs[0].natIP)")
   
   echo "VM external IP: $VM_IP"
   ```

   **Expected Output:**
   ```
   VM external IP: 34.123.456.789
   ```

3. **Test the Web Server:**
   ```bash
   curl http://$VM_IP
   ```

   **Expected Output:**
   ```html
   <html><body><h1>Hello from GCP VM</h1></body></html>
   ```

### Part 3: Implementing Google Cloud Armor

1. **Create a Google Cloud Armor Security Policy:**
   ```bash
   gcloud compute security-policies create $ARMOR_POLICY \
     --description="DDoS protection policy"
   ```

   **Expected Output:**
   ```
   Created [https://www.googleapis.com/compute/v1/projects/PROJECT_ID/global/securityPolicies/iatd-labs-gcp-ddos-armor].
   ```

2. **Add a Rule to Block Traffic from a Specific IP Range:**
   ```bash
   # Example: Block traffic from 1.2.3.0/24
   gcloud compute security-policies rules create 1000 \
     --security-policy=$ARMOR_POLICY \
     --description="Block specific IP range" \
     --src-ip-ranges="1.2.3.0/24" \
     --action="deny-403"
   ```

   **Expected Output:**
   ```
   Created [https://www.googleapis.com/compute/v1/projects/PROJECT_ID/global/securityPolicies/iatd-labs-gcp-ddos-armor/rules/1000].
   ```

3. **Add a Rate Limiting Rule:**
   ```bash
   gcloud compute security-policies rules create 2000 \
     --security-policy=$ARMOR_POLICY \
     --description="Rate limiting rule" \
     --src-ip-ranges="*" \
     --rate-limit-options="rate-limit-threshold=10,rate-limit-interval-sec=60,enforce-on-key=IP" \
     --action="rate-based-ban" \
     --ban-duration-sec=300
   ```

   **Expected Output:**
   ```
   Created [https://www.googleapis.com/compute/v1/projects/PROJECT_ID/global/securityPolicies/iatd-labs-gcp-ddos-armor/rules/2000].
   ```

### Part 4: Implementing Google Cloud Load Balancing

1. **Create a Health Check:**
   ```bash
   gcloud compute health-checks create http http-basic-check \
     --port=80 \
     --request-path=/
   ```

   **Expected Output:**
   ```
   Created [https://www.googleapis.com/compute/v1/projects/PROJECT_ID/global/healthChecks/http-basic-check].
   ```

2. **Create a Backend Service:**
   ```bash
   # Create an instance group
   gcloud compute instance-groups unmanaged create ig-$VM_NAME \
     --zone=$ZONE
   
   # Add the VM to the instance group
   gcloud compute instance-groups unmanaged add-instances ig-$VM_NAME \
     --instances=$VM_NAME \
     --zone=$ZONE
   
   # Create a backend service
   gcloud compute backend-services create $LB_NAME-backend \
     --protocol=HTTP \
     --port-name=http \
     --health-checks=http-basic-check \
     --global \
     --security-policy=$ARMOR_POLICY
   
   # Add the instance group as a backend
   gcloud compute backend-services add-backend $LB_NAME-backend \
     --instance-group=ig-$VM_NAME \
     --instance-group-zone=$ZONE \
     --global
   ```

   **Expected Output:**
   ```
   Created [https://www.googleapis.com/compute/v1/projects/PROJECT_ID/zones/us-central1-a/instanceGroups/ig-iatd-labs-gcp-ddos-vm].
   Instance group ig-iatd-labs-gcp-ddos-vm created.
   
   Updated [https://www.googleapis.com/compute/v1/projects/PROJECT_ID/zones/us-central1-a/instanceGroups/ig-iatd-labs-gcp-ddos-vm].
   
   Created [https://www.googleapis.com/compute/v1/projects/PROJECT_ID/global/backendServices/iatd-labs-gcp-ddos-lb-backend].
   
   Updated [https://www.googleapis.com/compute/v1/projects/PROJECT_ID/global/backendServices/iatd-labs-gcp-ddos-lb-backend].
   ```

3. **Create a URL Map:**
   ```bash
   gcloud compute url-maps create $LB_NAME-url-map \
     --default-service=$LB_NAME-backend
   ```

   **Expected Output:**
   ```
   Created [https://www.googleapis.com/compute/v1/projects/PROJECT_ID/global/urlMaps/iatd-labs-gcp-ddos-lb-url-map].
   ```

4. **Create a HTTP(S) Proxy:**
   ```bash
   gcloud compute target-http-proxies create $LB_NAME-http-proxy \
     --url-map=$LB_NAME-url-map
   ```

   **Expected Output:**
   ```
   Created [https://www.googleapis.com/compute/v1/projects/PROJECT_ID/global/targetHttpProxies/iatd-labs-gcp-ddos-lb-http-proxy].
   ```

5. **Create a Global Forwarding Rule:**
   ```bash
   gcloud compute forwarding-rules create $LB_NAME-http-rule \
     --global \
     --target-http-proxy=$LB_NAME-http-proxy \
     --ports=80
   ```

   **Expected Output:**
   ```
   Created [https://www.googleapis.com/compute/v1/projects/PROJECT_ID/global/forwardingRules/iatd-labs-gcp-ddos-lb-http-rule].
   ```

6. **Get the Load Balancer IP Address:**
   ```bash
   LB_IP=$(gcloud compute forwarding-rules describe $LB_NAME-http-rule \
     --global \
     --format="get(IPAddress)")
   
   echo "Load Balancer IP: $LB_IP"
   ```

   **Expected Output:**
   ```
   Load Balancer IP: 34.123.456.789
   ```

### Part 5: Testing DDoS Mitigation

1. **Test Access to the Web Server through the Load Balancer:**
   ```bash
   curl http://$LB_IP
   ```

   **Expected Output:**
   ```html
   <html><body><h1>Hello from GCP VM</h1></body></html>
   ```

2. **Simulate Legitimate Traffic:**
   ```bash
   # Generate legitimate traffic to your Load Balancer
   for i in {1..10}; do
     curl -s http://$LB_IP > /dev/null
     echo "Request $i sent"
     sleep 1
   done
   ```

   **Expected Output:**
   ```
   Request 1 sent
   Request 2 sent
   ...
   Request 10 sent
   ```

3. **View Cloud Armor Logs:**
   ```bash
   # View Cloud Armor logs in Cloud Logging
   gcloud logging read "resource.type=http_load_balancer AND jsonPayload.enforcedSecurityPolicy.name=$ARMOR_POLICY" \
     --limit=10 \
     --format="table(timestamp,jsonPayload.enforcedSecurityPolicy.outcome)"
   ```

   **Expected Output:**
   ```
   TIMESTAMP                  JSONPAYLOAD_ENFORCEDSECURITYPOLICY_OUTCOME
   2023-04-01T12:34:56.789Z  ACCEPT
   2023-04-01T12:34:55.789Z  ACCEPT
   ...
   ```

### Post-Lab: Cleanup

To avoid incurring unnecessary costs, it's essential to clean up the resources created during this lab.

1. **Delete the Load Balancer Resources:**
   ```bash
   # Delete forwarding rule
   gcloud compute forwarding-rules delete $LB_NAME-http-rule \
     --global \
     --quiet
   
   # Delete HTTP proxy
   gcloud compute target-http-proxies delete $LB_NAME-http-proxy \
     --quiet
   
   # Delete URL map
   gcloud compute url-maps delete $LB_NAME-url-map \
     --quiet
   
   # Delete backend service
   gcloud compute backend-services delete $LB_NAME-backend \
     --global \
     --quiet
   
   # Delete instance group
   gcloud compute instance-groups unmanaged delete ig-$VM_NAME \
     --zone=$ZONE \
     --quiet
   
   # Delete health check
   gcloud compute health-checks delete http-basic-check \
     --quiet
   ```

   **Expected Output:**
   No output is expected if the deletion is successful.

2. **Delete Cloud Armor Security Policy:**
   ```bash
   gcloud compute security-policies delete $ARMOR_POLICY \
     --quiet
   ```

   **Expected Output:**
   No output is expected if the deletion is successful.

3. **Delete Compute Engine Instance:**
   ```bash
   gcloud compute instances delete $VM_NAME \
     --zone=$ZONE \
     --quiet
   ```

   **Expected Output:**
   No output is expected if the deletion is successful.

4. **Delete Firewall Rule:**
   ```bash
   gcloud compute firewall-rules delete $FW_NAME \
     --quiet
   ```

   **Expected Output:**
   No output is expected if the deletion is successful.

5. **Delete VPC Network and Subnet:**
   ```bash
   # Delete subnet
   gcloud compute networks subnets delete $SUBNET_NAME \
     --region=$REGION \
     --quiet
   
   # Delete VPC
   gcloud compute networks delete $VPC_NAME \
     --quiet
   ```

   **Expected Output:**
   No output is expected if the deletion is successful.

6. **Verify All Resources Are Deleted:**
   ```bash
   # Check for VM instances
   gcloud compute instances list \
     --filter="name=$VM_NAME"
   
   # Check for VPC networks
   gcloud compute networks list \
     --filter="name=$VPC_NAME"
   ```

   **Expected Output:**
   No output indicates that all resources have been successfully deleted.

**Congratulations!** You've successfully implemented DDoS mitigation techniques in Google Cloud Platform using Cloud Armor and Cloud Load Balancing. This lab has demonstrated key concepts in GCP DDoS protection that parallel the DDoS Protection implementations covered in the Azure labs.

### Troubleshooting Guide

1. **Cloud Armor Issues:**
   - Verify that the security policy is correctly associated with the backend service
   - Check that the security policy rules are properly configured
   - Review the Cloud Armor logs in Cloud Logging for any errors

2. **Load Balancer Issues:**
   - Ensure that the backend VM is healthy and accessible
   - Verify that the health check is passing
   - Check the load balancer logs for any errors

3. **VM Instance Issues:**
   - Verify that the firewall rule allows HTTP traffic
   - Check that the web server is running (`systemctl status apache2`)
   - Review the system logs for any errors (`/var/log/syslog`)

4. **Common Mistakes:**
   - Forgetting to enable HTTP traffic in the firewall rule
   - Not waiting for the load balancer to be fully provisioned before testing
   - Incorrectly configuring the backend service
   - Not associating the security policy with the backend service
