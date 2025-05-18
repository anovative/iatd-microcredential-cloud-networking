## IATD Microcredential Cloud Networking: Lab 1 - Planning, Deploying, and Optimizing a Hybrid Network with Azure ExpressRoute

**Objective:** This lab focuses on the practical aspects of planning, deploying, and optimizing a hybrid network using Azure ExpressRoute. You'll simulate the connection between an on-premises data centre and Azure, configure a multi-tier application's VNet, monitor network performance, and implement optimizations to address bandwidth constraints.

**Estimated Time:** 90 - 120 minutes

**Prerequisites:**

1.  **Azure Subscription:** Requires an active Azure subscription. A free account is available at [https://azure.microsoft.com/free/](https://azure.microsoft.com/free/).
2.  **Basic knowledge of Azure Networking:** Familiarity with Virtual Networks, Subnets, Network Security Groups, and Route Tables.
3.  **Understanding of ExpressRoute (Conceptual):** This lab focuses on *simulating* ExpressRoute. You don't need a real ExpressRoute circuit.

### Lab Conventions

*   **Azure Cloud Shell:** All PowerShell and CLI interactions will occur within the Azure Cloud Shell environment.
*   **Azure Portal:** The Azure Portal will be used for visualization and configuration tasks.
*   **Naming Conventions:** Resource names follow the convention `iatd_labs_01_*`.
*   **IP Address Range:** The `172.16.0.0/16` range will be used for the Azure VNet. The `172.16.100.0/24` range will represent the on-premises network (following the standardized IP addressing scheme).
*   **Location:** Choose a consistent Azure region.

#### Resource Naming

*   **Resource Group:** `iatd_labs_01_rg`
*   **Virtual Network:** `iatd_labs_01_vnet`
*   **Subnet-Web:** `iatd_labs_01_subnet_web`
*   **Subnet-App:** `iatd_labs_01_subnet_app`
*   **Subnet-DB:** `iatd_labs_01_subnet_db`
*   **Route Table - AppSubnet:** `iatd_labs_01_routetable_app`
*   **VM-Web-01:** `iatd_labs_01_vm_web_01`
*   **VM-App-01:** `iatd_labs_01_vm_app_01`
*   **VM-DB-01:** `iatd_labs_01_vm_db_01`

### Part 1: Planning a Hybrid Network and Simulating ExpressRoute

1.  **Planning the Address Space:**
    *   **Azure VNet:** `172.16.0.0/16`
    *   **On-Premises Network:** `172.16.100.0/24`
    *   Ensure that these address spaces do not overlap.

2.  **Subnet Planning:**
    *   `iatd_labs_01_subnet_web`: `172.16.1.0/24` (Public-facing web tier)
    *   `iatd_labs_01_subnet_app`: `172.16.2.0/24` (Application tier)
    *   `iatd_labs_01_subnet_db`: `172.16.3.0/24` (Database tier)

3.  **Simulating ExpressRoute (Conceptual):**
    *   Discuss the key components of ExpressRoute:
        *   **ExpressRoute Circuit:** Represents the physical connection between your on-premises network and Azure.
        *   **Private Peering:** Enables private connectivity to Azure VMs and cloud services.
    *   Explain that we won't be setting up a real ExpressRoute circuit in this lab. Instead, we will simulate the connectivity by:
        *   Creating a Route Table to represent the ExpressRoute connection.
        *   Manually adding routes to simulate traffic flow.

### Part 2: Deploying the Azure Virtual Network and Subnets

1.  **Create a Resource Group:**

    ```bash
    RESOURCE_GROUP="iatd_labs_01_rg"
    LOCATION="eastus" # Your Region

    az group create --name $RESOURCE_GROUP --location $LOCATION
    ```

2.  **Create a Virtual Network with Subnets:**

    ```bash
    VNET_NAME="iatd_labs_01_vnet"
    SUBNET_WEB_NAME="iatd_labs_01_subnet_web"
    SUBNET_APP_NAME="iatd_labs_01_subnet_app"
    SUBNET_DB_NAME="iatd_labs_01_subnet_db"
    ADDRESS_PREFIX="172.16.0.0/16"
    SUBNET_WEB_PREFIX="172.16.1.0/24"
    SUBNET_APP_PREFIX="172.16.2.0/24"
    SUBNET_DB_PREFIX="172.16.3.0/24"

    az network vnet create \
        --resource-group $RESOURCE_GROUP \
        --name $VNET_NAME \
        --address-prefixes $ADDRESS_PREFIX \
        --location $LOCATION \
        --subnet-name $SUBNET_WEB_NAME \
        --subnet-prefixes $SUBNET_WEB_PREFIX

    az network vnet subnet create \
        --resource-group $RESOURCE_GROUP \
        --vnet-name $VNET_NAME \
        --name $SUBNET_APP_NAME \
        --address-prefix $SUBNET_APP_PREFIX

    az network vnet subnet create \
        --resource-group $RESOURCE_GROUP \
        --vnet-name $VNET_NAME \
        --name $SUBNET_DB_NAME \
        --address-prefix $SUBNET_DB_PREFIX
    ```

3.  **Create App and DB Virtual Machines (Portal):**
    *   Create `iatd_labs_01_vm_app_01` in `iatd_labs_01_subnet_app` (no public IP).
    *   Create `iatd_labs_01_vm_db_01` in `iatd_labs_01_subnet_db` (no public IP).
    *   Create `iatd_labs_01_vm_web_01` in `iatd_labs_01_subnet_web` (with a public IP).

### Part 3: Simulating On-Premises Connectivity with Route Tables

1.  **Create a Route Table for App Subnet:**

    ```bash
    ROUTE_TABLE_NAME="iatd_labs_01_routetable_app"

    az network route-table create \
        --resource-group $RESOURCE_GROUP \
        --name $ROUTE_TABLE_NAME \
        --location $LOCATION
    ```

2.  **Add a Route to the Route Table (Simulating ExpressRoute):**

    ```bash
    ROUTE_TABLE_NAME="iatd_labs_01_routetable_app"
    ONPREM_NEXT_HOP="172.16.0.100" #Replace with IP of your simulated on-prem router

    az network route-table route create \
        --resource-group $RESOURCE_GROUP \
        --name RouteToOnPrem \
        --route-table-name $ROUTE_TABLE_NAME \
        --address-prefix 172.16.100.0/24 \
        --next-hop-type VirtualAppliance \
        --next-hop-ip-address $ONPREM_NEXT_HOP
    ```

3.  **Associate the Route Table with the App Subnet:**

    ```bash
    az network vnet subnet update \
        --resource-group $RESOURCE_GROUP \
        --vnet-name $VNET_NAME \
        --name $SUBNET_APP_NAME \
        --route-table $ROUTE_TABLE_NAME
    ```

### Part 4: Monitoring VNet Performance with Azure Monitor

1.  **Enable VM Insights (Portal):**
    *   Navigate to the `iatd_labs_01_vm_app_01` VM in the Azure Portal.
    *   Select **Insights** under **Monitoring**.
    *   If prompted, configure the workspace to a Log Analytics Workspace.

2.  **Examine Key Metrics (Portal):**
    *   Navigate to the `iatd_labs_01_vnet` VNet in the Azure Portal.
    *   Select **Metrics** under **Monitoring**.
    *   Explore the following metrics:
        *   **Bytes In:** Total number of bytes received on all VMs in the VNet.
        *   **Bytes Out:** Total number of bytes sent by all VMs in the VNet.
    *   Observe the trend of these metrics over a period of time (e.g., 1 hour).

3.  **Setting up Maintenance Windows (Conceptual):**
     *  For demonstration reasons we cannot configure an actual maintenance window, this would require a lot of overhead to setup a sample system
    *   Discuss the importance of scheduling updates and maintenance during off-peak hours to minimize disruption.
    *   Mention that Azure Update Management can be used to automate the update process.

### Part 5: Optimizing ExpressRoute Bandwidth

1.  **Simulating Increased Traffic:**

    *   Imagine that after a new office is opened, the network team notes that there is an increase in network traffic

2.  **Identifying Bottlenecks (Conceptual):**

    *   Explain that identifying bottlenecks typically involves analyzing network traffic patterns, CPU utilization, and memory usage. Tools like Azure Network Watcher can be used to capture and analyze network traffic.

3.  **Implementing Optimizations:**

    *   **Increasing ExpressRoute Bandwidth:**
        *   Discuss that if the ExpressRoute circuit is consistently reaching its maximum capacity, you may need to increase the bandwidth.
    *   **Prioritizing Traffic with QoS:**
        *   Quality of Service (QoS) can be used to prioritize critical traffic (e.g., VoIP) over less important traffic (e.g., file transfers).
    *   **Implementing Caching:**
        *   Caching frequently accessed data closer to the users can reduce the load on the network.

### Post-Lab: Cleanup

To avoid incurring unnecessary costs, it's essential to clean up the resources created during this lab.

1.  **Delete Resource Group:**

    ```bash
    az group delete --name $RESOURCE_GROUP --yes
    ```

**Learning Outcomes:**

*   Successfully planned a hybrid network with Azure ExpressRoute.
*   Successfully deployed an Azure Virtual Network with subnets for a multi-tier application.
*   Learned how to use Azure Monitor to track VNet performance.
*   Understood the importance of applying updates during maintenance windows.
*   Explored techniques for optimizing ExpressRoute bandwidth, including increasing bandwidth, prioritizing traffic with QoS, and implementing caching.
