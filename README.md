# HOSTINGER_VPC-CI-CD-JENKINS-SETUP


# Jenkins CI/CD Setup on Hostinger VPS (Docker + Docker Compose)

This repository provides a **complete, production-ready, step-by-step guide** to set up **Jenkins CI/CD on a Hostinger VPS** using **Docker and Docker Compose**.

---

## Architecture Overview

```
Developer Push → GitHub → Jenkins (Hostinger VPS)
                             ↓
                       Docker Build
                             ↓
                    Docker Compose Deploy
```

---

## Prerequisites

- Hostinger VPS (Ubuntu 20.04 / 22.04)
- GitHub repository
- Basic Linux knowledge
- Ports open in Hostinger Firewall:
  - 22 (SSH)
  - 8080 (Jenkins)
  - 80 / 443 (Application)

---

## Step 1: Connect to Hostinger VPS

```bash
ssh root@<VPS_PUBLIC_IP>
```

---

## Step 2: Update System

```bash
apt update && apt upgrade -y
reboot
```

Reconnect after reboot.

---

## Step 3: Install Docker

```bash
curl -fsSL https://get.docker.com | sh
systemctl enable docker
systemctl start docker
```

Verify:
```bash
docker ps
```

---

## Step 4: Install Docker Compose

```bash
apt install docker-compose-plugin -y
docker compose version
```

---

## Step 5: Install Java (Required for Jenkins)

```bash
apt install openjdk-17-jdk -y
java -version
```

---

## Step 6: Install Jenkins

```bash
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null

echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/ | tee /etc/apt/sources.list.d/jenkins.list > /dev/null

apt update
apt install jenkins -y
systemctl enable jenkins
systemctl start jenkins
```

Verify:
```bash
systemctl status jenkins
```

---

## Step 7: Allow Jenkins to Use Docker (CRITICAL)

```bash
usermod -aG docker jenkins
reboot
```

---

## Step 8: Configure Firewall

```bash
ufw allow 22
ufw allow 8080
ufw allow 80
ufw allow 443
ufw enable
```

---

## Step 9: Access Jenkins UI

Open browser:
```
http://<VPS_PUBLIC_IP>:8080
```

Get admin password:
```bash
cat /var/lib/jenkins/secrets/initialAdminPassword
```

- Install suggested plugins
- Create admin user

---

## Step 10: Project Structure

```
.
├── Dockerfile
├── docker-compose.yml
├── Jenkinsfile
└── app/
```

---

## Step 11: Sample Dockerfile

```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["npm", "start"]
```

---

## Step 12: Sample docker-compose.yml

```yaml
version: "3.8"
services:
  app:
    build: .
    ports:
      - "3000:3000"
    restart: always
```

---

## Step 13: Jenkinsfile (CI/CD Pipeline)

```groovy
pipeline {
    agent any

    stages {

        stage('Clone Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/your-org/your-repo.git'
            }
        }

        stage('Build') {
            steps {
                sh 'docker compose build'
            }
        }

        stage('Stop Existing Containers') {
            steps {
                sh 'docker compose down || true'
            }
        }

        stage('Deploy') {
            steps {
                sh 'docker compose up -d'
            }
        }

        stage('Verify Deployment') {
            steps {
                sh 'docker ps'
            }
        }
    }

    post {
        success {
            echo 'CI/CD Pipeline executed successfully'
        }
        failure {
            echo 'CI/CD Pipeline failed'
        }
    }
}
```

---

## Step 14: Create Jenkins Pipeline Job

1. Jenkins Dashboard → New Item
2. Select **Pipeline**
3. Choose **Pipeline from SCM**
4. Add GitHub repository URL
5. Branch: `main`
6. Script Path: `Jenkinsfile`
7. Save

---

## Step 15: Enable GitHub Webhook (Optional)

GitHub → Settings → Webhooks

Payload URL:
```
http://<VPS_PUBLIC_IP>:8080/github-webhook/
```

Content type:
```
application/json
```

Event:
```
Push
```

---

## Final Result

- Code push triggers Jenkins
- Jenkins builds Docker image
- Docker Compose deploys application automatically

---

## Common Issues

### Docker Permission Denied
```bash
usermod -aG docker jenkins
reboot
```

### Jenkins Not Accessible
- Check firewall rules
- Verify port 8080 is open

---

## Author
DevOps CI/CD Setup Guide

---

## License
MIT
