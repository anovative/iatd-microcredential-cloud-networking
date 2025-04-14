## IATD Microcredential Cloud Networking: Lab 13 - Enforcing NSG Presence on VNets with Azure Policy

**Objective:** This lab focuses on using Azure Policy to enforce the presence of a Network Security Group (NSG) on all Virtual Networks (VNets) within your Azure subscription. You'll learn how to define a policy, assign it to a scope, and verify that it is enforced during ARM template deployments. This ensures consistent security configurations and helps prevent unintended misconfigurations.

**Estimated Time:** 90 - 120 minutes

**Prerequisites:**

1.  **Azure Subscription:** Requires an active Azure subscription. A free account is available at [https://azure.microsoft.com/free/](https://azure.microsoft.com/free/).
2.  **Basic knowledge of Azure Networking:** Familiarity with Virtual Networks, Subnets, and Network Security Groups.
3.  **Basic understanding of ARM Templates:** Familiarity with ARM template structure is helpful.
4.  **Familiarity with Azure Policy:** While not strictly required, some familiarity with Azure Policy concepts is helpful.

### Lab Conventions

*   **Azure Cloud Shell:** All PowerShell and CLI interactions will occur within the Azure Cloud Shell environment.
*   **Azure Portal:** The Azure Portal will be used for policy creation, assignment, and verification tasks.
*   **Naming Conventions:** Resource names follow the convention `iatd_labs_13_*`.
*   **Location:** Choose a consistent Azure region.

#### Resource Naming

*   **Resource Group:** `iatd_labs_13_rg`
*   **Virtual Network (Compliant):** `iatd_labs_13_vnet_compliant`
*   **Virtual Network (Non-Compliant):** `iatd_labs_13_vnet_noncompliant`
*   **Subnet (Compliant VNet):** `iatd_labs_13_subnet_compliant`
*   **Subnet (Non-Compliant VNet):** `iatd_labs_13_subnet_noncompliant`
*   **Network Security Group:** `iatd_labs_13_nsg`
*   **Policy Definition:** `EnforceNSGOnSubnets`
*   **Policy Assignment:** `EnforceNSGAssignment`
*   **ARM Template File (Compliant):** `vnet_compliant_template.json`
*   **ARM Template File (Non-Compliant):** `vnet_noncompliant_template.json`

### Part 1: Creating a Network Security Group

1.  **Create a Network Security Group (Portal):**

    *   Search for "Network security groups" and select.
    *   Click **Create**.
        *   Subscription: Your subscription.
        *   Resource group: `iatd_labs_13_rg`
        *   Name: `iatd_labs_13_nsg`
        *   Region: Your region.
        * You can leave all configurations default
    *   Click **Review + create**, then **Create**.

### Part 2: Creating the Azure Policy Definition

1.  **Create the Policy Definition (Portal):**

    *   Search for "Policy" and select **Policy**.
    *   Select **Definitions** under **Authoring**.
    *   Click **+ Policy definition**.
        *   Name: `EnforceNSGOnSubnets`
        *   Description: "This policy enforces that all subnets have a Network Security Group associated with them."
        *   Category: Create a new category called "Networking" or choose an existing one.
        *   Policy rule: Paste the following JSON. This policy uses `existenceCondition` to check if a subnet has an NSG associated:

        ```json
        {
          "mode": "Indexed",
          "policyRule": {
            "if": {
              "allOf": [
                {
                  "field": "type",
                  "equals": "Microsoft.Network/virtualNetworks/subnets"
                },
                {
                  "field": "Microsoft.Network/virtualNetworks/subnets/networkSecurityGroup.id",
                  "exists": false
                }
              ]
            },
            "then": {
              "effect": "deny"
            }
          },
          "metadata": {
            "category": "Networking"
          }
        }
        ```

    *   Click **Save**.

### Part 3: Assigning the Azure Policy

1.  **Assign the Policy (Portal):**

    *   Select **Assignments** under **Authoring** in the Policy service.
    *   Click **Assign policy**.
        *   Scope: Select your Azure subscription or a specific resource group.
        *   Policy definition: Select the `EnforceNSGOnSubnets` policy definition you just created.
        *   Assignment name: `EnforceNSGAssignment`
    *   Click **Review + create**, then **Create**.

### Part 4: Creating Compliant and Non-Compliant ARM Templates

1.  **Create Compliant ARM Template (vnet_compliant_template.json):**

    *   In Cloud Shell (Bash), create a file named `vnet_compliant_template.json`:

        ```bash
        code vnet_compliant_template.json
        ```

    *   Paste in the following JSON. This template creates a VNet and a subnet *with* an NSG associated:

        ```json
        {
          "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
          "contentVersion": "1.0.0.0",
          "parameters": {
            "vnetName": {
              "type": "string",
              "defaultValue": "iatd_labs_13_vnet_compliant",
              "metadata": {
                "description": "Virtual Network Name"
              }
            },
            "vnetAddressPrefix": {
              "type": "string",
              "defaultValue": "10.1.0.0/16",
              "metadata": {
                "description": "Virtual Network Address Prefix"
              }
            },
            "subnetName": {
              "type": "string",
              "defaultValue": "iatd_labs_13_subnet_compliant",
              "metadata": {
                "description": "Subnet Name"
              }
            },
            "subnetPrefix": {
              "type": "string",
              "defaultValue": "10.1.1.0/24",
              "metadata": {
                "description": "Subnet Address Prefix"
              }
            },
            "nsgId": {
              "type": "string",
              "metadata": {
                "description": "Resource ID of the Network Security Group"
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
                    "name": "[parameters('subnetName')]",
                    "properties": {
                      "addressPrefix": "[parameters('subnetPrefix')]",
                      "networkSecurityGroup": {
                        "id": "[parameters('nsgId')]"
                      }
                    }
                  }
                ]
              }
            }
          ]
        }
        ```

    *   Save the file.

2.  **Create Non-Compliant ARM Template (vnet_noncompliant_template.json):**

    *   In Cloud Shell (Bash), create a file named `vnet_noncompliant_template.json`:

        ```bash
        code vnet_noncompliant_template.json
        ```

    *   Paste in the following JSON. This template creates a VNet and a subnet *without* an NSG associated:

        ```json
        {
          "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
          "contentVersion": "1.0.0.0",
          "parameters": {
            "vnetName": {
              "type": "string",
              "defaultValue": "iatd_labs_13_vnet_noncompliant",
              "metadata": {
                "description": "Virtual Network Name"
              }
            },
            "vnetAddressPrefix": {
              "type": "string",
              "defaultValue": "10.2.0.0/16",
              "metadata": {
                "description": "Virtual Network Address Prefix"
              }
            },
            "subnetName": {
              "type": "string",
              "defaultValue": "iatd_labs_13_subnet_noncompliant",
              "metadata": {
                "description": "Subnet Name"
              }
            },
            "subnetPrefix": {
              "type": "string",
              "defaultValue": "10.2.1.0/24",
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

### Part 5: Deploying the ARM Templates and Observing Policy Enforcement

1.  **Set Variables:**

    ```bash
    RESOURCE_GROUP="iatd_labs_13_rg"
    LOCATION="eastus"  # Replace with your desired location
    NSG_ID="<Replace with NSG Resource ID from Part 1>" # Example: /subscriptions/xxxx/resourceGroups/xxx/providers/Microsoft.Network/networkSecurityGroups/xxx
    ```

2.  **Deploy the Compliant ARM Template:**

    ```bash
    TEMPLATE_FILE="vnet_compliant_template.json"
    az deployment group create \
      --resource-group $RESOURCE_GROUP \
      --template-file $TEMPLATE_FILE \
      --parameters nsgId="$NSG_ID" location=$LOCATION
    ```

    *   The deployment should succeed because the VNet and subnet are being created with an NSG associated.

3.  **Deploy the Non-Compliant ARM Template:**

    ```bash
    TEMPLATE_FILE="vnet_noncompliant_template.json"
    az deployment group create \
      --resource-group $RESOURCE_GROUP \
      --template-file $TEMPLATE_FILE \
      --parameters location=$LOCATION
    ```

    *   The deployment should be **blocked** by the Azure Policy. You will see an error message indicating that the policy `EnforceNSGOnSubnets` has denied the deployment.

### Part 6: Examining Policy Compliance

1.  **Navigate to the Policy Assignment (Portal):**

    *   Search for "Policy" and select **Policy**.
    *   Select **Assignments** under **Authoring**.
    *   Click on the `EnforceNSGAssignment` policy assignment.

2.  **Review Compliance State:**

    *   Examine the **Compliance state** of the policy assignment. It should show that the deployment of the non-compliant VNet was denied.
    *   Click on the "Non-compliant resources" link to see the details of the non-compliant resource (the VNet without an NSG).

### Post-Lab: Cleanup

To avoid incurring unnecessary costs, it's essential to clean up the resources created during this lab.

1.  **Delete Policy Assignment:**

    *   Select **Assignments** under **Authoring** in the Policy service.
    *   Select the `EnforceNSGAssignment` policy assignment and click **Delete**.

2.  **Delete Policy Definition:**

    *   Select **Definitions** under **Authoring** in the Policy service.
    *   Select the `EnforceNSGOnSubnets` policy definition and click **Delete**.

3.  **Delete Resource Group:**

    ```bash
    az group delete --name $RESOURCE_GROUP --yes
    ```

**Learning Outcomes:**

*   Successfully created an Azure Policy definition to enforce the presence of NSGs on subnets.
*   Successfully assigned the Azure Policy to a scope (subscription or resource group).
*   Successfully deployed a compliant ARM template that creates a VNet with an associated NSG.
*   Successfully verified that Azure Policy blocked the deployment of a non-compliant ARM template (VNet without an NSG).
*   Understood how to review policy compliance status in the Azure Portal.
*   Demonstrated the ability to use Azure Policy to enforce governance and security standards in Azure environments.
