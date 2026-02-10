# docker-challenge
docker-push challenge

---

# Docker & DevOps Challenge 🚀

## Overview

This repository documents a complete **Docker & DevOps hands-on challenge**, covering:

* Building **custom Docker images**
* Pushing images to **Docker Hub** and **Amazon ECR**
* Provisioning **EC2 using Terraform**
* Installing and configuring **Jenkins**
* Creating a **CI pipeline** to build & push Docker images
* Dockerizing **Frontend, Java, Node.js applications**
* Using **Docker Compose** to deploy WordPress with MySQL

This project demonstrates real-world DevOps workflows using **Docker, AWS, Terraform, Jenkins, and CI/CD pipelines**.

---

## 1️⃣ Docker Challenge – Build & Push Images

### 1.1 Create a Customized Docker Image (Flask App)

#### Project Structure

```
myapp/
├── app.py
└── Dockerfile
```

#### app.py

```python
from flask import Flask

app = Flask(__name__)

@app.route("/")
def hello():
    return "Hello from custom Docker image!"

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```

#### Dockerfile

```dockerfile
FROM python:3.11-slim
WORKDIR /app
RUN pip install flask
COPY app.py .
EXPOSE 5000
CMD ["python", "app.py"]
```

#### Build Image

```bash
docker build -t my-custom-app:1.0 .
```

Verify:

```bash
docker images
```

---

### 1.2 Push Image to Docker Hub

```bash
docker login
docker tag my-custom-app:1.0 shahbazz16/my-custom-app:1.0
docker push shahbazz16/my-custom-app:1.0
```

✅ Image successfully pushed to Docker Hub.

---

### 1.3 Push the Same Image to Amazon ECR

#### Create ECR Repository

```bash
aws ecr create-repository \
--repository-name my-custom-app \
--region us-east-1
```

#### Login to ECR

```bash
aws ecr get-login-password --region us-east-1 |
docker login --username AWS --password-stdin 444152781336.dkr.ecr.us-east-1.amazonaws.com
```

#### Tag & Push Image

```bash
docker tag my-custom-app:1.0 \
444152781336.dkr.ecr.us-east-1.amazonaws.com/my-custom-app:1.0

docker push 444152781336.dkr.ecr.us-east-1.amazonaws.com/my-custom-app:1.0
```

✅ Image successfully pushed to Amazon ECR.

---

## 2️⃣ Terraform + Jenkins CI/CD Setup

### 2.1 Provision EC2 with Terraform & Install Jenkins

#### Folder Structure

```
terraform-jenkins/
└── main.tf
```

#### Terraform Configuration (main.tf)

* Provisions **EC2**
* Installs **Java 17**
* Installs **Jenkins**
* Installs **Docker**
* Adds Jenkins & Ubuntu users to Docker group

(Refer to `main.tf` in repository)

#### Terraform Commands

```bash
terraform init
terraform validate
terraform plan
terraform apply -auto-approve
```

✅ Jenkins EC2 instance created.

---

### 2.2 Access Jenkins

```text
http://<EC2_PUBLIC_IP>:8080
```

Initial Admin Password:

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

---

### 2.3 Jenkins Pipeline – Build & Push Docker Image

#### Required Plugins

* Git
* Docker Pipeline

#### Docker Hub Credentials

| Field    | Value           |
| -------- | --------------- |
| ID       | dockerhub-creds |
| Username | shahbazz16      |
| Password | ********        |

#### Jenkins Pipeline Script

```groovy
pipeline {
    agent any

    environment {
        IMAGE_NAME = "python-app"
        IMAGE_TAG  = "v1"
        DOCKERHUB_USER = "shahbazz16"
    }

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/shaikhshahbazz/Python-app.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }

        stage('Tag Image') {
            steps {
                sh "docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}"
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh """
                    echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin
                    docker push ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}
                    """
                }
            }
        }
    }
}
```

✅ Image pushed to Docker Hub via Jenkins CI pipeline.

---

## 3️⃣ Dockerizing Multi-Tier Applications

### 3.1 Frontend Application (NGINX)

**Repo:** [https://github.com/betawins/docker-tasks.git](https://github.com/betawins/docker-tasks.git)

```dockerfile
FROM nginx:latest
LABEL maintainer="shahbazz"
RUN rm -rf /usr/share/nginx/html/*
COPY index.html /usr/share/nginx/html/index.html
EXPOSE 80
```

```bash
docker build -t shahbazz16/frontend-app .
docker run -d -p 8080:80 shahbazz16/frontend-app
docker push shahbazz16/frontend-app
```

---

### 3.2 Java Application

```dockerfile
FROM maven:3.9.6-eclipse-temurin-17 AS build
WORKDIR /app
COPY . .
RUN mvn clean package -DskipTests

FROM eclipse-temurin:17-jdk-alpine
WORKDIR /app
COPY --from=build /app/target/*.jar app.jar
EXPOSE 8080
CMD ["java", "-jar", "app.jar"]
```

```bash
docker build -t shahbazz16/java-app .
docker run -d -p 8081:8080 shahbazz16/java-app
docker push shahbazz16/java-app
```

---

### 3.3 Node.js Application

```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["npm", "start"]
```

```bash
docker build -t shahbazz16/node-app:v1 .
docker run -d -p 3000:3000 shahbazz16/node-app:v1
docker push shahbazz16/node-app:v1
```

---

## 4️⃣ Docker Compose – WordPress with MySQL

### docker-compose.yml

```yaml
version: "3.8"

services:
  mysql:
    image: mysql:5.7
    environment:
      MYSQL_ROOT_PASSWORD: root123
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wpuser
      MYSQL_PASSWORD: wp123
    volumes:
      - mysql_data:/var/lib/mysql

  wordpress:
    image: wordpress:latest
    ports:
      - "8080:80"
    environment:
      WORDPRESS_DB_HOST: mysql:3306
      WORDPRESS_DB_USER: wpuser
      WORDPRESS_DB_PASSWORD: wp123
      WORDPRESS_DB_NAME: wordpress
    depends_on:
      - mysql

volumes:
  mysql_data:
```

Run:

```bash
docker-compose up -d
```

Access:

```text
http://<EC2-IP>:8080
```

---

## ✅ Key Learnings

* Docker image lifecycle (build → tag → push)
* Docker Hub & Amazon ECR
* Infrastructure as Code with Terraform
* Jenkins CI/CD pipelines
* Multi-stage Docker builds
* Docker Compose for multi-container apps
* Real-world DevOps workflows

---

## 👨‍💻 Author

**Shaikh Shahbaz**
DevOps | Docker | AWS | Terraform | Jenkins

---

