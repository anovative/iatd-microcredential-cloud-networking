## IATD Microcredential Cloud Networking: Lab 6 - Monitoring and Logging in Azure - Advanced

**Objective:** This lab builds upon the fundamentals by exploring more advanced monitoring and logging techniques in Azure. You will learn how to use Azure Network Watcher, NSG Flow Logs, Azure Resource Graph, and Azure Service Health.

**Estimated Time:** 75 - 90 minutes

**Prerequisites:**

1.  **Azure Subscription:** Requires an active Azure subscription.
2.  **Completion of Lab 5:** This lab assumes you have a working knowledge of monitoring and logging concepts and core Azure monitoring services.

### Lab Conventions

*   **Azure Cloud Shell:** All PowerShell and CLI interactions will occur within the Azure Cloud Shell environment.
*   **Azure Portal:** The Azure Portal will be used for visualization and configuration tasks.
*   **Naming Conventions:** Resource names follow the convention `iatd_labs_06_*`.
*   **IP Address Range:** The `172.16.0.0/16` range will be used (as per standardization).
*   **Location:** Choose a consistent Azure region.

#### Resource Naming (re-use from Lab 5 where applicable)

*   **Resource Group:** `iatd_labs_05_rg` (re-use)
*   **Virtual Network:** `iatd_labs_05_vnet` (re-use)
*   **Subnet:** `iatd_labs_05_subnet` (re-use)
*   **VM-Target:** `iatd_labs_05_vm_target` (re-use)
*   **Public IP:** `iatd_labs_05_pip` (re-use)
*   **Network Security Group:** `iatd_labs_05_nsg` (re-use)
*   **Log Analytics Workspace:** `iatd_labs_05_law` (re-use)
*   **Application Insights:** `iatd_labs_05_appi` (re-use)

### Part 1: Azure Network Watcher

1.  **Explore Network Watcher Features (Portal):**
    *   Search for "Network Watcher" in the Azure Portal.
    *   Explore the following features:
        *   **Topology:** Visualize the network topology of your virtual network.
        *   **Connection monitor:** Monitor network connectivity between VMs and endpoints.
        *   **IP flow verify:** Verify if traffic is allowed or denied by NSG rules.
        *   **Packet capture:** Capture network traffic for analysis.

2.  **Use IP Flow Verify (Portal):**
    *   Select **IP flow verify** under **Network diagnostic tools**.
    *   Configure the test to check if traffic is allowed from your local machine's IP address to the `iatd_labs_05_vm_target` VM on port 80.

3.  **Use Connection Monitor (Portal):**
    *   Select **Connection monitor** under **Monitoring**.
    *   Create a connection monitor to monitor connectivity between your local machine and the `iatd_labs_05_vm_target` VM on port 80.

### Part 2: NSG Flow Logs and Traffic Analytics

1.  **Enable NSG Flow Logs (Portal):**
    *   If not already enabled in Lab 5, enable NSG Flow Logs for the `iatd_labs_05_nsg` Network Security Group.
        *   Search for "Network Watcher" and select.
        *   Select **NSG flow logs** under **Logs**.
        *   Select the `iatd_labs_05_nsg` Network Security Group.
        *   Choose a storage account to store the flow logs.

2.  **Enable Traffic Analytics (Portal):**
    *   In the NSG Flow Logs configuration, enable Traffic Analytics and configure it to use the `iatd_labs_05_law` Log Analytics Workspace.

3.  **Analyze Traffic Patterns (Portal):**
    *   After collecting flow logs for a period of time, navigate to the `iatd_labs_05_law` Log Analytics Workspace.
    *   Run the following KQL query to analyze traffic patterns:

        ```kusto
        AzureNetworkAnalytics_CL
        | summarize sum(AllowedOutboundFlows_d), sum(DeniedOutboundFlows_d), sum(AllowedInboundFlows_d), sum(DeniedInboundFlows_d) by Subnet_s, DestinationPort_d
        ```

4.  **Review Traffic Analytics Data (Portal):**
    *   In Network Watcher, select **Traffic analytics** under **Logs**.
    *   Explore the visualizations and dashboards to gain insights into network traffic patterns.

### Part 3: Azure Resource Graph

1.  **Explore Azure Resource Graph (Portal):**
    *   Search for "Resource Graph Explorer" and select.
    *   Use Azure Resource Graph to query and explore your Azure resources.

2.  **Run Queries (Portal):**
    *   Run the following query to list all virtual machines in your subscription:

        ```kusto
        Resources
        | where type == "microsoft.compute/virtualmachines"
        | project name, location, resourceGroup
        ```

    *   Run the following query to list all NSGs and their associated subnets:

        ```kusto
        Resources
        | where type == "microsoft.network/networksecuritygroups"
        | extend subnet = properties.subnets[0].id
        | project name, location, resourceGroup, subnet
        ```

### Part 4: Azure Service Health

1.  **Explore Azure Service Health (Portal):**
    *   Search for "Service Health" and select.
    *   Review the health of Azure services in your region.

2.  **Configure Health Alerts (Portal):**
    *   Create health alerts to notify you of service incidents, planned maintenance, and security advisories that may affect your resources.
    *   Select **Health alerts** and click **Add service health alert**.
    *   Configure the alert to monitor specific services and regions.

### Part 5: Azure Cost Management + Billing

1.  **Explore Azure Cost Management (Portal):**
    *   Search for "Cost Management + Billing" and select.
    *   Review your Azure costs and identify areas where you can optimize spending.

2.  **Create a Cost Alert (Portal):**
    *   Create a cost alert to notify you when your spending exceeds a predefined budget.
    *   Select **Cost alerts** and click **Add**.
    *   Configure the alert to monitor your spending and notify you when it exceeds your budget.

### Post-Lab: Cleanup

To avoid incurring unnecessary costs, it's essential to clean up the resources created during this lab.

1.  **Delete all Alert Rules:**
    *   Delete all alert rules you have created
2.  **Delete Resource Group:**

    ```bash
    az group delete --name iatd_labs_05_rg --yes
    ```

**Outcomes to be Achieved:**

*   Utilize Azure Network Watcher to diagnose network connectivity issues.
*   Analyze network traffic patterns using NSG Flow Logs and Traffic Analytics.
    .
*   Use Azure Resource Graph to query and explore Azure resources.
*   Monitor the health of Azure services using Azure Service Health.
*   Track and manage Azure costs using Azure Cost Management + Billing.
