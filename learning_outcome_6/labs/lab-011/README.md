## IATD Microcredential Cloud Networking: Lab 11 - Deploying Parameterized VNets with ARM Templates for Flexible Configuration

**Objective:** This lab focuses on creating and deploying Azure Virtual Networks (VNets) using parameterized ARM templates. By using parameters, you'll learn how to create flexible and reusable templates that can be easily customized to fit different environments and requirements.

**Estimated Time:** 60 - 75 minutes

**Prerequisites:**

1.  **Azure Subscription:** Requires an active Azure subscription. A free account is available at [https://azure.microsoft.com/free/](https://azure.microsoft.com/free/).
2.  **Basic knowledge of Azure Networking:** Familiarity with Virtual Networks and Subnets.
3.  **Good understanding of ARM Templates:** Familiarity with ARM template structure and parameter usage is essential.

### Lab Conventions

*   **Azure Cloud Shell:** All PowerShell and CLI interactions will occur within the Azure Cloud Shell environment (Bash).
*   **Azure Portal:** The Azure Portal will be used for verification tasks.
*   **Naming Conventions:** Resource names follow the convention `iatd_labs_11_*`.
*   **Location:** Choose a consistent Azure region.

#### Resource Naming

*   **Resource Group:** `iatd_labs_11_rg`
*   **Virtual Network:** `iatd_labs_11_vnet_param`
*   **Subnet 1:** `iatd_labs_11_subnet1_param`
*   **Subnet 2:** `iatd_labs_11_subnet2_param`
*   **ARM Template File:** `vnet_parameterized_template.json`

### Part 1: Creating the Parameterized ARM Template

1.  **Open Cloud Shell:** In the Azure Portal, click the Cloud Shell icon. Select **Bash** if prompted.

2.  **Create ARM Template File:**

    *   In Cloud Shell (Bash), create a file named `vnet_parameterized_template.json`:

        ```bash
        code vnet_parameterized_template.json
        ```

    *   Paste in the following JSON. This template uses parameters for VNet name, address prefix, subnet names, and subnet prefixes:

        ```json
        {
          "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
          "contentVersion": "1.0.0.0",
          "parameters": {
            "vnetName": {
              "type": "string",
              "defaultValue": "iatd_labs_11_vnet_param",
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
              "defaultValue": "iatd_labs_11_subnet1_param",
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
              "defaultValue": "iatd_labs_11_subnet2_param",
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

### Part 2: Deploying the ARM Template with Parameter Overrides

1.  **Set Variables:**

    ```bash
    RESOURCE_GROUP="iatd_labs_11_rg"
    TEMPLATE_FILE="vnet_parameterized_template.json"
    LOCATION="eastus"  # Replace with your desired location

    # Create resource group if it doesn't exist
    az group create --name $RESOURCE_GROUP --location $LOCATION --output json --only-show-errors
    ```

2.  **Deploy ARM Template with Inline Parameter Overrides:**

    ```bash
    az deployment group create \
      --resource-group $RESOURCE_GROUP \
      --template-file $TEMPLATE_FILE \
      --parameters vnetName=iatd_labs_11_vnet_dev \
                   vnetAddressPrefix=10.100.0.0/16 \
                   subnet1Name=webSubnet \
                   subnet1Prefix=10.100.1.0/24 \
                   subnet2Name=appSubnet \
                   subnet2Prefix=10.100.2.0/24 \
      --location $LOCATION
    ```

    *   This command deploys the ARM template and overrides the default values of the parameters with new values specified inline.

3.  **Deploy ARM Template with a Parameter File (Optional):**

    *   Create a parameter file named `vnet_parameters.json`:

        ```json
        {
          "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
          "contentVersion": "1.0.0.0",
          "parameters": {
            "vnetName": {
              "value": "iatd_labs_11_vnet_test"
            },
            "vnetAddressPrefix": {
              "value": "10.101.0.0/16"
            },
            "subnet1Name": {
              "value": "webSubnetTest"
            },
            "subnet1Prefix": {
              "value": "10.101.1.0/24"
            },
            "subnet2Name": {
              "value": "appSubnetTest"
            },
            "subnet2Prefix": {
              "value": "10.101.2.0/24"
            },
            "location": {
              "value": "westus2"
            }
          }
        }
        ```

    *   Deploy the ARM template using the parameter file:

        ```bash
        az deployment group create \
          --resource-group $RESOURCE_GROUP \
          --template-file $TEMPLATE_FILE \
          --parameters @vnet_parameters.json
        ```

### Part 3: Verifying the Deployment

1.  **Verify in Azure Portal:**

    *   Navigate to the `iatd_labs_11_rg` Resource Group in the Azure Portal.
    *   Verify that the Virtual Networks have been created with the specified names (e.g., `iatd_labs_11_vnet_dev` and `iatd_labs_11_vnet_test`).
    *   Check the subnets of each VNet and ensure that they have the correct names and address prefixes.

2.  **Verify using Azure CLI:**

    *   Use the `az network vnet show` command to verify the configuration of the deployed VNets.

    ```bash
    az network vnet show --name iatd_labs_11_vnet_dev --resource-group $RESOURCE_GROUP
    az network vnet show --name iatd_labs_11_vnet_test --resource-group $RESOURCE_GROUP
    ```

    *   Examine the output and confirm the address prefixes and subnets are configured correctly.

### Post-Lab: Cleanup

To avoid incurring unnecessary costs, it's essential to clean up the resources created during this lab.

1.  **Delete Resource Group:**

    ```bash
    az group delete --name $RESOURCE_GROUP --yes
    ```

**Learning Outcomes:**

*   Successfully created a parameterized ARM template to define an Azure Virtual Network with flexible subnet configurations.
*   Successfully deployed the ARM template, overriding parameter values inline to customize the deployment.
*   Successfully deployed the ARM template using a parameter file to provide configuration values.
*   Successfully verified the deployments using the Azure Portal and Azure CLI.
*   Understood the benefits of using parameters in ARM templates for reusable and customizable infrastructure as code.
