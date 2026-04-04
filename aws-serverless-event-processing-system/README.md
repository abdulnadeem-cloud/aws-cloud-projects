# 🚀 Project 2: Serverless Event Processing System

![AWS](https://img.shields.io/badge/AWS-Cloud-orange?logo=amazonaws)
![Lambda](https://img.shields.io/badge/AWS-Lambda-orange?logo=awslambda)
![S3](https://img.shields.io/badge/AWS-S3-green?logo=amazons3)
![DynamoDB](https://img.shields.io/badge/AWS-DynamoDB-blue?logo=amazondynamodb)
![SNS](https://img.shields.io/badge/AWS-SNS-red)
![CloudWatch](https://img.shields.io/badge/AWS-CloudWatch-purple)

---

## 🎯 Objective

Build a fully serverless, event-driven file processing system on AWS where:
- A user uploads a file to Amazon S3
- The upload automatically triggers AWS Lambda (no manual intervention)
- Lambda processes the file, stores metadata, and sends a notification
- All logs are recorded in CloudWatch
- Zero servers to manage — fully automated!

---

## 🧱 Architecture Overview

```
User uploads file
       |
  [Amazon S3]  ── uploads/ folder
       |
       | S3 Event Trigger (PUT)
       |
  [AWS Lambda] ── process-file-function
       |
       |── Copy file ──► S3 /processed/ folder
       |── Save record ► Amazon DynamoDB
       |── Send email ──► Amazon SNS ──► Email Notification
       |── Write logs ──► Amazon CloudWatch
```

---

## 🔧 AWS Services Used

| Service | Purpose |
|---|---|
| **Amazon S3** | File storage — upload trigger source |
| **AWS Lambda** | Serverless compute — runs processing logic |
| **Amazon DynamoDB** | NoSQL database — stores file metadata |
| **Amazon SNS** | Notification service — sends email alerts |
| **Amazon CloudWatch** | Monitoring — stores logs and metrics |
| **AWS IAM** | Security — manages permissions |

---

## 📌 Step-by-Step Implementation

### Step 1 — Create S3 Bucket
- Bucket name: `event-bucket-nadeem123`
- Region: `ap-south-1` (Mumbai)
- Folders created: `uploads/` and `processed/`
- Public access: Disabled (secure)

### Step 2 — Create IAM Role
- Role name: `lambda-event-role`
- Attached policies:
  - AmazonS3FullAccess
  - AmazonDynamoDBFullAccess
  - AmazonSNSFullAccess
  - CloudWatchLogsFullAccess

### Step 3 — Create DynamoDB Table
- Table name: `EventMetadata`
- Partition key: `file_id` (String)
- Capacity mode: On-demand (no idle cost)

### Step 4 — Create SNS Topic
- Topic name: `file-processing-topic`
- Type: Standard
- Subscription: Email (confirmed)

### Step 5 — Create Lambda Function
- Function name: `process-file-function`
- Runtime: Python 3.12
- IAM Role: `lambda-event-role`
- Timeout: 30 seconds
- Environment variable: `SNS_TOPIC_ARN`

### Step 6 — Configure S3 Event Trigger
- Event type: PUT
- Prefix: `uploads/`
- Destination: Lambda → `process-file-function`

---

## 💻 Lambda Function Code

```python
import json
import boto3
import uuid
from datetime import datetime
import os

s3 = boto3.client('s3')
dynamodb = boto3.resource('dynamodb')
sns = boto3.client('sns')

table = dynamodb.Table('EventMetadata')
SNS_TOPIC_ARN = os.environ['SNS_TOPIC_ARN']

def lambda_handler(event, context):
    try:
        for record in event['Records']:
            bucket = record['s3']['bucket']['name']
            key = record['s3']['object']['key']

            file_id = str(uuid.uuid4())
            processed_key = "processed/" + key.split('/')[-1]

            s3.copy_object(
                Bucket=bucket,
                CopySource={'Bucket': bucket, 'Key': key},
                Key=processed_key
            )

            table.put_item(
                Item={
                    'file_id': file_id,
                    'file_name': key,
                    'processed_file': processed_key,
                    'timestamp': str(datetime.now()),
                    'bucket': bucket
                }
            )

            message = "File " + key + " processed. Stored at: " + processed_key

            sns.publish(
                TopicArn=SNS_TOPIC_ARN,
                Subject="File Processed Successfully",
                Message=message
            )

            print("SUCCESS: Processed " + key)

        return {'statusCode': 200, 'body': 'success'}

    except Exception as e:
        print("ERROR: " + str(e))
        raise e
```

---

## 🧪 Testing

1. Upload any file to `uploads/` folder in S3
2. Verify results:

| Check | Location | Expected |
|---|---|---|
| File copied | S3 → `processed/` folder | File appears there |
| Metadata saved | DynamoDB → EventMetadata | New record inserted |
| Email received | Your inbox | AWS notification |
| Logs recorded | CloudWatch → Log groups | SUCCESS message |

---

## 📸 Screenshots

| # | Screenshot | Description |
|---|---|---|
| 1 | `IAM_Role.png` | IAM role with 4 policies |
| 2 | `DynamoDB_Table.png` | EventMetadata table active |
| 3 | `dynamodb_record.png` | Record inserted after upload |
| 4 | `Connect_S3_to_Lambda.png` | S3 event trigger configured |
| 5 | `Lambda_Code_with_SNS_ARN.png` | Environment variable set |
| 6 | `Lambda.png` | Lambda function code |
| 7 | `S3_Bucket.png` | S3 bucket with folders |
| 8 | `SNS_Topic.png` | SNS topic details |
| 9 | `test-file-uploaded.png` | File upload success |

---

## 🔐 Security Features

- IAM role with least privilege policies
- S3 public access blocked
- DynamoDB encrypted at rest (default)
- SNS topic restricted to confirmed subscribers
- Environment variables used (no hardcoded ARNs)

---

## 💰 Cost Optimization

| Service | Free Tier | Cost |
|---|---|---|
| Lambda | 1M requests/month | ~$0 |
| S3 | 5GB storage | ~$0 |
| DynamoDB | 25GB + 200M requests | ~$0 |
| SNS | 1M notifications | ~$0 |
| CloudWatch | 5GB logs | ~$0 |

**Total: ~$0 within AWS Free Tier** 🎉

---

## 📈 Scaling

- **Lambda** auto-scales with every event — no infrastructure management
- **S3** supports unlimited file uploads
- **DynamoDB** scales automatically on demand
- **Fully event-driven** — no bottlenecks, no idle servers

---

## 👨‍💻 Author

**Abdul Nadeem**
- AWS Serverless Event Processing Project
- Region: Asia Pacific (Mumbai) — ap-south-1
- Built with AWS Free Tier

---

## 📚 What I Learned

- How event-driven architecture works on AWS
- How S3 triggers Lambda automatically on file upload
- How to store metadata in DynamoDB using Lambda
- How to send notifications using SNS
- How to monitor functions using CloudWatch
- How to use IAM roles and environment variables securely
