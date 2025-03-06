## IATD Microcredential Cloud Networking: Lab 6 - Configuring Route Prioritization with Overlapping Routes

**Objective:** This lab demonstrates how Azure prioritizes routes when User-Defined Routes (UDRs) have overlapping address prefixes. You will create two UDRs with the same destination, but different prefix lengths, to observe the "longest prefix match" rule in action.

**Estimated Time:** 60 - 90 minutes

**Prerequisites:**

1.  **Azure Subscription:** An active Azure subscription.
2.  **Azure Cloud Shell:** Access to Azure Cloud Shell (Bash or PowerShell).
3.  **Basic Understanding:** Familiarity with Virtual Networks, subnets, and UDRs.

**Let's get started!**

### Lab Conventions

*   **Azure Cloud Shell:** All PowerShell and CLI interactions will occur within the Azure Cloud Shell environment.
*   **Naming Conventions:** Resource names follow the convention `iatd_labs_06_*`.
*   **IP Address Range:** The `172.16.x.x` range will be used.
*   **Location:** Choose a consistent Azure region (e.g., `australiaeast`).

#### Resource Naming

*   **Resource Group:** `iatd_labs_06_rg`
*   **Virtual Network:** `iatd_labs_06_vnet`
*   **Subnet:** `iatd_labs_06_subnet`
*   **Network Interface:** `iatd_labs_06_nic`
*   **VM:** `iatd_labs_06_vm`
*   **Public IP:** `iatd_labs_06_vm_pip`
*   **Route Table:** `iatd_labs_06_rt`
*   **UDR /16:** `iatd_labs_06_route_16`
*   **UDR /24:** `iatd_labs_06_route_24`
*   **Destination IP for testing:** `10.0.0.5` (This is an example and can be anything not in the VNet's address space)

### Part 1: Creating the Virtual Network, Subnet, and VM

1.  **Open Cloud Shell:** Open Azure Cloud Shell and select **PowerShell**.

2.  **Define Variables:**

    ```powershell
    $resourceGroupName = "iatd_labs_06_rg"
    $location = "australiaeast" # Replace with your preferred region

    # VNet and Subnet Variables
    $vnetName = "iatd_labs_06_vnet"
    $vnetAddressPrefix = "172.16.0.0/16"
    $subnetName = "iatd_labs_06_subnet"
    $subnetPrefix = "172.16.0.0/24"

    # VM Variables
    $vmName = "iatd_labs_06_vm"
    $vmSize = "Standard_DS1_v2" # Change if required
    $adminUsername = "cloudadmin"
    $adminPassword = "YourStrongPassword123!" # Replace with a strong password
    ```

3.  **Create Resource Group (PowerShell):**

    ```powershell
    New-AzResourceGroup -Name $resourceGroupName -Location $location -ErrorAction SilentlyContinue
    ```

4.  **Create Virtual Network and Subnet (PowerShell):**

    ```powershell
    # Create VNet
    $vnet = New-AzVirtualNetwork -Name $vnetName -ResourceGroupName $resourceGroupName -Location $location -AddressPrefix $vnetAddressPrefix

    # Create Subnet
    $subnet = Add-AzVirtualNetworkSubnetConfig -Name $subnetName -VirtualNetwork $vnet -AddressPrefix $subnetPrefix
    $vnet | Set-AzVirtualNetwork
    ```

5.  **Create Public IP Address (CLI):**

    *   Open Cloud Shell and select **Bash**.

    ```bash
    az network public-ip create \
      --resource-group $resourceGroupName \
      --name iatd_labs_06_vm_pip \
      --allocation-method Static \
      --sku Standard \
      --location $location
    ```

6.  **Create VM (CLI):**

    ```bash
    az vm create \
      --resource-group $resourceGroupName \
      --name $vmName \
      --image UbuntuLTS \
      --admin-username $adminUsername \
      --admin-password "$adminPassword" \
      --vnet-name $vnetName \
      --subnet $subnetName \
      --public-ip-address iatd_labs_06_vm_pip \
      --size $vmSize \
      --open-ports 22 80 \
      --location $location
    ```

### Part 2: Creating the Route Table and UDRs

1.  **Create Route Table (Portal):**

    *   Go to the Azure Portal ([https://portal.azure.com/](https://portal.azure.com/)).
    *   Search for "Route tables" and select.
    *   Click "+ Create".
        *   **Subscription:** Your subscription.
        *   **Resource group:** `iatd_labs_06_rg`.
        *   **Region:** Your region.
        *   **Name:** `iatd_labs_06_rt`.
        *   Click "Review + create" then "Create".

2.  **Create UDR /16 (Portal):**

    *   Go to the `iatd_labs_06_rt` Route Table that you created.
    *   In the "Settings" blade, click on "Routes".
    *   Click "+ Add".
        *   **Route name:** `iatd_labs_06_route_16`.
        *   **Address prefix:** `10.0.0.0/16`.
        *   **Next hop type:** "Internet".
        *   Click "Add".

3.  **Create UDR /24 (Portal):**

    *   Still in the `iatd_labs_06_rt` Route Table, click "+ Add".
        *   **Route name:** `iatd_labs_06_route_24`.
        *   **Address prefix:** `10.0.0.0/24`.
        *   **Next hop type:** "Internet".
        *   Click "Add".

4.  **Associate Route Table to Subnet (Portal):**

    *   Search for "Virtual networks" and select `iatd_labs_06_vnet`.
    *   In the "Settings" blade, click on "Subnets".
    *   Click on the `iatd_labs_06_subnet`.
    *   Under "Route table", select `iatd_labs_06_rt`.
    *   Click "Save".

### Part 3: Testing Route Prioritization

1.  **Get VM Private IP (PowerShell):** We need the private IP to test routes *from inside* the VM.

    ```powershell
    $vm = Get-AzVM -Name $vmName -ResourceGroupName $resourceGroupName
    $vmPrivateIP = ($vm.PrivateIPAddresses)[0]
    Write-Host "VM Private IP: $vmPrivateIP"
    ```

2.  **Connect to the VM via SSH:**

    *   Use an SSH client (e.g., PuTTY, Terminal) to connect to the VM's public IP using the `cloudadmin` username and your password. You may need to configure your local network security group for SSH traffic.
    *   `ssh cloudadmin@<your_vm_public_ip>`

3.  **Test Connectivity from within the VM (Bash):**

    *   Use the `traceroute` command (or `tracert` on Windows if you were using a Windows VM) to test the route to `10.0.0.5`.  This command will show the path the packets take.

    ```bash
    traceroute 10.0.0.5
    ```

    **Expected Output:**
    ```
    traceroute to 10.0.0.5 (10.0.0.5), 30 hops max, 60 byte packets
     1  * * *
     2  * * *
     3  * * *
    ```
    This output shows the traffic attempting to reach the Internet (timing out at each hop) via the more specific /24 route, which takes precedence over the /16 route as expected.

    **If you do *not* see the expected outcome, consider:**
        *   **Route Propagation:** Verify that the route table is correctly associated with the subnet.
        *   **Security Groups:** Ensure the NSG associated with the NIC does not block the traffic.
        *   **Network Connectivity:**  In some cases, the traceroute might show hops. This doesn't necessarily mean the longest prefix match isn't working. The important thing is *where* the traffic is routed.

4.  **Explanation of Route Prioritization:** The key takeaway is that the `/24` route (a more specific prefix) takes precedence over the `/16` route. Azure uses the "longest prefix match" principle. If the destination IP address falls within both route prefixes, the route with the longer prefix (more specific) is chosen.

### Post-Lab: Cleanup

To avoid unnecessary costs, clean up the created resources using any of these methods:

1.  **Azure Portal:**
    * Navigate to Resource Groups
    * Select `iatd_labs_06_rg`
    * Click 'Delete resource group'
    * Type the resource group name to confirm
    * Click 'Delete'

2.  **PowerShell:**
    ```powershell
    Remove-AzResourceGroup -Name $resourceGroupName -Force
    ```
    Expected Output:
    ```
    True
    ```

3.  **Azure CLI:**
    ```bash
    az group delete --name $resourceGroupName --yes
    ```
    Expected Output:
    ```
    No output indicates successful deletion
    ```

Verify Cleanup:
* Check the Azure Portal to ensure the resource group and all resources are deleted
* Run `Get-AzResourceGroup` or `az group list` to verify the resource group no longer exists

**Congratulations!** You've successfully demonstrated route prioritization with overlapping prefixes. You've learned how Azure's "longest prefix match" rule determines which route is used when multiple UDRs apply to the same destination.
