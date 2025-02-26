OK. Here’s the updated lab guide, which now starts at Lab 3.

## Lab 3: Creating a Dual-Stack Virtual Network (VNet) and Virtual Machine

**Objective:** This lab guides you through the process of creating an Azure Virtual Network (VNet) with both IPv4 and IPv6 address spaces. Then, you’ll deploy a Windows virtual machine (VM) into this dual-stack VNet, assigning both IPv4 and IPv6 addresses to the VM. This lab will guide you through three methods to perform this task: Azure Portal, Azure CLI, and PowerShell.

**Estimated Time:** 45 minutes - 1 hour

### Prerequisites

1.  **Azure Subscription:** You need an active Azure subscription. If you don't have one, you can sign up for a free trial at <https://azure.microsoft.com/free/>.
2.  **Azure Account:** You will need an Azure account with appropriate permissions to create resources, including VNets and Virtual Machines.
3.  **CloudShell:** It is preferable that the CLI and PowerShell commands are run via the Azure CloudShell.
4.  **PowerShell Module:** To perform tasks using PowerShell locally, ensure you have the Azure Az PowerShell module installed.  Refer to the Azure documentation for installation instructions.

### Task 1: Create a Resource Group

All resources need to be created in a Resource Group. It's best practice to create a new one for each lab. This can be achieved using the Azure Portal, Azure CLI, or PowerShell.

#### Method 1: Azure Portal

1.  Navigate to the Azure portal at <https://portal.azure.com>.
2.  In the search bar, type "Resource groups" and select **Resource groups**.
3.  Click **Create**.
4.  Select your subscription, enter `iatd_labs_03_rg` for the name, and choose a region (e.g., `Australia East`).
5.  Click **Review + create**, then **Create**.

#### Method 2: Azure CLI

1.  Open the Azure Cloud Shell.

    *   Navigate to the Azure portal at <https://portal.azure.com>.
    *   Click the Cloud Shell icon in the top navigation bar. If prompted, select "Bash" as your preferred shell environment.

    ```
    Welcome to Azure Cloud Shell

    Type "help" to learn about Cloud Shell features.
    ```

2.  Create a resource group:

    ```azurecli
    az group create --name iatd_labs_03_rg --location australiasoutheast
    ```

    *   Replace `australiasoutheast` with your preferred Azure region.

    **Sample output:**

    ```json
    {
      "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/iatd_labs_03_rg",
      "location": "australiasoutheast",
      "managedBy": null,
      "name": "iatd_labs_03_rg",
      "properties": {
        "provisioningState": "Succeeded"
      },
      "tags": null,
      "type": "Microsoft.Resources/resourceGroups"
    }
    ```

#### Method 3: PowerShell

1.  Open the Azure Cloud Shell.

    *   Navigate to the Azure portal at <https://portal.azure.com>.
    *   Click the Cloud Shell icon in the top navigation bar. If prompted, select "PowerShell" as your preferred shell environment.

    ```powershell
    Welcome to Azure Cloud Shell

    Type "help" to learn about Cloud Shell features.
    ```

2.  Create the resource group:

    ```powershell
    New-AzResourceGroup -Name "iatd_labs_03_rg" -Location "australiasoutheast"
    ```

    *   Replace `australiasoutheast` with your preferred Azure region.

    **Sample output:**

    ```text
    ResourceGroupName : iatd_labs_03_rg
    Location          : australiasoutheast
    ProvisioningState : Succeeded
    Tags              :
    ```

### Task 2: Create a Dual-Stack Virtual Network (VNet)

In this task, you will create a VNet with both IPv4 and IPv6 address spaces. You will use the Azure Portal, Azure CLI and PowerShell to complete the task.

#### Method 1: Azure Portal

1.  In the Azure portal search bar, search for "Virtual networks" and select **Virtual networks**.
2.  Click **Create**.
3.  On the **Basics** tab:
    *   Select your subscription.
    *   Select `iatd_labs_03_rg` for the **Resource group**.
    *   Enter `iatd_labs_03_vnet` for the **Name**.
    *   Choose a region.
4.  Go to the **IP Addresses** tab:
    *   For **IPv4 address space**, enter `172.16.1.0/16`.
    *   Check the **Add IPv6 address space** box.
    *   Enter an IPv6 address space (e.g., `2404:f800:8000:123::/63`).
    *   Click **Add subnet**.  Create 2 subnets with the following details:

        *   **Subnet-1:**
            *   *Subnet name:* Subnet-1
            *   *Subnet address range:* 172.16.1.0/24
            *   *Subnet IPv6 address range:* 2404:f800:8000:123::/64

        *   **Subnet-2:**
            *   *Subnet name:* Subnet-2
            *   *Subnet address range:* 172.16.2.0/24
            *   *Subnet IPv6 address range:* 2404:f800:8000:123:1::/64
5.  Click **Review + create** and then **Create**.

#### Method 2: Azure CLI

1.  Open the Azure Cloud Shell with the "Bash" environment selected.

2.  Create the VNet:

    ```azurecli
    az network vnet create \
      --resource-group iatd_labs_03_rg \
      --name iatd_labs_03_vnet \
      --address-prefixes 172.16.1.0/16 \
      --ipv6-address-prefix 2404:f800:8000:123::/63 \
      --location australiasoutheast
    ```

    *   `--resource-group`: Specifies the resource group where the VNet will be created.
    *   `--name`: Sets the name of the VNet.
    *   `--address-prefixes`: Defines the IPv4 address space.
    *   `--ipv6-address-prefix`: Enables IPv6 and assigns the address space.
    *   `--location`: Specifies the Azure region.

    **Sample Output:**

    ```json
    {
      "addressSpace": {
        "addressPrefixes": [
          "172.16.1.0/16"
        ]
      },
      "ddosSettings": null,
      "dhcpOptions": {
        "dnsServers": []
      },
      "enableDdosProtection": false,
      "enableVmProtection": false,
      "etag": "W/\"xxxxxxxxxxxxxxxxxxxxxxxx\"",
      "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/iatd_labs_03_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_03_vnet",
      "ipv6Space": {
        "addressPrefix": "2404:f800:8000:123::/63"
      },
      "location": "australiasoutheast",
      "name": "iatd_labs_03_vnet",
      "provisioningState": "Succeeded",
      "resourceGuid": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "resourceGroupName": "iatd_labs_03_rg",
      "subnets": [],
      "tags": null,
      "type": "Microsoft.Network/virtualNetworks",
      "virtualNetworkPeerings": []
    }
    ```

3.  Create the subnets:

    ```azurecli
    az network vnet subnet create \
      --resource-group iatd_labs_03_rg \
      --vnet-name iatd_labs_03_vnet \
      --name Subnet-1 \
      --address-prefixes 172.16.1.0/24 \
      --ipv6-prefix 2404:f800:8000:123::/64
    ```

    ```azurecli
    az network vnet subnet create \
      --resource-group iatd_labs_03_rg \
      --vnet-name iatd_labs_03_vnet \
      --name Subnet-2 \
      --address-prefixes 172.16.2.0/24 \
      --ipv6-prefix 2404:f800:8000:123:1::/64
    ```

    *   `--resource-group`: The resource group containing the VNet.
    *   `--vnet-name`: The name of the VNet.
    *   `--name`: The name of the subnet.
    *   `--address-prefixes`: The IPv4 address prefix for the subnet.
    *   `--ipv6-prefix`: The IPv6 address prefix for the subnet.

    **Sample Output:**

    ```json
    {
      "addressPrefix": "172.16.1.0/24",
      "addressPrefixes": [
        "172.16.1.0/24"
      ],
      "delegations": [],
      "etag": "W/\"xxxxxxxxxxxxxxxxxxxxxxxx\"",
      "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/iatd_labs_03_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_03_vnet/subnets/Subnet-1",
      "ipConfigurations": [],
      "ipv6AddressPrefix": "2404:f800:8000:123::/64",
      "ipv6AddressPrefixes": [
        "2404:f800:8000:123::/64"
      ],
      "name": "Subnet-1",
      "natGateway": null,
      "networkSecurityGroup": null,
      "privateEndpointNetworkPolicies": "Disabled",
      "privateLinkServiceNetworkPolicies": "Enabled",
      "provisioningState": "Succeeded",
      "purpose": null,
      "resourceNavigationLinks": [],
      "resourceGroupName": "iatd_labs_03_rg",
      "serviceAssociationLinks": [],
      "serviceEndpointPolicies": [],
      "serviceEndpoints": [],
      "type": "Microsoft.Network/virtualNetworks/subnets"
    }
    ```

#### Method 3: PowerShell

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

    **Sample Output:** (truncated)

    ```text
    ResourceType         : Microsoft.Network/virtualNetworks
    Name                 : iatd_labs_03_vnet
    Etag                 : W/"..."
    Id                   : /subscriptions/.../resourceGroups/iatd_labs_03_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_03_vnet
    Location             : australiasoutheast
    ProvisioningState    : Succeeded
    ResourceGroupName    : iatd_labs_03_rg
    AddressSpace         : {
                               "AddressPrefixes":  [
                                                     "172.16.1.0/16"
                                                   ]
                           }
    ...
    ```

### Task 3: Create a Windows Virtual Machine in the Dual-Stack VNet

In this task, you will create a Windows VM within the VNet and assign both IPv4 and IPv6 addresses. You will create one Public IP for IPv4 and another for IPv6, create a Network Interface (NIC), and assign both Public IPs and IPv4/IPv6 configurations.

#### Method 1: Azure Portal

1.  **Create the Virtual Machine:**
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
        *   **Public IP:** Create new. Create IPv4 `iatd_labs_03_pipv4` and IPv6 `iatd_labs_03_pipv6`.
        *   **NIC network security group:** Basic
        *   **Public inbound ports:** Allow selected ports
        *   **Select inbound ports:** RDP (3389)

    *   **Management, Advanced, Tags:** Leave the defaults.
    *   **Review + create:** Review the configuration and click **Create**.

#### Method 2: Azure CLI

1.  **Create Public IP Addresses:**

    ```azurecli
    az network public-ip create \
      --resource-group iatd_labs_03_rg \
      --name iatd_labs_03_public_ip_ipv4 \
      --version IPv4 \
      --allocation-method Static \
      --location australiasoutheast
    ```

    ```azurecli
    az network public-ip create \
      --resource-group iatd_labs_03_rg \
      --name iatd_labs_03_public_ip_ipv6 \
      --version IPv6 \
      --allocation-method Static \
      --location australiasoutheast
    ```

    *   `--resource-group`: The resource group for the public IP.
    *   `--name`: The name for the public IP address.
    *   `--version`: Configures whether the IP address will be IPv4 or IPv6.
    *   `--allocation-method`: Specifies the allocation method to Static.
    *   `--location`: The Azure region.

    **Sample Output:**

    ```json
    {
      "allocationMethod": "Static",
      "ddosSettings": null,
      "deleteOption": "Delete",
      "dnsSettings": null,
      "etag": "W/\"xxxxxxxxxxxxxxxxxxxxxxxx\"",
      "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/iatd_labs_03_rg/providers/Microsoft.Network/publicIPAddresses/iatd_labs_03_public_ip_ipv4",
      "idleTimeoutInMinutes": 4,
      "ipAddress": "20.xx.xx.xx",
      "ipConfiguration": null,
      "ipTags": [],
      "location": "australiasoutheast",
      "migrationPhase": "None",
      "name": "iatd_labs_03_public_ip_ipv4",
      "natGateway": null,
      "publicIPAddressVersion": "IPv4",
      "publicIPAllocationMethod": "Static",
      "publicIPPrefix": null,
      "resourceGuid": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "resourceGroupName": "iatd_labs_03_rg",
      "servicePublicIPAddress": null,
      "sku": {
        "name": "Basic",
        "tier": "Regional"
      },
      "tags": null,
      "type": "Microsoft.Network/publicIPAddresses",
      "zones": []
    }
    ```

2.  **Create Network Interface (NIC) with IPv4 and IPv6 Configuration:**

    ```azurecli
    az network nic create \
      --resource-group iatd_labs_03_rg \
      --name iatd_labs_03_nic \
      --vnet-name iatd_labs_03_vnet \
      --subnet Subnet-1 \
      --public-ip-address iatd_labs_03_public_ip_ipv4 \
      --location australiasoutheast
    ```

    ```azurecli
    az network nic ip-config create \
      --resource-group iatd_labs_03_rg \
      --nic-name iatd_labs_03_nic \
      --name ipconfig2 \
      --vnet-name iatd_labs_03_vnet \
      --subnet Subnet-2 \
      --public-ip-address iatd_labs_03_public_ip_ipv6 \
      --version IPv6
    ```

    *   `--resource-group`: The resource group where the NIC will be created.
    *   `--name`: The name of the network interface.
    *   `--vnet-name`: Specifies the VNet.
    *   `--subnet`: Specifies the subnet for IPv4.  Make sure the initial subnet is specified when creating the NIC.
    *   `--subnet`: Specifies the subnet for IPv6
    *   `--public-ip-address`: Associates the public IP address to the NIC.
    *   `--location`: The Azure region.
    *   `--version`: IP Version

    **Sample Output:**

    ```json
    {
      "dnsSettings": {
        "appliedDnsServers": [],
        "dnsServers": [],
        "internalDnsNameLabel": null,
        "internalDomainNameSuffix": null,
        "internalFqdn": null
      },
      "enableAcceleratedNetworking": false,
      "enableIpForwarding": false,
      "etag": "W/\"xxxxxxxxxxxxxxxxxxxxxxxx\"",
      "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/iatd_labs_03_rg/providers/Microsoft.Network/networkInterfaces/iatd_labs_03_nic",
      "ipConfigurations": [
        {
          "applicationGatewayBackendAddressPools": [],
          "applicationSecurityGroups": [],
          "etag": "W/\"xxxxxxxxxxxxxxxxxxxxxxxx\"",
          "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/iatd_labs_03_rg/providers/Microsoft.Network/networkInterfaces/iatd_labs_03_nic/ipConfigurations/ipconfig1",
          "name": "ipconfig1",
          "privateIpAddress": null,
          "privateIpAddressVersion": "IPv4",
          "privateIpAllocationMethod": "Dynamic",
          "provisioningState": "Succeeded",
          "publicIpAddress": {
            "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/iatd_labs_03_rg/providers/Microsoft.Network/publicIPAddresses/iatd_labs_03_public_ip_ipv4"
          },
          "resourceGroupName": "iatd_labs_03_rg",
          "subnet": {
            "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/iatd_labs_03_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_03_vnet/subnets/Subnet-1"
          },
          "type": "Microsoft.Network/networkInterfaces/ipConfigurations",
          "virtualNetworkTaps": []
        },
          {
          "applicationGatewayBackendAddressPools": [],
          "applicationSecurityGroups": [],
          "etag": "W/\"xxxxxxxxxxxxxxxxxxxxxxxx\"",
          "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/iatd_labs_03_rg/providers/Microsoft.Network/networkInterfaces/iatd_labs_03_nic/ipConfigurations/ipconfig2",
          "name": "ipconfig2",
          "privateIpAddress": null,
          "privateIpAddressVersion": "IPv6",
          "privateIpAllocationMethod": "Dynamic",
          "provisioningState": "Succeeded",
          "publicIpAddress": {
            "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/iatd_labs_03_rg/providers/Microsoft.Network/publicIPAddresses/iatd_labs_03_public_ip_ipv6"
          },
          "resourceGroupName": "iatd_labs_03_rg",
          "subnet": {
            "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/iatd_labs_03_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_03_vnet/subnets/Subnet-2"
          },
          "type": "Microsoft.Network/networkInterfaces/ipConfigurations",
          "virtualNetworkTaps": []
        }
      ],
      "location": "australiasoutheast",
      "macAddress": null,
      "migrationPhase": "None",
      "name": "iatd_labs_03_nic",
      "networkSecurityGroup": null,
      "primary": false,
      "provisioningState": "Succeeded",
      "resourceGuid": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "resourceGroupName": "iatd_labs_03_rg",
      "tags": null,
      "type": "Microsoft.Network/networkInterfaces",
      "virtualMachine": null
    }
    ```

3.  **Create the Virtual Machine:**

    ```azurecli
    az vm create \
      --resource-group iatd_labs_03_rg \
      --name iatd_labs_03_vm \
      --nics iatd_labs_03_nic \
      --image Win2022AzureEditionCore \
      --size Standard_D2s_v3 \
      --admin-username azureuser \
      --admin-password COMPLEX_PASSWORD
    ```

    *   `--resource-group`: The resource group.
    *   `--name`: The name of the VM.
    *   `--nics`: The NIC to associate with the VM.
    *   `--image`: The operating system image.
    *   `--size`: The VM size.
    *   `--admin-username`: The administrator username.
    *   `--admin-password`: A strong password for the administrator account.

    **Note:** Replace `COMPLEX_PASSWORD` with a strong password.

    **Sample Output:**

    ```json
    {
      "fqdn": "iatd-labs-03-vm.australiasoutheast.cloudapp.azure.com",
      "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/iatd_labs_03_rg/providers/Microsoft.Compute/virtualMachines/iatd_labs_03_vm",
      "location": "australiasoutheast",
      "macAddress": "xx-xx-xx-xx-xx-xx",
      "powerState": "VM running",
      "privateIpAddress": "172.16.1.4",
      "publicIpAddress": "20.xx.xx.xx",
      "resourceGroup": "iatd_labs_03_rg",
      "zones": []
    }
    ```

#### Method 3: PowerShell

1.  **Create Public IP Addresses:**

    ```powershell
    $rgName = "iatd_labs_03_rg"
    $location = "australiasoutheast"

    # Create IPv4 Public IP
    $publicIpIPv4 = New-AzPublicIpAddress `
        -Name "iatd_labs_03_public_ip_ipv4" `
        -ResourceGroupName $rgName `
        -Location $location `
        -AllocationMethod Static `
        -IPAddressVersion IPv4 `
        -Sku Basic

    # Create IPv6 Public IP
    $publicIpIPv6 = New-AzPublicIpAddress `
        -Name "iatd_labs_03_public_ip_ipv6" `
        -ResourceGroupName $rgName `
        -Location $location `
        -AllocationMethod Static `
        -IPAddressVersion IPv6 `
        -Sku Basic
    ```

    **Sample Output**

    ```text
    Name                      : iatd_labs_03_public_ip_ipv4
    ResourceGroupName         : iatd_labs_03_rg
    Location                  : australiasoutheast
    Id                        : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/iatd_labs_03_rg/providers/Microsoft.Network/publicIPAddresses/iatd_labs_03_public_ip_ipv4
    Etag                      : W/"xxxxxxxxxxxxxxxxxxxxxxxx"
    ...
    ```

    *   Remember to adjust the `$location` to your preferred region.

2.  **Create Network Interface (NIC) with IPv4 and IPv6 Configuration:**

    ```powershell
    # Set variables
    $nicName = "iatd_labs_03_nic"
    $vnetName = "iatd_labs_03_vnet"
    $subnet1Name = "Subnet-1"
    $subnet2Name = "Subnet-2"

    #Get subnet object
    $subnet1 = Get-AzVirtualNetworkSubnetConfig -name $subnet1Name -VirtualNetwork $vnet

    # Create an IPv4 IP configuration
    $ipConfigIPv4 = New-AzNetworkInterfaceIpConfig `
        -Name "ipconfig1" `
        -Subnet $subnet1 `
        -PublicIpAddress $publicIpIPv4 `
        -PrivateIpAddressVersion IPv4

    # Get subnet2 object
    $subnet2 = Get-AzVirtualNetworkSubnetConfig -name $subnet2Name -VirtualNetwork $vnet

    # Create an IPv6 IP configuration
    $ipConfigIPv6 = New-AzNetworkInterfaceIpConfig `
        -Name "ipconfig2" `
        -Subnet $subnet2 `
        -PublicIpAddress $publicIpIPv6 `
        -PrivateIpAddressVersion IPv6

    # Create the NIC
    $nic = New-AzNetworkInterface `
        -Name $nicName `
        -ResourceGroupName $rgName `
        -Location $location `
        -IpConfiguration @($ipConfigIPv4,$ipConfigIPv6)
    ```

    *You may encounter an error*
    *The resource 'Public IP name' could not be found in данном subscription. Please check if the resource name is valid and ensure that it exists in the same subscription.*

    You might get an error for missing public IPs or you need to wait until the IPv6 are recognised.

    ```powershell
     # Set variables
        $rgName = "iatd_labs_03_rg"
        $nicName = "iatd_labs_03_nic"
        $location = "australiasoutheast"

        # Get existing public IP
        $ExistingIPv4 = Get-AzPublicIpAddress -Name "iatd_labs_03_public_ip_ipv4" -ResourceGroupName $rgName
        $ExistingIPv6 = Get-AzPublicIpAddress -Name "iatd_labs_03_public_ip_ipv6" -ResourceGroupName $rgName

       #Get subnet objects
        $vnet = Get-AzVirtualNetwork -Name "iatd_labs_03_vnet" -ResourceGroupName $rgName
        $subnet1 = Get-AzVirtualNetworkSubnetConfig -name "Subnet-1" -VirtualNetwork $vnet
        $subnet2 = Get-AzVirtualNetworkSubnetConfig -name "Subnet-2" -VirtualNetwork $vnet

    # Create an IPv4 IP configuration
    $ipConfigIPv4 = New-AzNetworkInterfaceIpConfig `
        -Name "ipconfig1" `
        -Subnet $subnet1 `
        -PublicIpAddress $ExistingIPv4 `
        -PrivateIpAddressVersion IPv4

    # Create an IPv6 IP configuration
    $ipConfigIPv6 = New-AzNetworkInterfaceIpConfig `
        -Name "ipconfig2" `
        -Subnet $subnet2 `
        -PublicIpAddress $ExistingIPv6 `
        -PrivateIpAddressVersion IPv6

       # Create the NIC
       $nic = New-AzNetworkInterface `
           -Name $nicName `
           -ResourceGroupName $rgName `
           -Location $location `
           -IpConfiguration @($ipConfigIPv4,$ipConfigIPv6)
    ```

    **Sample output:**

    ```text
    Name                      : iatd_labs_03_nic
    ResourceGroupName         : iatd_labs_03_rg
    Location                  : australiasoutheast
    Id                        : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/iatd_labs_03_rg/providers/Microsoft.Network/networkInterfaces/iatd_labs_03_nic
    Etag                      : W/"xxxxxxxxxxxxxxxxxxxxxxxx"
    ...
    ```

3.  **Create the Virtual Machine:**

    ```powershell
    # Set variables
    $vmName = "iatd_labs_03_vm"
    $image = "Win2022AzureEditionCore"
    $vmSize = "Standard_D2s_v3"
    $adminUsername = "azureuser"
    $adminPassword = "COMPLEX_PASSWORD"

    #Set the vm configuration
    $vmConfig = New-AzVMConfig -VMName $vmName -VMSize $vmSize
    $vmConfig = Set-AzVMOperatingSystem -VM $vmConfig -Windows -ComputerName $vmName -AdminUsername $adminUsername -AdminPassword $adminPassword
    $vmConfig = Set-AzVMSourceImage -VM $vmConfig -PublisherName "MicrosoftWindowsServer" -Offer "WindowsServer" -Skus "2022-datacenter" -Version latest

    # Create the VM
    New-AzVM `
        -ResourceGroupName $rgName `
        -Location $location `
        -NetworkInterfaceIDs $nic.Id `
        -VM $vmConfig
    ```

    *   Make sure to replace `COMPLEX_PASSWORD` with a strong password.

    **Sample Output:** (truncated)

    ```text
    ResourceGroupName : iatd_labs_03_rg
    Location          : australiasoutheast
    VmId              : xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
    Name              : iatd_labs_03_vm
    HardwareProfile   : {
                          "VmSize":  "Standard_D2s_v3"
                        }
    ...
    ```

### Post-Lab Clean-up

To avoid incurring unnecessary costs, remove the resources created during this lab:

#### Method 1: Azure Portal

1.  In the Azure portal, search for "Resource groups" and select it.
2.  Select the `iatd_labs_03_rg` resource group.
3.  Click **Delete resource group**.
4.  Confirm the deletion by typing the resource group name and clicking **Delete**.

#### Method 2: Azure CLI

```azurecli
az group delete --name iatd_labs_03_rg --yes
```

This command deletes the resource group `iatd_labs_03_rg` and all resources contained within it.

#### Method 3: PowerShell

```powershell
Remove-AzResourceGroup -Name "iatd_labs_03_rg" -Force
```

This command removes the resource group and all its resources. The `-Force` parameter suppresses the confirmation prompt.
