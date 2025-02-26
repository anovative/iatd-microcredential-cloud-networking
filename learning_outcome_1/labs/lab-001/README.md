# IATD Microcredential: Cloud Networking - Lab 1: Creating Azure Virtual Networks

## Overview

This lab guides you through creating Azure Virtual Networks (VNets) using four methods: the Azure Portal, Azure PowerShell, Azure CLI, and Azure Resource Manager (ARM) templates. After this lab, you’ll understand VNet creation and configuration. We'll use Azure Cloud Shell for a consistent experience.

## Objectives

*   Create a Virtual Network using the Azure Portal.
*   Create a Virtual Network using Azure PowerShell.
*   Create a Virtual Network using Azure CLI.
*   Create a Virtual Network using Azure Resource Manager (ARM) templates.
*   Verify the configurations of the created VNets.

## Duration

60 - 75 minutes

## Prerequisites

1.  **Azure Subscription:** An active Azure subscription is required. Create a free Azure account at [https://azure.microsoft.com/free/](https://azure.microsoft.com/free/) if needed.

## Lab Setup & Conventions

*   Use Azure Cloud Shell for PowerShell and CLI interactions.
*   All resource names follow the `iatd_labs_01_*` convention.
*   The `172.16.x.x` IP address range will be used for VNets and subnets.

### Naming Conventions

*   **Resource Group:** `iatd_labs_01_rg`
*   **Virtual Network (Portal):** `iatd_labs_01_vnet_portal`
*   **Subnet (Portal):** `iatd_labs_01_subnet_portal`
*   **Virtual Network (PowerShell):** `iatd_labs_01_vnet_ps`
*   **Subnet (PowerShell):** `iatd_labs_01_subnet_ps`
*   **Virtual Network (CLI):** `iatd_labs_01_vnet_cli`
*   **Subnet (CLI):** `iatd_labs_01_subnet_cli`
*   **Virtual Network (ARM):** `iatd_labs_01_vnet_arm`
*   **Subnet (ARM):** `iatd_labs_01_subnet_arm`

### General Instructions

1.  **Azure Portal Access:** Access the Azure Portal at [https://portal.azure.com/](https://portal.azure.com/) with your Azure account.
2.  **Cloud Shell Activation:** Activate Cloud Shell within the Azure Portal when PowerShell or CLI commands are required.

## Lab Execution

### Task 1: Creating a Virtual Network using the Azure Portal

1.  **Sign in to the Azure Portal:** Go to [https://portal.azure.com/](https://portal.azure.com/) and log in.
2.  **Create a Resource Group:**
    *   In the search bar at the top, type "Resource groups" and select it.
    *   Click **Create**.
    *   Choose your subscription.
    *   Enter `iatd_labs_01_rg` as the name.
    *   Select your desired region (e.g., East US).
    *   Click **Review + Create** then **Create**.
3.  **Create a Virtual Network:**
    *   In the search bar, type "Virtual networks" and select it.
    *   Click **Create**.
4.  **Configure the Virtual Network:**
    *   On the **Basics** tab:
        *   **Project details:**
            *   Select your subscription.
            *   Select `iatd_labs_01_rg` as the resource group.
        *   **Instance details:**
            *   Enter `iatd_labs_01_vnet_portal` as the name.
            *   Choose the same region as your resource group (e.g., East US).
    *   Go to the **IP Addresses** tab or click **Next: IP Addresses >**.
        *   Configure the IP address space:
            *   Change the default IPv4 address space to `172.16.0.0/16`.
        *   Configure the subnet:
            *   Select the default subnet.
            *   Change the subnet name to `iatd_labs_01_subnet_portal`.
            *   Change the subnet address range to `172.16.0.0/24`.
        *   Click **Save**.
    *   Click **Review + create** then **Create**.
5.  **Verify the VNet:** After deployment, navigate to the `iatd_labs_01_rg` resource group and find `iatd_labs_01_vnet_portal`. Click the VNet to review details, address space, and subnet configuration.

### Task 2: Creating a Virtual Network using Azure PowerShell

1.  **Launch Azure Cloud Shell:**
    *   In the Azure Portal, click the Cloud Shell icon in the top navigation bar.
    *   If prompted, select **PowerShell** as the shell environment.
    *   If this is the first time using Cloud Shell, create a storage account.

2.  **Set Variables:** In Cloud Shell, define the following variables:

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
    ```    *Example Output:*

    ```
    ResourceGroupName : iatd_labs_01_rg
    Location          : eastus
    ProvisioningState : Succeeded
    Tags              :
    ```

4.  **Create the Subnet Configuration:**
    ```powershell
    $subnet = New-AzVirtualNetworkSubnetConfig -Name $subnetName -AddressPrefix $subnetPrefix
    ```
    *Example Output:*
    ```
    Name             : iatd_labs_01_subnet_ps
    Id               : /subscriptions/.../resourceGroups/iatd_labs_01_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_01_vnet_ps/subnets/iatd_labs_01_subnet_ps
    AddressPrefix    : 172.16.1.0/24
    ...
    ```
5.  **Create the Virtual Network:**
    ```powershell
    New-AzVirtualNetwork -Name $vnetName -ResourceGroupName $resourceGroupName -Location $location -AddressPrefix $addressPrefix -Subnet $subnet
    ```
    *Example Output:*
    ```
    Name              : iatd_labs_01_vnet_ps
    ResourceGroupName : iatd_labs_01_rg
    Location          : eastus
    ...
    AddressSpace      : {172.16.1.0/16}
    ...
    Subnets           : [
                         {
                           "Name": "iatd_labs_01_subnet_ps",
                           "Id": "/subscriptions/.../resourceGroups/iatd_labs_01_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_01_vnet_ps/subnets/iatd_labs_01_subnet_ps",
                           ...
                           "AddressPrefix": "172.16.1.0/24"
                         }
                       ]
    ```

6.  **Verify the Virtual Network:**
    ```powershell
    Get-AzVirtualNetwork -Name $vnetName -ResourceGroupName $resourceGroupName
    ```
    Inspect the output to confirm VNet creation.

### Task 3: Creating a Virtual Network using Azure CLI

1.  **Launch Azure Cloud Shell:** Follow the same steps as in Task 2, but choose **Bash**.

2.  **Set Variables:** Define these variables in Cloud Shell:

    ```azurecli
    RESOURCE_GROUP="iatd_labs_01_rg"
    VNET_NAME="iatd_labs_01_vnet_cli"
    LOCATION="eastus" # Replace with your region
    ADDRESS_PREFIX="172.16.2.0/16"
    SUBNET_NAME="iatd_labs_01_subnet_cli"
    SUBNET_PREFIX="172.16.2.0/24"
    ```

3.  **Create Resource Group (if it doesn't exist):**
    ```azurecli
    az group create --name $RESOURCE_GROUP --location $LOCATION --output json --only-show-errors
    ```    *Example Output:*

    ```json
    {
      "id": "/subscriptions/.../resourceGroups/iatd_labs_01_rg",
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

4.  **Create the Virtual Network:**

    ```azurecli
    az network vnet create \
      --resource-group $RESOURCE_GROUP \
      --name $VNET_NAME \
      --address-prefixes $ADDRESS_PREFIX \
      --subnet-name $SUBNET_NAME \
      --subnet-prefixes $SUBNET_PREFIX \
      --location $LOCATION
    ```
    *Example Output:*

    ```json
    {
      "newVNet": {
        "addressSpace": {
          "addressPrefixes": [
            "172.16.2.0/16"
          ]
        },
        "dhcpOptions": {
          "dnsServers": []
        },
        "enableDdosProtection": false,
        "enableVmProtection": false,
        "id": "/subscriptions/.../resourceGroups/iatd_labs_01_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_01_vnet_cli",
        "location": "eastus",
        "name": "iatd_labs_01_vnet_cli",
        ...
    ```

5.  **Verify the Virtual Network:**

    ```azurecli
    az network vnet show --name $VNET_NAME --resource-group $RESOURCE_GROUP
    ```
    Review the output to confirm the VNet.

### Task 4: Creating a Virtual Network using Azure Resource Manager (ARM) Templates

1.  **Create ARM Template File:**
    *   In Cloud Shell, use the `code` command to create a new file:
        ```bash
        code vnet_template.json
        ```
        This opens the Cloud Shell editor.

    *   Paste the JSON code into `vnet_template.json`:

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

    *   Save the file (Ctrl+S or Cmd+S).

2.  **Deploy ARM Template:** Deploy the template using Azure CLI:

    ```azurecli
    RESOURCE_GROUP="iatd_labs_01_rg"
    TEMPLATE_FILE="vnet_template.json"

    az deployment group create \
      --resource-group $RESOURCE_GROUP \
      --template-file $TEMPLATE_FILE
    ```
    *Example Output:*

    ```json
    {
      "id": "/subscriptions/.../resourceGroups/iatd_labs_01_rg/providers/Microsoft.Resources/deployments/vnet_template",
      "location": "eastus",
      "name": "vnet_template",
      "properties": {
        "correlationId": "...",
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
        "providers": [
          {
            "id": null,
            "namespace": "Microsoft.Network",
            "providerAuthorizationConsent": null,
            "registrationPolicy": null,
            "resourceTypes": [
              {
                "aliases": null,
                "apiVersions": null,
                "capabilities": null,
                "defaultApiVersion": null,
                "locationRequired": true,
                "locations": null,
                "properties": null,
                "resourceType": "virtualNetworks",
                "zoneMappings": null
              }
            ]
          }
        ],
        "provisioningState": "Succeeded",
        "templateContent": {
          "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
          ...
    ```

3.  **Verify the Virtual Network:** Verify the VNet creation using the Azure Portal, Azure PowerShell, or Azure CLI.  For example, using Azure CLI:
    ```azurecli
    az network vnet show --name "iatd_labs_01_vnet_arm" --resource-group $RESOURCE_GROUP
    ```

## Post-Lab Tasks: Clean Up Resources

Delete the created resources to avoid costs.  It's best to delete the entire resource group.

1.  **Delete the Resource Group:**

    **Using Azure Portal:**

    *   Navigate to the `iatd_labs_01_rg` resource group.
    *   Click "Delete resource group."
    *   Confirm by typing the resource group name and clicking "Delete."

    **Using Azure PowerShell:**

    ```powershell
    Remove-AzResourceGroup -Name $resourceGroupName -Force
    ```

    **Using Azure CLI:**

    ```azurecli
    az group delete --name $RESOURCE_GROUP --yes
    ```

## Conclusion

You've created Azure Virtual Networks using the Azure Portal, Azure PowerShell, Azure CLI, and ARM Templates. You should understand how to provision and configure VNets using different approaches. Always clean up your Azure resources afterward. This completes Lab 1.
