## IATD Microcredential Cloud Networking: Lab 6 - Hybrid Connectivity with VPN Gateway and ExpressRoute

**Objective:** This lab covers hybrid networking solutions, focusing on Site-to-Site VPNs and Azure ExpressRoute. You'll learn about hybrid connectivity models, configure a basic Site-to-Site VPN connection (simulated), and understand the benefits and considerations of Azure ExpressRoute.

**Estimated Time:** 75 - 90 minutes

**Prerequisites:**

1.  **Azure Subscription:** Requires an active Azure subscription. A free account is available at [https://azure.microsoft.com/free/](https://azure.microsoft.com/free/).
2.  **Basic knowledge of Azure Networking:** Familiarity with Virtual Networks, Subnets, and VPN Gateways.
3.  **Understanding of VPNs (Conceptual):** Basic understanding of VPN concepts, IPsec, and IKE.
4.  **Understanding of ExpressRoute (Conceptual):** Knowledge of ExpressRoute circuits and peering types is helpful.

### Lab Conventions

*   **Azure Cloud Shell:** All PowerShell and CLI interactions will occur within the Azure Cloud Shell environment.
*   **Azure Portal:** The Azure Portal will be used for visualization and configuration tasks.
*   **Naming Conventions:** Resource names follow the convention `iatd_labs_06_*`.
*   **Location:** Choose a consistent Azure region.

#### Resource Naming

*   **Resource Group:** `iatd_labs_06_rg`
*   **Virtual Network:** `iatd_labs_06_vnet`
*   **Subnet-Gateway:** `iatd_labs_06_subnet_gateway`
*   **VPN Gateway:** `iatd_labs_06_vpngw` (Simulated)
*   **Local Network Gateway:** `iatd_labs_06_lng` (Simulated)
*   **Connection:** `iatd_labs_06_connection` (Simulated)

### Part 1: Hybrid Networking Basics (Conceptual)

1.  **Hybrid Connectivity Models:**

    ```
    +------------------------+                      +------------------------+
    |                        |                      |                        |
    |   On-premises Network  |                      |     Azure VNet         |
    |                        |                      |                        |
    +------------------------+                      +------------------------+
              |                                              |
              |                                              |
              v                                              v
    +------------------------+      Internet      +------------------------+
    |                        |<----------------->|                        |
    |    VPN Gateway        |                    |   Azure VPN Gateway   |
    |                        |                    |                        |
    +------------------------+                    +------------------------+
                                Site-to-Site VPN
    ```

    *   **Site-to-Site VPN:** A secure connection over the public internet between your on-premises network and Azure VNet. Suitable for smaller workloads and non-critical applications.
    *   **Point-to-Site VPN:** Allows individual clients (e.g., laptops) to connect to your Azure VNet over a VPN connection.
    *   **Azure ExpressRoute:** Provides a dedicated, private network connection between your on-premises network and Azure. Suitable for mission-critical applications requiring high bandwidth and low latency.

    ```
    +------------------------+                                  +------------------------+
    |                        |                                  |                        |
    |   On-premises Network  |                                  |     Azure VNet         |
    |                        |                                  |                        |
    +------------------------+                                  +------------------------+
              |                                                          |
              |                                                          |
              v                                                          v
    +------------------------+    Dedicated Private Connection    +------------------------+
    |                        |<--------------------------------->|                        |
    |  Enterprise Edge      |                                   |  Microsoft Edge       |
    |                        |                                   |                        |
    +------------------------+                                   +------------------------+
                                      ExpressRoute
    ```

2.  **Network Address Spaces Planning:**
    *   **Importance:** Ensuring that your Azure VNet address space does not overlap with your on-premises network address space.
    * **This lab will simulate a VNet connection, there are some checks and limitations for connection to resources in real time*

3.  **Connection Type Comparison:**

    * The validation must be from point to point since we are simulating*
    *   Discuss the key differences between Site-to-Site VPN and ExpressRoute:
        *   **Connectivity:** Public internet vs. dedicated private connection.
        *   **Bandwidth:** Lower (up to 10 Gbps) vs. Higher (up to 100 Gbps).
        *   **Latency:** Higher vs. Lower.
        *   **Security:** Public internet security vs. dedicated private connection.
        *   **Cost:** Lower vs. Higher.

### Part 2: Configuring a Site-to-Site VPN Connection (Simulated)

1.  **Create a Resource Group:**

    ```bash
    RESOURCE_GROUP="iatd_labs_06_rg"
    LOCATION="eastus" # Your Region

    az group create --name $RESOURCE_GROUP --location $LOCATION
    ```

2.  **Create a Virtual Network and Gateway Subnet:**

    ```bash
    VNET_NAME="iatd_labs_06_vnet"
    SUBNET_GATEWAY_NAME="GatewaySubnet"  # REQUIRED name for VPN Gateway subnet
    ADDRESS_PREFIX="172.16.0.0/16"
    SUBNET_GATEWAY_PREFIX="172.16.0.0/24"

    az network vnet create \
        --resource-group $RESOURCE_GROUP \
        --name $VNET_NAME \
        --address-prefixes $ADDRESS_PREFIX \
        --location $LOCATION

    az network vnet subnet create \
        --resource-group $RESOURCE_GROUP \
        --vnet-name $VNET_NAME \
        --name $SUBNET_GATEWAY_NAME \
        --address-prefix $SUBNET_GATEWAY_PREFIX
    ```

3.  **Simulating VPN Gateway and Local Network Gateway (Conceptual):**

    *Explain that for a real connection there is some points to have

        * There is a Public IP to work as VPN
        * There is a VPN Conection, this requires the public IP and a Share Key.
        * Local Network Gateway on Azure.
        * The onPremises VPN Device must have the parameters
    *For the purpose of this lab we will validate the requirements, since it´s required a real VPN Device that is beyond the scope of this*

### Part 3: Setting up basic connection

1.  **Review the VPN Devices requirements**
     *The requirements for the device can be seen here https://learn.microsoft.com/en-us/azure/vpn-gateway/vpn-gateway-aboutvpn-devices*

2.  **Setup Public IP Adress to configure your Device**
 * To Setup is device review the steps here https://learn.microsoft.com/en-us/azure/vpn-gateway/tutorial-site-to-site-vpn-aws*

### Part 4: Understanding ExpressRoute

1.  **ExpressRoute Circuit Provisioning (Conceptual):**
   This step is important to implement real connectivity
    *   Explain that provisioning an ExpressRoute circuit involves working with a connectivity provider to establish a physical connection between your on-premises network and Azure.
    *   Discuss the different ExpressRoute circuit bandwidth options.

2.  **ExpressRoute Peering Configuration (Conceptual):**
 This requires a configured connection to take validation of the settings
    *You will need a configured peer route (Private or Public) to complete this*

3.  **Redundancy Setup (Conceptual):**

    *   Explain that ExpressRoute provides redundancy through dual circuits and multiple peering locations.
You need to plan and setup the redundacy of both conections, its important to consider location and performance*

4.  **Performance Optimization (Conceptual):**
     * You can config some configuration to get the performance well configured, we would like to discuss about TCP window size and Jumbo Frames https://learn.microsoft.com/en-us/azure/expressroute/expressroute-design#mtu*

### Post-Lab: Cleanup

To avoid incurring unnecessary costs, it's essential to clean up the resources created during this lab.

1.  **Delete Resource Group:**

    ```bash
    az group delete --name $RESOURCE_GROUP --yes
    ```

**Learning Outcomes:**

*   Understood different hybrid connectivity models (Site-to-Site VPN, Point-to-Site VPN, ExpressRoute).
*   Understood the importance of network address space planning in hybrid environments.
*   Learned how to configure a basic Site-to-Site VPN connection (simulated).
*   Understood the key components of ExpressRoute, including circuit provisioning and peering configuration.
*   Learned about redundancy options and performance optimization techniques for ExpressRoute.
*   Understood security requirements for hybrid connections
