# Domain 5: Reliability, Operations & Official Case Studies — Quiz 2

**Exam:** Google Cloud Certified: Professional Cloud Architect (PCA)  
**Format:** 10 Scenario-Based Multiple-Choice Questions with In-Depth Technical Explanations

---

### Question 1 (Case Study: EHR Healthcare)
EHR Healthcare plans to establish a private network connection between their on-premises data centers and their Google Cloud Shared VPC. They have two corporate data center locations in Chicago and Dallas. They require 20 Gbps of dedicated bandwidth, high availability with a 99.99% SLA, and encryption for specific sensitive subnets. How should this connectivity be designed?
- A. Cloud VPN over public internet
- B. Dedicated Interconnect with dual connections in Chicago and dual connections in Dallas (4 circuits total) across redundant Cloud Routers, with IPsec over Interconnect (HA VPN over Cloud Interconnect) for sensitive subnets
- C. Single Partner Interconnect 50 Mbps circuit
- D. Open public firewall ports

---

### Question 2 (Case Study: Mountkirk Games)
Mountkirk Games collects 10 billion telemetry events daily from game clients worldwide. Telemetry includes player deaths, weapon selections, and level completion times. The data must be ingested continuously, enriched with user demographic metadata, and streamed into BigQuery for real-time game balancing. Which pipeline provides exactly-once stream processing?
- A. Pub/Sub -> Cloud Dataflow (Apache Beam) -> BigQuery
- B. Compute Engine cron job copying files via FTP
- C. Direct writes from mobile clients to BigQuery streaming buffer with no validation
- D. Cloud Storage Nearline

---

### Question 3 (Case Study: Cymbal Direct)
Cymbal Direct is designing a personalized recommendation engine for their online store. When a user views an item, the website must display 5 personalized recommendations in under 50 milliseconds. Machine learning models are trained nightly in Vertex AI using BigQuery historical data. Where should pre-computed recommendation vectors be stored for sub-millisecond serving?
- A. Cloud Memorystore for Redis
- B. Cloud Storage Archive
- C. Compute Engine cold disk
- D. Cloud Logging

---

### Question 4 (Case Study: TerramEarth)
TerramEarth field service technicians use ruggedized tablets to diagnose vehicle issues during maintenance visits. The diagnostic application needs to query real-time vehicle sensor readings by `vehicle_id` and timestamp. The database must sustain hundreds of thousands of concurrent writes from vehicles while responding to technician reads in under 10ms. Which database should power the diagnostic application?
- A. Cloud Bigtable
- B. Cloud SQL with 1 vCPU
- C. Cloud Storage Coldline
- D. BigQuery table with no partitioning

---

### Question 5 (Disaster Recovery: Warm Standby)
An enterprise implements a "Warm Standby" disaster recovery strategy across `us-east4` (primary) and `us-central1` (DR). What does this architecture look like in practice during normal operations?
- A. Zero resources deployed in `us-central1`; only nightly tape backups exist.
- B. A minimal, scaled-down footprint running in `us-central1` (e.g., cross-region asynchronous database read replica active, MIG sized to 1 instance); scaled up to full production capacity only during a disaster.
- C. Full production scale running in both regions with 50/50 live traffic.
- D. Cloud Spanner multi-region active-active.

---

### Question 6 (Reliability: Thundering Herd)
After a 30-minute network outage, 500,000 IoT edge devices reconnect simultaneously and send queued telemetry messages to the backend API, causing the receiving microservices and database to crash repeatedly. How should the client devices be programmed to prevent this failure mode?
- A. Have all devices retry every second continuously.
- B. Implement exponential backoff with randomized jitter on reconnection attempts to distribute the reconnection spike across a wide time window.
- C. Hardcode a fixed 5-minute wait time on all devices.
- D. Delete the client firmware.

---

### Question 7 (Case Study: EHR Healthcare)
EHR Healthcare developers need to run machine learning experiments against real patient records in a non-production development project. To maintain HIPAA compliance, all direct identifiers (names, Social Security numbers, medical record numbers) must be replaced with surrogate tokens or masked before entering the development environment. Which Google Cloud service automates this de-identification pipeline?
- A. Sensitive Data Protection (Cloud DLP)
- B. Cloud Armor
- C. Cloud Trace
- D. Cloud Deploy

---

### Question 8 (Case Study: Cymbal Direct)
During flash sales, Cymbal Direct's frontend web application experiences unexpected slow-downs. Monitoring indicates that the Cloud Run frontend instances are scaling up rapidly, but the backend Cloud SQL database instance is exhausted with 100% CPU and maxed-out connection limits. What architectural component should be placed between Cloud Run and Cloud SQL?
- A. Cloud Armor
- B. Cloud SQL Auth Proxy with connection pooling (e.g. PgBouncer) or Memorystore Redis caching for read-heavy queries
- C. Direct public IP connection
- D. Cloud NAT

---

### Question 9 (Reliability: Health Checks)
An architect configures an auto-healing policy on a Compute Engine Regional Managed Instance Group (MIG). The health check is configured with an HTTP check on port 8080. If an application instance fails the health check 3 consecutive times, what action does the MIG take automatically?
- A. Sends an email to the on-call engineer.
- B. Recreates (replaces) the unhealthy VM instance automatically in the same zone.
- C. Deletes the entire managed instance group.
- D. Shuts down the load balancer.

---

### Question 10 (Case Study: Mountkirk Games)
Mountkirk Games wants to analyze player churn. They need to join real-time telemetry streaming in from active sessions with historical player purchase records spanning the past 3 years. Which data warehouse engine supports streaming inserts while performing high-speed analytical joins over petabytes of data?
- A. Google BigQuery
- B. Cloud Memorystore
- C. Local SSD
- D. Filestore

---

## Answer Key & Rationales

1. **B** — A 99.99% Dedicated Interconnect SLA requires 4 connections across dual metro locations; HA VPN over Interconnect adds IPsec encryption for sensitive traffic.
2. **A** — Pub/Sub provides durable streaming buffering, and Cloud Dataflow (Apache Beam) delivers managed exactly-once stream processing and BigQuery output.
3. **A** — Memorystore for Redis delivers sub-millisecond in-memory cache reads, ideal for real-time recommendation retrieval during web requests.
4. **A** — Cloud Bigtable scales horizontally to ingest massive concurrent time-series sensor writes while serving sub-10ms operational queries.
5. **B** — Warm standby maintains a minimal, scaled-down secondary environment with continuous data replication, rapidly scaled up on demand.
6. **B** — Adding randomized jitter to exponential backoff prevents synchronized retry storms (thundering herds) when systems recover.
7. **A** — Cloud DLP automatically scans, identifies, and de-identifies HIPAA PHI using cryptographic tokenization or masking before data reaches dev/test environments.
8. **B** — Connection pooling (PgBouncer) and read caching prevent serverless auto-scaling microservices from overwhelming database connection limits.
9. **B** — Compute Engine auto-healing health checks detect unhealthy application instances and automatically replace the VM to maintain service availability.
10. **A** — BigQuery natively supports streaming inserts alongside petabyte-scale relational SQL joins across historical and real-time datasets.
