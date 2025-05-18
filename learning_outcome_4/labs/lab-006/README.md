## IATD Microcredential Cloud Networking: Lab 6 - Securing Azure SQL Database with Private Endpoint and NSG, Leveraging Service Tags

**Objective:** This lab demonstrates how to secure an Azure SQL Database using a Private Endpoint, making it accessible only from within your Virtual Network (VNet). You will also configure an NSG rule on the application subnet to allow outbound traffic only to Azure Storage on port 443, using Service Tags to simplify the rule configuration.

**Estimated Time:** 90 - 120 minutes

**Prerequisites:**

1.  **Azure Subscription:** Requires an active Azure subscription. A free account is available at [https://azure.microsoft.com/free/](https://azure.microsoft.com/free/).
2.  **Basic knowledge of Azure Networking:** Familiarity with Virtual Networks, Subnets, and Network Security Groups.
3.  **Existing Azure SQL Database:** You will need an existing Azure SQL Database. If you don't have one, you'll need to create it before starting the lab (this can take some time). Note the server name and database name.
4.  **Existing Azure Storage Account:** You will need an existing Azure Storage Account.

## Let's get started!

### Lab Conventions

*   **Azure Cloud Shell:** All PowerShell and CLI interactions will occur within the Azure Cloud Shell environment.
*   **Azure Portal:** The Azure Portal will be used for visualization and configuration tasks.
*   **Naming Conventions:** Resource names follow the convention `iatd_labs_06_*`.
*   **IP Address Range:** The `172.16.0.0/16` range will be used (as per standardization).
*   **Location:** Choose a consistent Azure region.

#### Resource Naming

*   **Resource Group:** `iatd_labs_06_rg`
*   **Virtual Network:** `iatd_labs_06_vnet`
*   **Subnet-App:** `iatd_labs_06_subnet_app`
*   **Subnet-DB:** `iatd_labs_06_subnet_db`
*   **Private Endpoint:** `iatd_labs_06_sqldb_pe`
*   **NSG-App:** `iatd_labs_06_nsg_app`
*   **VM-App-01:** `iatd_labs_06_vm_app_01`
*   **Azure SQL Server:** (Use the name of your existing SQL Server)
*   **Azure SQL Database:** (Use the name of your existing SQL Database)
*   **Storage Account:** (Use the name of your existing storage account)

### Part 1: Setting up the Virtual Network and Subnets

1.  **Create a Resource Group:**

    ```bash
    RESOURCE_GROUP="iatd_labs_06_rg"
    LOCATION="eastus" # Your Region

    az group create --name $RESOURCE_GROUP --location $LOCATION
    ```

    **Expected Output:**
    ```json
    {
      "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_06_rg",
      "location": "eastus",
      "managedBy": null,
      "name": "iatd_labs_06_rg",
      "properties": {
        "provisioningState": "Succeeded"
      },
      "tags": null,
      "type": "Microsoft.Resources/resourceGroups"
    }
    ```

2.  **Create a Virtual Network with Subnets:**

    ```bash
    VNET_NAME="iatd_labs_06_vnet"
    SUBNET_APP_NAME="iatd_labs_06_subnet_app"
    SUBNET_DB_NAME="iatd_labs_06_subnet_db"
    ADDRESS_PREFIX="10.0.0.0/16"
    SUBNET_APP_PREFIX="10.0.1.0/24"
    SUBNET_DB_PREFIX="10.0.2.0/24"

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
        "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_06_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_06_vnet",
        "location": "eastus",
        "name": "iatd_labs_06_vnet",
        "provisioningState": "Succeeded",
        "resourceGuid": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "subnets": [
          {
            "addressPrefix": "172.16.0.0/24",
            "delegations": [],
            "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_06_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_06_vnet/subnets/iatd_labs_06_subnet_app",
            "name": "iatd_labs_06_subnet_app",
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
        --address-prefix $SUBNET_DB_PREFIX \
        --disable-private-endpoint-network-policies true #Important: Disable Private Endpoint Network Policies
    ```

### Part 2: Creating the Private Endpoint for Azure SQL Database

1.  **Get Azure SQL Server Resource ID:**

    ```bash
    SQL_SERVER_NAME="<Your Azure SQL Server Name>" #e.g. iatd-sql-server
    az sql server show --name $SQL_SERVER_NAME --resource-group $RESOURCE_GROUP --query id -o tsv
    ```

2.  **Create the Private Endpoint (Portal):**
    *   In the Azure portal, search for "Private endpoints" and select it.
    *   Click **Create**.
        *   **Basics Tab:**
            *   Subscription: Your subscription.
            *   Resource group: `iatd_labs_06_rg`
            *   Name: `iatd_labs_06_sqldb_pe`
            *   Region: Your region.
        *   **Resource Tab:**
            *   Connection method: "My directory"
            *   Subscription: Your subscription.
            *   Resource type: Microsoft.Sql/servers
            *   Resource: Select your SQL Server (or paste the Resource ID obtained above)
            *   Target subresource: SqlServer
        *   **Virtual Network Tab:**
            *   Virtual network: `iatd_labs_06_vnet`
            *   Subnet: `iatd_labs_06_subnet_db`
        *   **DNS Tab:**
            *   Integrate with private DNS zone: Yes (or No, and configure DNS manually - see below)
            *   Subscription: Your subscription
            *   Private DNS zone: Create new. Name it `privatelink.database.windows.net` if it doesn't automatically provide a name

    *   Click **Review + create**, then **Create**.

3.  **(Alternative - Manual DNS Configuration):** If you choose "No" for Private DNS Zone Integration, you'll need to manually create a Private DNS Zone and DNS records to resolve the SQL Server's FQDN to the Private Endpoint's IP address. See Microsoft's Documentation for detailed instructions.

### Part 3: Configuring NSG for Application Subnet with Service Tags

1.  **Create NSG for App Subnet:**

    ```bash
    NSG_APP_NAME="iatd_labs_06_nsg_app"

    az network nsg create \
        --resource-group $RESOURCE_GROUP \
        --name $NSG_APP_NAME \
        --location $LOCATION
    ```

    **Expected Output:**
    ```json
    {
      "defaultSecurityRules": [
        {
          "access": "Allow",
          "description": "Allow inbound traffic from all VMs in VNET",
          "destinationAddressPrefix": "VirtualNetwork",
          "destinationPortRange": "*",
          "direction": "Inbound",
          "name": "AllowVnetInBound",
          "priority": 65000,
          "protocol": "*",
          "provisioningState": "Succeeded",
          "sourceAddressPrefix": "VirtualNetwork",
          "sourcePortRange": "*"
        },
        {
          "access": "Allow",
          "description": "Allow inbound traffic from azure load balancer",
          "destinationAddressPrefix": "*",
          "destinationPortRange": "*",
          "direction": "Inbound",
          "name": "AllowAzureLoadBalancerInBound",
          "priority": 65001,
          "protocol": "*",
          "provisioningState": "Succeeded",
          "sourceAddressPrefix": "AzureLoadBalancer",
          "sourcePortRange": "*"
        },
        {
          "access": "Deny",
          "description": "Deny all inbound traffic",
          "destinationAddressPrefix": "*",
          "destinationPortRange": "*",
          "direction": "Inbound",
          "name": "DenyAllInBound",
          "priority": 65500,
          "protocol": "*",
          "provisioningState": "Succeeded",
          "sourceAddressPrefix": "*",
          "sourcePortRange": "*"
        }
      ],
      "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_06_rg/providers/Microsoft.Network/networkSecurityGroups/iatd_labs_06_nsg_app",
      "location": "eastus",
      "name": "iatd_labs_06_nsg_app",
      "provisioningState": "Succeeded",
      "resourceGuid": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "securityRules": [],
      "type": "Microsoft.Network/networkSecurityGroups"
    }
    ```

2.  **Associate NSG with App Subnet:**

    ```bash
    az network vnet subnet update \
        --resource-group $RESOURCE_GROUP \
        --vnet-name $VNET_NAME \
        --name $SUBNET_APP_NAME \
        --network-security-group $NSG_APP_NAME
    ```

    **Expected Output:**
    ```json
    {
      "addressPrefix": "172.16.0.0/24",
      "delegations": [],
      "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_06_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_06_vnet/subnets/iatd_labs_06_subnet_app",
      "name": "iatd_labs_06_subnet_app",
      "networkSecurityGroup": {
        "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_06_rg/providers/Microsoft.Network/networkSecurityGroups/iatd_labs_06_nsg_app",
        "resourceGroup": "iatd_labs_06_rg"
      },
      "privateEndpointNetworkPolicies": "Disabled",
      "privateLinkServiceNetworkPolicies": "Enabled",
      "provisioningState": "Succeeded",
      "resourceNavigationLinks": [],
      "serviceEndpointPolicies": [],
      "serviceEndpoints": []
    }
    ```

3.  **Create NSG Rule (Allow outbound to Azure Storage using Service Tag):**

    ```bash
    NSG_APP_NAME="iatd_labs_06_nsg_app"

    az network nsg rule create \
        --resource-group $RESOURCE_GROUP \
        --nsg-name $NSG_APP_NAME \
        --name AllowOutboundToStorage \
        --priority 100 \
        --source-address-prefixes 10.0.1.0/24 \
        --destination-service-tags Storage \
        --destination-port-ranges 443 \
        --access Allow \
        --protocol Tcp \
        --direction Outbound
    ```

4.  **Create NSG Rule (Deny all other outbound):**

    ```bash
    NSG_APP_NAME="iatd_labs_06_nsg_app"

    az network nsg rule create \
        --resource-group $RESOURCE_GROUP \
        --nsg-name $NSG_APP_NAME \
        --name DenyAllOtherOutbound \
        --priority 4096 \
        --source-address-prefixes 10.0.1.0/24 \
        --destination-port-ranges '*' \
        --access Deny \
        --protocol '*' \
        --direction Outbound
    ```

### Part 4: Creating Virtual Machine in App Subnet

1.  **Create App Virtual Machine (`iatd_labs_06_vm_app_01`) (Portal):**
    *   Search for "Virtual machines" and select.
    *   Click **Create** -> **Azure virtual machine**.
        *   **Basics Tab:**
            *   Subscription: Your subscription.
            *   Resource group: `iatd_labs_06_rg`
            *   Virtual machine name: `iatd_labs_06_vm_app_01`
            *   Region: Your region.
            *   Image: `Ubuntu Server 22.04 LTS`
            *   Size: Choose a size (e.g., Standard_B1ls).
            *   Username: `azureuser`
            *   Authentication type: `Password`
            *   Password: Set a password.
        *   **Networking Tab:**
            *   Virtual network: `iatd_labs_06_vnet`
            *   Subnet: `iatd_labs_06_subnet_app`
            *   Public IP: **None** (Important: No Public IP)
            *   NIC network security group: `None`
        *   **Management Tab:**
            *   Disable boot diagnostics
    *   Click **Review + create**, then **Create**.

### Part 5: Testing Connectivity

1.  **Connect to App VM:**
    *   You'll need a "jump box" VM with a public IP in the same VNet or use Azure Bastion to connect to the App VM.
    *   SSH into the jump box VM.
    *   From the jump box, SSH into the `iatd_labs_06_vm_app_01` VM using its private IP address.

2.  **Test Connectivity to Azure SQL Database:**

    *   Install `sqlcmd`

        ```bash
        sudo apt-get update
        sudo apt-get install unixodbc-dev
        sudo apt-get install apt-transport-https
        curl https://packages.microsoft.com/keys/microsoft.asc | apt-key add -
        curl https://packages.microsoft.com/config/ubuntu/$(lsb_release -rs)/prod.list > /etc/apt/sources.list.d/mssql-release.list
        sudo apt-get update
        sudo ACCEPT_EULA=Y apt-get install msodbcsql17
        sudo ACCEPT_EULA=Y apt-get install mssql-tools
        echo 'export PATH="$PATH:/opt/mssql-tools/bin"' >> ~/.bashrc
        source ~/.bashrc
        ```

    *   From the `iatd_labs_06_vm_app_01` VM, use `sqlcmd` to connect to the Azure SQL Database using its fully qualified domain name (FQDN) and the appropriate credentials. You will need to find the FQDN under the SQL server properties
    *   If the connection is successful, it confirms that the App VM can connect to the SQL Database *privately* through the Private Endpoint.

3.  **Test connectivity to Storage Account on Port 443:**

    ```bash
    curl -v https://<Your Storage Account Name>.blob.core.windows.net
    ```

    * The connection should be successful

4.  **Test Connectivity to Other Internet Destinations:**

    *   From the `iatd_labs_06_vm_app_01` VM, try to ping a public IP address (e.g., 8.8.8.8) or `curl` any web site other than Azure Storage.

    *   This *should* fail because the NSG-App blocks all outbound traffic to other destinations.

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
    Resource group 'iatd_labs_06_rg' could not be found.
    ```

3.  **Clean Local Environment Variables:**

    ```bash
    unset RESOURCE_GROUP LOCATION VNET_NAME SUBNET_APP_NAME SUBNET_PE_NAME ADDRESS_PREFIX SUBNET_APP_PREFIX SUBNET_PE_PREFIX NSG_APP_NAME
    ```

**Learning Outcomes:**

*   Successfully created an Azure Virtual Network with subnets.
*   Successfully created a Private Endpoint for an Azure SQL Database.
*   Successfully configured an NSG to allow outbound traffic to Azure Storage using a Service Tag.
*   Successfully verified that the application VM can connect to the SQL Database privately.
*   Successfully verified that the application VM can connect to the storage account.
*   Successfully verified that outbound traffic from the application VM is blocked from all other internet sources.
*   Understand how Private Endpoints and NSGs with Service Tags can be used to secure Azure resources and simplify rule management.
