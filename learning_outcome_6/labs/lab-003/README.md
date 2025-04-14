## IATD Microcredential Cloud Networking: Lab 3 - Network Change Management and Impact Assessment

**Objective:** This lab simulates a change management process within Azure networking. You'll evaluate requests for new VNet peering, assess the potential impact of new load balancing rules, apply and verify NSG rules, and analyze the performance impact of a new ExpressRoute peering.

**Estimated Time:** 75 - 90 minutes

**Prerequisites:**

1.  **Azure Subscription:** Requires an active Azure subscription. A free account is available at [https://azure.microsoft.com/free/](https://azure.microsoft.com/free/).
2.  **Basic knowledge of Azure Networking:** Familiarity with Virtual Networks, Subnets, Network Security Groups, Route Tables, VNet Peering, Load Balancers, and ExpressRoute.

### Lab Conventions

*   **Azure Cloud Shell:** All PowerShell and CLI interactions will occur within the Azure Cloud Shell environment.
*   **Azure Portal:** The Azure Portal will be used for visualization and configuration tasks.
*   **Naming Conventions:** Resource names follow the convention `iatd_labs_03_*`.
*   **Location:** Choose a consistent Azure region.

#### Resource Naming

*   **Resource Group:** `iatd_labs_03_rg`
*   **Existing Virtual Network:** `iatd_labs_03_existing_vnet`
*   **New Virtual Network:** `iatd_labs_03_new_vnet`
*   **Load Balancer:** `iatd_labs_03_lb`
*   **Network Security Group:** `iatd_labs_03_nsg`
*   **ExpressRoute Circuit:** `iatd_labs_03_exr` (Simulated)

### Part 1: Evaluating a Request to Add a New VNet Peering

1.  **Scenario:**
    *   A developer team requests to peer a new VNet (`iatd_labs_03_new_vnet`, `10.10.0.0/16`) with an existing VNet (`iatd_labs_03_existing_vnet`, `10.0.0.0/16`) to enable communication between their application and existing services.

2.  **Gathering Information (Conceptual):**
    *   Before approving the request, you need to gather the following information:
        *   **Business justification:** Why is the peering needed? What services need to communicate?
        *   **Security requirements:** What security policies need to be enforced on the peering?
        *   **Network impact:** What is the potential impact on network bandwidth and latency?

3.  **Simulating VNet Peering (Conceptual):**
    *Since we are only simulating, there will be no changes*
    *   You would typically create the peering using the Azure Portal or Azure CLI.

4.  **Security Considerations:**
    * Make sure that all traffic must route throught a firewall and all security requisites are addressed

### Part 2: Assessing a New Load Balancing Rule

1.  **Scenario:**
    *   A new load balancing rule is proposed for the Azure Load Balancer (`iatd_labs_03_lb`). This rule will direct a significant portion of incoming traffic to a new set of backend servers.

2.  **Assessing Potential Impact (Conceptual):**
    *   Before implementing the rule, you need to assess whether it could overload the backend servers.
    *   This involves:
        *   **Estimating the expected traffic volume:** How much traffic will the new rule direct to the backend servers?
        *   **Analyzing the capacity of the backend servers:** What is the maximum traffic load that the servers can handle?
        *   **Monitoring current server load:** Use Azure Monitor to check the CPU utilization, memory usage, and network traffic of the existing backend servers.

3.  **Simulating Load Increase:**
    *This simmulation cannot be done automatically, it needs to be done manually. There are no active traffic being simulated, it's only to check the configurations are according to therequirements*

### Part 3: Applying a New Azure NSG Rule and Verifying Traffic Flow

1.  **Scenario:**
    *   A new security threat has been identified that requires you to block traffic from a specific IP address range (`203.0.113.0/24`) to a specific port (8080) on the application subnet (`10.0.2.0/24`).

2.  **Create a Network Security Group Rule (Portal):**

    *   Search for "Network security groups" and select your NSG (`iatd_labs_03_nsg`).
    *   Select **Inbound security rules** under **Settings**.
    *   Click **Add**.
        *   Source: `IP Addresses`
        *   Source port range: `*`
        *   Destination: `Any`
        *   Destination port range: `8080`
        *   Protocol: `Any`
        *   Action: `Deny`
        *   Priority: Choose the appropriate value (e.g., 100).
        *   Name: `BlockMaliciousIP`
        *Source IP Adress: `203.0.113.0/24`
        *   Click **Add**.

3.  **Verify Traffic Flow (Conceptual):**
    *Testing this scenario requires the use of tools like the "IP flow verify" to verify the connections, since its needed a test flow to verify what you want.*

### Part 4: Reviewing the Impact of a New ExpressRoute Peering

1.  **Scenario:**
    *   A new ExpressRoute peering has been established to improve connectivity to a specific on-premises location.

2.  **Analyzing Latency (Conceptual):**
    *   After the peering is established, you need to review whether it has improved latency as expected.
    * Azure provide some basic tooling on Networking Watcher to evaluate latency
    *   Tools like Azure Network Watcher can be used to measure latency between Azure VMs and on-premises endpoints.

### Post-Lab: Cleanup

To avoid incurring unnecessary costs, it's essential to clean up the resources created during this lab.

1.  **Delete Resource Group:**

    ```bash
    az group delete --name $RESOURCE_GROUP --yes
    ```

**Learning Outcomes:**

*   Understood the key considerations when evaluating a request to add a new VNet peering.
*   Learned how to assess the potential impact of new load balancing rules.
*   Successfully created and applied an Azure NSG rule to block traffic from a specific IP address range.
*   Explored methods for reviewing whether a new ExpressRoute peering improved latency.
