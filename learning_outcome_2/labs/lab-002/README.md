## IATD Microcredential Cloud Networking: Lab 2 - Examining Azure System Routes

**Objective:** Explore and understand the system routes automatically configured in an Azure virtual network, including their purpose, behavior, and how they facilitate network communication.

**Estimated Time:** 60 - 75 minutes

**Prerequisites:**

1.  **Azure Subscription:** An active Azure subscription is required. A free account is available at [https://azure.microsoft.com/free/](https://azure.microsoft.com/free/).
2.  **Basic Networking Knowledge:** Familiarity with virtual networks, subnets, IP addressing, and basic routing concepts.

**Let's get started!**

### Lab Conventions

*   **Azure Cloud Shell:** PowerShell and CLI interactions will occur within the Azure Cloud Shell environment.
*   **Naming Conventions:** Resource names follow the convention `iatd_labs_02_*`.
*   **IP Address Range:** Using `172.16.0.0/16` for consistency across labs.
*   **Location:** Choose a consistent Azure region.

#### Resource Naming

*   **Resource Group:** `iatd_labs_02_rg`
*   **Virtual Network:** `iatd_labs_02_vnet`
*   **Backend Subnet:** `backend-subnet`
*   **Frontend Subnet:** `frontend-subnet`
*   **Backend VM:** `backend-vm`
*   **Frontend VM:** `frontend-vm`
*   **Public IPs:** `<vm-name>-pip`

### Part 1: Setting up the Environment - Azure Portal

1.  **Sign in to the Azure Portal:**
    *   Browse to [https://portal.azure.com/](https://portal.azure.com/) and sign in.

2.  **Create Resource Group:**
    *   Search for "Resource groups" and select.
    *   Click **Create**.
    *   Select your subscription.
    *   Name: `iatd_labs_02_rg`
    *   Region: Select your region.
    *   Click **Review + create**, then **Create**.

3.  **Create Virtual Network:**
    *   Search for "Virtual networks" and select.
    *   Click **Create**.
    *   **Basics Tab:**
        *   Subscription: Your subscription.
        *   Resource group: `iatd_labs_02_rg`
        *   Name: `iatd_labs_02_vnet`
        *   Region: Your region.
    *   **IP Addresses Tab:**
        *   IPv4 address space: `172.16.0.0/16`
        *   Add two subnets:
            1. Name: `backend-subnet`, Address range: `172.16.0.0/24`
            2. Name: `frontend-subnet`, Address range: `172.16.1.0/24`
    *   Click **Review + create**, then **Create**.

### Part 2: Creating Test VMs - Azure CLI

1.  **Open Cloud Shell:** In the Azure Portal, click the Cloud Shell icon. Select **Bash**.

2.  **Set Variables:**

    ```bash
    # Set variables
    RESOURCE_GROUP="iatd_labs_02_rg"
    LOCATION="australiaeast"  # Replace with your region
    VNET_NAME="iatd_labs_02_vnet"
    BACKEND_SUBNET="backend-subnet"
    FRONTEND_SUBNET="frontend-subnet"

    # Create backend VM
    az vm create \
      --resource-group $RESOURCE_GROUP \
      --name backend-vm \
      --image UbuntuLTS \
      --vnet-name $VNET_NAME \
      --subnet $BACKEND_SUBNET \
      --size Standard_B1ls \
      --admin-username azureuser \
      --generate-ssh-keys \
      --public-ip-sku Standard \
      --public-ip-address backend-vm-pip
    ```

    **Expected Output:**
    ```json
    {
      "fqdns": "",
      "id": "/subscriptions/your-subscription-id/resourceGroups/iatd_labs_02_rg/providers/Microsoft.Compute/virtualMachines/backend-vm",
      "location": "australiaeast",
      "macAddress": "00-0D-3A-1B-2C-3D",
      "powerState": "VM running",
      "privateIpAddress": "172.16.0.4",
      "publicIpAddress": "20.53.123.45",
      "resourceGroup": "iatd_labs_02_rg",
      "zones": ""
    }
    ```

    ```bash
    # Create frontend VM
    az vm create \
      --resource-group $RESOURCE_GROUP \
      --name frontend-vm \
      --image UbuntuLTS \
      --vnet-name $VNET_NAME \
      --subnet $FRONTEND_SUBNET \
      --size Standard_B1ls \
      --admin-username azureuser \
      --generate-ssh-keys \
      --public-ip-sku Standard \
      --public-ip-address frontend-vm-pip
    ```

    **Expected Output:**
    ```json
    {
      "fqdns": "",
      "id": "/subscriptions/your-subscription-id/resourceGroups/iatd_labs_02_rg/providers/Microsoft.Compute/virtualMachines/frontend-vm",
      "location": "australiaeast",
      "macAddress": "00-0D-3A-4E-5F-6G",
      "powerState": "VM running",
      "privateIpAddress": "172.16.1.4",
      "publicIpAddress": "20.53.234.56",
      "resourceGroup": "iatd_labs_02_rg",
      "zones": ""
    }
    ```

### Part 3: Examining System Routes - PowerShell

1.  **Switch to PowerShell:** In Cloud Shell, select PowerShell.

2.  **View Effective Routes:**

    ```powershell
    # Get the Network Interface ID for the backend VM
    $nicId = $(az vm show -g $RESOURCE_GROUP -n backend-vm --query "networkProfile.networkInterfaces[0].id" -o tsv)
    
    # View effective routes
    az network nic show-effective-route-table --ids $nicId -o table
    ```

    **Expected Output:**
    ```
    Source    State    Address Prefix    Next Hop Type     Next Hop IP
    --------  -------  ---------------   ---------------   -----------
    Default   Active   172.16.0.0/16    VnetLocal        -
    Default   Active   0.0.0.0/0        Internet         -
    Default   Active   10.0.0.0/8       None             -
    Default   Active   192.168.0.0/16   None             -
    Default   Active   100.64.0.0/10    None             -
    ```

### Part 4: Testing Network Connectivity

1.  **Test Internal VNet Communication:**

    ```powershell
    # Get frontend VM's private IP
    $frontendIP = $(az vm show -g $RESOURCE_GROUP -n frontend-vm --query "privateIps" -o tsv)
    
    # Test connection to frontend VM
    Test-NetConnection -ComputerName $frontendIP -Port 3389
    ```

    **Expected Output:**
    ```
    ComputerName           : 172.16.1.4
    RemoteAddress          : 172.16.1.4
    RemotePort            : 3389
    InterfaceAlias        : Vnet
    SourceAddress         : 172.16.0.4
    PingSucceeded         : True
    TcpTestSucceeded      : True
    ```

2.  **Test Internet Connectivity:**

    ```powershell
    Test-NetConnection -ComputerName www.microsoft.com -Port 80
    ```

    **Expected Output:**
    ```
    ComputerName           : www.microsoft.com
    RemoteAddress          : 23.45.229.117
    RemotePort            : 80
    InterfaceAlias        : Vnet
    SourceAddress         : 172.16.0.4
    PingSucceeded         : True
    TcpTestSucceeded      : True
    ```

### Part 5: Cleanup

1.  **Delete Resource Group:**

    *   **Azure Portal:**
        *   Navigate to the Resource Group `iatd_labs_02_rg`
        *   Click **Delete resource group**
        *   Enter the resource group name to confirm
        *   Click **Delete**

    *   **Azure CLI:**
        ```bash
        az group delete --name iatd_labs_02_rg --yes --no-wait
        ```

### Post-Lab Questions

1. What are the default system routes in an Azure VNet and why are they important?
2. How does the VnetLocal route facilitate communication between subnets?
3. Under what circumstances would you need to override system routes with UDRs?

**Additional Resources:**
- [Azure Virtual Network Traffic Routing](https://docs.microsoft.com/azure/virtual-network/virtual-networks-udr-overview)
- [System Routes Documentation](https://docs.microsoft.com/azure/virtual-network/virtual-networks-udr-overview#system-routes)
