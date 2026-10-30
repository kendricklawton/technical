# Domain 5: Reliability, Operations & Official Case Studies — Architectural White Papers

---

## White Paper 1: Real-Time Stateful Multiplayer Game Architecture (Mountkirk Games Case Study)

### Abstract
Stateful multiplayer games cannot run on standard stateless HTTP container platforms because active game sessions require persistent, low-latency bidirectional UDP sockets. This paper details the architecture implemented for Mountkirk Games using Google Kubernetes Engine, Agones, and Cloud Spanner.

### Mountkirk Games System Topology
```
[ 10,000,000+ Mobile Clients ]
               │
               ▼ UDP Game Traffic (Direct to Node Port)
[ GKE Standard Node Pool (Managed by Agones Game Server SDK) ]
   ├── Pod: GameServer-Session-A (64 Players, UDP 7000-8000)
   └── Pod: GameServer-Session-B (64 Players, UDP 7000-8000)
               │
               ├───────────────────────────────┐
               ▼ Async State Updates           ▼ Telemetry
[ Cloud Bigtable ]                     [ Cloud Pub/Sub ]
(High-throughput real-time leaderboards,        │
 sub-5ms match state persistence)               ▼
                                       [ Cloud Dataflow ]
                                                │
               ┌────────────────────────────────┴───────────────┐
               ▼                                                ▼
[ Google BigQuery ]                                    [ Cloud Spanner Multi-Region ]
(Monetization & cheat analytics)                       (Player Profiles, Wallets, ACID)
```

---

## White Paper 2: Connected Vehicle Telemetry & Predictive Maintenance (TerramEarth Case Study)

### Abstract
Operating a global fleet of 20 million agricultural and mining vehicles across six continents requires an ingestion architecture resilient to intermittent cellular connectivity, capable of processing Petabytes of time-series sensor data, and driving predictive ML models.

### TerramEarth Telemetry Ingestion Flow
```
[ 20 Million Connected Vehicles ]
   ├── Edge Buffer: Flash storage caches 100+ sensor metrics
   └── Uplink: Opportunistic Cellular / Satellite / Dealer Wi-Fi
               │
               ▼ Streaming REST / MQTT via HTTPS
[ Google Cloud Pub/Sub ]
   ├── Partitioned topics by Vehicle VIN / Region
   └── 7-day retention buffer for downstream resilience
               │
               ▼ Continuous Stream Processing
[ Cloud Dataflow (Apache Beam) ]
   ├── Deduplication of retransmitted packets
   ├── 5-minute tumbling window aggregations
   └── Anomaly detection trigger (overheating engine alerts)
               │
       ┌───────┴───────────────────────┐
       ▼                               ▼
[ Cloud Bigtable ]              [ Google BigQuery ]
(Time-series storage by VIN#ts, (Historical multi-year analytics,
 fast dealer query lookup)       Vertex AI model training)
```
