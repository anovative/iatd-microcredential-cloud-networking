## IATD Microcredential Cloud Networking: Lab 9 - Enforcing NSG Rules Across Azure VNets with ARM Templates and Remediation

**Objective:** This lab will guide you through using Azure Resource Manager (ARM) templates to enforce consistent Network Security Group (NSG) rules across multiple Azure Virtual Networks (VNets). You'll also learn how to schedule periodic template re-applications to detect and correct configuration drift, ensuring ongoing security compliance.

**Estimated Time:** 90 - 120 minutes

**Prerequisites:**

1.  **Azure Subscription:** Requires an active Azure subscription. A free account is available at [https://azure.microsoft.com/free/](https://azure.microsoft.com/free/).
2.  **Solid knowledge of Azure Networking:** Familiarity with Virtual Networks, Subnets, and Network Security Groups.
3.  **Good understanding of ARM Templates:** Familiarity with ARM template structure and deployment methods is required.
4.  **Two Existing Azure Virtual Networks:** You need two existing VNets (`iatd_labs_09_vnet1` and `iatd_labs_09_vnet2`) in your Azure subscription. They don't need to be peered.
5.  **Azure Automation Account (Optional):** An Azure Automation Account will be used for scheduling template re-applications.

### Lab Conventions

*   **Azure Cloud Shell:** All PowerShell and CLI interactions will occur within the Azure Cloud Shell environment (Bash).
*   **Azure Portal:** The Azure Portal will be used for verification and configuration tasks.
*   **Naming Conventions:** Resource names follow the convention `iatd_labs_09_*`.
*   **Location:** Choose a consistent Azure region.

#### Resource Naming

*   **Resource Group:** `iatd_labs_09_rg`
*   **Virtual Network 1:** `iatd_labs_09_vnet1` (Existing)
*   **Subnet in VNet 1:** `iatd_labs_09_subnet1` (Existing)
*   **Virtual Network 2:** `iatd_labs_09_vnet2` (Existing)
*   **Subnet in VNet 2:** `iatd_labs_09_subnet2` (Existing)
*   **Network Security Group:** `iatd_labs_09_nsg`
*   **ARM Template File:** `nsg_template.json`

### Part 1: Creating the ARM Template for NSG Rules

1.  **Open Cloud Shell:** In the Azure Portal, click the Cloud Shell icon. Select **Bash** if prompted.

2.  **Create ARM Template File:**

    *   In Cloud Shell (Bash), create a file named `nsg_template.json`:

        ```bash
        code nsg_template.json
        ```

    *   Paste in the following JSON. *Customize the rules to fit your needs*. This example allows SSH and HTTP access:

        ```json
        {
          "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
          "contentVersion": "1.0.0.0",
          "parameters": {
            "nsgName": {
              "type": "string",
              "defaultValue": "iatd_labs_09_nsg",
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
          ],
          "outputs": {
            "networkSecurityGroupId": {
              "type": "string",
              "value": "[resourceId('Microsoft.Network/networkSecurityGroups', parameters('nsgName'))]"
            }
          }
        }
        ```

    *   Save the file.

### Part 2: Deploying the ARM Template

1.  **Set Variables:**

    ```bash
    RESOURCE_GROUP="iatd_labs_09_rg"
    TEMPLATE_FILE="nsg_template.json"
    LOCATION="eastus"  # Replace with your desired location
    NSG_NAME="iatd_labs_09_nsg"

    # Create resource group if it doesn't exist
    az group create --name $RESOURCE_GROUP --location $LOCATION --output json --only-show-errors
    ```

2.  **Deploy ARM Template:**

    ```bash
    az deployment group create \
      --resource-group $RESOURCE_GROUP \
      --template-file $TEMPLATE_FILE \
      --parameters nsgName=$NSG_NAME location=$LOCATION
    ```

3.  **Get the NSG Resource ID:**

    ```bash
    NSG_ID=$(az deployment group show \
      --resource-group $RESOURCE_GROUP \
      --name $(az deployment group list --resource-group $RESOURCE_GROUP --query "[].name" -o tsv) \
      --query properties.outputs.networkSecurityGroupId.value \
      -o tsv)

    echo $NSG_ID
    ```

### Part 3: Associating the NSG with Subnets

1.  **Associate NSG with Subnet in VNet 1:**

    ```bash
    VNET1_NAME="iatd_labs_09_vnet1" # Replace with your actual VNet 1 name
    SUBNET1_NAME="iatd_labs_09_subnet1" # Replace with your actual Subnet 1 name

    az network vnet subnet update \
      --resource-group $RESOURCE_GROUP \
      --vnet-name $VNET1_NAME \
      --name $SUBNET1_NAME \
      --network-security-group $NSG_ID
    ```

2.  **Associate NSG with Subnet in VNet 2:**

    ```bash
    VNET2_NAME="iatd_labs_09_vnet2" # Replace with your actual VNet 2 name
    SUBNET2_NAME="iatd_labs_09_subnet2" # Replace with your actual Subnet 2 name

    az network vnet subnet update \
      --resource-group $RESOURCE_GROUP \
      --vnet-name $VNET2_NAME \
      --name $SUBNET2_NAME \
      --network-security-group $NSG_ID
    ```

### Part 4: Verifying the NSG Rules

1.  **Verify in Azure Portal:**

    *   Navigate to `iatd_labs_09_vnet1` and `iatd_labs_09_vnet2` in the Azure Portal.
    *   Select each one and search subnets to verify that each one has the network security group applied correctly.

2.  **Test connectivity to VMs**

     * Check if you can ping or SSH into the VM

### Part 5: Simulating Configuration Drift and Implementing Remediation

1.  **Simulating Configuration Drift (Manual):**

    *   In the Azure Portal, navigate to the `iatd_labs_09_nsg` Network Security Group.
    *   Select **Inbound security rules** under **Settings**.
    *   *Manually* delete the "AllowHTTP" rule. This simulates a configuration change made outside of the ARM template.
        * This is to demostrate it

2.  **Deploying Remediation:**

    *   Execute the same ARM template deployment commands from Part 2 again for both VNet 1 and VNet 2.
    *   This will re-apply the NSG rules defined in the template and correct the configuration drift.

    ```bash
    RESOURCE_GROUP="iatd_labs_09_rg"
    TEMPLATE_FILE="nsg_template.json"
    VNET1_SUBNET_ID="<Replace with Subnet 1 Resource ID>"
    NSG_NAME="iatd_labs_09_nsg"

    az deployment group create \
      --resource-group $RESOURCE_GROUP \
      --template-file $TEMPLATE_FILE \
      --parameters nsgName=$NSG_NAME location=$LOCATION

    az network vnet subnet update \
      --id $VNET1_SUBNET_ID \
      --network-security-group $NSG_NAME
    ```

    ```bash
    RESOURCE_GROUP="iatd_labs_09_rg"
    TEMPLATE_FILE="nsg_template.json"
    VNET2_SUBNET_ID="<Replace with Subnet 2 Resource ID>"
    NSG_NAME="iatd_labs_09_nsg"

    az deployment group create \
      --resource-group $RESOURCE_GROUP \
      --template-file $TEMPLATE_FILE \
      --parameters nsgName=$NSG_NAME location=$LOCATION

    az network vnet subnet update \
      --id $VNET2_SUBNET_ID \
      --network-security-group $NSG_NAME
    ```

3.  **Verify Remediation (Portal):**

    *   Navigate to the `iatd_labs_09_nsg` Network Security Group in the Azure Portal.
    *   Select **Inbound security rules** under **Settings**.
    *   Verify that the missing NSG rule (e.g., "AllowHTTP") has been restored.

### Part 6: Scheduling Periodic Template Re-applications with Azure Automation (Optional)

1.  **Create an Automation Account (Portal):**
        *Skip this step if you want to run only manual)
    *   Search for "Automation Accounts" and select.
    *   Click **Create**.
        *   Subscription: Your subscription.
        *   Resource group: `iatd_labs_09_rg`
        *   Name: `iatd_labs_09_automation`
        *   Region: Your region.
    * Click **Review + Create** and then **Create**.

2.  **Create a Runbook (Conceptual):**
 *Skip this step if you want to run only manual)
    *   In the Automation Account, create a PowerShell Runbook that executes the ARM template deployment commands from Part 2.

3.  **Schedule the Runbook (Conceptual):**
 *Skip this step if you want to run only manual)
    *   Schedule the Runbook to run periodically (e.g., daily or weekly) to ensure that the NSG rules are consistently enforced and any configuration drift is automatically corrected.

### Post-Lab: Cleanup

To avoid incurring unnecessary costs, it's essential to clean up the resources created during this lab.

1.  **Remove All NSG Associations from all Subnets (Important):**
    *   Navigate to each subnet in your VNets and remove the association with the Network Security Group.
2.  **Delete Network Security Group:**

    ```bash
    az network nsg delete --name iatd_labs_09_nsg --resource-group $RESOURCE_GROUP
    ```3.  **Delete Automation Account (If Created):**

4.  **Delete Resource Group:**

    ```bash
    az group delete --name $RESOURCE_GROUP --yes
    ```

**Learning Outcomes:**

*   Successfully created an ARM template to define NSG rules.
*   Successfully deployed the ARM template to enforce the NSG rules across multiple VNets.
*   Simulated configuration drift and successfully remediated it by re-applying the ARM template.
*   Understood how to schedule periodic template re-applications using Azure Automation to correct drift.
*   Demonstrated the ability to use infrastructure as code to maintain consistent and secure network configurations in Azure.
