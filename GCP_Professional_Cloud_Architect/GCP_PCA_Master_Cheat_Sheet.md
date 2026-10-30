# Google Cloud Professional Cloud Architect (PCA) — Master Cheat Sheet

> [!IMPORTANT]
> **Exam Strategy:** The PCA exam tests trade-offs between managed Google Cloud services, cost efficiency, high availability ($99.99\%$ vs. $99.999\%$), zero-trust security, and business continuity. Remember Google's core design philosophy: **prefer managed serverless over IaaS**, **prefer global over regional where applicable**, **enforce least privilege via IAM**, and **never expose compute instances to public IPs unless strictly necessary**.

---

## 1. Compute Platform Decision Matrix

```
Requirement                                            Recommended Google Cloud Compute
─────────────────────────────────────────────────────────────────────────────────────────────
Event-driven serverless functions (<60 min timeout)    Cloud Run functions (2nd gen)
Stateless containerized HTTP/gRPC services             Cloud Run (Fully Managed Serverless)
Stateful containers, complex microservices, vCluster   Google Kubernetes Engine (GKE Autopilot)
High-performance gaming UDP servers (Agones)           GKE Standard (Custom CNI, Node control)
Legacy OS kernels, Windows Server, custom kernel mods  Compute Engine (MIG with Auto-healing)
High Performance Computing (HPC), GPU/TPU clusters     Compute Engine (A3/G2 GPU Instances)
Bare metal, VMware vSphere lift-and-shift              Google Cloud VMware Engine (GCVE)
```

---

## 2. Storage & Database Selection Matrix

* **Cloud Storage (GCS)**
  * **Data Model:** Unstructured Object (REST API).
  * **Scalability:** Exabytes; automatic scaling; strong read-after-write consistency.
  * **High Availability:** Multi-Region / Dual-Region ($99.99\%$).
  * **Exam Use Case:** Media streaming, raw batch data lake, machine learning training data, backups.

* **Cloud SQL**
  * **Data Model:** Managed Relational (MySQL, PostgreSQL, SQL Server).
  * **Scalability:** Up to 64 TB storage, 96 vCPUs, up to 10 read replicas.
  * **High Availability:** Regional HA with synchronous persistent disk replication and auto-failover in 60–120s ($99.95\%$).
  * **Exam Use Case:** Standard enterprise OLTP applications, CMS backends, departmental databases.

* **AlloyDB for PostgreSQL**
  * **Data Model:** Fully managed PostgreSQL-compatible relational database with decoupled compute and storage.
  * **Scalability:** Up to 4x faster for transactions and 100x faster for analytical queries than standard PostgreSQL.
  * **High Availability:** Non-disruptive failover in $<10\text{ seconds}$ ($99.99\%$).
  * **Exam Use Case:** Modernizing demanding on-premises Oracle and SQL Server workloads onto PostgreSQL.

* **Cloud Spanner**
  * **Data Model:** Horizontally scalable NewSQL with ANSI SQL and TrueTime external consistency.
  * **Scalability:** Petabytes; unlimited horizontal write scaling across multiple zones and regions.
  * **High Availability:** Multi-Region configuration with **99.999% SLA (Five 9s)** ($<5.26\text{ minutes}$ downtime/year).
  * **Exam Use Case:** Global financial ledgers, multinational retail inventory, core banking systems requiring absolute consistency and zero downtime.

* **Cloud Bigtable**
  * **Data Model:** High-throughput, wide-column NoSQL (Apache HBase API).
  * **Scalability:** Petabytes; sub-10ms read/write latency; linear scalability per node.
  * **High Availability:** Multi-cluster replication with automatic failover ($99.99\%$).
  * **Exam Use Case:** IoT time-series telematics, AdTech clickstreams, Mountkirk Games real-time leaderboards, fraud detection.

* **Cloud Firestore**
  * **Data Model:** Serverless NoSQL document database (Native Mode or Datastore Mode).
  * **Scalability:** Millions of concurrent mobile and web clients with live snapshot listeners.
  * **High Availability:** Multi-Region replication ($99.999\%$ SLA).
  * **Exam Use Case:** Mobile and web application state, user profiles, real-time gaming inventories.

* **Memorystore**
  * **Data Model:** Fully managed in-memory key-value data store (Redis / Memcached).
  * **Scalability:** Up to 300 GB memory; sub-millisecond latency.
  * **High Availability:** Regional HA with automatic failover and read replicas.
  * **Exam Use Case:** Web session caching, high-speed rate limiting, leaderboard caches.

* **BigQuery**
  * **Data Model:** Serverless columnar enterprise data warehouse (ANSI SQL).
  * **Scalability:** Petabytes; separate compute (slots) and storage; built-in ML (BigQuery ML).
  * **Pricing Models:** On-Demand (\$6.25/TB scanned) or BigQuery Editions (Standard, Enterprise, Enterprise Plus with slot autoscaling).
  * **Exam Use Case:** Enterprise BI reporting, historical telemetry analysis, machine learning feature stores.

---

## 3. Networking, Hybrid Connectivity & Edge Architecture

### 3.1 VPC Topologies

* **Shared VPC:** Central **Host Project** manages network, subnets, and firewalls. Attached **Service Projects** deploy compute resources into specific subnets via `roles/compute.networkUser`. Used for centralized governance within a single Google Cloud Organization.
* **VPC Network Peering:** Direct RFC 1918 connectivity between two distinct VPCs. **Strictly non-transitive** (VPC A $\leftrightarrow$ B and B $\leftrightarrow$ C does NOT allow A $\leftrightarrow$ C). No overlapping CIDRs allowed.
* **Network Connectivity Center (NCC):** Hub-and-spoke WAN connecting on-premises data centers, branch offices (via Router Appliance), and VPC spokes using Google's global fiber network as the corporate transit backbone.

### 3.2 Private Service Integrations

* **Private Google Access (PGA):** Allows private VMs (without external public IPs) to reach Google APIs via internal DNS pointing to `restricted.googleapis.com` (`199.36.153.8/30`) or `private.googleapis.com`.
* **Private Services Access (PSA):** Peering connection between customer VPC and Google-managed service VPC (used for Cloud SQL and Memorystore). Consumes customer RFC 1918 IP space; non-transitive.
* **Private Service Connect (PSC):** Modern endpoint architecture projecting Google APIs, partner services, or internal producer services as local private IP addresses inside the consumer's subnet. Overcomes VPC Peering non-transitivity and supports overlapping CIDRs.

### 3.3 Hybrid Connectivity & Cloud Router

* **HA VPN:**
  * **SLA:** **99.99% availability** with dual interfaces and 2 or 4 tunnels.
  * **Routing:** Strictly requires **Dynamic BGP Routing** via Cloud Router. Static routing is unsupported.
  * **Throughput:** 3 Gbps per tunnel.
* **Dedicated Interconnect:**
  * **99.9% SLA:** 2 connections in 1 colocation facility (metro) terminating on 2 Google Edge Routers.
  * **99.99% SLA:** **4 connections across 2 separate colocation facilities (2 distinct metros)** terminating on 4 Google Edge Routers, connected to 2 Cloud Routers in GCP.
* **Active-Passive BGP Failover:** Set lower **MED (Multi-Exit Discriminator)** or shorter AS-Path on the primary on-premises router (e.g., primary MED `100`, backup MED `200`).

### 3.4 Google Cloud Load Balancing (GCLB) Quick Guide

* **Global External Application LB:** Layer 7 Envoy proxy with Anycast global IP; SSL offloading, URL routing, Cloud Armor WAF, Cloud CDN.
* **Regional External Application LB:** Layer 7 regional proxy for strict regional data residency compliance.
* **Internal Application LB:** Layer 7 internal proxy for microservices routing and canary traffic splitting.
* **External Passthrough Network LB:** Layer 4 Maglev non-proxied load balancer; preserves client source IP; Direct Server Return (DSR); ideal for UDP game servers and high-throughput streaming.
* **Internal Passthrough Network LB:** Layer 4 internal non-proxied load balancer; can act as next-hop for high-availability firewall appliances.

---

## 4. Identity, Security & Compliance Quick Reference

* **Workload Identity Federation (WIF):** Authenticates AWS, Azure, GitHub Actions, and Kubernetes workloads to Google Cloud APIs via OIDC tokens and STS token exchange. **Never download service account JSON keys!**
* **Organization Policies:**
  * `constraints/iam.disableServiceAccountKeyCreation`: Prevents creation of downloadable service account keys.
  * `constraints/compute.vmExternalIpAccess`: Restricts assignment of external public IPs.
  * `constraints/gcp.resourceLocations`: Enforces geographic data residency.
* **Encryption Hierarchy:**
  * **Default Encryption:** AES-256 with Google-managed keys.
  * **CMEK (Cloud KMS):** Customer manages Key Ring and CryptoKey rotation (e.g., 90 days); can revoke access or destroy keys instantly to shred data.
  * **CSEK:** Customer passes raw 256-bit AES keys in API headers; Google never stores keys on disk; data lost if key is forgotten.
  * **Cloud EKM:** Keys stored in customer on-premises third-party HSM.
* **VPC Service Controls (VPC-SC):** Security perimeter around multi-tenant Google APIs (GCS, BigQuery) preventing data exfiltration even if IAM credentials are stolen. Always test in **Dry-Run mode** before enforcing.
* **Identity-Aware Proxy (IAP):** BeyondCorp zero-trust proxy for HTTPS web apps and TCP SSH/RDP administrative forwarding (`gcloud compute ssh --tunnel-through-iap`) without public IPs or bastion hosts.
* **Access Transparency & Access Approval:**
  * **Access Transparency:** Real-time audit logs of Google engineers accessing customer data during support tickets.
  * **Access Approval:** Requires explicit customer cryptographic approval before Google personnel can access data.

---

## 5. Official PCA Case Studies Cram Sheet

```
Case Study               Core Technology Stack & Architectural Decisions
─────────────────────────────────────────────────────────────────────────────────────────────
EHR Healthcare           Dedicated Interconnect (99.99%) + Cloud HA VPN backup;
                         Apigee X (modernize legacy SOAP to secure RESTful APIs);
                         GKE Autopilot (HIPAA/PCI-DSS compliant microservices);
                         Cloud Spanner (clinical records multi-region availability);
                         Cloud DLP (PHI de-identification) -> BigQuery research;
                         CMEK with Cloud KMS; VPC-SC perimeter; Access Transparency.

Mountkirk Games          GKE Standard + Agones Game Server Controller (dedicated UDP);
                         External Passthrough Network LB (UDP line-rate Maglev);
                         Spot VMs in secondary node pools for dynamic game match scaling;
                         Cloud Spanner (player profiles, virtual currency, ACID purchases);
                         Cloud Bigtable (sub-10ms global leaderboards & game state);
                         Pub/Sub -> Dataflow (real-time cheat detection) -> BigQuery.

TerramEarth              Dual Ingestion: 200k connected vehicles publish to Pub/Sub;
                         19.8M remote vehicles dump batch sensor data to dealer depots;
                         Depots sync to GCS via Storage Transfer Service / Transfer Appliance;
                         Unified Cloud Dataflow pipeline (Beam) processes both streams;
                         Cloud Bigtable (time-series sensor telemetry, smart row keys);
                         BigQuery (historical sensor analytics) -> Vertex AI predictive maintenance.

Helicopter Racing League Global External Application LB + Cloud CDN (low-latency video streaming);
(HRL) / Cymbal Direct    Cloud Armor (WAF and Layer 7 DDoS / anti-bot mitigation);
                         GKE with GPU accelerator pools for real-time video telemetry AI;
                         Cloud Run + Firestore Native Mode for real-time fan live voting;
                         Cloud Spanner for official race timings and ledger balances.
```

---

## 6. Top 15 PCA Exam Traps & Anti-Patterns

1. **Service Account JSON Keys:** Any question suggesting "download a service account JSON key and store it on an AWS EC2 instance or CI/CD runner" is an **immediate distractor**. The correct answer is **Workload Identity Federation**.
2. **Bastion Hosts / Jump Boxes:** If an exam question asks how to provide secure administrative SSH/RDP access to private GCE VMs, the answer is **Identity-Aware Proxy (IAP) TCP forwarding (`--tunnel-through-iap`)**, NOT deploying a bastion host with a public IP.
3. **VPC Peering Transitivity:** Peered VPCs cannot route traffic through intermediate peered VPCs. If transitive routing is required, use **Network Connectivity Center (NCC)** or an **Internal Load Balancer with Hub-and-Spoke NVA**.
4. **Cloud VPN HA Routing:** HA VPN strictly requires **Dynamic BGP Routing via Cloud Router**. Static routes are unsupported.
5. **Bigtable Row Key Hotspotting:** Never use sequential IDs, sequential timestamps, or domain names as Bigtable row key prefixes. Always reverse domain names or prepend hashed identifiers (`tenant_id#metric#reverse_timestamp`).
6. **BigQuery SELECT \*:** Running unpartitioned `SELECT *` on petabyte datasets is a critical cost anti-pattern under On-Demand pricing. Enforce partition filters and cluster tables.
7. **Cloud Run Inbound Protocols:** Cloud Run only supports HTTP/HTTPS, HTTP/2, WebSocket, and gRPC. It does **NOT** support incoming raw TCP or UDP traffic (use GKE or Compute Engine for UDP).
8. **VPC Service Controls Rollout:** Never enforce a VPC-SC perimeter directly without testing in **Dry-Run mode** first to inspect audit logs for broken internal API dependencies.
9. **Single Cloud SQL Instance for 99.99% Availability:** Standard Cloud SQL HA provides $99.95\%$ SLA. If $99.999\%$ (five 9s) is required, use **Cloud Spanner Multi-Region**.
10. **Storage Transfer Service vs. Transfer Appliance:** Use Storage Transfer Service if dedicated bandwidth can complete the transfer within days. Use **Transfer Appliance** for offline migration of $>100\text{ TB}$ when network bandwidth is constrained ($<1\text{ Gbps}$).
11. **GCS Object Lock:** Bucket Lock retention policies are immutable once locked. Even Google Cloud Support cannot delete locked buckets or shorten retention periods.
12. **Shared VPC IAM:** Never grant `compute.networkAdmin` in the Host Project to service project teams. Grant `compute.networkUser` only on specific designated subnets.
13. **Cloud NAT Incoming Traffic:** Cloud NAT only allows **outbound-initiated** connections from private VMs to the internet. It does not allow external internet clients to initiate inbound connections to private VMs.
14. **Dedicated Interconnect 99.99% SLA:** Requires **4 connections across 2 separate metros** terminating on 4 Google Edge Routers. Configuring connections in a single metro only provides $99.9\%$ SLA.
15. **BigQuery Editions Slot Management:** For unpredictable bursty query workloads, configure **Autoscaling Reservations** with a baseline number of slots plus maximum burst slots.
