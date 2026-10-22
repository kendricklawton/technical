# GCP Professional Cloud Architect — Practice Exam 2

**Difficulty:** Professional Cloud Architect Scenario Exam  
**Format:** 15 multi-paragraph scenario questions · single answer · four options (A–D)  
**Time Allowed:** 75 minutes  
**Passing Target:** 12 / 15 (80%)

---

### Question 1 (Domain 2: Dedicated Interconnect & Resiliency)
An enterprise requires a dedicated hybrid connection between its primary datacenter in New York and Google Cloud. The workload requires 40 Gbps of bandwidth, sub-10ms network latency, and an enterprise 99.99% availability SLA. How must the Dedicated Interconnect be architected to achieve the 99.99% SLA?
- A. A single 100 Gbps Dedicated Interconnect circuit in New York.
- B. Four Dedicated Interconnect connections deployed across two different colocation facilities (dual metros) and two distinct Cloud Routers in Google Cloud.
- C. Cloud HA VPN with 2 tunnels over the public internet.
- D. Partner Interconnect with a single 10 Gbps circuit.

---

### Question 2 (Domain 3: IAM & Privileged Access)
A cloud security architect wants to ensure that developers do not possess permanent administrative permissions in the production Google Cloud project. When a production incident occurs, an engineer must request temporary, time-bound elevated access that requires explicit approval from an SRE manager and produces an audit trail. Which Google Cloud service enables this workflow?
- A. Cloud Identity Free
- B. Privileged Access Manager (PAM)
- C. Static Service Account JSON keys
- D. Cloud Armor

---

### Question 3 (Domain 5: Cymbal Direct Case Study)
Cymbal Direct experiences malicious web scraping where competitor bots scrape pricing information thousands of times per second from their product catalog, degrading frontend performance for human shoppers. How can the architect protect the application at the network edge without altering backend code?
- A. Add CAPTCHA checks to every single database query.
- B. Deploy Cloud Armor security policies with Bot Management (reCAPTCHA Enterprise integration) and rate-limiting rules attached to the Global External HTTPS Load Balancer.
- C. Shut down the web store during night hours.
- D. Block all traffic from the internet.

---

### Question 4 (Domain 1: Compute Engine vs GKE)
An engineering team is evaluating whether to deploy a new microservices application on GKE Autopilot or Compute Engine Managed Instance Groups (MIGs). The application consists of 40 small containerized services written in Node.js and Go that experience variable traffic throughout the day. The team has no dedicated infrastructure management staff. What is the primary architectural justification for choosing GKE Autopilot?
- A. GKE Autopilot is completely free of charge.
- B. GKE Autopilot eliminates the operational burden of node provisioning, OS patching, and cluster security hardening while billing exclusively for the CPU and memory requested by active pods.
- C. Compute Engine does not support containers.
- D. GKE Autopilot does not use Kubernetes.

---

### Question 5 (Domain 4: Data Lake Migration)
A media corporation needs to migrate 1.5 Petabytes of archival video files from Amazon S3 to Google Cloud Storage. The migration must run continuously in the background, synchronize newly added files, validate checksums to ensure zero bit rot, and not require provisioning or managing compute instances. Which tool should be used?
- A. Google Cloud Transfer Appliance
- B. Storage Transfer Service (STS) configured with an Amazon S3 source and Cloud Storage destination
- C. Running `aws s3 cp` on a developer laptop
- D. Writing a Python script using boto3 on a single VM

---

### Question 6 (Domain 3: Data Encryption with Cloud EKM)
A government defense agency is adopting Google Cloud for data analytics in BigQuery. Strict national security regulations mandate that cryptographic encryption keys must reside exclusively inside physical Hardware Security Modules (HSMs) situated within the agency's physical premises, and Google must never store or cache key material. Which key management architecture is required?
- A. Google default encryption
- B. Cloud External Key Manager (Cloud EKM)
- C. Customer-Managed Encryption Keys (CMEK) with software keys in Cloud KMS
- D. Hardcoding AES keys in SQL queries

---

### Question 7 (Domain 5: Mountkirk Games Case Study)
Mountkirk Games wants to analyze player telemetry to detect fraudulent behavior and game tampering in real time. Game clients send 50,000 telemetry events per second. The pipeline must detect anomalies within 30 seconds and alert the anti-cheat team. What data architecture meets this requirement?
- A. Telemetry published to Cloud Pub/Sub -> processed in real time by Cloud Dataflow using sliding time windows -> anomaly flags written to BigQuery and pushed to an operational dashboard.
- B. Exporting database dumps to CSV once per week.
- C. Writing logs to local VM disk files.
- D. Running daily batch MapReduce jobs.

---

### Question 8 (Domain 2: Private Service Connect)
A banking platform running in VPC-A needs to securely query an account verification service managed by an external FinTech provider running in VPC-B in another GCP organization. The connection must be private, unidirectional (VPC-A can initiate calls to VPC-B, but VPC-B cannot access VPC-A), and must not require coordinating non-overlapping IP address ranges. Which technology should be chosen?
- A. VPC Network Peering
- B. Private Service Connect (PSC) using Service Attachments and Forwarding Rules
- C. Public IP addresses with Basic Auth
- D. Cloud NAT

---

### Question 9 (Domain 1: Database Selection)
An enterprise application requires an operational database that stores user session profiles and shopping cart data. The application requires flexible JSON document schemas, real-time client synchronization, offline mobile database support, and multi-region automatic replication with 99.999% availability. Which database should be selected?
- A. Cloud SQL MySQL
- B. Firestore in Native mode
- C. Cloud Bigtable
- D. Filestore

---

### Question 10 (Domain 4: BigQuery Cost Optimization)
A retail company's monthly BigQuery bill increased dramatically due to multiple departments running unoptimized queries. The platform architect wants to establish predictable monthly billing, isolate compute capacity between departments so one team's queries do not starve another, and prevent billing surprises. Which BigQuery feature should be implemented?
- A. On-Demand pricing with no limits
- B. BigQuery Editions (Standard or Enterprise) with Slot Reservations and dedicated reservation assignments for each department
- C. Exporting data to Google Drive
- D. Deleting old datasets

---

### Question 11 (Domain 5: EHR Healthcare Case Study)
EHR Healthcare is designing a data exchange platform to share patient health records with clinical research laboratories. Before any records are exported from BigQuery to researchers, direct patient identifiers (SSNs, names, phone numbers) must be automatically inspected and replaced with irreversible cryptographic tokens. Which Google Cloud service natively provides this automated de-identification?
- A. Sensitive Data Protection (Cloud DLP)
- B. Cloud Armor
- C. Cloud Trace
- D. Cloud Composer

---

### Question 12 (Domain 2: Cloud HA VPN Dynamic Routing)
When deploying Cloud HA VPN to connect an on-premises network to a Google Cloud VPC, the network engineer configures Cloud Router. What routing protocol must be used between Cloud Router and the on-premises VPN gateway to support dynamic failover and qualify for Google's 99.99% HA VPN SLA?
- A. Static routing
- B. Border Gateway Protocol (BGP)
- C. RIPv2
- D. OSPF

---

### Question 13 (Domain 3: Organization Hierarchy Governance)
An organization has three main divisions: Sales, Engineering, and Finance. The enterprise architect must enforce that no project in the Engineering folder can ever create resources outside of European regions (`europe-west1` and `europe-west4`) to comply with corporate data governance. Where should this constraint be configured?
- A. On every VM manually
- B. Apply the `constraints/gcp.resourceLocations` Organization Policy constraint at the `Engineering` Folder level
- C. In the DNS configuration
- D. Ask developers to check the region before clicking deploy

---

### Question 14 (Domain 5: TerramEarth Case Study)
TerramEarth wants to run complex analytical SQL queries against 50 Terabytes of daily machine telemetry data to calculate fleet fuel efficiency by engine model. The queries must return results in seconds and support standard ANSI SQL syntax without managing database server instances or clusters. Which service should be used?
- A. Google BigQuery
- B. Cloud Bigtable
- C. Cloud SQL PostgreSQL
- D. Memorystore Redis

---

### Question 15 (Domain 4: Spot VMs in Batch Architectures)
A genomics institute processes DNA sequencing workloads using Compute Engine instances. The batch pipeline can be interrupted, checkpointed, and resumed without losing progress. The institute needs to minimize compute costs for 2,000 vCPUs. What compute model should the architect recommend?
- A. High-memory M3 instances on on-demand pricing
- B. Spot VMs configured in a Regional Managed Instance Group with stateful checkpointing
- C. 3-Year Committed Use Discounts
- D. Bare Metal Solution

---

## Answer Key & Rationales

1. **B** — A 99.99% Dedicated Interconnect SLA mandates 4 circuits deployed across two distinct metro colocation facilities and dual Cloud Routers.
2. **B** — Privileged Access Manager (PAM) provides automated just-in-time privilege escalation, eliminating permanent administrative privileges.
3. **B** — Cloud Armor Bot Management integrates reCAPTCHA Enterprise and rate-limiting rules at the global edge to mitigate scraper traffic.
4. **B** — GKE Autopilot manages node provisioning, scaling, and security hardening automatically, charging only for requested pod resources.
5. **B** — Storage Transfer Service provides fully managed, scalable, serverless data migration from S3 to Cloud Storage with built-in checksum verification.
6. **B** — Cloud External Key Manager (Cloud EKM) allows BigQuery to encrypt data using keys stored in customer-controlled on-premises HSMs.
7. **A** — Pub/Sub ingestion, Dataflow streaming analytics with sliding windows, and BigQuery storage deliver sub-minute anomaly detection.
8. **B** — Private Service Connect enables private, unidirectional service publication and consumption without network peering complexity or IP overlap.
9. **B** — Firestore Native mode offers real-time synchronization, mobile offline data caching, and multi-region 99.999% availability.
10. **B** — BigQuery Editions with Slot Reservations provide predictable, fixed monthly compute costs and eliminate query resource contention.
11. **A** — Cloud DLP automatically discovers, inspects, and de-identifies sensitive HIPAA PHI using cryptographic tokenization or masking.
12. **B** — Dynamic routing using Cloud Router and BGP across dual HA VPN tunnels is required to satisfy the 99.99% availability SLA.
13. **B** — Applying Organization Policy constraints at the Folder level enforces non-bypassable resource location restrictions across all child projects.
14. **A** — BigQuery is Google's serverless data warehouse, capable of executing ANSI SQL queries across multi-terabyte datasets in seconds.
15. **B** — Spot VMs provide up to 91% discount on Compute Engine instances, ideal for fault-tolerant, checkpointable batch genomics pipelines.
