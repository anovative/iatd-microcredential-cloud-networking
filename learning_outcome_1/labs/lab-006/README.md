## IATD Microcredential Cloud Networking: Lab 6 - Creating Virtual Machines with Public/Private IPs and NAT Gateway Configuration

**Objective:** In this lab, you will learn how to create Azure Virtual Machines (VMs) with either a public IP address or only a private IP address. You will then configure a NAT Gateway to provide outbound internet access for VMs with only private IPs.

**Estimated Time:** 45 - 60 minutes

**Prerequisites:**

1.  **Azure Subscription:** You need an active Azure subscription.
2.  **Azure Cloud Shell:** This lab utilizes the Azure Cloud Shell, so ensure you have access to it through the Azure portal.
3.  **Basic Understanding of Networking:** A general understanding of IP addressing, subnets, and NAT is helpful.

**Lab Conventions:**

*   **Naming Conventions:** All resources created in this lab will be prefixed with `iatd_labs_06` to ensure easy identification and cleanup.
*   **IP Address Range:** The `172.16.0.0/16` range will be used (as per standardization).
*   **Location:** Choose a consistent Azure region.

**Let's get started!**

### Part 1: Create a Resource Group

We'll use the Azure CLI in Cloud Shell for most of this lab.

**Method: Azure CLI**

1.  **Open Azure Cloud Shell:** Go to the Azure portal and launch Cloud Shell (Bash).

2.  **Create a Resource Group:**

    ```azurecli
    az group create --name iatd_labs_06_rg --location australiaeast
    ```

    **Example Output:**

    ```json
    {
      "id": "/subscriptions/YOUR_SUBSCRIPTION_ID/resourceGroups/iatd_labs_06_rg",
      "location": "australiaeast",
      "managedBy": null,
      "name": "iatd_labs_06_rg",
      "properties": {
        "provisioningState": "Succeeded"
      },
      "tags": null,
      "type": "Microsoft.Resources/resourceGroups"
    }
    ```

### Part 2: Create a Virtual Network and Subnets

We'll create a VNet with two subnets: one for VMs with public IPs and another for VMs with only private IPs.

**Method: Azure CLI**

1.  **Create VNet:**

    ```azurecli
    az network vnet create \
      --resource-group iatd_labs_06_rg \
      --name iatd_labs_06_vnet \
      --address-prefixes 172.16.0.0/16 \
      --location australiaeast
    ```

    **Example Output:**

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
        "id": "/subscriptions/YOUR_SUBSCRIPTION_ID/resourceGroups/iatd_labs_06_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_06_vnet",
        "location": "australiaeast",
        "name": "iatd_labs_06_vnet",
        "provisioningState": "Succeeded",
        "resourceGuid": "YOUR_RESOURCE_GUID",
        "resourceGroupName": "iatd_labs_06_rg",
        "subnets": [],
        "type": "Microsoft.Network/virtualNetworks",
        "virtualNetworkPeerings": []
      }
    }
    ```

2.  **Create Subnet for Public IP VMs:**

    ```azurecli
    az network vnet subnet create \
      --resource-group iatd_labs_06_rg \
      --vnet-name iatd_labs_06_vnet \
      --name iatd_labs_06_subnet_public \
      --address-prefixes 172.16.1.0/24
    ```

    **Example Output:**

    ```json
    {
      "addressPrefix": "172.16.1.0/24",
      "delegations": [],
      "id": "/subscriptions/YOUR_SUBSCRIPTION_ID/resourceGroups/iatd_labs_06_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_06_vnet/subnets/iatd_labs_06_subnet_public",
      "name": "iatd_labs_06_subnet_public",
      "privateEndpointNetworkPolicies": "Disabled",
      "privateLinkServiceNetworkPolicies": "Enabled",
      "provisioningState": "Succeeded",
      "resourceGuid": "YOUR_RESOURCE_GUID",
      "serviceEndpointPolicies": []
    }
    ```

3.  **Create Subnet for Private IP VMs:**

    ```azurecli
    az network vnet subnet create \
      --resource-group iatd_labs_06_rg \
      --vnet-name iatd_labs_06_vnet \
      --name iatd_labs_06_subnet_private \
      --address-prefixes 172.16.2.0/24
    ```

    **Example Output:**

    ```json
    {
      "addressPrefix": "172.16.2.0/24",
      "delegations": [],
      "id": "/subscriptions/YOUR_SUBSCRIPTION_ID/resourceGroups/iatd_labs_06_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_06_vnet/subnets/iatd_labs_06_subnet_private",
      "name": "iatd_labs_06_subnet_private",
      "privateEndpointNetworkPolicies": "Disabled",
      "privateLinkServiceNetworkPolicies": "Enabled",
      "provisioningState": "Succeeded",
      "resourceGuid": "YOUR_RESOURCE_GUID",
      "serviceEndpointPolicies": []
    }
    ```

### Part 3: Create a Virtual Machine with a Public IP Address

**Method: Azure CLI**

1.  **Create Public IP:** Create a Public IP Address that will be associated with the VM

    ```azurecli
    az network public-ip create \
      --resource-group iatd_labs_06_rg \
      --name iatd_labs_06_public_ip \
      --allocation-method Static \
      --sku Standard \
      --location australiaeast
    ```

    **Example Output:**

    ```json
    {
      "allocationMethod": "Static",
      "deleteOption": "Delete",
      "dnsSettings": null,
      "etag": "W/\"YOUR_ETAG\"",
      "id": "/subscriptions/YOUR_SUBSCRIPTION_ID/resourceGroups/iatd_labs_06_rg/providers/Microsoft.Network/publicIPAddresses/iatd_labs_06_public_ip",
      "idleTimeoutInMinutes": 4,
      "ipAddress": "YOUR_PUBLIC_IP_ADDRESS",
      "ipConfiguration": null,
      "ipTags": [],
      "location": "australiaeast",
      "name": "iatd_labs_06_public_ip",
      "provisioningState": "Succeeded",
      "publicIPAddressVersion": "IPv4",
      "publicIPAllocationMethod": "Static",
      "resourceGuid": "YOUR_RESOURCE_GUID",
      "resourceGroupName": "iatd_labs_06_rg",
      "sku": {
        "name": "Standard",
        "tier": "Regional"
      },
      "tags": null,
      "type": "Microsoft.Network/publicIPAddresses",
      "zones": []
    }
    ```

2.  **Create VM with Public IP:**

    ```azurecli
    az vm create \
      --resource-group iatd_labs_06_rg \
      --name iatd_labs_06_vm_public \
      --image UbuntuLTS \
      --admin-username azureuser \
      --generate-ssh-keys \
      --vnet-name iatd_labs_06_vnet \
      --subnet iatd_labs_06_subnet_public \
      --public-ip-address iatd_labs_06_public_ip
    ```

    **Example Output:**

    ```json
    {
      "fqdns": "",
      "id": "/subscriptions/YOUR_SUBSCRIPTION_ID/resourceGroups/iatd_labs_06_rg/providers/Microsoft.Compute/virtualMachines/iatd_labs_06_vm_public",
      "location": "australiaeast",
      "macAddress": "YOUR_MAC_ADDRESS",
      "powerState": "VM running",
      "privateIpAddress": "172.16.1.4",
      "publicIpAddress": "YOUR_PUBLIC_IP_ADDRESS",
      "resourceGroup": "iatd_labs_06_rg",
      "zones": null
    }
    ```

### Part 4: Create a Virtual Machine with Only a Private IP Address

**Method: Azure CLI**

1.  **Create VM with Private IP Only:** We do *not* specify `--public-ip-address`.

    ```azurecli
    az vm create \
      --resource-group iatd_labs_06_rg \
      --name iatd_labs_06_vm_private \
      --image UbuntuLTS \
      --admin-username azureuser \
      --generate-ssh-keys \
      --vnet-name iatd_labs_06_vnet \
      --subnet iatd_labs_06_subnet_private
    ```

    **Example Output:**

    ```json
    {
      "fqdns": "",
      "id": "/subscriptions/YOUR_SUBSCRIPTION_ID/resourceGroups/iatd_labs_06_rg/providers/Microsoft.Compute/virtualMachines/iatd_labs_06_vm_private",
      "location": "australiaeast",
      "macAddress": "YOUR_MAC_ADDRESS",
      "powerState": "VM running",
      "privateIpAddress": "172.16.2.4",
      "publicIpAddress": null,
      "resourceGroup": "iatd_labs_06_rg",
      "zones": null
    }
    ```

    **Important:** Notice that `publicIpAddress` is `null` in the output.  This confirms the VM has no public IP and cannot directly access the internet.

### Part 5: Create a NAT Gateway and Associate It with the Private Subnet

Now we'll create a NAT Gateway to allow the private VM to access the internet.

**Method: Azure CLI**

1.  **Create Public IP for NAT Gateway:** The NAT Gateway needs a public IP for outbound connections.

    ```azurecli
    az network public-ip create \
      --resource-group iatd_labs_06_rg \
      --name iatd_labs_06_nat_public_ip \
      --allocation-method Static \
      --sku Standard \
      --location australiaeast
    ```

     **Example Output:**

    ```json
    {
      "allocationMethod": "Static",
      "deleteOption": "Delete",
      "dnsSettings": null,
      "etag": "W/\"YOUR_ETAG\"",
      "id": "/subscriptions/YOUR_SUBSCRIPTION_ID/resourceGroups/iatd_labs_06_rg/providers/Microsoft.Network/publicIPAddresses/iatd_labs_06_nat_public_ip",
      "idleTimeoutInMinutes": 4,
      "ipAddress": "YOUR_NAT_PUBLIC_IP",
      "ipConfiguration": null,
      "ipTags": [],
      "location": "australiaeast",
      "name": "iatd_labs_06_nat_public_ip",
      "provisioningState": "Succeeded",
      "publicIPAddressVersion": "IPv4",
      "publicIPAllocationMethod": "Static",
      "resourceGuid": "YOUR_RESOURCE_GUID",
      "resourceGroupName": "iatd_labs_06_rg",
      "sku": {
        "name": "Standard",
        "tier": "Regional"
      },
      "tags": null,
      "type": "Microsoft.Network/publicIPAddresses",
      "zones": []
    }
    ```

2.  **Create NAT Gateway:**

    ```azurecli
    az network nat gateway create \
      --resource-group iatd_labs_06_rg \
      --name iatd_labs_06_nat_gateway \
      --location australiaeast \
      --public-ip-addresses iatd_labs_06_nat_public_ip
    ```

    **Example Output:**

    ```json
    {
      "etag": "W/\"YOUR_ETAG\"",
      "id": "/subscriptions/YOUR_SUBSCRIPTION_ID/resourceGroups/iatd_labs_06_rg/providers/Microsoft.Network/natGateways/iatd_labs_06_nat_gateway",
      "idleTimeoutInMinutes": 4,
      "location": "australiaeast",
      "name": "iatd_labs_06_nat_gateway",
      "provisioningState": "Succeeded",
      "publicIpAddresses": [
        {
          "id": "/subscriptions/YOUR_SUBSCRIPTION_ID/resourceGroups/iatd_labs_06_rg/providers/Microsoft.Network/publicIPAddresses/iatd_labs_06_nat_public_ip"
        }
      ],
      "resourceGuid": "YOUR_RESOURCE_GUID",
      "resourceGroupName": "iatd_labs_06_rg",
      "sku": {
        "name": "Standard"
      },
      "subnets": [],
      "tags": null,
      "type": "Microsoft.Network/natGateways",
      "zones": []
    }
    ```

3.  **Associate NAT Gateway with the Private Subnet:**  This step connects the NAT Gateway to the subnet where your private VM resides.

    ```azurecli
    az network vnet subnet update \
      --resource-group iatd_labs_06_rg \
      --vnet-name iatd_labs_06_vnet \
      --name iatd_labs_06_subnet_private \
      --nat-gateway iatd_labs_06_nat_gateway
    ```

    **Example Output:**

    ```json
    {
      "addressPrefix": "172.16.2.0/24",
      "delegations": [],
      "id": "/subscriptions/YOUR_SUBSCRIPTION_ID/resourceGroups/iatd_labs_06_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_06_vnet/subnets/iatd_labs_06_subnet_private",
      "name": "iatd_labs_06_subnet_private",
      "natGateway": {
        "id": "/subscriptions/YOUR_SUBSCRIPTION_ID/resourceGroups/iatd_labs_06_rg/providers/Microsoft.Network/natGateways/iatd_labs_06_nat_gateway"
      },
      "privateEndpointNetworkPolicies": "Disabled",
      "privateLinkServiceNetworkPolicies": "Enabled",
      "provisioningState": "Succeeded",
      "resourceGuid": "YOUR_RESOURCE_GUID",
      "serviceEndpointPolicies": []
    }
    ```

### Part 6: Test Internet Connectivity

1.  **Connect to the Public VM:** Use SSH to connect to `iatd_labs_06_vm_public` using its public IP address. You can find the public IP in the Azure portal.

2.  **Test Internet Access from Public VM:** From the SSH session, try pinging a public website (e.g., `ping google.com`).  It should work.

3.  **Connect to the Private VM (Indirectly):** You cannot directly SSH into `iatd_labs_06_vm_private` because it has no public IP. You need to connect to it *through* the public VM.

    *   **SSH Tunneling (Example):** From your *local* machine, use SSH tunneling to forward a port on your local machine to the private VM.
        ```bash
        ssh -L 8080:<private_vm_private_ip>:22 azureuser@<public_vm_public_ip>
        ```
        Replace `<private_vm_private_ip>` with the *private* IP address of `iatd_labs_06_vm_private` and `<public_vm_public_ip>` with the *public* IP of `iatd_labs_06_vm_public`. You can find both IPs in the Azure portal.
        * from within that first ssh connection to the public IP, you need to then connect to the private VM `ssh azureuser@<private_vm_private_ip>`
    * now you are in a terminal for the private VM.
4.  **Test Internet Access from Private VM:** From the SSH session to the private VM, try pinging a public website (e.g., `ping google.com`). Thanks to the NAT Gateway, it *should* now work!

### Post-Lab: Cleanup

To avoid incurring unnecessary costs, it's essential to clean up the resources created during this lab.

1.  **Delete Resource Group:**

    ```azurecli
    az group delete --name iatd_labs_06_rg --yes --no-wait
    ```
    
    **Example Output:**
    
    ```
    (No output due to --no-wait parameter, but the deletion process will start in the background)
    ```

### Verification

*   Confirm you can SSH into the public VM directly.
*   Confirm you can SSH into the private VM *indirectly* through the public VM, using SSH tunneling.
*   Confirm both VMs can access the internet.
*   Verify that the outbound traffic from the private VM is using the Public IP address from the NAT gateway (Check by going to the public IP page for the NAT Gateway. In the charts view, select Bytes. Ensure you select the proper time range when looking for activity, it defaults to 24 hours, but if the connection was made less than 24 hours ago you will not see the Bytes traverse through the NAT.)

**Congratulations!** This lab covers creating VMs with different IP configurations and setting up a NAT Gateway. This configuration is common when you want to minimize the number of VMs with direct public internet access for security reasons. Make sure you clean up afterwards!