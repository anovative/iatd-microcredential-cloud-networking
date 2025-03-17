## IATD Microcredential Cloud Networking: Lab 8 - Optimizing Traffic with a Network Virtual Appliance

**Objective:** This lab focuses on deploying a Network Virtual Appliance (NVA) and configuring traffic routing to optimize traffic between two subnets. We'll simulate a scenario where an NVA compresses traffic, improving performance. Note, for the purpose of this lab we won't be using an NVA with real compression functionality, as deploying and configuring a specific NVA is beyond the scope. We'll focus on the routing aspect.

**Estimated Time:** 60 - 90 minutes

**Prerequisites:**

1.  **Azure Subscription:** An active Azure subscription.
2.  **Azure Cloud Shell:** Access to Azure Cloud Shell (Bash or PowerShell).
3.  **Basic Understanding:** Familiarity with Virtual Networks, subnets, User Defined Routes (UDRs), and Network Security Groups (NSGs).
4.  **Region Selection:** Choose a region.

**Let's get started!**

### Lab Conventions

*   **Azure Cloud Shell:** All PowerShell and CLI interactions will occur within the Azure Cloud Shell environment.
*   **Naming Conventions:** Resource names follow the convention `iatd_labs_08_*`.
*   **IP Address Range:** The `172.16.x.x` range will be used.
*   **Location:** Choose a consistent Azure region (e.g., `australiaeast`).

#### Resource Naming

*   **Resource Group:** `iatd_labs_08_rg`
*   **Virtual Network:** `iatd_labs_08_vnet`
*   **Subnet A (Source):** `iatd_labs_08_subnet_a`
*   **Subnet B (Destination):** `iatd_labs_08_subnet_b`
*   **NVA Subnet:** `iatd_labs_08_nva_subnet`
*   **NVA:** `iatd_labs_08_nva`
*   **NVA Public IP:** `iatd_labs_08_nva_pip`
*   **VM A (in Subnet A):** `iatd_labs_08_vm_a`
*   **VM B (in Subnet B):** `iatd_labs_08_vm_b`
*   **VM A Public IP:** `iatd_labs_08_vm_a_pip`
*   **VM B Public IP:** `iatd_labs_08_vm_b_pip`
*   **Route Table:** `iatd_labs_08_rt`
*   **NSG for VM A:** `iatd_labs_08_nsg_vm_a`
*   **NSG for VM B:** `iatd_labs_08_nsg_vm_b`
*   **NSG for NVA:** `iatd_labs_08_nsg_nva`

### Part 1: Creating the Virtual Network, Subnets, and VMs

1.  **Open Cloud Shell:** Open Azure Cloud Shell and select **PowerShell**.

2.  **Define Variables:**

    ```powershell
    $resourceGroupName = "iatd_labs_08_rg"
    $location = "australiaeast" # Replace with your preferred region

    # VNet and Subnet Variables
    $vnetName = "iatd_labs_08_vnet"
    $vnetAddressPrefix = "172.16.0.0/16"
    $subnetAName = "iatd_labs_08_subnet_a"
    $subnetAPrefix = "172.16.0.0/24"
    $subnetBName = "iatd_labs_08_subnet_b"
    $subnetBPrefix = "172.16.1.0/24"
    $nvaSubnetName = "iatd_labs_08_nva_subnet"
    $nvaSubnetPrefix = "172.16.2.0/24"

    # NVA Variables (We'll use a simple VM as a stand-in)
    $nvaName = "iatd_labs_08_nva"
    $nvaSize = "Standard_DS1_v2"
    $nvaAdminUsername = "cloudadmin"
    $nvaAdminPassword = "YourStrongPassword123!"

    # VM Variables
    $vmAName = "iatd_labs_08_vm_a"
    $vmASize = "Standard_DS1_v2"
    $vmBName = "iatd_labs_08_vm_b"
    $vmBSize = "Standard_DS1_v2"
    $vmAdminUsername = "cloudadmin"
    $vmAdminPassword = "YourStrongPassword123!" # Replace with a strong password
    ```

3.  **Create Resource Group (PowerShell):**

    ```powershell
    New-AzResourceGroup -Name $resourceGroupName -Location $location -ErrorAction SilentlyContinue
    ```

    Expected Output:
    ```
    ResourceGroupName : iatd_labs_08_rg
    Location          : australiaeast
    ProvisioningState : Succeeded
    Tags              : 
    ResourceId        : /subscriptions/your-subscription-id/resourceGroups/iatd_labs_08_rg
    ```

4.  **Create Virtual Network and Subnets (PowerShell):**

    ```powershell
    # Create VNet
    $vnet = New-AzVirtualNetwork -Name $vnetName -ResourceGroupName $resourceGroupName -Location $location -AddressPrefix $vnetAddressPrefix

    # Create Subnet A
    $subnetA = Add-AzVirtualNetworkSubnetConfig -Name $subnetAName -VirtualNetwork $vnet -AddressPrefix $subnetAPrefix

    # Create Subnet B
    $subnetB = Add-AzVirtualNetworkSubnetConfig -Name $subnetBName -VirtualNetwork $vnet -AddressPrefix $subnetBPrefix

    # Create NVA Subnet
    $nvaSubnet = Add-AzVirtualNetworkSubnetConfig -Name $nvaSubnetName -VirtualNetwork $vnet -AddressPrefix $nvaSubnetPrefix

    $vnet | Set-AzVirtualNetwork
    ```

    Expected Output:
    ```
    Name                : iatd_labs_08_vnet
    ResourceGroupName   : iatd_labs_08_rg
    Location            : australiaeast
    AddressSpace        : {172.16.0.0/16}
    Subnets             : [
                            {
                              "Name": "iatd_labs_08_subnet_a",
                              "AddressPrefix": "172.16.0.0/24"
                            },
                            {
                              "Name": "iatd_labs_08_subnet_b",
                              "AddressPrefix": "172.16.1.0/24"
                            },
                            {
                              "Name": "iatd_labs_08_nva_subnet",
                              "AddressPrefix": "172.16.2.0/24"
                            }
                          ]
    ```

    **Verify Subnets:**
    ```powershell
    Get-AzVirtualNetwork -Name $vnetName -ResourceGroupName $resourceGroupName | 
    Select-Object -ExpandProperty Subnets | 
    Format-Table Name, AddressPrefix
    ```

    Expected Output:
    ```
    Name                    AddressPrefix
    ----                    -------------
    iatd_labs_08_subnet_a   172.16.0.0/24
    iatd_labs_08_subnet_b   172.16.1.0/24
    iatd_labs_08_nva_subnet 172.16.2.0/24
    ```

5.  **Create Public IPs for VMs (CLI):**

    *   Open Cloud Shell and select **Bash**.

    ```bash
    # Create Public IP for VM A
    az network public-ip create \
      --resource-group $resourceGroupName \
      --name iatd_labs_08_vm_a_pip \
      --allocation-method Static \
      --sku Standard \
      --location $location
    ```

    Expected Output:
    ```json
    {
      "publicIp": {
        "name": "iatd_labs_08_vm_a_pip",
        "resourceGroup": "iatd_labs_08_rg",
        "location": "australiaeast",
        "sku": {
          "name": "Standard"
        },
        "publicIPAllocationMethod": "Static",
        "ipAddress": "20.1.2.3"
      }
    }
    ```

    ```bash
    # Create Public IP for VM B
    az network public-ip create \
      --resource-group $resourceGroupName \
      --name iatd_labs_08_vm_b_pip \
      --allocation-method Static \
      --sku Standard \
      --location $location
    ```

    Expected Output:
    ```json
    {
      "publicIp": {
        "name": "iatd_labs_08_vm_b_pip",
        "resourceGroup": "iatd_labs_08_rg",
        "location": "australiaeast",
        "sku": {
          "name": "Standard"
        },
        "publicIPAllocationMethod": "Static",
        "ipAddress": "20.1.2.4"
      }
    }
    ```

    ```bash
    # Create Public IP for NVA
    az network public-ip create \
      --resource-group $resourceGroupName \
      --name iatd_labs_08_nva_pip \
      --allocation-method Static \
      --sku Standard \
      --location $location
    ```

    Expected Output:
    ```json
    {
      "publicIp": {
        "name": "iatd_labs_08_nva_pip",
        "resourceGroup": "iatd_labs_08_rg",
        "location": "australiaeast",
        "sku": {
          "name": "Standard"
        },
        "publicIPAllocationMethod": "Static",
        "ipAddress": "20.1.2.5"
      }
    }
    ```

    **Verify Public IPs:**
    ```bash
    az network public-ip list \
      --resource-group $resourceGroupName \
      --query "[].{Name:name, IPAddress:ipAddress, AllocationMethod:publicIPAllocationMethod}" \
      -o table
    ```

    Expected Output:
    ```
    Name                    IPAddress    AllocationMethod
    ----------------------  -----------  -----------------
    iatd_labs_08_vm_a_pip  20.1.2.3     Static
    iatd_labs_08_vm_b_pip  20.1.2.4     Static
    iatd_labs_08_nva_pip   20.1.2.5     Static
    ```

6.  **Create VM A (CLI):**

    ```bash
    az vm create \
      --resource-group $resourceGroupName \
      --name $vmAName \
      --image UbuntuLTS \
      --admin-username $vmAdminUsername \
      --admin-password "$vmAdminPassword" \
      --vnet-name $vnetName \
      --subnet $subnetAName \
      --public-ip-address iatd_labs_08_vm_a_pip \
      --size $vmASize \
      --open-ports 22 80 \
      --location $location
    ```

7.  **Create VM B (CLI):**

    ```bash
    az vm create \
      --resource-group $resourceGroupName \
      --name $vmBName \
      --image UbuntuLTS \
      --admin-username $vmAdminUsername \
      --admin-password "$vmAdminPassword" \
      --vnet-name $vnetName \
      --subnet $subnetBName \
      --public-ip-address iatd_labs_08_vm_b_pip \
      --size $vmBSize \
      --open-ports 22 80 \
      --location $location
    ```

8.  **Create the NVA (CLI):** This will be a standard VM, simulating the NVA.

    ```bash
    az vm create \
      --resource-group $resourceGroupName \
      --name $nvaName \
      --image UbuntuLTS \
      --admin-username $nvaAdminUsername \
      --admin-password "$nvaAdminPassword" \
      --vnet-name $vnetName \
      --subnet $nvaSubnetName \
      --public-ip-address iatd_labs_08_nva_pip \
      --size $nvaSize \
      --open-ports 22 \
      --location $location
    ```

    Expected Output:
    ```json
    {
      "id": "/subscriptions/your-subscription-id/resourceGroups/iatd_labs_08_rg/providers/Microsoft.Compute/virtualMachines/iatd_labs_08_nva",
      "location": "australiaeast",
      "name": "iatd_labs_08_nva",
      "powerState": "VM running",
      "privateIpAddress": "172.16.2.4",
      "publicIpAddress": "20.1.2.5",
      "resourceGroup": "iatd_labs_08_rg",
      "zones": ""
    }
    ```

    **Configure IP Forwarding on NVA:**
    ```bash
    # Enable IP forwarding on NVA's NIC
    az network nic update \
      --name iatd_labs_08_nva-nic \
      --resource-group $resourceGroupName \
      --ip-forwarding true
    ```

    Expected Output:
    ```json
    {
      "enableIpForwarding": true,
      "id": "/subscriptions/your-subscription-id/resourceGroups/iatd_labs_08_rg/providers/Microsoft.Network/networkInterfaces/iatd_labs_08_nva-nic",
      "name": "iatd_labs_08_nva-nic",
      "resourceGroup": "iatd_labs_08_rg"
    }
    ```

### Part 2: Configuring Network Security Groups (NSGs)

1.  **Create NSG for VM A (Portal):**

    *   Go to the Azure Portal.
    *   Search for "Network security groups" and select.
    *   Click "+ Create".
        *   **Subscription:** Your subscription.
        *   **Resource group:** `iatd_labs_08_rg`.
        *   **Region:** Your region.
        *   **Name:** `iatd_labs_08_nsg_vm_a`.
        *   Click "Review + create" then "Create".

    *   Go to the `iatd_labs_08_nsg_vm_a` NSG.
    *   In the "Settings" blade, click on "Inbound security rules".
    *   Click "+ Add".
        *   **Source:** "Any".
        *   **Source port ranges:** "*".
        *   **Destination:** "Any".
        *   **Destination port ranges:** "22".
        *   **Protocol:** "TCP".
        *   **Action:** "Allow".
        *   **Priority:** "100".
        *   **Name:** "Allow-SSH".
        *   Click "Add".
    *   Click "+ Add".
        *   **Source:** "Any".
        *   **Source port ranges:** "*".
        *   **Destination:** "Any".
        *   **Destination port ranges:** "80".
        *   **Protocol:** "TCP".
        *   **Action:** "Allow".
        *   **Priority:** "101".
        *   **Name:** "Allow-HTTP".
        *   Click "Add".

    *   Associate NSG to Subnet A:
        *   Search for "Virtual networks" and select.
        *   Click on the `iatd_labs_08_vnet`
        *   In the "Settings" blade, click on "Subnets".
        *   Click on `iatd_labs_08_subnet_a`
        *   Under "Network security group", select `iatd_labs_08_nsg_vm_a`.
        *   Click "Save".

2.  **Create NSG for VM B (Portal):**

    *   Go to the Azure Portal.
    *   Search for "Network security groups" and select.
    *   Click "+ Create".
        *   **Subscription:** Your subscription.
        *   **Resource group:** `iatd_labs_08_rg`.
        *   **Region:** Your region.
        *   **Name:** `iatd_labs_08_nsg_vm_b`.
        *   Click "Review + create" then "Create".

    *   Go to the `iatd_labs_08_nsg_vm_b` NSG.
    *   In the "Settings" blade, click on "Inbound security rules".
    *   Click "+ Add".
        *   **Source:** "Any".
        *   **Source port ranges:** "*".
        *   **Destination:** "Any".
        *   **Destination port ranges:** "22".
        *   **Protocol:** "TCP".
        *   **Action:** "Allow".
        *   **Priority:** "100".
        *   **Name:** "Allow-SSH".
        *   Click "Add".
    *   Click "+ Add".
        *   **Source:** "Any".
        *   **Source port ranges:** "*".
        *   **Destination:** "Any".
        *   **Destination port ranges:** "80".
        *   **Protocol:** "TCP".
        *   **Action:** "Allow".
        *   **Priority:** "101".
        *   **Name:** "Allow-HTTP".
        *   Click "Add".

    *   Associate NSG to Subnet B:
        *   Search for "Virtual networks" and select.
        *   Click on the `iatd_labs_08_vnet`
        *   In the "Settings" blade, click on "Subnets".
        *   Click on `iatd_labs_08_subnet_b`
        *   Under "Network security group", select `iatd_labs_08_nsg_vm_b`.
        *   Click "Save".

3.  **Create NSG for NVA (Portal):**

    *   Go to the Azure Portal.
    *   Search for "Network security groups" and select.
    *   Click "+ Create".
        *   **Subscription:** Your subscription.
        *   **Resource group:** `iatd_labs_08_rg`.
        *   **Region:** Your region.
        *   **Name:** `iatd_labs_08_nsg_nva`.
        *   Click "Review + create" then "Create".

    *   Go to the `iatd_labs_08_nsg_nva` NSG.
    *   In the "Settings" blade, click on "Inbound security rules".
    *   Click "+ Add".
        *   **Source:** "Any".
        *   **Source port ranges:** "*".
        *   **Destination:** "Any".
        *   **Destination port ranges:** "22".
        *   **Protocol:** "TCP".
        *   **Action:** "Allow".
        *   **Priority:** "100".
        *   **Name:** "Allow-SSH".
        *   Click "Add".
    *   In the "Settings" blade, click on "Outbound security rules".
    *   Click "+ Add".
        *   **Destination:** "Any".
        *   **Destination port ranges:** "*".
        *   **Protocol:** "*".
        *   **Action:** "Allow".
        *   **Priority:** "100".
        *   **Name:** "Allow-All-Outbound".
        *   Click "Add".

    *   Associate NSG to NVA Subnet:
        *   Search for "Virtual networks" and select.
        *   Click on the `iatd_labs_08_vnet`
        *   In the "Settings" blade, click on "Subnets".
        *   Click on `iatd_labs_08_nva_subnet`
        *   Under "Network security group", select `iatd_labs_08_nsg_nva`.
        *   Click "Save".

### Part 3: Configuring UDRs to Route Traffic Through the NVA

1.  **Create Route Table (PowerShell):**

    ```powershell
    $routeTableName = "iatd_labs_08_rt"
    $routeTable = New-AzRouteTable -Name $routeTableName -ResourceGroupName $resourceGroupName -Location $location
    ```

2.  **Create UDR for Subnet A (PowerShell):**  This route sends traffic destined for Subnet B *through* the NVA.

    ```powershell
    # Get NVA Private IP
    $nva = Get-AzVM -Name $nvaName -ResourceGroupName $resourceGroupName
    $nvaPrivateIP = ($nva.PrivateIPAddresses)[0]

    # Create Route to Subnet B via NVA
    $routeAName = "iatd_labs_08_route_a"
    $routeA = New-AzRouteConfig -Name $routeAName -AddressPrefix "172.16.1.0/24" -NextHopType "VirtualAppliance" -NextHopIpAddress $nvaPrivateIP
    Set-AzRouteTable -RouteTable $routeTable -Route $routeA
    ```

3.  **Associate Route Table to Subnet A (PowerShell):**

    ```powershell
    Set-AzVirtualNetworkSubnetConfig -Name $subnetAName -VirtualNetwork $vnet -ResourceGroupName $resourceGroupName -RouteTable $routeTable | Set-AzVirtualNetwork
    ```

### Part 4: Verifying Traffic Flow (This section assumes a simple test scenario, as actual traffic optimization is not performed)

1.  **Get VM A and VM B Private IPs (PowerShell):**

    ```powershell
    $vmA = Get-AzVM -Name $vmAName -ResourceGroupName $resourceGroupName
    $vmAPrivateIP = ($vmA.PrivateIPAddresses)[0]
    $vmB = Get-AzVM -Name $vmBName -ResourceGroupName $resourceGroupName
    $vmBPrivateIP = ($vmB.PrivateIPAddresses)[0]

    Write-Host "VM A Private IP: $vmAPrivateIP"
    Write-Host "VM B Private IP: $vmBPrivateIP"
    ```

    Expected Output:
    ```
    VM A Private IP: 172.16.0.4
    VM B Private IP: 172.16.1.4
    ```

2.  **Verify Route Table Configuration (PowerShell):**

    ```powershell
    Get-AzRouteTable -ResourceGroupName $resourceGroupName -Name $routeTableName | 
    Select-Object -ExpandProperty Routes | 
    Format-Table Name, AddressPrefix, NextHopType, NextHopIpAddress
    ```

    Expected Output:
    ```
    Name                  AddressPrefix    NextHopType      NextHopIpAddress
    ----                  -------------    -----------      ---------------
    iatd_labs_08_route_a  172.16.1.0/24   VirtualAppliance 172.16.2.4
    ```

3.  **Test Traffic Flow through NVA:**

    a. SSH into VM A:
    ```bash
    ssh cloudadmin@<VM_A_PUBLIC_IP>
    ```

    b. Install traceroute:
    ```bash
    sudo apt-get update && sudo apt-get install traceroute -y
    ```

    c. Run traceroute to VM B's private IP:
    ```bash
    traceroute <VM_B_PRIVATE_IP>
    ```

    Expected Output:
    ```
    traceroute to 172.16.1.4 (172.16.1.4), 30 hops max, 60 byte packets
     1  172.16.2.4 (172.16.2.4)  1.123 ms  1.045 ms  0.978 ms  # NVA IP
     2  172.16.1.4 (172.16.1.4)  1.890 ms  1.812 ms  1.745 ms  # VM B IP
    ```

    This output confirms that traffic is being routed through the NVA (hop 1) before reaching VM B.

4.  **Verify IP Forwarding on NVA:**

    ```bash
    az network nic show \
      --resource-group $resourceGroupName \
      --name iatd_labs_08_nva-nic \
      --query "enableIpForwarding" \
      -o tsv
    ```

    Expected Output:
    ```
    true
    ```

5.  **Test Network Performance (Optional):**

    a. Install iperf3 on both VMs:
    ```bash
    sudo apt-get update && sudo apt-get install iperf3 -y
    ```

    b. Start iperf3 server on VM B:
    ```bash
    iperf3 -s
    ```

    c. Run iperf3 client on VM A:
    ```bash
    iperf3 -c <VM_B_PRIVATE_IP>
    ```

    Example Output:
    ```
    [ ID] Interval           Transfer     Bitrate
    [  5]   0.00-10.00  sec   1.12 GBytes   961 Mbits/sec
    ```

    Note: The actual performance will vary based on VM size, network conditions, and whether a real NVA with optimization capabilities is used.

### Post-Lab: Cleanup

To avoid unnecessary costs, clean up the created resources using any of these methods:

1.  **Azure Portal:**
    * Navigate to Resource Groups
    * Select `iatd_labs_08_rg`
    * Click 'Delete resource group'
    * Type the resource group name to confirm
    * Click 'Delete'

2.  **PowerShell:**
    ```powershell
    Remove-AzResourceGroup -Name $resourceGroupName -Force
    ```
    Expected Output:
    ```
    True
    ```

3.  **Azure CLI:**
    ```bash
    az group delete --name $resourceGroupName --yes
    ```
    Expected Output:
    ```
    No output indicates successful deletion
    ```

Verify Cleanup:
* Check the Azure Portal to ensure the resource group and all resources are deleted
* Run `Get-AzResourceGroup` or `az group list` to verify the resource group no longer exists

### Post-Lab Summary

**Key Takeaways:**
* Network Virtual Appliances (NVAs) can optimize traffic flow in virtual networks
* User-Defined Routes (UDRs) control traffic routing through NVAs
* IP forwarding must be enabled on NVA network interfaces
* NSGs protect and control access to network resources
* Proper subnet design is crucial for NVA deployment

**Next Steps:**
* Explore different types of NVAs available in Azure Marketplace
* Learn about high availability configurations for NVAs
* Study NVA performance monitoring and scaling
* Practice implementing more complex routing scenarios
* Experiment with actual traffic optimization NVAs

**Additional Resources:**
* [Network Virtual Appliances Overview](https://docs.microsoft.com/en-us/azure/virtual-network/virtual-network-appliance-overview)
* [User-Defined Routes Documentation](https://docs.microsoft.com/en-us/azure/virtual-network/virtual-networks-udr-overview)
* [Azure Networking Best Practices](https://docs.microsoft.com/en-us/azure/cloud-adoption-framework/ready/azure-best-practices/networking-best-practices)
* [High Availability for NVAs](https://docs.microsoft.com/en-us/azure/architecture/reference-architectures/dmz/nva-ha)

**Congratulations!** You've successfully deployed a Network Virtual Appliance and configured traffic routing through it using User-Defined Routes. This lab has demonstrated the fundamental concepts of NVA deployment and traffic routing in Azure, providing a foundation for implementing more sophisticated network optimization solutions.
