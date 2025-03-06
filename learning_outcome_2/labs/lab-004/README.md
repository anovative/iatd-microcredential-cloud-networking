## IATD Microcredential Cloud Networking: Lab 4 - Associating Route Tables with Subnets

**Objective:** Learn to associate route tables with subnets and verify effective routes to control network traffic within Azure virtual networks.

**Estimated Time:** 60 - 75 minutes

**Prerequisites:**

1.  **Azure Subscription:** An active Azure subscription is required. A free account is available at [https://azure.microsoft.com/free/](https://azure.microsoft.com/free/).
2.  **Networking Knowledge:** Familiarity with virtual networks, subnets, IP addressing, and User Defined Routes.
3.  **Previous Labs:** Completion of Labs 1-3.

**Let's get started!**

### Lab Conventions

*   **Azure Cloud Shell:** PowerShell and CLI interactions will occur within the Azure Cloud Shell environment.
*   **Naming Conventions:** Resource names follow the convention `iatd_labs_04_*`.
*   **IP Address Range:** Using `172.16.0.0/16` for consistency across labs.
*   **Location:** Choose a consistent Azure region.

#### Resource Naming

*   **Resource Group:** `iatd_labs_04_rg`
*   **Virtual Network:** `iatd_labs_04_vnet`
*   **Subnet:** `backend-subnet`
*   **Virtual Machine:** `iatd_labs_04_vm`
*   **Route Table:** `iatd_labs_04_rt`
*   **Public IP:** `iatd_labs_04_vm_pip`

### Part 1: Infrastructure Setup - Azure CLI

1.  **Open Cloud Shell:** In the Azure Portal, click the Cloud Shell icon. Select **Bash**.

2.  **Create Basic Infrastructure:**

    ```bash
    # Set variables
    RESOURCE_GROUP="iatd_labs_04_rg"
    LOCATION="australiaeast"  # Replace with your region
    VNET_NAME="iatd_labs_04_vnet"
    SUBNET_NAME="backend-subnet"

    # Create resource group
    az group create \
      --name $RESOURCE_GROUP \
      --location $LOCATION

    # Create VNet and subnet
    az network vnet create \
      --resource-group $RESOURCE_GROUP \
      --name $VNET_NAME \
      --address-prefixes "172.16.0.0/16" \
      --subnet-name $SUBNET_NAME \
      --subnet-prefixes "172.16.0.0/24"
    ```

    **Expected Output:**
    ```json
    {
      "newVNet": {
        "addressSpace": {
          "addressPrefixes": [
            "172.16.0.0/16"
          ]
        },
        "name": "iatd_labs_04_vnet",
        "provisioningState": "Succeeded",
        "resourceGroup": "iatd_labs_04_rg",
        "subnets": [
          {
            "addressPrefix": "172.16.0.0/24",
            "name": "backend-subnet",
            "provisioningState": "Succeeded"
          }
        ]
      }
    }
    ```

### Part 2: Route Table Creation - Azure CLI

1.  **Create Route Table:**

    ```bash
    # Create route table
    az network route-table create \
      --resource-group $RESOURCE_GROUP \
      --name iatd_labs_04_rt \
      --location $LOCATION

    # Add a custom route
    az network route-table route create \
      --resource-group $RESOURCE_GROUP \
      --route-table-name iatd_labs_04_rt \
      --name default-route \
      --address-prefix "0.0.0.0/0" \
      --next-hop-type VirtualNetworkGateway
    ```

    **Expected Output:**
    ```json
    {
      "addressPrefix": "0.0.0.0/0",
      "name": "default-route",
      "nextHopType": "VirtualNetworkGateway",
      "provisioningState": "Succeeded"
    }
    ```

### Part 3: Route Table Association - Azure CLI

1.  **Associate Route Table with Subnet:**

    ```bash
    # Associate route table with subnet
    az network vnet subnet update \
      --resource-group $RESOURCE_GROUP \
      --vnet-name $VNET_NAME \
      --name $SUBNET_NAME \
      --route-table iatd_labs_04_rt
    ```

### Part 4: Test VM Creation and Verification

1.  **Create Test VM:**

    ```bash
    # Create VM
    az vm create \
      --resource-group $RESOURCE_GROUP \
      --name iatd_labs_04_vm \
      --image UbuntuLTS \
      --size Standard_B1ls \
      --admin-username azureuser \
      --generate-ssh-keys \
      --vnet-name $VNET_NAME \
      --subnet $SUBNET_NAME \
      --public-ip-address iatd_labs_04_vm_pip \
      --public-ip-sku Standard
    ```

2.  **Verify Effective Routes:**

    ```bash
    # Get VM's NIC ID
    VM_NIC_ID=$(az vm show \
      --resource-group $RESOURCE_GROUP \
      --name iatd_labs_04_vm \
      --query "networkProfile.networkInterfaces[0].id" -o tsv)

    # View effective routes
    az network nic show-effective-route-table \
      --ids $VM_NIC_ID \
      --output table
    ```

    **Expected Output:**
    ```
    Source    State    Address Prefix    Next Hop Type           Next Hop IP
    --------  -------  ---------------   -------------------     -----------
    Default   Active   172.16.0.0/16    VnetLocal              -
    User      Active   0.0.0.0/0        VirtualNetworkGateway  -
    ```

### Part 5: Network Connectivity Testing

1.  **Test Network Connectivity:**

    ```bash
    # Get VM's public IP
    VM_IP=$(az network public-ip show \
      --resource-group $RESOURCE_GROUP \
      --name iatd_labs_04_vm_pip \
      --query ipAddress -o tsv)

    # SSH to VM and test connectivity
    ssh azureuser@$VM_IP

    # Test internet connectivity
    ping 8.8.8.8
    traceroute 8.8.8.8
    ```

2.  **Use Network Watcher:**

    ```bash
    # Get VM resource ID
    VM_ID=$(az vm show \
      --resource-group $RESOURCE_GROUP \
      --name iatd_labs_04_vm \
      --query id -o tsv)

    # Test connectivity
    az network watcher test-connectivity \
      --resource-group $RESOURCE_GROUP \
      --source-resource $VM_ID \
      --dest-address www.microsoft.com \
      --dest-port 80
    ```

### Part 6: Cleanup

1.  **Delete All Resources:**

    *   **Azure Portal:**
        *   Navigate to Resource Group `iatd_labs_04_rg`
        *   Click **Delete resource group**
        *   Enter the resource group name to confirm
        *   Click **Delete**

    *   **Azure CLI:**
        ```bash
        az group delete --name $RESOURCE_GROUP --yes
        ```

### Post-Lab Questions

1. **Route Table Association:**
   - What happens to existing connections when associating a route table?
   - Can multiple route tables be associated with a single subnet?

2. **Route Processing:**
   - How does Azure process multiple matching routes?
   - What is the order of precedence for different route types?

3. **Troubleshooting:**
   - How can you verify route table association is working?
   - What tools are available for route troubleshooting?

**Additional Resources:**
- [Route Tables Overview](https://docs.microsoft.com/azure/virtual-network/virtual-networks-udr-overview)
- [Network Watcher](https://docs.microsoft.com/azure/network-watcher/network-watcher-monitoring-overview)
- [Troubleshooting Routes](https://docs.microsoft.com/azure/virtual-network/virtual-network-routes-troubleshoot-portal)
