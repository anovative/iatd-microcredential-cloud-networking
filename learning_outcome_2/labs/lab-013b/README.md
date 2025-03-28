## IATD Microcredential Cloud Networking: Lab 13b - Advanced BGP: Route Filtering Using Prefix Lists

**Objective:** Implement prefix lists to filter routes and create access control lists in a BGP environment, allowing precise control over which networks are advertised over your ExpressRoute connection.

**Estimated Time:** 60 minutes

**Prerequisites:**

1. **Azure Subscription:** An active Azure subscription.
2. **Azure Cloud Shell:** Access to Azure Cloud Shell (PowerShell).
3. **ExpressRoute Circuit:** An existing ExpressRoute circuit. This is an external resource that will not be created as part of this lab.
4. **Existing Azure Route Server:** You must have already deployed the Azure Route Server from previous labs.
5. **On-Premises Router Access:** Administrative access to your on-premises router connected to ExpressRoute for configuring BGP settings.
6. **Completion of Lab 13a:** Familiarity with AS numbers and basic route filtering concepts from Lab 13a.

**Let's get started!**

### Lab Conventions

* **Azure Cloud Shell:** All PowerShell interactions will occur within the Azure Cloud Shell environment.
* **Naming Conventions:** Resource names follow the convention `iatd_labs_13b_*`.
* **IP Address Range:** The `172.16.x.x` range will be used for Azure resources and `192.168.0.0/24` for on-premises networks.
* **Location:** Choose a consistent Azure region where your ExpressRoute circuit is deployed.

#### Resource Naming

* **Resource Group:** `iatd_labs_13b_rg` (or reusing previous lab resources)
* **Virtual Network:** `iatd_labs_13b_vnet` (or reusing previous lab resources)
* **Route Server:** `iatd_labs_13b_routeserver` (or reusing previous lab resources)
* **ExpressRoute Circuit:** Your existing ExpressRoute circuit

### Part 1: Prepare the Environment

1. **Open Cloud Shell:** Open Azure Cloud Shell and select **PowerShell**.

2. **Define Variables:**

   ```powershell
   # Replace these values with your environment's values
   $resourceGroupName = "iatd_labs_12a_rg" # Use your resource group name
   $location = "australiaeast" # Use the region of your ExpressRoute
   $vnetName = "iatd_labs_12a_vnet" # Use your VNet's name
   $routeServerName = "iatd_labs_12a_routeserver" # Name of your route server
   $expressRouteCircuitName = "your-expressroute-circuit-name" # Replace with your circuit name
   $peeringType = "PrivatePeering" # or "MicrosoftPeering", if relevant
   $onPremBgpAsn = "65000" # Replace with the ASN of your on-premises BGP router
   ```

3. **Get Route Server Details:**

   ```powershell
   $routeServer = Get-AzRouteServer -Name $routeServerName -ResourceGroupName $resourceGroupName
   ```

   Expected Output (partial):
   ```
   Name                : iatd_labs_12a_routeserver
   ResourceGroupName   : iatd_labs_12a_rg
   Location            : australiaeast
   ProvisioningState   : Succeeded
   ```

4. **Get ExpressRoute Circuit Details:**

   ```powershell
   $expressRouteCircuit = Get-AzExpressRouteCircuit -Name $expressRouteCircuitName -ResourceGroupName $resourceGroupName
   ```

   Expected Output (partial):
   ```
   Name                : your-expressroute-circuit-name
   ResourceGroupName   : iatd_labs_12a_rg
   Location            : australiaeast
   ProvisioningState   : Succeeded
   ```

5. **Verify BGP Peering Status:**

   ```powershell
   # Get the BGP peering configuration
   $peering = Get-AzExpressRouteCircuitPeeringConfig -Name $peeringType -ExpressRouteCircuit $expressRouteCircuit
   
   # Display peering details
   $peering | Select-Object PeeringType, AzureASN, PeerASN, PrimaryPeerAddressPrefix, SecondaryPeerAddressPrefix
   ```

   Expected Output (example):
   ```
   PeeringType PrimaryPeerAddressPrefix SecondaryPeerAddressPrefix AzureASN PeerASN
   ----------- ------------------------- --------------------------- -------- -------
   AzurePrivatePeering 172.16.0.0/30               172.16.0.4/30                12076   65000
   ```

### Part 2: Implementing Route Filtering on Your On-Premises Router

1. **Connect to Your On-Premises Router:**

   You'll need to connect to your on-premises router using your preferred method (SSH, Telnet, console connection, etc.).

   ```
   # Example (not run in Azure Cloud Shell)
   ssh admin@your-router-ip
   ```

2. **Verify Current BGP Configuration:**

   Before making changes, check the current BGP configuration and routes being advertised.

   ```
   # For Cisco IOS (commands may vary based on your router OS)
   show ip bgp summary
   show ip bgp neighbors x.x.x.x advertised-routes  # Replace x.x.x.x with Azure BGP peer IP
   ```

3. **Create Prefix Lists:**

   Configure prefix lists to control which networks are advertised to Azure. In this example, we'll allow only the 192.168.0.0/24 subnet to be advertised.

   ```
   # For Cisco IOS (commands may vary based on your router OS)
   
   # Create a prefix list that permits only 192.168.0.0/24
   ip prefix-list AZURE_SUBNETS seq 5 permit 192.168.0.0/24
   
   # Create a route map that references the prefix list
   route-map AZURE_ADVERTISE permit 10
     match ip address prefix-list AZURE_SUBNETS
   ```

4. **Apply the Route Map to BGP:**

   Apply the route map to your BGP configuration to filter outbound route advertisements to Azure.

   ```
   # For Cisco IOS (commands may vary based on your router OS)
   
   router bgp 65000  # Replace with your actual ASN
     neighbor 172.16.0.1 remote-as 12076  # Replace with Azure BGP peer IP and ASN
     address-family ipv4 unicast
       neighbor 172.16.0.1 route-map AZURE_ADVERTISE out
   ```

   > **Note:** Make sure to apply the same configuration to all BGP peers connected to your ExpressRoute circuit.

### Part 3: Verifying Route Filtering

1. **Verify Route Advertisements on Your On-Premises Router:**

   After applying the route filtering configuration, verify that only the allowed routes are being advertised.

   ```
   # For Cisco IOS (commands may vary based on your router OS)
   show ip bgp neighbors 172.16.0.1 advertised-routes
   ```

   Expected Output (example):
   ```
   BGP table version is 12, local router ID is 10.0.0.1
   Status codes: s suppressed, d damped, h history, * valid, > best, i - internal
   
      Network          Next Hop            Metric LocPrf Weight Path
   *> 192.168.0.0/24   0.0.0.0                  0         32768 i
   ```

2. **Verify Learned Routes in Azure:**

   Check the routes learned by Azure from your on-premises network using the Azure Portal or PowerShell.

   ```powershell
   # Get the name of your BGP peering
   $peeringName = "your-bgp-peering-name" # Replace with your peering name
   
   # View routes learned from this peer
   Get-AzRouteServerPeerRoute -Name $peeringName -ResourceGroupName $resourceGroupName -RouteServerName $routeServerName
   ```

   Expected Output (example):
   ```
   LocalAddress Network        NextHop        SourcePeer     Origin AsPath            Weight
   ------------ -------        -------        ----------     ------ ------            ------
   172.16.0.1   192.168.0.0/24 192.168.0.1    192.168.0.1    EBgp   65000             32768
   ```

3. **Test Connectivity:**

   If you have a test VM in Azure, test connectivity to your on-premises network to verify that the route filtering is working correctly.

   ```powershell
   # Connect to your test VM and run ping
   ping 192.168.0.100 # Should succeed (IP in allowed subnet)
   ping 192.168.1.100 # Should fail (IP in filtered subnet)
   ```

### Part 4: Advanced Route Filtering (Optional)

1. **Multiple Subnet Filtering:**

   If you need to allow multiple subnets, you can add more entries to your prefix list:

   ```
   # For Cisco IOS (commands may vary based on your router OS)
   
   # Add another subnet to the prefix list
   ip prefix-list AZURE_SUBNETS seq 10 permit 192.168.1.0/24
   
   # No need to modify the route map as it already references the prefix list
   ```

2. **Using AS Path Filtering:**

   You can also filter routes based on AS path attributes:

   ```
   # For Cisco IOS (commands may vary based on your router OS)
   
   # Create an AS path access list
   ip as-path access-list 10 permit ^$
   
   # Create a route map using the AS path access list
   route-map AZURE_ADVERTISE permit 20
     match as-path 10
   ```

### Part 5: Clean Up Resources

1. **Reverting On-Premises Configuration:**

   If you need to revert the changes made to your on-premises router, you can remove the route filtering configuration:

   ```
   # For Cisco IOS (commands may vary based on your router OS)
   
   router bgp 65000  # Replace with your actual ASN
     neighbor 172.16.0.1 remote-as 12076  # Replace with Azure BGP peer IP and ASN
     address-family ipv4 unicast
       no neighbor 172.16.0.1 route-map AZURE_ADVERTISE out
   
   no route-map AZURE_ADVERTISE
   no ip prefix-list AZURE_SUBNETS
   ```

2. **Note on Azure Resources:**

   Since we are primarily working with your on-premises router configuration in this lab, there are no Azure resources to clean up unless you created test VMs specifically for this lab.

   ```powershell
   # If you created test VMs, you can remove them as follows
   Remove-AzVM -Name "your-test-vm" -ResourceGroupName $resourceGroupName -Force
   ```

   > **Important Note:** This lab focuses on configuring your on-premises router and does not create new Azure resources. Any configurations made on your on-premises router should be reviewed and removed if no longer needed.

### Post-Lab Summary

In this lab, you:
1. Configured prefix lists on your on-premises router to filter route advertisements
2. Applied route filtering to your BGP configuration
3. Verified that only allowed routes are being advertised to Azure
4. Tested connectivity to ensure the route filtering is working correctly
5. (Optional) Explored advanced route filtering techniques

Route filtering using prefix lists is a powerful technique for controlling which networks are advertised over your ExpressRoute connection. This helps improve security by limiting the attack surface and optimizes routing by reducing the size of routing tables.