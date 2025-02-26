# IATD Microcredential: Cloud Networking - Lab 1: Virtual Network Creation

## Overview

This lab guides you through the process of creating an Azure Virtual Network (VNet), the core building block for your private network. You'll learn to create VNets using the Azure Portal, Azure PowerShell, Azure CLI, and ARM templates.

## Objectives

*   Create a VNet using the Azure Portal.
*   Create a VNet using Azure PowerShell (via Cloud Shell).
*   Create a VNet using Azure CLI (via Cloud Shell).
*   Create a VNet using an ARM template (deployed through Cloud Shell).
*   Verify VNet configuration using the Azure Portal.

## Duration

Estimated Completion Time: 45 - 60 minutes

## Prerequisites

1.  **Azure Subscription:** Requires an active Azure subscription. A free account is available at [https://azure.microsoft.com/free/](https://azure.microsoft.com/free/).

## Lab Conventions

*   **Azure Cloud Shell:** All PowerShell and CLI interactions will occur within the Azure Cloud Shell environment.
*   **Naming Conventions:** Resource names follow the convention `iatd_labs_01_*`.
*   **IP Address Range:** The `172.16.x.x` range will be used.
*   **Location:** Choose a consistent Azure region.

### Resource Naming

*   **Resource Group:** `iatd_labs_01_rg`
*   **Virtual Network (Portal):** `iatd_labs_01_vnet_portal`
*   **Subnet (Portal):** `iatd_labs_01_subnet_portal`
*   **Virtual Network (PowerShell):** `iatd_labs_01_vnet_ps`
*   **Subnet (PowerShell):** `iatd_labs_01_subnet_ps`
*   **Virtual Network (CLI):** `iatd_labs_01_vnet_cli`
*   **Subnet (CLI):** `iatd_labs_01_subnet_cli`
*   **Virtual Network (ARM):** `iatd_labs_01_vnet_arm`
*   **Subnet (ARM):** `iatd_labs_01_subnet_arm`

## Lab Steps

### Task 1: VNet Creation - Azure Portal

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

### Task 2: VNet Creation - Azure PowerShell

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
    Review output to confirm the correct VNet settings.

### Task 3: VNet Creation - Azure CLI

1.  **Open Cloud Shell:** Open Cloud Shell in the Azure Portal. Choose **Bash** this time.

2.  **Set Variables:**

    ```azurecli
    RESOURCE_GROUP="iatd_labs_01_rg"
    VNET_NAME="iatd_labs_01_vnet_cli"
    LOCATION="eastus" # Your Region
    ADDRESS_PREFIX="172.16.2.0/16"
    SUBNET_NAME="iatd_labs_01_subnet_cli"
    SUBNET_PREFIX="172.16.2.0/24"
    ```

3.  **Create Resource Group (if it doesn't exist):**

    ```azurecli
    az group create --name $RESOURCE_GROUP --location $LOCATION --output json --only-show-errors
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

5.  **Verify:**

    ```azurecli
    az network vnet show --name $VNET_NAME --resource-group $RESOURCE_GROUP
    ```

### Task 4: VNet Creation - ARM Template

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

    ```azurecli
    RESOURCE_GROUP="iatd_labs_01_rg"
    TEMPLATE_FILE="vnet_template.json"

    az deployment group create \
      --resource-group $RESOURCE_GROUP \
      --template-file $TEMPLATE_FILE
    ```

3.  **Verify:**

    ```azurecli
    az network vnet show --name "iatd_labs_01_vnet_arm" --resource-group $RESOURCE_GROUP
    ```

## Post-Lab Clean Up

Delete the `iatd_labs_01_rg` resource group to avoid incurring unwanted costs.

*   **Azure Portal:** Locate the Resource Group and Delete from the portal..
*   **PowerShell:** `Remove-AzResourceGroup -Name "iatd_labs_01_rg" -Force`
*   **CLI:** `az group delete --name iatd_labs_01_rg --yes`

## Conclusion

You have now successfully created Azure Virtual Networks via the Azure Portal, Azure PowerShell, Azure CLI, and using ARM Templates. Congratulations - you've laid a key foundation in Azure Networking! Remember to clean up to avoid incurring any testcharges.
