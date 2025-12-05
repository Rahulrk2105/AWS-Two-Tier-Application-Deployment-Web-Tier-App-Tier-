 AWS Two-Tier Application Deployment (Web Tier + App Tier )

This project is my implementation of a secure and scalable Two-Tier Architecture on AWS. I built everything manually using the AWS Management Console to strengthen my understanding of networking, compute, security, and architectural design.

The project consists of a public Web Tier running NGINX behind a Classic Load Balancer, and a private Application Tier running PHP/PHP-FPM. Both tiers communicate internally through private IPs inside a custom VPC.

My goal with this project was to design a clean, professional, production-style AWS architecture while staying fully within the AWS Free Tier.

🏗️ Architecture Diagram & Summary
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
                     
I created a full two-tier setup with isolated responsibilities:

1️⃣ Web Tier (Public Subnets)

Runs NGINX on EC2

Placed in public subnets

Exposed to the internet only through a Classic Load Balancer

Handles all incoming HTTP requests

Sends backend requests to the private App Tier

2️⃣ Application Tier (Private Subnets)

Runs PHP and PHP-FPM

Completely private (no public IP)

Accessible only from the Web Tier via Security Groups

Uses NAT Gateway for software updates

3️⃣ Networking

Custom VPC with CIDR 10.0.0.0/16

Two public subnets and two private subnets

Internet Gateway for public traffic

NAT Gateway for private outbound traffic

Separate route tables for public and private networks

4️⃣ Security

Web Security Group allows HTTP and SSH

App Security Group only allows traffic from Web SG

Tier isolation follows AWS best practices

This architecture design matches how enterprise environments separate frontend and backend workloads.

🌐 VPC and Networking Setup

I started by creating a new VPC:

VPC CIDR: 10.0.0.0/16

DNS Hostnames: Enabled

DNS Resolution: Enabled

Subnets I created:

Public Subnet 1 – 10.0.1.0/24

Public Subnet 2 – 10.0.2.0/24

Private Subnet 1 – 10.0.11.0/24

Private Subnet 2 – 10.0.12.0/24

Then I added:

Internet Gateway (attached to VPC)

NAT Gateway (created in Public Subnet 1)

Custom Route Tables for both public and private subnets

This gave me a clean separation between public-facing and internal resources.

🔐 Security Groups

I configured two main security groups:

Web-SG

HTTP (80) from anywhere

SSH (22) from my IP

Allowed to talk to App-SG

App-SG

Accepts only backend ports (8080 or 9000)

Source allowed only from Web-SG

No public access allowed

This ensures that only the Web Tier can communicate with the App Tier.

⚙️ Classic Load Balancer Setup

Since ALB was not available in my Free Tier account, I used a Classic Load Balancer (CLB) instead.

My CLB configuration:

Listener on HTTP port 80

Health checks on port 80

Placed in both public subnets

Uses Web-SG

Registered Web EC2 instance

The CLB distributes incoming traffic across my Web Tier and provides a single, stable public endpoint.

🖥️ Web Tier (NGINX EC2 – Public Subnet)

I launched my Web EC2 instance in the public subnet.
After connecting, I installed and configured NGINX:

sudo dnf update -y
sudo dnf install nginx -y
sudo systemctl enable nginx
sudo systemctl start nginx


I replaced the default index page with my own custom frontend:

Hello from Web EC2 (Frontend)
Backend Response:


I configured NGINX to forward backend requests to the App Tier using the private IP address of the application instance.

🧩 Application Tier (PHP-FPM EC2 – Private Subnet)

The App EC2 instance was launched in a private subnet with no public IP.
I installed PHP and PHP-FPM:

sudo dnf update -y
sudo dnf install php php-fpm -y
sudo systemctl enable php-fpm
sudo systemctl start php-fpm


I added a simple backend PHP script:

<?php
echo "Hello from APP EC2 Backend";
?>


Then I confirmed connectivity from the Web Tier:

curl http://<APP-Private-IP>:8080


Once I saw the expected output, I knew the private networking and SG rules were correct.

🧪 Application Testing
✔ Access through Classic Load Balancer

I opened the CLB DNS name in my browser and was able to reach the Web Tier frontend.

✔ Private backend test

Web EC2 successfully communicated with the App Tier over private IP.

✔ No public access to App Tier

This confirmed proper isolation.

📊 Monitoring

I monitored my EC2 instances using:

CloudWatch CPU alarms

Web and App server logs

Health checks from the load balancer

This helped me validate the system’s stability.

🧹 Cleanup

I removed all resources in correct order:

Classic Load Balancer

EC2 Instances

NAT Gateway

Internet Gateway

Subnets & Route Tables

Security Groups

VPC

This ensured no leftover charges.

🎯 What I Learned

By completing this project, I developed strong hands-on experience in:

Designing AWS Two-Tier architectures

Creating and configuring a custom VPC

Working with public/private subnets

Deploying Classic Load Balancers

Installing and configuring NGINX and PHP-FPM

Implementing NAT Gateway for private outbound access

Writing secure networking rules using Security Groups

Troubleshooting EC2, routing, and connectivity issues

Understanding application traffic flow inside AWS

This project helped me understand how real production environments are structured using AWS networking and compute services.
