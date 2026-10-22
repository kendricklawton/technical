# Domain 4: Analyzing and Optimizing Technical and Business Processes — Architectural White Papers

---

## White Paper 1: Large-Scale Heterogeneous Cloud Migration Architecture

### Abstract
Migrating multi-petabyte enterprise environments from legacy data centers to Google Cloud requires careful selection of data transfer pathways based on data volume, network bandwidth, downtime tolerance, and database change rates.

### Decision Matrix for Data Migration Pathways
```
Dataset Size       Available Bandwidth    Recommended Migration Tool
─────────────────────────────────────────────────────────────────────────────
< 10 TB            > 100 Mbps             gsutil / gcloud storage rsync
10 TB - 100 TB     > 1 Gbps               Storage Transfer Service (STS Agent)
> 100 TB           < 1 Gbps               Google Transfer Appliance (Physical)
Relational DB      Continuous CDC         Database Migration Service (DMS)
VMware VMs         Live Replication       Migrate to Virtual Machines
```

### Continuous Replication & Minimal-Downtime Cutover
```
[ On-Premises VMware / Database ]
         │
         │ 1. Initial Full Snapshot (Online via Dedicated Interconnect)
         ▼
[ Google Cloud Target Storage / Compute ]
         │
         │ 2. Continuous Delta Change Data Capture (CDC / Changed Blocks)
         ▼
[ Synchronized Cloud Replica (Lag < 2 seconds) ]
         │
         │ 3. Scheduled 5-minute Maintenance Window:
         │    a. Freeze on-premises writes
         │    b. Replicate final delta transaction logs
         │    c. Promote Cloud SQL / Compute Engine to Primary
         ▼
[ Production Live on Google Cloud ]
```

---

## White Paper 2: Cloud FinOps Architecture & Continuous Cost Governance

### Abstract
Uncontrolled cloud adoption without architectural governance leads to runaway operational expenditures. This paper outlines an automated FinOps architecture integrating BigQuery billing export, Cloud Functions budget enforcement, and Active Assist Recommenders.

### Automated Cost Governance Topology
```
[ Google Cloud Billing Account ]
               │
               ▼ Continuous Streaming Export
[ BigQuery Billing Data Warehouse ]
   ├── Looker Studio FinOps Dashboards
   └── Scheduled SQL Queries for Anomaly Spikes
               │
               ▼ Budget Alerts (>80%, >100%, >120%)
[ Cloud Pub/Sub Topic: "billing-alerts" ]
               │
               ▼
[ Cloud Run Budget Enforcer Function ]
   ├── Reads project cost run-rate
   ├── If project > 120% and environment == 'sandbox':
   │     - Disables billing on sandbox project
   │     - Shuts down Compute Engine Spot MIGs
   └── Sends P0 Slack alert to Engineering Director
```
