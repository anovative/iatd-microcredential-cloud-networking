## IATD Microcredential Cloud Networking: Lab 8 - Building a Fully Isolated Three-Tier Application in Azure

**Objective:** This lab demonstrates how to build a fully isolated three-tier application in Azure, encompassing web, application, and database layers. You'll configure VNets, subnets, NSGs, Private Endpoints, UDRs, Azure Firewall, and VNet peering to achieve a high level of security and isolation.

**Estimated Time:** 150 - 180 minutes

**Prerequisites:**

1.  **Azure Subscription:** Requires an active Azure subscription. A free account is available at [https://azure.microsoft.com/free/](https://azure.microsoft.com/free/).
2.  **Strong knowledge of Azure Networking:** Familiarity with Virtual Networks, Subnets, Network Security Groups, Route Tables, VNet Peering, and Azure Firewall.
3.  **Existing Azure SQL Database:** You will need an existing Azure SQL Database. Note the server name and database name.
4.  **Existing Azure Storage Account:** You will need an existing Azure Storage Account.

## Let's get started!

### Lab Conventions

*   **Azure Cloud Shell:** All PowerShell and CLI interactions will occur within the Azure Cloud Shell environment.
*   **Azure Portal:** The Azure Portal will be used for visualization and configuration tasks.
*   **Naming Conventions:** Resource names follow the convention `iatd_labs_08_*`.
*   **IP Address Range:** The `172.16.0.0/16` range will be used (as per standardization).
*   **Location:** Choose a consistent Azure region.

#### Resource Naming

*   **Resource Group:** `iatd_labs_08_rg`
*   **Spoke VNet (myVNet):** `iatd_labs_08_spoke_vnet`
*   **Subnet-Web:** `iatd_labs_08_subnet_web`
*   **Subnet-App:** `iatd_labs_08_subnet_app`
*   **Subnet-DB:** `iatd_labs_08_subnet_db`
*   **Subnet-AzureFirewall:** `iatd_labs_08_subnet_fw`
*   **NSG-Web:** `iatd_labs_08_nsg_web`
*   **NSG-App:** `iatd_labs_08_nsg_app`
*   **NSG-DB:** `iatd_labs_08_nsg_db`
*   **Azure Firewall:** `iatd_labs_08_azurefirewall`
*   **Public IP - AzureFirewall:** `iatd_labs_08_pip_fw`
*   **Route Table - AppSubnet:** `iatd_labs_08_routetable_app`
*   **Hub VNet:** (Assuming you have a Hub VNet) `iatd_labs_08_hub_vnet`
*   **Private Endpoint:** `iatd_labs_08_sqldb_pe`
*   **VM-Web-01:** `iatd_labs_08_vm_web_01`
*   **VM-App-01:** `iatd_labs_08_vm_app_01`

*   **Azure SQL Server:** (Use the name of your existing SQL Server)
*   **Azure SQL Database:** (Use the name of your existing SQL Database)
*   **Storage Account:** (Use the name of your existing storage account)

### Part 1: Setting up the Spoke VNet (myVNet) and Subnets

1.  **Create a Resource Group:**

    ```bash
    RESOURCE_GROUP="iatd_labs_08_rg"
    LOCATION="eastus" # Your Region

    az group create --name $RESOURCE_GROUP --location $LOCATION
    ```

2.  **Create the Spoke VNet (myVNet) and Subnets:**

    ```bash
    SPOKE_VNET_NAME="iatd_labs_08_spoke_vnet"
    SUBNET_WEB_NAME="iatd_labs_08_subnet_web"
    SUBNET_APP_NAME="iatd_labs_08_subnet_app"
    SUBNET_DB_NAME="iatd_labs_08_subnet_db"
    SUBNET_FW_NAME="AzureFirewallSubnet"  # REQUIRED name
    ADDRESS_PREFIX="10.0.0.0/16"
    SUBNET_WEB_PREFIX="10.0.1.0/24"
    SUBNET_APP_PREFIX="10.0.2.0/24"
    SUBNET_DB_PREFIX="10.0.3.0/24"
    SUBNET_FW_PREFIX="10.0.0.0/24"

    az network vnet create \
        --resource-group $RESOURCE_GROUP \
        --name $SPOKE_VNET_NAME \
        --address-prefixes $ADDRESS_PREFIX \
        --location $LOCATION \
        --subnet-name $SUBNET_WEB_NAME \
        --subnet-prefixes $SUBNET_WEB_PREFIX

    az network vnet subnet create \
        --resource-group $RESOURCE_GROUP \
        --vnet-name $SPOKE_VNET_NAME \
        --name $SUBNET_APP_NAME \
        --address-prefix $SUBNET_APP_PREFIX

    az network vnet subnet create \
        --resource-group $RESOURCE_GROUP \
        --vnet-name $SPOKE_VNET_NAME \
        --name $SUBNET_DB_NAME \
        --address-prefix $SUBNET_DB_PREFIX \
        --disable-private-endpoint-network-policies true

    az network vnet subnet create \
        --resource-group $RESOURCE_GROUP \
        --vnet-name $SPOKE_VNET_NAME \
        --name $SUBNET_FW_NAME \
        --address-prefix $SUBNET_FW_PREFIX
    ```

### Part 2: Setting up Network Security Groups (NSGs)

1.  **Create NSGs:**

    ```bash
    NSG_WEB_NAME="iatd_labs_08_nsg_web"
    NSG_APP_NAME="iatd_labs_08_nsg_app"
    NSG_DB_NAME="iatd_labs_08_nsg_db"

    az network nsg create \
        --resource-group $RESOURCE_GROUP \
        --name $NSG_WEB_NAME \
        --location $LOCATION

    az network nsg create \
        --resource-group $RESOURCE_GROUP \
        --name $NSG_APP_NAME \
        --location $LOCATION

    az network nsg create \
        --resource-group $RESOURCE_GROUP \
        --name $NSG_DB_NAME \
        --location $LOCATION
    ```

2.  **Create NSG Rules (Web NSG):**

    ```bash
    NSG_WEB_NAME="iatd_labs_08_nsg_web"

    az network nsg rule create \
        --resource-group $RESOURCE_GROUP \
        --nsg-name $NSG_WEB_NAME \
        --name AllowHTTP \
        --priority 100 \
        --source-address-prefixes Internet \
        --destination-port-ranges 80,443 \
        --access Allow \
        --protocol Tcp \
        --direction Inbound

    az network nsg rule create \
        --resource-group $RESOURCE_GROUP \
        --nsg-name $NSG_WEB_NAME \
        --name AllowOutboundToApp \
        --priority 100 \
        --source-address-prefixes 10.0.1.0/24 \
        --destination-address-prefixes 10.0.2.0/24 \
        --destination-port-ranges '*' \
        --access Allow \
        --protocol '*' \
        --direction Outbound
    ```

3.  **Create NSG Rules (App NSG):**

    ```bash
    NSG_APP_NAME="iatd_labs_08_nsg_app"

    az network nsg rule create \
        --resource-group $RESOURCE_GROUP \
        --nsg-name $NSG_APP_NAME \
        --name AllowInboundFromWeb \
        --priority 100 \
        --source-address-prefixes 10.0.1.0/24 \
        --destination-port-ranges 8080 \
        --access Allow \
        --protocol Tcp \
        --direction Inbound

    az network nsg rule create \
        --resource-group $RESOURCE_GROUP \
        --nsg-name $NSG_APP_NAME \
        --name AllowOutboundToDB \
        --priority 100 \
        --source-address-prefixes 10.0.2.0/24 \
        --destination-address-prefixes 10.0.3.0/24 \
        --destination-port-ranges '*' \
        --access Allow \
        --protocol '*' \
        --direction Outbound

        az network nsg rule create \
        --resource-group $RESOURCE_GROUP \
        --nsg-name $NSG_APP_NAME \
        --name AllowOutboundToStorage \
        --priority 110 \
        --source-address-prefixes 10.0.2.0/24 \
        --destination-service-tags Storage \
        --destination-port-ranges 443 \
        --access Allow \
        --protocol Tcp \
        --direction Outbound
    ```

4.  **Create NSG Rules (DB NSG):**

    ```bash
    NSG_DB_NAME="iatd_labs_08_nsg_db"

    az network nsg rule create \
        --resource-group $RESOURCE_GROUP \
        --nsg-name $NSG_DB_NAME \
        --name AllowInboundFromApp \
        --priority 100 \
        --source-address-prefixes 10.0.2.0/24 \
        --destination-port-ranges 1433 \
        --access Allow \
        --protocol Tcp \
        --direction Inbound

    az network nsg rule create \
        --resource-group $RESOURCE_GROUP \
        --nsg-name $NSG_DB_NAME \
        --name DenyAllOtherInbound \
        --priority 4096 \
        --source-address-prefixes '*' \
        --destination-port-ranges '*' \
        --access Deny \
        --protocol '*' \
        --direction Inbound
    ```

5.  **Associate NSGs with Subnets:**

    ```bash
    az network vnet subnet update \
        --resource-group $RESOURCE_GROUP \
        --vnet-name $SPOKE_VNET_NAME \
        --name $SUBNET_WEB_NAME \
        --network-security-group $NSG_WEB_NAME

    az network vnet subnet update \
        --resource-group $RESOURCE_GROUP \
        --vnet-name $SPOKE_VNET_NAME \
        --name $SUBNET_APP_NAME \
        --network-security-group $NSG_APP_NAME

    az network vnet subnet update \
        --resource-group $RESOURCE_GROUP \
        --vnet-name $SPOKE_VNET_NAME \
        --name $SUBNET_DB_NAME \
        --network-security-group $NSG_DB_NAME
    ```

### Part 3: Deploying and Configuring Azure Firewall

1.  **Create a Public IP Address for Azure Firewall:**

    ```bash
    FW_PUBLIC_IP_NAME="iatd_labs_08_pip_fw"

    az network public-ip create \
        --resource-group $RESOURCE_GROUP \
        --name $FW_PUBLIC_IP_NAME \
        --allocation-method Static \
        --sku Standard \
        --location $LOCATION
    ```

2.  **Deploy Azure Firewall (Portal):**
    *   Search for "Firewalls" and select.
    *   Click **Create**.
        *   Subscription: Your subscription
        *   Resource group: `iatd_labs_08_rg`
        *   Name: `iatd_labs_08_azurefirewall`
        *   Region: Your Region.
        *   Tier: Standard
        *   Virtual network: Select `iatd_labs_08_spoke_vnet`
        *   Public IP address: Select `iatd_labs_08_pip_fw`
    * Click **Review + Create** and then **Create**

3.  **Get Azure Firewall Private IP Address:**

    *   After the deployment completes, navigate to the `iatd_labs_08_azurefirewall` in the Azure Portal.
    *   Note the Private IP address of the firewall. You'll need this for the UDRs.

### Part 4: Creating Private Endpoint for Azure SQL Database

1.  **Get Azure SQL Server Resource ID:**

    ```bash
    SQL_SERVER_NAME="<Your Azure SQL Server Name>"
    az sql server show --name $SQL_SERVER_NAME --resource-group $RESOURCE_GROUP --query id -o tsv
    ```

2.  **Create the Private Endpoint (Portal):**
    *   In the Azure portal, search for "Private endpoints" and select it.
    *   Click **Create**.
        *   **Basics Tab:**
            *   Subscription: Your subscription.
            *   Resource group: `iatd_labs_08_rg`
            *   Name: `iatd_labs_08_sqldb_pe`
            *   Region: Your region.
        *   **Resource Tab:**
            *   Connection method: "My directory"
            *   Subscription: Your subscription.
            *   Resource type: Microsoft.Sql/servers
            *   Resource: Select your SQL Server (or paste the Resource ID obtained above)
            *   Target subresource: SqlServer
        *   **Virtual Network Tab:**
            *   Virtual network: `iatd_labs_08_spoke_vnet`
            *   Subnet: `iatd_labs_08_subnet_db`
        *   **DNS Tab:**
            *   Integrate with private DNS zone: Yes (or No, and configure DNS manually)

    *   Click **Review + create**, then **Create**.

### Part 5: Configuring User-Defined Route (UDR)

1.  **Create a Route Table for App Subnet:**

    ```bash
    ROUTE_TABLE_APP_NAME="iatd_labs_08_routetable_app"

    az network route-table create \
        --resource-group $RESOURCE_GROUP \
        --name $ROUTE_TABLE_APP_NAME \
        --location $LOCATION
    ```

2.  **Add a Route to the Route Table (Direct all traffic to Azure Firewall):**

    ```bash
    ROUTE_TABLE_APP_NAME="iatd_labs_08_routetable_app"
    FIREWALL_PRIVATE_IP="<Replace with Azure Firewall's Private IP address>"

    az network route-table route create \
        --resource-group $RESOURCE_GROUP \
        --name RouteToDBSubnetViaFirewall \
        --route-table-name $ROUTE_TABLE_APP_NAME \
        --address-prefix 10.0.3.0/24 \
        --next-hop-type VirtualAppliance \
        --next-hop-ip-address $FIREWALL_PRIVATE_IP

    az network route-table route create \
        --resource-group $RESOURCE_GROUP \
        --name RouteToInternetViaFirewall \
        --route-table-name $ROUTE_TABLE_APP_NAME \
        --address-prefix 0.0.0.0/0 \
        --next-hop-type VirtualAppliance \
        --next-hop-ip-address $FIREWALL_PRIVATE_IP
    ```

3.  **Associate the Route Table with the App Subnet:**

    ```bash
    az network vnet subnet update \
        --resource-group $RESOURCE_GROUP \
        --vnet-name $SPOKE_VNET_NAME \
        --name $SUBNET_APP_NAME \
        --route-table $ROUTE_TABLE_APP_NAME
    ```

### Part 6: Configuring Azure Firewall Rules

1.  **Create a Firewall Rule (Portal):**
    *   Navigate to the `iatd_labs_08_azurefirewall` in the Azure Portal.
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
            *   Source: 10.0.2.0/24 (App Subnet)
            *   Destination type: Address
            *   Destination: 10.0.3.0/24 (DB Subnet)
            *   Destination ports: 1433
    * Click **Add** and then **Create**.
2.  **Enable Diagnostic Settings for Azure Firewall:**

    *   This sends Azure Firewall logs to a Log Analytics Workspace for analysis.

### Part 7: Peering with Hub VNet

1.  **Note**: This assumes you have already created a Hub VNet
2.  **Create VNet Peering from Spoke to Hub:**

    ```bash
    HUB_VNET_NAME="<Your Hub VNet Name>" #e.g iatd_labs_08_hub_vnet
    az network vnet peering create \
        --resource-group $RESOURCE_GROUP \
        --name SpokeToHub \
        --vnet-name $SPOKE_VNET_NAME \
        --remote-vnet $HUB_VNET_NAME \
        --allow-vnet-access
        --allow-forwarded-traffic
        --use-remote-gateways
    ```

3.  **Create VNet Peering from Hub to Spoke:**

    ```bash
     HUB_VNET_NAME="<Your Hub VNet Name>" #e.g iatd_labs_08_hub_vnet

    az network vnet peering create \
        --resource-group $RESOURCE_GROUP \
        --name HubToSpoke \
        --vnet-name  $HUB_VNET_NAME \
        --remote-vnet $SPOKE_VNET_NAME \
        --allow-vnet-access
        --allow-forwarded-traffic
        --use-remote-gateways
    ```

### Part 8: Creating Virtual Machines

1.  **Create Web Virtual Machine (`iatd_labs_08_vm_web_01`) (Portal):**
    *   Search for "Virtual machines" and select.
    *   Click **Create** -> **Azure virtual machine**.
        *   **Basics Tab:**
            *   Subscription: Your subscription.
            *   Resource group: `iatd_labs_08_rg`
            *   Virtual machine name: `iatd_labs_08_vm_web_01`
            *   Region: Your region.
            *   Image: `Ubuntu Server 22.04 LTS`
            *   Size: Choose a size (e.g., Standard_B1ls).
            *   Username: `azureuser`
            *   Authentication type: `Password`
            *   Password: Set a password.
        *   **Networking Tab:**
            *   Virtual network: `iatd_labs_08_spoke_vnet`
            *   Subnet: `iatd_labs_08_subnet_web`
            *   Public IP: Create a new public IP address
            *   NIC network security group: `None`
        *   **Management Tab:**
            *   Disable boot diagnostics
    *   Click **Review + create**, then **Create**.

2.  **Create App Virtual Machine (`iatd_labs_08_vm_app_01`) (Portal):**
    *   Repeat the process above, but with the following changes:
        *   Virtual machine name: `iatd_labs_08_vm_app_01`
        *   Subnet: `iatd_labs_08_subnet_app`
        *   Public IP: **None** (Important: No Public IP)
        *   NIC network security group: `None`
    *   **Management Tab:**
        *   Disable boot diagnostics

### Part 9: Testing Connectivity and Isolation

1.  **Connect to Web VM:**
    *   Connect to the `iatd_labs_08_vm_web_01` VM using SSH through its public IP address.

2.  **Test Connectivity from Web VM to App VM on port 8080:**
     *  Since we don't have anything actively listening on the App VM, we can use `telnet` to confirm the connection is possible, but not necessarily successful. If you wanted to validate full traffic, you'd need to deploy an application.

        ```bash
        telnet <private_ip_of_app_vm> 8080
        ```

3.  **Test Connectivity from App VM to DB VM on Port 1433:**
        *   You need a "jump box" VM with a public IP in the same VNet or use Azure Bastion to connect to the App VM.
        *   SSH into the jump box VM.
        *   From the jump box, SSH into the `iatd_labs_08_vm_app_01` VM using its private IP address.
     * From the `iatd_labs_08_vm_app_01` VM, use `sqlcmd` to connect to the Azure SQL Database using its fully qualified domain name (FQDN) and the appropriate credentials. You will need to find the FQDN under the SQL server properties
    *   If the connection is successful, it confirms that the App VM can connect to the SQL Database *privately* through the Private Endpoint.

4.  **Test Connectivity to Other Internet Destinations from App VM:**

    *   From the `iatd_labs_08_vm_app_01` VM, try to ping a public IP address (e.g., 8.8.8.8).

    *   This *should* fail because all traffic is routed to Azure Firewall, and we have not enabled any rule to allow general internet access.

5.  **Review Azure Firewall Logs:**
    *   Examine the Azure Firewall logs in your Log Analytics Workspace to verify that traffic between the App and DB subnets is being logged.

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
    Resource group 'iatd_labs_08_rg' could not be found.
    ```

3.  **Clean Local Environment Variables:**

    ```bash
    unset RESOURCE_GROUP LOCATION HUB_VNET_NAME HUB_SUBNET_FW_NAME HUB_SUBNET_PE_NAME HUB_ADDRESS_PREFIX HUB_SUBNET_FW_PREFIX HUB_SUBNET_PE_PREFIX APP_VNET_NAME APP_SUBNET_NAME APP_ADDRESS_PREFIX APP_SUBNET_PREFIX WEB_VNET_NAME WEB_SUBNET_NAME WEB_ADDRESS_PREFIX WEB_SUBNET_PREFIX ROUTE_TABLE_APP_NAME ROUTE_TABLE_WEB_NAME FIREWALL_PRIVATE_IP NSG_WEB_NAME NSG_APP_NAME
    ```

**Learning Outcomes:**

*   Successfully created a fully isolated three-tier application in Azure.
*   Successfully configured VNets, subnets, NSGs, Private Endpoints, UDRs, Azure Firewall, and VNet peering.
*   Understood the principle of least privilege.
*   Successfully deployed and configured Azure Firewall to inspect and log traffic between tiers.
*   Successfully verified that the web tier is publicly accessible but isolated from direct app/DB access.
*   Successfully verified that the app tier is private, communicating only with the web and DB tiers through Azure Firewall.
*   Successfully verified that the DB tier is fully isolated, accessible only via Private Endpoint.
*   Successfully set up a hub-and-spoke topology for centralized security services.
*   Demonstrated a comprehensive understanding of Azure network security best practices.
