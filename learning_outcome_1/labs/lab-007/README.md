## IATD Microcredential Cloud Networking: Lab 7 - Using Azure Site Recovery to Replicate Virtual Machines to a Secondary Region

**Objective:** In this lab, you will learn how to use Azure Site Recovery to replicate Azure Virtual Machines (VMs) to a secondary Azure region. You will then perform a test failover to simulate a disaster recovery scenario.

**Estimated Time:** 60 - 75 minutes

**Prerequisites:**

1.  **Azure Subscription:** You need an active Azure subscription.
2.  **Existing Virtual Machine:** You should have an existing Azure Virtual Machine in a primary region (e.g., `australiaeast`). This will be the VM you replicate. If you don't have one, create a basic one. It's best if the VM has some basic software installed (e.g., a web server) so you can easily verify it's working after failover. Ensure you make a note of your vm login details, these will be required for the DR environment post failover.
3.  **Basic Understanding of Azure and Virtualization:** Familiarity with Azure Virtual Machines, Resource Groups, and regions.

**Lab Conventions:**

*   **Naming Conventions:** All resources created in this lab will be prefixed with `iatd_labs_07` for easy identification and cleanup.
*   **IP Address Range:** The `172.16.0.0/16` range will be used (as per standardization).
*   **Location:** Choose a consistent Azure region for your primary region.

**Let's get started!**

### Part 1: Identify Primary and Secondary Regions

*   **Primary Region:** The region where your existing VM (`iatd_labs_07_vm` if you create one) is located (e.g., `australiaeast`).
*   **Secondary Region:** Choose a different Azure region as your secondary region for disaster recovery (e.g., `australiasoutheast`). This should be a supported region pair for Azure Site Recovery with your primary region. You can use the table in this document to verify the pairs. [https://learn.microsoft.com/en-us/azure/reliability/cross-region-replication-azure](https://learn.microsoft.com/en-us/azure/reliability/cross-region-replication-azure)

### Part 2: Create a Resource Group in the Secondary Region

This resource group will hold resources related to Site Recovery in the secondary region.

**Method: Azure Portal**

1.  **Log into the Azure Portal:** Go to [https://portal.azure.com/](https://portal.azure.com/) and sign in to your account.

2.  **Create a Resource Group:**

    *   Click on "Resource groups" in the left-hand menu.
    *   Click "+ Create".
    *   In the "Basics" tab:
        *   **Subscription:** Select your Azure subscription.
        *   **Resource group:** Enter `iatd_labs_07_dr_rg`
        *   **Region:** Select your secondary region (e.g., `australiasoutheast`).
    *   Click "Review + create", then "Create".

### Part 3: Create a Recovery Services Vault

The Recovery Services vault is the central resource for managing Azure Site Recovery.

**Method: Azure Portal**

1.  **Navigate to Recovery Services Vaults:**
    *   In the Azure portal, search for "Recovery Services vaults" and select it.
    *   Click "+ Create".

2.  **Configure the Vault:**
    *   **Subscription:** Select your Azure subscription.
    *   **Resource group:** Select `iatd_labs_07_dr_rg` (the one you created in the secondary region).
    *   **Vault name:** Enter `iatd_labs_07_rsv`.
    *   **Region:** Select your secondary region (e.g., `australiasoutheast`).
    *   Click "Review + create", then "Create".

### Part 4: Configure Replication for Your Virtual Machine

**Method: Azure Portal**

1.  **Navigate to Your Recovery Services Vault:**
    *   In the Azure portal, go to the Recovery Services vault you created (`iatd_labs_07_rsv`).

2.  **Start Replication Setup:**
    *   In the vault dashboard, under "Getting started", click "Site Recovery".
    *   Under "Azure virtual machines", click "Enable replication".

3.  **Configure Replication Settings:**

    *   **Protection goal:**
        *   "Where are your machines located?" = "Azure"
        *   "Where do you want to replicate your machines to?" = "To another Azure region"
        *   "Source region" = Your primary region (e.g., `australiaeast`)
        *   "Target region" = Your secondary region (e.g., `australiasoutheast`)
        *   "Target resource group" = `iatd_labs_07_dr_rg`
        *   "Target virtual network" = "Create new" (Name it `iatd_labs_07_dr_vnet`)
        *   "Cache storage account" = "Create new" (Name it `iatdlabs07cache`)
        *   Click "Next".

4.  **Select Virtual Machines:**
    *   Select the VM you want to replicate (e.g., `iatd_labs_07_vm`).
    *   Click "Next".

5.  **Configure the Compute and Network settings:**
    *   Review the VM configuration for the target region.
    *   You can adjust settings like VM size, availability zone, and network configuration if needed.
    *   Click "Next".

6.  **Configure Replication Settings:**
    *   Leave the default replication policy.
    *   Click "Next".

7.  **Review and Start Replication:**
    *   Review all settings.
    *   Click "Enable replication".

### Part 5: Monitor Replication Status

**Method: Azure Portal**

1.  **Navigate to Your Recovery Services Vault:**
    *   In the Azure portal, go to your Recovery Services vault (`iatd_labs_07_rsv`).

2.  **Check Replication Status:**
    *   Under "Protected items", click "Replicated items".
    *   Select your VM to see detailed replication status.

3.  **Wait for Initial Replication:**
    *   The initial replication may take some time depending on the size of your VM.
    *   The replication status should eventually show as "Protected" when the initial replication is complete.

### Part 6: Perform a Test Failover

A test failover allows you to verify your disaster recovery setup without affecting your production workload.

**Method: Azure Portal**

1.  **Navigate to Your Recovery Services Vault:**
    *   In the Azure portal, go to your Recovery Services vault (`iatd_labs_07_rsv`).

2.  **Start Test Failover:**
    *   Under "Protected items", click "Replicated items".
    *   Select your VM (e.g., `iatd_labs_07_vm`).
    *   Click "Test failover".

3.  **Configure Test Failover:**
    *   **Recovery point:** Choose "Latest processed" (or select a specific recovery point if needed).
    *   **Azure virtual network:** Select the virtual network you created for the target region (e.g., `iatd_labs_07_dr_vnet`).
    *   Click "OK".

4.  **Monitor Test Failover Progress:**
    *   Azure will create a test VM in your secondary region.
    *   You can monitor the progress in the "Site Recovery jobs" section.

In task configuration of step number **5: Configure the Compute and Network settings (Optional)** networking setting configuration page, associate a public IP address with Virtual Machine. Alternatively and additional open the 3389 and other custom network configuration details via:

1.  Search and Click **Network Security Group**.
2.  If none already exists, Create one.
3.  Navigate and Associate to test network created in Networking configuration settings.

### Part 7: Verify the Test Failover

**Method: Azure Portal**

1.  **Navigate to Virtual Machines:**
    *   In the Azure portal, go to "Virtual machines".
    *   You should see a new VM in your secondary region with a name like `iatd_labs_07_vm-test`.

2.  **Connect to the Test VM:**
    *   Select the test VM.
    *   Click "Connect" and choose the appropriate connection method (RDP for Windows, SSH for Linux).
    *   Download the RDP file or note the SSH command.
    *   Connect to the VM using the credentials of your original VM.

3.  **Verify VM Functionality:**
    *   Ensure that the VM is running correctly.
    *   If you installed any software on the original VM (e.g., a web server), verify that it's working.

1.  Connect the Test machine by Public Ip and login details. Ensure VM login page details works such as **username** and **password.** Confirm with your login detail record previously.

2.  Once login Confirm drive is mounted.

3.  If VM login credential failed, this because machine could not connect with Domain Controller(Active Directory). For troubleshooting it login as the **local machine credentials**.

*If a Domain Controller for Test-Fail-Over does not exists. it can configured for local directory: To troubleshoot and configure:
a. Go to the Network security settings created previously
b. Open Network interface - associated previously, note it somewhere
c. Reset login Credentials of VM*

### Part 8: Clean Up the Test Failover

It's crucial to clean up the test failover to avoid leaving running resources in your secondary region.

**Method: Azure Portal**

1.  **Navigate to your Recovery Services Vault:** In the Azure portal, go to your `iatd_labs_07_rsv`.
2.  **Stop Test Failover:**
    *   Under "Protected Items", click "Replicated items".
    *   Select the VM you performed the test failover on (e.g., `iatd_labs_07_vm`).
    *   Click "Clean up test failover".
    *   In the "Notes" section, enter a brief description of your testing.
    *   Select the checkbox "Testing is complete. Delete test failover virtual machine."
    *   Click "OK".

Azure Site Recovery will now remove the test VM from your secondary region.

### Part 9: (Optional) Perform a Planned Failover (If Desired)

A *planned failover* is used in a real disaster or when you are migrating workloads. It involves shutting down the primary VM gracefully (if possible) and then failing over to the secondary region.

**Important: Only do this if you are prepared for potential downtime of your primary VM.**

The steps are similar to a test failover, but you choose "Failover" instead of "Test Failover" in Part 6. After a planned failover, you will need to "Commit" the failover if you intend to make the secondary region your new primary, or "Re-protect" to return replication to the primary region once it is available again.

### Post-Lab: Cleanup

To avoid incurring unnecessary charges, delete the resources you created in this lab:

1.  **Disable Replication:** In your Recovery Services vault (`iatd_labs_07_rsv`), under "Replicated items," select your VM and click "Disable replication."
2.  **Delete the Recovery Services Vault:** Once replication is disabled, you can delete the Recovery Services vault (`iatd_labs_07_rsv`).
3.  **Delete Secondary Region Resources:** Delete the resource group you created in the secondary region (`iatd_labs_07_dr_rg`).
4.  **If you created a test network** clean all NSG associated etc... before deletion.

### Verification

*   Verify that the replication status for your VM in the Recovery Services vault shows as "Healthy."
*   Confirm that you can successfully start a test failover and that the test VM is created in the secondary region.
*   Verify that you can connect to the test VM (using RDP or SSH).

### Important Considerations

*   **Network Configuration:** Pay close attention to network settings during replication and failover. Ensure that the virtual networks and subnets in your secondary region are correctly configured.
*   **DNS:** In a real disaster recovery scenario, you will need to update your DNS records to point to the new public IP addresses of your failed-over VMs.
*   **Testing:** Regularly test your failover process to ensure that it works as expected.

**Congratulations!** This lab provides a hands-on introduction to Azure Site Recovery. Remember that a robust disaster recovery plan requires careful planning and testing!
