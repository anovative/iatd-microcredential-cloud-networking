## IATD Microcredential Cloud Networking: Lab 5 - Simulating Route Propagation with Virtual Network Peering

**Objective:** Configure Virtual Network (VNet) peering with route propagation to enable automatic exchange of routing information between two virtual networks.

**Scenario:** Your organization has two virtual networks that need to communicate securely and efficiently. You need to set up VNet peering with route propagation to ensure that any changes in routing configuration are automatically shared between the networks. This lab demonstrates how to configure VNet peering, enable route propagation, and verify the automatic exchange of routing information.

**Outcomes:**

* Creation of two virtual networks with distinct address spaces
* Configuration of VNet peering between the networks
* Implementation of route propagation settings
* Deployment of test VMs in each VNet
* Verification of automatic route exchange and connectivity
* Understanding of route propagation behavior in peered networks

**Estimated Time:** 60 - 90 minutes

**Prerequisites:**

1. **Azure Subscription:** Requires an active Azure subscription. A free account is available at [https://azure.microsoft.com/free/](https://azure.microsoft.com/free/).
2. **Basic Networking Knowledge:** Familiarity with VNets, subnets, IP addressing, and peering concepts.
3. **Previous Labs:** Understanding of concepts from Labs 1-4, particularly routing configuration.

**Lab Conventions**

* **Azure Cloud Shell:** All PowerShell and CLI interactions will occur within the Azure Cloud Shell environment.
* **Naming Conventions:** Resource names follow the convention `iatd_labs_05_*`.
* **IP Address Range:** The `172.16.x.x` range will be used for consistency across labs.
* **Location:** Choose a consistent Azure region (e.g., `australiaeast`).

#### Resource Naming

* **Resource Group:** `iatd_labs_05_rg`
* **Virtual Networks:**
  * VNet1: `iatd_labs_05_vnet1` (172.16.1.0/24)
  * VNet2: `iatd_labs_05_vnet2` (172.16.2.0/24)
* **Subnets:**
  * Subnet1: `iatd_labs_05_subnet1` (172.16.1.0/24)
  * Subnet2: `iatd_labs_05_subnet2` (172.16.2.0/24)
* **Virtual Machines:**
  * VM1: `iatd_labs_05_vm1`
  * VM2: `iatd_labs_05_vm2`
* **Network Interfaces:**
  * VM1 NIC: `iatd_labs_05_vm1_nic`
  * VM2 NIC: `iatd_labs_05_vm2_nic`
* **Public IPs:**
  * VM1 PIP: `iatd_labs_05_vm1_pip`
  * VM2 PIP: `iatd_labs_05_vm2_pip`
* **VNet Peering:**
  * VNet1 to VNet2: `iatd_labs_05_peering1`
  * VNet2 to VNet1: `iatd_labs_05_peering2`

### Part 1: Creating the Virtual Networks and Subnets

1.  **Open Cloud Shell:** Open Azure Cloud Shell and select **PowerShell**.

2.  **Define Variables:**

    ```powershell
    $resourceGroupName = "iatd_labs_05_rg"
    $location = "australiaeast" # Replace with your preferred region

    # VNet1 Variables
    $vnet1Name = "iatd_labs_05_vnet1"
    $vnet1AddressPrefix = "172.16.1.0/24"
    $subnet1Name = "iatd_labs_05_subnet1"
    $subnet1Prefix = "172.16.1.0/24"

    # VNet2 Variables
    $vnet2Name = "iatd_labs_05_vnet2"
    $vnet2AddressPrefix = "172.16.2.0/24"
    $subnet2Name = "iatd_labs_05_subnet2"
    $subnet2Prefix = "172.16.2.0/24"

    # VM Variables
    $vm1Name = "iatd_labs_05_vm1"
    $vm2Name = "iatd_labs_05_vm2"
    $vmSize = "Standard_DS1_v2" # Change if required
    $adminUsername = "cloudadmin"
    $adminPassword = "YourStrongPassword123!" # Replace with a strong password
    ```

3.  **Create Resource Group (PowerShell):**

    ```powershell
    New-AzResourceGroup -Name $resourceGroupName -Location $location -ErrorAction SilentlyContinue
    ```

4.  **Create VNet1 and Subnet (PowerShell):**

    ```powershell
    # Create VNet1
    $vnet1 = New-AzVirtualNetwork -Name $vnet1Name -ResourceGroupName $resourceGroupName -Location $location -AddressPrefix $vnet1AddressPrefix

    # Create Subnet1
    $subnet1 = Add-AzVirtualNetworkSubnetConfig -Name $subnet1Name -VirtualNetwork $vnet1 -AddressPrefix $subnet1Prefix
    $vnet1 | Set-AzVirtualNetwork
    ```

5.  **Create VNet2 and Subnet (CLI):**

    *   Open Cloud Shell and select **Bash**.

    ```bash
    az network vnet create \
      --resource-group $resourceGroupName \
      --name $vnet2Name \
      --address-prefixes $vnet2AddressPrefix \
      --location $location
    az network vnet subnet create \
      --resource-group $resourceGroupName \
      --vnet-name $vnet2Name \
      --name $subnet2Name \
      --address-prefixes $subnet2Prefix
    ```

6.  **Verify VNets and Subnets (Portal):**

    *   Go to the Azure Portal ([https://portal.azure.com/](https://portal.azure.com/)).
    *   Search for "Virtual networks" and select.
    *   You should see `iatd_labs_05_vnet1` and `iatd_labs_05_vnet2` listed. Click on each to verify the subnets and address spaces.

    *   Verify `iatd_labs_05_vnet1` subnet
        *   Click on `iatd_labs_05_vnet1`
        *   In Settings, select `Subnets`
        *   Should see subnet `iatd_labs_05_subnet1`
    *   Verify `iatd_labs_05_vnet2` subnet
        *   Click on `iatd_labs_05_vnet2`
        *   In Settings, select `Subnets`
        *   Should see subnet `iatd_labs_05_subnet2`

### Part 2: Creating VMs in each VNet

1.  **Create Public IP for VM1 (PowerShell):**

    ```powershell
    $publicIP1 = New-AzPublicIpAddress -Name iatd_labs_05_vm1_pip -ResourceGroupName $resourceGroupName -AllocationMethod Static -Sku Standard -Location $location
    ```

2.  **Create VM1 (PowerShell):**

    ```powershell
    New-AzVm -ResourceGroupName $resourceGroupName -Name $vm1Name -Location $location -VirtualNetworkName $vnet1Name -SubnetName $subnet1Name -PublicIpAddressName $publicIP1.Name -OpenPorts 22,80 -Size $vmSize -Credential (Get-Credential -UserName $adminUsername -Message "Enter password")
    ```

3.  **Create Public IP for VM2 (CLI):**

    *   Open Cloud Shell and select **Bash**.

    ```bash
    az network public-ip create \
      --resource-group $resourceGroupName \
      --name iatd_labs_05_vm2_pip \
      --allocation-method Static \
      --sku Standard \
      --location $location
    ```

4.  **Create VM2 (CLI):**

    ```bash
    az vm create \
      --resource-group $resourceGroupName \
      --name $vm2Name \
      --image UbuntuLTS \
      --admin-username $adminUsername \
      --admin-password "$adminPassword" \
      --vnet-name $vnet2Name \
      --subnet $subnet2Name \
      --public-ip-address iatd_labs_05_vm2_pip \
      --size $vmSize \
      --open-ports 22 80 \
      --location $location
    ```

### Part 3: Configuring VNet Peering with Route Propagation

1.  **Create VNet Peering from VNet1 to VNet2 (Portal):**

    *   In the Azure Portal, navigate to the `iatd_labs_05_vnet1` Virtual Network.
    *   In the Settings blade, click on "Peerings".
    *   Click "+ Add".
        *   **Peering link name:** `iatd_labs_05_peering1`
        *   **Traffic to remote virtual network:**
            *   **Virtual network deployment model:** "Resource Manager"
            *   **Subscription:** Your Subscription.
            *   **Virtual network:** Select `iatd_labs_05_vnet2`.
            *   **Allow virtual network access:** Select "Enabled".
            *   **Allow forwarded traffic:** Select "Enabled".
            *   **Allow gateway transit:** Select "Disabled".
            *   **Use remote gateways:** Select "Enabled".
            *   **Propagate gateway routes:** "Enabled"
        *   Click "Add".

2.  **Create VNet Peering from VNet2 to VNet1 (CLI):**

    *   Open Cloud Shell and select **Bash**.

    ```bash
    az network vnet peering create \
      --name iatd_labs_05_peering2 \
      --resource-group $resourceGroupName \
      --vnet-name $vnet2Name \
      --remote-vnet $vnet1Name \
      --allow-vnet-access true \
      --allow-forwarded-traffic true \
      --allow-gateway-transit false \
      --use-remote-gateways true
    ```

    Expected Output:
    ```json
    {
      "allowForwardedTraffic": true,
      "allowGatewayTransit": false,
      "allowVirtualNetworkAccess": true,
      "name": "iatd_labs_05_peering2",
      "peeringState": "Connected",
      "provisioningState": "Succeeded",
      "remoteVirtualNetwork": {
        "id": "/subscriptions/your-subscription-id/resourceGroups/iatd_labs_05_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_05_vnet1"
      },
      "useRemoteGateways": true
    }
    ```

### Post-Lab: Cleanup

To avoid unnecessary costs, clean up the created resources using any of these methods:

1.  **Azure Portal:**
    * Navigate to Resource Groups
    * Select `iatd_labs_05_rg`
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
* VNet peering enables direct network connectivity between virtual networks
* Route propagation automatically shares routing information between peered networks
* Both PowerShell and Azure CLI can be used to manage and verify network configurations
* Proper network security and access controls are essential in peered networks

**Next Steps:**
* Explore more complex routing scenarios with multiple VNets
* Learn about transitive routing in VNet peering
* Study VNet peering costs and limitations
* Practice implementing network security groups (NSGs) in peered networks

**Additional Resources:**
* [Virtual Network Peering Documentation](https://docs.microsoft.com/en-us/azure/virtual-network/virtual-network-peering-overview)
* [Route Propagation in Azure](https://docs.microsoft.com/en-us/azure/virtual-network/virtual-networks-udr-overview)
* [Azure Networking Best Practices](https://docs.microsoft.com/en-us/azure/cloud-adoption-framework/ready/azure-best-practices/networking-best-practices)

**Congratulations!** You've successfully configured VNet peering with route propagation between two virtual networks. You've learned how to enable automatic exchange of routing information and verified connectivity between VMs in different VNets.
      --resource-group $resourceGroupName \
      --vnet-name $vnet2Name \
      --remote-vnet /subscriptions/$(az account show --query id --output tsv)/resourceGroups/$resourceGroupName/providers/Microsoft.Network/virtualNetworks/$vnet1Name \
      --allow-forwarded-traffic \
      --allow-vnet-access \
      --allow-gateway-transit
    ```

3.  **Verify Peering Status (PowerShell):**

    *   Open Cloud Shell and select **PowerShell**.

    ```powershell
    Get-AzVirtualNetworkPeering -ResourceGroupName $resourceGroupName -VirtualNetworkName $vnet1Name
    Get-AzVirtualNetworkPeering -ResourceGroupName $resourceGroupName -VirtualNetworkName $vnet2Name
    ```

    **Example Output (truncated):**

    ```
    #Example Output for VNet1
    Name                         : iatd_labs_05_peering1
    ResourceGroupName            : iatd_labs_05_rg
    VirtualNetworkName           : iatd_labs_05_vnet1
    RemoteVirtualNetworkName     : iatd_labs_05_vnet2
    AllowVirtualNetworkAccess    : True
    AllowForwardedTraffic        : True
    AllowGatewayTransit          : False
    UseRemoteGateways            : True
    PeeringState                 : Connected
    ...

    #Example Output for VNet2
    Name                         : iatd_labs_05_peering2
    ResourceGroupName            : iatd_labs_05_rg
    VirtualNetworkName           : iatd_labs_05_vnet2
    RemoteVirtualNetworkName     : iatd_labs_05_vnet1
    AllowVirtualNetworkAccess    : True
    AllowForwardedTraffic        : True
    AllowGatewayTransit          : True
    UseRemoteGateways            : False
    PeeringState                 : Connected
    ...
    ```

### Part 4: Testing Connectivity and Route Propagation

1.  **Get VM Private IPs (PowerShell):**  We will use the private IPs to test connectivity, as VMs are not directly accessible through the public IP on initial boot.

    ```powershell
    $vm1 = Get-AzVM -Name $vm1Name -ResourceGroupName $resourceGroupName
    $vm2 = Get-AzVM -Name $vm2Name -ResourceGroupName $resourceGroupName
    ```

    Expected Output:
    ```
    ResourceGroupName : iatd_labs_05_rg
    Name              : iatd_labs_05_vm1
    Location          : australiaeast
    Type              : Microsoft.Compute/virtualMachines
    Tags              : {}

    ResourceGroupName : iatd_labs_05_rg
    Name              : iatd_labs_05_vm2
    Location          : australiaeast
    Type              : Microsoft.Compute/virtualMachines
    Tags              : {}
    ```

    Get the private IPs:
    ```powershell
    $vm1PrivateIP = (Get-AzNetworkInterface -ResourceGroupName $resourceGroupName | Where-Object { $_.Name -like "*vm1*" }).IpConfigurations[0].PrivateIpAddress
    $vm2PrivateIP = (Get-AzNetworkInterface -ResourceGroupName $resourceGroupName | Where-Object { $_.Name -like "*vm2*" }).IpConfigurations[0].PrivateIpAddress
    Write-Host "VM1 Private IP: $vm1PrivateIP"
    Write-Host "VM2 Private IP: $vm2PrivateIP"
    ```

    Expected Output:
    ```
    VM1 Private IP: 172.16.1.4
    VM2 Private IP: 172.16.2.4
    ```
    $vm1PrivateIP = ($vm1.PrivateIPAddresses)[0]
    $vm2PrivateIP = ($vm2.PrivateIPAddresses)[0]

    Write-Host "VM1 Private IP: $vm1PrivateIP"
    Write-Host "VM2 Private IP: $vm2PrivateIP"
    ```

2.  **Test Connectivity (ICMP/Ping) from VNet1 to VNet2 (PowerShell):**  From Cloud Shell, use `Test-NetConnection`.  This command checks if a TCP connection can be established.  It also provides information about the network path.

    ```powershell
    Test-NetConnection -ComputerName $vm2PrivateIP -Port 22
    ```

    **Expected Result**: Should show the TCP test successful. This verifies the connectivity between the VMs.

    **Test Connectivity (ICMP/Ping):**
    ```powershell
    Test-NetConnection -ComputerName $vm2PrivateIP -Port 22
    ```

    **Expected Result**: Should show the TCP test successful. This verifies the connectivity between the VMs.

3.  **Verify Route Propagation:**

    **Using PowerShell:**
    ```powershell
    # Get the peering status for VNet1
    $vnet1 = Get-AzVirtualNetwork -Name $vnet1Name -ResourceGroupName $resourceGroupName
    $peering1 = Get-AzVirtualNetworkPeering -VirtualNetworkName $vnet1Name -ResourceGroupName $resourceGroupName -Name iatd_labs_05_peering1
    $peering1 | Format-List PeeringState, AllowForwardedTraffic, AllowGatewayTransit, UseRemoteGateways
    ```

    Expected Output:
    ```
    PeeringState         : Connected
    AllowForwardedTraffic: True
    AllowGatewayTransit  : False
    UseRemoteGateways    : True
    ```

    **Using Azure CLI:**
    ```bash
    # Get the peering status for VNet2
    az network vnet peering show \
      --name iatd_labs_05_peering2 \
      --resource-group $resourceGroupName \
      --vnet-name $vnet2Name \
      --query '{PeeringState:peeringState, AllowForwardedTraffic:allowForwardedTraffic, AllowGatewayTransit:allowGatewayTransit, UseRemoteGateways:useRemoteGateways}' \
      -o json
    ```

    Expected Output:
    ```json
    {
      "PeeringState": "Connected",
      "AllowForwardedTraffic": true,
      "AllowGatewayTransit": false,
      "UseRemoteGateways": true
    }
    ```

    **Using the Portal:**
    1. Navigate to the Azure Portal
    2. Go to Virtual Networks > `iatd_labs_05_vnet1` > Peerings
    3. Click on `iatd_labs_05_peering1`
    4. Verify the following settings:
       * Peering status: Connected
       * Allow forwarded traffic: Yes
       * Allow gateway transit: No
       * Use remote gateways: Yes

    **Test Connectivity Between VMs:**
    ```bash
    # From VM1, ping VM2's private IP
    ssh cloudadmin@$vm1PublicIP "ping -c 4 $vm2PrivateIP"
    ```

    Expected Output:
    ```
    PING 172.16.2.4 (172.16.2.4) 56(84) bytes of data.
    64 bytes from 172.16.2.4: icmp_seq=1 ttl=64 time=1.45 ms
    64 bytes from 172.16.2.4: icmp_seq=2 ttl=64 time=1.12 ms
    64 bytes from 172.16.2.4: icmp_seq=3 ttl=64 time=1.08 ms
    64 bytes from 172.16.2.4: icmp_seq=4 ttl=64 time=1.21 ms

    --- 172.16.2.4 ping statistics ---
    4 packets transmitted, 4 received, 0% packet loss, time 3005ms
    rtt min/avg/max/mdev = 1.080/1.215/1.450/0.147 ms
    ```
        *   Select the VNets one at a time and see the effective routes using the following steps.
        *   Click on `iatd_labs_05_vnet1`
        *   In Settings, select `Subnets`
        *   Click on the subnet `iatd_labs_05_subnet1`
        *   In Settings, select `Route tables`
        *   Select `Effective routes`
        *   You should see the routes from `iatd_labs_05_vnet2`
    *   **Using the CLI (Inside the VM, Not CloudShell)**
        *   Connect to `VM1` using SSH (You will need to allow port 22 for your public IP in network security group).
        *   run the command `ip route show`
        *   You should see the route to `172.16.2.0/24`

4.  **Further Use Cases (Optional - Discussion):**
    *   **Network Virtual Appliances (NVAs):**  Discuss how NVAs (e.g., firewalls) can be placed in one VNet and used to inspect traffic between peered VNets.  Route propagation would ensure traffic is directed through the NVA.
    *   **Firewalls:** Discuss how to create a firewall in Azure and route all traffic through it using UDRs and route propagation.
    *   **VPN Gateway Route Advertisement:**  Discuss how VPN Gateways can use BGP to advertise routes learned from on-premises networks and how route propagation can be used to share those routes with other peered VNets.

### Post-Lab: Cleanup

To avoid unnecessary costs, clean up the created resources.

1.  **Delete the Resource Group (PowerShell):**

    ```powershell
    Remove-AzResourceGroup -Name $resourceGroupName -Force
    ```

**Congratulations!** You've successfully configured VNet peering with route propagation, enabling seamless communication between your virtual networks. You have also explored additional advanced concepts.
