# Secure Production-Grade AWS Architecture

This repository contains the deployment details and application code for a secure, highly available, and production-ready AWS cloud infrastructure. The design is based on AWS best practices for application security and network isolation.

## 🛠️ Architecture Overview

The architecture is built to ensure high availability and robust security by isolating application layers:

- **Custom VPC & Multi-AZ Deployment:** Deployed across two Availability Zones (AZs) to prevent a single point of failure.
- **Network Isolation (Public vs. Private Subnets):**
  - **Public Subnets:** Host the **Application Load Balancer (ALB)** and **NAT Gateways**. These are the only entry and exit points exposed to the internet.
  - **Private Subnets:** Host the actual **Application Servers (EC2 Instances)**. These instances have no public IP addresses and are completely shielded from direct internet access.
- **Secure Network Access:**
  - **NAT Gateway:** Allows private instances to securely download updates and connect to external APIs without exposing their private IP addresses.
  - **Bastion Host (Jump Server):** Configured in the public subnet to serve as a single, secure gateway for administrative SSH access to the private instances.
- **Compute & Auto Scaling:** An **Auto Scaling Group (ASG)** automatically manages the EC2 instances in the private subnets to handle traffic spikes and maintain reliability.

---

## 📂 Repository Structure

```text
├── app/
│   ├── index.html          # Simple HTML frontend webpage
│   └── start_server.sh     # Bash script to spin up the web server
└── README.md               # Project documentation and architecture details

---

## 🚀 How to Deploy This Architecture

### 1. Network Setup (VPC)
- Create a custom VPC with a CIDR block (e.g., `10.0.0.0/16`).
- Create 2 Public Subnets and 2 Private Subnets across 2 different Availability Zones.
- Attach an **Internet Gateway (IGW)** to the VPC and associate it with the Public Subnet route table.
- Deploy a **NAT Gateway** in a Public Subnet and route outbound internet traffic from the Private Subnets through it.

### 2. Compute & Bastion Configuration
- Launch a **Bastion Host** (t2.micro) in the Public Subnet with SSH (Port 22) allowed only from your personal IP address.
- Create a **Launch Template** configuring the Ubuntu instances for your application.
- Deploy an **Auto Scaling Group (ASG)** using the template to launch instances inside the Private Subnets.

### 3. Application Deployment
- Securely copy (`SCP`) your private key (`.pem` file) and application folder to the Bastion Host.
- SSH into the Bastion Host, and from there, SSH into the private EC2 instances.
- Run the `./start_server.sh` script inside the private instances to start the Python web server on Port `8000`.

### 4. Load Balancer Configuration
- Create an internet-facing **Application Load Balancer (ALB)** in the Public Subnets.
- Configure a **Target Group** pointing to the private EC2 instances on Port `8000`.
- Update Security Groups to ensure the private instances only accept HTTP traffic coming directly from the ALB.
- Access the secure application using the ALB's DNS name.

---

## 🔒 Security Best Practices Implemented
- **Least Privilege Access:** Security groups restrict SSH traffic strictly through the Bastion Host.
- **No Public IPs in Private Subnets:** Application servers are protected from automated internet scans and brute-force attacks.
- **Encrypted & Monitored Access:** Administrative traffic is centralized, allowing for simplified audit logging.
