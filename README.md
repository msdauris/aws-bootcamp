## 📋 Table of Contents

1. [Information for Students](#-aws-bootcamp)  
2. [Infrastructure & Architecture](#1-infrastructure-architecture)  
3. [VPC and Networking Configuration](#2-vpc-and-networking-configuration)  
4. [Security Group Configuration](#3-security-group-configuration)  
5. [EC2 Instance Deployment Strategy](#4-ec2-instance-deployment-strategy)  
6. [S3 Bucket Configuration](#5-s3-bucket-configuration)  
7. [Workshop Execution Guide](#6-workshop-execution-guide)  
8. [Load Balancer Configuration](#7-load-balancer-configuration)  
9. [Monitoring and Troubleshooting](#8-monitoring-and-troubleshooting)  
10. [Workshop Cleanup Procedures](#9-workshop-cleanup-procedures)  
11. [Advanced Workshop Extensions](#10-advanced-workshop-extensions)  
12. [Summary](#summary)

---

# 🪖 AWS Bootcamp

**Your Mission Begins Now**

- The cloud collapsed. Only the skilled survived.  
- This 4-day AWS Bootcamp is your crash course in survival — built for those aiming to pass the Cloud Practitioner or prepare for the Solutions Architect exam.  
- No fluff. No theory marathons. Just critical AWS skills, fast.

### ✅ What You'll Master

- 🚀 Launch and connect to EC2  
- 💾 Store and secure data with S3  
- ⚖️ Load balance, scale, and recover  
- 📊 Query data via RDS and Athena  

Alone or with your squad, complete the hands-on drills. Earn your place. Build momentum. Stay sharp.  
At the end? You’ll walk away with real AWS skills — and the fire to go further.  
**The world runs in the cloud now. Do your duty.**

---

## 🔐 AWS Account & Region

- **Login:** [AWS Console](https://signin.aws.amazon.com/console)  
- **Region:** `eu-west-1 (Ireland)` — **LOCKED**

---

### 🏷️ Tagging Convention

All deployed resources must include the following tags:

- `aws-bootcamp`  
- `Owner=[your-name-or-team]`

---

### 🎯 Operation Cloudstrike Checklist

By the end of the mission, each recruit will have:

- 🎨 Customized the AWS Console (theme + bookmarks)  
- 🛡️ Created IAM user, role, and custom policy  
- 🔍 Tested access using IAM Policy Simulator  
- 💻 Launched an EC2 instance and connected via SSH  
- 🔧 Installed a web server (Apache or Nginx)  
- 💾 Attached and mounted an EBS volume  
- 📦 Created and managed an S3 bucket  
- 🔐 Applied S3 permissions and tested object access  
- ⚖️ Set up an Application Load Balancer + listener rules  
- 📈 Configured an Auto Scaling Group and tested scale events  
- 🗄️ Deployed an RDS instance and connected via CLI  
- 📊 Ran SQL queries (CREATE, INSERT, SELECT, UPDATE, DELETE)  
- 🕵️ Queried structured data from S3 using Athena

# AWS Workshop Complete Network Setup Guide - for facilitators.

## Overview
This comprehensive setup provides pre-configured networking infrastructure for the AWS Compute & Storage workshop, eliminating common networking issues and allowing students to focus on core AWS services.

---

## 1. Infrastructure Architecture

### Workshop Topology
```
Internet Gateway
    |
┌───▼────────────────────────────────────────────────────────┐
│                aws-bootcamp-vpc (10.0.0.0/16)              │
│                                                            │
│  ┌─────────────────────┐    ┌─────────────────────┐        │
│  │   Public Subnet A   │    │   Public Subnet B   │        │
│  │   10.0.1.0/24       │    │   10.0.2.0/24       │        │
│  │   (eu-west-1a)      │    │   (eu-west-1b)      │        │
│  │                     │    │                     │        │
│  │ • EC2 Instance      │    │ • (OPT) Web Server 2│        │
│  │ • ALB Target        │    │ • (OPT)ALB Target   │        │
│  │                     │    │                     │        │
│  └─────────┬───────────┘    └─────────────────────┘        │
│            │                                               │
│  ┌─────────▼───────────┐    ┌─────────────────────┐        │
│  │  Private Subnet A   │    │  Private Subnet B   │        │
│  │   10.0.11.0/24      │    │   10.0.12.0/24      │        │
│  │   (eu-west-1a)      │    │   (eu-west-1b)      │        │
│  │                     │    │                     │        │
│  │ • Database Server   │    │ • (OPT)App Server 2 │        │
│  │                     │    │                     │        │
│  └─────────────────── ─┘    └─────────────────────┘        │
└────────────────────────────────────────────────────────────┘
```

---

## 2. VPC and Networking Configuration

### Core Network Resources

| **Component** | **Configuration** | **Purpose** |
|---------------|-------------------|-------------|
| **Workshop VPC** | 10.0.0.0/16 (aws-workshop-vpc) ✅ Enable DNS resolution* ✅ Enable DNS hostnames* | Main workshop environment |
| **Default VPC** | 172.31.0.0/16 (aws-workshop-default) | Fallback/comparison environment |
| **Internet Gateway** | Attached to workshop VPC | Public internet access |
| **NAT Gateway** | Public Subnet A (10.0.1.0/24) | Private subnet internet access |

- *RDS needs DNS to assign a hostname like mydb.xxxx.rds.amazonaws.com
- *Without DNS hostnames/resolution, private subnets can't resolve that endpoint internally either


### Subnet Architecture

| **Subnet Type** | **CIDR Block** | **Availability Zone** | **Use Case** |
|-----------------|----------------|----------------------|--------------|
| **Public Subnet A** | 10.0.1.0/24 | eu-west-1a | Web servers, bastion host, NAT gateway |
| **Public Subnet B** | 10.0.2.0/24 | eu-west-1b | Load balancer targets, redundancy |
| **Private Subnet A** | 10.0.11.0/24 | eu-west-1a | Application servers, databases |
| **Private Subnet B** | 10.0.12.0/24 | eu-west-1b | App server redundancy |

### Route Tables

| **Route Table** | **Subnet Association** | **Routes** |
|-----------------|------------------------|------------|
| **Public RT** | Public Subnets A & B | 0.0.0.0/0 → Internet Gateway |
| **Private RT** | Private Subnets A & B | No 0.0.0.0/0 route (local VPC only) |

---

## 3. Security Group Configuration

### AEM Example Tiered Security Architecture

```
Internet (Port 80/443)
    ↓
┌─────────────────────────┐
│  Application Load       │
│  Balancer               │
│  - Port 80 → 4502/4503  │
└─────────┬───────────────┘
          │
    ┌─────┴─────┐
    │           │
    ▼           ▼
┌─────────┐ ┌─────────┐
│AEM      │ │AEM      │
│Author   │ │Publisher│
│:4502    │ │:4503    │
│(Private)│ │(Private)│
└─────────┘ └─────────┘
    │           │
    └─────┬─────┘
          ▼
    ┌─────────┐
    │Database │
    │:3306    │
    │(Private)│
    └─────────┘
```

### Security Group Details

#### Web Tier Security Groups

🟢 1. Web Tier SG (aws-bootcamp-sg-web001)
```
Inbound:
- HTTP (80) from 0.0.0.0/0
- HTTPS (443) from 0.0.0.0/0
- SSH (22) from YOUR_IP/32  # Optional if direct access needed

Outbound:
- MySQL (3306) to sg-db001
- All traffic (0.0.0.0/0)  # Or restrict more tightly
```
🔸 Purpose: Public EC2 instance for testing Apache and RDS access

🟢 2. Database Tier SG (aws-bootcamp-sg-db001)
```
Inbound:
- MySQL (3306) from sg-web001

Outbound:
- All traffic (0.0.0.0/0)  # Optional, typically not used
```
🔸 Purpose: Private DB access from web instance only

---

## 4. EC2 Instance Deployment Strategy

### Instance Placement Matrix

| **Instance** | **Type** | **Subnet** | **Security Group** | **Public IP** | **Purpose** |
|--------------|----------|------------|-------------------|---------------|-------------|
| **Bastion Host (🟡 Optional)** |   t3.micro  | Public A (10.0.1.0/24)  | aws-bootcamp-sg-bastion001  |     Yes    |   SSH gateway  |
| **Web Server 1** |   t3.micro  | Public A (10.0.1.0/24)  | aws-bootcamp-sg-web001 |     Yes    |   Apache/Nginx  |
| **Web Server 2 (🟡 Optional)** |  t3.micro  | Public B (10.0.2.0/24)   |aws-bootcamp-sg-web001 |    Yes     |   Load balancer target|
| **App Server 1 (🟡 Optional)** | t3.micro  | Private A (10.0.11.0/24) |aws-bootcamp-sg-app001 |   No       |   AEM Author (4502)|
| **App Server 2 (🟡 Optional)** | t3.micro  | Private B (10.0.12.0/24) |aws-bootcamp-sg-app001 |   No       |   AEM Publisher (4503)|
| **Database** |      t3.micro | Private A (10.0.11.0/24) |aws-bootcamp-sg-db001 |   No       |   Data storage|

### User Data Scripts by Instance Type

#### Web Server User Data
```bash
#!/bin/bash
yum update -y
yum install -y httpd
systemctl start httpd
systemctl enable httpd
echo "<h1>Hello World from $(hostname -f)</h1>" > /var/www/html/index.html
```

---

## 5. S3 Bucket Configuration

### Bucket Architecture

| **Bucket Name** | **Purpose** | **Access Level** | **Features** |
|-----------------|-------------|------------------|--------------|
| **aws-workshop-static-[account-id]** | Static website hosting | Public read | Versioning, AES-256 encryption |
| **aws-workshop-logs-[account-id]** | Application logs | Private | Versioning, KMS encryption |
| **aws-workshop-backups-[account-id]** | Instance backups | Private | Versioning, KMS encryption |
| **aws-workshop-data-[account-id]** | Sample data files | Private | Versioning, AES-256 encryption |

### S3 Bucket Policies

#### Static Website Bucket Policy (CLARIFY)
```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Sid": "PublicReadGetObject",
    "Effect": "Allow",
    "Principal": "*",
    "Action": "s3:GetObject",
    "Resource": "arn:aws:s3:::aws-workshop-static-ACCOUNT_ID/*"
  }]
}
```

#### Application Access Policy (CLARIFY)
```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Sid": "AppServerAccess",
    "Effect": "Allow",
    "Principal": {"AWS": "arn:aws:iam::ACCOUNT_ID:role/WorkshopAppRole"},
    "Action": [
      "s3:GetObject",
      "s3:PutObject",
      "s3:DeleteObject"
    ],
    "Resource": [
      "arn:aws:s3:::aws-workshop-logs-ACCOUNT_ID/*",
      "arn:aws:s3:::aws-workshop-data-ACCOUNT_ID/*"
    ]
  }]
}
```

---

## 6. Workshop Execution Guide

### Phase 1: Infrastructure Verification

**Step 1: VPC Validation**
```bash
# Check VPC setup
aws ec2 describe-vpcs --filters "Name=tag:Name,Values=aws-workshop-vpc"
aws ec2 describe-subnets --filters "Name=vpc-id,Values=vpc-xxxxxxxxx"
```

**Step 2: Security Group Verification**
```bash
# List workshop security groups
aws ec2 describe-security-groups --filters "Name=group-name,Values=workshop-*"
```

**Step 3: S3 Bucket Access**
```bash
# Test bucket access
aws s3 ls | grep aws-workshop
aws s3 cp test-file.txt s3://aws-workshop-data-ACCOUNT_ID/
```

### Phase 2: EC2 Instance Deployment

**Instance Launch Checklist:**
1. ✅ Select correct AMI (Amazon Linux 2)
2. ✅ Choose appropriate instance type
3. ✅ Configure network settings:
   - VPC: aws-workshop-vpc
   - Subnet: Based on instance purpose
   - Auto-assign Public IP: Yes for public subnets
4. ✅ Select pre-configured security group
5. ✅ Add user data script
6. ✅ Configure key pair for SSH access

### Phase 3: Application Configuration

**Web Server Setup:**
1. Launch instances in public subnets
2. Install and configure Apache
3. Create custom "Hello World" page
4. Test HTTP access from internet

**OPTIONAL**
**Application Server Setup:**
1. Launch instances in private subnets
2. Install Node.js/Python application
3. Configure application to listen on port 8080
4. Test connectivity from web servers

**Database Setup:**
1. Launch database instance in private subnet
2. Install MySQL/PostgreSQL
3. Configure access from application servers only
4. Test database connectivity

---

## 7. Load Balancer Configuration

### Application Load Balancer Setup

**Target Group Configuration:**
- **Protocol:** HTTP
- **Port:** 80
- **Health Check Path:** /
- **Health Check Interval:** 30 seconds
- **Healthy Threshold:** 2
- **Unhealthy Threshold:** 5

**Load Balancer Configuration:**
- **Scheme:** Internet-facing
- **IP Address Type:** IPv4
- **Subnets:** Public Subnet A and B
- **Security Group:** workshop-alb-sg

**Listener Rules:**
```
Rule 1: Default
- Condition: Default
- Action: Forward to target group (web-servers-tg)

Rule 2: Static Content
- Condition: Path pattern (/static/*)
- Action: Redirect to S3 static website
```

---

## 8. Monitoring and Troubleshooting

### CloudWatch Metrics to Monitor

**EC2 Metrics:**
- CPU Utilization
- Network In/Out
- Disk Read/Write Operations
- Status Check Failed

**EBS Metrics:**
- Volume Read/Write Ops
- Volume Queue Length
- Volume Throughput Percentage
- Burst Balance (for gp2 volumes)

**S3 Metrics:**
- Number of Objects
- Bucket Size Bytes
- All Requests
- 4xx/5xx Errors

### Common Troubleshooting Scenarios

#### Scenario 1: Web Server Not Accessible
**Symptoms:** HTTP timeout or connection refused
**Troubleshooting Steps:**
1. Check security group rules (HTTP port 80 open)
2. Verify instance is in running state
3. Check Apache service status: `systemctl status httpd`
4. Review user data script execution: `/var/log/cloud-init-output.log`
5. Test local connectivity: `curl localhost`

#### Scenario 2: SSH Access Issues
**Symptoms:** Connection timeout or permission denied
**Troubleshooting Steps:**
1. Verify key pair permissions: `chmod 400 key.pem`
2. Check security group SSH rule (port 22)
3. Confirm public IP assignment for public instances
4. For private instances, use bastion host
5. Check NACLs if default rules modified

#### Scenario 3: EBS Performance Issues
**Symptoms:** Slow application response, high I/O wait
**Troubleshooting Steps:**
1. Monitor CloudWatch EBS metrics
2. Check volume type (gp2 vs io1/io2)
3. Review instance type EBS optimization
4. Analyze application I/O patterns
5. Consider volume resizing or type change

#### Scenario 4: S3 Access Denied
**Symptoms:** 403 Forbidden errors
**Troubleshooting Steps:**
1. Verify bucket policy configuration
2. Check IAM role permissions
3. Confirm bucket name spelling
4. Review object-level permissions
5. Test with pre-signed URLs

---

## 9. Workshop Cleanup Procedures

### Automated Cleanup Script

```bash
#!/bin/bash
echo "Starting AWS Workshop Cleanup..."

# Set variables
ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
WORKSHOP_TAG="Workshop=AWS-Session2"

# Terminate EC2 instances
echo "Terminating EC2 instances..."
INSTANCE_IDS=$(aws ec2 describe-instances \
  --filters "Name=tag:$WORKSHOP_TAG" "Name=instance-state-name,Values=running,stopped" \
  --query 'Reservations[].Instances[].InstanceId' --output text)

if [ ! -z "$INSTANCE_IDS" ]; then
  aws ec2 terminate-instances --instance-ids $INSTANCE_IDS
  echo "Waiting for instances to terminate..."
  aws ec2 wait instance-terminated --instance-ids $INSTANCE_IDS
fi

# Delete EBS snapshots
echo "Deleting EBS snapshots..."
SNAPSHOT_IDS=$(aws ec2 describe-snapshots --owner-ids self \
  --filters "Name=tag:$WORKSHOP_TAG" \
  --query 'Snapshots[].SnapshotId' --output text)

for snapshot in $SNAPSHOT_IDS; do
  aws ec2 delete-snapshot --snapshot-id $snapshot
done

# Empty and delete S3 buckets
echo "Cleaning up S3 buckets..."
BUCKETS=("aws-workshop-static-$ACCOUNT_ID" "aws-workshop-logs-$ACCOUNT_ID" 
         "aws-workshop-backups-$ACCOUNT_ID" "aws-workshop-data-$ACCOUNT_ID")

for bucket in "${BUCKETS[@]}"; do
  if aws s3 ls "s3://$bucket" 2>/dev/null; then
    echo "Emptying bucket: $bucket"
    aws s3 rm "s3://$bucket" --recursive
    aws s3 rb "s3://$bucket"
  fi
done

# Delete load balancers
echo "Deleting load balancers..."
LB_ARNS=$(aws elbv2 describe-load-balancers \
  --query 'LoadBalancers[?contains(LoadBalancerName, `workshop`)].LoadBalancerArn' \
  --output text)

for lb_arn in $LB_ARNS; do
  aws elbv2 delete-load-balancer --load-balancer-arn $lb_arn
done

# Delete CloudFormation stack (if used)
echo "Deleting CloudFormation stack..."
aws cloudformation delete-stack --stack-name aws-workshop-infrastructure

echo "Cleanup completed!"
```

### Manual Cleanup Checklist

- [ ] Terminate all EC2 instances
- [ ] Delete EBS volumes and snapshots
- [ ] Empty and delete S3 buckets
- [ ] Delete load balancers and target groups
- [ ] Remove security groups (after instances terminated)
- [ ] Delete CloudFormation stack
- [ ] Verify no remaining charges in billing

---

## 10. Advanced Workshop Extensions

### Optional Advanced Scenarios

#### High Availability Setup
- Deploy Auto Scaling Groups
- Configure multi-AZ RDS
- Implement CloudFront distribution
- Set up Route 53 health checks

#### Security Enhancements
- Implement AWS Systems Manager Session Manager
- Configure VPC Flow Logs
- Set up AWS Config rules
- Enable GuardDuty threat detection

#### Monitoring and Alerting
- Create CloudWatch dashboards
- Set up SNS alerts for critical metrics
- Configure log aggregation with CloudWatch Logs
- Implement X-Ray tracing

#### Cost Optimization
- Implement S3 Intelligent Tiering
- Configure EBS gp3 optimization
- Set up AWS Budgets alerts
- Analyze costs with Cost Explorer

---

## Summary

This comprehensive network setup provides:

✅ **Simplified Learning Experience:** Students focus on EC2, EBS, and S3 rather than networking complexity  
✅ **Production-Ready Architecture:** Proper security layers and high availability design  
✅ **Scalable Infrastructure:** Easy to extend for advanced workshops  
✅ **Cost-Effective:** Efficient resource utilization with cleanup procedures  
✅ **Troubleshooting Ready:** Common scenarios and solutions documented  

The pre-configured infrastructure eliminates networking bottlenecks while maintaining educational value, ensuring students gain hands-on experience with core AWS compute and storage services in a realistic, secure environment.


