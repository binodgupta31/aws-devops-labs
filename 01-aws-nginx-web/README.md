## 🚀 Host a Website on Amazon Linux 2 with Nginx (EC2 User Data Script)

<img width="716" height="474" alt="image" src="https://github.com/user-attachments/assets/cea8bcc0-601f-443e-aec9-fa6becf84c64" />



### 📌 Project Goal

Automate hosting a static website using **Amazon EC2 (t2.micro)** with **Amazon Linux 2**, **Nginx**, and a **user data script** inside **default VPC**.

---

### 🔧 Prerequisites

* AWS Account
* Key pair for EC2 (PEM file)
* GitHub account to store this doc

---

### 🛠️ Steps to Launch EC2 and Host Website

#### ✅ Step 1: Launch EC2 Instance

1. Go to **EC2 Dashboard** → Click **Launch Instance**
2. **Name**: `aws-nginx-web`
3. **AMI**: Amazon Linux 2 (x86\_64)
4. **Instance Type**: `t2.micro` (Free Tier)
5. **Key Pair**: Select or create a `.pem` file
6. **Network Settings**:

   * VPC: **default**
   * Subnet: **default**
   * Auto-assign Public IP: **Enabled**
   * Add **inbound rules**:

     * **HTTP (port 80)** — 0.0.0.0/0
     * **SSH (port 22)** — Your IP only (for security)

#### ✅ Step 2: Add User Data Script

In **Advanced Details → User data**, paste the following script:

```bash
#!/bin/bash
yum update -y
amazon-linux-extras enable nginx1
yum install nginx -y
systemctl start nginx
systemctl enable nginx

cat > /usr/share/nginx/html/index.html <<EOF
<html>
  <head><title>Welcome</title></head>
  <body>
    <h1>My Website on Amazon Linux using Nginx</h1>
    <p>Deployed automatically via EC2 user data script.</p>
  </body>
</html>
EOF

systemctl restart nginx
```

7. Click **Launch Instance**

---

### 🌐 Step 3: Access Your Website

* Go to your **EC2 Dashboard**
* Copy the **Public IPv4 address**
* Open your browser:
  `http://<your-public-ip>`

✅ You should see your custom web page.

## 🔍 Output Screenshot

<img width="729" height="297" alt="image" src="https://github.com/user-attachments/assets/8588cd27-ca6a-4ef3-a195-5ffa2624d971" />



---

### 🔐 Optional: SSH into EC2

If needed for troubleshooting:

```bash
ssh -i your-key.pem ec2-user@<your-public-ip>
```

---

### 📘 Notes

* You can replace the HTML content in the script with your actual website.
* You can pull code from GitHub/S3 in user data for dynamic deployment.


![Website Screenshot](screenshots/web-output.png)








