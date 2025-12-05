# AWS Two-Tier Application Deployment (Web Tier + App Tier)

This project is my implementation of a secure and scalable Two-Tier Architecture on AWS. I built everything manually using the AWS Management Console to strengthen my understanding of networking, compute, and security.

---

## Step 1️⃣ Architecture Diagram & Summary

INTERNET  
   │  
   ▼  
AWS Internet Gateway  
   │  
   ▼  
VPC (10.0.0.0/16)  
   ├── Public Subnet-1 (10.0.1.0/24)  
   │     ├── EC2 Instance: my-web-ec2  
   │     │     └── Security Group: my-web-sg  
   │     └── NAT Gateway (Elastic IP)  
   │  
   └── Private Subnet-1 (10.0.11.0/24)  
         ├── EC2 Instance: my-app-ec2  
         │     └── Security Group: my-app-sg  
         │  
         └── API Call ───────────────► Python Backend (Private Subnet-2)  
            Private Subnet-2 (10.0.12.0/24)  
               ├── Python Backend (Port 8080)  
               │     └── SimpleHTTPServer  
               └── Security Group: app-sg  
                     └── Allows inbound on port 8030 from my-web-sg

---

## Step 2️⃣ Web Tier (Public Subnets)

- Runs **NGINX on EC2**
- Placed in public subnets
- Exposed to the internet only through a Classic Load Balancer
- Handles all incoming HTTP requests
- Sends backend requests to the private App Tier

---

## Step 3️⃣ Application Tier (Private Subnets)

- Runs **PHP and PHP-FPM**
- Completely private (no public IP)
- Accessible only from the Web Tier via Security Groups
- Uses NAT Gateway for software updates

---

## Step 4️⃣ Networking

- Custom VPC with CIDR 10.0.0.0/16
- Two public subnets and two private subnets
- Internet Gateway for public traffic
- NAT Gateway for private outbound traffic
- Separate route tables for public and private networks

---

## Step 5️⃣ Security

- Web Security Group allows HTTP and SSH
- App Security Group only allows traffic from Web SG
- Tier isolation follows AWS best practices

This architecture design matches how enterprise environments separate frontend and backend workloads.

---

## Step 6️⃣ VPC and Networking Setup

1. Created a new VPC:
   - VPC CIDR: 10.0.0.0/16
   - DNS Hostnames: Enabled
   - DNS Resolution: Enabled

2. Created Subnets:
   - Public Subnet 1 – 10.0.1.0/24
   - Public Subnet 2 – 10.0.2.0/24
   - Private Subnet 1 – 10.0.11.0/24
   - Private Subnet 2 – 10.0.12.0/24

3. Added Networking Resources:
   - Internet Gateway (attached to VPC)
   - NAT Gateway (created in Public Subnet 1)
   - Custom Route Tables for both public and private subnets

This gave me a clean separation between public-facing and internal resources.

---

## Step 7️⃣ Security Groups

Configured two main security groups:

- **Web-SG**
  - HTTP (80) from anywhere
  - SSH (22) from my IP
  - Allowed to talk to App-SG

- **App-SG**
  - Accepts only backend ports (8080 or 9000)
  - Source allowed only from Web-SG
  - No public access allowed

This ensures that only the Web Tier can communicate with the App Tier.

---

## Step 8️⃣ Classic Load Balancer Setup

Since ALB was not available in my Free Tier account, I used a Classic Load Balancer (CLB) instead.

- Listener on HTTP port 80
- Health checks on port 80
- Placed in both public subnets
- Uses Web-SG
- Registered Web EC2 instance

CLB distributes incoming traffic across my Web Tier and provides a single, stable public endpoint.

---

## Step 9️⃣ Web Tier (NGINX EC2 – Public Subnet)

I launched my Web EC2 instance in the public subnet.

After connecting, I installed and configured NGINX:
```sh
sudo dnf update -y
sudo dnf install nginx -y
sudo systemctl enable nginx
sudo systemctl start nginx
```

I replaced the default index page with my own custom frontend:
```
Hello from Web EC2 (Frontend)
Backend Response:
```

Configured NGINX to forward backend requests to the App Tier using the private IP address of the application instance.

---

## Step 🔟 Application Tier (PHP-FPM EC2 – Private Subnet)

The App EC2 instance was launched in a private subnet with no public IP.  
Installed PHP and PHP-FPM:
```sh
sudo dnf update -y
sudo dnf install php php-fpm -y
sudo systemctl enable php-fpm
sudo systemctl start php-fpm
```

Added a simple backend PHP script:
```php
<?php
echo "Hello from APP EC2 Backend";
?>
```

Confirmed connectivity from the Web Tier:
```sh
curl http://<APP-Private-IP>:8080
```

Once I saw the expected output, I knew the private networking and SG rules were correct.

---

## Step 11️⃣ Application Testing

- ✔ Access through Classic Load Balancer:  
  Opened the CLB DNS name in my browser and was able to reach the Web Tier frontend.

- ✔ Private backend test:  
  Web EC2 successfully communicated with the App Tier over private IP.

- ✔ No public access to App Tier:  
  This confirmed proper isolation.

---

## Step 12️⃣ Monitoring

I monitored my EC2 instances using:

- CloudWatch CPU alarms
- Web and App server logs
- Health checks from the load balancer

This helped me validate the system’s stability.

---

## Step 13️⃣ Cleanup

Removed all resources in required order:

1. Classic Load Balancer
2. EC2 Instances
3. NAT Gateway
4. Internet Gateway
5. Subnets & Route Tables
6. Security Groups
7. VPC

This ensured no leftover charges.

---

## Step 14️⃣ What I Learned

By completing this project, I developed strong hands-on experience in:

- Designing AWS Two-Tier architectures
- Creating and configuring a custom VPC
- Working with public/private subnets
- Deploying Classic Load Balancers
- Installing and configuring NGINX and PHP-FPM
- Implementing NAT Gateway for private outbound access
- Writing secure networking rules using Security Groups
- Troubleshooting EC2, routing, and connectivity issues
- Understanding application traffic flow inside AWS

This project helped me understand how real production environments are structured using AWS networking and compute services.

---
