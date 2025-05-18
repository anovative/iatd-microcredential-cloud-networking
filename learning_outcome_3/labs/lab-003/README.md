## IATD Microcredential Cloud Networking: Lab 3 - DDoS Mitigation in Azure - Fundamentals

**Objective:** This lab introduces you to the fundamental concepts of DDoS attacks and how to mitigate them using Azure services. You will learn about different types of DDoS attacks, common mitigation techniques, and how to configure basic DDoS protection in Azure.

**Estimated Time:** 75 - 90 minutes

**Prerequisites:**

1.  **Azure Subscription:** Requires an active Azure subscription. A free account is available at [https://azure.microsoft.com/free/](https://azure.microsoft.com/free/).
2.  **Basic knowledge of Azure Networking:** Familiarity with Virtual Networks, Network Security Groups, and Virtual Machines.

### Lab Conventions

*   **Azure Cloud Shell:** All PowerShell and CLI interactions will occur within the Azure Cloud Shell environment.
*   **Naming Conventions:** Resource names follow the convention `iatd_labs_03_*`.
*   **IP Address Range:** The `172.16.0.0/16` range will be used (as per standardization).
*   **Location:** Choose a consistent Azure region.

#### Resource Naming

*   **Resource Group:** `iatd_labs_03_rg`
*   **Virtual Network:** `iatd_labs_03_vnet`
*   **Subnet:** `iatd_labs_03_subnet`
*   **VM-Target:** `iatd_labs_03_vm_target`
*   **Public IP:** `iatd_labs_03_pip`
*   **Network Security Group:** `iatd_labs_03_nsg`

### Part 1: Simulating a Basic DDoS Attack (Conceptual)

1.  **Understanding DDoS Attack Types:**
    *   **Volume-Based Attacks:** Overwhelm the target with high volumes of traffic (e.g., UDP floods, ICMP floods).
    *   **Protocol Attacks:** Exploit weaknesses in network protocols (e.g., SYN floods).
    *   **Application-Layer Attacks:** Target specific application features (e.g., HTTP floods, slowloris).

2.  **Simulating a Volume-Based Attack (Conceptual):**
    *   Discuss how tools like `hping3` or `nmap` can be used to generate large volumes of traffic.
    *   Explain that, for this lab, we will *not* actually launch a real attack, as this is unethical and potentially illegal. We will focus on *simulating* the *effects* of an attack.

### Part 2: Setting up a Target Virtual Machine

1.  **Create a Resource Group:**

    ```bash
    RESOURCE_GROUP="iatd_labs_03_rg"
    LOCATION="eastus" # Your Region

    az group create --name $RESOURCE_GROUP --location $LOCATION
    ```

    **Expected Output:**
    ```json
    {
      "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/iatd_labs_03_rg",
      "location": "eastus",
      "managedBy": null,
      "name": "iatd_labs_03_rg",
      "properties": {
        "provisioningState": "Succeeded"
      },
      "tags": null,
      "type": "Microsoft.Resources/resourceGroups"
    }
    ```

2.  **Create a Virtual Network and Subnet:**

    ```bash
    VNET_NAME="iatd_labs_03_vnet"
    SUBNET_NAME="iatd_labs_03_subnet"
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
        "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/iatd_labs_03_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_03_vnet",
        "location": "eastus",
        "name": "iatd_labs_03_vnet",
        "provisioningState": "Succeeded",
        "resourceGroup": "iatd_labs_03_rg",
        "subnets": [
          {
            "addressPrefix": "172.16.0.0/24",
            "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/iatd_labs_03_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_03_vnet/subnets/iatd_labs_03_subnet",
            "name": "iatd_labs_03_subnet",
            "provisioningState": "Succeeded",
            "resourceGroup": "iatd_labs_03_rg"
          }
        ],
        "type": "Microsoft.Network/virtualNetworks"
      }
    }
    ```

3.  **Create a Public IP Address:**

    ```bash
    PUBLIC_IP_NAME="iatd_labs_03_pip"

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
        "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/iatd_labs_03_rg/providers/Microsoft.Network/publicIPAddresses/iatd_labs_03_pip",
        "location": "eastus",
        "name": "iatd_labs_03_pip",
        "provisioningState": "Succeeded",
        "publicIPAddressVersion": "IPv4",
        "publicIPAllocationMethod": "Static",
        "resourceGroup": "iatd_labs_03_rg",
        "sku": {
          "name": "Standard"
        },
        "type": "Microsoft.Network/publicIPAddresses"
      }
    }
    ```

4.  **Create a Network Security Group:**

    ```bash
    NSG_NAME="iatd_labs_03_nsg"

    az network nsg create \
        --resource-group $RESOURCE_GROUP \
        --name $NSG_NAME \
        --location $LOCATION
    ```

    **Expected Output:**
    ```json
    {
      "NewNSG": {
        "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/iatd_labs_03_rg/providers/Microsoft.Network/networkSecurityGroups/iatd_labs_03_nsg",
        "location": "eastus",
        "name": "iatd_labs_03_nsg",
        "provisioningState": "Succeeded",
        "resourceGroup": "iatd_labs_03_rg",
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
      "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/iatd_labs_03_rg/providers/Microsoft.Network/networkSecurityGroups/iatd_labs_03_nsg/securityRules/AllowHTTP",
      "name": "AllowHTTP",
      "priority": 100,
      "protocol": "Tcp",
      "provisioningState": "Succeeded",
      "resourceGroup": "iatd_labs_03_rg",
      "sourceAddressPrefix": null,
      "sourceAddressPrefixes": [
        "Internet"
      ],
      "sourcePortRange": "*",
      "sourcePortRanges": [],
      "type": "Microsoft.Network/networkSecurityGroups/securityRules"
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
      "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/iatd_labs_03_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_03_vnet/subnets/iatd_labs_03_subnet",
      "name": "iatd_labs_03_subnet",
      "networkSecurityGroup": {
        "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/iatd_labs_03_rg/providers/Microsoft.Network/networkSecurityGroups/iatd_labs_03_nsg",
        "resourceGroup": "iatd_labs_03_rg"
      },
      "provisioningState": "Succeeded",
      "resourceGroup": "iatd_labs_03_rg"
    }
    ```

7.  **Create a Virtual Machine (Portal):**
    *   Search for "Virtual machines" and select.
    *   Click **Create** -> **Azure virtual machine**.
        *   **Basics Tab:**
            *   Subscription: Your subscription.
            *   Resource group: `iatd_labs_03_rg`
            *   Virtual machine name: `iatd_labs_03_vm_target`
            *   Region: Your region.
            *   Image: `Ubuntu Server 22.04 LTS`
            *   Size: Choose a size (e.g., Standard_B1ls).
            *   Username: `azureuser`
            *   Authentication type: `Password`
            *   Password: Set a password.
        *   **Networking Tab:**
            *   Virtual network: `iatd_labs_03_vnet`
            *   Subnet: `iatd_labs_03_subnet`
            *   Public IP: Select `iatd_labs_03_pip`
            *   NIC network security group: `None`
        *   **Management Tab:**
            *   Disable boot diagnostics
    *   Click **Review + create**, then **Create**.

8.  **Install a Web Server (e.g., Nginx) on the VM:**

    *   Connect to the VM via SSH:
    
    ```bash
    # Get the public IP address of the VM
    VM_IP=$(az network public-ip show --resource-group $RESOURCE_GROUP --name $PUBLIC_IP_NAME --query ipAddress -o tsv)
    
    # Connect to the VM
    ssh azureuser@$VM_IP
    ```
    
    *   Run the following commands on the VM:

    ```bash
    sudo apt update
    sudo apt install nginx -y
    sudo systemctl start nginx
    sudo systemctl enable nginx
    ```

    **Expected Output:**
    ```
    Reading package lists... Done
    Building dependency tree... Done
    Reading state information... Done
    ...
    The following NEW packages will be installed:
      nginx nginx-common nginx-core
    ...
    Setting up nginx (1.18.0-6ubuntu14.4) ...
    Processing triggers for man-db (2.10.2-1) ...
    ```

9.  **Test the Web Server:**
    *   Open a web browser and navigate to the Public IP address of the VM. You should see the Nginx welcome page.
    *   Alternatively, you can test using curl:
    
    ```bash
    # Exit the SSH session if you're still connected
    exit
    
    # Test the web server using curl
    curl http://$VM_IP
    ```
    
    **Expected Output:**
    ```html
    <!DOCTYPE html>
    <html>
    <head>
    <title>Welcome to nginx!</title>
    <style>
        body {
            width: 35em;
            margin: 0 auto;
            font-family: Tahoma, Verdana, Arial, sans-serif;
        }
    </style>
    </head>
    <body>
    <h1>Welcome to nginx!</h1>
    <p>If you see this page, the nginx web server is successfully installed and
    working. Further configuration is required.</p>
    ...
    </body>
    </html>
    ```

### Part 3: Simulating DDoS Attack Indicators and Observing Effects

1.  **Simulating Network-Level Indicators:**
    *   Discuss how a DDoS attack would manifest as a sudden surge in network traffic to the VM's Public IP.
    *   Explain that metrics like "Network In" and "Network Out" in Azure Monitor would spike.
    *   To observe these metrics:
        *   Navigate to the VM in the Azure Portal.
        *   Select **Metrics** under **Monitoring**.
        *   Add metrics for "Network In" and "Network Out".

2.  **Simulating Server-Level Indicators:**
    *   Discuss how a DDoS attack could lead to high CPU utilization and memory consumption on the VM.
    *   Explain that tools like `top` or `htop` within the VM would show these symptoms.
    *   To observe these metrics:
        *   Connect to the VM via SSH.
        *   Install and run `htop`:
        
        ```bash
        sudo apt install htop -y
        htop
        ```
        
        *   Press `q` to exit `htop`.

3.  **Simulating Application-Level Indicators:**
    *   Discuss how a DDoS attack targeting the web server could result in slow response times or connection errors.
    *   Explain that web server logs would show a large number of requests from various IP addresses.
    *   To view Nginx access logs:
    
    ```bash
    sudo tail -f /var/log/nginx/access.log
    ```
    
    *   Generate some traffic to see log entries:
    
    ```bash
    # In a separate terminal
    for i in {1..10}; do curl http://$VM_IP; done
    ```

### Part 4: Implementing Basic DDoS Mitigation Techniques

1.  **Traffic Filtering with NSGs:**
    *   Discuss how NSG rules can be used to block traffic from specific IP addresses or ranges that are identified as malicious.
    *   Create a rule to block traffic from a specific IP address (for demonstration purposes):
    
    ```bash
    # Replace with an IP address you want to block
    BLOCKED_IP="1.2.3.4"
    
    az network nsg rule create \
        --resource-group $RESOURCE_GROUP \
        --nsg-name $NSG_NAME \
        --name BlockMaliciousIP \
        --priority 90 \
        --source-address-prefixes $BLOCKED_IP \
        --destination-port-ranges '*' \
        --access Deny \
        --protocol '*' \
        --direction Inbound
    ```
    
    **Expected Output:**
    ```json
    {
      "access": "Deny",
      "description": null,
      "destinationAddressPrefix": "*",
      "destinationAddressPrefixes": [],
      "destinationPortRange": null,
      "destinationPortRanges": [
        "*"
      ],
      "direction": "Inbound",
      "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/iatd_labs_03_rg/providers/Microsoft.Network/networkSecurityGroups/iatd_labs_03_nsg/securityRules/BlockMaliciousIP",
      "name": "BlockMaliciousIP",
      "priority": 90,
      "protocol": "*",
      "provisioningState": "Succeeded",
      "resourceGroup": "iatd_labs_03_rg",
      "sourceAddressPrefix": null,
      "sourceAddressPrefixes": [
        "1.2.3.4"
      ],
      "sourcePortRange": "*",
      "sourcePortRanges": [],
      "type": "Microsoft.Network/networkSecurityGroups/securityRules"
    }
    ```
    
    *   Explain the limitations of this approach for large-scale, distributed attacks.

2.  **Rate Limiting with Nginx (Practical):**
    *   Connect to the VM via SSH.
    *   Install the Nginx rate limiting module:
    
    ```bash
    # No additional installation needed as rate limiting is built into Nginx
    ```
    
    *   Edit the Nginx configuration to implement rate limiting:
    
    ```bash
    sudo nano /etc/nginx/nginx.conf
    ```
    
    *   Add the following lines inside the `http` block:
    
    ```
    # Define a zone for rate limiting
    limit_req_zone $binary_remote_addr zone=mylimit:10m rate=10r/s;
    ```
    
    *   Edit the default site configuration:
    
    ```bash
    sudo nano /etc/nginx/sites-available/default
    ```
    
    *   Add the following line inside the `location /` block:
    
    ```
    limit_req zone=mylimit burst=20 nodelay;
    ```
    
    *   Test the configuration and reload Nginx:
    
    ```bash
    sudo nginx -t
    sudo systemctl reload nginx
    ```
    
    **Expected Output:**
    ```
    nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
    nginx: configuration file /etc/nginx/nginx.conf test is successful
    ```
    
    *   Explain that this configuration limits each IP address to 10 requests per second, with a burst of 20 requests allowed.

3.  **Testing Rate Limiting:**
    *   Use a tool like `ab` (Apache Benchmark) to generate a high number of requests:
    
    ```bash
    # Install Apache Benchmark
    sudo apt install apache2-utils -y
    
    # Generate 100 requests with 10 concurrent connections
    ab -n 100 -c 10 http://localhost/
    ```
    
    *   Check the Nginx error log to see if any requests were rate-limited:
    
    ```bash
    sudo tail -f /var/log/nginx/error.log
    ```
    
    *   You should see entries like:
    
    ```
    2023/04/01 12:34:56 [error] 12345#12345: *123 limiting requests, excess: 1.234 by zone "mylimit", client: 127.0.0.1, server: localhost, request: "GET / HTTP/1.0", host: "localhost"
    ```

### Post-Lab: Cleanup

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
    Resource group 'iatd_labs_03_rg' could not be found.
    ```

**Outcomes to be Achieved:**

*   Understand different types of DDoS attacks.
*   Simulate the effects of a DDoS attack and identify key indicators.
*   Provision a target virtual machine for DDoS attack simulation.
*   Configure basic network security using Azure Network Security Groups (NSGs).
*   Implement and test rate limiting as a basic DDoS mitigation technique.
*   Understand the limitations of basic DDoS mitigation techniques and the need for more advanced solutions (covered in Lab 4).

**Congratulations!** You have successfully completed this lab on DDoS mitigation fundamentals in Azure. You've learned about different types of DDoS attacks, how to identify attack indicators, and how to implement basic mitigation techniques using NSGs and rate limiting.
