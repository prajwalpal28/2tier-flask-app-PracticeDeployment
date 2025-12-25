# 🚀 2-Tier Flask + MySQL Application Deployment on AWS EC2 using Docker

This guide explains **step-by-step** how to deploy a **2-tier application** (Flask + MySQL) on an **AWS EC2 instance** using **Docker**.

---

## 🧱 Architecture Overview

* **Frontend / Backend**: Flask (Python)
* **Database**: MySQL
* **Platform**: AWS EC2 (Ubuntu)
* **Containerization**: Docker
* **Networking**: Docker custom bridge network

```
User → EC2 Public IP:5000 → Flask Container → MySQL Container
```

---

## 1️⃣ Create EC2 Instance & Connect

* Launch an **Ubuntu EC2 instance**
* Allow **ports 22, 5000, 3306** in Security Group
* Connect using SSH from local terminal

```bash
ssh -i key.pem ubuntu@<EC2_PUBLIC_IP>
```

---

## 2️⃣ Update System Packages

Keeps the OS updated and avoids dependency issues.

```bash
sudo apt update && sudo apt upgrade -y
```

---

## 3️⃣ Install Docker Engine

### Install required packages

```bash
sudo apt install -y ca-certificates curl gnupg lsb-release
```

### Add Docker GPG key

```bash
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

### Add Docker repository

```bash
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
https://download.docker.com/linux/ubuntu \
$(lsb_release -cs) stable" | \
sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

### Install Docker

```bash
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

### Start & enable Docker

```bash
sudo systemctl start docker
sudo systemctl enable docker
```

### Verify installation

```bash
docker --version
sudo docker run hello-world
```

### Run Docker without sudo

```bash
sudo usermod -aG docker $USER
newgrp docker
docker run hello-world
```

---

## 4️⃣ Clone Application Repository

```bash
git clone <repo_URL>
cd 2tier-flask-app-PracticeDeployment
```

---

## 5️⃣ Create Dockerfile for Flask App

This Dockerfile builds a lightweight Flask image with MySQL client support.

```dockerfile
FROM python:3.9-slim

WORKDIR /app

RUN apt-get update -y && \
    apt-get install -y \
        gcc \
        default-libmysqlclient-dev \
        pkg-config && \
    rm -rf /var/lib/apt/lists/*

COPY requirements.txt .

RUN pip install --upgrade pip
RUN pip install -r requirements.txt

COPY . .

EXPOSE 5000

CMD ["python", "app.py"]
```

---

## 6️⃣ Build & Run Flask Docker Image

### Build image

```bash
docker build -t flask-app .
```

### Run Flask container (standalone test)

```bash
docker run -d -p 5000:5000 --name flask-app flask-app
```

### Verify

```bash
docker ps
```

Access:

```
http://<EC2_PUBLIC_IP>:5000
```

---

## 7️⃣ Run MySQL Container (Standalone)

```bash
docker run -d \
 -p 3306:3306 \
 --name mysql \
 -e MYSQL_ROOT_PASSWORD=admin \
 mysql:5.7
```

---

## 8️⃣ Create Docker Network

This allows containers to communicate using container names.

```bash
docker network create two-tier-network
```

---

## 9️⃣ Re-run Containers on Same Network

### Remove old containers

```bash
docker rm -f mysql flask-app
```

### Run MySQL on network

```bash
docker run -d \
  -p 3306:3306 \
  --name mysql \
  --network two-tier-network \
  -e MYSQL_ROOT_PASSWORD=admin \
  -e MYSQL_DATABASE=myDB \
  -e MYSQL_USER=admin \
  -e MYSQL_PASSWORD=admin \
  mysql:5.7
```

### Run Flask on network

```bash
docker run -d -p 5000:5000 \
--network=two-tier-network \
-e MYSQL_HOST=mysql \
-e MYSQL_USER=admin \
-e MYSQL_PASSWORD=admin \
-e MYSQL_DB=myDB \
--name flaskapp \
flask-app:latest
```

---

## 🔟 Verify Networking & Containers

```bash
docker ps
docker network ls
docker network inspect two-tier-network
```

---

## 1️⃣1️⃣ Access MySQL Inside Container

```bash
docker exec -it mysql bash
mysql -u root -p
```

SQL Commands:

```sql
SHOW DATABASES;
USE myDB;
CREATE TABLE messages (
  id INT AUTO_INCREMENT PRIMARY KEY,
  message TEXT
);
```

---

## 1️⃣2️⃣ Debug & Logs

```bash
docker logs flaskapp
```

---

## 1️⃣3️⃣ Access Final Application

```
http://<EC2_PUBLIC_IP>:5000
```

---

## ✅ What You Achieved

* Dockerized Flask application
* MySQL running in separate container
* Custom Docker network
* Environment-based configuration
* Real-world 2-tier architecture

---

## 🚀 Next Improvements

* docker-compose.yml
* Push images to AWS ECR
* Deploy on Kubernetes (EKS)
* Add Jenkins CI/CD pipeline

---

**Author:** Prajwal Pal
**Purpose:** DevOps / Docker / AWS Practice Deployment
