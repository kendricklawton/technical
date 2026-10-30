# GCP Professional Cloud Architect — Practice Exam 1

**Difficulty:** Professional Cloud Architect Scenario Exam  
**Format:** 15 multi-paragraph scenario questions · single answer · four options (A–D)  
**Time Allowed:** 75 minutes  
**Passing Target:** 12 / 15 (80%)

---

### Question 1 (Domain 1: Compute & Database Architecture)
A multinational investment bank is building a real-time global trade settlement system. The system must process transactions originating from London, New York, and Singapore simultaneously. The database must guarantee strict ACID compliance and serializability across all regions, support petabyte-scale growth, and provide five 9s ($99.999\%$) availability with zero data loss if an entire cloud region fails. What database solution meets these requirements?
- A. Cloud SQL for PostgreSQL with cross-region read replicas
- B. Cloud Spanner configured with a multi-region instance spanning Europe, North America, and Asia
- C. Cloud Bigtable with multi-cluster replication
- D. Firestore in Datastore mode

---

### Question 2 (Domain 5: Mountkirk Games Case Study)
Mountkirk Games is releasing an online multiplayer mobile action title. During peak gaming hours, 500,000 concurrent players join 64-player matches. The dedicated game server processes run on UDP and maintain stateful in-memory physics simulations. Which compute platform should the architect choose to deploy and automatically scale these game servers?
- A. Cloud Run with concurrency set to 80
- B. Google Kubernetes Engine (GKE) Standard node pools running the open-source Agones game server orchestration framework
- C. App Engine Standard
- D. Cloud Functions triggered by Pub/Sub

---

### Question 3 (Domain 2: Hybrid Networking & Shared VPC)
An enterprise is migrating 30 departmental applications to Google Cloud. The central network security team requires exclusive control over subnets, firewall rules, and hybrid connectivity to on-premises data centers via Dedicated Interconnect. However, application development teams must be able to deploy Compute Engine instances and GKE clusters into their respective subnets without granting them network administrator rights. How should this network architecture be structured?
- A. Create a single GCP project containing all 30 applications and grant everyone `Owner`.
- B. Implement a Shared VPC architecture with a central Host Project managed by the network team, and attach departmental Service Projects to the Host Project, granting developers `roles/compute.networkUser` on their specific subnets.
- C. Build 30 separate VPCs and connect them using a full mesh of 435 VPC Network Peering links.
- D. Use public IP addresses for all VMs and secure them with Linux iptables.

---

### Question 4 (Domain 5: TerramEarth Case Study)
TerramEarth operates 20 million connected agricultural and construction vehicles worldwide. Vehicles collect sensor metrics (engine temperature, hydraulic pressure, GPS coordinates) every second. In remote rural areas, cellular connectivity is unavailable for days. When connectivity is re-established, vehicles upload up to 20 GB of buffered sensor readings. How should the edge and cloud ingestion pipeline be designed to prevent data loss?
- A. Stream sensor data over direct HTTP POST to a single Compute Engine VM.
- B. Buffer sensor data on vehicle flash storage, upload to Cloud Pub/Sub with automatic retry and exponential backoff when connected, process via Cloud Dataflow, and write time-series records to Cloud Bigtable.
- C. Transmit data via SMS text messages.
- D. Discard readings whenever cellular connectivity is lost.

---

### Question 5 (Domain 3: Security & Compliance)
A healthcare company stores patient medical history records in Google Cloud Storage. Strict HIPAA compliance dictates that all data must be encrypted with keys owned and managed by the healthcare company, rotated every 90 days, and capable of instantaneous cryptographic shredding. Furthermore, any access by Google Cloud technicians during support cases must generate an immutable audit log. Which configuration fulfills these requirements?
- A. Google default encryption and Cloud Trace
- B. Customer-Managed Encryption Keys (CMEK) stored in Cloud KMS, combined with Access Transparency logging
- C. Customer-Supplied Encryption Keys (CSEK) stored in a public GitHub repo
- D. Password-protected zip archives stored in Nearline buckets

---

### Question 6 (Domain 4: FinOps & Cost Optimization)
An enterprise runs 400 stateless web frontend VMs on Compute Engine in `us-central1` that have maintained steady, 24/7 resource utilization over the past two years and will continue running for the next three years. The CFO demands an immediate reduction in compute spending without changing machine types or introducing downtime. Which purchasing option should the architect implement?
- A. Spot VMs
- B. 3-Year Resource-Based Committed Use Discounts (CUDs)
- C. BigQuery On-Demand reservations
- D. Preemptive billing credits

---

### Question 7 (Domain 5: Cymbal Direct Case Study)
Cymbal Direct is preparing for a nationwide holiday flash sale where order volumes are expected to surge tenfold for two hours. During the sale, buyers across the United States must see real-time inventory counts to prevent selling products that are out of stock. Which architecture provides atomic inventory decrements and high-concurrency order placement?
- A. Cloud Storage Nearline files updated via FTP
- B. Cloud Spanner multi-region instance executing read-write transactions for inventory and order processing, fronted by Cloud Run microservices
- C. Cloud SQL single-node with asynchronous read replicas
- D. Cloud Bigtable with eventual consistency

---

### Question 8 (Domain 2: Load Balancing & CDN)
A global media streaming platform hosts video-on-demand assets in Cloud Storage. Users in Australia and Europe complain that video thumbnail images take over 1.5 seconds to load. How can the architect reduce thumbnail latency to under 100 milliseconds worldwide while lowering origin egress bandwidth costs?
- A. Replicate the images manually across 50 Compute Engine VMs with local disks.
- B. Place a Global External HTTP(S) Load Balancer in front of the Cloud Storage bucket and enable Cloud CDN caching.
- C. Compress images into text files.
- D. Switch from HTTPS to unencrypted HTTP.

---

### Question 9 (Domain 3: Zero Trust Architecture)
An enterprise security team requires that internal corporate web applications running on private Compute Engine VMs without public IPs be accessible to remote employees worldwide. Access must be granted based on user identity, corporate device verification, and location without requiring a corporate VPN client. Which Google Cloud solution implements this zero-trust architecture?
- A. BeyondCorp Enterprise with Identity-Aware Proxy (IAP)
- B. Cloud NAT
- C. Opening firewall port 80 to `0.0.0.0/0`
- D. Cloud VPN with static routes

---

### Question 10 (Domain 1: Serverless & Event-Driven Architecture)
An architect is designing an automated order processing pipeline. When a customer places an order, an invoice PDF must be generated, a payment processed, an inventory reservation made, and a confirmation email sent. These microservices are serverless and can execute in parallel or fail independently with automated retries. Which service should orchestrate this workflow?
- A. Compute Engine Startup Scripts
- B. Cloud Workflows
- C. Cloud DNS
- D. Cloud Shell

---

### Question 11 (Domain 4: Data Migration)
A financial firm needs to migrate an on-premises 12 TB Oracle database to Cloud SQL for PostgreSQL. The business can only tolerate a 15-minute maintenance cutover window over the weekend. Which migration approach satisfies this minimal-downtime requirement?
- A. Take a physical tape backup and mail it to Google.
- B. Use Database Migration Service (DMS) to perform an initial data load followed by continuous Change Data Capture (CDC) replication until the maintenance window, then promote Cloud SQL to primary.
- C. Run a single `pg_dump` script during the 15-minute window.
- D. Recreate the database manually.

---

### Question 12 (Domain 5: Reliability & Disaster Recovery)
An architect is evaluating disaster recovery architectures for an enterprise ERP system. The business executives require a Recovery Point Objective (RPO) of less than 15 minutes and a Recovery Time Objective (RTO) of less than 1 hour, while minimizing idle infrastructure costs during normal operations. Which DR pattern matches these parameters?
- A. Multi-Region Active-Active with duplicate infrastructure running at 100% capacity in both regions.
- B. Warm Standby: Replicating database changes continuously to a secondary region where a minimal compute footprint runs, configured to auto-scale to full capacity during a disaster failover.
- C. Cold Backup: Weekly file backups stored in an on-premises tape drive.
- D. Single-zone deployment without backups.

---

### Question 13 (Domain 3: Data Exfiltration Prevention)
An organization operates in a highly regulated financial sector. An architect must ensure that developers and automated services in project `fin-analytics` cannot copy BigQuery datasets to unauthorized Cloud Storage buckets outside the company's Google Cloud organization, even if the service accounts possess administrative permissions. Which security mechanism enforces this perimeter?
- A. Cloud Armor
- B. VPC Service Controls (VPC-SC)
- C. Cloud NAT
- D. Cloud DNS Peering

---

### Question 14 (Domain 1: Big Data & Analytics)
A marketing analytics platform ingests 2 Terabytes of user clickstream logs daily into BigQuery. Analysts run queries that frequently filter by `event_timestamp` and group by `user_country` and `campaign_id`. Over time, queries have become slow and expensive. How should the BigQuery table be architected to optimize performance and reduce cost?
- A. Create 365 separate tables each year.
- B. Partition the table by `event_timestamp` (Day) and cluster by `user_country` and `campaign_id`.
- C. Export all data to CSV files.
- D. Run queries on Cloud SQL instead.

---

### Question 15 (Domain 5: EHR Healthcare Case Study)
EHR Healthcare needs to expose patient health metrics to third-party mobile health applications developed by external partners. The system must support OAuth 2.0 token validation, developer self-service registration, rate limiting to prevent API abuse, and translation of legacy SOAP/XML responses into modern JSON REST responses. Which Google Cloud service meets all of these criteria?
- A. Cloud Endpoints
- B. Apigee API Management
- C. Cloud NAT
- D. Google Cloud Armor

---

## Answer Key & Rationales

1. **B** — Cloud Spanner multi-region provides globally distributed ACID transactions and 99.999% availability backed by Google's TrueTime atomic clocks.
2. **B** — Agones on GKE Standard provides Kubernetes controllers specifically designed to scale stateful, low-latency UDP multiplayer game servers.
3. **B** — Shared VPC enforces least privilege by centralizing network administration in the Host Project while allowing Service Projects to deploy compute on subnets.
4. **B** — Buffering on flash storage and publishing to Pub/Sub when connectivity is acquired ensures zero data loss during intermittent connectivity.
5. **B** — CMEK in Cloud KMS grants customers key lifecycle control, while Access Transparency provides auditability of Google technician data access.
6. **B** — 3-Year Resource-Based Committed Use Discounts provide up to 57% savings on steady, predictable compute capacity with zero downtime.
7. **B** — Cloud Spanner's horizontal scale and strong external consistency prevent race conditions and overselling during massive concurrent flash sales.
8. **B** — Global External HTTPS LB with Cloud CDN caches static assets at edge PoPs globally, reducing latency to <100ms and reducing origin egress costs.
9. **A** — BeyondCorp Enterprise with IAP enforces zero-trust context-aware access based on user identity and device posture without requiring a VPN.
10. **B** — Cloud Workflows coordinates serverless microservices with stateful branching, parallel execution, and built-in retry logic.
11. **B** — Database Migration Service (DMS) utilizes continuous CDC replication to keep target databases in sync, enabling cutovers within 15 minutes.
12. **B** — Warm standby balances cost and recovery velocity by maintaining continuous data replication and scaling compute on-demand during failover.
13. **B** — VPC Service Controls establish security perimeters around Google Cloud APIs, preventing unauthorized data exfiltration outside authorized boundaries.
14. **B** — Partitioning by date restricts scanned data to relevant time slices, and clustering organizes data by query filter keys, slashing query costs.
15. **B** — Apigee provides enterprise API gateway governance, OAuth 2.0 authorization, rate limiting, and SOAP-to-REST payload transformation.
