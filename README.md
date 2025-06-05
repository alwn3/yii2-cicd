# 🚀 Yii2 CI/CD Pipeline with Docker, GitHub Actions & Ansible

This project demonstrates a complete CI/CD setup for a **Yii2 PHP application**, containerized with Docker, deployed with Ansible, and automated through GitHub Actions to an EC2 instance.

---

## 📦 Features

- 🐳 Dockerized PHP 8.2 + Apache + Composer environment
- 🔄 CI/CD with GitHub Actions — Build, Push, Deploy on every push to `main`
- 📡 Ansible playbook to provision and deploy app to Ubuntu-based EC2
- 🌐 Nginx reverse proxy for routing public traffic to Dockerized Yii2 app

---

## 🛠️ Local Setup Instructions

### 🔧 Prerequisites

- [Docker](https://www.docker.com/)
- [Docker Compose](https://docs.docker.com/compose/)
- [Ansible](https://www.ansible.com/) (optional, for remote deployment)
- [GitHub CLI](https://cli.github.com/) (to manage secrets easily)

### 🚀 Run Locally with Docker

git clone https://github.com/alwn3/yii2-cicd.git
cd yii2-cicd
docker-compose up --build -d

Now visit: http://localhost:8080

### 🐳 Dockerfile Overview
Path: docker/Dockerfile

This Dockerfile builds an Apache + PHP 8.2 image with required PHP extensions, installs Composer, sets document root to /web, and installs Yii2 dependencies.

Key steps:

PHP extensions: pdo, pdo_mysql, mbstring, intl, zip

Apache rewrite module enabled

Composer installed globally

Source copied from src/

Permissions fixed for Apache to serve content

To build manually:
docker build -t alwn3/yii2-app:latest -f docker/Dockerfile .

⚙️ GitHub Actions: CI/CD Pipeline
Path: .github/workflows/deploy.yml

### 🔁 Workflow Summary
Checkout source code

Build Docker image

Push image to Docker Hub

SSH into EC2 and:

Check if yii2_app service exists

Create or update the Docker Swarm service

### 🔐 Required GitHub Secrets
Secret Name	Description
DOCKERHUB_USERNAME	Your Docker Hub username
DOCKERHUB_TOKEN	Docker Hub access token
EC2_HOST	EC2 public IP or DNS
EC2_SSH_KEY	Private key contents for EC2 SSH

### 📦 Ansible Deployment
Path: ansible/playbook.yml

### 📋 What It Does
Installs Docker, Nginx, and Git

Initializes Docker Swarm

Deploys the Docker service yii2_app (if not present)

Configures Nginx with nginx/yii2.conf

Enables the site and restarts Nginx

### 🗂️ Inventory Example
Create ansible/inventory.yml:

all:
  hosts:
    yii2-server:
      ansible_host: <host ip>
      ansible_user: ubuntu        # or your server user (e.g., ec2-user, root)
      ansible_ssh_private_key_file: <path to key file>

The inventory.yml is added to .gitignore and wont be pushed to repo so the public IP wont be exposed

### ▶️ Run the Playbook

ansible-playbook -i ansible/inventory.yml ansible/playbook.yml

Ensure Nginx config file exists at nginx/yii2.conf

### 🌐 Nginx Reverse Proxy Configuration
Path: nginx/yii2.conf

This config allows Nginx to proxy traffic to your Dockerized Yii2 app:

server {
    listen 80;
    server_name 127.0.0.1;

    location / {
        proxy_pass http://127.0.0.1:8080; # Port your Yii2 container is mapped to
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }

    access_log /var/log/nginx/yii2_access.log;
    error_log /var/log/nginx/yii2_error.log;
}

You can update server_name and ports if needed.

### 🧠 Assumptions
EC2 instance is Ubuntu-based with SSH access

Docker image is public at alwn3/yii2-app

Yii2 app is located under src/ and uses /web as the document root

Docker Swarm is used (even for single-node deployments)

### ✅ Remotely (GitHub Actions)
Push to the main branch → GitHub Actions automatically:

Builds the Docker image

Pushes it to Docker Hub

SSHes into EC2 and deploys it
