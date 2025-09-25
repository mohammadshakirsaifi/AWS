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
