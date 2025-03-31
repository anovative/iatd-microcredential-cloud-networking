## IATD Microcredential Cloud Networking: Supplementary Lab - AWS DDoS Mitigation Fundamentals

**Objective:** This lab introduces you to DDoS mitigation techniques using AWS services like AWS Shield, AWS WAF, and Amazon CloudFront. You'll learn how these security mechanisms compare to Azure's DDoS Protection and how to implement layered security in AWS environments.

**Estimated Time:** 90-120 minutes

**Prerequisites:**
1. **AWS Account:** An active AWS account with administrative access
2. **AWS CLI:** Installed and configured with appropriate credentials
3. **Basic Understanding:** 
   - AWS VPC concepts
   - AWS Shield and AWS WAF
   - Basic networking principles
   - EC2 instance management

### Resource Naming Convention

* **EC2 Instance:** `iatd_labs_aws_ddos_ec2`
* **Security Group:** `iatd_labs_aws_ddos_sg`
* **AWS WAF Web ACL:** `iatd_labs_aws_ddos_waf`
* **CloudFront Distribution:** `iatd_labs_aws_ddos_cloudfront` (Optional)

### Part 1: Setting up a Target EC2 Instance (AWS CLI and Console)

1. **Create a VPC and Subnet:**
   ```bash
   # Set variables
   AWS_REGION="us-east-1"
   VPC_CIDR="172.16.0.0/16"
   SUBNET_CIDR="172.16.1.0/24"
   
   # Create VPC
   VPC_ID=$(aws ec2 create-vpc \
     --cidr-block $VPC_CIDR \
     --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=iatd_labs_aws_ddos_vpc}]' \
     --query 'Vpc.VpcId' \
     --output text)
   
   echo "VPC created with ID: $VPC_ID"
   ```

   **Expected Output:**
   ```
   VPC created with ID: vpc-0123456789abcdef0
   ```

2. **Create a Security Group:**
   ```bash
   # Create security group
   SG_ID=$(aws ec2 create-security-group \
     --group-name iatd_labs_aws_ddos_sg \
     --description "Security group for DDoS lab" \
     --vpc-id $VPC_ID \
     --query 'GroupId' \
     --output text)
   
   echo "Security group created with ID: $SG_ID"
   
   # Allow HTTP traffic
   aws ec2 authorize-security-group-ingress \
     --group-id $SG_ID \
     --protocol tcp \
     --port 80 \
     --cidr 0.0.0.0/0
   ```

   **Expected Output:**
   ```
   Security group created with ID: sg-0123456789abcdef0
   ```

3. **Launch an EC2 Instance:**
   ```bash
   # Get the latest Amazon Linux 2 AMI ID
   AMI_ID=$(aws ec2 describe-images \
     --owners amazon \
     --filters "Name=name,Values=amzn2-ami-hvm-*-x86_64-gp2" "Name=state,Values=available" \
     --query "sort_by(Images, &CreationDate)[-1].ImageId" \
     --output text)
   
   # Launch EC2 instance
   INSTANCE_ID=$(aws ec2 run-instances \
     --image-id $AMI_ID \
     --instance-type t2.micro \
     --security-group-ids $SG_ID \
     --subnet-id $SUBNET_ID \
     --associate-public-ip-address \
     --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=iatd_labs_aws_ddos_ec2}]' \
     --query 'Instances[0].InstanceId' \
     --output text)
   
   echo "EC2 instance launched with ID: $INSTANCE_ID"
   ```

   **Expected Output:**
   ```
   EC2 instance launched with ID: i-0123456789abcdef0
   ```

### Part 2: Implementing AWS WAF (AWS CLI and Console)

1. **Create an AWS WAF Web ACL:**
   ```bash
   # Create WAF Web ACL
   aws wafv2 create-web-acl \
     --name iatd_labs_aws_ddos_waf \
     --scope REGIONAL \
     --region $AWS_REGION \
     --default-action Allow={} \
     --visibility-config SampledRequestsEnabled=true,CloudWatchMetricsEnabled=true,MetricName=iatd_labs_aws_ddos_waf \
     --description "Web ACL for DDoS protection"
   ```

   **Expected Output:**
   ```json
   {
       "Summary": {
           "Name": "iatd_labs_aws_ddos_waf",
           "Id": "a1b2c3d4-5678-90ab-cdef-EXAMPLE11111",
           "ARN": "arn:aws:wafv2:us-east-1:123456789012:regional/webacl/iatd_labs_aws_ddos_waf/a1b2c3d4-5678-90ab-cdef-EXAMPLE11111"
       }
   }
   ```

### Part 3: Implementing AWS Shield (Console)

1. **Enable AWS Shield Standard:**
   * AWS Shield Standard is automatically enabled for all AWS customers at no additional cost
   * It provides protection against common and most frequently occurring infrastructure (Layer 3 and 4) attacks

### Part 4: Implementing Amazon CloudFront (AWS CLI and Console)

1. **Create a CloudFront Distribution:**
   ```bash
   # Create CloudFront distribution
   aws cloudfront create-distribution \
     --origin-domain-name $PUBLIC_IP \
     --default-root-object index.html \
     --tags "Items=[{Key=Name,Value=iatd_labs_aws_ddos_cloudfront}]"
   ```

   **Expected Output:**
   ```json
   {
       "Distribution": {
           "Id": "E1EXAMPLE",
           "ARN": "arn:aws:cloudfront::123456789012:distribution/E1EXAMPLE",
           "Status": "InProgress",
           "DomainName": "d1234abcdef8.cloudfront.net"
       }
   }
   ```

### Part 5: Testing DDoS Mitigation

1. **Simulate Legitimate Traffic:**
   ```bash
   # Generate legitimate traffic to your CloudFront distribution
   for i in {1..10}; do
     curl -s https://$CLOUDFRONT_DOMAIN > /dev/null
     echo "Request $i sent"
     sleep 1
   done
   ```

   **Expected Output:**
   ```
   Request 1 sent
   Request 2 sent
   ...
   Request 10 sent
   ```

### Post-Lab: Cleanup

To avoid incurring unnecessary costs, it's essential to clean up the resources created during this lab.

1. **Delete CloudFront Distribution:**
   ```bash
   # Get the distribution ID
   DISTRIBUTION_ID=$(aws cloudfront list-distributions \
     --query "DistributionList.Items[?Tags.Items[?Key=='Name' && Value=='iatd_labs_aws_ddos_cloudfront']].Id" \
     --output text)
   
   # Delete the distribution (after it's disabled and deployed)
   aws cloudfront delete-distribution \
     --id $DISTRIBUTION_ID \
     --if-match <ETag-from-get-distribution-config>
   ```

2. **Delete AWS WAF Web ACL:**
   ```bash
   # Delete the Web ACL
   aws wafv2 delete-web-acl \
     --name iatd_labs_aws_ddos_waf \
     --scope REGIONAL \
     --region $AWS_REGION \
     --lock-token $(aws wafv2 get-web-acl --name iatd_labs_aws_ddos_waf --scope REGIONAL --region $AWS_REGION --query 'WebACL.LockToken' --output text)
   ```

3. **Terminate EC2 Instance:**
   ```bash
   aws ec2 terminate-instances \
     --instance-ids $INSTANCE_ID
   
   echo "Terminating EC2 instance: $INSTANCE_ID"
   ```

   **Expected Output:**
   ```
   Terminating EC2 instance: i-0123456789abcdef0
   ```

**Congratulations!** You've successfully implemented DDoS mitigation techniques in AWS using AWS Shield, AWS WAF, and Amazon CloudFront. This lab has demonstrated key concepts in AWS DDoS protection that parallel the DDoS Protection implementations covered in the Azure labs.

### Troubleshooting Guide

1. **AWS WAF Issues:**
   - Verify that the Web ACL is correctly associated with your resources
   - Check that the managed rule groups are properly configured
   - Review the WAF logs in CloudWatch for any errors

2. **CloudFront Issues:**
   - Ensure that the origin (EC2 instance) is accessible
   - Verify that the CloudFront distribution is enabled and deployed
   - Check the CloudFront access logs for any errors

3. **EC2 Instance Issues:**
   - Verify that the security group allows HTTP traffic
   - Check that the web server is running (`systemctl status httpd`)
   - Review the system logs for any errors (`/var/log/messages`)
