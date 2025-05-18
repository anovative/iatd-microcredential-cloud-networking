## IATD Microcredential Cloud Networking: Lab 7 - Implementing Network Security with Network Virtual Appliances (NVAs) and Azure Firewall in a Hub-and-Spoke Topology

**Objective:** This lab demonstrates how to implement network security using a Network Virtual Appliance (NVA) and Azure Firewall. You will set up a basic hub-and-spoke network topology and configure traffic routing through the firewall or NVA to inspect and control traffic between spokes.

**Estimated Time:** 120 - 150 minutes

**Prerequisites:**

1.  **Azure Subscription:** Requires an active Azure subscription. A free account is available at [https://azure.microsoft.com/free/](https://azure.microsoft.com/free/).
2.  **Basic knowledge of Azure Networking:** Familiarity with Virtual Networks, Subnets, Network Security Groups, and Virtual Machines.
3.  **Familiarity with Azure Firewall:** While not strictly required, some familiarity with Azure Firewall is helpful.
4.  **Basic understanding of Network Virtual Appliances (NVAs):** This lab assumes you understand the concept of NVAs. *Note: You will not be deploying an actual Palo Alto NVA due to licensing and complexity. This lab focuses on the routing and architecture aspects.*

## Let's get started!

### Lab Conventions

*   **Azure Cloud Shell:** All PowerShell and CLI interactions will occur within the Azure Cloud Shell environment.
*   **Azure Portal:** The Azure Portal will be used for visualization and configuration tasks.
*   **Naming Conventions:** Resource names follow the convention `iatd_labs_07_*`.
*   **IP Address Range:** The `172.16.0.0/16` range will be used (as per standardization).
*   **Location:** Choose a consistent Azure region.

#### Resource Naming

*   **Resource Group:** `iatd_labs_07_rg`
*   **Hub VNet:** `iatd_labs_07_hub_vnet`
*   **Hub Subnet - AzureFirewall:** `iatd_labs_07_hub_subnet_fw`
*   **Spoke1 VNet:** `iatd_labs_07_spoke1_vnet`
*   **Spoke1 Subnet-App:** `iatd_labs_07_spoke1_subnet_app`
*   **Spoke2 VNet:** `iatd_labs_07_spoke2_vnet`
*   **Spoke2 Subnet-DB:** `iatd_labs_07_spoke2_subnet_db`
*   **Azure Firewall:** `iatd_labs_07_azurefirewall`
*   **Public IP - AzureFirewall:** `iatd_labs_07_pip_fw`
*   **Route Table - Spoke1:** `iatd_labs_07_routetable_spoke1`
*   **Route Table - Spoke2:** `iatd_labs_07_routetable_spoke2`
*   **VM-App-01:** `iatd_labs_07_vm_app_01`
*   **VM-DB-01:** `iatd_labs_07_vm_db_01`
*   **NVA VM:** `iatd_labs_07_nva_vm` *Note: This represents the Palo Alto NVA conceptually.*

### Part 1: Setting up the Hub VNet and Azure Firewall

1.  **Create a Resource Group:**

    ```bash
    RESOURCE_GROUP="iatd_labs_07_rg"
    LOCATION="eastus" # Your Region

    az group create --name $RESOURCE_GROUP --location $LOCATION
    ```

    **Expected Output:**
    ```json
    {
      "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_07_rg",
      "location": "eastus",
      "managedBy": null,
      "name": "iatd_labs_07_rg",
      "properties": {
        "provisioningState": "Succeeded"
      },
      "tags": null,
      "type": "Microsoft.Resources/resourceGroups"
    }
    ```

2.  **Create the Hub VNet and AzureFirewallSubnet:**

    ```bash
    HUB_VNET_NAME="iatd_labs_07_hub_vnet"
    HUB_FW_SUBNET_NAME="AzureFirewallSubnet"  # REQUIRED name
    HUB_ADDRESS_PREFIX="10.0.0.0/16"
    HUB_FW_SUBNET_PREFIX="10.0.0.0/24"

    az network vnet create \
        --resource-group $RESOURCE_GROUP \
        --name $HUB_VNET_NAME \
        --address-prefixes $HUB_ADDRESS_PREFIX \
        --location $LOCATION

    az network vnet subnet create \
        --resource-group $RESOURCE_GROUP \
        --vnet-name $HUB_VNET_NAME \
        --name $HUB_FW_SUBNET_NAME \
        --address-prefix $HUB_FW_SUBNET_PREFIX
    ```

3.  **Create a Public IP Address for Azure Firewall:**

    ```bash
    FW_PUBLIC_IP_NAME="iatd_labs_07_pip_fw"

    az network public-ip create \
        --resource-group $RESOURCE_GROUP \
        --name $FW_PUBLIC_IP_NAME \
        --allocation-method Static \
        --sku Standard \
        --location $LOCATION
    ```

4.  **Deploy Azure Firewall (Portal):**
    *   Search for "Firewalls" and select.
    *   Click **Create**.
        *   Subscription: Your subscription
        *   Resource group: `iatd_labs_07_rg`
        *   Name: `iatd_labs_07_azurefirewall`
        *   Region: Your Region.
        *   Tier: Standard
        *   Virtual network: Select `iatd_labs_07_hub_vnet`
        *   Public IP address: Select `iatd_labs_07_pip_fw`
    * Click **Review + Create** and then **Create**

5.  **Get Azure Firewall Private IP Address:**

    *   After the deployment completes, navigate to the `iatd_labs_07_azurefirewall` in the Azure Portal.
    *   Note the Private IP address of the firewall. You'll need this for the UDRs.

### Part 2: Setting up the Spoke VNets and Subnets

1.  **Create Spoke1 VNet and Subnet:**

    ```bash
    SPOKE1_VNET_NAME="iatd_labs_07_spoke1_vnet"
    SPOKE1_SUBNET_APP_NAME="iatd_labs_07_spoke1_subnet_app"
    SPOKE1_ADDRESS_PREFIX="10.1.0.0/16"
    SPOKE1_SUBNET_APP_PREFIX="10.1.1.0/24"

    az network vnet create \
        --resource-group $RESOURCE_GROUP \
        --name $SPOKE1_VNET_NAME \
        --address-prefixes $SPOKE1_ADDRESS_PREFIX \
        --location $LOCATION

    az network vnet subnet create \
        --resource-group $RESOURCE_GROUP \
        --vnet-name $SPOKE1_VNET_NAME \
        --name $SPOKE1_SUBNET_APP_NAME \
        --address-prefix $SPOKE1_SUBNET_APP_PREFIX
    ```

2.  **Create Spoke2 VNet and Subnet:**

    ```bash
    SPOKE2_VNET_NAME="iatd_labs_07_spoke2_vnet"
    SPOKE2_SUBNET_DB_NAME="iatd_labs_07_spoke2_subnet_db"
    SPOKE2_ADDRESS_PREFIX="10.2.0.0/16"
    SPOKE2_SUBNET_DB_PREFIX="10.2.1.0/24"

    az network vnet create \
        --resource-group $RESOURCE_GROUP \
        --name $SPOKE2_VNET_NAME \
        --address-prefixes $SPOKE2_ADDRESS_PREFIX \
        --location $LOCATION

    az network vnet subnet create \
        --resource-group $RESOURCE_GROUP \
        --vnet-name $SPOKE2_VNET_NAME \
        --name $SPOKE2_SUBNET_DB_NAME \
        --address-prefix $SPOKE2_SUBNET_DB_PREFIX
    ```

### Part 3: Peering the VNets

1.  **Create VNet Peering from Hub to Spoke1:**

    ```bash
    az network vnet peering create \
        --resource-group $RESOURCE_GROUP \
        --name HubToSpoke1 \
        --vnet-name $HUB_VNET_NAME \
        --remote-vnet $SPOKE1_VNET_NAME \
        --allow-vnet-access
        --allow-forwarded-traffic
        --allow-gateway-transit
        --use-remote-gateways
    ```

2.  **Create VNet Peering from Spoke1 to Hub:**

    ```bash
    az network vnet peering create \
        --resource-group $RESOURCE_GROUP \
        --name Spoke1ToHub \
        --vnet-name $SPOKE1_VNET_NAME \
        --remote-vnet $HUB_VNET_NAME \
        --allow-vnet-access
        --allow-forwarded-traffic
        --allow-gateway-transit
        --use-remote-gateways
    ```

3.  **Create VNet Peering from Hub to Spoke2:**

    ```bash
        az network vnet peering create \
        --resource-group $RESOURCE_GROUP \
        --name HubToSpoke2 \
        --vnet-name $HUB_VNET_NAME \
        --remote-vnet $SPOKE2_VNET_NAME \
        --allow-vnet-access
        --allow-forwarded-traffic
        --allow-gateway-transit
        --use-remote-gateways
    ```

4.  **Create VNet Peering from Spoke2 to Hub:**

    ```bash
        az network vnet peering create \
        --resource-group $RESOURCE_GROUP \
        --name Spoke2ToHub \
        --vnet-name $SPOKE2_VNET_NAME \
        --remote-vnet $HUB_VNET_NAME \
        --allow-vnet-access
        --allow-forwarded-traffic
        --allow-gateway-transit
        --use-remote-gateways
    ```

### Part 4: Configuring User-Defined Routes (UDRs)

1.  **Create Route Table for Spoke1:**

    ```bash
    ROUTE_TABLE_SPOKE1_NAME="iatd_labs_07_routetable_spoke1"

    az network route-table create \
        --resource-group $RESOURCE_GROUP \
        --name $ROUTE_TABLE_SPOKE1_NAME \
        --location $LOCATION
    ```

2.  **Add a Route to Spoke1's Route Table (Direct all traffic to Azure Firewall):**

    ```bash
    ROUTE_TABLE_SPOKE1_NAME="iatd_labs_07_routetable_spoke1"
    FIREWALL_PRIVATE_IP="<Replace with Azure Firewall's Private IP address>"

    az network route-table route create \
        --resource-group $RESOURCE_GROUP \
        --name RouteToAzureFirewall \
        --route-table-name $ROUTE_TABLE_SPOKE1_NAME \
        --address-prefix 10.2.0.0/16 \
        --next-hop-type VirtualAppliance \
        --next-hop-ip-address $FIREWALL_PRIVATE_IP

     az network route-table route create \
        --resource-group $RESOURCE_GROUP \
        --name RouteToInternet \
        --route-table-name $ROUTE_TABLE_SPOKE1_NAME \
        --address-prefix 0.0.0.0/0 \
        --next-hop-type VirtualAppliance \
        --next-hop-ip-address $FIREWALL_PRIVATE_IP
    ```

3.  **Associate Spoke1's Route Table with the App Subnet:**

    ```bash
    az network vnet subnet update \
        --resource-group $RESOURCE_GROUP \
        --vnet-name $SPOKE1_VNET_NAME \
        --name $SPOKE1_SUBNET_APP_NAME \
        --route-table $ROUTE_TABLE_SPOKE1_NAME
    ```

4.  **Create Route Table for Spoke2:**

    ```bash
    ROUTE_TABLE_SPOKE2_NAME="iatd_labs_07_routetable_spoke2"

    az network route-table create \
        --resource-group $RESOURCE_GROUP \
        --name $ROUTE_TABLE_SPOKE2_NAME \
        --location $LOCATION
    ```

5.  **Add a Route to Spoke2's Route Table (Direct all traffic to Azure Firewall):**

    ```bash
    ROUTE_TABLE_SPOKE2_NAME="iatd_labs_07_routetable_spoke2"
    FIREWALL_PRIVATE_IP="<Replace with Azure Firewall's Private IP address>" # e.g. 172.16.0.4

    az network route-table route create \
        --resource-group $RESOURCE_GROUP \
        --name RouteToAzureFirewall \
        --route-table-name $ROUTE_TABLE_SPOKE2_NAME \
        --address-prefix 10.1.0.0/16 \
        --next-hop-type VirtualAppliance \
        --next-hop-ip-address $FIREWALL_PRIVATE_IP

    az network route-table route create \
        --resource-group $RESOURCE_GROUP \
        --name RouteToInternet \
        --route-table-name $ROUTE_TABLE_SPOKE2_NAME \
        --address-prefix 0.0.0.0/0 \
        --next-hop-type VirtualAppliance \
        --next-hop-ip-address $FIREWALL_PRIVATE_IP
    ```

6.  **Associate Spoke2's Route Table with the DB Subnet:**

    ```bash
    az network vnet subnet update \
        --resource-group $RESOURCE_GROUP \
        --vnet-name $SPOKE2_VNET_NAME \
        --name $SPOKE2_SUBNET_DB_NAME \
        --route-table $ROUTE_TABLE_SPOKE2_NAME
    ```

### Part 5: Configuring Azure Firewall Rule

1.  **Create a Firewall Rule (Portal):**
    *   Navigate to the `iatd_labs_07_azurefirewall` in the Azure Portal.
    *   Select **Rules** under **Settings**.
    *   Click **Add rule collection**.
        *   Name: AllowAppToDB
        *   Rule collection type: Network
        *   Priority: 100
        *   Action: Allow
        *   Add Rule:
            *   Name: AllowSQL
            *   Protocol: TCP
            *   Source type: Address
            *   Source: 10.1.1.0/24 (App Subnet)
            *   Destination type: Address
            *   Destination: 10.2.1.0/24 (DB Subnet)
            *   Destination ports: 1433
    * Click **Add** and then **Create**.

### Part 6: Creating Virtual Machines in Subnets

1.  **Create App Virtual Machine (`iatd_labs_07_vm_app_01`) (Portal):**
    *   Search for "Virtual machines" and select.
    *   Click **Create** -> **Azure virtual machine**.
        *   **Basics Tab:**
            *   Subscription: Your subscription.
            *   Resource group: `iatd_labs_07_rg`
            *   Virtual machine name: `iatd_labs_07_vm_app_01`
            *   Region: Your region.
            *   Image: `Ubuntu Server 22.04 LTS`
            *   Size: Choose a size (e.g., Standard_B1ls).
            *   Username: `azureuser`
            *   Authentication type: `Password`
            *   Password: Set a password.
        *   **Networking Tab:**
            *   Virtual network: `iatd_labs_07_spoke1_vnet`
            *   Subnet: `iatd_labs_07_spoke1_subnet_app`
            *   Public IP: **None** (Important: No Public IP)
            *   NIC network security group: `None`
        *   **Management Tab:**
            *   Disable boot diagnostics
    *   Click **Review + create**, then **Create**.

2.  **Create DB Virtual Machine (`iatd_labs_07_vm_db_01`) (Portal):**
    *   Repeat the process above, but with the following changes:
        *   Virtual machine name: `iatd_labs_07_vm_db_01`
        *   Subnet: `iatd_labs_07_spoke2_subnet_db`
        *   Public IP: **None** (Important: No Public IP)
        *   NIC network security group: `None`
    *   **Management Tab:**
        *   Disable boot diagnostics

### Part 7: Simulating an NVA (Conceptual)

1.  **Conceptual NVA Deployment:**
    *   For the purpose of this lab, we'll simulate the presence of a Palo Alto NVA conceptually. In a real-world scenario, you would deploy a Palo Alto NVA from the Azure Marketplace and configure it according to the vendor's instructions.
    *   *Since we are not deploying an actual NVA, there are no configuration steps here.* However, it's important to understand that in a real-world deployment, the NVA would be configured to inspect traffic between the app and DB subnets and block any malicious activity (e.g., SQL injection attempts).

### Part 8: Testing Network Traffic Flow

1.  **Connect to App VM:**
    *   You'll need a "jump box" VM with a public IP in the same VNet or use Azure Bastion to connect to the App VM.
    *   SSH into the jump box VM.
    *   From the jump box, SSH into the `iatd_labs_07_vm_app_01` VM using its private IP address.

2.  **(Optional) Install telnet and test Connectivity from App VM to DB VM on Port 1433:**

    *   From the `iatd_labs_07_vm_app_01` VM, try to telnet to the private IP address of the `iatd_labs_07_vm_db_01` VM on port 1433.

        ```bash
        sudo apt update
        sudo apt install telnet -y
        telnet <private_ip_of_db_vm> 1433
        ```

    *   If the connection is successful, it will show a blank screen or a message related to SQL Server. This confirms that the App VM can connect to the DB VM on port 1433 *through the Azure Firewall*. If telnet is not installed, it can be replaced with nmap: `nmap -p 1433 <private_ip_of_db_vm>`.

3.  **Test Connectivity from App VM to DB VM on other ports(Portal):**
    *   Remove The rule created in step 1 of this part, and observe that connection to the DB VM via port 1433 is no longer working.
4.  **Verify UDR (Portal):**
    *   To confirm that the UDR is working, try to ping a public IP address (e.g., 8.8.8.8) from the `iatd_labs_07_vm_app_01` VM. Since we have not created an Azure Firewall rule to allow internet access, the attempt will fail.

### Post-Lab: Cleanup

To avoid incurring unnecessary costs, it's essential to clean up the resources created during this lab.

1.  **Delete Resource Group:**

    ```bash
    az group delete --name $RESOURCE_GROUP --yes
    ```

    **Expected Output:**
    ```
    The resource group deletion is in progress and will complete in the background.
    ```

2.  **Verify Resource Deletion:**

    ```bash
    az group show --name $RESOURCE_GROUP
    ```

    **Expected Output:**
    ```
    Resource group 'iatd_labs_07_rg' could not be found.
    ```

3.  **Clean Local Environment Variables:**

    ```bash
    unset RESOURCE_GROUP LOCATION HUB_VNET_NAME HUB_SUBNET_FW_NAME HUB_SUBNET_NVA_NAME HUB_ADDRESS_PREFIX HUB_SUBNET_FW_PREFIX HUB_SUBNET_NVA_PREFIX SPOKE1_VNET_NAME SPOKE1_SUBNET_APP_NAME SPOKE1_ADDRESS_PREFIX SPOKE1_SUBNET_APP_PREFIX SPOKE2_VNET_NAME SPOKE2_SUBNET_DB_NAME SPOKE2_ADDRESS_PREFIX SPOKE2_SUBNET_DB_PREFIX ROUTE_TABLE_SPOKE1_NAME ROUTE_TABLE_SPOKE2_NAME FIREWALL_PRIVATE_IP
    ```

**Learning Outcomes:**

*   Successfully created a hub-and-spoke network topology with VNet peering.
*   Successfully deployed and configured Azure Firewall in the hub VNet.
*   Successfully created User-Defined Routes (UDRs) to route traffic through the Azure Firewall.
*   Understands the concept of NVA to inspect traffic between spokes
*   Successfully configured Azure Firewall rules to allow specific traffic between spokes.
*   Successfully verified that traffic is routed through the Azure Firewall.
*   Understand how UDRs and Azure Firewall can be used to control and secure network traffic in a hub-and-spoke topology.
*   Learned how NVAs enhance security by inspecting traffic for malicious activity.
