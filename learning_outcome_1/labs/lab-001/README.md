# IATD Microcredential: Cloud Networking - Lab 1: Creating Azure Virtual Networks (VNets)

## Overview

This hands-on lab provides step-by-step instructions on how to create Azure Virtual Networks (VNets) using four different methods: the Azure Portal, Azure PowerShell, Azure CLI, and Azure Resource Manager (ARM) templates. By the end of this lab, you'll gain a practical understanding of VNet deployment and configuration in Azure. We'll utilize Azure Cloud Shell for a streamlined and consistent experience with both PowerShell and CLI.

## Objectives

Upon completion of this lab, you will be able to:

*   Create a VNet using the Azure Portal.
*   Create a VNet using Azure PowerShell (within Cloud Shell).
*   Create a VNet using Azure CLI (within Cloud Shell).
*   Create a VNet using Azure Resource Manager (ARM) templates (deployed via Cloud Shell).
*   Verify and validate the configuration of each VNet created.

## Estimated Duration

60-75 minutes

## Prerequisites

1.  **Azure Subscription:** You'll need an active Azure subscription. If you don't already have one, you can create a free Azure account at [https://azure.microsoft.com/free/](https://azure.microsoft.com/free/).

## Lab Setup and Conventions

*   This lab is designed to be executed primarily within the Azure Cloud Shell for both PowerShell and CLI tasks. This ensures a consistent and pre-configured environment.
*   All resource names will adhere to the `iatd_labs_01_*` naming convention to maintain organization and clarity.
*   We will consistently use the `172.16.x.x` IP address range for our VNets and subnets throughout the lab. Each VNet created with a different method will use a unique subnet within this range.

### Naming Convention Summary

| Resource                       | Naming Convention          |
| ------------------------------ | -------------------------- |
| **Resource Group**            | `iatd_labs_01_rg`           |
| **Virtual Network (Portal)**    | `iatd_labs_01_vnet_portal` |
| **Subnet (Portal)**           | `iatd_labs_01_subnet_portal` |
| **Virtual Network (PowerShell)** | `iatd_labs_01_vnet_ps`     |
| **Subnet (PowerShell)**        | `iatd_labs_01_subnet_ps`    |
| **Virtual Network (CLI)**       | `iatd_labs_01_vnet_cli`      |
| **Subnet (CLI)**              | `iatd_labs_01_subnet_cli`     |
| **Virtual Network (ARM)**       | `iatd_labs_01_vnet_arm`      |
| **Subnet (ARM)**              | `iatd_labs_01_subnet_arm`     |

### Key Instructions

1.  **Azure Portal Access:** Ensure you have access to the Azure Portal at [https://portal.azure.com/](https://portal.azure.com/) with a valid Azure account.
2.  **Cloud Shell Activation:** Launch Azure Cloud Shell directly from the Azure Portal whenever you need to execute PowerShell or CLI commands.

## Lab Execution

### Task 1: Creating a Virtual Network using the Azure Portal

This task will guide you through creating a VNet using the Azure Portal's graphical user interface.

1.  **Sign in to the Azure Portal:** Open your web browser and navigate to [https://portal.azure.com/](https://portal.azure.com/). Sign in using your Azure account credentials.
2.  **Create a Resource Group:**
    *   Use the search bar at the top to find and select "Resource groups".
    *   Click on **Create**.
    *   Select your Azure subscription from the dropdown menu.
    *   Enter `iatd_labs_01_rg` as the name for the resource group.
    *   Choose an appropriate region (e.g., East US) from the Location dropdown.
    *   Click **Review + Create**, and then click **Create** to deploy the resource group.
3.  **Create a Virtual Network:**
    *   Use the search bar at the top to find and select "Virtual networks."
    *   Click on **Create**.
4.  **Configure the Virtual Network:**
    *   On the **Basics** tab:
        *   **Project details:**
            *   Select your Azure subscription.
            *   Choose `iatd_labs_01_rg` as the resource group.
        *   **Instance details:**
            *   Enter `iatd_labs_01_vnet_portal` as the virtual network name.
            *   Select the same region you selected for the resource group (e.g., East US).
    *   Navigate to the **IP Addresses** tab (or click **Next: IP Addresses >**).
        *   Configure the IPv4 address space:
            *   Change the default IPv4 address space to `172.16.0.0/16`.
        *   Configure the subnet:
            *   Select the existing "default" subnet.
            *   In the "Edit subnet" pane:
                *   Change the Subnet name to `iatd_labs_01_subnet_portal`.
                *   Modify the Subnet address range to `172.16.0.0/24`.
            *   Click **Save**.
    *   Click **Review + create** and then **Create**.
5.  **Verify the VNet Deployment:** After the deployment completes, navigate to the `iatd_labs_01_rg` resource group. You should find the `iatd_labs_01_vnet_portal` virtual network listed. Click on the virtual network to explore its settings, including the configured address space and subnet.

### Task 2: Creating a Virtual Network using Azure PowerShell

This task demonstrates how to create a VNet using Azure PowerShell commands within the Azure Cloud Shell.

1.  **Launch Azure Cloud Shell:**
    *   In the Azure Portal, locate and click the Cloud Shell icon in the top navigation bar.
    *   When prompted, select **PowerShell** as your preferred shell environment.
    *   If this is your first time using Cloud Shell, you might be asked to create a storage account. Follow the prompts to complete the setup.
2.  **Declare Variables:** Within the Cloud Shell, define the following variables. These will be used in subsequent commands:

    ```powershell
    $resourceGroupName = "iatd_labs_01_rg"
    $vnetName = "iatd_labs_01_vnet_ps"
    $location = "EastUS"  # Replace with your Azure region if different
    $addressPrefix = "172.16.1.0/16"
    $subnetName = "iatd_labs_01_subnet_ps"
    $subnetPrefix = "172.16.1.0/24"
    ```

3.  **Create the Resource Group (Conditionally):** The following command will create the resource group only if it does not already exist.
    ```powershell
    New-AzResourceGroup -Name $resourceGroupName -Location $location -ErrorAction SilentlyContinue
    ```
    *Example Output (if the resource group is created):*

    ```
    ResourceGroupName : iatd_labs_01_rg
    Location          : eastus
    ProvisioningState : Succeeded
    Tags              :
    ```

4.  **Define the Subnet Configuration:** Use the `New-AzVirtualNetworkSubnetConfig` command to define the configuration for the subnet:

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
5.  **Create the Virtual Network:** Now, use the `New-AzVirtualNetwork` command to create the virtual network itself, referencing the subnet configuration we defined above:

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

6.  **Verification:** To verify the successful creation of the VNet, use the `Get-AzVirtualNetwork` command:

    ```powershell
    Get-AzVirtualNetwork -Name $vnetName -ResourceGroupName $resourceGroupName
    ```

    Inspect the output to ensure the virtual network has been created and that its configuration matches the parameters you defined.

### Task 3: Creating a Virtual Network using Azure CLI

In this task, you will create a virtual network using Azure CLI commands in the Cloud Shell.

1.  **Launch Azure Cloud Shell:** Follow the same steps as in Task 2, but choose **Bash** as your shell environment instead of PowerShell.

2.  **Define Variables:** Set the following variables in the Cloud Shell:

    ```azurecli
    RESOURCE_GROUP="iatd_labs_01_rg"
    VNET_NAME="iatd_labs_01_vnet_cli"
    LOCATION="eastus" # Replace with your Azure region if different
    ADDRESS_PREFIX="172.16.2.0/16"
    SUBNET_NAME="iatd_labs_01_subnet_cli"
    SUBNET_PREFIX="172.16.2.0/24"
    ```

3.  **Create Resource Group (Conditionally):**
    The below command will only create the resource group if it does not already exist

    ```azurecli
    az group create --name $RESOURCE_GROUP --location $LOCATION --output json --only-show-errors
    ```

    *Example Output (if resource group created):*

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

4.  **Create the Virtual Network:** Use the `az network vnet create` command to create the virtual network:

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
          "dnsServers": [],
        ...
        "id": "/subscriptions/.../resourceGroups/iatd_labs_01_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_01_vnet_cli",
        "location": "eastus",
        "name": "iatd_labs_01_vnet_cli",
        ...
    ```

5.  **Verification:** Use the `az network vnet show` command to verify that the virtual network has been successfully created and configured:

    ```azurecli
    az network vnet show --name $VNET_NAME --resource-group $RESOURCE_GROUP
    ```

    Review the output to confirm the virtual network's properties.

### Task 4: Creating a Virtual Network using Azure Resource Manager (ARM) Templates

This task guides you through creating and deploying an ARM template to define your VNet infrastructure as code.

1.  **Create the ARM Template File:**
    *   In the Cloud Shell, use the `code` command to create a new file named `vnet_template.json`:

        ```bash
        code vnet_template.json
        ```

        This will open the built-in Cloud Shell editor.
    *   Paste the following JSON code into the `vnet_template.json` file. This template defines a VNet with a single subnet:

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
    *   Save the `vnet_template.json` file (using Ctrl+S or Cmd+S).

2.  **Deploy the ARM Template:** Use the following Azure CLI command to deploy the ARM template.  Ensure you are still in the Cloud Shell (Bash).

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
        "providers": [ ... ],
        "provisioningState": "Succeeded",
        "templateContent": { ... }
    ```

3.  **Verification:** After the deployment is complete, verify the creation of the virtual network using one of the following methods:

    *   **Azure Portal:** Navigate to the `iatd_labs_01_rg` resource group and find the `iatd_labs_01_vnet_arm` virtual network.
    *   **Azure PowerShell:** Use the `Get-AzVirtualNetwork` command as shown in Task 2, replacing the VNet name with `iatd_labs_01_vnet_arm`.
    *   **Azure CLI:** Use the `az network vnet show` command as shown in Task 3, replacing the VNet name with `iatd_labs_01_vnet_arm`.

## Post-Lab Tasks: Clean Up Resources

To prevent incurring unnecessary Azure costs, it is crucial to delete the resources you created during this lab. The easiest way to do this is to delete the entire resource group, which will remove all resources contained within it.

1.  **Delete the Resource Group:** Choose **one** of the following methods to delete the `iatd_labs_01_rg` resource group:

    *   **Using the Azure Portal:**
        *   Navigate to the `iatd_labs_01_rg` resource group in the Azure Portal.
        *   Click on "Delete resource group" in the top menu.
        *   Confirm the deletion by typing the resource group name and clicking "Delete".
    *   **Using Azure PowerShell:** In the Cloud Shell (PowerShell), run the following command:
        ```powershell
        Remove-AzResourceGroup -Name $resourceGroupName -Force
        ```
    *   **Using Azure CLI:** In the Cloud Shell (Bash), run the following command:
        ```azurecli
        az group delete --name $RESOURCE_GROUP --yes
        ```

## Conclusion

Congratulations! You have successfully completed Lab 1 and gained hands-on experience creating Azure Virtual Networks using the Azure Portal, Azure PowerShell, Azure CLI, and ARM templates. This lab has provided you with a foundational understanding of VNet deployment, configuration, and the different methods available to achieve this in Azure. Always remember to clean up your resources to avoid unexpected charges!
