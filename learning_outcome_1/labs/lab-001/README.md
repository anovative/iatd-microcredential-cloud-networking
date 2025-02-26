Okay, here's a refined and comprehensive lab guide focused on creating Virtual Networks (VNets) in Azure using various methods. This version prioritizes clarity, conciseness, and adheres to your requirements for using Cloud Shell, specifying IP ranges, and adhering to the naming convention.

# IATD Microcredential: Cloud Networking Lab 1 - Creating a Virtual Network

## Overview

This hands-on lab will guide you through the process of creating Azure Virtual Networks (VNets) using multiple methods: the Azure Portal, Azure PowerShell, Azure CLI, and Azure Resource Manager (ARM) Templates. VNets are fundamental to creating your private network in Azure, providing isolation and secure communication for your resources. Upon completing this lab, you will understand the different approaches for deploying and configuring VNets to suit various operational styles.

## Objectives

*   Create a VNet using the Azure Portal.
*   Create a VNet using Azure PowerShell (via Cloud Shell).
*   Create a VNet using Azure CLI (via Cloud Shell).
*   Create a VNet using Azure Resource Manager (ARM) Templates (deployed from Cloud Shell).
*   Verify the settings of all created VNets.

## Duration

Approximately 45 minutes - 1 hour.

## Prerequisites

1.  **Azure Subscription:** You need an active Azure subscription. If you don't have one, create a free account [here](https://azure.microsoft.com/free/).

### Step-by-Step Prerequisite Setup

#### 1. Azure Account Setup

1.  **Navigate to Azure Portal:** Open your web browser and go to [https://portal.azure.com/](https://portal.azure.com/).
2.  **Sign In:** Sign in using your Azure account credentials. If you do not have one, follow the on-screen prompts to create a free one.

## Lab Execution

### Task 1: Creating a VNet using the Azure Portal

1.  **Navigate to Azure Portal:** Open your web browser and go to [https://portal.azure.com/](https://portal.azure.com/).

2.  **Sign In:** Authenticate using your Azure account credentials.

3.  **Create a Resource:**

    *   Click "+ Create a resource" in the top-left corner.
    *   Search for "Virtual network" and select "Virtual network" (published by Microsoft).
    *   Click "Create".

4.  **Configure the VNet:** On the "Create virtual network" page:

    *   **Subscription:** Choose your Azure subscription.
    *   **Resource group:** Create a new resource group named `iatd_labs_01`.
    *   **Name:** Enter `iatd_labs_01_vnet_portal`.
    *   **Region:** Choose a region appropriate for you (e.g., East US).
    *   **IP Addresses Tab**:
        *  Click on the IP Addresses tab.
        *  Click on the "+ Add subnet" button
        *   **IPv4 address space:** Change to  `172.16.0.0/16`.
        *   **Subnet name:** `iatd_labs_01_subnet_portal`.
        *   **Subnet address range:** `172.16.0.0/24`.

    *   **Security:** Leave the default security settings.
    *   Click "Review + create".

5.  **Review and Create:** Verify your settings, then click "Create."  Wait for the VNet to be deployed.

6.  **Verify the VNet:**
    Once deployed, go to the `iatd_labs_01` resource group and select the `iatd_labs_01_vnet_portal` VNet. Examine its details, specifically the address space and subnet configuration.

### Task 2: Creating a VNet using Azure PowerShell

1.  **Launch Cloud Shell:**

    *   Click the "Cloud Shell" icon (usually at the top of the Azure Portal).
    *   If prompted, choose "PowerShell."
    *   First-time users may need to create a storage account for Cloud Shell.  Follow the instructions to create one.

2.  **Set Variables:** Define the variables for the VNet configuration:

    ```powershell
    $resourceGroupName = "iatd_labs_01"
    $vnetName = "iatd_labs_01_vnet_ps"
    $location = "EastUS"
    $addressPrefix = "172.16.1.0/16"
    $subnetName = "iatd_labs_01_subnet_ps"
    $subnetPrefix = "172.16.1.0/24"
    ```

3.  **Create Resource Group:** (If it doesn't already exist from Task 1):

    ```powershell
    New-AzResourceGroup -Name $resourceGroupName -Location $location
    ```

    *Sample Output:*

    ```
    ResourceGroupName : iatd_labs_01
    Location          : eastus
    ProvisioningState : Succeeded
    Tags              :
    ```

4.  **Create Subnet Configuration:**

    ```powershell
    $subnet = New-AzVirtualNetworkSubnetConfig -Name $subnetName -AddressPrefix $subnetPrefix
    ```

    *Sample Output:*

    ```
    Name              : iatd_labs_01_subnet_ps
    Id                : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/iatd_labs_01/providers/Microsoft.Network/virtualNetworks/iatd_labs_01_vnet_ps/subnets/iatd_labs_01_subnet_ps
    AddressPrefix     : 172.16.1.0/24
    ...
    ```

5.  **Create Virtual Network:**

    ```powershell
    New-AzVirtualNetwork -Name $vnetName -ResourceGroupName $resourceGroupName -Location $location -AddressPrefix $addressPrefix -Subnet $subnet
    ```

    *Sample Output:*

    ```
    Name              : iatd_labs_01_vnet_ps
    ResourceGroupName : iatd_labs_01
    Location          : eastus
    ...
    AddressSpace      : {172.16.1.0/16}
    ...
    ```

6.  **Verify the VNet:** Retrieve and inspect the VNet's configuration.

    ```powershell
    Get-AzVirtualNetwork -Name $vnetName -ResourceGroupName $resourceGroupName
    ```

    Confirm the VNet parameters match the ones you defined.

### Task 3: Creating a VNet using Azure CLI

1.  **Launch Cloud Shell:**

    *   Click the "Cloud Shell" icon in the Azure portal toolbar.
    *   Select "Bash" if prompted.
    *   Set up a storage account if this is your first time using Cloud Shell.

2.  **Set Variables:** Define variables for the VNet creation.

    ```azurecli
    RESOURCE_GROUP="iatd_labs_01"
    VNET_NAME="iatd_labs_01_vnet_cli"
    LOCATION="eastus"
    ADDRESS_PREFIX="172.16.2.0/16"
    SUBNET_NAME="iatd_labs_01_subnet_cli"
    SUBNET_PREFIX="172.16.2.0/24"
    ```

3.  **Create Resource Group:**  (If it doesn't exist yet)

    ```azurecli
    az group create --name $RESOURCE_GROUP --location $LOCATION
    ```

    *Sample Output:*

    ```json
    {
      "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/iatd_labs_01",
      "location": "eastus",
      "managedBy": null,
      "name": "iatd_labs_01",
      "properties": {
        "provisioningState": "Succeeded"
      },
      "tags": null,
      "type": "Microsoft.Resources/resourceGroups"
    }
    ```

4.  **Create Virtual Network:**

    ```azurecli
    az network vnet create \
      --resource-group $RESOURCE_GROUP \
      --name $VNET_NAME \
      --address-prefixes $ADDRESS_PREFIX \
      --subnet-name $SUBNET_NAME \
      --subnet-prefixes $SUBNET_PREFIX \
      --location $LOCATION
    ```

    *Sample Output:*

    ```json
    {
      "newVNet": {
        "addressSpace": {
          "addressPrefixes": [
            "172.16.2.0/16"
          ]
        },
        "dhcpOptions": {
          "dnsServers": [],
        },
        "enableDdosProtection": false,
        "enableVmProtection": false,
        "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/iatd_labs_01/providers/Microsoft.Network/virtualNetworks/iatd_labs_01_vnet_cli",
        "location": "eastus",
        "name": "iatd_labs_01_vnet_cli",
        ...
    ```

5.  **Verify the VNet:**

    ```azurecli
    az network vnet show --name $VNET_NAME --resource-group $RESOURCE_GROUP
    ```

    Review the output to validate the VNet configuration.

### Task 4: Creating a VNet using Azure Resource Manager (ARM) Templates

1.  **Create ARM Template:**  Create a file named `vnet_template.json` directly in Cloud Shell's editor. Use the command `code vnet_template.json` to open the editor.

2.  **Paste the Following JSON:**

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

3.  **Deploy ARM Template:** Deploy the template using Azure CLI within Cloud Shell:

    ```azurecli
    RESOURCE_GROUP="iatd_labs_01"
    TEMPLATE_FILE="vnet_template.json"

    az deployment group create \
      --resource-group $RESOURCE_GROUP \
      --template-file $TEMPLATE_FILE
    ```

    *Sample Output:*

    ```json
    {
      "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/iatd_labs_01/providers/Microsoft.Resources/deployments/vnet_template",
      "location": "eastus",
      "name": "vnet_template",
      "properties": {
        "correlationId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "dependencies": [],
        "outputs": {},
        "parameters": {
          "location": {
            "type": "String",
            "value": "eastus"
          },
          "subnetName": {
            "type": "String",
            "value": "iatd_labs_01_subnet_arm"
          },
          "subnetPrefix": {
            "type": "String",
            "value": "172.16.3.0/24"
          },
          "vnetAddressPrefix": {
            "type": "String",
            "value": "172.16.3.0/16"
          },
          "vnetName": {
            "type": "String",
            "value": "iatd_labs_01_vnet_arm"
          }
        },
        "parametersLink": null,
        "providers": [ ... ],
        "provisioningState": "Succeeded",
        "templateContent": { ... }
    }
    ```

4.  **Verify the VNet:** Confirm creation and settings by using the Azure Portal, Azure PowerShell, or Azure CLI to view the deployed VNet (named `iatd_labs_01_vnet_arm`).  For example, with Azure CLI:

    ```azurecli
    az network vnet show --name "iatd_labs_01_vnet_arm" --resource-group $RESOURCE_GROUP
    ```

## Post-Lab Tasks

1.  **Clean up Resources:** Delete the resource group `iatd_labs_01` to avoid unnecessary costs. Use one of the following methods:

    **Azure Portal:**

    *   Go to the `iatd_labs_01` resource group.
    *   Click "Delete resource group".
    *   Enter the resource group name to confirm and click "Delete".

    **Azure PowerShell:**

    ```powershell
    Remove-AzResourceGroup -Name $resourceGroupName -Force
    ```

    **Azure CLI:**

    ```azurecli
    az group delete --name $RESOURCE_GROUP --yes
    ```

## Conclusion

You have now successfully created Azure Virtual Networks using the Azure Portal, Azure PowerShell, Azure CLI, and ARM Templates. Each method provides a different balance of control and automation. Understanding when and how to leverage these tools is vital to effectively manage your Azure infrastructure. Remember to deallocate or remove resources that are no longer needed.
