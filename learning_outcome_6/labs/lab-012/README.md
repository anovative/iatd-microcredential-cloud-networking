## IATD Microcredential Cloud Networking: Lab 12 - Deploying VNets with Azure CLI using `az deployment group create`

**Objective:** This lab focuses on deploying Azure Virtual Networks (VNets) using the `az deployment group create` command in the Azure CLI. You'll learn the essential steps to define your infrastructure as code with ARM templates and then deploy it using a single, powerful CLI command.

**Estimated Time:** 45 - 60 minutes

**Prerequisites:**

1.  **Azure Subscription:** Requires an active Azure subscription. A free account is available at [https://azure.microsoft.com/free/](https://azure.microsoft.com/free/).
2.  **Basic knowledge of Azure Networking:** Familiarity with Virtual Networks and Subnets.
3.  **Basic understanding of ARM Templates:** Familiarity with ARM template structure is helpful.
4.  **Azure CLI Installed and Configured:** Ensure the Azure CLI is installed and configured on your local machine or in Cloud Shell.

### Lab Conventions

*   **Azure Cloud Shell:** All interactions will occur within the Azure Cloud Shell environment (Bash).
*   **Azure Portal:** The Azure Portal will be used for verification tasks.
*   **Naming Conventions:** Resource names follow the convention `iatd_labs_12_*`.
*   **Location:** Choose a consistent Azure region.

#### Resource Naming

*   **Resource Group:** `iatd_labs_12_rg`
*   **Virtual Network:** `iatd_labs_12_vnet_cli`
*   **Subnet 1:** `iatd_labs_12_subnet1_cli`
*   **Subnet 2:** `iatd_labs_12_subnet2_cli`
*   **ARM Template File:** `vnet_cli_template.json`

### Part 1: Creating the ARM Template

1.  **Open Cloud Shell:** In the Azure Portal, click the Cloud Shell icon. Select **Bash** if prompted.

2.  **Create ARM Template File:**

    *   In Cloud Shell (Bash), create a file named `vnet_cli_template.json`:

        ```bash
        code vnet_cli_template.json
        ```

    *   Paste in the following JSON:

        ```json
        {
          "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
          "contentVersion": "1.0.0.0",
          "parameters": {
            "vnetName": {
              "type": "string",
              "defaultValue": "iatd_labs_12_vnet_cli",
              "metadata": {
                "description": "Virtual Network Name"
              }
            },
            "vnetAddressPrefix": {
              "type": "string",
              "defaultValue": "10.0.0.0/16",
              "metadata": {
                "description": "Virtual Network Address Prefix"
              }
            },
            "subnet1Name": {
              "type": "string",
              "defaultValue": "iatd_labs_12_subnet1_cli",
              "metadata": {
                "description": "Subnet 1 Name"
              }
            },
            "subnet1Prefix": {
              "type": "string",
              "defaultValue": "10.0.1.0/24",
              "metadata": {
                "description": "Subnet 1 Address Prefix"
              }
            },
            "subnet2Name": {
              "type": "string",
              "defaultValue": "iatd_labs_12_subnet2_cli",
              "metadata": {
                "description": "Subnet 2 Name"
              }
            },
            "subnet2Prefix": {
              "type": "string",
              "defaultValue": "10.0.2.0/24",
              "metadata": {
                "description": "Subnet 2 Address Prefix"
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
                    "name": "[parameters('subnet1Name')]",
                    "properties": {
                      "addressPrefix": "[parameters('subnet1Prefix')]"
                    }
                  },
                  {
                    "name": "[parameters('subnet2Name')]",
                    "properties": {
                      "addressPrefix": "[parameters('subnet2Prefix')]"
                    }
                  }
                ]
              }
            }
          ]
        }
        ```

    *   Save the file.

### Part 2: Deploying the ARM Template with Azure CLI

1.  **Set Variables:**

    ```bash
    RESOURCE_GROUP="iatd_labs_12_rg"
    TEMPLATE_FILE="vnet_cli_template.json"
    LOCATION="eastus"  # Replace with your desired location

    # Create resource group if it doesn't exist
    az group create --name $RESOURCE_GROUP --location $LOCATION --output json --only-show-errors
    ```

2.  **Deploy ARM Template using `az deployment group create`:**

    ```bash
    az deployment group create \
      --resource-group $RESOURCE_GROUP \
      --template-file $TEMPLATE_FILE \
      --parameters location=$LOCATION
    ```

    *   This command uses the `az deployment group create` command to deploy the ARM template to the specified resource group.
        *   `--resource-group` specifies the name of the resource group where the resources will be deployed.
        *   `--template-file` specifies the path to the ARM template file.
        *   `--parameters` allows you to override parameter values in the template (optional).
        *   `location` is a parameter defined in the ARM template, allowing to define location with paramater

### Part 3: Verifying the Deployment

1.  **Verify in Azure Portal:**

    *   Navigate to the `iatd_labs_12_rg` Resource Group in the Azure Portal.
    *   Verify that the `iatd_labs_12_vnet_cli` Virtual Network has been created.
    *   Check the subnets of the VNet and ensure that `iatd_labs_12_subnet1_cli` and `iatd_labs_12_subnet2_cli` exist with the correct address prefixes.

2.  **Verify using Azure CLI:**

    *   Use the `az network vnet show` command to verify the configuration of the deployed VNet.

    ```bash
    az network vnet show --name iatd_labs_12_vnet_cli --resource-group $RESOURCE_GROUP
    ```

    *   Examine the output and confirm the address prefixes and subnets are configured correctly.

### Post-Lab: Cleanup

To avoid incurring unnecessary costs, it's essential to clean up the resources created during this lab.

1.  **Delete Resource Group:**

    ```bash
    az group delete --name $RESOURCE_GROUP --yes
    ```

**Learning Outcomes:**

*   Successfully used the `az deployment group create` command to deploy an ARM template for an Azure Virtual Network.
*   Successfully defined a Virtual Network and its subnets using an ARM template.
*   Successfully verified the deployment using the Azure Portal and Azure CLI.
*   Understood the simplicity and power of deploying infrastructure as code with a single Azure CLI command.
