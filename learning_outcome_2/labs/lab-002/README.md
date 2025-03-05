Okay, let's craft that Lab Guide on examining Azure System Routes, aiming for that sweet spot balance between Azure Portal and PowerShell/CLI:

## IATD Microcredential Cloud Networking: Lab 2 - Examining Azure System Routes

**Objective:** Explore the system routes automatically configured in an Azure virtual network.

**Scenario:** You have deployed a virtual network in Azure with several subnets and a VPN gateway connection to your on-premises network. This lab will examine the routes Azure automatically creates (system routes), understand their purpose, and verify basic connectivity. Even without creating custom User Defined Routes (UDRs), a default route table always exists.

**Outcomes:**

*   Inspection of the system routes in a route table (the default route table).
*   Identification of the VNet Local, On-premises (if a VPN gateway exists), and Internet routes.
*   Understanding of the purpose of each system route.
*   Verification of connectivity based on system routes.
*   Demonstrate local VNet, internet bound and gateway bound traffic flow with simple PowerShell testing.

**Estimated Time:** 60 - 75 minutes

**Prerequisites:**

1.  **Azure Subscription:** An active Azure subscription is required. A free account is available at [https://azure.microsoft.com/free/](https://azure.microsoft.com/free/).
2.  **Basic Networking Knowledge:** Familiarity with virtual networks, subnets, IP addressing, and basic routing concepts.

**Lab Conventions:**

*   **Azure Cloud Shell:** PowerShell interactions will primarily occur within the Azure Cloud Shell environment.
*   **Naming Conventions:** Resource names generally follow the `iatd_labs_02_*` convention, though some existing resource names might be reused for brevity.
*   **IP Address Range:** While examining existing routes, observe any address ranges configured.
*   **Location:** Consistent Azure region usage is recommended.

#### Resource Naming (Example):

*   **Resource Group:** `iatd_labs_02_rg` (or an existing resource group)
*   **Virtual Network:** `iatd_labs_02_vnet` (or an existing virtual network)

### Part 1: Setting up the Environment

We will reuse some components to keep costs and time down so some objects won't adhere to the naming standard, so make sure to update names for these resources as needed.

1.  **Sign in to the Azure Portal:** Browse to [https://portal.azure.com/](https://portal.azure.com/) and sign in.

2.  **Create a Resource Group:** (If you don't have one from the previous lab you can use)

    *   Search for "Resource groups" and select.
    *   Click **Create**.
    *   Select your subscription.
    *   Name: `iatd_labs_02_rg`
    *   Region: Select your region.
    *   Click **Review + create**, then **Create**.

3.  **Create a Virtual Network:** (If you don't have one from the previous lab you can use)

    *   Search for "Virtual networks" and select.
    *   Click **Create**.
    *   **Basics Tab:**
        *   Subscription: Your subscription.
        *   Resource group: `iatd_labs_02_rg`
        *   Name: `iatd_labs_02_vnet`
        *   Region: Your region.
    *   **IP Addresses Tab:**
        *   IPv4 address space: `172.16.10.0/24`
        *   Click **Review + create**, then **Create**.

4.  **Create Subnet:**

    *   Navigate to the `iatd_labs_02_vnet` resource.
    *   Click on **Subnets** under **Settings**.
    *   Click **Add subnet**.
    *   Subnet name: `backend`
    *   Subnet address range: `172.16.10.0/27`
    *   Click **Save**.
    *   Click **Add subnet**.
    *   Subnet name: `frontend`
    *   Subnet address range: `172.16.10.32/27`
    *   Click **Save**.

### Part 2: Examining System Routes in the Azure Portal

1.  **Navigate to the Virtual Network:**

    *   In the Azure portal, search for and select "Virtual networks".
    *   Select your virtual network (e.g., `iatd_labs_02_vnet`).

2.  **Access Effective Routes:**

    *Under 'Settings', select 'Subnets'.*
    *Select a Subnet (e.g., 'backend').*
    *Under 'Settings', select 'Effective routes'.*

3.  **Identify System Routes:**

    *Examine the listed routes. You will typically see the following:*

    *   **VNet Local:** This route enables communication within the virtual network. The destination address prefix will be the address space of your VNet (e.g., `172.16.10.0/24` if that's your VNet's address space).  The "Next hop type" will be `VNetLocal`.
    *   **Internet:** This route allows outbound communication to the internet. The destination address prefix is `0.0.0.0/0`, and the "Next hop type" is `Internet`.
    *   **None:** "Next hop type" will be `None`.  "Address prefix" destination will vary on IP ranges within the environment
    *   **On-premises:** If you have a VPN gateway or ExpressRoute connection, you'll see routes representing your on-premises network(s). The destination address prefixes will be the address ranges of your on-premises network, and the "Next hop type" will be `Virtual network gateway`. (This might not be present if you don't have a VPN gateway connected.)

4.  **Repeat for Other Subnets:**

    *   Repeat steps 2 and 3 for the other subnet(s) in your virtual network. Note that the system routes are generally the same for all subnets *unless* you've associated a User Defined Route table with a specific subnet.

### Part 3: Verifying Connectivity

Let's create two VMs to test network connectivity.

1.  **Open Cloud Shell:** In the Azure Portal, click the Cloud Shell icon. Select **PowerShell** if prompted.
2.  **Create 2 Virtual Machine in the Azure Portal:**

    *   Search for "Virtual machines" and select.
    *   Click **Create**, then **Azure virtual machine**.
    *   **Basics Tab:**
        *   Subscription: Your subscription.
        *   Resource group: `iatd_labs_02_rg`
        *   Virtual machine name: `backend-vm`
        *   Region: Your region.
        *   Image: `UbuntuLTS`
        *   Size: `Standard_B1ls`
        *   Username: `<your_username>`
        *   Password: `<your_password>`
    *   **Networking Tab:**
        *   Virtual network: `iatd_labs_02_vnet`
        *   Subnet: `backend`
        *   Public IP: Create New one called `backend-ip`
    *   Click **Review + create**, then **Create**.

    *   Search for "Virtual machines" and select.
    *   Click **Create**, then **Azure virtual machine**.
    *   **Basics Tab:**
        *   Subscription: Your subscription.
        *   Resource group: `iatd_labs_02_rg`
        *   Virtual machine name: `frontend-vm`
        *   Region: Your region.
        *   Image: `UbuntuLTS`
        *   Size: `Standard_B1ls`
        *   Username: `<your_username>`
        *   Password: `<your_password>`
    *   **Networking Tab:**
        *   Virtual network: `iatd_labs_02_vnet`
        *   Subnet: `frontend`
        *   Public IP: Create New one called `frontend-ip`
    *   Click **Review + create**, then **Create**.

    **Note down the private IP address for the following VMs as seen in Azure Portal in Virtual Machines Section as these are needed in a later task!**
    *   `backend-vm` - example private IP `172.16.10.4`
    *   `frontend-vm` - example private IP `172.16.10.36`

    **Note down the Public IP address for the following VMs as seen in Azure Portal in Virtual Machines Section as these are needed in a later task!**
    *   `backend-vm` - example public IP `20.98.76.54`
    *   `frontend-vm` - example public IP `20.123.45.67`

### Part 4: PowerShell testing traffic Flow
We can check traffic by a number of methods, however one simple methods for testing from PowerShell using Test-NetConnection. For all tasks, execute them from cloudShell within the portal to do this, click the **Cloud Shell** icon in the top navigation bar. Select PowerShell if prompted.

*note: Remember to wait for VMs to provision before moving forward or commands may not resolve addresses correctly as Azure sets them up.*

1.  **Checking internal Connectivity:** From Cloudshell check connectivity is established between your two VMs (you will need your `backend-vm` private IP you saved to do this)

    ```powershell
    Test-NetConnection -ComputerName 172.16.10.4 -port 3389
    ```

    Example of successful output:
    ```text
    ComputerName     : 172.16.10.4
    RemoteAddress    : 172.16.10.4
    RemotePort       : 3389
    InterfaceAlias   : Virtual
    SourceAddress    : 172.17.0.5
    TcpTestSucceeded : True
    ```

2.  **Checking Internet Connectivity:** From Cloudshell check internet connectivity works correctly from Cloudshell

    ```powershell
    Test-NetConnection -ComputerName www.google.com -port 80
    ```

    Example of successful output:
    ```text
    ComputerName     : www.google.com
    RemoteAddress    : 142.250.207.142
    RemotePort       : 80
    InterfaceAlias   : Virtual
    SourceAddress    : 172.17.0.5
    TcpTestSucceeded : True
    ```

3.  **Checking inter-VNET Connectivity:** This task involves VPN setup. As this might be time consuming we will skip for this Lab!

### Part 5: Cleanup

To avoid unnecessary charges, remove the resources created in this lab.
1.  Delete Resource Group in the Azure Portal

    *   **Azure Portal:** Locate the `iatd_labs_02_rg` Resource Group and Delete from the portal.
    **OR**

2.  Open Cloud Shell

    *   **Azure Cloud Shell:** Execute the following command:

```azurecli
   az group delete --name $RESOURCE_GROUP --yes --no-wait
```

### Post-Lab Questions:

1.  What are Azure system routes and why are they important?
2.  Explain the purpose of the "VNetLocal" system route.
3.  Under what circumstances might you NOT see an "On-premises" route in your system routes?
4.  How do User Defined Routes (UDRs) interact with system routes?  What happens when a UDR conflicts with a system route?
5.  What are some potential issues that could arise if system routes are not correctly configured or understood?
6.  Why the `Test-NetConnection` is suitable to test those routing configuration?
7.  In what situation it would be more appropriate to utilise `TCPing` than `Test-NetConnection`

**Congratulations!** You've explored Azure system routes and verified basic network connectivity. Understanding these fundamental routes is crucial for building more complex and customized network topologies in Azure.
