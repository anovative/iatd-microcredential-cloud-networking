
## IATD Microcredential Cloud Networking: Lab 3 - Configuring Different Next Hop Types

**Objective:** Configure User Defined Routes (UDRs) with different next hop types to control traffic flow within and out of an Azure Virtual Network.

**Scenario:** You have a virtual network with multiple subnets. You want to:

*   Route traffic from one subnet to another through a virtual appliance (simulating a firewall or intrusion detection system).
*   Direct traffic from a subnet to a peered virtual network.
*   Send traffic from a subnet directly to the internet, bypassing a default route.

**Outcomes:**

*   Configuration of UDRs with Virtual Appliance, Virtual Network Peering, and Internet next hop types.
*   Verification of traffic flow based on the configured next hop types.
*   Understanding of how different next hop types impact network traffic paths.
*   Use network watching tools to verify that all configured routes are behaving as planned.

**Estimated Time:** 75-90 minutes

**Prerequisites:**

1.  **Azure Subscription:** Active Azure subscription ([https://azure.microsoft.com/free/](https://azure.microsoft.com/free/)).
2.  **Networking Fundamentals:** VNETs, subnets, IP addressing, routing, Network Virtual Appliances, VNET Peering.
3.  **Familiarity with previous labs:** Assumed knowledge from previous lab exercises.

**Lab Conventions:**

*   **Azure Cloud Shell:** PowerShell and CLI will be used in Cloud Shell.
*   **Naming:** `iatd_labs_03_*`.
*   **IP Addresses:** Use `172.16.x.x` range unless adapting existing infrastructure.
*   **Location:** Use the same Azure region for all resources.

#### Resource Naming:

*   **Resource Group:** `iatd_labs_03_rg`
*   **Virtual Network (Main):** `iatd_labs_03_vnet_main`
*   **Subnet (Protected):** `protected-subnet`
*   **Subnet (NVA):** `nva-subnet`
*   **Virtual Network (Peered):** `iatd_labs_03_vnet_peered`
*   **Subnet (Peered):** `peered-subnet`
*   **NVA VM:** `iatd_labs_03_nva`
*   **NVA NIC:** `iatd_labs_03_nva_nic`
*   **NVA Public IP:** `iatd_labs_03_nva_pip`
*   **VM (Protected):** `iatd_labs_03_vm_protected`
*   **VM (Peered):** `iatd_labs_03_vm_peered`
*   **Route Table (Protected):** `iatd_labs_03_rt_protected`
*   **Route (NVA):** `route_to_nva`
*   **Route (Peered):** `route_to_peered`
*   **Route (Internet):** `route_to_internet`

### Part 1: Infrastructure Setup

1.  **Sign in to Azure Portal:** [https://portal.azure.com/](https://portal.azure.com/).

2.  **Create Resource Group:**

    *   If you have existing one use that otherwise Create
    *   Search for "Resource groups" and select.
    *   Click **Create**.
    *   Subscription: Your subscription.
    *   Resource group: `iatd_labs_03_rg`
    *   Region: Your preferred region.
    *   Click **Review + create**, then **Create**.

3.  **Open Cloud Shell:** In the Azure Portal, click the Cloud Shell icon. Select **Bash** if prompted.

4.  **Create Virtual Networks and Subnets:**

    ```bash
    RESOURCE_GROUP="iatd_labs_03_rg"
    LOCATION="australiaeast"  # Your region

    # Main VNet
    VNET_MAIN="iatd_labs_03_vnet_main"
    MAIN_ADDRESS_PREFIX="172.16.0.0/16"
    PROTECTED_SUBNET="protected-subnet"
    PROTECTED_PREFIX="172.16.1.0/24"
    NVA_SUBNET="nva-subnet"
    NVA_PREFIX="172.16.2.0/24"

    # Peered VNet
    VNET_PEERED="iatd_labs_03_vnet_peered"
    PEERED_ADDRESS_PREFIX="172.16.100.0/24"
    PEERED_SUBNET="peered-subnet"
    PEERED_PREFIX="172.16.100.0/24"

    # Create Main VNet and subnets
    az network vnet create -g $RESOURCE_GROUP -n $VNET_MAIN --address-prefixes $MAIN_ADDRESS_PREFIX --subnet-name $PROTECTED_SUBNET --subnet-prefixes $PROTECTED_PREFIX
    az network vnet subnet create -g $RESOURCE_GROUP --vnet-name $VNET_MAIN -n $NVA_SUBNET --address-prefixes $NVA_PREFIX

    # Create Peered VNet and subnet
    az network vnet create -g $RESOURCE_GROUP -n $VNET_PEERED --address-prefixes $PEERED_ADDRESS_PREFIX --subnet-name $PEERED_SUBNET --subnet-prefixes $PEERED_PREFIX
    ```

5.  **Create NVA Virtual Machine:**  Use CLI in Cloud Shell.

    ```bash
    NVA_NAME="iatd_labs_03_nva"
    NVA_NIC="iatd_labs_03_nva_nic"
    NVA_PIP="iatd_labs_03_nva_pip"
    NVA_SUBNET_ID=$(az network vnet subnet show -g $RESOURCE_GROUP --vnet-name $VNET_MAIN -n $NVA_SUBNET --query id -o tsv)

    # Create Public IP for NVA
    az network public-ip create -g $RESOURCE_GROUP -n $NVA_PIP --allocation-method Static --sku Standard

    # Create NIC for NVA
    NVA_PUBLIC_IP_ID=$(az network public-ip show -g $RESOURCE_GROUP -n $NVA_PIP --query id -o tsv)
    az network nic create -g $RESOURCE_GROUP -n $NVA_NIC --subnet $NVA_SUBNET_ID --public-ip-address $NVA_PUBLIC_IP_ID

    # Create NVA VM
    az vm create -g $RESOURCE_GROUP -n $NVA_NAME --nics $NVA_NIC --image UbuntuLTS --size Standard_B1ls --admin-username <your_username> --admin-password <your_password>
    ```

    *Replace `<your_username>` and `<your_password>` with secure credentials.*
6.  **Enable IP Forwarding on NVA:**

    ```bash
    az network nic update -g $RESOURCE_GROUP -n $NVA_NIC --enable-ip-forwarding true
    ```

7.  **Create Test Virtual Machines:** Create `iatd_labs_03_vm_protected` in `protected-subnet` and `iatd_labs_03_vm_peered` in `peered-subnet`. Using what you learned previously do this step. Note down their Public IP and Private IPs respectively as it is needed in future Tasks! You can name PIP as `<vm_name>-pip` for their corresponding Virtual Machines
    * **`iatd_labs_03_vm_protected`**: in `protected-subnet` example private IP is `172.16.1.4` and the corresponding public IP such as `20.188.230.84`
    * **`iatd_labs_03_vm_peered`**: in `peered-subnet` example private IP is `172.16.100.4` and the corresponding public IP such as `20.190.232.91`

8.  **Configure the NVAs internal routing!:** Follow similar procedure than lab 1 Part 8 Section 2, use their noted IPs, to set their internal routing
    ```bash
        sudo su
        echo "net.ipv4.ip_forward=1" > /etc/sysctl.d/forwarding.conf
        sysctl -p /etc/sysctl.d/forwarding.conf
        exit
    ```

### Part 2: Virtual Appliance Next Hop

1.  **Create Route Table:**

    *   In the Azure portal, search for "Route tables" and select.
    *   Click **Create**.
        *   Subscription: Your subscription.
        *   Resource group: `iatd_labs_03_rg`
        *   Name: `iatd_labs_03_rt_protected`
        *   Region: Your region.
        *   Click **Review + create**, then **Create**.

2.  **Create Route to NVA:** Navigate to newly created `iatd_labs_03_rt_protected`.

    *   Click **Routes** then **Add**.
        *   Name: `route_to_nva`
        *   Address prefix destination: `172.16.100.0/24` (Peered VNet address space)
        *   Next hop type: Virtual appliance
3.  **Open Cloud Shell** Retrieve Internal NVA address and assign, by executing this from Bash in Cloud Shell and taking the relevant IP
        ```bash
        NVA_NIC="iatd_labs_03_nva_nic"
        RESOURCE_GROUP="iatd_labs_03_rg"
        NVA_PRIVATE_IP=$(az network nic ip-config list \
          --resource-group $RESOURCE_GROUP \
          --nic-name $NVA_NIC \
          --query "[0].properties.privateIpAddress" \
          --output tsv)
        echo $NVA_PRIVATE_IP
        ```
    *   Click **Routes** then the Newly created **Route** and use IP.
        * Next hop address: Copy from last execution from **Cloud Shell**, examaple result IP will be similar too `172.16.2.4`

        *   Click **Add**.

4.  **Associate Route Table:**

    *   Go to Virtual Network (`iatd_labs_03_vnet_main`) and select **Subnets**.
    *   Select `protected-subnet`.
        *   Route table:  `iatd_labs_03_rt_protected`
    *   Click **Save**.

### Part 3: Virtual Network Peering Next Hop

1.  **Create VNet Peering:** *(From Main VNet)*

    *   Go to Virtual Network (`iatd_labs_03_vnet_main`).
    *   Under **Settings**, select **Peerings**.
    *   Click **Add**.
        *   Peering link name: `main_to_peered`
        *   This virtual network: Select  `iatd_labs_03_vnet_main`
        *   Remote virtual network: select  `iatd_labs_03_vnet_peered`
        *   Peering link name: `peered_to_main`
        *   Remote virtual network: Select `iatd_labs_03_vnet_main`

    *   Accept defaults (typically sufficient) and click **Add**.

### Part 4: Internet Next Hop

1.  **Create Route to Internet:**

    *   Navigate to your `iatd_labs_03_rt_protected` Route Table.
    *   Click **Routes** then **Add**.
        *   Name: `route_to_internet`
        *   Address prefix destination: `0.0.0.0/0`
        *   Next hop type: Internet
        *   Click **Add**.

*Note: In many scenarios, a default route to the internet already exists. This step demonstrates explicitly creating one.*

### Part 5: Verification and Testing

1.  **Connect to VMs:** SSH into the `iatd_labs_03_vm_protected` VM to test outbound traffic from the `protected-subnet`. SSH into `iatd_labs_03_vm_peered`. You can access both as noted using credentials in **Part 1 step 7!** In your Cloud Shell open separate connections by selecting the `+` and then `Open new session` to the test VMs from different Cloud Shells
2.  Testing inter VNET routing: (test using each connection, will only be tested and flow one way

```bash
ping 172.16.100.4
```

1.  **Test traffic to Internet from iatd\_labs\_03\_vm\_protected!:** ping any Internet Resource example bellow and confirm it works from same testing VM using the instructions provided. Ping has issues you can leverage traceroute instead. `Traceroute 8.8.8.8`

```bash
ping www.google.com
```

1.  Test internal only! **test ONLY `ping 172.16.100.4` DOESNT work FROM Cloud Shell. Since routing ONLY goes from Main -- Peered network! not Bi-Directional (However this isn't directly doable)** However one thing we could achieve instead is to do testing from your end machine **ONLY IF you setup a VPN!** For sake of completeness and that a VM doesnt live inside VNET but cloud shell lives inside a network we need another hoster and it might not exists! For more information read section *Interpreting Results and further considerations!* instead on potential traffic.

### Interpreting Results and further considerations!

*After the following considerations, one potential alternative traffic might be to host a simple webserver within Peered network and Main subnet to showcase the full connectivity. **Please review if you have the Time!***

*Traffic from protected subnet is routed to the peer network using an NVA in main!*
*Internet routes are flowing outbound only in Main.*

*This configuration may seem counterintuitive. The default outbound doesn't flow because its not connected inbound*

1.  **Network Watcher:** *(Optional but highly recommended)*
    *   Explore Azure Network Watcher's connection troubleshoot and IP flow verify tools to visually confirm traffic flow. You will have a further lab focusing specifically with Network Watcher tool!

### Part 6: Cleanup

```bash
az group delete -g $RESOURCE_GROUP --yes --no-wait
```

**Post-Lab Questions:**

1.  Describe the key differences between the `VirtualAppliance`, `VirtualNetworkPeering`, and `Internet` next hop types.
2.  Why was it necessary to enable IP forwarding on the NVA virtual machine?
3.  Explain the routing logic that allowed traffic to flow from the protected subnet to the peered virtual network in this lab.
4.  In what real-world scenarios would you use each of these next hop types?
5.  What are some limitations or security considerations when using User Defined Routes, especially with the `Internet` next hop?
6.  If peered route wasn't setup how would your configuration flow
7.  From an Express Route Perspective when and why this would affect and impact this overall deployment?

**Congratulations!** You've mastered configuring User Defined Routes with different next hop types. You're now able to precisely control network traffic flow in Azure!
