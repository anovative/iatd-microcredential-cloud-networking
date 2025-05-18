## IATD Microcredential Cloud Networking: Lab 3 - Creating a Dual-Stack Virtual Network (VNet) and Virtual Machine

**Objective:** This lab guides you through creating an Azure Virtual Network (VNet) with both IPv4 and IPv6 address spaces. You'll then deploy a Windows virtual machine (VM) into this dual-stack VNet, assigning both IPv4 and IPv6 addresses to the VM. This lab strategically uses the Azure Portal, Azure CLI, and PowerShell to expose you to each method.

**Estimated Time:** 45 minutes - 1 hour

### Prerequisites

1.  **Azure Subscription:** An active Azure subscription is required. Sign up for a free trial at <https://azure.microsoft.com/free/> if needed.
2.  **Azure Account:** Ensure your Azure account has sufficient permissions to create resources (VNets, VMs, etc.).
3.  **Cloud Shell:** The CLI and PowerShell commands should be run within the Azure Cloud Shell.
4.  **PowerShell Module:** If you plan to execute PowerShell commands locally, install the Azure Az PowerShell module from the Azure documentation.

**Lab Conventions:**

*   **Azure Cloud Shell:** All PowerShell and CLI interactions will occur within the Azure Cloud Shell environment.
*   **Naming Conventions:** Resource names follow the convention `iatd_labs_03_*`.
*   **IP Address Range:** The `172.16.0.0/16` range will be used (as per standardization) for IPv4 addressing.
*   **Location:** Choose a consistent Azure region.

### Task 1: Create a Resource Group

#### Method: Azure CLI

1.  Open the Azure Cloud Shell with the "Bash" environment selected.

2.  Create the resource group:

    ```azurecli
    az group create --name iatd_labs_03_rg --location australiasoutheast
    ```

    *   Replace `australiasoutheast` with your preferred Azure region.

### Task 2: Create a Dual-Stack Virtual Network (VNet)

In this task, create a VNet with both IPv4 and IPv6 address spaces.

#### Method: PowerShell

1.  Open the Azure Cloud Shell with the "PowerShell" environment selected.

2.  Create the VNet and subnets:

    ```powershell
    # Set variables
    $resourceGroupName = "iatd_labs_03_rg"
    $vnetName = "iatd_labs_03_vnet"
    $location = "australiasoutheast" # Change to your desired location
    $vnetAddressPrefix = "172.16.1.0/16"
    $ipv6AddressPrefix = "2404:f800:8000:123::/63"
    $subnet1Name = "Subnet-1"
    $subnet1Prefix = "172.16.1.0/24"
    $subnet1IPv6Prefix = "2404:f800:8000:123::/64"
    $subnet2Name = "Subnet-2"
    $subnet2Prefix = "172.16.2.0/24"
    $subnet2IPv6Prefix = "2404:f800:8000:123:1::/64"

    # Create the virtual network
    $vnet = New-AzVirtualNetwork `
        -Name $vnetName `
        -ResourceGroupName $resourceGroupName `
        -Location $location `
        -AddressPrefix $vnetAddressPrefix `
        -IPv6AddressPrefix $ipv6AddressPrefix

    # Add subnet 1 configuration
    $subnet1 = Add-AzVirtualNetworkSubnetConfig `
        -Name $subnet1Name `
        -VirtualNetwork $vnet `
        -AddressPrefix $subnet1Prefix `
        -IPv6AddressPrefix $subnet1IPv6Prefix

    # Add subnet 2 configuration
    $subnet2 = Add-AzVirtualNetworkSubnetConfig `
        -Name $subnet2Name `
        -VirtualNetwork $vnet `
        -AddressPrefix $subnet2Prefix `
        -IPv6AddressPrefix $subnet2IPv6Prefix

    # Set the virtual network with the new subnet configuration
    $vnet | Set-AzVirtualNetwork
    ```

    *   Update the `$location` variable to your desired Azure region.

### Task 3: Create a Windows Virtual Machine in the Dual-Stack VNet

In this task, you will create a Windows VM within the VNet and assign both IPv4 and IPv6 addresses. You will create one Public IP for IPv4 and another for IPv6, create a Network Interface (NIC), and assign both Public IPs and IPv4/IPv6 configurations.

#### Method: Azure Portal

1.  **Create the Public IPs and VM using Azure Portal:**
    *   In the Azure portal, search for "Virtual machines" and select it.
    *   Click **Create** -> **Azure virtual machine**.
    *   **Project details:**
        *   **Subscription:** Select your Azure subscription.
        *   **Resource group:** Select `iatd_labs_03_rg`.
    *   **Instance details:**
        *   **Virtual machine name:** Enter `iatd_labs_03_vm`.
        *   **Region:** Choose the same region as your resource group and VNet.
        *   **Image:** Select "Windows Server 2022 Datacenter" (or a later version).
        *   **Size:** Choose a size suitable for testing (e.g., Standard\_DS1\_v2).
    *   **Administrator account:**
        *   **Username:** Enter a username (e.g., `azureadmin`).
        *   **Password:** Enter a strong password.
    *   **Inbound port rules:**
        *   **Public inbound ports:** Select "Allow selected ports".
        *   **Select inbound ports:** Check "RDP (3389)".
    *   **Networking:**
        *   **Virtual network:** Select `iatd_labs_03_vnet`.
        *   **Subnet:** Select `Subnet-1`.
        * **Public IP:** Create new

           IPv4 Public IP Name: iatd\_labs\_03\_public\_ip\_ipv4
           IPv6 Public IP Name: iatd\_labs\_03\_public\_ip\_ipv6
        *   **NIC network security group:** Basic
        *   **Public inbound ports:** Allow selected ports
        *   **Select inbound ports:** RDP (3389)

    *   **Management, Advanced, Tags:** Leave the defaults.
    *   **Review + create:** Review the configuration and click **Create**.

### Post-Lab Clean-up

To avoid incurring unnecessary costs, remove the resources created during this lab:

#### Method: Azure CLI

1.  Open the Azure Cloud Shell with the "Bash" environment selected.

```azurecli
az group delete --name iatd_labs_03_rg --yes
```

This command deletes the resource group `iatd_labs_03_rg` and all resources contained within it.
