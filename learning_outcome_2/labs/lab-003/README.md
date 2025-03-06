## IATD Microcredential Cloud Networking: Lab 3 - Configuring Different Next Hop Types

**Objective:** Configure User Defined Routes (UDRs) with different next hop types to control traffic flow within and out of an Azure Virtual Network.

**Estimated Time:** 75-90 minutes

**Prerequisites:**

1.  **Azure Subscription:** Active Azure subscription ([https://azure.microsoft.com/free/](https://azure.microsoft.com/free/)).
2.  **Networking Fundamentals:** Understanding of VNETs, subnets, IP addressing, routing, Network Virtual Appliances, and VNET Peering.
3.  **Previous Labs:** Completion of Labs 1 and 2.

**Let's get started!**

### Lab Conventions

*   **Azure Cloud Shell:** PowerShell and CLI interactions will occur within the Azure Cloud Shell environment.
*   **Naming Conventions:** Resource names follow the convention `iatd_labs_03_*`.
*   **IP Address Range:** Using `172.16.0.0/16` for consistency across labs.
*   **Location:** Choose a consistent Azure region.

#### Resource Naming

*   **Resource Group:** `iatd_labs_03_rg`
*   **Virtual Network (Main):** `iatd_labs_03_vnet_main`
*   **Virtual Network (Peered):** `iatd_labs_03_vnet_peered`
*   **Subnets:**
    * Protected: `protected-subnet`
    * NVA: `nva-subnet`
    * Peered: `peered-subnet`
*   **Virtual Machines:**
    * NVA: `iatd_labs_03_nva`
    * Protected: `iatd_labs_03_vm_protected`
    * Peered: `iatd_labs_03_vm_peered`
*   **Route Table:** `iatd_labs_03_rt_protected`

### Part 1: Infrastructure Setup - Azure CLI

1.  **Open Cloud Shell:** In the Azure Portal, click the Cloud Shell icon. Select **Bash**.

2.  **Create Resource Group and Networks:**

    ```bash
    # Set variables
    RESOURCE_GROUP="iatd_labs_03_rg"
    LOCATION="australiaeast"  # Replace with your region
    VNET_MAIN="iatd_labs_03_vnet_main"
    VNET_PEERED="iatd_labs_03_vnet_peered"

    # Create resource group
    az group create --name $RESOURCE_GROUP --location $LOCATION

    # Create main VNet and subnets
    az network vnet create \
      --resource-group $RESOURCE_GROUP \
      --name $VNET_MAIN \
      --address-prefixes "172.16.0.0/16" \
      --subnet-name protected-subnet \
      --subnet-prefixes "172.16.1.0/24"

    az network vnet subnet create \
      --resource-group $RESOURCE_GROUP \
      --vnet-name $VNET_MAIN \
      --name nva-subnet \
      --address-prefixes "172.16.2.0/24"

    # Create peered VNet
    az network vnet create \
      --resource-group $RESOURCE_GROUP \
      --name $VNET_PEERED \
      --address-prefixes "172.16.100.0/24" \
      --subnet-name peered-subnet \
      --subnet-prefixes "172.16.100.0/24"
    ```

### Part 2: NVA Setup - Azure CLI

1.  **Create NVA Components:**

    ```bash
    # Create public IP for NVA
    az network public-ip create \
      --resource-group $RESOURCE_GROUP \
      --name iatd_labs_03_nva_pip \
      --sku Standard \
      --allocation-method Static

    # Get subnet ID for NVA
    NVA_SUBNET_ID=$(az network vnet subnet show \
      --resource-group $RESOURCE_GROUP \
      --vnet-name $VNET_MAIN \
      --name nva-subnet \
      --query id -o tsv)

    # Create NIC for NVA
    az network nic create \
      --resource-group $RESOURCE_GROUP \
      --name iatd_labs_03_nva_nic \
      --subnet $NVA_SUBNET_ID \
      --public-ip-address iatd_labs_03_nva_pip \
      --ip-forwarding true

    # Create NVA VM
    az vm create \
      --resource-group $RESOURCE_GROUP \
      --name iatd_labs_03_nva \
      --image UbuntuLTS \
      --size Standard_B1ls \
      --admin-username azureuser \
      --generate-ssh-keys \
      --nics iatd_labs_03_nva_nic
    ```

2.  **Configure IP Forwarding on NVA:**

    ```bash
    # Get NVA's public IP
    NVA_IP=$(az network public-ip show \
      --resource-group $RESOURCE_GROUP \
      --name iatd_labs_03_nva_pip \
      --query ipAddress -o tsv)

    # SSH into NVA and enable IP forwarding
    ssh azureuser@$NVA_IP "sudo bash -c 'echo \"net.ipv4.ip_forward=1\" > /etc/sysctl.d/forwarding.conf && sysctl -p /etc/sysctl.d/forwarding.conf'"
    ```

### Part 3: Route Table Configuration - Azure CLI

1.  **Create and Configure Route Table:**

    ```bash
    # Create route table
    az network route-table create \
      --resource-group $RESOURCE_GROUP \
      --name iatd_labs_03_rt_protected

    # Get NVA private IP
    NVA_PRIVATE_IP=$(az network nic show \
      --resource-group $RESOURCE_GROUP \
      --name iatd_labs_03_nva_nic \
      --query "ipConfigurations[0].privateIpAddress" -o tsv)

    # Add route to peered network via NVA
    az network route-table route create \
      --resource-group $RESOURCE_GROUP \
      --route-table-name iatd_labs_03_rt_protected \
      --name route_to_peered \
      --address-prefix "172.16.100.0/24" \
      --next-hop-type VirtualAppliance \
      --next-hop-ip-address $NVA_PRIVATE_IP

    # Associate route table with protected subnet
    az network vnet subnet update \
      --resource-group $RESOURCE_GROUP \
      --vnet-name $VNET_MAIN \
      --name protected-subnet \
      --route-table iatd_labs_03_rt_protected
    ```

### Part 4: VNet Peering Configuration - Azure CLI

1.  **Create VNet Peering:**

    ```bash
    # Create peering from main to peered VNet
    az network vnet peering create \
      --name main_to_peered \
      --resource-group $RESOURCE_GROUP \
      --vnet-name $VNET_MAIN \
      --remote-vnet $VNET_PEERED \
      --allow-vnet-access

    # Create peering from peered to main VNet
    az network vnet peering create \
      --name peered_to_main \
      --resource-group $RESOURCE_GROUP \
      --vnet-name $VNET_PEERED \
      --remote-vnet $VNET_MAIN \
      --allow-vnet-access
    ```

### Part 5: Testing and Verification

1.  **Create Test VMs:**

    ```bash
    # Create protected VM
    az vm create \
      --resource-group $RESOURCE_GROUP \
      --name iatd_labs_03_vm_protected \
      --image UbuntuLTS \
      --size Standard_B1ls \
      --admin-username azureuser \
      --generate-ssh-keys \
      --vnet-name $VNET_MAIN \
      --subnet protected-subnet

    # Create peered VM
    az vm create \
      --resource-group $RESOURCE_GROUP \
      --name iatd_labs_03_vm_peered \
      --image UbuntuLTS \
      --size Standard_B1ls \
      --admin-username azureuser \
      --generate-ssh-keys \
      --vnet-name $VNET_PEERED \
      --subnet peered-subnet
    ```

2.  **Test Connectivity:**

    ```bash
    # Get protected VM's public IP
    PROTECTED_VM_IP=$(az network public-ip show \
      --resource-group $RESOURCE_GROUP \
      --name iatd_labs_03_vm_protected-ip \
      --query ipAddress -o tsv)

    # SSH to protected VM and test connectivity
    ssh azureuser@$PROTECTED_VM_IP

    # From protected VM, ping peered VM
    ping 172.16.100.4

    # Verify route through NVA
    traceroute 172.16.100.4
    ```

### Part 6: Cleanup

1.  **Delete All Resources:**

    *   **Azure Portal:**
        *   Navigate to Resource Group `iatd_labs_03_rg`
        *   Click **Delete resource group**
        *   Enter the resource group name to confirm
        *   Click **Delete**

    *   **Azure CLI:**
        ```bash
        az group delete --name $RESOURCE_GROUP --yes
        ```

### Post-Lab Questions

1. What are the key differences between VirtualAppliance and VirtualNetworkGateway as next hop types?
2. How does the order of route processing affect traffic flow?
3. What security considerations should be taken into account when implementing NVAs?

**Additional Resources:**
- [Virtual Network Peering](https://docs.microsoft.com/azure/virtual-network/virtual-network-peering-overview)
- [User-Defined Routes](https://docs.microsoft.com/azure/virtual-network/virtual-networks-udr-overview)
- [Network Virtual Appliances](https://docs.microsoft.com/azure/virtual-network/virtual-networks-udr-overview#user-defined)
