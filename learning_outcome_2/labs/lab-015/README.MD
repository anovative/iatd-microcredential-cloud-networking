**IATD Microcredential Cloud Networking: Lab 15 - Wide Area Network (WAN) Fundamentals (Alternative - VNet Peering - CLI/Portal/PowerShell Balance)**

**Objective:** Explore the hub and spoke WAN architecture in Azure using Virtual Network Peering as a connectivity method, providing instructions using the Azure Portal, CLI, and PowerShell.

**Scenario:** Your organization is expanding its Azure footprint and needs to implement a hub and spoke network architecture to centralize connectivity and security services. This lab demonstrates how to create and configure a hub and spoke topology using Azure Virtual Network Peering, with implementation options via the Azure Portal, CLI, and PowerShell.

**Outcomes:**
* Creation of hub and spoke virtual networks
* Configuration of VNet peering between hub and spoke networks
* Verification of connectivity between networks
* Understanding of traffic flow in a hub and spoke architecture
* Experience with multiple implementation methods (Portal, CLI, PowerShell)

**Estimated Time:** 60 - 75 minutes

**Prerequisites:**
1. **Azure Subscription:** An active Azure subscription is required.
2. **Basic Azure Knowledge:** Familiarity with the Azure Portal, Resource Groups, and basic networking concepts.

### Lab Conventions

* **Naming Conventions:** Resource names follow the convention `iatd_labs_15_*`.
* **IP Address Range:** The `172.16.x.x` range will be used. Ensure non-overlapping address spaces for Hub and Spoke VNets.
* **Location:** Choose a consistent Azure region.

#### Resource Naming

* **Resource Group (Hub):** `iatd_labs_15_hub_rg`
* **Virtual Network (Hub):** `iatd_labs_15_hub_vnet`
* **Subnet (Hub):** `iatd_labs_15_hub_subnet`
* **Resource Group (Spoke 1):** `iatd_labs_15_spoke1_rg`
* **Virtual Network (Spoke 1):** `iatd_labs_15_spoke1_vnet`
* **Subnet (Spoke 1):** `iatd_labs_15_spoke1_subnet`

### Part 1: Hub and Spoke Network Creation (Portal, CLI & PowerShell)

In this part, you'll create the core Hub and Spoke virtual networks using the Azure Portal, CLI, and PowerShell.

**1.  Resource Group Creation:**

*   **Azure Portal:**
    1.  Search for "Resource groups" in the Azure Portal.
    2.  Click **Create**.
    3.  Enter the details:
        *   Subscription: Your subscription
        *   Resource group: `iatd_labs_15_hub_rg` for the Hub, and `iatd_labs_15_spoke1_rg` for Spoke 1
        *   Region: Your chosen region.
    4.  Click **Review + create**, then **Create**.

*   **Azure CLI:**

    ```bash
    # Create Hub Resource Group
    RESOURCE_GROUP_HUB="iatd_labs_15_hub_rg"
    LOCATION="australiaeast" # Or your location

    az group create --name $RESOURCE_GROUP_HUB --location $LOCATION --output json --only-show-errors

    # Create Spoke 1 Resource Group
    RESOURCE_GROUP_SPOKE1="iatd_labs_15_spoke1_rg"

    az group create --name $RESOURCE_GROUP_SPOKE1 --location $LOCATION --output json --only-show-errors
    ```

*   **Azure PowerShell:**

    ```powershell
    # Create Hub Resource Group
    $hubRgName = "iatd_labs_15_hub_rg"
    $location = "AustraliaEast" # Or your location

    New-AzResourceGroup -Name $hubRgName -Location $location -ErrorAction SilentlyContinue

    # Create Spoke 1 Resource Group
    $spoke1RgName = "iatd_labs_15_spoke1_rg"

    New-AzResourceGroup -Name $spoke1RgName -Location $location -ErrorAction SilentlyContinue
    ```

**2.  Virtual Network and Subnet Creation:**

*   **Azure Portal:**
    1.  Search for "Virtual networks" in the Azure Portal.
    2.  Click **Create**.
    3.  **Basics Tab:**
        *   Subscription: Your subscription
        *   Resource group: `iatd_labs_15_hub_rg` (or `iatd_labs_15_spoke1_rg` for Spoke)
        *   Name: `iatd_labs_15_hub_vnet` (or `iatd_labs_15_spoke1_vnet` for Spoke)
        *   Region: Your chosen region.
    4.  **IP Addresses Tab:**
        *   IPv4 address space: `172.16.0.0/16` for Hub and `172.16.1.0/16` for Spoke (ENSURE THESE ARE DIFFERENT!)
        *   Subnet name: `iatd_labs_15_hub_subnet` (or `iatd_labs_15_spoke1_subnet` for Spoke)
        *   Subnet address range: `172.16.0.0/24` (or `172.16.1.0/24` for Spoke)
    5.  Click **Review + create**, then **Create**.

*   **Azure CLI:**

    ```bash
    # Hub VNet and Subnet
    RESOURCE_GROUP_HUB="iatd_labs_15_hub_rg"
    VNET_NAME_HUB="iatd_labs_15_hub_vnet"
    ADDRESS_PREFIX_HUB="172.16.0.0/16"
    SUBNET_NAME_HUB="iatd_labs_15_hub_subnet"
    SUBNET_PREFIX_HUB="172.16.0.0/24"
    LOCATION="australiaeast" # Or your location

    az network vnet create \
      --resource-group $RESOURCE_GROUP_HUB \
      --name $VNET_NAME_HUB \
      --address-prefixes $ADDRESS_PREFIX_HUB \
      --subnet-name $SUBNET_NAME_HUB \
      --subnet-prefixes $SUBNET_PREFIX_HUB \
      --location $LOCATION

    # Spoke 1 VNet and Subnet
    RESOURCE_GROUP_SPOKE1="iatd_labs_15_spoke1_rg"
    VNET_NAME_SPOKE1="iatd_labs_15_spoke1_vnet"
    ADDRESS_PREFIX_SPOKE1="172.16.1.0/16"
    SUBNET_NAME_SPOKE1="iatd_labs_15_spoke1_subnet"
    SUBNET_PREFIX_SPOKE1="172.16.1.0/24"

    az network vnet create \
      --resource-group $RESOURCE_GROUP_SPOKE1 \
      --name $VNET_NAME_SPOKE1 \
      --address-prefixes $ADDRESS_PREFIX_SPOKE1 \
      --subnet-name $SUBNET_NAME_SPOKE1 \
      --subnet-prefixes $SUBNET_PREFIX_SPOKE1 \
      --location $LOCATION
    ```

*   **Azure PowerShell:**

    ```powershell
    # Hub VNet and Subnet
    $hubRgName = "iatd_labs_15_hub_rg"
    $hubVnetName = "iatd_labs_15_hub_vnet"
    $hubSubnetName = "iatd_labs_15_hub_subnet"
    $hubAddressPrefix = "172.16.0.0/16"
    $hubSubnetPrefix = "172.16.0.0/24"
    $location = "AustraliaEast" # Or your location

    $hubSubnet = New-AzVirtualNetworkSubnetConfig -Name $hubSubnetName -AddressPrefix $hubSubnetPrefix
    New-AzVirtualNetwork -Name $hubVnetName -ResourceGroupName $hubRgName -Location $location -AddressPrefix $hubAddressPrefix -Subnet $hubSubnet

    # Spoke 1 VNet and Subnet
    $spoke1RgName = "iatd_labs_15_spoke1_rg"
    $spoke1VnetName = "iatd_labs_15_spoke1_vnet"
    $spoke1SubnetName = "iatd_labs_15_spoke1_subnet"
    $spoke1AddressPrefix = "172.16.1.0/16"
    $spoke1SubnetPrefix = "172.16.1.0/24"

    $spoke1Subnet = New-AzVirtualNetworkSubnetConfig -Name $spoke1SubnetName -AddressPrefix $spoke1SubnetPrefix
    New-AzVirtualNetwork -Name $spoke1VnetName -ResourceGroupName $spoke1RgName -Location $location -AddressPrefix $spoke1AddressPrefix -Subnet $spoke1Subnet
    ```

### Part 2: Configuring Virtual Network Peering (Portal, CLI & PowerShell)

This part demonstrates how to configure VNet Peering using the Azure Portal, Azure CLI, and PowerShell.

1.  **Get Resource IDs (CLI & PowerShell Prep):**

    Before you can create the peerings with the CLI and PowerShell, you'll need the *Resource IDs* of both virtual networks.

    *   **Azure CLI:**

        ```bash
        # Get Hub VNet Resource ID
        HUB_VNET_ID=$(az network vnet show \
            --resource-group $RESOURCE_GROUP_HUB \
            --name $VNET_NAME_HUB \
            --query id \
            --output tsv)

        echo "Hub VNet ID: $HUB_VNET_ID"

        # Get Spoke 1 VNet Resource ID
        SPOKE1_VNET_ID=$(az network vnet show \
            --resource-group $RESOURCE_GROUP_SPOKE1 \
            --name $VNET_NAME_SPOKE1 \
            --query id \
            --output tsv)

        echo "Spoke 1 VNet ID: $SPOKE1_VNET_ID"
        ```

    *   **Azure PowerShell:**

        ```powershell
        # Get Hub VNet Resource ID
        $hubVnet = Get-AzVirtualNetwork -Name $hubVnetName -ResourceGroupName $hubRgName
        $HUB_VNET_ID = $hubVnet.Id
        Write-Host "Hub VNet ID: $($HUB_VNET_ID)"

        # Get Spoke 1 VNet Resource ID
        $spoke1Vnet = Get-AzVirtualNetwork -Name $spoke1VnetName -ResourceGroupName $spoke1RgName
        $SPOKE1_VNET_ID = $spoke1Vnet.Id
        Write-Host "Spoke 1 VNet ID: $($SPOKE1_VNET_ID)"
        ```

2.  **Create Peerings:**

    *   **(Azure Portal - No Changes to Steps Themselves)** Just make sure students are using the correct `iatd_labs_15_*` resource names when selecting resources.
    *   **Azure CLI:**

        ```bash
        # Create Peering: Hub to Spoke 1
        az network vnet peering create \
          --name HubToSpoke1 \
          --resource-group $RESOURCE_GROUP_HUB \
          --vnet-name $VNET_NAME_HUB \
          --remote-vnet $SPOKE1_VNET_ID \
          --allow-virtual-network-access \
          --allow-forwarded-traffic

        # Create Peering: Spoke 1 to Hub
        az network vnet peering create \
          --name Spoke1ToHub \
          --resource-group $RESOURCE_GROUP_SPOKE1 \
          --vnet-name $VNET_NAME_SPOKE1 \
          --remote-vnet $HUB_VNET_ID \
          --allow-virtual-network-access \
          --allow-forwarded-traffic
        ```

    *   **Azure PowerShell:**

        ```powershell
        # Create Peering: Hub to Spoke 1
        Add-AzVirtualNetworkPeering `
          -Name "HubToSpoke1" `
          -VirtualNetwork $hubVnet `
          -RemoteVirtualNetworkId $SPOKE1_VNET_ID `
          -AllowVirtualNetworkAccess `
          -AllowForwardedTraffic

        # Create Peering: Spoke 1 to Hub
        Add-AzVirtualNetworkPeering `
          -Name "Spoke1ToHub" `
          -VirtualNetwork $spoke1Vnet `
          -RemoteVirtualNetworkId $HUB_VNET_ID `
          -AllowVirtualNetworkAccess `
          -AllowForwardedTraffic
        ```

    **Note:** With CLI and PowerShell, default settings apply when you don't explicitly set `-AllowGatewayTransit` or `-UseRemoteGateways`. I have intentionally omitted them for brevity and because we are *not* using gateways in this base scenario. To fully disable "gateway transit" and "use remote gateways", add parameters  `-AllowGatewayTransit:$false` and `-UseRemoteGateways:$false`.

3.  **Verification (Portal):**

### Part 3: Validation - Testing Connectivity

In this part, you'll create virtual machines in both the hub and spoke networks to test connectivity between them.

**1. Create Test VMs:**

* **Azure Portal:**
  1. Search for "Virtual machines" in the Azure Portal.
  2. Click **Create**.
  3. **Basics Tab:**
     * Subscription: Your subscription
     * Resource group: `iatd_labs_15_hub_rg` for hub VM, `iatd_labs_15_spoke1_rg` for spoke VM
     * VM name: `iatd_labs_15_hub_vm` or `iatd_labs_15_spoke1_vm`
     * Region: Same as your VNets
     * Image: Ubuntu Server 20.04 LTS
     * Size: Standard_B1s (or any small size)
     * Authentication: Password
     * Username: azureuser
     * Password: Create a secure password and note it down
  4. **Networking Tab:**
     * Virtual network: `iatd_labs_15_hub_vnet` or `iatd_labs_15_spoke1_vnet`
     * Subnet: `iatd_labs_15_hub_subnet` or `iatd_labs_15_spoke1_subnet`
     * Public IP: Create new
  5. Complete the wizard and create both VMs.

* **Azure CLI:**
  ```bash
  # Create Hub VM
  az vm create \
    --resource-group $RESOURCE_GROUP_HUB \
    --name iatd_labs_15_hub_vm \
    --image UbuntuLTS \
    --admin-username azureuser \
    --admin-password "YourSecurePassword123!" \
    --vnet-name $VNET_NAME_HUB \
    --subnet $SUBNET_NAME_HUB \
    --size Standard_B1s \
    --public-ip-address iatd_labs_15_hub_vm_pip
    
  # Create Spoke VM
  az vm create \
    --resource-group $RESOURCE_GROUP_SPOKE1 \
    --name iatd_labs_15_spoke1_vm \
    --image UbuntuLTS \
    --admin-username azureuser \
    --admin-password "YourSecurePassword123!" \
    --vnet-name $VNET_NAME_SPOKE1 \
    --subnet $SUBNET_NAME_SPOKE1 \
    --size Standard_B1s \
    --public-ip-address iatd_labs_15_spoke1_vm_pip
  ```

* **Azure PowerShell:**
  ```powershell
  # Create Hub VM
  $hubVmName = "iatd_labs_15_hub_vm"
  $cred = New-Object PSCredential "azureuser", (ConvertTo-SecureString "YourSecurePassword123!" -AsPlainText -Force)
  
  New-AzVM -ResourceGroupName $hubRgName -Name $hubVmName -Location $location `
    -VirtualNetworkName $hubVnetName -SubnetName $hubSubnetName `
    -Image UbuntuLTS -Size Standard_B1s -Credential $cred
  
  # Create Spoke VM
  $spokeVmName = "iatd_labs_15_spoke1_vm"
  
  New-AzVM -ResourceGroupName $spoke1RgName -Name $spokeVmName -Location $location `
    -VirtualNetworkName $spoke1VnetName -SubnetName $spoke1SubnetName `
    -Image UbuntuLTS -Size Standard_B1s -Credential $cred
  ```

**2. Test Connectivity Between VMs:**

* **Connect to the Hub VM:**
  1. In the Azure Portal, navigate to the Hub VM.
  2. Click **Connect** and select **SSH**.
  3. Use the public IP address to connect via SSH.
  
  ```bash
  ssh azureuser@<hub-vm-public-ip>
  ```

* **Test Connectivity to Spoke VM:**
  1. Once connected to the Hub VM, ping the private IP of the Spoke VM.
  
  ```bash
  # Get the private IP of the Spoke VM first
  # You can find this in the Azure Portal or use CLI/PowerShell
  ping <spoke-vm-private-ip>
  ```
  
  **Expected Output:**
  ```
  PING 172.16.1.4 (172.16.1.4) 56(84) bytes of data.
  64 bytes from 172.16.1.4: icmp_seq=1 ttl=64 time=1.20 ms
  64 bytes from 172.16.1.4: icmp_seq=2 ttl=64 time=0.91 ms
  64 bytes from 172.16.1.4: icmp_seq=3 ttl=64 time=0.88 ms
  ```

**3. Verify Peering Status:**

* **Azure CLI:**
  ```bash
  # Check Hub to Spoke peering status
  az network vnet peering show \
    --resource-group $RESOURCE_GROUP_HUB \
    --vnet-name $VNET_NAME_HUB \
    --name HubToSpoke1 \
    --query peeringState \
    --output tsv
  
  # Check Spoke to Hub peering status
  az network vnet peering show \
    --resource-group $RESOURCE_GROUP_SPOKE1 \
    --vnet-name $VNET_NAME_SPOKE1 \
    --name Spoke1ToHub \
    --query peeringState \
    --output tsv
  ```
  
  **Expected Output:**
  ```
  Connected
  Connected
  ```

* **Azure PowerShell:**
  ```powershell
  # Check Hub to Spoke peering status
  $hubPeering = Get-AzVirtualNetworkPeering -ResourceGroupName $hubRgName -VirtualNetworkName $hubVnetName -Name HubToSpoke1
  Write-Host "Hub to Spoke peering status: $($hubPeering.PeeringState)"
  
  # Check Spoke to Hub peering status
  $spokePeering = Get-AzVirtualNetworkPeering -ResourceGroupName $spoke1RgName -VirtualNetworkName $spoke1VnetName -Name Spoke1ToHub
  Write-Host "Spoke to Hub peering status: $($spokePeering.PeeringState)"
  ```
  
  **Expected Output:**
  ```
  Hub to Spoke peering status: Connected
  Spoke to Hub peering status: Connected
  ```

### Post-Lab: Cleanup

To avoid incurring unnecessary costs, clean up the resources created during this lab using your preferred method.

1. **Delete Resource Groups:**

   * **Azure Portal:**  Locate the Resource Groups (`iatd_labs_15_hub_rg` and `iatd_labs_15_spoke1_rg`) and delete them.

   * **Azure CLI:**
     ```bash
     az group delete --name $RESOURCE_GROUP_HUB --yes
     az group delete --name $RESOURCE_GROUP_SPOKE1 --yes
     ```

   * **Azure PowerShell:**
     ```powershell
     Remove-AzResourceGroup -Name $hubRgName -Force
     Remove-AzResourceGroup -Name $spoke1RgName -Force
     ```

2. **Verify Resource Deletion:**
   * Ensure all resources have been properly deleted to avoid unexpected charges.
   * Check the Azure Portal to confirm that the resource groups no longer exist.

### Summary

**Congratulations!** You've successfully created a hub and spoke network architecture using Azure Virtual Network Peering. In this lab, you:

1. Created hub and spoke virtual networks with appropriate subnets
2. Configured VNet peering between the hub and spoke networks
3. Deployed virtual machines to test connectivity
4. Verified the peering status and network connectivity
5. Cleaned up resources to avoid unnecessary costs

**Key Takeaways:**

* **Hub and Spoke Architecture:** Provides a centralized approach to network connectivity in Azure
* **VNet Peering:** Enables direct connectivity between virtual networks in the same region
* **Multiple Implementation Methods:** Azure Portal, CLI, and PowerShell each offer different advantages for deployment and management
* **Network Validation:** Testing connectivity between VMs is crucial to verify proper network configuration

**Next Steps:**

* Explore more complex hub and spoke architectures with multiple spoke networks
* Implement network security groups (NSGs) to control traffic flow
* Consider adding Azure Firewall or other NVAs in the hub for centralized security
* Investigate gateway transit for connecting on-premises networks to the hub and spoke topology
