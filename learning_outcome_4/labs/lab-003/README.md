## IATD Microcredential Cloud Networking: Lab 3 - Securing Communication with Network Security Groups (NSGs)

**Objective:** This lab demonstrates how to secure communication between different subnets within an Azure Virtual Network (VNet) using Network Security Groups (NSGs). You'll configure NSG rules to allow only specific traffic between an application subnet and a database subnet, implementing a basic security posture.

**Estimated Time:** 60 - 75 minutes

**Prerequisites:**

1.  **Azure Subscription:** Requires an active Azure subscription. A free account is available at [https://azure.microsoft.com/free/](https://azure.microsoft.com/free/).
2.  **Basic knowledge of Azure Networking:** Familiarity with Virtual Networks, Subnets, and Network Security Groups.

**Let's get started!**

### Lab Conventions

*   **Azure Cloud Shell:** All PowerShell and CLI interactions will occur within the Azure Cloud Shell environment.
*   **Azure Portal:** The Azure Portal will be used for visualization and configuration tasks.
*   **Naming Conventions:** Resource names follow the convention `iatd_labs_03_*`.
*   **IP Address Range:** The `172.16.0.0/16` range will be used (as per standardization).
*   **Location:** Choose a consistent Azure region.

#### Resource Naming

*   **Resource Group:** `iatd_labs_03_rg`
*   **Virtual Network:** `iatd_labs_03_vnet`
*   **Subnet-App:** `iatd_labs_03_subnet_app`
*   **Subnet-DB:** `iatd_labs_03_subnet_db`
*   **NSG-App:** `iatd_labs_03_nsg_app`
*   **NSG-DB:** `iatd_labs_03_nsg_db`
*   **VM-App-01:** `iatd_labs_03_vm_app_01`
*   **VM-DB-01:** `iatd_labs_03_vm_db_01`

### Part 1: Setting up the Virtual Network and Subnets

1.  **Create a Resource Group:**

    ```bash
    RESOURCE_GROUP="iatd_labs_03_rg"
    LOCATION="eastus" # Your Region

    az group create --name $RESOURCE_GROUP --location $LOCATION
    ```

    **Expected Output:**
    ```json
    {
      "id": "/subscriptions/<your_subscription_id>/resourceGroups/iatd_labs_03_rg",
      "location": "eastus",
      "managedBy": null,
      "name": "iatd_labs_03_rg",
      "properties": {
        "provisioningState": "Succeeded"
      },
      "tags": null,
      "type": "Microsoft.Resources/resourceGroups"
    }
    ```

2.  **Create a Virtual Network with Subnets:**

    ```bash
    VNET_NAME="iatd_labs_03_vnet"
    SUBNET_APP_NAME="iatd_labs_03_subnet_app"
    SUBNET_DB_NAME="iatd_labs_03_subnet_db"
    ADDRESS_PREFIX="172.16.0.0/16"
    SUBNET_APP_PREFIX="172.16.0.0/24"
    SUBNET_DB_PREFIX="172.16.1.0/24"

    az network vnet create \
        --resource-group $RESOURCE_GROUP \
        --name $VNET_NAME \
        --address-prefixes $ADDRESS_PREFIX \
        --subnet-name $SUBNET_APP_NAME \
        --subnet-prefixes $SUBNET_APP_PREFIX \
        --location $LOCATION
    ```

    **Expected Output for VNet creation:**
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
        "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_03_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_03_vnet",
        "location": "eastus",
        "name": "iatd_labs_03_vnet",
        "provisioningState": "Succeeded",
        "resourceGuid": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "subnets": [
          {
            "addressPrefix": "172.16.0.0/24",
            "delegations": [],
            "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_03_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_03_vnet/subnets/iatd_labs_03_subnet_app",
            "name": "iatd_labs_03_subnet_app",
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
        --vnet-name $VNET_NAME \
        --name $SUBNET_DB_NAME \
        --address-prefix $SUBNET_DB_PREFIX
    ```

    **Expected Output for DB Subnet creation:**
    ```json
    {
      "addressPrefix": "172.16.1.0/24",
      "delegations": [],
      "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_03_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_03_vnet/subnets/iatd_labs_03_subnet_db",
      "name": "iatd_labs_03_subnet_db",
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

### Part 2: Creating and Configuring Network Security Groups (NSGs)

1.  **Create NSG for App Subnet:**

    ```bash
    NSG_APP_NAME="iatd_labs_03_nsg_app"

    az network nsg create \
        --resource-group $RESOURCE_GROUP \
        --name $NSG_APP_NAME \
        --location $LOCATION
    ```

    **Expected Output:**
    ```json
    {
      "defaultSecurityRules": [
        {
          "access": "Allow",
          "description": "Allow inbound traffic from all VMs in VNET",
          "destinationAddressPrefix": "VirtualNetwork",
          "destinationPortRange": "*",
          "direction": "Inbound",
          "name": "AllowVnetInBound",
          "priority": 65000,
          "protocol": "*",
          "provisioningState": "Succeeded",
          "sourceAddressPrefix": "VirtualNetwork",
          "sourcePortRange": "*"
        },
        {
          "access": "Allow",
          "description": "Allow inbound traffic from azure load balancer",
          "destinationAddressPrefix": "*",
          "destinationPortRange": "*",
          "direction": "Inbound",
          "name": "AllowAzureLoadBalancerInBound",
          "priority": 65001,
          "protocol": "*",
          "provisioningState": "Succeeded",
          "sourceAddressPrefix": "AzureLoadBalancer",
          "sourcePortRange": "*"
        },
        {
          "access": "Deny",
          "description": "Deny all inbound traffic",
          "destinationAddressPrefix": "*",
          "destinationPortRange": "*",
          "direction": "Inbound",
          "name": "DenyAllInBound",
          "priority": 65500,
          "protocol": "*",
          "provisioningState": "Succeeded",
          "sourceAddressPrefix": "*",
          "sourcePortRange": "*"
        }
      ],
      "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_03_rg/providers/Microsoft.Network/networkSecurityGroups/iatd_labs_03_nsg_app",
      "location": "eastus",
      "name": "iatd_labs_03_nsg_app",
      "provisioningState": "Succeeded",
      "resourceGuid": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "securityRules": [],
      "type": "Microsoft.Network/networkSecurityGroups"
    }
    ```

2.  **Create NSG for DB Subnet:**

    ```bash
    NSG_DB_NAME="iatd_labs_03_nsg_db"

    az network nsg create \
        --resource-group $RESOURCE_GROUP \
        --name $NSG_DB_NAME \
        --location $LOCATION
    ```

    **Expected Output:**
    ```json
    {
      "defaultSecurityRules": [
        {
          "access": "Allow",
          "description": "Allow inbound traffic from all VMs in VNET",
          "destinationAddressPrefix": "VirtualNetwork",
          "destinationPortRange": "*",
          "direction": "Inbound",
          "name": "AllowVnetInBound",
          "priority": 65000,
          "protocol": "*",
          "provisioningState": "Succeeded",
          "sourceAddressPrefix": "VirtualNetwork",
          "sourcePortRange": "*"
        },
        {
          "access": "Allow",
          "description": "Allow inbound traffic from azure load balancer",
          "destinationAddressPrefix": "*",
          "destinationPortRange": "*",
          "direction": "Inbound",
          "name": "AllowAzureLoadBalancerInBound",
          "priority": 65001,
          "protocol": "*",
          "provisioningState": "Succeeded",
          "sourceAddressPrefix": "AzureLoadBalancer",
          "sourcePortRange": "*"
        },
        {
          "access": "Deny",
          "description": "Deny all inbound traffic",
          "destinationAddressPrefix": "*",
          "destinationPortRange": "*",
          "direction": "Inbound",
          "name": "DenyAllInBound",
          "priority": 65500,
          "protocol": "*",
          "provisioningState": "Succeeded",
          "sourceAddressPrefix": "*",
          "sourcePortRange": "*"
        }
      ],
      "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_03_rg/providers/Microsoft.Network/networkSecurityGroups/iatd_labs_03_nsg_db",
      "location": "eastus",
      "name": "iatd_labs_03_nsg_db",
      "provisioningState": "Succeeded",
      "resourceGuid": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "securityRules": [],
      "type": "Microsoft.Network/networkSecurityGroups"
    }
    ```

3.  **Associate NSGs with Subnets:**

    ```bash
    az network vnet subnet update \
        --resource-group $RESOURCE_GROUP \
        --vnet-name $VNET_NAME \
        --name $SUBNET_APP_NAME \
        --network-security-group $NSG_APP_NAME
    ```

    **Expected Output:**
    ```json
    {
      "addressPrefix": "172.16.0.0/24",
      "delegations": [],
      "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_03_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_03_vnet/subnets/iatd_labs_03_subnet_app",
      "name": "iatd_labs_03_subnet_app",
      "networkSecurityGroup": {
        "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_03_rg/providers/Microsoft.Network/networkSecurityGroups/iatd_labs_03_nsg_app",
        "resourceGroup": "iatd_labs_03_rg"
      },
      "privateEndpointNetworkPolicies": "Disabled",
      "privateLinkServiceNetworkPolicies": "Enabled",
      "provisioningState": "Succeeded",
      "resourceNavigationLinks": [],
      "serviceEndpointPolicies": [],
      "serviceEndpoints": []
    }
    ```

    ```bash
    az network vnet subnet update \
        --resource-group $RESOURCE_GROUP \
        --vnet-name $VNET_NAME \
        --name $SUBNET_DB_NAME \
        --network-security-group $NSG_DB_NAME
    ```

    **Expected Output:**
    ```json
    {
      "addressPrefix": "172.16.1.0/24",
      "delegations": [],
      "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_03_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_03_vnet/subnets/iatd_labs_03_subnet_db",
      "name": "iatd_labs_03_subnet_db",
      "networkSecurityGroup": {
        "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_03_rg/providers/Microsoft.Network/networkSecurityGroups/iatd_labs_03_nsg_db",
        "resourceGroup": "iatd_labs_03_rg"
      },
      "privateEndpointNetworkPolicies": "Disabled",
      "privateLinkServiceNetworkPolicies": "Enabled",
      "provisioningState": "Succeeded",
      "resourceNavigationLinks": [],
      "serviceEndpointPolicies": [],
      "serviceEndpoints": []
    }
    ```

4.  **Configure NSG-App (Allow Outbound to DB Subnet):**

    ```bash
    az network nsg rule create \
        --resource-group $RESOURCE_GROUP \
        --nsg-name $NSG_APP_NAME \
        --name AllowOutboundToDB \
        --priority 100 \
        --source-address-prefixes 172.16.0.0/24 \
        --destination-address-prefixes 172.16.1.0/24 \
        --destination-port-ranges '*' \
        --access Allow \
        --protocol '*' \
        --direction Outbound
    ```

    **Expected Output:**
    ```json
    {
      "access": "Allow",
      "description": null,
      "destinationAddressPrefix": null,
      "destinationAddressPrefixes": [
        "172.16.1.0/24"
      ],
      "destinationPortRange": null,
      "destinationPortRanges": [
        "*"
      ],
      "direction": "Outbound",
      "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_03_rg/providers/Microsoft.Network/networkSecurityGroups/iatd_labs_03_nsg_app/securityRules/AllowOutboundToDB",
      "name": "AllowOutboundToDB",
      "priority": 100,
      "protocol": "*",
      "provisioningState": "Succeeded",
      "sourceAddressPrefix": null,
      "sourceAddressPrefixes": [
        "172.16.0.0/24"
      ],
      "sourcePortRange": "*",
      "sourcePortRanges": []
    }
    ```

5.  **Configure NSG-DB (Allow Inbound from App Subnet):**

    ```bash
    az network nsg rule create \
        --resource-group $RESOURCE_GROUP \
        --nsg-name $NSG_DB_NAME \
        --name AllowInboundFromApp \
        --priority 100 \
        --source-address-prefixes 172.16.0.0/24 \
        --destination-port-ranges '*' \
        --access Allow \
        --protocol '*' \
        --direction Inbound
    ```

    **Expected Output:**
    ```json
    {
      "access": "Allow",
      "description": null,
      "destinationAddressPrefix": "*",
      "destinationAddressPrefixes": [],
      "destinationPortRange": null,
      "destinationPortRanges": [
        "*"
      ],
      "direction": "Inbound",
      "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_03_rg/providers/Microsoft.Network/networkSecurityGroups/iatd_labs_03_nsg_db/securityRules/AllowInboundFromApp",
      "name": "AllowInboundFromApp",
      "priority": 100,
      "protocol": "*",
      "provisioningState": "Succeeded",
      "sourceAddressPrefix": null,
      "sourceAddressPrefixes": [
        "172.16.0.0/24"
      ],
      "sourcePortRange": "*",
      "sourcePortRanges": []
    }
    ```

6.  **Configure NSG-DB (Deny All Other Inbound):**

    ```bash
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

    **Expected Output:**
    ```json
    {
      "access": "Deny",
      "description": null,
      "destinationAddressPrefix": "*",
      "destinationAddressPrefixes": [],
      "destinationPortRange": null,
      "destinationPortRanges": [
        "*"
      ],
      "direction": "Inbound",
      "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_03_rg/providers/Microsoft.Network/networkSecurityGroups/iatd_labs_03_nsg_db/securityRules/DenyAllOtherInbound",
      "name": "DenyAllOtherInbound",
      "priority": 4096,
      "protocol": "*",
      "provisioningState": "Succeeded",
      "sourceAddressPrefix": "*",
      "sourceAddressPrefixes": [],
      "sourcePortRange": "*",
      "sourcePortRanges": []
    }
    ```

### Part 3: Creating Virtual Machines in Subnets

1.  **Create App Virtual Machine (`iatd_labs_03_vm_app_01`) (Portal):**
    *   Search for "Virtual machines" and select.
    *   Click **Create** -> **Azure virtual machine**.
        *   **Basics Tab:**
            *   Subscription: Your subscription.
            *   Resource group: `iatd_labs_03_rg`
            *   Virtual machine name: `iatd_labs_03_vm_app_01`
            *   Region: Your region.
            *   Image: `Ubuntu Server 22.04 LTS`
            *   Size: Choose a size (e.g., Standard_B1ls).
            *   Username: `azureuser`
            *   Authentication type: `Password`
            *   Password: Set a password.
        *   **Networking Tab:**
            *   Virtual network: `iatd_labs_03_vnet`
            *   Subnet: `iatd_labs_03_subnet_app`
            *   Public IP: **None** (Important: No Public IP)
            *   NIC network security group: `None`
        *   **Management Tab:**
            *   Disable boot diagnostics
    *   Click **Review + create**, then **Create**.

2.  **Create DB Virtual Machine (`iatd_labs_03_vm_db_01`) (Portal):**
    *   Repeat the process above, but with the following changes:
        *   Virtual machine name: `iatd_labs_03_vm_db_01`
        *   Subnet: `iatd_labs_03_subnet_db`
        *   Public IP: **None** (Important: No Public IP)
        *   NIC network security group: `None`
    *   **Management Tab:**
        *   Disable boot diagnostics

### Part 4: Testing Network Security Group Rules

1.  **Connect to App VM:**
    *   You'll need a "jump box" VM with a public IP in the same VNet or use Azure Bastion to connect to the App VM.
    *   SSH into the jump box VM.
    *   From the jump box, SSH into the `iatd_labs_03_vm_app_01` VM using its private IP address.

2.  **Test Connectivity from App VM to DB VM:**
    *   From the `iatd_labs_03_vm_app_01` VM, ping the private IP address of the `iatd_labs_03_vm_db_01` VM.

        ```bash
        ping <private_ip_of_db_vm>
        ```

    *   The ping *should* be successful.

3.  **Test Connectivity from Internet to DB VM:**
    *   Attempt to ping the DB VM from your local machine or any other machine outside the VNet (This will not work without a jumpbox)
    *   Even if the DB VM had a Public IP, the ping *should* fail because the NSG-DB blocks all inbound traffic from the internet.

4.  **(Optional) Test Specific Port Connectivity:**

    *   Install `netcat` on both VMs: `sudo apt install netcat -y`
    *   On the DB VM, listen on a port (e.g., 5432): `nc -l -p 5432`
    *   On the App VM, connect to the DB VM on that port: `nc <private_ip_of_db_vm> 5432`
    *   Type some text on either VM and see if it appears on the other.

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
    Resource group 'iatd_labs_03_rg' could not be found.
    ```

3.  **Clean Local Environment Variables:**

    ```bash
    unset RESOURCE_GROUP LOCATION VNET_NAME SUBNET_APP_NAME SUBNET_DB_NAME ADDRESS_PREFIX SUBNET_APP_PREFIX SUBNET_DB_PREFIX NSG_APP_NAME NSG_DB_NAME
    ```

**Learning Outcomes:**

*   Successfully created an Azure Virtual Network with two subnets.
*   Successfully created and associated Network Security Groups (NSGs) with the subnets.
*   Successfully configured NSG rules to allow outbound traffic from the application subnet to the database subnet.
*   Successfully configured NSG rules to allow inbound traffic to the database subnet only from the application subnet.
*   Successfully verified that traffic is allowed between the application and database subnets.
*   Successfully verified that inbound traffic to the database subnet is blocked from all other sources.
*   Understand how NSGs can be used to secure communication between different subnets within a VNet.
