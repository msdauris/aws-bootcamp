# Session 3: Scaling & Load Balancing

## 📋 Table of Contents

- [16. Application Load Balancer Deployment](#16-application-load-balancer-deployment)
- [17. ALB Configuration Walkthrough](#17-alb-configuration-walkthrough)
- [18. Connection Draining Setup](#18-connection-draining-setup)
- [19. Auto Scaling Configuration](#19-auto-scaling-configuration)
- [20. Auto Scaling Group Deployment](#20-auto-scaling-group-deployment)
- [21. Session Checkpoint & Cleanup](#21-session-checkpoint--cleanup)

---

## 16. Application Load Balancer Deployment

### Deploy ALB for Multi-Instance Routing

**Objective:** Deploy an Application Load Balancer to route traffic to different EC2 instances based on ports (e.g., Author and Publisher AEM instances)

#### Architecture Overview

The load balancer will:
- Listen for traffic on port 80 (or 443 for HTTPS)
- Route traffic to respective target groups:

- In aem we'd for example route to:
  - **Author instances:** Port 4502
  - **Publisher instances:** Port 4503
- Ensure standard web traffic (port 80/443) is correctly routed to AEM instances on their respective ports

#### Security Considerations
- Configure security groups to allow traffic on required ports
- Restrict access to only trusted sources
- Use HTTPS with SSL certificates for production environments

#### 🎯 Outcome
Traffic routing based on application requirements with proper port mapping and security controls.

---

## 17. ALB Configuration Walkthrough

#### Prerequisites for saregent - show operators!
1. Create NAT Gateway FIRST (before launching instances)
   see https://github.com/msdauris/aws-bootcamp/blob/main/vpc.png 

- Create NAT Gateway in your public subnet
- Ensure it gets an Elastic IP
- Critical: Do this before launching private instances

2. Configure Route Tables

- Private subnet route table should have:
- 0.0.0.0/0 → NAT Gateway (for internet access)
- Local VPC CIDR → Local (for internal communication)

Now your instances will have internet access through NAT Gateway
They can download packages, updates, software during launch

2. Launch two private instances

- Private A
- Private B

> ✅ **Tip:** Think **first boot, root access** — automates setup without manual login.

   ```bash
   #!/bin/bash
   # Install and configure Apache web server
   yum update -y
   yum install -y httpd
   systemctl start httpd
   systemctl enable httpd
   echo "<h1>Hello World from $(hostname -f)</h1>" > /var/www/html/index.html
   ```

### Step-by-Step ALB Setup

**Objective:** Configure Application Load Balancer with detailed instructions

#### Steps

1. **Navigate to Load Balancer Console**
   - Go to EC2 Dashboard in AWS Console
   - Under **Load Balancing**, select **Load Balancers**
   - Click **Create Load Balancer**

2. **Choose Load Balancer Type**
   - Select **Application Load Balancer**

3. **Basic Configuration**
   - **Name:** Provide descriptive name for your ALB
   - **Scheme:** Internet-facing (for public access)
   - **IP Address Type:** IPv4

4. **Availability Zones**
   - Select your VPC
   - Choose at least 2 availability zones within public 
   - Select appropriate public subnets

5. **Security Groups**
   - Select aws-bootcamp-public-subnet (web)
   - Allow traffic on required ports:
     - HTTP: Port 80
     - HTTPS: Port 443
     - Custom ports: 4502, 4503

6. **Configure Routing**
   - Create **Target Groups**:

   **Example Author Target Group:**
   - **Name:** `author-target-group`
   - **Protocol:** HTTP
   - **Port:** 4502
   - **Health Check Path:** `/system/console/bundles`

   **Example Publisher Target Group:**
   - **Name:** `publisher-target-group`
   - **Protocol:** HTTP
   - **Port:** 4503
   - **Health Check Path:** `/system/console/bundles`
  
7. **Listeners Configuration**
   - Add **HTTP (port 80)** listener
   - Add **HTTPS (port 443)** listener (if needed)
   - If using HTTPS, assign SSL certificate from ACM

8. **Register Targets**

9. **Create Load Balancer**
   - Review all settings
   - Click **Create**
   - Wait for ALB to provision (typically 2-5 minutes)

#### Verification

1. **Check ALB Status**
   - Ensure ALB status shows as **Active**

2. **Verify Target Health**
   - Go to Target Groups
   - Check that health checks show **Healthy** for all targets

3. **Test Load Balancer**
   - Use ALB DNS name to access applications
   - Verify traffic routing to correct instances

---

## 18. Connection Draining Setup

### Configure Graceful Instance Removal

**Objective:** Allow active connections to complete before an instance is removed from the ALB (graceful shutdown)

#### Understanding Connection Draining

Connection draining (deregistration delay) ensures:
- Existing requests to de-registering instances complete
- Smooth operation during instance maintenance or failure
- No abrupt disconnection of active users

#### Configuration Steps

1. **Navigate to Target Groups**
   - Go to EC2 Dashboard
   - Under **Load Balancing**, select **Target Groups**

2. **Select Target Group**
   - Choose the target group to configure

3. **Modify Attributes**
   - Click the **Attributes** tab
   - Click **Edit attributes**

4. **Set Deregistration Delay**
   - **Deregistration delay timeout:** 600 seconds (recommended)
   - Range: 0-3600 seconds
   - Adjust based on application requirements

5. **Save Configuration**
   - Click **Save**

#### Important Notes

- Setting applies to **all targets** in the target group
- For different settings per target, use separate target groups
- Part of comprehensive high availability strategy
- Consider auto-scaling, multi-AZ deployment, and health monitoring

#### 🧠 Benefits
- Prevents client disconnections during maintenance
- Enables graceful scaling operations
- Improves user experience during deployments

---

## 19. Auto Scaling Configuration

### Understanding Auto Scaling Components

**Objective:** Configure Auto Scaling for EC2 instances to handle varying load demands

#### Auto Scaling Components

**Groups:**
- Logical components of Auto Scaling
- Examples: Web server group, database group

**Configuration Templates:**
- Launch templates or launch configurations
- Define how to configure and launch new instances
- Specify AMI, instance type, security groups, etc.

**Scaling Options:**
- **Manual Scaling:** Manually adjust capacity
- **Scheduled Scaling:** Scale based on time
- **Dynamic Scaling:** Scale based on metrics
- **Predictive Scaling:** ML-based scaling

#### Create Launch Template

1. **Navigate to Launch Templates**
   - Go to EC2 Dashboard
   - Under **INSTANCES**, click **Launch Templates**
   - Click **Create launch template**

2. **Template Configuration**
   - **Template name:** `web-server-template`
   - **Description:** Template for web server instances

3. **Launch Template Contents**
   - **AMI:** Select appropriate Amazon Machine Image
   - **Instance type:** Choose suitable instance type (e.g., t3.micro)
   - **Key pair:** Select existing key pair
   - **Security groups:** Configure appropriate security groups
   - **Storage:** Configure EBS volumes as needed

4. **Advanced Details**
   - **IAM instance profile:** Select role if needed
   - **User data:** Include startup scripts
   - **Monitoring:** Enable detailed monitoring (optional)

5. **Create Template**
   - Review configuration
   - Click **Create launch template**

#### User Data Example

```bash
#!/bin/bash
yum update -y
yum install -y httpd
systemctl start httpd
systemctl enable httpd
echo "<h1>Auto Scaled Web Server - $(hostname -f)</h1>" > /var/www/html/index.html
```

---

## 20. Auto Scaling Group Deployment

### Create and Configure Auto Scaling Group

**Objective:** Deploy Auto Scaling Group using the launch template

#### Steps

1. **Navigate to Auto Scaling Groups**
   - Go to EC2 Dashboard
   - Under **AUTO SCALING**, click **Auto Scaling groups**
   - Click **Create Auto Scaling group**

2. **Choose Launch Template**
   - **Launch template:** Select previously created template
   - **Version:** Choose `$Latest` or specific version
   - Click **Next**

3. **Configure Settings**
   - **Group name:** `web-server-asg`
   - **Group size:**
     - **Desired capacity:** 2
     - **Minimum capacity:** 1
     - **Maximum capacity:** 5

4. **Network Configuration**
   - **VPC:** Select appropriate VPC
   - **Subnets:** Choose multiple subnets across AZs
   - **Load balancing:** Attach to existing load balancer
   - **Target groups:** Select previously created target groups

5. **Health Checks**
   - **Health check type:** ELB (recommended)
   - **Health check grace period:** 300 seconds
   - **Enable group metrics collection:** ✅

6. **Group Size and Scaling**
   - Configure scaling policies (optional):

   **Scale Out Policy:**
   - **Metric:** CPU Utilization
   - **Target value:** 70%
   - **Instance warm-up:** 300 seconds

   **Scale In Policy:**
   - **Metric:** CPU Utilization
   - **Target value:** 30%
   - **Cooldown period:** 300 seconds

7. **Add Notifications** (Optional)
   - Configure SNS notifications for scaling events

8. **Add Tags**
   - **Key:** `Environment`
   - **Value:** `Production`
   - **Key:** `Application`
   - **Value:** `WebServer`

9. **Review and Create**
   - Review all settings
   - Click **Create Auto Scaling group**

#### Verification

1. **Check Auto Scaling Group**
   - Verify ASG is created and active
   - Check current capacity matches desired capacity

2. **Monitor Instance Launch**
   - Watch instances launch automatically
   - Verify instances register with load balancer

3. **Test Scaling**
   - **Load Testing:** Artificially increase load to trigger scale-out
   - **Scheduled Actions:** Set future scaling actions for testing
   - **Manual Scaling:** Temporarily adjust desired capacity

#### 🧠 Important Notes

- Auto Scaling Group may take several minutes to launch new instances
- Instance termination during scale-in also takes time
- Monitor CloudWatch metrics for scaling decisions
- Consider using multiple instance types for cost optimization

---

## 21. Session Checkpoint & Cleanup

### Resource Management and Cost Control

**Objective:** Clean up resources to avoid unwanted charges and review session learnings

#### Cleanup Tasks

1. **Auto Scaling Group Cleanup**
   - Set desired capacity to 0
   - Wait for instances to terminate
   - Delete Auto Scaling group

2. **Load Balancer Cleanup**
   - Delete Application Load Balancer
   - Delete Target Groups
   - Remove associated security groups (if not needed)

3. **Launch Template Cleanup**
   - Delete launch templates (if not needed for future use)

4. **EC2 Instance Cleanup**
   - Terminate any manually launched instances
   - Verify all instances are terminated

#### Cost Optimization Tips

- Use **Reserved Instances** for predictable workloads
- Consider **Spot Instances** for fault-tolerant applications
- Monitor usage with **AWS Cost Explorer**
- Set up **billing alerts** for cost control

#### Session Review

**Key Concepts Covered:**
- ✅ Application Load Balancer configuration and routing
- ✅ Target Groups and health checks
- ✅ Connection draining for graceful scaling
- ✅ Auto Scaling Groups and launch templates
- ✅ Scaling policies and monitoring

**Practical Skills Gained:**
- ✅ Deploy and configure ALB with multiple target groups
- ✅ Set up connection draining for high availability
- ✅ Create launch templates for consistent instance deployment
- ✅ Configure Auto Scaling Groups with scaling policies
- ✅ Monitor and test scaling behavior

#### Next Steps

Participants are now ready for **Session 4: Databases & Analytics**, where they'll explore managed database services and serverless analytics solutions.

#### 🎯 Learning Outcomes

- Understand load balancing concepts and implementation
- Configure automatic scaling based on demand
- Implement high availability and fault tolerance
- Apply best practices for cost optimization
- Prepare for database and analytics services
