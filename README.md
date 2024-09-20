# Travel Memory Application Deployment

## Overview

This repository contains the code and instructions for deploying the Travel Memory application. The application consists of a **Node.js** backend and a **React** frontend, deployed on two EC2 instances, with load balancing and reverse proxy setup using **NGINX**. Additionally, **Cloudflare** is used to manage DNS and security.

---

## Table of Contents
1. [Architecture Overview](#architecture-overview)
2. [Prerequisites](#prerequisites)
3. [Backend Deployment](#backend-deployment)
4. [Frontend Deployment](#frontend-deployment)
5. [NGINX Configuration](#nginx-configuration)
6. [Security Group Setup](#security-group-setup)
7. [Load Balancer and Target Groups Setup](#load-balancer-and-target-groups-setup)
8. [Cloudflare DNS Setup](#cloudflare-dns-setup)
9. [Accessing the Application](#accessing-the-application)

---

## Architecture Overview

The Travel Memory application architecture includes:
- Two EC2 instances: one for the **frontend** and one for the **backend**.
- **NGINX** reverse proxy to route traffic from port 80 to the respective application.
- **Load Balancers (LB)** for both frontend and backend to handle traffic.
- **Cloudflare** to manage DNS and SSL for the domain.

---

## Prerequisites

Before you begin, ensure you have the following:
- AWS account with permissions to create EC2 instances, security groups, AMI images, and load balancers.
- A purchased domain name configured in **Cloudflare**.
- Access to **MongoDB Atlas** or a similar database service for the backend.
- Basic knowledge of **NGINX**, **Node.js**, and **React**.

---

## Backend Deployment

1. **Create EC2 Instance**:
   - **Instance Public IP**: `54.212.227.53`
   - **VPC**: `vpc-0321f38a7b594180d`
   - **Subnet ID**: `subnet-0f30c30418def6379`

2. **Install Required Packages both on frontend and backend ec2 instance**:
   ```bash
   sudo apt update
   sudo apt install nodejs -y
   sudo apt install npm -y
   sudo apt install nginx -y
   ```

3. **Clone the Repository**:
   ```bash
   sudo su
   cd /
   git clone https://github.com/UnpredictablePrashant/TravelMemory
   ```

4. **Setup Environment Variables**:
   ```bash
   cd /TravelMemory/backend
   touch .env
   ```
   Add the following to the `.env` file:
   ```
   MONGO_URI='mongodb+srv://<user>:<password>@<cluster>.mongodb.net/travelmemory'
   PORT=3001
   ```

5. **Install Dependencies & Run the Backend**:
   ```bash
   npm install
   node index.js &
   ```

---

## Frontend Deployment

1. **Create EC2 Instance**:
   - **Instance Public IP**: `54.184.128.108`
   - **VPC**: `vpc-0321f38a7b594180d`
   - **Subnet ID**: `subnet-0f30c30418def6379`

2. **Install Required Packages**:
   ```bash
   sudo apt update
   sudo apt install nodejs -y
   sudo apt install npm -y
   sudo apt install nginx -y
   ```

3. **Clone the Repository**:
   ```bash
   sudo su
   cd /
   git clone https://github.com/UnpredictablePrashant/TravelMemory
   ```

4. **Update Backend API URL**:
   - Navigate to the frontend directory:
   ```bash
   cd /TravelMemory/frontend/src
   vim url.js
   ```
   - Replace the URL with your backend's public IP:
   ```js
   const backendURL = 'http://<your-backend-ec2-public-ip>:3001';
   ```

5. **Install Dependencies & Run the Frontend**:
   ```bash
   cd /TravelMemory/frontend
   npm install
   npm start &
   ```

---

## NGINX Configuration

1. **Backend NGINX Configuration**:
   - Unlink the default configuration:
   ```bash
   sudo unlink /etc/nginx/sites-enabled/default
   ```
   - Create a custom configuration:
   ```bash
   cd /etc/nginx/sites-available/
   sudo vim custom_server.conf
   ```
   - Add the following configuration for the backend:
   ```nginx
   server {
       listen 80;
       location / {
           proxy_pass http://<backend-ec2-public-ip>:3001;
           proxy_set_header Host $host;
           proxy_set_header X-Real-IP $remote_addr;
           proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
           proxy_set_header X-Forwarded-Proto $scheme;
       }
   }
   ```
   - Restart NGINX:
   ```bash
   sudo systemctl restart nginx
   ```

2. **Frontend NGINX Configuration**:
   - Follow the same steps for the frontend instance, updating the `proxy_pass` directive to point to the frontend application.
   - Unlink the default configuration:
   ```bash
   sudo unlink /etc/nginx/sites-enabled/default
   ```
   - Create a custom configuration:
   ```bash
   cd /etc/nginx/sites-available/
   sudo vim custom_server.conf
   ```
   - Add the following configuration for the backend:
   ```nginx
   server {
       listen 80;
       location / {
           proxy_pass http://<backend-ec2-public-ip>:3001;
           proxy_set_header Host $host;
           proxy_set_header X-Real-IP $remote_addr;
           proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
           proxy_set_header X-Forwarded-Proto $scheme;
       }
   }
   ```
   - Restart NGINX:
   ```bash
   sudo systemctl restart nginx
   ```
---

## Security Group Setup

Create a security group with the following rules:
- **Name**: `launch-wizard-63`
- **Ingress Ports**:
   - SSH (port 22)
   - HTTP (port 80)
   - Backend (port 3001)
   - Frontend (port 3001)

---

## Load Balancer and Target Groups Setup

1. **Create Target Groups**:
   - **Backend Target Group**: `tm-app-backend-tg`
   - **Frontend Target Group**: `tm-app-frontend-tg`

2. **Create Load Balancers**:
   - **Backend Load Balancer**: `tm-app-backend-lb`
   - **Frontend Load Balancer**: `tm-app-frontend-lb`

---

## Cloudflare DNS Setup

1. Login to **Cloudflare** and go to the DNS settings.
2. Add **CNAME** records for both the frontend and backend load balancers.
3. Use the nameservers provided by Cloudflare to update your domain’s nameservers.

---

## Accessing the Application

Once the deployment is complete, you can access the application using your domain name. For example:
- **Frontend**: `http://<your-domain>`
- **Backend**: `http://<your-domain>/api`

---

## License

This project is licensed under the MIT License.

---

This **README.md** file provides a step-by-step guide for deploying the Travel Memory application, from configuring EC2 instances to setting up Cloudflare for domain management. Make sure to replace the placeholder values (e.g., IP addresses, MongoDB credentials, etc.) with your actual configuration details.
