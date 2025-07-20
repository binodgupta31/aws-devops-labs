
# 🧪 AWS ALB Lab with Route 53 and HTTPS using ACM

## Hosted on: `https://groceryhub.tech`

This lab demonstrates how to launch a 2-tier architecture with EC2 instances behind an Application Load Balancer (ALB), secured with HTTPS via ACM, and DNS managed by Route 53.

> ✅ **All steps are performed manually from the AWS Management Console.**
> 🔐 Domain used: `groceryhub.tech` (hosted on Hostinger)

---

## 🔧 Prerequisites

* AWS Free Tier Account
* Domain purchased (e.g., from Hostinger, GoDaddy, etc.)
* Basic understanding of AWS VPC, EC2, ALB, Route 53

---

## 📚 Architecture Overview

```
Client (Browser)
      |
      v
Route 53 (DNS)
      |
      v
Application Load Balancer (ALB)
  |                 |
EC2-1 (AZ-1a)   EC2-2 (AZ-1b)
```

---

## ✅ Step-by-Step Lab Implementation

---

### 1️⃣ Create a VPC

* Go to **VPC > Your VPCs > Create VPC**
* Name: `alb-lab-vpc`
* IPv4 CIDR: `10.0.0.0/16`
* Keep rest as default
* Click **Create VPC**

---

### 2️⃣ Create Public Subnets in 2 AZs

* Go to **Subnets > Create Subnet**
* VPC: `alb-lab-vpc`
* Subnet 1:

  * Name: `public-subnet-1`
  * AZ: `ap-south-1a`
  * CIDR: `10.0.1.0/24`
* Subnet 2:

  * Name: `public-subnet-2`
  * AZ: `ap-south-1b`
  * CIDR: `10.0.2.0/24`

✅ Both will be used for EC2 in different AZs.

---

### 3️⃣ Create Internet Gateway and Attach to VPC

* Go to **Internet Gateways > Create**

  * Name: `alb-lab-igw`
* Create and then **Attach to VPC** `alb-lab-vpc`

---

### 4️⃣ Create and Associate Public Route Table

* Go to **Route Tables > Create**

  * Name: `public-rt`
  * VPC: `alb-lab-vpc`
* Edit Routes:

  * Add route: `0.0.0.0/0` → target: `Internet Gateway (alb-lab-igw)`
* Edit Subnet Associations:

  * Attach: `public-subnet-1` and `public-subnet-2`

---

### 5️⃣ Create Security Groups

#### a) ALB SG: `alb-sg`

* Allow:

  * HTTP (80) – from `0.0.0.0/0`
  * HTTPS (443) – from `0.0.0.0/0`

#### b) Web Server SG: `web-sg`

* Allow:

  * HTTP (80) – from **security group** `alb-sg`
  * HTTPS (443) – from **security group** `alb-sg`
  * SSH (22) – from **My IP**

---

### 6️⃣ Launch EC2 Instances in Public Subnets

#### Create 2 EC2 Instances:

* AMI: **Amazon Linux 2**
* Instance type: `t2.micro`
* Network: `alb-lab-vpc`
* Subnet: Use `public-subnet-1` for EC2-1, `public-subnet-2` for EC2-2
* Enable: **Auto-assign public IP**
* Security Group: `web-sg`
* User data (installs NGINX):

```bash
#!/bin/bash
yum update -y
amazon-linux-extras enable nginx1
yum install -y nginx
systemctl start nginx
systemctl enable nginx
echo "<h1>Welcome to EC2 $(hostname -f)</h1>" > /usr/share/nginx/html/index.html
```

✅ Launch both instances successfully.

---

### 7️⃣ Create Target Group (TG)

* Go to **EC2 > Target Groups > Create**
* Name: `alb-tg`
* Target type: **Instance**
* Protocol: **HTTP**, Port: **80**
* VPC: `alb-lab-vpc`
* Health check path: `/`
* Register both EC2s as targets

---

### 8️⃣ Create Application Load Balancer (ALB)

* Go to **EC2 > Load Balancers > Create ALB**
* Name: `alb-lab`
* Scheme: **Internet-facing**
* Network: `alb-lab-vpc`
* Subnets: Select both public subnets
* Security Group: `alb-sg`
* Listener: **HTTP (80)** (you'll add HTTPS later)
* Target group: `alb-tg`

✅ ALB now forwards traffic to EC2s.

---

### 9️⃣ Create Route 53 Hosted Zone

* Go to **Route 53 > Hosted Zones > Create**
* Domain: `groceryhub.tech`
* Type: **Public Hosted Zone**

---

### 🔁 Map Domain to Route 53

* Go to Hostinger (or your registrar)
* Open DNS panel
* Replace existing **NS records** with **Route 53 NS records**

⏱ Wait for 10–15 mins for DNS propagation

---

### 🔄 Create Alias A Record in Route 53

* Hosted Zone: `groceryhub.tech`
* Click **Create Record**

  * Record type: **A**
  * Name: (Leave blank for root domain, or enter `www`)
  * Alias: **Yes**
  * Target: Select your ALB DNS name

---

### 🔒 Issue ACM Certificate for HTTPS

* Go to **ACM (Certificate Manager) > Request Certificate**
* Request public certificate:

  * Domain: `groceryhub.tech`
* Choose **DNS validation**
* ACM gives DNS record – click **“Create record in Route 53”**
* Wait until **Status: Issued**

---

### ➕ Add HTTPS Listener to ALB

* Go to **Load Balancer > Listeners > Add Listener**

  * Protocol: **HTTPS**
  * Port: **443**
  * Choose ACM certificate: `groceryhub.tech`
  * Action: Forward to `alb-tg`

✅ ALB now serves both HTTP and HTTPS traffic.

---

### ✅ Final Test

Open a browser and visit:

* `http://groceryhub.tech` → should auto-redirect or serve content
* `https://groceryhub.tech` → secured with SSL and NGINX welcome page

<img width="648" height="335" alt="web-server-1-images" src="https://github.com/user-attachments/assets/f94bb101-f3aa-4358-a735-f6163126b949" />

<img width="660" height="323" alt="web-server-2-image" src="https://github.com/user-attachments/assets/de4f8596-4d22-47ac-96ca-4b8c79506b93" />



---

## 🧹 Optional Cleanup (To Avoid Charges)

1. Terminate EC2 instances
2. Delete Target Group
3. Delete Load Balancer
4. Delete Hosted Zone and NS records from domain provider
5. Delete ACM Certificate
6. Delete Security Groups
7. Delete VPC and subnets

---

## 🧾 Credits

* Prepared by: Binod Gupta
* GitHub: https://github.com/binodgupta31/aws-devops-labs
* LinkedIn: https://www.linkedin.com/in/binodgupta94/
* Hosted on AWS Free Tier

