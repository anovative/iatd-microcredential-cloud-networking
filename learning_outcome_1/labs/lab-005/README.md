## IATD Microcredential Cloud Networking: Lab 5 - Assigning IP Addresses to Virtual Machines in Azure

**Objective:** In this lab, you will learn how to configure both static private and public IP addresses for Azure Virtual Machines. You'll also explore dynamic IP allocation.

**Estimated Time:** 45 - 60 minutes

**Prerequisites:**

1.  **Azure Subscription:** You need an active Azure subscription.
2.  **Existing Virtual Network:** You'll need an existing Virtual Network (VNet) and subnet. If you don't have one, create it using the methods learned in Lab 1.
3.  **Existing Virtual Machine:** You should have an existing Azure Virtual Machine. If you don't have one, create a basic one (e.g., using the Azure Portal) before starting this lab. You can use a Linux or Windows VM.

**Lab Conventions:**

*   **Naming Conventions:** All resources created/modified in this lab will be prefixed with `iatd_labs_05` for easy identification and cleanup.
*   **IP Address Range:** The `172.16.0.0/16` range will be used (as per standardization).
*   **Location:** Choose a consistent Azure region.

**Let's get started!**

### Part 1: Create a Resource Group (If You Don't Have One)

If you already have a resource group with a VNet and VM, skip this task. Otherwise, create one:

**Method: Azure CLI**

1.  **Open Azure Cloud Shell:** Go to the Azure portal and launch Cloud Shell (Bash).

2.  **Create a Resource Group:**

    ```azurecli
    az group create --name iatd_labs_05_rg --location australiaeast
    ```

    **Example Output:**
    
    ```json
    {
      "id": "/subscriptions/your_subscription_id/resourceGroups/iatd_labs_05_rg",
      "location": "australiaeast",
      "managedBy": null,
      "name": "iatd_labs_05_rg",
      "properties": {
        "provisioningState": "Succeeded"
      },
      "tags": null,
      "type": "Microsoft.Resources/resourceGroups"
    }
    ```

### Part 2: Create a Virtual Network (If You Don't Have One)

If you already have a VNet, skip this task. Otherwise, create one:

**Method: Azure CLI**

1.  **Create VNet:** Execute the following command in Cloud Shell:

    ```azurecli
    az network vnet create \
      --resource-group iatd_labs_05_rg \
      --name iatd_labs_05_vnet \
      --address-prefixes 172.16.0.0/16 \
      --subnet-name iatd_labs_05_subnet \
      --subnet-prefixes 172.16.0.0/24 \
      --location australiaeast
    ```
    
    **Example Output:**
    
    ```json
    {
      "newVNet": {
        "addressSpace": {
          "addressPrefixes": [
            "172.16.0.0/16"
          ]
        },
        "enableDdosProtection": false,
        "id": "/subscriptions/your_subscription_id/resourceGroups/iatd_labs_05_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_05_vnet",
        "location": "australiaeast",
        "name": "iatd_labs_05_vnet",
        "provisioningState": "Succeeded",
        "resourceGroup": "iatd_labs_05_rg",
        "subnets": [
          {
            "addressPrefix": "172.16.0.0/24",
            "id": "/subscriptions/your_subscription_id/resourceGroups/iatd_labs_05_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_05_vnet/subnets/iatd_labs_05_subnet",
            "name": "iatd_labs_05_subnet",
            "provisioningState": "Succeeded",
            "resourceGroup": "iatd_labs_05_rg"
          }
        ],
        "type": "Microsoft.Network/virtualNetworks"
      }
    }
    ```

### Part 3: Create a Virtual Machine (If You Don't Have One)

If you already have a VM, skip this task. Otherwise, create one. Ensure you create a Public IP Address when creating the VM, and associate this IP address with the Network Interface

**Method: Azure CLI**

```azurecli
az vm create \
  --resource-group iatd_labs_05_rg \
  --name iatd_labs_05_vm \
  --image UbuntuLTS \
  --admin-username azureuser \
  --generate-ssh-keys \
  --vnet-name iatd_labs_05_vnet \
  --subnet iatd_labs_05_subnet \
  --public-ip-sku Standard
```

**Example Output:**

```json
{
  "fqdns": "",
  "id": "/subscriptions/your_subscription_id/resourceGroups/iatd_labs_05_rg/providers/Microsoft.Compute/virtualMachines/iatd_labs_05_vm",
  "location": "australiaeast",
  "macAddress": "00-0D-3A-XX-XX-XX",
  "powerState": "VM running",
  "privateIpAddress": "172.16.0.4",
  "publicIpAddress": "20.XX.XX.XX",
  "resourceGroup": "iatd_labs_05_rg",
  "zones": ""
}
```

### Part 4: Assigning a Static Private IP Address to a Virtual Machine

We will use the Azure Portal for this Task

1.  **Find your Virtual Machine:** In the Azure portal, search for "Virtual machines" and select "Virtual machines". Select the `iatd_labs_05_vm` (or your existing VM).

2.  **Navigate to Networking:** In the left-hand menu, under "Settings," click "Networking."

3.  **Access the Network Interface:** You will see the network interface associated with the VM (e.g., `iatd_labs_05_vmNetworkInterface`). Click on it.

4.  **Configure IP Configuration:** In the left-hand menu of the Network Interface, under "Settings," click "IP configurations."

5.  **Select IP Configuration:** You will likely see an IP configuration named "ipconfig1" or similar. Click on it.

6.  **Change Assignment to Static:**

    *   Under "Private IP address settings", change the "Assignment" from "Dynamic" to "Static."

7.  **Choose a Static IP Address:**

    *   Azure will suggest an available static IP address from within your subnet's address range.  You can either accept this or manually enter a different available IP address. **Important:** Choose an IP address *outside* of the dynamic IP address range configured for your subnet, if any is configured.  For example, if your subnet is `172.16.0.0/24` and you want to assign the static IP `172.16.0.10`, ensure that `172.16.0.10` isn't within a dynamic range. Set the IP address to `172.16.0.10`.

8.  **Save the Configuration:** Click "Save." Azure will update the IP configuration.

9.  **Reboot the VM (Important):** For the static IP address assignment to take full effect within the VM's operating system, *reboot the virtual machine*.

### Part 5: Configuring a Virtual Machine to Use Dynamic IP Allocation

Now, we will revert the VM to using Dynamic IP Allocation.  We'll continue using the Azure Portal.

1.  **Repeat Steps 1-5 from Task 4** to navigate to the IP configuration settings for your VM's network interface.

2.  **Change Assignment to Dynamic:** Under "Private IP address settings," change the "Assignment" from "Static" back to "Dynamic."

3.  **Save the Configuration:** Click "Save."

4.  **Reboot the VM:** Reboot the VM for the changes to take full effect within the operating system. The VM will now obtain a new IP Address dynamically from the available address pool.

### Part 6: Allocating a Static Public IP Address

When you create a VM you typically create a Standard Public IP which will be Dynamic. To make it static:

**Method: Azure Portal**

1.  **Find the Public IP Address:** In the Azure portal, search for "Public IP addresses." Locate the Public IP address associated with your `iatd_labs_05_vm`. (if not see prereq requirements above).

2.  **Change Assignment to Static:**
    *   Under "Settings", select Configuration
    *   Change Assignment from "Dynamic" to "Static"
    *   Click Save

### Post-Lab: Cleanup

To avoid incurring unnecessary costs, it's essential to clean up the resources created during this lab.

1.  **Delete Resource Group:**

    ```azurecli
    az group delete --name iatd_labs_05_rg --yes --no-wait
    ```
    
    **Example Output:**
    
    ```
    (No output due to --no-wait parameter, but the deletion process will start in the background)
    ```

### Verification

*   **Static Private IP:** After rebooting the VM with a static private IP, log into the VM and use OS-specific tools (e.g., `ipconfig` on Windows, `ip addr` on Linux) to verify that the VM is using the assigned static IP address.
*   **Dynamic Private IP:** After reverting to dynamic allocation and rebooting, verify that the VM has a *different* IP address than the static one you assigned. The command to use for verification remains the same as above (`ipconfig` or `ip addr`).
*   **Static Public IP:** After assigning a static public IP, check the Public IP address in the portal is the same IP after stopping and starting the VM.

### Important Considerations

*   **Static IP Addresses and DNS:** When using static IP addresses, you may need to manually configure DNS records to map hostnames to the IP addresses for proper name resolution.
*   **Subnet IP Ranges:** Be careful when choosing static IP addresses. Avoid using addresses reserved by Azure (the first four and the last IP address in each subnet). Also, ensure the address is not within any dynamic IP address range configured on the subnet.
*   **VM Reboots:** Remember to *always* reboot the VM after making changes to its IP configuration for the changes to take full effect.

**Congratulations!** This lab guide should give you a good practical understanding of managing IP addresses for your VMs in Azure. Make sure you clean up to avoid unexpected billing!
