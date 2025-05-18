## IATD Microcredential Cloud Networking: Lab 5 - Monitoring and Logging in Azure - Fundamentals

**Objective:** This lab introduces you to the fundamental concepts of monitoring and logging in Azure. You will learn about different monitoring types, log analysis techniques, performance baselines, alert thresholds, and core Azure monitoring services.

**Estimated Time:** 75 - 90 minutes

**Prerequisites:**

1.  **Azure Subscription:** Requires an active Azure subscription. A free account is available at [https://azure.microsoft.com/free/](https://azure.microsoft.com/free/).
2.  **Basic knowledge of Azure Networking:** Familiarity with Virtual Networks, Network Security Groups, and Virtual Machines.

**Let's get started!**

### Lab Conventions

*   **Azure Cloud Shell:** All PowerShell and CLI interactions will occur within the Azure Cloud Shell environment.
*   **Azure Portal:** The Azure Portal will be used for visualization and configuration tasks.
*   **Naming Conventions:** Resource names follow the convention `iatd_labs_05_*`.
*   **IP Address Range:** The `172.16.0.0/16` range will be used (as per standardization).
*   **Location:** Choose a consistent Azure region.

#### Resource Naming

*   **Resource Group:** `iatd_labs_05_rg`
*   **Virtual Network:** `iatd_labs_05_vnet`
*   **Subnet:** `iatd_labs_05_subnet`
*   **VM-Target:** `iatd_labs_05_vm_target`
*   **Public IP:** `iatd_labs_05_pip`
*   **Network Security Group:** `iatd_labs_05_nsg`
*   **Log Analytics Workspace:** `iatd_labs_05_law`
*   **Application Insights:** `iatd_labs_05_appi`

### Part 1: Setting up a Monitored Environment

1.  **Create a Resource Group:**

    ```bash
    RESOURCE_GROUP="iatd_labs_05_rg"
    LOCATION="eastus" # Your Region

    az group create --name $RESOURCE_GROUP --location $LOCATION
    ```

    **Expected Output:**
    ```json
    {
      "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/iatd_labs_05_rg",
      "location": "eastus",
      "managedBy": null,
      "name": "iatd_labs_05_rg",
      "properties": {
        "provisioningState": "Succeeded"
      },
      "tags": null,
      "type": "Microsoft.Resources/resourceGroups"
    }
    ```

2.  **Create a Virtual Network and Subnet:**

    ```bash
    VNET_NAME="iatd_labs_05_vnet"
    SUBNET_NAME="iatd_labs_05_subnet"
    ADDRESS_PREFIX="172.16.0.0/16"
    SUBNET_PREFIX="172.16.0.0/24"

    az network vnet create \
        --resource-group $RESOURCE_GROUP \
        --name $VNET_NAME \
        --address-prefixes $ADDRESS_PREFIX \
        --subnet-name $SUBNET_NAME \
        --subnet-prefixes $SUBNET_PREFIX \
        --location $LOCATION
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
        "enableDdosProtection": false,
        "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/iatd_labs_05_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_05_vnet",
        "location": "eastus",
        "name": "iatd_labs_05_vnet",
        "provisioningState": "Succeeded",
        "resourceGroup": "iatd_labs_05_rg",
        "subnets": [
          {
            "addressPrefix": "172.16.0.0/24",
            "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/iatd_labs_05_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_05_vnet/subnets/iatd_labs_05_subnet",
            "name": "iatd_labs_05_subnet",
            "provisioningState": "Succeeded",
            "resourceGroup": "iatd_labs_05_rg"
          }
        ],
        "type": "Microsoft.Network/virtualNetworks"
      }
    }
    ```

3.  **Create a Public IP Address:**

    ```bash
    PUBLIC_IP_NAME="iatd_labs_05_pip"

    az network public-ip create \
        --resource-group $RESOURCE_GROUP \
        --name $PUBLIC_IP_NAME \
        --allocation-method Static \
        --sku Standard \
        --location $LOCATION
    ```

    **Expected Output:**
    ```json
    {
      "publicIp": {
        "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/iatd_labs_05_rg/providers/Microsoft.Network/publicIPAddresses/iatd_labs_05_pip",
        "location": "eastus",
        "name": "iatd_labs_05_pip",
        "provisioningState": "Succeeded",
        "publicIPAddressVersion": "IPv4",
        "publicIPAllocationMethod": "Static",
        "resourceGroup": "iatd_labs_05_rg",
        "sku": {
          "name": "Standard"
        },
        "type": "Microsoft.Network/publicIPAddresses"
      }
    }
    ```

4.  **Create a Network Security Group:**

    ```bash
    NSG_NAME="iatd_labs_05_nsg"

    az network nsg create \
        --resource-group $RESOURCE_GROUP \
        --name $NSG_NAME \
        --location $LOCATION
    ```

    **Expected Output:**
    ```json
    {
      "NewNSG": {
        "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/iatd_labs_05_rg/providers/Microsoft.Network/networkSecurityGroups/iatd_labs_05_nsg",
        "location": "eastus",
        "name": "iatd_labs_05_nsg",
        "provisioningState": "Succeeded",
        "resourceGroup": "iatd_labs_05_rg",
        "securityRules": [],
        "type": "Microsoft.Network/networkSecurityGroups"
      }
    }
    ```

5.  **Create an NSG Rule to Allow HTTP Traffic:**

    ```bash
    az network nsg rule create \
        --resource-group $RESOURCE_GROUP \
        --nsg-name $NSG_NAME \
        --name AllowHTTP \
        --priority 100 \
        --source-address-prefixes Internet \
        --destination-port-ranges 80 \
        --access Allow \
        --protocol Tcp \
        --direction Inbound
    ```

    **Expected Output:**
    ```json
    {
      "access": "Allow",
      "description": null,
      "destinationAddressPrefix": "*",
      "destinationAddressPrefixes": [],
      "destinationPortRange": null,
      "destinationPortRanges": [
        "80"
      ],
      "direction": "Inbound",
      "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/iatd_labs_05_rg/providers/Microsoft.Network/networkSecurityGroups/iatd_labs_05_nsg/securityRules/AllowHTTP",
      "name": "AllowHTTP",
      "priority": 100,
      "protocol": "Tcp",
      "provisioningState": "Succeeded",
      "resourceGroup": "iatd_labs_05_rg",
      "sourceAddressPrefix": null,
      "sourceAddressPrefixes": [
        "Internet"
      ],
      "sourcePortRange": "*",
      "sourcePortRanges": []
    }
    ```

6.  **Associate the NSG with the Subnet:**

    ```bash
    az network vnet subnet update \
        --resource-group $RESOURCE_GROUP \
        --vnet-name $VNET_NAME \
        --name $SUBNET_NAME \
        --network-security-group $NSG_NAME
    ```

    **Expected Output:**
    ```json
    {
      "addressPrefix": "172.16.0.0/24",
      "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/iatd_labs_05_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_05_vnet/subnets/iatd_labs_05_subnet",
      "name": "iatd_labs_05_subnet",
      "networkSecurityGroup": {
        "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/iatd_labs_05_rg/providers/Microsoft.Network/networkSecurityGroups/iatd_labs_05_nsg",
        "resourceGroup": "iatd_labs_05_rg"
      },
      "provisioningState": "Succeeded",
      "resourceGroup": "iatd_labs_05_rg"
    }
    ```

7.  **Create a Virtual Machine (Portal):**
    *   Search for "Virtual machines" and select.
    *   Click **Create** -> **Azure virtual machine**.
        *   **Basics Tab:**
            *   Subscription: Your subscription.
            *   Resource group: `iatd_labs_05_rg`
            *   Virtual machine name: `iatd_labs_05_vm_target`
            *   Region: Your region.
            *   Image: `Ubuntu Server 22.04 LTS`
            *   Size: Choose a size (e.g., Standard_B1ls).
            *   Username: `azureuser`
            *   Authentication type: `Password`
            *   Password: Set a password.
        *   **Networking Tab:**
            *   Virtual network: `iatd_labs_05_vnet`
            *   Subnet: `iatd_labs_05_subnet`
            *   Public IP: Select `iatd_labs_05_pip`
            *   NIC network security group: `None`
        *   **Management Tab:**
            *   Disable boot diagnostics
    *   Click **Review + create**, then **Create**.

8.  **Install a Web Server (e.g., Nginx) on the VM:**

    *   Connect to the VM via SSH:
    
        ```bash
        # Get the public IP address of the VM
        VM_IP=$(az vm show -d -g $RESOURCE_GROUP -n iatd_labs_05_vm_target --query publicIps -o tsv)
        
        # SSH into the VM
        ssh azureuser@$VM_IP
        ```
    
    *   Run the following commands:

        ```bash
        sudo apt update
        sudo apt install nginx -y
        sudo systemctl start nginx
        sudo systemctl enable nginx
        ```

9.  **Test the Web Server:**
    *   Open a web browser and navigate to the Public IP address of the VM. You should see the Nginx welcome page.

### Part 2: Setting up Azure Monitoring Services

1.  **Create a Log Analytics Workspace (Portal):**
    *   Search for "Log Analytics workspaces" and select.
    *   Click **Create**.
        *   Subscription: Your subscription.
        *   Resource group: `iatd_labs_05_rg`
        *   Name: `iatd_labs_05_law`
        *   Region: Your region.
    *   Click **Review + create**, then **Create**.

2.  **Enable VM Insights (Portal):**
    *   Navigate to the `iatd_labs_05_vm_target` VM in the Azure Portal.
    *   Select **Insights** under **Monitoring**.
    *   If prompted, configure the workspace to the newly created `iatd_labs_05_law` Log Analytics Workspace.

3.  **Create an Application Insights Resource (Portal):**
    *   Search for "Application Insights" and select.
    *   Click **Create**.
        *   Subscription: Your subscription.
        *   Resource group: `iatd_labs_05_rg`
        *   Name: `iatd_labs_05_appi`
        *   Region: Your region.
        *   Resource Mode: Classic
        *   Log Analytics Workspace: `iatd_labs_05_law`

    *   Click **Review + create**, then **Create**.

### Part 3: Monitoring Types and Log Analysis

1.  **Active Monitoring:**
    *   **Concept:** Actively probing systems to check their status (e.g., pinging a server).
    *   **Implementation:** Use `ping` or `traceroute` from Cloud Shell to the VM's Public IP and observe the results.
    
        ```bash
        # Ping the VM
        ping -c 4 $VM_IP
        
        # Traceroute to the VM
        traceroute $VM_IP
        ```
    
    *   **Scenario:** Verifying network connectivity.

2.  **Passive Monitoring:**
    *   **Concept:** Collecting data without actively probing (e.g., analyzing server logs).
    *   **Implementation:** Examine the Nginx access logs on the VM (`/var/log/nginx/access.log`).
    
        ```bash
        # SSH into the VM
        ssh azureuser@$VM_IP
        
        # View the Nginx access logs
        sudo tail -f /var/log/nginx/access.log
        ```
    
    *   **Scenario:** Identifying traffic patterns and potential errors.

3.  **Synthetic Monitoring:**
    *   **Concept:** Simulating user behavior to test application performance and availability.
    *   **Implementation:** In Application Insights, configure a "URL ping test" to periodically check the availability of the VM's web server.
        *   Navigate to the `iatd_labs_05_appi` Application Insights resource.
        *   Select **Availability** under **Investigate**.
        *   Click **Add test**.
        *   Configure the test to ping the VM's Public IP address every 5 minutes.

4.  **Flow-Based Monitoring:**
    *   **Concept:** Analyzing network traffic flow data to identify patterns and anomalies.
    *   **Implementation:** Enable NSG Flow Logs for the `iatd_labs_05_nsg` Network Security Group.
        *   Search for "Network Watcher" and select.
        *   Select **NSG flow logs** under **Logs**.
        *   Select the `iatd_labs_05_nsg` Network Security Group.
        *   Choose a storage account to store the flow logs.

5.  **Log Analysis with Kusto Query Language (KQL):**
    *   Navigate to the `iatd_labs_05_law` Log Analytics Workspace in the Azure Portal.
    *   Select **Logs** and run the following KQL query to analyze VM performance:

        ```kusto
        Perf
        | where ObjectName == "Processor" and CounterName == "% Processor Time"
        | summarize avg(CounterValue) by bin(TimeGenerated, 1h), Computer
        | render timechart
        ```

### Part 4: Performance Baselines and Alert Thresholds

1.  **Establishing Performance Baselines:**
    *   **Concept:** Determining the normal operating parameters of a system.
    *   **Implementation:** Monitor the VM's CPU, memory, and network usage over a period of time to establish a baseline.
        *   Navigate to the `iatd_labs_05_vm_target` VM in the Azure Portal.
        *   Select **Metrics** under **Monitoring**.
        *   Add metrics for CPU, memory, and network usage.
        *   Observe the patterns over time to establish a baseline.

2.  **Setting Alert Thresholds:**
    *   **Concept:** Defining thresholds that trigger alerts when exceeded.
    *   **Implementation:** Create an alert rule to notify you when the VM's CPU usage exceeds 80%.
        *   Navigate to the `iatd_labs_05_vm_target` VM in the Azure Portal.
        *   Select **Alerts** under **Monitoring**.
        *   Click **Create alert rule**.
        *   Configure the alert to trigger when the VM's CPU usage exceeds 80%.

3.  **Static vs. Dynamic Thresholds:**
    *   **Concept:**
        *   **Static Thresholds:** Fixed values that trigger alerts (e.g., CPU > 80%).
        *   **Dynamic Thresholds:** Automatically adjust based on historical data and seasonality.
    *   **Implementation:** Discuss the pros and cons of each approach and when to use them.

### Post-Lab: Cleanup

To avoid incurring unnecessary costs, it's essential to clean up the resources created during this lab.

1.  **Delete Resource Group:**

    ```bash
    az group delete --name $RESOURCE_GROUP --yes
    ```

**Outcomes to be Achieved:**

*   Understand different monitoring types (active, passive, synthetic, flow-based).
*   Analyze logs using Kusto Query Language (KQL).
*   Establish performance baselines and set alert thresholds.
*   Configure and use core Azure monitoring services (Log Analytics, VM Insights, Application Insights).

**Congratulations!** You have successfully completed this lab on monitoring and logging fundamentals in Azure. You've learned about different monitoring types, log analysis techniques, performance baselines, alert thresholds, and core Azure monitoring services.
