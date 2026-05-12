# 🏗️ AWS End-to-End Data Pipeline: RDS → Redshift

A cloud-native data lakehouse pipeline built on AWS that replicates production data from Amazon RDS into Amazon Redshift for analytics and business intelligence, following a **Bronze → Silver → Gold** architecture.

![Architecture Diagram](architecture.png)

---

## 📐 Architecture Overview

```
Amazon RDS (Source)
     │
     ▼
AWS DMS (Replication Instance)
     │  Source Endpoint → Replication Task → Target Endpoint
     ▼
Amazon S3 — Bronze Layer (Raw)
     │
     ▼  AWS Glue (ETL)
     │
     ▼
Amazon S3 — Silver Layer (Cleaned)
     │
     ▼  AWS Glue (Transform)
     │
     ▼
Amazon Redshift (Staging → Target via Stored Procedure)
     │
     ▼
Amazon QuickSight (BI & Dashboards)
```

---

## 🧩 Services Used

| Service | Role |
|---|---|
| **Amazon RDS** | Source relational database |
| **AWS DMS** | Database Migration Service – replicates data continuously |
| **Amazon S3 (Bronze)** | Raw data landing zone |
| **AWS Glue** | ETL jobs for data transformation |
| **Amazon S3 (Silver)** | Cleaned and structured data |
| **AWS Glue Crawler** | Catalogs data into the AWS Glue Data Catalog |
| **AWS Glue Data Catalog** | Centralized metadata repository |
| **Amazon Redshift** | Cloud data warehouse for analytics |
| **Amazon QuickSight** | Business intelligence and dashboarding |
| **AWS Step Functions** | Orchestrates the ETL workflow |
| **Amazon EventBridge** | Triggers the pipeline on schedule or events |
| **AWS Lambda** | Lightweight compute for automation |
| **AWS Secrets Manager** | Securely stores DB credentials |
| **AWS STS** | Manages temporary IAM credentials |
| **Amazon VPC** | Network isolation for RDS and Redshift |

---

## 🔄 Pipeline Flow

1. **Ingestion** – AWS DMS continuously replicates data from Amazon RDS to S3 (Bronze layer) inside a VPC.
2. **Orchestration** – Amazon EventBridge triggers an AWS Step Functions workflow on a schedule.
3. **Crawl & Catalog** – AWS Glue Crawler scans the Bronze S3 data and updates the Glue Data Catalog.
4. **Transform (Bronze → Silver)** – AWS Glue ETL job cleans and structures the raw data, writing it to the Silver S3 bucket.
5. **Load to Redshift** – A second Glue job loads Silver data into a Redshift staging table via a Network Interface inside the VPC.
6. **Stored Procedure** – A Redshift stored procedure promotes data from staging to the target production table.
7. **Visualize** – Amazon QuickSight connects to Redshift to serve dashboards and reports.

---

## 🔐 Security

- All data traffic within a **VPC** with private subnets and VPC Endpoints
- Credentials managed by **AWS Secrets Manager**
- IAM roles with least-privilege access via **AWS STS**
- Redshift accessed only through a **Network Interface** (no public exposure)

---

## 📁 Repo Structure

```
├── glue/
│   ├── bronze_to_silver.py       # Glue ETL job: raw → cleaned
│   └── silver_to_redshift.py     # Glue ETL job: load to Redshift
├── step_functions/
│   └── pipeline_definition.json  # Step Functions state machine
├── lambda/
│   └── trigger_handler.py        # Lambda for custom triggers
├── redshift/
│   └── stored_procedure.sql      # Staging → target promotion logic
├── dms/
│   └── task_config.json          # DMS replication task settings
├── architecture.png              # Architecture diagram
└── README.md
```

---

## 🚀 Getting Started

### Prerequisites
- AWS account with appropriate IAM permissions
- Python 3.8+
- AWS CLI configured

### Deployment Steps

1. **Set up VPC** – Create subnets, route tables, and VPC endpoints for S3 and Glue.
2. **Configure RDS** – Enable binary logging (for MySQL) or logical replication (for PostgreSQL).
3. **Create DMS** – Set up source/target endpoints and a replication task.
4. **Deploy Glue Jobs** – Upload scripts from `glue/` and configure job parameters.
5. **Create Step Functions** – Deploy the state machine from `step_functions/pipeline_definition.json`.
6. **Set up EventBridge** – Create a scheduled rule to trigger the Step Functions workflow.
7. **Configure Redshift** – Run the stored procedure from `redshift/stored_procedure.sql`.
8. **Connect QuickSight** – Add Redshift as a data source in QuickSight.

---

## 📊 Key Design Decisions

- **Medallion Architecture (Bronze/Silver/Gold)** – Keeps raw data intact while enabling iterative refinement.
- **DMS for CDC** – Change Data Capture ensures near-real-time replication with minimal source load.
- **Step Functions for orchestration** – Visual workflow with built-in retry and error handling.
- **Redshift Stored Procedure** – Atomic promotion from staging to production prevents partial loads.

---

## 🛑 Cost Management

> ⚠️ When not in use, stop or delete the following resources to avoid charges:
> - DMS Replication Instance
> - Redshift Cluster (pause or delete)
> - Glue Jobs (no charge when idle, but remove crawlers if not needed)
> - QuickSight subscription

---

## 📄 License

MIT License — free to use and modify.
