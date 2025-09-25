how to structure deployment on AWS EC2. Here's a practical and scalable guide for deploying any application (web app, API, microservices, etc.) using EC2.

✅ TL;DR: What You’ll Learn

How to deploy your app to EC2 the right way

Tools like Nginx, PM2, Docker, GitHub Actions, etc.

How to set up production-ready infrastructure

Optional add-ons: SSL, domain, auto-scaling, backups

🔧 1. Launch an EC2 Instance (Ubuntu)

Go to AWS EC2 Dashboard → "Launch Instance"

Choose Ubuntu 22.04 LTS

Pick instance type: t3.micro for dev, m5.large or c6i.large for prod

Create or use a key pair

Configure Security Group:

Open ports: 22 (SSH), 80 (HTTP), 443 (HTTPS)

✅ Tip: Enable auto-assign public IP for SSH and traffic.

⚙️ 2. Connect and Prepare the Server

SSH into your instance:

ssh -i my-key.pem ubuntu@your-ec2-ip


Then update the server:

sudo apt update && sudo apt upgrade -y


Install essentials:

sudo apt install git nginx curl ufw -y

🚀 3. Deploy Your App (3 Common Ways)

Choose one based on your preference:

🅰️ A. Manual Deployment (Good for Beginners)

Example: Node.js + Express app

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


✅ Good For: Simple apps or learning environment

🅱️ B. Docker-Based Deployment (Recommended for Prod)

Steps:

# Install Docker
sudo apt install docker.io -y
sudo systemctl start docker
sudo systemctl enable docker

# Optional: Docker Compose
sudo apt install docker-compose -y

# Clone your project
git clone https://github.com/your-user/your-repo.git
cd your-repo

# Build and run
docker build -t my-app .
docker run -d -p 3000:3000 my-app


✅ Good For: Scalable, repeatable, team-friendly deployments

🆎 C. CI/CD Deployment with GitHub Actions

Set up GitHub Actions to SSH and deploy automatically when you push code.

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


✅ Good For: Production-ready, zero-downtime deployments

🌐 4. Set Up NGINX as a Reverse Proxy

Route incoming traffic from port 80 to your app.

sudo nano /etc/nginx/sites-available/default


Replace contents with:

server {
    listen 80;
    server_name your-domain.com;

    location / {
        proxy_pass http://localhost:3000;  # or your app port
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}


Then restart NGINX:

sudo systemctl restart nginx

🔒 5. Set Up SSL (Free HTTPS via Let's Encrypt)

Install Certbot:

sudo apt install certbot python3-certbot-nginx -y
sudo certbot --nginx -d your-domain.com


Auto-renew:

sudo crontab -e
# Add:
0 0 * * * /usr/bin/certbot renew --quiet

🔁 6. Optional: Auto Scaling + Load Balancing

For high availability and scalability:

Use Elastic Load Balancer (ELB) to distribute traffic

Use Auto Scaling Group (ASG) to add/remove EC2s

Store app config and secrets in SSM Parameter Store or Secrets Manager

Use EFS or S3 for shared file storage

📦 7. Optional: Backup and Monitoring

Snapshots: Use EC2 AMI snapshots regularly

Monitoring: Enable CloudWatch for metrics/logs

Alerts: Set up alarms for high CPU, memory, downtime, etc.

✅ Deployment Flow Summary
[ GitHub Repo ]
       ↓ (CI/CD push)
[ EC2 Instance ] ← NGINX ← User Requests
       ↑         ↑
    PM2 / Docker | SSL via Certbot
       ↑
[ App Code ]

💬 Want Specific Help?