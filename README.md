# Three-Tier AWS Architecture with Load Balancer

[![GitHub](https://img.shields.io/badge/GitHub-ShaikhAteeb02-181717?style=for-the-badge&logo=github)](https://github.com/ShaikhAteeb02)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-AteebShaikh-0077B5?style=for-the-badge&logo=linkedin)](https://www.linkedin.com/in/ateeb-ahmed-shaikh-932234192/)

---

## Features

- Full AWS three-tier (Web, App, DB) architecture in a custom VPC
- Distinct public, private, and database subnets for secure separation
- Application Load Balancer (ALB) for high-availability and traffic distribution
- NAT and Internet Gateways for controlled internet access
- Granular Security Groups for each tier restricting traffic appropriately
- Stepwise deployment instructions for beginners and professionals
- User-data scripts for automated EC2 instance initialization

---

## Architecture Overview

This project provisions a robust and scalable three-tier AWS architecture:

- **VPC:** Named `ThreeTierVPC` with three subnets:
    - **Public Subnet:** Hosts 2 web servers (EC2) with public IPs and HTTP/SSH access.
    - **App Subnet:** Contains application server (EC2) isolated from public internet, only exposed to the web tier.
    - **DB Subnet:** Houses the DB server (EC2 or RDS) locked down to only accept traffic from the application layer.
- **Internet Gateway** enables outbound internet access for web servers.
- **NAT Gateway** allows the app tier instances to securely reach the internet for updates.
- **Security Groups:**
    - `Web-SG`: Allows HTTP from anywhere and SSH from your IP.
    - `App-SG`: Allows application traffic only from `Web-SG`.
    - `DB-SG`: Allows DB access only from `App-SG`.
    - `ALB-SG`: Manages HTTP traffic to/from ALB.
- **Application Load Balancer:** Distributes HTTP traffic across web servers for high availability.
- **Automated Routing:** Public subnet routes public traffic via IGW; app subnet uses NAT for outbound; DB subnet has no internet route for security.
- **Resource tagging** for easy management and troubleshooting.

---

## Setup Instructions

### 1. VPC & Subnet Creation

- Login to AWS Console and create a VPC (`ThreeTierVPC`).
- Add three subnets in different Availability Zones:
    - `Public-Subnet` (e.g., 10.0.1.0/24)
    - `App-Subnet` (e.g., 10.0.2.0/24)
    - `DB-Subnet` (e.g., 10.0.3.0/24)

### 2. Gateways and Route Tables

- Create and attach an **Internet Gateway** (`ThreeTier-IGW`) to the VPC.
- Create and allocate an **Elastic IP** for the **NAT Gateway**.
- Launch the **NAT Gateway** in the `Public-Subnet` with the Elastic IP.
- Configure **Route Tables**:
    - Public RT: Routes 0.0.0.0/0 to IGW (associated with `Public-Subnet`).
    - App Private RT: Routes 0.0.0.0/0 to NAT Gateway (associated with `App-Subnet`).
    - DB Private RT: Local routes only (associated with `DB-Subnet`).

### 3. Security Groups

- `Web-SG`: Inbound HTTP (80) from 0.0.0.0/0, SSH (22) from your IP.
- `App-SG`: Inbound custom TCP (8080) from `Web-SG`.
- `DB-SG`: Inbound MySQL/Aurora (3306) from `App-SG`, outbound restricted to VPC CIDR.
- `ALB-SG`: Inbound HTTP (80) from 0.0.0.0/0.

### 4. Launch EC2 Instances

- **Web Tier:** 2 EC2 instances in `Public-Subnet` (Amazon Linux or Ubuntu), assign public IP, attach `Web-SG`. Use user-data to install/launch web server (`httpd`/`nginx`).
- **App Tier:** 1 EC2 instance in `App-Subnet`, no public IP, attach `App-SG`.
- **DB Tier:** 1 EC2 instance (or RDS) in `DB-Subnet`, no public IP, attach `DB-SG`.

### 5. Configure Application Load Balancer

- AWS EC2 > Load Balancers > Create Application Load Balancer.
- Scheme: Internet-facing, HTTP port 80.
- Network mapping: Select VPC and `Public-Subnet`.
- Assign `ALB-SG`.
- Create Target Group for web tier EC2 instances.
- Register the 2 web tier instances as targets.
- Complete ALB setup and note ALB DNS name.

---

## Usage

- **App Access:** Use your ALB DNS name to access the deployed application.
- **Test Load Balancing:** Refresh the ALB DNS endpoint — responses should rotate between web servers if index content differs.
- **Scaling:** Adjust EC2 instance counts in the web/app tiers to test architectural scalability.
- **Security Testing:** Attempt connections only permitted by security group rules to verify isolation between tiers.
- **Experiment:** Modify user-data scripts or routing tables for learning/enhancement.

---

For any queries or collaboration, connect via the badges above!

```
