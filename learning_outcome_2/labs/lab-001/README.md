## IATD Microcredential Cloud Networking: Lab 1 - Implementing User-Defined Routes for Traffic Inspection

**Objective:** Configure a User-Defined Route (UDR) to route traffic from a subnet to a Network Virtual Appliance (NVA) for inspection.

**Scenario:** You have a subnet in Azure that hosts virtual machines processing sensitive data. All traffic from this subnet must be inspected by an NVA before being allowed to reach the internet. This lab demonstrates how to create and implement a UDR to ensure all outbound traffic from the subnet is routed through a virtual appliance acting as a firewall.

**Outcomes:**

*   Creation of a route table.
*   Configuration of a UDR within the route table that directs traffic to the NVA.
*   Association of the route table with the subnet.
*   Verification that traffic from the subnet passes through the NVA.
*   Understanding how UDRs override default routing behaviour.

**Estimated Time:** 60 - 90 minutes

**Prerequisites:**

1.  **Azure Subscription:** Requires an active Azure subscription. A free account is available at [https://azure.microsoft.com/free/](https://azure.microsoft.com/free/).
2.  **Basic Networking Knowledge:** Familiarity with VNETS, subnets, and IP addressing.

**Lab Conventions**

*   **Azure Cloud Shell:** All PowerShell and CLI interactions will occur within the Azure Cloud Shell environment.
*   **Naming Conventions:** Resource names follow the convention `iatd_labs_01_*`.
*   **IP Address Range:** The `172.16.x.x` range will be used.
*   **Location:** Choose a consistent Azure region (e.g., `australiaeast`).

#### Resource Naming

*   **Resource Group:** `iatd_labs_01_rg`
*   **Virtual Network:** `iatd_labs_01_vnet`
*   **Protected Subnet:** `protected-subnet`
*   **NVA Subnet:** `nva-subnet`
*   **NVA VM:** `iatd_labs_01_nva`
*   **NVA NIC:** `iatd_labs_01_nva_nic`
*   **NVA Public IP:** `iatd_labs_01_nva_pip`
*   **Route Table:** `iatd_labs_01_route_table`
*   **Route:** `default-route`
*   **Protected VM:** `iatd_labs_01_vm`
*   **Protected VM NIC:** `iatd_labs_01_vm_nic`
*   **Protected VM Public IP:** `iatd_labs_01_vm_pip`

**Part 1: Setting up the Environment**

1.  **Open Cloud Shell:** In the Azure Portal, click the Cloud Shell icon. Select **Bash** if prompted.
2.  **Create a Resource Group:**

    ```bash
    RESOURCE_GROUP="iatd_labs_01_rg"
    LOCATION="australiaeast"  # Replace with your desired region

    az group create --name $RESOURCE_GROUP --location $LOCATION
    ```

    **Example Output:**

    ```json
    {
      "id": "/subscriptions/<your_subscription_id>/resourceGroups/iatd_labs_01_rg",
      "location": "australiaeast",
      "managedBy": null,
      "name": "iatd_labs_01_rg",
      "properties": {
        "provisioningState": "Succeeded"
      },
      "tags": null,
      "type": "Microsoft.Resources/resourceGroups"
    }
    ```

**Part 2: Create Virtual Network and Subnets**

1.  **Define Variables:**

    ```bash
    VNET_NAME="iatd_labs_01_vnet"
    ADDRESS_PREFIX="172.16.0.0/16"
    PROTECTED_SUBNET_NAME="protected-subnet"
    PROTECTED_SUBNET_PREFIX="172.16.1.0/24"
    NVA_SUBNET_NAME="nva-subnet"
    NVA_SUBNET_PREFIX="172.16.2.0/24"
    ```

2.  **Create the Virtual Network and Subnets:**

    ```bash
    az network vnet create \
      --resource-group $RESOURCE_GROUP \
      --name $VNET_NAME \
      --address-prefixes $ADDRESS_PREFIX \
      --subnet-name $PROTECTED_SUBNET_NAME \
      --subnet-prefixes $PROTECTED_SUBNET_PREFIX

    az network vnet subnet create \
      --resource-group $RESOURCE_GROUP \
      --vnet-name $VNET_NAME \
      --name $NVA_SUBNET_NAME \
      --address-prefixes $NVA_SUBNET_PREFIX
    ```

    **Example Output (`az network vnet create`):**

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
        "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_01_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_01_vnet",
        "location": "australiaeast",
        "name": "iatd_labs_01_vnet",
        "newSubnets": [
          {
            "addressPrefix": "172.16.1.0/24",
            "delegations": [],
            "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_01_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_01_vnet/subnets/protected-subnet",
            "name": "protected-subnet",
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
        ],
        "provisioningState": "Succeeded",
        "resourceGuid": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "subnets": [
          {
            "addressPrefix": "172.16.1.0/24",
            "delegations": [],
            "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_01_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_01_vnet/subnets/protected-subnet",
            "name": "protected-subnet",
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
        ],
        "tags": null,
        "type": "Microsoft.Network/virtualNetworks"
      }
    }
    ```

    **Example Output (`az network vnet subnet create`):**

    ```json
    {
      "addressPrefix": "172.16.2.0/24",
      "delegations": [],
      "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_01_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_01_vnet/subnets/nva-subnet",
      "name": "nva-subnet",
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

**Part 3: Create Network Virtual Appliance (NVA) VM**

1.  **Define Variables:**

    ```bash
    NVA_NAME="iatd_labs_01_nva"
    NIC_NAME="iatd_labs_01_nva_nic"
    ```

2.  **Create a Public IP Address for the NVA:**

    ```bash
    az network public-ip create --resource-group $RESOURCE_GROUP --name iatd_labs_01_nva_pip --allocation-method Static --sku Standard
    ```

    **Example Output:**

    ```json
    {
      "allocationMethod": "Static",
      "deleteOption": "Detach",
      "dnsSettings": null,
      "etag": "\"00000000-0000-0000-0000-000000000000\"",
      "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_01_rg/providers/Microsoft.Network/publicIPAddresses/iatd_labs_01_nva_pip",
      "idleTimeoutInMinutes": 4,
      "ipAddress": "20.188.229.75",
      "ipConfiguration": null,
      "ipTags": [],
      "location": "australiaeast",
      "migrationPhase": "None",
      "name": "iatd_labs_01_nva_pip",
      "natGateway": null,
      "publicIPAddressVersion": "IPv4",
      "publicIPAllocationMethod": "Static",
      "publicIPPrefix": null,
      "resourceGuid": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "sku": {
        "name": "Standard",
        "tier": "Regional"
      },
      "tags": null,
      "type": "Microsoft.Network/publicIPAddresses",
      "zones": []
    }
    ```

3.  **Get the ID of the new public IP:**

    ```bash
    PUBLIC_IP_ID=$(az network public-ip show --resource-group $RESOURCE_GROUP --name iatd_labs_01_nva_pip --query id --output tsv)
    ```

4.  **Create a Network Interface for the NVA:**

    ```bash
    NVA_SUBNET_ID=$(az network vnet subnet show --resource-group $RESOURCE_GROUP --vnet-name $VNET_NAME --name $NVA_SUBNET_NAME --query id --output tsv)

    az network nic create --resource-group $RESOURCE_GROUP --name $NIC_NAME --subnet $NVA_SUBNET_ID  --public-ip-address $PUBLIC_IP_ID
    ```

    **Example Output:**

    ```json
    {
      "auxiliaryMode": "None",
      "auxiliarySku": "None",
      "disableTcpStateTracking": false,
      "dnsSettings": {
        "appliedDnsServers": [],
        "dnsServers": [],
        "internalDnsNameLabel": null,
        "internalDomainNameSuffix": "xxxxxxxxxxxxxxxxxxxx.ax.internal.cloudapp.net",
        "internalFqdn": null
      },
      "enableAcceleratedNetworking": false,
      "enableIpForwarding": false,
      "etag": "\"00000000-0000-0000-0000-000000000000\"",
      "hostedWorkloads": [],
      "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_01_rg/providers/Microsoft.Network/networkInterfaces/iatd_labs_01_nva_nic",
      "ipConfigurations": [
        {
          "applicationGatewayBackendAddressPools": [],
          "applicationSecurityGroups": [],
          "etag": "\"00000000-0000-0000-0000-000000000000\"",
          "gatewayLoadBalancerTunnelInterface": null,
          "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_01_rg/providers/Microsoft.Network/networkInterfaces/iatd_labs_01_nva_nic/ipConfigurations/ipconfig1",
          "name": "ipconfig1",
          "privateIpAddress": "172.16.2.4",
          "privateIpAddressVersion": "IPv4",
          "privateIpAllocationMethod": "Dynamic",
          "provisioningState": "Succeeded",
          "publicIpAddress": {
            "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_01_rg/providers/Microsoft.Network/publicIPAddresses/iatd_labs_01_nva_pip",
            "resourceGuid": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
          },
          "resourceGuid": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
          "subnet": {
            "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_01_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_01_vnet/subnets/nva-subnet"
          },
          "virtualNetworkTaps": []
        }
      ],
      "location": "australiaeast",
      "macAddress": "00-0D-3A-AA-BB-CC",
      "migrationPhase": "None",
      "name": "iatd_labs_01_nva_nic",
      "networkSecurityGroup": null,
      "privateEndpoint": null,
      "provisioningState": "Succeeded",
      "resourceGuid": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "tapConfigurations": [],
      "tags": null,
      "type": "Microsoft.Network/networkInterfaces",
      "virtualMachine": null
    }
    ```

5.  **Create the NVA VM:**

    ```bash
    az vm create \
      --resource-group $RESOURCE_GROUP \
      --name $NVA_NAME \
      --nics $NIC_NAME \
      --image UbuntuLTS \
      --size Standard_B1ls \
      --admin-username azureuser \
      --generate-ssh-keys
    ```

    **Expected Output:**
    ```json
    {
      "fqdn": "",
      "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_01_rg/providers/Microsoft.Compute/virtualMachines/iatd_labs_01_nva",
      "location": "australiaeast",
      "macAddress": "00-0D-3A-AA-BB-CC",
      "powerState": "VM running",
      "privateIpAddress": "172.16.2.4",
      "publicIpAddress": "20.188.229.75",
      "resourceGroup": "iatd_labs_01_rg",
      "zones": ""
    }
    ```

**Part 4: Enable IP Forwarding on the NVA**

1.  **Enable IP Forwarding on the NIC:**

    ```bash
    az network nic update \
      --resource-group $RESOURCE_GROUP \
      --name $NIC_NAME \
      --enable-ip-forwarding true
    ```

    **Example Output:**

    ```json
    {
      "auxiliaryMode": "None",
      "auxiliarySku": "None",
      "disableTcpStateTracking": false,
      "dnsSettings": {
        "appliedDnsServers": [],
        "dnsServers": [],
        "internalDnsNameLabel": null,
        "internalDomainNameSuffix": "xxxxxxxxxxxxxxxxxxxx.ax.internal.cloudapp.net",
        "internalFqdn": null
      },
      "enableAcceleratedNetworking": false,
      "enableIpForwarding": true,
      "etag": "\"00000000-0000-0000-0000-000000000000\"",
      "hostedWorkloads": [],
      "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_01_rg/providers/Microsoft.Network/networkInterfaces/iatd_labs_01_nva_nic",
      "ipConfigurations": [
        {
          "applicationGatewayBackendAddressPools": [],
          "applicationSecurityGroups": [],
          "etag": "\"00000000-0000-0000-0000-000000000000\"",
          "gatewayLoadBalancerTunnelInterface": null,
          "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_01_rg/providers/Microsoft.Network/networkInterfaces/iatd_labs_01_nva_nic/ipConfigurations/ipconfig1",
          "name": "ipconfig1",
          "privateIpAddress": "172.16.2.4",
          "privateIpAddressVersion": "IPv4",
          "privateIpAllocationMethod": "Dynamic",
          "provisioningState": "Succeeded",
          "publicIpAddress": {
            "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_01_rg/providers/Microsoft.Network/publicIPAddresses/iatd_labs_01_nva_pip",
            "resourceGuid": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
          },
          "resourceGuid": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
          "subnet": {
            "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_01_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_01_vnet/subnets/nva-subnet"
          },
          "virtualNetworkTaps": []
        }
      ],
      "location": "australiaeast",
      "macAddress": "00-0D-3A-AA-BB-CC",
      "migrationPhase": "None",
      "name": "iatd_labs_01_nva_nic",
      "networkSecurityGroup": null,
      "privateEndpoint": null,
      "provisioningState": "Succeeded",
      "resourceGuid": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "tapConfigurations": [],
      "tags": null,
      "type": "Microsoft.Network/networkInterfaces",
    }
    ```

**Part 5: Create the Route Table and User-Defined Route**

1.  **Define Variables:**

    ```bash
    ROUTE_TABLE_NAME="iatd_labs_01_route_table"
    ROUTE_NAME="default-route"
    ```

2.  **Create the Route Table:**

    ```bash
    az network route-table create \
      --resource-group $RESOURCE_GROUP \
      --name $ROUTE_TABLE_NAME \
      --location $LOCATION
    ```

    **Example Output:**

    ```json
    {
      "disableBgpRoutePropagation": false,
      "etag": "\"00000000-0000-0000-0000-000000000000\"",
      "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_01_rg/providers/Microsoft.Network/routeTables/iatd_labs_01_route_table",
      "location": "australiaeast",
      "name": "iatd_labs_01_route_table",
      "provisioningState": "Succeeded",
      "resourceGuid": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "routes": [],
      "subnets": [],
      "tags": null,
      "type": "Microsoft.Network/routeTables"
    }
    ```

3.  **Get the private IP address of the NVA:**

    ```bash
    NVA_PRIVATE_IP=$(az network nic ip-config list \
      --resource-group $RESOURCE_GROUP \
      --nic-name $NIC_NAME \
      --query "[0].properties.privateIpAddress" \
      --output tsv)
    ```

4.  **Add a Route to the Route Table:**

    ```bash
    az network route-table route create \
      --resource-group $RESOURCE_GROUP \
      --route-table-name $ROUTE_TABLE_NAME \
      --name $ROUTE_NAME \
      --address-prefix "0.0.0.0/0" \
      --next-hop-type VirtualAppliance \
      --next-hop-ip-address $NVA_PRIVATE_IP
    ```

    **Example Output:**

    ```json
    {
      "addressPrefix": "0.0.0.0/0",
      "etag": "\"00000000-0000-0000-0000-000000000000\"",
      "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_01_rg/providers/Microsoft.Network/routeTables/iatd_labs_01_route_table/routes/default-route",
      "name": "default-route",
      "nextHopIpAddress": "172.16.2.4",
      "nextHopType": "VirtualAppliance",
      "provisioningState": "Succeeded"
    }
    ```

**Part 6: Associate the Route Table with the Protected Subnet**

1.  **Associate the route table:**

    ```bash
    az network vnet subnet update \
      --resource-group $RESOURCE_GROUP \
      --vnet-name $VNET_NAME \
      --name $PROTECTED_SUBNET_NAME \
      --route-table $ROUTE_TABLE_NAME
    ```

    **Example Output:**

    ```json
    {
      "addressPrefix": "172.16.1.0/24",
      "delegations": [],
      "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_01_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_01_vnet/subnets/protected-subnet",
      "name": "protected-subnet",
      "networkSecurityGroup": null,
      "privateEndpointNetworkPolicies": "Disabled",
      "privateLinkServiceNetworkPolicies": "Enabled",
      "provisioningState": "Succeeded",
      "resourceNavigationLinks": [],
      "routeTable": {
        "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_01_rg/providers/Microsoft.Network/routeTables/iatd_labs_01_route_table",
        "resourceGroup": "iatd_labs_01_rg"
      },
      "serviceAssociationLinks": [],
      "serviceEndpointPolicies": [],
      "serviceEndpoints": []
    }
    ```

**Part 7: Create a VM in the Protected Subnet (for testing)**

1.  **Define Variables:**

    ```bash
    VM_NAME="iatd_labs_01_vm"
    NIC_NAME_VM="iatd_labs_01_vm_nic"
    ```

2.  **Create a Public IP Address for the VM:**

    ```bash
    az network public-ip create --resource-group $RESOURCE_GROUP --name iatd_labs_01_vm_pip --allocation-method Static --sku Standard
    ```

    **Example Output:**

    ```json
    {
      "allocationMethod": "Static",
      "deleteOption": "Detach",
      "dnsSettings": null,
      "etag": "\"00000000-0000-0000-0000-000000000000\"",
      "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_01_rg/providers/Microsoft.Network/publicIPAddresses/iatd_labs_01_vm_pip",
      "idleTimeoutInMinutes": 4,
      "ipAddress": "20.188.230.84",
      "ipConfiguration": null,
      "ipTags": [],
      "location": "australiaeast",
      "migrationPhase": "None",
      "name": "iatd_labs_01_vm_pip",
      "natGateway": null,
      "publicIPAddressVersion": "IPv4",
      "publicIPAllocationMethod": "Static",
      "publicIPPrefix": null,
      "resourceGuid": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "sku": {
        "name": "Standard",
        "tier": "Regional"
      },
      "tags": null,
      "type": "Microsoft.Network/publicIPAddresses",
      "zones": []
    }
    ```

3.  **Get the ID of the new public IP:**

    ```bash
    PUBLIC_IP_ID_VM=$(az network public-ip show --resource-group $RESOURCE_GROUP --name iatd_labs_01_vm_pip --query id --output tsv)
    ```

4.  **Create a Network Interface for the VM:**

    ```bash
    PROTECTED_SUBNET_ID=$(az network vnet subnet show --resource-group $RESOURCE_GROUP --vnet-name $VNET_NAME --name $PROTECTED_SUBNET_NAME --query id --output tsv)

    az network nic create --resource-group $RESOURCE_GROUP --name $NIC_NAME_VM --subnet $PROTECTED_SUBNET_ID  --public-ip-address $PUBLIC_IP_ID_VM
    ```

    **Example Output:**

    ```json
    {
      "auxiliaryMode": "None",
      "auxiliarySku": "None",
      "disableTcpStateTracking": false,
      "dnsSettings": {
        "appliedDnsServers": [],
        "dnsServers": [],
        "internalDnsNameLabel": null,
        "internalDomainNameSuffix": "xxxxxxxxxxxxxxxxxxxx.ax.internal.cloudapp.net",
        "internalFqdn": null
      },
      "enableAcceleratedNetworking": false,
      "enableIpForwarding": false,
      "etag": "\"00000000-0000-0000-0000-000000000000\"",
      "hostedWorkloads": [],
      "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_01_rg/providers/Microsoft.Network/networkInterfaces/iatd_labs_01_vm_nic",
      "ipConfigurations": [
        {
          "applicationGatewayBackendAddressPools": [],
          "applicationSecurityGroups": [],
          "etag": "\"00000000-0000-0000-0000-000000000000\"",
          "gatewayLoadBalancerTunnelInterface": null,
          "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_01_rg/providers/Microsoft.Network/networkInterfaces/iatd_labs_01_vm_nic/ipConfigurations/ipconfig1",
          "name": "ipconfig1",
          "privateIpAddress": "172.16.1.4",
          "privateIpAddressVersion": "IPv4",
          "privateIpAllocationMethod": "Dynamic",
          "provisioningState": "Succeeded",
          "publicIpAddress": {
            "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_01_rg/providers/Microsoft.Network/publicIPAddresses/iatd_labs_01_vm_pip",
            "resourceGuid": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
          },
          "resourceGuid": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
          "subnet": {
            "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_01_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_01_vnet/subnets/protected-subnet"
          },
          "virtualNetworkTaps": []
        }
      ],
      "location": "australiaeast",
      "macAddress": "00-0D-3A-DD-EE-FF",
      "migrationPhase": "None",
      "name": "iatd_labs_01_vm_nic",
      "networkSecurityGroup": null,
      "privateEndpoint": null,
      "provisioningState": "Succeeded",
      "resourceGuid": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "tapConfigurations": [],
      "tags": null,
      "type": "Microsoft.Network/networkInterfaces",
    }
    ```

5.  **Create the Protected VM:**

    ```bash
    az vm create \
      --resource-group $RESOURCE_GROUP \
      --name $VM_NAME \
      --nics $NIC_NAME_VM \
      --image UbuntuLTS \
      --size Standard_B1ls \
      --admin-username azureuser \
      --generate-ssh-keys
    ```

    **Expected Output:**
    ```json
    {
      "fqdn": "",
      "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_01_rg/providers/Microsoft.Compute/virtualMachines/iatd_labs_01_vm",
      "location": "australiaeast",
      "macAddress": "00-0D-3A-DD-EE-FF",
      "powerState": "VM running",
      "privateIpAddress": "172.16.1.4",
      "publicIpAddress": "20.188.230.84",
      "resourceGroup": "iatd_labs_01_rg",
      "zones": ""
    }
    ```

**Part 8: Verification (Traffic Inspection)**

*This part requires you to simulate traffic and observe that it is indeed being routed through the NVA.*  *In a real-world scenario, you'd configure your NVA (firewall) to log traffic, and you'd check those logs.*

1.  **Connect to the NVA:** Use SSH to connect to the NVA VM using the public IP address. You'll need the username and password you set when creating the VM.

    ```bash
    ssh azureuser@<NVA_Public_IP>
    ```

    *(Replace `<NVA_Public_IP>` with the actual public IP of your NVA VM.)*

2.  **Configure NVA as a router:**

    *   Once connected, enable IP forwarding on the NVA using `sysctl`. Create a temporary file to run commands without losing state

```bash
    sudo su
    echo "net.ipv4.ip_forward=1" > /etc/sysctl.d/forwarding.conf
    sysctl -p /etc/sysctl.d/forwarding.conf
    exit
```

3.  **Connect to the Protected VM:**  Use SSH to connect to the protected VM (iatd\_labs\_01\_vm) using the public IP address.

    ```bash
    ssh azureuser@<Protected_VM_Public_IP>
    ```
    *(Replace `<Protected_VM_Public_IP>` with the actual public IP of your Protected VM.)*

4.  **Test Outbound Connectivity from the Protected VM:**

```bash
   ping 8.8.8.8
```

5.  **Traceroute:** While connected to the Protected VM, run a traceroute to an external address (e.g., 8.8.8.8 – Google's public DNS server).

```bash
   traceroute 8.8.8.8
```

You should see the traffic first hop to the NVA's private IP address (`172.16.2.4`), then to the internet. If the first hop is the NVA, the UDR is working correctly.

**Important Considerations:**

*   **Security:** This lab uses basic Ubuntu VMs. In a production environment, you would harden the NVA with appropriate security configurations (firewall rules, intrusion detection, etc.).
*   **NVA Configuration:** The specific steps to configure your NVA (firewall, etc.) will vary depending on the appliance you are using. Consult the vendor's documentation.
*   **Cost:** Remember to deallocate the VMs when you're finished to avoid unnecessary charges.

**Part 9: Clean up resources**

To avoid unnecessary costs, remove the resources created in this lab.

1.  In Cloud Shell, execute the following command:

```azurecli
   az group delete --name $RESOURCE_GROUP --yes --no-wait
```

**Post-Lab Questions:**

1.  What is the purpose of a User-Defined Route (UDR)?
2.  How does a UDR override the default routing behaviour in Azure?
3.  Explain the difference between the "VirtualAppliance" and "Internet" next hop types in a UDR.
4.  What are some real-world scenarios where you would use UDRs?

**Congratulations!** You have successfully configured a UDR to route traffic through an NVA for inspection. This is a fundamental technique for securing your Azure workloads.
