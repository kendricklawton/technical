# Domain 1: Designing and Planning a Cloud Solution Architecture — Drills & Exercises

---

## Exercise 1: Provision a High-Availability Cloud SQL PostgreSQL Instance

```bash
# 1. Create a private VPC IP range for Private Services Access
gcloud compute addresses create google-managed-services-default   --global   --purpose=VPC_PEERING   --prefix-length=16   --network=default   --project=my-architect-project

# 2. Establish VPC peering connection
gcloud services vpc-peerings connect   --service=servicenetworking.googleapis.com   --ranges=google-managed-services-default   --network=default   --project=my-architect-project

# 3. Create Cloud SQL HA instance with automated backups and private IP
gcloud sql instances create prod-pg-ha   --database-version=POSTGRES_15   --tier=db-custom-4-16384   --region=us-central1   --availability-type=REGIONAL   --enable-bin-log   --backup-start-time=02:00   --network=default   --no-assign-ip   --project=my-architect-project
```

---

## Exercise 2: Design and Deploy an Auto-scaling Cloud Run Microservice

```bash
# Deploy a stateless container to Cloud Run with strict concurrency and scaling limits
gcloud run deploy orders-api   --image="us-docker.pkg.dev/cloudrun/container/hello"   --region="us-central1"   --platform="managed"   --min-instances=3   --max-instances=100   --concurrency=80   --cpu=2   --memory=2Gi   --allow-unauthenticated   --project=my-architect-project
```

---

## Exercise 3: Partitioning and Clustering in BigQuery

```sql
-- Create an optimized BigQuery table partitioned by day and clustered by customer_id and transaction_type
CREATE OR REPLACE TABLE `my-architect-project.analytics.transactions` (
  transaction_id STRING NOT NULL,
  customer_id STRING NOT NULL,
  amount NUMERIC(10, 2),
  transaction_type STRING,
  created_at TIMESTAMP
)
PARTITION BY DATE(created_at)
CLUSTER BY customer_id, transaction_type
OPTIONS (
  partition_expiration_days = 365,
  description = "Partitioned and clustered financial transactions table"
);
```
