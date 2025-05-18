## IATD Microcredential Cloud Networking: Lab 8 - Creating and Configuring an Azure DNS Zone

**Objective:** In this lab, you will learn how to create an Azure DNS zone, add various types of DNS records, and delegate the zone to Azure DNS.

**Estimated Time:** 60 - 75 minutes

**Prerequisites:**

1.  **Azure Subscription:** You need an active Azure subscription.
2.  **Domain Name:** You need to own a domain name or have access to manage its DNS records at your domain registrar.
3.  **Azure Cloud Shell:** This lab utilizes the Azure Cloud Shell.

**Lab Conventions:**

*   **Naming Conventions:** All resources created in this lab will be prefixed with `iatd_labs_08` to ensure easy identification and cleanup.
*   **IP Address Range:** The `172.16.0.0/16` range will be used (as per standardization) for any virtual networks created.
*   **Location:** Choose a consistent Azure region.

**Let's get started!**

### Part 1: Create an Azure DNS Zone

We will create a DNS zone using the Azure portal, PowerShell, and CLI to showcase the different methods.

**Method 1: Azure Portal**

1.  **Log into the Azure Portal:** Go to [https://portal.azure.com/](https://portal.azure.com/) and sign in to your account.

2.  **Create DNS Zone:**

    *   Search for "DNS zones" in the Azure portal and select "DNS zones".
    *   Click "+ Create".
    *   In the "Create DNS zone" blade:
        *   **Subscription:** Select your Azure subscription.
        *   **Resource group:** Create a new resource group named `iatd_labs_08_rg` and select location `australiaeast`.
        *   **Name:** Enter your domain name prefixed with the lab prefix (e.g., `iatd_labs_08.yourdomain.com`). **Important:** Replace `yourdomain.com` with a domain you own.
        *   Click "Review + create", then click "Create".

**Method 2: Azure PowerShell**

1.  **Open Azure Cloud Shell:** Go to the Azure portal and launch Cloud Shell (PowerShell).

2.  **Create DNS Zone:** Execute the following commands in Cloud Shell:

    ```powershell
    # Provide your domain name and desired resource group name
    $resourceGroupName = "iatd_labs_08_rg"  #Use existing RG
    $dnsZoneName = "iatd_labs_08.yourdomain.com" # Replace yourdomain.com
    $location = "australiaeast"

    # Check if the resource group exists, if not, create one
    if (!(Get-AzResourceGroup -Name $resourceGroupName -ErrorAction SilentlyContinue)) {
      New-AzResourceGroup -Name $resourceGroupName -Location $location
    }

    # Create the DNS zone
    New-AzDnsZone -Name $dnsZoneName -ResourceGroupName $resourceGroupName -Location $location
    ```

    **Example Output:**

    ```
    Name              : iatd_labs_08.yourdomain.com
    ResourceGroupName : iatd_labs_08_rg
    Location          : australiaeast
    Etag              : "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
    NameServers       : {ns1-01.azure-dns.com., ns2-01.azure-dns.net.,
                          ns3-01.azure-dns.org., ns4-01.azure-dns.info.}
    ```

**Method 3: Azure CLI**

1.  **Open Azure Cloud Shell:** Go to the Azure portal and launch Cloud Shell (Bash).

2.  **Create DNS Zone:** Execute the following commands in Cloud Shell:

    ```azurecli
    # Provide your domain name
    DOMAIN_NAME="iatd_labs_08.yourdomain.com" # Replace yourdomain.com
    RESOURCE_GROUP="iatd_labs_08_rg" # Use existing RG
    LOCATION="australiaeast"

    az network dns zone create \
      --resource-group $RESOURCE_GROUP \
      --name $DOMAIN_NAME \
      --location $LOCATION
    ```

    **Example Output:**

    ```json
    {
      "etag": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "id": "/subscriptions/YOUR_SUBSCRIPTION_ID/resourceGroups/iatd_labs_08_rg/providers/Microsoft.Network/dnszones/iatd_labs_08.yourdomain.com",
      "location": "australiaeast",
      "maxNumberOfRecordSets": 10000,
      "name": "iatd_labs_08.yourdomain.com",
      "nameServers": [
        "ns1-01.azure-dns.com.",
        "ns2-01.azure-dns.net.",
        "ns3-01.azure-dns.org.",
        "ns4-01.azure-dns.info."
      ],
      "numberOfRecordSets": 2,
      "resourceGroup": "iatd_labs_08_rg",
      "tags": {},
      "type": "Microsoft.Network/dnsZones"
    }
    ```

### Part 2: Add DNS Records

We will add A, CNAME, MX, and TXT records to the zone. We'll use a mix of methods for variety.

**A Record:**

*   Associate `www.iatd_labs_08.yourdomain.com` with an IP address.

**Method: Azure Portal**

1.  **Navigate to your DNS Zone:** In the Azure portal, go to the DNS zone you created (e.g., `iatd_labs_08.yourdomain.com`).

2.  **Add Record Set:**

    *   Click "+ Record set".
    *   **Name:** Enter `www`.
    *   **Type:** Select "A".
    *   **TTL:** Enter `3600` (seconds).
    *   **IP address:** Enter `203.0.113.10` (a sample IP address – use a real one if you have a server).
    *   Click "OK".

**CNAME Record:**

*   Create a CNAME record for `alias.iatd_labs_08.yourdomain.com` pointing to `www.iatd_labs_08.yourdomain.com`.

**Method: Azure CLI**

1.  **Open Azure Cloud Shell:** Go to the Azure portal and launch Cloud Shell (Bash).

2.  **Add CNAME Record:**

    ```azurecli
    DOMAIN_NAME="iatd_labs_08.yourdomain.com" # Replace yourdomain.com
    RESOURCE_GROUP="iatd_labs_08_rg"
    ALIAS_NAME="alias"

    az network dns record-set cname create \
      --resource-group $RESOURCE_GROUP \
      --zone-name $DOMAIN_NAME \
      --name $ALIAS_NAME

    az network dns record-set cname set-record \
      --resource-group $RESOURCE_GROUP \
      --zone-name $DOMAIN_NAME \
      --record-set-name $ALIAS_NAME \
      --cname www.iatd_labs_08.yourdomain.com
    ```

**MX Record:**

*   Create an MX record for `iatd_labs_08.yourdomain.com` pointing to `mail.yourdomain.com` with a preference of 10.

**Method: Azure PowerShell**

1.  **Open Azure Cloud Shell:** Go to the Azure portal and launch Cloud Shell (PowerShell).

2.  **Add MX Record:**

    ```powershell
    $resourceGroupName = "iatd_labs_08_rg"
    $dnsZoneName = "iatd_labs_08.yourdomain.com" # Replace yourdomain.com

    $mxRecord = New-AzDnsRecordConfig -Exchange "mail.yourdomain.com" -Preference 10
    New-AzDnsRecordSet -Name "@" -RecordType MX -ResourceGroupName $resourceGroupName -ZoneName $dnsZoneName -Ttl 3600 -DnsRecords $mxRecord
    ```

**TXT Record:**

*   Add a TXT record with some sample text (e.g., "This is a test TXT record").

**Method: Azure Portal**

1.  **Navigate to your DNS Zone:** In the Azure portal, go to your DNS zone.

2.  **Add Record Set:**

    *   Click "+ Record set".
    *   **Name:** Enter `@` (for the zone root).
    *   **Type:** Select "TXT".
    *   **TTL:** Enter `3600` (seconds).
    *   **Text:** Enter `"This is a test TXT record"` (include the quotes).
    *   Click "OK".

### Part 3: Verify DNS Records

Use `nslookup` or `dig` to verify that the DNS records were created correctly.

1.  **Open a Command Prompt or Terminal:** On your local machine (not Cloud Shell).
2.  **Use nslookup (Windows):**

    ```
    nslookup -type=A www.iatd_labs_08.yourdomain.com
    nslookup -type=CNAME alias.iatd_labs_08.yourdomain.com
    nslookup -type=MX iatd_labs_08.yourdomain.com
    nslookup -type=TXT iatd_labs_08.yourdomain.com
    ```

3.  **Use dig (Linux/macOS):**

    ```
    dig A www.iatd_labs_08.yourdomain.com
    dig CNAME alias.iatd_labs_08.yourdomain.com
    dig MX iatd_labs_08.yourdomain.com
    dig TXT iatd_labs_08.yourdomain.com
    ```

    The output should show the records you created in Azure DNS. Note: It may take some time for the records to propagate across the internet.

### Part 4: Delegate the Zone to Azure DNS

1.  **Get Azure DNS Name Servers:** In the Azure portal, go to your DNS zone (`iatd_labs_08.yourdomain.com`). You will see a list of "Name servers" in the overview section. These are the Azure DNS name servers you need to use at your domain registrar. There are usually 4 servers

    **Example:**

    ```
    ns1-01.azure-dns.com.
    ns2-01.azure-dns.net.
    ns3-01.azure-dns.org.
    ns4-01.azure-dns.info.
    ```

2.  **Update NS Records at Your Domain Registrar:**

    *   Log in to your domain registrar's website (e.g., GoDaddy, Namecheap, AWS Route 53 - if your domain is in AWS).
    *   Find the DNS settings for your domain (`yourdomain.com`).
    *   Replace the existing name servers with the Azure DNS name servers you obtained in the previous step.
    *   **Important:** This process varies depending on your registrar. Refer to your registrar's documentation for specific instructions.

3.  **Verify Delegation:** It can take 24-48 hours for DNS delegation changes to propagate across the internet. You can use online tools or command-line tools to verify that the delegation has been successful.

    *   **Using `dig`:**

        ```
        dig NS yourdomain.com
        ```

        The output should show the Azure DNS name servers.

    *   **Using online tools:** Use websites like [https://www.whatsmydns.net/](https://www.whatsmydns.net/) to check the NS records for your domain.

### Post-Lab: Cleanup

To avoid incurring unnecessary charges, delete the resources you created in this lab:

1.  **Delete DNS Zone:** In the Azure portal, go to your DNS zone (`iatd_labs_08.yourdomain.com`) and click "Delete".
2.  **Delete Resource Group:** Delete the resource group (`iatd_labs_08_rg`).

### Important Considerations

*   **DNS Propagation:** DNS changes can take time to propagate. Be patient when verifying your records and delegation.
*   **Domain Ownership:** You must own or have control over the domain name to delegate it to Azure DNS.
*   **Name Server Accuracy:** Ensure that you enter the Azure DNS name servers correctly at your domain registrar. Typos can prevent proper delegation.
*   **TTL Values:** Shorter TTL values cause more frequent DNS queries but allow for faster updates. Longer TTL values reduce query load but can cause delays in updates.

**Congratulations!** This lab provides a practical guide to creating and configuring an Azure DNS zone. Remember to replace `yourdomain.com` with your actual domain name!
