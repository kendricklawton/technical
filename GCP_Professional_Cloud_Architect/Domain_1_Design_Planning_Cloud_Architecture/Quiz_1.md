# Domain 1: Designing and Planning a Cloud Solution Architecture — Quiz 1

**Exam:** Google Cloud Certified: Professional Cloud Architect (PCA)  
**Format:** 10 Scenario-Based Multiple-Choice Questions with In-Depth Technical Explanations

---

### Question 1
A multinational retail banking corporation requires a database for their new global payment ledger. The system must support ACID transactions across multiple continents, handle millions of transactions per day with horizontal scalability, and guarantee $99.999\%$ (five 9s) availability without data inconsistency. Which Google Cloud database service is designed for this architecture?
- A. Cloud SQL with read replicas
- B. Cloud Spanner with a multi-region instance configuration
- C. Cloud Bigtable with single-cluster replication
- D. Firestore in Datastore mode

---

### Question 2
An online media publisher is experiencing rapid unpredictable spikes in web traffic whenever breaking news occurs. Their application is a stateless Go container that parses web requests and responds within 15 milliseconds. The team has no dedicated infrastructure management staff and wants zero operational overhead, scaling to zero when traffic is low and scaling to hundreds of instances instantaneously during news events. Which compute platform should they choose?
- A. Compute Engine single VM
- B. Cloud Run (fully managed)
- C. Google Kubernetes Engine Standard with manual node pool sizing
- D. Google Cloud VMware Engine (GCVE)

---

### Question 3
A connected autonomous vehicle company collects sensor data from 100,000 vehicles every 5 seconds. The telemetry stream reaches 500,000 writes per second. The operations team needs a database capable of sub-10ms write latency and linear horizontal scaling to ingest time-series data for operational dashboards. Which database meets these requirements?
- A. Cloud Spanner
- B. Cloud Bigtable
- C. Cloud SQL for PostgreSQL
- D. Cloud Storage Nearline

---

### Question 4
An architect is designing a batch and streaming analytics pipeline using Cloud Pub/Sub and Apache Beam on Google Cloud. The system must process financial transactions and guarantee that every transaction is processed exactly once, even in the event of worker node crashes or network partitions. Which service should be used to execute the pipeline?
- A. Compute Engine Cron jobs
- B. Cloud Dataflow
- C. Cloud Functions
- D. Cloud Tasks

---

### Question 5
A data warehouse team runs complex analytical queries over a 500 TB dataset in BigQuery. Users complain that queries are becoming expensive and slow. Analysis shows that queries frequently filter by `transaction_date` and filter/group by `store_id` and `department_id`. What table design optimization should the architect recommend?
- A. Export data to Cloud Storage and query using Google Sheets.
- B. Partition the BigQuery table by `transaction_date` and cluster by `store_id` and `department_id`.
- C. Convert the data to uncompressed CSV.
- D. Move the entire dataset to Cloud SQL.

---

### Question 6
An architect needs to deploy a legacy enterprise Windows Server application that relies on specific Active Directory Group Policies and custom kernel drivers. The application cannot be containerized. High availability and automated recovery from VM hardware failures are required. What should the architect propose?
- A. Cloud Run
- B. Compute Engine Regional Managed Instance Group (MIG) with an auto-healing health check and stateful configuration
- C. App Engine Standard Environment
- D. Cloud Functions

---

### Question 7
A mobile gaming studio is designing a real-time multiplayer backend. Dedicated game server instances require low-latency UDP networking and custom process management to host 64-player matches. What container orchestration platform is specifically optimized for this workload?
- A. Cloud Run
- B. Google Kubernetes Engine (GKE) with the open-source Agones game server management platform
- C. App Engine Flexible
- D. Google Cloud Storage

---

### Question 8
An e-commerce company must store product image catalogs. The images must be accessible with sub-second latency from any continent, have 99.99% availability, and provide strong consistency so that updated product images are immediately visible to all buyers globally upon upload. Which storage solution should be used?
- A. Cloud Storage Multi-Region (Standard storage class)
- B. Cloud Storage Coldline
- C. Compute Engine Persistent Disk with multi-attach
- D. Filestore Basic

---

### Question 9
A financial services firm runs an OLTP trading application on Cloud SQL for PostgreSQL. The company requires a Recovery Point Objective (RPO) of near-zero and a Recovery Time Objective (RTO) of less than 60 seconds if an entire Google Cloud zone suffers a hardware outage. How should Cloud SQL be configured?
- A. Cloud SQL single zone with nightly backups
- B. Cloud SQL configured with Regional High Availability (synchronous standby in another zone with automatic failover)
- C. Exporting database dumps to Cloud Storage every hour
- D. Migrating to Google Cloud Bigtable

---

### Question 10
An architect is designing an event-driven decoupled architecture where an order processing service publishes events to multiple downstream subscribers (Fraud Detection, Inventory, and Shipping). Each subscriber processes messages at its own pace. What messaging pattern and service should be used?
- A. Cloud Tasks with a single task queue
- B. Cloud Pub/Sub with a single topic and multiple independent subscriptions
- C. Shared Compute Engine persistent disk
- D. Direct HTTP calls with retry loops

---

## Answer Key & Rationales

1. **B** — Cloud Spanner is Google's globally scalable relational database supporting ACID transactions and 99.999% availability backed by TrueTime.
2. **B** — Cloud Run provides fully managed serverless container execution with instant auto-scaling to zero and no node management overhead.
3. **B** — Cloud Bigtable is optimized for massive time-series sensor ingestion at single-digit millisecond write latencies.
4. **B** — Cloud Dataflow (Apache Beam) provides managed stream/batch processing with built-in exactly-once processing guarantees.
5. **B** — Partitioning by date and clustering by high-cardinality query keys restricts data scanned by queries, optimizing speed and reducing cost.
6. **B** — Compute Engine Regional MIGs with auto-healing provide multi-zone redundancy and automatic VM replacement for legacy operating systems.
7. **B** — Agones on GKE provides specialized Kubernetes primitives for managing stateful, low-latency UDP multiplayer game servers.
8. **A** — Cloud Storage Multi-Region Standard delivers global low-latency access, 99.99% availability, and global strong consistency for object mutations.
9. **B** — Regional HA in Cloud SQL provides synchronous standby replication in a secondary zone with sub-minute automatic failover.
10. **B** — Cloud Pub/Sub publish-subscribe fan-out allows a single topic to broadcast to multiple decoupled subscriptions independently.
