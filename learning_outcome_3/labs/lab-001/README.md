## IATD Microcredential Cloud Networking: Lab 1 - Network Security Groups (NSGs) and Application Security Groups (ASGs) Fundamentals

**Objective:** This lab guides you through the process of creating and configuring Network Security Groups (NSGs) and Application Security Groups (ASGs) in Azure. You'll learn how to implement layered security, logically group virtual machines, and troubleshoot NSG configurations.

**Estimated Time:** 60 - 75 minutes

**Prerequisites:**

1.  **Azure Subscription:** Requires an active Azure subscription. A free account is available at [https://azure.microsoft.com/free/](https://azure.microsoft.com/free/).

**Let's get started!**

### Lab Conventions

*   **Azure Cloud Shell:** All PowerShell and CLI interactions will occur within the Azure Cloud Shell environment.
*   **Naming Conventions:** Resource names follow the convention `iatd_labs_01_*`.
*   **IP Address Range:** The `172.16.0.0/16` range will be used (as per standardization).
*   **Location:** Choose a consistent Azure region.

#### Resource Naming

*   **Resource Group:** `iatd_labs_01_rg`
*   **Virtual Network:** `iatd_labs_01_vnet`
*   **Subnet-Web:** `iatd_labs_01_subnet_web`
*   **Subnet-App:** `iatd_labs_01_subnet_app`
*   **NSG-Web:** `iatd_labs_01_nsg_web`
*   **NSG-App:** `iatd_labs_01_nsg_app`
*   **VM-Web-01:** `iatd_labs_01_vm_web_01`
*   **VM-App-01:** `iatd_labs_01_vm_app_01`
*   **Public IP-Web:** `iatd_labs_01_pip_web`
*   **Public IP-App:** `iatd_labs_01_pip_app`
*   **ASG-Web:** `iatd_labs_01_asg_web`
*   **ASG-App:** `iatd_labs_01_asg_app`

### Part 1: Environment Setup - Virtual Network and Subnets (CLI)

1.  **Sign in to the Azure Portal:** Browse to [https://portal.azure.com/](https://portal.azure.com/) and sign in.

2.  **Open Cloud Shell:** In the Azure Portal, click the Cloud Shell icon. Select **Bash** if prompted.

3.  **Set Variables:**

    ```bash
    RESOURCE_GROUP="iatd_labs_01_rg"
    LOCATION="eastus" # Your Region
    VNET_NAME="iatd_labs_01_vnet"
    SUBNET_WEB_NAME="iatd_labs_01_subnet_web"
    SUBNET_APP_NAME="iatd_labs_01_subnet_app"
    VNET_PREFIX="172.16.0.0/16"
    SUBNET_WEB_PREFIX="172.16.0.0/24"
    SUBNET_APP_PREFIX="172.16.1.0/24"
    ```

4.  **Create Resource Group:**

    ```bash
    az group create --name $RESOURCE_GROUP --location $LOCATION
    ```

    **Example Output:**

    ```json
    {
      "id": "/subscriptions/<your_subscription_id>/resourceGroups/iatd_labs_01_rg",
      "location": "eastus",
      "managedBy": null,
      "name": "iatd_labs_01_rg",
      "properties": {
        "provisioningState": "Succeeded"
      },
      "tags": null,
      "type": "Microsoft.Resources/resourceGroups"
    }
    ```

5.  **Create Virtual Network with Web Subnet:**

    ```bash
    az network vnet create \
        --resource-group $RESOURCE_GROUP \
        --name $VNET_NAME \
        --address-prefixes $VNET_PREFIX \
        --subnet-name $SUBNET_WEB_NAME \
        --subnet-prefixes $SUBNET_WEB_PREFIX \
        --location $LOCATION
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
        "dhcpOptions": {
          "dnsServers": []
        },
        "enableDdosProtection": false,
        "enableVmProtection": false,
        "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_01_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_01_vnet",
        "location": "eastus",
        "name": "iatd_labs_01_vnet",
        "provisioningState": "Succeeded",
        "resourceGuid": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "subnets": [
          {
            "addressPrefix": "172.16.0.0/24",
            "delegations": [],
            "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_01_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_01_vnet/subnets/iatd_labs_01_subnet_web",
            "name": "iatd_labs_01_subnet_web",
            "networkSecurityGroup": null,
            "privateEndpointNetworkPolicies": "Disabled",
            "privateLinkServiceNetworkPolicies": "Enabled",
            "provisioningState": "Succeeded",
            "resourceNavigationLinks": [],
            "routeTable": null,
            "serviceEndpointPolicies": [],
            "serviceEndpoints": []
          }
        ],
        "type": "Microsoft.Network/virtualNetworks"
      }
    }
    ```

6.  **Add App Subnet to Virtual Network:**

    ```bash
    az network vnet subnet create \
        --resource-group $RESOURCE_GROUP \
        --vnet-name $VNET_NAME \
        --name $SUBNET_APP_NAME \
        --address-prefix $SUBNET_APP_PREFIX
    ```

    **Example Output:**

    ```json
    {
      "addressPrefix": "172.16.1.0/24",
      "delegations": [],
      "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_01_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_01_vnet/subnets/iatd_labs_01_subnet_app",
      "name": "iatd_labs_01_subnet_app",
      "networkSecurityGroup": null,
      "privateEndpointNetworkPolicies": "Disabled",
      "privateLinkServiceNetworkPolicies": "Enabled",
      "provisioningState": "Succeeded",
      "resourceNavigationLinks": [],
      "routeTable": null,
      "serviceAssociationLinks": [],
      "serviceEndpointPolicies": [],
      "serviceEndpoints": []
    }
    ```

### Part 2: Creating and Configuring Network Security Groups (NSGs) (Portal & CLI)

1.  **Create NSG for Web Subnet (Portal):**
    *   Search for "Network security groups" and select.
    *   Click **Create**.
    *   Subscription: Your subscription.
    *   Resource group: `iatd_labs_01_rg`
    *   Name: `iatd_labs_01_nsg_web`
    *   Region: Your region.
    *   Click **Review + create**, then **Create**.

2.  **Create NSG for App Subnet (CLI):**

    ```bash
    NSG_APP_NAME="iatd_labs_01_nsg_app"

    az network nsg create \
        --resource-group $RESOURCE_GROUP \
        --name $NSG_APP_NAME \
        --location $LOCATION
    ```

    **Example Output:**

    ```json
    {
      "defaultSecurityRules": [
        {
          "access": "Allow",
          "description": "Allow inbound traffic from all VMs in VNET",
          "destinationAddressPrefix": "VirtualNetwork",
          "destinationPortRange": "*",
          "direction": "Inbound",
          "name": "AllowVnetInBound",
          "priority": 65000,
          "protocol": "*",
          "provisioningState": "Succeeded",
          "sourceAddressPrefix": "VirtualNetwork",
          "sourcePortRange": "*"
        },
        {
          "access": "Allow",
          "description": "Allow inbound traffic from azure load balancer",
          "destinationAddressPrefix": "*",
          "destinationPortRange": "*",
          "direction": "Inbound",
          "name": "AllowAzureLoadBalancerInBound",
          "priority": 65001,
          "protocol": "*",
          "provisioningState": "Succeeded",
          "sourceAddressPrefix": "AzureLoadBalancer",
          "sourcePortRange": "*"
        },
        {
          "access": "Deny",
          "description": "Deny all inbound traffic",
          "destinationAddressPrefix": "*",
          "destinationPortRange": "*",
          "direction": "Inbound",
          "name": "DenyAllInBound",
          "priority": 65500,
          "protocol": "*",
          "provisioningState": "Succeeded",
          "sourceAddressPrefix": "*",
          "sourcePortRange": "*"
        }
      ],
      "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_01_rg/providers/Microsoft.Network/networkSecurityGroups/iatd_labs_01_nsg_app",
      "location": "eastus",
      "name": "iatd_labs_01_nsg_app",
      "provisioningState": "Succeeded",
      "resourceGuid": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "securityRules": [],
      "type": "Microsoft.Network/networkSecurityGroups"
    }
    ```

3.  **Associate NSGs with Subnets (Portal):**
    *   Navigate to the `iatd_labs_01_nsg_web` NSG in the Azure Portal.
    *   Select **Subnets** under **Settings**.
    *   Click **Associate**.
    *   Virtual network: `iatd_labs_01_vnet`
    *   Subnet: `iatd_labs_01_subnet_web`
    *   Click **OK**.

4.  **Associate `iatd_labs_01_nsg_app` with `iatd_labs_01_subnet_app` (CLI):**

    ```bash
    az network vnet subnet update \
    --resource-group $RESOURCE_GROUP \
    --vnet-name $VNET_NAME \
    --name $SUBNET_APP_NAME \
    --network-security-group $NSG_APP_NAME
    ```

    **Example Output:**

    ```json
    {
      "addressPrefix": "172.16.1.0/24",
      "delegations": [],
      "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_01_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_01_vnet/subnets/iatd_labs_01_subnet_app",
      "name": "iatd_labs_01_subnet_app",
      "networkSecurityGroup": {
        "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_01_rg/providers/Microsoft.Network/networkSecurityGroups/iatd_labs_01_nsg_app",
        "resourceGroup": "iatd_labs_01_rg"
      },
      "privateEndpointNetworkPolicies": "Disabled",
      "privateLinkServiceNetworkPolicies": "Enabled",
      "provisioningState": "Succeeded",
      "resourceNavigationLinks": [],
      "routeTable": null,
      "serviceEndpointPolicies": [],
      "serviceEndpoints": []
    }
    ```

5.  **Configure Inbound Security Rules for `iatd_labs_01_nsg_web` (Portal):**
    *   Navigate to the `iatd_labs_01_nsg_web` NSG in the Azure Portal.
    *   Select **Inbound security rules** under **Settings**.
    *   Click **Add**.
        *   Source: `Any`
        *   Source port range: `*`
        *   Destination: `Any`
        *   Service: `HTTP`
        *   Action: `Allow`
        *   Priority: `100`
        *   Name: `AllowHTTP`
        *   Click **Add**.
    *   Add another rule to allow SSH access:
        *   Source: `Any`
        *   Source port range: `*`
        *   Destination: `Any`
        *   Destination port range: `22`
        *   Protocol: `TCP`
        *   Action: `Allow`
        *   Priority: `110`
        *   Name: `AllowSSH`
        *   Click **Add**.

6.  **Configure Inbound Security Rules for `iatd_labs_01_nsg_app` (CLI):**

    ```bash
    NSG_APP_NAME="iatd_labs_01_nsg_app"
    az network nsg rule create \
        --resource-group $RESOURCE_GROUP \
        --nsg-name $NSG_APP_NAME \
        --name AllowAppTraffic \
        --priority 100 \
        --source-address-prefixes VirtualNetwork \
        --destination-port-ranges 8080 \
        --access Allow \
        --protocol Tcp \
        --direction Inbound

    az network nsg rule create \
        --resource-group $RESOURCE_GROUP \
        --nsg-name $NSG_APP_NAME \
        --name AllowSSH \
        --priority 110 \
        --source-address-prefixes '*' \
        --destination-port-ranges 22 \
        --access Allow \
        --protocol Tcp \
        --direction Inbound
    ```

    **Example Output (for the first rule):**

    ```json
    {
      "access": "Allow",
      "description": null,
      "destinationAddressPrefix": "*",
      "destinationAddressPrefixes": [],
      "destinationApplicationSecurityGroups": null,
      "destinationPortRange": null,
      "destinationPortRanges": [
        "8080"
      ],
      "direction": "Inbound",
      "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_01_rg/providers/Microsoft.Network/networkSecurityGroups/iatd_labs_01_nsg_app/securityRules/AllowAppTraffic",
      "name": "AllowAppTraffic",
      "priority": 100,
      "protocol": "Tcp",
      "provisioningState": "Succeeded",
      "resourceGroup": "iatd_labs_01_rg",
      "sourceAddressPrefix": null,
      "sourceAddressPrefixes": [
        "VirtualNetwork"
      ],
      "sourceApplicationSecurityGroups": null,
      "sourcePortRange": "*",
      "sourcePortRanges": [],
      "type": "Microsoft.Network/networkSecurityGroups/securityRules"
    }
    ```

### Part 3: Creating Virtual Machines and Testing NSG Rules (Portal)

1.  **Create Web Virtual Machine (`iatd_labs_01_vm_web_01`):**
    *   Search for "Virtual machines" and select.
    *   Click **Create** -> **Azure virtual machine**.
    *   **Basics Tab:**
        *   Subscription: Your subscription.
        *   Resource group: `iatd_labs_01_rg`
        *   Virtual machine name: `iatd_labs_01_vm_web_01`
        *   Region: Your region.
        *   Image: `Ubuntu Server 22.04 LTS`
        *   Size: Choose a size (e.g., Standard_B1ls).
        *   Username: `azureuser`
        *   Authentication type: `Password`
        *   Password: Set a password.
    *   **Networking Tab:**
        *   Virtual network: `iatd_labs_01_vnet`
        *   Subnet: `iatd_labs_01_subnet_web`
        *   Public IP: Create new
            *   Name: `iatd_labs_01_pip_web`
            *   SKU: Basic
        *   NIC network security group: `None`
        *   Public inbound ports: Allow selected ports
            *   Select inbound ports: HTTP (80), SSH (22)
    *   **Management Tab:**
        *   Disable boot diagnostics
    *   Click **Review + create**, then **Create**.

2.  **Create App Virtual Machine (`iatd_labs_01_vm_app_01`):**
    *   Repeat the process above, but with the following changes:
        *   Virtual machine name: `iatd_labs_01_vm_app_01`
        *   Subnet: `iatd_labs_01_subnet_app`
        *   Public IP: Create new
            *   Name: `iatd_labs_01_pip_app`
            *   SKU: Basic
        *   NIC network security group: `None`
        *   Public inbound ports: Allow selected ports
            *   Select inbound ports: SSH (22)
    *   **Management Tab:**
        *   Disable boot diagnostics

3.  **Testing:**
    *   Once VMs are deployed, get the Public IP address of `iatd_labs_01_vm_web_01`.
    *   Open a web browser and navigate to the VM's Public IP address. You should be able to access the default Ubuntu web page. This verifies the HTTP rule is working.

        **Expected Result:** You should see the default Apache2 Ubuntu page with the message "It works!" or similar. If you don't see this page, you may need to install and start Apache2 on the VM:
        
        ```bash
        sudo apt update
        sudo apt install apache2 -y
        sudo systemctl start apache2
        ```

    *   Attempt to SSH into both VMs. This verifies the SSH rules are working.

        ```bash
        ssh azureuser@<web-vm-public-ip>
        ```

        **Expected Result:** You should be able to successfully connect to the VM via SSH. You'll be prompted for the password you set during VM creation.

    *   From `iatd_labs_01_vm_web_01` attempt to connect to port 8080 of `iatd_labs_01_vm_app_01`. This connection should fail, this verifies the NSG configuration.

        ```bash
        # First, get the private IP of the app VM
        APP_VM_PRIVATE_IP=$(az vm list-ip-addresses -g iatd_labs_01_rg -n iatd_labs_01_vm_app_01 --query "[0].virtualMachine.network.privateIpAddresses[0]" -o tsv)
        
        # Then, from the web VM, try to connect to the app VM on port 8080
        nc -zv $APP_VM_PRIVATE_IP 8080
        ```

        **Expected Result:** The connection should fail because we haven't yet configured the NSG to allow traffic from the web VM to the app VM on port 8080.

### Part 4: Implementing Application Security Groups (ASGs) (CLI)

1.  **Create ASG for Web Tier:**

    ```bash
    ASG_WEB_NAME="iatd_labs_01_asg_web"
    az network asg create \
        --resource-group $RESOURCE_GROUP \
        --name $ASG_WEB_NAME \
        --location $LOCATION
    ```

    **Example Output:**

    ```json
    {
      "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_01_rg/providers/Microsoft.Network/applicationSecurityGroups/iatd_labs_01_asg_web",
      "location": "eastus",
      "name": "iatd_labs_01_asg_web",
      "provisioningState": "Succeeded",
      "resourceGroup": "iatd_labs_01_rg",
      "resourceGuid": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "type": "Microsoft.Network/applicationSecurityGroups"
    }
    ```

2.  **Create ASG for App Tier:**

    ```bash
    ASG_APP_NAME="iatd_labs_01_asg_app"
    az network asg create \
        --resource-group $RESOURCE_GROUP \
        --name $ASG_APP_NAME \
        --location $LOCATION
    ```

    **Example Output:**

    ```json
    {
      "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_01_rg/providers/Microsoft.Network/applicationSecurityGroups/iatd_labs_01_asg_app",
      "location": "eastus",
      "name": "iatd_labs_01_asg_app",
      "provisioningState": "Succeeded",
      "resourceGroup": "iatd_labs_01_rg",
      "resourceGuid": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "type": "Microsoft.Network/applicationSecurityGroups"
    }
    ```

3.  **Get Network Interface IDs (CLI):**

    First, you need to get the resource IDs of the network interfaces associated with your VMs. Run these commands and note the `id` values:

    ```bash
    WEB_VM_NIC=$(az network nic list --resource-group $RESOURCE_GROUP --query "[?contains(name,'iatd_labs_01_vm_web_01')].id" --output tsv)
    echo $WEB_VM_NIC

    APP_VM_NIC=$(az network nic list --resource-group $RESOURCE_GROUP --query "[?contains(name,'iatd_labs_01_vm_app_01')].id" --output tsv)
    echo $APP_VM_NIC
    ```

    **Example Output (for the first command):**

    ```
    /subscriptions/<subscription_id>/resourceGroups/iatd_labs_01_rg/providers/Microsoft.Network/networkInterfaces/iatd_labs_01_vm_web_01VMNic
    ```

4.  **Associate NICs with ASGs (CLI):**

    ```bash
    az network nic ip-config update \
        --resource-group $RESOURCE_GROUP \
        --nic-name $(az network nic show --ids $WEB_VM_NIC --query name -o tsv) \
        --name ipconfig1 \
        --application-security-groups $ASG_WEB_NAME

    az network nic ip-config update \
        --resource-group $RESOURCE_GROUP \
        --nic-name $(az network nic show --ids $APP_VM_NIC --query name -o tsv) \
        --name ipconfig1 \
        --application-security-groups $ASG_APP_NAME
    ```

    **Example Output (for the first command):**

    ```json
    {
      "applicationSecurityGroups": [
        {
          "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_01_rg/providers/Microsoft.Network/applicationSecurityGroups/iatd_labs_01_asg_web",
          "resourceGroup": "iatd_labs_01_rg"
        }
      ],
      "etag": "W/\"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx\"",
      "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_01_rg/providers/Microsoft.Network/networkInterfaces/iatd_labs_01_vm_web_01VMNic/ipConfigurations/ipconfig1",
      "name": "ipconfig1",
      "primary": true,
      "privateIpAddress": "172.16.0.4",
      "privateIpAddressVersion": "IPv4",
      "privateIpAllocationMethod": "Dynamic",
      "provisioningState": "Succeeded",
      "publicIpAddress": {
        "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_01_rg/providers/Microsoft.Network/publicIPAddresses/iatd_labs_01_pip_web",
        "resourceGroup": "iatd_labs_01_rg"
      },
      "resourceGroup": "iatd_labs_01_rg",
      "subnet": {
        "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_01_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_01_vnet/subnets/iatd_labs_01_subnet_web",
        "resourceGroup": "iatd_labs_01_rg"
      },
      "type": "Microsoft.Network/networkInterfaces/ipConfigurations"
    }
    ```

5.  **Update NSG Rules to Use ASGs (Portal):**
    *   Navigate to the `iatd_labs_01_nsg_web` NSG in the Azure Portal.
    *   Select **Inbound security rules** under **Settings**.
    *   Select the `AllowHTTP` rule.
    *   Change Source to `ApplicationSecurityGroup`.
    *   Application security group: `iatd_labs_01_asg_web`
    *   Click **Save**.
    *   Navigate to the `iatd_labs_01_nsg_app` NSG in the Azure Portal.
    *   Select **Inbound security rules** under **Settings**.
    *   Select the `AllowAppTraffic` rule.
    *   Change Source to `ApplicationSecurityGroup`.
    *   Application security group: `iatd_labs_01_asg_web`
    *   Click **Save**.

6.  **Testing ASGs:**
    *   Testing will remain the same as it was in the NSG testing, verify the ASG rules have not changed the ability to reach resources.
    *   The HTTP traffic should still be accessible from the internet to the `iatd_labs_01_vm_web_01`
    *   The connection from `iatd_labs_01_vm_web_01` to `iatd_labs_01_vm_app_01` on port 8080 should still be possible.

    **Verification Steps:**

    1. **Verify HTTP Access:**
       * Open a web browser and navigate to the public IP of `iatd_labs_01_vm_web_01`
       * **Expected Result:** You should still be able to access the default web page

    2. **Verify App Tier Access:**
       * SSH into the web VM:
         ```bash
         ssh azureuser@<web-vm-public-ip>
         ```
       * From the web VM, try to connect to the app VM on port 8080:
         ```bash
         # Get the private IP of the app VM
         APP_VM_PRIVATE_IP=$(az vm list-ip-addresses -g iatd_labs_01_rg -n iatd_labs_01_vm_app_01 --query "[0].virtualMachine.network.privateIpAddresses[0]" -o tsv)
         
         # Test the connection
         nc -zv $APP_VM_PRIVATE_IP 8080
         ```
       * **Expected Result:** The connection should now succeed because we've configured the NSG to allow traffic from the web ASG to the app VM on port 8080

### Part 5: Troubleshooting NSG Configurations using Effective Network Diagnostic Tools and Flow Verification (Portal)

1.  **Using Azure Network Watcher to Test IP Flow:**
    *   Search for "Network Watcher" in the Azure Portal.
    *   Select **IP flow verify** under **Network diagnostic tools**.
    *   **Configure the test:**
        *   Direction: Inbound
        *   Protocol: TCP
        *   Local IP address: Select the IP configuration for `iatd_labs_01_vm_web_01`.
        *   Local port: 80
        *   Remote IP address: Your local machine's public IP address (you can find this by searching "what is my IP" on Google).
        *   Remote port: `8080` (or any open port on your machine).
    *   Click **Check**.
    *   The result should show that the traffic is allowed by the `AllowHTTP` rule in the `iatd_labs_01_nsg_web` NSG.
    *   Change the Local port to `3389` and click Check.
    *   The result should show that the traffic is denied, because there is no rule allowing that port.

### Post-Lab: Cleanup (CLI)

To avoid incurring unnecessary costs, it's essential to clean up the resources created during this lab.

1.  **Delete Resource Group:**

    ```bash
    az group delete --name $RESOURCE_GROUP --yes
    ```

    **Expected Output:**
    The command will not produce immediate output but will start the deletion process. You can verify the deletion is in progress by checking the resource group list:

    ```bash
    az group list --query "[?name=='$RESOURCE_GROUP']" -o table
    ```

    The resource group should eventually disappear from the list, indicating successful deletion. This process may take several minutes to complete.

2.  **Verify Cleanup:**

    To ensure all resources have been properly cleaned up, verify that the resource group no longer exists:

    ```bash
    az group show --name $RESOURCE_GROUP
    ```

    **Expected Output:**
    ```
    Resource group 'iatd_labs_01_rg' could not be found.
    ```

**Congratulations!** You have now successfully created and configured Network Security Groups (NSGs) and Application Security Groups (ASGs) in Azure, implemented layered security, grouped virtual machines logically, and troubleshooted NSG configurations. You've strengthened your understanding of Azure network security!
