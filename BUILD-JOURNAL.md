### Do you want to see all screenshots from this project? 
👉 [All screenshots](Screenshots/)

# 🛠️ Build Journal: AWS Cloud Gaming EC2 + Spot Instances + Parsec

This build journal documents the process of setting up a **GPU-powered Windows Server EC2 instance** for cloud gaming using **Parsec**, and implementing an **automated shutdown system** for Spot Instances to optimize costs. The setup was created from scratch out of curiosity, exploring how AWS infrastructure could be leveraged for personal gaming in the cloud.

---

## 📖 Overview
- **Region:** `eu-north-1 (Stockholm)`
- **Instance Type:** `g4dn.xlarge (NVIDIA Tesla T4 GPU)`
- **Operating System:** Windows Server 2019
- **Goal:** Run cloud gaming workloads with Parsec and automate shutdown of Spot Instances to minimize costs.

---

## 🚀 Steps

### 1. VPC Setup
- Created a **dedicated VPC** for gaming:
  - CIDR Block: `10.50.0.0/20 (4096 IPs)`
  - 3 public subnets (1 in each AZ)
  - Enabled S3 VPC Endpoint for direct access to S3 storage (useful for EC2 snapshots).
- All components tagged with `Gaming` for easy identification.

**Screenshot:**
![VPC Setup](screenshots/vpc-setup.png)
<img src="Screenshots/SNS idle 15 email.png" width="750">

---

### 2. Security Groups
- Created a **Gaming Security Group** with inbound rules:
  - RDP (TCP 3389) – for first-time setup and configuration.
  - Custom UDP ports (8000–8200) – for Parsec connections.
- Restricted access to **my IP** for security.

**Screenshot:**
![Security Group](screenshots/security-group.png)

---

### 3. EC2 Gaming Instance Creation
- Launched **Windows Server 2019 Base** on a `g4dn.xlarge` instance.
- Requested a **vCPU limit increase** for G and VT GPU instance types via AWS Service Quotas.
- Created a new key pair `Gaming_Key` and downloaded the `.pem` file for RDP access.
- Enabled Auto-Assign Public IP and attached Elastic IP to keep a consistent external address.
- Attached the `Gaming` Security Group.
- Configured 100 GiB `gp3` storage.

**Screenshot:**
![EC2 Instance](screenshots/ec2-instance.png)

---

### 4. First-Time RDP Connection
- Used RDP client to log in.
- Decrypted the password using the downloaded `.pem` key.
- Disabled **IE Enhanced Security Configuration** to download Google Chrome.

**Screenshot:**
![RDP Connection](screenshots/rdp-connection.png)

---

### 5. Parsec Installation and Troubleshooting
- Used [Parsec Cloud Preparation Tool](https://github.com/parsec-cloud/Parsec-Cloud-Preparation-Tool).
- Encountered multiple issues:
  - GPU drivers failing to install properly.
  - Parsec resolution mismatch and software encoding instead of GPU acceleration.
  - Games not launching despite installing DirectX and dependencies.
- Solution:
  - Found a community fix [here](https://github.com/parsec-cloud/Parsec-Cloud-Preparation-Tool/issues/102#issuecomment-1522106779).
  - Installed Nvidia GPU driver version `572.16` using PowerShell scripts.
  - Modified PostInstall script in Parsec Cloud Prep Tool.
  - Verified GPU activity with `nvidia-smi` and Task Manager.

**Screenshot:**
![Parsec Running](screenshots/parsec-running.png)

---

### 6. Automating Spot Instance Cost Optimization
- Created an **AMI Image** from the configured EC2 instance.
- Launched Spot Fleet requests using multiple instance pools (`g4dn.xlarge`, `g4dn.2xlarge`, `g4dn.4xlarge`).
- Fixed Security Group assignment issue by creating a Launch Template with the correct Gaming Security Group.
- **Cost Savings:** Achieved ~68% reduction in hourly cost using Spot Instances.

**Screenshot:**
![Spot Instances](screenshots/spot-instances.png)

---

### 7. CloudWatch + Lambda Automation
- **Custom CloudWatch Metric:** `IdleState`
  - PowerShell scripts running on EC2:
    - Send `IdleState=0` (active) on startup.
    - Send `IdleState=1` (idle) every 5 minutes if Parsec is disconnected.
- **CloudWatch Alarms:**
  - `Idle_15_Alarm`: Warning email via SNS after 15 min idle.
  - `Idle_30_Alarm`: Shutdown trigger via SNS + Lambda after 30 min idle.
- **Lambda Function:**
  - Reduces Spot Fleet capacity to 0, terminating the instance.
  - Sends `IdleState=0` to reset alarms.

**Screenshots:**
| SNS Email Notification | Auto Shutdown Proof |
|-------------------------|-----------------------|
| ![SNS Email](screenshots/sns-email.png) | ![Instance Shutdown](screenshots/instance-shutdown.png) |

---

### 🎮 Final Test: Gaming Performance
- Installed Steam and successfully launched **Palworld**.
- Parsec delivered smooth gameplay with matching resolution and GPU acceleration.

**Screenshot:**
![Game Running](screenshots/game-running.png)

---

## ⚡ Challenges
- GPU driver compatibility on Windows Server 2019.
- Parsec resolution and encoding issues.
- CloudWatch metric reset behavior (“Maximum” vs “Average” statistic).
- Spot Fleet Security Group assignment.

## ✅ Outcome
- Cloud gaming setup fully functional with Parsec.
- Automated idle detection and shutdown of Spot Instances.
- **~68% cost savings** using Spot pricing.
- Fully documented for future replication and improvement.

---

## 🔗 References
- [Parsec Cloud Preparation Tool](https://github.com/parsec-cloud/Parsec-Cloud-Preparation-Tool)

---

## 🏷️ Tags
`AWS` `EC2` `Spot Instances` `Cloud Gaming` `Parsec` `CloudWatch` `Lambda` `Automation`




## 🧑‍💻 Author
👋 Milos Faktor 💼 [LinkedIn](https://www.linkedin.com/in/milos-faktor-78b429255/)