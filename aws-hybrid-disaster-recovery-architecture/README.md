# 🌐 Hybrid Cloud Disaster Recovery Architecture on AWS

## 📌 Project Overview

This project demonstrates a **low-cost Disaster Recovery (DR) architecture** using AWS services. It ensures that application data is continuously backed up and can be restored quickly in case of a failure of the primary system.

The architecture simulates a real-world scenario where a production server fails, and a standby DR server takes over using backup data stored in Amazon S3.

---

## 🎯 Objective

* Design a **cost-effective disaster recovery solution**
* Ensure **data durability and availability**
* Minimize **downtime (RTO)** and **data loss (RPO)**
* Simulate **real-time failure and recovery**

---

## 🏗️ Architecture Components

* **Primary EC2 Instance** → Hosts application and generates data
* **S3 Bucket** → Stores backup data
* **DR EC2 Instance** → Restores and serves application during failure
* **VPC** → Provides network isolation and control
* **IAM Role** → Secure access between EC2 and S3
* **Cron Job** → Automates periodic backup

---

## ⚙️ Step-by-Step Implementation

### 🔹 Step 1: Create VPC

* Created a custom VPC for network isolation
* Defined CIDR block

---

### 🔹 Step 2: Create Subnets

* Created public subnets for EC2 instances
* Enabled internet access

---

### 🔹 Step 3: Attach Internet Gateway

* Attached IGW to VPC
* Enabled external connectivity

---

### 🔹 Step 4: Configure Route Table

* Added route:

```
0.0.0.0/0 → Internet Gateway
```

---

### 🔹 Step 5: Launch Primary EC2

* Created EC2 instance in public subnet
* Installed Apache web server

---

### 🔹 Step 6: Create Application Data

```
mkdir data
echo "important file from primary" > data/file1.txt
```

---

### 🔹 Step 7: Create S3 Bucket

* Created S3 bucket to store backups

---

### 🔹 Step 8: Attach IAM Role

* Attached role with S3 access permissions to EC2

---

### 🔹 Step 9: Sync Data to S3

```
aws s3 sync data/ s3://<your-bucket-name>/
```

---

### 🔹 Step 10: Automate Backup using Cron

```
*/5 * * * * /usr/bin/aws s3 sync /home/ec2-user/data/ s3://<your-bucket-name>/ >> /home/ec2-user/cron.log 2>&1
```

---

### 🔹 Step 11: Launch DR EC2

* Created second EC2 instance
* Acts as standby recovery server

---

### 🔹 Step 12: Restore Data on DR

```
aws s3 sync s3://<your-bucket-name>/ /home/ec2-user/data/
```

---

### 🔹 Step 13: Install Web Server on DR

```
sudo dnf install httpd -y
sudo systemctl start httpd
sudo systemctl enable httpd
```

---

### 🔹 Step 14: Display Recovered Data

```
echo "DR Server Active" | sudo tee /var/www/html/index.html
cat /home/ec2-user/data/file1.txt | sudo tee -a /var/www/html/index.html
```

---

### 🔹 Step 15: Test DR Setup

* Access DR EC2 via browser
* Verify recovered data is displayed

---

### 🔹 Step 16: Simulate Disaster

* Stop Primary EC2 instance
* Access DR EC2 → Application should still work

---

## 🧪 Testing & Validation

✔ Primary server serves application
✔ Data successfully synced to S3
✔ DR server restores data from S3
✔ Application runs on DR after primary failure

---

## 📸 Screenshots (Add these)



* EC2 Instances (Primary + DR)
* S3 Bucket with uploaded files
* `aws s3 sync` execution
* Cron job configuration
|                                              |
| -------------------------------------------- |
| ![cron](screenshots/cron.png)                |
| *Cron job configured for periodic execution* |

* DR data restoration
* Application running on Primary
* Application running on DR
* Disaster simulation (Primary stopped)

---

## 🧠 Key Concepts Demonstrated

### 🔹 Disaster Recovery (DR)

Ability to recover system after failure

### 🔹 RPO (Recovery Point Objective)

* Data synced every 5 minutes
* Max data loss: 5 minutes

### 🔹 RTO (Recovery Time Objective)

* Time taken to switch to DR system

---

## 💰 Cost Optimization

* Used **Free Tier EC2 instances (t2.micro)**
* Used **S3 for low-cost storage**
* Avoided expensive DR services
* Manual failover to reduce cost

---

## ⚠️ Limitations

* No automatic failover
* No load balancing
* Single region setup
* Manual disaster recovery

---

## 🚀 Future Improvements

* Add Route 53 for DNS failover
* Use Load Balancer for high availability
* Implement multi-region DR
* Automate failover

---

## 🏁 Conclusion

This project successfully demonstrates a **basic disaster recovery architecture** using AWS services. It ensures that data is continuously backed up and can be restored quickly, reducing downtime and data loss in case of system failure.

---

## 👨‍💻 Author

Built as part of AWS Cloud learning journey 🚀
