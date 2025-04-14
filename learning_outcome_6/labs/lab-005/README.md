## IATD Microcredential Cloud Networking: Lab 5 - Network Performance Monitoring and Troubleshooting

**Objective:** This lab focuses on the practical aspects of network performance monitoring and troubleshooting. You'll learn how to monitor round-trip time on an ExpressRoute circuit, measure throughput on a Site-to-Site VPN, check packet loss on a public internet link, and monitor error rates on a switch port.

**Estimated Time:** 75 - 90 minutes

**Prerequisites:**

1.  **Azure Subscription:** Requires an active Azure subscription. A free account is available at [https://azure.microsoft.com/free/](https://azure.microsoft.com/free/).
2.  **Basic knowledge of Azure Networking:** Familiarity with Virtual Networks, Subnets, VPN Gateways, ExpressRoute, and common networking concepts.
3.  **Simulated Network Components:** For this lab, we'll be *simulating* various network components. *You don't need real ExpressRoute circuits, Site-to-Site VPN connections, or access to physical switch ports.*
4.  **Azure Network Watcher:** Familiarity with Azure Network Watcher is helpful.

### Lab Conventions

*   **Azure Cloud Shell:** All PowerShell and CLI interactions will occur within the Azure Cloud Shell environment.
*   **Azure Portal:** The Azure Portal will be used for visualization and configuration tasks.
*   **Naming Conventions:** Resource names follow the convention `iatd_labs_05_*`.
*   **Location:** Choose a consistent Azure region.

#### Resource Naming

*   **Resource Group:** `iatd_labs_05_rg`
*   **Virtual Network:** `iatd_labs_05_vnet`
*   **ExpressRoute Circuit:** `iatd_labs_05_exr` (Simulated)
*   **VPN Gateway:** `iatd_labs_05_vpngw` (Simulated)
*   **VM-Monitor:** `iatd_labs_05_vm_monitor` (To simulate public internet test)

### Part 1: Monitoring Round-Trip Time on an ExpressRoute Circuit (Simulated)

1.  **Simulating an ExpressRoute Circuit (Conceptual):**
    *   Explain that an ExpressRoute circuit provides a dedicated, private connection between your on-premises network and Azure.
2.  **Setting the Scenario:** For the purposes of this lab, we will setup Azure Monitor.
        *   Simulate the condition to test all the process

3.  **Using Azure Monitor to Track Round-Trip Time**
    * Search for "Monitor" in the Azure Portal.
    * Select **Alerts** under **Monitoring**
    * Set the scope to the resource, subscription, or resource group you want to apply the alarm
        * Add a condition with a Custom Log Query
                Resources
                | where type == "microsoft.network/expressroutecircuits"
                | where name == "name"
                | where Properties contains "BGP"
                | project Properties
        *On the Actions Tab, create an Action Group
                Name: Create your Action Group Name
                Short Name: Create Short Name
        *Resource Group: Select your lab resource Group
        *On Notification Tab, select type Email/SMS/Push/Voice to simulate getting a report and alert.
        *Set a name for to alert and define how you want to solve it.

### Part 2: Measuring Throughput on a Site-to-Site VPN (Simulated)

1.  **Simulating a Site-to-Site VPN (Conceptual):**
    *   Explain that a Site-to-Site VPN creates a secure tunnel between your on-premises network and your Azure VNet over the public internet.

2.  **Simulating Throughput Measurement (Conceptual):**
    *   Testing requires a lot of throughput to test, that can be simulated using iperf

3. Azure Monitor: We will create a performance rule so we can use azure tooling to validate.
        *   You receive an alert from Azure Monitor indicating a VPN that have a low preformance from what is expected
        * Select **Alerts** under **Monitoring**
        * Set the scope to the resource, subscription, or resource group you want to apply the alarm
            * Add a condition with a Custom Log Query, in this case AzureMetrics
                * In Signal Logic:
                    * Metric Name: GatewayAverageLatency
                    * Aggregation Type: Average
                * After that, setup the Alert Rules with the appropriate threshhold.
        * Set the rest of the Alarm details and wait for metrics.

### Part 3: Checking Packet Loss on a Public Internet Link

1.  **The scenario:**
    * Diagnose and detect if there is call quality issues between the Web VM and your local computer or simulated network, and implement a basic diagnose from Azure

2.   **VM setup:** Deploy a low-cost VM to ping/evaluate the connection. To simulate Teams we need something to do this test
   * Select **Create** -> **Azure virtual machine**.
        *   **Basics Tab:**
            *   Subscription: Your subscription.
            *   Resource group: `iatd_labs_05_rg`
            *   Virtual machine name: `iatd_labs_05_vm_monitor`
            *   Region: Your region.
            *   Image: `Ubuntu Server 22.04 LTS`
            *   Size: Choose a size (e.g., Standard_B1ls).
            *   Username: `azureuser`
            *   Authentication type: `Password`
            *   Password: Set a password.
        *   **Networking Tab:**
            *   Virtual network: `iatd_labs_05_vnet`
            *   Subnet:  *`Select a subnet with Internet Access like` iatd_labs_05_subnet_web*
            *   Public IP:  *Create a new public IP address to allow external access*
            *   NIC network security group: `None`
        *   **Management Tab:**
            *   Disable boot diagnostics
    *   Click **Review + create**, then **Create**.

3.  **Packet Loss test**:
   * Install the appropriate tools on the VM, and check connectivity.
   * Run basic "ping google.com -t" and view if have package loss

### Part 4: Monitoring Error Rates on a Switch Port (Conceptual)

1.  **Explain Monitoring Switches and how you can monitor it from Azure**
   *You can configure Azure to monitor the devices, for this you need to deploy Azure Monitor Agent, and also configure the switches that are monitored*

2.  **Azure Network Monitoring**
    * Create a collector to obtain network information for the device
    * Create a flow for that conector with the parameters needed.
    * Check logs to verify if have high rates of errors.

### Post-Lab: Cleanup

To avoid incurring unnecessary costs, it's essential to clean up the resources created during this lab.

1.  **Delete Resource Group:**

    ```bash
    az group delete --name $RESOURCE_GROUP --yes
    ```

**Learning Outcomes:**

*   Explored how to use Azure Monitor Alerts to simulate round-trip time monitoring on an ExpressRoute circuit.
*   Understood methods for measuring throughput on a Site-to-Site VPN in Azure.
*   Learned how to check packet loss on a public internet link to diagnose call quality issues in Teams.
*   Learned how configure Network Monitoring on Azure Portal.
