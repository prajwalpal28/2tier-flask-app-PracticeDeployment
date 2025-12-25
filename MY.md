create EC2 instance ----> connect with local terminal

sudo apt install

**install docker-**
-sudo apt update && sudo apt upgrade -y
-sudo apt install -y ca-certificates curl gnupg lsb-release
-sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
-echo \
"deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
https://download.docker.com/linux/ubuntu \
$(lsb_release -cs) stable" | \
sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
-sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
-sudo systemctl start docker
sudo systemctl enable docker
-docker --version
-sudo docker run hello-world
-sudo usermod -aG docker $USER
-newgrp docker
-docker run hello-world


**Clone Git Repo**
-git clone <repo_URL>

**Create DockerFile**
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

