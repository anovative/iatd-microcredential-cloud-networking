## IATD Microcredential Cloud Networking: Lab 8 - Deploying Azure Virtual Networks using ARM Templates

**Objective:** This lab will guide you through creating and deploying an Azure Virtual Network (VNet) using Azure Resource Manager (ARM) templates. You'll learn how to define infrastructure as code, enabling repeatable and consistent deployments across different Azure subscriptions and environments.

**Estimated Time:** 60 - 75 minutes

**Prerequisites:**

1.  **Azure Subscription:** Requires an active Azure subscription. A free account is available at [https://azure.microsoft.com/free/](https://azure.microsoft.com/free/).
2.  **Basic understanding of Azure Networking:** Familiarity with Virtual Networks and Subnets.
3.  **Basic understanding of ARM Templates:** Familiarity with the structure of ARM templates is helpful.

### Lab Conventions

*   **Azure Cloud Shell:** All interactions will occur within the Azure Cloud Shell environment (Bash).
*   **Azure Portal:** The Azure Portal will be used for verification tasks.
*   **Naming Conventions:** Resource names follow the convention `iatd_labs_08_*`.
*   **Location:** Choose a consistent Azure region.

#### Resource Naming

*   **Resource Group:** `iatd_labs_08_rg`
*   **Virtual Network:** `iatd_labs_08_vnet_arm`
*   **Subnet-Web:** `iatd_labs_08_subnet_web_arm`
*   **Subnet-App:** `iatd_labs_08_subnet_app_arm`
*   **ARM Template File:** `vnet_template.json`

### Part 1: Creating the ARM Template

1.  **Open Cloud Shell:** In the Azure Portal, click the Cloud Shell icon. Select **Bash** if prompted.

2.  **Create ARM Template File:**

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
              "defaultValue": "iatd_labs_08_vnet_arm",
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
            "subnetWebName": {
              "type": "string",
              "defaultValue": "iatd_labs_08_subnet_web_arm",
              "metadata": {
                "description": "Subnet Name for Web Tier"
              }
            },
            "subnetWebPrefix": {
              "type": "string",
              "defaultValue": "10.0.1.0/24",
              "metadata": {
                "description": "Subnet Address Prefix for Web Tier"
              }
            },
            "subnetAppName": {
              "type": "string",
              "defaultValue": "iatd_labs_08_subnet_app_arm",
              "metadata": {
                "description": "Subnet Name for App Tier"
              }
            },
            "subnetAppPrefix": {
              "type": "string",
              "defaultValue": "10.0.2.0/24",
              "metadata": {
                "description": "Subnet Address Prefix for App Tier"
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
                    "name": "[parameters('subnetWebName')]",
                    "properties": {
                      "addressPrefix": "[parameters('subnetWebPrefix')]"
                    }
                  },
                  {
                    "name": "[parameters('subnetAppName')]",
                    "properties": {
                      "addressPrefix": "[parameters('subnetAppPrefix')]"
                    }
                  }
                ]
              }
            }
          ]
        }
        ```

    *   Save the file.

### Part 2: Deploying the ARM Template

1.  **Set Variables:**

    ```bash
    RESOURCE_GROUP="iatd_labs_08_rg"
    TEMPLATE_FILE="vnet_template.json"
    LOCATION="eastus"  # Replace with your desired location

    # Create resource group if it doesn't exist
    az group create --name $RESOURCE_GROUP --location $LOCATION --output json --only-show-errors
    ```

2.  **Deploy ARM Template:**

    ```bash
    az deployment group create \
      --resource-group $RESOURCE_GROUP \
      --template-file $TEMPLATE_FILE
    ```

### Part 3: Verifying the Deployment

1.  **Verify in Azure Portal:**

    *   Navigate to the `iatd_labs_08_rg` Resource Group in the Azure Portal.
    *   Verify that the `iatd_labs_08_vnet_arm` Virtual Network has been created.
    *   Check the subnets and ensure that `iatd_labs_08_subnet_web_arm` and `iatd_labs_08_subnet_app_arm` exist with the correct address prefixes.

2.  **Verify using Azure CLI:**

    ```bash
    az network vnet show --name iatd_labs_08_vnet_arm --resource-group $RESOURCE_GROUP
    ```

    *   Examine the output and confirm the address prefixes and subnets are configured correctly.

### Part 4: Reusing the ARM Template in Another Subscription

1.  **Simulating a New Subscription (Conceptual):**
    *   Assume you have another Azure subscription where you want to deploy the same VNet.

2.  **Deploy ARM Template to New Subscription:**
        *   Make sure you are on the context of the other subscription, can use the following command to change it
        ```bash
        az account set --subscription "<Subscription ID>"
        ```
    *  Run the ARM command again to provision the resources

    ```bash
    az deployment group create \
      --resource-group $RESOURCE_GROUP \
      --template-file $TEMPLATE_FILE
    ```

### Post-Lab: Cleanup

To avoid incurring unnecessary costs, it's essential to clean up the resources created during this lab.

1.  **Delete Resource Group:**

    ```bash
    az group delete --name $RESOURCE_GROUP --yes
    ```

**Learning Outcomes:**

*   Successfully created an ARM template to define an Azure Virtual Network with subnets.
*   Successfully deployed the ARM template to create the VNet in your Azure subscription.
*   Successfully verified the deployment using the Azure Portal and Azure CLI.
*   Understood the benefits of using ARM templates for infrastructure as code and repeatable deployments.
*   Learned how to reuse an ARM template to deploy the same VNet configuration in another Azure subscription.
