# 👵️ Getting Started with AWS CloudTrail

**Track Everything. Know Who Did What, When, and Where.**

CloudTrail is a service that records actions made by users, roles, or services in AWS. It’s like a security camera for your cloud environment.

---

## 🚀 Why Use CloudTrail?

* See **who created**, modified, or deleted resources (like EC2, S3, IAM, etc.)
* Monitor suspicious activity
* Troubleshoot issues
* Meet compliance and audit requirements

---

## 🛠️ How to Enable CloudTrail (Step-by-Step)

### 1. **Sign into AWS Console**

Log into the AWS Management Console using your AWS credentials.

---

### 2. **Open CloudTrail Service**

* In the **Services** menu, search for and open **CloudTrail**

---

### 3. **Create a Trail**

* Click **"Create trail"**
* Give your trail a name (e.g., `account-activity-trail`)
* **Choose "Create a new S3 bucket"** or select an existing one for log storage
  *(This is where all CloudTrail logs will be saved)*

---

### 4. **Trail Settings**

* Choose to apply the trail to **All Regions** (Recommended)
* For simplicity, keep the rest of the settings at default unless instructed otherwise
* Click **Next**, review the settings, and then **Create trail**

---

## 🔍 View Logs

* Go to the **S3 bucket** linked to your trail
* You’ll see folders organized by date
* Download a log file and open it (they are in JSON format)
* You can also search and view logs in **CloudTrail > Event history** directly

---

## 🧠 Tips

* Enable CloudTrail as early as possible—it **doesn’t record retroactively**
* Combine with **AWS Config** for full visibility of resource changes
* Use **filters** in Event History to quickly find actions like `RunInstances`, `CreateUser`, etc.

---

## 💡 Sample Use Cases

| Use Case                      | What to Look For                            |
| ----------------------------- | ------------------------------------------- |
| Who deleted an EC2 instance?  | `TerminateInstances` event                  |
| When was an IAM user created? | `CreateUser` event                          |
| Who changed a security group? | `AuthorizeSecurityGroupIngress` / `Revoke*` |
| Suspicious login attempt?     | `ConsoleLogin` events with `Failure` result |

---

## 🚩 Don’t Forget

* CloudTrail is not retroactive — it only logs events **after** it’s turned on.
* **Keep your logs secure** — use S3 bucket policies and encryption.
* Consider integrating with **CloudWatch Logs** for real-time monitoring and alerts.

---

## ✅ Next Steps

* Test it: Create or delete a resource, then view it in **Event History**
* Try filtering logs by event type, user, or resource
* Share this setup with your team to encourage best practices!
