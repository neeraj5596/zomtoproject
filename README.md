# 🚀DevOps CI/CD Pipeline with Monitoring (Jenkins + Docker + SonarQube + Trivy + Prometheus + Grafana using Helm)

This project demonstrates a **complete production-style DevOps pipeline** deployed on an AWS EC2 Ubuntu instance.

The setup includes:

* CI/CD automation using Jenkins
* Containerization using Docker
* Code quality analysis using SonarQube
* Security scanning using Trivy
* Docker image scanning using Docker Scout
* Container registry using DockerHub
* Monitoring using Prometheus and Grafana deployed with Helm

This project simulates a **real-world DevOps environment** where code goes through build, scan, containerization, deployment, and monitoring.

---

# 🏗 Project Architecture

Developer → GitHub Repository → Jenkins Pipeline → SonarQube Code Analysis → Trivy Security Scan → Docker Image Build → DockerHub Push → Container Deployment → Prometheus Metrics Collection → Grafana Visualization

---

# ☁️ Infrastructure

| Component      | Configuration |
| -------------- | ------------- |
| Cloud Provider | AWS           |
| Instance       | EC2           |
| OS             | Ubuntu 24.04  |
| Instance Type  | t2.large      |
| Storage        | 30 GB         |

---

# 🔓 Required Ports

| Port | Service    |
| ---- | ---------- |
| 22   | SSH        |
| 8080 | Jenkins    |
| 9000 | SonarQube  |
| 3000 | Grafana    |
| 9090 | Prometheus |

---

# 1️⃣ Connect to EC2 Instance

```bash
ssh ubuntu@<EC2-PUBLIC-IP>
```

Switch to root user

```bash
sudo su
```

Update system

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

Install Java (required for Jenkins)

```bash
apt update -y

wget -O - https://packages.adoptium.net/artifactory/api/gpg/key/public | tee /etc/apt/keyrings/adoptium.asc

echo "deb [signed-by=/etc/apt/keyrings/adoptium.asc] https://packages.adoptium.net/artifactory/deb $(awk -F= '/^VERSION_CODENAME/{print$2}' /etc/os-release) main" | tee /etc/apt/sources.list.d/adoptium.list

apt update -y

apt install temurin-17-jdk -y
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
systemctl enable jenkins
systemctl status jenkins
```

Access Jenkins

```
http://<EC2-PUBLIC-IP>:8080
```

Get initial password

```bash
cat /var/lib/jenkins/secrets/initialAdminPassword
```

---

# 4️⃣ Install Docker

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

Add Docker permissions

```bash
usermod -aG docker ubuntu

chmod 777 /var/run/docker.sock

newgrp docker
```

Verify installation

```bash
docker --version
```

---

# 5️⃣ Install Trivy (Security Scanner)

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

# 6️⃣ Install SonarQube Using Docker

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

Default login

```
username: admin
password: admin
```

Generate **SonarQube Token** for Jenkins integration.

---

# 7️⃣ Install Kubernetes (for Monitoring Stack)

```bash
curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl"

chmod +x kubectl

mv kubectl /usr/local/bin/
```

Install Minikube

```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64

install minikube-linux-amd64 /usr/local/bin/minikube
```

Start Kubernetes cluster

```bash
minikube start
```

---

# 8️⃣ Install Helm

```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

Verify

```bash
helm version
```

---

# 9️⃣ Deploy Prometheus & Grafana using Helm

Add Helm repository

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

helm repo update
```

Install monitoring stack

```bash
helm install monitoring prometheus-community/kube-prometheus-stack
```

Check pods

```bash
kubectl get pods
```

---

# 🔟 Access Grafana

Forward Grafana port

```bash
kubectl port-forward svc/monitoring-grafana 3000:80
```

Open browser

```
http://localhost:3000
```

Get Grafana password

```bash
kubectl get secret monitoring-grafana -o jsonpath="{.data.admin-password}" | base64 --decode
```

---

# 1️⃣1️⃣ Jenkins Plugins

Install plugins:

* Git
* Docker
* Docker Pipeline
* SonarQube Scanner
* Email Extension
* Pipeline
* Credentials Binding

---

# 1️⃣2️⃣ Configure Jenkins

Configure:

* SonarQube Server
* DockerHub Credentials
* Email Notification
* SonarQube Token

Location:

```
Manage Jenkins → Configure System
```

---

# 1️⃣3️⃣ Jenkins Pipeline Stages

Pipeline performs:

1️⃣ Code Checkout from GitHub
2️⃣ Build Application
3️⃣ SonarQube Code Analysis
4️⃣ Trivy Security Scan
5️⃣ Docker Image Build
6️⃣ Push Image to DockerHub
7️⃣ Deploy Docker Container
8️⃣ Email Notification
9️⃣ Monitoring using Prometheus & Grafana

---

# 📊 Monitoring Stack (Helm Deployment)

Prometheus collects metrics from:

* Node Exporter
* Kubernetes Cluster
* Docker containers

Grafana visualizes metrics using dashboards such as:

* Node Exporter Full Dashboard
* Kubernetes Cluster Monitoring
* Container Resource Usage

---

# 🛠 Tools Used

| Tool       | Purpose                    |
| ---------- | -------------------------- |
| AWS EC2    | Infrastructure             |
| Jenkins    | CI/CD                      |
| Docker     | Containerization           |
| SonarQube  | Code Quality               |
| Trivy      | Security Scanning          |
| DockerHub  | Container Registry         |
| Kubernetes | Container Orchestration    |
| Helm       | Kubernetes Package Manager |
| Prometheus | Metrics Collection         |
| Grafana    | Monitoring Dashboard       |

---

# 🎯 Outcome

This project demonstrates a **complete DevOps pipeline with integrated monitoring**:

✔ Automated CI/CD pipeline
✔ Code quality analysis
✔ Container security scanning
✔ Docker image management
✔ Automated deployment
✔ Kubernetes monitoring using Helm
✔ Real-time dashboards with Grafana

---

# 👨‍💻 Author

**Neeraj Kumar Verma**

Cloud & DevOps Engineer
Skills: Linux • Docker • Jenkins • Kubernetes • Terraform • AWS • Azure

Email: [neerajverma9264@gmail.com](mailto:neerajbhanu143@@gmail.com)




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


