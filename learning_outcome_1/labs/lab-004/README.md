## IATD Microcredential Cloud Networking: Lab 4 - Setting up a Site-to-Site VPN Between Azure and AWS

**Objective:**

In this lab, you will establish a secure Site-to-Site VPN connection between an Azure Virtual Network (VNet) and an AWS Virtual Private Cloud (VPC). This will enable network traffic to flow securely between your cloud environments.

**Estimated Time:** 45 - 60 minutes

**Prerequisites:**

1.  **Azure Subscription:** You need an active Azure subscription. If you don't have one, you can create a free account at [https://azure.microsoft.com/free/](https://azure.microsoft.com/free/).
2.  **AWS Account:** You need an active AWS account. If you don't have one, you can create a free account at [https://aws.amazon.com/](https://aws.amazon.com/).
3.  **Basic understanding of networking concepts:** Familiarity with IP addressing, subnets, routing, and VPNs is helpful.

**Lab Conventions:**

*   **Naming Conventions:** All resources created in this lab will be prefixed with `iatd_labs_04` to ensure easy identification and cleanup.
*   **IP Address Range:** The `172.16.0.0/16` range will be used (as per standardization), with Azure using 172.16.2.0/24 and AWS using 172.16.1.0/24.
*   **Location:** Choose a consistent Azure region.

**Lab Instructions:**

**Phase 1: AWS Setup**

**Task 1: Create an AWS VPC**

We'll create a VPC with a specified CIDR block.  We'll use the AWS Management Console for this task.

1.  **Log into the AWS Management Console:**  Go to [https://aws.amazon.com/](https://aws.amazon.com/) and sign in to your account.

2.  **Navigate to the VPC Service:** In the AWS Management Console, search for "VPC" and select "VPC" from the results.

3.  **Create VPC:**
    *   In the VPC Dashboard, click "Create VPC".
    *   Under "Name tag", enter `iatd_labs_04_aws_vpc`.
    *   For "IPv4 CIDR block", enter `172.16.1.0/24`.
    *   Leave the other options as default and click "Create VPC".

**Task 2: Create an AWS Subnet**

Create a subnet within the VPC.

1.  **Navigate to Subnets:** In the VPC Dashboard, select "Subnets" in the left-hand navigation pane.

2.  **Create Subnet:**
    *   Click "Create Subnet".
    *   Select the `iatd_labs_04_aws_vpc` you created earlier for "VPC ID".
    *   Under "Subnet name", enter `iatd_labs_04_aws_subnet`.
    *   For "Availability Zone", choose any available zone (e.g., `ap-southeast-2a`).
    *   For "IPv4 CIDR block", enter `172.16.1.0/27`.
    *   Click "Create Subnet".

**Task 3: Create an Internet Gateway (IGW)**

An Internet Gateway allows communication between your VPC and the internet.

1.  **Navigate to Internet Gateways:** In the VPC Dashboard, select "Internet Gateways" in the left-hand navigation pane.

2.  **Create Internet Gateway:**
    *   Click "Create internet gateway".
    *   Under "Name tag", enter `iatd_labs_04_aws_igw`.
    *   Click "Create internet gateway".

3.  **Attach the Internet Gateway to your VPC:**
    *   Select the `iatd_labs_04_aws_igw` you just created.
    *   Click "Actions" and select "Attach to VPC".
    *   Select the `iatd_labs_04_aws_vpc` and click "Attach internet gateway".

**Task 4: Create a Route Table and Add a Route**

A route table directs network traffic within your VPC.  We'll add a route to send all internet-bound traffic to the Internet Gateway.

1.  **Navigate to Route Tables:** In the VPC Dashboard, select "Route Tables" in the left-hand navigation pane.

2.  **Create Route Table:**
    *   Click "Create route table".
    *   Under "Name tag", enter `iatd_labs_04_aws_route_table`.
    *   Select the `iatd_labs_04_aws_vpc` for "VPC".
    *   Click "Create route table".

3.  **Associate the Route Table with your Subnet:**
    *   Select the `iatd_labs_04_aws_route_table` you just created.
    *   Click the "Subnet Associations" tab.
    *   Click "Edit subnet associations".
    *   Select the `iatd_labs_04_aws_subnet` and click "Save associations".

4.  **Add a Route to the Route Table:**
    *   Select the `iatd_labs_04_aws_route_table` you just created.
    *   Click the "Routes" tab.
    *   Click "Edit routes".
    *   Click "Add route".
    *   For "Destination", enter `0.0.0.0/0` (this represents all IP addresses).
    *   For "Target", select "Internet Gateway" and choose the `iatd_labs_04_aws_igw`.
    *   Click "Save changes".

**Task 5: Create a Customer Gateway in AWS**

The Customer Gateway represents your Azure VPN Gateway within AWS.  You'll need the public IP address of the Azure VPN Gateway for this.  *We'll get this IP in Phase 2, Task 2*. For now, use a placeholder IP address; you'll need to edit this later. 

1.  **Navigate to Customer Gateways:** In the VPC Dashboard, select "Customer Gateways" in the left-hand navigation pane.

2.  **Create Customer Gateway:**
    *   Click "Create customer gateway".
    *   Under "Name tag", enter `iatd_labs_04_aws_customer_gateway`.
    *   For "Routing", select "Static".
    *   For "IP address", enter a *temporary placeholder* IP address (e.g., `203.0.113.1`).  *You will update this later.*
    *   For "BGP ASN", enter `65000`.
    *   Click "Create customer gateway".

**Task 6: Create a Virtual Private Gateway (VGW) in AWS**

The VGW is the VPN Gateway on the AWS side.

1.  **Navigate to Virtual Private Gateways:** In the VPC Dashboard, select "Virtual Private Gateways" in the left-hand navigation pane.

2.  **Create Virtual Private Gateway:**
    *   Click "Create virtual private gateway".
    *   Under "Name tag", enter `iatd_labs_04_aws_vpn_gateway`.
    *   For "Amazon side ASN", enter `64512`.
    *   Click "Create virtual private gateway".

3.  **Attach the VGW to your VPC:**
    *   Select the `iatd_labs_04_aws_vpn_gateway` you just created.
    *   Click "Actions" and select "Attach to VPC".
    *   Select the `iatd_labs_04_aws_vpc` and click "Attach virtual private gateway".

**Task 7: Create a VPN Connection in AWS**

This connects your VGW to your Customer Gateway, creating the VPN tunnel.

1.  **Navigate to VPN Connections:** In the VPC Dashboard, select "VPN Connections" in the left-hand navigation pane.

2.  **Create VPN Connection:**
    *   Click "Create VPN connection".
    *   Under "Name tag", enter `iatd_labs_04_aws_vpn_connection`.
    *   For "Target gateway type", select "Virtual Private Gateway" and choose `iatd_labs_04_aws_vpn_gateway`.
    *   For "Customer gateway", select "Existing" and choose `iatd_labs_04_aws_customer_gateway`.
    *   For "Routing Options", select "Static".
    *   For "Static Route Prefix", enter `172.16.2.0/24` (This is the Azure VNet address space).
    *  Under "Tunnel Options", specify the Pre-Shared Key (PSK) for Tunnel 1 and Tunnel 2:
            * **Tunnel 1 Pre-Shared Key:** `IATD_Labs_04_PSK`
            * **Tunnel 2 Pre-Shared Key:** `IATD_Labs_04_PSK`
    *   Click "Create VPN connection".

**Phase 2: Azure Setup**

**Task 1: Create a Resource Group**

A resource group is a container that holds related resources for an Azure solution.  We'll use Azure Cloud Shell and Azure CLI for this task.

1.  **Open Azure Cloud Shell:** Go to the Azure portal ([https://portal.azure.com/](https://portal.azure.com/)) and click the Cloud Shell icon in the top right corner. If prompted, select "Bash" as your preferred shell.

2.  **Create Resource Group:**  Execute the following command in Cloud Shell:

    ```azurecli
    az group create --name iatd_labs_04_rg --location australiaeast
    ```

    **Example Output:**

    ```json
    {
      "id": "/subscriptions/YOUR_SUBSCRIPTION_ID/resourceGroups/iatd_labs_04_rg",
      "location": "australiaeast",
      "managedBy": null,
      "name": "iatd_labs_04_rg",
      "properties": {
        "provisioningState": "Succeeded"
      },
      "tags": null,
      "type": "Microsoft.Resources/resourceGroups"
    }
    ```

**Task 2: Create a Virtual Network (VNet)**

We'll create a VNet with a specified address space and a subnet.

1.  **Create VNet:** Execute the following command in Cloud Shell:

    ```azurecli
    az network vnet create \
      --resource-group iatd_labs_04_rg \
      --name iatd_labs_04_vnet \
      --address-prefixes 172.16.2.0/24 \
      --subnet-name iatd_labs_04_subnet \
      --subnet-prefixes 172.16.2.0/27
    ```

    **Example Output:**

    ```json
    {
      "newVNet": {
        "addressSpace": {
          "addressPrefixes": [
            "172.16.2.0/24"
          ]
        },
        "dhcpOptions": {
          "dnsServers": []
        },
        "enableDdosProtection": false,
        "enableVmProtection": false,
        "id": "/subscriptions/YOUR_SUBSCRIPTION_ID/resourceGroups/iatd_labs_04_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_04_vnet",
        "location": "australiaeast",
        "name": "iatd_labs_04_vnet",
        "provisioningState": "Succeeded",
        "resourceGuid": "YOUR_RESOURCE_GUID",
        "resourceGroupName": "iatd_labs_04_rg",
        "subnets": [
          {
            "addressPrefix": "172.16.2.0/27",
            "delegations": [],
            "id": "/subscriptions/YOUR_SUBSCRIPTION_ID/resourceGroups/iatd_labs_04_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_04_vnet/subnets/iatd_labs_04_subnet",
            "name": "iatd_labs_04_subnet",
            "privateEndpointNetworkPolicies": "Disabled",
            "privateLinkServiceNetworkPolicies": "Enabled",
            "provisioningState": "Succeeded",
            "resourceGuid": "YOUR_RESOURCE_GUID",
            "serviceEndpointPolicies": []
          }
        ],
        "type": "Microsoft.Network/virtualNetworks",
        "virtualNetworkPeerings": []
      }
    }
    ```

**Task 3: Create a Virtual Network Gateway**

This is the VPN gateway on the Azure side.  This process can take 30-45 minutes to complete.

1.  **Create Public IP Address:**  First, create a public IP address for the VPN gateway.

    ```azurecli
    az network public-ip create \
      --resource-group iatd_labs_04_rg \
      --name iatd_labs_04_gateway_pip \
      --allocation-method Static \
      --sku Standard
    ```

    **Example Output:**

    ```json
    {
      "allocationMethod": "Static",
      "deleteOption": "Delete",
      "dnsSettings": null,
      "etag": "W/\"YOUR_ETAG\"",
      "id": "/subscriptions/YOUR_SUBSCRIPTION_ID/resourceGroups/iatd_labs_04_rg/providers/Microsoft.Network/publicIPAddresses/iatd_labs_04_gateway_pip",
      "idleTimeoutInMinutes": 4,
      "ipAddress": "YOUR_PUBLIC_IP_ADDRESS",
      "ipConfiguration": null,
      "ipTags": [],
      "location": "australiaeast",
      "name": "iatd_labs_04_gateway_pip",
      "provisioningState": "Succeeded",
      "publicIPAddressVersion": "IPv4",
      "publicIPAllocationMethod": "Static",
      "resourceGuid": "YOUR_RESOURCE_GUID",
      "resourceGroupName": "iatd_labs_04_rg",
      "sku": {
        "name": "Standard",
        "tier": "Regional"
      },
      "tags": null,
      "type": "Microsoft.Network/publicIPAddresses",
      "zones": []
    }
    ```

    *   **Important:**  Note the `ipAddress` from the output.  You will need this in Task 5.

2.  **Create Virtual Network Gateway:**  Now, create the VPN gateway.  This uses a `VpnGw1` SKU for testing purposes.  For production, consider a higher SKU.

    ```azurecli
    az network vpn-gateway create \
      --resource-group iatd_labs_04_rg \
      --name iatd_labs_04_vnet_gateway \
      --location australiaeast \
      --vnet iatd_labs_04_vnet \
      --gateway-type Vpn \
      --vpn-type RouteBased \
      --sku VpnGw1 \
      --public-ip-address iatd_labs_04_gateway_pip \
      --enable-bgp false
    ```

    **Important Considerations:**

    *   Creating a VPN gateway can take a *significant* amount of time (30-45 minutes). The Azure CLI will show you the process.
    *   `--enable-bgp false`:  We're using static routing in this lab for simplicity.  In a production environment, BGP is generally preferred.

**Task 4: Create a Local Network Gateway**

The Local Network Gateway represents your AWS VGW within Azure.

1.  **Create Local Network Gateway:** Execute the following command in Cloud Shell.  You'll need the CIDR block of your AWS VPC (`172.16.1.0/24`) and the Public IP Address of the AWS VGW. *See AWS Post Lab tasks*.

    ```azurecli
    az network local-gateway create \
      --resource-group iatd_labs_04_rg \
      --name iatd_labs_04_local_gateway \
      --gateway-ip-address <AWS_VGW_PUBLIC_IP> \
      --local-address-prefixes 172.16.1.0/24 \
      --location australiaeast
    ```

    Replace `<AWS_VGW_PUBLIC_IP>` with the actual public IP address of your AWS Virtual Private Gateway.

    **Example:**

    ```azurecli
    az network local-gateway create \
      --resource-group iatd_labs_04_rg \
      --name iatd_labs_04_local_gateway \
      --gateway-ip-address 52.62.23.14 \
      --local-address-prefixes 172.16.1.0/24 \
      --location australiaeast
    ```

    **Example Output:**

    ```json
    {
      "bgpSettings": null,
      "etag": "W/\"YOUR_ETAG\"",
      "gatewayIpAddress": "52.62.23.14",
      "id": "/subscriptions/YOUR_SUBSCRIPTION_ID/resourceGroups/iatd_labs_04_rg/providers/Microsoft.Network/localNetworkGateways/iatd_labs_04_local_gateway",
      "localAddressPrefixes": [
        "172.16.1.0/24"
      ],
      "location": "australiaeast",
      "name": "iatd_labs_04_local_gateway",
      "provisioningState": "Succeeded",
      "resourceGuid": "YOUR_RESOURCE_GUID",
      "resourceGroupName": "iatd_labs_04_rg",
      "tags": null,
      "type": "Microsoft.Network/localNetworkGateways"
    }
    ```

**Task 5: Create the Azure VPN Connection**

This creates the connection between your Azure VNet Gateway and the AWS Local Network Gateway.

1.  **Create VPN Connection:** Execute the following command in Cloud Shell:

    ```azurecli
    az network vpn-connection create \
      --resource-group iatd_labs_04_rg \
      --name iatd_labs_04_connection \
      --location australiaeast \
      --vnet-gateway1 iatd_labs_04_vnet_gateway \
      --local-gateway2 iatd_labs_04_local_gateway \
      --shared-key IATD_Labs_04_PSK
    ```

    **Example Output:**

    ```json
    {
      "connectionStatus": "Unknown",
      "egressBytesTransferred": 0,
      "enableBgp": false,
      "etag": "W/\"YOUR_ETAG\"",
      "id": "/subscriptions/YOUR_SUBSCRIPTION_ID/resourceGroups/iatd_labs_04_rg/providers/Microsoft.Network/connections/iatd_labs_04_connection",
      "ingressBytesTransferred": 0,
      "location": "australiaeast",
      "name": "iatd_labs_04_connection",
      "peer": {
        "id": "/subscriptions/YOUR_SUBSCRIPTION_ID/resourceGroups/iatd_labs_04_rg/providers/Microsoft.Network/localNetworkGateways/iatd_labs_04_local_gateway"
      },
      "provisioningState": "Succeeded",
      "resourceGuid": "YOUR_RESOURCE_GUID",
      "resourceGroupName": "iatd_labs_04_rg",
      "routingWeight": null,
      "sharedKey": "IATD_Labs_04_PSK",
      "tags": null,
      "type": "Microsoft.Network/connections",
      "useLocalAzureIpAddress": false,
      "usePolicyBasedTrafficSelectors": false,
      "vpnConnectionProtocolType": "IKEv2",
      "vpnLinkConnections": null,
      "vpnSiteLinkConnections": null
    }
    ```

**Phase 3: Post-Configuration & Testing**

**Task 1: Update AWS Customer Gateway with Azure VPN Gateway IP**

1.  **Obtain Azure VPN Gateway Public IP:** If you didn't save it earlier, retrieve the public IP address of your Azure VPN Gateway (`iatd_labs_04_gateway_pip`) using the Azure portal or the following Azure CLI command:

    ```azurecli
    az network public-ip show --resource-group iatd_labs_04_rg --name iatd_labs_04_gateway_pip --query ipAddress -o tsv
    ```

2.  **Update AWS Customer Gateway:**
    *   In the AWS Management Console, navigate to "VPC" -> "Customer Gateways".
    *   Select the `iatd_labs_04_aws_customer_gateway`.
    *   Click "Actions" -> "Modify customer gateway".
    *   Enter the Azure VPN Gateway's public IP address you obtained in step 1.
    *   Click "Modify customer gateway".

**Task 2: Obtain AWS VGW Public IP Addresses**

1. In the AWS Management Console, navigate to "VPC" -> "Virtual Private Gateways".
2. Select the `iatd_labs_04_aws_vpn_gateway`.
3. Under the "Details" tab, find the "Tunnel Endpoints" for Tunnel 1 and Tunnel 2. Note down the "Outside IP Address" for both tunnels.
4. The outside IP addresses are in the form of a DNS name; you may need to resolve these DNS names to their corresponding IP addresses.

**Task 3: Testing Connectivity**

1.  **Create Test VMs:**  Create one virtual machine in the `iatd_labs_04_subnet` subnet in Azure and another in the `iatd_labs_04_aws_subnet` subnet in AWS.  Choose a minimal Linux distribution (e.g., Ubuntu Server) for both. *Ensure that you create the VMs with a Public IP*

2.  **Configure Security Groups/Network Security Groups:**
    *   **Azure:**  Allow inbound ICMP (ping) traffic to the Azure VM from the AWS VPC CIDR block (172.16.1.0/24).
    *   **AWS:** Allow inbound ICMP (ping) traffic to the AWS VM from the Azure VNet CIDR block (172.16.2.0/24).

3.  **Ping Test:** From the Azure VM, ping the private IP address of the AWS VM.  From the AWS VM, ping the private IP address of the Azure VM.

    *   If the pings are successful, your Site-to-Site VPN is working correctly!

**Post-Lab Cleanup:**

To avoid incurring unnecessary charges, delete the resources you created in this lab:

**Azure:**

1.  ```azurecli
    az group delete --name iatd_labs_04_rg --yes --no-wait
    ```

**AWS:**

1.  Delete the VPN Connection (`iatd_labs_04_aws_vpn_connection`).
2.  Detach and then delete the Virtual Private Gateway (`iatd_labs_04_aws_vpn_gateway`).
3.  Delete the Customer Gateway (`iatd_labs_04_aws_customer_gateway`).
4.  Detach the Internet Gateway (`iatd_labs_04_aws_igw`) from the VPC and then delete the Internet Gateway.
5.  Delete the Subnet (`iatd_labs_04_aws_subnet`).
6.  Delete the Route Table (`iatd_labs_04_aws_route_table`).
7.  Delete the VPC (`iatd_labs_04_aws_vpc`).

**Troubleshooting:**

*   **VPN Connection Status:** Check the status of the VPN connection in both Azure and AWS.  Look for error messages or disconnected states.
*   **Security Groups/Network Security Groups:**  Ensure that your security rules allow traffic between the Azure VNet and AWS VPC.
*   **Route Tables:** Verify that your route tables are configured correctly to direct traffic through the VPN tunnel.
*   **IKE/IPsec Configuration:** Double-check the IKE/IPsec settings (especially the shared key) on both sides.  Mismatched settings will prevent the VPN from establishing.
*   **Firewall Rules:** If you have firewalls running on your VMs, ensure they are not blocking traffic.

This detailed lab guide provides a comprehensive, hands-on experience setting up a Site-to-Site VPN between Azure and AWS. Remember to replace placeholder values with your actual resource names and IP addresses. Good luck!
