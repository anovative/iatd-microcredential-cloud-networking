## IATD Microcredential Cloud Networking: Lab 11b - Advanced ExpressRoute BGP Configuration

**Objective:** Explore advanced BGP configuration options with Azure ExpressRoute, including route filtering, route maps, and BGP communities.

**Estimated Time:** 75 - 90 minutes

**Prerequisites:**

1. **Azure Subscription:** An active Azure subscription.
2. **Azure Cloud Shell:** Access to Azure Cloud Shell (PowerShell).
3. **Existing ExpressRoute Circuit:** A functional Azure ExpressRoute circuit with private or Microsoft peering configured. This is an external resource that will not be created as part of this lab.
4. **Completion of Lab 11a:** Understanding of basic ExpressRoute BGP configuration.
5. **On-premises Router Access:** Administrative access to the on-premises router connected to ExpressRoute.

**Let's get started!**

### Lab Conventions

* **Azure Cloud Shell:** All PowerShell interactions will occur within the Azure Cloud Shell environment.
* **Naming Conventions:** Resource names follow the convention `iatd_labs_11b_*`.
* **IP Address Range:** The `172.16.x.x` range will be used.
* **Location:** Choose a consistent Azure region (e.g., `australiaeast`).

#### Resource Naming

* **Resource Group:** `iatd_labs_11b_rg`
* **ExpressRoute Circuit:** Your existing ExpressRoute circuit
* **Virtual Network:** `iatd_labs_11b_vnet`
* **Subnet:** `iatd_labs_11b_subnet`
* **Test VM:** `iatd_labs_11b_vm` (if needed)
* **Test VM Public IP:** `iatd_labs_11b_vm_pip` (if needed)

### Part 1: Setting up the Environment

1. **Open Cloud Shell:** Open Azure Cloud Shell and select **PowerShell**.

2. **Define Variables:**

   ```powershell
   $RESOURCE_GROUP = "iatd_labs_11b_rg"
   $LOCATION = "australiaeast" # Replace with your preferred region
   $VNET_NAME = "iatd_labs_11b_vnet"
   $SUBNET_NAME = "iatd_labs_11b_subnet"
   $SUBNET_PREFIX = "172.16.1.0/24"
   $EXPRESSROUTE_CIRCUIT_NAME = "your-expressroute-circuit" # Replace with your ExpressRoute circuit name
   $EXPRESSROUTE_CIRCUIT_RG = "your-expressroute-circuit-rg" # Replace with the resource group of your ExpressRoute circuit
   ```

3. **Create Resource Group:**

   ```powershell
   New-AzResourceGroup -Name $RESOURCE_GROUP -Location $LOCATION
   ```

   Expected Output:
   ```
   ResourceGroupName : iatd_labs_11b_rg
   Location          : australiaeast
   ProvisioningState : Succeeded
   Tags              : 
   ResourceId        : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/iatd_labs_11b_rg
   ```

4. **Create Virtual Network and Subnet:**

   ```powershell
   $vnet = New-AzVirtualNetwork -ResourceGroupName $RESOURCE_GROUP -Location $LOCATION -Name $VNET_NAME -AddressPrefix "172.16.0.0/16"
   $subnet = Add-AzVirtualNetworkSubnetConfig -Name $SUBNET_NAME -AddressPrefix $SUBNET_PREFIX -VirtualNetwork $vnet
   $vnet | Set-AzVirtualNetwork
   ```

   Expected Output:
   ```
   Name                   : iatd_labs_11b_vnet
   ResourceGroupName      : iatd_labs_11b_rg
   Location               : australiaeast
   Id                     : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/iatd_labs_11b_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_11b_vnet
   Etag                   : W/"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
   ResourceGuid           : xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
   ProvisioningState      : Succeeded
   Tags                   : 
   AddressSpace           : {172.16.0.0/16}
   DhcpOptions            : {}
   Subnets                : [iatd_labs_11b_subnet]
   VirtualNetworkPeerings : {}
   EnableDdosProtection   : false
   DdosProtectionPlan     : 
   ```

5. **Get ExpressRoute Circuit Information:**

   ```powershell
   $circuit = Get-AzExpressRouteCircuit -Name $EXPRESSROUTE_CIRCUIT_NAME -ResourceGroupName $EXPRESSROUTE_CIRCUIT_RG
   ```

   Expected Output (partial):
   ```
   Name                             : your-expressroute-circuit
   ResourceGroupName                : your-expressroute-circuit-rg
   Location                         : australiaeast
   Id                               : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/your-expressroute-circuit-rg/providers/Microsoft.Network/expressRouteCircuits/your-expressroute-circuit
   Etag                             : W/"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
   ProvisioningState                : Succeeded
   ```

### Part 2: Understanding BGP Communities in Azure

1. **Azure BGP Communities Overview:**

   Azure uses BGP communities to categorize routes based on their origin and service type. These communities help in implementing more granular routing policies.

   Key Azure BGP Communities:
   * **12076:50001** - Routes from all Azure regions (public regions)
   * **12076:50002** - Routes from all Microsoft Edge locations
   * **12076:50003** - Routes from all Azure regions (public and Azure Government)
   * **12076:51001** - Routes for Azure ExpressRoute
   * **12076:52001** - Routes for Azure Storage
   * **12076:52002** - Routes for Azure SQL
   * **12076:52003** - Routes for Azure Websites
   * **12076:52004** - Routes for Azure Virtual Machines

2. **View BGP Communities for Your ExpressRoute Circuit:**

   ```powershell
   $peerConfig = Get-AzExpressRouteCircuitPeeringConfig -ExpressRouteCircuit $circuit -Name "AzurePrivatePeering"
   $peerConfig.Communities
   ```

   Expected Output (example):
   ```
   12076:50001
   12076:51001
   ```

### Part 3: Configuring Route Filters for Microsoft Peering

1. **Create a Route Filter:**

   ```powershell
   $routeFilter = New-AzRouteFilter -Name "iatd_labs_11b_route_filter" -ResourceGroupName $RESOURCE_GROUP -Location $LOCATION
   ```

   Expected Output:
   ```
   Name                : iatd_labs_11b_route_filter
   ResourceGroupName   : iatd_labs_11b_rg
   Location            : australiaeast
   Id                  : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/iatd_labs_11b_rg/providers/Microsoft.Network/routeFilters/iatd_labs_11b_route_filter
   Etag                : W/"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
   ProvisioningState   : Succeeded
   Rules               : []
   PeeredVirtualNetworks : []
   ```

2. **Add a Rule to the Route Filter:**

   ```powershell
   Add-AzRouteFilterRuleConfig -Name "allow-azure-storage" -RouteFilter $routeFilter -Access Allow -RouteFilterRuleType Community -CommunityValue "12076:52001"
   $routeFilter | Set-AzRouteFilter
   ```

   Expected Output:
   ```
   Name                : iatd_labs_11b_route_filter
   ResourceGroupName   : iatd_labs_11b_rg
   Location            : australiaeast
   Id                  : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/iatd_labs_11b_rg/providers/Microsoft.Network/routeFilters/iatd_labs_11b_route_filter
   Etag                : W/"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
   ProvisioningState   : Succeeded
   Rules               : [allow-azure-storage]
   PeeredVirtualNetworks : []
   ```

3. **Apply Route Filter to Microsoft Peering (if available):**

   ```powershell
   $microsoftPeering = Get-AzExpressRouteCircuitPeeringConfig -ExpressRouteCircuit $circuit -Name "MicrosoftPeering"
   $microsoftPeering.RouteFilter = $routeFilter
   Set-AzExpressRouteCircuit -ExpressRouteCircuit $circuit
   ```

   Expected Output (partial):
   ```
   Name                             : your-expressroute-circuit
   ResourceGroupName                : your-expressroute-circuit-rg
   Location                         : australiaeast
   ProvisioningState                : Succeeded
   Peerings                         : [AzurePrivatePeering, MicrosoftPeering]
   ```

### Part 4: Advanced BGP Configuration on the On-premises Router (Conceptual)

1. **Route Filtering with AS Path Access Lists:**

   On your on-premises router, you can configure AS path access lists to filter routes based on the AS path attribute. This is useful for controlling which routes are accepted from Azure.

   Example configuration (Cisco IOS syntax):
   ```
   ip as-path access-list 10 permit ^$
   ip as-path access-list 10 permit _12076_
   
   router bgp 65001
    neighbor 172.16.0.1 remote-as 12076
    neighbor 172.16.0.1 filter-list 10 in
   ```

   This configuration accepts routes that originate from your AS (^$) or pass through Microsoft's AS (12076).

2. **Route Maps for BGP Policy:**

   Route maps provide more granular control over BGP routing policies. You can use them to modify BGP attributes or to filter routes based on multiple criteria.

   Example configuration (Cisco IOS syntax):
   ```
   ip prefix-list AZURE_SUBNETS seq 10 permit 172.16.0.0/16
   
   route-map AZURE_IN permit 10
    match ip address prefix-list AZURE_SUBNETS
    set local-preference 200
   
   router bgp 65001
    neighbor 172.16.0.1 remote-as 12076
    neighbor 172.16.0.1 route-map AZURE_IN in
   ```

   This configuration increases the local preference for routes matching the Azure subnet prefix list.

3. **BGP Communities for Route Tagging:**

   You can use BGP communities to tag routes with specific attributes, making it easier to apply consistent policies across your network.

   Example configuration (Cisco IOS syntax):
   ```
   ip community-list 100 permit 12076:52001
   
   route-map AZURE_STORAGE permit 10
    match community 100
    set local-preference 300
   
   router bgp 65001
    neighbor 172.16.0.1 remote-as 12076
    neighbor 172.16.0.1 route-map AZURE_STORAGE in
   ```

   This configuration increases the local preference for Azure Storage routes.

### Part 5: Creating a Virtual Network Gateway Connection to ExpressRoute

1. **Create Gateway Subnet:**

   ```powershell
   $vnet = Get-AzVirtualNetwork -Name $VNET_NAME -ResourceGroupName $RESOURCE_GROUP
   Add-AzVirtualNetworkSubnetConfig -Name "GatewaySubnet" -AddressPrefix "172.16.255.0/27" -VirtualNetwork $vnet
   $vnet | Set-AzVirtualNetwork
   ```

   Expected Output (partial):
   ```
   Name                   : iatd_labs_11b_vnet
   ResourceGroupName      : iatd_labs_11b_rg
   Location               : australiaeast
   Subnets                : [iatd_labs_11b_subnet, GatewaySubnet]
   ```

2. **Create Virtual Network Gateway:**

   ```powershell
   $gwpip = New-AzPublicIpAddress -Name "iatd_labs_11b_gw_pip" -ResourceGroupName $RESOURCE_GROUP -Location $LOCATION -AllocationMethod Dynamic
   $subnet = Get-AzVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet
   $gwconfig = New-AzVirtualNetworkGatewayIpConfig -Name "gwipconfig" -SubnetId $subnet.Id -PublicIpAddressId $gwpip.Id
   
   New-AzVirtualNetworkGateway -Name "iatd_labs_11b_gw" -ResourceGroupName $RESOURCE_GROUP -Location $LOCATION -IpConfigurations $gwconfig -GatewayType "ExpressRoute" -GatewaySku "Standard"
   ```

   Expected Output (partial):
   ```
   Name                   : iatd_labs_11b_gw
   ResourceGroupName      : iatd_labs_11b_rg
   Location               : australiaeast
   GatewayType            : ExpressRoute
   VpnType                : 
   EnableBgp              : False
   GatewaySku             : Standard
   VpnClientConfiguration : 
   BgpSettings            : 
   CustomRoutes           : 
   ResourceGuid           : xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
   ProvisioningState      : Succeeded
   ```

   Note: This step may take 30-45 minutes to complete.

3. **Connect Virtual Network to ExpressRoute Circuit:**

   ```powershell
   $gw = Get-AzVirtualNetworkGateway -Name "iatd_labs_11b_gw" -ResourceGroupName $RESOURCE_GROUP
   $circuit = Get-AzExpressRouteCircuit -Name $EXPRESSROUTE_CIRCUIT_NAME -ResourceGroupName $EXPRESSROUTE_CIRCUIT_RG
   
   New-AzVirtualNetworkGatewayConnection -Name "iatd_labs_11b_connection" -ResourceGroupName $RESOURCE_GROUP -Location $LOCATION -VirtualNetworkGateway1 $gw -PeerId $circuit.Id -ConnectionType ExpressRoute -AuthorizationKey $null
   ```

   Expected Output (partial):
   ```
   Name                   : iatd_labs_11b_connection
   ResourceGroupName      : iatd_labs_11b_rg
   Location               : australiaeast
   ProvisioningState      : Succeeded
   ConnectionType         : ExpressRoute
   RoutingWeight          : 0
   SharedKey              : 
   ```

### Part 6: Verifying Connectivity and BGP Routes

1. **Check Connection Status:**

   ```powershell
   Get-AzVirtualNetworkGatewayConnection -Name "iatd_labs_11b_connection" -ResourceGroupName $RESOURCE_GROUP
   ```

   Expected Output (partial):
   ```
   Name                   : iatd_labs_11b_connection
   ResourceGroupName      : iatd_labs_11b_rg
   Location               : australiaeast
   ProvisioningState      : Succeeded
   ConnectionStatus       : Connected
   EgressBytesTransferred : 1024
   IngressBytesTransferred: 2048
   ```

2. **View Learned BGP Routes (if BGP is enabled on the gateway):**

   ```powershell
   Get-AzVirtualNetworkGatewayLearnedRoute -VirtualNetworkGatewayName "iatd_labs_11b_gw" -ResourceGroupName $RESOURCE_GROUP
   ```

   Expected Output (example):
   ```
   Network            NextHop       SourcePeer    Origin    AsPath                Weight
   -------            -------       ----------    ------    ------                ------
   10.0.0.0/16        172.16.0.1    172.16.0.1    EBgp      65001                 32768
   172.16.0.0/16      10.0.0.1      10.0.0.1      IBgp      12076                 32768
   ```

3. **View Advertised BGP Routes:**

   ```powershell
   Get-AzVirtualNetworkGatewayAdvertisedRoute -VirtualNetworkGatewayName "iatd_labs_11b_gw" -ResourceGroupName $RESOURCE_GROUP -Peer 172.16.0.1
   ```

   Expected Output (example):
   ```
   Network            NextHop       SourcePeer    Origin    AsPath                Weight
   -------            -------       ----------    ------    ------                ------
   172.16.0.0/16      10.0.0.1      10.0.0.1      Igp       12076                 0
   ```

### Part 7: Clean Up Resources

1. **Delete Route Filter (if created):**

   ```powershell
   # Remove the route filter if you created one
   Remove-AzRouteFilter -Name "iatd_labs_11b_routefilter" -ResourceGroupName $RESOURCE_GROUP -Force
   ```

2. **Delete Virtual Network Gateway (if created):**

   ```powershell
   # Remove the virtual network gateway if you created one
   Remove-AzVirtualNetworkGateway -Name "iatd_labs_11b_gw" -ResourceGroupName $RESOURCE_GROUP -Force
   ```

3. **Delete Virtual Network (if created):**

   ```powershell
   # Remove the virtual network if you created one
   Remove-AzVirtualNetwork -Name "iatd_labs_11b_vnet" -ResourceGroupName $RESOURCE_GROUP -Force
   ```

4. **Delete Resource Group:**

   ```powershell
   # Clean up Azure resources created in this lab
   # Note: This will NOT delete your ExpressRoute circuit if it's in a different resource group
   Remove-AzResourceGroup -Name $RESOURCE_GROUP -Force
   ```

   Expected Output:
   ```
   True
   ```

5. **Verify Deletion:**

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
   Resource group iatd_labs_11b_rg has been deleted successfully.
   ```

> **Important Note:** This cleanup process only removes the Azure resources created specifically for this lab. Your ExpressRoute circuit (which is a prerequisite for this lab) will not be affected if it's in a different resource group. Any configurations made on your on-premises router should be reviewed and removed if no longer needed.

### Post-Lab Summary

In this lab, you:
1. Learned about BGP communities in Azure and how they can be used for route filtering
2. Created and configured route filters for ExpressRoute Microsoft peering
3. Explored advanced BGP configuration options for on-premises routers
4. Created a virtual network gateway and connected it to an ExpressRoute circuit
5. Verified BGP route propagation and connectivity
6. Gained hands-on experience with advanced ExpressRoute BGP configurations

These advanced BGP concepts are essential for designing and implementing robust hybrid connectivity solutions with Azure ExpressRoute. The knowledge gained in this lab will help you optimize routing between on-premises networks and Azure, implement traffic engineering, and ensure secure and efficient connectivity.
