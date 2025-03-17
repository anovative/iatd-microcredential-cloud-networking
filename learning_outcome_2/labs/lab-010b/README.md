## IATD Microcredential Cloud Networking: Lab 10b - BGP Route Filtering and Manipulation

**Objective:** To understand BGP route filtering and manipulation techniques, including AS_PATH prepending, route filtering, and route summarization.

**Estimated Time:** 60 - 75 minutes

**Prerequisites:**

1. **Azure Subscription:** An active Azure subscription.
2. **Azure Cloud Shell:** Access to Azure Cloud Shell (Bash or PowerShell).
3. **Basic Understanding:** Familiarity with BGP concepts covered in Labs 9a, 9b, and 10a.

**Let's get started!**

### Part 1: Setting up the Environment

1. **Open Cloud Shell:** Open Azure Cloud Shell and select **PowerShell**.

2. **Define Variables:**

   ```powershell
   $RESOURCE_GROUP = "iatd_labs_10b_rg"
   $LOCATION = "australiaeast" # Replace with your preferred region
   $VNET_NAME_AS1 = "iatd_labs_10b_vnet_as1"
   $VNET_NAME_AS2 = "iatd_labs_10b_vnet_as2"
   $SUBNET_NAME = "iatd_labs_10b_subnet"
   $SUBNET_PREFIX_AS1 = "172.16.1.0/24"
   $SUBNET_PREFIX_AS2 = "172.16.2.0/24"
   $VM_AS1_NAME = "iatd_labs_10b_vm_as1"
   $VM_AS2_NAME = "iatd_labs_10b_vm_as2"
   $ADMIN_USERNAME = "cloudadmin"
   $ADMIN_PASSWORD = "YourStrongPassword123!"  # REPLACE with your password
   ```

3. **Create Resource Group:**

   ```powershell
   New-AzResourceGroup -Name $RESOURCE_GROUP -Location $LOCATION
   ```

   Expected Output:
   ```
   ResourceGroupName : iatd_labs_10b_rg
   Location          : australiaeast
   ProvisioningState : Succeeded
   Tags              : 
   ResourceId        : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/iatd_labs_10b_rg
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
   ```

   Expected Output (for each VNet):
   ```
   Name                   : iatd_labs_10b_vnet_as1
   ResourceGroupName      : iatd_labs_10b_rg
   Location               : australiaeast
   Id                     : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/iatd_labs_10b_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_10b_vnet_as1
   Etag                   : W/"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
   ResourceGuid           : xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
   ProvisioningState      : Succeeded
   Tags                   : 
   AddressSpace           : {172.16.1.0/24}
   DhcpOptions            : {}
   Subnets                : [iatd_labs_10b_subnet]
   VirtualNetworkPeerings : {}
   EnableDdosProtection   : false
   DdosProtectionPlan     : 
   ```

5. **Create VNet Peerings:**

   ```powershell
   # Get the updated VNet objects
   $vnetAs1 = Get-AzVirtualNetwork -Name $VNET_NAME_AS1 -ResourceGroupName $RESOURCE_GROUP
   $vnetAs2 = Get-AzVirtualNetwork -Name $VNET_NAME_AS2 -ResourceGroupName $RESOURCE_GROUP
   
   # Create peering from AS1 to AS2
   Add-AzVirtualNetworkPeering -Name "AS1-to-AS2" -VirtualNetwork $vnetAs1 -RemoteVirtualNetworkId $vnetAs2.Id -AllowForwardedTraffic
   
   # Create peering from AS2 to AS1
   Add-AzVirtualNetworkPeering -Name "AS2-to-AS1" -VirtualNetwork $vnetAs2 -RemoteVirtualNetworkId $vnetAs1.Id -AllowForwardedTraffic
   ```

   Expected Output (for each peering):
   ```
   Name                 : AS1-to-AS2
   Id                   : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/iatd_labs_10b_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_10b_vnet_as1/virtualNetworkPeerings/AS1-to-AS2
   Etag                 : W/"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
   ResourceGroupName    : iatd_labs_10b_rg
   VirtualNetworkName   : iatd_labs_10b_vnet_as1
   ProvisioningState    : Succeeded
   RemoteVirtualNetwork : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/iatd_labs_10b_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_10b_vnet_as2
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
   ```

   Expected Output (for each VM):
   ```
   ResourceGroupName        : iatd_labs_10b_rg
   Id                       : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/iatd_labs_10b_rg/providers/Microsoft.Compute/virtualMachines/iatd_labs_10b_vm_as1
   VmId                     : xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
   Name                     : iatd_labs_10b_vm_as1
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
   
   Write-Host "VM AS1 Private IP: $vm1PrivateIp"
   Write-Host "VM AS2 Private IP: $vm2PrivateIp"
   ```

   Expected Output:
   ```
   VM AS1 Private IP: 172.16.1.4
   VM AS2 Private IP: 172.16.2.4
   ```

### Part 2: AS_PATH Prepending (Conceptual Discussion)

1. **AS_PATH Prepending Overview:**

   AS_PATH prepending is a technique to influence BGP route selection by artificially making a path appear longer. This is done by adding the same AS number multiple times to the AS_PATH attribute.

   Since BGP prefers shorter AS paths, prepending makes a route less preferred, which can be used to influence inbound traffic.

2. **Use Cases for AS_PATH Prepending:**

   * **Traffic Engineering:** Direct traffic away from certain paths to balance load or avoid congestion.
   * **Backup Links:** Make primary links more preferred than backup links.
   * **Provider Selection:** Influence which provider's network is used for inbound traffic.

3. **Example Scenario:**

   * AS1 has two connections to the internet: one through ISP-A and another through ISP-B.
   * AS1 wants to prefer traffic through ISP-A for all destinations.
   * AS1 can prepend its AS number multiple times in the routes advertised to ISP-B, making those routes less preferred.

### Part 3: Simulating AS_PATH Prepending (Conceptual Exercise)

1. **Scenario Setup:**

   * AS1 (65001) and AS2 (65002) are directly connected.
   * AS1 advertises its network (172.16.1.0/24) to AS2.
   * AS2 advertises its network (172.16.2.0/24) to AS1.

2. **Normal Route Advertisement:**

   * When AS1 advertises 172.16.1.0/24 to AS2, the AS_PATH seen by AS2 is "65001".
   * When AS2 advertises 172.16.2.0/24 to AS1, the AS_PATH seen by AS1 is "65002".

3. **With AS_PATH Prepending:**

   * AS1 prepends its AS number twice when advertising to AS2: "65001 65001 65001".
   * This makes the path to AS1's network appear longer (3 hops instead of 1).
   * If AS2 had another path to reach AS1's network with a shorter AS_PATH, it would prefer that path.

4. **Verification:**

   * In a real BGP environment, you would verify the effect of AS_PATH prepending by checking the BGP table on the receiving router.
   * The route with the prepended AS_PATH would be less preferred if there's an alternative route with a shorter AS_PATH.

### Part 4: BGP Route Filtering (Conceptual Discussion)

1. **Route Filtering Overview:**

   BGP route filtering is a mechanism to control which routes are advertised to or received from BGP neighbors. It allows network administrators to implement routing policies and prevent unwanted routes from being propagated.

   Common methods for BGP route filtering include:

   * **Prefix Lists:** Filter routes based on IP prefix and prefix length.
   * **AS Path Access Lists:** Filter routes based on AS path attributes.
   * **Route Maps:** Provide more complex filtering and route manipulation capabilities.
   * **Community Lists:** Filter routes based on BGP community attributes.

2. **Use Cases for Route Filtering:**

   * **Security:** Prevent unauthorized or malicious route advertisements.
   * **Policy Enforcement:** Implement routing policies based on business requirements.
   * **Traffic Engineering:** Control traffic flow by selectively advertising routes.
   * **Resource Optimization:** Reduce the size of routing tables by filtering unnecessary routes.

### Part 5: Route Summarization (Conceptual Discussion)

1. **Route Summarization Overview:**

   Route summarization (also known as route aggregation) is the process of combining multiple specific routes into a single, more general route. This helps reduce the size of routing tables and improves network stability by minimizing the impact of route flaps.

   For example, instead of advertising four separate routes:
   * 172.16.1.0/24
   * 172.16.2.0/24
   * 172.16.3.0/24
   * 172.16.4.0/24

   You can summarize them into a single route: 172.16.0.0/22

2. **Benefits of Route Summarization:**

   * **Reduced Routing Table Size:** Fewer routes mean smaller routing tables and less memory usage.
   * **Improved Convergence:** Fewer routes to process means faster convergence after network changes.
   * **Enhanced Stability:** Route summarization can isolate parts of the network from route flaps in other parts.

3. **Calculating Route Summaries:**

   To calculate a route summary, you need to find the common bits in the network addresses and determine the appropriate prefix length.

   Example: To summarize 172.16.1.0/24 and 172.16.2.0/24
   * Convert to binary: 10101100.00010000.00000001.00000000 and 10101100.00010000.00000010.00000000
   * Find common bits: 10101100.00010000.000000xx.xxxxxxxx (first 22 bits are common)
   * Summary route: 172.16.0.0/22

### Part 6: Route Redistribution (Conceptual Discussion)

1. **Route Redistribution Overview:**

   Route redistribution is the process of importing routes from one routing protocol into another. In the context of BGP, this typically involves redistributing routes from interior gateway protocols (IGPs) like OSPF or EIGRP into BGP, or vice versa.

2. **Use Cases for Route Redistribution:**

   * **Hybrid Networks:** When parts of the network use different routing protocols.
   * **Migration:** During migration from one routing protocol to another.
   * **Connectivity:** To provide connectivity between different routing domains.

3. **Challenges and Best Practices:**

   * **Routing Loops:** Improper redistribution can cause routing loops.
   * **Route Filtering:** Always apply filters when redistributing routes to prevent unwanted routes from being propagated.
   * **Default Route:** Be cautious when redistributing default routes.
   * **Metrics:** Set appropriate metrics for redistributed routes.

### Part 7: Simulating BGP Route Filtering and Manipulation (Conceptual Exercise)

1. **Scenario 1: Route Filtering with Prefix Lists**

   * **Setup:** Imagine AS1 wants to filter routes from AS2 to only accept routes in the 172.16.2.0/24 range and reject all others.
   * **Configuration (Conceptual):**
     * Create a prefix list that permits 172.16.2.0/24 and denies all others.
     * Apply the prefix list to inbound routes from AS2.
   * **Result:** AS1 will only accept the 172.16.2.0/24 route from AS2 and reject all others.

2. **Scenario 2: Route Summarization**

   * **Setup:** Imagine AS1 has multiple subnets (172.16.1.0/24, 172.16.2.0/24, 172.16.3.0/24, 172.16.4.0/24) and wants to advertise a summary route to AS2.
   * **Configuration (Conceptual):**
     * Configure AS1 to summarize these routes as 172.16.0.0/22.
     * Advertise only the summary route to AS2.
   * **Result:** AS2 will receive a single route (172.16.0.0/22) instead of four separate routes.

3. **Scenario 3: Route Redistribution**

   * **Setup:** Imagine AS1 is running OSPF internally and wants to redistribute OSPF routes into BGP to advertise them to AS2.
   * **Configuration (Conceptual):**
     * Configure AS1 to redistribute OSPF routes into BGP.
     * Apply a route map to filter and set attributes for redistributed routes.
   * **Result:** OSPF routes from AS1 will be advertised to AS2 via BGP.

### Part 8: Verifying Connectivity in Our Lab Environment

1. **Get VM Public IPs:**

   ```powershell
   $vm1PublicIp = (Get-AzPublicIpAddress -ResourceGroupName $RESOURCE_GROUP | Where-Object {$_.Name -like "*$VM_AS1_NAME*"}).IpAddress
   $vm2PublicIp = (Get-AzPublicIpAddress -ResourceGroupName $RESOURCE_GROUP | Where-Object {$_.Name -like "*$VM_AS2_NAME*"}).IpAddress
   
   Write-Host "VM AS1 Public IP: $vm1PublicIp"
   Write-Host "VM AS2 Public IP: $vm2PublicIp"
   ```

   Expected Output:
   ```
   VM AS1 Public IP: 20.1.2.3
   VM AS2 Public IP: 20.1.2.4
   ```

2. **Connect to VM_AS1 and Test Connectivity:**

   ```bash
   ssh cloudadmin@<VM_AS1_PUBLIC_IP>
   ```

   Once connected, test connectivity to VM_AS2:

   ```bash
   ping <VM_AS2_PRIVATE_IP> -c 4
   ```

   Expected Output:
   ```
   PING 172.16.2.4 (172.16.2.4) 56(84) bytes of data.
   64 bytes from 172.16.2.4: icmp_seq=1 ttl=64 time=0.935 ms
   64 bytes from 172.16.2.4: icmp_seq=2 ttl=64 time=0.652 ms
   64 bytes from 172.16.2.4: icmp_seq=3 ttl=64 time=0.495 ms
   64 bytes from 172.16.2.4: icmp_seq=4 ttl=64 time=0.428 ms
   ```

3. **Trace Route to Verify Path:**

   ```bash
   traceroute <VM_AS2_PRIVATE_IP>
   ```

   Expected Output:
   ```
   traceroute to 172.16.2.4 (172.16.2.4), 30 hops max, 60 byte packets
    1  172.16.2.4 (172.16.2.4)  0.935 ms  0.652 ms  0.495 ms
   ```

### Part 9: Cleanup

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
   Resource group iatd_labs_10b_rg has been deleted successfully.
   ```

### Post-Lab Summary

In this lab, you:
1. Learned about BGP route filtering techniques and their applications
2. Understood the concept of route summarization and its benefits
3. Explored route redistribution between different routing protocols
4. Applied these concepts in simulated scenarios
5. Set up a lab environment with two VMs representing different Autonomous Systems
6. Verified connectivity between the VMs
7. Learned about AS_PATH prepending and how it can be used to influence BGP route selection

These advanced BGP concepts are essential for designing and managing complex networks with multiple routing domains and policies. In the next lab, you will explore Azure ExpressRoute and how it integrates with BGP for hybrid connectivity.
