## IATD Microcredential Cloud Networking: Lab 4 - DDoS Mitigation in Azure - Advanced

**Objective:** This lab builds upon the fundamentals by exploring more advanced DDoS mitigation techniques using Azure's built-in DDoS protection services and Web Application Firewall (WAF).

**Estimated Time:** 75 - 90 minutes

**Prerequisites:**

1.  **Azure Subscription:** Requires an active Azure subscription.
2.  **Completion of Lab 3:** This lab assumes you have a working knowledge of DDoS attacks and basic mitigation techniques.
3.  **Familiarity with Azure Web Application Firewall (WAF) is helpful.**

### Lab Conventions

*   **Azure Cloud Shell:** All PowerShell and CLI interactions will occur within the Azure Cloud Shell environment.
*   **Naming Conventions:** Resource names follow the convention `iatd_labs_04_*`.
*   **IP Address Range:** The `172.16.0.0/16` range will be used (as per standardization).
*   **Location:** Choose a consistent Azure region.

#### Resource Naming (re-use from Lab 3 where applicable)

*   **Resource Group:** `iatd_labs_03_rg` (re-use)
*   **Virtual Network:** `iatd_labs_03_vnet` (re-use)
*   **Subnet:** `iatd_labs_03_subnet` (re-use)
*   **VM-Target:** `iatd_labs_03_vm_target` (re-use)
*   **Public IP:** `iatd_labs_03_pip` (re-use)
*   **Web Application Firewall:** `iatd_labs_04_waf`
*   **Front Door:** `iatd_labs_04_frontdoor` (optional)

### Part 1: Exploring Azure DDoS Protection Tiers

1.  **Azure DDoS Network Protection:**
    *   Discuss the capabilities of Azure DDoS Network Protection, including always-on traffic monitoring, adaptive tuning, and rapid mitigation.
    *   Explain that it provides enhanced protection for entire virtual networks.
    *   **Note:** Enabling this typically requires a paid plan.

2.  **Azure DDoS IP Protection:**
    *   Discuss the capabilities of Azure DDoS IP Protection, including always-on traffic monitoring, and mitigation.
    *   Explain that it provides protection to single public IPs
    *   **Note:** This is free.

### Part 2: Enabling Azure DDoS IP Protection

1.  **Set Variables (from Lab 3):**

    ```bash
    RESOURCE_GROUP="iatd_labs_03_rg"
    LOCATION="eastus" # Your Region
    PUBLIC_IP_NAME="iatd_labs_03_pip"
    ```

2.  **Enable DDoS IP Protection on the Public IP (CLI):**

    ```bash
    az network public-ip update \
        --resource-group $RESOURCE_GROUP \
        --name $PUBLIC_IP_NAME \
        --ddos-protection-mode "VirtualNetworkInherited"
    ```

    **Expected Output:**
    ```json
    {
      "ddosProtectionMode": "VirtualNetworkInherited",
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
    ```

3.  **Verify DDoS IP Protection is Enabled (CLI):**

    ```bash
    az network public-ip show \
        --resource-group $RESOURCE_GROUP \
        --name $PUBLIC_IP_NAME \
        --query ddosProtectionMode -o tsv
    ```

    **Expected Output:**
    ```
    VirtualNetworkInherited
    ```

### Part 3: Implementing Azure Web Application Firewall (WAF)

1.  **Create a WAF Policy (CLI):**

    ```bash
    WAF_POLICY_NAME="iatd_labs_04_waf_policy"

    az network application-gateway waf-policy create \
        --resource-group $RESOURCE_GROUP \
        --name $WAF_POLICY_NAME \
        --location $LOCATION \
        --policy-setting mode=Prevention \
        --policy-setting state=Enabled
    ```

    **Expected Output:**
    ```json
    {
      "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/iatd_labs_03_rg/providers/Microsoft.Network/ApplicationGatewayWebApplicationFirewallPolicies/iatd_labs_04_waf_policy",
      "location": "eastus",
      "name": "iatd_labs_04_waf_policy",
      "provisioningState": "Succeeded",
      "resourceGroup": "iatd_labs_03_rg",
      "type": "Microsoft.Network/ApplicationGatewayWebApplicationFirewallPolicies"
    }
    ```

2.  **Configure WAF Policy with OWASP Rules (CLI):**

    ```bash
    az network application-gateway waf-policy managed-rule rule-set add \
        --resource-group $RESOURCE_GROUP \
        --policy-name $WAF_POLICY_NAME \
        --type OWASP \
        --version 3.2
    ```

    **Expected Output:**
    ```
    Rule set 'OWASP' version '3.2' added to policy 'iatd_labs_04_waf_policy'.
    ```

3.  **Create an Application Gateway with WAF (Portal):**
    *   Search for "Application Gateway" in the Azure Portal and select.
    *   Click **Create**.
    *   **Basics Tab:**
        *   Subscription: Your subscription.
        *   Resource group: `iatd_labs_03_rg`
        *   Application gateway name: `iatd_labs_04_waf`
        *   Region: Your region.
        *   Tier: WAF V2
        *   Enable autoscaling: No
        *   Instance count: 1
        *   Availability zone: None
    *   **Frontends Tab:**
        *   Frontend IP address type: Public
        *   Public IP address: Create new
        *   Name: `iatd_labs_04_waf_pip`
    *   **Backends Tab:**
        *   Add a backend pool:
            *   Name: `iatd_labs_04_backend_pool`
            *   Add a target: `iatd_labs_03_vm_target` (select the VM's IP address)
    *   **Configuration Tab:**
        *   Add a routing rule:
            *   Rule name: `iatd_labs_04_rule`
            *   Priority: 100
            *   Listener name: `iatd_labs_04_listener`
            *   Frontend IP: Public
            *   Protocol: HTTP
            *   Port: 80
            *   Backend targets:
                *   Target type: Backend pool
                *   Backend target: `iatd_labs_04_backend_pool`
                *   HTTP settings: Create new
                    *   HTTP settings name: `iatd_labs_04_http_settings`
                    *   Backend protocol: HTTP
                    *   Backend port: 80
                    *   Cookie-based affinity: Disable
    *   **WAF Tab:**
        *   Firewall status: Enabled
        *   Firewall mode: Prevention
        *   WAF Policy: Select the `iatd_labs_04_waf_policy` created earlier
    *   Click **Review + create**, then **Create**.

4.  **Verify Application Gateway Deployment:**

    ```bash
    az network application-gateway show \
        --resource-group $RESOURCE_GROUP \
        --name "iatd_labs_04_waf" \
        --query "{ name: name, provisioningState: provisioningState, operationalState: operationalState }" \
        -o json
    ```

    **Expected Output:**
    ```json
    {
      "name": "iatd_labs_04_waf",
      "operationalState": "Running",
      "provisioningState": "Succeeded"
    }
    ```

5.  **Get the Public IP Address of the Application Gateway:**

    ```bash
    WAF_IP=$(az network public-ip show \
        --resource-group $RESOURCE_GROUP \
        --name "iatd_labs_04_waf_pip" \
        --query ipAddress -o tsv)

    echo "Application Gateway Public IP: $WAF_IP"
    ```

    **Expected Output:**
    ```
    Application Gateway Public IP: 20.xxx.xxx.xxx
    ```

### Part 4: Testing DDoS Mitigation with WAF

1.  **Test Access to the Web Server through the WAF:**

    ```bash
    curl http://$WAF_IP
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

2.  **Simulate a Basic Attack (Safe Test):**

    ```bash
    # Attempt to access a URL that would trigger a WAF rule
    curl "http://$WAF_IP/?id=1%27%20OR%20%271%27=%271%27--"
    ```

    **Expected Output:**
    ```html
    <html>
    <head><title>403 Forbidden</title></head>
    <body>
    <center><h1>403 Forbidden</h1></center>
    <hr><center>Microsoft-Azure-Application-Gateway/v2</center>
    </body>
    </html>
    ```

    This indicates that the WAF has blocked the request due to a potential SQL injection attack.

3.  **View WAF Metrics in the Azure Portal:**
    *   Navigate to the `iatd_labs_04_waf` Application Gateway in the Azure Portal.
    *   Select **Metrics** under **Monitoring**.
    *   Add metrics for "Web Application Firewall Request Count" and "Web Application Firewall Blocked Request Count".
    *   You should see that some requests were blocked by the WAF.

4.  **Configure Custom WAF Rules (Portal):**
    *   Navigate to the `iatd_labs_04_waf_policy` WAF Policy in the Azure Portal.
    *   Select **Custom rules** under **Settings**.
    *   Click **Add custom rule**.
    *   Configure the rule:
        *   Name: `BlockSpecificIP`
        *   Priority: 100
        *   Rule type: Match
        *   Condition:
            *   Match type: IP Address
            *   IP address or range: `1.2.3.4` (example IP)
            *   Match variable: Remote Addr
            *   Operator: Equals
        *   Action: Block
    *   Click **Add**.

### Part 5: (Optional) Implementing Azure Front Door for Global DDoS Protection

1.  **Create an Azure Front Door (Portal):**
    *   Search for "Front Doors and CDN profiles" and select.
    *   Click **Create a Front Door**.
    *   **Basics Tab:**
        *   Subscription: Your subscription.
        *   Resource group: `iatd_labs_03_rg`
        *   Name: `iatd_labs_04_frontdoor`
        *   Tier: Standard
    *   **Endpoint Tab:**
        *   Add an endpoint:
            *   Name: `iatd-labs-04-endpoint`
            *   Enable this endpoint: Yes
    *   **Origin groups Tab:**
        *   Add an origin group:
            *   Name: `iatd-labs-04-origin-group`
    *   **Origins Tab:**
        *   Add an origin:
            *   Name: `iatd-labs-04-origin`
            *   Origin type: Custom
            *   Host name: Enter the public IP address of your Application Gateway
            *   HTTP port: 80
            *   HTTPS port: 443
            *   Priority: 1
            *   Weight: 1000
    *   **Routes Tab:**
        *   Add a route:
            *   Name: `iatd-labs-04-route`
            *   Domains: Select the endpoint created earlier
            *   Patterns to match: /*
            *   Accepted protocols: HTTP, HTTPS
            *   Origin group: Select the origin group created earlier
            *   Forwarding protocol: HTTP Only
    *   Click **Review + create**, then **Create**.

2.  **Configure WAF on Front Door (Portal):**
    *   Navigate to the `iatd_labs_04_frontdoor` Front Door in the Azure Portal.
    *   Select **Security** under **Settings**.
    *   Click **Create a new WAF policy**.
    *   Configure the WAF policy:
        *   Name: `iatd_labs_04_fd_waf_policy`
        *   Policy mode: Prevention
        *   Managed rules: OWASP 3.2
    *   Click **Create**.
    *   Associate the WAF policy with the Front Door endpoint.

3.  **Test Access through Front Door:**

    ```bash
    # Get the Front Door endpoint URL
    FD_URL=$(az afd endpoint show \
        --resource-group $RESOURCE_GROUP \
        --profile-name "iatd_labs_04_frontdoor" \
        --endpoint-name "iatd-labs-04-endpoint" \
        --query hostName -o tsv)

    echo "Front Door URL: $FD_URL"

    # Test access
    curl "https://$FD_URL"
    ```

    **Expected Output:**
    ```html
    <!DOCTYPE html>
    <html>
    <head>
    <title>Welcome to nginx!</title>
    ...
    </html>
    ```

### Post-Lab: Cleanup

To avoid incurring unnecessary costs, it's essential to clean up the resources created during this lab.

1.  **Delete the Application Gateway:**

    ```bash
    az network application-gateway delete \
        --resource-group $RESOURCE_GROUP \
        --name "iatd_labs_04_waf" \
        --no-wait
    ```

    **Expected Output:**
    No output is expected. The command will start the deletion process in the background.

2.  **Delete the WAF Policy:**

    ```bash
    az network application-gateway waf-policy delete \
        --resource-group $RESOURCE_GROUP \
        --name $WAF_POLICY_NAME
    ```

    **Expected Output:**
    No output is expected if the deletion is successful.

3.  **Delete Front Door (if created):**

    ```bash
    az afd profile delete \
        --resource-group $RESOURCE_GROUP \
        --profile-name "iatd_labs_04_frontdoor"
    ```

    **Expected Output:**
    No output is expected if the deletion is successful.

4.  **Delete Resource Group:**

    ```bash
    az group delete --name $RESOURCE_GROUP --yes
    ```

    **Expected Output:**
    The command will not produce immediate output but will start the deletion process. You can verify the deletion is in progress by checking the resource group list:

    ```bash
    az group list --query "[?name=='$RESOURCE_GROUP']" -o table
    ```

    The resource group should eventually disappear from the list, indicating successful deletion. This process may take several minutes to complete.

5.  **Verify Cleanup:**

    To ensure all resources have been properly cleaned up, verify that the resource group no longer exists:

    ```bash
    az group show --name $RESOURCE_GROUP
    ```

    **Expected Output:**
    ```
    Resource group 'iatd_labs_03_rg' could not be found.
    ```

**Outcomes to be Achieved:**

*   Understand the capabilities of Azure DDoS Network Protection and Azure DDoS IP Protection.
*   Enable Azure DDoS IP Protection for a Public IP address.
*   Implement Azure Web Application Firewall (WAF) to protect against application-layer attacks.
*   Configure WAF rules and custom rules to filter malicious traffic.
*   Observe WAF logs to identify and analyze blocked requests.
*   (Optional) Implement Azure Front Door for global DDoS protection.

**Congratulations!** You have successfully completed this lab on advanced DDoS mitigation techniques in Azure. You've learned how to implement and configure Azure DDoS Protection and Web Application Firewall to protect your applications from various types of DDoS attacks.
