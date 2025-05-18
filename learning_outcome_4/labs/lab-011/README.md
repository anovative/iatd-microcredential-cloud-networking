## IATD Microcredential Cloud Networking: Lab 11 - Designing and Configuring Secure Service Integration Solutions

**Objective:** This lab focuses on designing and implementing secure service integration solutions in Azure. You'll learn how to choose appropriate communication patterns, select network integration components, implement service discovery, secure integrations, and configure service access using both public and private endpoints.

**Estimated Time:** 90 - 120 minutes

**Prerequisites:**

1.  **Azure Subscription:** Requires an active Azure subscription. A free account is available at [https://azure.microsoft.com/free/](https://azure.microsoft.com/free/).
2.  **Basic knowledge of Azure Networking:** Familiarity with Virtual Networks and Subnets.
3.  **Familiarity with Azure Services:** Basic understanding of Azure App Service, Azure Functions, Azure Service Bus, and Azure API Management is helpful.
4.  **Existing Azure Services:** You'll need two existing Azure App Services or Azure Functions to simulate services that need to be integrated.
5.  **Existing Azure Key Vault:** You will need an existing Azure Key Vault.

### Lab Conventions

*   **Azure Cloud Shell:** All PowerShell and CLI interactions will occur within the Azure Cloud Shell environment.
*   **Azure Portal:** The Azure Portal will be used for visualization and configuration tasks.
*   **Naming Conventions:** Resource names follow the convention `iatd_labs_11_*`.
*   **IP Address Range:** The `172.16.0.0/16` range will be used (as per standardization).
*   **Location:** Choose a consistent Azure region.

#### Resource Naming

*   **Resource Group:** `iatd_labs_11_rg`
*   **Virtual Network:** `iatd_labs_11_vnet`
*   **Subnet-Services:** `iatd_labs_11_subnet_services` (172.16.0.0/24)
*   **Subnet-Integration:** `iatd_labs_11_subnet_int` (172.16.1.0/24)
*   **App Service 1:** (Use the name of your existing App Service/Function)
*   **App Service 2:** (Use the name of your existing App Service/Function)
*   **API Management Service:** `iatd_labs_11_apim`
*   **Key Vault:** (Use the name of your existing keyvault)
*   **Private Endpoint (APIM):** `iatd_labs_11_apim_pe`

### Part 1: Designing Service Integration

1.  **Choosing Communication Patterns (Conceptual):**

    *   **Synchronous (Sync):** Request/Response pattern (e.g., HTTP calls). Suitable for real-time interactions where a response is immediately required.
    *   **Asynchronous (Async):** Message-based pattern (e.g., using Azure Service Bus). Suitable for decoupling services, improving scalability, and handling intermittent connectivity.
    *   **For this lab, let's assume you'll use HTTP for synchronous communication between App Service 1 and App Service 2.**

2.  **Selecting Network Integration Components:**

    *   **API Gateway (Azure API Management):** Provides a single point of entry for your services, enabling routing, security, and monitoring.
    *   **Message Broker (Azure Service Bus):** Enables asynchronous communication between services.
    *   **For this lab, you'll use Azure API Management (APIM) to expose App Service 1 and control access to App Service 2.**

3.  **Implementing Dynamic Service Discovery (Conceptual):**

    *   Discuss that in dynamic environments, services can be registered with a service discovery mechanism (e.g., Consul, etcd, or Azure DNS) so that other services can automatically discover their endpoints. For the purpose of this lab, we will use manually configuration, because dynamic service discovery is beyond of scope of this lab.

4.  **Securing Integrations (Conceptual):**

    *   **Authentication (mTLS):** Mutual TLS authentication ensures that both the client and server verify each other's identities using certificates. Not in scope for this lab due complexity

    *   **Authorisation (RBAC):** Role-Based Access Control (RBAC) allows you to define roles and permissions to control access to resources.
        * In the lab we will use APIM to implement this.

    *   **Encryption:** Use HTTPS to encrypt traffic in transit.

### Part 2: Configuring Service Access

1.  **Create Virtual Network and Subnets:**

    ```bash
    RESOURCE_GROUP="iatd_labs_11_rg"
    LOCATION="eastus" # Your Region
    VNET_NAME="iatd_labs_11_vnet"
    SUBNET_SERVICES="iatd_labs_11_subnet_services"
    SUBNET_INT="iatd_labs_11_subnet_int"
    ADDRESS_PREFIX="172.16.0.0/16"
    SUBNET_SERVICES_PREFIX="172.16.0.0/24"
    SUBNET_INT_PREFIX="172.16.1.0/24"

    # Create Resource Group
    az group create --name $RESOURCE_GROUP --location $LOCATION

    # Create VNet with first subnet
    az network vnet create \
        --resource-group $RESOURCE_GROUP \
        --name $VNET_NAME \
        --address-prefixes $ADDRESS_PREFIX \
        --subnet-name $SUBNET_SERVICES \
        --subnet-prefixes $SUBNET_SERVICES_PREFIX

    # Add integration subnet
    az network vnet subnet create \
        --resource-group $RESOURCE_GROUP \
        --vnet-name $VNET_NAME \
        --name $SUBNET_INT \
        --address-prefix $SUBNET_INT_PREFIX
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
        "subnets": [
          {
            "addressPrefix": "172.16.0.0/24",
            "name": "iatd_labs_11_subnet_services"
          }
        ],
        "name": "iatd_labs_11_vnet",
        "resourceGroup": "iatd_labs_11_rg"
      }
    }
    ```

2.  **Create and Configure Azure API Management Service (APIM) (Portal):**
    *   Search for "API Management services" and select.
    *   Click **Create**.
        *   **Basics Tab:**
            *   Subscription: Your subscription.
            *   Resource group: `iatd_labs_12_rg`
            *   Name: `iatd_labs_12_apim`
            *   Region: Your region.
            *   Organization name: Your organization name
            *   Administrator email: Your email address
            *   Pricing tier: Developer
        *   **Networking Tab:**
            *   Connectivity type: Internal
            *   Location: choose the virtual network and the services subnet `iatd_labs_12_subnet_services`
    *   Click **Review + create**, then **Create**.

3.  **Configure APIM to Access App Service 2 (Portal):**

    *   After the APIM service is deployed, navigate to it in the Azure Portal.
    *   Select **APIs** under **APIs**.
    *   Click **Add API**
    * Select "HTTP"
    * Fill the form and select "Create"
        * Web service URL: Enter the URL of `App Service 2`.
        * API URL suffix: a desired suffix.
        * Display Name: name of your service
    * Add the API
    *  Click **All operations**
    *  Click "+ Add operation"

4.  **Secure Integration with Authentication and Authorisation (Portal):**
    *   In the APIM service, select **APIs** and then the API you just created.
    *   Select **Settings**.
        * Select Authentication
        * In the Inbound Processing tap select Validate JWT:
            * Select: New JWT Policy
            * Authorisation server: Create a new configuration to add manual key information

    *Select Access control (IAM)
        * Configure new rules and access to manage authentication
        *  Set up RBAC to control which users or groups can access the API.

### Part 5: Implement KeyVault Authentication

1.  **Create a user-assigned managed identity.(Portal):**
    *   Navigate to Managed Identities
    *   Click Create
    * Select a name and location

2.  **Assign permissions to the API management instance to read secrets from Key Vault (Portal):**
    *   Navigate to your Key Vault in the Azure Portal.
    *   Select **Access configuration** under **Settings**, and ensure that the permission model is set to *Azure role-based access control*.
    *   Select **Access control (IAM)**, click **Add role assignment**, and assign the *Key Vault Secrets User* role to the new managed identity.
        * Select Managed Identity Tab
        * Select the subscription and Managed Identity that was created
    * Click Create.

3.  **Create or update the Api Management instance, and enable the system-assigned managed identity. (CLI):**

    ```bash
    APIM_NAME="iatd_labs_11_apim"

    az apim update --name $APIM_NAME -g $RESOURCE_GROUP --identity-type SystemAssigned,UserAssigned --user-assigned-identity  "/subscriptions/<Subscription Id>/resourcegroups/<Resource Group Name>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<User Assigned Identity Name>"
    ```

4.  **Set authorization-identity policy (Portal):**
    *   Navigate to your APIM instance
        *   Select API's
            *Select Code view
            * You can now set up the keyValut Referencing Policy to the JWT Validation in your Api Management

5.  **Setting Private endpoint for APIM:**
     ```bash
    APIM_NAME="iatd_labs_11_apim"
    SUBNET_SERVICES="iatd_labs_11_subnet_services"
    RG_NAME="iatd_labs_12_rg"
    LOC_NAME="eastus"
    az network private-endpoint create   --connection-name apim-sql-connection   --name apim-private-endpoint   --private-connection-resource-id /subscriptions/<subscription ID>/resourceGroups/<resource group name>/providers/Microsoft.ApiManagement/service/<apim service name>   --group-id apim   --vnet-name $APIM_NAME   --subnet $SUBNET_SERVICES   --resource-group $RG_NAME -l $LOC_NAME

    az network private-dns zone create -g $RG_NAME -n privatelink.azure-api.net
    ```

### Post-Lab: Cleanup

To avoid incurring unnecessary costs, it's essential to clean up the resources created during this lab.

1.  **Delete Resource Group:**

    ```bash
    az group delete --name $RESOURCE_GROUP --yes
    ```

**Learning Outcomes:**

*   Understood how to choose appropriate communication patterns (sync vs. async) for service integration.
*   Successfully selected and configured Azure API Management (APIM) as a network integration component.
*   Understood the concepts of dynamic service discovery, and apply them with a code sample
*   Successfully secured service integration using authentication, authorisation, and encryption techniques.
*   Successfully created an Azure API Management service and configured it to access other Azure services.
*   Successfully use RBAC to implement an authoritazion-identity policy
*   Successfully use PE to create endpoints and DNS
