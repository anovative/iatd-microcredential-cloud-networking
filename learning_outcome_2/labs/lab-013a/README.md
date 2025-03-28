## IATD Microcredential Cloud Networking: Lab 13a - Advanced BGP: Route Filtering and AS Number Management

**Objective:** Implement route filtering techniques and understand the principles of Autonomous System (AS) number management to control route advertisements and secure your network.

**Estimated Time:** 60 minutes

**Prerequisites:**

1. **Azure Subscription:** An active Azure subscription.
2. **Azure Cloud Shell:** Access to Azure Cloud Shell (PowerShell).
3. **ExpressRoute Circuit:** An existing ExpressRoute circuit. This is an external resource that will not be created as part of this lab.
4. **On-Premises Router Access:** Administrative access to your on-premises router connected to ExpressRoute for configuring BGP settings.
5. **Understanding of BGP Concepts:** Familiarity with basic BGP concepts and terminology.

**Let's get started!**

### Lab Conventions

* **Azure Cloud Shell:** All PowerShell interactions will occur within the Azure Cloud Shell environment.
* **Naming Conventions:** Resource names follow the convention `iatd_labs_13a_*`.
* **IP Address Range:** The `172.16.x.x` range will be used for Azure resources.
* **Location:** Choose a consistent Azure region where your ExpressRoute circuit is deployed.

#### Resource Naming

* **Resource Group:** `iatd_labs_13a_rg`
* **Virtual Network:** `iatd_labs_13a_vnet`
* **ExpressRoute Circuit:** Your existing ExpressRoute circuit

### Part 1: Understanding Autonomous System (AS) Numbers

1. **AS Numbers: Public vs. Private**

   Autonomous System Numbers (ASNs) are unique identifiers assigned to networks for BGP routing purposes. There are two types of ASNs:

   * **Public ASNs:** Globally unique, registered with regional Internet registries (RIRs)
     * Ranges: 1-64495 and 131072-4199999999
     * Used when connecting to the public internet or via ExpressRoute
     * Require registration and typically involve an annual fee

   * **Private ASNs:** Used within private networks, not globally routable
     * Ranges: 64512-65535 (2-byte ASN) and 4200000000-4294967295 (4-byte ASN)
     * Can be reused across different private networks
     * No registration required

2. **ASNs and ExpressRoute**

   ExpressRoute typically uses public ASNs to ensure global uniqueness and proper interconnectivity. Let's verify the ASN configuration in your ExpressRoute circuit:

   1. Open the Azure Portal
   2. Search for "ExpressRoute circuits"
   3. Select your ExpressRoute circuit
   4. Go to **Peerings**
   5. Note the following information:
      * Microsoft ASN (typically 12076 or 8075)
      * Your ASN (the ASN assigned to your on-premises network)
      * Primary and secondary peer IP addresses
      * Advertised and learned routes

3. **Verifying On-Premises Settings**

   It's essential to ensure that your on-premises router is configured with the same ASN and peering information as shown in the Azure Portal. On your on-premises router, verify:

   * The configured ASN matches what's shown in the Azure Portal
   * The BGP peer IP addresses match the Azure ExpressRoute peer IPs
   * The BGP session is established and routes are being exchanged

### Part 2: Implementing Route Filtering

1. **The Concept of Route Filtering**

   Route filtering allows you to control which routes are advertised or accepted via BGP. Benefits include:

   * **Security:** Prevents unauthorized access by restricting route advertisements
   * **Efficiency:** Reduces routing table size by filtering unnecessary routes
   * **Stability:** Prevents routing loops and improves network stability
   * **Traffic Engineering:** Directs traffic along preferred paths

2. **Route Filtering Techniques**

   Several methods can be used to filter routes in BGP:

   * **Prefix Lists:** Define specific IP address ranges to allow or deny
   * **AS_PATH Access Lists:** Filter routes based on the AS path attribute
   * **Community Lists:** Filter routes based on BGP community attributes

   In this lab, we'll focus on using prefix lists to control route advertisements from your on-premises network to Azure.

3. **Implementing Prefix Lists on Your On-Premises Router**

   The following example shows how to create prefix lists on a Cisco IOS router to allow only specific subnets to be advertised to Azure. The exact commands may vary depending on your router's operating system.

   Example (Cisco IOS syntax):
   ```
   ! Create prefix lists to allow specific subnets
   ip prefix-list ALLOW_SUBNET_1 seq 5 permit 192.168.1.0/24
   ip prefix-list ALLOW_SUBNET_2 seq 5 permit 10.0.0.0/8 le 24
   
   ! Create a route map to apply the prefix lists
   route-map AZURE_ADVERTISE permit 10
     match ip address prefix-list ALLOW_SUBNET_1
   !
   route-map AZURE_ADVERTISE permit 20
     match ip address prefix-list ALLOW_SUBNET_2
   !
   
   ! Apply the route map to the BGP neighbor
   router bgp 65001 ! Replace with your ASN
     neighbor 172.16.3.4 remote-as 12076 ! Replace with Azure BGP Peer IP and ASN
     neighbor 172.16.3.4 route-map AZURE_ADVERTISE out
   ```

4. **Verifying Route Filtering**

   After implementing route filtering, verify that only the allowed routes are being advertised to Azure:

   * **On your on-premises router:**
     ```
     show ip bgp neighbors 172.16.3.4 advertised-routes
     ```

   * **In the Azure Portal:**
     1. Navigate to your ExpressRoute circuit
     2. Select the peering configuration
     3. View the "Learned routes" section
     4. Confirm that only the allowed routes are present

5. **Testing Connectivity**

   Test connectivity to verify that your route filtering is working correctly:

   * **From Azure to on-premises:**
     * You should be able to reach IP addresses in the allowed subnets
     * You should not be able to reach IP addresses in filtered subnets

   * **From on-premises to Azure:**
     * Connectivity should work as expected since we're only filtering routes advertised from on-premises to Azure

### Part 3: Using Route Filters in Azure (Optional)

1. **Azure Route Filters**

   Azure Route Filters can be used with ExpressRoute Microsoft peering to control which Microsoft services are accessible through your ExpressRoute circuit.

   To create a route filter in Azure:

   ```powershell
   # Create a new resource group for the route filter
   $resourceGroup = "iatd_labs_13a_rg"
   $location = "australiaeast" # Replace with your region
   
   # Create a new route filter
   $routeFilterName = "iatd_labs_13a_routefilter"
   New-AzRouteFilter -Name $routeFilterName -ResourceGroupName $resourceGroup -Location $location
   
   # Add a rule to the route filter
   $rule = New-AzRouteFilterRuleConfig -Name "AllowAzureServices" -Access Allow -RouteFilterRuleType Community -CommunityList "12076:5010" -Force
   
   # Update the route filter with the new rule
   Set-AzRouteFilter -Name $routeFilterName -ResourceGroupName $resourceGroup -Rule $rule
   ```

2. **Associating the Route Filter with ExpressRoute Microsoft Peering**

   ```powershell
   # Get the route filter
   $routeFilter = Get-AzRouteFilter -Name $routeFilterName -ResourceGroupName $resourceGroup
   
   # Get the ExpressRoute circuit
   $circuitName = "your-expressroute-circuit-name" # Replace with your circuit name
   $circuit = Get-AzExpressRouteCircuit -Name $circuitName -ResourceGroupName $resourceGroup
   
   # Associate the route filter with Microsoft peering
   Set-AzExpressRouteCircuitPeeringConfig -Name "MicrosoftPeering" -ExpressRouteCircuit $circuit -PeeringType MicrosoftPeering -RouteFilter $routeFilter
   
   # Update the circuit
   Set-AzExpressRouteCircuit -ExpressRouteCircuit $circuit
   ```

### Clean Up Resources

1. **Note on Resource Cleanup:**

   If you created any Azure resources specifically for this lab, you can clean them up as follows:

   ```powershell
   # Remove the route filter (if created)
   Remove-AzRouteFilter -Name $routeFilterName -ResourceGroupName $resourceGroup -Force
   
   # Remove the resource group (if created specifically for this lab)
   Remove-AzResourceGroup -Name $resourceGroup -Force
   ```

   > **Important Note:** This cleanup process will not affect your ExpressRoute circuit if it's in a different resource group. Any configurations made on your on-premises router should be reviewed and removed if no longer needed.

2. **Reverting On-Premises Configuration:**

   If you need to revert the changes made to your on-premises router, you can remove the route filtering configuration:

   ```
   router bgp 65001 ! Replace with your ASN
     no neighbor 172.16.3.4 route-map AZURE_ADVERTISE out
   !
   no route-map AZURE_ADVERTISE
   no ip prefix-list ALLOW_SUBNET_1
   no ip prefix-list ALLOW_SUBNET_2
   ```

### Post-Lab Summary

In this lab, you:
1. Learned about the differences between public and private AS numbers
2. Understood the importance of using public ASNs with ExpressRoute
3. Implemented route filtering using prefix lists on your on-premises router
4. Verified route filtering and tested connectivity
5. (Optional) Explored Azure Route Filters for Microsoft peering

These advanced BGP concepts are essential for designing and implementing secure and efficient hybrid connectivity solutions with Azure. The knowledge gained in this lab will help you control route advertisements, prevent routing loops, and secure your network.