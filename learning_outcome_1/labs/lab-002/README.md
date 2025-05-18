## IATD Microcredential Cloud Networking: Lab 2 - Working with Subnets

**Objective:** This lab will guide you through creating subnets in Azure using the Azure Portal, Azure PowerShell and Azure CLI. You'll also learn to associate Network Security Groups (NSGs) and Route Tables to control network traffic, and configure service endpoints to securely access Azure services.

**Estimated Time:** 45 minutes - 1 hour

**Prerequisites:**

1.  **Azure Subscription:** You need an active Azure subscription.
2.  **Web Browser:** A modern web browser (Chrome, Firefox, Safari, Edge).
3.  **Cloud Shell:** We will use Azure Cloud Shell.
4.  **Virtual Network:** An existing virtual network is required. If you completed Lab 1, you can use a VNet created there. If not, create one using the Azure Portal, PowerShell, or CLI. The VNet should be named `iatd_labs_02_vnet`, in resource group `iatd_labs_02_rg`, with address space `172.16.0.0/16`.

**Lab Conventions:**

*   **Azure Cloud Shell:** All PowerShell and CLI interactions will occur within the Azure Cloud Shell environment.
*   **Naming Conventions:** Resource names follow the convention `iatd_labs_02_*`.
*   **IP Address Range:** The `172.16.0.0/16` range will be used (as per standardization).
*   **Location:** Choose a consistent Azure region.

**Let's get started!**

### Part 1: Creating Subnets

#### 1.1 Using the Azure Portal

1.  **Log in to the Azure Portal:** Open your web browser and navigate to [https://portal.azure.com](https://portal.azure.com). Log in using your Azure account credentials.
2.  **Navigate to the Virtual Network:** Search for "Virtual networks" and select your existing virtual network: `iatd_labs_02_vnet` (or the VNet you choose to use).
3.  **Add a Subnet:**
    *   In the left-hand menu, click "Subnets".
    *   Click "+ Subnet".
    *   For "Name", enter `iatd_labs_02_subnet_portal`.
    *   For "Subnet address range", enter `172.16.2.0/24`.
    *   Leave other settings as default and click "Save".

#### 1.2 Using Azure PowerShell via Cloud Shell

1.  **Launch Cloud Shell:** In the Azure portal, click the Cloud Shell icon. Ensure you are in the PowerShell environment.

2.  **Create the Subnet:**

    ```powershell
    $resourceGroupName = "iatd_labs_02_rg" # Replace if you are using a different resource group
    $vnetName = "iatd_labs_02_vnet"       # Replace if you are using a different vnet name
    $subnetName = "iatd_labs_02_subnet_ps"
    $subnetAddressPrefix = "172.16.3.0/24"

    $vnet = Get-AzVirtualNetwork -Name $vnetName -ResourceGroupName $resourceGroupName

    Add-AzVirtualNetworkSubnetConfig `
        -Name $subnetName `
        -AddressPrefix $subnetAddressPrefix `
        -VirtualNetwork $vnet

    $vnet | Set-AzVirtualNetwork
    ```

    **Example Output (truncated):**

    ```
    Name              : iatd_labs_02_vnet
    ResourceGroupName : iatd_labs_02_rg
    Location          : australiaeast
    Id                : /subscriptions/your_subscription_id/resourceGroups/iatd_labs_02_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_02_vnet
    Etag              : W/"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
    ResourceGuid      : yyyyyyyy-yyyy-yyyy-yyyy-yyyyyyyyyyyy
    ProvisioningState : Succeeded
    ...
    Subnets           : [
                           {
                             "Name": "subnet-1",
                             "Etag": "W/\"zzzzzzzz-zzzz-zzzz-zzzz-zzzzzzzzzzzz\"",
                             "Id": "/subscriptions/your_subscription_id/resourceGroups/iatd_labs_02_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_02_vnet/subnets/subnet-1",
                             "AddressPrefix": "172.16.1.0/24",
                             ...
                           },
                           {
                             "Name": "iatd_labs_02_subnet_ps",
                             "Etag": "W/\"aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa\"",
                             "Id": "/subscriptions/your_subscription_id/resourceGroups/iatd_labs_02_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_02_vnet/subnets/iatd_labs_02_subnet_ps",
                             "AddressPrefix": "172.16.3.0/24",
                             ...
                           }
                         ]
    ...
    ```

#### 1.3 Using Azure CLI via Cloud Shell

1.  **Launch Cloud Shell:** In the Azure portal, click the Cloud Shell icon. Ensure you are in the **Bash** environment.

2.  **Create the Subnet:**

    ```bash
    resourceGroupName="iatd_labs_02_rg" # Replace if you are using a different resource group
    vnetName="iatd_labs_02_vnet"       # Replace if you are using a different vnet name
    subnetName="iatd_labs_02_subnet_cli"
    subnetPrefix="172.16.4.0/24"

    az network vnet subnet create \
        --resource-group $resourceGroupName \
        --vnet-name $vnetName \
        --name $subnetName \
        --address-prefixes $subnetPrefix
    ```

    **Example Output:**

    ```json
    {
      "addressPrefix": "172.16.4.0/24",
      "delegations": [],
      "id": "/subscriptions/your_subscription_id/resourceGroups/iatd_labs_02_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_02_vnet/subnets/iatd_labs_02_subnet_cli",
      "name": "iatd_labs_02_subnet_cli",
      "networkSecurityGroup": null,
      "privateEndpointNetworkPolicies": "Disabled",
      "privateLinkServiceNetworkPolicies": "Enabled",
      "provisioningState": "Succeeded",
      "resourceGroup": "iatd_labs_02_rg",
      "serviceEndpointPolicies": [],
      "serviceEndpoints": []
    }
    ```

3.  **Verify Subnets:** In the Azure portal, navigate to your virtual network (`iatd_labs_02_vnet`). Click on "Subnets" to see the newly created subnets listed alongside the original one.

### Part 2: Associating NSGs and Route Tables to Subnets

#### 2.1 Create a New NSG and a New Route Table (Using the Azure Portal)

1.  **Create NSG:**
    *   Search for "Network security groups" and click "+ Create".
    *   Select your subscription.
    *   Select resource group `iatd_labs_02_rg`.
    *   Enter name `iatd_labs_02_nsg`.
    *   Select your region (e.g., Australia East).
    *   Click "Review + create" and then "Create".

2.  **Create Route Table:**
    *   Search for "Route tables" and click "+ Create".
    *   Select your subscription.
    *   Select resource group `iatd_labs_02_rg`.
    *   Enter name `iatd_labs_02_routetable`.
    *   Select your region (e.g., Australia East).
    *   Click "Review + create" and then "Create".

#### 2.2 Associate the NSG and Route Table with an Existing Subnet (Using the Azure Portal)

1.  **Navigate to the Subnet:** Go to your virtual network (`iatd_labs_02_vnet`) and click "Subnets". Select `iatd_labs_02_subnet_portal`.

2.  **Associate NSG:**
    *   Click on "Network security group".
    *   Choose `iatd_labs_02_nsg` from the dropdown.
    *   Click "Save".

3.  **Associate Route Table:**
    *   Click on "Route table".
    *   Choose `iatd_labs_02_routetable` from the dropdown.
    *   Click "Save".

#### 2.3 Verify that the NSG and Route Table are Applied to the Subnet (Using the Azure Portal)

1.  Navigate back to the subnet (`iatd_labs_02_subnet_portal`).
2.  In the "Overview" section of the subnet, verify that "Network security group" shows `iatd_labs_02_nsg` and "Route table" shows `iatd_labs_02_routetable`.

### Part 3: Configuring Service Endpoints on Subnets

#### 3.1 Create an Azure Storage Account (Using the Azure Portal)

1.  **Create Storage Account:**
    *   Search for "Storage accounts" and click "+ Create".
    *   Select your subscription.
    *   Select resource group `iatd_labs_02_rg`.
    *   Enter a unique name (e.g., `iatdlabs02storage[your initials]`). *Important: Storage account names must be globally unique.*
    *   Select your region (e.g., Australia East).
    *   For "Performance", select "Standard".
    *   For "Redundancy", select "Locally-redundant storage (LRS)".
    *   Click "Review + create" and then "Create".

#### 3.2 Enable a Service Endpoint for Azure Storage on an Existing Subnet (Using the Azure Portal)

1.  **Navigate to the Subnet:** Go to your virtual network (`iatd_labs_02_vnet`) and click "Subnets". Select `iatd_labs_02_subnet_portal`.

2.  **Configure Service Endpoint:**
    *   Under "Service endpoints", select "Microsoft.Storage" from the "Services" dropdown.
    *   Click "Save".

#### 3.3 Configure the Storage Account's Firewall (Using the Azure Portal)

1.  **Navigate to the Storage Account:** Go to the storage account you created (e.g., `iatdlabs02storage[your initials]`).
2.  **Configure Firewall:**
    *   In the left-hand menu, under "Security + networking", click "Networking".
    *   Select "Enabled from selected virtual networks and IP addresses".
    *   Under "Virtual networks", click "+ Add virtual network".
    *   Select your subscription.
    *   Select virtual network `iatd_labs_02_vnet`.
    *   Select subnet `iatd_labs_02_subnet_portal`.
    *   Click "Add".
    *   Click "Save".

#### 3.4 Verify Access (Conceptual)

To fully verify, you would need to:

1.  Create a virtual machine within the `iatd_labs_02_subnet_portal` subnet.
2.  Attempt to access the Storage Account from the VM. You should be able to access it without needing to configure public IP addresses or other external access methods. If you try to access it from a VM *outside* the subnet, access should be denied unless other firewall rules are configured.

Since creating a VM is beyond the scope of this lab, this step is conceptual.

### Post-Lab: Cleanup

To avoid incurring unnecessary costs, it's essential to clean up the resources created during this lab.

1.  **Delete Storage Account:**

    *   **Azure Portal:** Navigate to your storage account and click "Delete". Confirm the deletion by typing the storage account name, and click "Delete".

2.  **Delete NSG and Route Table:**

    *   **Azure Portal:** Delete `iatd_labs_02_nsg` and `iatd_labs_02_routetable` from the Azure Portal.

3.  **Delete Subnets:**

    *   **Azure Portal:** Navigate to your virtual network (`iatd_labs_02_vnet`). Delete subnets `iatd_labs_02_subnet_portal` , `iatd_labs_02_subnet_ps`, and `iatd_labs_02_subnet_cli`.

4.  **Delete Resource Group:**

    *   **Azure Portal:** Delete resource group `iatd_labs_02_rg`.

        Alternatively, use Azure PowerShell:

        ```powershell
        Remove-AzResourceGroup -Name "iatd_labs_02_rg" -Force
        ```

        Or, use Azure CLI:

        ```bash
        az group delete --name "iatd_labs_02_rg" --yes
        ```

**Congratulations!** You have successfully created subnets, associated NSGs and Route Tables, and configured service endpoints. This lab provided hands-on experience with network segmentation and security within Azure Virtual Networks.
