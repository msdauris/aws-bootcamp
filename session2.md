# Session 2: Compute & Storage (EC2 & S3)

## 📋 Table of Contents

- [9. EC2 Instance Management](#9-ec2-instance-management)
- [10. EC2 Troubleshooting](#10-ec2-troubleshooting)
- [11. EBS Volume Management](#11-ebs-volume-management)
- [12. EBS Troubleshooting Scenarios](#12-ebs-troubleshooting-scenarios)
- [13. S3 Bucket Operations](#13-s3-bucket-operations)
- [14. S3 Advanced Features](#14-s3-advanced-features)
- [15. Cleanup](#15-cleanup)

---

## 9. EC2 Instance Management

### Launch and Configure an EC2 Instance

**Objective:** Create a web server running Apache with a "Hello World" page

#### Steps

1. **Launch Instance**
   - Navigate to EC2 → Instances → "Launch Instance"
   - Provide an instance name

2. **Configuration**
   - **AMI:** Select Amazon Linux AMI
   - **Instance Type:** Choose t2.micro (general purpose)
   - **Key Pair:** Create new keypair with default settings
     - Save to `~/.ssh/name-of-your-key.pem`
     - Store securely (cannot be re-downloaded)

3. **Network Settings**
   - Select exisiting security group - aws-bootcamp-sg-web001??? (CLARIFY)
   - Allow SSH & HTTP traffic from anywhere (0.0.0.0/0) (CLARIFY)

4. **Storage**
   - Configure with default size

5. **Advanced Details - User Data**

### 🧠 EC2 User Data – Exam Tip

- Runs **once** on **first boot**
- Executes **as root** (no `sudo` needed)
- Used to auto-install packages, run scripts, etc.

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

6. **Launch**
   - Click "Launch Instance"

#### Verification

- Copy the public IP address
- Open in web browser to verify Apache is serving the "Hello World" page
- If it doesn’t load, try **`http://` instead of `https://`**

---

### SSH into EC2 Instance

**Objective:** Connect to EC2 instance using SSH (CLARIFY - use bastion host?)

#### Steps

1. **Set Key Permissions**
   ```bash
   chmod 400 ~/.ssh/name-of-your-key.pem
   ```

2. **Connect via SSH**

   ```bash
   ssh -i ~/.ssh/name-of-your-key.pem ec2-user@XX.XX.XX.XXX
   ```
   *(Replace XX.XX.XX.XXX with your instance's public IP)*

---

## 10. EC2 Troubleshooting

### Troubleshooting Exercise 1

**Problem:** Web server not accessible via browser

#### Troubleshooting Checklist

1. **Security Group Rules**
   - Verify HTTP (TCP port 80) and SSH (TCP port 22) are open to `0.0.0.0/0`

2. **Instance Status**
   - Confirm instance is in "running" state

3. **User Data Script Verification**
   - SSH into instance and check:

     ```bash
     # Check if Apache is installed
     rpm -q httpd
     
     # Check if Apache is running
     systemctl status httpd
     
     # Start Apache if needed
     sudo systemctl start httpd
     
     # Verify index.html exists
     ls -la /var/www/html/
     ```

4. **Instance Logs**
   - Access via EC2 Dashboard → Instance → Actions → Instance Settings → Get System Log

#### Test

After completing checks, try accessing `http://your-public-ip` again.

---

## 11. EBS Volume Management

### Attach EBS Volume to EC2

**Objective:** Practice EBS volume attachment and configuration

#### Steps

1. **Create Volume**
   - Navigate to EC2 → Volumes → "Create Volume"
   - Configure size, type, and **availability zone** (must match EC2 instance)

2. **Attach Volume**
   - Select volume → Actions → "Attach Volume"
   - Choose target instance and device name
   - Click "Attach"

3. **Verify Attachment**
   - SSH into instance
   - Run `lsblk` to see new volume in specified device path

#### Next Steps

- **TODO:** Format and mount the disk

---

### Snapshots and Recycle Bin

**Objective:** Practice EBS backup and recovery features

#### Tasks

- **Snapshot Creation:** Create EBS snapshots for backup
- **Cross-Region Replication:** Copy snapshots to different regions  
- **Volume Restoration:** Recreate volumes from snapshots
- **Recycle Bin:** Configure retention rules for deleted resources

---

## 12. EBS Troubleshooting Scenarios

### Scenario 1: Slow Disk Performance

**Problem:** Users experiencing slow disk performance on EC2 with attached EBS volume

#### Troubleshooting Steps

1. **Instance Type Review**
   - Verify instance type meets IOPS/throughput requirements

2. **EBS Volume Type**
   - Ensure appropriate volume type for workload
   - Consider upgrading to Provisioned IOPS SSD (io1/io2)

3. **Resource Utilization**
   - Monitor CPU, memory, network usage via CloudWatch

4. **I/O Load Analysis**
   - Check if I/O exceeds volume performance limits
   - Consider workload distribution or volume upgrades

5. **Instance-Level Issues**
   - Review application configuration
   - Check for network stack bottlenecks

#### Hands-on Simulation

- Analyze CloudWatch charts showing high queue depth/throttling on gp2 volumes
- Compare EC2 instance types for IOPS-heavy workloads
- Practice switching from gp2 to io1/io2 volumes

---

### Scenario 2: Volume Detachment Failure

**Problem:** Unable to detach EBS volume from EC2 instance

#### Troubleshooting Steps:
1. **Instance State**
   - Ensure instance is stopped before detachment

2. **Active Processes**
   - Check for applications/services using the volume
   - Verify no open files accessing the volume

3. **Permissions**
   - Confirm IAM user/role has `ec2:DetachVolume` permission

4. **Volume State**
   - Verify volume is healthy and not in error state

5. **Retry Process**
   - Wait and retry detachment if temporary issues suspected

#### Hands-on Simulation:
- Attempt detachment while volume is in use (demonstrates failure)
- Review IAM permissions requirements
- Analyze CloudTrail logs for "unauthorizedOperation" events

---

### Scenario 3: Data Corruption

**Problem:** Data corruption observed on EBS volume with file system errors

#### Troubleshooting Steps:
1. **Underlying Issues**
   - Check for hardware failures or network connectivity problems

2. **File System Integrity**
   - Run consistency checks:
     - **Linux:** `fsck`
     - **Windows:** `CHKDSK`

3. **Snapshot Analysis**
   - Compare corrupted data with previous snapshots
   - Identify changes causing corruption

4. **Application Behavior**
   - Investigate application-specific corruption
   - Check for misconfigured or faulty applications

5. **Data Restoration**
   - Restore from clean backups/snapshots
   - Verify backup integrity before restoration

#### Hands-on Simulation:
- Mount broken volume and run `fsck` diagnostics
- Practice snapshot restoration procedures
- Analyze `fsck` output for error identification

---

## 13. S3 Bucket Operations

### Create S3 Bucket and Upload Files

#### 🪣 Step 1: Create S3 Bucket
**Objective:** Create new S3 bucket for file storage

**Steps:**
1. Navigate to AWS Console → S3 service
2. Click "Create bucket"
3. Enter globally unique bucket name
4. Select region (default or closest to you)
5. Leave default settings (ACLs, encryption)
6. Click "Create bucket"

#### 📁 Step 2: Upload Files
**Objective:** Upload files to your S3 bucket

**Steps:**
1. Click on bucket name
2. Click "Upload" → "Add files" e.g. https://github.com/msdauris/aws-bootcamp/blob/main/s3/aws-bootcamp-example.png
3. Select files from local machine
4. Click "Upload"

---

### Create Bucket Policy

#### 🔐 Step 3: Make Files Public (Optional)
**Objective:** Configure public access to uploaded files

**Steps:**
1. **Method 1:** Select file → Actions → "Make public"
2. **Method 2:** Bucket Policy
   - Navigate to Permissions → Bucket Policy
   - Add JSON policy: 

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PublicReadGetObject",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::example-bucket-name/*"
        }
    ]
}
```

e.g.
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "Statement1",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::student-2-s3/*"
        }
    ]
}
```
**Result:** Files accessible via: `https://your-bucket-name.s3.amazonaws.com/your-file-name`

---

## 14. S3 Advanced Features

### S3 Pre-signed URLs

**Objective:** Share temporary access to private files

#### Using AWS CLI:
```bash
aws s3 presign s3://your-bucket-name/your-file-name
```

**Features:**
- Valid for 1 hour by default
- Temporary access without making files permanently public
- Ideal for support staff or external user access

---

### S3 Versioning

**Objective:** Maintain multiple versions of objects for recovery

#### Steps:
1. Go to S3 → Your bucket → Properties tab
2. Find "Bucket Versioning" section
3. Click "Enable" → Save
4. **Test:** Upload file, modify it, upload again
5. Check "Versions" tab to view all versions

**⚠️ Important:**
| Action                 | What Happens                                              |
| ---------------------- | --------------------------------------------------------- |
| Upload same file again | New version is created, old one kept                      |
| Show versions          | You can view all versions (v1, v2...) and delete markers  |
| Delete file (normal)   | Adds a **delete marker**, doesn't erase versions          |
| Don’t show versions    | File looks deleted (hidden by delete marker)              |
| Permanent delete       | Must manually delete all versions including delete marker |


---

### Static Website Hosting

#### 🌍 Step 6: Enable Static Website Hosting
**Objective:** Convert S3 bucket into public website

#### Steps:
1. Navigate to Properties → Static website hosting
2. Click "Edit" → Enable hosting
3. Set index document to `index.html`
4. Save changes and upload HTML files

**Result:** Public URL: `http://your-bucket-name.s3-website-region.amazonaws.com`

**⚠️ Important:** Bucket must be public for website hosting to work & make sure to copy when prompted!

---

### S3 Replication

**Objective:** Automatically replicate files between buckets

#### Prerequisites:
- Both source and destination buckets must have versioning enabled

#### Steps:
1. Create second S3 bucket (destination)
2. In source bucket: Management → Replication rules
3. Click "Create replication rule"
4. Configure:
   - **Source:** This bucket
   - **Destination:** New bucket
   - **Permissions:** IAM role (auto-created)
5. **Test:** Upload file to source, verify appearance in destination

---

### S3 Event Notifications

**Objective:** Trigger actions when files are added/modified

#### Steps:
1. Navigate to Properties → Event notifications
2. Click "Create event notification"
3. Configure:
   - **Name:** Your event name
   - **Event type:** (e.g., "All object create events")
   - **Destination:** Lambda function, SNS topic, or SQS queue
4. Save configuration
5. **Test:** Upload file to trigger event

---

---

### S3 Storage Classes

**Objective:** Optimize cost based on access patterns

While your slide deck includes a detailed table, here’s a quick summary:

- **Standard:** For frequently accessed data  
- **Intelligent-Tiering:** Automatically moves data to lower-cost tiers  
- **Standard-IA / One Zone-IA:** For infrequent access, lower cost  
- **Glacier / Glacier Deep Archive:** Long-term archival  
- **Reduced Redundancy (Legacy):** Not recommended — largely deprecated

> 💡 Use lifecycle policies to transition objects between storage classes over time.

---

### AWS Snowball

**Objective:** Physically transfer large volumes of data to AWS

- Used when data transfer over the internet is too slow or impractical
- Comes in two options:
  - **Snowball Edge Storage Optimized** – for large-scale data transfer
  - **Snowball Edge Compute Optimized** – includes local processing power
- Tamper-resistant, shipping-tracked, and encrypted end-to-end

> ✅ **Exam Tip:** Think **“petabyte-scale data transfer”** and **“offline migration to AWS”**


## 15. Cleanup

### Checkpoint

**Objective:** Clean up resources to avoid charges

#### Tasks:
- 🧹 Delete S3 buckets and files
- Disable replication rules
- Turn off versioning if no longer needed
- Terminate EC2 instances
- Delete EBS volumes and snapshots
