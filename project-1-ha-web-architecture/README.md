<div align="center">

# 🏗️ High Availability Web Architecture on AWS

### A production-grade, highly available web infrastructure built on AWS
#### Designed for Scalability · Security · Fault Tolerance

<br/>

![AWS](https://img.shields.io/badge/AWS-Cloud-FF9900?style=for-the-badge&logo=amazon-aws&logoColor=white)
![EC2](https://img.shields.io/badge/EC2-t2.micro-232F3E?style=for-the-badge&logo=amazon-aws&logoColor=white)
![RDS](https://img.shields.io/badge/RDS-MySQL-4479A1?style=for-the-badge&logo=mysql&logoColor=white)
![VPC](https://img.shields.io/badge/VPC-Networking-8C4FFF?style=for-the-badge&logo=amazon-aws&logoColor=white)
![ALB](https://img.shields.io/badge/ALB-Load%20Balancer-FF9900?style=for-the-badge&logo=amazon-aws&logoColor=white)
![Status](https://img.shields.io/badge/Status-Completed-2ea44f?style=for-the-badge)
![Region](https://img.shields.io/badge/Region-ap--south--1-blue?style=for-the-badge&logo=amazon-aws&logoColor=white)

<br/>

---

### 🚀 Built With

| Service | Purpose | Type |
|:---:|:---:|:---:|
| ![](https://img.shields.io/badge/VPC-Network%20Isolation-8C4FFF?style=flat-square&logo=amazon-aws&logoColor=white) | Private Network | Networking |
| ![](https://img.shields.io/badge/EC2-Web%20Servers-FF9900?style=flat-square&logo=amazon-aws&logoColor=white) | Application Layer | Compute |
| ![](https://img.shields.io/badge/ALB-Traffic%20Distribution-FF9900?style=flat-square&logo=amazon-aws&logoColor=white) | Load Balancing | Networking |
| ![](https://img.shields.io/badge/ASG-Auto%20Scaling-FF9900?style=flat-square&logo=amazon-aws&logoColor=white) | High Availability | Compute |
| ![](https://img.shields.io/badge/RDS-MySQL%20Database-4479A1?style=flat-square&logo=mysql&logoColor=white) | Data Layer | Database |
| ![](https://img.shields.io/badge/CloudWatch-Monitoring-FF4F8B?style=flat-square&logo=amazon-aws&logoColor=white) | Alerts & Metrics | Monitoring |

</div>

---

## 📐 Architecture Overview
```
┌─────────────────────────────────────────────────────────────┐
│                        🌐  Internet                         │
└──────────────────────────────┬──────────────────────────────┘
                               │
┌──────────────────────────────▼──────────────────────────────┐
│                ⚖️  Application Load Balancer                │
│                  (Internet-facing · HTTP:80)                 │
│            public-subnet-1    │    public-subnet-2           │
└───────────────┬───────────────────────────────┬─────────────┘
                │                               │
┌───────────────▼─────────────┐   ┌─────────────▼────────────┐
│      🖥️  EC2 Instance 1     │   │     🖥️  EC2 Instance 2   │
│        (ap-south-1a)        │   │       (ap-south-1b)       │
│       private-subnet-1      │   │      private-subnet-2     │
└───────────────┬─────────────┘   └─────────────┬────────────┘
                │                               │
                └───────────────┬───────────────┘
                                │
┌───────────────────────────────▼─────────────────────────────┐
│                    📈  Auto Scaling Group                   │
│               Min : 1  │  Desired : 1  │  Max : 2           │
└──────────────────────────────┬──────────────────────────────┘
                               │
┌──────────────────────────────▼──────────────────────────────┐
│                       🔀  NAT Gateway                       │
│                      (public-subnet-1)                      │
└──────────────────────────────┬──────────────────────────────┘
                               │
┌──────────────────────────────▼──────────────────────────────┐
│                  🗄️  RDS MySQL  (Single-AZ)                 │
│                    private subnet · ha-db                   │
└─────────────────────────────────────────────────────────────┘
```


---
## 🚀 Services Used

| Service | Purpose |
|---|---|
| **VPC** | Isolated network with public & private subnets |
| **EC2 (t2.micro)** | Web servers in private subnets |
| **Application Load Balancer** | Distributes traffic across EC2 instances |
| **Auto Scaling Group** | Automatically scales EC2 based on CPU load |
| **NAT Gateway** | Allows private EC2 to access internet securely |
| **RDS MySQL** | Managed relational database in private subnet |
| **CloudWatch** | Monitoring and CPU alarm (>70% threshold) |
| **Security Groups** | Firewall rules for each layer |
| **Bastion Host** | Secure SSH access to private instances |

---

## 🔒 Security Design

| Layer | Security Measure |
|---|---|
| ALB | Open to internet (HTTP:80) |
| EC2 | Only accessible from ALB security group |
| EC2 SSH | Only accessible from Bastion security group |
| RDS | Only accessible from EC2 security group (port 3306) |
| Private Subnets | No direct internet access |

---

## 🌐 Network Design

### VPC CIDR: `10.0.0.0/16`

| Subnet | Type | AZ | CIDR |
|---|---|---|---|
| public-subnet-1 | Public | ap-south-1a | 10.0.1.0/24 |
| public-subnet-2 | Public | ap-south-1b | 10.0.2.0/24 |
| private-subnet-1 | Private | ap-south-1a | 10.0.3.0/24 |
| private-subnet-2 | Private | ap-south-1b | 10.0.4.0/24 |

---

## ⚙️ Auto Scaling Configuration

```
Desired Capacity : 2
Minimum          : 1
Maximum          : 2
Scaling Policy   : Target Tracking (CPU > 50%)
Health Check     : ELB Health Check
```

---

## 📦 EC2 User Data Script

Every EC2 instance launched by ASG automatically runs this script:

```bash
#!/bin/bash
dnf update -y
dnf install httpd -y
systemctl start httpd
systemctl enable httpd
echo "Hello from $(hostname)" > /var/www/html/index.html
```

---

## 📸 Screenshots

<table>
  <tr>
    <td align="center">
      <b>🌐 VPC Created</b><br/><br/>
      <img src="screenshots/vpc.png" width="90%" style="border: 3px solid #2563EB; border-radius: 8px;"/>
    </td>
  </tr>
  <tr>
    <td align="center">
      <b>🌐 VPC Overview</b><br/><br/>
      <img src="screenshots/vpc_full.png" width="90%" style="border: 3px solid #2563EB; border-radius: 8px;"/>
    </td>
  </tr>
  <tr>
    <td align="center">
      <b>🔀 Subnets (Public + Private across 2 AZs)</b><br/><br/>
      <img src="screenshots/subnets.png" width="90%" style="border: 3px solid #2563EB; border-radius: 8px;"/>
    </td>
  </tr>
  <tr>
    <td align="center">
      <b>🌍 Internet Gateway</b><br/><br/>
      <img src="screenshots/igw.png" width="90%" style="border: 3px solid #2563EB; border-radius: 8px;"/>
    </td>
  </tr>
  <tr>
    <td align="center">
      <b>🗺️ Route Tables Public</b><br/><br/>
      <img src="screenshots/route-tables_public.png" width="90%" style="border: 3px solid #2563EB; border-radius: 8px;"/>
    </td>
  </tr>
  <tr>
    <td align="center">
      <b>🗺️ Route Tables Private</b><br/><br/>
      <img src="screenshots/route-tables_private.png" width="90%" style="border: 3px solid #2563EB; border-radius: 8px;"/>
    </td>
  </tr>
  <tr>
    <td align="center">
      <b>🔒 Security Groups for ALB</b><br/><br/>
      <img src="screenshots/security-groups_alb.png" width="90%" style="border: 3px solid #2563EB; border-radius: 8px;"/>
    </td>
  </tr>
  <tr>
    <td align="center">
      <b>🔒 Security Groups for Baston Host EC2</b><br/><br/>
      <img src="screenshots/security-groups_baston.png" width="90%" style="border: 3px solid #2563EB; border-radius: 8px;"/>
    </td>
  </tr>
  <tr>
    <td align="center">
      <b>🔒 Security Groups for Private EC2</b><br/><br/>
      <img src="screenshots/security-groups_privateEC2.png" width="90%" style="border: 3px solid #2563EB; border-radius: 8px;"/>
    </td>
  </tr>
  <tr>
    <td align="center">
      <b>🖥️ Bastion Host</b><br/><br/>
      <img src="screenshots/bastion-host_ec2.png" width="90%" style="border: 3px solid #2563EB; border-radius: 8px;"/>
    </td>
  </tr>
  <tr>
    <td align="center">
      <b>🔐 Private EC2 (No Public IP)</b><br/><br/>
      <img src="screenshots/ec2-private.png" width="90%" style="border: 3px solid #2563EB; border-radius: 8px;"/>
    </td>
  </tr>
  <tr>
    <td align="center">
      <b>🎯 Target Group</b><br/><br/>
      <img src="screenshots/target-group.png" width="90%" style="border: 3px solid #2563EB; border-radius: 8px;"/>
    </td>
  </tr>
  <tr>
    <td align="center">
      <b>⚖️ Application Load Balancer</b><br/><br/>
      <img src="screenshots/alb.png" width="90%" style="border: 3px solid #2563EB; border-radius: 8px;"/>
    </td>
  </tr>
  <tr>
    <td align="center">
      <b>📋 Launch Template</b><br/><br/>
      <img src="screenshots/launch-template.png" width="90%" style="border: 3px solid #2563EB; border-radius: 8px;"/>
    </td>
  </tr>
  <tr>
    <td align="center">
      <b>📈 Auto Scaling Group (2 instances across AZs)</b><br/><br/>
      <img src="screenshots/asg.png" width="90%" style="border: 3px solid #2563EB; border-radius: 8px;"/>
    </td>
  </tr>
   <tr>
    <td align="center">
      <b>🗄️ NAT Gateway</b><br/><br/>
      <img src="screenshots/NatGatway.png" width="90%" style="border: 3px solid #2563EB; border-radius: 8px;"/>
    </td>
  </tr>
  <tr>
    <td align="center">
      <b>🗄️ RDS Database</b><br/><br/>
      <img src="screenshots/RDS.png" width="90%" style="border: 3px solid #2563EB; border-radius: 8px;"/>
    </td>
  </tr>
  <tr>
    <td align="center">
      <b>📊 CloudWatch Alarm</b><br/><br/>
      <img src="screenshots/cloudwatch_alarm_a.png" width="90%" style="border: 3px solid #2563EB; border-radius: 8px;"/>
    </td>
  </tr>
  <tr>
    <td align="center">
      <b>🌐 Outputs & Execution Results </b><br/><br/>
      <img src="screenshots/alb-output.png" width="90%" style="border: 3px solid #2563EB; border-radius: 8px;"/>
    </td>
  </tr>
  <tr>
    <td align="center">
      <b>🔄 Outputs & Execution Results </b><br/><br/>
      <img src="screenshots/load-balancing.png" width="90%" style="border: 3px solid #2563EB; border-radius: 8px;"/>
    </td>
  </tr>
</table>
---

## 🔥 How This Handles Scale

| Traffic Level | How System Responds |
|---|---|
| **100 users** | 1-2 EC2 instances handle load via ALB |
| **10,000 users** | ASG scales up to 2+ instances automatically |
| **100,000 users** | Add CloudFront CDN + ElastiCache |
| **1,000,000 users** | Multi-region + RDS read replicas + SQS queues |

---

## 💡 Key Design Decisions

**Why private subnet for EC2?**
> Prevents direct internet exposure. Only ALB can send traffic to EC2, reducing attack surface.

**Why 2 Availability Zones?**
> If one AWS data center goes down, the other AZ continues serving traffic — zero downtime.

**Why Bastion Host?**
> Secure, controlled SSH access to private instances without exposing them to the internet.

**Why NAT Gateway?**
> Allows private EC2 instances to download packages and updates without being publicly accessible.

**Why Auto Scaling Group?**
> Automatically replaces failed instances and scales capacity based on real traffic demand.

---


## 👨‍💻 Author

**Abdul Nadeem**
- AWS Solutions Architect Associate in Training
- Building production-grade cloud architectures
- Region: Asia Pacific (Mumbai) — ap-south-1

---

## 📚 What I Learned

- Designing **multi-layer security** with security groups
- Building **highly available** systems across multiple AZs
- Configuring **auto scaling** with launch templates and user data
- Setting up **load balancing** with health checks
- Managing **private networking** with NAT Gateway and route tables
- Monitoring infrastructure with **CloudWatch alarms**

---

> ⭐ If you found this helpful, please star the repository!
