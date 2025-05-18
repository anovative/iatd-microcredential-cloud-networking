## IATD Microcredential Cloud Networking: Lab 1 - Securing Azure Service Access with Service Endpoints

**Objective:** This lab guides you through the process of configuring Azure Service Endpoints. You'll learn how to limit network access to Azure services to only resources within your Virtual Network, enhancing security and control.

**Estimated Time:** 60 - 75 minutes

**Prerequisites:**

1.  **Azure Subscription:** Requires an active Azure subscription. A free account is available at [https://azure.microsoft.com/free/](https://azure.microsoft.com/free/).
2.  **Basic knowledge of Azure Networking:** Familiarity with Virtual Networks and Subnets.
3.  **Azure Storage Account:** You'll need an existing Azure Storage Account or create a new one.

## Let's get started!

### Lab Conventions

*   **Azure Cloud Shell:** All PowerShell and CLI interactions will occur within the Azure Cloud Shell environment.
*   **Azure Portal:** The Azure Portal will be used for visualization and configuration tasks.
*   **Naming Conventions:** Resource names follow the convention `iatd_labs_01_*`.
*   **IP Address Range:** The `172.16.0.0/16` range will be used (as per standardization).
*   **Location:** Choose a consistent Azure region.

#### Resource Naming

*   **Resource Group:** `iatd_labs_01_rg`
*   **Virtual Network:** `iatd_labs_01_vnet`
*   **Subnet:** `iatd_labs_01_subnet`
*   **Storage Account:** `iatdlabs01storage` (Or an existing storage account)

### Part 1: Setting up the Virtual Network

1.  **Create a Resource Group:**

    ```bash
    RESOURCE_GROUP="iatd_labs_01_rg"
    LOCATION="eastus" # Your Region

    az group create --name $RESOURCE_GROUP --location $LOCATION
    ```

    **Expected Output:**
    ```json
    {
      "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_01_rg",
      "location": "eastus",
      "managedBy": null,
      "name": "iatd_labs_01_rg",
      "properties": {
        "provisioningState": "Succeeded"
      },
      "tags": null,
      "type": "Microsoft.Resources/resourceGroups"
    }
    ```

2.  **Create a Virtual Network and Subnet:**

    ```bash
    VNET_NAME="iatd_labs_01_vnet"
    SUBNET_NAME="iatd_labs_01_subnet"
    ADDRESS_PREFIX="172.16.0.0/16"
    SUBNET_PREFIX="172.16.0.0/24"

    az network vnet create \
        --resource-group $RESOURCE_GROUP \
        --name $VNET_NAME \
        --address-prefixes $ADDRESS_PREFIX \
        --subnet-name $SUBNET_NAME \
        --subnet-prefixes $SUBNET_PREFIX \
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
        "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_01_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_01_vnet",
        "location": "eastus",
        "name": "iatd_labs_01_vnet",
        "provisioningState": "Succeeded",
        "resourceGuid": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "subnets": [
          {
            "addressPrefix": "172.16.0.0/24",
            "delegations": [],
            "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_01_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_01_vnet/subnets/iatd_labs_01_subnet",
            "name": "iatd_labs_01_subnet",
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

### Part 2: Configuring Service Endpoints for Azure Storage

1.  **Enable Service Endpoint on Subnet (Azure Portal):**
    *   Navigate to your Virtual Network (`iatd_labs_01_vnet`) in the Azure Portal.
    *   Select **Subnets** under **Settings**.
    *   Select the `iatd_labs_01_subnet` subnet.
    *   Under **Service endpoints**, select `Microsoft.Storage` from the "Services" dropdown.
    *   Click **Save**.

2.  **Enable Service Endpoint on Subnet (Azure CLI):**

    ```bash
    RESOURCE_GROUP="iatd_labs_01_rg"
    VNET_NAME="iatd_labs_01_vnet"
    SUBNET_NAME="iatd_labs_01_subnet"

    az network vnet subnet update \
        --resource-group $RESOURCE_GROUP \
        --vnet-name $VNET_NAME \
        --name $SUBNET_NAME \
        --service-endpoints Microsoft.Storage
    ```

    **Expected Output:**
    ```json
    {
      "addressPrefix": "172.16.0.0/24",
      "delegations": [],
      "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_01_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_01_vnet/subnets/iatd_labs_01_subnet",
      "name": "iatd_labs_01_subnet",
      "networkSecurityGroup": null,
      "privateEndpointNetworkPolicies": "Disabled",
      "privateLinkServiceNetworkPolicies": "Enabled",
      "provisioningState": "Succeeded",
      "resourceNavigationLinks": [],
      "routeTable": null,
      "serviceEndpointPolicies": [],
      "serviceEndpoints": [
        {
          "locations": [
            "eastus",
            "westus"
          ],
          "provisioningState": "Succeeded",
          "service": "Microsoft.Storage"
        }
      ]
    }
    ```

3.  **Verify Route Table Update (Azure CLI):**

    ```bash
    RESOURCE_GROUP="iatd_labs_01_rg"
    VNET_NAME="iatd_labs_01_vnet"
    SUBNET_NAME="iatd_labs_01_subnet"

    az network vnet subnet show \
        --resource-group $RESOURCE_GROUP \
        --vnet-name $VNET_NAME \
        --name $SUBNET_NAME \
        --query "routeTable"
    ```

    *   Examine the output. You should see a route with an address prefix similar to `Storage` and a `nextHopType` of `VirtualNetworkServiceEndpoint`. This indicates that traffic to Azure Storage is now optimized to use the service endpoint.

### Part 3: Configure Storage Account Firewall

1.  **Navigate to the Storage Account (Azure Portal):**
    *   Search for your existing Azure Storage Account (or create a new one).
2.  **Configure Network Access:**
    *   Select **Networking** under **Security + networking**.
    *   Under "Firewalls and virtual networks", select "Enabled from selected virtual networks and IP addresses".
    *   Click "+ Add existing virtual network".
    *   Select your subscription.
    *   Select the `iatd_labs_01_vnet` Virtual Network.
    *   Select the `iatd_labs_01_subnet` Subnet.
    *   Click **Add**.
    *   Click **Save**.

### Part 4: Testing Connectivity (Conceptual)

*   **Deploy a VM to the Subnet:** Create a VM within the `iatd_labs_01_subnet` subnet.
*   **Attempt to Access Storage Account:** From within the VM, attempt to access the Storage Account (e.g., list blobs).
*   **Expected Result:** The access *should* be successful because the VM is in a subnet with a service endpoint enabled and the Storage Account allows access from that VNet/Subnet.
*   **If you were to deploy another VM to the VNET but a different subnet, and DID NOT enable service endpoint on that subnet, the access to the storage account would be denied**

### Post-Lab: Cleanup

To avoid incurring unnecessary costs, it's essential to clean up the resources created during this lab.

1.  **Remove Storage Account Network Rule:** Remove the virtual network rule from the storage account's firewall settings.
2.  **Delete Resource Group:**

    ```bash
    az group delete --name $RESOURCE_GROUP --yes
    ```

    **Expected Output:**
    ```
    The resource group deletion is in progress and will complete in the background.
    ```

3.  **Verify Resource Deletion:**

    ```bash
    az group show --name $RESOURCE_GROUP
    ```

    **Expected Output:**
    ```
    Resource group 'iatd_labs_01_rg' could not be found.
    ```

4.  **Clean Local Environment Variables:**

    ```bash
    unset RESOURCE_GROUP LOCATION VNET_NAME SUBNET_NAME ADDRESS_PREFIX SUBNET_PREFIX
    ```

**Outcomes to be Achieved:**

*   Successfully created an Azure Virtual Network and Subnet.
*   Successfully enabled a Service Endpoint for Azure Storage on a Subnet.
*   Successfully verified the route table update.
*   Successfully configured a Storage Account firewall to allow access from the VNet/Subnet.
*   Understand how Service Endpoints enhance security by limiting network access to Azure services.
