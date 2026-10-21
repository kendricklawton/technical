# PE Domain 8: System Design — 25 Core Interview Questions

Master these questions across System Design rounds (Mocks 5 and 6) and Staff/Principal Architecture loops.

---

## Tier 1: Core / Screening Questions (Questions 1–10)

### Q1: What is the difference between the Control Plane and the Data Plane?
- **Short Answer:** The Control Plane manages configuration, state reconciliation, and orchestration (e.g., Kubernetes API server, Argo CD, Portal). The Data Plane processes real user traffic and application workloads (e.g., microservices, Envoy proxies, databases).
- **Deep Dive:** A core platform design tenet is **Static Stability**: the Data Plane must continue operating with zero user degradation even if the Control Plane is completely offline.
- **Trade-offs & Realities:** Coupling the two (e.g., having application pods make synchronous calls to the control plane on every request) is a critical design failure.
- **Follow-up Probes:** "What happens to running pods if `kube-apiserver` goes down?" (Pods continue serving traffic, but autoscaling, scheduling, and rollouts pause).

### Q2: What is Little's Law, and how is it used in platform capacity estimation?
- **Short Answer:** $L = \lambda \times W$: The average number of items in a system ($L$) equals the arrival rate ($\lambda$) multiplied by the average time spent in the system ($W$).
- **Deep Dive:** Used to calculate required concurrency. Example: If CI receives 120 jobs/hour ($\lambda = 2 \text{ jobs/min}$) and jobs take 15 minutes ($W = 15$), you need $L = 2 \times 15 = 30$ concurrent runners to maintain steady state without queueing.
- **Trade-offs & Realities:** Assumes steady-state equilibrium; bursty traffic requires sizing capacity above $L$.
- **Follow-up Probes:** "What happens to the queue if arrival rate $\lambda$ exceeds processing capacity?" (Queue grows to infinity).

### Q3: What is the principle of Static Stability?
- **Short Answer:** Designing systems so that they continue operating normally during failures using pre-provisioned capacity and local cached configurations, without requiring reactive control plane interventions.
- **Deep Dive:** Instead of dynamically requesting resources during an outage, the system pre-warms capacity ($N/(N-1)$ across Availability Zones). Proxies cache routing tables locally so DNS or discovery service outages don't break routing.
- **Trade-offs & Realities:** Static stability increases baseline cloud spend because capacity is pre-provisioned rather than spun up on demand.
- **Follow-up Probes:** "Give an example of static stability in AWS." (Running sufficient EC2 instances across 3 AZs so that an entire AZ failure can be tolerated without needing auto-scaling).

### Q4: What is Shuffle Sharding, and how does it contain blast radius?
- **Short Answer:** A technique where each tenant is assigned a unique subset of workers (a shard) from a shared pool, drastically reducing the probability that a toxic tenant impacts another specific tenant.
- **Deep Dive:** With 8 workers assigned in pairs, there are 28 unique combinations. If Tenant A sends a malformed request that crashes its 2 workers, 26 out of the other 27 tenant pairings remain 100% operational.
- **Trade-offs & Realities:** Requires routing layers (Route53, Envoy) capable of hashing tenant requests to their specific worker combination.
- **Follow-up Probes:** "What is the formula for shuffle sharding combinations?" ($\binom{N}{K} = \frac{N!}{K!(N-K)!}$).

### Q5: What is an Idempotent API, and how is it implemented?
- **Short Answer:** An API where making the same request multiple times produces the identical result as making it once.
- **Deep Dive:** Implemented using an `Idempotency-Key` header. When a client sends a request, the server stores the key in an atomic database transaction. If a retried request arrives with the same key, the server returns the cached response rather than re-executing.
- **Trade-offs & Realities:** Idempotency keys must have a time-to-live (TTL, e.g., 24 hours) to avoid filling database storage.
- **Follow-up Probes:** "Which HTTP methods are naturally idempotent according to RFC 7231?" (GET, PUT, DELETE, HEAD, OPTIONS).

### Q6: What is a Cell-Based Architecture?
- **Short Answer:** An architectural pattern where an entire platform stack is partitioned into small, isolated, identical units called "Cells", each serving a capped subset of users.
- **Deep Dive:** A single cell might serve 10,000 users. If a software bug or corrupted database destroys Cell 3, only those 10,000 users are impacted; the rest of the company's millions of users operate smoothly in other cells.
- **Trade-offs & Realities:** Cross-cell data sharing requires complex global routing routers and global data synchronization.
- **Follow-up Probes:** "What is the routing component that directs users to their assigned cell?" (The Cell Router).

### Q7: Explain the Strangler Fig migration pattern.
- **Short Answer:** An incremental migration strategy where a legacy system is gradually replaced by routing specific pieces of functionality to a modern implementation behind an API facade until the old system is completely decommissioned.
- **Deep Dive:** Eliminates the catastrophic risk of big-bang cutovers. Teams validate small services in production, adjusting architecture based on real telemetry without risking the entire business.
- **Trade-offs & Realities:** Maintaining dual systems during the migration period incurs operational overhead and temporary infrastructure costs.
- **Follow-up Probes:** "How do you route traffic between legacy and modern systems?" (Using an L7 reverse proxy like Envoy or AWS ALB with path-based routing rules).

### Q8: What is a Circuit Breaker in distributed systems?
- **Short Answer:** A stability pattern that intercepts calls to an external service; when failures exceed a threshold, it trips open, failing fast immediately without calling the dependency, allowing it time to recover.
- **Deep Dive:** States: Closed (normal), Open (failing fast), and Half-Open (sending trial probes to test recovery). Prevents cascading resource exhaustion in thread and connection pools.
- **Trade-offs & Realities:** Applications must have sensible fallback logic (returning cached data or default responses) when the breaker trips open.
- **Follow-up Probes:** "What is the difference between a timeout and a circuit breaker?" (A timeout bounds a single request; a circuit breaker stops sending requests entirely once failure trends are detected).

### Q9: What is Exponential Backoff with Jitter?
- **Short Answer:** A retry algorithm where the wait duration doubles on each attempt and includes a randomized jitter factor to desynchronize concurrent client retries.
- **Deep Dive:** Formula: $t = \text{base} \times 2^{\text{attempt}} + \text{rand}(0, \text{jitter})$. Prevents the "Thundering Herd" problem where hundreds of clients retry at the exact same second, repeatedly re-crashing a recovering server.
- **Trade-offs & Realities:** Always cap the maximum backoff interval (e.g., max 30s) to prevent retries from hanging indefinitely.
- **Follow-up Probes:** "What is Full Jitter vs Equal Jitter?" (Full jitter randomizes between 0 and the backoff cap; Equal jitter keeps half the backoff fixed and randomizes the other half).

### Q10: What is the CAP Theorem?
- **Short Answer:** In a distributed data store, in the presence of a network Partition (P), you must choose between Consistency (C - all nodes see the same data simultaneously) and Availability (A - every request receives a non-error response).
- **Deep Dive:** Because network partitions are physical realities that cannot be prevented, systems are either CP (favoring correctness, rejecting writes during partition) or AP (favoring uptime, returning potentially stale data).
- **Trade-offs & Realities:** Extended by the PACELC theorem to cover non-partition states (Latency vs Consistency).
- **Follow-up Probes:** "Is etcd a CP or an AP system?" (etcd is a CP system; it rejects writes if Raft quorum is lost).

---

## Tier 2: Deep Dive & Mid-Senior Questions (Questions 11–20)

### Q11: How do you design an Ephemeral Development Environment platform that provisions in < 3 minutes?
- **Short Answer:** Use vCluster to provide isolated virtual control planes inside shared Kubernetes worker nodes, pre-seed lightweight database fixtures, and generate dynamic routes via Gateway API.
- **Deep Dive:**
  - *Compute:* vCluster provisions in ~20 seconds (spins up a virtual API server pod).
  - *Data Layer:* Avoid full database dumps. Use lightweight containerized databases initialized from 50MB SQL fixture files or branchable database engines (Neon, PlanetScale).
  - *Routing:* Gateway API `HTTPRoute` creates `pr-123.preview.company.com`. ExternalDNS registers the wildcard route.
  - *Lifecycle:* TTL controller monitors commit timestamps and Slack activity, automatically destroying idle environments after 4 hours to eliminate cloud waste.
- **Trade-offs & Realities:** High-fidelity staging parity vs speed; developers must accept mock data for external third-party dependencies.
- **Follow-up Probes:** "How do you handle secrets in preview environments?" (Fetch staging secrets via External Secrets Operator).

### Q12: Walk through the design of a Global Container Registry Mirroring architecture.
- **Short Answer:** A primary registry in the primary region with asynchronous push replication to local edge mirrors in target regions, combined with P2P daemon distribution for large clusters.
- **Deep Dive:**
  - *CI Pipeline:* Builds and signs image (Cosign) in primary region (US).
  - *Replication:* OCI registry replicates layers asynchronously to EU and APAC edge registries (Harbor/ECR).
  - *DNS/Anycast Routing:* In-cluster nodes query a regional endpoint (`registry.eu.company.com`).
  - *In-Cluster P2P:* Deploy Dragonfly or Kraken on worker nodes; node 1 pulls from the regional registry and distributes chunks to peer nodes over 10Gbps local VPC switches, eliminating registry bottlenecks.
- **Trade-offs & Realities:** Asynchronous replication introduces short propagation delays; CI must verify replication readiness before triggering multi-region GitOps deploys.
- **Follow-up Probes:** "How do you verify image integrity across mirrors?" (Verify Cosign signatures and cryptographic layer digests match the primary).

### Q13: How do you design a multi-tenant platform to eliminate "Noisy Neighbor" starvation?
- **Short Answer:** Enforce multi-dimensional quotas (CPU, RAM, network bandwidth), apply Fair Queueing with concurrency limits per tenant, and enforce rate limiting at the ingress layer.
- **Deep Dive:**
  - *Admission Layer:* Ingress Envoy proxies rate-limit requests per tenant API key using a token-bucket algorithm (returning 429).
  - *Compute Layer:* In Kubernetes, enforce `ResourceQuotas` and `LimitRanges` per namespace.
  - *Worker Queue Layer:* Implement Weighted Fair Queuing: maintain per-tenant queues; workers process round-robin with concurrency caps so a 10,000-message burst from Tenant A cannot block Tenant B.
- **Trade-offs & Realities:** Strict partitioning can lead to lower overall hardware utilization; dynamic borrowing with priority tiers balances cost and isolation.
- **Follow-up Probes:** "What is the difference between Token Bucket and Leaky Bucket?" (Token bucket allows bursts up to bucket capacity; leaky bucket smooths output to a strict constant rate).

### Q14: Explain the architecture of a Distributed Idempotency Engine.
- **Short Answer:** An atomic state store (Redis or DynamoDB) that maps unique idempotency keys to execution states (`IN_PROGRESS`, `COMPLETED`, `FAILED`) and cached responses.
- **Deep Dive:**
  1. Client sends `POST /clusters` with `Idempotency-Key: abc-123`.
  2. Server attempts an atomic conditional write: `SET abc-123 IN_PROGRESS NX EX 300`.
  3. If key already exists with `IN_PROGRESS`, return HTTP 409 Conflict ("Operation already running").
  4. If key exists with `COMPLETED`, return the cached HTTP response payload immediately without executing backend logic.
  5. If key does not exist, server executes resource creation, updates key state to `COMPLETED` with response body, and sets a 24-hour TTL.
- **Trade-offs & Realities:** If a process crashes while `IN_PROGRESS`, keys must expire safely or support state recovery.
- **Follow-up Probes:** "What storage backend is optimal for idempotency keys?" (Redis for speed; DynamoDB with conditional writes for durability).

### Q15: How do you estimate memory requirements for an OpenTelemetry tail-based sampling collector fleet?
- **Short Answer:** Multiply incoming span rate by average span size, retention buffer duration, and safety overhead multiplier.
- **Deep Dive:**
  - *Inputs:* 50,000 spans/sec. Average span size = 2 KB. Buffer duration = 30 seconds.
  - *In-Memory Data:* $50,000 \times 2 \text{ KB} = 100 \text{ MB/sec}$.
  - *30-Second Buffer:* $100 \text{ MB/sec} \times 30 \text{ sec} = 3,000 \text{ MB} \approx 3 \text{ GB}$.
  - *Overhead:* Add 100% safety margin for Go GC overhead and routing hash tables = $6 \text{ GB}$ total RAM required across the collector fleet.
- **Trade-offs & Realities:** Sizing buffers too small causes spans to be prematurely evaluated before downstream child spans arrive.
- **Follow-up Probes:** "How do you shard traces across collector replicas?" (Hash `trace_id` using the OpenTelemetry load-balancing exporter).

### Q16: How do you design an Outbox Pattern for atomic database updates and event publishing?
- **Short Answer:** Store outgoing events in an `outbox` table within the same ACID database transaction that mutates business entities; use a Change Data Capture (CDC) engine to stream events from the outbox table to Kafka.
- **Deep Dive:**
  ```sql
  BEGIN TRANSACTION;
  UPDATE accounts SET balance = balance - 100 WHERE id = 1;
  INSERT INTO outbox (event_id, topic, payload) VALUES (uuid(), 'payments', '{"amount":100}');
  COMMIT;
  ```
  Debezium reads the PostgreSQL WAL (write-ahead log) asynchronously, publishing the outbox event to Kafka with at-least-once delivery guarantees.
- **Trade-offs & Realities:** Requires maintaining an outbox table cleanup cron to delete processed rows and avoid disk bloat.
- **Follow-up Probes:** "Why not just send the Kafka message directly in the application code?" (If Kafka is down or the network fails after the DB commit, the event is permanently lost).

### Q17: What are Fencing Tokens, and how do they resolve the distributed locking race condition?
- **Short Answer:** Monotonically increasing numbers issued by the lock service on every grant; storage systems reject writes presenting a token lower than the highest token previously observed.
- **Deep Dive:** Solves the GC pause race: Client 1 acquires lock (Token 101), enters a 60-second Stop-the-World GC pause. Lock lease expires. Lock service grants lock to Client 2 (Token 102). Client 2 writes data. Client 1 wakes up, still believing it holds the lock, and attempts to write. Storage inspects token 101, observes 102 was already processed, and rejects Client 1's write.
- **Trade-offs & Realities:** Requires the underlying storage backend to support token validation checks.
- **Follow-up Probes:** "Does ZooKeeper provide fencing tokens natively?" (Yes, via `zxid` transaction IDs).

### Q18: How do you design an automated Canary Analysis engine that evaluates both error rates and latency?
- **Short Answer:** Deploy Argo Rollouts with an `AnalysisTemplate` executing parameterized PromQL queries calculating error rate percentage and 99th percentile latency ratios over short evaluation windows.
- **Deep Dive:**
  - *Metric 1 (Error Rate):*
    `sum(rate(http_5xx[1m])) / sum(rate(http_total[1m])) < 0.01`
  - *Metric 2 (p99 Latency Ratio):*
    `histogram_quantile(0.99, rate(canary_bucket[2m])) / histogram_quantile(0.99, rate(stable_bucket[2m])) < 1.2`
  - *Execution:* Evaluate every 30 seconds across a 10-minute pause. If either condition fails twice (`failureLimit: 2`), the controller automatically triggers rollback.
- **Trade-offs & Realities:** Canaries must receive sufficient real-world traffic to avoid false-positive divisions by zero.
- **Follow-up Probes:** "What is a Dry-Run Canary?" (Running the canary with traffic mirroring to evaluate metrics without returning responses to users).

### Q19: Explain the design of a High-Availability, Disaster-Resilient etcd cluster.
- **Short Answer:** Deploy an odd number of nodes (3 or 5) across distinct Availability Zones; back up snapshots periodically to S3; isolate etcd on dedicated fast NVMe disks (low write fsync latency).
- **Deep Dive:** Raft requires quorum: $\lfloor N/2 \rfloor + 1$. A 5-node cluster across 3 AZs (2 in AZ1, 2 in AZ2, 1 in AZ3) tolerates the complete loss of an entire AZ plus one additional node failure. Disks must guarantee < 10ms fsync latency to prevent heartbeat drops.
- **Trade-offs & Realities:** Increasing node count from 5 to 7 increases write latency because Raft consensus requires more cross-network round-trips.
- **Follow-up Probes:** "What tool tests etcd disk fsync performance?" (`fio`).

### Q20: What is a Metastable Failure, and how do you design recovery mechanisms against it?
- **Short Answer:** A permanent state of system degradation where feedback loops (retries, cache thrashing) sustain overload even after the triggering event has ceased.
- **Deep Dive:**
  - *Trigger:* Brief database stall causes client timeouts.
  - *Feedback Loop:* Clients retry with aggressive timers, generating 3x load; database tries to process retried requests, running out of memory; garbage collection freezes threads, causing more timeouts.
  - *Recovery Design:*
    1. Shed load early: Drop incoming traffic at the edge (Envoy) returning 503.
    2. Disable retries temporarily or enforce strict circuit breaking.
    3. Bypass cold cache stampedes using pre-warming scripts before admitting traffic.
- **Trade-offs & Realities:** Automated recovery often fails during metastability; manual human intervention to shed load is frequently required.
- **Follow-up Probes:** "How does constant work prevent metastability?" (System processes fixed-size batches regardless of load, avoiding nonlinear explosion).

---

## Tier 3: Staff & Principal Architect Scenarios (Questions 21–25)

### Q21: Design the complete architecture of an Internal Developer Platform (IDP) for a 1,000-engineer enterprise.
- **Candidate Framework:**
  - *5-Plane Reference Architecture:*
    - *Developer Control Plane:* Backstage developer portal with Software Catalog, TechDocs, and Software Templates; internal CLI (`pctl`).
    - *Integration & Delivery Plane:* GitHub Actions with Actions Runner Controller (ARC) for ephemeral CI; Argo CD for GitOps reconciliation across 30 Kubernetes clusters.
    - *Resource Orchestration Plane:* Crossplane and HCP Terraform for cloud infrastructure (RDS, S3, IAM) vending via declarative CRDs.
    - *Security & Governance Plane:* Kyverno admission guardrails; HashiCorp Vault + External Secrets Operator; Sigstore Cosign for image verification.
    - *Infrastructure Plane:* Multi-account AWS landing zone managed by AWS Control Tower; EKS clusters auto-scaled via Karpenter.
  - *Adoption Strategy:* Partner with 2 lighthouse teams; build paved roads for the 80% use case (stateless HTTP microservices); measure success via DORA and developer NPS.
- **Red Flags:** Focusing exclusively on the Backstage UI without explaining the underlying orchestration and delivery automation.

### Q22: Design a multi-region, zero-data-loss active-active infrastructure for a global financial platform.
- **Candidate Framework:**
  - *Compute:* Identical EKS clusters deployed across two primary cloud regions (`us-east-1` and `us-west-2`), sized statically to handle 100% of global traffic in either region ($N/(N-1)$ capacity).
  - *Traffic Routing:* AWS Route53 Application Recovery Controller (ARC) with Anycast routing controls, capable of shifting 100% of traffic in <60 seconds.
  - *Data Layer:* Multi-Region Aurora Global Database with asynchronous cross-region replication for read workloads; CockroachDB or AWS Spanner for strictly serializable financial ledger transactions.
  - *Static Stability:* Ingress proxies cache routing tables; regional clusters operate autonomously if cross-region WAN links sever.
  - *Chaos Testing:* Automated monthly GameDays injecting network isolation between regions.
- **Red Flags:** Claiming zero data loss with asynchronous replication (RPO > 0 unless using distributed consensus multi-region writes).

### Q23: Design a Real-Time Multi-Cloud Cost Allocation and Anomaly Detection Engine.
- **Candidate Framework:**
  - *Data Collection:* Deploy OpenCost agents in all Kubernetes clusters; ingest AWS Cost and Usage Reports (CUR) and GCP billing exports.
  - *Stream Processing:* Ingest pod CPU/RAM usage events and node pricing streams into Apache Kafka.
  - *Analytics Store:* Stream into ClickHouse. Columnar schema partitioned by `date` and `tenant_id` enables sub-second cost queries across billions of records.
  - *Anomaly Detection:* Background worker executes rolling 7-day Holt-Winters statistical forecasting; if a service's hourly spend exceeds 3 standard deviations above baseline, post an alert to Slack.
  - *Developer Interface:* Expose spend per microservice directly inside Backstage service dashboards with unit economic tracking.
- **Red Flags:** Storing high-volume telemetry events in relational PostgreSQL (causes write exhaustion).

### Q24: How would you architect an enterprise database migration strategy to move 200 production PostgreSQL databases from self-hosted EC2 to AWS Aurora without downtime?
- **Candidate Framework:**
  - *Assessment & Sizing:* Analyze current IOPS, storage, and CPU; size Aurora clusters with auto-scaling read replicas.
  - *Replication Pipeline:*
    1. Enable logical decoding (`wal_level = logical`) on legacy PostgreSQL.
    2. Deploy AWS Database Migration Service (DMS) or Debezium for continuous Change Data Capture (CDC).
    3. Execute initial full-load snapshot, followed by continuous replication.
  - *Verification:* Run automated checksum verification scripts comparing row counts and cryptographic hashes between source and target.
  - *Cutover:*
    1. Shift read-only traffic to Aurora read replicas.
    2. During a 60-second maintenance window, pause writes at the application connection pool (PgBouncer).
    3. Wait for CDC replication lag to reach 0.
    4. Update DNS/connection strings to Aurora primary.
    5. Unpause application writes.
  - *Rollback Plan:* Maintain reverse replication (Aurora -> legacy EC2) for 48 hours in case unexpected performance defects emerge.
- **Red Flags:** Proposing an offline 12-hour dump and restore for production banking databases.

### Q25: Architect a multi-tenant CI/CD runner fleet serving 10,000 builds/day with strict security isolation, fast cold-starts, and cost optimization.
- **Candidate Framework:**
  - *Compute Engine:* Actions Runner Controller (ARC) on dedicated Kubernetes clusters using ephemeral runner scale sets.
  - *Security Isolation:* Every job executes in a single-use ephemeral pod that is destroyed upon job completion. Enforce `readOnlyRootFilesystem`, non-root execution, and block host namespaces via Kyverno.
  - *Cold-Start Optimization:*
    - Maintain a warm pool of pre-warmed idle runner pods (e.g., 20 pods) during peak hours.
    - Pre-bake heavy runner container images (Docker, Go, Node, Java) into the node AMI to avoid image pull latency.
  - *Cache Acceleration:* Centralized BuildKit daemon with remote OCI-backed caching; ephemeral runners mount shared RAM/SSD cache volumes.
  - *Cost Optimization:* Deploy runners on AWS Spot Instances managed by Karpenter with automated consolidation, cutting compute spend by 70%.
- **Red Flags:** Sharing persistent VMs across teams or allowing Docker socket mounts.
