# Session 4: Databases & Analytics

## 📋 Table of Contents

- [22. RDS (Relational Database Service)](#22-rds-relational-database-service)
- [23. Connect to RDS from EC2](#23-connect-to-rds-from-ec2)
- [24. Athena + S3 Querying](#24-athena--s3-querying)
- [25. Session Checkpoint & Cleanup](#25-session-checkpoint--cleanup)

---

## 22. RDS (Relational Database Service) -  (Optional Facilitators Only — Educational Text) 
If not creating own database Students skip to and continue to use the demo DB [23. Connect to RDS from EC2](#23-connect-to-rds-from-ec2)

### Set Up Production-Ready RDS Instance


**Objective:** Create a production-aware RDS instance with high availability and backups for enterprise use

**Duration:** 45 minutes

#### Why RDS for Enterprise?

RDS provides managed database services that enable:
- **Strategic Decision Making:** Optimal data security and scalability
- **High Availability:** Multi-AZ deployments for fault tolerance
- **Automated Management:** Backups, patching, and monitoring
- **Cost Optimization:** Right-sizing and performance tuning

#### Step-by-Step Instructions

### Step 1: Navigate to RDS Console

1. **Access RDS Service**
   - Log into AWS Management Console
   - Go to **Services** → Search for **"RDS"** under Databases
   - Click **Create database**

### Step 2: Choose Database Engine

1. **Select Engine Type**
   - Choose **MySQL** (or **PostgreSQL**) (recommended for this exercise)
   - Skip "Easy create" option for full configuration control

2. **Edition Selection**
   - **Free Tier:** For learning and development
   - **Dev/Test:** For non-production environments
   - **Production:** For enterprise workloads

### Step 3: Configure Database Settings

1. **Creation Method**
   - Choose **Standard Create** for full control

2. **Database Identification**
   - **DB instance identifier:** `aws-bootcamp-[name]-myrds-db`
   - **Master username:** `admin`
   - **Master password:** Create secure password (8+ characters)
   - ✅ Store password securely

### Step 4: Instance Configuration

1. **Instance Class**
   - **Testing:** `db.t3.micro` (if free tier eligible)
   - **Production:** `db.m5.large` or higher based on requirements

2. **Storage Configuration**
   - **Storage type:** General Purpose SSD (gp2)
   - **Allocated storage:** 20 GB minimum
   - **Storage autoscaling:** Enable for production
   - **Maximum storage threshold:** 100 GB

3. **High Availability** (Production Consideration)
   - **Multi-AZ deployment:** Enable for production (skip for workshop to save time)
   - Provides automatic failover to standby instance

### Step 5: Configure Networking

1. **Network Settings**
   - **VPC:** Choose aws-bootcamp-vpc
   - **Subnet group:** Use aws-bootcamp-db-subnet-group
   - **Public access:** Disable for workshop (disable for production)

2. **Security Configuration**
   - **VPC security group:** Choose aws-bootcamp-sg-db001
   - **Database port:** 
     - MySQL: 3306
     - (PostgreSQL: 5432)

### Step 6: Enable Backups and Monitoring

1. **Backup Configuration**
   - **Backup retention period:** 7 days (1-35 days available)
   - **Backup window:** Specify preferred time
   - **Copy tags to snapshots:** Enable

2. **Monitoring Settings**
   - **Enhanced monitoring:** Enable (optional for workshop)
   - **Monitoring role:** Auto-create or choose existing
   - **Performance insights:** Enable for production workloads

3. **Maintenance Settings**
   - **Auto minor version upgrade:** Enable
   - **Maintenance window:** Configure for low-traffic periods

### Step 7: Additional Settings

1. **Database Options**
   - **Initial database name:** Optional (can create later)
   - **Parameter group:** Use default or custom
   - **Option group:** Use default

2. **Security Features**
   - **Encryption:** Enable (uses AWS KMS by default)
   - **Authentication:** Password authentication (default)
   - **IAM database authentication:** Enable for enhanced security

3. **Deletion Protection**
   - **Enable deletion protection:** Recommended for production

### Step 8: Launch Database

1. **Final Review**
   - Review all configurations
   - Estimate monthly costs
   - Click **Create database**

2. **Provisioning Time**
   - Database creation takes ~10 minutes
   - Use this time to discuss:
     - RDS use cases and benefits
     - High availability strategies
     - Cost optimization techniques

#### 💡 Discussion Topics (During Provisioning)

**High Availability Features:**
- Multi-AZ deployments for automatic failover
- Read replicas for read scaling and disaster recovery
- Cross-region backups for compliance

**Security Best Practices:**
- IAM database authentication
- VPC security groups and NACLs
- Encryption at rest and in transit
- Regular security patches through maintenance windows

**Cost Optimization:**
- Reserved Instances for predictable workloads
- Right-sizing based on CloudWatch metrics
- Storage autoscaling to avoid over-provisioning

---

## 23. Connect to RDS from EC2
Students follow from here

### Secure Database Access and SQL Operations

**Objective:** Launch EC2 instance to securely connect to RDS and practice SQL operations

### Step 1: Launch EC2 Instance for Database Access

1. **Navigate to EC2**
   - Go to EC2 Dashboard → Click **Launch instance**

2. **Instance Configuration**
   - **AMI:** Amazon Linux 2 (for MySQL) or Ubuntu (for PostgreSQL)
   - **Instance type:** t2.micro or t3.micro
   - **Key pair:** Use existing or create new

3. **Network Configuration**
   - **VPC:** Same VPC as RDS instance
   - **Subnet:** Same or different subnet (ensure connectivity)
   - **Auto-assign public IP:** Enable
   - **Security group:** Allow SSH (port 22) and outbound internet access

4. **Security Group Rules**
   - **Inbound:** SSH (port 22) from your IP
   - **Outbound:** All traffic (for package installation)

5. **Launch Instance**
   - Review and launch
   - Connect via SSH once ready

### Step 2: Configure RDS Security Group

1. **Update RDS Security Group**
   - Navigate to RDS → Your database → Connectivity & security
   - Click on the security group link
   - Add inbound rule:
     - **Type:** MySQL/Aurora (3306) or PostgreSQL (5432)
     - **Source:** EC2 security group or EC2 private IP
     - **Description:** Access from EC2 instance

### Step 3: Install Database Client

**Connect to EC2 Instance:**
```bash
ssh -i your-key.pem ec2-user@your-ec2-public-ip
```

**Install MySQL Client (Amazon Linux):**
```bash
sudo yum update -y
sudo yum install -y mariadb105
```

**Install PostgreSQL Client (Ubuntu):**
```bash
sudo apt-get update
sudo apt-get install postgresql-client -y
```

### Step 4: Connect to RDS Instance

1. **Get RDS Endpoint**
   - RDS Console → Your database → Connectivity & security
   - Copy the endpoint URL

2. **Connect to Database**

**MySQL Connection:**
```bash
mysql -h your-rds-endpoint -u admin -p
```

**PostgreSQL Connection:**
```bash
psql -h your-rds-endpoint -U admin -d postgres
```

3. **Enter Password**
   - Enter the master password you created during RDS setup

### Step 5: Run Basic SQL Commands

#### Show Databases

**MySQL:**
```sql
SHOW DATABASES;
```

#### Create Database

**MySQL:**
```sql
CREATE DATABASE bootcamp;
USE bootcamp;

CREATE TABLE employees (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100),
    role VARCHAR(100),
    salary DECIMAL(10,2),
    hire_date DATE
);
```

#### Create Table

**MySQL:**
```sql
CREATE TABLE employees (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100),
    role VARCHAR(100),
    salary DECIMAL(10,2),
    hire_date DATE
);
```

**PostgreSQL:**
```sql
CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    role VARCHAR(100),
    salary DECIMAL(10,2),
    hire_date DATE
);
```

#### Insert Sample Data

```sql
INSERT INTO employees (name, role, salary, hire_date) VALUES 
('John Doe', 'Software Engineer', 80000.00, '2023-01-15'),
('Jane Smith', 'Data Analyst', 70000.00, '2023-02-01'),
('Bob Brown', 'Senior Engineer', 95000.00, '2022-11-10'),
('Lisa Ray', 'Product Manager', 85000.00, '2023-03-01'),
('Tina Chen', 'Marketing', 85000.00, '2023-03-01');
```

#### Query Data

```sql
-- Select all employees
SELECT * FROM employees;

-- Filter by role
SELECT name, salary FROM employees WHERE role LIKE '%Engineer%';

-- Order by salary
SELECT name, role, salary FROM employees ORDER BY salary DESC;

-- Average salary by role
SELECT role, AVG(salary) as avg_salary, COUNT(*) as count
FROM employees 
GROUP BY role;
```

#### Update Records

```sql
-- Give John a promotion
UPDATE employees 
SET role = 'Senior Software Engineer', salary = 90000.00 
WHERE name = 'John Doe';
```

#### Delete Records

```sql
-- Remove an employee
DELETE FROM employees WHERE name = 'Jane Smith';
```

### Step 6: Backups & Snapshots

1. **Manual Snapshots**
   - RDS Console → Snapshots → **Take snapshot**
   - Provide snapshot name and description
   - Wait for completion

2. **Snapshot Types**
   - **Automated backups:** Scheduled, auto-deleted based on retention period
   - **Manual snapshots:** User-controlled retention, persist until manually deleted

3. **Restore from Snapshot**
   - Select snapshot → **Restore**
   - Creates new RDS instance
   - Useful for point-in-time recovery

### Step 7: Monitoring

1. **CloudWatch Metrics**
   - RDS Console → Your database → **Monitoring** tab
   - Review key metrics:
     - CPU Utilization
     - Database Connections
     - Free Storage Space
     - Read/Write IOPS

2. **Set Up Alarms**
   - Click **Create alarm** for critical metrics
   - Configure SNS notifications
   - Set thresholds based on application requirements

### Step 8: Scaling Options

1. **Vertical Scaling**
   - RDS Console → **Modify** DB instance
   - Change instance class for more CPU/memory
   - Apply immediately or during maintenance window

2. **Storage Scaling**
   - Increase allocated storage
   - Enable storage autoscaling for automatic growth

3. **Horizontal Scaling (Read Replicas)**
   - Actions → **Create read replica**
   - Choose different region for disaster recovery
   - Use for read-heavy workloads

### Step 9: High Availability (Multi-AZ)

1. **Enable Multi-AZ**
   - **Modify** DB instance
   - Enable **Multi-AZ deployment**
   - AWS provisions synchronous standby in different AZ

2. **Benefits**
   - Automatic failover (1-2 minutes)
   - Protection against AZ-level failures
   - No application changes required

---

## 24. Athena + S3 Querying

### Serverless Analytics with Athena and Glue

**Objective:** Learn serverless querying of S3 data using Athena with schema discovery via AWS Glue

**Duration:** 30-45 minutes

#### Goals

- ✅ Query S3 data using Athena
- ✅ Understand schema discovery using AWS Glue
- ✅ Practice SQL on semi-structured data
- ✅ Experience serverless analytics workflow

### Step 1: Prepare Mock Data

1. **Create Sample CSV File**
   - Open text editor or spreadsheet application
   - Create file named `employees.csv` with following content:

```csv
id,name,department,salary,hire_date,location
1,John Doe,Engineering,80000,2023-01-15,Seattle
2,Jane Smith,Marketing,70000,2023-02-01,New York
3,Bob Brown,Engineering,82000,2022-11-10,Seattle
4,Lisa Ray,Finance,90000,2023-03-01,Chicago
5,Tina Chen,Marketing,72000,2023-01-20,New York
6,Mark Wilson,Engineering,95000,2022-08-15,Seattle
7,Sarah Davis,Finance,88000,2023-02-15,Chicago
8,Tom Johnson,Marketing,75000,2022-12-01,New York
```

2. **Save File Locally**
   - Save as `employees.csv`
   - Ensure proper CSV formatting

### Step 2: Upload to S3

1. **Create S3 Bucket**
   - Navigate to S3 Console
   - Click **Create bucket**
   - Bucket name: `athena-workshop-data-[your-name]` (must be globally unique)
   - Choose appropriate region

2. **Create Folder Structure**
   - Inside bucket, create folder: `data/employees/`
   - This organization helps with partitioning and organization

3. **Upload CSV File**
   - Navigate to `data/employees/` folder
   - Upload `employees.csv`
   - Verify file is accessible

### Step 3: Set Up Schema Discovery (Glue Crawler)

1. **Navigate to AWS Glue Console**
   - Go to AWS Glue service
   - Click **Crawlers** in left navigation
   - Click **Create crawler**

2. **Crawler Configuration**
   - **Name:** `EmployeeDataCrawler`
   - **Description:** Crawler for employee CSV data

3. **Data Source Configuration**
   - **Data source type:** S3
   - **S3 path:** `s3://your-bucket-name/data/employees/`
   - **Subsequent crawler runs:** Crawl all folders

4. **IAM Role Setup**
   - **Create new role** or use existing
   - Role name: `AWSGlueServiceRole-EmployeeCrawler`
   - Ensure role has permissions to:
     - Access S3 bucket
     - Write to Glue Data Catalog

5. **Database Configuration**
   - **Target database:** Create new database
   - **Database name:** `employee_analytics_db`

6. **Crawler Schedule**
   - **Frequency:** On demand (for workshop)
   - **For production:** Set appropriate schedule

7. **Review and Create**
   - Review all settings
   - Click **Create crawler**

### Step 4: Run Crawler and Verify Schema

1. **Execute Crawler**
   - Select your crawler
   - Click **Run crawler**
   - Wait for completion (typically 1-2 minutes)

2. **Verify Table Creation**
   - Navigate to **Databases** in Glue Console
   - Click on `employee_analytics_db`
   - Verify `employees` table is created
   - Click on table to view schema

3. **Review Detected Schema**
   - Verify column names and data types
   - Note any data type inferences by Glue

### Step 5: Query with Athena

1. **Open Athena Console**
   - Navigate to Amazon Athena service
   - Click **Launch query editor**

2. **Configure Query Results Location**
   - Go to **Settings** tab
   - Set **Query result location:** `s3://your-bucket-name/query-results/`
   - Create folder if it doesn't exist

3. **Select Database**
   - In query editor, select `employee_analytics_db` from dropdown
   - Verify `employees` table appears in left panel

4. **Basic Query**
   ```sql
   SELECT * FROM employees LIMIT 10;
   ```

5. **Verify Results**
   - Click **Run query**
   - Review results in lower panel
   - Check query execution time and data scanned

### Step 6: Advanced SQL Queries

#### Filter by Department

```sql
SELECT name, salary, hire_date 
FROM employees 
WHERE department = 'Engineering'
ORDER BY salary DESC;
```

#### Average Salary by Department

```sql
SELECT 
    department, 
    AVG(salary) as avg_salary,
    COUNT(*) as employee_count,
    MIN(salary) as min_salary,
    MAX(salary) as max_salary
FROM employees 
GROUP BY department
ORDER BY avg_salary DESC;
```

#### Salary Analysis by Location

```sql
SELECT 
    location,
    department,
    AVG(salary) as avg_salary,
    COUNT(*) as count
FROM employees 
GROUP BY location, department
ORDER BY location, avg_salary DESC;
```

#### Top Earners

```sql
SELECT 
    name, 
    department, 
    salary,
    location
FROM employees 
ORDER BY salary DESC 
LIMIT 3;
```

#### Recent Hires

```sql
SELECT 
    name, 
    department, 
    hire_date,
    salary
FROM employees 
WHERE hire_date >= DATE('2023-01-01')
ORDER BY hire_date DESC;
```

#### Complex Analytical Query

```sql
SELECT 
    department,
    location,
    COUNT(*) as headcount,
    AVG(salary) as avg_salary,
    SUM(salary) as total_payroll,
    MIN(hire_date) as earliest_hire,
    MAX(hire_date) as latest_hire
FROM employees 
GROUP BY department, location
HAVING COUNT(*) > 1
ORDER BY total_payroll DESC;
```

### Step 7: Advanced Features (Optional)

#### Partitioning for Performance

1. **Create Partitioned Data Structure**
   - Organize data in S3 by partition keys
   - Example: `s3://bucket/data/employees/year=2023/month=01/`

2. **Create Partitioned Table**
   ```sql
   CREATE EXTERNAL TABLE employees_partitioned (
       id int,
       name string,
       department string,
       salary double,
       hire_date date,
       location string
   )
   PARTITIONED BY (year int, month int)
   STORED AS INPUTFORMAT 'org.apache.hadoop.mapred.TextInputFormat'
   OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
   LOCATION 's3://your-bucket/data/employees_partitioned/'
   TBLPROPERTIES ('skip.header.line.count'='1');
   ```

#### Data Formats and Compression

- **Parquet:** Columnar format for better performance
- **ORC:** Optimized row columnar format
- **Compression:** GZIP, Snappy, LZO for smaller file sizes

### Step 8: Cost Optimization

#### Monitor Query Costs

1. **Check Data Scanned**
   - Review "Data scanned" metric after each query
   - Athena charges based on data scanned

2. **Optimization Techniques**
   - Use column projection (SELECT specific columns)
   - Implement partitioning
   - Use compressed file formats
   - Optimize file sizes (128MB-1GB recommended)

#### Example Cost Comparison

```sql
-- Expensive query (scans all data)
SELECT * FROM employees;

-- Optimized query (scans less data)
SELECT name, salary FROM employees WHERE department = 'Engineering';
```

---

## 25. Session Checkpoint & Cleanup

### Resource Cleanup and Cost Management

**Objective:** Properly clean up resources to avoid unexpected charges and review session learnings

#### Cleanup Checklist

### AWS RDS Cleanup

1. **Delete RDS Instance**
   - RDS Console → Select database
   - **Actions** → **Delete**
   - ⚠️ **Disable** "Create final snapshot" for workshop
   - Type confirmation text
   - Wait for deletion to complete

2. **Remove DB Subnet Groups** (if custom created)
   - RDS Console → Subnet groups
   - Delete custom subnet groups

3. **Clean Up Snapshots**
   - RDS Console → Snapshots
   - Delete any manual snapshots created during workshop

### EC2 Instance Cleanup

1. **Terminate EC2 Instances**
   - EC2 Console → Instances
   - Select instances → **Instance state** → **Terminate**
   - Confirm termination

2. **Delete Key Pairs** (if created for workshop)
   - EC2 Console → Key pairs
   - Delete workshop-specific key pairs

3. **Remove Security Groups**
   - EC2 Console → Security groups
   - Delete custom security groups (ensure no dependencies)

### S3 and Analytics Cleanup

1. **Delete S3 Objects**
   - S3 Console → Your bucket
   - Select all objects → **Delete**
   - Empty bucket completely

2. **Delete S3 Bucket**
   - Select bucket → **Delete**
   - Type bucket name to confirm
   - Wait for deletion completion

### AWS Glue Cleanup

1. **Delete Glue Tables**
   - Glue Console → Databases → `employee_analytics_db`
   - Select tables → **Delete**
   - Confirm deletion

2. **Delete Glue Database**
   - Select database → **Delete**
   - Confirm deletion

3. **Delete Glue Crawlers**
   - Glue Console → Crawlers
   - Select `EmployeeDataCrawler` → **Delete**
   - Confirm deletion

### IAM Cleanup

1. **Remove Custom IAM Roles**
   - IAM Console → Roles
   - Delete workshop-specific roles:
     - `AWSGlueServiceRole-EmployeeCrawler`
     - Any EC2 instance roles created

2. **Delete IAM Policies** (if custom created)
   - IAM Console → Policies
   - Delete custom policies

### Additional Cleanup Items

1. **CloudWatch Alarms**
   - CloudWatch Console → Alarms
   - Delete any alarms created during workshop

2. **VPC Resources** (if custom created)
   - Only delete if created specifically for workshop
   - Ensure no other resources depend on them

#### Cost Verification

1. **AWS Cost Explorer**
   - Navigate to Cost Explorer
   - Review charges for workshop day
   - Verify all resources are terminated

2. **Billing Dashboard**
   - Check current month charges
   - Set up billing alerts if not already configured

#### Session Reflection

### Key Learnings Achieved

**RDS Mastery:**
- ✅ Created production-ready RDS instance
- ✅ Configured high availability and backup strategies
- ✅ Established secure connectivity from EC2
- ✅ Performed database operations and monitoring

**Analytics Capabilities:**
- ✅ Implemented serverless querying with Athena
- ✅ Automated schema discovery using Glue
- ✅ Executed complex SQL analytics on S3 data
- ✅ Understood cost optimization strategies

**Enterprise Readiness:**
- ✅ Applied security best practices
- ✅ Configured monitoring and alerting
- ✅ Implemented backup and recovery procedures
- ✅ Managed costs and resource cleanup

### Technical Skills Developed

**Database Administration:**
- RDS instance configuration and management
- MySQL/PostgreSQL client operations
- Backup and snapshot management
- Performance monitoring and scaling

**Data Analytics:**
- S3-based data lake architecture
- ETL processes with AWS Glue
- SQL analytics with Amazon Athena
- Cost-effective serverless analytics

**Security and Networking:**
- VPC security group configuration
- Database access control
- IAM role and policy management
- Network connectivity troubleshooting
