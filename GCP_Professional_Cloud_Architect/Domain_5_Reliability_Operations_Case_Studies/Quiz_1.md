# Domain 5: Reliability, Operations & Official Case Studies — Quiz 1

**Exam:** Google Cloud Certified: Professional Cloud Architect (PCA)  
**Format:** 10 Scenario-Based Multiple-Choice Questions with In-Depth Technical Explanations

---

### Question 1 (Case Study: EHR Healthcare)
EHR Healthcare is migrating its core customer insurance and electronic medical records systems to Google Cloud. The architecture must integrate with legacy on-premises SOAP-based web services, expose standardized RESTful APIs to authorized hospital partners, and enforce rate-limiting and developer key verification. Which Google Cloud solution should be deployed at the API layer?
- A. Apigee API Management
- B. VPC Firewall rules
- C. Cloud Endpoints
- D. Cloud NAT

---

### Question 2 (Case Study: Mountkirk Games)
Mountkirk Games is launching a new real-time multiplayer mobile game where 64 players battle simultaneously in fast-paced 15-minute matches. The game servers require persistent bidirectional UDP network connections and stateful process management. Which compute platform should the architect choose?
- A. Cloud Run
- B. Google Kubernetes Engine (GKE) with the open-source Agones game server orchestration framework
- C. App Engine Standard
- D. Cloud Functions

---

### Question 3 (Case Study: Cymbal Direct)
Cymbal Direct is preparing for Black Friday where website traffic is projected to surge by 1,500%. To prevent inventory overselling across physical retail and online channels, the checkout system requires a database that supports strict ACID transactions, horizontal multi-region scalability, and $99.999\%$ availability. Which database meets these requirements?
- A. Cloud Spanner with a multi-region instance configuration
- B. Cloud Bigtable with single-cluster replication
- C. Cloud SQL MySQL with read replicas
- D. Memorystore for Memcached

---

### Question 4 (Case Study: TerramEarth)
TerramEarth vehicles operate in remote agricultural and mining locations where cellular connectivity is frequently unavailable for days. When a vehicle enters cellular range or returns to a dealer maintenance bay, it uploads up to 50 GB of sensor logs. How should the vehicle edge software and cloud ingestion pipeline be designed?
- A. Stream raw sensor data synchronously over HTTP directly into BigQuery.
- B. Transmit all data via SMS text messages.
- C. Cache sensor readings on local vehicle flash storage, and asynchronously publish buffered records to Cloud Pub/Sub when connectivity is established, with automatic retry and backoff.
- D. Discard data whenever connectivity is lost.

---

### Question 5 (Disaster Recovery: RPO & RTO)
An enterprise financial service requires a Disaster Recovery architecture with a Recovery Point Objective (RPO) of 0 (zero data loss) and a Recovery Time Objective (RTO) of less than 10 seconds if an entire Google Cloud region experiences a catastrophic event. Which architecture fulfills this requirement?
- A. Warm standby Compute Engine MIG with manual DNS failover
- B. Active-Active multi-region deployment behind a Global External HTTP(S) Load Balancer backed by Cloud Spanner multi-region instance
- C. Nightly database backups to Cloud Storage Archive
- D. Single-zone Cloud SQL database

---

### Question 6 (Case Study: EHR Healthcare)
EHR Healthcare must comply with strict HIPAA regulations. All patient data stored in Cloud Storage must be encrypted using keys that EHR Healthcare can rotate and cryptographically revoke at any time. Furthermore, data access by Google Cloud personnel during technical support must be recorded in an immutable log. Which two Google Cloud features are required?
- A. Customer-Managed Encryption Keys (CMEK) via Cloud KMS, and Access Transparency
- B. Cloud Trace and Cloud Profiler
- C. Spot VMs and Cloud Build
- D. Google default encryption and Cloud NAT

---

### Question 7 (Case Study: Mountkirk Games)
Mountkirk Games needs to store and query global player leaderboards with millions of active users. Leaderboard lookups must complete with single-digit millisecond latency during intense gaming events. Which Google Cloud database is best suited for real-time leaderboards and fast key-value lookups?
- A. BigQuery batch queries
- B. Cloud Storage Nearline
- C. Cloud SQL single-node
- D. Cloud Bigtable or Memorystore for Redis

---

### Question 8 (Case Study: TerramEarth)
TerramEarth's engineering team wants to train machine learning models to predict mechanical engine failures before they occur in the field. They need to analyze 5 years of historical sensor data (200 Petabytes) alongside maintenance service records. Which Google Cloud analytics and AI platform should they use?
- A. Cloud Run functions
- B. Cloud Firestore
- C. Compute Engine single VM with Excel
- D. BigQuery for petabyte-scale data warehousing and Vertex AI for predictive model training

---

### Question 9 (Reliability & Circuit Breaking)
A microservice on GKE calls an external credit score verification API over HTTPS. When the external API experiences severe latency and starts failing, incoming requests to the GKE microservice pile up, exhausting thread pools and crashing the pod. What architectural design pattern prevents this cascading failure?
- A. Disabling Kubernetes health checks
- B. Increasing pod memory requests by 1,000%
- C. Circuit Breaker pattern with exponential backoff and fallback responses
- D. Infinite retries without backoff

---

### Question 10 (Case Study: Cymbal Direct)
Cymbal Direct wants to prevent malicious botnets from scraping product prices and executing credential-stuffing attacks during holiday flash sales. Which Google Cloud service provides Layer 7 Web Application Firewall (WAF) and bot management protection at the global edge?
- A. Cloud NAT
- B. Cloud DNS
- C. Cloud Armor
- D. VPC Peering

---

## Answer Key & Rationales

1. **A** — Apigee provides enterprise API governance, protocol transformation (SOAP to REST), rate-limiting, and developer portal management.
2. **B** — Agones on GKE provides native Kubernetes controllers for running and scaling stateful, dedicated UDP game server processes.
3. **A** — Cloud Spanner multi-region delivers horizontally scalable ACID transactions and 99.999% availability, preventing race conditions in global inventory.
4. **C** — Buffering data locally on vehicle flash storage and publishing to Pub/Sub when connected handles intermittent network connectivity gracefully.
5. **B** — Active-Active multi-region with Anycast Global HTTPS LB and multi-region Cloud Spanner delivers RPO=0 and near-instantaneous automated RTO.
6. **A** — CMEK provides customer lifecycle control over encryption keys, while Access Transparency records Google personnel interventions for HIPAA compliance.
7. **D** — Cloud Bigtable and Memorystore Redis are optimized for sub-10ms key-value lookups required by high-throughput real-time gaming leaderboards.
8. **D** — BigQuery stores and queries multi-petabyte datasets effortlessly, while Vertex AI builds and trains predictive maintenance ML pipelines.
9. **C** — Circuit breakers fail fast when downstream dependencies fail, shielding the calling service from thread exhaustion and cascading crashes.
10. **C** — Cloud Armor integrates with the Global External HTTP(S) Load Balancer to provide rate limiting, bot detection, and OWASP WAF defense at the network edge.
