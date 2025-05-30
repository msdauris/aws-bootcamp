# aws-bootcamp

<br>🪖 AWS Bootcamp: Your Mission Begins Now
<br>The cloud collapsed. Only the skilled survived.
<br>This 4-day AWS Bootcamp is your crash course in survival — built for those aiming to pass the Cloud Practitioner or prepare for the Solutions Architect exam.
<br>No fluff. No theory marathons. Just critical AWS skills, fast.

<br>✅ Launch EC2
<br>✅ Store in S3
<br>✅ Load balance, scale, secure
<br>✅ Query, replicate, recover

<br>Alone or with your squad, complete the hands-on drills. Earn your place. Build momentum. Stay sharp.
<br>At the end? You’ll walk away with real AWS skills — and the fire to go further.
<br>The world runs in the cloud now. Do your duty.

#AWS Account
https://730492949633.signin.aws.amazon.com/console

# AWS Region - LOCKED 
eu-west-1 Ireland

```text
AWS Account
├── IAM Users & Roles
│
├── VPC (default or custom)
│   ├── EC2 Instance 1 (General use)
│   ├── EC2 Instance 2 (For RDS access)
│   ├── RDS (MySQL/Postgres)
│   ├── S3 Bucket
│   └── Glue Crawler → Athena Table
```

| **Team**       | **Users**      | **Users**     | **Users** | **Users** |
|----------------|----------------|---------------|-----------|-----------|
| Admins         | soyred_admin   |               |           |           |
| Team 1         | Team1-User1    | Team1-User2   |  etc      |    etc    |
| Team 2         | Team2-User1    | Team2-User2   |  etc      |    etc    |

### CIDR Blocks, Availability Zones, Subnets and Route Tables

| **Team**       | **CIDR Block**  | **Availability Zone** | **Public Subnet**     | **Private Subnet**    | **Public Route Table**                | **Private Route Table**                            |
|----------------|-----------------|-----------------------|-----------------------|-----------------------|---------------------------------------|----------------------------------------------------|
| Demo Account   | 10.0.0.0/16     | eu-west-1c            | 10.0.0.0/24           | 10.0.16.0/20          | 0.0.0.0/0 via Internet Gateway        | 0.0.0.0/0 via NAT Instance/Gateway or VPC Endpoint |
| VPC 1         | 10.2.0.0/16     | eu-west-1a            | 10.2.0.0/24           | 10.2.16.0/20          | 0.0.0.0/0 via Internet Gateway        | 0.0.0.0/0 via NAT Instance/Gateway or VPC Endpoint |
| VPC 2         | 10.3.0.0/16     | eu-west-1b            | 10.3.0.0/24           | 10.3.16.0/20          | 0.0.0.0/0 via Internet Gateway        | 0.0.0.0/0 via NAT Instance/Gateway or VPC Endpoint |
| VPC 3         | 10.4.0.0/16     | eu-west-1c            | 10.4.0.0/24           | 10.4.16.0/20          | 0.0.0.0/0 via Internet Gateway        | 0.0.0.0/0 via NAT Instance/Gateway or VPC Endpoint |
| VPC 4     | 10.5.0.0/16     |                       | 10.5.0.0/24           | 10.5.16.0/20          | 0.0.0.0/0 via Internet Gateway        | 0.0.0.0/0 via NAT Instance/Gateway or VPC Endpoint |


### Security Group Rules

| **Team**       | **Security Group**    | **Inbound Rules**                                                                                                                      | **Outbound Rules**            |
|----------------|-----------------------|----------------------------------------------------------------------------------------------------------------------------------------|-------------------------------|
| All Accounts   | Public SG             | Allow SSH (22) from your IP (Restricts EC2 Connect) & Allow HTTP 80 from  (0.0.0.0/0)                                                  | Allow all traffic (0.0.0.0/0) |
|                | Private SG            | Allow SSH (22) from Public SG/Private IP of Public aka Bastion Host                                                                    | Allow all traffic (0.0.0.0/0) |
|                | Nat SG                | Allow HTTP (80) from VPC CIDR<br>Allow HTTPS (443) from VPC CIDR<br>Allow SSH (22) from VPC CIDR<br>Allow ALL ICMP-IPv4 from 0.0.0.0/0 | Allow all traffic (0.0.0.0/0) |

By the end of Operation Cloudstrike, each recruit will have:<br>
<br>-🎨 Customized their AWS Console (theme + bookmarks)
<br>-🛡️ Created IAM user, role, and custom policy
<br>-🔍 Used IAM Policy Simulator to test access
<br>-💻 Launched an EC2 instance and connected via SSH
<br>-🔧 Installed a web server (Apache or Nginx)
<br>-💾 Attached and mounted an EBS volume
<br>-📦 Created and managed an S3 bucket
<br>-🔐 Applied S3 permissions and tested object access
<br>-⚖️ Set up an Application Load Balancer + listener rules
<br>-📈 Configured Auto Scaling Group and tested scale events
<br>-🗄️ Deployed an RDS instance and connected via CLI
<br>-📊 Ran SQL queries (CREATE, INSERT, SELECT, UPDATE, DELETE)
<br>-🕵️ Queried structured data from S3 using Athena

