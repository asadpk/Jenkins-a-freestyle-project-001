# Building Your CI/CD Foundation: Jenkins Installation on AWS

## Introduction: Why Automation Matters

Imagine you’re part of a team building a web application. Every small code change requires testing, building, deploying, and validating again. Doing this manually is slow, error‑prone, and exhausting.

This is where **Jenkins** shines. Jenkins automates these steps into a repeatable pipeline, saving time, reducing mistakes, and letting teams focus on building features instead of babysitting deployments.

In this module, you’ll set up **Jenkins on AWS** from scratch. This guide is written as a practical, real‑world walkthrough — the same approach used when setting up Jenkins for actual teams and learning environments.

---

## Prerequisites: What You Need Before Starting

### Your Toolkit Checklist

* ✅ **AWS Account**
  A free‑tier AWS account is enough for learning.

* ✅ **Basic Linux Knowledge**
  Familiarity with commands like `ls`, `cd`, `sudo` is sufficient.

* ✅ **SSH Key Pair**
  We’ll use a key named `vscode-key`.

  **Create it from:**
  AWS Console → EC2 → Key Pairs → Create key pair → Download `.pem`

* ✅ **AWS CLI (Optional)**
  You may use AWS CloudShell instead if preferred.

---

## Part 1: Launching the Jenkins Server on AWS

### Architecture Overview

```
┌─────────────────────────────────────────┐
│              AWS Cloud                  │
│  ┌─────────────────────────────────┐   │
│  │        Ubuntu EC2 Instance       │   │
│  │  ┌───────────────────────────┐  │   │
│  │  │          Jenkins           │  │   │
│  │  │  ┌─────────────────────┐  │  │   │
│  │  │  │ Git | Java | Docker  │  │  │   │
│  │  │  │ AWS CLI               │  │  │   │
│  │  │  └─────────────────────┘  │  │   │
│  │  └───────────────────────────┘  │   │
│  └─────────────────────────────────┘   │
└─────────────────────────────────────────┘
```

### Why Ubuntu?

Ubuntu is beginner‑friendly, well‑documented, and widely supported by Jenkins and the DevOps community.

### Instance Type Recommendation

| Instance Type | Cost / Hour | Use Case         |
| ------------- | ----------- | ---------------- |
| t2.micro      | Low         | Testing only     |
| **t3.medium** | ⭐ Best      | Learning & CI/CD |
| t3.large      | Higher      | Small teams      |

We’ll use **t3.medium** for stability and performance.

---

## Part 2: Automating Server Setup with User Data

Instead of installing tools manually, we’ll use an **initialization script** that runs automatically when the EC2 instance starts.

### Setup Script: `jk-tools.sh`

```bash
#!/bin/bash

# Clean shell prompt
echo 'export PS1="$ "' >> /home/ubuntu/.bashrc

# Update system
apt update -y

# Install Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sh get-docker.sh
usermod -aG docker ubuntu

# Install AWS CLI
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o awscliv2.zip
apt install unzip -y
unzip awscliv2.zip
./aws/install

# Install Git
apt install git -y

# Install Java (required for Jenkins)
apt install openjdk-21-jre -y

# Cleanup
rm -rf get-docker.sh awscliv2.zip aws/
```

---

## Part 3: Launching the EC2 Instance

```bash
aws ec2 run-instances \
  --image-id ami-020cba7c55df1f615 \
  --count 1 \
  --instance-type t3.medium \
  --key-name vscode-key \
  --security-groups default \
  --block-device-mappings '[{"DeviceName":"/dev/sda1","Ebs":{"VolumeSize":30}}]' \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=jk-server}]' \
  --user-data file://jk-tools.sh
```

### Verify Setup

```bash
ssh -i "vscode-key.pem" ubuntu@<EC2_PUBLIC_IP>
cloud-init status
```

Once complete:

```bash
git --version
docker --version
aws --version
java -version
```

Enable Docker without sudo:

```bash
sudo usermod -aG docker $USER
newgrp docker
```

---

## Part 4: Installing Jenkins

### Add Jenkins Repository

```bash
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null

echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
```

### Install Jenkins

```bash
sudo apt update -y
sudo apt install jenkins -y
jenkins --version
```

### Start Jenkins Service

```bash
sudo systemctl enable jenkins
sudo systemctl start jenkins
sudo systemctl status jenkins
```

---

## Part 5: Accessing Jenkins

### Open Port 8080

AWS Console → EC2 → Security Groups → Inbound Rules

| Type       | Port | Source    |
| ---------- | ---- | --------- |
| Custom TCP | 8080 | 0.0.0.0/0 |

> ⚠️ For production, restrict to your IP only.

### Unlock Jenkins

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

Open in browser:

```
http://<EC2_PUBLIC_IP>:8080
```

---

## Part 6: Initial Jenkins Setup

* Install **Suggested Plugins**
* Create Admin User
* Confirm Jenkins URL

---

## Part 7: Jenkins Dashboard Overview

### Key Sections

* **New Item** – Create jobs & pipelines
* **Build History** – Execution timeline
* **Manage Jenkins** – Global settings

### Job Types

* **Freestyle Project** – Simple tasks
* **Pipeline** – Code‑based CI/CD
* **Multibranch Pipeline** – One pipeline per Git branch

### Build Status Indicators

| Icon | Meaning  |
| ---- | -------- |
| 🟢   | Success  |
| 🔴   | Failed   |
| 🟡   | Unstable |
| 🔵   | Running  |

---

## Part 8: Credentials & Security Best Practices

* Store secrets in **Manage Jenkins → Credentials**
* Never hardcode passwords in pipelines
* Use SSH keys and tokens

---

## Common Issues & Fixes

### Jenkins Not Accessible

```bash
sudo ufw allow 8080
```

### Docker Permission Issue

```bash
sudo usermod -aG docker $USER
newgrp docker
```

### Jenkins Not Starting

```bash
sudo tail -f /var/log/jenkins/jenkins.log
```

---

## Next Steps

### Week 1

* Create a simple Freestyle Job
* Run shell commands

### Week 2

* Create a Jenkins Pipeline
* Connect GitHub repository

### Week 3

* Build Docker images
* Deploy applications

---

## Useful Commands Reference

```bash
sudo systemctl start jenkins
sudo systemctl stop jenkins
sudo systemctl restart jenkins
sudo systemctl status jenkins
```

---

## Important Paths

| Purpose      | Path                       |
| ------------ | -------------------------- |
| Jenkins Home | `/var/lib/jenkins`         |
| Logs         | `/var/log/jenkins/`        |
| Plugins      | `/var/lib/jenkins/plugins` |

---

## Final Thoughts

This Jenkins setup is your **CI/CD playground**. Experiment, break things, rebuild them, and learn. Every experienced DevOps engineer started exactly where you are now.

You’ve built a real automation foundation — and this is only the beginning.

🚀
