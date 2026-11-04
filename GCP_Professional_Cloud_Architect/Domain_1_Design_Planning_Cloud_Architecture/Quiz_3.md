# Domain 1: Designing and Planning a Cloud Solution Architecture — Quiz 3

**Exam:** Google Cloud Certified: Professional Cloud Architect (PCA)  
**Format:** 10 Scenario-Based Multiple-Choice Questions with In-Depth Technical Explanations

---

### Question 1
A media streaming service is migrating its monolithic video processing service to Google Cloud. The system has three main components: an upload receiver, an audio transcoder, and a video thumbnail generator. The transcoder and thumbnail generator have wildly different processing speeds and resource profiles. How should the architect structure the system?
- A. Store everything in a single SQL database table.
- B. Decouple the components into microservices communicating asynchronously via Cloud Pub/Sub topics, allowing each component to scale independently on Cloud Run or GKE.
- C. Run all three services in a single large Compute Engine VM.
- D. Synchronously call each component via HTTP inside a single thread.

---

### Question 2
An enterprise wants to deploy an AI inference service that requires Nvidia L4 GPUs. The application must scale from 1 to 50 GPU instances automatically based on inference queue depth in Pub/Sub. Which compute platform natively supports automated GPU node pool autoscaling?
- A. App Engine Standard
- B. Cloud Spanner
- C. Cloud Functions
- D. Google Kubernetes Engine (GKE) with GPU Node Pools and KEDA (Kubernetes Event-driven Autoscaling)

---

### Question 3
A financial services firm runs an international currency exchange service. They must ensure that users in Tokyo, London, and New York always experience under 50ms read latency for exchange rates, while updates made in London must be strongly consistent across all locations immediately. Which architecture should the architect propose?
- A. Cloud SQL in London with asynchronous cross-region read replicas
- B. Memcached in each region without synchronization
- C. Cloud Spanner with a multi-region instance configuration spanning Asia, Europe, and North America
- D. Cloud Storage Nearline

---

### Question 4
An architect is designing an event-driven workflow where an upload of a medical scan PDF to Cloud Storage must trigger a text extraction pipeline, update a database record, and send an email notification to the doctor. What serverless coordination service should orchestrate these sequential steps with retry logic?
- A. Cloud DNS
- B. Cloud Workflows
- C. Cloud Shell
- D. Compute Engine Startup Scripts

---

### Question 5
A data platform ingests streaming sensor data from 500,000 electric vehicles into BigQuery. The data team wants to ensure that analytical queries analyzing data from the last 7 days do not scan the entire multi-year historical dataset. What BigQuery configuration is essential?
- A. Increasing user IAM permissions
- B. Ingestion-time or event-timestamp Partitioning on the BigQuery table
- C. Exporting data to Google Drive daily
- D. Converting all columns to strings

---

### Question 6
An architect needs to implement a distributed cache for an e-commerce website deployed on GKE. The cache must survive cluster node restarts, provide high availability, and support Redis cluster sharding across multiple nodes. Which service should be provisioned?
- A. Cloud Storage Coldline
- B. Bigtable
- C. Local ephemeral SSD on worker nodes
- D. Cloud Memorystore for Redis Cluster

---

### Question 7
A company is developing a collaborative real-time mobile whiteboard application where thousands of users sketch and collaborate simultaneously. The application requires real-time document synchronization, offline caching on mobile devices, and automatic multi-region data replication. Which database should be selected?
- A. Persistent Disk
- B. Cloud Bigtable
- C. Cloud SQL MySQL
- D. Firestore in Native mode

---

### Question 8
When choosing between GKE Standard and GKE Autopilot, what is a valid technical reason for selecting GKE Standard?
- A. The workload requires privileged containers, custom Linux kernel modules, specialized CNI network plugins, or specific hardware daemonsets not permitted under Autopilot security baselines.
- B. GKE Standard does not require Kubernetes manifests.
- C. GKE Autopilot does not support containers.
- D. GKE Standard is always cheaper than Autopilot.

---

### Question 9
A logistics company tracks parcel locations across 50 distribution hubs. The system processes 10,000 barcode scan events per second. The architect must ensure that scan events are never lost if the backend processing microservice experiences temporary downtime. What architecture provides this durability buffer?
- A. Writing scan events to temporary local text files
- B. Sending scan events over UDP
- C. Ingesting scan events into Cloud Pub/Sub, which persists messages across multiple zones for up to 7 days until acknowledged
- D. Direct HTTP POST from scanners to the backend microservice

---

### Question 10
An architect is reviewing an application where a Compute Engine VM writes application logs to a local disk file that fills up and crashes the instance weekly. What is the cloud-native architectural remediation?
- A. Install the Google Cloud Ops Agent on the VM to stream logs directly to Google Cloud Logging, configure log retention policies, and enable log-based alerting.
- B. Purchase a larger 10 TB persistent disk.
- C. Reboot the VM every night using a cron job.
- D. Disable all application logging.

---

## Answer Key & Rationales

1. **B** — Decoupling components with Cloud Pub/Sub buffers load differences and allows independent autoscaling of each microservice.
2. **D** — GKE with GPU node pools and KEDA provides event-driven horizontal autoscaling of GPU workloads based on Pub/Sub metrics.
3. **C** — Cloud Spanner multi-region instance configurations provide globally synchronized, strongly consistent reads and writes with low regional latency.
4. **B** — Cloud Workflows orchestrates serverless microservices and APIs with built-in retries, branching, and state tracking.
5. **B** — Partitioning restricts BigQuery scan operations to specific daily partitions, avoiding full-table scans and lowering query costs.
6. **D** — Memorystore for Redis Cluster delivers managed high availability, automated sharding, and multi-terabyte in-memory caching.
7. **D** — Firestore Native mode offers built-in client SDKs with real-time listeners, offline synchronization, and multi-region replication.
8. **A** — GKE Autopilot enforces strict security baselines; workloads needing custom host kernel modules or privileged mode require GKE Standard.
9. **C** — Cloud Pub/Sub provides durable multi-zone message retention (up to 7 days by default, configurable to 31), decoupling producers from consumer availability.
10. **A** — Streaming logs via Ops Agent to Cloud Logging decouples telemetry from VM disk capacity and enables centralized analysis and alerting.
