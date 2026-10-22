# Domain 5: Reliability, Operations & Official Case Studies — Study Guide

**Exam:** Google Cloud Certified: Professional Cloud Architect (PCA)  
**Weight:** ~21% of the certification curriculum

---

## 1. Official Google Cloud Case Studies Architecture Blueprint

### A. EHR Healthcare
- **Company Profile:** Leading healthcare provider managing Electronic Medical Records (EMR), patient portals, and insurance claims.
- **Business Challenges:** Legacy on-premises architectures, slow feature release cycles, strict HIPAA / PCI-DSS compliance mandates, need for open APIs for insurance partners.
- **Architectural Solution:**
  - **Hybrid Connectivity:** Dual 10 Gbps Dedicated Interconnect circuits across redundant metros with Cloud HA VPN backup.
  - **API Governance:** Apigee API Management to expose secure, rate-limited RESTful APIs while transforming legacy SOAP backends.
  - **Compute Platform:** GKE Autopilot for containerized HIPAA-compliant microservices.
  - **Security & Compliance:** Customer-Managed Encryption Keys (CMEK) via Cloud KMS, VPC Service Controls perimeters around BigQuery/Cloud Storage, Cloud DLP for automated PHI de-identification.

### B. Cymbal Direct
- **Company Profile:** Heritage department store transitioning to a high-growth Direct-to-Consumer (DTC) digital retail platform.
- **Business Challenges:** Extreme seasonal traffic volatility (10x-20x during Black Friday / Cyber Monday), inventory synchronization race conditions, international expansion.
- **Architectural Solution:**
  - **Database Tier:** Cloud Spanner multi-region instance for global inventory management and ACID checkout transactions with $99.999\%$ availability.
  - **Compute Tier:** Cloud Run for stateless frontend APIs; GKE Autopilot for asynchronous order processing and fulfillment microservices.
  - **Edge & Performance:** Global External HTTP(S) Load Balancer paired with Cloud CDN for static media caching and Cloud Armor for Layer 7 WAF/anti-bot protection.
  - **Analytics & Personalization:** Event stream ingestion via Pub/Sub -> BigQuery, driving Vertex AI real-time recommendation models.

### C. Mountkirk Games
- **Company Profile:** Global game studio developing a hit multiplayer mobile action title with fast-paced real-time combat.
- **Business Challenges:** Unpredictable player adoption spikes, low-latency UDP multiplayer networking requirements, dynamic matchmaking, real-time analytics.
- **Architectural Solution:**
  - **Game Server Hosting:** GKE Standard with open-source **Agones** game server orchestrator managing dedicated UDP container processes with custom node pools.
  - **Persistent State & Economy:** Cloud Spanner for player accounts, virtual currency, and transaction history.
  - **Leaderboards & Session State:** Cloud Bigtable for high-throughput, sub-10ms leaderboard lookups and real-time player game state.
  - **Telemetry Pipeline:** Cloud Pub/Sub -> Cloud Dataflow -> BigQuery for game telemetry, cheat detection, and live-event monetization analytics.

### D. TerramEarth
- **Company Profile:** Global manufacturer of heavy agricultural and mining machinery operating 20 million connected vehicles worldwide.
- **Business Challenges:** Vehicles operate in remote environments with intermittent cellular/satellite links; 200 petabytes of sensor data collected annually; need for predictive vehicle maintenance to minimize field breakdown downtime.
- **Architectural Solution:**
  - **Telemetry Ingestion:** Edge devices buffer readings locally; opportunistic uploads via Cloud Pub/Sub when cellular/satellite connectivity is acquired.
  - **Bulk Ingestion:** Storage Transfer Service and Google Transfer Appliance for legacy field dealer bulk sensor dumps.
  - **Stream Processing:** Cloud Dataflow for windowing, deduplication, and anomaly detection.
  - **Storage Architecture:** Cloud Bigtable for time-series sensor ingestion; BigQuery for historical machine analytics and Vertex AI predictive maintenance training.

---

## 2. Disaster Recovery & High Availability Strategies

| Strategy | RTO Target | RPO Target | Cost Index | Architectural Blueprint |
| :--- | :--- | :--- | :--- | :--- |
| **Cold Standby** | Hours to Days | Hours | $\$$ | Automated backups and VM snapshots stored in Multi-Region Cloud Storage; Terraform pipelines provision infrastructure on disaster declaration. |
| **Warm Standby** | Minutes to Hours | Minutes | $\$ \$$ | Minimal core infrastructure running in secondary region (e.g. 1-node database read replica, MIG minimum size 1); auto-scales up on primary failure. |
| **Hot Standby (Active-Passive)** | Seconds to Minutes | Near Zero | $\$ \$ \$$ | Full production replica running in secondary region; synchronous database replication; automated DNS / Load Balancer failover. |
| **Active-Active (Multi-Region)** | Zero | Zero | $\$ \$ \$ \$$ | Traffic served concurrently across multiple regions via Global External HTTPS Load Balancer with Anycast routing and Cloud Spanner multi-region. |

---

## 3. Reliability Engineering Patterns in System Architecture
- **Graceful Degradation:** When downstream databases experience saturation, non-essential features (e.g. recommendation carousels) return fallback defaults without failing the core checkout workflow.
- **Circuit Breaker Pattern:** Temporarily cuts off calls to an unresponsive dependency after a failure threshold, returning immediate fallback errors and preventing thread exhaustion.
- **Thundering Herd & Jitter:** Exponential backoff with randomized jitter prevents retry storms from crashing recovering databases.
