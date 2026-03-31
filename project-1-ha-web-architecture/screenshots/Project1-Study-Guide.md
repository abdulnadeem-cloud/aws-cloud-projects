# 📘 Project 1 — High Availability Web Architecture
## Complete Study Guide | Objective + Theory + Interview Q&A

---

## 🎯 PROJECT OBJECTIVE

> Design and deploy a **production-grade, highly available web infrastructure** on AWS that can handle traffic automatically, recover from failures without human intervention, and scale based on demand — all while keeping the application secure and the database protected.

### In Simple Words:
Build a system where:
- ✅ Website never goes down even if one server crashes
- ✅ Traffic automatically distributed across multiple servers
- ✅ New servers launch automatically when traffic increases
- ✅ Database is protected inside private network
- ✅ No one can directly attack your web servers

---

## 🏗️ WHAT IS THIS PROJECT?

This project is a **3-tier web architecture** built on AWS:

```
Tier 1 → Presentation Layer  → Application Load Balancer (ALB)
Tier 2 → Application Layer   → EC2 instances (Auto Scaling Group)
Tier 3 → Database Layer      → RDS MySQL
```

### What Each Service Does:

| Service | What It Is | What It Does in This Project |
|---|---|---|
| **VPC** | Virtual Private Cloud | Your private network on AWS — isolates all resources |
| **Subnets** | Sub-divisions of VPC | Separates public and private resources |
| **IGW** | Internet Gateway | Main door connecting VPC to internet |
| **Route Tables** | Traffic rules | Decides where network traffic goes |
| **ALB** | Application Load Balancer | Receives user requests and distributes to EC2 |
| **EC2** | Elastic Compute Cloud | Virtual servers that run your web application |
| **ASG** | Auto Scaling Group | Automatically manages number of EC2 instances |
| **Launch Template** | EC2 Blueprint | Template for creating identical EC2 instances |
| **Bastion Host** | Jump Server | Secure way to access private EC2 instances |
| **NAT Gateway** | Network Address Translation | Allows private EC2 to access internet safely |
| **RDS** | Relational Database Service | Managed MySQL database |
| **Security Groups** | Virtual Firewall | Controls inbound and outbound traffic |
| **CloudWatch** | Monitoring Service | Monitors metrics and triggers alarms |

---

## ❓ WHY THIS PROJECT?

### Problem Without This Architecture:

```
Single EC2 → Single Point of Failure

If this EC2 crashes:
→ Website goes down ❌
→ Users can't access app ❌
→ Business loses money ❌
→ Manual intervention needed ❌
```

### Solution With This Architecture:

```
ALB + ASG + Multi-AZ EC2:

If one EC2 crashes:
→ ALB stops routing to it ✅
→ ASG launches replacement ✅
→ Website stays up ✅
→ Zero manual intervention ✅
→ Users never notice ✅
```

### Business Reasons:
- **High Availability** → 99.99% uptime
- **Scalability** → Handle traffic spikes automatically
- **Security** → Multiple layers of protection
- **Cost Efficiency** → Pay only for what you use
- **Disaster Recovery** → Survive AZ failures

---

## 📅 WHEN IS THIS ARCHITECTURE USED?

### Real World Use Cases:

| Company Type | Example | Why They Use This |
|---|---|---|
| E-commerce | Amazon, Flipkart | Handle traffic spikes during sales |
| Banking | HDFC, ICICI apps | Zero downtime requirement |
| Healthcare | Hospital portals | Critical availability needs |
| SaaS Products | Any web application | Scalability for growing users |
| Media | News websites | Handle viral traffic spikes |

### When To Choose This Architecture:

✅ When you expect **variable traffic** (more users at some times)
✅ When your app needs **high availability** (can't go down)
✅ When you need **automatic recovery** from failures
✅ When you need **security** for backend servers
✅ When you need **database** protection

### When NOT To Use This:
❌ Simple personal projects (overkill + cost)
❌ Internal tools with 5 users
❌ Static websites (use S3 + CloudFront instead)

---

## 🔄 HOW IT ALL WORKS TOGETHER — FLOW

### User Request Flow:
```
1. User opens browser → types your website URL
         ↓
2. DNS resolves to ALB DNS name
         ↓
3. ALB receives request on Port 80
         ↓
4. ALB checks which EC2 instances are healthy
         ↓
5. ALB forwards request to healthy EC2
         ↓
6. EC2 processes request → returns webpage
         ↓
7. User sees "Hello from ip-xxxx"
```

### Auto Scaling Flow:
```
1. Traffic increases → CPU goes above 50%
         ↓
2. CloudWatch detects high CPU
         ↓
3. ASG receives signal to scale out
         ↓
4. ASG launches new EC2 using Launch Template
         ↓
5. User data script runs → installs httpd
         ↓
6. New EC2 registered in Target Group
         ↓
7. ALB starts sending traffic to new EC2
         ↓
8. Traffic decreases → CPU drops
         ↓
9. ASG terminates extra instances (scale in)
```

### Failure Recovery Flow:
```
1. EC2 instance crashes suddenly
         ↓
2. ALB health check fails for that instance
         ↓
3. ALB stops routing traffic to crashed EC2
         ↓
4. ASG detects instance is unhealthy
         ↓
5. ASG terminates crashed instance
         ↓
6. ASG launches fresh replacement EC2
         ↓
7. New EC2 becomes healthy → ALB routes to it
         ↓
8. Zero downtime for users ✅
```

---

## 🔒 SECURITY DESIGN — EXPLAINED

```
Internet
   ↓
[ALB] ← Only accepts HTTP:80 from internet (alb-sg)
   ↓
[EC2] ← Only accepts traffic FROM alb-sg (ec2-sg)
         Only accepts SSH FROM bastion-sg (ec2-sg)
   ↓
[RDS] ← Only accepts MySQL:3306 FROM ec2-sg (rds-sg)
```

### Why This Matters:
- EC2 has **NO public IP** → Cannot be directly attacked
- RDS has **NO public access** → Cannot be directly accessed
- Only ALB faces the internet → Single controlled entry point
- Bastion is the only SSH entry → Controlled admin access

---

## 🌐 NETWORK DESIGN — EXPLAINED

### Why Public AND Private Subnets?

| Resource | Subnet Type | Why |
|---|---|---|
| ALB | Public | Needs internet access to receive user requests |
| Bastion | Public | Needs public IP for admin SSH access |
| EC2 | Private | No direct internet exposure = more secure |
| RDS | Private | Database should never be publicly accessible |
| NAT Gateway | Public | Needs internet to forward private EC2 traffic |

### Why 2 Availability Zones?
```
ap-south-1a (AZ1)          ap-south-1b (AZ2)
├── public-subnet-1         ├── public-subnet-2
├── private-subnet-1        ├── private-subnet-2
├── EC2 Instance 1          ├── EC2 Instance 2
└── ALB node                └── ALB node

If ap-south-1a data center BURNS DOWN:
→ ap-south-1b still running ✅
→ Website still accessible ✅
→ Zero downtime ✅
```

---

## 💼 INTERVIEW QUESTIONS & ANSWERS

---

### 🔷 VPC & NETWORKING

**Q1: What is a VPC and why did you create one?**
> A VPC (Virtual Private Cloud) is a logically isolated network within AWS where I can launch resources. I created it to have complete control over my network topology including IP addressing, subnets, route tables and security. Without a VPC, all resources would be on a shared public network which is insecure.

---

**Q2: Why did you use CIDR 10.0.0.0/16?**
> 10.0.0.0/16 gives me 65,536 IP addresses which is more than enough for this architecture. The /16 prefix gives flexibility to create multiple subnets. I chose the 10.x.x.x range because it is a private IP range that doesn't conflict with public internet addresses.

---

**Q3: What is the difference between public and private subnet?**
> A public subnet has a route to the Internet Gateway so resources can communicate with the internet directly. A private subnet has no route to the Internet Gateway so resources cannot be directly accessed from or reach the internet. In my project, ALB and Bastion are in public subnets while EC2 and RDS are in private subnets for security.

---

**Q4: Why did you put EC2 in private subnet?**
> Putting EC2 in a private subnet means it has no public IP and cannot be directly accessed from the internet. This significantly reduces the attack surface. All traffic must come through the ALB which acts as a controlled entry point. This is a security best practice in production environments.

---

**Q5: What is an Internet Gateway?**
> An Internet Gateway is a horizontally scaled, redundant VPC component that allows communication between resources in the VPC and the internet. It serves two purposes — providing a target in route tables for internet-routable traffic, and performing network address translation for instances with public IPs.

---

**Q6: Why do you need a NAT Gateway?**
> NAT Gateway allows private EC2 instances to initiate outbound connections to the internet (like downloading packages) while preventing the internet from initiating connections to those instances. Without NAT Gateway, my private EC2 instances couldn't download the httpd package during bootstrapping.

---

**Q7: What is the difference between NAT Gateway and Internet Gateway?**
> Internet Gateway allows two-way communication — both inbound and outbound. Resources with public IPs can receive traffic from internet. NAT Gateway only allows outbound communication — private instances can reach internet but internet cannot initiate connections to private instances. NAT Gateway provides internet access without exposing instances publicly.

---

**Q8: What are route tables and why did you create two?**
> Route tables contain rules that determine where network traffic is directed. I created two — a public route table with a route to the Internet Gateway for public subnets, and a private route table with a route to the NAT Gateway for private subnets. This ensures public resources can receive internet traffic while private resources remain protected.

---

### 🔷 SECURITY GROUPS

**Q9: What is a Security Group?**
> A Security Group acts as a virtual firewall for EC2 instances controlling inbound and outbound traffic. Unlike traditional firewalls, Security Groups are stateful — if you allow inbound traffic, the response is automatically allowed outbound. They operate at the instance level, not subnet level.

---

**Q10: Why did you create separate security groups for ALB, EC2 and RDS?**
> Separate security groups implement the principle of least privilege. ALB security group only allows internet HTTP traffic. EC2 security group only allows traffic from ALB security group and SSH from Bastion, not from the entire internet. RDS security group only allows MySQL traffic from EC2 security group. This creates layered security — even if one layer is compromised, others remain protected.

---

**Q11: What is the difference between Security Group and NACL?**
> Security Groups are stateful and operate at the instance level — response traffic is automatically allowed. NACLs (Network Access Control Lists) are stateless and operate at the subnet level — you must explicitly allow both inbound and outbound traffic. Security Groups only allow rules, NACLs can both allow and deny. I used Security Groups in this project as they are sufficient for instance-level control.

---

### 🔷 LOAD BALANCER

**Q12: What is an Application Load Balancer?**
> An Application Load Balancer operates at Layer 7 (application layer) of the OSI model. It distributes incoming HTTP/HTTPS traffic across multiple targets like EC2 instances based on rules. It supports path-based routing, host-based routing, and health checks to ensure traffic only goes to healthy instances.

---

**Q13: Why ALB and not Classic Load Balancer or NLB?**
> ALB is ideal for HTTP/HTTPS traffic with advanced routing capabilities. Classic Load Balancer is outdated. NLB (Network Load Balancer) operates at Layer 4 and is better for TCP/UDP traffic requiring ultra-high performance. Since my application is HTTP based and needs health check integration with ASG, ALB is the right choice.

---

**Q14: Why is the ALB in a public subnet?**
> ALB needs to receive traffic from the internet so it must be in a public subnet with internet access. However the EC2 instances behind the ALB are in private subnets. This way ALB acts as a secure front door — internet traffic hits ALB, ALB forwards to private EC2 instances.

---

**Q15: What is a Target Group?**
> A Target Group is a logical grouping of targets (EC2 instances) that the ALB routes requests to. It performs health checks on registered targets and only routes traffic to healthy ones. When integrated with ASG, new instances are automatically registered in the target group when they launch.

---

**Q16: What are Health Checks and why are they important?**
> Health checks are periodic requests the ALB sends to targets to verify they are functioning correctly. I configured HTTP health checks on path "/" every 30 seconds. If an instance fails 2 consecutive health checks it is marked unhealthy and ALB stops sending traffic to it. This ensures users only reach working instances.

---

### 🔷 AUTO SCALING

**Q17: What is Auto Scaling Group?**
> An Auto Scaling Group is a collection of EC2 instances treated as a logical unit for scaling and management. It automatically increases or decreases the number of instances based on demand, maintains desired capacity, and replaces unhealthy instances automatically. It is the backbone of high availability in AWS.

---

**Q18: What is a Launch Template?**
> A Launch Template is a reusable configuration for EC2 instances. It contains AMI ID, instance type, key pair, security groups and user data script. When ASG needs to launch a new instance it uses the launch template to ensure every instance is identical. This eliminates configuration drift between instances.

---

**Q19: What is User Data in EC2?**
> User Data is a script that runs automatically when an EC2 instance first launches. I used it to install and start the Apache web server (httpd) on every instance launched by the ASG. This ensures every new instance is immediately ready to serve web traffic without manual configuration.

---

**Q20: What scaling policies did you use?**
> I used Target Tracking scaling policy with CPU utilization as the metric and 50% as the target. This means ASG automatically adds instances when average CPU exceeds 50% and removes instances when CPU drops below 50%. It is the simplest and most effective policy for most web applications.

---

**Q21: What happens when an EC2 instance fails?**
> When an EC2 instance fails, the ALB health check detects it and stops routing traffic to it within seconds. Simultaneously the ASG detects the instance is unhealthy, terminates it, and automatically launches a replacement using the launch template. The entire recovery is automatic with zero human intervention, typically completing within 2-3 minutes.

---

**Q22: What is the difference between horizontal and vertical scaling?**
> Vertical scaling (scale up) means increasing the size of existing instances — like going from t2.micro to t2.large. It has a limit and requires downtime. Horizontal scaling (scale out) means adding more instances — which is what ASG does. It has virtually no limit and requires no downtime. My architecture uses horizontal scaling which is more resilient and cost-effective.

---

### 🔷 RDS

**Q23: Why RDS and not installing MySQL on EC2?**
> RDS is a managed service — AWS handles backups, patching, replication, failover and maintenance. Installing MySQL on EC2 would require me to manage all of this manually. RDS also provides Multi-AZ for automatic failover, automated backups, and easy scaling. For production, RDS is always preferred over self-managed MySQL.

---

**Q24: Why is RDS in a private subnet?**
> The database contains sensitive application data. Putting it in a private subnet means it has no public IP and cannot be accessed directly from the internet. Only EC2 instances with the correct security group can connect to RDS on port 3306. This protects data from unauthorized access.

---

**Q25: What is Multi-AZ RDS?**
> Multi-AZ RDS maintains a synchronous standby replica in a different Availability Zone. If the primary database fails, RDS automatically fails over to the standby replica, typically within 60-120 seconds with no data loss. The application doesn't need any changes — the DNS endpoint automatically points to the new primary. This provides high availability for the database tier.

---

### 🔷 BASTION HOST

**Q26: What is a Bastion Host and why did you use it?**
> A Bastion Host is a special EC2 instance in a public subnet that acts as a secure gateway for SSH access to private instances. Since my web servers are in private subnets with no public IP, I cannot SSH to them directly. The Bastion is the only entry point for administrative access, and it is locked down to only allow SSH from authorized IPs.

---

**Q27: Is Bastion Host the only way to access private instances?**
> No. Alternatives include AWS Systems Manager Session Manager which doesn't require SSH or open ports at all, AWS VPN which creates a private tunnel to the VPC, and AWS Direct Connect for physical dedicated connections. Session Manager is actually more secure than Bastion because it requires no open SSH ports. For this project I used Bastion as it is the most commonly understood pattern.

---

### 🔷 HIGH AVAILABILITY & ARCHITECTURE

**Q28: How does your architecture achieve high availability?**
> My architecture achieves high availability through multiple mechanisms: Multi-AZ deployment across ap-south-1a and ap-south-1b ensures regional fault tolerance. ALB distributes traffic and detects failures. ASG automatically replaces failed instances and maintains desired capacity. RDS Multi-AZ provides database failover. No single component failure can bring down the entire system.

---

**Q29: What is the difference between High Availability and Fault Tolerance?**
> High Availability means the system remains accessible with minimal downtime, typically achieved through redundancy and quick recovery. Fault Tolerance means the system continues operating even when components fail with zero downtime or data loss. My architecture is highly available — if an EC2 fails, there is brief detection time before recovery. True fault tolerance would require active-active configurations with no recovery period.

---

**Q30: How would you scale this from 100 to 1 million users?**

> **100 users:**
> Current architecture handles this — 1-2 EC2 instances, single ALB

> **10,000 users:**
> ASG scales EC2 to 4-10 instances, ALB distributes traffic

> **100,000 users:**
> Add CloudFront CDN to cache static content globally
> Add ElastiCache (Redis) for session and database caching
> Add RDS Read Replicas to distribute read queries

> **1,000,000 users:**
> Multi-region deployment with Route53 latency routing
> Database sharding or migration to Aurora Global Database
> SQS queues for async processing
> Microservices architecture to scale components independently

---

**Q31: What would you improve in this architecture?**
> Several improvements for production:
> 1. Add HTTPS using ACM certificates on ALB
> 2. Use AWS WAF to protect against web attacks
> 3. Add CloudFront CDN for global content delivery
> 4. Replace Bastion with AWS Session Manager for keyless access
> 5. Add RDS Read Replicas for read-heavy workloads
> 6. Implement ElastiCache for database query caching
> 7. Add VPC Flow Logs for network monitoring
> 8. Use AWS Secrets Manager for database credentials

---

**Q32: What is the cost of this architecture per month?**
> For a minimal setup:
> - EC2 t2.micro x2 → Free tier (750 hrs/month)
> - ALB → ~$18/month
> - NAT Gateway → ~$33/month
> - RDS db.t3.micro → Free tier
> - CloudWatch → Free tier
> Total estimated cost → ~$50/month for minimal production setup
> For a demo project I deleted resources after completion to avoid charges.

---

## 📝 KEY TERMS TO REMEMBER

| Term | One Line Definition |
|---|---|
| VPC | Your private network on AWS |
| Subnet | Subdivision of VPC (public = internet access, private = no internet) |
| IGW | Door connecting VPC to internet |
| NAT Gateway | Lets private instances access internet without being exposed |
| Route Table | Traffic rules for network |
| Security Group | Firewall for EC2 instances |
| ALB | Distributes traffic across multiple EC2 instances |
| Target Group | List of EC2s that ALB sends traffic to |
| Health Check | ALB's way of verifying EC2 is working |
| ASG | Automatically manages number of EC2 instances |
| Launch Template | Blueprint for creating EC2 instances |
| User Data | Script that runs when EC2 first starts |
| Bastion Host | Jump server for secure SSH to private instances |
| RDS | Managed database service |
| Multi-AZ | Running in multiple data centers for high availability |
| CloudWatch | Monitoring and alerting service |
| High Availability | System stays up even when components fail |
| Horizontal Scaling | Adding more instances to handle load |
| Vertical Scaling | Making existing instances bigger |

---

## 🏆 HOW TO EXPLAIN PROJECT IN INTERVIEW

> "I designed and built a highly available web architecture on AWS following production best practices. The architecture uses a VPC with public and private subnets across two availability zones for network isolation and fault tolerance. An Application Load Balancer in the public subnet distributes traffic to EC2 web servers in private subnets managed by an Auto Scaling Group. The ASG uses a Launch Template with user data to automatically bootstrap instances, and scales based on CPU utilization. A NAT Gateway provides secure outbound internet access for private instances. The database layer uses RDS MySQL in a private subnet with restricted security group access. All components are monitored via CloudWatch alarms. This architecture eliminates single points of failure and can automatically recover from instance or availability zone failures without human intervention."

---

*Study this thoroughly — you will clear any cloud interview with confidence! 🚀*
