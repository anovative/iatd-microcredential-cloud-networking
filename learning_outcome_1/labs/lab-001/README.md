## IATD Microcredential Cloud Networking: Lab 1 - Virtual Network Creation

**Objective:** This lab guides you through the process of creating an Azure Virtual Network (VNet), the core building block for your private network. You'll learn to create VNets using the Azure Portal, Azure PowerShell, Azure CLI, and ARM templates.

**Estimated Time:** 45 - 60 minutes

**Prerequisites:**

1.  **Azure Subscription:** Requires an active Azure subscription. A free account is available at [https://azure.microsoft.com/free/](https://azure.microsoft.com/free/).

**Let's get started!**

### Lab Conventions

*   **Azure Cloud Shell:** All PowerShell and CLI interactions will occur within the Azure Cloud Shell environment.
*   **Naming Conventions:** Resource names follow the convention `iatd_labs_01_*`.
*   **IP Address Range:** The `172.16.0.0/16` range will be used (as per standardization).
*   **Location:** Choose a consistent Azure region.

#### Resource Naming

*   **Resource Group:** `iatd_labs_01_rg`
*   **Virtual Network (Portal):** `iatd_labs_01_vnet_portal`
*   **Subnet (Portal):** `iatd_labs_01_subnet_portal`
*   **Virtual Network (PowerShell):** `iatd_labs_01_vnet_ps`
*   **Subnet (PowerShell):** `iatd_labs_01_subnet_ps`
*   **Virtual Network (CLI):** `iatd_labs_01_vnet_cli`
*   **Subnet (CLI):** `iatd_labs_01_subnet_cli`
*   **Virtual Network (ARM):** `iatd_labs_01_vnet_arm`
*   **Subnet (ARM):** `iatd_labs_01_subnet_arm`

### Part 1: VNet Creation - Azure Portal

1.  **Sign in to the Azure Portal:** Browse to [https://portal.azure.com/](https://portal.azure.com/) and sign in.
2.  **Create Resource Group:**
    *   Search for "Resource groups" and select.
    *   Click **Create**.
    *   Select your subscription.
    *   Name: `iatd_labs_01_rg`
    *   Region: Select your region.
    *   Click **Review + create**, then **Create**.
3.  **Create Virtual Network:**
    *   Search for "Virtual networks" and select.
    *   Click **Create**.
    *   **Basics Tab:**
        *   Subscription: Your subscription.
        *   Resource group: `iatd_labs_01_rg`
        *   Name: `iatd_labs_01_vnet_portal`
        *   Region: Your region.
    *   **IP Addresses Tab:**
        *   IPv4 address space: `172.16.0.0/16`
        *   Subnet name: `iatd_labs_01_subnet_portal`
        *   Subnet address range: `172.16.0.0/24`
    *   Click **Review + create**, then **Create**.
4.  **Verify:** Go to the `iatd_labs_01_rg` resource group and select the `iatd_labs_01_vnet_portal` Virtual Network to see the settings

### Part 2: VNet Creation - Azure PowerShell

1.  **Open Cloud Shell:** In the Azure Portal, click the Cloud Shell icon. Select PowerShell if prompted.

2.  **Define Variables:**

    ```powershell
    $resourceGroupName = "iatd_labs_01_rg"
    $vnetName = "iatd_labs_01_vnet_ps"
    $location = "EastUS"  # Replace with your region
    $addressPrefix = "172.16.1.0/16"
    $subnetName = "iatd_labs_01_subnet_ps"
    $subnetPrefix = "172.16.1.0/24"
    ```

3.  **Create Resource Group (if it doesn't exist):**

    ```powershell
    New-AzResourceGroup -Name $resourceGroupName -Location $location -ErrorAction SilentlyContinue
    ```

4.  **Create Subnet Configuration:**

    ```powershell
    $subnet = New-AzVirtualNetworkSubnetConfig -Name $subnetName -AddressPrefix $subnetPrefix
    ```

5.  **Create Virtual Network:**

    ```powershell
    New-AzVirtualNetwork -Name $vnetName -ResourceGroupName $resourceGroupName -Location $location -AddressPrefix $addressPrefix -Subnet $subnet
    ```

6.  **Verify:**

    ```powershell
    Get-AzVirtualNetwork -Name $vnetName -ResourceGroupName $resourceGroupName
    ```
    
    **Example Output (truncated):**
    
    ```
    Name              : iatd_labs_01_vnet_ps
    ResourceGroupName : iatd_labs_01_rg
    Location          : eastus
    Id                : /subscriptions/your_subscription_id/resourceGroups/iatd_labs_01_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_01_vnet_ps
    Etag              : W/"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
    ResourceGuid      : yyyyyyyy-yyyy-yyyy-yyyy-yyyyyyyyyyyy
    ProvisioningState : Succeeded
    ...
    Subnets           : [
                           {
                             "Name": "iatd_labs_01_subnet_ps",
                             "Etag": "W/\"zzzzzzzz-zzzz-zzzz-zzzz-zzzzzzzzzzzz\"",
                             "Id": "/subscriptions/your_subscription_id/resourceGroups/iatd_labs_01_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_01_vnet_ps/subnets/iatd_labs_01_subnet_ps",
                             "AddressPrefix": "172.16.1.0/24",
                             ...
                           }
                         ]
    ...
    ```

### Part 3: VNet Creation - Azure CLI

1.  **Open Cloud Shell:** Open Cloud Shell in the Azure Portal. Choose **Bash** this time.

2.  **Set Variables:**

    ```bash
    RESOURCE_GROUP="iatd_labs_01_rg"
    VNET_NAME="iatd_labs_01_vnet_cli"
    LOCATION="eastus" # Your Region
    ADDRESS_PREFIX="172.16.0.0/16"
    SUBNET_NAME="iatd_labs_01_subnet_cli"
    SUBNET_PREFIX="172.16.2.0/24"
    ```

3.  **Create Resource Group (if it doesn't exist):**

    ```bash
    az group create --name $RESOURCE_GROUP --location $LOCATION --output json --only-show-errors
    ```

4.  **Create Virtual Network:**

    ```bash
    az network vnet create \
      --resource-group $RESOURCE_GROUP \
      --name $VNET_NAME \
      --address-prefixes $ADDRESS_PREFIX \
      --subnet-name $SUBNET_NAME \
      --subnet-prefixes $SUBNET_PREFIX \
      --location $LOCATION
    ```

5.  **Verify:**

    ```bash
    az network vnet show --name $VNET_NAME --resource-group $RESOURCE_GROUP
    ```
    
    **Example Output:**
    
    ```json
    {
      "addressSpace": {
        "addressPrefixes": [
          "172.16.2.0/16"
        ]
      },
      "id": "/subscriptions/your_subscription_id/resourceGroups/iatd_labs_01_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_01_vnet_cli",
      "name": "iatd_labs_01_vnet_cli",
      "provisioningState": "Succeeded",
      "resourceGroup": "iatd_labs_01_rg",
      "subnets": [
        {
          "addressPrefix": "172.16.2.0/24",
          "id": "/subscriptions/your_subscription_id/resourceGroups/iatd_labs_01_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_01_vnet_cli/subnets/iatd_labs_01_subnet_cli",
          "name": "iatd_labs_01_subnet_cli",
          "provisioningState": "Succeeded"
        }
      ]
    }
    ```

### Part 4: VNet Creation - ARM Template

1.  **Create ARM Template File:**

    *   In Cloud Shell (Bash), create a file named `vnet_template.json`:
        ```bash
        code vnet_template.json
        ```

    *   Paste in the following JSON:

        ```json
        {
          "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
          "contentVersion": "1.0.0.0",
          "parameters": {
            "vnetName": {
              "type": "string",
              "defaultValue": "iatd_labs_01_vnet_arm",
              "metadata": {
                "description": "Virtual Network Name"
              }
            },
            "vnetAddressPrefix": {
              "type": "string",
              "defaultValue": "172.16.3.0/16",
              "metadata": {
                "description": "Virtual Network Address Prefix"
              }
            },
            "subnetName": {
              "type": "string",
              "defaultValue": "iatd_labs_01_subnet_arm",
              "metadata": {
                "description": "Subnet Name"
              }
            },
            "subnetPrefix": {
              "type": "string",
              "defaultValue": "172.16.3.0/24",
              "metadata": {
                "description": "Subnet Address Prefix"
              }
            },
            "location": {
              "type": "string",
              "defaultValue": "eastus",
              "metadata": {
                "description": "Location for all resources."
              }
            }
          },
          "variables": {},
          "resources": [
            {
              "type": "Microsoft.Network/virtualNetworks",
              "apiVersion": "2020-06-01",
              "name": "[parameters('vnetName')]",
              "location": "[parameters('location')]",
              "properties": {
                "addressSpace": {
                  "addressPrefixes": [
                    "[parameters('vnetAddressPrefix')]"
                  ]
                },
                "subnets": [
                  {
                    "name": "[parameters('subnetName')]",
                    "properties": {
                      "addressPrefix": "[parameters('subnetPrefix')]"
                    }
                  }
                ]
              }
            }
          ]
        }
        ```

    *   Save the file.

2.  **Deploy ARM Template:**

    ```bash
    RESOURCE_GROUP="iatd_labs_01_rg"
    TEMPLATE_FILE="vnet_template.json"

    az deployment group create \
      --resource-group $RESOURCE_GROUP \
      --template-file $TEMPLATE_FILE
    ```

3.  **Verify:**

    ```bash
    az network vnet show --name "iatd_labs_01_vnet_arm" --resource-group $RESOURCE_GROUP
    ```

### Post-Lab: Cleanup

To avoid incurring unnecessary costs, it's essential to clean up the resources created during this lab.

1.  **Delete Resource Group:**

    *   **Azure Portal:** Locate the Resource Group and Delete from the portal.
    
    *   **PowerShell:**
    
        ```powershell
        Remove-AzResourceGroup -Name "iatd_labs_01_rg" -Force
        ```
        
    *   **CLI:**
    
        ```bash
        az group delete --name iatd_labs_01_rg --yes
        ```

**Congratulations!** You have now successfully created Azure Virtual Networks via the Azure Portal, Azure PowerShell, Azure CLI, and using ARM Templates. You've laid a key foundation in Azure Networking!
