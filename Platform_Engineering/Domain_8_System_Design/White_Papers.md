# Domain 8: System Design — Essential White Papers & Architecture Guide

> **Core Focus:** Enterprise Distributed Systems, Cell-Based Architecture, Fault Isolation Boundaries, Consensus Protocols (Raft, Paxos), CAP/PACELC Theorems, Event Sourcing, CQRS, and Sharding.

---

## 1. Priority White Papers & Foundational Treatises

### 1.1 [Reducing the Scope of Impact with Cell-Based Architecture (AWS Whitepaper)](https://docs.aws.amazon.com/wellarchitected/latest/reducing-scope-of-impact-with-cell-based-architecture/welcome.html)
* **Canonical Scope:** Architectural blueprint for scaling massive distributed platforms by partitioning workloads into isolated, self-contained units ("cells").
* **Key Mechanics:**
  * **Cell Routers:** Lightweight, stateless routing layer mapping incoming tenant requests to specific cells using partition keys.
  * **Blast Radius Containment:** With 100 cells, a catastrophic software bug or hardware failure in one cell impacts only $1\%$ of customers, while $99\%$ continue operating uninterrupted.
  * **Eliminating Cross-Cell Dependencies:** Cells share no state, databases, or internal network links with other cells.

### 1.2 [AWS Fault Isolation Boundaries Whitepaper](https://docs.aws.amazon.com/whitepapers/latest/aws-fault-isolation-boundaries/welcome.html)
* **Canonical Scope:** Understanding failure domains across global, regional, zonal, and cell scopes.
* **Hierarchy:**
  * *Global Services:* IAM, Route 53, CloudFront (Global control planes must have static stability so data planes survive outages).
  * *Regional Services:* S3, DynamoDB, SQS (Independent control and data planes per region).
  * *Zonal Services:* EC2, EBS, VPC Subnets (Completely independent power, networking, and flood plains).

### 1.3 [In Search of an Understandable Consensus Algorithm (Raft)](https://raft.github.io/raft.pdf)
* **Canonical Scope:** The consensus algorithm powering modern cloud-native systems (etcd, Kubernetes, CockroachDB).
* **Core Roles & States:** Leader, Follower, Candidate.
* **Quorum Invariant:** Requires $\lfloor N/2 \rfloor + 1$ healthy nodes to commit a state change. A 3-node cluster tolerates 1 node failure; a 5-node cluster tolerates 2 node failures.

---

## 2. Core Architectural Patterns & Design Principles

* **CAP Theorem & PACELC:** If there is a Partition (P), choose Availability (A) or Consistency (C); Else (E), choose Latency (L) or Consistency (C).
* **CQRS (Command Query Responsibility Segregation) & Event Sourcing:** Separating write models (appending immutable events to an event store) from read models (materialized views projected for high-speed queries).
* **Idempotency Keys:** Enforcing deduplication on API requests via unique UUID keys cached in Redis/DynamoDB with TTLs to prevent double-charging or duplicate resource creation.

---

## 3. Staff-Level Interview Angle & Architectural Reflexes

1. **"Design a Global Multi-Tenant Control Plane":**
   * *Structure:* Always separate the **Control Plane** (authoring manifests, updating configuration) from the **Data Plane** (executing runtime workloads). The data plane must be **statically stable**—if the control plane dies completely, existing customer workloads continue operating without degradation.
2. **Addressing the "Split-Brain" Problem:**
   * Never deploy an even number of consensus nodes (e.g., 4 or 6 nodes). Always deploy odd numbers (3 or 5) to guarantee a mathematical majority quorum during network partitions.

---

## 4. Reading & Study Checklist

- [ ] Read *Reducing the Scope of Impact with Cell-Based Architecture*
- [ ] Read *AWS Fault Isolation Boundaries Whitepaper*
- [ ] Read the original *Raft Consensus Paper (Ongaro & Ousterhout)*
- [ ] Master CAP vs PACELC architectural tradeoffs
- [ ] Design an end-to-end multi-tenant platform control plane from scratch
