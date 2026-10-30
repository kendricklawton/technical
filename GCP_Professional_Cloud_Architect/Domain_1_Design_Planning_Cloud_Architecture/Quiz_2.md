# Domain 1: Designing and Planning a Cloud Solution Architecture — Quiz 2

**Exam:** Google Cloud Certified: Professional Cloud Architect (PCA)  
**Format:** 10 Scenario-Based Multiple-Choice Questions with In-Depth Technical Explanations

---

### Question 1
A startup wants to ingest 50,000 user activity clickstream events per second. The events must be enriched with user demographic data stored in Cloud SQL and written to both BigQuery for business intelligence and Memorystore Redis for real-time recommendation caching. What is the most cost-effective and scalable architecture?
- A. Write events directly from mobile apps to Cloud SQL.
- B. Ingest clickstream into Cloud Pub/Sub, stream process and enrich via Cloud Dataflow, and write out to BigQuery and Memorystore.
- C. Save clickstream events to local disk on Compute Engine VMs and run cron jobs.
- D. Send events via email to an SRE.

---

### Question 2
An enterprise operates a high-performance in-memory caching layer that requires sub-millisecond response times for session state across 50 microservices running on GKE. What managed service should the architect recommend?
- A. Cloud Bigtable
- B. Cloud Memorystore for Redis
- C. Cloud Storage Nearline
- D. Firestore

---

### Question 3
A document archiving platform needs to store 10 Petabytes of historical tax audit records. The records are accessed fewer than once every three years, but when requested, must be retrievable within a few minutes. Low storage cost is the primary governing constraint. Which Cloud Storage class meets these requirements?
- A. Standard
- B. Nearline
- C. Coldline
- D. Archive

---

### Question 4
An architect is designing the database schema for a massive IoT fleet in Cloud Bigtable. The device readings are queried based on `device_id` and `timestamp`. Which row key design avoids the "hotspotting" antipattern in Bigtable?
- A. `YYYYMMDD#HHMMSS` (timestamp at the beginning)
- B. `device_id#timestamp` (or hashed `device_id#timestamp`)
- C. Monotonically increasing auto-incrementing integer (1, 2, 3...)
- D. Fixed static string `"reading"` for all rows

---

### Question 5
A team is migrating an existing Kubernetes application to Google Cloud. The team wants Google to manage the node infrastructure, autoscaling, security hardening, and OS updates, while only paying for the CPU, memory, and storage requested by the active pods. What GKE operational mode should be selected?
- A. GKE Standard with manual node pools
- B. GKE Autopilot
- C. Compute Engine with kubeadm
- D. Cloud Run functions

---

### Question 6
An architect needs to select a compute platform for an asynchronous batch processing job that converts user-uploaded video files into multiple bitrates. The processing runs for up to 6 hours per video. What is the most cost-effective compute choice?
- A. Cloud Run (with 60-minute maximum timeout)
- B. Compute Engine Managed Instance Group utilizing Spot VMs with automated job checkpointing and Cloud Tasks / Pub/Sub queuing
- C. App Engine Standard
- D. Cloud Functions

---

### Question 7
A company needs to implement a Disaster Recovery strategy for an on-premises Oracle database migrating to Google Cloud. The business requires an RPO of 15 minutes and an RTO of 2 hours. What architecture satisfies these requirements while minimizing idle cloud infrastructure costs?
- A. Active-active multi-region Cloud Spanner
- B. Warm Standby: Replicate database changes continuously via Database Migration Service or GoldenGate to an idle or small Cloud SQL / Compute Engine instance, scaling up compute capacity upon failover
- C. Cold backup: Taking weekly tape backups
- D. Multi-cloud active replication

---

### Question 8
Which Google Cloud serverless technology allows developers to deploy code written in Python, Node.js, Go, or Java directly from source code without writing a Dockerfile, automatically packaging it via Google Cloud Buildpacks?
- A. Compute Engine
- B. Cloud Run / Cloud Functions
- C. GKE Standard
- D. Google Cloud VMware Engine

---

### Question 9
A healthcare company needs to ensure that patient records uploaded to Cloud Storage cannot be deleted, overwritten, or modified by any user, including the root project owner, for a mandatory legal retention period of 10 years. Which feature provides this guarantee?
- A. Object Versioning
- B. Cloud Storage Bucket Lock (Locked Retention Policy)
- C. Cloud KMS CMEK
- D. IAM deny policy

---

### Question 10
An architect is designing an API layer for a legacy core banking system hosted on-premises. The API must support developer portal onboarding, API key management, rate limiting, and SOAP-to-REST transformation. Which Google Cloud platform is specifically built for enterprise API management?
- A. Cloud Endpoints
- B. Apigee API Management
- C. Cloud NAT
- D. Cloud Load Balancing alone

---

## Answer Key & Rationales

1. **B** — Cloud Pub/Sub buffers high-volume clickstream ingestion, while Cloud Dataflow enriches and fans out writes to BigQuery and Memorystore.
2. **B** — Cloud Memorystore for Redis provides sub-millisecond in-memory caching for session management and low-latency state.
3. **D** — Cloud Storage Archive class offers the lowest cost per GB for data accessed less than once a year with immediate availability (not hours of tape delay).
4. **B** — Bigtable row keys should prefix with device IDs or hashes to distribute writes across tablets evenly; leading timestamps cause hotspotting.
5. **B** — GKE Autopilot manages node provisioning, security, and maintenance automatically, billing strictly for pod resource requests.
6. **B** — Spot VMs provide up to 91% discount on compute costs; paired with job checkpointing, they are ideal for fault-tolerant long-running batch jobs.
7. **B** — Warm standby balances cost and recovery speed by continuously replicating data while deferring compute scale-up until a disaster occurs.
8. **B** — Cloud Run and Cloud Functions natively support direct source deployment using automated Google Cloud Buildpacks.
9. **B** — Cloud Storage Bucket Lock enforces SEC Rule 17a-4 and HIPAA-compliant WORM storage, preventing object modification or deletion.
10. **B** — Apigee provides enterprise API governance, developer portals, rate limiting, analytics, and legacy protocol transformation.
