## IATD Microcredential Cloud Networking: Lab 10 - Triggering ARM Template Deployments with Azure DevOps to Create a VNet

**Objective:** This lab demonstrates how to integrate Azure DevOps with Azure Resource Manager (ARM) templates to automate the deployment of Azure infrastructure. You'll learn how to set up a basic Azure DevOps pipeline that triggers an ARM template deployment to create a Virtual Network (VNet).

**Estimated Time:** 90 - 120 minutes

**Prerequisites:**

1.  **Azure Subscription:** Requires an active Azure subscription. A free account is available at [https://azure.microsoft.com/free/](https://azure.microsoft.com/free/).
2.  **Basic knowledge of Azure Networking:** Familiarity with Virtual Networks and Subnets.
3.  **Basic understanding of ARM Templates:** Familiarity with ARM template structure is helpful.
4.  **Azure DevOps Organization:** You need an Azure DevOps organization. If you don't have one, you can create a free one at [https://dev.azure.com/](https://dev.azure.com/).
5.  **Azure DevOps Project:** You need a project within your Azure DevOps organization.
6.  **Azure Service Principal:** You need an Azure Service Principal with contributor rights on your Azure subscription. Note the `appId` (client ID), `password` (client secret), and `tenantId`.

### Lab Conventions

*   **Azure Cloud Shell:** All PowerShell and CLI interactions will occur within the Azure Cloud Shell environment.
*   **Azure Portal:** The Azure Portal will be used for creating the Service Principal.
*   **Azure DevOps:** Azure DevOps will be used for pipeline creation and management.
*   **Naming Conventions:** Resource names follow the convention `iatd_labs_10_*`.
*   **Location:** Choose a consistent Azure region.

#### Resource Naming

*   **Resource Group:** `iatd_labs_10_rg`
*   **Virtual Network:** `iatd_labs_10_vnet_devops`
*   **Subnet:** `iatd_labs_10_subnet_devops`
*   **ARM Template File:** `vnet_template.json`
*   **Azure DevOps Project:** (Your Azure DevOps Project Name)
*   **Azure Service Connection:** `iatd-azure-connection`

### Part 1: Preparing the ARM Template

1.  **Create ARM Template File:**

    *   Use the same `vnet_template.json` file from Lab 8, or create a new one. A basic example is provided below:

        ```json
        {
          "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
          "contentVersion": "1.0.0.0",
          "parameters": {
            "vnetName": {
              "type": "string",
              "defaultValue": "iatd_labs_10_vnet_devops",
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
            "subnetName": {
              "type": "string",
              "defaultValue": "iatd_labs_10_subnet_devops",
              "metadata": {
                "description": "Subnet Name"
              }
            },
            "subnetPrefix": {
              "type": "string",
              "defaultValue": "10.0.1.0/24",
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

    *   Save this `vnet_template.json` file.

### Part 2: Storing the ARM Template in a Repository

1.  **Create a Repository in Azure DevOps:**

    *   In your Azure DevOps project, select **Repos**.
    *   Click **Create a new repository**.
    *   Choose a name for your repository (e.g., `azure-networking`).
    *   Click **Create**.

2.  **Commit and Push the ARM Template:**

    * Add the ARM template to the Repo and push it or use Cloud Shell:

        ```bash
        # Example using local git commands (assuming you have git installed)
        git init
        git add vnet_template.json
        git commit -m "Add VNet ARM template"
        git remote add origin <Your Azure DevOps Repo URL>
        git push -u origin main
        ```

### Part 3: Creating an Azure Service Connection

1.  **Create a Service Principal:**
    *Skip it if you already created and used it for the last exercises, it's a nice to have one to use it with all labs*
    *   In the Azure Portal, search for "App registrations" and select.
    *   Click **New registration**.
        *   Name: `AzureDevOpsServicePrincipal`
        *   Supported account types: Select "Single tenant" or "Multi-tenant" based on your needs.
        *   Redirect URI: Leave blank.
    *   Click **Register**.

2.  **Get the Application (client) ID and Directory (tenant) ID:**

    *   In your App registration, note the **Application (client) ID** and the **Directory (tenant) ID**.

3.  **Create a Client Secret:**

    *   In your App registration, select **Certificates & secrets** under **Manage**.
    *   Click **New client secret**.
    *   Description: `AzureDevOpsSecret`
    *   Expires: Choose an appropriate expiry time.
    *   Click **Add**.
    *   *Important:* Copy the **Value** of the client secret immediately. You won't be able to retrieve it later.

4.  **Assign Contributor Role to the Service Principal:**

    *   Navigate to your Azure subscription in the Azure Portal.
    *   Select **Access control (IAM)**.
    *   Click **Add** -> **Add role assignment**.
        *   Role: `Contributor`
        *   Assign access to: `User, group, or service principal`
        *   Select members: Search for `AzureDevOpsServicePrincipal` and select it.
    *   Click **Review + assign**, then **Assign**.

5.  **Create a New Service Connection in Azure DevOps:**

    *   In your Azure DevOps project, select **Project settings** (bottom left).
    *   Select **Service connections** under **Pipelines**.
    *   Click **New service connection**.
    *   Select **Azure Resource Manager** and click **Next**.
    *   Authentication method: **Service principal (manual)**
        *   Subscription ID: Your Azure subscription ID.
        *   Subscription name: Your Azure subscription name.
        *   Service principal client ID: The `appId` (client ID) of your Service Principal.
        *   Service principal key: The `password` (client secret) of your Service Principal.
        *   Tenant ID: The `tenantId` of your Service Principal.
    *   Connection name: `iatd-azure-connection`
    *   Click **Verify and save**.

### Part 4: Creating the Azure DevOps Pipeline

1.  **Create a New Pipeline:**

    *   In your Azure DevOps project, select **Pipelines**.
    *   Click **Create Pipeline**.
    *   Select **Use the classic editor**
    *   Select source: `Azure Repos Git`
    *   Select your repository (`azure-networking`).
    *   Click **Continue**.
    *   Select a template: **Empty job**.

2.  **Configure the Pipeline:**

    *   **Agent Phase 1:**
        *   Click the "+" icon to add a task to the agent job.
        *   Search for "ARM template deployment" and click **Add**.
        *   Configure the ARM template deployment task:
            *   Display name: `Deploy VNet`
            *   Azure subscription: Select your Azure service connection (`iatd-azure-connection`).
            *   Resource group: `iatd_labs_10_rg`
            *   Location: Select your Azure region.
            *   Template location: `Linked artifact`
            *   Template: Browse to your `vnet_template.json` file in the repository.
            *   Override template parameters: `-vnetName iatd_labs_10_vnet_devops -location eastus` (Customize as needed)
            * Deployment mode: Incremental

    *   **Save & Queue:**

        *   Click **Save & queue** to save and run the pipeline.

### Part 5: Verifying the Deployment

1.  **Monitor the Pipeline Run:**

    *   In Azure DevOps, go to your pipeline and monitor the progress of the run.
    *   Verify that all tasks complete successfully.

2.  **Verify in Azure Portal:**

    *   Navigate to the `iatd_labs_10_rg` Resource Group in the Azure Portal.
    *   Verify that the `iatd_labs_10_vnet_devops` Virtual Network has been created with the correct address prefix and subnet.

### Post-Lab: Cleanup

To avoid incurring unnecessary costs, it's essential to clean up the resources created during this lab.

1.  **Delete Resource Group:**

    ```bash
    az group delete --name $RESOURCE_GROUP --yes
    ```

**Learning Outcomes:**

*   Successfully created an ARM template to define an Azure Virtual Network.
*   Successfully stored the ARM template in an Azure DevOps repository.
*   Successfully created an Azure Service Connection to connect Azure DevOps to your Azure subscription.
*   Successfully created an Azure DevOps pipeline to trigger ARM template deployments.
*   Successfully deployed the VNet using the Azure DevOps pipeline.
*   Understood the benefits of using Azure DevOps to automate infrastructure deployments.
