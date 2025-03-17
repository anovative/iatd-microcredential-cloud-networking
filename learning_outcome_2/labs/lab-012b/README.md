## IATD Microcredential Cloud Networking: Lab 12b - Advanced Route Filtering Using Route Server and Communities

**Objective:** Demonstrate how to filter routes using the Azure Route Server and BGP communities to control route advertisements and optimize routing tables.

**Estimated Time:** 60 - 75 minutes

**Prerequisites:**

1. **Azure Subscription:** An active Azure subscription.
2. **Azure Cloud Shell:** Access to Azure Cloud Shell (PowerShell).
3. **Completion of Lab 12a:** You must have completed Lab 12a which sets up the Azure Route Server.
4. **Existing VNet and Route Server:** Ensure you have an existing VNet and Azure Route Server deployed as described in Lab 12a.
5. **ExpressRoute Circuit:** An existing ExpressRoute circuit (Private Peering or Microsoft Peering) is required to test the filtering. This is an external resource that will not be created as part of this lab.
6. **Access to On-Premises Router:** Administrative access to the on-premises router connected to ExpressRoute for configuring BGP settings.

**Let's get started!**

### Lab Conventions

* **Azure Cloud Shell:** All PowerShell interactions will occur within the Azure Cloud Shell environment.
* **Naming Conventions:** Resource names follow the convention `iatd_labs_12b_*`.
* **IP Address Range:** The `172.16.x.x` range will be used.
* **Location:** Choose a consistent Azure region (e.g., `australiaeast`).

#### Resource Naming

* **Resource Group:** `iatd_labs_12a_rg` (reusing from Lab 12a)
* **Virtual Network:** `iatd_labs_12a_vnet` (reusing from Lab 12a)
* **Route Server:** `iatd_labs_12a_routeserver` (reusing from Lab 12a)
* **ExpressRoute Circuit:** Your existing ExpressRoute circuit

### Part 1: Prepare the Environment

1. **Open Cloud Shell:** Open Azure Cloud Shell and select **PowerShell**.

2. **Define Variables:**

   ```powershell
   $resourceGroupName = "iatd_labs_12a_rg" # Reusing Resource Group from Lab 12a
   $location = "australiaeast"  # Replace with your preferred region
   $vnetName = "iatd_labs_12a_vnet"
   $routeServerName = "iatd_labs_12a_routeserver"
   $expressRouteCircuitName = "your-expressroute-circuit-name" # Replace with your circuit name
   $peeringType = "PrivatePeering"  # Replace with your peering type (Private or Microsoft)
   ```

3. **Get Route Server and ExpressRoute Circuit Details:**

   ```powershell
   $routeServer = Get-AzRouteServer -Name $routeServerName -ResourceGroupName $resourceGroupName
   $expressRouteCircuit = Get-AzExpressRouteCircuit -Name $expressRouteCircuitName -ResourceGroupName $resourceGroupName
   ```

   Expected Output (partial):
   ```
   Name                : iatd_labs_12a_routeserver
   ResourceGroupName   : iatd_labs_12a_rg
   Location            : australiaeast
   ProvisioningState   : Succeeded
   ```

### Part 2: Understanding Prefix Lists for Route Filtering

1. **Prefix Lists Overview:**

   Prefix lists are ordered lists of IP prefixes (networks) used to filter routes. They are a fundamental filtering tool in BGP configurations. In this section, we'll explore how to create and apply prefix lists on your on-premises router to control which routes are advertised to Azure.

2. **Creating Prefix Lists on Your On-Premises Router:**

   The following examples show how to create prefix lists on a Cisco IOS router. The exact commands may vary depending on your router's operating system.

   Example (Cisco IOS syntax):
   ```
   ! Create a prefix list to allow a specific subnet
   ip prefix-list AZURE_ALLOWED_SUBNETS seq 5 permit 192.168.1.0/24
   
   ! Create another prefix list to deny a different subnet or range
   ip prefix-list AZURE_DENY_SUBNETS seq 5 deny 10.0.0.0/8 le 24
   ```

3. **Applying Prefix Lists to BGP:**

   After creating the prefix lists, you need to apply them to your BGP configuration using route maps.

   Example (Cisco IOS syntax):
   ```
   ! Create a route map to apply the prefix lists
   route-map AZURE_ADVERTISE permit 10
    match ip address prefix-list AZURE_ALLOWED_SUBNETS
   !
   route-map AZURE_ADVERTISE deny 20
    match ip address prefix-list AZURE_DENY_SUBNETS
   !
   route-map AZURE_ADVERTISE permit 100
   !
   ! Apply the route map to the BGP neighbor
   router bgp 65001 ! Replace with your ASN
    neighbor 172.16.3.4 remote-as 65515 ! Replace with your Azure peer IP
    neighbor 172.16.3.4 route-map AZURE_ADVERTISE out
    neighbor 172.16.3.5 remote-as 65515 ! Replace with your Azure peer IP
    neighbor 172.16.3.5 route-map AZURE_ADVERTISE out
   ```

### Part 3: Using BGP Communities for Route Control

1. **BGP Communities Overview:**

   BGP communities are tags attached to routes that can be used to control route advertisement and influence traffic flow. Microsoft provides pre-defined BGP communities that you can use with ExpressRoute Microsoft peering.

   Key Microsoft-defined communities:
   * **39998:1** - Advertise to the global network
   * **39998:2** - Advertise to specific regions
   * **39998:800** - Advertise to a specific peering location

2. **Configuring BGP Communities on Your On-Premises Router:**

   The following example shows how to apply BGP communities to routes advertised to Azure.

   Example (Cisco IOS syntax):
   ```
   ! Create a route map to set communities
   route-map SET_COMMUNITY permit 10
    match ip address prefix-list AZURE_ALLOWED_SUBNETS
    set community 39998:2
   !
   route-map SET_COMMUNITY deny 20
   !
   ! Apply the route map to the BGP neighbor
   router bgp 65001 ! Replace with your ASN
    neighbor 172.16.3.4 remote-as 65515 ! Replace with your Azure peer IP
    neighbor 172.16.3.4 route-map SET_COMMUNITY out
    neighbor 172.16.3.5 remote-as 65515 ! Replace with your Azure peer IP
    neighbor 172.16.3.5 route-map SET_COMMUNITY out
   ```

3. **Impact of Communities:**

   When you apply a community to a route, it affects how that route is advertised within the Azure network. For example:
   * Routes with the 39998:1 community will be advertised globally across all Azure regions
   * Routes with the 39998:2 community will only be advertised to specific regions
   * Routes with the 39998:800 community will only be advertised to a specific peering location

### Part 4: Verifying Route Filtering

1. **Verify Learned Routes Using PowerShell:**

   ```powershell
   # Get the name of your BGP peering
   $peeringName = "ExpressRoutePeering" # Replace with your peering name
   
   # View routes learned from this peer
   Get-AzRouteServerPeerRoute -Name $peeringName -ResourceGroupName $resourceGroupName -RouteServerName $routeServerName
   ```

   Expected Output (example):
   ```
   LocalAddress Network        NextHop        SourcePeer     Origin AsPath            Weight
   ------------ -------        -------        ----------     ------ ------            ------
   172.16.3.4   192.168.1.0/24 192.168.0.1    192.168.0.1    EBgp   65001             32768
   ```

2. **Verify Routes in the Azure Portal:**

   You can also verify the learned routes in the Azure Portal:
   1. Navigate to your ExpressRoute circuit
   2. Select the peering configuration
   3. View the "Learned routes" section
   4. Confirm that the routes you expected to be advertised are present

3. **Test Connectivity (if applicable):**

   If you have a test VM in Azure, you can test connectivity to your on-premises network:

   ```powershell
   # Connect to your test VM and run ping
   ping 192.168.1.100 # Replace with an IP address in your on-premises network
   ```

### Part 5: Clean Up Resources

1. **Note on Resource Cleanup:**

   Since we are reusing resources from Lab 12a, we will not delete them here. If you are completely finished with both labs, you can delete the resources as follows:

   ```powershell
   # Remove the Route Server
   Remove-AzRouteServer -Name $routeServerName -ResourceGroupName $resourceGroupName -Force
   
   # Remove the virtual network
   Remove-AzVirtualNetwork -Name $vnetName -ResourceGroupName $resourceGroupName -Force
   
   # Clean up Azure resources created in Lab 12a
   # Note: This will NOT delete your ExpressRoute circuit if it's in a different resource group
   Remove-AzResourceGroup -Name $resourceGroupName -Force
   ```

   Expected Output:
   ```
   True
   ```

2. **Verify Deletion:**

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

> **Important Note:** This cleanup process removes all Azure resources created for Labs 12a and 12b. Your ExpressRoute circuit will not be affected if it's in a different resource group. Any configurations made on your on-premises router should be reviewed and removed if no longer needed.

### Post-Lab Summary

In this lab, you:
1. Learned how to use prefix lists to filter routes advertised to Azure
2. Explored BGP communities and how they can be used to control route advertisement
3. Applied route filtering techniques to optimize your routing table
4. Verified route filtering using PowerShell and the Azure Portal
5. Gained hands-on experience with advanced BGP configuration

These advanced BGP concepts are essential for designing and implementing robust hybrid connectivity solutions with Azure. The knowledge gained in this lab will help you optimize routing between on-premises networks and Azure, implement traffic engineering, and ensure secure and efficient connectivity.