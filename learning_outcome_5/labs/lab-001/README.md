## IATD Microcredential Cloud Networking: Lab 1 - Load Balancing Fundamentals and Azure Load Balancer Basics

**Objective:** This lab introduces you to the fundamental concepts of load balancing and the Azure Load Balancer service. You'll learn the core principles, distribution algorithms, health probes, session persistence, and different load balancer types, laying the foundation for more advanced load balancing scenarios. 

**Estimated Time:** 60 - 75 minutes

**Prerequisites:**

1.  **Azure Subscription:** Requires an active Azure subscription. A free account is available at [https://azure.microsoft.com/free/](https://azure.microsoft.com/free/).
2.  **Basic knowledge of Azure Networking:** Familiarity with Virtual Networks and Subnets.

### Lab Conventions

*   **Azure Cloud Shell:** All PowerShell and CLI interactions will occur within the Azure Cloud Shell environment.
*   **Azure Portal:** The Azure Portal will be used for visualization and configuration tasks.
*   **Naming Conventions:** Resource names follow the convention `iatd_labs_01_*`.
*   **Location:** Choose a consistent Azure region.

#### Resource Naming

*   **Resource Group:** `iatd_labs_01_rg`
*   **Virtual Network:** `iatd_labs_01_vnet`
*   **Subnet-Backend:** `iatd_labs_01_subnet_backend`
*   **Load Balancer:** `iatd_labs_01_lb`
*   **Public IP:** `iatd_labs_01_lb_pip`
*   **Backend VM 1:** `iatd_labs_01_vm_backend1`
*   **Backend VM 2:** `iatd_labs_01_vm_backend2`

### Network Topology

#### Azure Load Balancer Architecture

```
+------------------------------------------+
|                                          |
|             Internet                     |
|                                          |
+------------------+---------------------+
                   |
                   | Public IP
                   | (iatd_labs_01_lb_pip)
                   |
+------------------v---------------------+
|                                          |
|          Azure Load Balancer            |
|          (iatd_labs_01_lb)              |
|                                          |
+------------------+---------------------+
                   |
                   | Health Probe (Port 80)
                   | Load Balancing Rule (Port 80)
                   |
+------------------v---------------------+
|                                          |
|           Backend Pool                   |
|                                          |
+--------+--------------------------+------+
         |                          |
         |                          |
+--------v----------+    +---------v---------+
|                   |    |                   |
| Backend VM 1      |    | Backend VM 2      |
| (No Public IP)    |    | (No Public IP)    |
|                   |    |                   |
+-------------------+    +-------------------+

```

#### Network Configuration

- **Virtual Network:** `iatd_labs_01_vnet` (172.16.0.0/16)
- **Subnet:** `iatd_labs_01_subnet_backend` (172.16.1.0/24)
- **Load Balancer Type:** Standard SKU
- **Distribution Algorithm:** Round Robin (default)

#### Traffic Flow

1. Client requests reach the Azure Load Balancer through its public IP address
2. Load Balancer distributes traffic to healthy backend VMs based on the configured algorithm
3. Health probes continuously check the health of backend VMs
4. If a backend VM fails health checks, it's removed from the rotation

#### Testing Scenario

```
+-------------+     +----------------+     +-------------+
|             |     |                |     |             |
|   Client    +---->+ Load Balancer  +---->+  Backend 1  |
|             |     |                |     |             |
+-------------+     |                |     +-------------+
                    |                |
                    |                |     +-------------+
                    |                |     |             |
                    |                +---->+  Backend 2  |
                    |                |     |             |
                    +----------------+     +-------------+
```

When a client makes multiple requests to the load balancer, traffic is distributed between Backend 1 and Backend 2 in a round-robin fashion.

### Part 1: Understanding Load Balancing Concepts (Conceptual)

1.  **Load Balancing Concepts:**
    *   **Definition:** Distributing network traffic across multiple servers to ensure high availability, scalability, and performance.
    *   **Benefits:** Increased reliability, improved response times, efficient resource utilization, and fault tolerance.

2.  **Distribution Algorithms:**
    *   **Round Robin:** Distributes traffic sequentially to each server in the backend pool.
    *   **Hash-based:** traffic is always directed to the same backend for a given client session, useful for maintaining session persistence.
    *   **Least connections:** Directs traffic to the server with the fewest active connections.
    *   **Explain the advantages and disadvantages of each algorithm.**

3.  **Health Probes:**
    *   **Definition:** Automated checks to monitor the health and availability of backend servers.
    *   **Types:** HTTP, TCP, HTTPS.
    *   **Importance:** Ensure that traffic is only directed to healthy servers.
    * The test and validations will be performed on the Lab

4.  **Session Persistence:**
    *   **Definition:** Maintaining a user's session with a specific backend server.
    *   **Use Cases:** Applications that require stateful sessions (e.g., e-commerce websites).

5.  **Load Balancer Types:**
    *   **Layer 4 Load Balancer:** Operates at the transport layer (TCP/UDP). Distributes traffic based on IP addresses and ports. Example: Azure Load Balancer (basic).
    *   **Layer 7 Load Balancer:** Operates at the application layer (HTTP/HTTPS). Distributes traffic based on URL, headers, and other application-specific information. Example: Azure Application Gateway, Azure Front Door.
 *Explain the benefits and types of load balancing, this is a only a conceptual lab. This will be explained during the configuration*

### Part 2: Deploying the Azure Load Balancer and Backend VMs

1.  **Create a Resource Group:**

    ```bash
    RESOURCE_GROUP="iatd_labs_01_rg"
    LOCATION="eastus" # Your Region

    az group create --name $RESOURCE_GROUP --location $LOCATION
    ```

2.  **Create a Virtual Network and Subnet:**

    ```bash
    VNET_NAME="iatd_labs_01_vnet"
    SUBNET_BACKEND_NAME="iatd_labs_01_subnet_backend"
    ADDRESS_PREFIX="172.16.0.0/16"
    SUBNET_BACKEND_PREFIX="172.16.1.0/24"

    az network vnet create \
        --resource-group $RESOURCE_GROUP \
        --name $VNET_NAME \
        --address-prefixes $ADDRESS_PREFIX \
        --subnet-name $SUBNET_BACKEND_NAME \
        --subnet-prefixes $SUBNET_BACKEND_PREFIX \
        --location $LOCATION
    ```

3.  **Create a Public IP Address for the Load Balancer:**

    ```bash
    PUBLIC_IP_NAME="iatd_labs_01_lb_pip"

    az network public-ip create \
        --resource-group $RESOURCE_GROUP \
        --name $PUBLIC_IP_NAME \
        --allocation-method Static \
        --sku Standard \
        --location $LOCATION
    ```

4.  **Create Azure Load Balancer (Portal):**

    *   Search for "Load balancers" and select.
    *   Click **Create**.
        *   Subscription: Your subscription.
        *   Resource group: `iatd_labs_01_rg`
        *   Name: `iatd_labs_01_lb`
        *   Region: Your region.
        *   Tier: Standard
        *   Type: Public
        *   SKU: Standard
    * Click **Review + Create** and then **Create**

### Part 3: Configuring the Load Balancer

1.  **Create Backend Pool (Portal):**

    *   Navigate to your Load Balancer (`iatd_labs_01_lb`) in the Azure Portal.
    *   Select **Backend pools** under **Settings**.
    *   Click **Add**.
        *   Name: `backendPool`
        *   Virtual network: Select `iatd_labs_01_vnet`
        *   Backend pool type: IP Address
    *Click "Save"

2.  **Create Health Probe (Portal):**

    *   Select **Health probes** under **Settings**.
    *   Click **Add**.
        *   Name: `httpProbe`
        *   Protocol: `HTTP`
        *   Port: `80`
        *   Path: `/`
        *   Interval: `15`
        *   Unhealthy threshold: `2`
    *Click "Save"

3.  **Create Load Balancing Rule (Portal):**

    *   Select **Load balancing rules** under **Settings**.
    *   Click **Add**.
        *   Name: `httpRule`
        *   Frontend IP address: Select `iatd_labs_01_lb_pip`
        *   Protocol: `TCP`
        *   Port: `80`
        *   Backend pool: Select `backendPool`
        *   Health probe: Select `httpProbe`
        *   Session persistence: `None`
        *   Idle timeout (minutes): `4`
        *   Enable TCP reset: `Checked`
    *Click "Save"

### Part 4: Deploying Backend VMs

1.  **Create Backend Virtual Machine 1 (`iatd_labs_01_vm_backend1`) (Portal):**
    *   Search for "Virtual machines" and select.
    *   Click **Create** -> **Azure virtual machine**.
        *   **Basics Tab:**
            *   Subscription: Your subscription.
            *   Resource group: `iatd_labs_01_rg`
            *   Virtual machine name: `iatd_labs_01_vm_backend1`
            *   Region: Your region.
            *   Image: `Ubuntu Server 22.04 LTS`
            *   Size: Choose a size (e.g., Standard_B1ls).
            *   Username: `azureuser`
            *   Authentication type: `Password`
            *   Password: Set a password.
        *   **Networking Tab:**
            *   Virtual network: `iatd_labs_01_vnet`
            *   Subnet: `iatd_labs_01_subnet_backend`
            *   Public IP: **None** (Important: No Public IP)
            *   NIC network security group: `None`
        *   **Management Tab:**
            *   Disable boot diagnostics

2.  **Create Backend Virtual Machine 2 (`iatd_labs_01_vm_backend2`) (Portal):**
    *   Repeat the process above, but with the following changes:
        *   Virtual machine name: `iatd_labs_01_vm_backend2`
        * **Networking Tab:**
            *   Virtual network: `iatd_labs_01_vnet`
            *   Subnet: `iatd_labs_01_subnet_backend`
            *   Public IP: **None** (Important: No Public IP)
            *   NIC network security group: `None`
        *   **Management Tab:**
            *   Disable boot diagnostics

### Part 5: Adding Backend VMs to the Load Balancer

1.  **Adding VMs to Backend Pool (Portal):**

    *   Navigate to your Load Balancer (`iatd_labs_01_lb`) in the Azure Portal.
    *   Select **Backend pools** under **Settings**.
    *   Select the `backendPool`.
    *   Under "Backend pool", click "+ Add".
        * Select Virtual Machines and the 2 new backend machines.

### Part 6: Testing Load Balancing

1.  **Install Web Server on Backend VMs:**

    * First, we need to access our VMs through a jumpbox or bastion host since they don't have public IPs.
    * Create a simple jumpbox VM with public IP in the same VNet:

    ```bash
    # Create a jumpbox VM
    JUMPBOX_NAME="iatd_labs_01_vm_jumpbox"
    
    az vm create \
      --resource-group $RESOURCE_GROUP \
      --name $JUMPBOX_NAME \
      --image UbuntuLTS \
      --admin-username azureuser \
      --generate-ssh-keys \
      --vnet-name $VNET_NAME \
      --subnet $SUBNET_BACKEND_NAME \
      --size Standard_B1ls
    ```

    * SSH into the jumpbox:
    ```bash
    # Get the public IP of the jumpbox
    JUMPBOX_IP=$(az vm show -d -g $RESOURCE_GROUP -n $JUMPBOX_NAME --query publicIps -o tsv)
    
    # SSH into the jumpbox
    ssh azureuser@$JUMPBOX_IP
    ```

    * From the jumpbox, install Apache on both backend VMs:
    ```bash
    # SSH into backend VM 1 and install Apache
    ssh azureuser@iatd_labs_01_vm_backend1
    sudo apt update
    sudo apt install -y apache2
    echo "<h1>Welcome to Backend Server 1</h1>" | sudo tee /var/www/html/index.html
    exit
    
    # SSH into backend VM 2 and install Apache
    ssh azureuser@iatd_labs_01_vm_backend2
    sudo apt update
    sudo apt install -y apache2
    echo "<h1>Welcome to Backend Server 2</h1>" | sudo tee /var/www/html/index.html
    exit
    ```

2.  **Accessing the Application:**

    * Get the Public IP address of the Load Balancer:
    ```bash
    LB_PUBLIC_IP=$(az network public-ip show \
      --resource-group $RESOURCE_GROUP \
      --name $PUBLIC_IP_NAME \
      --query ipAddress \
      --output tsv)
    
    echo $LB_PUBLIC_IP
    ```

    * Expected output: An IP address like `52.x.x.x`
    
    * Open a web browser and navigate to the Load Balancer's Public IP address.
    * You should see either "Welcome to Backend Server 1" or "Welcome to Backend Server 2".

3.  **Verify Load Balancing:**

    * Refresh the browser multiple times. You should see the response alternating between Backend Server 1 and Backend Server 2.
    
    * You can also use curl to test this from the jumpbox:
    ```bash
    # Run this multiple times to see load balancing in action
    for i in {1..10}; do curl -s $LB_PUBLIC_IP | grep Welcome; done
    ```

    * Expected output: A mix of responses from both backend servers:
    ```
    <h1>Welcome to Backend Server 1</h1>
    <h1>Welcome to Backend Server 2</h1>
    <h1>Welcome to Backend Server 1</h1>
    <h1>Welcome to Backend Server 2</h1>
    ...
    ```
    
    * This demonstrates that the Azure Load Balancer is distributing traffic between the two backend VMs using the default round-robin algorithm.

4.  **Test Health Probe:**

    * You can simulate a server failure by stopping the Apache service on one of the backend VMs:
    ```bash
    # SSH into backend VM 1
    ssh azureuser@iatd_labs_01_vm_backend1
    
    # Stop Apache service
    sudo systemctl stop apache2
    exit
    ```
    
    * Wait about 15-30 seconds for the health probe to detect the failure
    
    * Now test the load balancer again:
    ```bash
    # Run this multiple times
    for i in {1..5}; do curl -s $LB_PUBLIC_IP | grep Welcome; done
    ```
    
    * Expected output: All responses should now come from Backend Server 2:
    ```
    <h1>Welcome to Backend Server 2</h1>
    <h1>Welcome to Backend Server 2</h1>
    <h1>Welcome to Backend Server 2</h1>
    <h1>Welcome to Backend Server 2</h1>
    <h1>Welcome to Backend Server 2</h1>
    ```
    
    * This demonstrates that the health probe has detected the failure and the load balancer is no longer sending traffic to the unhealthy backend VM.

5.  **Restore Service:**

    * Restart the Apache service on backend VM 1:
    ```bash
    # SSH into backend VM 1
    ssh azureuser@iatd_labs_01_vm_backend1
    
    # Start Apache service
    sudo systemctl start apache2
    exit
    ```
    
    * Wait about 15-30 seconds for the health probe to detect the recovery
    
    * Test the load balancer again to verify both servers are receiving traffic

### Post-Lab: Cleanup

To avoid incurring unnecessary costs, it's essential to clean up the resources created during this lab.

1.  **Delete Resource Group:**

    ```bash
    az group delete --name $RESOURCE_GROUP --yes
    ```

**Learning Outcomes:**

*   Understood fundamental load balancing concepts, including distribution algorithms, health probes, and session persistence.
*   Successfully deployed an Azure Load Balancer with a backend pool and health probe.
*   Successfully deployed backend virtual machines and added them to the load balancer's backend pool.
*   Learned how to test load balancing to ensure that traffic is distributed across the backend servers.
