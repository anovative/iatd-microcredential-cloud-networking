
## IATD Microcredential Cloud Networking: Lab 2 - Private Communication within an Azure Virtual Network

**Objective:** This lab demonstrates how resources within an Azure Virtual Network (VNet) can communicate privately with each other, isolated from the public internet. You'll create a VNet with two subnets (one for a web app and one for a database) and verify the private communication between them.

**Estimated Time:** 60 - 75 minutes

**Prerequisites:**

1.  **Azure Subscription:** Requires an active Azure subscription. A free account is available at [https://azure.microsoft.com/free/](https://azure.microsoft.com/free/).
2.  **Basic knowledge of Azure Networking:** Familiarity with Virtual Networks and Subnets.

## Let's get started!

### Lab Conventions

*   **Azure Cloud Shell:** All PowerShell and CLI interactions will occur within the Azure Cloud Shell environment.
*   **Azure Portal:** The Azure Portal will be used for visualization and configuration tasks.
*   **Naming Conventions:** Resource names follow the convention `iatd_labs_02_*`.
*   **IP Address Range:** The `172.16.0.0/16` range will be used (as per standardization).
*   **Location:** Choose a consistent Azure region.

#### Resource Naming

*   **Resource Group:** `iatd_labs_02_rg`
*   **Virtual Network:** `iatd_labs_02_vnet`
*   **Subnet-Web:** `iatd_labs_02_subnet_web`
*   **Subnet-DB:** `iatd_labs_02_subnet_db`
*   **VM-Web-01:** `iatd_labs_02_vm_web_01`
*   **VM-DB-01:** `iatd_labs_02_vm_db_01`

### Part 1: Setting up the Virtual Network

1.  **Create a Resource Group:**

    ```bash
    RESOURCE_GROUP="iatd_labs_02_rg"
    LOCATION="eastus" # Your Region

    az group create --name $RESOURCE_GROUP --location $LOCATION
    ```

    **Expected Output:**
    ```json
    {
      "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_02_rg",
      "location": "eastus",
      "managedBy": null,
      "name": "iatd_labs_02_rg",
      "properties": {
        "provisioningState": "Succeeded"
      },
      "tags": null,
      "type": "Microsoft.Resources/resourceGroups"
    }
    ```

2.  **Create a Virtual Network with Subnets:**

    ```bash
    VNET_NAME="iatd_labs_02_vnet"
    SUBNET_WEB_NAME="iatd_labs_02_subnet_web"
    SUBNET_DB_NAME="iatd_labs_02_subnet_db"
    ADDRESS_PREFIX="172.16.0.0/16"
    SUBNET_WEB_PREFIX="172.16.0.0/24"
    SUBNET_DB_PREFIX="172.16.1.0/24"

    az network vnet create \
        --resource-group $RESOURCE_GROUP \
        --name $VNET_NAME \
        --address-prefixes $ADDRESS_PREFIX \
        --subnet-name $SUBNET_WEB_NAME \
        --subnet-prefixes $SUBNET_WEB_PREFIX \
        --location $LOCATION
    ```

    **Expected Output for VNet creation:**
    ```json
    {
      "newVNet": {
        "addressSpace": {
          "addressPrefixes": [
            "172.16.0.0/16"
          ]
        },
        "dhcpOptions": {
          "dnsServers": []
        },
        "enableDdosProtection": false,
        "enableVmProtection": false,
        "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_02_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_02_vnet",
        "location": "eastus",
        "name": "iatd_labs_02_vnet",
        "provisioningState": "Succeeded",
        "resourceGuid": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "subnets": [
          {
            "addressPrefix": "172.16.0.0/24",
            "delegations": [],
            "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_02_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_02_vnet/subnets/iatd_labs_02_subnet_web",
            "name": "iatd_labs_02_subnet_web",
            "networkSecurityGroup": null,
            "privateEndpointNetworkPolicies": "Disabled",
            "privateLinkServiceNetworkPolicies": "Enabled",
            "provisioningState": "Succeeded",
            "resourceNavigationLinks": [],
            "routeTable": null,
            "serviceEndpointPolicies": [],
            "serviceEndpoints": []
          }
        ],
        "type": "Microsoft.Network/virtualNetworks"
      }
    }
    ```

    ```bash
    az network vnet subnet create \
        --resource-group $RESOURCE_GROUP \
        --vnet-name $VNET_NAME \
        --name $SUBNET_DB_NAME \
        --address-prefix $SUBNET_DB_PREFIX
    ```

    **Expected Output for DB Subnet creation:**
    ```json
    {
      "addressPrefix": "172.16.1.0/24",
      "delegations": [],
      "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_02_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_02_vnet/subnets/iatd_labs_02_subnet_db",
      "name": "iatd_labs_02_subnet_db",
      "networkSecurityGroup": null,
      "privateEndpointNetworkPolicies": "Disabled",
      "privateLinkServiceNetworkPolicies": "Enabled",
      "provisioningState": "Succeeded",
      "resourceNavigationLinks": [],
      "routeTable": null,
      "serviceAssociationLinks": [],
      "serviceEndpointPolicies": [],
      "serviceEndpoints": []
    }
    ```

### Part 2: Creating Virtual Machines in Subnets

1.  **Create Web Virtual Machine (`iatd_labs_02_vm_web_01`) (Portal):**
    *   Search for "Virtual machines" and select.
    *   Click **Create** -> **Azure virtual machine**.
        *   **Basics Tab:**
            *   Subscription: Your subscription.
            *   Resource group: `iatd_labs_02_rg`
            *   Virtual machine name: `iatd_labs_02_vm_web_01`
            *   Region: Your region.
            *   Image: `Ubuntu Server 22.04 LTS`
            *   Size: Choose a size (e.g., Standard_B1ls).
            *   Username: `azureuser`
            *   Authentication type: `Password`
            *   Password: Set a password.
        *   **Networking Tab:**
            *   Virtual network: `iatd_labs_02_vnet`
            *   Subnet: `iatd_labs_02_subnet_web`
            *   Public IP: **None** (Important: No Public IP)
            *   NIC network security group: `None`
        *   **Management Tab:**
            *   Disable boot diagnostics
    *   Click **Review + create**, then **Create**.

2.  **Create DB Virtual Machine (`iatd_labs_02_vm_db_01`) (Portal):**
    *   Repeat the process above, but with the following changes:
        *   Virtual machine name: `iatd_labs_02_vm_db_01`
        *   Subnet: `iatd_labs_02_subnet_db`
        *   Public IP: **None** (Important: No Public IP)
        *   NIC network security group: `None`
    *   **Management Tab:**
        *   Disable boot diagnostics

### Part 3: Testing Private Communication

1.  **Connect to Web VM:**
    *   Since the Web VM has no Public IP, you'll need a "jump box" VM with a public IP in the same VNet or use Azure Bastion. For simplicity, we'll assume you have a jump box VM.
    *   SSH into the jump box VM.
    *   From the jump box, SSH into the `iatd_labs_02_vm_web_01` VM using its private IP address (you can find this in the Azure Portal).

2.  **Test Connectivity from Web VM to DB VM:**
    *   From the `iatd_labs_02_vm_web_01` VM, ping the private IP address of the `iatd_labs_02_vm_db_01` VM.

        ```bash
        ping <private_ip_of_db_vm>
        ```

    *   The ping *should* be successful, demonstrating private communication within the VNet.

3.  **(Optional) Install `netcat` and test port connectivity:**
    *   Install `netcat` on both VMs: `sudo apt install netcat -y`
    *   On the DB VM, listen on a port (e.g., 5432): `nc -l -p 5432`
    *   On the Web VM, connect to the DB VM on that port: `nc <private_ip_of_db_vm> 5432`
    *   Type some text on either VM and see if it appears on the other.

### Part 4: Verifying Isolation from the Internet

1.  **Attempt to ping a public IP from the Web VM:**
    *   From the `iatd_labs_02_vm_web_01` VM, try to ping a public IP address (e.g., 8.8.8.8).

        ```bash
        ping 8.8.8.8
        ```

    *   The ping *should* fail, demonstrating that the VM has no direct internet access.

### Post-Lab: Cleanup

To avoid incurring unnecessary costs, it's essential to clean up the resources created during this lab.

1.  **Delete Resource Group:**

    ```bash
    az group delete --name $RESOURCE_GROUP --yes
    ```

    **Expected Output:**
    ```
    The resource group deletion is in progress and will complete in the background.
    ```

2.  **Verify Resource Deletion:**

    ```bash
    az group show --name $RESOURCE_GROUP
    ```

    **Expected Output:**
    ```
    Resource group 'iatd_labs_02_rg' could not be found.
    ```

3.  **Clean Local Environment Variables:**

    ```bash
    unset RESOURCE_GROUP LOCATION VNET_NAME SUBNET_WEB_NAME SUBNET_DB_NAME ADDRESS_PREFIX SUBNET_WEB_PREFIX SUBNET_DB_PREFIX
    ```

**Learning Outcomes:**

*   Successfully created an Azure Virtual Network with two subnets.
*   Successfully deployed virtual machines into the subnets without public IP addresses.
*   Successfully demonstrated private communication between VMs within the same VNet.
*   Successfully verified that VMs without public IP addresses are isolated from the internet.
*   Understand the concept of private networking within Azure VNets.
