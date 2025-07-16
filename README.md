### Do you want to see all screenshots from this project? 
👉 [All screenshots](Screenshots/)

### Want the full build journey with errors, fixes, lessons, and AWS tweaks?  
👉 [BUILD-JOURNAL.md](BUILD-JOURNAL.md)

# AWS Cloud Gaming with EC2, Spot Instances & Parsec – Automatic Shutdown

## 📖 Description
This project was born out of my own curiosity to see if cloud gaming could be achieved on AWS. I designed and built a **GPU-powered Windows Server EC2 setup** from scratch for running **Parsec**, integrating cost optimization with **Spot Instances** and automation workflows using **CloudWatch** and **Lambda**.

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

### 1. VPC Setup
- Created a dedicated **Gaming VPC** (`10.50.0.0/20`) in `eu-north-1` region with 3 public subnets.
- Added an **S3 VPC Endpoint** for direct snapshot access.

### 2. Security Groups
- Configured inbound rules for RDP (TCP 3389) and Parsec (UDP 8000–8200).

### 3. EC2 Instance Configuration
- Launched **g4dn.xlarge** GPU instance with **Windows Server 2019 Base**.
- Requested vCPU limit increase for GPU instances.
- Attached Elastic IP for stable external connectivity.

### 4. Parsec Setup & Troubleshooting
- Used Parsec Cloud Preparation Tool from GitHub.
- Resolved GPU driver and resolution issues (Nvidia driver version **538.67**).
- Fixed Parsec login and encoding issues.

### 5. Spot Instances & Cost Optimization
- Created AMI image of configured EC2 instance.
- Deployed Spot Fleet across 3 AZs with 9 instance pools.
- Automated assignment of correct Security Groups using Launch Templates.

### 6. Automation: CloudWatch & Lambda
- Created custom CloudWatch metric `IdleState`.
- Set alarms for 15 min (warning) & 30 min (auto-shutdown).
- Lambda reduces Spot Fleet capacity to 0 on idle.

### ✅ Result
- Cloud gaming via Parsec with **automatic shutdown when idle**.
- Achieved **~68% cost reduction** using Spot Instances.

---

## 🖼️ Screenshots

### VPC Screenshot

<img src="Screenshots/1- VPC set up.png" width="750">

### Parsec Game Running

<img src="Screenshots/9.1-Palworld is running.png" width="750">

### CloudWatch

<img src="Screenshots/idle 15 alarm metric went off.png" width="750">

### SNS Email Notification

<img src="Screenshots/SNS idle 15 email.png" width="750">

### Auto Shutdown Proof

<img src="Screenshots/Log Events - Lambda.png" width="750">

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


## 🧑‍💻 Author
👋 Milos Faktor 💼 [LinkedIn](https://www.linkedin.com/in/milos-faktor-78b429255/)