## IATD Microcredential Cloud Networking: Lab 2 - Configuring and Managing Azure Load Balancer

**Objective:** This lab focuses on the practical aspects of configuring the Azure Load Balancer service. You'll learn how to create and manage a basic load balancer, add backend pools, configure health probes, set up load balancing rules, and test traffic distribution within a single region.

**Estimated Time:** 75 - 90 minutes

**Prerequisites:**

1.  **Azure Subscription:** Requires an active Azure subscription. A free account is available at [https://azure.microsoft.com/free/](https://azure.microsoft.com/).
2.  **Basic knowledge of Azure Networking:** Familiarity with Virtual Networks and Subnets.
3.  **Completion of Lab 1:** Assumes knowledge of load balancing concepts from Lab 1.

### Lab Conventions

*   **Azure Cloud Shell:** All PowerShell and CLI interactions will occur within the Azure Cloud Shell environment.
*   **Azure Portal:** The Azure Portal will be used for visualization and configuration tasks.
*   **Naming Conventions:** Resource names follow the convention `iatd_labs_02_*`.
*   **Location:** Choose a consistent Azure region.

#### Resource Naming

*   **Resource Group:** `iatd_labs_02_rg`
*   **Virtual Network:** `iatd_labs_02_vnet`
*   **Subnet-Backend:** `iatd_labs_02_subnet_backend`
*   **Load Balancer:** `iatd_labs_02_lb`
*   **Public IP:** `iatd_labs_02_lb_pip`
*   **Backend VM 1:** `iatd_labs_02_vm_backend1`
*   **Backend VM 2:** `iatd_labs_02_vm_backend2`
*   **Network Security Group:** `iatd_labs_02_nsg`

### Part 1: Setting up the Virtual Network and NSG

1.  **Create a Resource Group:**

    ```bash
    RESOURCE_GROUP="iatd_labs_02_rg"
    LOCATION="eastus" # Your Region

    az group create --name $RESOURCE_GROUP --location $LOCATION
    ```

2.  **Create a Virtual Network and Subnet:**

    ```bash
    VNET_NAME="iatd_labs_02_vnet"
    SUBNET_BACKEND_NAME="iatd_labs_02_subnet_backend"
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

3.  **Create a Network Security Group (NSG):**

    ```bash
    NSG_NAME="iatd_labs_02_nsg"

    az network nsg create \
        --resource-group $RESOURCE_GROUP \
        --name $NSG_NAME \
        --location $LOCATION
    ```

4.  **Create Inbound NSG Rule to Allow HTTP Traffic:**

    ```bash
    NSG_NAME="iatd_labs_02_nsg"

    az network nsg rule create \
        --resource-group $RESOURCE_GROUP \
        --nsg-name $NSG_NAME \
        --name AllowHTTP \
        --priority 100 \
        --source-address-prefixes Internet \
        --destination-port-ranges 80 \
        --access Allow \
        --protocol Tcp \
        --direction Inbound
    ```

5.  **Associate the NSG with the Backend Subnet:**

    ```bash
    az network vnet subnet update \
        --resource-group $RESOURCE_GROUP \
        --vnet-name $VNET_NAME \
        --name $SUBNET_BACKEND_NAME \
        --network-security-group $NSG_NAME
    ```

### Part 2: Deploying and Configuring the Azure Load Balancer

1.  **Create a Public IP Address for the Load Balancer:**

    ```bash
    PUBLIC_IP_NAME="iatd_labs_02_lb_pip"

    az network public-ip create \
        --resource-group $RESOURCE_GROUP \
        --name $PUBLIC_IP_NAME \
        --allocation-method Static \
        --sku Standard \
        --location $LOCATION
    ```

2.  **Create Azure Load Balancer (Portal):**

    *   Search for "Load balancers" and select.
    *   Click **Create**.
        *   Subscription: Your subscription.
        *   Resource group: `iatd_labs_02_rg`
        *   Name: `iatd_labs_02_lb`
        *   Region: Your region.
        *   Tier: Standard
        *   Type: Public
        *   SKU: Standard
    * Click **Review + Create** and then **Create**

### Part 3: Configuring the Load Balancer

1.  **Create Backend Pool (Portal):**

    *   Navigate to your Load Balancer (`iatd_labs_02_lb`) in the Azure Portal.
    *   Select **Backend pools** under **Settings**.
    *   Click **Add**.
        *   Name: `backendPool`
        *   Virtual network: Select `iatd_labs_02_vnet`
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
        *   Frontend IP address: Select `iatd_labs_02_lb_pip`
        *   Protocol: `TCP`
        *   Port: `80`
        *   Backend pool: Select `backendPool`
        *   Health probe: Select `httpProbe`
        *   Session persistence: `None`
        *   Idle timeout (minutes): `4`
        *   Enable TCP reset: `Checked`
    *Click "Save"

### Part 4: Deploying Backend VMs

1.  **Create Backend Virtual Machine 1 (`iatd_labs_02_vm_backend1`) (Portal):**
    *   Search for "Virtual machines" and select.
    *   Click **Create** -> **Azure virtual machine**.
        *   **Basics Tab:**
            *   Subscription: Your subscription.
            *   Resource group: `iatd_labs_02_rg`
            *   Virtual machine name: `iatd_labs_02_vm_backend1`
            *   Region: Your region.
            *   Image: `Ubuntu Server 22.04 LTS`
            *   Size: Choose a size (e.g., Standard_B1ls).
            *   Username: `azureuser`
            *   Authentication type: `Password`
            *   Password: Set a password.
        *   **Networking Tab:**
            *   Virtual network: `iatd_labs_02_vnet`
            *   Subnet: `iatd_labs_02_subnet_backend`
            *   Public IP: **None** (Important: No Public IP)
            *   NIC network security group: `None`
        *   **Management Tab:**
            *   Disable boot diagnostics

2.  **Create Backend Virtual Machine 2 (`iatd_labs_02_vm_backend2`) (Portal):**
    *   Repeat the process above, but with the following changes:
        *   Virtual machine name: `iatd_labs_02_vm_backend2`
        * **Networking Tab:**
            *   Virtual network: `iatd_labs_02_vnet`
            *   Subnet: `iatd_labs_02_subnet_backend`
            *   Public IP: **None** (Important: No Public IP)
            *   NIC network security group: `None`
        *   **Management Tab:**
            *   Disable boot diagnostics

### Part 5: Adding Backend VMs to the Load Balancer

1.  **Adding VMs to Backend Pool (Portal):**

    *   Navigate to your Load Balancer (`iatd_labs_02_lb`) in the Azure Portal.
    *   Select **Backend pools** under **Settings**.
    *   Select the `backendPool`.
    *   Under "Backend pool", click "+ Add".
        * Select Virtual Machines and the 2 new backend machines.

### Part 6: Configuring Web Servers and Testing Load Balancing

1.  **Install Nginx on Both Backend VMs:**

    *   Connect to each backend VM via SSH (You will need a jump box VM for this)
    *   Run the following commands:

        ```bash
        sudo apt update
        sudo apt install nginx -y
        sudo systemctl start nginx
        sudo systemctl enable nginx
        ```

2.  **Customize the Nginx Default Page (Optional):**

    *   Edit the `/var/www/html/index.nginx-debian.html` file on each VM to display a unique identifier (e.g., "Backend VM 1" and "Backend VM 2") to easily identify which server is responding.

3.  **Accessing the Application:**

    *   Get the Public IP address of the Load Balancer (`iatd_labs_02_lb_pip`).
    *   Open a web browser and navigate to the Load Balancer's Public IP address.

4.  **Verify Load Balancing:**

    *   Refresh the web browser multiple times. You should see the content alternating between "Backend VM 1" and "Backend VM 2", demonstrating that the load balancer is distributing traffic across the backend servers.

### Post-Lab: Cleanup

To avoid incurring unnecessary costs, it's essential to clean up the resources created during this lab.

1.  **Delete Resource Group:**

    ```bash
    az group delete --name $RESOURCE_GROUP --yes
    ```

**Learning Outcomes:**

*   Successfully created and configured an Azure Load Balancer.
*   Successfully created a backend pool and added backend virtual machines to it.
*   Successfully configured a health probe to monitor the availability of backend servers.
*   Successfully created a load balancing rule to distribute traffic across the backend pool.
*   Successfully verified that the load balancer is distributing traffic across the backend servers as expected.
*   Understood how to configure basic load balancing within a single Azure region.
