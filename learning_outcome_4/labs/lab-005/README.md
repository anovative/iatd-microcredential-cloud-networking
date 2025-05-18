## IATD Microcredential Cloud Networking: Lab 5 - Controlling Network Traffic with User-Defined Routes (UDRs) and Azure Firewall

**Objective:** This lab demonstrates how to control network traffic within an Azure Virtual Network (VNet) using User-Defined Routes (UDRs) and Azure Firewall. You'll create a UDR to route all outbound traffic from the application subnet to an Azure Firewall, which will then be configured to allow only specific traffic to the database subnet, enhancing security and control.

**Estimated Time:** 90 - 120 minutes

**Prerequisites:**

1.  **Azure Subscription:** Requires an active Azure subscription. A free account is available at [https://azure.microsoft.com/free/](https://azure.microsoft.com/free/).
2.  **Basic knowledge of Azure Networking:** Familiarity with Virtual Networks, Subnets, Network Security Groups, and Virtual Machines.
3.  **Familiarity with Azure Firewall:** While not strictly required, some familiarity with Azure Firewall is helpful.

## Let's get started!

### Lab Conventions

*   **Azure Cloud Shell:** All PowerShell and CLI interactions will occur within the Azure Cloud Shell environment.
*   **Azure Portal:** The Azure Portal will be used for visualization and configuration tasks.
*   **Naming Conventions:** Resource names follow the convention `iatd_labs_05_*`.
*   **IP Address Range:** The `172.16.0.0/16` range will be used (as per standardization).
*   **Location:** Choose a consistent Azure region.

#### Resource Naming

*   **Resource Group:** `iatd_labs_05_rg`
*   **Virtual Network:** `iatd_labs_05_vnet`
*   **Subnet-App:** `iatd_labs_05_subnet_app`
*   **Subnet-DB:** `iatd_labs_05_subnet_db`
*   **Subnet-AzureFirewall:** `iatd_labs_05_subnet_fw`
*   **Azure Firewall:** `iatd_labs_05_azurefirewall`
*   **Public IP - AzureFirewall:** `iatd_labs_05_pip_fw`
*   **Route Table - AppSubnet:** `iatd_labs_05_routetable_app`
*   **VM-App-01:** `iatd_labs_05_vm_app_01`
*   **VM-DB-01:** `iatd_labs_05_vm_db_01`

### Part 1: Setting up the Virtual Network and Subnets

1.  **Create a Resource Group:**

    ```bash
    RESOURCE_GROUP="iatd_labs_05_rg"
    LOCATION="eastus" # Your Region

    az group create --name $RESOURCE_GROUP --location $LOCATION
    ```

    **Expected Output:**
    ```json
    {
      "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_05_rg",
      "location": "eastus",
      "managedBy": null,
      "name": "iatd_labs_05_rg",
      "properties": {
        "provisioningState": "Succeeded"
      },
      "tags": null,
      "type": "Microsoft.Resources/resourceGroups"
    }
    ```

2.  **Create a Virtual Network with Subnets:**

    ```bash
    VNET_NAME="iatd_labs_05_vnet"
    SUBNET_APP_NAME="iatd_labs_05_subnet_app"
    SUBNET_DB_NAME="iatd_labs_05_subnet_db"
    SUBNET_FW_NAME="AzureFirewallSubnet" # This name is REQUIRED for Azure Firewall
    ADDRESS_PREFIX="10.0.0.0/16"
    SUBNET_APP_PREFIX="10.0.1.0/24"
    SUBNET_DB_PREFIX="10.0.2.0/24"
    SUBNET_FW_PREFIX="10.0.0.0/24" #Important Azure Firewall subnet

    az network vnet create \
        --resource-group $RESOURCE_GROUP \
        --name $VNET_NAME \
        --address-prefixes $ADDRESS_PREFIX \
        --subnet-name $SUBNET_APP_NAME \
        --subnet-prefixes $SUBNET_APP_PREFIX \
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
        "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_05_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_05_vnet",
        "location": "eastus",
        "name": "iatd_labs_05_vnet",
        "provisioningState": "Succeeded",
        "resourceGuid": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "subnets": [
          {
            "addressPrefix": "172.16.1.0/24",
            "delegations": [],
            "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_05_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_05_vnet/subnets/iatd_labs_05_subnet_app",
            "name": "iatd_labs_05_subnet_app",
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

    az network vnet subnet create \
        --resource-group $RESOURCE_GROUP \
        --vnet-name $VNET_NAME \
        --name $SUBNET_DB_NAME \
        --address-prefix $SUBNET_DB_PREFIX
    ```

    **Expected Output for DB Subnet creation:**
    ```json
    {
      "addressPrefix": "172.16.2.0/24",
      "delegations": [],
      "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_05_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_05_vnet/subnets/iatd_labs_05_subnet_db",
      "name": "iatd_labs_05_subnet_db",
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

    az network vnet subnet create \
        --resource-group $RESOURCE_GROUP \
        --vnet-name $VNET_NAME \
        --name $SUBNET_FW_NAME \
        --address-prefix $SUBNET_FW_PREFIX
    ```

    **Expected Output for Firewall Subnet creation:**
    ```json
    {
      "addressPrefix": "172.16.0.0/24",
      "delegations": [],
      "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_05_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_05_vnet/subnets/AzureFirewallSubnet",
      "name": "AzureFirewallSubnet",
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

### Part 2: Deploying and Configuring Azure Firewall

1.  **Create a Public IP Address for Azure Firewall:**

    ```bash
    FW_PUBLIC_IP_NAME="iatd_labs_05_pip_fw"

    az network public-ip create \
        --resource-group $RESOURCE_GROUP \
        --name $FW_PUBLIC_IP_NAME \
        --allocation-method Static \
        --sku Standard \
        --location $LOCATION
    ```

2.  **Deploy Azure Firewall (Portal):**
    *   Search for "Firewalls" and select.
    *   Click **Create**.
        *   Subscription: Your subscription
        *   Resource group: `iatd_labs_05_rg`
        *   Name: `iatd_labs_05_azurefirewall`
        *   Region: Your Region.
        *   Tier: Standard
        *   Virtual network: Select `iatd_labs_05_vnet`
        *   Public IP address: Select `iatd_labs_05_pip_fw`
    * Click **Review + Create** and then **Create**

3.  **Get Azure Firewall Private IP Address:**

    *   After the deployment completes, navigate to the `iatd_labs_05_azurefirewall` in the Azure Portal.
    *   Note the Private IP address of the firewall. You'll need this for the UDR.

### Part 3: Configuring User-Defined Route (UDR)

1.  **Create a Route Table:**

    ```bash
    ROUTE_TABLE_NAME="iatd_labs_05_routetable_app"

    az network route-table create \
        --resource-group $RESOURCE_GROUP \
        --name $ROUTE_TABLE_NAME \
        --location $LOCATION
    ```

    **Expected Output:**
    ```json
    {
      "disableBgpRoutePropagation": false,
      "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_05_rg/providers/Microsoft.Network/routeTables/iatd_labs_05_routetable_app",
      "location": "eastus",
      "name": "iatd_labs_05_routetable_app",
      "provisioningState": "Succeeded",
      "resourceGroup": "iatd_labs_05_rg",
      "routes": [],
      "subnets": null,
      "tags": null,
      "type": "Microsoft.Network/routeTables"
    }
    ```

2.  **Add a Route to the Route Table (Direct all traffic to Azure Firewall):**

    ```bash
    ROUTE_TABLE_NAME="iatd_labs_05_routetable_app"
    FIREWALL_PRIVATE_IP="<Replace with Azure Firewall's Private IP address>" # e.g. 172.16.0.4

    az network route-table route create \
        --resource-group $RESOURCE_GROUP \
        --name RouteToAzureFirewall \
        --route-table-name $ROUTE_TABLE_NAME \
        --address-prefix 0.0.0.0/0 \
        --next-hop-type VirtualAppliance \
        --next-hop-ip-address $FIREWALL_PRIVATE_IP
    ```

3.  **Associate the Route Table with the App Subnet:**

    ```bash
    az network vnet subnet update \
        --resource-group $RESOURCE_GROUP \
        --vnet-name $VNET_NAME \
        --name $SUBNET_APP_NAME \
        --route-table $ROUTE_TABLE_NAME
    ```

    **Expected Output:**
    ```json
    {
      "addressPrefix": "172.16.1.0/24",
      "delegations": [],
      "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_05_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_05_vnet/subnets/iatd_labs_05_subnet_app",
      "name": "iatd_labs_05_subnet_app",
      "networkSecurityGroup": null,
      "privateEndpointNetworkPolicies": "Disabled",
      "privateLinkServiceNetworkPolicies": "Enabled",
      "provisioningState": "Succeeded",
      "resourceNavigationLinks": [],
      "routeTable": {
        "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_05_rg/providers/Microsoft.Network/routeTables/iatd_labs_05_routetable_app",
        "resourceGroup": "iatd_labs_05_rg"
      },
      "serviceEndpointPolicies": [],
      "serviceEndpoints": []
    }
    ```

### Part 4: Configuring Azure Firewall Rule

1.  **Create a Firewall Rule (Portal):**
    *   Navigate to the `iatd_labs_05_azurefirewall` in the Azure Portal.
    *   Select **Rules** under **Settings**.
    *   Click **Add rule collection**.
        *   Name: AllowSQL
        *   Rule collection type: Network
        *   Priority: 100
        *   Action: Allow
        *   Add Rule:
            *   Name: AllowSQL
            *   Protocol: TCP
            *   Source type: Address
            *   Source: 10.0.1.0/24 (App Subnet)
            *   Destination type: Address
            *   Destination: 10.0.2.0/24 (DB Subnet)
            *   Destination ports: 1433
    * Click **Add** and then **Create**.

### Part 5: Creating Virtual Machines in Subnets

1.  **Create App Virtual Machine (`iatd_labs_05_vm_app_01`) (Portal):**
    *   Search for "Virtual machines" and select.
    *   Click **Create** -> **Azure virtual machine**.
        *   **Basics Tab:**
            *   Subscription: Your subscription.
            *   Resource group: `iatd_labs_05_rg`
            *   Virtual machine name: `iatd_labs_05_vm_app_01`
            *   Region: Your region.
            *   Image: `Ubuntu Server 22.04 LTS`
            *   Size: Choose a size (e.g., Standard_B1ls).
            *   Username: `azureuser`
            *   Authentication type: `Password`
            *   Password: Set a password.
        *   **Networking Tab:**
            *   Virtual network: `iatd_labs_05_vnet`
            *   Subnet: `iatd_labs_05_subnet_app`
            *   Public IP: **None** (Important: No Public IP)
            *   NIC network security group: `None`
        *   **Management Tab:**
            *   Disable boot diagnostics
    *   Click **Review + create**, then **Create**.

2.  **Create DB Virtual Machine (`iatd_labs_05_vm_db_01`) (Portal):**
    *   Repeat the process above, but with the following changes:
        *   Virtual machine name: `iatd_labs_05_vm_db_01`
        *   Subnet: `iatd_labs_05_subnet_db`
        *   Public IP: **None** (Important: No Public IP)
        *   NIC network security group: `None`
    *   **Management Tab:**
        *   Disable boot diagnostics

### Part 6: Testing Network Traffic Flow

1.  **Connect to App VM:**
    *   You'll need a "jump box" VM with a public IP in the same VNet or use Azure Bastion to connect to the App VM.
    *   SSH into the jump box VM.
    *   From the jump box, SSH into the `iatd_labs_05_vm_app_01` VM using its private IP address.

2.  **(Optional) Install telnet and test Connectivity from App VM to DB VM on Port 1433:**

    *   From the `iatd_labs_05_vm_app_01` VM, try to telnet to the private IP address of the `iatd_labs_05_vm_db_01` VM on port 1433.

        ```bash
        sudo apt update
        sudo apt install telnet -y
        telnet <private_ip_of_db_vm> 1433
        ```

    *   If the connection is successful, it will show a blank screen or a message related to SQL Server. This confirms that the App VM can connect to the DB VM on port 1433 *through the Azure Firewall*. If telnet is not installed, it can be replaced with nmap: `nmap -p 1433 <private_ip_of_db_vm>`.

3.  **Test Connectivity from App VM to DB VM on other ports(Portal):**
    *   Remove The rule created in step 1 of this part, and observe that connection to the DB VM via port 1433 is no longer working.
4.  **Verify UDR (Portal):**
    *   To confirm that the UDR is working, try to ping a public IP address (e.g., 8.8.8.8) from the `iatd_labs_05_vm_app_01` VM. Since we have not created an Azure Firewall rule to allow internet access, the attempt will fail.

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
    Resource group 'iatd_labs_05_rg' could not be found.
    ```

3.  **Clean Local Environment Variables:**

    ```bash
    unset RESOURCE_GROUP LOCATION VNET_NAME SUBNET_APP_NAME SUBNET_DB_NAME SUBNET_FW_NAME ADDRESS_PREFIX SUBNET_APP_PREFIX SUBNET_DB_PREFIX SUBNET_FW_PREFIX ROUTE_TABLE_NAME FIREWALL_PRIVATE_IP
    ```

**Learning Outcomes:**

*   Successfully created an Azure Virtual Network with subnets.
*   Successfully deployed and configured Azure Firewall.
*   Successfully created a User-Defined Route (UDR) to route traffic to Azure Firewall.
*   Successfully associated the Route Table with the Application Subnet.
*   Successfully configured Azure Firewall rules to allow specific traffic to the Database Subnet.
*   Successfully verified that traffic is routed through the Azure Firewall.
*   Understand how UDRs and Azure Firewall can be used to control and secure network traffic within a VNet.
