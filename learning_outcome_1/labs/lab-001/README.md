Okay, here's a revised, comprehensive lab guide for creating Azure Virtual Networks, covering the Portal, PowerShell, CLI, and ARM templates. It’s designed to be text-based, leverage Cloud Shell, maintain consistent naming, use specific IP ranges, and offer clear instructions.

# IATD Microcredential: Cloud Networking – Lab 1: Creating Virtual Networks in Azure

## Overview

This lab provides a hands-on experience in creating Azure Virtual Networks (VNets) using four different methods: the Azure Portal, Azure PowerShell, Azure CLI, and Azure Resource Manager (ARM) templates.  You'll gain practical knowledge of VNet deployment and configuration, equipping you to choose the right method for your specific needs. We’ll be using Azure Cloud Shell throughout this lab for PowerShell and CLI operations.

## Objectives

*   Create an Azure Virtual Network using the Azure Portal.
*   Create an Azure Virtual Network using Azure PowerShell within Cloud Shell.
*   Create an Azure Virtual Network using Azure CLI within Cloud Shell.
*   Create an Azure Virtual Network using an Azure Resource Manager (ARM) template deployed from Cloud Shell.
*   Validate the configuration of the created Virtual Networks.

## Duration

Approximately 60-75 minutes

## Prerequisites

1.  **Azure Subscription:** You must have an active Azure subscription. If you don't have one, create a free Azure account at [https://azure.microsoft.com/free/](https://azure.microsoft.com/free/).

## Lab Setup and Conventions

*   We will primarily use the Azure Cloud Shell for interacting with Azure using PowerShell and CLI.
*   All resource names will adhere to the `iatd_labs_01_*` naming convention for easy identification and management.
*   The IP address range `172.16.x.x` will be used for the VNet and subnet address spaces.  Each method will use a unique `x` value.

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

1.  **Azure Portal Access:** Make sure you can access the Azure Portal by navigating to [https://portal.azure.com/](https://portal.azure.com/) and logging in with your Azure account credentials.
2.  **Azure Cloud Shell:** Launch the Azure Cloud Shell directly from within the Azure Portal whenever you are instructed to execute PowerShell or CLI commands.

## Lab Execution

### Task 1: Creating a Virtual Network using the Azure Portal

In this task, you will create a Virtual Network through the Azure portal’s graphical user interface.

1.  **Sign in to the Azure Portal:** Open your web browser and navigate to [https://portal.azure.com/](https://portal.azure.com/). Sign in using your Azure account.

2.  **Create a Resource Group:**  A resource group is a container for related Azure resources.
    *   In the Azure portal, search for and select **Resource groups**.
    *   Click **Create**.
    *   Select your Azure subscription.
    *   In the **Resource group name** field, enter `iatd_labs_01_rg`.
    *   Choose a **Region** that is appropriate for you (e.g., East US).
    *   Click **Review + create**.
    *   Click **Create**.

3.  **Create the Virtual Network:**
    *   In the Azure portal, search for and select **Virtual networks**.
    *   Click **Create**.
    *   On the **Basics** tab, configure the following settings:
        *   **Project details:**
            *   **Subscription:** Select your Azure subscription.
            *   **Resource group:** Select `iatd_labs_01_rg`.
        *   **Instance details:**
            *   **Name:** Enter `iatd_labs_01_vnet_portal`.
            *   **Region:** Choose the same region as your resource group (e.g., East US).

    *   Navigate to the **IP Addresses** tab (or click **Next: IP Addresses >**).
        *   Under **IPv4 address space**, change the address range to `172.16.0.0/16`.
        *   Click on the existing **subnet** listed.
        *   In the **Edit subnet** pane, configure the following:
            *   **Subnet name:**  Enter `iatd_labs_01_subnet_portal`.
            *   **Subnet address range:** Enter `172.16.0.0/24`.
        *   Click **Save**.
    *   Click **Review + create**.
    *   Click **Create**. The deployment process will begin.

4.  **Validate the Virtual Network Creation:** Once the deployment completes successfully, navigate to the `iatd_labs_01_rg` resource group. Locate and select the `iatd_labs_01_vnet_portal` Virtual Network. Verify its address space (`172.16.0.0/16`) and subnet configuration (`iatd_labs_01_subnet_portal` with address range `172.16.0.0/24`).

### Task 2: Creating a Virtual Network using Azure PowerShell

In this task, you will use Azure PowerShell commands within the Cloud Shell to create a VNet.

1.  **Launch Azure Cloud Shell:**
    *   From the Azure Portal, click the Cloud Shell icon located in the top navigation bar.
    *   When prompted, choose **PowerShell** as your preferred shell environment.
    *   If this is your first time using the Cloud Shell, you might be prompted to create a storage account. Follow the on-screen instructions to create one.

2.  **Declare Variables:** Within the Cloud Shell (PowerShell environment), define the following variables. Modify the `$location` variable if needed to reflect your desired Azure region.

    ```powershell
    $resourceGroupName = "iatd_labs_01_rg"
    $vnetName = "iatd_labs_01_vnet_ps"
    $location = "EastUS"  # Change this to your desired Azure region
    $addressPrefix = "172.16.1.0/16"
    $subnetName = "iatd_labs_01_subnet_ps"
    $subnetPrefix = "172.16.1.0/24"
    ```

3.  **Create Resource Group (if it doesn't already exist):** Use the `New-AzResourceGroup` cmdlet to create the resource group. The `-ErrorAction SilentlyContinue` parameter ensures that the command does not produce an error if the resource group already exists.

    ```powershell
    New-AzResourceGroup -Name $resourceGroupName -Location $location -ErrorAction SilentlyContinue
    ```

    Example Output:

    ```
    ResourceGroupName : iatd_labs_01_rg
    Location          : eastus
    ProvisioningState : Succeeded
    Tags              :
    ```

4.  **Create Subnet Configuration:** Create a subnet configuration using the `New-AzVirtualNetworkSubnetConfig` cmdlet. This configuration defines the subnet's name and address prefix.

    ```powershell
    $subnet = New-AzVirtualNetworkSubnetConfig -Name $subnetName -AddressPrefix $subnetPrefix
    ```

    Example Output:

    ```
    Name             : iatd_labs_01_subnet_ps
    Id               : /subscriptions/.../resourceGroups/iatd_labs_01_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_01_vnet_ps/subnets/iatd_labs_01_subnet_ps
    AddressPrefix    : 172.16.1.0/24
    ...
    ```

5.  **Create Virtual Network:**  Use the `New-AzVirtualNetwork` cmdlet to create the VNet, referencing the previously defined resource group, location, address prefix, and subnet configuration.

    ```powershell
    New-AzVirtualNetwork -Name $vnetName -ResourceGroupName $resourceGroupName -Location $location -AddressPrefix $addressPrefix -Subnet $subnet
    ```

    Example Output:

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

6.  **Verify Virtual Network Creation:** Use the `Get-AzVirtualNetwork` cmdlet to retrieve the newly created virtual network and confirm its settings.

    ```powershell
    Get-AzVirtualNetwork -Name $vnetName -ResourceGroupName $resourceGroupName
    ```

    Inspect the output to ensure that the VNet was successfully created with the correct address space (`172.16.1.0/16`) and subnet configuration (`iatd_labs_01_subnet_ps` with address prefix `172.16.1.0/24`).

### Task 3: Creating a Virtual Network using Azure CLI

In this task, you’ll create a Virtual Network using Azure CLI commands executed within the Cloud Shell.

1.  **Launch Azure Cloud Shell:**  Follow the same steps as in Task 2, but choose **Bash** as your preferred shell environment within the Cloud Shell.

2.  **Define Variables:** In the Cloud Shell (Bash environment), set the following variables. Modify the `LOCATION` variable as needed to match your target Azure region.

    ```azurecli
    RESOURCE_GROUP="iatd_labs_01_rg"
    VNET_NAME="iatd_labs_01_vnet_cli"
    LOCATION="eastus" # Modify to match your region
    ADDRESS_PREFIX="172.16.2.0/16"
    SUBNET_NAME="iatd_labs_01_subnet_cli"
    SUBNET_PREFIX="172.16.2.0/24"
    ```

3.  **Create Resource Group (if it doesn't exist):**  Use the `az group create` command to create the resource group if it doesn't already exist. The `--output json --only-show-errors` parameters ensure that the command outputs JSON only if an error occurs.

    ```azurecli
    az group create --name $RESOURCE_GROUP --location $LOCATION --output json --only-show-errors
    ```

    Example Output:

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

4.  **Create Virtual Network:** Use the `az network vnet create` command to create the virtual network. This command specifies the resource group, VNet name, address prefixes, subnet name, subnet prefixes, and location.

    ```azurecli
    az network vnet create \
      --resource-group $RESOURCE_GROUP \
      --name $VNET_NAME \
      --address-prefixes $ADDRESS_PREFIX \
      --subnet-name $SUBNET_NAME \
      --subnet-prefixes $SUBNET_PREFIX \
      --location $LOCATION
    ```

    Example Output:

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
        }
        "enableDdosProtection": false,
        "enableVmProtection": false,
        "id": "/subscriptions/.../resourceGroups/iatd_labs_01_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_01_vnet_cli",
        "location": "eastus",
        "name": "iatd_labs_01_vnet_cli",
        ...
    }
    ```

5.  **Verify Virtual Network Creation:** Use the `az network vnet show` command to retrieve the properties of the newly created virtual network and verify its configuration.

    ```azurecli
    az network vnet show --name $VNET_NAME --resource-group $RESOURCE_GROUP
    ```

    Review the output and ensure the VNet's configuration matches the parameters defined earlier, including the address space (`172.16.2.0/16`) and subnet (`iatd_labs_01_subnet_cli` with prefix `172.16.2.0/24`).

### Task 4: Creating a Virtual Network using Azure Resource Manager (ARM) Templates

In this task, you'll create a VNet by deploying an ARM template using Azure CLI within Cloud Shell.

1.  **Create the ARM Template File:**
    *   From within the Cloud Shell, use the `code` command to create a new file named `vnet_template.json`. This will open the built-in Cloud Shell editor.

        ```bash
        code vnet_template.json
        ```

    *   Paste the following JSON into the `vnet_template.json` file.  This template defines a VNet with a single subnet.

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

2.  **Deploy the ARM Template:** Deploy the ARM template using the `az deployment group create` command in Azure CLI.

    ```azurecli
    RESOURCE_GROUP="iatd_labs_01_rg"
    TEMPLATE_FILE="vnet_template.json"

    az deployment group create \
      --resource-group $RESOURCE_GROUP \
      --template-file $TEMPLATE_FILE
    ```

    Example Output:

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

3.  **Verify Virtual Network Creation:**  Use Azure CLI to verify that the VNet was created.

    ```azurecli
    az network vnet show --name "iatd_labs_01_vnet_arm" --resource-group $RESOURCE_GROUP
    ```

    Examine the output. Ensure that the VNet was created with the settings defined in the ARM template (address space `172.16.3.0/16` and subnet `iatd_labs_01_subnet_arm` with prefix `172.16.3.0/24`).  You can also verify through the Azure Portal.

## Post-Lab Tasks: Clean Up Resources

To avoid incurring unwanted Azure charges, it is crucial to delete the resources you created during this lab. Remove the resource group to ensure that all contained resources are deleted.

1.  **Delete the Resource Group:**

    *   **Using the Azure Portal:**
        *   In the Azure portal, navigate to the `iatd_labs_01_rg` resource group.
        *   Click **Delete resource group**.
        *   Confirm the deletion by typing the name of the resource group and clicking **Delete**.

    *   **Using Azure PowerShell:**

        ```powershell
        Remove-AzResourceGroup -Name $resourceGroupName -Force
        ```

    *   **Using Azure CLI:**

        ```azurecli
        az group delete --name $RESOURCE_GROUP --yes
        ```

## Conclusion

You have successfully completed Lab 1 and gained hands-on experience creating Azure Virtual Networks using the Azure Portal, Azure PowerShell, Azure CLI, and Azure Resource Manager (ARM) templates.  You should now have a solid understanding of how to provision and configure VNets using different methods. Always remember to clean up your Azure resources to avoid incurring unnecessary costs.