### Do you want to see all screenshots from this project? 
👉 [All screenshots](../Screenshots/)

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

## 1. VPC Creation (Gaming Network)
To isolate gaming workloads, I created a dedicated VPC:

- **Action:** In the VPC service, clicked on *“Create VPC”* → *“VPC and more”*.
- **Naming:** Set name tag to `Gaming` so all components inherit the same tag.
- **CIDR Block:** `10.50.0.0/20` (4096 IPs).
  - Why? Using a non-default CIDR allows future VPC peering with other VPCs using default IP ranges.
- **Region:** `eu-north-1 (Stockholm)`.
- **Subnets:** Created 3 public subnets, each in a different AZ for high availability.
- **VPC Endpoint:** Added S3 VPC Endpoint for private access to S3 (e.g., storing EC2 snapshots).

**Screenshot:**

<img src="../Screenshots/1- VPC set up.png" width="750">

---

## 2. Security Group Setup
Configured security rules to allow controlled access:

- **Action:** In EC2 service → *Security Groups* → *Create Security Group*.
- **Details:**
  - **Name:** `Gaming-SG`
  - **Attached VPC:** `Gaming`
  - **Inbound Rules:**
    - TCP 3389 (RDP) – For initial configuration (restricted to my IP).
    - UDP 8000–8200 – For Parsec connection (restricted to my IP).

**Screenshot:**

<img src="../Screenshots/2-Security Group.png" width="750">

---

## 3. EC2 Instance Launch (GamingServer)

### **a.** vCPU Limit Increase
- **Issue:** AWS imposes a **vCPU quota = 0** for GPU instances by default.
- **Solution:** Requested limit increase in *AWS Service Quotas* → *On-Demand G and VT Instances* & *Spot Instances*. Waited for AWS approval.

### **b.** EC2 Instance Configuration
- **Action:** EC2 service → *Launch Instance*.
- **Details:**
  - **Name:** `GamingServer`
  - **AMI:** Windows Server 2019 Base (Free Tier eligible)
  - **Instance Type:** g4dn.xlarge (NVIDIA Tesla T4 GPU)
  - **Price:** $0.742/hour (Windows, On-Demand)
  - **Key Pair:** Created `Gaming_Key` (.pem file downloaded for RDP access)
  - **Network Settings:**
    - VPC: `Gaming`
    - Subnet: Public Subnet (AZ-specific)
    - Auto-Assign Public IP: Enabled
    - Security Group: `Gaming-SG`
  - **Storage:** 100 GiB gp3 volume

### **c.** Elastic IP Allocation
- Allocated an Elastic IPv4 address and associated it with the EC2 instance’s private IP to ensure a **static public IP**, even after stop/start cycles.

**Screenshot:**

<img src="../Screenshots/3- Quotas EC2 instances.png" width="750">

---

## 4. First RDP Connection

### **a.** Connecting
- EC2 Console → *GamingServer* → *Connect* → *RDP Client*.
- Downloaded RDP file → Opened with RDP client.
- A certificate warning appeared – clicked *“Connect Anyway”*.

### **b.** Password Decryption
- In EC2 Console → *Get Password* → Uploaded `Gaming_Key.pem` → Decrypted password → Used for login.

### **c.** Post-Login Setup
- On first login, attempted to download Google Chrome.
- **Issue:** Blocked by *Internet Explorer Enhanced Security Configuration*.
- **Solution:**
  - Server Manager → *Local Server* → Disabled *IE Enhanced Security Configuration*.
  - Downloaded and installed Chrome.

**Screenshot:**

<img src="../Screenshots/4.1-RDP connect.png" width="750">

<img src="../Screenshots/4.3-first log in .png" width="750">

---

## 5. Parsec Preparation Tool Installation & Fixes

### **a.** Initial Setup Attempt
- **Tool Used:** [Parsec Cloud Preparation Tool](https://github.com/parsec-cloud/Parsec-Cloud-Preparation-Tool)
- Purpose: Automate Windows Server GPU configuration and install prerequisites for gaming.
- **Issue:**
  - The tool did not work as documented.
  - Encountered multiple driver and GPU configuration problems.

### **b.** Debugging & Troubleshooting
- Attempted multiple Nvidia driver versions manually.
- Verified GPU installation using:
  - Command: `nvidia-smi`
  - GPU activity in Task Manager.
- **Problems Faced:**
  - Parsec window resolution mismatch (only top-left quarter displayed).
  - Parsec used software encoding instead of GPU acceleration.
  - Games failed to launch (no error messages).
  - Tried Steam launch options (`-windowed`, `-dx12`) unsuccessfully.

### **c.** Solution & Community Fix
- Found a fix from a GitHub issue: [Community Fix Reference](https://github.com/parsec-cloud/Parsec-Cloud-Preparation-Tool/issues/102#issuecomment-1522106779)
- **Steps:**
  1. **Attach IAM Role:**
     - Created `GamingRoleEC2` with `AmazonS3ReadOnlyAccess`.
     - Assigned role to EC2 instance under *Actions → Security → Modify IAM Role*.
  2. **Install Correct Drivers:**
     - Ran PowerShell script from community fix.
     - Installed Nvidia driver version `572.16`.
     - Rebooted and verified GPU installation.
  3. **Parsec Cloud Prep Tool Modification:**
     - Edited `PostInstall` script: replaced `vigem\10\x64` with `vdd`.
     - Installed Parsec and logged in successfully.

**Screenshot:**

<img src="../Screenshots/7.4-Parsec connection-successfull.png" width="750">

---

## 6. Final Configuration & Game Test

### **a.** Parsec Connection
- Configured Parsec host settings:
  - Host Start Port: `8000`
  - Client configured to use port `8000`.
- Disconnected RDP and successfully connected via Parsec.

### **b.** Auto Login Configuration
- Enabled automatic login:
  - `Win + R → netplwiz`
  - Disabled “Users must enter a username and password.”
  - Applied settings and rebooted.

### **c.** Security Adjustments
- Restricted inbound traffic to **UDP 8000–8200** from my IP.
- Removed Elastic IP as Parsec does not require it.

### **d.** Game Test
- Installed Steam and tested **Palworld**.
- Result: *Palworld running smoothly with GPU acceleration.*

**Screenshot:**

<img src="../Screenshots/9.1-Palworld is running.png" width="750">

---

## 7. Spot Instances Configuration

### **a.** AMI Creation
- Created an **Amazon Machine Image (AMI)** of the configured Parsec EC2 instance.
- Purpose: Use this AMI to launch Spot Instances for cloud gaming.
- **Why Spot Instances?**
  - Cost-effective for personal gaming (but not recommended for production due to potential interruptions).

**Screenshot:**

<img src="../Screenshots/10.0-AMI created.png" width="750">

### **b.** Spot Fleet Setup
- Navigated to EC2 → *Spot Requests* → *Create Spot Fleet Request*.
- Chose newly created AMI.
- Configured:
  - **Target Capacity:** 1 instance.
  - **Instance Pools:**
    - g4dn.xlarge, g4dn.2xlarge, g4dn.4xlarge across 3 Availability Zones (total 9 pools).
  - Selected `Gaming VPC` and subnets in all AZs.
- Clicked *Launch* to create the Spot Fleet.

### **c.** Security Group Issue & Fix
- **Problem:** Newly launched Spot Instances were assigned the **default Security Group** instead of `Gaming-SG`.
- **Solution:**
  - Created a **Launch Template** with:
    - The Gaming AMI.
    - Correct Security Group (`Gaming-SG`).
  - Launched a new Spot Fleet using this Launch Template.
- Outcome: Spot Instances now correctly use the intended Security Group.

**Screenshot:**

<img src="../Screenshots/11.1- Security group fixed.png" width="750">

<img src="../Screenshots/10.2-spot request fullfilled.png" width="750">

### ✅ Result
- Successfully launched Spot Instance.
- Parsec connection worked seamlessly without RDP.
- **Cost Savings:** ~68% reduction compared to On-Demand pricing.

**Screenshot:**

<img src="../Screenshots/11.2- Spot savings.png" width="750">

---

## 8. Automating Idle Shutdown

### **a.** Objective
Automatically shut down Spot Instances when idle to further optimize costs.

### **b.** Implementation Steps

1. **Custom CloudWatch Metric:** `IdleState`
   - Two PowerShell scripts on the EC2 instance:
     - **Script 1 (Startup):** Sends `IdleState=0` (active) to CloudWatch.
     - **Script 2 (Idle Check):** Runs every 5 minutes; if no Parsec connection, sends `IdleState=1`.

**Screenshot:**

<img src="../Screenshots/13.0 - Parsec scripts.png" width="750">

<img src="../Screenshots/13.1- ec2 aws cli cloudwatch testing.png" width="750">

<img src="../Screenshots/13.2-task scheduler.png" width="750">

2. **CloudWatch Alarms:**
   - **Idle_15_Alarm:**
     - Triggers after 15 minutes idle.
     - Sends SNS warning email.
   - **Idle_30_Alarm:**
     - Triggers after 30 minutes idle.
     - Sends SNS shutdown email and invokes Lambda.

**Screenshot:**

<img src="../Screenshots/15 miute alarm went off.png" width="750">

<img src="../Screenshots/Alarm idle 30 triggered and  lambda sent 0 to metrics to bring alarms to OK state.png" width="750">

3. **Lambda Function:**
   - Invoked by SNS topic.
   - Reduces Spot Fleet capacity to 0 (terminates instance).
   - Sends `IdleState=0` to reset CloudWatch alarm.

4. **IAM & AWS CLI Configuration:**
   - Attached `cloudwatch:PutMetricData` to `GamingRoleEC2`.
   - Installed AWS CLI on EC2 instance (configured region `eu-north-1`).
   - Verified metrics upload to CloudWatch.

**Screenshot:**

<img src="../Screenshots/Lambda permissions 1.png" width="750">

### **c.** Testing & Validation
- Verified CloudWatch metrics appear under `TestidleParsec/IdleState`.
- Tested alarms by manually forcing `IdleState=1`.
- Triggered Lambda manually and confirmed Spot Fleet capacity reduced to 0.
- **Challenge:**
  - CloudWatch metric reset failed due to "Maximum" statistic configuration.
  - **Fix:** Switched to "Average" statistic for precise alarm behavior.

**Screenshot:**

<img src="../Screenshots/SNS idle 30 email.png" width="750">

<img src="../Screenshots/Log Events - Lambda.png" width="750">

<img src="../Screenshots/Lambda triggered and modifying spot request.png" width="750">

<img src="../Screenshots/Instance shutting down.png" width="750">

---

## ✅ Outcome
- Spot Instances shut down automatically after 30 minutes idle.
- Users receive:
  - 15 min warning email.
  - 30 min shutdown notification.
- Automation fully integrated with CloudWatch, Lambda, and Spot Fleet.

**Result:**
- **Cloud gaming system is cost-optimized and self-regulating.**

## ⚡ Challenges
- Nvidia GPU drivers compatibility on Windows Server.
- Parsec resolution and encoding issues.
- CloudWatch metric reset (“Maximum” vs “Average”).
- Spot Fleet Security Group assignment.

## ✅ Outcome
- Fully functional cloud gaming setup with Parsec.
- Spot Instances shut down automatically after 30 minutes idle.
- Achieved ~68% cost savings using Spot pricing.

---

## 🔗 References
- [Parsec Cloud Preparation Tool](https://github.com/parsec-cloud/Parsec-Cloud-Preparation-Tool)

---

## 🏷️ Tags
`AWS` `EC2` `Spot Instances` `Cloud Gaming` `Parsec` `CloudWatch` `Lambda` `Automation`

## 🧑‍💻 Author
👋 Milos Faktor 💼 [LinkedIn](https://www.linkedin.com/in/milos-faktor-78b429255/)
