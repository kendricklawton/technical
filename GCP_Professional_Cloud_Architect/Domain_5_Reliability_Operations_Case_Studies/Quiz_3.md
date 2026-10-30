# Domain 5: Reliability, Operations & Official Case Studies — Quiz 3

**Exam:** Google Cloud Certified: Professional Cloud Architect (PCA)  
**Format:** 10 Scenario-Based Multiple-Choice Questions with In-Depth Technical Explanations

---

### Question 1 (Case Study: TerramEarth)
TerramEarth currently collects 200 Petabytes of historical sensor data stored in legacy on-premises SAN storage across 5 continents. The business wants to migrate this entire historical dataset into Google Cloud Storage within 6 months. Due to bandwidth limitations in several remote countries, network transfer over the public internet is impossible. What data transfer strategy should the architect recommend?
- A. Ship Google Transfer Appliances to remote facilities to transfer data locally, and use Storage Transfer Service for locations with sufficient broadband connectivity.
- B. Use `rsync` over dial-up internet connections.
- C. Print the sensor logs to microfiche.
- D. Wait for 5G cellular coverage to expand globally.

---

### Question 2 (Case Study: Cymbal Direct)
Cymbal Direct operates in both North America and Europe. They want European shoppers to experience sub-20ms latency when loading product pages, while ensuring that EU customer personal profile data is stored exclusively in European data centers to comply with GDPR. How should the storage and compute tiers be architected?
- A. Single-region deployment in `us-central1`.
- B. Regional GKE Autopilot clusters and regional Cloud SQL / Spanner instances located in European regions (`europe-west1`), behind a Global External HTTPS Load Balancer with regional routing policies and Organization Policy resource location constraints.
- C. Replicating all customer data to `us-east1`.
- D. Disallowing European users from purchasing items.

---

### Question 3 (Case Study: EHR Healthcare)
An architect is reviewing the disaster recovery plan for EHR Healthcare's electronic prescription service. The business dictates that the service must survive the loss of an entire Google Cloud region with an RTO of under 2 minutes and an RPO of 0. Which database architecture satisfies this requirement?
- A. Cloud SQL in a single region with nightly snapshots
- B. Multi-Region Cloud Spanner instance configuration spanning redundant geographic regions
- C. Firestore in single-region mode
- D. BigQuery batch loads

---

### Question 4 (Case Study: Mountkirk Games)
Mountkirk Games is releasing a major game update. They want to test new weapon balance mechanics on 5% of real players without deploying a separate mobile client build or risking negative reviews if the balance is unpopular. What architectural pattern enables this capability?
- A. In-game dynamic feature flags controlled by an API backend, allowing real-time percentage-based rollout and instant rollback.
- B. Publishing a new APK to the Google Play Store for 5% of users.
- C. Changing cluster IP addresses.
- D. Terminating 95% of game servers.

---

### Question 5 (Reliability: Graceful Degradation)
During peak shopping events, Cymbal Direct's third-party payment fraud evaluation service begins timing out. If the checkout microservice fails synchronously, all purchases will fail. How should the architect design the checkout flow to ensure business continuity?
- A. Crash the entire checkout application.
- B. Implement graceful degradation: if fraud evaluation times out after 1 second, flag the transaction for asynchronous post-purchase review in a Pub/Sub queue and allow the checkout to proceed.
- C. Discard the customer order without notice.
- D. Retry the fraud service 100 times synchronously.

---

### Question 6 (Reliability: Chaos Testing)
An enterprise wants to validate that their GKE microservices can automatically survive random node terminations, network latency spikes, and zone outages before going live with a high-profile product launch. What practice and tooling should be incorporated into their staging environment?
- A. Chaos Engineering using tools like Chaos Mesh or LitmusChaos to inject controlled faults and observe system recovery.
- B. Disabling all monitoring and alerts.
- C. Running stress tests on developer laptops only.
- D. Never testing failure modes in pre-production.

---

### Question 7 (Case Study: TerramEarth)
TerramEarth wants to provide predictive maintenance alerts to agricultural tractor operators via in-cab displays. An alert must be displayed within 3 seconds if engine sensor telemetry indicates imminent hydraulic pump failure. What processing architecture meets this low-latency SLA?
- A. Overnight BigQuery batch SQL queries.
- B. Stream processing using Cloud Pub/Sub and Cloud Dataflow with tumbling window anomaly detection rules, pushing real-time alerts via Firebase Cloud Messaging / WebSockets.
- C. Storage Transfer Service running once per week.
- D. Exporting sensor dumps to CSV files on Cloud Storage.

---

### Question 8 (Disaster Recovery: Cold Standby)
A non-profit organization wants to establish a disaster recovery solution for their internal document archiving application on Google Cloud. The organization has very limited budget and can tolerate an RTO of 24 hours and an RPO of 12 hours. What DR tier is appropriate?
- A. Multi-Region active-active Cloud Spanner
- B. Cold Standby: Scheduled database snapshots exported to Cloud Storage Multi-Region, with automated Terraform scripts to provision Compute Engine VMs only when DR is declared.
- C. Dedicated Interconnect with 4 circuits.
- D. Hot standby running 24/7.

---

### Question 9 (Case Study: EHR Healthcare)
EHR Healthcare requires that all outbound internet traffic from healthcare microservices running in a private VPC pass through a centralized inspection proxy to prevent malware command-and-control communication. How can traffic be routed to the inspection appliance?
- A. Configure custom VPC routes with next-hop pointing to the internal IP of the inspection proxy instance / internal load balancer.
- B. Assign public IP addresses to all healthcare VMs.
- C. Open port 80 to `0.0.0.0/0`.
- D. Disable routing on the VPC.

---

### Question 10 (Case Study: Cymbal Direct)
Cymbal Direct needs to deliver high-resolution product catalog images to mobile devices worldwide. Images should load in under 100ms regardless of whether the user is in Sydney, London, or San Francisco. How should static image delivery be architected?
- A. Compute Engine VM running Nginx in `us-central1`.
- B. Google Cloud Storage Multi-Region bucket configured as a backend bucket for a Global External HTTP(S) Load Balancer with Cloud CDN enabled.
- C. Sending images via email attachment.
- D. Storing images in BigQuery.

---

## Answer Key & Rationales

1. **A** — Google Transfer Appliance provides offline, physical data transfer for remote, bandwidth-constrained sites, complemented by STS where networks allow.
2. **B** — Deploying regional clusters and database instances in European regions with Anycast routing and Org Policy constraints satisfies GDPR residency mandates.
3. **B** — Multi-Region Cloud Spanner provides continuous synchronous multi-region replication with automatic failover, delivering RPO=0 and RTO < 2 minutes.
4. **A** — Backend-driven dynamic feature flags allow controlled percentage-based feature testing and instant server-side rollback without client updates.
5. **B** — Graceful degradation permits critical revenue transactions to complete while deferring non-blocking validations to asynchronous queues.
6. **A** — Chaos Engineering proactively injects real-world failures into controlled environments to validate fault tolerance and self-healing mechanisms.
7. **B** — Cloud Pub/Sub and Cloud Dataflow deliver sub-second stream processing and real-time alert dispatch to connected IoT devices.
8. **B** — Cold Standby stores immutable snapshots in multi-region storage and deploys infrastructure via IaC on-demand, minimizing idle costs.
9. **A** — Custom VPC routes with next-hop pointing to a proxy or internal load balancer force all egress traffic through inspection appliances.
10. **B** — Cloud Storage Multi-Region paired with Global External HTTPS Load Balancer and Cloud CDN caches image assets at edge PoPs globally for sub-100ms delivery.
