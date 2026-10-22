# Google Cloud Professional Cloud Architect (PCA) — Master Cheat Sheet

---

## 1. Compute Selection Decision Matrix

```
Requirement                                            Recommended Compute
─────────────────────────────────────────────────────────────────────────────────────────────
Event-driven micro-functions (<60 min timeout)        Cloud Functions / Cloud Run functions
Stateless containerized HTTP/gRPC services             Cloud Run (Fully Managed Serverless)
Complex multi-container orchestration, stateful sets   Google Kubernetes Engine (GKE Autopilot)
High-performance gaming servers (UDP, Agones)          GKE Standard (Custom CNI, Node control)
Legacy OS kernels, Windows Server, custom kernel mods  Compute Engine (MIG with Auto-healing)
High Performance Computing (HPC), GPU clusters         Compute Engine (A3/G2 GPU Instances)
Bare metal, VMware workloads lift-and-shift            Google Cloud VMware Engine (GCVE)
```

---

## 2. Storage & Database Selection Matrix

| Service | Data Model | Scalability | Consistency | High Availability | Typical Use Case |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Cloud Storage** | Unstructured Object | Exabytes, auto-scale | Strong consistency | Multi-Region / Dual-Region (99.99%) | Images, videos, backups, data lake |
| **Cloud SQL** | Relational (MySQL, Postgres, SQL Server) | Up to 64 TB, 96 vCPUs | ACID (Single primary) | Regional HA with auto-failover | Standard OLTP web apps, ERP systems |
| **Cloud Spanner** | Relational with SQL | Petabytes, horizontal scaling | Strong external consistency (TrueTime) | Multi-Region 99.999% SLA (Five 9s) | Global finance, ledger, multi-region retail |
| **Cloud Bigtable** | Wide-column NoSQL | Petabytes, linear scaling | Strong (single cluster), Eventual (replicated) | Multi-cluster replication (99.99%) | IoT time-series, AdTech, Mountkirk telemetry |
| **Firestore** | Document NoSQL | Millions of concurrent clients | Strong consistency | Multi-Region (99.999%) | Mobile app backend, user profiles, real-time sync |
| **Memorystore** | In-memory key-value (Redis/Memcached) | Up to 300 GB memory | Sub-millisecond latency | High Availability with read replicas | Session storage, caching, leaderboard |
| **BigQuery** | Columnar Relational OLAP | Petabyte to Exabyte | ACID | Regional or Multi-Region | Data warehousing, real-time SQL analytics |

---

## 3. Hybrid Networking & Connectivity Decision Tree

```
Connectivity Need                                    Recommended Solution
─────────────────────────────────────────────────────────────────────────────────────────────
Low bandwidth (<3 Gbps), encrypted over public net   Cloud HA VPN (IPsec, 99.99% SLA, Dynamic BGP)
High bandwidth (>10 Gbps), mission-critical, SLA     Dedicated Interconnect (10G/100G circuits)
No colocation facility presence, service provider    Partner Interconnect (50 Mbps to 50 Gbps)
Private consumption of SaaS / GCP producer APIs      Private Service Connect (PSC endpoints)
Private access to Google APIs from private RFC1918   Private Google Access (Internal IP routing)
Cross-VPC communication without gateway / NAT        VPC Network Peering (Non-transitive)
Centralized enterprise network administration        Shared VPC (Host project + Service projects)
```

---

## 4. Disaster Recovery Strategies (RPO vs RTO)

| Strategy | RTO (Recovery Time Objective) | RPO (Recovery Point Objective) | Cost | Architecture Mechanics |
| :--- | :--- | :--- | :--- | :--- |
| **Cold Standby** | Hours to Days | Hours | $\$$ | Snapshots and backups stored in Cloud Storage; infrastructure deployed via Terraform on disaster. |
| **Warm Standby** | Minutes to Hours | Minutes | $\$\$$ | Minimal footprint running in secondary region (e.g. 1-node database replica, idle MIGs scaled to 1). Scaled up on failover. |
| **Hot Standby (Active-Passive)** | Seconds to Minutes | Near Zero | $\$\$\$$ | Full replica running in secondary region; synchronous replication (Spanner or regional DB); automated DNS/LB failover. |
| **Multi-Region (Active-Active)** | Zero | Zero | $\$\$\$\$$ | Services active in both regions behind Global External HTTPS Load Balancer; Cloud Spanner multi-region; traffic routed Anycast. |

---

## 5. Official Case Studies Executive Synthesis

### 1. EHR Healthcare
- **Business Goal:** Modernize legacy insurance and Electronic Medical Record (EMR) systems, achieve HIPAA/PCI compliance, build API platform for healthcare providers.
- **Key Architectures:** Apigee API Management for legacy SOAP/REST transformation; Cloud Interconnect to on-premises data centers; GKE for containerized microservices; Cloud Storage with Customer-Managed Encryption Keys (CMEK); IAM with Cloud Identity.

### 2. Cymbal Direct
- **Business Goal:** Direct-to-Consumer (DTC) digital retail expansion; support 10x traffic spikes during seasonal flash sales; real-time personalized recommendations.
- **Key Architectures:** Cloud Spanner for globally consistent inventory and checkout; Cloud Run & GKE Autopilot for microservices; BigQuery and Vertex AI for personalized recommendation engine; Global External HTTP(S) Load Balancer with Cloud CDN and Cloud Armor.

### 3. Mountkirk Games
- **Business Goal:** Next-generation global mobile multiplayer game; dynamic low-latency matchmaking; rapid ingestion of telemetry.
- **Key Architectures:** Agones open-source game server management on GKE Standard; Cloud Spanner for player accounts and economy; Cloud Bigtable for real-time leaderboards and player game state; Pub/Sub -> Dataflow -> BigQuery for telemetry analytics.

### 4. TerramEarth
- **Business Goal:** Connected global agriculture and mining fleet (20M vehicles); capture telemetry over low-bandwidth cellular/satellite; predictive vehicle maintenance.
- **Key Architectures:** Cloud IoT Core / MQTT ingestion via Cloud Pub/Sub; Dataflow for stream processing and deduplication; Bigtable for high-throughput time-series sensor ingestion; BigQuery and Vertex AI for predictive maintenance models; Storage Transfer Service for bulk batch data imports.
