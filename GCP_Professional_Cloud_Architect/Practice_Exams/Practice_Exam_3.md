# GCP Professional Cloud Architect — Practice Exam 3

**Difficulty:** Professional Cloud Architect Scenario Exam  
**Format:** 15 multi-paragraph scenario questions · single answer · four options (A–D)  
**Time Allowed:** 75 minutes  
**Passing Target:** 12 / 15 (80%)

---

### Question 1 (Domain 5: Cymbal Direct Case Study)
Cymbal Direct is deploying its core checkout API across two Google Cloud regions: `us-central1` and `us-east4`. During peak holiday sales, an unexpected regional datacenter failure occurs in `us-central1`. The architecture must ensure that user checkout requests are instantly routed to `us-east4` without requiring DNS propagation or manual DNS record modifications. What load balancing architecture provides this automated sub-second failover?
- A. Cloud DNS with round-robin routing
- B. Global External HTTP(S) Load Balancer using Anycast IP routing with backend services deployed in both regions
- C. External TCP/UDP Network Load Balancer in each region
- D. Internal HTTP(S) Load Balancer

---

### Question 2 (Domain 1: Storage Class Lifecycle Management)
An insurance provider generates 500,000 claims documents per month in Cloud Storage. Documents must be accessible with sub-second latency for the first 30 days after creation. Between 30 and 90 days, documents are accessed less than once a month. After 90 days, documents are accessed fewer than once a year but must be retained for 7 years to satisfy regulatory compliance. What is the most cost-effective storage lifecycle policy?
- A. Keep all documents in Standard storage forever.
- B. Transition from Standard to Nearline after 30 days, to Coldline after 90 days, to Archive after 365 days, and delete after 2,555 days (7 years).
- C. Delete documents after 30 days.
- D. Download all files to local hard drives after 90 days.

---

### Question 3 (Domain 5: EHR Healthcare Case Study)
EHR Healthcare operates a legacy Oracle database on-premises that contains patient prescription history. They are developing a cloud-native patient mobile portal on GKE. The mobile portal needs to read prescription records with under 20ms latency. The Oracle database cannot be decommissioned for 18 months. What hybrid architecture should the architect implement?
- A. Connect GKE directly to on-premises Oracle over the public internet with plain text credentials.
- B. Establish a Dedicated Interconnect, continuously replicate prescription changes from Oracle to a Cloud SQL or Cloud Spanner database using Database Migration Service / GoldenGate CDC, and have GKE read from the cloud database.
- C. Move the mobile application back to on-premises.
- D. Export CSV files once a week.

---

### Question 4 (Domain 3: IAM Service Account Security)
A security audit reveals that several Compute Engine VM instances in a production project are running with the default Compute Engine service account (`PROJECT_NUMBER-compute@developer.gserviceaccount.com`), which has been granted the `Editor` role. Why is this an architectural security hazard, and what is the remediation?
- A. Default service accounts slow down VM boot times; delete the VMs.
- B. The `Editor` role violates the Principle of Least Privilege by granting broad mutation and deletion permissions across all GCP resources; replace it with a dedicated custom service account granted only the specific roles required by the application.
- C. Default service accounts incur extra billing charges.
- D. No action needed; `Editor` is recommended by Google.

---

### Question 5 (Domain 2: Cloud NAT & Outbound Security)
An enterprise deploys a fleet of private Compute Engine VMs in a backend subnet. The VMs must download OS software security updates from the public internet. However, security policy strictly prohibits instances from having public external IP addresses or accepting inbound internet connections. What networking component should be deployed?
- A. Cloud Interconnect
- B. Cloud NAT (Network Address Translation) configured on Cloud Router
- C. Cloud Armor
- D. Assigning static external IPs to all VMs

---

### Question 6 (Domain 5: TerramEarth Case Study)
TerramEarth wants to deploy an AI-driven predictive maintenance pipeline. Vehicle sensors capture engine vibration data. When vibration patterns match known bearing failure signatures, a replacement part should be dispatched automatically to the nearest dealership before the machine breaks down. What Google Cloud AI and data services should be chained together?
- A. Pub/Sub -> Dataflow -> Vertex AI custom trained model -> Cloud Functions to trigger ERP shipment
- B. Cloud SQL with manual Excel queries
- C. Cloud DNS and Cloud Trace
- D. Storage Transfer Service running monthly

---

### Question 7 (Domain 4: Cloud Migration - 6 Rs)
A company evaluates its application portfolio before a major cloud migration. One legacy reporting application runs on an unsupported 32-bit operating system, has no active developer maintainers, and its functionality has already been replaced by a modern BigQuery dashboard. Under the "6 Rs" framework, what action should be taken?
- A. Refactor to Cloud Run
- B. Retire (Decommission the application)
- C. Rehost to Compute Engine
- D. Repurchase

---

### Question 8 (Domain 1: GKE Ingress & Gateway API)
A platform engineering team is managing 20 microservices deployed in a GKE cluster. They want to implement traffic splitting (canary releases), cross-namespace routing, and HTTP header-based routing using modern Kubernetes standards supported natively by Google Cloud Load Balancing. Which resource should they configure?
- A. Classic Kubernetes Ingress (`extensions/v1beta1`)
- B. Kubernetes Gateway API (`gateway.networking.k8s.io`) with GKE Gateway controller
- C. NodePort services for each pod
- D. Modifying cluster CoreDNS records

---

### Question 9 (Domain 3: Cloud KMS Key Rotation)
An enterprise requires that all cryptographic keys used to encrypt customer database backups in Cloud Storage be automatically rotated every 90 days. What happens to existing backups encrypted with older key versions when a Cloud KMS key is rotated?
- A. The older backups are deleted automatically.
- B. The older backups remain accessible because Cloud KMS retains older key versions to allow decryption of existing ciphertexts, while the new key version is used for all subsequent encryption operations.
- C. The backups must be manually downloaded and re-encrypted within 24 hours.
- D. Cloud Storage throws an access denied error.

---

### Question 10 (Domain 5: Mountkirk Games Case Study)
Mountkirk Games is running a global gaming event. A sudden network routing glitch between Europe and North America causes high packet loss. The matchmaking microservice in Europe is unable to reach the primary user profile database in North America. What architectural design pattern allows players in Europe to continue playing without crashing the game client?
- A. Hard crash with an unhandled exception.
- B. Graceful degradation: serve cached player profile data from a local European cache (Memorystore / local replica) in read-only mode, queue match results locally, and sync back when the connection restores.
- C. Delete all European player accounts.
- D. Infinite synchronous retries that freeze the mobile UI.

---

### Question 11 (Domain 2: Hierarchical Firewall Policies)
An enterprise security director wants to block all inbound traffic on port 23 (Telnet) and port 21 (FTP) across all current and future projects in the organization. The rule must be enforced permanently and cannot be overridden or deleted by project-level administrators. How should this firewall policy be implemented?
- A. Create VPC firewall rules in each project manually.
- B. Create a Hierarchical Firewall Policy at the Organization root level with a rule that denies ingress on ports 21 and 23.
- C. Ask developers not to install Telnet.
- D. Run an antivirus scanner on each VM.

---

### Question 12 (Domain 4: Storage Transfer Service vs gsutil)
A data team needs to migrate 80 Terabytes of genomic data from an on-premises POSIX file server to Cloud Storage over an existing 10 Gbps Cloud Interconnect. The transfer must support parallel multi-threaded chunking, bandwidth rate limiting during business hours, and automated failure retries. Why should Storage Transfer Service be chosen over manual `gsutil` / `gcloud storage` scripts?
- A. `gsutil` does not support Cloud Storage.
- B. Storage Transfer Service is a managed, enterprise-grade service that handles scheduling, checkpointing, bandwidth throttling, and automatic retries without maintaining custom script infrastructure.
- C. Storage Transfer Service is free; `gsutil` is paid.
- D. `gsutil` only works on Windows.

---

### Question 13 (Domain 5: Disaster Recovery Testing)
An SRE team wants to verify that their multi-region active-active deployment can withstand a complete region loss without customer disruption. During a scheduled game day exercise, how should the team safely simulate the regional failure of `us-central1` at the load balancing tier?
- A. Delete all Compute Engine instances in `us-central1`.
- B. Update the Global External HTTPS Load Balancer backend service to set the capacity scaler / max utilization of the `us-central1` backend group to 0, observing Anycast traffic reroute to the healthy alternate region.
- C. Cut the physical fiber cable.
- D. Delete the GCP project.

---

### Question 14 (Domain 1: Bigtable Row Key Design)
A connected vehicle company designs a Cloud Bigtable schema to store sensor readings. The row key is designed as `TIMESTAMP#VEHICLE_ID`. During peak driving hours, writes slow down significantly and write latencies jump to 2,000ms. Why is this occurring, and how should the row key be redesigned?
- A. Bigtable cannot handle sensor data; migrate to Cloud SQL.
- B. Sequential timestamp prefixes cause all writes to hit a single Bigtable tablet server (hotspotting); redesign the row key to `VEHICLE_ID#TIMESTAMP` or prefix with a hash/salt to distribute writes across all cluster nodes.
- C. The cluster needs to be rebooted.
- D. Bigtable requires all data to be stored in a single row.

---

### Question 15 (Domain 3: Sensitive Data Protection)
A retail company stores raw customer chat logs in Cloud Storage. To comply with GDPR, customer credit card numbers and passport details must be purged or tokenized from the logs. Which automated architecture fulfills this compliance requirement?
- A. Sensitive Data Protection (Cloud DLP) triggered via Cloud Functions to scan, de-identify, and mask sensitive info types before moving sanitized logs to the analytics bucket.
- B. Manual inspection by customer service reps.
- C. Cloud Armor.
- D. Changing bucket permissions to public.

---

## Answer Key & Rationales

1. **B** — Global External HTTPS Load Balancers use Anycast global IPs to direct traffic to healthy backends with sub-second failover, bypassing DNS delays.
2. **B** — Transitioning through Standard -> Nearline (30d) -> Coldline (90d) -> Archive (365d) matches access drop-offs to storage pricing tiers.
3. **B** — Continuous CDC replication via Dedicated Interconnect maintains a local cloud read replica, delivering low-latency queries without altering the on-prem DB.
4. **B** — The default service account with `Editor` privileges violates least privilege; custom service accounts restrict access to necessary operations.
5. **B** — Cloud NAT provides managed outbound internet connectivity for private instances without allocating public IP addresses.
6. **A** — Pub/Sub ingestion, Dataflow streaming analytics, Vertex AI inference, and serverless execution form a real-time predictive maintenance pipeline.
7. **B** — The "Retire" strategy decommissions legacy, redundant applications that no longer serve a business purpose.
8. **B** — The Kubernetes Gateway API is the modern standard for advanced ingress traffic routing, canary splitting, and multi-namespace routing on GKE.
9. **B** — Cloud KMS retains prior key versions to ensure existing ciphertexts can be decrypted, while utilizing the latest primary key for new encryptions.
10. **B** — Graceful degradation ensures business continuity by falling back to local cached data and queuing mutations during transient network partitions.
11. **B** — Hierarchical Firewall Policies enforce organization-wide, immutable security rules that cannot be overridden by project-level administrators.
12. **B** — Storage Transfer Service automates large-scale data transfers with built-in checkpointing, scheduling, and error handling without custom scripts.
13. **B** — Setting backend capacity to zero at the load balancer safely validates global Anycast failover routing without destroying infrastructure.
14. **B** — Monotonically increasing timestamp prefixes cause write hotspotting on single tablets; prefixing with `VEHICLE_ID` distributes writes across nodes.
15. **A** — Cloud DLP natively inspects unstructured text and redacts sensitive PII/credit card data, ensuring GDPR compliance.
