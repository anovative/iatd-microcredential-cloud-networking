## IATD Microcredential Cloud Networking: Lab 9b - BGP Fundamentals: iBGP vs. eBGP and Basic Peering Configuration

**Objective:** To differentiate between iBGP and eBGP peering, and set up basic BGP peering between VMs using simulated BGP commands.

**Estimated Time:** 45 - 60 minutes

**Prerequisites:**

1. **Azure Subscription:** An active Azure subscription.
2. **Azure Cloud Shell:** Access to Azure Cloud Shell (Bash or PowerShell).
3. **Basic Understanding:** Familiarity with Virtual Networks, subnets, and basic networking concepts.
4. **Completed Lab 9a:** This lab builds on concepts from Lab 9a but uses new resources.

**Let's get started!**

### Lab Conventions

* **Azure Cloud Shell:** All PowerShell interactions will occur within the Azure Cloud Shell environment.
* **Naming Conventions:** Resource names follow the convention `iatd_labs_09b_*`.
* **IP Address Range:** The `172.16.x.x` range will be used.
* **Location:** Choose a consistent Azure region (e.g., `australiaeast`).

#### Resource Naming

* **Resource Group:** `iatd_labs_09b_rg`
* **Virtual Network AS1:** `iatd_labs_09b_vnet_as1`
* **Virtual Network AS2:** `iatd_labs_09b_vnet_as2`
* **Subnet:** `iatd_labs_09b_subnet`
* **VM AS1:** `iatd_labs_09b_vm_as1`
* **VM AS2:** `iatd_labs_09b_vm_as2`

### Part 1: Setting up the Environment 

1. **Open Cloud Shell:** Open Azure Cloud Shell and select **PowerShell**.

2. **Define Variables:**

   ```powershell
   $RESOURCE_GROUP = "iatd_labs_09b_rg"  # Using lab 9b resource group
   $LOCATION = "australiaeast" # Replace with your preferred region
   $VNET_NAME_AS1 = "iatd_labs_09b_vnet_as1"
   $VNET_NAME_AS2 = "iatd_labs_09b_vnet_as2"
   $SUBNET_NAME = "iatd_labs_09b_subnet"
   $SUBNET_PREFIX = "172.16.1.0/24" # Confirm this is the prefix in both VNets
   $VM_AS1_NAME = "iatd_labs_09b_vm_as1"
   $VM_AS2_NAME = "iatd_labs_09b_vm_as2"
   $ADMIN_USERNAME = "cloudadmin"
   $ADMIN_PASSWORD = "YourStrongPassword123!"  # REPLACE with your password
   ```

3. **Get VM Private IPs:**

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

### Part 2: Conceptual: iBGP vs eBGP (Discussion)

1. **iBGP vs. eBGP (Explanation):**
   * **eBGP (External BGP):** Explain that eBGP is used for peering between routers in *different* Autonomous Systems (ASes). It's how networks exchange routing information with each other.
   * **iBGP (Internal BGP):** Explain that iBGP is used within an *AS* for route distribution between internal BGP routers.
   * **Hop Count Limit:** In eBGP, the TTL is set to 1 to avoid routing loops if packets get mis-routed.
   * **Key Differences:**
     * **AS Numbers:** eBGP uses different ASNs; iBGP uses the same ASN.
     * **Hop Count:** eBGP has a TTL=1; iBGP does not.
     * **Route Advertisement:** eBGP advertises routes to external ASes; iBGP is used to synchronize routes internally.
   * **Full Mesh/Route Reflectors:** In iBGP, you need a full mesh of peering or route reflectors, to avoid a situation where a route is not exchanged.
   * The VMs in this Lab will demonstrate a simplified version of how BGP peering can be established between two different Autonomous Systems.

### Part 3: Simulating eBGP Peering (Conceptual and Shell)

1. **Conceptual - Simulate ASN and Peering:**
   * Explain that eBGP peering requires configuring the BGP neighbor (in this case, the other VM's private IP) and its ASN.
   * **Simulate ASN using a comment.**
     * `VM_AS1 AS_NUMBER = 65001`
     * `VM_AS2 AS_NUMBER = 65002`
     * These values are for demonstration only and don't correspond to any real-world assignments.

2. **Connect to VM_AS1 (Bash):**
   * Find the public IP of `$VM_AS1_NAME`:
   
   ```powershell
   $vm1PublicIp = (Get-AzPublicIpAddress -ResourceGroupName $RESOURCE_GROUP | Where-Object {$_.Name -like "*$VM_AS1_NAME*"}).IpAddress
   Write-Host "VM AS1 Public IP: $vm1PublicIp"
   ```
   
   Expected Output:
   ```
   VM AS1 Public IP: 20.1.2.3
   ```
   
   * Connect to the VM:
   ```bash
   ssh cloudadmin@<VM_AS1_PUBLIC_IP>
   ```

3. **Connect to VM_AS2 (Bash):**
   * Find the public IP of `$VM_AS2_NAME`:
   
   ```powershell
   $vm2PublicIp = (Get-AzPublicIpAddress -ResourceGroupName $RESOURCE_GROUP | Where-Object {$_.Name -like "*$VM_AS2_NAME*"}).IpAddress
   Write-Host "VM AS2 Public IP: $vm2PublicIp"
   ```
   
   Expected Output:
   ```
   VM AS2 Public IP: 20.1.2.4
   ```
   
   * Connect to the VM:
   ```bash
   ssh cloudadmin@<VM_AS2_PUBLIC_IP>
   ```

4. **Simulate BGP Commands (Bash - on the VMs):**
   * **On VM_AS1 (Bash):**
     * The following command will be run on VM_AS1
     * `neighbor <VM_AS2_Private_IP> remote-as 65002`  (Simulates setting up the eBGP peering with VM_AS2)
   * **On VM_AS2 (Bash):**
     * The following command will be run on VM_AS2
     * `neighbor <VM_AS1_Private_IP> remote-as 65001` (Simulates setting up the eBGP peering with VM_AS1)

5. **Simulate Route Advertisement and Verification (Bash)**

   * **Conceptual - Route Advertisement** Explain that each VM must *advertise* its own subnet.
     * For VM_AS1 (which is on 172.16.1.0/24), the BGP configuration would include the 'network 172.16.1.0/24' command. This command advertises a network.
     * For VM_AS2 (which is on 172.16.2.0/24), the BGP configuration would include the 'network 172.16.2.0/24' command.

6. **Simulate Verification of Peering and Route (Bash):**
   * **Bash (on VM_AS1):**
     * Simulate checking the status of the BGP session with the neighbor. This can be done using a command like `show ip bgp summary`. Because we are using a simplified view, this command will return nothing.
     * Simulate checking the BGP route table. The command `ip route show` can be used to see if the route to 172.16.2.0/24 was advertised. The output will show that the route is available.
     * To verify connectivity:

     ```bash
     ping <VM_AS2_Private_IP>
     ```

     Expected Output:
     ```
     PING 172.16.2.4 (172.16.2.4) 56(84) bytes of data.
     64 bytes from 172.16.2.4: icmp_seq=1 ttl=64 time=0.935 ms
     64 bytes from 172.16.2.4: icmp_seq=2 ttl=64 time=0.652 ms
     64 bytes from 172.16.2.4: icmp_seq=3 ttl=64 time=0.495 ms
     64 bytes from 172.16.2.4: icmp_seq=4 ttl=64 time=0.428 ms
     ```
     
   * **Bash (on VM_AS2):**
     * Repeat the commands to check the BGP session and routes.
     * To verify connectivity:

     ```bash
     ping <VM_AS1_Private_IP>
     ```

     Expected Output:
     ```
     PING 172.16.1.4 (172.16.1.4) 56(84) bytes of data.
     64 bytes from 172.16.1.4: icmp_seq=1 ttl=64 time=0.935 ms
     64 bytes from 172.16.1.4: icmp_seq=2 ttl=64 time=0.652 ms
     64 bytes from 172.16.1.4: icmp_seq=3 ttl=64 time=0.495 ms
     64 bytes from 172.16.1.4: icmp_seq=4 ttl=64 time=0.428 ms
     ```

### Part 4: Clean Up Resources

1. **Delete Resource Group (PowerShell):**

   ```powershell
   # Clean up resources when you're done
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
   Resource group iatd_labs_09b_rg has been deleted successfully.
   ```

> **Important Note:** This lab creates new resources with the `iatd_labs_09b_*` naming convention. Make sure to clean up these resources as instructed above to avoid unnecessary charges. The resources from lab-009a are not affected by this cleanup process.

### Post-Lab Summary

In this lab, you:
1. Learned the differences between iBGP and eBGP peering
2. Simulated eBGP peering between two VMs in different Autonomous Systems
3. Understood the concept of route advertisement in BGP
4. Verified connectivity between the peered systems

This lab built on the foundation established in Lab 9a by introducing the practical aspects of BGP peering between different Autonomous Systems. In the next lab, you will explore BGP path attributes and route selection criteria.
