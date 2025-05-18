## IATD Microcredential Cloud Networking: Lab 14 - Remediating NSG Drift with `az deployment group create`

**Objective:** This lab focuses on using the `az deployment group create` command in the Azure CLI to re-apply an ARM template and remediate configuration drift in Network Security Groups (NSGs). You'll simulate NSG rule changes made outside of your infrastructure as code (IaC) process and then use the CLI to restore the NSG to its desired state as defined in the ARM template.

**Estimated Time:** 60 - 75 minutes

**Prerequisites:**

1.  **Azure Subscription:** Requires an active Azure subscription. A free account is available at [https://azure.microsoft.com/free/](https://azure.microsoft.com/free/).
2.  **Basic knowledge of Azure Networking:** Familiarity with Virtual Networks, Subnets, and Network Security Groups.
3.  **Basic understanding of ARM Templates:** Familiarity with ARM template structure is helpful.
4.  **Azure CLI Installed and Configured:** Ensure the Azure CLI is installed and configured on your local machine or in Cloud Shell.
5.  **Existing Virtual Network and Subnet:** You need an existing VNet (`iatd_labs_14_vnet`) and subnet (`iatd_labs_14_subnet`) in your Azure subscription.
6.  **Existing Network Security Group:** You need an existing NSG (`iatd_labs_14_nsg`) associated with the subnet.

### Lab Conventions

*   **Azure Cloud Shell:** All interactions will occur within the Azure Cloud Shell environment (Bash).
*   **Azure Portal:** The Azure Portal will be used for verification tasks.
*   **Naming Conventions:** Resource names follow the convention `iatd_labs_14_*`.
*   **Location:** Choose a consistent Azure region.

#### Resource Naming

*   **Resource Group:** `iatd_labs_14_rg`
*   **Virtual Network:** `iatd_labs_14_vnet` (Existing)
*   **Subnet:** `iatd_labs_14_subnet` (Existing)
*   **Network Security Group:** `iatd_labs_14_nsg` (Existing)
*   **ARM Template File:** `nsg_template.json`

### Part 1: Preparing the ARM Template

1.  **Open Cloud Shell:** In the Azure Portal, click the Cloud Shell icon. Select **Bash** if prompted.

2.  **Create ARM Template File:**

    *   In Cloud Shell (Bash), create a file named `nsg_template.json`:

        ```bash
        code nsg_template.json
        ```

    *   Paste in the following JSON. *Customize this template with your desired NSG rules*. This example allows SSH and HTTP access:

        ```json
        {
          "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
          "contentVersion": "1.0.0.0",
          "parameters": {
            "nsgName": {
              "type": "string",
              "defaultValue": "iatd_labs_14_nsg",
              "metadata": {
                "description": "Name of the Network Security Group"
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
              "type": "Microsoft.Network/networkSecurityGroups",
              "apiVersion": "2020-06-01",
              "name": "[parameters('nsgName')]",
              "location": "[parameters('location')]",
              "properties": {
                "securityRules": [
                  {
                    "name": "AllowSSH",
                    "properties": {
                      "priority": 100,
                      "direction": "Inbound",
                      "access": "Allow",
                      "protocol": "Tcp",
                      "sourcePortRange": "*",
                      "destinationPortRange": "22",
                      "sourceAddressPrefix": "*",
                      "destinationAddressPrefix": "*"
                    }
                  },
                  {
                    "name": "AllowHTTP",
                    "properties": {
                      "priority": 110,
                      "direction": "Inbound",
                      "access": "Allow",
                      "protocol": "Tcp",
                      "sourcePortRange": "*",
                      "destinationPortRange": "80",
                      "sourceAddressPrefix": "*",
                      "destinationAddressPrefix": "*"
                    }
                  }
                ]
              }
            }
          ]
        }
        ```

    *   Save the file.

### Part 2: Simulating Configuration Drift

1.  **Simulate NSG Rule Change (Portal):**

    *   In the Azure Portal, navigate to the `iatd_labs_14_nsg` Network Security Group.
    *   Select **Inbound security rules** under **Settings**.
    *   *Manually* delete the "AllowHTTP" rule. This simulates a configuration change made outside of the ARM template.

### Part 3: Redeploying the ARM Template to Correct Drift

1.  **Set Variables:**

    ```bash
    RESOURCE_GROUP="iatd_labs_14_rg"
    TEMPLATE_FILE="nsg_template.json"
    LOCATION="eastus"  # Replace with your desired location
    NSG_NAME="iatd_labs_14_nsg"
    ```

2.  **Deploy ARM Template using `az deployment group create`:**

    ```bash
    az deployment group create \
      --resource-group $RESOURCE_GROUP \
      --template-file $TEMPLATE_FILE \
      --parameters nsgName=$NSG_NAME location=$LOCATION
    ```

    *   This command uses the `az deployment group create` command to redeploy the ARM template to the specified resource group.  It will effectively re-apply all NSG rules defined in the template, correcting the drift.

### Part 4: Verifying the Remediation

1.  **Verify in Azure Portal:**

    *   Navigate to the `iatd_labs_14_nsg` Network Security Group in the Azure Portal.
    *   Select **Inbound security rules** under **Settings**.
    *   Verify that the missing NSG rule (e.g., "AllowHTTP") has been restored.

2.  **Testing Connectivity (Optional):**

    *You can deploy a Virtual machine and perform connectivity tests, but it requires additional and uneccesary resources to be deployed*

### Post-Lab: Cleanup

To avoid incurring unnecessary costs, it's essential to clean up the resources created during this lab.

1.  **Delete Resource Group:**

    ```bash
    az group delete --name $RESOURCE_GROUP --yes
    ```

**Learning Outcomes:**

*   Successfully simulated configuration drift in an Azure Network Security Group.
*   Successfully used the `az deployment group create` command to re-apply an ARM template and remediate the NSG drift.
*   Verified that the missing NSG rule was restored after the ARM template deployment.
*   Understood how to use infrastructure as code and Azure CLI to maintain consistent and secure network configurations in Azure and correct unintended changes.
