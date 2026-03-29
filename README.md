# 🚀 CI/CD DevOps Pipeline Setup (Jenkins + Docker + SonarQube + Trivy)

This project demonstrates a **complete DevOps CI/CD pipeline setup** on an Ubuntu EC2 instance using **Jenkins, Docker, SonarQube, Trivy, and Docker Scout**.

The pipeline automates:

* Code build
* Static code analysis
* Security scanning
* Docker image creation
* Image push to DockerHub
* Deployment using Docker containers
* Email notifications for build status

---

# 🏗 Architecture

Developer → GitHub Repository → Jenkins Pipeline → SonarQube Analysis → Trivy Security Scan → Docker Build → DockerHub Push → Container Deployment

---

# ☁️ Infrastructure Setup

Launch an EC2 instance with the following configuration:

| Parameter     | Value          |
| ------------- | -------------- |
| OS            | Ubuntu 24.04   |
| Instance Type | t2.large       |
| Storage       | 30 GB          |
| Ports         | 22, 8080, 9000 |

---

# 1️⃣ Connect to EC2 Instance

```bash
ssh ubuntu@<EC2-PUBLIC-IP>
```

Switch to root user

```bash
sudo su
```

Update packages

```bash
apt update -y
```

---

# 2️⃣ Install AWS CLI

```bash
apt install unzip -y

curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"

unzip awscliv2.zip

./aws/install
```

Verify installation

```bash
aws --version
```

---

# 3️⃣ Install Jenkins

```bash
apt update -y

wget -O - https://packages.adoptium.net/artifactory/api/gpg/key/public | tee /etc/apt/keyrings/adoptium.asc

echo "deb [signed-by=/etc/apt/keyrings/adoptium.asc] https://packages.adoptium.net/artifactory/deb $(awk -F= '/^VERSION_CODENAME/{print$2}' /etc/os-release) main" | tee /etc/apt/sources.list.d/adoptium.list

apt update -y

apt install temurin-17-jdk -y

java --version
```

Install Jenkins

```bash
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null

echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/ | tee /etc/apt/sources.list.d/jenkins.list > /dev/null

apt update -y

apt install jenkins -y
```

Start Jenkins

```bash
systemctl start jenkins
systemctl status jenkins
```

Verify

```bash
jenkins --version
```

---

# 4️⃣ Access Jenkins

Open port **8080** in EC2 Security Group.

Access Jenkins in browser:

```
http://<EC2-PUBLIC-IP>:8080
```

Get initial admin password:

```bash
cat /var/lib/jenkins/secrets/initialAdminPassword
```

Complete Jenkins setup wizard.

---

# 5️⃣ Install Docker

```bash
apt-get update

apt-get install ca-certificates curl -y

install -m 0755 -d /etc/apt/keyrings

curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc

chmod a+r /etc/apt/keyrings/docker.asc
```

Add Docker repository

```bash
echo \
"deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
$(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
tee /etc/apt/sources.list.d/docker.list > /dev/null
```

Install Docker

```bash
apt-get update

apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
```

Add user to Docker group

```bash
usermod -aG docker ubuntu

chmod 777 /var/run/docker.sock

newgrp docker
```

Verify

```bash
docker --version
```

---

# 6️⃣ Install Trivy (Container Security Scanner)

```bash
apt-get install wget apt-transport-https gnupg -y

wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | gpg --dearmor | tee /usr/share/keyrings/trivy.gpg > /dev/null

echo "deb [signed-by=/usr/share/keyrings/trivy.gpg] https://aquasecurity.github.io/trivy-repo/deb generic main" | tee -a /etc/apt/sources.list.d/trivy.list

apt-get update

apt-get install trivy -y
```

Verify

```bash
trivy --version
```

---

# 7️⃣ Install SonarQube Using Docker

Run SonarQube container

```bash
docker run -d --name sonar -p 9000:9000 sonarqube:lts-community
```

Check container

```bash
docker ps
```

Access SonarQube

```
http://<EC2-PUBLIC-IP>:9000
```

Default credentials

```
username: admin
password: admin
```

---

# 8️⃣ Jenkins Plugin Installation

Install the following plugins from **Manage Jenkins → Plugins**

* Docker
* Docker Pipeline
* SonarQube Scanner
* Pipeline
* Git
* Email Extension
* Credentials Binding
* OWASP Dependency Check

---

# 9️⃣ Configure SonarQube in Jenkins

Go to:

```
Manage Jenkins → Configure System
```

Add SonarQube server.

Then configure tools:

```
Manage Jenkins → Global Tool Configuration
```

Add:

* SonarQube Scanner

---

# 🔑 Credentials Setup

Add credentials in Jenkins:

1️⃣ SonarQube Token
2️⃣ DockerHub Username & Password
3️⃣ Email SMTP credentials

Location:

```
Manage Jenkins → Manage Credentials
```

---

# 📧 Email Notification Setup

Configure SMTP settings in Jenkins so that **build notifications are sent via email** after each pipeline execution.

---

# 🔗 SonarQube Webhook

Create webhook in SonarQube:

```
Administration → Webhooks
```

Add Jenkins webhook URL.

---

# ⚙️ Jenkins Pipeline

Create a **Pipeline Job** in Jenkins and configure the Jenkinsfile.

Pipeline stages include:

* Checkout code
* Build application
* SonarQube code analysis
* Trivy vulnerability scan
* Docker image build
* Push image to DockerHub
* Deploy container
* Send email notification

---

# 🐳 DockerHub Integration

In pipeline configuration, replace:

```
<your-dockerhub-username>
```

With your DockerHub username in these stages:

* Tag and Push to DockerHub
* DockerScoutImage
* Deploy to Container

---

# 📌 Tools Used

| Tool         | Purpose                      |
| ------------ | ---------------------------- |
| Jenkins      | CI/CD automation             |
| Docker       | Containerization             |
| SonarQube    | Code quality analysis        |
| Trivy        | Security scanning            |
| Docker Scout | Image vulnerability scanning |
| AWS EC2      | Infrastructure               |

---

# 🎯 Outcome

This project demonstrates a **production-style DevOps pipeline** that integrates:

* Continuous Integration
* Continuous Security Scanning
* Containerization
* Automated Deployment

---

# 👨‍💻 Author

**Neeraj Kumar Verma**

Cloud & DevOps Engineer
Skills: Linux • Docker • Jenkins • Kubernetes • Terraform • AWS • Azure

Email: [neerajverma9264@gmail.com](mailto:neerajverma9264@gmail.com)



After successfully deploying
===============================

Grafana
===============================================
<img width="950" height="464" alt="image" src="https://github.com/user-attachments/assets/c3c16c26-4d57-4f88-a09c-04a78ba66133" />

Jenkins
=================================================

<img width="929" height="379" alt="image" src="https://github.com/user-attachments/assets/6d08487c-4e8d-4a81-9c22-85ea9b88d4ee" />

Sonarqube
====================================================
<img width="955" height="440" alt="image" src="https://github.com/user-attachments/assets/e384785f-74a1-4785-895a-4a056be9008b" />

Email Notification
=====================================================
<img width="731" height="341" alt="image" src="https://github.com/user-attachments/assets/ef3cae67-044c-4e6b-87a4-6fc762138d87" />


