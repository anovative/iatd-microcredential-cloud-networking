## IATD Microcredential Cloud Networking: Lab 3 - Deploying GCP VPCs using Jinja Templates

**Objective:** This lab guides you through the process of creating and deploying a Google Cloud Platform (GCP) Virtual Private Cloud (VPC) with predefined firewall rules using Jinja templates. You'll learn how to leverage templating for infrastructure as code, enabling consistent and secure deployments across different GCP projects.

**Estimated Time:** 75 - 90 minutes

**Prerequisites:**

1.  **GCP Account:** Requires an active GCP account. A free trial account is available at [https://cloud.google.com/free/](https://cloud.google.com/free/).
2.  **Basic knowledge of GCP Compute Engine:** Familiarity with VPCs, Subnets, Firewall Rules, and Compute Engine instances.
3.  **Basic understanding of Jinja Templating:** Familiarity with Jinja syntax is helpful.
4.  **Google Cloud SDK (gcloud CLI) Installed:** Make sure the gcloud CLI is installed and configured on your local machine or in Cloud Shell.

### Lab Conventions

*   **GCP Cloud Shell or Local Terminal:** All interactions will occur within the GCP Cloud Shell or a local terminal with the gcloud CLI configured.
*   **GCP Console:** The GCP Console will be used for verification tasks.
*   **Naming Conventions:** Resource names follow the convention `iatd-gcp-lab-10-*`.
*   **Region:** Choose a consistent GCP region (e.g., us-central1).
*   **Project ID:** Replace `<YOUR_PROJECT_ID>` with your actual GCP Project ID.

#### Resource Naming

*   **VPC:** `iatd-gcp-lab-10-vpc`
*   **Subnet:** `iatd-gcp-lab-10-subnet`
*   **Firewall Rule - Allow SSH:** `iatd-gcp-lab-10-fw-allow-ssh`
*   **Firewall Rule - Allow HTTP:** `iatd-gcp-lab-10-fw-allow-http`
*   **Jinja Template File:** `vpc_template.j2`

### Part 1: Creating the Jinja Template

1.  **Open Cloud Shell or Local Terminal:** Activate Cloud Shell in the GCP Console or open your local terminal.

2.  **Create Jinja Template File:**

    *   Create a file named `vpc_template.j2`:
        ```bash
        touch vpc_template.j2
        ```

    *   Open the file with a text editor (e.g., `nano vpc_template.j2`) and paste in the following Jinja template:

        ```jinja
        resources:
        - name: {{ vpc_name }}
          type: compute.v1.network
          properties:
            name: {{ vpc_name }}
            autoCreateSubnetworks: false
        - name: {{ subnet_name }}
          type: compute.v1.subnet
          properties:
            name: {{ subnet_name }}
            network: $(ref.{{ vpc_name }}.selfLink)
            ipCidrRange: {{ subnet_cidr }}
            region: {{ region }}
        - name: {{ fw_allow_ssh_name }}
          type: compute.v1.firewall
          properties:
            name: {{ fw_allow_ssh_name }}
            network: $(ref.{{ vpc_name }}.selfLink)
            allowed:
            - IPProtocol: tcp
              ports:
              - '22'
            sourceRanges:
            - {{ ssh_source_range }}
            targetTags:
            - allow-ssh
        - name: {{ fw_allow_http_name }}
          type: compute.v1.firewall
          properties:
            name: {{ fw_allow_http_name }}
            network: $(ref.{{ vpc_name }}.selfLink)
            allowed:
            - IPProtocol: tcp
              ports:
              - '80'
            sourceRanges:
            - 0.0.0.0/0
            targetTags:
            - allow-http
        ```

    *   Save the file.

### Part 2: Creating the Configuration File

1.  **Create the Configuration File:**

    *   Create a file named `vpc_config.yaml`:
        ```bash
        touch vpc_config.yaml
        ```

    *   Open the file with a text editor and paste in the following configuration:

        ```yaml
        vpc_name: iatd-gcp-lab-10-vpc
        subnet_name: iatd-gcp-lab-10-subnet
        subnet_cidr: 10.0.0.0/24
        region: us-central1
        fw_allow_ssh_name: iatd-gcp-lab-10-fw-allow-ssh
        fw_allow_http_name: iatd-gcp-lab-10-fw-allow-http
        ssh_source_range: 0.0.0.0/0 # Replace with a more restrictive range in a real-world scenario
        ```

    *   Save the file.

### Part 3: Deploying the Jinja Template

1.  **Deploy the Template using `gcloud`:**

    *   Run the following command in Cloud Shell or your local terminal:

        ```bash
        gcloud deployment-manager deployments create iatd-gcp-lab-10-deployment \
            --template vpc_template.j2 \
            --config vpc_config.yaml
        ```

    *   This command uses the `gcloud deployment-manager deployments create` command to deploy the Jinja template.
        *   `iatd-gcp-lab-10-deployment` is the name of the deployment.
        *   `vpc_template.j2` is the Jinja template file.
        *   `vpc_config.yaml` is the configuration file that provides values for the template parameters.

### Part 4: Verifying the Deployment

1.  **Navigate to VPC Networks:**
    *   In the GCP Console, search for "VPC network" and select **VPC networks**.
    *   Verify that the `iatd-gcp-lab-10-vpc` VPC has been created.

2.  **Check Subnets:**
    *   Click on the `iatd-gcp-lab-10-vpc` VPC.
    *   Verify that the `iatd-gcp-lab-10-subnet` subnet has been created with the correct CIDR block.

3.  **Check Firewall Rules:**
    *   In the GCP Console, search for "Firewall" and select **Firewall**.
    *   Verify that the following firewall rules have been created:
        *   `iatd-gcp-lab-10-fw-allow-ssh` (allowing SSH traffic)
        *   `iatd-gcp-lab-10-fw-allow-http` (allowing HTTP traffic)
    *   Ensure that the firewall rules have the correct source IP ranges and target tags.

### Part 5: Reusing the Jinja Template in Another Project

1.  **Change Project (if applicable):**
    *If you want to deploy to another project you need to change to that*
    *   If you want to deploy the VPC to a different GCP project, switch to that project using the gcloud CLI:
        ```bash
        gcloud config set project <ANOTHER_PROJECT_ID>
        ```

2.  **Deploy the Template to New Project:**
    *   Repeat the deployment process from Part 3 to create a new deployment using the same `vpc_template.j2` and `vpc_config.yaml` files.

### Post-Lab: Cleanup

To avoid incurring unnecessary costs, it's essential to clean up the resources created during this lab.

1.  **Delete Deployment:**
    *   In the GCP Console, search for "Deployment Manager" and select **Deployment Manager**.
    *   Select the `iatd-gcp-lab-10-deployment` deployment.
    *   Click **Delete**.

**Learning Outcomes:**

*   Successfully created a Jinja template to define a GCP VPC with subnets and firewall rules.
*   Successfully deployed the Jinja template using `gcloud` to create the VPC in your GCP project.
*   Successfully verified the deployment using the GCP Console.
*   Understood the benefits of using Jinja templates for infrastructure as code and repeatable deployments.
*   Learned how to reuse a Jinja template to deploy the same VPC configuration in another GCP project.
