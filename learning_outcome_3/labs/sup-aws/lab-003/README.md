## IATD Microcredential Cloud Networking: AWS Supplementary Lab 3 - Monitoring and Logging in AWS

**Objective:** This lab introduces you to the fundamental concepts of monitoring and logging in AWS. You will learn about CloudWatch, CloudTrail, and AWS Config to monitor resources, analyze logs, establish performance baselines, and set alert thresholds.

**Estimated Time:** 75 - 90 minutes

**Prerequisites:**

1.  **AWS Account:** Requires an active AWS account. A free tier account is available at [https://aws.amazon.com/free/](https://aws.amazon.com/free/).
2.  **Basic knowledge of AWS Networking:** Familiarity with VPCs, Security Groups, and EC2 instances.

**Let's get started!**

### Lab Conventions

*   **AWS CLI:** All CLI interactions will occur within your local terminal or AWS CloudShell.
*   **AWS Management Console:** The AWS Console will be used for visualization and configuration tasks.
*   **Naming Conventions:** Resource names follow the convention `iatd_labs_aws_03_*`.
*   **IP Address Range:** The `172.16.x.x` range will be used.
*   **Region:** Choose a consistent AWS region (e.g., us-east-1).

#### Resource Naming

*   **VPC:** `iatd_labs_aws_03_vpc`
*   **Subnet:** `iatd_labs_aws_03_subnet`
*   **EC2 Instance:** `iatd_labs_aws_03_ec2`
*   **Security Group:** `iatd_labs_aws_03_sg`
*   **CloudWatch Dashboard:** `iatd_labs_aws_03_dashboard`
*   **CloudWatch Alarm:** `iatd_labs_aws_03_alarm`
*   **CloudWatch Log Group:** `iatd_labs_aws_03_log_group`

### Part 1: Setting up a Monitored Environment

1.  **Create a VPC:**

    ```bash
    AWS_REGION="us-east-1" # Your Region
    VPC_NAME="iatd_labs_aws_03_vpc"
    SUBNET_NAME="iatd_labs_aws_03_subnet"
    VPC_CIDR="172.16.0.0/16"
    SUBNET_CIDR="172.16.0.0/24"
    
    # Create VPC
    VPC_ID=$(aws ec2 create-vpc \
        --cidr-block $VPC_CIDR \
        --tag-specifications "ResourceType=vpc,Tags=[{Key=Name,Value=$VPC_NAME}]" \
        --region $AWS_REGION \
        --output text \
        --query 'Vpc.VpcId')
    
    # Enable DNS hostnames for the VPC
    aws ec2 modify-vpc-attribute \
        --vpc-id $VPC_ID \
        --enable-dns-hostnames \
        --region $AWS_REGION
    ```

    **Expected Output:**
    ```
    vpc-0a1b2c3d4e5f6g7h8
    ```

2.  **Create a Subnet:**

    ```bash
    # Create subnet
    SUBNET_ID=$(aws ec2 create-subnet \
        --vpc-id $VPC_ID \
        --cidr-block $SUBNET_CIDR \
        --availability-zone ${AWS_REGION}a \
        --tag-specifications "ResourceType=subnet,Tags=[{Key=Name,Value=$SUBNET_NAME}]" \
        --region $AWS_REGION \
        --output text \
        --query 'Subnet.SubnetId')
    ```

    **Expected Output:**
    ```
    subnet-0a1b2c3d4e5f6g7h8
    ```

3.  **Create an Internet Gateway and Attach to VPC:**

    ```bash
    IGW_NAME="iatd_labs_aws_03_igw"
    
    # Create Internet Gateway
    IGW_ID=$(aws ec2 create-internet-gateway \
        --tag-specifications "ResourceType=internet-gateway,Tags=[{Key=Name,Value=$IGW_NAME}]" \
        --region $AWS_REGION \
        --output text \
        --query 'InternetGateway.InternetGatewayId')
    
    # Attach Internet Gateway to VPC
    aws ec2 attach-internet-gateway \
        --internet-gateway-id $IGW_ID \
        --vpc-id $VPC_ID \
        --region $AWS_REGION
    ```

    **Expected Output for create-internet-gateway:**
    ```
    igw-0a1b2c3d4e5f6g7h8
    ```

4.  **Create a Route Table and Add Routes:**

    ```bash
    RT_NAME="iatd_labs_aws_03_rt"
    
    # Create Route Table
    RT_ID=$(aws ec2 create-route-table \
        --vpc-id $VPC_ID \
        --tag-specifications "ResourceType=route-table,Tags=[{Key=Name,Value=$RT_NAME}]" \
        --region $AWS_REGION \
        --output text \
        --query 'RouteTable.RouteTableId')
    
    # Add route to Internet Gateway
    aws ec2 create-route \
        --route-table-id $RT_ID \
        --destination-cidr-block 0.0.0.0/0 \
        --gateway-id $IGW_ID \
        --region $AWS_REGION
    
    # Associate Route Table with Subnet
    aws ec2 associate-route-table \
        --route-table-id $RT_ID \
        --subnet-id $SUBNET_ID \
        --region $AWS_REGION
    ```

    **Expected Output for create-route:**
    ```json
    {
        "Return": true
    }
    ```

5.  **Create a Security Group:**

    ```bash
    SG_NAME="iatd_labs_aws_03_sg"
    
    # Create Security Group
    SG_ID=$(aws ec2 create-security-group \
        --group-name $SG_NAME \
        --description "Security group for IATD AWS Lab 3" \
        --vpc-id $VPC_ID \
        --region $AWS_REGION \
        --output text \
        --query 'GroupId')
    
    # Add inbound rule for SSH
    aws ec2 authorize-security-group-ingress \
        --group-id $SG_ID \
        --protocol tcp \
        --port 22 \
        --cidr 0.0.0.0/0 \
        --region $AWS_REGION
    
    # Add inbound rule for HTTP
    aws ec2 authorize-security-group-ingress \
        --group-id $SG_ID \
        --protocol tcp \
        --port 80 \
        --cidr 0.0.0.0/0 \
        --region $AWS_REGION
    ```

    **Expected Output for create-security-group:**
    ```
    sg-0a1b2c3d4e5f6g7h8
    ```

6.  **Launch an EC2 Instance:**

    ```bash
    EC2_NAME="iatd_labs_aws_03_ec2"
    AMI_ID="ami-0c55b159cbfafe1f0" # Amazon Linux 2 AMI ID (may vary by region)
    INSTANCE_TYPE="t2.micro"
    KEY_NAME="your-key-pair" # Replace with your key pair name
    
    # Create EC2 instance
    INSTANCE_ID=$(aws ec2 run-instances \
        --image-id $AMI_ID \
        --instance-type $INSTANCE_TYPE \
        --key-name $KEY_NAME \
        --subnet-id $SUBNET_ID \
        --security-group-ids $SG_ID \
        --associate-public-ip-address \
        --tag-specifications "ResourceType=instance,Tags=[{Key=Name,Value=$EC2_NAME}]" \
        --user-data "#!/bin/bash\napt-get update -y\napt-get install -y nginx\nsystemctl start nginx\nsystemctl enable nginx" \
        --region $AWS_REGION \
        --output text \
        --query 'Instances[0].InstanceId')
    ```

    **Expected Output:**
    ```
    i-0a1b2c3d4e5f6g7h8
    ```

7.  **Get the Public IP of the EC2 Instance:**

    ```bash
    # Get public IP
    PUBLIC_IP=$(aws ec2 describe-instances \
        --instance-ids $INSTANCE_ID \
        --region $AWS_REGION \
        --output text \
        --query 'Reservations[0].Instances[0].PublicIpAddress')
    
    echo "EC2 Instance Public IP: $PUBLIC_IP"
    ```

    **Expected Output:**
    ```
    EC2 Instance Public IP: 54.x.x.x
    ```

8.  **Test the Web Server:**
    *   Open a web browser and navigate to the Public IP address of the EC2 instance. You should see the Nginx welcome page.

### Part 2: Setting up AWS Monitoring Services

1.  **Enable Detailed Monitoring for EC2 Instance:**

    ```bash
    aws ec2 monitor-instances \
        --instance-ids $INSTANCE_ID \
        --region $AWS_REGION
    ```

    **Expected Output:**
    ```json
    {
        "InstanceMonitorings": [
            {
                "InstanceId": "i-0a1b2c3d4e5f6g7h8",
                "Monitoring": {
                    "State": "pending"
                }
            }
        ]
    }
    ```

2.  **Create a CloudWatch Dashboard (Console):**
    *   Navigate to the CloudWatch console in the AWS Management Console.
    *   Select **Dashboards** from the left navigation pane.
    *   Click **Create dashboard**.
    *   Enter `iatd_labs_aws_03_dashboard` as the dashboard name.
    *   Click **Create dashboard**.
    *   Add widgets to monitor CPU, memory, disk, and network metrics for your EC2 instance.

3.  **Create a CloudWatch Log Group:**

    ```bash
    LOG_GROUP_NAME="iatd_labs_aws_03_log_group"
    
    aws logs create-log-group \
        --log-group-name $LOG_GROUP_NAME \
        --region $AWS_REGION
    ```

    **Expected Output:**
    No output is returned if the command is successful.

4.  **Install and Configure CloudWatch Agent on EC2 Instance:**
    *   Connect to your EC2 instance via SSH.
    *   Install the CloudWatch agent:

        ```bash
        sudo yum install -y amazon-cloudwatch-agent
        ```

    *   Create a CloudWatch agent configuration file:

        ```bash
        sudo tee /opt/aws/amazon-cloudwatch-agent/bin/config.json > /dev/null << 'EOF'
        {
          "agent": {
            "metrics_collection_interval": 60,
            "run_as_user": "root"
          },
          "logs": {
            "logs_collected": {
              "files": {
                "collect_list": [
                  {
                    "file_path": "/var/log/nginx/access.log",
                    "log_group_name": "iatd_labs_aws_03_log_group",
                    "log_stream_name": "nginx-access"
                  },
                  {
                    "file_path": "/var/log/nginx/error.log",
                    "log_group_name": "iatd_labs_aws_03_log_group",
                    "log_stream_name": "nginx-error"
                  }
                ]
              }
            }
          },
          "metrics": {
            "metrics_collected": {
              "cpu": {
                "measurement": [
                  "cpu_usage_idle",
                  "cpu_usage_iowait",
                  "cpu_usage_user",
                  "cpu_usage_system"
                ],
                "metrics_collection_interval": 60,
                "totalcpu": false
              },
              "disk": {
                "measurement": [
                  "used_percent",
                  "inodes_free"
                ],
                "metrics_collection_interval": 60,
                "resources": [
                  "*"
                ]
              },
              "diskio": {
                "measurement": [
                  "io_time"
                ],
                "metrics_collection_interval": 60,
                "resources": [
                  "*"
                ]
              },
              "mem": {
                "measurement": [
                  "mem_used_percent"
                ],
                "metrics_collection_interval": 60
              },
              "swap": {
                "measurement": [
                  "swap_used_percent"
                ],
                "metrics_collection_interval": 60
              }
            }
          }
        }
        EOF
        ```

    *   Start the CloudWatch agent:

        ```bash
        sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -a fetch-config -m ec2 -s -c file:/opt/aws/amazon-cloudwatch-agent/bin/config.json
        ```

### Part 3: Monitoring Types and Log Analysis

1.  **Active Monitoring with CloudWatch Alarms:**
    *   **Concept:** Actively monitoring metrics and triggering alarms when thresholds are exceeded.
    *   **Implementation:** Create a CloudWatch alarm to monitor CPU utilization of your EC2 instance.

        ```bash
        ALARM_NAME="iatd_labs_aws_03_alarm"
        
        aws cloudwatch put-metric-alarm \
            --alarm-name $ALARM_NAME \
            --alarm-description "Alarm when CPU exceeds 80%" \
            --metric-name CPUUtilization \
            --namespace AWS/EC2 \
            --statistic Average \
            --period 300 \
            --threshold 80 \
            --comparison-operator GreaterThanThreshold \
            --dimensions Name=InstanceId,Value=$INSTANCE_ID \
            --evaluation-periods 2 \
            --alarm-actions arn:aws:sns:$AWS_REGION:$(aws sts get-caller-identity --query 'Account' --output text):$ALARM_NAME \
            --region $AWS_REGION
        ```

        **Expected Output:**
        No output is returned if the command is successful.

2.  **Passive Monitoring with CloudWatch Logs:**
    *   **Concept:** Collecting and analyzing logs without actively probing the system.
    *   **Implementation:** Navigate to the CloudWatch Logs console and explore the Nginx access and error logs collected by the CloudWatch agent.

3.  **Synthetic Monitoring with CloudWatch Synthetics:**
    *   **Concept:** Simulating user behavior to test application performance and availability.
    *   **Implementation:** Create a CloudWatch Synthetics canary to monitor the availability of your web server.

        ```bash
        CANARY_NAME="iatd-labs-aws-03-canary"
        S3_BUCKET="iatd-labs-aws-03-canary-results" # Replace with a unique bucket name
        
        # Create S3 bucket for canary results
        aws s3 mb s3://$S3_BUCKET --region $AWS_REGION
        
        # Create CloudWatch Synthetics canary
        aws synthetics create-canary \
            --name $CANARY_NAME \
            --artifact-s3-location "S3Bucket=$S3_BUCKET,S3Key=canary/" \
            --execution-role-arn "arn:aws:iam::$(aws sts get-caller-identity --query 'Account' --output text):role/service-role/CloudWatchSyntheticsRole-$CANARY_NAME" \
            --schedule "Expression='rate(5 minutes)'" \
            --run-config "TimeoutInSeconds=60" \
            --code "Handler=heartbeat.handler,S3Bucket=cloudwatch-synthetics-code,S3Key=canary/nodejs/node_modules/heartbeat-canary.zip" \
            --success-retention-period 30 \
            --failure-retention-period 30 \
            --runtime-version syn-nodejs-puppeteer-3.1 \
            --region $AWS_REGION
        ```

        **Note:** This command assumes you have already created the necessary IAM role for CloudWatch Synthetics. If not, you'll need to create it first.

4.  **Flow-Based Monitoring with VPC Flow Logs:**
    *   **Concept:** Analyzing network traffic flow data to identify patterns and anomalies.
    *   **Implementation:** Enable VPC Flow Logs for your VPC.

        ```bash
        FLOW_LOG_NAME="iatd_labs_aws_03_flow_log"
        LOG_DESTINATION="arn:aws:logs:$AWS_REGION:$(aws sts get-caller-identity --query 'Account' --output text):log-group:$LOG_GROUP_NAME"
        
        aws ec2 create-flow-logs \
            --resource-type VPC \
            --resource-ids $VPC_ID \
            --traffic-type ALL \
            --log-destination-type cloud-watch-logs \
            --log-destination $LOG_DESTINATION \
            --deliver-logs-permission-arn "arn:aws:iam::$(aws sts get-caller-identity --query 'Account' --output text):role/flowlogsRole" \
            --region $AWS_REGION
        ```

        **Note:** This command assumes you have already created the necessary IAM role for VPC Flow Logs. If not, you'll need to create it first.

5.  **Log Analysis with CloudWatch Logs Insights:**
    *   Navigate to the CloudWatch Logs Insights console in the AWS Management Console.
    *   Select the `iatd_labs_aws_03_log_group` log group.
    *   Run the following query to analyze Nginx access logs:

        ```
        fields @timestamp, @message
        | filter @message like /GET/
        | stats count(*) as requestCount by bin(30m)
        | sort requestCount desc
        ```

### Part 4: Performance Baselines and Alert Thresholds

1.  **Establishing Performance Baselines:**
    *   **Concept:** Determining the normal operating parameters of a system.
    *   **Implementation:** Monitor the EC2 instance's CPU, memory, disk, and network usage over a period of time to establish a baseline.
        *   Navigate to the CloudWatch console in the AWS Management Console.
        *   Select **Metrics** from the left navigation pane.
        *   Explore the EC2 metrics for your instance.
        *   Observe the patterns over time to establish a baseline.

2.  **Setting Alert Thresholds:**
    *   **Concept:** Defining thresholds that trigger alerts when exceeded.
    *   **Implementation:** Create a CloudWatch alarm to notify you when the EC2 instance's disk usage exceeds 80%.

        ```bash
        DISK_ALARM_NAME="iatd_labs_aws_03_disk_alarm"
        
        aws cloudwatch put-metric-alarm \
            --alarm-name $DISK_ALARM_NAME \
            --alarm-description "Alarm when disk usage exceeds 80%" \
            --metric-name disk_used_percent \
            --namespace CWAgent \
            --statistic Average \
            --period 300 \
            --threshold 80 \
            --comparison-operator GreaterThanThreshold \
            --dimensions Name=InstanceId,Value=$INSTANCE_ID Name=path,Value=/ Name=device,Value=xvda1 Name=fstype,Value=xfs \
            --evaluation-periods 2 \
            --alarm-actions arn:aws:sns:$AWS_REGION:$(aws sts get-caller-identity --query 'Account' --output text):$DISK_ALARM_NAME \
            --region $AWS_REGION
        ```

3.  **Static vs. Dynamic Thresholds:**
    *   **Concept:**
        *   **Static Thresholds:** Fixed values that trigger alerts (e.g., CPU > 80%).
        *   **Dynamic Thresholds:** Automatically adjust based on historical data and seasonality.
    *   **Implementation:** Create a CloudWatch alarm with anomaly detection to detect unusual CPU patterns.

        ```bash
        ANOMALY_ALARM_NAME="iatd_labs_aws_03_anomaly_alarm"
        
        aws cloudwatch put-metric-alarm \
            --alarm-name $ANOMALY_ALARM_NAME \
            --alarm-description "Alarm when CPU behaves anomalously" \
            --metric-name CPUUtilization \
            --namespace AWS/EC2 \
            --statistic Average \
            --period 300 \
            --threshold-metric-id ad1 \
            --comparison-operator GreaterThanUpperThreshold \
            --dimensions Name=InstanceId,Value=$INSTANCE_ID \
            --evaluation-periods 2 \
            --alarm-actions arn:aws:sns:$AWS_REGION:$(aws sts get-caller-identity --query 'Account' --output text):$ANOMALY_ALARM_NAME \
            --threshold-metric-id "ad1" \
            --metrics '[{"Id":"m1","MetricStat":{"Metric":{"Namespace":"AWS/EC2","MetricName":"CPUUtilization","Dimensions":[{"Name":"InstanceId","Value":"'$INSTANCE_ID'"}]},"Period":300,"Stat":"Average"},"ReturnData":true},{"Id":"ad1","Expression":"ANOMALY_DETECTION_BAND(m1, 2)","Label":"CPUUtilization (expected)"}]' \
            --region $AWS_REGION
        ```

### Part 5: AWS CloudTrail for Audit Logging

1.  **Enable CloudTrail:**

    ```bash
    TRAIL_NAME="iatd_labs_aws_03_trail"
    S3_BUCKET="iatd-labs-aws-03-trail" # Replace with a unique bucket name
    
    # Create S3 bucket for CloudTrail logs
    aws s3 mb s3://$S3_BUCKET --region $AWS_REGION
    
    # Create CloudTrail trail
    aws cloudtrail create-trail \
        --name $TRAIL_NAME \
        --s3-bucket-name $S3_BUCKET \
        --is-multi-region-trail \
        --region $AWS_REGION
    
    # Start logging for the trail
    aws cloudtrail start-logging \
        --name $TRAIL_NAME \
        --region $AWS_REGION
    ```

    **Expected Output for create-trail:**
    ```json
    {
        "Name": "iatd_labs_aws_03_trail",
        "S3BucketName": "iatd-labs-aws-03-trail",
        "IsMultiRegionTrail": true,
        "TrailARN": "arn:aws:cloudtrail:us-east-1:123456789012:trail/iatd_labs_aws_03_trail"
    }
    ```

2.  **Explore CloudTrail Logs:**
    *   Navigate to the CloudTrail console in the AWS Management Console.
    *   Select **Event history** from the left navigation pane.
    *   Explore the recorded API activity in your AWS account.

### Post-Lab: Cleanup

To avoid incurring unnecessary costs, it's essential to clean up the resources created during this lab.

1.  **Delete CloudWatch Alarms:**

    ```bash
    aws cloudwatch delete-alarms \
        --alarm-names $ALARM_NAME $DISK_ALARM_NAME $ANOMALY_ALARM_NAME \
        --region $AWS_REGION
    ```

2.  **Delete CloudWatch Dashboard:**

    ```bash
    aws cloudwatch delete-dashboards \
        --dashboard-names "iatd_labs_aws_03_dashboard" \
        --region $AWS_REGION
    ```

3.  **Delete CloudWatch Synthetics Canary:**

    ```bash
    aws synthetics delete-canary \
        --name $CANARY_NAME \
        --region $AWS_REGION
    ```

4.  **Delete CloudWatch Log Group:**

    ```bash
    aws logs delete-log-group \
        --log-group-name $LOG_GROUP_NAME \
        --region $AWS_REGION
    ```

5.  **Delete CloudTrail Trail:**

    ```bash
    aws cloudtrail delete-trail \
        --name $TRAIL_NAME \
        --region $AWS_REGION
    ```

6.  **Delete S3 Buckets:**

    ```bash
    aws s3 rm s3://$S3_BUCKET --recursive
    aws s3 rb s3://$S3_BUCKET --force
    
    aws s3 rm s3://iatd-labs-aws-03-canary-results --recursive
    aws s3 rb s3://iatd-labs-aws-03-canary-results --force
    ```

7.  **Terminate EC2 Instance:**

    ```bash
    aws ec2 terminate-instances \
        --instance-ids $INSTANCE_ID \
        --region $AWS_REGION
    ```

8.  **Delete Security Group:**

    ```bash
    # Wait for EC2 instance to terminate
    aws ec2 wait instance-terminated \
        --instance-ids $INSTANCE_ID \
        --region $AWS_REGION
    
    # Delete security group
    aws ec2 delete-security-group \
        --group-id $SG_ID \
        --region $AWS_REGION
    ```

9.  **Delete Route Table:**

    ```bash
    # Disassociate route table from subnet
    ASSOCIATION_ID=$(aws ec2 describe-route-tables \
        --route-table-ids $RT_ID \
        --region $AWS_REGION \
        --query 'RouteTables[0].Associations[0].RouteTableAssociationId' \
        --output text)
    
    aws ec2 disassociate-route-table \
        --association-id $ASSOCIATION_ID \
        --region $AWS_REGION
    
    # Delete route table
    aws ec2 delete-route-table \
        --route-table-id $RT_ID \
        --region $AWS_REGION
    ```

10. **Detach and Delete Internet Gateway:**

    ```bash
    # Detach internet gateway from VPC
    aws ec2 detach-internet-gateway \
        --internet-gateway-id $IGW_ID \
        --vpc-id $VPC_ID \
        --region $AWS_REGION
    
    # Delete internet gateway
    aws ec2 delete-internet-gateway \
        --internet-gateway-id $IGW_ID \
        --region $AWS_REGION
    ```

11. **Delete Subnet:**

    ```bash
    aws ec2 delete-subnet \
        --subnet-id $SUBNET_ID \
        --region $AWS_REGION
    ```

12. **Delete VPC:**

    ```bash
    aws ec2 delete-vpc \
        --vpc-id $VPC_ID \
        --region $AWS_REGION
    ```

13. **Verify Deletion:**
    *   Confirm that all resources have been deleted by checking the AWS Management Console or using the AWS CLI.

**Outcomes to be Achieved:**

*   Understand different monitoring types in AWS (CloudWatch metrics, logs, synthetics).
*   Analyze logs using CloudWatch Logs Insights.
*   Establish performance baselines and set alert thresholds using CloudWatch Alarms.
*   Configure and use core AWS monitoring services (CloudWatch, CloudTrail).
*   Implement VPC Flow Logs for network traffic analysis.

**Congratulations!** You have successfully completed this lab on monitoring and logging in AWS. You've learned about CloudWatch, CloudTrail, and AWS Config to monitor resources, analyze logs, establish performance baselines, and set alert thresholds.
