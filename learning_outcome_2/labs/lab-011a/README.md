## IATD Microcredential Cloud Networking: Lab 11a - ExpressRoute BGP and Basic Configuration

**Objective:** Configure BGP peering over an Azure ExpressRoute connection and understand the fundamental BGP configuration on the Azure side.

**Estimated Time:** 60 - 90 minutes

**Prerequisites:**

1. **Azure Subscription:** An active Azure subscription.
2. **Azure Cloud Shell:** Access to Azure Cloud Shell (PowerShell).
3. **Existing ExpressRoute Circuit:** This lab *requires* an existing and functional Azure ExpressRoute circuit with either private or Microsoft peering configured. You *must* have access to the on-premises network connected to the ExpressRoute circuit. This is an external resource that will not be created as part of this lab.
4. **Understanding:** Basic networking concepts.

**Let's get started!**

### Lab Conventions

* **Azure Cloud Shell:** All PowerShell interactions will occur within the Azure Cloud Shell environment.
* **Naming Conventions:** Resource names follow the convention `iatd_labs_11a_*`.
* **IP Address Range:** The `172.16.x.x` range will be used.
* **Location:** Choose a consistent Azure region (e.g., `australiaeast`).

#### Resource Naming

* **Resource Group:** `iatd_labs_11a_rg`
* **ExpressRoute Circuit:** Your existing ExpressRoute circuit
* **Virtual Network:** `iatd_labs_11a_vnet` (if creating a test VM)
* **Subnet:** `iatd_labs_11a_subnet` (if creating a test VM)
* **Test VM:** `iatd_labs_11a_vm` (if creating a test VM)
* **Test VM Public IP:** `iatd_labs_11a_vm_pip` (if creating a test VM)

### Part 1: Define Variables and Connect

1. **Open Cloud Shell:** Open Azure Cloud Shell and select **PowerShell**.

2. **Define Variables:**

   ```powershell
   $resourceGroupName = "iatd_labs_11a_rg" # Or create a new one
   $location = "australiaeast"  # Replace with your ExpressRoute circuit's region
   $expressRouteCircuitName = "your-expressroute-circuit-name"  # REPLACE!
   $peeringType = "PrivatePeering"  # Or "MicrosoftPeering"
   ```

3. **Create Resource Group (PowerShell):**

   ```powershell
   New-AzResourceGroup -Name $resourceGroupName -Location $location -ErrorAction SilentlyContinue
   ```

   Expected Output:
   ```
   ResourceGroupName : iatd_labs_11a_rg
   Location          : australiaeast
   ProvisioningState : Succeeded
   Tags              : 
   ResourceId        : /subscriptions/your-subscription-id/resourceGroups/iatd_labs_11a_rg
   ```

4. **Get ExpressRoute Circuit Details (PowerShell):**

   ```powershell
   $circuit = Get-AzExpressRouteCircuit -Name $expressRouteCircuitName -ResourceGroupName $resourceGroupName
   ```

5. **Verify ExpressRoute Circuit (PowerShell):**

   ```powershell
   Write-Host "ExpressRoute Circuit Name: $($circuit.Name)"
   Write-Host "ExpressRoute Circuit Provisioning State: $($circuit.ProvisioningState)"
   Write-Host "ExpressRoute Circuit Peering Type: $($circuit.Peerings.PeeringType)"
   Write-Host "ExpressRoute Circuit Peerings: $($circuit.Peerings | Select-Object -Property Name, PeerASN, PrimaryPeerAddressPrefix, SecondaryPeerAddressPrefix)"
   ```

   Expected Output:
   ```
   ExpressRoute Circuit Name: your-expressroute-circuit-name
   ExpressRoute Circuit Provisioning State: Succeeded
   ExpressRoute Circuit Peering Type: PrivatePeering
   ExpressRoute Circuit Peerings: 
   
   Name           : PrivatePeering
   PeerASN        : 65001
   PrimaryPeerAddressPrefix   : 172.16.0.0/30
   SecondaryPeerAddressPrefix : 172.16.0.4/30
   ```

### Part 2: Inspecting ExpressRoute BGP Configuration (Portal)

1. **Azure Portal: Examine Peering Information:**
   * Navigate to your ExpressRoute circuit in the Azure Portal.
   * Click on "Peerings" in the "Settings" blade.
   * Select your peering (Private or Microsoft, as configured).
   * Examine the following:
     * **Peer ASN:** Should match your on-premises ASN.
     * **BGP Peer IPs:** These are the IPs Microsoft assigned to you.
     * **Advertised Routes:** Routes advertised *from* Azure *to* your on-premises network.
     * **Learned Routes:** Routes advertised *from* your on-premises network *to* Azure.
     * **Verify Route Exchange:** Confirm that routes are being exchanged between your on-premises network and Azure. If this is a new connection you might not see any routes.

### Part 3: Testing BGP Connectivity (Conceptual and Testing)

1. **Conceptual - On-Premises Configuration (Verification):**
   * **On your On-Premises BGP Router (Outside the Scope of this Lab - You will need to confirm these)**
     * Ensure that BGP is properly configured.
     * Verify that your on-premises router can successfully reach the Microsoft Peer IP addresses over the ExpressRoute connection.
     * Verify your router is using the correct ASN.
     * Confirm that you are advertising the routes to Azure that you expect.
   * This part validates that you're peering with the Microsoft side of the ExpressRoute.

2. **Testing from Azure VM (If applicable):**
   * Create a VM in Azure within the VNet peered with your ExpressRoute (if you have connectivity).
   
   ```powershell
   # Define VM variables
   $vmName = "iatd_labs_11a_vm"
   $vmSize = "Standard_DS1_v2"
   $adminUsername = "cloudadmin"
   $adminPassword = "YourStrongPassword123!" # Replace with a strong password
   
   # Create a test VM
   New-AzVm -ResourceGroupName $resourceGroupName -Name $vmName -Location $location -VirtualNetworkName "iatd_labs_11a_vnet" -SubnetName "iatd_labs_11a_subnet" -OpenPorts 22,3389 -Size $vmSize -Credential (New-Object System.Management.Automation.PSCredential ($adminUsername, (ConvertTo-SecureString $adminPassword -AsPlainText -Force)))
   ```
   
   Expected Output:
   ```
   ResourceGroupName        : iatd_labs_11a_rg
   Id                       : /subscriptions/your-subscription-id/resourceGroups/iatd_labs_11a_rg/providers/Microsoft.Compute/virtualMachines/iatd_labs_11a_vm
   VmId                     : 12345678-abcd-1234-efgh-123456789012
   Name                     : iatd_labs_11a_vm
   Type                     : Microsoft.Compute/virtualMachines
   Location                 : australiaeast
   ...
   ```
   
   * Get the private IP addresses and then Ping a resource in on-premises using the Private IP and confirm connectivity.
   * This helps demonstrate end-to-end connectivity.

### Part 4: Clean Up Resources

1. **Delete Resource Group (PowerShell):**

   ```powershell
   # Clean up Azure resources created in this lab
   # Note: This will NOT delete your ExpressRoute circuit if it's in a different resource group
   Remove-AzResourceGroup -Name $resourceGroupName -Force
   ```

   Expected Output:
   ```
   True
   ```

2. **Verify Deletion (PowerShell):**
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
   Resource group iatd_labs_11a_rg has been deleted successfully.
   ```

> **Important Note:** This cleanup process only removes the Azure resources created specifically for this lab. Your ExpressRoute circuit (which is a prerequisite for this lab) will not be affected if it's in a different resource group.

### Post-Lab Summary

In this lab, you:
1. Connected to an existing ExpressRoute circuit
2. Examined the BGP configuration details
3. Verified BGP peering status
4. Tested connectivity between Azure and on-premises (if applicable)
5. Learned how to identify and interpret BGP-related information

This lab demonstrated how Azure uses BGP to exchange routing information between your on-premises network and Azure over an ExpressRoute connection. The BGP peering established allows for dynamic routing updates, ensuring that both networks are aware of each other's routes.
