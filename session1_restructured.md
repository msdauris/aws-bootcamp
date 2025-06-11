# Session 1: Foundations & IAM

## 📋 Table of Contents

- [Prerequisites for Facilitators](#prerequisites-for-facilitators)
- [1. AWS Console Navigation](#1-aws-console-navigation)
- [2. IAM Groups & Users Analysis](#2-iam-groups--users-analysis)
- [3. IAM Policy Simulator](#3-iam-policy-simulator)
- [4. Policy Analysis - Answer Key](#4-policy-analysis---answer-key)
- [5. IAM Role Creation](#5-iam-role-creation)
- [6. Access Keys Configuration](#6-access-keys-configuration)
- [7. Multi-Factor Authentication (MFA)](#7-multi-factor-authentication-mfa)
- [8. Session Checkpoint](#8-session-checkpoint)

---

## Prerequisites for Facilitators

### Required Setup Items

**Before the Workshop:**
- SSH key pairs (or guide for generating)
- Example CSV file for Athena exercises
- Sample IAM policy templates
- User data scripts for EC2 auto-installation
- Pre-created VPC with private and public subnets (or use default)
- Quick reference sheet / Cheatsheet PDF

### ⚠️ CRITICAL: Enable AWS CloudTrail

**Objective:** Enable CloudTrail logging before workshop begins

#### Steps

1. **Navigate to CloudTrail Console**
   - Go to: https://console.aws.amazon.com/cloudtrail

2. **Create Trail**
   - Choose `Trails` from the sidebar
   - If none exists, click "Create trail"

3. **Trail Configuration**
   - **Trail Name:** `WorkshopTrail`
   - **Storage Location:** Choose or create S3 bucket (e.g., `aws-workshop-cloudtrail-logs`)

4. **Coverage Settings**
   - ✅ **Apply Trail To All Regions**
   - *Recommended: Ensures activity tracking even if students switch regions*

5. **Event Types**
   - **Management Events:** Enable Read and Write events
   - **Insight Events:** (Optional) Helps detect unusual activity

6. **Finalize**
   - Click "Create Trail"

📝 **Note:** CloudTrail starts recording immediately but events may take a few minutes to appear in logs.

---

## 1. AWS Console Navigation

### Getting Familiar with the AWS Console

**Objective:** Build confidence navigating the AWS Console interface

#### Tasks

1. **Theme Customization**
   - Change console theme (dark/light mode)
   - Adjust to personal preference

2. **Create Bookmarks**
   - Add bookmarks for frequently used services:
     - IAM
     - EC2
     - S3
     - CloudWatch

3. **Explore Console Features**
   - Practice service search functionality
   - Understand region selection
   - Navigate between different service consoles

#### 🎯 Outcome
Participants should feel comfortable navigating the console and accessing services efficiently.

---

## 2. IAM Groups & Users Analysis

### Examine Demo Groups and Users

**Objective:** Understand IAM group structure and user permissions

#### Groups to Analyze

**Developers Group**
- Users should have permissions to:
  - Read/write to AWS CodeCommit
  - Deploy code to Elastic Beanstalk or EC2

**QA Group**  
- Users should have permissions to:
  - Read-only access to CodeCommit
  - View logs in CloudWatch
  - Access X-Ray for troubleshooting

#### Tasks

1. **Navigate to IAM Console**
   - Go to IAM → Groups

2. **Examine Group Policies**
   - Click on each group
   - Review attached policies
   - Note the permissions granted

3. **Check User Assignments**
   - Verify users are properly assigned to groups
   - Review individual user permissions

#### 🎯 Expected Results
- **Developers:** Full deployment rights
- **QA:** Read-only + diagnostics access

---

## 3. IAM Policy Simulator

### Test User Permissions

**Objective:** Validate permissions using AWS IAM Policy Simulator

#### Access the Simulator
- Navigate to: IAM → Policy Simulator
- Or visit: https://policysim.aws.amazon.com/

#### Test Cases

**For Developers:**
- ✅ Verify they can push code to CodeCommit
- ✅ Test deployment permissions to Elastic Beanstalk/EC2
- ✅ Check S3 bucket access

**For QA Engineers:**
- ✅ Verify read-only access to CodeCommit
- ✅ Test CloudWatch logs access
- ✅ Check X-Ray troubleshooting permissions
- ❌ Confirm they cannot push code or deploy

#### How to Use Policy Simulator

1. **Select User/Role**
   - Choose the user or role to simulate

2. **Select Service**
   - Pick AWS service (e.g., CodeCommit, S3, CloudWatch)

3. **Select Actions**
   - Choose specific actions to test

4. **Run Simulation**
   - Click "Run Simulation"
   - Review results (Allow/Deny)

---

## 4. Policy Analysis - Answer Key

### Developers Group Policies

**Attached Policies:**

- **`AWSCodeCommitReadOnly`**
  - Allows read access to CodeCommit repository
  - ⚠️ *Does not allow pushing code*

- **`AWSCodeDeployFullAccess`**
  - Enables code deployment using AWS CodeDeploy

- **`AmazonS3FullAccess`**
  - Read and write access to S3 buckets

- **Custom Elastic Beanstalk/EC2 Policy**
  - Create, update, and delete application resources

#### 🔍 Note
To allow developers to push code, attach `AWSCodeCommitPowerUser` or include `codecommit:GitPush` in a custom policy.

---

### QA Engineers Group Policies

**Attached Policies:**

- **`AWSCodeCommitReadOnly`**
  - Read access to CodeCommit repository

- **`AmazonS3ReadOnlyAccess`**
  - Read access to project-related S3 buckets

- **`CloudWatchReadOnlyAccess`**
  - Access logs from Amazon CloudWatch

- **`AWSXRayReadOnlyAccess`**
  - Read access for troubleshooting and performance analysis

#### 🔍 Note
QA engineers have read-only access to logs and tracing tools, suitable for diagnosis without impacting system state.

---

## 5. IAM Role Creation

### Create EC2 Service Role

**Objective:** Create IAM role for EC2 instances to access S3

#### Steps

1. **Navigate to IAM Roles**
   - Go to IAM → Roles → "Create role"

2. **Select Trusted Entity**
   - Choose **AWS Service**
   - Select **EC2** as the service

3. **Attach Policies**
   - Search and select `AmazonS3ReadOnlyAccess`
   - This allows EC2 instances to read from S3

4. **Role Configuration**
   - **Role Name:** `EC2-S3-ReadOnly-Role`
   - **Description:** "Allows EC2 instances to read from S3 buckets"

5. **Create Role**
   - Review settings and click "Create role"

#### Usage
When launching or modifying EC2 instances, select this IAM Role to grant S3 access without embedding credentials.

---

## 6. Access Keys Configuration

### Create and Configure AWS CLI Access

**Objective:** Set up programmatic access for non-admin user

#### Create Access Keys

1. **Navigate to IAM Users**
   - Select a non-admin user from your groups

2. **Generate Access Keys**
   - Go to Security credentials tab
   - Click "Create access key"
   - Choose "Command Line Interface (CLI)"
   - Copy the Access Key ID and Secret Access Key

⚠️ **Important:** Save credentials securely - secret key cannot be retrieved later

#### Configure AWS CLI

1. **Install AWS CLI** (if not already installed)

2. **Configure Credentials**
   ```bash
   aws configure
   ```

3. **Enter Credentials**
   - Access Key ID: [Your Access Key]
   - Secret Access Key: [Your Secret Key]
   - Default region: [Your preferred region]
   - Default output format: json

#### Verify Configuration

**Test CLI Access:**
```bash
aws iam list-users
```

**Expected Result:** User should only see resources permitted by their IAM group permissions.

---

## 7. Multi-Factor Authentication (MFA)

### Secure Account with MFA

**Objective:** Add an extra layer of security to user accounts

#### Steps

1. **Navigate to User Security**
   - IAM → Users → Select user → Security credentials tab

2. **Enable MFA Device**
   - Click "Assign MFA device"
   - Choose MFA device type:
     - Virtual MFA device (recommended for workshop)
     - Hardware MFA device
     - SMS text message (not recommended for production)

3. **Configure Virtual MFA**
   - Use authenticator app (Google Authenticator, Authy, etc.)
   - Scan QR code with app
   - Enter two consecutive MFA codes
   - Click "Assign MFA"

#### Test MFA

- Sign out and sign back in
- Verify MFA prompt appears during login
- Enter code from authenticator app

---

## 8. Session Checkpoint

### Learning Objectives Verification

**After completing this session, participants should understand:**

#### Core IAM Concepts
- ✅ Purpose and differences between IAM users, groups, roles, and policies
- ✅ How to apply the **principle of least privilege**
- ✅ Relationship between users, groups, and policy inheritance

#### Practical Skills
- ✅ Navigate AWS Console efficiently
- ✅ Use **IAM Policy Simulator** to test and validate permissions
- ✅ Create and configure IAM roles for AWS services
- ✅ Set up programmatic access with access keys
- ✅ Enable and configure Multi-Factor Authentication

#### Security Best Practices
- ✅ Understand why least privilege principle is important
- ✅ Know when to use roles vs. users vs. groups
- ✅ Recognize the importance of MFA for account security
- ✅ Understand CloudTrail for audit and compliance

#### Next Steps
Participants are now ready to proceed to Session 2: Compute & Storage (EC2 & S3), where they'll apply IAM roles and policies to real AWS resources.