## IATD Microcredential Cloud Networking: GCP Lab 2 - Hybrid Connectivity with Cloud VPN and Cloud Interconnect

**Objective:** This lab guides you through implementing hybrid connectivity solutions in Google Cloud Platform (GCP). You'll learn how to set up Cloud VPN for secure connections between your on-premises network (simulated in GCP) and Google Cloud VPCs, and understand the concepts of Cloud Interconnect for dedicated connectivity.

**Estimated Time:** 90 - 120 minutes

**Prerequisites:**

1.  **GCP Account:** Requires an active GCP account. A free tier account is available at [https://cloud.google.com/free/](https://cloud.google.com/free/).
2.  **Basic understanding of GCP Networking:** Familiarity with VPC networks, subnets, and Compute Engine instances.
3.  **Basic understanding of VPN concepts:** Familiarity with IPsec, IKE, and VPN tunnels.

### Lab Conventions

*   **Google Cloud Console:** Most interactions will occur via the Google Cloud Console.
*   **Naming Conventions:** Resource names follow the convention `iatd-gcp-hybrid-*`.
*   **IP Address Range:** The `172.16.0.0/16` range will be used (as per standardization).
*   **Region:** Choose a consistent GCP region (e.g., us-central1).

#### Resource Naming

*   **On-premises VPC:** `iatd-gcp-hybrid-onprem-vpc`
*   **Cloud VPC:** `iatd-gcp-hybrid-cloud-vpc`
*   **On-premises Subnet:** `iatd-gcp-hybrid-onprem-subnet`
*   **Cloud Subnet:** `iatd-gcp-hybrid-cloud-subnet`
*   **On-premises VM:** `iatd-gcp-hybrid-onprem-vm`
*   **Cloud VM:** `iatd-gcp-hybrid-cloud-vm`
*   **Cloud VPN Gateway:** `iatd-gcp-hybrid-cloud-vpn-gw`
*   **On-premises VPN Gateway:** `iatd-gcp-hybrid-onprem-vpn-gw`
*   **VPN Tunnel - Cloud to On-premises:** `iatd-gcp-hybrid-cloud-to-onprem-tunnel`
*   **VPN Tunnel - On-premises to Cloud:** `iatd-gcp-hybrid-onprem-to-cloud-tunnel`

### Part 1: Setting up the VPC Networks

1.  **Sign in to the Google Cloud Console:** Browse to [https://console.cloud.google.com/](https://console.cloud.google.com/) and sign in.

2.  **Create the On-premises VPC Network:**
    *   Navigate to **VPC network** > **VPC networks**.
    *   Click **Create VPC Network**.
    *   Configure the VPC network:
        *   **Name:** `iatd-gcp-hybrid-onprem-vpc`
        *   **Description:** `Simulated on-premises network`
        *   **Subnet creation mode:** Custom
        *   Add a subnet:
            *   **Name:** `iatd-gcp-hybrid-onprem-subnet`
            *   **Region:** us-central1
            *   **IP address range:** `172.16.1.0/24`
    *   Click **Create**.

3.  **Create the Cloud VPC Network:**
    *   Click **Create VPC Network** again.
    *   Configure the VPC network:
        *   **Name:** `iatd-gcp-hybrid-cloud-vpc`
        *   **Description:** `Cloud network for hybrid connectivity`
        *   **Subnet creation mode:** Custom
        *   Add a subnet:
            *   **Name:** `iatd-gcp-hybrid-cloud-subnet`
            *   **Region:** us-central1
            *   **IP address range:** `172.16.2.0/24`
    *   Click **Create**.

4.  **Create Firewall Rules:**
    *   Navigate to **VPC network** > **Firewall**.
    *   Click **Create Firewall Rule**.
    *   Configure the firewall rule for the on-premises VPC:
        *   **Name:** `iatd-gcp-hybrid-onprem-allow-ssh-icmp`
        *   **Network:** `iatd-gcp-hybrid-onprem-vpc`
        *   **Priority:** 1000
        *   **Direction of traffic:** Ingress
        *   **Action on match:** Allow
        *   **Targets:** All instances in the network
        *   **Source filter:** IP ranges
        *   **Source IP ranges:** `0.0.0.0/0` (for SSH), `172.16.0.0/16` (for internal communication)
        *   **Protocols and ports:**
            *   Check TCP and enter port 22
            *   Check ICMP
    *   Click **Create**.
    *   Repeat this process for the cloud VPC, creating a similar firewall rule named `iatd-gcp-hybrid-cloud-allow-ssh-icmp`.

    Expected output:
    ```
    Two VPC networks created successfully:
    - On-premises VPC with subnet 172.16.1.0/24
    - Cloud VPC with subnet 172.16.2.0/24
    - Firewall rules allowing SSH and ICMP traffic
    ```

### Part 2: Creating Compute Engine Instances

1.  **Create the On-premises VM:**
    *   Navigate to **Compute Engine** > **VM instances**.
    *   Click **Create instance**.
    *   Configure the instance:
        *   **Name:** `iatd-gcp-hybrid-onprem-vm`
        *   **Region:** us-central1
        *   **Zone:** us-central1-a
        *   **Machine configuration:** e2-micro (2 vCPU, 1 GB memory)
        *   **Boot disk:** Debian GNU/Linux 11 (bullseye)
        *   **Networking:**
            *   **Network:** `iatd-gcp-hybrid-onprem-vpc`
            *   **Subnet:** `iatd-gcp-hybrid-onprem-subnet`
            *   **External IP:** Ephemeral (for SSH access)
    *   Click **Create**.

2.  **Create the Cloud VM:**
    *   Click **Create instance** again.
    *   Configure the instance:
        *   **Name:** `iatd-gcp-hybrid-cloud-vm`
        *   **Region:** us-central1
        *   **Zone:** us-central1-a
        *   **Machine configuration:** e2-micro (2 vCPU, 1 GB memory)
        *   **Boot disk:** Debian GNU/Linux 11 (bullseye)
        *   **Networking:**
            *   **Network:** `iatd-gcp-hybrid-cloud-vpc`
            *   **Subnet:** `iatd-gcp-hybrid-cloud-subnet`
            *   **External IP:** Ephemeral (for SSH access)
    *   Click **Create**.

    Expected output:
    ```
    Two VM instances created successfully:
    - On-premises VM in the on-premises VPC
    - Cloud VM in the cloud VPC
    - Both VMs have external IPs for SSH access
    ```

### Part 3: Setting up Cloud VPN

1.  **Create the Cloud VPN Gateway:**
    *   Navigate to **VPC network** > **Cloud VPN gateways**.
    *   Click **Create VPN gateway**.
    *   Configure the VPN gateway:
        *   **Name:** `iatd-gcp-hybrid-cloud-vpn-gw`
        *   **Description:** `Cloud VPN gateway for hybrid connectivity`
        *   **Network:** `iatd-gcp-hybrid-cloud-vpc`
        *   **Region:** us-central1
    *   Click **Create**.

2.  **Create the On-premises VPN Gateway (simulated in GCP):**
    *   Click **Create VPN gateway** again.
    *   Configure the VPN gateway:
        *   **Name:** `iatd-gcp-hybrid-onprem-vpn-gw`
        *   **Description:** `Simulated on-premises VPN gateway`
        *   **Network:** `iatd-gcp-hybrid-onprem-vpc`
        *   **Region:** us-central1
    *   Click **Create**.

3.  **Create VPN Tunnels:**
    *   Navigate to **VPC network** > **VPN**.
    *   Click **Create VPN connection**.
    *   Select **Classic VPN** and click **Continue**.
    *   Configure the VPN connection:
        *   **Name:** `iatd-gcp-hybrid-cloud-to-onprem-tunnel`
        *   **Description:** `Tunnel from cloud to on-premises`
        *   **VPN gateway:** `iatd-gcp-hybrid-cloud-vpn-gw`
        *   **Peer VPN gateway:** Create a new peer VPN gateway
            *   **Name:** `iatd-gcp-hybrid-onprem-peer-gw`
            *   **IP address:** Enter the external IP address of the on-premises VPN gateway
        *   **IKE version:** IKEv2
        *   **Shared secret:** Generate a strong shared secret
        *   **Routing options:** Static
        *   **Remote network IP ranges:** `172.16.1.0/24` (On-premises subnet)
        *   **Local IP ranges:** `172.16.2.0/24` (Cloud subnet)
    *   Click **Create**.

4.  **Create the Reverse VPN Tunnel:**
    *   Click **Create VPN connection** again.
    *   Select **Classic VPN** and click **Continue**.
    *   Configure the VPN connection:
        *   **Name:** `iatd-gcp-hybrid-onprem-to-cloud-tunnel`
        *   **Description:** `Tunnel from on-premises to cloud`
        *   **VPN gateway:** `iatd-gcp-hybrid-onprem-vpn-gw`
        *   **Peer VPN gateway:** Create a new peer VPN gateway
            *   **Name:** `iatd-gcp-hybrid-cloud-peer-gw`
            *   **IP address:** Enter the external IP address of the cloud VPN gateway
        *   **IKE version:** IKEv2
        *   **Shared secret:** Use the same shared secret as the first tunnel
        *   **Routing options:** Static
        *   **Remote network IP ranges:** `172.16.2.0/24` (Cloud subnet)
        *   **Local IP ranges:** `172.16.1.0/24` (On-premises subnet)
    *   Click **Create**.

    Expected output:
    ```
    VPN gateways and tunnels created successfully:
    - Two VPN gateways, one for the cloud VPC and one for the on-premises VPC
    - Two VPN tunnels, one in each direction
    - Static routes configured to route traffic between the subnets
    ```

### Part 4: Testing the VPN Connection

1.  **Verify VPN Tunnel Status:**
    *   Navigate to **VPC network** > **VPN**.
    *   Check the status of both VPN tunnels. They should show as "Established" after a few minutes.

2.  **Test Connectivity:**
    *   Connect to the cloud VM via SSH.
    *   Ping the internal IP address of the on-premises VM:

        ```bash
        ping <internal-ip-of-onprem-vm>
        ```

    *   The ping should be successful, indicating that traffic is flowing through the VPN tunnels.
    *   Connect to the on-premises VM via SSH.
    *   Ping the internal IP address of the cloud VM:

        ```bash
        ping <internal-ip-of-cloud-vm>
        ```

    *   The ping should also be successful.

    Expected outcome:
    ```
    - VPN tunnels show as "Established" in the GCP console
    - Successful ping between the cloud VM and on-premises VM using internal IP addresses
    - Traffic is routed through the VPN tunnels
    ```

### Part 5: Understanding Cloud Interconnect (Conceptual)

1.  **Cloud Interconnect Overview:**
    *   Cloud Interconnect provides dedicated physical connections between your on-premises network and Google's network.
    *   There are two types of Cloud Interconnect:
        *   **Dedicated Interconnect:** Provides direct physical connections to Google's network.
        *   **Partner Interconnect:** Connects to Google's network through a supported service provider.

2.  **Dedicated Interconnect:**
    *   Requires physical connections in a colocation facility where Google has a point of presence.
    *   Provides 10 Gbps or 100 Gbps connections.
    *   Requires redundant connections for 99.99% SLA.
    *   Suitable for high-bandwidth, mission-critical workloads.

3.  **Partner Interconnect:**
    *   Works with supported service providers to establish connectivity.
    *   Available in capacities from 50 Mbps to 10 Gbps.
    *   More flexible in terms of location and capacity requirements.
    *   Suitable for organizations that can't reach a colocation facility or need less than 10 Gbps of capacity.

4.  **Cloud Interconnect vs. Cloud VPN:**
    *   **Connectivity:** Dedicated physical connection vs. encrypted tunnel over the internet.
    *   **Bandwidth:** Higher (up to 100 Gbps) vs. Lower (limited by internet connection).
    *   **Latency:** Lower vs. Higher.
    *   **Security:** Private connection vs. encrypted over public internet.
    *   **Cost:** Higher vs. Lower.

    ```
    +------------------------+                                  +------------------------+
    |                        |                                  |                        |
    |   On-premises Network  |                                  |     Google Cloud VPC    |
    |                        |                                  |                        |
    +------------------------+                                  +------------------------+
              |                                                          |
              |                                                          |
              v                                                          v
    +------------------------+    Dedicated Physical Connection    +------------------------+
    |                        |<--------------------------------->|                        |
    |  Customer Edge Router |                                   |  Google Edge Router    |
    |                        |                                   |                        |
    +------------------------+                                   +------------------------+
                                      Cloud Interconnect
    ```

### Post-Lab: Cleanup

To avoid incurring unnecessary costs, it's essential to clean up the resources created during this lab.

1.  **Delete VPN Tunnels:**
    *   Navigate to **VPC network** > **VPN**.
    *   Select both VPN tunnels.
    *   Click **Delete**.
    *   Confirm deletion.

2.  **Delete VPN Gateways:**
    *   Navigate to **VPC network** > **Cloud VPN gateways**.
    *   Select both VPN gateways.
    *   Click **Delete**.
    *   Confirm deletion.

3.  **Delete VM Instances:**
    *   Navigate to **Compute Engine** > **VM instances**.
    *   Select both VM instances.
    *   Click **Delete**.
    *   Confirm deletion.

4.  **Delete Firewall Rules:**
    *   Navigate to **VPC network** > **Firewall**.
    *   Select the firewall rules created for this lab.
    *   Click **Delete**.
    *   Confirm deletion.

5.  **Delete VPC Networks:**
    *   Navigate to **VPC network** > **VPC networks**.
    *   Select both VPC networks.
    *   Click **Delete**.
    *   Confirm deletion.

**Learning Outcomes:**

*   Successfully created and configured Cloud VPN for connecting a simulated on-premises network to Google Cloud.
*   Successfully tested connectivity between VMs in different networks through VPN tunnels.
*   Understood the key components of hybrid connectivity in GCP, including VPN gateways and tunnels.
*   Learned about Cloud Interconnect options for dedicated connectivity.
*   Understood the differences between Cloud VPN and Cloud Interconnect for hybrid connectivity scenarios.
