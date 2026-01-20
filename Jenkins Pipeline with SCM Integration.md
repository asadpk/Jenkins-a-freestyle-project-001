Jenkins Pipeline with SCM Integration
📋 Overview

This guide demonstrates how to create a Jenkins Pipeline using “Pipeline Script from SCM”.
The pipeline automates clone, build, and deploy steps while keeping the Jenkinsfile versioned in Git for better maintainability, traceability, and consistency.

🎯 Prerequisites

Jenkins server with Docker installed

GitHub repository access

Docker installed on Jenkins server

Git client installed on local machine

🔧 Step 1: Environment Cleanup

Before starting, clean up any existing Docker containers and images:

# Stop and remove existing containers
docker stop webapp_ctr 2>/dev/null || true
docker rm webapp_ctr 2>/dev/null || true

# Clean Docker system
docker system prune -a -f --volumes

# Verify cleanup
docker ps -a
docker images

🖥️ Step 2: Jenkins IP Auto-Update Script (AWS EC2)

This script automatically updates the Jenkins URL with the EC2 public IP after every reboot.

📌 Create the Script
sudo nano /usr/local/bin/update-jenkins-ip.sh

📄 Script Content
#!/bin/bash
echo "$(date): Updating Jenkins IP..."

# Get EC2 public IP using IMDSv2
TOKEN=$(curl -X PUT "http://169.254.169.254/latest/api/token" \
  -H "X-aws-ec2-metadata-token-ttl-seconds: 21600" 2>/dev/null)

NEW_IP=$(curl -H "X-aws-ec2-metadata-token: $TOKEN" \
  http://169.254.169.254/latest/meta-data/public-ipv4 2>/dev/null)

if [ -z "$NEW_IP" ]; then
    echo "Error: Could not retrieve the IP address"
    exit 1
fi

echo "New IP: $NEW_IP"

CONFIG_FILE="/var/lib/jenkins/jenkins.model.JenkinsLocationConfiguration.xml"

if [ -f "$CONFIG_FILE" ]; then
    sudo sed -i \
      "s|<jenkinsUrl>http://[^:]*:8080/</jenkinsUrl>|<jenkinsUrl>http://$NEW_IP:8080/</jenkinsUrl>|" \
      "$CONFIG_FILE"

    sudo systemctl restart jenkins
    echo "Jenkins updated to: http://$NEW_IP:8080/"
else
    echo "Jenkins configuration file not found. Update manually via UI."
fi

🔐 Set Permissions & Test
sudo chmod +x /usr/local/bin/update-jenkins-ip.sh
sudo /usr/local/bin/update-jenkins-ip.sh

⚙️ Automate with systemd
📌 Create Service File
sudo nano /etc/systemd/system/jenkins-ip-update.service

📄 Service Definition
[Unit]
Description=Update Jenkins IP
After=jenkins.service
Wants=jenkins.service

[Service]
Type=oneshot
ExecStart=/usr/local/bin/update-jenkins-ip.sh
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target

▶ Enable & Reload
sudo systemctl daemon-reload
sudo systemctl enable jenkins-ip-update.service

🔍 Verify
# Fetch EC2 public IP
TOKEN=$(curl -X PUT "http://169.254.169.254/latest/api/token" \
  -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")

curl -H "X-aws-ec2-metadata-token: $TOKEN" \
  http://169.254.169.254/latest/meta-data/public-ipv4

# Check Jenkins config
cat /var/lib/jenkins/jenkins.model.JenkinsLocationConfiguration.xml

🔄 Step 3: Local Git Workflow
📥 Clone Repository
git clone https://github.com/asadpk/Jenkins-Tasks.git
cd Jenkins-Tasks

📄 Create Jenkinsfile

File: Jenkinsfile

pipeline {
    agent any

    stages {
        stage('Cloning Git Repository') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/asadpk/Jenkins-Tasks.git'
            }
        }

        stage('Building Docker Image') {
            steps {
                sh 'docker build -t webapp:${BUILD_NUMBER} .'
            }
        }

        stage('Deploy Application') {
            steps {
                sh '''
                  docker run -d --rm \
                  -p 3000:3000 \
                  --name webapp_ctr \
                  webapp:${BUILD_NUMBER}
                '''
            }
        }
    }
}

📤 Commit & Push
git add Jenkinsfile
git commit -m "Add Jenkinsfile for Jenkins pipeline with SCM"
git push origin main

⚙️ Step 4: Jenkins Pipeline Configuration

Jenkins Dashboard → New Item

Item Name: jk-pipeline-scm

Select Pipeline → OK

🔧 Pipeline Settings

Definition: Pipeline script from SCM

SCM: Git

Repository URL:

https://github.com/asadpk/Jenkins-Tasks.git


Branch: */main

Script Path: Jenkinsfile

👉 Save the configuration

🚀 Step 5: Run & Verify
▶ Run Pipeline

Click Build Now

Monitor stages in Blue Ocean / Console Output

🌐 Access Application
http://<jenkins-server-ip>:3000

🐛 Troubleshooting
🔐 Git Authentication Errors

Use GitHub Personal Access Token

Add token in Jenkins Credentials

🐳 Docker Permission Issues
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins

🔌 Port Conflicts
sudo netstat -tulpn | grep :3000

📁 Repository Structure
Jenkins-Tasks/
├── Jenkinsfile        # Jenkins pipeline definition
├── Dockerfile         # Docker image instructions
├── src/               # Application source code
├── package.json       # Node.js dependencies
└── README.md          # Project documentation

🔗 Useful Links

Jenkins Pipeline Documentation

Docker Documentation

GitHub Documentation

📝 Notes

Jenkins IP auto-updates on every EC2 reboot

Pipeline logic is fully version-controlled

Use tokens instead of passwords for GitHub

Jenkinsfile acts as disaster recovery for pipeline

✅ Verification Checklist

 Jenkins IP auto-update working

 Jenkinsfile pushed to GitHub

 Pipeline configured with SCM

 All stages execute successfully

 App accessible on port 3000

 Docker images tagged with build number

👤 Maintainer

Asad

📦 Repository

https://github.com/asadpk/Jenkins-Tasks

📅 Last Updated
2026-01-20
