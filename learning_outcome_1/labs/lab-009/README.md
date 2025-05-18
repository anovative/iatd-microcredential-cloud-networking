## IATD Microcredential Cloud Networking: Lab 9 - Implementing Private DNS Zones in Azure

**Objective:** In this lab, you will learn how to create a private DNS zone in Azure, link it to a virtual network, add DNS records, and verify name resolution from a virtual machine within the network.

**Estimated Time:** 45-60 minutes

**Prerequisites:**

1.  **Azure Subscription:** You need an active Azure subscription.
2.  **Azure Cloud Shell Access:**  This lab utilizes the Azure Cloud Shell, which requires a storage account.  If you haven't used it before, the first time you launch it, you will be prompted to create one.
3.  **Basic understanding of DNS concepts:** Familiarity with DNS records like A and CNAME is helpful.
4.  **Familiarity with Virtual Networks:** A basic understanding of Azure Virtual Networks is needed.

**Lab Conventions:**

*   **Naming Conventions:** All resources created in this lab will be prefixed with `iatd_labs_09` to ensure easy identification and cleanup.
*   **IP Address Range:** The `172.16.0.0/16` range will be used (as per standardization) for virtual networks.
*   **Location:** Choose a consistent Azure region.

**Let's get started!**

### Part 1: Setting Up the Lab Environment

In this part, you will create a basic VNet to work with if you don't already have one.

**Method 1: Azure Cloud Shell**

1.  **Launch Azure Cloud Shell:**
    *   Go to the Azure portal ([https://portal.azure.com](https://portal.azure.com)).
    *   Click the Cloud Shell icon in the top navigation bar (it looks like a terminal prompt).
    *   If prompted, select "Bash" as your Cloud Shell environment.

2.  **Create a Resource Group (if you don't have one):**

    ```bash
    # Set a location
    location="australiaeast"
    # Create a resource group
    rgName="iatd_labs_09_rg"
    az group create --name $rgName --location $location
    ```

    Expected output:
    ```json
    {
      "id": "/subscriptions/your_subscription_id/resourceGroups/iatd_labs_09_rg",
      "location": "australiaeast",
      "managedBy": null,
      "name": "iatd_labs_09_rg",
      "properties": {
        "provisioningState": "Succeeded"
      },
      "tags": null,
      "type": "Microsoft.Resources/resourceGroups"
    }
    ```

3.  **Create a Virtual Network (if you don't have one):**

    ```bash
    # Define Vnet name
    vnetName="iatd_labs_09_vnet"
    # Create the virtual network
    az network vnet create \
      --resource-group $rgName \
      --name $vnetName \
      --address-prefixes 172.16.0.0/16 \
      --subnet-name default \
      --subnet-prefixes 172.16.0.0/24
    ```

    Expected output:
    ```json
    {
      "newVNet": {
        "addressSpace": {
          "addressPrefixes": [
            "172.16.0.0/16"
          ]
        },
        "dhcpOptions": {
          "dnsServers": []
        },
        "enableDdosProtection": false,
        "enableVmProtection": false,
        "id": "/subscriptions/your_subscription_id/resourceGroups/iatd_labs_09_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_09_vnet",
        "location": "australiaeast",
        "name": "iatd_labs_09_vnet",
        "provisioningState": "Succeeded",
        "resourceGroup": "iatd_labs_09_rg",
        "resourceGuid": "...",
        "subnets": [
          {
            "addressPrefix": "172.16.0.0/24",
            "delegations": [],
            "id": "/subscriptions/your_subscription_id/resourceGroups/iatd_labs_09_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_09_vnet/subnets/default",
            "name": "default",
            "networkSecurityGroup": null,
            "privateEndpointNetworkPolicies": "Disabled",
            "privateLinkServiceNetworkPolicies": "Enabled",
            "provisioningState": "Succeeded",
            "resourceNavigationLinks": [],
            "serviceAssociationLinks": [],
            "serviceEndpointPolicies": [],
            "serviceEndpoints": []
          }
        ],
        "tags": null,
        "type": "Microsoft.Network/virtualNetworks",
        "virtualNetworkPeerings": []
      }
    }
    ```

### Part 2: Creating a Private DNS Zone

In this part, you will create a private DNS zone named `private.iatdlabs.com`.

**Method 1: Azure Portal**

1.  In the Azure portal, search for and select "Private DNS zones".
2.  Click "+ Create".
3.  On the "Create private DNS zone" page:
    *   **Subscription:** Select your Azure subscription.
    *   **Resource Group:** Select the resource group you created earlier (iatd\_labs\_09\_rg).
    *   **Name:** Enter `private.iatdlabs.com`.
    *   **Review + create** and then **Create**.

**Method 2: PowerShell**

1.  Launch Azure Cloud Shell and select "PowerShell".

2.  Use the `New-AzPrivateDnsZone` cmdlet:

    ```powershell
    $rgName = "iatd_labs_09_rg"
    $dnsZoneName = "private.iatdlabs.com"
    New-AzPrivateDnsZone -Name $dnsZoneName -ResourceGroupName $rgName
    ```

    Expected output:
    ```
    Name              : private.iatdlabs.com
    ResourceGroupName : iatd_labs_09_rg
    Location          : global
    Etag              : "..."
    Id                : /subscriptions/your_subscription_id/resourceGroups/iatd_labs_09_rg/providers/Microsoft.Network/privateDnsZones/private.iatdlabs.com
    ZoneType          : Private
    ```

**Method 3: Azure CLI**

1.  Ensure you are in the Cloud Shell with "Bash" selected.

2.  Use the `az network private-dns zone create` command:

    ```bash
    rgName="iatd_labs_09_rg"
    dnsZoneName="private.iatdlabs.com"
    az network private-dns zone create --resource-group $rgName --name $dnsZoneName
    ```

    Expected output:
    ```json
    {
      "etag": "...",
      "id": "/subscriptions/your_subscription_id/resourceGroups/iatd_labs_09_rg/providers/Microsoft.Network/privateDnsZones/private.iatdlabs.com",
      "location": "global",
      "name": "private.iatdlabs.com",
      "nameServers": [],
      "numberOfRecordSets": 2,
      "provisioningState": "Succeeded",
      "resourceGuid": "...",
      "tags": {},
      "type": "Microsoft.Network/privateDnsZones"
    }
    ```

### Part 3: Linking the Zone to a VNet

Now, link the `private.iatdlabs.com` zone to the VNet you created (or an existing one).

**Method 1: Azure Portal**

1.  In the Azure portal, navigate to your private DNS zone (`private.iatdlabs.com`).
2.  Under "Settings", select "Virtual network links".
3.  Click "+ Add".
4.  On the "Add virtual network link" page:
    *   **Link name:** Enter `iatd-vnet-link`.
    *   **Subscription:** Select your subscription.
    *   **Virtual network:** Select your VNet (iatd\_labs\_09\_vnet or your existing VNet).
    *   **Enable auto registration:**  Leave this unchecked for this lab (we'll cover this in a later lab).
    *   Click "OK".

**Method 2: PowerShell**

1.  Use the `New-AzPrivateDnsVirtualNetworkLink` cmdlet.

    ```powershell
    $rgName = "iatd_labs_09_rg"
    $dnsZoneName = "private.iatdlabs.com"
    $vnetName = "iatd_labs_09_vnet"
    $linkName = "iatd-vnet-link"

    $VirtualNetwork = Get-AzVirtualNetwork -Name $vnetName -ResourceGroupName $rgName
    New-AzPrivateDnsVirtualNetworkLink -Name $linkName -ResourceGroupName $rgName -ZoneName $dnsZoneName -VirtualNetworkId $VirtualNetwork.Id -RegistrationEnabled:$false
    ```

    Expected output:
    ```
    Name              : iatd-vnet-link
    Etag              : "..."
    Id                : /subscriptions/your_subscription_id/resourceGroups/iatd_labs_09_rg/providers/Microsoft.Network/privateDnsZones/private.iatdlabs.com/virtualNetworkLinks/iatd-vnet-link
    VirtualNetwork    : @{Id=/subscriptions/your_subscription_id/resourceGroups/iatd_labs_09_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_09_vnet}
    RegistrationEnabled : False
    ProvisioningState : Succeeded
    ```

**Method 3: Azure CLI**

1.  Use the `az network private-dns link vnet create` command:

    ```bash
    rgName="iatd_labs_09_rg"
    dnsZoneName="private.iatdlabs.com"
    vnetName="iatd_labs_09_vnet"
    linkName="iatd-vnet-link"
    vnetId=$(az network vnet show --name $vnetName --resource-group $rgName --query id --output tsv)

    az network private-dns link vnet create --resource-group $rgName --zone-name $dnsZoneName --name $linkName --virtual-network $vnetId --registration-enabled false
    ```

    Expected output:
    ```json
    {
      "etag": "...",
      "id": "/subscriptions/your_subscription_id/resourceGroups/iatd_labs_09_rg/providers/Microsoft.Network/privateDnsZones/private.iatdlabs.com/virtualNetworkLinks/iatd-vnet-link",
      "name": "iatd-vnet-link",
      "properties": {
        "provisioningState": "Succeeded",
        "registrationEnabled": false,
        "virtualNetwork": {
          "id": "/subscriptions/your_subscription_id/resourceGroups/iatd_labs_09_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_09_vnet"
        }
      },
      "type": "Microsoft.Network/privateDnsZones/virtualNetworkLinks"
    }
    ```

### Part 4: Creating DNS Records

Let's add an A record and a CNAME record to the zone.

**Method 1: Azure Portal**

1.  In the Azure portal, navigate to your private DNS zone (`private.iatdlabs.com`).
2.  Click "+ Record set".
3.  **A Record:**
    *   **Name:** `webserver`
    *   **Type:** `A`
    *   **Alias record set:** No
    *   **TTL:** 3600
    *   **IP address:** `172.16.0.100` (Choose an IP from your VNet's address space that isn't in use).
    *   Click "OK".

4.  **CNAME Record:**
    *   Click "+ Record set" again.
    *   **Name:** `www`
    *   **Type:** `CNAME`
        *   **Alias record set:** No
    *   **TTL:** 3600
    *   **Alias:** `webserver.private.iatdlabs.com`
    *   Click "OK".

**Method 2: PowerShell**

1.  Use the `New-AzPrivateDnsRecordSet` and `Add-AzPrivateDnsRecordConfig` cmdlets.

    ```powershell
    $rgName = "iatd_labs_09_rg"
    $dnsZoneName = "private.iatdlabs.com"

    # Create A record
    $aRecordName = "webserver"
    $aRecordIP = "172.16.0.100"
    $aRecordSet = New-AzPrivateDnsRecordSet -Name $aRecordName -RecordType A -ZoneName $dnsZoneName -ResourceGroupName $rgName -Ttl 3600
    $aRecordSet | Add-AzPrivateDnsRecordConfig -Ipv4Address $aRecordIP | Set-AzPrivateDnsRecordSet

    # Create CNAME record
    $cnameRecordName = "www"
    $cnameAlias = "webserver.private.iatdlabs.com"
    $cnameRecordSet = New-AzPrivateDnsRecordSet -Name $cnameRecordName -RecordType CName -ZoneName $dnsZoneName -ResourceGroupName $rgName -Ttl 3600
    $cnameRecordSet | Add-AzPrivateDnsRecordConfig -Alias $cnameAlias | Set-AzPrivateDnsRecordSet
    ```

**Method 3: Azure CLI**

1.  Use the `az network private-dns record-set` commands.

    ```bash
    rgName="iatd_labs_09_rg"
    dnsZoneName="private.iatdlabs.com"

    # Create A record
    aRecordName="webserver"
    aRecordIP="172.16.0.100"
    az network private-dns record-set a create --resource-group $rgName --zone-name $dnsZoneName --name $aRecordName --ttl 3600
    az network private-dns record-set a add-record --resource-group $rgName --zone-name $dnsZoneName --record-set-name $aRecordName --ipv4-address $aRecordIP

    # Create CNAME record
    cnameRecordName="www"
    cnameAlias="webserver.private.iatdlabs.com"
    az network private-dns record-set cname create --resource-group $rgName --zone-name $dnsZoneName --name $cnameRecordName --ttl 3600 --cname $cnameAlias
    ```

### Part 5: Verifying Resolution

1.  **Create a Virtual Machine:**
    *   **Important:** Ensure you deploy this VM into the same VNet you linked to the Private DNS Zone (iatd\_labs\_09\_vnet or your existing one).  Give it a public IP address so you can connect to it.
    *   **Resource Group:** iatd\_labs\_09\_rg (or your resource group).
    *   **Virtual machine name:** iatd-vm-09
    *   **Region:** Australia East
    *   **Image:** Ubuntu Server 22.04 LTS
    *   **Size:** Standard\_B1ls (or another suitable size)
    *   **Username:** azureuser
    *   **Password:**  (Set a secure password)
    *   **Public inbound ports:** Allow SSH (22)

    ```azurecli
    # set variables
    location="australiaeast"
    rgName="iatd_labs_09_rg"
    vmName="iatd-vm-09"
    vnetName="iatd_labs_09_vnet"
    subnetName="default"
    nicName="iatd-nic-09"
    publicIpName="iatd-publicip-09"
    username="azureuser"
    password="<YourPassword>"

    #Create a public IP address
    az network public-ip create --resource-group $rgName --name $publicIpName --location $location

    #Get subnet ID
    subnetId=$(az network vnet subnet show --name $subnetName --vnet-name $vnetName --resource-group $rgName --query id --output tsv)

    #Create Network Interface
    az network nic create --resource-group $rgName --name $nicName --location $location --subnet $subnetId --public-ip-address $publicIpName

    #Create VM
    az vm create --resource-group $rgName --name $vmName --location $location --nics $nicName --image Ubuntu2204 --admin-username $username --admin-password $password
    ```

2.  **Connect to the Virtual Machine:** Use an SSH client (e.g., PuTTY, OpenSSH) to connect to the VM using its public IP address. Use the username and password you specified during VM creation.

3.  **Verify DNS Resolution:**
    *   Once connected to the VM, use `nslookup` or `dig` to query the DNS records in your private zone.  If `nslookup` is not found install it with `sudo apt install dnsutils`

    ```bash
    nslookup webserver.private.iatdlabs.com
    ```

    Expected output:
    ```
    Server:         127.0.0.53
    Address:        127.0.0.53#53

    Name:   webserver.private.iatdlabs.com
    Address: 172.16.0.100
    ```

    ```bash
    nslookup www.private.iatdlabs.com
    ```

    Expected output:
    ```
    Server:         127.0.0.53
    Address:        127.0.0.53#53

    www.private.iatdlabs.com canonical name = webserver.private.iatdlabs.com.
    Name:   webserver.private.iatdlabs.com
    Address: 172.16.0.100
    ```

    If the `nslookup` commands return the correct IP address (172.16.0.100), you have successfully configured your private DNS zone!

### Cleanup

To avoid incurring charges, it's essential to clean up the resources you created:

```azurecli
# Delete resource group
az group delete --name iatd_labs_09_rg --yes
```

### Key Takeaways

*   **Private DNS Zones:** Private DNS zones allow you to host your DNS domain in Azure.
*   **Virtual Network Linking:** You can link a private DNS zone to a virtual network to enable DNS resolution within the network.
*   **DNS Record Management:** You can manage DNS records, such as A and CNAME records, within your private DNS zone.

**Congratulations!** You have successfully completed Lab 9. In this lab, you learned how to create a private DNS zone in Azure, link it to a virtual network, add DNS records, and verify name resolution from a virtual machine within the network.
