## IATD Microcredential Cloud Networking: Lab 2 - Advanced NSG Rule Management, Service Tags, and Security Policies

**Objective:** This lab builds upon the fundamentals of NSGs by diving into more advanced rule management techniques, focusing on optimizing rule configuration with service tags and implementing security policies that align with organizational compliance requirements.

**Estimated Time:** 75 - 90 minutes

**Prerequisites:**

1.  **Azure Subscription:** Requires an active Azure subscription. A free account is available at [https://azure.microsoft.com/free/](https://azure.microsoft.com/free/).
2.  **Completion of Lab 1:** This lab assumes you have a working knowledge of NSGs and ASGs, as covered in Lab 1.

**Let's get started!**

### Lab Conventions

*   **Azure Cloud Shell:** All PowerShell and CLI interactions will occur within the Azure Cloud Shell environment.
*   **Naming Conventions:** Resource names follow the convention `iatd_labs_02_*`.
*   **IP Address Range:** The `172.16.0.0/16` range will be used (as per standardization).
*   **Location:** Choose a consistent Azure region.

#### Resource Naming

*   **Resource Group:** `iatd_labs_02_rg`
*   **Virtual Network:** `iatd_labs_02_vnet`
*   **Subnet-Web:** `iatd_labs_02_subnet_web`
*   **Subnet-App:** `iatd_labs_02_subnet_app`
*   **NSG-Web:** `iatd_labs_02_nsg_web`
*   **NSG-App:** `iatd_labs_02_nsg_app`
*   **VM-Web-01:** `iatd_labs_02_vm_web_01`
*   **VM-App-01:** `iatd_labs_02_vm_app_01`
*   **ASG-Web:** `iatd_labs_02_asg_web`
*   **ASG-App:** `iatd_labs_02_asg_app`

### Part 1: Environment Setup (CLI)

This part will recreate a simple environment similar to the end of Lab 1.

1.  **Sign in to the Azure Portal:** Browse to [https://portal.azure.com/](https://portal.azure.com/) and sign in.

2.  **Open Cloud Shell:** In the Azure Portal, click the Cloud Shell icon. Select **Bash** if prompted.

3.  **Set Variables:**

    ```bash
    RESOURCE_GROUP="iatd_labs_02_rg"
    LOCATION="eastus" # Your Region
    VNET_NAME="iatd_labs_02_vnet"
    SUBNET_WEB_NAME="iatd_labs_02_subnet_web"
    SUBNET_APP_NAME="iatd_labs_02_subnet_app"
    NSG_WEB_NAME="iatd_labs_02_nsg_web"
    NSG_APP_NAME="iatd_labs_02_nsg_app"
    ASG_WEB_NAME="iatd_labs_02_asg_web"
    ASG_APP_NAME="iatd_labs_02_asg_app"

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
      "id": "/subscriptions/<your_subscription_id>/resourceGroups/iatd_labs_02_rg",
      "location": "eastus",
      "managedBy": null,
      "name": "iatd_labs_02_rg",
      "properties": {
        "provisioningState": "Succeeded"
      },
      "tags": null,
      "type": "Microsoft.Resources/resourceGroups"
    }
    ```

5.  **Create Virtual Network with Web and App Subnets:**

    ```bash
    az network vnet create \
        --resource-group $RESOURCE_GROUP \
        --name $VNET_NAME \
        --address-prefixes $VNET_PREFIX \
        --subnet-name $SUBNET_WEB_NAME \
        --subnet-prefixes $SUBNET_WEB_PREFIX \
        --location $LOCATION

    az network vnet subnet create \
        --resource-group $RESOURCE_GROUP \
        --vnet-name $VNET_NAME \
        --name $SUBNET_APP_NAME \
        --address-prefix $SUBNET_APP_PREFIX
    ```

    **Example Output (for the first command):**

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
        "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_02_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_02_vnet",
        "location": "eastus",
        "name": "iatd_labs_02_vnet",
        "provisioningState": "Succeeded",
        "resourceGuid": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "subnets": [
          {
            "addressPrefix": "172.16.0.0/24",
            "delegations": [],
            "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_02_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_02_vnet/subnets/iatd_labs_02_subnet_web",
            "name": "iatd_labs_02_subnet_web",
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

    **Example Output (for the second command):**

    ```json
    {
      "addressPrefix": "172.16.1.0/24",
      "delegations": [],
      "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_02_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_02_vnet/subnets/iatd_labs_02_subnet_app",
      "name": "iatd_labs_02_subnet_app",
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

6.  **Create NSGs:**

    ```bash
    az network nsg create \
        --resource-group $RESOURCE_GROUP \
        --name $NSG_WEB_NAME \
        --location $LOCATION

    az network nsg create \
        --resource-group $RESOURCE_GROUP \
        --name $NSG_APP_NAME \
        --location $LOCATION
    ```

    **Example Output (for the first command):**

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
      "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_02_rg/providers/Microsoft.Network/networkSecurityGroups/iatd_labs_02_nsg_web",
      "location": "eastus",
      "name": "iatd_labs_02_nsg_web",
      "provisioningState": "Succeeded",
      "resourceGuid": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "securityRules": [],
      "type": "Microsoft.Network/networkSecurityGroups"
    }
    ```

    **Example Output (for the second command):**

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
      "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_02_rg/providers/Microsoft.Network/networkSecurityGroups/iatd_labs_02_nsg_app",
      "location": "eastus",
      "name": "iatd_labs_02_nsg_app",
      "provisioningState": "Succeeded",
      "resourceGuid": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "securityRules": [],
      "type": "Microsoft.Network/networkSecurityGroups"
    }
    ```

7.  **Associate NSGs with Subnets:**

    ```bash
    az network vnet subnet update \
        --resource-group $RESOURCE_GROUP \
        --vnet-name $VNET_NAME \
        --name $SUBNET_WEB_NAME \
        --network-security-group $NSG_WEB_NAME

    az network vnet subnet update \
        --resource-group $RESOURCE_GROUP \
        --vnet-name $VNET_NAME \
        --name $SUBNET_APP_NAME \
        --network-security-group $NSG_APP_NAME
    ```

    **Example Output (for the first command):**

    ```json
    {
      "addressPrefix": "172.16.0.0/24",
      "delegations": [],
      "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_02_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_02_vnet/subnets/iatd_labs_02_subnet_web",
      "name": "iatd_labs_02_subnet_web",
      "networkSecurityGroup": {
        "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_02_rg/providers/Microsoft.Network/networkSecurityGroups/iatd_labs_02_nsg_web",
        "resourceGroup": "iatd_labs_02_rg"
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

    **Example Output (for the second command):**

    ```json
    {
      "addressPrefix": "172.16.1.0/24",
      "delegations": [],
      "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_02_rg/providers/Microsoft.Network/virtualNetworks/iatd_labs_02_vnet/subnets/iatd_labs_02_subnet_app",
      "name": "iatd_labs_02_subnet_app",
      "networkSecurityGroup": {
        "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_02_rg/providers/Microsoft.Network/networkSecurityGroups/iatd_labs_02_nsg_app",
        "resourceGroup": "iatd_labs_02_rg"
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

8.  **Create ASGs:**

    ```bash
    az network asg create \
        --resource-group $RESOURCE_GROUP \
        --name $ASG_WEB_NAME \
        --location $LOCATION

    az network asg create \
        --resource-group $RESOURCE_GROUP \
        --name $ASG_APP_NAME \
        --location $LOCATION
    ```

    **Example Output (for the first command):**

    ```json
    {
      "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_02_rg/providers/Microsoft.Network/applicationSecurityGroups/iatd_labs_02_asg_web",
      "location": "eastus",
      "name": "iatd_labs_02_asg_web",
      "provisioningState": "Succeeded",
      "resourceGroup": "iatd_labs_02_rg",
      "resourceGuid": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "type": "Microsoft.Network/applicationSecurityGroups"
    }
    ```

    **Example Output (for the second command):**

    ```json
    {
      "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_02_rg/providers/Microsoft.Network/applicationSecurityGroups/iatd_labs_02_asg_app",
      "location": "eastus",
      "name": "iatd_labs_02_asg_app",
      "provisioningState": "Succeeded",
      "resourceGroup": "iatd_labs_02_rg",
      "resourceGuid": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "type": "Microsoft.Network/applicationSecurityGroups"
    }
    ```

### Part 2: Advanced NSG Rule Management and Prioritization (Portal)

1.  **Examine Default Rules:**
    *   Navigate to the `iatd_labs_02_nsg_web` NSG in the Azure Portal.
    *   Select **Inbound security rules** and **Outbound security rules** under **Settings**.
    *   Observe the default rules that allow VNet traffic and deny internet inbound traffic. Note the priorities (65000, 65001, 65500).

2.  **Create Conflicting Rules (Portal):**
    *   In `iatd_labs_02_nsg_web`, add the following **inbound** rule:
        *   Source: `Any`
        *   Source port range: `*`
        *   Destination: `Any`
        *   Destination port range: `80`
        *   Protocol: `TCP`
        *   Action: `Deny`
        *   Priority: `100`
        *   Name: `DenyHTTP`
        *   Click **Add**.

    *   Add the following **inbound** rule:
        *   Source: `Any`
        *   Source port range: `*`
        *   Destination: `Any`
        *   Destination port range: `80`
        *   Protocol: `TCP`
        *   Action: `Allow`
        *   Priority: `110`
        *   Name: `AllowHTTP`
        *   Click **Add**.

3.  **Testing Rule Prioritization:**
    *   Create a VM in `iatd_labs_02_subnet_web`. You can quickly deploy one without associating an NSG during VM creation. Associate the network interface with `iatd_labs_02_asg_web`.
    *   Attempt to access the VM's public IP address via HTTP in a web browser.
    *   The HTTP access *should* be allowed because `AllowHTTP` has a higher priority (110) than `DenyHTTP` (100). The lower the number, the higher the priority.
    *   If HTTP access is denied, double-check the rule priorities and that the NSG is associated with the subnet correctly.

### Part 3: Implementing Service Tags (CLI)

1.  **Remove the `AllowHTTP` and `DenyHTTP` Rules (Portal):** Delete both rules from `iatd_labs_02_nsg_web`.

2.  **Create a Rule Using the `Internet` Service Tag (CLI):**

    ```bash
    az network nsg rule create \
        --resource-group $RESOURCE_GROUP \
        --nsg-name $NSG_WEB_NAME \
        --name AllowInternetHTTP \
        --priority 100 \
        --source-address-prefixes Internet \
        --destination-port-ranges 80,443 \
        --access Allow \
        --protocol Tcp \
        --direction Inbound
    ```

    **Example Output:**

    ```json
    {
      "access": "Allow",
      "description": null,
      "destinationAddressPrefix": "*",
      "destinationAddressPrefixes": [],
      "destinationApplicationSecurityGroups": null,
      "destinationPortRange": null,
      "destinationPortRanges": [
        "80",
        "443"
      ],
      "direction": "Inbound",
      "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_02_rg/providers/Microsoft.Network/networkSecurityGroups/iatd_labs_02_nsg_web/securityRules/AllowInternetHTTP",
      "name": "AllowInternetHTTP",
      "priority": 100,
      "protocol": "Tcp",
      "provisioningState": "Succeeded",
      "resourceGroup": "iatd_labs_02_rg",
      "sourceAddressPrefix": null,
      "sourceAddressPrefixes": [
        "Internet"
      ],
      "sourceApplicationSecurityGroups": null,
      "sourcePortRange": "*",
      "sourcePortRanges": [],
      "type": "Microsoft.Network/networkSecurityGroups/securityRules"
    }
    ```

3.  **Test the `Internet` Service Tag:**
    *   Try to access the VM's public IP address using both HTTP (port 80) and HTTPS (port 443). Both should now be allowed from any internet address.

    **Verification Steps:**
    
    1. Deploy a simple web VM in the web subnet if you haven't already:
       ```bash
       az vm create \
         --resource-group $RESOURCE_GROUP \
         --name iatd_labs_02_vm_web_01 \
         --image UbuntuLTS \
         --admin-username azureuser \
         --generate-ssh-keys \
         --vnet-name $VNET_NAME \
         --subnet $SUBNET_WEB_NAME \
         --public-ip-address iatd_labs_02_vm_web_01_pip
       ```
    
    2. Install a web server on the VM:
       ```bash
       # Get the public IP
       WEB_VM_IP=$(az vm show -g $RESOURCE_GROUP -n iatd_labs_02_vm_web_01 -d --query publicIps -o tsv)
       
       # Install Apache
       ssh azureuser@$WEB_VM_IP "sudo apt update && sudo apt install -y apache2"
       ```
    
    3. Test HTTP access by opening a browser to http://<web-vm-ip>
       
       **Expected Result:** You should see the default Apache2 Ubuntu page, confirming that the Internet service tag is correctly allowing HTTP traffic.

4.  **Create a Rule Using the `AppService` Service Tag (CLI):**

    ```bash
    az network nsg rule create \
        --resource-group $RESOURCE_GROUP \
        --nsg-name $NSG_APP_NAME \
        --name AllowAppServiceAccess \
        --priority 100 \
        --source-address-prefixes AppService \
        --destination-port-ranges 8080 \
        --access Allow \
        --protocol Tcp \
        --direction Inbound
    ```

    **Example Output:**

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
      "id": "/subscriptions/<subscription_id>/resourceGroups/iatd_labs_02_rg/providers/Microsoft.Network/networkSecurityGroups/iatd_labs_02_nsg_app/securityRules/AllowAppServiceAccess",
      "name": "AllowAppServiceAccess",
      "priority": 100,
      "protocol": "Tcp",
      "provisioningState": "Succeeded",
      "resourceGroup": "iatd_labs_02_rg",
      "sourceAddressPrefix": null,
      "sourceAddressPrefixes": [
        "AppService"
      ],
      "sourceApplicationSecurityGroups": null,
      "sourcePortRange": "*",
      "sourcePortRanges": [],
      "type": "Microsoft.Network/networkSecurityGroups/securityRules"
    }
    ```

5.  **Explore Available Service Tags (Portal):**
    *   In the Azure Portal, when creating an NSG rule, you can select "Service Tag" as the source or destination. Click the dropdown to see a list of available service tags and their descriptions. Note the variety of tags available.

### Part 4: Define a Custom Security Policy (Portal & CLI)

Let's simulate defining a security policy based on a fictitious compliance requirement.

**Scenario:** *Contoso Corp requires that all web-tier VMs must only accept inbound HTTP/HTTPS traffic from the internet, and all application-tier VMs must only accept traffic from the web-tier on port 8080.*

1.  **Verify Web-Tier NSG (`iatd_labs_02_nsg_web`) (Portal):**
    *   Ensure the NSG only has the `AllowInternetHTTP` rule (created in Part 3) allowing inbound traffic on ports 80 and 443 from the `Internet` service tag. Remove any other inbound rules.

2.  **Configure App-Tier NSG (`iatd_labs_02_nsg_app`) (CLI):**
    *   First, get the address prefix of `iatd_labs_02_subnet_web`:

        ```bash
        WEB_SUBNET_PREFIX=$(az network vnet subnet show --resource-group $RESOURCE_GROUP --vnet-name $VNET_NAME --name $SUBNET_WEB_NAME --query addressPrefix -o tsv)
        echo $WEB_SUBNET_PREFIX
        ```

    *   Now, create the rule to allow traffic from the web subnet:

        ```bash
        az network nsg rule create \
            --resource-group $RESOURCE_GROUP \
            --nsg-name $NSG_APP_NAME \
            --name AllowFromWebTier \
            --priority 100 \
            --source-address-prefixes $WEB_SUBNET_PREFIX \
            --destination-port-ranges 8080 \
            --access Allow \
            --protocol Tcp \
            --direction Inbound
        ```

    *   Remove any other inbound rules from `iatd_labs_02_nsg_app`.

3.  **Testing the Security Policy:**
    *   Ensure the web-tier VM can still be accessed via HTTP/HTTPS from the internet.
    *   Ensure the application-tier VM *cannot* be accessed directly from the internet.
    *   If you have a mechanism for the web-tier VM to communicate with the application-tier VM on port 8080, verify that this communication is allowed.

### Part 5: Security Policy Documentation (Conceptual)

This part is conceptual and doesn't involve hands-on configuration.

1.  **Documenting Security Policies:**
    *   Create a document (e.g., Word document, Wiki page) outlining the security policies implemented in this lab. Include:
        *   A description of the policy: "Contoso Corp requires that all web-tier VMs must only accept inbound HTTP/HTTPS traffic from the internet, and all application-tier VMs must only accept traffic from the web-tier on port 8080."
        *   The rationale behind the policy (e.g., compliance requirements, security best practices).
        *   A detailed description of the NSG rules that implement the policy, including screenshots of the rule configurations.
        *   Diagrams illustrating the network topology and traffic flow.

2.  **Change Management Procedures:**
    *   Outline the process for requesting and implementing changes to the security policies. This should include:
        *   A formal change request process.
        *   Review and approval by security stakeholders.
        *   Testing and validation of changes before deployment.
        *   Documentation of changes made.

3.  **Security Baselines:**
    *   Define a security baseline for all VMs in your environment. This should include:
        *   Minimum requirements for operating system hardening.
        *   Requirements for patching and updates.
        *   Requirements for logging and monitoring.

### Post-Lab: Cleanup

To avoid incurring unnecessary costs, it's essential to clean up the resources created during this lab.

1.  **Delete Resource Group (CLI):**

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
    Resource group 'iatd_labs_02_rg' could not be found.
    ```

**Congratulations!** You have successfully completed this lab on advanced NSG rule management, service tags, and security policies. You've learned how to implement more sophisticated network security configurations in Azure, which are essential skills for securing cloud environments in enterprise settings.
