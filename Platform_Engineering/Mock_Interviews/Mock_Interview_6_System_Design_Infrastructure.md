# Mock Interview 6: System Design — Infrastructure & Distributed Platforms

Simulates an advanced 60-minute infrastructure system design interview focusing on high-availability, multi-region resilience, cellular architecture, and fleet management.

---

## Candidate Brief (Safe to read in advance)

- **Duration:** 60 minutes.
- **Format:** Open-ended architectural system design. You will drive the conversation through requirements gathering, back-of-the-envelope estimation, high-level architecture, deep dives into fault isolation and state synchronization, and failure mode analysis.
- **Evaluation Criteria:** Requirements clarification, blast radius reduction, distributed systems trade-offs (CAP theorem, PACELC), operational reality, and platform cost efficiency.

---

## Interviewer Packet (Confidential — Do not read beforehand)

### Variant A: Multi-Region Active-Active Kubernetes Fleet
- **Prompt:** "Design a multi-region active-active infrastructure platform across AWS US-East-1, US-West-2, and EU-Central-1 capable of serving 150,000 requests per second with sub-50ms p99 latency, zero-data-loss failover for transactional workloads, and unified GitOps cluster management."
- **Hidden Company Fact Sheet (Reveal ONLY when asked):**
  - *Data Layer:* 70% reads (cached/replicated), 30% writes (strongly consistent ACID transactions). Aurora Global Database with write forwarding or distributed CockroachDB / Spanner.
  - *Traffic Routing:* Anycast DNS (Route53 / Cloudflare) + Global Accelerator.
  - *Stateful services:* Kafka clusters deployed per-region with MirrorMaker 2 active-passive replication for asynchronous event processing.
  - *Blast Radius Requirement:* Total loss of an entire AWS region must not breach 99.99% overall uptime SLA. Failover time must be $< 30$ seconds.
- **Curveballs:**
  - *Minute 25:* "A split-brain network partition isolates EU-Central-1 from US regions for 12 minutes. How does your ingress routing and distributed storage prevent duplicate writes and dirty reads?"
  - *Minute 42:* "A bad deployment of a shared CoreDNS ConfigMap rolled out across all regions simultaneously and crashed cluster DNS. How does your deployment platform prevent global correlated failures?"
    - *Expected answer:* Phased/Canary cluster rollouts, cellular fleet management, independent control planes with Argo CD ApplicationSet progressive rollouts.

### Variant B: Cellular Architecture & Shuffle Sharding for 10M Tenants
- **Prompt:** "Design an isolation architecture for a multi-tenant SaaS platform serving 10 million organizations. A small fraction of 'noisy neighbors' periodically run rogue queries that exhaust CPU and connection pools. Ensure that no single customer can take down more than a tiny, bounded fraction of other customers."
- **Hidden Company Fact Sheet (Reveal ONLY when asked):**
  - *Scale:* 10,000 RPS average, peaking at 150,000 RPS.
  - *Current setup:* Single massive 500-node EKS cluster with shared RDS Postgres. Outages affect 100% of tenants simultaneously.
  - *Infrastructure budget:* Cannot afford a dedicated Kubernetes cluster per tenant (too expensive for free/tier-1 tenants).
- **Key Concepts to Test:**
  - Cellular Architecture: Self-contained, independent deployment units ("cells") that share no runtime dependencies.
  - Shuffle Sharding: Each tenant is assigned to an overlapping subset of cells (e.g., 2 cells out of 8). Total combinations: $\binom{8}{2} = 28$. Two tenants only collide if both assign to the exact same pair.
- **Curveballs:**
  - *Minute 30:* "A DDoS attack targets a specific enterprise customer assigned to cells 3 and 7. What happens to tenants sharing cell 3? How does client routing redirect unaffected traffic to cell 7?"

### Variant C: Global Ingress & Zero-Trust Ephemeral Mesh
- **Prompt:** "Design an enterprise-grade Ingress and Service Mesh platform supporting 800 microservices across 12 Kubernetes clusters. Must support automatic mutual TLS (mTLS) with SPIFFE/SPIRE cryptographic identities, dynamic canary traffic splitting (0.5% increments), and edge WAF inspection with sub-millisecond overhead."
- **Hidden Company Fact Sheet (Reveal ONLY when asked):**
  - *Compliance:* FIPS 140-2 Level 3 cryptographic modules required; zero cleartext traffic over VPC peering or transit gateways.
  - *Performance:* Sidecar injection is causing 4ms p99 latency regressions and 64GB memory overhead per node.
  - *Interview Expectation:* Explore eBPF-based ambient mesh (e.g., Cilium or Istio Ambient) to bypass sidecar proxy overhead.

---

## Hint Ladder
- *Level 1 (Nudge):* "How can we structure the clusters so an outage in one region doesn't propagate to others?"
- *Level 2 (Guiding Question):* "If all clusters pull from the exact same Git branch at the same minute, what happens when a breaking manifest is pushed?"
- *Level 3 (Explicit Cue):* "Consider using progressive delivery across cluster rings: Ring 0 (Canary/Internal), Ring 1 (Single region), Ring 2 (Global fleet)."

---

## Scoring Rubric

| Score | Anchor | Behavioral Criteria |
| :---: | :--- | :--- |
| **1** | Strong No Hire | Proposes single centralized database across regions without considering speed-of-light latency; deploys everything simultaneously to all clusters. |
| **2** | No Hire | Understands multi-region basics but ignores data consistency, network partitions, or correlated failure domains. |
| **3** | Hire | Designs isolated failure domains; articulates CAP/PACELC trade-offs accurately; implements progressive ring rollouts for fleet management. |
| **4** | Strong Hire | Masterful understanding of cellular architecture, shuffle sharding, eBPF data planes, and quorum consensus under Byzantine conditions. |

---

## Self-Debrief Template
- **Architecture Chosen:** Multi-Region Active-Active / Cellular / Mesh
- **Blast Radius Strategy Identified:**
- **Failure Mode Probing Score (1-4):**
- **Action Items & Gaps:**
