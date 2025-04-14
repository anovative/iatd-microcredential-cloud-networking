## IATD Microcredential Cloud Networking: Lab 4 - Responding to Network Incidents and Maintaining Application Availability

**Objective:** This lab simulates real-world incident response scenarios related to Azure networking. You'll learn how to respond to alerts, diagnose routing issues, update load balancer rules, and ensure application availability while documenting the remediation steps in runbooks.

**Estimated Time:** 75 - 90 minutes

**Prerequisites:**

1.  **Azure Subscription:** Requires an active Azure subscription. A free account is available at [https://azure.microsoft.com/free/](https://azure.microsoft.com/free/).
2.  **Basic knowledge of Azure Networking:** Familiarity with Virtual Networks, Subnets, Load Balancers, Application Gateway, and Azure Front Door.
3.  **Existing Azure Application Gateway:** You'll need an existing Azure Application Gateway.
4.  **Existing Azure Front Door:** You'll need an existing Azure Front Door.
5.  **Azure Monitor Logs Setup:** Ensure your Application Gateway and Front Door services are configured to send logs to a Log Analytics Workspace for analysis.

### Lab Conventions

*   **Azure Cloud Shell:** All PowerShell and CLI interactions will occur within the Azure Cloud Shell environment.
*   **Azure Portal:** The Azure Portal will be used for visualization and configuration tasks.
*   **Naming Conventions:** Resource names follow the convention `iatd_labs_04_*`.
*   **Location:** Choose a consistent Azure region.

#### Resource Naming

*   **Resource Group:** `iatd_labs_04_rg`
*   **Existing Virtual Network:** (Assumed)
*   **Application Gateway:** `iatd_labs_04_appgw`
*   **Azure Front Door:** `iatd_labs_04_frontdoor`
*   **Log Analytics Workspace:** (Assumed)

### Part 1: Responding to an Application Gateway Health Probe Failure

1.  **Scenario:**
    *   You receive an alert from Azure Monitor indicating a failed health probe on your Application Gateway (`iatd_labs_04_appgw`). This suggests that the backend servers are not responding to health checks, and the Application Gateway may be directing traffic to unhealthy instances.

2.  **Investigating the Health Probe Failure (Portal):**
    *   Navigate to your Application Gateway in the Azure Portal.
    *   Select **Backend health** under **Monitoring**.
    *   Examine the health status of each backend server. Identify which servers are reporting as unhealthy and the reason for the failure (e.g., timeout, connection refused).

3.  **Troubleshooting the Backend Servers (Conceptual):**
    *   The next step would be to investigate the backend servers to determine the cause of the health probe failures.
        * Example cases, troubleshoot VM, check Web application service is running.

### Part 2: Checking Azure Front Door Logs for Routing Issues

1.  **Scenario:**
    *   Users report that they are unable to access a specific part of your application through Azure Front Door (`iatd_labs_04_frontdoor`). You suspect that there may be a routing misconfiguration.

2.  **Accessing Azure Front Door Logs (Portal):**

    *   Navigate to your Azure Front Door in the Azure Portal.
    *   Select **Diagnostic settings** under **Monitoring**.
    *   Verify diagnostic settings are enabled and configured to send logs to a Log Analytics workspace.
    *   Select **Logs** under **Monitoring** to access Log Analytics.

3.  **Analyzing Logs (KQL Query):**
    *   Run a Kusto Query Language (KQL) query to analyze the AzureFrontDoorAccessLog table to identify the routing issues. A possible scenario is the backend is failing
    *   Example Query:
        ```kusto
        AzureFrontDoorAccessLog
        | where TimeGenerated > ago(1h)
        | where RuleName == "<Your Rule Name>"
        | summarize count() by bin(TimeGenerated, 5m), HttpStatusCode
        ```
    *   Analyze the results to identify common HTTP status codes, identify any routing configurations are in failure, and determine the root cause of the routing problem.

### Part 3: Updating a Load Balancer Rule to Restore Traffic Flow

1.  **Scenario:**
    *  Because the problem came from an backend server, and has a high traffic to that server. We will take it out of the pool to restore Traffic Flow to avoid the failure of more requests.
    *   Based on the previous analysis, you determine that a load balancer rule is directing traffic to an unhealthy backend pool.

2.  **Updating the Load Balancer Rule (Portal):**

    *   Search for "Load balancers" and select your Load Balancer.
    *   Select **Backend pools** under **Settings**.
    *   Edit the load balancer rules to address the issue.
    *   For example, you might:
        *   Remove the unhealthy backend server from the backend pool.
        *   Adjust the load distribution settings to direct traffic to healthy servers.

3.  **Implementing new load balancer (Portal):**

    *   Because you removed the bad server, this must be replaced for a new one to keep the HA
    *   Navigate to your Azure Load Balancer in the Azure Portal.
    *   Select **Backend pools** under **Settings**.
    *   Add the new server IP as backend.
    *    Select Health probe and test it.
    * Click "Save"
    *If the health probe fails troubleshoot VM*

### Part 4: Documentation and Verification

1.  **Documenting Changes in Runbooks:**
    *   Create or update a runbook (a documented set of procedures) to record the steps taken to resolve the incident.
    *   Include the following information:
        *   Date and time of the incident.
        *   Description of the problem.
        *   Root cause analysis.
        *   Steps taken to resolve the incident.
        *   Configuration changes made.
        *   Verification steps.

2.  **Verifying Application Gateway Handles Requests (Portal):**
    *   Navigate to your Application Gateway in the Azure Portal.
    *   Select **Backend health** under **Monitoring**.
    *   Verify that all backend servers are now reporting as healthy.
    * You can validate the access in the metrics blade too

### Post-Lab: Cleanup

To avoid incurring unnecessary costs, it's essential to clean up the resources created during this lab.

1.  **Delete Resource Group:**

    ```bash
    az group delete --name $RESOURCE_GROUP --yes
    ```

**Learning Outcomes:**

*   Successfully responded to an Azure Monitor alert indicating a failed health probe on Application Gateway.
*   Successfully checked Azure Front Door logs to identify the root cause of routing issues.
*   Successfully updated a load balancer rule to restore traffic flow and ensure application availability.
*   Successfully documented incident response steps in a runbook.
*   Demonstrated the ability to troubleshoot and resolve common Azure network issues.
*   Understood the importance of proactive monitoring and well-documented procedures for maintaining application availability.
