## IATD Microcredential Cloud Networking: Lab 3 - Advanced Load Balancing with Application Gateway - Part 1

**Objective:** This lab introduces Azure Application Gateway. You'll learn about its features, use cases, and how it differs from the Azure Load Balancer. You'll also configure SSL offloading and certificate management to secure your web applications.

**Estimated Time:** 90 - 120 minutes

**Prerequisites:**

1.  **Azure Subscription:** Requires an active Azure subscription. A free account is available at [https://azure.microsoft.com/free/](https://azure.microsoft.com/free/).
2.  **Basic knowledge of Azure Networking:** Familiarity with Virtual Networks and Subnets.
3.  **Completion of Lab 2:** Assumes knowledge of basic load balancing concepts and Azure Load Balancer configuration.
4.  **Existing Web Application:** You will need an existing web application or two virtual machines running web servers to act as backend servers.
5.   **A valid SSL Certificate**: You will need a valid certificate to enable HTTPS (see below for options).

### Lab Conventions

*   **Azure Cloud Shell:** All PowerShell and CLI interactions will occur within the Azure Cloud Shell environment.
*   **Azure Portal:** The Azure Portal will be used for visualization and configuration tasks.
*   **Naming Conventions:** Resource names follow the convention `iatd_labs_03_*`.
*   **Location:** Choose a consistent Azure region.

#### Resource Naming

*   **Resource Group:** `iatd_labs_03_rg`
*   **Virtual Network:** `iatd_labs_03_vnet`
*   **Subnet-AppGateway:** `iatd_labs_03_subnet_agw`
*   **Subnet-Backend:** `iatd_labs_03_subnet_backend`
*   **Application Gateway:** `iatd_labs_03_appgw`
*   **Public IP:** `iatd_labs_03_agw_pip`
*   **Backend VM 1:** `iatd_labs_03_vm_backend1`
*   **Backend VM 2:** `iatd_labs_03_vm_backend2`

### Part 1: Setting up the Virtual Network and Backend Servers

1.  **Ensure a Resource Group Exists:**

    ```bash
    RESOURCE_GROUP="iatd_labs_03_rg"
    LOCATION="eastus" # Your Region

    az group create --name $RESOURCE_GROUP --location $LOCATION --output json --only-show-errors
    ```

2.  **Ensure a Virtual Network with Subnets Exists:**

    ```bash
    VNET_NAME="iatd_labs_03_vnet"
    SUBNET_AGW_NAME="iatd_labs_03_subnet_agw"
    SUBNET_BACKEND_NAME="iatd_labs_03_subnet_backend"
    ADDRESS_PREFIX="172.16.0.0/16"
    SUBNET_AGW_PREFIX="172.16.1.0/24"
    SUBNET_BACKEND_PREFIX="172.16.2.0/24"

    az network vnet create \
        --resource-group $RESOURCE_GROUP \
        --name $VNET_NAME \
        --address-prefixes $ADDRESS_PREFIX \
        --location $LOCATION \
        --subnet-name $SUBNET_AGW_NAME \
        --subnet-prefixes $SUBNET_AGW_PREFIX

    az network vnet subnet create \
        --resource-group $RESOURCE_GROUP \
        --vnet-name $VNET_NAME \
        --name $SUBNET_BACKEND_NAME \
        --address-prefix $SUBNET_BACKEND_PREFIX
    ```

3.  **Ensure two Backend VMs exist**

### Part 2: Deploying Azure Application Gateway

1.  **Create a Public IP Address for the Application Gateway:**

    ```bash
    PUBLIC_IP_NAME="iatd_labs_03_agw_pip"

    az network public-ip create \
        --resource-group $RESOURCE_GROUP \
        --name $PUBLIC_IP_NAME \
        --allocation-method Static \
        --sku Standard \
        --location $LOCATION
    ```

2.  **Deploy Azure Application Gateway (Portal):**

    *   Search for "Application Gateways" and select.
    *   Click **Create**.
        *   **Basics Tab:**
            *   Subscription: Your subscription.
            *   Resource group: `iatd_labs_03_rg`
            *   Application gateway name: `iatd_labs_03_appgw`
            *   Region: Your region.
            *   Tier: Standard
            *   Virtual network: Select `iatd_labs_03_vnet`
            *   Subnet: Select `iatd_labs_03_subnet_agw`
            *   Frontend IP configuration: Public
            *   Public IP address: Select `iatd_labs_03_agw_pip`
        *   **Backends Tab:**
            * Add a Backend Pool, then add the 2 IPs as Backend targets
        *   **Configuration Tab:**
            * Listener:
                *Protocol: HTTP
            * Backend Settings:
                * Protocol: HTTP
            * Routing rule:
                * Backend targets - Select your previous Backend Pool and select HTTP settings
    * Click **Review + Create** and then **Create**

### Part 3: Understanding Key Application Gateway Features

1.  **Layer 7 Load Balancing:**
    *   Explain that Application Gateway operates at Layer 7 (HTTP/HTTPS) and can make routing decisions based on URL paths, hostnames, and other application-specific information.
    *   Compare and contrast this with the Azure Load Balancer, which operates at Layer 4 (TCP/UDP) and uses IP addresses and ports for routing.
    This will be validated on lab 4

2.  **Web Application Firewall (WAF):**
    *   Explain that Application Gateway can be integrated with WAF to protect against common web exploits, such as SQL injection and cross-site scripting (XSS). This will be validated on lab 4
    *   Discuss the different WAF modes (Detection vs. Prevention) and rule sets.

3.  **Session Affinity (Cookie-based Affinity):**
  * Will be explain on lab 4
   * The backend target persistency will be enabled on lab 4

### Part 4: Configuring SSL Offloading and Certificate Management

1.  **Obtain an SSL Certificate:**
Skip this test if you didn´t have an valid certificate for testing*
    *  You need a valid SSL certificate for your domain. You can:
        *   Purchase a certificate from a Certificate Authority (CA).
        *   Use a self-signed certificate for testing purposes * (Not recommended for production environments)*.
    * Keyvault can be used to manage and store SSL
2.  **Configure HTTPS Listener:**
    *This requires an advanced setup with valid domain name and a configured web server.

    *   Navigate to your Application Gateway (`iatd_labs_03_appgw`) in the Azure Portal.
    *   Select **Listeners** under **Settings**.
    *   Click **Add listener**.
        *   Listener name: `httpsListener`
        *   Frontend IP address: Select `iatd_labs_03_agw_pip`
        *   Protocol: `HTTPS`
        *   Port: `443`
        *   Choose a certificate: Upload the .PFX certificate and provide the password.
    *Click "Save"

3.  **Configure Routing Rule to Use HTTPS Listener:**

    * Select Basic at Add Rule
    * Select previous listener
    * Select the Backend Pool

4.  **Testing HTTPS Access:**
    * This requires an advanced setup with valid domain name and a configured web server.
    *   Open a web browser and navigate to the DNS name of your Azure Application Gateway.
    *   Verify that you are able to access your web application over HTTPS.
    *   Check the certificate details in your browser to confirm that the SSL certificate is valid and being used correctly.

    Expected outcome:
    ```
    - Browser should show a secure connection (lock icon in address bar)
    - Certificate information should match what was uploaded to Application Gateway
    - Web application content should load correctly over HTTPS
    - No certificate errors or warnings should appear
    ```

### Post-Lab: Cleanup

To avoid incurring unnecessary costs, it's essential to clean up the resources created during this lab.

1.  **Delete Resource Group:**

    ```bash
    az group delete --name $RESOURCE_GROUP --yes
    ```

**Learning Outcomes:**

*   Successfully deployed an Azure Application Gateway.
*   Understood the key features and use cases of Application Gateway.
*   Learned how to configure SSL offloading to secure web applications.
*   Learned how to configure keyvault to store certificates.
*   You can configure basic web applications through APIM, if configured on you backend.
