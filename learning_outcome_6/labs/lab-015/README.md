## IATD Microcredential Cloud Networking: Lab 15 - Automating VNet Deployments with Azure DevOps Pipelines and Code Commits

**Objective:** This lab demonstrates how to set up a CI/CD (Continuous Integration/Continuous Deployment) pipeline in Azure DevOps that automatically deploys ARM templates for Virtual Networks (VNets) whenever changes are committed to your code repository. This enables a fully automated infrastructure deployment workflow.

**Estimated Time:** 90 - 120 minutes

**Prerequisites:**

1.  **Azure Subscription:** Requires an active Azure subscription. A free account is available at [https://azure.microsoft.com/free/](https://azure.microsoft.com/free/).
2.  **Basic knowledge of Azure Networking:** Familiarity with Virtual Networks and Subnets.
3.  **Basic understanding of ARM Templates:** Familiarity with ARM template structure is helpful.
4.  **Azure DevOps Organization:** You need an Azure DevOps organization. If you don't have one, you can create a free one at [https://dev.azure.com/](https://dev.azure.com/).
5.  **Azure DevOps Project:** You need a project within your Azure DevOps organization.
6.  **Azure Service Principal:** You need an Azure Service Principal with contributor rights on your Azure subscription. Note the `appId` (client ID), `password` (client secret), and `tenantId`.
7.  **Azure Repos Git Repository:** You need an Azure Repos Git repository containing your ARM template.

### Lab Conventions

*   **Azure Cloud Shell:** All PowerShell and CLI interactions will occur within the Azure Cloud Shell environment.
*   **Azure Portal:** The Azure Portal will be used for creating the Service Principal.
*   **Azure DevOps:** Azure DevOps will be used for pipeline creation and management.
*   **Naming Conventions:** Resource names follow the convention `iatd_labs_15_*`.
*   **Location:** Choose a consistent Azure region.

#### Resource Naming

*   **Resource Group:** `iatd_labs_15_rg`
*   **Virtual Network:** `iatd_labs_15_vnet_cicd`
*   **Subnet:** `iatd_labs_15_subnet_cicd`
*   **ARM Template File:** `vnet_template.json`
*   **Azure DevOps Project:** (Your Azure DevOps Project Name)
*   **Azure Service Connection:** `iatd-azure-connection`
*   **Azure DevOps Repository:** (Your Azure DevOps Repo - e.g., `azure-vnet-templates`)

### Part 1: Preparing the ARM Template and Repository

1.  **Ensure you have an ARM Template:**
    *   Use the same `vnet_template.json` file from Lab 8 or Lab 11, or create a new one. The template should define a basic VNet.

2.  **Ensure the ARM Template is in a Repository:**
    *   Verify that your ARM template is stored in an Azure Repos Git repository. You can reuse the repository from Lab 11 or create a new one.
    *   If you are not sure follow again Lab 11
    * Make sure you have comitted and pushed all code

### Part 2: Setting up the Azure DevOps Pipeline

1.  **Create a New Pipeline:**

    *   In your Azure DevOps project, select **Pipelines**.
    *   Click **Create Pipeline**.
    *   Select **Use the classic editor**
    *   Select source: `Azure Repos Git`
    *   Select your repository (e.g., `azure-vnet-templates`).
    *   Click **Continue**.
    *   Select a template: **Empty job**.

2.  **Add the ARM Template Deployment Task:**

    *   **Agent Phase 1:**
        *   Click the "+" icon to add a task to the agent job.
        *   Search for "ARM template deployment" and click **Add**.

3.  **Configure the ARM Template Deployment Task:**

    *   Select the "ARM template deployment" task you just added.
    *   Configure the task:
        *   Display name: `Deploy VNet`
        *   Azure subscription: Select your Azure service connection (`iatd-azure-connection`).
        *   Resource group: `iatd_labs_15_rg`
        *   Location: Select your Azure region.
        *   Template location: `Linked artifact`
        *   Template: Browse to your `vnet_template.json` file in the repository.
        *   Override template parameters: `-vnetName iatd_labs_15_vnet_cicd -subnetName iatd_labs_15_subnet_cicd -location eastus` (Customize as needed)
        *   Deployment mode: `Incremental`

4.  **Enable Continuous Integration (CI) Trigger:**

    *   Select **Triggers** in the pipeline editor.
    *   Check the box to **Enable continuous integration**.
    *  You can also setup build validation on pull request to test and check

5.  **Save and Queue the Pipeline:**

    *   Click **Save & queue** to save the pipeline. This will also trigger an initial run of the pipeline.

### Part 3: Testing the Automated Deployment

1.  **Make a Change to the ARM Template:**

    *   Open the `vnet_template.json` file in your repository.
    *   Modify the `vnetAddressPrefix` parameter value (e.g., change it from `10.0.0.0/16` to `10.1.0.0/16`).

2.  **Commit and Push the Change:**

    *   Commit and push the changes to your repository:

        ```bash
        git add vnet_template.json
        git commit -m "Update VNet address prefix"
        git push origin main
        ```

3.  **Monitor the Pipeline Run:**

    *   In Azure DevOps, go to your pipeline and monitor the progress of the new run. The pipeline should be automatically triggered by the code commit.

4.  **Verify Deployment in Azure Portal:**

    *   Navigate to the `iatd_labs_15_rg` Resource Group in the Azure Portal.
    *   Verify that the `iatd_labs_15_vnet_cicd` Virtual Network has been updated with the new address prefix (`10.1.0.0/16`).

### Part 4: Configuring Notifications and approvals

1.  **Add Approvals and Checks(Portal):**

   * Check the steps on that microsoft page to increase the security

    *   Add a approval step with you
    * Set other resources to check

### Post-Lab: Cleanup

To avoid incurring unnecessary costs, it's essential to clean up the resources created during this lab.

1.  **Delete Resource Group:**

    ```bash
    az group delete --name $RESOURCE_GROUP --yes
    ```

**Learning Outcomes:**

*   Successfully created an Azure DevOps pipeline to deploy ARM templates for VNets.
*   Successfully configured the pipeline to trigger automatically on code commits.
*   Successfully modified the ARM template and verified that the changes were automatically deployed.
*   Demonstrated a fully automated CI/CD workflow for deploying Azure infrastructure.
*   Understand the benefits of automating infrastructure deployments with Azure DevOps.
*   Set up build validations, Approval and Checks
