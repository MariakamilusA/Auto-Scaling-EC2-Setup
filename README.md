# Auto-Scaling EC2 Setup

This project demonstrates how to set up a highly available, auto-scaling web application on AWS using Amazon EC2, Application Load Balancer (ALB), and Auto Scaling Groups (ASG). 

The infrastructure is defined as Code (IaC) using AWS CloudFormation, which allows you to deploy everything with a single click.

## Architecture Overview
This CloudFormation template automatically provisions:
- **VPC**: A custom Virtual Private Cloud with 2 Public Subnets across 2 Availability Zones.
- **Internet Gateway**: To provide internet access to the VPC.
- **Security Groups**: 
  - ALB SG: Allows HTTP traffic from the internet.
  - WebServer SG: Allows HTTP traffic ONLY from the ALB, and SSH traffic from anywhere.
- **Application Load Balancer**: Distributes incoming HTTP traffic across multiple EC2 instances.
- **Launch Template**: Configures the EC2 instances. It installs `httpd` (Apache Web Server) and `stress` (a tool to simulate CPU load) automatically upon launch using User Data.
- **Auto Scaling Group**: Maintains a desired capacity of 2 instances, scaling between 1 and 4 instances.
- **Target Tracking Scaling Policy**: Automatically adds more instances if average CPU utilization goes above 50%.

## Prerequisites
1. An AWS Account.
2. (Optional) An EC2 Key Pair created in your AWS account if you want to SSH into the instances.

## Deployment Instructions

1. Log in to the [AWS Management Console](https://console.aws.amazon.com/).
2. Navigate to the **CloudFormation** service.
3. Click **Create stack** > **With new resources (standard)**.
4. Under "Prerequisite - Prepare template", select **Template is ready**.
5. Under "Specify template", select **Upload a template file**, click **Choose file**, and select the `template.yaml` file from this repository. Click **Next**.
6. Enter a **Stack name** (e.g., `AutoScalingProject`).
7. (Optional) Under **Parameters**, select an existing EC2 `KeyName` if you have one. Click **Next**.
8. Leave "Configure stack options" as default and click **Next**.
9. Review the details, scroll to the bottom, and click **Submit**.
10. Wait a few minutes for the status to change to **CREATE_COMPLETE**.

## Testing the Application

1. In the CloudFormation console, select your stack and click on the **Outputs** tab.
2. Copy the URL next to `LoadBalancerDNS` and paste it into your web browser.
3. You should see a webpage saying: **"Hello from Auto-Scaled EC2 Instance!"** along with the unique Instance ID.
4. Refresh the page a few times. You might notice the Instance ID changing as the Load Balancer routes your request to different instances.

## Testing Auto-Scaling (Simulating High CPU Load)

To see the Auto-Scaling in action, we can simulate high CPU usage on one of the instances.

1. Connect to one of your EC2 instances via SSH or EC2 Instance Connect from the AWS Console.
2. Run the `stress` tool (which was pre-installed via our launch template):
   ```bash
   stress --cpu 4 --timeout 300
   ```
   *(This command stresses the CPU for 5 minutes, simulating high load).*
3. Go back to your AWS Console and open the **EC2 Dashboard** > **Auto Scaling Groups**.
4. Select your group and navigate to the **Activity** tab.
5. After a couple of minutes, the CPU utilization metric will breach the 50% threshold, and you will see new activity indicating that AWS is automatically launching a new EC2 instance to handle the load.
6. Once the `stress` command finishes (after 5 minutes), the CPU load will drop. After a cool-down period, the ASG will terminate the extra instances to save costs.

## Cleanup

To avoid incurring future charges on your AWS account, make sure to delete the resources when you are done.

1. Go to the **CloudFormation** console.
2. Select your stack (e.g., `AutoScalingProject`).
3. Click **Delete**.
4. This will automatically terminate all EC2 instances, Load Balancers, and networking resources created for this project.
