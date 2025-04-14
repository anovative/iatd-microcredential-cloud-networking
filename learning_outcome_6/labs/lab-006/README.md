## IATD Microcredential Cloud Networking: Lab 6 - Capacity Planning and Optimization in Azure Networking

**Objective:** This lab focuses on capacity planning and optimization techniques for Azure networking. You'll learn how to analyze Azure Monitor logs, forecast bandwidth needs, upgrade ExpressRoute circuits, and leverage Azure Front Door for global traffic management.

**Estimated Time:** 75 - 90 minutes

**Prerequisites:**

1.  **Azure Subscription:** Requires an active Azure subscription. A free account is available at [https://azure.microsoft.com/free/](https://azure.microsoft.com/free/).
2.  **Basic knowledge of Azure Networking:** Familiarity with Virtual Networks, Subnets, Load Balancers, Application Gateway, Azure Front Door, ExpressRoute, and VPN Gateways.
3.  **Existing Azure Services:** Access to an existing Azure web app and a Log Analytics Workspace is beneficial (though the core concepts can be learned without them).

### Lab Conventions

*   **Azure Cloud Shell:** All PowerShell and CLI interactions will occur within the Azure Cloud Shell environment.
*   **Azure Portal:** The Azure Portal will be used for visualization and configuration tasks.
*   **Naming Conventions:** Resource names follow the convention `iatd_labs_06_*`.
*   **Location:** Choose a consistent Azure region.

#### Resource Naming

*   **Resource Group:** `iatd_labs_06_rg`
*   **Virtual Network:** `iatd_labs_06_vnet`
*   **Web App:** `iatd_labs_06_webapp` (Your existing web app)
*   **Log Analytics Workspace:** (Your existing Log Analytics Workspace)
*   **ExpressRoute Circuit:** `iatd_labs_06_exr` (Simulated)
*   **Site-to-Site VPN Gateway:** `iatd_labs_06_vpngw` (Simulated)
*   **Azure Front Door:** `iatd_labs_06_frontdoor` (If used)

### Part 1: Reviewing Azure Monitor Logs for Peak Traffic Times

1.  **Scenario:**
    *   You need to understand the peak traffic times for an existing web app (`iatd_labs_06_webapp`) to plan for scaling and maintenance.

2.  **Accessing Azure Monitor Logs (Portal):**

    *   Navigate to your Log Analytics Workspace in the Azure Portal.
    *   Select **Logs** under **General**.

3.  **Analyzing Web App Traffic (KQL Query):**

    *   Run a Kusto Query Language (KQL) query to analyze the traffic patterns for your web app. Replace placeholders with your actual resource names and log table names.
    *  Replace the names and test

        ```kusto
        AppServiceHTTPLogs
        | where TimeGenerated > ago(7d) // Analyze the last 7 days
        | summarize count() by bin(TimeGenerated, 1h) // Group by hour
        | render timechart
        ```

    *   Analyze the resulting time chart to identify the hours with the highest traffic volume. This will inform your decisions about when to schedule maintenance or scale up resources.

### Part 2: Forecasting Bandwidth Needs for a New Branch

1.  **Scenario:**
    *   A new branch office is being connected to your Azure VNet via a Site-to-Site VPN (`iatd_labs_06_vpngw`). You need to forecast the bandwidth needs to ensure adequate performance.

2.  **Gathering Information (Conceptual):**
    *   To forecast bandwidth needs, you need to gather the following information:
        *   **Number of users in the new branch:** How many employees will be using the VPN connection?
        *   **Typical applications used:** What applications will the users be accessing over the VPN?
        *   **Estimated data usage per user:** How much data will each user consume per day?
        *   **Peak usage times:** When will the highest traffic volume occur?

3.  **Bandwidth Calculation Example:**

    * Example with 100 employes and basic usage:
    * Everage bandwidth = 100 Employees * 10Mbps

### Part 3: Upgrading an ExpressRoute Circuit for Anticipated Growth (Conceptual)

1.  **Scenario:**
    *   Your organization anticipates significant growth in the coming year, and you need to ensure that your ExpressRoute circuit (`iatd_labs_06_exr`) can handle the increased traffic.

2.  **Analyzing Current Bandwidth Utilization:**
 *Since you have an expressroute circuit you can view its performance over time and use forecasting*
    *   Use Azure Monitor to track the bandwidth utilization of your existing ExpressRoute circuit. Identify the peak utilization and the average utilization.
         *You can implement a rule to report that value*

3.  **Upgrading the ExpressRoute Circuit (Conceptual):**
    *   To upgrade the ExpressRoute circuit, you would typically:
        *   Contact your ExpressRoute provider.
        *   Request an upgrade to a higher bandwidth tier (e.g., from 1 Gbps to 10 Gbps).
        *   Coordinate the upgrade with your provider to minimize downtime.

### Part 4: Using Azure Front Door for Global Traffic Management

1.  **Scenario:**
    *   You are currently using regional load balancers to distribute traffic to your web application. You want to improve performance and availability for users around the world by using Azure Front Door (`iatd_labs_06_frontdoor`).

2.  **Setting up Azure Front Door (Conceptual):**

    * Configure AFD to forward traffic to the resources and locations.
    * Change local loadbalancers from global to local to avoid multiple points.
    * Deploy Front Door on global to improve performance

3.  **Benefits of Azure Front Door:**

    *   **Global traffic management:** Directs users to the closest available backend, improving performance.
    *   **SSL offloading:** Offloads SSL encryption and decryption to the edge, reducing the load on backend servers.
    *   **Web Application Firewall (WAF):** Provides protection against common web exploits.
    *   **Caching:** Caches static content at the edge, reducing latency and improving performance.

### Post-Lab: Cleanup

To avoid incurring unnecessary costs, it's essential to clean up the resources created during this lab.

1.  **Delete Resource Group:**

    ```bash
    az group delete --name $RESOURCE_GROUP --yes
    ```

**Learning Outcomes:**

*   Learned how to review Azure Monitor logs to identify peak traffic times for a web app.
*   Understood the process of forecasting bandwidth needs for a new branch connecting via Site-to-Site VPN.
*   Explored the steps involved in upgrading an ExpressRoute circuit for anticipated growth.
*   Understood how to leverage Azure Front Door for global traffic management.
