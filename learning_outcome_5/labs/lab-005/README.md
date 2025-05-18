## IATD Microcredential Cloud Networking: Lab 5 - Global Load Balancing with Azure Front Door

**Objective:** This lab introduces global load balancing using Azure Front Door. You'll learn how to configure Front Door to distribute traffic across multiple regions, improve application availability, optimize performance for global users, implement geo-filtering, and configure custom domains.

**Estimated Time:** 90 - 120 minutes

**Prerequisites:**

1.  **Azure Subscription:** Requires an active Azure subscription. A free account is available at [https://azure.microsoft.com/free/](https://azure.microsoft.com/free/).
2.  **Basic knowledge of Azure Networking:** Familiarity with Virtual Networks, Subnets, Load Balancers, and DNS.
3.  **Existing Web Applications:** You'll need at least two web applications (or VMs running web servers with distinct content) deployed in *different Azure regions*. This is crucial for demonstrating Front Door's global capabilities.
4.   **A valid DNS Name and SSL Certificate**: You will need a valid certificate to enable HTTPS, please see below all options

### Lab Conventions

*   **Azure Cloud Shell:** All PowerShell and CLI interactions will occur within the Azure Cloud Shell environment.
*   **Azure Portal:** The Azure Portal will be used for visualization and configuration tasks.
*   **Naming Conventions:** Resource names follow the convention `iatd_labs_05_*`.
*   **Location:** Resources will be deployed across multiple Azure regions.

#### Resource Naming

*   **Resource Group:** `iatd_labs_05_rg`
*   **Front Door:** `iatd_labs_05_frontdoor`
*   **Origin Group:** `iatd_labs_05_origin_group`
*   **Web App - Region 1:** `iatd_labs_05_webapp_us` (Example - Replace with your actual web app name)
*   **Web App - Region 2:** `iatd_labs_05_webapp_eu` (Example - Replace with your actual web app name)
*   **Custom Domain:** `www.yourdomain.com` (Replace with your actual domain)

### Part 1: Setting up Web Applications in Different Regions

1.  **Verify Existing Web Applications:**
    *   Ensure that you have two web applications (or VMs running web servers) deployed in different Azure regions (e.g., East US and West Europe).
    *   Note the public IP addresses or hostnames of these web applications. These will be your origins for Azure Front Door.

2.  **Customize Web Application Content:**

    *   Customize the content of each web application to clearly identify which region is serving the content (e.g., "Web App in East US" and "Web App in West Europe").

### Part 2: Deploying and Configuring Azure Front Door

1.  **Create an Azure Front Door (Portal):**
    *   Search for "Front Doors and CDN profiles" and select.
    *   Click **Create a Front Door**.
    *   Quick Create
        * Select DNS Name as `www.yourdomain.com`
        * Origin type: App Services
        * Select `iatd_labs_05_webapp_us`
     * Advanced Configurations requires :
        * Frontends/domains: Add DNS Name as "`www.yourdomain.com`"
        * Origin groups:
            * Add one first to the webapp in US
            * Add another to the webapp in EU
        * Rules: Add 2 routing rules
            * First One - WebApp-US
                 * Patterns to match: ["/*"]
                 * Route Details - Select US webapp backend
            * Second One - WebApp-EU
                 * Patterns to match: ["/*"]
                 * Route Details - Select EU webapp backend
       *Click "Save" in all

### Part 3: Configuring Routing Rules and Health Probes

1.  **Configure Routing Rules (Portal):**

    *   Navigate to your Azure Front Door (`iatd_labs_05_frontdoor`) in the Azure Portal.
    *Go to Origin Group and Configure Health Probes
    *   Update the Health PROBE configurations
            *   Path: `/`
            *   Protocol: `HTTP`
        *   *Click Save*
    * The Health Probes should look healthy in a few minutes

### Part 4: Implementing Geo-filtering (Optional)

1.  **Create a Custom Rule:**

    * In Security tab, select **+ Add a rule**.
    *   Give the rule a name, for example *Block-Africa*.
    * Add Rule Condition by:
        *   Match type: Is GeoMatch
        *   Operator: is
        *   Match value: Africa
    * Define that action as block
### Part 5: Add Valid DNS Name and Certificates* (Optional)

1.  **To add a Custom domain** You need to own a valid certificate, it can be provided from multiple ways:
    *   You can buy directly
    * Use Let's Encrypt.
*Select Security Tab, Certificate Management, after the process the DNS should point directly to your APIM domain name
**Validate that it´s propogated and working*
### Part 6: Testing Global Traffic Distribution and Access

1.  **Accessing the Application:**

    *   Get the Front Door hostname from the Azure Portal:
        * Navigate to your Front Door resource
        * Copy the Front Door endpoint URL (e.g., `iatd-labs-05-frontdoor.azurefd.net`)
    
    *   Test global routing:
        * Open a web browser and navigate to the Front Door's hostname or your custom domain
        * Refresh the page multiple times to observe which backend is serving content
        * Use a command line tool to validate which backend is being used:

        ```bash
        # Run traceroute to see the network path
        traceroute <front-door-hostname>
        
        # Or use curl with verbose output to see response headers
        curl -v <front-door-hostname>
        ```
    
    Expected outcome:
    ```
    - The web application should load successfully through the Front Door endpoint
    - Response headers may show which region is serving the content
    - Users from different geographic locations should be routed to their closest backend
    - If one backend is down, traffic should automatically route to the healthy backend
    ```
    
    Validation tips:
    ```
    - Use a VPN to simulate accessing from different regions
    - Check the Azure Front Door metrics in the portal to see traffic distribution
    - Temporarily disable one backend to verify failover functionality
    ```

### Post-Lab: Cleanup

To avoid incurring unnecessary costs, it's essential to clean up the resources created during this lab.

1.  **Delete Resource Group:**

    ```bash
    az group delete --name $RESOURCE_GROUP --yes
    ```

**Learning Outcomes:**

*   Successfully deployed and configured Azure Front Door for global load balancing.
*   Successfully configured origin groups and routing rules to distribute traffic across multiple regions.
*   Optionally, you can secure access to the server and custom names*
*   Verified that traffic is routed to the closest available backend, improving performance.
*   Understood the benefits of using Azure Front Door for global traffic management, high availability, and optimized performance.
