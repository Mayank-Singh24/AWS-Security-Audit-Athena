# AWS Security Auditor & Threat Hunting Pipeline

![AWS CloudTrail](https://img.shields.io/badge/AWS_CloudTrail-%23FF9900.svg?style=for-the-badge&logo=amazon-aws&logoColor=white)
![Amazon S3](https://img.shields.io/badge/Amazon_S3-%23569A31.svg?style=for-the-badge&logo=amazon-s3&logoColor=white)
![Amazon Athena](https://img.shields.io/badge/Amazon_Athena-%23FF4F8B.svg?style=for-the-badge&logo=amazon-aws&logoColor=white)
![SQL](https://img.shields.io/badge/SQL-%23003B57.svg?style=for-the-badge&logo=mysql&logoColor=white)

A serverless security auditing pipeline built on AWS to automatically capture, store, and query account-wide API activity for threat detection and incident response.

## 🏗️ Architecture Overview

This project simulates a real-world Security Operations Center (SOC) environment by tracking every single action taken within an AWS account and transforming raw JSON logs into a structured, queryable relational database.

<img width="2816" height="1536" alt="Gemini_Generated_Image_urh29rurh29rurh2" src="https://github.com/user-attachments/assets/1950f6b2-7ba7-447a-b7c7-fd77611f0dc2" />

1. **AWS CloudTrail (The Security Camera):** Continuously monitors and logs all Management Events (API calls) across the AWS account.
2. **Amazon S3 (The Vault):** Stores raw, Gzip-compressed `.json.gz` CloudTrail logs in a highly durable, partitioned folder structure.
3. **Amazon Athena (The Analytical Engine):** Projects a relational schema over the raw S3 data lake (`Schema-on-Read`), allowing security analysts to hunt for threats using standard SQL queries without dedicated database servers.


## ✨ Key Features

* **Continuous Security Logging:** Captures account-wide management events and administrative API operations in real time.
* **Serverless Log Analytics:** Eliminates server overhead by using Amazon Athena to execute on-demand SQL queries directly against compressed S3 objects.
* **Cost-Optimized Storage:** Stores logs in Gzip-compressed format (`.json.gz`) organized by region and date partitions to minimize storage footprint and query costs.
* **Proactive Threat Hunting:** Enables targeted identification of unauthorized access attempts, privilege escalation attempts, and failed console sign-ins.

---

## 🛠️ Technologies Used

* **Cloud Provider:** Amazon Web Services (AWS)
* **Core Services:** AWS CloudTrail, Amazon S3, Amazon Athena, AWS CloudShell
* **Query Language:** Standard ANSI SQL
* **Data Formats:** Compressed JSON (`.json.gz`), DDL Schema Definition

---

## 📋 Prerequisites

* An active **AWS Account** with root or IAM administrative permissions.
* Basic familiarity with SQL queries (`SELECT`, `WHERE`, `ORDER BY`, `LIKE`).
* Access to **AWS CloudShell** or local AWS CLI configured with credentials.

---

## 🚀 Step-by-Step Deployment Guide

### Step 1: Create the Secure Storage Vault (S3)
1. Navigate to the **Amazon S3** console and click **Create bucket**.
2. Name the bucket (e.g., `security-audit-logs-xxxx`).
3. Keep default settings ("Block *all* public access" enabled).
4. Click **Create bucket**.

### Step 2: Configure AWS CloudTrail
1. Open the **AWS CloudTrail** console and go to **Trails** > **Create trail**.
2. Set the trail name to `account-audit-trail`.
3. Select **Use existing S3 bucket** and choose your newly created bucket.
4. Uncheck **Log file SSE-KMS encryption** to prevent KMS key charges.
5. Under **Log events**, keep **Management events** checked and click **Create trail**.

### Step 3: Map Schema & Setup Amazon Athena
1. In CloudTrail, select **Event history** > click **Create Athena table**.
2. Select your log bucket and execute the auto-generated DDL statement.
3. Open **Amazon Athena** > **Query editor** > **Settings**.
4. Set the query result location to `s3://YOUR-BUCKET-NAME/athena-results/`.

---

## ⚔️ Red Team vs. Blue Team Simulation

### The Attack (Red Team)
To test the pipeline, an unauthorized role assumption was attempted via AWS CLI:

# bash
aws sts assume-role --role-arn arn:aws:iam::123456789012:role/FakeAdminRole --role-session-name HackSession

The Threat Hunt (Blue Team)
Using Amazon Athena, a security query was executed against the raw log table to isolate failed operations:

SELECT 
    eventtime, 
    eventname, 
    sourceipaddress, 
    useridentity.username, 
    errorcode, 
    errormessage
FROM cloudtrail_logs
WHERE errorcode IN ('AccessDenied', 'Client.UnauthorizedOperation')
   OR (eventname = 'ConsoleLogin' AND errormessage LIKE '%Failed%')
ORDER BY eventtime DESC;

## 🔍 Threat-Hunting SQL Query Library

1. Detect Suspicious Failed Logins

SELECT eventtime, sourceipaddress, useridentity.username, errormessage
FROM cloudtrail_logs
WHERE eventname = 'ConsoleLogin' AND errormessage = 'Failed authentication'
ORDER BY eventtime DESC;

2. Track IAM User Creation & Security Changes

SELECT eventtime, eventname, sourceipaddress, useridentity.username, requestparameters
FROM cloudtrail_logs
WHERE eventname IN ('CreateUser', 'CreateAccessKey', 'AttachUserPolicy')
ORDER BY eventtime DESC;

## 🧹 Resource Teardown & Cost Optimization

To avoid ongoing charges after completing the audit project:

Delete S3 Bucket Contents: Empty the S3 bucket (including all /AWSLogs and /athena-results prefixes) and delete the bucket.

Delete CloudTrail: Select account-audit-trail in the CloudTrail console and click Delete.

Drop Athena Table: In the Athena Query Editor, run DROP TABLE cloudtrail_logs;.

## 🛣️ Future Enhancements

Automated Incident Response: Integrate AWS GuardDuty and EventBridge to trigger automated Lambda remediation when threats are detected.

AI-Powered Natural Language Querying: Connect Amazon Bedrock / LLM agents to convert natural language prompts into Athena SQL queries for automated threat reporting.



