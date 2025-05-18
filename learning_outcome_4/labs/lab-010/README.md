## IATD Microcredential Cloud Networking: Lab 10 - Azure Private Endpoints: Configuration and Secure Access Patterns

**Objective:** This lab dives deep into Azure Private Endpoints. You'll configure Private Endpoints for various Azure services, explore different connectivity options, implement network isolation, and examine typical access patterns.

**Estimated Time:** 90 - 120 minutes

**Prerequisites:**

1.  **Azure Subscription:** Requires an active Azure subscription. A free account is available at [https://azure.microsoft.com/free/](https://azure.microsoft.com/free/).
2.  **Basic knowledge of Azure Networking:** Familiarity with Virtual Networks, Subnets, and Network Security Groups.
3.  **Existing Azure SQL Database:** You will need an existing Azure SQL Database. Note the server name and database name.
4.  **Existing Azure Storage Account:** You will need an existing Azure Storage Account.
5.  **Existing Azure Key Vault:** You will need an existing Azure Key Vault.

## Let's get started!

### Lab Conventions

*   **Azure Cloud Shell:** All PowerShell and CLI interactions will occur within the Azure Cloud Shell environment.
*   **Azure Portal:** The Azure Portal will be used for visualization and configuration tasks.
*   **Naming Conventions:** Resource names follow the convention `iatd_labs_10_*`.
*   **IP Address Range:** The `172.16.0.0/16` range will be used (as per standardization).
*   **Location:** Choose a consistent Azure region.

#### Resource Naming

*   **Resource Group:** `iatd_labs_10_rg`
*   **Virtual Network:** `iatd_labs_10_vnet`
*   **Subnet-App:** `iatd_labs_10_subnet_app`
*   **Subnet-PrivateEndpoint:** `iatd_labs_10_subnet_pe`
*   **Private Endpoint - SQL DB:** `iatd_labs_10_sqldb_pe`
*   **Private Endpoint - Storage:** `iatd_labs_10_storage_pe`
*   **Private Endpoint - Key Vault:** `iatd_labs_10_keyvault_pe`
*   **VM-App-01:** `iatd_labs_10_vm_app_01`

*   **Azure SQL Server:** (Use the name of your existing SQL Server)
*   **Azure SQL Database:** (Use the name of your existing SQL Database)
*   **Storage Account:** (Use the name of your existing storage account)
*    **Key Vault:** (Use the name of your existing keyvault)

### Part 1: Setting up the Virtual Network and Subnets

1.  **Create a Resource Group:**

    ```bash
    RESOURCE_GROUP="iatd_labs_10_rg"
    LOCATION="eastus" # Your Region

    az group create --name $RESOURCE_GROUP --location $LOCATION
    ```

    **Expected Output:**
    ```json
    {
      "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_10_rg",
      "location": "eastus",
      "managedBy": null,
      "name": "iatd_labs_10_rg",
      "properties": {
        "provisioningState": "Succeeded"
      },
      "tags": null,
      "type": "Microsoft.Resources/resourceGroups"
    }
    ```

2.  **Create a Virtual Network and Subnets:**

    ```bash
    VNET_NAME="iatd_labs_10_vnet"
    SUBNET_APP_NAME="iatd_labs_10_subnet_app"
    SUBNET_PE_NAME="iatd_labs_10_subnet_pe"
    ADDRESS_PREFIX="172.16.0.0/16"
    SUBNET_APP_PREFIX="172.16.0.0/24"
    SUBNET_PE_PREFIX="172.16.1.0/24"

    az network vnet create \
        --resource-group $RESOURCE_GROUP \
        --name $VNET_NAME \
        --address-prefixes $ADDRESS_PREFIX \
        --location $LOCATION \
        --subnet-name $SUBNET_APP_NAME \
        --subnet-prefixes $SUBNET_APP_PREFIX
    ```

    **Expected Output:**
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
        "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_10_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_10_vnet",
        "location": "eastus",
        "name": "iatd_labs_10_vnet",
        "provisioningState": "Succeeded",
        "resourceGuid": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "subnets": [
          {
            "addressPrefix": "172.16.0.0/24",
            "delegations": [],
            "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_10_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_10_vnet/subnets/iatd_labs_10_subnet_app",
            "name": "iatd_labs_10_subnet_app",
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
        --name $SUBNET_PE_NAME \
        --address-prefix $SUBNET_PE_PREFIX \
        --disable-private-endpoint-network-policies true # Important! Disable network policies on this Subnet
    ```

    **Expected Output:**
    ```json
    {
      "addressPrefix": "172.16.1.0/24",
      "delegations": [],
      "etag": "W/\"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx\"",
      "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_10_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_10_vnet/subnets/iatd_labs_10_subnet_pe",
      "name": "iatd_labs_10_subnet_pe",
      "privateEndpointNetworkPolicies": "Disabled",
      "privateLinkServiceNetworkPolicies": "Enabled",
      "provisioningState": "Succeeded",
      "resourceGroup": "iatd_labs_10_rg",
      "type": "Microsoft.Network/virtualNetworks/subnets"
    }
    ```

### Part 2: Creating Private Endpoints

1.  **SQL Database Private Endpoint (CLI):**

    ```bash
    # Set your SQL Server name and resource group
    SQL_SERVER_NAME="your-sql-server-name" # Replace with your SQL Server name
    SQL_SERVER_RG="your-sql-server-rg" # Replace with your SQL Server resource group

    # Get the resource ID of your SQL Server
    SQL_SERVER_ID=$(az sql server show --name $SQL_SERVER_NAME --resource-group $SQL_SERVER_RG --query id --output tsv)

    # Create private endpoint for SQL Server
    SQLDB_PE_NAME="iatd_labs_10_sqldb_pe"

    az network private-endpoint create \
        --resource-group $RESOURCE_GROUP \
        --name $SQLDB_PE_NAME \
        --vnet-name $VNET_NAME \
        --subnet $SUBNET_PE_NAME \
        --private-connection-resource-id $SQL_SERVER_ID \
        --group-id sqlServer \
        --connection-name "sqldbconnection" \
        --location $LOCATION
    ```

    **Expected Output:**
    ```json
    {
      "customDnsConfigs": null,
      "customNetworkInterfaceName": null,
      "etag": "W/\"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx\"",
      "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_10_rg/providers/Microsoft.Network/privateEndpoints/iatd_labs_10_sqldb_pe",
      "location": "eastus",
      "manualPrivateLinkServiceConnections": null,
      "name": "iatd_labs_10_sqldb_pe",
      "networkInterfaces": [
        {
          "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_10_rg/providers/Microsoft.Network/networkInterfaces/iatd_labs_10_sqldb_pe.nic.xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
          "resourceGroup": "iatd_labs_10_rg"
        }
      ],
      "privateLinkServiceConnections": [
        {
          "etag": "W/\"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx\"",
          "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_10_rg/providers/Microsoft.Network/privateEndpoints/iatd_labs_10_sqldb_pe/privateLinkServiceConnections/sqldbconnection",
          "name": "sqldbconnection",
          "privateLinkServiceConnectionState": {
            "actionsRequired": "None",
            "description": "Auto-approved",
            "status": "Approved"
          },
          "privateLinkServiceId": "/subscriptions/<subscription_id>/resourceGroups/your-sql-server-rg/providers/Microsoft.Sql/servers/your-sql-server-name",
          "provisioningState": "Succeeded",
          "type": "Microsoft.Network/privateEndpoints/privateLinkServiceConnections"
        }
      ],
      "provisioningState": "Succeeded",
      "resourceGroup": "iatd_labs_10_rg",
      "subnet": {
        "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_10_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_10_vnet/subnets/iatd_labs_10_subnet_pe",
        "resourceGroup": "iatd_labs_10_rg"
      },
      "type": "Microsoft.Network/privateEndpoints"
    }
    ```

    **Alternative: Create SQL Database Private Endpoint (Portal):**
    *   In the Azure portal, search for "Private endpoints" and select it.
    *   Click **Create**.
        *   **Basics Tab:**
            *   Subscription: Your subscription.
            *   Resource group: `iatd_labs_10_rg`
            *   Name: `iatd_labs_10_sqldb_pe`
            *   Region: Your region.
        *   **Resource Tab:**
            *   Connection method: "My directory"
            *   Subscription: Your subscription.
            *   Resource type: Microsoft.Sql/servers
            *   Resource: Select your SQL Server
            *   Target subresource: SqlServer
        *   **Virtual Network Tab:**
            *   Virtual network: `iatd_labs_10_vnet`
            *   Subnet: `iatd_labs_10_subnet_pe`
        *   **DNS Tab:**
            *   Integrate with private DNS zone: Yes
            * This zone must be named `privatelink.database.windows.net`

    *   Click **Review + create**, then **Create**.

2.  **Storage Account Private Endpoint (CLI):**

    ```bash
    # Set your Storage Account name and resource group
    STORAGE_ACCOUNT_NAME="yourstorageaccount" # Replace with your Storage Account name
    STORAGE_ACCOUNT_RG="your-storage-account-rg" # Replace with your Storage Account resource group

    # Get the resource ID of your Storage Account
    STORAGE_ACCOUNT_ID=$(az storage account show --name $STORAGE_ACCOUNT_NAME --resource-group $STORAGE_ACCOUNT_RG --query id --output tsv)

    # Create private endpoint for Storage Account (blob)
    STORAGE_PE_NAME="iatd_labs_10_storage_pe"

    az network private-endpoint create \
        --resource-group $RESOURCE_GROUP \
        --name $STORAGE_PE_NAME \
        --vnet-name $VNET_NAME \
        --subnet $SUBNET_PE_NAME \
        --private-connection-resource-id $STORAGE_ACCOUNT_ID \
        --group-id blob \
        --connection-name "storageconnection" \
        --location $LOCATION
    ```

    **Expected Output:**
    ```json
    {
      "customDnsConfigs": null,
      "customNetworkInterfaceName": null,
      "etag": "W/\"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx\"",
      "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_10_rg/providers/Microsoft.Network/privateEndpoints/iatd_labs_10_storage_pe",
      "location": "eastus",
      "manualPrivateLinkServiceConnections": null,
      "name": "iatd_labs_10_storage_pe",
      "networkInterfaces": [
        {
          "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_10_rg/providers/Microsoft.Network/networkInterfaces/iatd_labs_10_storage_pe.nic.xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
          "resourceGroup": "iatd_labs_10_rg"
        }
      ],
      "privateLinkServiceConnections": [
        {
          "etag": "W/\"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx\"",
          "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_10_rg/providers/Microsoft.Network/privateEndpoints/iatd_labs_10_storage_pe/privateLinkServiceConnections/storageconnection",
          "name": "storageconnection",
          "privateLinkServiceConnectionState": {
            "actionsRequired": "None",
            "description": "Auto-approved",
            "status": "Approved"
          },
          "privateLinkServiceId": "/subscriptions/<subscription_id>/resourceGroups/your-storage-account-rg/providers/Microsoft.Storage/storageAccounts/yourstorageaccount",
          "provisioningState": "Succeeded",
          "type": "Microsoft.Network/privateEndpoints/privateLinkServiceConnections"
        }
      ],
      "provisioningState": "Succeeded",
      "resourceGroup": "iatd_labs_10_rg",
      "subnet": {
        "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_10_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_10_vnet/subnets/iatd_labs_10_subnet_pe",
        "resourceGroup": "iatd_labs_10_rg"
      },
      "type": "Microsoft.Network/privateEndpoints"
    }
    ```

    **Alternative: Create Storage Account Private Endpoint (Portal):**
    *   Repeat the process above, but with the following changes:
        *   Name: `iatd_labs_10_storage_pe`
        *   Resource type: Microsoft.Storage/storageAccounts
        *   Resource: Select your Storage Account
        *   Target subresource: blob
        *   Integrate with private DNS zone: Yes
        *    This zone must be named `privatelink.blob.core.windows.net`

3.  **Key Vault Private Endpoint (CLI):**

    ```bash
    # Set your Key Vault name and resource group
    KEY_VAULT_NAME="your-key-vault-name" # Replace with your Key Vault name
    KEY_VAULT_RG="your-key-vault-rg" # Replace with your Key Vault resource group

    # Get the resource ID of your Key Vault
    KEY_VAULT_ID=$(az keyvault show --name $KEY_VAULT_NAME --resource-group $KEY_VAULT_RG --query id --output tsv)

    # Create private endpoint for Key Vault
    KEYVAULT_PE_NAME="iatd_labs_10_keyvault_pe"

    az network private-endpoint create \
        --resource-group $RESOURCE_GROUP \
        --name $KEYVAULT_PE_NAME \
        --vnet-name $VNET_NAME \
        --subnet $SUBNET_PE_NAME \
        --private-connection-resource-id $KEY_VAULT_ID \
        --group-id vault \
        --connection-name "keyvaultconnection" \
        --location $LOCATION
    ```

    **Expected Output:**
    ```json
    {
      "customDnsConfigs": null,
      "customNetworkInterfaceName": null,
      "etag": "W/\"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx\"",
      "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_10_rg/providers/Microsoft.Network/privateEndpoints/iatd_labs_10_keyvault_pe",
      "location": "eastus",
      "manualPrivateLinkServiceConnections": null,
      "name": "iatd_labs_10_keyvault_pe",
      "networkInterfaces": [
        {
          "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_10_rg/providers/Microsoft.Network/networkInterfaces/iatd_labs_10_keyvault_pe.nic.xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
          "resourceGroup": "iatd_labs_10_rg"
        }
      ],
      "privateLinkServiceConnections": [
        {
          "etag": "W/\"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx\"",
          "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_10_rg/providers/Microsoft.Network/privateEndpoints/iatd_labs_10_keyvault_pe/privateLinkServiceConnections/keyvaultconnection",
          "name": "keyvaultconnection",
          "privateLinkServiceConnectionState": {
            "actionsRequired": "None",
            "description": "Auto-approved",
            "status": "Approved"
          },
          "privateLinkServiceId": "/subscriptions/<subscription_id>/resourceGroups/your-key-vault-rg/providers/Microsoft.KeyVault/vaults/your-key-vault-name",
          "provisioningState": "Succeeded",
          "type": "Microsoft.Network/privateEndpoints/privateLinkServiceConnections"
        }
      ],
      "provisioningState": "Succeeded",
      "resourceGroup": "iatd_labs_10_rg",
      "subnet": {
        "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_10_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_10_vnet/subnets/iatd_labs_10_subnet_pe",
        "resourceGroup": "iatd_labs_10_rg"
      },
      "type": "Microsoft.Network/privateEndpoints"
    }
    ```

    **Alternative: Create Key Vault Private Endpoint (Portal):**
     *   Repeat the process above, but with the following changes:
        *   Name: `iatd_labs_10_keyvault_pe`
        *   Resource type: Microsoft.KeyVault/vaults
        *   Resource: Select your Key Vault
        *   Target subresource: vault
        *   Integrate with private DNS zone: Yes
        *    This zone must be named `privatelink.vaultcore.azure.net`

### Part 3: Creating Virtual Machine

1.  **Create App Virtual Machine (`iatd_labs_10_vm_app_01`) (Portal):**
    *   Search for "Virtual machines" and select.
    *   Click **Create** -> **Azure virtual machine**.
        *   **Basics Tab:**
            *   Subscription: Your subscription.
            *   Resource group: `iatd_labs_10_rg`
            *   Virtual machine name: `iatd_labs_10_vm_app_01`
            *   Region: Your region.
            *   Image: `Ubuntu Server 22.04 LTS`
            *   Size: Choose a size (e.g., Standard_B1ls).
            *   Username: `azureuser`
            *   Authentication type: `Password`
            *   Password: Set a password.
        *   **Networking Tab:**
            *   Virtual network: `iatd_labs_10_vnet`
            *   Subnet: `iatd_labs_10_subnet_app`
            *   Public IP: **None** (Important: No Public IP)
            *   NIC network security group: `None`
        *   **Management Tab:**
            *   Disable boot diagnostics
    *   Click **Review + create**, then **Create**.

### Part 4: Validating Connectivity

1.  **Connect to App VM:**
    *   You'll need a "jump box" VM with a public IP in the same VNet or use Azure Bastion to connect to the App VM.
    *   SSH into the jump box VM.
    *   From the jump box, SSH into the `iatd_labs_10_vm_app_01` VM

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

    *   From the `iatd_labs_10_vm_app_01` VM, use `sqlcmd` to connect to the Azure SQL Database using its fully qualified domain name (FQDN) and the appropriate credentials. You will need to find the FQDN under the SQL server properties
    *   If the connection is successful, it confirms that the App VM can connect to the SQL Database *privately* through the Private Endpoint.

3.  **Test Connectivity to Azure Storage Account:**

    *   Install the Azure CLI:

        ```bash
        curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
        ```

    *   Login to Azure: `az login`
    *   From the `iatd_labs_10_vm_app_01` VM, use the Azure CLI to list the storage containers in your storage account, using the FQDN:

        ```bash
        STORAGE_ACCOUNT_NAME="<Your Storage Account Name>"
        az storage container list --account-name $STORAGE_ACCOUNT_NAME
        ```

    *   If you can list the containers, it confirms you are accessing the storage account *privately* through the Private Endpoint.

4.  **Test Connectivity to Azure Key Vault:**

    *   Install the Azure CLI (if not already installed).
    *   Enable Managed Identity for the VM:
        *   In the Azure Portal, navigate to the `iatd_labs_10_vm_app_01` VM.
        *   Select **Identity** under **Settings**.
        *   Set "System assigned" to **On**.
        *   Click **Save**.
    *   Grant the VM permissions to access Key Vault:
        *   Navigate to your Key Vault in the Azure Portal.
        *   Select **Access configuration** under **Settings**, and ensure that the permission model is set to *Azure role-based access control*.
        *   Select **Access control (IAM)**, click **Add role assignment**, and assign the *Key Vault Reader* role to the VM's managed identity.
    *  Run command to view secrets from `iatd_labs_10_vm_app_01`:
         ```bash
        KEYVAULT_NAME="<Your Key Vault Name>"
        az keyvault secret list --vault-name $KEYVAULT_NAME
        ```
    *   If you can list the secrets, it confirms you are accessing the key vault *privately* through the Private Endpoint.

### Part 5: Network Isolation Verification

1.  **Disable Public Network Access (Portal):**
    *   Navigate to your Azure SQL Server in the Azure Portal.
    *   Select **Networking** under **Security**.
    *   Set "Public network access" to **Disabled**.
    *   Click **Save**.
2.  **Now test if from your local PC you can connect to the SQL DB or KeyVault?**
    *   You should not be able to connect to resources, to confirm that the public access is blocked
    *  *Note: This confirms isolation, and should only be managed via the VNET now*.

### Post-Lab: Cleanup

To avoid incurring unnecessary costs, it's essential to clean up the resources created during this lab.

### Part 5: Verification and Testing

1.  **Verify Private Endpoints Status:**

    ```bash
    # Verify SQL Database Private Endpoint
    az network private-endpoint show \
        --resource-group $RESOURCE_GROUP \
        --name $SQLDB_PE_NAME \
        --query privateLinkServiceConnections[0].privateLinkServiceConnectionState.status -o tsv
    ```

    **Expected Output:**
    ```
    Approved
    ```

    ```bash
    # Verify Storage Account Private Endpoint
    az network private-endpoint show \
        --resource-group $RESOURCE_GROUP \
        --name $STORAGE_PE_NAME \
        --query privateLinkServiceConnections[0].privateLinkServiceConnectionState.status -o tsv
    ```

    **Expected Output:**
    ```
    Approved
    ```

    ```bash
    # Verify Key Vault Private Endpoint
    az network private-endpoint show \
        --resource-group $RESOURCE_GROUP \
        --name $KEYVAULT_PE_NAME \
        --query privateLinkServiceConnections[0].privateLinkServiceConnectionState.status -o tsv
    ```

    **Expected Output:**
    ```
    Approved
    ```

2.  **List All Private Endpoints:**

    ```bash
    az network private-endpoint list \
        --resource-group $RESOURCE_GROUP \
        --query "[].{Name:name, PrivateIP:networkInterfaces[0].id, Status:privateLinkServiceConnections[0].privateLinkServiceConnectionState.status}" \
        --output table
    ```

    **Expected Output:**
    ```
    Name                       PrivateIP                                                                                                                                 Status
    -------------------------  ---------------------------------------------------------------------------------------------------------------------------------------  --------
    iatd_labs_10_sqldb_pe      /subscriptions/<subscription_id>/resourceGroups/iatd_labs_10_rg/providers/Microsoft.Network/networkInterfaces/iatd_labs_10_sqldb_pe.nic.xxxx      Approved
    iatd_labs_10_storage_pe    /subscriptions/<subscription_id>/resourceGroups/iatd_labs_10_rg/providers/Microsoft.Network/networkInterfaces/iatd_labs_10_storage_pe.nic.xxxx    Approved
    iatd_labs_10_keyvault_pe   /subscriptions/<subscription_id>/resourceGroups/iatd_labs_10_rg/providers/Microsoft.Network/networkInterfaces/iatd_labs_10_keyvault_pe.nic.xxxx   Approved
    ```

### Cleanup

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
    Resource group 'iatd_labs_10_rg' could not be found.
    ```

3.  **Clean Local Environment Variables:**

    ```bash
    unset RESOURCE_GROUP LOCATION VNET_NAME SUBNET_APP_NAME SUBNET_PE_NAME ADDRESS_PREFIX SUBNET_APP_PREFIX SUBNET_PE_PREFIX SQL_SERVER_NAME SQL_SERVER_RG SQL_SERVER_ID SQLDB_PE_NAME STORAGE_ACCOUNT_NAME STORAGE_ACCOUNT_RG STORAGE_ACCOUNT_ID STORAGE_PE_NAME KEY_VAULT_NAME KEY_VAULT_RG KEY_VAULT_ID KEYVAULT_PE_NAME
    ```

**Learning Outcomes:**

*   Successfully created an Azure Virtual Network with subnets.
*   Successfully created Private Endpoints for Azure SQL Database, Azure Storage Account and Azure Key Vault.
*   Successfully deployed and configured a Virtual Machine in the application subnet.
*   Successfully verified private connectivity from the application VM to Azure SQL Database, Azure Storage and Azure Key Vault using Private Endpoints.
*   Successfully demonstrated how to disable public network access to Azure SQL Database.
*   Understand the benefits of using Private Endpoints for secure access to Azure services.
