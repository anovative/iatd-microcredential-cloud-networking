## IATD Microcredential Cloud Networking: Lab 2 - Monitoring, Maintaining, and Documenting Azure Network Security

**Objective:** This lab focuses on the practical aspects of monitoring, maintaining, and documenting Azure network security configurations. You'll learn how to monitor VPN latency, update firewall rules, and document Azure Front Door rules for simplified troubleshooting.

**Estimated Time:** 75 - 90 minutes

**Prerequisites:**

1.  **Azure Subscription:** Requires an active Azure subscription. A free account is available at [https://azure.microsoft.com/free/](https://azure.microsoft.com/free/).
2.  **Basic knowledge of Azure Networking:** Familiarity with Virtual Networks and Network Security Groups.
3.  **Basic understanding of Site-to-Site VPNs:** Conceptual understanding of how Site-to-Site VPNs work. *Note: You don't need a real Site-to-Site VPN connection for this lab.*
4.  **Basic understanding of Azure Firewall:** While not strictly required, some familiarity with Azure Firewall is helpful.
5.  **Basic understanding of Azure Front Door:** While not strictly required, some familiarity with Azure Front Door is helpful.

### Lab Conventions

*   **Azure Cloud Shell:** All PowerShell and CLI interactions will occur within the Azure Cloud Shell environment.
*   **Azure Portal:** The Azure Portal will be used for visualization and configuration tasks.
*   **Naming Conventions:** Resource names follow the convention `iatd_labs_02_*`.
*   **Location:** Choose a consistent Azure region.

#### Resource Naming

*   **Resource Group:** `iatd_labs_02_rg`
*   **Virtual Network:** `iatd_labs_02_vnet`
*   **VPN Gateway:** `iatd_labs_02_vpngw` (Simulated)
*   **Azure Firewall:** `iatd_labs_02_azurefirewall` (if used)
*   **Azure Front Door:** `iatd_labs_02_frontdoor` (if used)

### Part 1: Monitoring Latency on a Site-to-Site VPN (Simulated)

1.  **Simulating a Site-to-Site VPN (Conceptual):**
    *   Explain that a Site-to-Site VPN creates a secure tunnel between your on-premises network and your Azure VNet.
    *   We will *not* be setting up a real Site-to-Site VPN connection in this lab. We will focus on *simulating* the monitoring aspects.
    *   We will assume VPN Gateway exists to better illustrate the Azure Monitoring and Logging.

2.  **Azure Network Watcher Connection Monitor:**
    * Search for "Network Watcher" in the Azure Portal.
    * Select **Connection Monitor** under **Monitoring**
    * Configure the test:
        * Name: <ConnetcionMonitor Name>
        * Subscription: Select your subscription
        * Region: Select the Lab´s Azure Region
    * Configure Test groups:
        *  Add Source:
            * Region: Select the Lab´s Azure Region
            * Resources: List all the resources in Azure Resources, if you have a VM there or select any other resource to simulate an OnPrem machine. Select some
        *  Add Destination:
            * Endpoints: Select External Adresses
            * Add one sample destination like "google.com" or "bing.com"
        *  Add Test Configuration:
            * Protocol: Select "TCP"
            * Destination Port: Select "80"
            * Test Frequency: Select "30 seconds"
            * Success Thresholds: Checks %
    * The result should show success

### Part 2: Updating Firewall Rules to Address New Security Threats

*Note: You can skip the step of deploying a real Azure Firewall if you don't have one already. You can still demonstrate the process of updating rules without a live firewall.*

1.  **Identifying a New Security Threat (Simulated):**
    *   Imagine you receive an alert from your security team about a new malware campaign originating from a specific IP address range (e.g., `203.0.113.0/24`).

2.  **Updating Azure Firewall Rule (Portal):**
    * If you didn´t deploy one in the last exercise, you can deploy one, if not skip this first setp
    *   Search for "Firewalls" and select your Azure Firewall (`iatd_labs_02_azurefirewall`).
    *   Select **Rules** under **Settings**.
    *   **Network Rule Collection:**
        * Select a rule Collection or create another new rule to drop the traffic from the indicated IP range
        * Click **Add rule**.
        *   Name: `BlockMaliciousIPRange`
        *   Source type: `IP Address`
        *   Source: `203.0.113.0/24`
        *   Destination type: `Any`
        *   Destination: `*`
        *   Protocol: `Any`
        *   Port: `*`
        *   Action: `Deny`
        *   Click **Add**.
    *Click "Save"

3.  **Verify the updated rule (Portal):**
    *Click on diagnostic settings to export logs to Log Analytics to check block flows*

### Part 3: Documenting Azure Front Door Rules

*Note: This part assumes that Azure Front Door has been configured*

1.  **Access Azure Front Door:**
    *   Search for "Front Doors and CDN profiles" and select your Azure Front Door service (`iatd_labs_02_frontdoor`).
    *  Click "Front Door Designer"
    *  Click the "+" icon on FrontDoor Designer and review all rules

2.  **Documenting Rules:**

    *  Record configuration steps or just apply a screen capture
    *Create a Word or MarkDown File to register all the data or parameters of each rule*
    *Rule Types: Routing rules, Firewall policies*

### Post-Lab: Cleanup

To avoid incurring unnecessary costs, it's essential to clean up the resources created during this lab.

1.  **Delete Resource Group:**

    ```bash
    az group delete --name $RESOURCE_GROUP --yes
    ```

**Learning Outcomes:**

*   Learned how to use Azure Network Watcher to monitor latency in a simulated Site-to-Site VPN connection.
*   Successfully updated Azure Firewall rules to address new security threats.
*   Learned how to properly document Azure Front Door rules for simplified troubleshooting and knowledge sharing.
*   Understood the importance of continuous monitoring, proactive maintenance, and accurate documentation in maintaining a secure and reliable Azure network.
