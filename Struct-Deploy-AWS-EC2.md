# Deploying Applications on AWS EC2: Practical & Scalable Guide

This guide covers how to deploy any application (web app, API, microservices, etc.) on AWS EC2 the *right* way — production-ready, scalable, and maintainable.

---

## ✅ What You'll Learn

- How to deploy your app to EC2 correctly  
- Useful tools like Nginx, PM2, Docker, GitHub Actions  
- How to build production-ready infrastructure  
- Optional add-ons: SSL, domain setup, auto-scaling, backups  

---

## 🔧 1. Launch an EC2 Instance (Ubuntu)

1. Go to **AWS EC2 Dashboard → Launch Instance**  
2. Choose **Ubuntu 22.04 LTS**  
3. Select instance type:  
   - `t3.micro` for development/testing  
   - `m5.large` or `c6i.large` for production  
4. Create or use an existing **key pair**  
5. Configure Security Group to allow:  
   - Port **22** (SSH)  
   - Port **80** (HTTP)  
   - Port **443** (HTTPS)  
6. Enable **Auto-assign Public IP** for SSH and HTTP/S access  

---

## ⚙️ 2. Connect and Prepare the Server

SSH into your instance:

```bash
ssh -i my-key.pem ubuntu@your-ec2-ip
```
Update packages:
```bash
sudo apt update && sudo apt upgrade -y
```
Install essentials:
```bash
sudo apt install git nginx curl ufw -y
```
🚀 3. Deploy Your App (Choose One)
🅰️ Manual Deployment (Good for Beginners)

Example: Node.js + Express app
```bash
# Install Node.js
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install -y nodejs

# Clone your project
git clone https://github.com/your-user/your-repo.git
cd your-repo

# Install dependencies
npm install

# Run with PM2 (auto-restart on crash)
sudo npm install -g pm2
pm2 start index.js
pm2 startup
pm2 save
```
Good For: Simple apps or learning environment

🅱️ Docker-Based Deployment (Recommended for Production)
```bash
# Install Docker
sudo apt install docker.io -y
sudo systemctl start docker
sudo systemctl enable docker

# Optional: Docker Compose
sudo apt install docker-compose -y

# Clone your project
git clone https://github.com/your-user/your-repo.git
cd your-repo

# Build and run container
docker build -t my-app .
docker run -d -p 3000:3000 my-app

```
Good For: Scalable, repeatable, team-friendly deployments

🆎 CI/CD Deployment with GitHub Actions

Automate deployment on push to main branch:
```bash
# .github/workflows/deploy.yml
name: Deploy to EC2

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: SSH and Deploy
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ubuntu
          key: ${{ secrets.EC2_SSH_KEY }}
          script: |
            cd /home/ubuntu/your-repo
            git pull
            npm install
            pm2 restart all

```
Good For: Production-ready, zero-downtime deployments

🌐 4. Set Up NGINX as a Reverse Proxy

Edit NGINX config:

sudo nano /etc/nginx/sites-available/default


Replace with:
```bash
server {
    listen 80;
    server_name your-domain.com;

    location / {
        proxy_pass http://localhost:3000;  # Change port if needed
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}

```
Restart NGINX:
```bash
sudo systemctl restart nginx
```
🔒 5. Set Up SSL (Free HTTPS via Let's Encrypt)

Install Certbot and generate SSL certificate:
```bash
sudo apt install certbot python3-certbot-nginx -y
sudo certbot --nginx -d your-domain.com
```

Set up auto-renewal:
```bash
sudo crontab -e
# Add the following line:
0 0 * * * /usr/bin/certbot renew --quiet
```
🔁 6. Optional: Auto Scaling & Load Balancing

For high availability and scalability:

Use Elastic Load Balancer (ELB) to distribute traffic across multiple EC2 instances

Use Auto Scaling Group (ASG) to automatically add/remove EC2 instances based on load

Store secrets/config in AWS SSM Parameter Store or Secrets Manager

Use EFS or S3 for shared file storage across instances

📦 7. Optional: Backup and Monitoring

Regularly create EC2 AMI snapshots for backups

Enable CloudWatch for metrics and logs

Set up alarms for CPU, memory, downtime, etc.

✅ Deployment Flow Summary
[ GitHub Repo ]
       ↓ (CI/CD push)
[ EC2 Instance ] ← NGINX ← User Requests
       ↑         ↑
    PM2 / Docker | SSL via Certbot
       ↑
[ App Code ]

