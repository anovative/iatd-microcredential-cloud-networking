## IATD Microcredential Cloud Networking: Lab 7 - Incident Response and Troubleshooting in Azure Networking

**Objective:** This lab focuses on the practical aspects of incident response and troubleshooting common Azure networking issues. You'll learn how to respond to alerts, capture network traffic, verify NSG rules, adjust health probes, and ensure application availability while updating incident records.

**Estimated Time:** 90 - 120 minutes

**Prerequisites:**

1.  **Azure Subscription:** Requires an active Azure subscription. A free account is available at [https://azure.microsoft.com/free/](https://azure.microsoft.com/free/).
2.  **Solid knowledge of Azure Networking:** Familiarity with Virtual Networks, Subnets, Network Security Groups, Load Balancers, Application Gateway, Azure Front Door, and VPN Gateways.
3.  **Existing Azure Application Gateway:** You'll need an existing Azure Application Gateway.
4.  **Existing Azure Front Door:** You'll need an existing Azure Front Door.
5.  **Azure Monitor Logs Setup:** Ensure your Application Gateway and Front Door services are configured to send logs to a Log Analytics Workspace for analysis.
6.   **Site-To-Site VPN or ExpressRoute (Simulated):** Basic understanding of routing between on-premise and Azure.

### Lab Conventions

*   **Azure Cloud Shell:** All PowerShell and CLI interactions will occur within the Azure Cloud Shell environment.
*   **Azure Portal:** The Azure Portal will be used for visualization and configuration tasks.
*   **Naming Conventions:** Resource names follow the convention `iatd_labs_07_*`.
*   **Location:** Choose a consistent Azure region.

#### Resource Naming

*   **Resource Group:** `iatd_labs_07_rg`
*   **Virtual Network:** `iatd_labs_07_vnet`
*   **Application Gateway:** `iatd_labs_07_appgw`
*   **Azure Front Door:** `iatd_labs_07_frontdoor`
*   **Log Analytics Workspace:** (Your existing Log Analytics Workspace)
*    **VPN Gateway:** `iatd_labs_07_vpngw` (Simulated if used)

### Part 1: Responding to an Application Gateway Returning 503 Errors

1.  **Scenario:**
    *   You receive an alert from Azure Monitor indicating that your Application Gateway (`iatd_labs_07_appgw`) is returning 503 errors (Service Unavailable). This suggests that the backend servers are not responding to requests.

2.  **Investigating the Health Probe Failure (Portal):**
    *   Navigate to your Application Gateway in the Azure Portal.
    *   Select **Backend health** under **Monitoring**.
    *   Examine the health status of each backend server. Identify which servers are reporting as unhealthy and the reason for the failure (e.g., timeout, connection refused, HTTP status code).

3.  **Troubleshooting the Backend Servers:**
    *  Since you are receiving connection refused check NSG, UDR and Routing
    * Perform connectivity tests, it can be a DNS error or network related
    * Troubleshoot basic errors

### Part 2: Capturing Traffic to Diagnose VPN Connectivity Issues (Simulated)

1.  **Scenario:**
    *   Users are reporting intermittent connectivity issues to applications hosted in Azure from your on-premises network. You suspect there might be a problem with the Site-to-Site VPN connection.

2.  **Using Azure Network Watcher to Capture Traffic:**
    *   Search for "Network Watcher" in the Azure Portal.
    *   Select **Packet capture** under **Network diagnostic tools**.
    *   Configure a packet capture session:
        *   Subscription: Your subscription.
        *   Resource group: `iatd_labs_07_rg`.
        *   Target: Select one of the Backend Virtual Machines conected on Site-to-Site
        *   Create a filter to capture traffic that goes between VPN and your VM
        *   Storage location: Choose a storage account to save the packet capture file.

3.  **Analyzing the Packet Capture (Conceptual):**
    *   Once the packet capture is complete, you can download the .cap file and analyze it using a tool like Wireshark.
   *  You would have more details on the analysis like package loss, long times to respond e TCP errors.
   *Use those logs to address the Root Cause*

### Part 3: Verifying NSG Rules Blocking Traffic to a Load Balancer

1.  **Scenario:**
    *   Users are reporting that they cannot access a service fronted by an Azure Load Balancer. You suspect that an NSG rule might be inadvertently blocking the traffic.

2.  **Reviewing NSG Rules (Portal):**
    *   Identify the Network Security Group (NSG) that is associated with the Load Balancer´s Subnet
    *   Select **Effective security rules** under **Support + troubleshooting**
    *   Review all security rules and verify there isn't an explicit rule denying the connections

3.   **Using IP Flow Verify (Portal):**
        *   Search for "Network Watcher" in the Azure Portal.
        *   Select **IP flow verify** under **Network diagnostic tools**.
        *   Configure the test:
            *   Direction: `Inbound`
            *   Protocol: `TCP`
            *   Local IP address: The frontend IP address of your Load Balancer.
            *   Local port: The port that's not working.
            *   Remote IP address: `Your external IP address that should connect to the Load balancer.
            *   Remote port: `Random Open Port`
        *   The tests results are important here, if the traffic is blocked there will show which rule is blocking

### Part 4: Adjusting Health Probe Settings on Azure Front Door

1.  **Scenario:**
    *   Users are reporting that they are experiencing intermittent outages when accessing your application through Azure Front Door (`iatd_labs_07_frontdoor`). You suspect that the health probe settings might be too aggressive, causing Front Door to mark healthy backends as unhealthy.

2.  **Modifying Health Probe Settings (Portal):**
    *   Navigate to your Azure Front Door in the Azure Portal.
    * Click the "Origin groups"
    *Edit Configuration

    *Path: change the Path
    *Protocols: Can be changed to http
    *Method: HEAD, GET or POST
    *Interval: You can define the minutes between healtcheck

### Part 5: Restoring Access and Updating Incident Records

1.  **Verification:**
    * If all configurations and healthcheck are verified and OK, you can test the application and it should now working

2.  **Documenting Changes in Incident Records:**
    *   Create or update a runbook (a documented set of procedures) to record the steps taken to resolve the incident.
    *   Include the following information:
        *   Date and time of the incident.
        *   Description of the problem.
        *   Root cause analysis.
        *   Steps taken to resolve the incident.
        *   Configuration changes made.
        *   Verification steps.

### Post-Lab: Cleanup

To avoid incurring unnecessary costs, it's essential to clean up the resources created during this lab.

1.  **Delete Resource Group:**

    ```bash
    az group delete --name $RESOURCE_GROUP --yes
    ```

**Learning Outcomes:**

*   Successfully responded to an alert indicating a failed health probe on Application Gateway and identified potential causes.
*   Successfully used Azure Network Watcher to capture traffic to diagnose VPN connectivity issues.
*   Successfully verified NSG rules to ensure they were not blocking traffic to a load balancer.
*   Successfully adjusted health probe settings on Azure Front Door to restore traffic.
*   Successfully documented incident response steps in a runbook.
*   Demonstrated the ability to troubleshoot and resolve common Azure network issues, contributing to improved application availability.
