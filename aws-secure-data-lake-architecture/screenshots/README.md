<div align="center">

# 🔐 Secure Data Lake Analytics Platform

### Serverless · Encrypted · SQL-Queryable · Fully Audited

[![AWS](https://img.shields.io/badge/AWS-Cloud-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white)](https://aws.amazon.com)
[![S3](https://img.shields.io/badge/Amazon_S3-Data_Lake-569A31?style=for-the-badge&logo=amazons3&logoColor=white)](https://aws.amazon.com/s3/)
[![Athena](https://img.shields.io/badge/Amazon_Athena-Query_Engine-232F3E?style=for-the-badge&logo=amazonaws&logoColor=white)](https://aws.amazon.com/athena/)
[![Glue](https://img.shields.io/badge/AWS_Glue-ETL_Catalog-8C4FFF?style=for-the-badge&logo=amazonaws&logoColor=white)](https://aws.amazon.com/glue/)
[![Status](https://img.shields.io/badge/Status-Completed-2EA44F?style=for-the-badge)](.)
[![Cost](https://img.shields.io/badge/Cost-Under_₹10-blue?style=for-the-badge)](.)

</div>

---

## 📌 What I Built

A **production-style, serverless data lake** on AWS that ingests raw CSV data, automatically discovers schema, enables SQL querying directly on S3 — with end-to-end security, audit logging, and zero infrastructure to manage.

> **Core Idea:** Store raw data in S3 → let Glue discover its structure → query it with SQL using Athena. No database server, no ETL jobs, no manual schema setup. Just data in, insights out.

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    SECURE DATA LAKE — AWS                        │
│                                                                  │
│   CSV Upload                                                     │
│       │                                                          │
│       ▼                                                          │
│  ┌─────────┐   /raw/        ┌──────────────┐                    │
│  │  Amazon │──────────────▶│  AWS Glue    │                     │
│  │   S3    │   /processed/  │  Crawler     │                     │
│  │         │   /logs/       └──────┬───────┘                    │
│  └─────────┘                       │ schema discovery            │
│       │                            ▼                             │
│       │                   ┌──────────────────┐                  │
│       │                   │  Glue Data        │                  │
│       │                   │  Catalog          │                  │
│       │                   └──────┬────────────┘                 │
│       │                          │ metadata                      │
│       │                          ▼                               │
│       │                   ┌──────────────┐                      │
│       └──────────────────▶│   Amazon     │                      │
│            direct scan     │   Athena     │  SQL Analytics        │
│                            └──────────────┘                      │
│                                                                  │
│  🔒 Security Layer: IAM + Encryption + Bucket Policy             │
│  📋 Audit Layer:    CloudTrail + CloudWatch                      │
└─────────────────────────────────────────────────────────────────┘
```

---

## ⚙️ AWS Services Used

| Service | Role | Why This Service |
|---|---|---|
| **Amazon S3** | Data Lake Storage | Durable, cheap, serverless object storage. Acts as the raw data landing zone. |
| **AWS Glue Crawler** | Schema Discovery | Auto-scans CSV files, detects column names & types, writes to Data Catalog. |
| **Glue Data Catalog** | Metadata Layer | Central registry of table definitions. Athena reads this to understand data structure. |
| **Amazon Athena** | SQL Query Engine | Runs standard SQL directly on S3 files. Pay per query — no server needed. |
| **AWS IAM** | Access Control | Least-privilege roles — Glue gets only ReadOnly + GlueServiceRole. |
| **CloudTrail** | Audit Logging | Records every API call (who did what, when, from where) across all regions. |
| **Amazon CloudWatch** | Monitoring | Crawler execution logs, real-time log inspection, operational visibility. |

---

## 🔐 Security Implementation

| Control | Implementation | Why It Matters |
|---|---|---|
| **Block Public Access** | All 4 settings enabled (On) | Prevents any internet access — even if a permissive policy is accidentally added |
| **SSE-S3 Encryption** | Default encryption on all objects | Data encrypted at rest automatically, free, no key management needed |
| **IAM Least Privilege** | `GlueDataLakeRole` — S3ReadOnly + GlueServiceRole only | Service gets minimum permissions needed — no admin, no write access |
| **Bucket Policy** | Restricts access to `GlueDataLakeRole` ARN on `/raw/*` only | Even if IAM is misconfigured, bucket enforces its own access rules |
| **CloudTrail** | Multi-region trail, logs to S3 `/logs/` | Immutable audit record of every action — supports incident investigation |

---

## 📂 Project Structure

```
project-3-secure-data-lake/
│
├── README.md
├── policies/
│   └── bucket-policy.json          ← Fine-grained IAM bucket policy
│
├── sample-data/
│   └── sales_data.csv              ← Sample dataset used for testing
│
└── screenshots/
    ├── s3-setup.png                ← S3 bucket + folder structure
    ├── s3-raw-upload.png           ← sales_data.csv uploaded to /raw/
    ├── s3-block-public.png         ← Block all public access — ON
    ├── s3-encryption.png           ← SSE-S3 encryption enabled
    ├── iam-role.png                ← GlueDataLakeRole with both policies
    ├── cloudtrail.png              ← data-lake-trail active, multi-region
    ├── glue-crawler.png            ← Crawler config: S3 source + IAM role
    ├── glue-table.png              ← Auto-generated schema (5 columns)
    ├── athena-query1.png           ← SELECT * — all 8 rows returned
    ├── athena-query2.png           ← GROUP BY region — aggregation results
    ├── athena-query_Failure.png    ← Troubleshooting: TABLE_NOT_FOUND
    ├── bucket-policy.png           ← Fine-grained policy applied
    └── cloudwatch-logs.png         ← Glue crawler operational logs
```

---

## 🚀 Implementation Steps

### Step 1 — S3 Data Lake Setup

Created S3 bucket `secure-data-lake-nadeem` (ap-south-1) with zone architecture:

```
/raw/        ← landing zone for untouched input data
/processed/  ← reserved for cleaned/transformed data
/logs/       ← CloudTrail audit logs
```

| | |
|---|---|
| ![S3 Bucket Setup](screenshots/s3-setup.png) | ![CSV Uploaded](screenshots/s3-raw-upload.png) |
| *Bucket with 3-zone folder structure* | *sales_data.csv (280 B) in /raw/* |

---

### Step 2 — Security Configuration

Enabled both controls before uploading any data:

| | |
|---|---|
| ![Block Public Access](screenshots/s3-block-public.png) | ![Encryption](screenshots/s3-encryption.png) |
| *Block all public access — On* | *SSE-S3 default encryption enabled* |

---

### Step 3 — IAM Role (Least Privilege)

Created `GlueDataLakeRole` with exactly two policies — no more, no less:

![IAM Role](screenshots/iam-role.png)
*GlueDataLakeRole — AmazonS3ReadOnlyAccess + AWSGlueServiceRole*

> **Bonus:** In Step 8, this was further tightened with a bucket policy restricting access to only the `/raw/*` prefix on this specific bucket.

---

### Step 4 — CloudTrail Audit Logging

Multi-region trail logging all API calls into S3 `/logs/` folder:

![CloudTrail](screenshots/cloudtrail.png)
*data-lake-trail — Active logging, multi-region, stored in S3*

---

### Step 5 — AWS Glue Crawler

Crawler scanned `/raw/` and automatically detected the CSV schema — no manual column mapping needed:

| | |
|---|---|
| ![Glue Crawler](screenshots/glue-crawler.png) | ![Glue Table](screenshots/glue-table.png) |
| *Crawler config — S3 source + GlueDataLakeRole* | *Auto-generated table schema (5 columns)* |

**Schema detected:**

| Column | Type | Notes |
|---|---|---|
| `id` | bigint | Row identifier |
| `product` | string | Product name |
| `amount` | bigint | Sale amount |
| `region` | string | Sales region |
| `date` | string | Transaction date |

---

### Step 6 — Athena SQL Analytics

Queried S3 data directly using SQL — no database server, no data loading:

**Query 1 — Basic data exploration:**
```sql
SELECT * FROM raw LIMIT 10;
```

**Query 2 — Regional sales aggregation:**
```sql
SELECT region, SUM(amount) AS total_sales
FROM raw
GROUP BY region
ORDER BY total_sales DESC;
```

| | |
|---|---|
| ![Athena Query 1](screenshots/athena-query1.png) | ![Athena Query 2](screenshots/athena-query2.png) |
| *All 8 rows returned — 0.27 KB scanned* | *Sales by region: North leads at 1280* |

**Results:**

| Region | Total Sales |
|---|---|
| North | 1,280 |
| West | 390 |
| East | 275 |
| South | 85 |

---

### Step 7 — Bucket Policy (Fine-Grained IAM)

Replaced broad S3ReadOnlyAccess with a resource-specific policy restricting access to only the `/raw/` prefix:

```json
{
  "Sid": "AllowGlueRole",
  "Effect": "Allow",
  "Principal": {
    "AWS": "arn:aws:iam::843448676270:role/GlueDataLakeRole"
  },
  "Action": ["s3:GetObject", "s3:ListBucket"],
  "Resource": [
    "arn:aws:s3:::secure-data-lake-nadeem",
    "arn:aws:s3:::secure-data-lake-nadeem/raw/*"
  ]
}
```

![Bucket Policy](screenshots/bucket-policy.png)
*Policy applied successfully — Glue can only access /raw/, nothing else*

---

### Step 8 — CloudWatch Monitoring

Glue crawler execution logs captured in CloudWatch — showing each crawl phase in real time:

![CloudWatch Logs](screenshots/cloudwatch-logs.png)
*Crawler logs: schema classification → table creation → catalog write → READY*

---

## 🧪 Troubleshooting — Real Problem Encountered

During Athena testing, the first query failed:

```sql
SELECT * FROM non_existing_table;
-- ERROR: TABLE_NOT_FOUND
```

![Failure Screenshot](screenshots/athena-query_Failure.png)

**Root cause:** Glue Crawler names the table after the **folder** (`raw`), not the CSV filename (`sales_data`). The fix was checking Glue → Tables to get the exact table name, then using `FROM raw` in the query.

> This is a common real-world mistake — the table name in the Glue Catalog is always the S3 prefix name, not the file name.

---

## 💰 Cost Analysis

| Service | Usage | Cost |
|---|---|---|
| Amazon S3 | 280 B stored, ~20 requests | Free tier — ₹0 |
| AWS Glue Crawler | 1 run, ~1 minute | < ₹0.05 |
| Amazon Athena | 2 queries, 0.27 KB each | < ₹0.001 |
| CloudTrail | 1 trail (first trail free) | ₹0 |
| CloudWatch | Log ingestion | Free tier — ₹0 |
| IAM | Roles and policies | Always free |
| **Total** | | **< ₹1** |

---

## 🚀 Future Enhancements

- [ ] **Data Partitioning** — Store as `/raw/year=2024/month=01/` to reduce Athena scan cost by up to 99%
- [ ] **CSV → Parquet conversion** — Parquet is columnar and compressed; Athena queries are 10x cheaper
- [ ] **Glue ETL Job** — Automate raw → processed transformation pipeline
- [ ] **Lambda trigger** — Auto-run crawler when new files arrive in `/raw/`
- [ ] **QuickSight dashboard** — Connect Athena to visualize sales trends without code

---

## 📚 Key Learnings

- **Schema-on-read** architecture: data is stored raw; schema is applied only at query time — more flexible than loading into a rigid database
- **Glue table naming**: the table name comes from the S3 folder prefix, not the file name — a common real-world gotcha
- **IAM least privilege in practice**: replacing a broad managed policy with a resource-scoped bucket policy is what production security actually looks like
- **Serverless cost model**: this entire analytics pipeline ran for under ₹1 — no servers provisioned, no idle compute charged

---

<div align="center">

---

**Built with ☁️ on AWS | Region: ap-south-1 (Mumbai)**

**Nadeem** · Cloud Enthusiast · AWS Learner

</div>
