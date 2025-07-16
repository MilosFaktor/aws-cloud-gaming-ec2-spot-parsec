### Do you want to see all screenshots from project? 
👉 [All screenshots](Screenshots/)

### Want the full build journey with errors, fixes, lessons, and AWS tweaks?  
👉 [BUILD-JOURNAL.md](BUILD-JOURNAL.md)

# AWS Cloud Gaming with EC2, Spot Instances & Parsec – Automatic Shutdown

## 📖 Description
This project demonstrates how to deploy a **GPU-powered Windows Server EC2 instance** for cloud gaming using **Parsec**, with a cost-optimized setup leveraging **Spot Instances**, automated idle detection, and shutdown workflows using **CloudWatch** and **Lambda**.

It includes:
- A custom **VPC** setup for gaming workloads.
- **GPU driver installation & Parsec configuration** on Windows Server 2019.
- Automation to scale Spot Fleet capacity to zero when idle, saving **~68% in costs**.

---

## ☁️ Services Used
- **Amazon EC2** (GPU instances: g4dn.xlarge)
- **Amazon VPC** (custom networking and subnets)
- **Elastic IP**
- **IAM Roles and Policies**
- **Amazon S3** (VPC endpoint for snapshots)
- **Amazon CloudWatch** (custom metrics and alarms)
- **AWS Lambda** (Spot Fleet automation)
- **Amazon SNS** (notifications)
- **Amazon Machine Images (AMI)**
- **AWS CLI**

---

## 🚀 Features
- Cloud gaming with Windows Server 2019 + Parsec.
- Spot Instance setup for cost optimization.
- Custom automation for idle detection and auto-shutdown.
- Troubleshooting GPU driver and resolution issues.

---

## 📝 Setup Summary

### 1️⃣ VPC Setup
- Created a dedicated **Gaming VPC** (`10.50.0.0/20`) in `eu-north-1` region with 3 public subnets.
- Added an **S3 VPC Endpoint** for direct snapshot access.

### 2️⃣ Security Groups
- Configured inbound rules for RDP (TCP 3389) and Parsec (UDP 8000–8200).

### 3️⃣ EC2 Instance Configuration
- Launched **g4dn.xlarge** GPU instance with **Windows Server 2019 Base**.
- Requested vCPU limit increase for GPU instances.
- Attached Elastic IP for stable external connectivity.

### 4️⃣ Parsec Setup & Troubleshooting
- Used Parsec Cloud Preparation Tool from GitHub.
- Resolved GPU driver and resolution issues (Nvidia driver version **538.67**).
- Fixed Parsec login and encoding issues.

### 5️⃣ Spot Instances & Cost Optimization
- Created AMI image of configured EC2 instance.
- Deployed Spot Fleet across 3 AZs with 9 instance pools.
- Automated assignment of correct Security Groups using Launch Templates.

### 6️⃣ Automation: CloudWatch & Lambda
- Created custom CloudWatch metric `IdleState`.
- Set alarms for 15 min (warning) & 30 min (auto-shutdown).
- Lambda reduces Spot Fleet capacity to 0 on idle.

### ✅ Result
- Cloud gaming via Parsec with **automatic shutdown when idle**.
- Achieved **~68% cost reduction** using Spot Instances.

---

## 🖼️ Screenshots

| Custom VPC Setup | CloudWatch Automation |
|-------------------|------------------------|
| ![VPC Screenshot](Screenshots/1- VPC set up.png) | ![CloudWatch](Screenshots/cloudwatch-alarms.png) |

<img src="Screenshots/1- VPC set up.png" width="750">

<img src="Screenshots/Alarm idle 30 triggered and  lambda sent 0 to metrics to bring alarms to OK state.png" width="750">

---

## 📚 Build Journal
For a detailed step-by-step walkthrough and troubleshooting, see [BUILD-JOURNAL.md](BUILD-JOURNAL.md).

---

## 🔗 Links
- [Parsec Cloud Preparation Tool](https://github.com/parsec-cloud/Parsec-Cloud-Preparation-Tool)

---

## 💡 Challenges Solved
- GPU driver compatibility on Windows Server.
- CloudWatch alarm configuration for accurate idle detection.
- Parsec screen resolution and encoding fixes.
- Spot Fleet Security Group assignment using Launch Templates.

---

## 🏷️ Tags
`AWS` `EC2` `Cloud Gaming` `Spot Instances` `Parsec` `CloudWatch` `Lambda`


<img src="Screenshots/1- VPC set up.png" width="750">


``` bash

``` 

## 🧑‍💻 Author
👋 Milos Faktor 💼 [LinkedIn](https://www.linkedin.com/in/milos-faktor-78b429255/)