## IATD Microcredential Cloud Networking: Lab 7 - Configuring Cross-Region Routing with Virtual Network Peering

**Objective:** This lab focuses on configuring global virtual network peering to enable cross-region routing between two virtual networks. You will establish communication between resources in different Azure regions, understanding the implications of cross-region traffic.

**Estimated Time:** 60 - 90 minutes

**Prerequisites:**

1.  **Azure Subscription:** An active Azure subscription.
2.  **Azure Cloud Shell:** Access to Azure Cloud Shell (Bash or PowerShell).
3.  **Basic Understanding:** Familiarity with Virtual Networks, subnets, and peering concepts.
4.  **Region Selection:** Be prepared to choose two different Azure regions that support Virtual Network Peering.  `australiaeast` and another region.
5. **Knowledge:** Familiarity with creating resources via CLI and Portal.

**Let's get started!**

### Lab Conventions

*   **Azure Cloud Shell:** All PowerShell and CLI interactions will occur within the Azure Cloud Shell environment.
*   **Naming Conventions:** Resource names follow the convention `iatd_labs_07_*`.
*   **IP Address Range:** The `172.16.x.x` range will be used.
*   **Location:** Use the specified regions or replace with your preferred regions if necessary.

#### Resource Naming

*   **Resource Group 1 (Region 1):** `iatd_labs_07_rg1`
*   **Resource Group 2 (Region 2):** `iatd_labs_07_rg2`
*   **Virtual Network 1 (Region 1):** `iatd_labs_07_vnet1`
*   **Subnet 1 (VNet1):** `iatd_labs_07_subnet1`
*   **Virtual Network 2 (Region 2):** `iatd_labs_07_vnet2`
*   **Subnet 2 (VNet2):** `iatd_labs_07_subnet2`
*   **VM in VNet1 (Region 1):** `iatd_labs_07_vm1`
*   **VM in VNet2 (Region 2):** `iatd_labs_07_vm2`
*   **Public IP for VM1:** `iatd_labs_07_vm1_pip`
*   **Public IP for VM2:** `iatd_labs_07_vm2_pip`
*   **Peering 1 (VNet1 to VNet2):** `iatd_labs_07_peering1`
*   **Peering 2 (VNet2 to VNet1):** `iatd_labs_07_peering2`

### Part 1: Creating Virtual Networks and Subnets in Different Regions

1.  **Open Cloud Shell:** Open Azure Cloud Shell and select **PowerShell**.

2.  **Define Variables:**

    ```powershell
    # Region 1 Variables
    $resourceGroupName1 = "iatd_labs_07_rg1"
    $location1 = "australiaeast" # Replace with your first region
    $vnet1Name = "iatd_labs_07_vnet1"
    $vnet1AddressPrefix = "172.16.1.0/24"
    $subnet1Name = "iatd_labs_07_subnet1"
    $subnet1Prefix = "172.16.1.0/24"

    # Region 2 Variables
    $resourceGroupName2 = "iatd_labs_07_rg2"
    $location2 = "eastus" # Replace with your second region (Choose a different region than Region 1)
    $vnet2Name = "iatd_labs_07_vnet2"
    $vnet2AddressPrefix = "172.16.2.0/24"
    $subnet2Name = "iatd_labs_07_subnet2"
    $subnet2Prefix = "172.16.2.0/24"

    # VM Variables
    $vm1Name = "iatd_labs_07_vm1"
    $vm2Name = "iatd_labs_07_vm2"
    $vmSize = "Standard_DS1_v2"  # Change if required
    $adminUsername = "cloudadmin"
    $adminPassword = "YourStrongPassword123!" # Replace with a strong password
    ```

3.  **Create Resource Group 1 (PowerShell):**

    ```powershell
    New-AzResourceGroup -Name $resourceGroupName1 -Location $location1 -ErrorAction SilentlyContinue
    ```

    Expected Output:
    ```
    ResourceGroupName : iatd_labs_07_rg1
    Location          : australiaeast
    ProvisioningState : Succeeded
    Tags              : 
    ResourceId        : /subscriptions/your-subscription-id/resourceGroups/iatd_labs_07_rg1
    ```

4.  **Create VNet1 and Subnet (PowerShell):**

    ```powershell
    # Create VNet1
    $vnet1 = New-AzVirtualNetwork -Name $vnet1Name -ResourceGroupName $resourceGroupName1 -Location $location1 -AddressPrefix $vnet1AddressPrefix

    # Create Subnet1
    $subnet1 = Add-AzVirtualNetworkSubnetConfig -Name $subnet1Name -VirtualNetwork $vnet1 -AddressPrefix $subnet1Prefix
    $vnet1 | Set-AzVirtualNetwork
    ```

5.  **Create Resource Group 2 (CLI):**

    *   Open Cloud Shell and select **Bash**.

    ```bash
    az group create \
      --name $resourceGroupName2 \
      --location $location2
    ```

    Expected Output:
    ```json
    {
      "id": "/subscriptions/your-subscription-id/resourceGroups/iatd_labs_07_rg2",
      "location": "eastus",
      "managedBy": null,
      "name": "iatd_labs_07_rg2",
      "properties": {
        "provisioningState": "Succeeded"
      },
      "tags": null,
      "type": "Microsoft.Resources/resourceGroups"
    }
    ```

6.  **Create VNet2 and Subnet (CLI):**

    ```bash
    az network vnet create \
      --resource-group $resourceGroupName2 \
      --name $vnet2Name \
      --address-prefixes $vnet2AddressPrefix \
      --location $location2
    az network vnet subnet create \
      --resource-group $resourceGroupName2 \
      --vnet-name $vnet2Name \
      --name $subnet2Name \
      --address-prefixes $subnet2Prefix
    ```

7.  **Verify VNets and Subnets:**

    **Using PowerShell:**
    ```powershell
    # Verify VNets
    Get-AzVirtualNetwork -ResourceGroupName $resourceGroupName1 | Format-Table Name,ResourceGroupName,Location,@{Name='AddressSpace';Expression={$_.AddressSpace.AddressPrefixes -join ", "}}
    Get-AzVirtualNetwork -ResourceGroupName $resourceGroupName2 | Format-Table Name,ResourceGroupName,Location,@{Name='AddressSpace';Expression={$_.AddressSpace.AddressPrefixes -join ", "}}
    ```

    Expected Output:
    ```
    Name                ResourceGroupName Location      AddressSpace
    ----                ----------------- --------      ------------
    iatd_labs_07_vnet1  iatd_labs_07_rg1 australiaeast 172.16.1.0/24
    iatd_labs_07_vnet2  iatd_labs_07_rg2 eastus       172.16.2.0/24
    ```

    **Using Azure CLI:**
    ```bash
    # List all VNets
    az network vnet list \
      --query "[?contains(name, 'iatd_labs_07')].{Name:name, ResourceGroup:resourceGroup, Location:location, AddressSpace:addressSpace.addressPrefixes[0]}" \
      -o table
    ```

    Expected Output:
    ```
    Name                ResourceGroup    Location      AddressSpace
    ------------------  --------------- ------------- --------------
    iatd_labs_07_vnet1  iatd_labs_07_rg1 australiaeast 172.16.1.0/24
    iatd_labs_07_vnet2  iatd_labs_07_rg2 eastus       172.16.2.0/24
    ```

    **Using Portal:**
    *   Go to the Azure Portal ([https://portal.azure.com/](https://portal.azure.com/)).
    *   Search for "Virtual networks" and select.
    *   Filter by name "iatd_labs_07" to see both VNets.
    *   Verify:
        - VNet1 is in `australiaeast` with address space 172.16.1.0/24
        - VNet2 is in `eastus` with address space 172.16.2.0/24
        - Each VNet has one subnet with matching address space

### Part 2: Creating VMs in each VNet

1.  **Create Public IP for VM1 (PowerShell):**

    ```powershell
    $publicIP1 = New-AzPublicIpAddress -Name iatd_labs_07_vm1_pip -ResourceGroupName $resourceGroupName1 -AllocationMethod Static -Sku Standard -Location $location1
    ```

2.  **Create VM1 (PowerShell):**

    ```powershell
    New-AzVm -ResourceGroupName $resourceGroupName1 -Name $vm1Name -Location $location1 -VirtualNetworkName $vnet1Name -SubnetName $subnet1Name -PublicIpAddressName $publicIP1.Name -OpenPorts 22,80 -Size $vmSize -Credential (Get-Credential -UserName $adminUsername -Message "Enter password")
    ```

3.  **Create Public IP for VM2 (CLI):**

    *   Open Cloud Shell and select **Bash**.

    ```bash
    az network public-ip create \
      --resource-group $resourceGroupName2 \
      --name iatd_labs_07_vm2_pip \
      --allocation-method Static \
      --sku Standard \
      --location $location2
    ```

4.  **Create VM2 (CLI):**

    ```bash
    az vm create \
      --resource-group $resourceGroupName2 \
      --name $vm2Name \
      --image UbuntuLTS \
      --admin-username $adminUsername \
      --admin-password "$adminPassword" \
      --vnet-name $vnet2Name \
      --subnet $subnet2Name \
      --public-ip-address iatd_labs_07_vm2_pip \
      --size $vmSize \
      --open-ports 22 80 \
      --location $location2
    ```

### Part 3: Configuring Global VNet Peering

1.  **Create VNet Peering from VNet1 to VNet2 (Portal):**

    *   In the Azure Portal, navigate to the `iatd_labs_07_vnet1` Virtual Network (Region 1).
    *   In the Settings blade, click on "Peerings".
    *   Click "+ Add".
        *   **Peering link name:** `iatd_labs_07_peering1`.
        *   **Traffic to remote virtual network:**
            *   **Virtual network deployment model:** "Resource Manager".
            *   **Subscription:** Your Subscription.
            *   **Virtual network:** Select `iatd_labs_07_vnet2`.
            *   **Allow virtual network access:** Select "Enabled".
            *   **Allow forwarded traffic:** Select "Enabled".
            *   **Allow gateway transit:** Select "Disabled".
            *   **Use remote gateways:** Select "Disabled".
        *   Click "Add".

2.  **Create VNet Peering from VNet2 to VNet1 (CLI):**

    *   Open Cloud Shell and select **Bash**.

    ```bash
    az network vnet peering create \
      --name iatd_labs_07_peering2 \
      --resource-group $resourceGroupName2 \
      --vnet-name $vnet2Name \
      --remote-vnet /subscriptions/$(az account show --query id --output tsv)/resourceGroups/$resourceGroupName1/providers/Microsoft.Network/virtualNetworks/$vnet1Name \
      --allow-vnet-access true \
      --allow-forwarded-traffic true \
      --allow-gateway-transit false \
      --use-remote-gateways false
    ```

    Expected Output:
    ```json
    {
      "allowForwardedTraffic": true,
      "allowGatewayTransit": false,
      "allowVirtualNetworkAccess": true,
      "name": "iatd_labs_07_peering2",
      "peeringState": "Connected",
      "provisioningState": "Succeeded",
      "remoteVirtualNetwork": {
        "id": "/subscriptions/your-subscription-id/resourceGroups/iatd_labs_07_rg1/providers/Microsoft.Network/virtualNetworks/iatd_labs_07_vnet1"
      },
      "useRemoteGateways": false
    }
    ```

### Post-Lab: Cleanup

To avoid unnecessary costs, clean up the created resources using any of these methods:

1.  **Azure Portal:**
    * Navigate to Resource Groups
    * Delete both resource groups:
      * Select `iatd_labs_07_rg1` and delete
      * Select `iatd_labs_07_rg2` and delete
    * Type each resource group name to confirm
    * Click 'Delete'

2.  **PowerShell:**
    ```powershell
    Remove-AzResourceGroup -Name $resourceGroupName1 -Force
    Remove-AzResourceGroup -Name $resourceGroupName2 -Force
    ```
    Expected Output:
    ```
    True
    True
    ```

3.  **Azure CLI:**
    ```bash
    az group delete --name $resourceGroupName1 --yes
    az group delete --name $resourceGroupName2 --yes
    ```
    Expected Output:
    ```
    No output indicates successful deletion
    ```

Verify Cleanup:
* Check the Azure Portal to ensure both resource groups and all resources are deleted
* Run `Get-AzResourceGroup` or `az group list` to verify the resource groups no longer exist

### Post-Lab Summary

**Key Takeaways:**
* Global VNet peering enables direct communication between VNets in different regions
* Cross-region communication has higher latency compared to intra-region communication
* Both PowerShell and Azure CLI can be used to manage cross-region networking
* Resource naming and IP addressing should follow consistent conventions

**Next Steps:**
* Explore hub-spoke network topologies across regions
* Learn about Virtual WAN for more complex global networking
* Study network performance optimization techniques
* Practice implementing network security in multi-region deployments

**Additional Resources:**
* [Global VNet Peering Documentation](https://docs.microsoft.com/en-us/azure/virtual-network/virtual-network-peering-overview)
* [Cross-Region Networking Best Practices](https://docs.microsoft.com/en-us/azure/cloud-adoption-framework/ready/azure-best-practices/connectivity-to-azure)
* [Azure Networking Limits](https://docs.microsoft.com/en-us/azure/azure-resource-manager/management/azure-subscription-service-limits#networking-limits)

**Congratulations!** You've successfully configured cross-region virtual network peering and tested connectivity between VMs in different Azure regions.
      --allow-vnet-access
    ```

3.  **Verify Peering Status (PowerShell):**

    *   Open Cloud Shell and select **PowerShell**.

    ```powershell
    Get-AzVirtualNetworkPeering -ResourceGroupName $resourceGroupName1 -VirtualNetworkName $vnet1Name
    Get-AzVirtualNetworkPeering -ResourceGroupName $resourceGroupName2 -VirtualNetworkName $vnet2Name
    ```

    **Example Output (truncated):**

    ```
    #Example Output for VNet1
    Name                         : iatd_labs_07_peering1
    ResourceGroupName            : iatd_labs_07_rg1
    VirtualNetworkName           : iatd_labs_07_vnet1
    RemoteVirtualNetworkName     : iatd_labs_07_vnet2
    AllowVirtualNetworkAccess    : True
    AllowForwardedTraffic        : True
    AllowGatewayTransit          : False
    UseRemoteGateways            : False
    PeeringState                 : Connected
    ...

    #Example Output for VNet2
    Name                         : iatd_labs_07_peering2
    ResourceGroupName            : iatd_labs_07_rg2
    VirtualNetworkName           : iatd_labs_07_vnet2
    RemoteVirtualNetworkName     : iatd_labs_07_vnet1
    AllowVirtualNetworkAccess    : True
    AllowForwardedTraffic        : True
    AllowGatewayTransit          : False
    UseRemoteGateways            : False
    PeeringState                 : Connected
    ...
    ```

### Part 4: Testing Connectivity

1.  **Get VM Private IPs and Test Connectivity:**

    ```powershell
    # Get VM Private IPs
    $vm1PrivateIP = (Get-AzNetworkInterface -ResourceGroupName $resourceGroupName1 | Where-Object { $_.Name -like "*vm1*" }).IpConfigurations[0].PrivateIpAddress
    $vm2PrivateIP = (Get-AzNetworkInterface -ResourceGroupName $resourceGroupName2 | Where-Object { $_.Name -like "*vm2*" }).IpConfigurations[0].PrivateIpAddress
    Write-Host "VM1 Private IP: $vm1PrivateIP"
    Write-Host "VM2 Private IP: $vm2PrivateIP"
    ```

    Expected Output:
    ```
    VM1 Private IP: 172.16.1.4
    VM2 Private IP: 172.16.2.4
    ```

    **Test Cross-Region Connectivity:**
    ```bash
    # From VM1, ping VM2
    ssh cloudadmin@$vm1PublicIP "ping -c 4 $vm2PrivateIP"
    ```

    Expected Output:
    ```
    PING 172.16.2.4 (172.16.2.4) 56(84) bytes of data.
    64 bytes from 172.16.2.4: icmp_seq=1 ttl=64 time=45.2 ms
    64 bytes from 172.16.2.4: icmp_seq=2 ttl=64 time=44.8 ms
    64 bytes from 172.16.2.4: icmp_seq=3 ttl=64 time=44.9 ms
    64 bytes from 172.16.2.4: icmp_seq=4 ttl=64 time=45.1 ms

    --- 172.16.2.4 ping statistics ---
    4 packets transmitted, 4 received, 0% packet loss, time 3005ms
    rtt min/avg/max/mdev = 44.8/45.0/45.2/0.2 ms
    ```

    Note: The higher latency (around 45ms) is normal for cross-region communication.
    $vm1 = Get-AzVM -Name $vm1Name -ResourceGroupName $resourceGroupName1
    $vm2 = Get-AzVM -Name $vm2Name -ResourceGroupName $resourceGroupName2
    $vm1PrivateIP = ($vm1.PrivateIPAddresses)[0]
    $vm2PrivateIP = ($vm2.PrivateIPAddresses)[0]

    Write-Host "VM1 Private IP: $vm1PrivateIP"
    Write-Host "VM2 Private IP: $vm2PrivateIP"
    ```

2.  **Test Connectivity (ICMP/Ping) from VNet1 to VNet2 (PowerShell):**  From Cloud Shell, use `Test-NetConnection`.

    ```powershell
    Test-NetConnection -ComputerName $vm2PrivateIP -Port 22
    ```

    **Expected Result**: Should show the TCP test successful. This verifies the connectivity between the VMs.

    **Test Connectivity (ICMP/Ping):**
    ```powershell
    Test-NetConnection -ComputerName $vm2PrivateIP -Port 22
    ```

    **Expected Result**: Should show the TCP test successful. This verifies the connectivity between the VMs.

3.  **Test Connectivity (ICMP/Ping) from VNet2 to VNet1 (PowerShell):**  From Cloud Shell, use `Test-NetConnection`.

    ```powershell
    Test-NetConnection -ComputerName $vm1PrivateIP -Port 22
    ```

    **Expected Result**: Should show the TCP test successful. This verifies the connectivity between the VMs.

### Post-Lab: Cleanup

To avoid unnecessary costs, clean up the created resources.

1.  **Delete the Resource Groups (PowerShell):**

    ```powershell
    Remove-AzResourceGroup -Name $resourceGroupName1 -Force
    Remove-AzResourceGroup -Name $resourceGroupName2 -Force
    ```

**Congratulations!** You've successfully configured cross-region virtual network peering and verified connectivity between the two regions. Remember to consider latency and data transfer costs when designing cross-region solutions.
