# Highly Available Web Application on AWS

A hands-on AWS project that hosts a static website on **EC2**, puts it behind a **Classic Load Balancer**, and keeps it available and scalable with an **Auto Scaling Group**. **CloudWatch** monitors CPU utilization and **SNS** sends email alerts.

## Architecture

```
                 Internet
                    |
          Classic Load Balancer
             /              \
     EC2 Instance        EC2 Instance   (...up to 5)
             \              /
            Auto Scaling Group
                    |
          CloudWatch Monitoring
                    |
            SNS Email Alerts
```

## Learning Objectives

- Launch and configure EC2 instances
- Install and host a web application
- Create an Amazon Machine Image (AMI)
- Create a Launch Template
- Configure a Classic Load Balancer (CLB)
- Configure an Auto Scaling Group (ASG)
- Create CloudWatch alarms
- Configure SNS email notifications
- Verify high availability

## AWS Services Used

| Service | Purpose |
| --- | --- |
| EC2 | Host the website |
| Classic Load Balancer | Distribute incoming traffic |
| Auto Scaling Group | Automatically add/remove EC2 instances |
| CloudWatch | Monitor CPU utilization |
| SNS | Send email notifications |
| AMI | Reusable server image |
| Launch Template | Template for new EC2 instances |

## Prerequisites

- An AWS account
- A Key Pair
- A Security Group (SSH 22, HTTP 80)
- An email address (for SNS)

**Region used:** Asia Pacific (Hyderabad) `ap-south-2`

---

## Step 1 – Launch an EC2 Instance

Launch an EC2 instance with:

- Amazon Linux 2
- Security Group allowing **SSH (22)** and **HTTP (80)**
- An existing Key Pair

## Step 2 – Connect to EC2

Connect using PuTTY or EC2 Instance Connect from the console.

## Step 3 – Install Apache Web Server

```bash
sudo yum update -y            # update packages
sudo yum install httpd -y     # install Apache
sudo systemctl enable httpd   # start on boot
sudo systemctl start httpd    # start now
sudo systemctl status httpd   # verify
```

Expected status: `active (running)`

## Step 4 – Create the Web Page

Download any free template from [BootstrapMade](https://bootstrapmade.com/) and place it in `/var/www/html/`.

Verify by opening `http://<Public-IP>` in a browser.

## Step 5 – Create an Amazon Machine Image (AMI)

1. Select the EC2 instance
2. **Actions → Image and templates → Create image**
3. Wait until the AMI status is **Available**

![AMIs](images/03-amis.png)

## Step 6 – Create a Launch Template

**EC2 → Launch Templates → Create launch template**

| Setting | Value |
| --- | --- |
| Name | `pro1` / `pro2` |
| AMI | The AMI created in Step 5 |
| Instance type | `t3.micro` (free-tier eligible types such as `t2.micro` also work) |
| Key pair | Existing key pair |
| Security group | Existing security group |

![Launch Templates](images/02-launch-templates.png)

## Step 7 – Create a Classic Load Balancer

**EC2 → Load Balancers → Create load balancer → Classic Load Balancer**

| Setting | Value |
| --- | --- |
| Listener | HTTP : 80 |
| Availability Zones | Same AZs as the instances |
| Security group | Allow HTTP |
| Health check | HTTP, port 80, path `/` |
| Healthy threshold | 2 |
| Unhealthy threshold | 2 |
| Timeout | 5 s |
| Interval | 30 s |

![Load Balancer](images/04-load-balancer.png)

## Step 8 – Create an Auto Scaling Group

**EC2 → Auto Scaling Groups → Create Auto Scaling group**

| Setting | Value |
| --- | --- |
| Launch template | `pro1` |
| Network | Your VPC and subnets |
| Load balancer | Attach the existing Classic Load Balancer |
| Desired capacity | 2 |
| Minimum | 2 |
| Maximum | 5 |
| Health check type | ELB + EC2 |

![Auto Scaling Group](images/06-auto-scaling-group.png)

## Step 9 – Verify Auto Scaling & Load Balancing

Expected: the ASG launches EC2 instances and they register with the load balancer with status **InService**.

![EC2 Instances](images/01-ec2-instances.png)

![Target instances in the CLB](images/05-clb-target-instances.png)

Open the load balancer's DNS name in a browser to confirm the site loads.

## Step 10 – Create an SNS Topic

**SNS → Topics → Create topic** → Type: **Standard** → give it a name.

![SNS Topic](images/09-sns-topic.png)

## Step 11 – Subscribe an Email

**Create subscription** → Protocol: **Email** → enter your address → confirm the subscription from the email AWS sends. Status should show **Confirmed**.

![SNS Subscription](images/10-sns-subscription.png)

## Step 12 – CloudWatch Alarm & Scaling Policy

Create a CloudWatch alarm on **CPUUtilization** that triggers the Auto Scaling policy and notifies the SNS topic.

![CloudWatch Alarm](images/08-cloudwatch-alarm.png)

## Results

The Auto Scaling activity history shows the group:

- Launching its initial instances (0 → 2)
- Scaling out when the alarm fired (2 → 3 → 4 → 5)
- Automatically replacing a terminated instance (health-check replacement)

![ASG Activity History](images/07-asg-activity-history.png)

## Cleanup (avoid charges)

Delete in this order: Auto Scaling Group → Load Balancer → Launch Templates → AMIs (and their snapshots) → SNS topic → CloudWatch alarm → any leftover EC2 instances.

## Key Takeaways

- A Load Balancer spreads traffic across multiple instances.
- An ASG replaces unhealthy instances automatically and scales with demand.
- AMIs and Launch Templates make new servers identical and repeatable.
- CloudWatch + SNS give you monitoring and alerting.

## Author

**Lavanya** – AWS cloud project
