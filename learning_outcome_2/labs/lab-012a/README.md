## IATD Microcredential Cloud Networking: Lab 12a - Implementing Route Server and Utilizing BGP Communities

**Objective:** Implement an Azure Route Server and learn how BGP communities can be used for route control, traffic management, and applying routing policies.

**Estimated Time:** 60 - 75 minutes

**Prerequisites:**

1. **Azure Subscription:** An active Azure subscription.
2. **Azure Cloud Shell:** Access to Azure Cloud Shell (PowerShell).
3. **Virtual Network:** You must have an existing Virtual Network (VNet) in Azure. This VNet will house the Route Server.
4. **Subnet:** You must have a dedicated subnet within your VNet for the Route Server (minimum /28).
5. **ExpressRoute (Optional, but Recommended for Testing):** While not strictly required for Route Server deployment, having an ExpressRoute circuit (configured with Private or Microsoft Peering) enhances the lab. This is an external resource that will not be created as part of this lab.
6. **Understanding:** Basic networking concepts.

**Let's get started!**

### Lab Conventions

* **Azure Cloud Shell:** All PowerShell interactions will occur within the Azure Cloud Shell environment.
* **Naming Conventions:** Resource names follow the convention `iatd_labs_12a_*`.
* **IP Address Range:** The `172.16.x.x` range will be used.
* **Location:** Choose a consistent Azure region (e.g., `australiaeast`).

#### Resource Naming

* **Resource Group:** `iatd_labs_12a_rg`
* **Virtual Network:** `iatd_labs_12a_vnet`
* **Route Server Subnet:** `RouteServerSubnet` (This name is required by Azure)
* **Route Server:** `iatd_labs_12a_routeserver`
* **ExpressRoute Circuit:** Your existing ExpressRoute circuit (if applicable)

### Part 1: Prepare the Environment

1. **Open Cloud Shell:** Open Azure Cloud Shell and select **PowerShell**.

2. **Define Variables:**

   ```powershell
   $resourceGroupName = "iatd_labs_12a_rg"
   $location = "australiaeast"   # Replace with your preferred region
   $vnetName = "iatd_labs_12a_vnet"
   $routeServerSubnetName = "RouteServerSubnet"  # IMPORTANT: This name is required
   $routeServerSubnetPrefix = "172.16.3.0/28"   # Make sure this range is free
   $routeServerName = "iatd_labs_12a_routeserver"
   $expressRouteCircuitName = "your-expressroute-circuit-name" # Replace if using ExpressRoute
   $peeringType = "PrivatePeering" # Replace if using ExpressRoute
   ```

3. **Create Resource Group:**

   ```powershell
   New-AzResourceGroup -Name $resourceGroupName -Location $location -ErrorAction SilentlyContinue
   ```

   Expected Output:
   ```
   ResourceGroupName : iatd_labs_12a_rg
   Location          : australiaeast
   ProvisioningState : Succeeded
   Tags              : 
   ResourceId        : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/iatd_labs_12a_rg
   ```

4. **Create Virtual Network and Route Server Subnet:**

   ```powershell
   $vnet = Get-AzVirtualNetwork -Name $vnetName -ResourceGroupName $resourceGroupName -ErrorAction SilentlyContinue

   if (-not $vnet) {
       # Create the VNet
       $vnet = New-AzVirtualNetwork -Name $vnetName -ResourceGroupName $resourceGroupName -Location $location -AddressPrefix "172.16.0.0/16"

       # Create the Route Server Subnet
       $subnet = New-AzVirtualNetworkSubnetConfig -Name $routeServerSubnetName -AddressPrefix $routeServerSubnetPrefix
       $vnet | Set-AzVirtualNetwork
   }
   ```

   Expected Output:
   ```
   Name                   : iatd_labs_12a_vnet
   ResourceGroupName      : iatd_labs_12a_rg
   Location               : australiaeast
   Id                     : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/iatd_labs_12a_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_12a_vnet
   Etag                   : W/"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
   ResourceGuid           : xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
   ProvisioningState      : Succeeded
   Tags                   : 
   AddressSpace           : {172.16.0.0/16}
   DhcpOptions            : {}
   Subnets                : [RouteServerSubnet]
   VirtualNetworkPeerings : {}
   EnableDdosProtection   : false
   DdosProtectionPlan     : 
   ```

5. **Check for Existing Route Server:**

   ```powershell
   Get-AzRouteServer -ResourceGroupName $resourceGroupName -ErrorAction SilentlyContinue
   ```

### Part 2: Deploy the Azure Route Server

1. **Create Azure Route Server:**

   ```powershell
   $routeServer = New-AzRouteServer -Name $routeServerName -ResourceGroupName $resourceGroupName -Location $location -VirtualNetworkName $vnetName -SubnetName $routeServerSubnetName
   ```

   Expected Output:
   ```
   Name                : iatd_labs_12a_routeserver
   ResourceGroupName   : iatd_labs_12a_rg
   Location            : australiaeast
   Id                  : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/iatd_labs_12a_rg/providers/Microsoft.Network/virtualHubs/iatd_labs_12a_routeserver
   ProvisioningState   : Succeeded
   ```

   > **Note:** This deployment may take 15-20 minutes to complete.

### Part 3: Configure BGP Peering (ExpressRoute as an Example)

1. **Get Route Server Object:**

   ```powershell
   $routeServer = Get-AzRouteServer -Name $routeServerName -ResourceGroupName $resourceGroupName
   ```

2. **Route Server Peering with ExpressRoute Circuit:**

   ```powershell
   # Retrieve the ExpressRoute circuit
   $circuit = Get-AzExpressRouteCircuit -Name $expressRouteCircuitName -ResourceGroupName $resourceGroupName

   # Configure the BGP peering
   $peeringName = "ExpressRoutePeering"
   New-AzRouteServerPeer -PeerName $peeringName -RouteServer $routeServer -PeerAsn $($circuit.Peerings.PeerASN) -PeerAddress $($circuit.Peerings | Where-Object {$_.PeeringType -eq $peeringType} | Select-Object -ExpandProperty PeerMicrosoftPeeringAddresses[0].PrimaryPeerAddress)
   ```

   Expected Output:
   ```
   Name              : ExpressRoutePeering
   ResourceGroupName : iatd_labs_12a_rg
   PeerIp            : 192.168.1.1 (example IP)
   PeerAsn           : 65001 (example ASN)
   ProvisioningState : Succeeded
   ```

3. **Configure BGP Peering on the ExpressRoute Side:**

   This step would be performed on your on-premises router. You would need to configure BGP peering with the Route Server's IP addresses, which can be obtained using:

   ```powershell
   $routeServer.VirtualRouterIps
   ```

   Expected Output:
   ```
   172.16.3.4
   172.16.3.5
   ```

### Part 4: Understanding BGP Communities

1. **Azure BGP Communities Overview:**

   Azure uses BGP communities to categorize routes based on their origin and service type. These communities help in implementing more granular routing policies.

   Key Azure BGP Communities for ExpressRoute:
   * **12076:50001** - Routes from all Azure regions (public regions)
   * **12076:50002** - Routes from all Microsoft Edge locations
   * **12076:50003** - Routes from all Azure regions (public and Azure Government)
   * **12076:51001** - Routes for Azure ExpressRoute
   * **12076:52001** - Routes for Azure Storage
   * **12076:52002** - Routes for Azure SQL
   * **12076:52003** - Routes for Azure Websites
   * **12076:52004** - Routes for Azure Virtual Machines

2. **Using BGP Communities for Route Control:**

   On your on-premises router, you can use BGP communities to filter routes received from Azure. For example, you might want to accept only routes for Azure Storage and Azure Virtual Machines.

   Example configuration (Cisco IOS syntax):
   ```
   ip community-list 100 permit 12076:52001
   ip community-list 100 permit 12076:52004
   
   route-map AZURE_IN permit 10
    match community 100
    set local-preference 200
   
   route-map AZURE_IN deny 20
   
   router bgp 65001
    neighbor 172.16.3.4 remote-as 65515
    neighbor 172.16.3.4 route-map AZURE_IN in
    neighbor 172.16.3.5 remote-as 65515
    neighbor 172.16.3.5 route-map AZURE_IN in
   ```

   This configuration would accept only routes for Azure Storage and Azure Virtual Machines, and reject all other routes.

### Part 5: Clean Up Resources

1. **Delete Route Server:**

   ```powershell
   # Remove the Route Server
   Remove-AzRouteServer -Name $routeServerName -ResourceGroupName $resourceGroupName -Force
   ```

2. **Delete Virtual Network:**

   ```powershell
   # Remove the virtual network
   Remove-AzVirtualNetwork -Name $vnetName -ResourceGroupName $resourceGroupName -Force
   ```

3. **Delete Resource Group:**

   ```powershell
   # Clean up Azure resources created in this lab
   # Note: This will NOT delete your ExpressRoute circuit if it's in a different resource group
   Remove-AzResourceGroup -Name $resourceGroupName -Force
   ```

   Expected Output:
   ```
   True
   ```

4. **Verify Deletion:**

   ```powershell
   Get-AzResourceGroup -Name $resourceGroupName -ErrorVariable notPresent -ErrorAction SilentlyContinue
   if ($notPresent) {
       Write-Host "Resource group $resourceGroupName has been deleted successfully."
   } else {
       Write-Host "Resource group $resourceGroupName still exists. Please try deleting it again."
   }
   ```

   Expected Output:
   ```
   Resource group iatd_labs_12a_rg has been deleted successfully.
   ```

> **Important Note:** This cleanup process only removes the Azure resources created specifically for this lab. Your ExpressRoute circuit (which is an optional prerequisite for this lab) will not be affected if it's in a different resource group. Any configurations made on your on-premises router should be reviewed and removed if no longer needed.

### Post-Lab Summary

In this lab, you:
1. Created a Virtual Network with a dedicated subnet for Route Server
2. Deployed an Azure Route Server
3. Configured BGP peering between the Route Server and an ExpressRoute circuit (conceptually)
4. Learned about BGP communities and how they can be used for route filtering
5. Gained hands-on experience with Azure's routing capabilities

The Azure Route Server simplifies BGP peering in hybrid network scenarios by providing a central point for BGP peering, eliminating the need to establish multiple BGP sessions with each virtual network gateway or NVA. This knowledge is essential for designing and implementing robust hybrid connectivity solutions with Azure.
