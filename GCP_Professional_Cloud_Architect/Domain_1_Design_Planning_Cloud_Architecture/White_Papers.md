# Domain 1: Designing and Planning a Cloud Solution Architecture — Architectural White Papers

---

## White Paper 1: Globally Scalable Multi-Region OLTP Architecture with Cloud Spanner

### Abstract
Enterprise financial and retail systems historically faced the CAP theorem dilemma: sacrifice strong consistency for global scale, or sacrifice global scale for ACID transactions. This paper details how Cloud Spanner and Google's TrueTime infrastructure resolve this limitation to deliver $99.999\%$ multi-region availability with external consistency.

### TrueTime Mechanism & Consensus
Cloud Spanner uses GPS and atomic clock references deployed in every Google datacenter to bound clock drift to an uncertainty window $\epsilon$ (typically $< 7\text{ms}$).
```
Transaction T1 commits at timestamp t1.
Cloud Spanner enforces: commit_wait() until true_time.now().earliest > t1.
Guarantee: If T2 begins after T1 commits, then timestamp(T2) > timestamp(T1).
```
This guarantees strict serializability without global distributed locks across cross-continental links.

### Multi-Region Topology Pattern
- **Leader Region:** Serves read-write transactions and coordinates Paxos consensus (`us-central1`).
- **Read-Only / Witness Region:** Provides quorum voting without storing full data replicas (`us-east1`).
- **Serving Regions:** Serve stale or strongly consistent local reads with sub-10ms latency (`europe-west1`, `asia-east1`).

---

## White Paper 2: Real-Time High-Throughput Streaming Ingestion Pipeline

### Abstract
Handling millions of events per second from mobile clients, connected vehicles, and IoT sensors requires a decoupled, fault-tolerant ingestion pipeline that guarantees exactly-once processing and zero data loss.

### Architecture Topology
```
[ 100,000+ IoT / Mobile Clients ]
               │
               ▼ HTTPS / gRPC
[ Global External HTTP(S) Load Balancer ]
               │
               ▼
[ Cloud Run Ingestion Microservice / Cloud Pub/Sub ]
               │
               ▼ Streaming Topics
[ Cloud Pub/Sub Message Broker ]
               │
       ┌───────┴───────────────────────┐
       ▼                               ▼
[ Cloud Dataflow (Apache Beam) ]  [ Raw Long-Term Lake ]
  - Windowing (1-min tumbling)     (Cloud Storage Archive)
  - Schema validation
  - Exactly-once deduplication
       │
       ├───────────────────────────────┐
       ▼                               ▼
[ Cloud Bigtable ]              [ Google BigQuery ]
(Low-latency operational reads, (Ad-hoc SQL analytics,
 sub-10ms dashboard queries)     historical trend ML)
```
