# Domain 1: Designing and Planning a Cloud Solution Architecture — Study Guide

**Exam:** Google Cloud Certified: Professional Cloud Architect (PCA)  
**Weight:** ~24% of the certification curriculum

---

## 1. Compute Architecture & Workload Selection
- **Compute Engine (IaaS):**
  - **Managed Instance Groups (MIGs):** Stateless vs Stateful MIGs. Auto-healing with health checks, auto-scaling based on CPU utilization, Cloud Monitoring metrics, or load balancing capacity. Regional MIGs provide multi-zone fault tolerance.
  - **Machine Types:** E2 (cost-effective general purpose), N2/N2D (standard balanced), C3/C3D (compute-optimized with Intel Sapphire Rapids / AMD Genoa), M3 (memory-optimized for SAP HANA/in-memory databases), A3/G2 (GPU acceleration for LLMs and deep learning).
- **Google Kubernetes Engine (GKE):**
  - **GKE Standard vs Autopilot:** Autopilot manages node provisioning, scaling, OS patching, and security hardening with per-pod resource billing. Standard provides complete control over node pools, custom kernels, and specialized daemonsets.
  - **Gateway API & Service Mesh:** Standardized routing, multi-cluster services (MCS), traffic splitting, mTLS security.
- **Serverless Options:**
  - **Cloud Run:** Fully managed container runtime. Automatically scales down to zero and up to 1,000+ instances in seconds. Supports HTTP, gRPC, and WebSockets. Concurrency tuning (`concurrency: 80` default).
  - **Cloud Run functions (formerly Cloud Functions):** Event-driven execution triggered by Cloud Storage, Pub/Sub, Firestore, or HTTP webhooks.

---

## 2. Storage & Database Selection Patterns
- **Cloud Spanner:**
  - Vertically and horizontally scalable relational database offering strong external consistency (serializability) across multi-regional topologies.
  - Leverages Google's proprietary TrueTime API (atomic clocks and GPS receivers) to order distributed transactions without locking coordinator bottlenecks.
  - Provides five 9s ($99.999\%$) availability for multi-region configurations.
- **Cloud SQL:**
  - Fully managed MySQL, PostgreSQL, and SQL Server.
  - High Availability (HA) configuration uses synchronous replication to a standby instance in an alternate zone within the same region with automatic sub-minute failover.
  - Scalability limit: Up to 96 vCPUs, 624 GB RAM, 64 TB storage. Use Cloud Spanner when scale exceeds these limits.
- **Cloud Bigtable:**
  - Massively scalable, low-latency NoSQL wide-column store for high-throughput reads and writes (single-digit millisecond latency).
  - Key design: Row key design determines data distribution. Avoid sequential row keys (timestamps at the front) to prevent hotspotting; use hashed prefixes or reversed domain names.
  - Multi-cluster replication enables global low-latency reads and automatic failover.
- **Cloud Storage:**
  - Object storage with strong global consistency.
  - Storage Classes: Standard (hot data), Nearline (accessed < once/month, 30-day min), Coldline (accessed < once/quarter, 90-day min), Archive (accessed < once/year, 365-day min).
  - Dual-Region and Multi-Region buckets offer cross-region redundancy and automatic failover.

---

## 3. Data Pipelines & Messaging Architecture
- **Cloud Pub/Sub:**
  - Globally distributed, real-time asynchronous messaging service.
  - Guarantees at-least-once delivery; supports ordered messaging by ordering key.
  - Dead Letter Topics (DLT) for unprocessable poison-pill messages after $N$ retry attempts.
- **Cloud Dataflow (Apache Beam):**
  - Unified stream and batch processing service.
  - Exactly-once processing semantics.
  - Handles late-arriving data using watermarks, windowing (fixed, sliding, session), and triggers.
- **BigQuery:**
  - Serverless, highly scalable enterprise data warehouse.
  - Storage and compute are decoupled. Compute uses dynamic slot allocation (On-Demand or Editions: Standard, Enterprise, Enterprise Plus).
  - Partitioning (by ingestion time, timestamp, or integer range) and Clustering (up to 4 columns) optimize query performance and dramatically lower scanning costs.
