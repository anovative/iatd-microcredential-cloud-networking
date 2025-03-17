## IATD Microcredential Cloud Networking: Lab 9a - BGP Fundamentals: Autonomous Systems (AS) and Neighbor Relationships

**Objective:** To introduce the concept of Autonomous Systems (AS) and BGP neighbor relationships, setting the foundation for understanding BGP peering.

**Estimated Time:** 45 - 60 minutes

**Prerequisites:**

1. **Azure Subscription:** An active Azure subscription.
2. **Azure Cloud Shell:** Access to Azure Cloud Shell (Bash or PowerShell).
3. **Basic Understanding:** Familiarity with Virtual Networks, subnets, and basic networking concepts.
4. **Region Selection:** Choose a region.

**Let's get started!**

### Lab Conventions

* **Azure Cloud Shell:** All PowerShell interactions will occur within the Azure Cloud Shell environment.
* **Naming Conventions:** Resource names follow the convention `iatd_labs_09a_*`.
* **IP Address Range:** The `172.16.x.x` range will be used.
* **Location:** Choose a consistent Azure region (e.g., `australiaeast`).

#### Resource Naming

* **Resource Group:** `iatd_labs_09a_rg`
* **Virtual Network AS1:** `iatd_labs_09a_vnet_as1`
* **Virtual Network AS2:** `iatd_labs_09a_vnet_as2`
* **Subnet:** `iatd_labs_09a_subnet`
* **VM AS1:** `iatd_labs_09a_vm_as1`
* **VM AS2:** `iatd_labs_09a_vm_as2`

### Part 1: Creating the Virtual Network and VMs (Simulating ASes)

1. **Open Cloud Shell:** Open Azure Cloud Shell and select **PowerShell**.

2. **Define Variables:**

   ```powershell
   $RESOURCE_GROUP = "iatd_labs_09a_rg"
   $LOCATION = "australiaeast" # Replace with your preferred region
   $VNET_NAME_AS1 = "iatd_labs_09a_vnet_as1"
   $VNET_NAME_AS2 = "iatd_labs_09a_vnet_as2"
   $SUBNET_NAME = "iatd_labs_09a_subnet"
   $SUBNET_PREFIX = "172.16.1.0/24"
   $VM_AS1_NAME = "iatd_labs_09a_vm_as1"
   $VM_AS2_NAME = "iatd_labs_09a_vm_as2"
   $VM_SIZE = "Standard_DS1_v2" # Adjust as needed
   $ADMIN_USERNAME = "cloudadmin"
   $ADMIN_PASSWORD = "YourStrongPassword123!"  # REPLACE with a strong password!
   ```

3. **Create Resource Group:**

   ```powershell
   New-AzResourceGroup -Name $RESOURCE_GROUP -Location $LOCATION -ErrorAction SilentlyContinue
   ```

   Expected Output:
   ```
   ResourceGroupName : iatd_labs_09a_rg
   Location          : australiaeast
   ProvisioningState : Succeeded
   Tags              : 
   ResourceId        : /subscriptions/your-subscription-id/resourceGroups/iatd_labs_09a_rg
   ```

4. **Create Virtual Network for AS1:**

   ```powershell
   $vnetAs1 = New-AzVirtualNetwork -Name $VNET_NAME_AS1 -ResourceGroupName $RESOURCE_GROUP -Location $LOCATION -AddressPrefix "172.16.1.0/24"
   ```

   Expected Output:
   ```
   Name              : iatd_labs_09a_vnet_as1
   ResourceGroupName : iatd_labs_09a_rg
   Location          : australiaeast
   AddressSpace      : {172.16.1.0/24}
   Subnets           : {}
   ```

5. **Create Virtual Network for AS2:**

   ```powershell
   $vnetAs2 = New-AzVirtualNetwork -Name $VNET_NAME_AS2 -ResourceGroupName $RESOURCE_GROUP -Location $LOCATION -AddressPrefix "172.16.2.0/24"
   ```

   Expected Output:
   ```
   Name              : iatd_labs_09a_vnet_as2
   ResourceGroupName : iatd_labs_09a_rg
   Location          : australiaeast
   AddressSpace      : {172.16.2.0/24}
   Subnets           : {}
   ```

6. **Create Subnet in AS1:**

   ```powershell
   Add-AzVirtualNetworkSubnetConfig -Name $SUBNET_NAME -VirtualNetwork $vnetAs1 -AddressPrefix $SUBNET_PREFIX
   $vnetAs1 | Set-AzVirtualNetwork
   ```

   Expected Output:
   ```
   Name              : iatd_labs_09a_vnet_as1
   ResourceGroupName : iatd_labs_09a_rg
   Location          : australiaeast
   AddressSpace      : {172.16.1.0/24}
   Subnets           : {iatd_labs_09a_subnet}
   ```

7. **Create Subnet in AS2:**

   ```powershell
   Add-AzVirtualNetworkSubnetConfig -Name $SUBNET_NAME -VirtualNetwork $vnetAs2 -AddressPrefix $SUBNET_PREFIX
   $vnetAs2 | Set-AzVirtualNetwork
   ```

   Expected Output:
   ```
   Name              : iatd_labs_09a_vnet_as2
   ResourceGroupName : iatd_labs_09a_rg
   Location          : australiaeast
   AddressSpace      : {172.16.2.0/24}
   Subnets           : {iatd_labs_09a_subnet}
   ```

8. **Create VM for AS1:**

   ```powershell
   New-AzVm -ResourceGroupName $RESOURCE_GROUP -Name $VM_AS1_NAME -Location $LOCATION -VirtualNetworkName $VNET_NAME_AS1 -SubnetName $SUBNET_NAME -OpenPorts 22 -Size $VM_SIZE -Credential (Get-Credential -UserName $ADMIN_USERNAME -Message "Enter Password")
   ```

   Expected Output:
   ```
   ResourceGroupName        : iatd_labs_09a_rg
   Id                       : /subscriptions/your-subscription-id/resourceGroups/iatd_labs_09a_rg/providers/Microsoft.Compute/virtualMachines/iatd_labs_09a_vm_as1
   VmId                     : 12345678-abcd-1234-efgh-123456789012
   Name                     : iatd_labs_09a_vm_as1
   Type                     : Microsoft.Compute/virtualMachines
   Location                 : australiaeast
   ...
   ```

9. **Create VM for AS2:**

   ```powershell
   New-AzVm -ResourceGroupName $RESOURCE_GROUP -Name $VM_AS2_NAME -Location $LOCATION -VirtualNetworkName $VNET_NAME_AS2 -SubnetName $SUBNET_NAME -OpenPorts 22 -Size $VM_SIZE -Credential (Get-Credential -UserName $ADMIN_USERNAME -Message "Enter Password")
   ```

   Expected Output:
   ```
   ResourceGroupName        : iatd_labs_09a_rg
   Id                       : /subscriptions/your-subscription-id/resourceGroups/iatd_labs_09a_rg/providers/Microsoft.Compute/virtualMachines/iatd_labs_09a_vm_as2
   VmId                     : 87654321-abcd-1234-efgh-123456789012
   Name                     : iatd_labs_09a_vm_as2
   Type                     : Microsoft.Compute/virtualMachines
   Location                 : australiaeast
   ...
   ```

10. **Verify VNets and VMs (PowerShell):**

    ```powershell
    Get-AzVirtualNetwork -ResourceGroupName $RESOURCE_GROUP
    Get-AzVM -ResourceGroupName $RESOURCE_GROUP
    ```

    Expected Output:
    ```
    Name              : iatd_labs_09a_vnet_as1
    ResourceGroupName : iatd_labs_09a_rg
    Location          : australiaeast
    AddressSpace      : {172.16.1.0/24}
    Subnets           : {iatd_labs_09a_subnet}
    
    Name              : iatd_labs_09a_vnet_as2
    ResourceGroupName : iatd_labs_09a_rg
    Location          : australiaeast
    AddressSpace      : {172.16.2.0/24}
    Subnets           : {iatd_labs_09a_subnet}
    
    ResourceGroupName : iatd_labs_09a_rg
    Name              : iatd_labs_09a_vm_as1
    Location          : australiaeast
    
    ResourceGroupName : iatd_labs_09a_rg
    Name              : iatd_labs_09a_vm_as2
    Location          : australiaeast
    ```

### Part 2: Conceptual Understanding of AS and Neighbors (Discussion)

1. **Conceptual Explanation:**
   * **Autonomous System (AS):** Explain ASes as independent networks. In this lab, `$VNET_NAME_AS1` and `$VNET_NAME_AS2` represent two ASes. They are isolated networks.
   * **BGP Neighbors:** Explain that BGP routers in different ASes (eBGP) or within the same AS (iBGP) establish neighbor relationships to exchange routing information. The VMs in this lab will simulate routers.
   * **eBGP vs iBGP (Conceptual):** Briefly introduce eBGP (between ASes) and iBGP (within an AS). This lab focuses on eBGP.

### Part 3: Simulating Neighbor Adjacency (Bash - on the VMs)

1. **Connect to VM_AS1 (Bash):**
   * Find the public IP of `$VM_AS1_NAME`:
   
   ```powershell
   $vm1PublicIp = (Get-AzPublicIpAddress -ResourceGroupName $RESOURCE_GROUP | Where-Object {$_.Name -like "*$VM_AS1_NAME*"}).IpAddress
   $vm1PrivateIp = (Get-AzNetworkInterface -ResourceGroupName $RESOURCE_GROUP | Where-Object {$_.Name -like "*$VM_AS1_NAME*"}).IpConfigurations[0].PrivateIpAddress
   Write-Host "VM AS1 Public IP: $vm1PublicIp"
   Write-Host "VM AS1 Private IP: $vm1PrivateIp"
   ```
   
   Expected Output:
   ```
   VM AS1 Public IP: 20.1.2.3
   VM AS1 Private IP: 172.16.1.4
   ```
   
   * Connect to the VM:
   ```bash
   ssh cloudadmin@<VM_AS1_PUBLIC_IP>
   ```

2. **Connect to VM_AS2 (Bash):**
   * Find the public IP of `$VM_AS2_NAME`:
   
   ```powershell
   $vm2PublicIp = (Get-AzPublicIpAddress -ResourceGroupName $RESOURCE_GROUP | Where-Object {$_.Name -like "*$VM_AS2_NAME*"}).IpAddress
   $vm2PrivateIp = (Get-AzNetworkInterface -ResourceGroupName $RESOURCE_GROUP | Where-Object {$_.Name -like "*$VM_AS2_NAME*"}).IpConfigurations[0].PrivateIpAddress
   Write-Host "VM AS2 Public IP: $vm2PublicIp"
   Write-Host "VM AS2 Private IP: $vm2PrivateIp"
   ```
   
   Expected Output:
   ```
   VM AS2 Public IP: 20.1.2.4
   VM AS2 Private IP: 172.16.2.4
   ```
   
   * Connect to the VM:
   ```bash
   ssh cloudadmin@<VM_AS2_PUBLIC_IP>
   ```

3. **Run a Simple Command in each VM (Bash):**
   * Ping each VM.
   * VM AS1: 
   ```bash
   ping <VM_AS2_Private_IP>
   ```
   * VM AS2: 
   ```bash
   ping <VM_AS1_Private_IP>
   ```
   * This verifies connectivity between the VMs.

4. **Conceptual: Simulate BGP Peering (Discussion)**
   * Explain that to establish a BGP peering, you need to configure:
     * **The Neighbor's IP Address:** The IP of the other BGP router.
     * **The AS Number:** Each AS has a unique AS number. (Next lab will demonstrate this).
   * In a real-world scenario, you'd use a BGP daemon (like Quagga or FRR) on each VM. We're simulating the setup conceptually here.

### Part 4: Cleanup

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
   Resource group iatd_labs_09a_rg has been deleted successfully.
   ```

### Post-Lab Summary

In this lab, you:
1. Created two virtual networks representing separate Autonomous Systems
2. Deployed VMs in each network to simulate BGP routers
3. Established basic connectivity between the VMs
4. Learned about the concept of Autonomous Systems and BGP neighbor relationships

This lab provided the foundation for understanding BGP peering by introducing the concept of Autonomous Systems and neighbor relationships. In the next lab, you will explore the differences between iBGP and eBGP peering, and set up basic BGP peering between the VMs.
