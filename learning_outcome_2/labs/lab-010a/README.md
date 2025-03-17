## IATD Microcredential Cloud Networking: Lab 10a - BGP Path Attributes and Route Selection

**Objective:** To understand BGP path attributes and how they influence route selection in BGP.

**Estimated Time:** 45 - 60 minutes

**Prerequisites:**

1. **Azure Subscription:** An active Azure subscription.
2. **Azure Cloud Shell:** Access to Azure Cloud Shell (Bash or PowerShell).
3. **Basic Understanding:** Familiarity with BGP concepts covered in Labs 9a and 9b.

**Let's get started!**

### Lab Conventions

* **Azure Cloud Shell:** All PowerShell interactions will occur within the Azure Cloud Shell environment.
* **Naming Conventions:** Resource names follow the convention `iatd_labs_10a_*`.
* **IP Address Range:** The `172.16.x.x` range will be used.
* **Location:** Choose a consistent Azure region (e.g., `australiaeast`).

#### Resource Naming

* **Resource Group:** `iatd_labs_10a_rg`
* **Virtual Network AS1:** `iatd_labs_10a_vnet_as1`
* **Virtual Network AS2:** `iatd_labs_10a_vnet_as2`
* **Virtual Network AS3:** `iatd_labs_10a_vnet_as3`
* **Subnet:** `iatd_labs_10a_subnet`
* **VM AS1:** `iatd_labs_10a_vm_as1`
* **VM AS2:** `iatd_labs_10a_vm_as2`
* **VM AS3:** `iatd_labs_10a_vm_as3`

### Part 1: Setting up the Environment

1. **Open Cloud Shell:** Open Azure Cloud Shell and select **PowerShell**.

2. **Define Variables:**

   ```powershell
   $RESOURCE_GROUP = "iatd_labs_10a_rg"
   $LOCATION = "australiaeast" # Replace with your preferred region
   $VNET_NAME_AS1 = "iatd_labs_10a_vnet_as1"
   $VNET_NAME_AS2 = "iatd_labs_10a_vnet_as2"
   $VNET_NAME_AS3 = "iatd_labs_10a_vnet_as3"
   $SUBNET_NAME = "iatd_labs_10a_subnet"
   $SUBNET_PREFIX_AS1 = "172.16.1.0/24"
   $SUBNET_PREFIX_AS2 = "172.16.2.0/24"
   $SUBNET_PREFIX_AS3 = "172.16.3.0/24"
   $VM_AS1_NAME = "iatd_labs_10a_vm_as1"
   $VM_AS2_NAME = "iatd_labs_10a_vm_as2"
   $VM_AS3_NAME = "iatd_labs_10a_vm_as3"
   $ADMIN_USERNAME = "cloudadmin"
   $ADMIN_PASSWORD = "YourStrongPassword123!"  # REPLACE with your password
   ```

3. **Create Resource Group:**

   ```powershell
   New-AzResourceGroup -Name $RESOURCE_GROUP -Location $LOCATION
   ```

   Expected Output:
   ```
   ResourceGroupName : iatd_labs_10a_rg
   Location          : australiaeast
   ProvisioningState : Succeeded
   Tags              : 
   ResourceId        : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/iatd_labs_10a_rg
   ```

4. **Create Virtual Networks and Subnets:**

   ```powershell
   # Create VNet and Subnet for AS1
   $vnetAs1 = New-AzVirtualNetwork -ResourceGroupName $RESOURCE_GROUP -Location $LOCATION -Name $VNET_NAME_AS1 -AddressPrefix "172.16.1.0/24"
   $subnetAs1 = Add-AzVirtualNetworkSubnetConfig -Name $SUBNET_NAME -AddressPrefix $SUBNET_PREFIX_AS1 -VirtualNetwork $vnetAs1
   $vnetAs1 | Set-AzVirtualNetwork
   
   # Create VNet and Subnet for AS2
   $vnetAs2 = New-AzVirtualNetwork -ResourceGroupName $RESOURCE_GROUP -Location $LOCATION -Name $VNET_NAME_AS2 -AddressPrefix "172.16.2.0/24"
   $subnetAs2 = Add-AzVirtualNetworkSubnetConfig -Name $SUBNET_NAME -AddressPrefix $SUBNET_PREFIX_AS2 -VirtualNetwork $vnetAs2
   $vnetAs2 | Set-AzVirtualNetwork
   
   # Create VNet and Subnet for AS3
   $vnetAs3 = New-AzVirtualNetwork -ResourceGroupName $RESOURCE_GROUP -Location $LOCATION -Name $VNET_NAME_AS3 -AddressPrefix "172.16.3.0/24"
   $subnetAs3 = Add-AzVirtualNetworkSubnetConfig -Name $SUBNET_NAME -AddressPrefix $SUBNET_PREFIX_AS3 -VirtualNetwork $vnetAs3
   $vnetAs3 | Set-AzVirtualNetwork
   ```

   Expected Output (for each VNet):
   ```
   Name                   : iatd_labs_10a_vnet_as1
   ResourceGroupName      : iatd_labs_10a_rg
   Location               : australiaeast
   Id                     : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/iatd_labs_10a_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_10a_vnet_as1
   Etag                   : W/"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
   ResourceGuid           : xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
   ProvisioningState      : Succeeded
   Tags                   : 
   AddressSpace           : {172.16.1.0/24}
   DhcpOptions            : {}
   Subnets                : [iatd_labs_10a_subnet]
   VirtualNetworkPeerings : {}
   EnableDdosProtection   : false
   DdosProtectionPlan     : 
   ```

5. **Create VNet Peerings:**

   ```powershell
   # Get the updated VNet objects
   $vnetAs1 = Get-AzVirtualNetwork -Name $VNET_NAME_AS1 -ResourceGroupName $RESOURCE_GROUP
   $vnetAs2 = Get-AzVirtualNetwork -Name $VNET_NAME_AS2 -ResourceGroupName $RESOURCE_GROUP
   $vnetAs3 = Get-AzVirtualNetwork -Name $VNET_NAME_AS3 -ResourceGroupName $RESOURCE_GROUP
   
   # Create peering from AS1 to AS2
   Add-AzVirtualNetworkPeering -Name "AS1-to-AS2" -VirtualNetwork $vnetAs1 -RemoteVirtualNetworkId $vnetAs2.Id -AllowForwardedTraffic
   
   # Create peering from AS2 to AS1
   Add-AzVirtualNetworkPeering -Name "AS2-to-AS1" -VirtualNetwork $vnetAs2 -RemoteVirtualNetworkId $vnetAs1.Id -AllowForwardedTraffic
   
   # Create peering from AS2 to AS3
   Add-AzVirtualNetworkPeering -Name "AS2-to-AS3" -VirtualNetwork $vnetAs2 -RemoteVirtualNetworkId $vnetAs3.Id -AllowForwardedTraffic
   
   # Create peering from AS3 to AS2
   Add-AzVirtualNetworkPeering -Name "AS3-to-AS2" -VirtualNetwork $vnetAs3 -RemoteVirtualNetworkId $vnetAs2.Id -AllowForwardedTraffic
   ```

   Expected Output (for each peering):
   ```
   Name                 : AS1-to-AS2
   Id                   : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/iatd_labs_10a_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_10a_vnet_as1/virtualNetworkPeerings/AS1-to-AS2
   Etag                 : W/"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
   ResourceGroupName    : iatd_labs_10a_rg
   VirtualNetworkName   : iatd_labs_10a_vnet_as1
   ProvisioningState    : Succeeded
   RemoteVirtualNetwork : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/iatd_labs_10a_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_10a_vnet_as2
   AllowVirtualNetworkAccess : True
   AllowForwardedTraffic     : True
   AllowGatewayTransit       : False
   UseRemoteGateways         : False
   RemoteGateways            : null
   RemoteAddressSpace        : {172.16.2.0/24}
   RemoteVirtualNetworkAddressSpace : {172.16.2.0/24}
   ```

6. **Create VMs:**

   ```powershell
   # Create VM in AS1
   New-AzVM -ResourceGroupName $RESOURCE_GROUP -Location $LOCATION -Name $VM_AS1_NAME -VirtualNetworkName $VNET_NAME_AS1 -SubnetName $SUBNET_NAME -Image UbuntuLTS -Size Standard_B1s -Credential (New-Object System.Management.Automation.PSCredential ($ADMIN_USERNAME, (ConvertTo-SecureString $ADMIN_PASSWORD -AsPlainText -Force)))
   
   # Create VM in AS2
   New-AzVM -ResourceGroupName $RESOURCE_GROUP -Location $LOCATION -Name $VM_AS2_NAME -VirtualNetworkName $VNET_NAME_AS2 -SubnetName $SUBNET_NAME -Image UbuntuLTS -Size Standard_B1s -Credential (New-Object System.Management.Automation.PSCredential ($ADMIN_USERNAME, (ConvertTo-SecureString $ADMIN_PASSWORD -AsPlainText -Force)))
   
   # Create VM in AS3
   New-AzVM -ResourceGroupName $RESOURCE_GROUP -Location $LOCATION -Name $VM_AS3_NAME -VirtualNetworkName $VNET_NAME_AS3 -SubnetName $SUBNET_NAME -Image UbuntuLTS -Size Standard_B1s -Credential (New-Object System.Management.Automation.PSCredential ($ADMIN_USERNAME, (ConvertTo-SecureString $ADMIN_PASSWORD -AsPlainText -Force)))
   ```

   Expected Output (for each VM):
   ```
   ResourceGroupName        : iatd_labs_10a_rg
   Id                       : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/iatd_labs_10a_rg/providers/Microsoft.Compute/virtualMachines/iatd_labs_10a_vm_as1
   VmId                     : xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
   Name                     : iatd_labs_10a_vm_as1
   Type                     : Microsoft.Compute/virtualMachines
   Location                 : australiaeast
   LicenseType              : 
   Tags                     : {}
   HardwareProfile          : {VmSize}
   NetworkProfile           : {NetworkInterfaces}
   OSProfile                : {ComputerName, AdminUsername, LinuxConfiguration, Secrets, AllowExtensionOperations, RequireGuestProvisionSignal}
   ProvisioningState        : Succeeded
   StorageProfile           : {ImageReference, OsDisk, DataDisks}
   ```

7. **Get VM Private IPs:**

   ```powershell
   $vm1PrivateIp = (Get-AzNetworkInterface -ResourceGroupName $RESOURCE_GROUP | Where-Object {$_.Name -like "*$VM_AS1_NAME*"}).IpConfigurations[0].PrivateIpAddress
   $vm2PrivateIp = (Get-AzNetworkInterface -ResourceGroupName $RESOURCE_GROUP | Where-Object {$_.Name -like "*$VM_AS2_NAME*"}).IpConfigurations[0].PrivateIpAddress
   $vm3PrivateIp = (Get-AzNetworkInterface -ResourceGroupName $RESOURCE_GROUP | Where-Object {$_.Name -like "*$VM_AS3_NAME*"}).IpConfigurations[0].PrivateIpAddress
   
   Write-Host "VM AS1 Private IP: $vm1PrivateIp"
   Write-Host "VM AS2 Private IP: $vm2PrivateIp"
   Write-Host "VM AS3 Private IP: $vm3PrivateIp"
   ```

   Expected Output:
   ```
   VM AS1 Private IP: 172.16.1.4
   VM AS2 Private IP: 172.16.2.4
   VM AS3 Private IP: 172.16.3.4
   ```

### Part 2: BGP Path Attributes (Conceptual Discussion)

1. **BGP Path Attributes Overview:**

   BGP uses path attributes to determine the best route to a destination. These attributes are used in a specific order to select the best path when multiple paths to the same destination are available.

   The main BGP path attributes, in order of precedence, are:

   * **Weight (Cisco-specific):** Higher weight is preferred. This is a local attribute not advertised to peers.
   * **Local Preference:** Higher local preference is preferred. This is used within an AS to influence outbound traffic.
   * **Locally Originated:** Routes originated by the local router are preferred.
   * **AS Path Length:** Shorter AS path is preferred. Each AS that a route passes through adds its ASN to the AS path.
   * **Origin Type:** IGP (i) is preferred over EGP (e), which is preferred over Incomplete (?).
   * **MED (Multi-Exit Discriminator):** Lower MED is preferred. This is used to influence inbound traffic from neighboring ASes.
   * **eBGP over iBGP:** Routes learned via eBGP are preferred over routes learned via iBGP.
   * **IGP Metric to Next Hop:** Lower IGP metric to the BGP next hop is preferred.
   * **Router ID:** If all else is equal, the route from the BGP router with the lowest Router ID is preferred.

2. **Conceptual Scenario:**

   In our lab environment, we have three VMs representing routers in three different Autonomous Systems:
   * VM_AS1 in AS 65001
   * VM_AS2 in AS 65002
   * VM_AS3 in AS 65003

   The network topology is as follows:
   * AS1 (VM_AS1) is connected to AS2 (VM_AS2)
   * AS2 (VM_AS2) is connected to AS3 (VM_AS3)
   * AS1 (VM_AS1) is not directly connected to AS3 (VM_AS3)

   This means that for VM_AS1 to reach VM_AS3, it must go through VM_AS2, resulting in an AS path of 65001-65002-65003.

### Part 3: Simulating BGP Path Selection (Conceptual Exercise)

1. **Scenario 1: AS Path Length**

   * **Setup:** Imagine we have two possible paths from AS1 to a destination network in AS4:
     * Path 1: AS1 → AS2 → AS4 (AS Path: 65001-65002-65004)
     * Path 2: AS1 → AS2 → AS3 → AS4 (AS Path: 65001-65002-65003-65004)
   * **Question:** Which path would BGP select?
   * **Answer:** Path 1, because it has a shorter AS path length (2 hops vs. 3 hops).

2. **Scenario 2: Local Preference**

   * **Setup:** Imagine AS1 receives two routes to the same destination:
     * Route 1: via AS2 with Local Preference 100
     * Route 2: via AS3 with Local Preference 200
   * **Question:** Which route would BGP select?
   * **Answer:** Route 2, because it has a higher Local Preference value.

3. **Scenario 3: MED (Multi-Exit Discriminator)**

   * **Setup:** Imagine AS1 has two connections to AS2 and receives the same route through both connections:
     * Connection 1: with MED value 10
     * Connection 2: with MED value 5
   * **Question:** Which connection would BGP prefer for sending traffic to AS2?
   * **Answer:** Connection 2, because it has a lower MED value.

4. **Scenario 4: Origin Type**

   * **Setup:** Imagine AS1 receives two routes to the same destination:
     * Route 1: with Origin Type IGP (i)
     * Route 2: with Origin Type EGP (e)
   * **Question:** Which route would BGP select?
   * **Answer:** Route 1, because IGP origin is preferred over EGP origin.

5. **Scenario 5: eBGP vs. iBGP**

   * **Setup:** Imagine a router in AS1 receives two routes to the same destination:
     * Route 1: via eBGP from AS2
     * Route 2: via iBGP from another router in AS1
   * **Question:** Which route would BGP select?
   * **Answer:** Route 1, because eBGP routes are preferred over iBGP routes.

### Part 4: Verifying Connectivity in Our Lab Environment

1. **Get VM Public IPs:**

   ```powershell
   $vm1PublicIp = (Get-AzPublicIpAddress -ResourceGroupName $RESOURCE_GROUP | Where-Object {$_.Name -like "*$VM_AS1_NAME*"}).IpAddress
   $vm2PublicIp = (Get-AzPublicIpAddress -ResourceGroupName $RESOURCE_GROUP | Where-Object {$_.Name -like "*$VM_AS2_NAME*"}).IpAddress
   $vm3PublicIp = (Get-AzPublicIpAddress -ResourceGroupName $RESOURCE_GROUP | Where-Object {$_.Name -like "*$VM_AS3_NAME*"}).IpAddress
   
   Write-Host "VM AS1 Public IP: $vm1PublicIp"
   Write-Host "VM AS2 Public IP: $vm2PublicIp"
   Write-Host "VM AS3 Public IP: $vm3PublicIp"
   ```

   Expected Output:
   ```
   VM AS1 Public IP: 20.1.2.3
   VM AS2 Public IP: 20.1.2.4
   VM AS3 Public IP: 20.1.2.5
   ```

2. **Connect to VM_AS1 and Test Connectivity:**

   ```bash
   ssh cloudadmin@<VM_AS1_PUBLIC_IP>
   ```

   Once connected, test connectivity to VM_AS2 and VM_AS3:

   ```bash
   ping <VM_AS2_PRIVATE_IP> -c 4
   ping <VM_AS3_PRIVATE_IP> -c 4
   ```

   Expected Output for VM_AS2:
   ```
   PING 172.16.2.4 (172.16.2.4) 56(84) bytes of data.
   64 bytes from 172.16.2.4: icmp_seq=1 ttl=64 time=0.935 ms
   64 bytes from 172.16.2.4: icmp_seq=2 ttl=64 time=0.652 ms
   64 bytes from 172.16.2.4: icmp_seq=3 ttl=64 time=0.495 ms
   64 bytes from 172.16.2.4: icmp_seq=4 ttl=64 time=0.428 ms
   ```

   Expected Output for VM_AS3:
   ```
   PING 172.16.3.4 (172.16.3.4) 56(84) bytes of data.
   64 bytes from 172.16.3.4: icmp_seq=1 ttl=63 time=1.935 ms
   64 bytes from 172.16.3.4: icmp_seq=2 ttl=63 time=1.652 ms
   64 bytes from 172.16.3.4: icmp_seq=3 ttl=63 time=1.495 ms
   64 bytes from 172.16.3.4: icmp_seq=4 ttl=63 time=1.428 ms
   ```

   Note the difference in TTL values: TTL=64 for VM_AS2 (direct connection) and TTL=63 for VM_AS3 (connection through VM_AS2).

3. **Trace Route to Verify Path:**

   ```bash
   traceroute <VM_AS3_PRIVATE_IP>
   ```

   Expected Output:
   ```
   traceroute to 172.16.3.4 (172.16.3.4), 30 hops max, 60 byte packets
    1  172.16.2.4 (172.16.2.4)  0.935 ms  0.652 ms  0.495 ms
    2  172.16.3.4 (172.16.3.4)  1.935 ms  1.652 ms  1.495 ms
   ```

   This confirms that traffic from VM_AS1 to VM_AS3 goes through VM_AS2.

### Part 5: Cleanup

1. **Delete Resource Group (PowerShell):**

   ```powershell
   Remove-AzResourceGroup -Name $RESOURCE_GROUP -Force
   ```

   Expected Output:
   ```
   True
   ```

2. **Verify Deletion (PowerShell):**

   ```powershell
   Get-AzResourceGroup -Name $RESOURCE_GROUP -ErrorVariable notPresent -ErrorAction SilentlyContinue
   if ($notPresent) {
       Write-Host "Resource group $RESOURCE_GROUP has been deleted successfully."
   } else {
       Write-Host "Resource group $RESOURCE_GROUP still exists. Please try deleting it again."
   }
   ```

   Expected Output:
   ```
   Resource group iatd_labs_10a_rg has been deleted successfully.
   ```

### Post-Lab Summary

In this lab, you:
1. Learned about BGP path attributes and their role in route selection
2. Understood the order of precedence for BGP path attributes
3. Explored scenarios demonstrating how different path attributes affect route selection
4. Set up a multi-AS environment and verified connectivity between VMs in different ASes
5. Observed how traffic flows through intermediate ASes when direct connectivity is not available

This lab provided a foundation for understanding how BGP makes routing decisions based on path attributes. In the next lab, you will explore more advanced BGP concepts and configurations.
