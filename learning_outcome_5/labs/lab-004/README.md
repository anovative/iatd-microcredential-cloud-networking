## IATD Microcredential Cloud Networking: Lab 4 - Advanced Load Balancing with Application Gateway - Part 2

**Objective:** This lab builds upon the knowledge gained in Lab 3, focusing on configuring traffic distribution rules in Azure Application Gateway. You'll implement path-based routing and host-based routing to direct traffic to different backend pools based on URL paths or hostnames, enabling more complex application delivery scenarios. You'll also explore session affinity options and WAF capabilities.

**Estimated Time:** 90 - 120 minutes

**Prerequisites:**

1.  **Azure Subscription:** Requires an active Azure subscription. A free account is available at [https://azure.microsoft.com/free/](https://azure.microsoft.com/free/).
2.  **Solid knowledge of Azure Networking:** Familiarity with Virtual Networks and Subnets.
3.  **Completion of Lab 3:** Assumes a working Application Gateway deployment with basic HTTP/HTTPS configuration. You should have the ability to configure certificates and DNS
4.  **Two Web Applications or VMs:** You'll need two web applications (or VMs running web servers with distinct content) to act as different backend pools. Each with different content

### Lab Conventions

*   **Azure Cloud Shell:** All PowerShell and CLI interactions will occur within the Azure Cloud Shell environment.
*   **Azure Portal:** The Azure Portal will be used for visualization and configuration tasks.
*   **Naming Conventions:** Resource names follow the convention `iatd_labs_04_*`.
*   **Location:** Choose a consistent Azure region.

#### Resource Naming

*   **Resource Group:** `iatd_labs_03_rg` (Re-use from Lab 3)
*   **Virtual Network:** `iatd_labs_03_vnet` (Re-use from Lab 3)
*   **Subnet-AppGateway:** `iatd_labs_03_subnet_agw` (Re-use from Lab 3)
*   **Application Gateway:** `iatd_labs_03_appgw` (Re-use from Lab 3)
*   **Public IP:** `iatd_labs_03_agw_pip` (Re-use from Lab 3)
*   **Backend VM 1:** `iatd_labs_03_vm_backend1` (Re-use from Lab 3)
*   **Backend VM 2:** `iatd_labs_03_vm_backend2` (Re-use from Lab 3)
*   **Backend Pool - Images:** `iatd_labs_04_backend_images`
*   **Backend Pool - Videos:** `iatd_labs_04_backend_videos`

### Part 1: Setting up Backend Applications

1.  **Customize Web Application Content:**

    *   Connect to `iatd_labs_03_vm_backend1` and edit the default Nginx page (`/var/www/html/index.nginx-debian.html`) to display "Images Backend".
    *   Connect to `iatd_labs_03_vm_backend2` and edit the default Nginx page to display "Videos Backend".

2.  **Verify Basic Access:**

    *   Access the Application Gateway's public IP address and confirm that you can reach either the "Images Backend" or "Videos Backend" (depending on your default routing rule).

### Part 2: Configuring Path-Based Routing

1.  **Create Backend Pools for Images and Videos:**

    *   Navigate to your Application Gateway (`iatd_labs_03_appgw`) in the Azure Portal.
    *   Select **Backend pools** under **Settings**.
    *   Click **Add**.
        *   Name: `iatd_labs_04_backend_images`
        *   Virtual network: Select `iatd_labs_03_vnet`
        *  Add the VM IP for backend 1
    *   Click **Add**.
        *   Name: `iatd_labs_04_backend_videos`
        *   Virtual network: Select `iatd_labs_03_vnet`
    * Add the VM IP for backend 2

2.  **Add a Listener for Path-Based Routing:**
     * Select **Listeners** under **Settings**.
    * Add a new listener
     *Listener name: `pathBasedListener`
            *   Frontend IP address: Select `iatd_labs_03_agw_pip`
            *   Protocol: `HTTP`
            *   Port: `80`

3.  **Create Routing Rules (Path-Based) (Portal):**

    *   Select **Rules** under **Settings**.
    *   Click **Add rule Collection**.
        *   Name: PathBasedRule
        *   Listener: Select the `pathBasedListener`
        *   Associated Pools :
             *Path: `/images*`, Backend Pool `iatd_labs_04_backend_images`
             *Path: `/videos*`, Backend Pool `iatd_labs_04_backend_videos`

### Part 3: Testing Path-Based Routing

1.  **Accessing the Application with Different Paths:**

    *   Get the Public IP address of the Application Gateway (`iatd_labs_03_agw_pip`).
    *   Open a web browser and navigate to the following URLs:
        *   `http://<your_app_gateway_ip>/images` - Verify this routes to the "Images Backend".
        *   `http://<your_app_gateway_ip>/videos` - Verify this routes to the "Videos Backend".
    
    Expected outcome:
    ```
    - When accessing /images path: You should see the content from VM1 with "Images Backend" text
    - When accessing /videos path: You should see the content from VM2 with "Videos Backend" text
    - The same Application Gateway IP address should route to different backends based solely on the URL path
    ```
    
    Troubleshooting tips:
    ```
    - If both paths route to the same backend, verify your path-based routing rules are correctly configured
    - If you get 404 errors, ensure your backend servers are properly serving content
    - Check Application Gateway health probes to ensure both backends are healthy
    ```
    
    *You can test from the internet or from an internal virtual machine to verify the routing is working correctly*

### Part 4: Testing Traffic Flow

1.  **Test Basic Routing**
          *   Get the Public IP address of the Application Gateway (`iatd_labs_03_agw_pip`).
    *   Open a web browser and navigate to the Load Balancer's Public IP address.
    *   You should see the web page for app, if not test again.

### Post-Lab: Cleanup

To avoid incurring unnecessary costs, it's essential to clean up the resources created during this lab.

1.  **Delete Resource Group:**

    ```bash
    az group delete --name $RESOURCE_GROUP --yes
    ```

**Learning Outcomes:**

*   Successfully configured path-based routing in Azure Application Gateway.
*   Successfully verified that traffic is routed to the correct backend pools based on URL paths.
*   Understood how to use path-based routing to create more complex application delivery scenarios.
