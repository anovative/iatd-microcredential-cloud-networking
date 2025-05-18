## IATD Microcredential Cloud Networking: Lab 9 - Hybrid Connectivity and Secure Access to Azure Services

**Objective:** This lab simulates a hybrid cloud environment where an on-premises network connects to Azure via ExpressRoute. You'll design and implement a multi-tier application with private access to Azure SQL and Storage, leveraging Private Endpoints, Service Endpoints, NSGs, and VNet peering with a Hub VNet and Azure Firewall.

**Estimated Time:** 150 - 180 minutes

**Prerequisites:**

1.  **Azure Subscription:** Requires an active Azure subscription. A free account is available at [https://azure.microsoft.com/free/](https://azure.microsoft.com/free/).
2.  **Strong knowledge of Azure Networking:** Familiarity with Virtual Networks, Subnets, Network Security Groups, Route Tables, VNet Peering, Azure Firewall, Private Endpoints, and Service Endpoints.
3.  **Existing Azure SQL Database:** You will need an existing Azure SQL Database. Note the server name and database name.
4.  **Existing Azure Storage Account:** You will need an existing Azure Storage Account.
5.  **Simulated On-Premises Network (Conceptual):** This lab will *simulate* an on-premises network and ExpressRoute connection. *You will not be setting up actual ExpressRoute connectivity.*

## Let's get started!

### Lab Conventions

*   **Azure Cloud Shell:** All PowerShell and CLI interactions will occur within the Azure Cloud Shell environment.
*   **Azure Portal:** The Azure Portal will be used for visualization and configuration tasks.
*   **Naming Conventions:** Resource names follow the convention `iatd_labs_09_*`.
*   **IP Address Range:** The `172.16.0.0/16` range will be used (as per standardization).
*   **Location:** Choose a consistent Azure region.

#### Resource Naming

*   **Resource Group:** `iatd_labs_09_rg`
*   **Spoke VNet (myVNet):** `iatd_labs_09_spoke_vnet`
*   **Subnet-App:** `iatd_labs_09_subnet_app`
*   **Subnet-DB:** `iatd_labs_09_subnet_db`
*   **Subnet - AzureBastion:** `AzureBastionSubnet`
*   **Hub VNet:** `iatd_labs_09_hub_vnet`
*   **Subnet - Firewall:** `AzureFirewallSubnet`
*   **Subnet - Gateway:** `GatewaySubnet`
*   **Subnet - OnPrem:** `iatd_labs_09_subnet_onprem`
*   **Azure Firewall:** `iatd_labs_09_azurefirewall`
*   **Public IP - AzureFirewall:** `iatd_labs_09_pip_fw`
*   **Route Table - AppSubnet:** `iatd_labs_09_routetable_app`
*   **Route Table - OnPrem:** `iatd_labs_09_routetable_onprem`
*   **NSG - App:** `iatd_labs_09_nsg_app`
*   **NSG - DB:** `iatd_labs_09_nsg_db`
*   **Private Endpoint:** `iatd_labs_09_sqldb_pe`
*   **VM-App-01:** `iatd_labs_09_vm_app_01`
*   **VM-OnPrem-01:** `iatd_labs_09_vm_onprem_01`
*   **Azure SQL Server:** (Use the name of your existing SQL Server)
*   **Azure SQL Database:** (Use the name of your existing SQL Database)
*   **Storage Account:** (Use the name of your existing storage account)

### Part 1: Setting Up the Environment

1.  **Create a Resource Group:**

    ```bash
    RESOURCE_GROUP="iatd_labs_09_rg"
    LOCATION="eastus" # Your Region

    az group create --name $RESOURCE_GROUP --location $LOCATION
    ```

    **Expected Output:**
    ```json
    {
      "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_09_rg",
      "location": "eastus",
      "managedBy": null,
      "name": "iatd_labs_09_rg",
      "properties": {
        "provisioningState": "Succeeded"
      },
      "tags": null,
      "type": "Microsoft.Resources/resourceGroups"
    }
    ```

2.  **Create Hub VNet and Subnets:**

    ```bash
    HUB_VNET_NAME="iatd_labs_09_hub_vnet"
    HUB_SUBNET_FW_NAME="AzureFirewallSubnet" # This name is required by Azure
    HUB_SUBNET_GW_NAME="GatewaySubnet" # This name is required by Azure
    HUB_ADDRESS_PREFIX="172.16.0.0/16"
    HUB_SUBNET_FW_PREFIX="172.16.0.0/24"
    HUB_SUBNET_GW_PREFIX="172.16.1.0/24"

    az network vnet create \
        --resource-group $RESOURCE_GROUP \
        --name $HUB_VNET_NAME \
        --address-prefixes $HUB_ADDRESS_PREFIX \
        --subnet-name $HUB_SUBNET_FW_NAME \
        --subnet-prefixes $HUB_SUBNET_FW_PREFIX \
        --location $LOCATION
    ```

    **Expected Output for Hub VNet creation:**
    ```json
    {
      "newVNet": {
        "addressSpace": {
          "addressPrefixes": [
            "172.16.0.0/16"
          ]
        },
        "dhcpOptions": {
          "dnsServers": []
        },
        "enableDdosProtection": false,
        "enableVmProtection": false,
        "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_09_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_09_hub_vnet",
        "location": "eastus",
        "name": "iatd_labs_09_hub_vnet",
        "provisioningState": "Succeeded",
        "resourceGuid": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "subnets": [
          {
            "addressPrefix": "172.16.0.0/24",
            "delegations": [],
            "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_09_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_09_hub_vnet/subnets/AzureFirewallSubnet",
            "name": "AzureFirewallSubnet",
            "networkSecurityGroup": null,
            "privateEndpointNetworkPolicies": "Disabled",
            "privateLinkServiceNetworkPolicies": "Enabled",
            "provisioningState": "Succeeded",
            "resourceNavigationLinks": [],
            "routeTable": null,
            "serviceEndpointPolicies": [],
            "serviceEndpoints": []
          }
        ],
        "type": "Microsoft.Network/virtualNetworks"
      }
    }
    ```

    ```bash
    az network vnet subnet create \
        --resource-group $RESOURCE_GROUP \
        --vnet-name $HUB_VNET_NAME \
        --name $HUB_SUBNET_GW_NAME \
        --address-prefix $HUB_SUBNET_GW_PREFIX
    ```

    **Expected Output for Gateway Subnet creation:**
    ```json
    {
      "addressPrefix": "172.16.1.0/24",
      "delegations": [],
      "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_09_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_09_hub_vnet/subnets/GatewaySubnet",
      "name": "GatewaySubnet",
      "networkSecurityGroup": null,
      "privateEndpointNetworkPolicies": "Disabled",
      "privateLinkServiceNetworkPolicies": "Enabled",
      "provisioningState": "Succeeded",
      "resourceNavigationLinks": [],
      "routeTable": null,
      "serviceAssociationLinks": [],
      "serviceEndpointPolicies": [],
      "serviceEndpoints": []
    }
    ```

3.  **Create Spoke VNet and Subnets:**

    ```bash
    SPOKE_VNET_NAME="iatd_labs_09_spoke_vnet"
    SPOKE_SUBNET_APP_NAME="iatd_labs_09_subnet_app"
    SPOKE_SUBNET_DB_NAME="iatd_labs_09_subnet_db"
    SPOKE_SUBNET_BASTION_NAME="AzureBastionSubnet" # This name is required by Azure
    SPOKE_ADDRESS_PREFIX="172.16.2.0/16"
    SPOKE_SUBNET_APP_PREFIX="172.16.2.0/24"
    SPOKE_SUBNET_DB_PREFIX="172.16.2.128/24"
    SPOKE_SUBNET_BASTION_PREFIX="172.16.2.192/26"

    az network vnet create \
        --resource-group $RESOURCE_GROUP \
        --name $SPOKE_VNET_NAME \
        --address-prefixes $SPOKE_ADDRESS_PREFIX \
        --subnet-name $SPOKE_SUBNET_APP_NAME \
        --subnet-prefixes $SPOKE_SUBNET_APP_PREFIX \
        --location $LOCATION
    ```

    **Expected Output for Spoke VNet creation:**
    ```json
    {
      "newVNet": {
        "addressSpace": {
          "addressPrefixes": [
            "172.16.2.0/16"
          ]
        },
        "dhcpOptions": {
          "dnsServers": []
        },
        "enableDdosProtection": false,
        "enableVmProtection": false,
        "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_09_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_09_spoke_vnet",
        "location": "eastus",
        "name": "iatd_labs_09_spoke_vnet",
        "provisioningState": "Succeeded",
        "resourceGuid": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "subnets": [
          {
            "addressPrefix": "172.16.2.0/24",
            "delegations": [],
            "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_09_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_09_spoke_vnet/subnets/iatd_labs_09_subnet_app",
            "name": "iatd_labs_09_subnet_app",
            "networkSecurityGroup": null,
            "privateEndpointNetworkPolicies": "Disabled",
            "privateLinkServiceNetworkPolicies": "Enabled",
            "provisioningState": "Succeeded",
            "resourceNavigationLinks": [],
            "routeTable": null,
            "serviceEndpointPolicies": [],
            "serviceEndpoints": []
          }
        ],
        "type": "Microsoft.Network/virtualNetworks"
      }
    }
    ```

    ```bash
    az network vnet subnet create \
        --resource-group $RESOURCE_GROUP \
        --vnet-name $SPOKE_VNET_NAME \
        --name $SPOKE_SUBNET_DB_NAME \
        --address-prefix $SPOKE_SUBNET_DB_PREFIX
    ```

    **Expected Output for DB Subnet creation:**
    ```json
    {
      "addressPrefix": "172.16.2.128/24",
      "delegations": [],
      "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_09_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_09_spoke_vnet/subnets/iatd_labs_09_subnet_db",
      "name": "iatd_labs_09_subnet_db",
      "networkSecurityGroup": null,
      "privateEndpointNetworkPolicies": "Disabled",
      "privateLinkServiceNetworkPolicies": "Enabled",
      "provisioningState": "Succeeded",
      "resourceNavigationLinks": [],
      "routeTable": null,
      "serviceAssociationLinks": [],
      "serviceEndpointPolicies": [],
      "serviceEndpoints": []
    }
    ```

    ```bash
    az network vnet subnet create \
        --resource-group $RESOURCE_GROUP \
        --vnet-name $SPOKE_VNET_NAME \
        --name $SPOKE_SUBNET_BASTION_NAME \
        --address-prefix $SPOKE_SUBNET_BASTION_PREFIX
    ```

    **Expected Output for Bastion Subnet creation:**
    ```json
    {
      "addressPrefix": "172.16.2.192/26",
      "delegations": [],
      "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_09_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_09_spoke_vnet/subnets/AzureBastionSubnet",
      "name": "AzureBastionSubnet",
      "networkSecurityGroup": null,
      "privateEndpointNetworkPolicies": "Disabled",
      "privateLinkServiceNetworkPolicies": "Enabled",
      "provisioningState": "Succeeded",
      "resourceNavigationLinks": [],
      "routeTable": null,
      "serviceAssociationLinks": [],
      "serviceEndpointPolicies": [],
      "serviceEndpoints": []
    }
    ```

4.  **Create Simulated On-Premises VNet and Subnet:**

    ```bash
    ONPREM_VNET_NAME="iatd_labs_09_onprem_vnet"
    ONPREM_SUBNET_NAME="iatd_labs_09_subnet_onprem"
    ONPREM_ADDRESS_PREFIX="192.168.0.0/16"
    ONPREM_SUBNET_PREFIX="192.168.0.0/24"

    az network vnet create \
        --resource-group $RESOURCE_GROUP \
        --name $ONPREM_VNET_NAME \
        --address-prefixes $ONPREM_ADDRESS_PREFIX \
        --subnet-name $ONPREM_SUBNET_NAME \
        --subnet-prefixes $ONPREM_SUBNET_PREFIX \
        --location $LOCATION
    ```

    **Expected Output for On-Premises VNet creation:**
    ```json
    {
      "newVNet": {
        "addressSpace": {
          "addressPrefixes": [
            "192.168.0.0/16"
          ]
        },
        "dhcpOptions": {
          "dnsServers": []
        },
        "enableDdosProtection": false,
        "enableVmProtection": false,
        "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_09_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_09_onprem_vnet",
        "location": "eastus",
        "name": "iatd_labs_09_onprem_vnet",
        "provisioningState": "Succeeded",
        "resourceGuid": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "subnets": [
          {
            "addressPrefix": "192.168.0.0/24",
            "delegations": [],
            "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_09_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_09_onprem_vnet/subnets/iatd_labs_09_subnet_onprem",
            "name": "iatd_labs_09_subnet_onprem",
            "networkSecurityGroup": null,
            "privateEndpointNetworkPolicies": "Disabled",
            "privateLinkServiceNetworkPolicies": "Enabled",
            "provisioningState": "Succeeded",
            "resourceNavigationLinks": [],
            "routeTable": null,
            "serviceEndpointPolicies": [],
            "serviceEndpoints": []
          }
        ],
        "type": "Microsoft.Network/virtualNetworks"
      }
    }
    ```

### Part 2: Setting up VNet Peering

1.  **Create Peering from Hub to Spoke:**

    ```bash
    az network vnet peering create \
        --resource-group $RESOURCE_GROUP \
        --name "HubToSpoke" \
        --vnet-name $HUB_VNET_NAME \
        --remote-vnet $SPOKE_VNET_NAME \
        --allow-vnet-access \
        --allow-forwarded-traffic \
        --allow-gateway-transit
    ```

    **Expected Output:**
    ```json
    {
      "allowForwardedTraffic": true,
      "allowGatewayTransit": true,
      "allowVirtualNetworkAccess": true,
      "doNotVerifyRemoteGateways": false,
      "etag": "W/\"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx\"",
      "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_09_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_09_hub_vnet/virtualNetworkPeerings/HubToSpoke",
      "name": "HubToSpoke",
      "peeringState": "Connected",
      "provisioningState": "Succeeded",
      "remoteAddressSpace": {
        "addressPrefixes": [
          "172.16.2.0/16"
        ]
      },
      "remoteVirtualNetwork": {
        "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_09_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_09_spoke_vnet",
        "resourceGroup": "iatd_labs_09_rg"
      },
      "resourceGroup": "iatd_labs_09_rg",
      "type": "Microsoft.Network/virtualNetworks/virtualNetworkPeerings"
    }
    ```

2.  **Create Peering from Spoke to Hub:**

    ```bash
    az network vnet peering create \
        --resource-group $RESOURCE_GROUP \
        --name "SpokeToHub" \
        --vnet-name $SPOKE_VNET_NAME \
        --remote-vnet $HUB_VNET_NAME \
        --allow-vnet-access \
        --allow-forwarded-traffic \
        --use-remote-gateways
    ```

    **Expected Output:**
    ```json
    {
      "allowForwardedTraffic": true,
      "allowGatewayTransit": false,
      "allowVirtualNetworkAccess": true,
      "doNotVerifyRemoteGateways": false,
      "etag": "W/\"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx\"",
      "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_09_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_09_spoke_vnet/virtualNetworkPeerings/SpokeToHub",
      "name": "SpokeToHub",
      "peeringState": "Connected",
      "provisioningState": "Succeeded",
      "remoteAddressSpace": {
        "addressPrefixes": [
          "172.16.0.0/16"
        ]
      },
      "remoteVirtualNetwork": {
        "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_09_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_09_hub_vnet",
        "resourceGroup": "iatd_labs_09_rg"
      },
      "resourceGroup": "iatd_labs_09_rg",
      "type": "Microsoft.Network/virtualNetworks/virtualNetworkPeerings"
    }
    ```

3.  **Create Peering from Hub to On-Premises (Simulating ExpressRoute):**

    ```bash
    az network vnet peering create \
        --resource-group $RESOURCE_GROUP \
        --name "HubToOnPrem" \
        --vnet-name $HUB_VNET_NAME \
        --remote-vnet $ONPREM_VNET_NAME \
        --allow-vnet-access \
        --allow-forwarded-traffic \
        --allow-gateway-transit
    ```

    **Expected Output:**
    ```json
    {
      "allowForwardedTraffic": true,
      "allowGatewayTransit": true,
      "allowVirtualNetworkAccess": true,
      "doNotVerifyRemoteGateways": false,
      "etag": "W/\"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx\"",
      "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_09_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_09_hub_vnet/virtualNetworkPeerings/HubToOnPrem",
      "name": "HubToOnPrem",
      "peeringState": "Connected",
      "provisioningState": "Succeeded",
      "remoteAddressSpace": {
        "addressPrefixes": [
          "192.168.0.0/16"
        ]
      },
      "remoteVirtualNetwork": {
        "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_09_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_09_onprem_vnet",
        "resourceGroup": "iatd_labs_09_rg"
      },
      "resourceGroup": "iatd_labs_09_rg",
      "type": "Microsoft.Network/virtualNetworks/virtualNetworkPeerings"
    }
    ```

4.  **Create Peering from On-Premises to Hub (Simulating ExpressRoute):**

    ```bash
    az network vnet peering create \
        --resource-group $RESOURCE_GROUP \
        --name "OnPremToHub" \
        --vnet-name $ONPREM_VNET_NAME \
        --remote-vnet $HUB_VNET_NAME \
        --allow-vnet-access \
        --allow-forwarded-traffic \
        --use-remote-gateways
    ```

    **Expected Output:**
    ```json
    {
      "allowForwardedTraffic": true,
      "allowGatewayTransit": false,
      "allowVirtualNetworkAccess": true,
      "doNotVerifyRemoteGateways": false,
      "etag": "W/\"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx\"",
      "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_09_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_09_onprem_vnet/virtualNetworkPeerings/OnPremToHub",
      "name": "OnPremToHub",
      "peeringState": "Connected",
      "provisioningState": "Succeeded",
      "remoteAddressSpace": {
        "addressPrefixes": [
          "172.16.0.0/16"
        ]
      },
      "remoteVirtualNetwork": {
        "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_09_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_09_hub_vnet",
        "resourceGroup": "iatd_labs_09_rg"
      },
      "resourceGroup": "iatd_labs_09_rg",
      "type": "Microsoft.Network/virtualNetworks/virtualNetworkPeerings"
    }
    ```

### Part 3: Setting up Network Security Groups

1.  **Create NSG for App Subnet:**

    ```bash
    NSG_APP_NAME="iatd_labs_09_nsg_app"

    az network nsg create \
        --resource-group $RESOURCE_GROUP \
        --name $NSG_APP_NAME \
        --location $LOCATION
    ```

    **Expected Output:**
    ```json
    {
      "NewNSG": {
        "defaultSecurityRules": [...],
        "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_09_rg/providers/Microsoft.Network/networkSecurityGroups/iatd_labs_09_nsg_app",
        "location": "eastus",
        "name": "iatd_labs_09_nsg_app",
        "provisioningState": "Succeeded",
        "resourceGuid": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "securityRules": [],
        "type": "Microsoft.Network/networkSecurityGroups"
      }
    }
    ```

2.  **Create NSG for DB Subnet:**

    ```bash
    NSG_DB_NAME="iatd_labs_09_nsg_db"

    az network nsg create \
        --resource-group $RESOURCE_GROUP \
        --name $NSG_DB_NAME \
        --location $LOCATION
    ```

    **Expected Output:**
    ```json
    {
      "NewNSG": {
        "defaultSecurityRules": [...],
        "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_09_rg/providers/Microsoft.Network/networkSecurityGroups/iatd_labs_09_nsg_db",
        "location": "eastus",
        "name": "iatd_labs_09_nsg_db",
        "provisioningState": "Succeeded",
        "resourceGuid": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "securityRules": [],
        "type": "Microsoft.Network/networkSecurityGroups"
      }
    }
    ```

3.  **Configure NSG Rules for App Subnet:**

    ```bash
    # Allow inbound HTTP traffic from anywhere
    az network nsg rule create \
        --resource-group $RESOURCE_GROUP \
        --nsg-name $NSG_APP_NAME \
        --name AllowHTTP \
        --priority 100 \
        --destination-port-ranges 80 \
        --direction Inbound \
        --access Allow \
        --protocol Tcp \
        --source-address-prefixes '*' \
        --source-port-ranges '*' \
        --destination-address-prefixes '*'
    ```

    **Expected Output:**
    ```json
    {
      "access": "Allow",
      "description": null,
      "destinationAddressPrefix": "*",
      "destinationAddressPrefixes": [],
      "destinationPortRange": "80",
      "destinationPortRanges": [],
      "direction": "Inbound",
      "etag": "W/\"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx\"",
      "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_09_rg/providers/Microsoft.Network/networkSecurityGroups/iatd_labs_09_nsg_app/securityRules/AllowHTTP",
      "name": "AllowHTTP",
      "priority": 100,
      "protocol": "Tcp",
      "provisioningState": "Succeeded",
      "sourceAddressPrefix": "*",
      "sourceAddressPrefixes": [],
      "sourcePortRange": "*",
      "sourcePortRanges": [],
      "type": "Microsoft.Network/networkSecurityGroups/securityRules"
    }
    ```

    ```bash
    # Allow inbound SSH traffic from on-premises network only
    az network nsg rule create \
        --resource-group $RESOURCE_GROUP \
        --nsg-name $NSG_APP_NAME \
        --name AllowSSHFromOnPrem \
        --priority 110 \
        --destination-port-ranges 22 \
        --direction Inbound \
        --access Allow \
        --protocol Tcp \
        --source-address-prefixes $ONPREM_ADDRESS_PREFIX \
        --source-port-ranges '*' \
        --destination-address-prefixes '*'
    ```

    **Expected Output:**
    ```json
    {
      "access": "Allow",
      "description": null,
      "destinationAddressPrefix": "*",
      "destinationAddressPrefixes": [],
      "destinationPortRange": "22",
      "destinationPortRanges": [],
      "direction": "Inbound",
      "etag": "W/\"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx\"",
      "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_09_rg/providers/Microsoft.Network/networkSecurityGroups/iatd_labs_09_nsg_app/securityRules/AllowSSHFromOnPrem",
      "name": "AllowSSHFromOnPrem",
      "priority": 110,
      "protocol": "Tcp",
      "provisioningState": "Succeeded",
      "sourceAddressPrefix": "192.168.0.0/16",
      "sourceAddressPrefixes": [],
      "sourcePortRange": "*",
      "sourcePortRanges": [],
      "type": "Microsoft.Network/networkSecurityGroups/securityRules"
    }
    ```

4.  **Configure NSG Rules for DB Subnet:**

    ```bash
    # Allow inbound SQL traffic from App subnet only
    az network nsg rule create \
        --resource-group $RESOURCE_GROUP \
        --nsg-name $NSG_DB_NAME \
        --name AllowSQLFromApp \
        --priority 100 \
        --destination-port-ranges 1433 \
        --direction Inbound \
        --access Allow \
        --protocol Tcp \
        --source-address-prefixes $SPOKE_SUBNET_APP_PREFIX \
        --source-port-ranges '*' \
        --destination-address-prefixes '*'
    ```

    **Expected Output:**
    ```json
    {
      "access": "Allow",
      "description": null,
      "destinationAddressPrefix": "*",
      "destinationAddressPrefixes": [],
      "destinationPortRange": "1433",
      "destinationPortRanges": [],
      "direction": "Inbound",
      "etag": "W/\"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx\"",
      "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_09_rg/providers/Microsoft.Network/networkSecurityGroups/iatd_labs_09_nsg_db/securityRules/AllowSQLFromApp",
      "name": "AllowSQLFromApp",
      "priority": 100,
      "protocol": "Tcp",
      "provisioningState": "Succeeded",
      "sourceAddressPrefix": "172.16.2.0/24",
      "sourceAddressPrefixes": [],
      "sourcePortRange": "*",
      "sourcePortRanges": [],
      "type": "Microsoft.Network/networkSecurityGroups/securityRules"
    }
    ```

5.  **Associate NSGs with Subnets:**

    ```bash
    # Associate NSG with App Subnet
    az network vnet subnet update \
        --resource-group $RESOURCE_GROUP \
        --vnet-name $SPOKE_VNET_NAME \
        --name $SPOKE_SUBNET_APP_NAME \
        --network-security-group $NSG_APP_NAME
    ```

    **Expected Output:**
    ```json
    {
      "addressPrefix": "172.16.2.0/24",
      "delegations": [],
      "etag": "W/\"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx\"",
      "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_09_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_09_spoke_vnet/subnets/iatd_labs_09_subnet_app",
      "name": "iatd_labs_09_subnet_app",
      "networkSecurityGroup": {
        "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_09_rg/providers/Microsoft.Network/networkSecurityGroups/iatd_labs_09_nsg_app",
        "resourceGroup": "iatd_labs_09_rg"
      },
      "privateEndpointNetworkPolicies": "Disabled",
      "privateLinkServiceNetworkPolicies": "Enabled",
      "provisioningState": "Succeeded",
      "resourceGroup": "iatd_labs_09_rg",
      "serviceEndpointPolicies": [],
      "serviceEndpoints": []
    }
    ```

    ```bash
    # Associate NSG with DB Subnet
    az network vnet subnet update \
        --resource-group $RESOURCE_GROUP \
        --vnet-name $SPOKE_VNET_NAME \
        --name $SPOKE_SUBNET_DB_NAME \
        --network-security-group $NSG_DB_NAME
    ```

    **Expected Output:**
    ```json
    {
      "addressPrefix": "172.16.2.128/24",
      "delegations": [],
      "etag": "W/\"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx\"",
      "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_09_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_09_spoke_vnet/subnets/iatd_labs_09_subnet_db",
      "name": "iatd_labs_09_subnet_db",
      "networkSecurityGroup": {
        "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_09_rg/providers/Microsoft.Network/networkSecurityGroups/iatd_labs_09_nsg_db",
        "resourceGroup": "iatd_labs_09_rg"
      },
      "privateEndpointNetworkPolicies": "Disabled",
      "privateLinkServiceNetworkPolicies": "Enabled",
      "provisioningState": "Succeeded",
      "resourceGroup": "iatd_labs_09_rg",
      "serviceEndpointPolicies": [],
      "serviceEndpoints": []
    }
    ```

### Part 4: Setting up Azure Firewall

1.  **Create a Public IP for Azure Firewall:**

    ```bash
    FIREWALL_PIP_NAME="iatd_labs_09_pip_fw"

    az network public-ip create \
        --resource-group $RESOURCE_GROUP \
        --name $FIREWALL_PIP_NAME \
        --allocation-method Static \
        --sku Standard \
        --location $LOCATION
    ```

    **Expected Output:**
    ```json
    {
      "publicIp": {
        "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_09_rg/providers/Microsoft.Network/publicIPAddresses/iatd_labs_09_pip_fw",
        "location": "eastus",
        "name": "iatd_labs_09_pip_fw",
        "provisioningState": "Succeeded",
        "publicIPAddressVersion": "IPv4",
        "publicIPAllocationMethod": "Static",
        "resourceGuid": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "sku": {
          "name": "Standard"
        },
        "type": "Microsoft.Network/publicIPAddresses"
      }
    }
    ```

2.  **Create Azure Firewall:**

    ```bash
    FIREWALL_NAME="iatd_labs_09_azurefirewall"

    az network firewall create \
        --resource-group $RESOURCE_GROUP \
        --name $FIREWALL_NAME \
        --location $LOCATION \
        --vnet-name $HUB_VNET_NAME
    ```

    **Expected Output:**
    ```json
    {
      "applicationRuleCollections": null,
      "etag": "W/\"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx\"",
      "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_09_rg/providers/Microsoft.Network/azureFirewalls/iatd_labs_09_azurefirewall",
      "ipConfigurations": [],
      "location": "eastus",
      "name": "iatd_labs_09_azurefirewall",
      "natRuleCollections": null,
      "networkRuleCollections": null,
      "provisioningState": "Succeeded",
      "resourceGroup": "iatd_labs_09_rg",
      "sku": {
        "name": "AZFW_VNet",
        "tier": "Standard"
      },
      "tags": null,
      "threatIntelMode": "Alert",
      "type": "Microsoft.Network/azureFirewalls"
    }
    ```

3.  **Configure IP Configuration for Azure Firewall:**

    ```bash
    az network firewall ip-config create \
        --resource-group $RESOURCE_GROUP \
        --firewall-name $FIREWALL_NAME \
        --name "fw-ip-config" \
        --public-ip-address $FIREWALL_PIP_NAME \
        --vnet-name $HUB_VNET_NAME
    ```

    **Expected Output:**
    ```json
    {
      "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_09_rg/providers/Microsoft.Network/azureFirewalls/iatd_labs_09_azurefirewall/azureFirewallIpConfigurations/fw-ip-config",
      "name": "fw-ip-config",
      "privateIpAddress": "172.16.0.4",
      "publicIpAddress": {
        "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_09_rg/providers/Microsoft.Network/publicIPAddresses/iatd_labs_09_pip_fw",
        "resourceGroup": "iatd_labs_09_rg"
      },
      "subnet": {
        "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_09_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_09_hub_vnet/subnets/AzureFirewallSubnet",
        "resourceGroup": "iatd_labs_09_rg"
      },
      "type": "Microsoft.Network/azureFirewalls/azureFirewallIpConfigurations"
    }
    ```

4.  **Create Network Rules for Azure Firewall:**

    ```bash
    # Store the Firewall private IP for later use
    FIREWALL_PRIVATE_IP=$(az network firewall show --resource-group $RESOURCE_GROUP --name $FIREWALL_NAME --query "ipConfigurations[0].privateIpAddress" --output tsv)

    # Create a network rule collection to allow web traffic
    az network firewall network-rule create \
        --resource-group $RESOURCE_GROUP \
        --firewall-name $FIREWALL_NAME \
        --collection-name "AllowWebTraffic" \
        --priority 100 \
        --action "Allow" \
        --name "AllowHTTP" \
        --protocols "TCP" \
        --source-addresses "*" \
        --destination-addresses "*" \
        --destination-ports 80 443
    ```

    **Expected Output:**
    ```json
    {
      "action": {
        "type": "Allow"
      },
      "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_09_rg/providers/Microsoft.Network/azureFirewalls/iatd_labs_09_azurefirewall/networkRuleCollections/AllowWebTraffic",
      "name": "AllowWebTraffic",
      "priority": 100,
      "provisioningState": "Succeeded",
      "resourceGroup": "iatd_labs_09_rg",
      "rules": [
        {
          "destinationAddresses": [
            "*"
          ],
          "destinationPorts": [
            "80",
            "443"
          ],
          "name": "AllowHTTP",
          "protocols": [
            "TCP"
          ],
          "sourceAddresses": [
            "*"
          ]
        }
      ]
    }
    ```

    ```bash
    # Create a network rule collection to allow SSH from on-premises
    az network firewall network-rule create \
        --resource-group $RESOURCE_GROUP \
        --firewall-name $FIREWALL_NAME \
        --collection-name "AllowSSHFromOnPrem" \
        --priority 110 \
        --action "Allow" \
        --name "AllowSSH" \
        --protocols "TCP" \
        --source-addresses $ONPREM_ADDRESS_PREFIX \
        --destination-addresses "*" \
        --destination-ports 22
    ```

    **Expected Output:**
    ```json
    {
      "action": {
        "type": "Allow"
      },
      "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_09_rg/providers/Microsoft.Network/azureFirewalls/iatd_labs_09_azurefirewall/networkRuleCollections/AllowSSHFromOnPrem",
      "name": "AllowSSHFromOnPrem",
      "priority": 110,
      "provisioningState": "Succeeded",
      "resourceGroup": "iatd_labs_09_rg",
      "rules": [
        {
          "destinationAddresses": [
            "*"
          ],
          "destinationPorts": [
            "22"
          ],
          "name": "AllowSSH",
          "protocols": [
            "TCP"
          ],
          "sourceAddresses": [
            "192.168.0.0/16"
          ]
        }
      ]
    }
    ```

    az group create --name $RESOURCE_GROUP --location $LOCATION
    ```

### Part 5: Setting up Route Tables

1.  **Create Route Tables:**

    ```bash
    ROUTE_TABLE_APP_NAME="iatd_labs_09_routetable_app"
    ROUTE_TABLE_ONPREM_NAME="iatd_labs_09_routetable_onprem"

    # Create Route Table for App Subnet
    az network route-table create \
        --resource-group $RESOURCE_GROUP \
        --name $ROUTE_TABLE_APP_NAME \
        --location $LOCATION
    ```

    **Expected Output:**
    ```json
    {
      "disableBgpRoutePropagation": false,
      "etag": "W/\"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx\"",
      "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_09_rg/providers/Microsoft.Network/routeTables/iatd_labs_09_routetable_app",
      "location": "eastus",
      "name": "iatd_labs_09_routetable_app",
      "provisioningState": "Succeeded",
      "resourceGroup": "iatd_labs_09_rg",
      "routes": [],
      "type": "Microsoft.Network/routeTables"
    }
    ```bash
    # Create Route Table for On-Premises Subnet
    az network route-table create \
        --resource-group $RESOURCE_GROUP \
        --name $ROUTE_TABLE_ONPREM_NAME \
        --location $LOCATION
    ```

    **Expected Output:**
    ```json
    {
      "disableBgpRoutePropagation": false,
      "etag": "W/\"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx\"",
      "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_09_rg/providers/Microsoft.Network/routeTables/iatd_labs_09_routetable_onprem",
      "location": "eastus",
      "name": "iatd_labs_09_routetable_onprem",
      "provisioningState": "Succeeded",
      "resourceGroup": "iatd_labs_09_rg",
      "routes": [],
      "type": "Microsoft.Network/routeTables"
    }
    ```

2.  **Add Routes to Route Tables:**

    ```bash
    # Add route for internet traffic via Azure Firewall for App subnet
    az network route-table route create \
        --resource-group $RESOURCE_GROUP \
        --route-table-name $ROUTE_TABLE_APP_NAME \
        --name "ToInternet" \
        --address-prefix "0.0.0.0/0" \
        --next-hop-type VirtualAppliance \
        --next-hop-ip-address $FIREWALL_PRIVATE_IP
    ```

    **Expected Output:**
    ```json
    {
      "addressPrefix": "0.0.0.0/0",
      "etag": "W/\"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx\"",
      "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_09_rg/providers/Microsoft.Network/routeTables/iatd_labs_09_routetable_app/routes/ToInternet",
      "name": "ToInternet",
      "nextHopIpAddress": "172.16.0.4",
      "nextHopType": "VirtualAppliance",
      "provisioningState": "Succeeded",
      "resourceGroup": "iatd_labs_09_rg",
      "type": "Microsoft.Network/routeTables/routes"
    }
    ```

    ```bash
    # Add route for spoke VNet traffic via Azure Firewall for On-Premises subnet
    az network route-table route create \
        --resource-group $RESOURCE_GROUP \
        --route-table-name $ROUTE_TABLE_ONPREM_NAME \
        --name "ToSpokeVNet" \
        --address-prefix $SPOKE_ADDRESS_PREFIX \
        --next-hop-type VirtualAppliance \
        --next-hop-ip-address $FIREWALL_PRIVATE_IP
    ```

    **Expected Output:**
    ```json
    {
      "addressPrefix": "172.16.2.0/16",
      "etag": "W/\"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx\"",
      "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_09_rg/providers/Microsoft.Network/routeTables/iatd_labs_09_routetable_onprem/routes/ToSpokeVNet",
      "name": "ToSpokeVNet",
      "nextHopIpAddress": "172.16.0.4",
      "nextHopType": "VirtualAppliance",
      "provisioningState": "Succeeded",
      "resourceGroup": "iatd_labs_09_rg",
      "type": "Microsoft.Network/routeTables/routes"
    }
    ```

3.  **Associate Route Tables with Subnets:**

    ```bash
    # Associate route table with App subnet
    az network vnet subnet update \
        --resource-group $RESOURCE_GROUP \
        --vnet-name $SPOKE_VNET_NAME \
        --name $SPOKE_SUBNET_APP_NAME \
        --route-table $ROUTE_TABLE_APP_NAME
    ```

    **Expected Output:**
    ```json
    {
      "addressPrefix": "172.16.2.0/24",
      "delegations": [],
      "etag": "W/\"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx\"",
      "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_09_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_09_spoke_vnet/subnets/iatd_labs_09_subnet_app",
      "name": "iatd_labs_09_subnet_app",
      "networkSecurityGroup": {
        "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_09_rg/providers/Microsoft.Network/networkSecurityGroups/iatd_labs_09_nsg_app",
        "resourceGroup": "iatd_labs_09_rg"
      },
      "privateEndpointNetworkPolicies": "Disabled",
      "privateLinkServiceNetworkPolicies": "Enabled",
      "provisioningState": "Succeeded",
      "resourceGroup": "iatd_labs_09_rg",
      "routeTable": {
        "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_09_rg/providers/Microsoft.Network/routeTables/iatd_labs_09_routetable_app",
        "resourceGroup": "iatd_labs_09_rg"
      },
      "serviceEndpointPolicies": [],
      "serviceEndpoints": []
    }
    ```

    ```bash
    # Associate route table with On-Premises subnet
    az network vnet subnet update \
        --resource-group $RESOURCE_GROUP \
        --vnet-name $ONPREM_VNET_NAME \
        --name $ONPREM_SUBNET_NAME \
        --route-table $ROUTE_TABLE_ONPREM_NAME
    ```

    **Expected Output:**
    ```json
    {
      "addressPrefix": "192.168.0.0/24",
      "delegations": [],
      "etag": "W/\"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx\"",
      "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_09_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_09_onprem_vnet/subnets/iatd_labs_09_subnet_onprem",
      "name": "iatd_labs_09_subnet_onprem",
      "networkSecurityGroup": null,
      "privateEndpointNetworkPolicies": "Disabled",
      "privateLinkServiceNetworkPolicies": "Enabled",
      "provisioningState": "Succeeded",
      "resourceGroup": "iatd_labs_09_rg",
      "routeTable": {
        "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_09_rg/providers/Microsoft.Network/routeTables/iatd_labs_09_routetable_onprem",
        "resourceGroup": "iatd_labs_09_rg"
      },
      "serviceEndpointPolicies": [],
      "serviceEndpoints": []
    }
    ```

### Part 6: Setting up Private Endpoint for SQL Database

1.  **Enable Service Endpoints for DB Subnet:**

    ```bash
    # Enable Microsoft.Sql service endpoint on DB subnet
    az network vnet subnet update \
        --resource-group $RESOURCE_GROUP \
        --vnet-name $SPOKE_VNET_NAME \
        --name $SPOKE_SUBNET_DB_NAME \
        --service-endpoints Microsoft.Sql
    ```

    **Expected Output:**
    ```json
    {
      "addressPrefix": "172.16.2.128/24",
      "delegations": [],
      "etag": "W/\"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx\"",
      "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_09_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_09_spoke_vnet/subnets/iatd_labs_09_subnet_db",
      "name": "iatd_labs_09_subnet_db",
      "networkSecurityGroup": {
        "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_09_rg/providers/Microsoft.Network/networkSecurityGroups/iatd_labs_09_nsg_db",
        "resourceGroup": "iatd_labs_09_rg"
      },
      "privateEndpointNetworkPolicies": "Disabled",
      "privateLinkServiceNetworkPolicies": "Enabled",
      "provisioningState": "Succeeded",
      "resourceGroup": "iatd_labs_09_rg",
      "serviceEndpointPolicies": [],
      "serviceEndpoints": [
        {
          "locations": [
            "*"
          ],
          "provisioningState": "Succeeded",
          "service": "Microsoft.Sql"
        }
      ]
    }
    ```
2.  **Create Private Endpoint for SQL Database:**

    ```bash
    # Set your SQL Server name and resource group
    SQL_SERVER_NAME="your-sql-server-name" # Replace with your SQL Server name
    SQL_SERVER_RG="your-sql-server-rg" # Replace with your SQL Server resource group

    # Get the resource ID of your SQL Server
    SQL_SERVER_ID=$(az sql server show --name $SQL_SERVER_NAME --resource-group $SQL_SERVER_RG --query id --output tsv)

    # Create private endpoint
    PRIVATE_ENDPOINT_NAME="iatd_labs_09_sqldb_pe"

    az network private-endpoint create \
        --resource-group $RESOURCE_GROUP \
        --name $PRIVATE_ENDPOINT_NAME \
        --vnet-name $SPOKE_VNET_NAME \
        --subnet $SPOKE_SUBNET_DB_NAME \
        --private-connection-resource-id $SQL_SERVER_ID \
        --group-id sqlServer \
        --connection-name "sqldbconnection" \
        --location $LOCATION
    ```

    **Expected Output:**
    ```json
    {
      "customDnsConfigs": null,
      "customNetworkInterfaceName": null,
      "etag": "W/\"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx\"",
      "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_09_rg/providers/Microsoft.Network/privateEndpoints/iatd_labs_09_sqldb_pe",
      "location": "eastus",
      "manualPrivateLinkServiceConnections": null,
      "name": "iatd_labs_09_sqldb_pe",
      "networkInterfaces": [
        {
          "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_09_rg/providers/Microsoft.Network/networkInterfaces/iatd_labs_09_sqldb_pe.nic.xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
          "resourceGroup": "iatd_labs_09_rg"
        }
      ],
      "privateLinkServiceConnections": [
        {
          "etag": "W/\"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx\"",
          "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_09_rg/providers/Microsoft.Network/privateEndpoints/iatd_labs_09_sqldb_pe/privateLinkServiceConnections/sqldbconnection",
          "name": "sqldbconnection",
          "privateLinkServiceConnectionState": {
            "actionsRequired": "None",
            "description": "Auto-approved",
            "status": "Approved"
          },
          "privateLinkServiceId": "/subscriptions/<subscription_id>/resourceGroups/your-sql-server-rg/providers/Microsoft.Sql/servers/your-sql-server-name",
          "provisioningState": "Succeeded",
          "type": "Microsoft.Network/privateEndpoints/privateLinkServiceConnections"
        }
      ],
      "provisioningState": "Succeeded",
      "resourceGroup": "iatd_labs_09_rg",
      "subnet": {
        "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_09_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_09_spoke_vnet/subnets/iatd_labs_09_subnet_db",
        "resourceGroup": "iatd_labs_09_rg"
      },
      "type": "Microsoft.Network/privateEndpoints"
    }
    ```

### Part 7: Verification and Testing

1.  **Verify VNet Peering Status:**

    ```bash
    # Verify Hub to Spoke peering
    az network vnet peering show \
        --resource-group $RESOURCE_GROUP \
        --vnet-name $HUB_VNET_NAME \
        --name "HubToSpoke" \
        --query peeringState -o tsv
    ```

    **Expected Output:**
    ```
    Connected
    ```

    ```bash
    # Verify Spoke to Hub peering
    az network vnet peering show \
        --resource-group $RESOURCE_GROUP \
        --vnet-name $SPOKE_VNET_NAME \
        --name "SpokeToHub" \
        --query peeringState -o tsv
    ```

    **Expected Output:**
    ```
    Connected
    ```

2.  **Verify Azure Firewall Rules:**

    ```bash
    # Verify network rules
    az network firewall network-rule collection list \
        --resource-group $RESOURCE_GROUP \
        --firewall-name $FIREWALL_NAME \
        --query "[].{Name:name, Priority:priority, Action:action.type, Rules:[rules[].{Name:name, Protocols:protocols, SourceAddresses:sourceAddresses, DestinationAddresses:destinationAddresses, DestinationPorts:destinationPorts}]}" \
        --output table
    ```

    **Expected Output:**
    ```
    Name                Priority    Action    Rules
    ------------------  ----------  --------  ---------------------------------------------------------------------------------------------------------------------
    AllowWebTraffic    100         Allow     [{'Name': 'AllowHTTP', 'Protocols': ['TCP'], 'SourceAddresses': ['*'], 'DestinationAddresses': ['*'], 'DestinationPorts': ['80', '443']}]
    AllowSSHFromOnPrem  110         Allow     [{'Name': 'AllowSSH', 'Protocols': ['TCP'], 'SourceAddresses': ['192.168.0.0/16'], 'DestinationAddresses': ['*'], 'DestinationPorts': ['22']}]
    ```

3.  **Verify Private Endpoint:**

    ```bash
    # Verify private endpoint
    az network private-endpoint show \
        --resource-group $RESOURCE_GROUP \
        --name $PRIVATE_ENDPOINT_NAME \
        --query privateLinkServiceConnections[0].privateLinkServiceConnectionState.status -o tsv
    ```

    **Expected Output:**
    ```
    Approved
    ```

### Cleanup

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
    Resource group 'iatd_labs_09_rg' could not be found.
    ```

3.  **Clean Local Environment Variables:**

    ```bash
    unset RESOURCE_GROUP LOCATION HUB_VNET_NAME HUB_SUBNET_FW_NAME HUB_SUBNET_GW_NAME HUB_ADDRESS_PREFIX HUB_SUBNET_FW_PREFIX HUB_SUBNET_GW_PREFIX SPOKE_VNET_NAME SPOKE_SUBNET_APP_NAME SPOKE_SUBNET_DB_NAME SPOKE_SUBNET_BASTION_NAME SPOKE_ADDRESS_PREFIX SPOKE_SUBNET_APP_PREFIX SPOKE_SUBNET_DB_PREFIX SPOKE_SUBNET_BASTION_PREFIX ONPREM_VNET_NAME ONPREM_SUBNET_NAME ONPREM_ADDRESS_PREFIX ONPREM_SUBNET_PREFIX NSG_APP_NAME NSG_DB_NAME FIREWALL_PIP_NAME FIREWALL_NAME FIREWALL_PRIVATE_IP ROUTE_TABLE_APP_NAME ROUTE_TABLE_ONPREM_NAME PRIVATE_ENDPOINT_NAME SQL_SERVER_NAME SQL_SERVER_RG SQL_SERVER_ID
    ```

**Learning Outcomes:**

*   Successfully designed and implemented a hybrid cloud environment with secure connectivity between on-premises and Azure resources.
*   Configured a hub-and-spoke network topology with Azure Firewall for centralized security.
*   Implemented private access to Azure SQL Database using Private Endpoints.
*   Applied network security using NSGs with appropriate security rules.
*   Configured traffic routing using User-Defined Routes to ensure all traffic flows through Azure Firewall.
*   Simulated an ExpressRoute connection using VNet peering.

**Next Steps:**

*   Explore Azure ExpressRoute for production-grade hybrid connectivity.
*   Implement Azure Private DNS Zones for name resolution of private endpoints.
*   Set up Azure Monitor and Network Watcher for comprehensive network monitoring.
*   Implement Azure DDoS Protection for enhanced security.
        --resource-group $RESOURCE_GROUP \
        --name $SPOKE_VNET_NAME \
        --address-prefixes $ADDRESS_PREFIX \
        --location $LOCATION \
        --subnet-name $SUBNET_APP_NAME \
        --subnet-prefixes $SUBNET_APP_PREFIX

    az network vnet subnet create \
        --resource-group $RESOURCE_GROUP \
        --vnet-name $SPOKE_VNET_NAME \
        --name $SUBNET_DB_NAME \
        --address-prefix $SUBNET_DB_PREFIX \
        --disable-private-endpoint-network-policies true

    az network vnet subnet create \
        --resource-group $RESOURCE_GROUP \
        --vnet-name $SPOKE_VNET_NAME \
        --name AzureBastionSubnet \
        --address-prefix $SUBNET_BASTION
    ```

### Part 2: Setting up Network Security Groups (NSGs)

1.  **Create NSG for App Subnet:**

    ```bash
    NSG_APP_NAME="iatd_labs_09_nsg_app"

    az network nsg create \
        --resource-group $RESOURCE_GROUP \
        --name $NSG_APP_NAME \
        --location $LOCATION
    ```

2.  **Create NSG Rules (App NSG):**

    ```bash
    NSG_APP_NAME="iatd_labs_09_nsg_app"

    az network nsg rule create \
        --resource-group $RESOURCE_GROUP \
        --nsg-name $NSG_APP_NAME \
        --name AllowOutboundToDB \
        --priority 100 \
        --source-address-prefixes 10.0.1.0/24 \
        --destination-address-prefixes 10.0.2.0/24 \
        --destination-port-ranges '*' \
        --access Allow \
        --protocol Tcp \
        --direction Outbound

        az network nsg rule create \
        --resource-group $RESOURCE_GROUP \
        --nsg-name $NSG_APP_NAME \
        --name AllowOutboundToStorage \
        --priority 110 \
        --source-address-prefixes 10.0.1.0/24 \
        --destination-service-tags Storage \
        --destination-port-ranges 443 \
        --access Allow \
        --protocol Tcp \
        --direction Outbound
    ```

3.  **Associate NSGs with App Subnet:**

    ```bash
    az network vnet subnet update \
        --resource-group $RESOURCE_GROUP \
        --vnet-name $SPOKE_VNET_NAME \
        --name $SUBNET_APP_NAME \
        --network-security-group $NSG_APP_NAME
    ```

### Part 3: Creating Private Endpoint for Azure SQL Database

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
            *   Resource group: `iatd_labs_09_rg`
            *   Name: `iatd_labs_09_sqldb_pe`
            *   Region: Your region.
        *   **Resource Tab:**
            *   Connection method: "My directory"
            *   Subscription: Your subscription.
            *   Resource type: Microsoft.Sql/servers
            *   Resource: Select your SQL Server (or paste the Resource ID obtained above)
            *   Target subresource: SqlServer
        *   **Virtual Network Tab:**
            *   Virtual network: `iatd_labs_09_spoke_vnet`
            *   Subnet: `iatd_labs_09_subnet_db`
        *   **DNS Tab:**
            *   Integrate with private DNS zone: Yes
            * This zone must be named `privatelink.database.windows.net`

    *   Click **Review + create**, then **Create**.

### Part 4: Configuring User-Defined Routes (UDRs)

1.  **Create a Route Table for App Subnet:**

    ```bash
    ROUTE_TABLE_APP_NAME="iatd_labs_09_routetable_app"

    az network route-table create \
        --resource-group $RESOURCE_GROUP \
        --name $ROUTE_TABLE_APP_NAME \
        --location $LOCATION
    ```

2.  **Add a Route to the Route Table (Direct traffic to on-prem 192.168.1.0/24 via next hop):**

    ```bash
    ROUTE_TABLE_APP_NAME="iatd_labs_09_routetable_app"
    #Replace with On-Premises Network Virtual Appliance IP, or Hub Router
    NEXT_HOP_IP_ADDRESS="<Replace with NEXT HOP IP ADDRESS>"

    az network route-table route create \
        --resource-group $RESOURCE_GROUP \
        --name RouteToOnPrem \
        --route-table-name $ROUTE_TABLE_APP_NAME \
        --address-prefix 192.168.1.0/24 \
        --next-hop-type VirtualAppliance \
        --next-hop-ip-address $NEXT_HOP_IP_ADDRESS

    az network route-table route create \
        --resource-group $RESOURCE_GROUP \
        --name RouteToDB \
        --route-table-name $ROUTE_TABLE_APP_NAME \
        --address-prefix 10.0.2.0/24 \
        --next-hop-type VirtualAppliance \
        --next-hop-ip-address $NEXT_HOP_IP_ADDRESS

     az network route-table route create \
        --resource-group $RESOURCE_GROUP \
        --name RouteToOtherVnets \
        --route-table-name $ROUTE_TABLE_APP_NAME \
        --address-prefix 10.100.0.0/16 \
        --next-hop-type VirtualAppliance \
        --next-hop-ip-address $NEXT_HOP_IP_ADDRESS
    ```

3.  **Associate the Route Table with the App Subnet:**

    ```bash
    az network vnet subnet update \
        --resource-group $RESOURCE_GROUP \
        --vnet-name $SPOKE_VNET_NAME \
        --name $SUBNET_APP_NAME \
        --route-table $ROUTE_TABLE_APP_NAME
    ```

### Part 5: Implement Azure Bastion

1.  **Create Azure Bastion (Portal):**
    *   Search for "Bastion" and select.
    *   Click **Create**.
        *   Subscription: Your subscription
        *   Resource group: `iatd_labs_09_rg`
        *   Name: `iatd_labs_09_bastion`
        *   Region: Your Region.
        *   Virtual network: Select `iatd_labs_09_spoke_vnet`
        *   Subnet: Select `AzureBastionSubnet`
        *   Public IP address: Create new
    * Click **Review + Create** and then **Create**

### Part 6: Peering with Hub VNet

1.  **Assumptions**: You need to have permissions and access rights to modify a Hub VNet, and it should already exist. Replace all `<Placeholders>` accordingly.
2.  **Create VNet Peering from Spoke to Hub:**

    ```bash
    HUB_VNET_NAME="<Your Hub VNet Name>" #Replace with real vnet name from hub to peer
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
     HUB_VNET_NAME="<Your Hub VNet Name>" #Replace with real vnet name from hub to peer

    az network vnet peering create \
        --resource-group <Resource Group Hub> \
        --name HubToSpoke \
        --vnet-name  $HUB_VNET_NAME \
        --remote-vnet $SPOKE_VNET_NAME \
        --allow-vnet-access
        --allow-forwarded-traffic
        --use-remote-gateways
    ```

### Part 7: Configure Azure Firewall in Hub

1.  **Check for Azure Firewall in Hub:** Check the Hub VNet's resource group and ensure you have an active Azure Firewall. We are not deploying one in this lab, you need to ensure it exists, and you have permissions to administer it.
2.  **Create a Firewall Rule (Portal):**
    *   Navigate to the `<Your Hub Azure Firewall>` in the Azure Portal.
    *   Select **Rules** under **Settings**.
    *   Click **Add rule collection**.
        *   Name: AllowSQLFromSpoke
        *   Rule collection type: Network
        *   Priority: 100
        *   Action: Allow
        *   Add Rule:
            *   Name: AllowSQL
            *   Protocol: TCP
            *   Source type: Address
            *   Source: 10.0.1.0/24 (App Subnet)
            *   Destination type: Address
            *   Destination: 10.0.2.0/24 (DB Subnet)
            *   Destination ports: 1433
    * Click **Add** and then **Create**.
3.  **Enable Diagnostic Settings for Azure Firewall:**

    *   This sends Azure Firewall logs to a Log Analytics Workspace for analysis.

### Part 8: Creating Virtual Machine in App Subnet

1.  **Create App Virtual Machine (`iatd_labs_09_vm_app_01`) (Portal):**
    *   Search for "Virtual machines" and select.
    *   Click **Create** -> **Azure virtual machine**.
        *   **Basics Tab:**
            *   Subscription: Your subscription.
            *   Resource group: `iatd_labs_09_rg`
            *   Virtual machine name: `iatd_labs_09_vm_app_01`
            *   Region: Your region.
            *   Image: `Ubuntu Server 22.04 LTS`
            *   Size: Choose a size (e.g., Standard_B1ls).
            *   Username: `azureuser`
            *   Authentication type: `Password`
            *   Password: Set a password.
        *   **Networking Tab:**
            *   Virtual network: `iatd_labs_09_spoke_vnet`
            *   Subnet: `iatd_labs_09_subnet_app`
            *   Public IP: **None** (Important: No Public IP)
            *   NIC network security group: `None`
        *   **Management Tab:**
            *   Disable boot diagnostics
    *   Click **Review + create**, then **Create**.

### Part 9: Testing Connectivity and Isolation

1.  **Connect to App VM:**
    *   Connect to the `iatd_labs_09_vm_app_01` VM using Azure Bastion.

2.  **Test Connectivity from App VM to Azure SQL Database:**
       *   You need a "jump box" VM with a public IP in the same VNet or use Azure Bastion to connect to the App VM.
        *   SSH into the jump box VM.
        *   From the jump box, SSH into the `iatd_labs_09_vm_app_01` VM using Azure Bastion.
     * From the `iatd_labs_09_vm_app_01` VM, use `sqlcmd` to connect to the Azure SQL Database using its fully qualified domain name (FQDN) and the appropriate credentials. You will need to find the FQDN under the SQL server properties
    *   If the connection is successful, it confirms that the App VM can connect to the SQL Database *privately* through the Private Endpoint.

3.  **Test connectivity to Storage Account on Port 443:**

    ```bash
    curl -v https://<Your Storage Account Name>.blob.core.windows.net
    ```

    * The connection should be successful

4.  **Simulate On-Premises Access:**
    *  **Concept**: From the On-premises network, machines should be able to connect to Resources inside Azure. To simulate this you can create a VM with a public IP in on-premise network, or simulate it from your PC (this would mean there will be 2 network hops).
    *  Check that you can ping, telnet or use a database browser to access to resources inside azure network. You might need to setup special firewall or DNS rules for this to work

5.  **Review Azure Firewall Logs:**
    *   Examine the Azure Firewall logs in your Log Analytics Workspace to verify that traffic between the App and DB subnets is being logged.

### Post-Lab: Cleanup

To avoid incurring unnecessary costs, it's essential to clean up the resources created during this lab.

1.  **Delete Resource Group:**

    ```bash
    az group delete --name $RESOURCE_GROUP --yes
    ```

**Learning Outcomes:**

*   Successfully created a hybrid cloud environment with a simulated ExpressRoute connection.
*   Successfully deployed and configured VNets, subnets, and NSGs to isolate tiers.
*   Successfully created a Private Endpoint for an Azure SQL Database.
*   Successfully configured UDRs to route traffic through Azure Firewall.
*   Successfully configured Azure Firewall rules to allow specific traffic between tiers.
*   Successfully configured peering with Hub VNet and Azure Firewall.
*   Successfully verified that the on-premises network can access Azure SQL Database privately.
*   Successfully verified that the web tier is publicly accessible but isolated from direct app/DB access.
*   Successfully verified that the app tier is private, communicating only with the specified resources through Azure Firewall.
*   Successfully verified that the DB tier is fully isolated, accessible only via Private Endpoint.
*   Demonstrated a comprehensive understanding of Azure hybrid networking and security best practices.
