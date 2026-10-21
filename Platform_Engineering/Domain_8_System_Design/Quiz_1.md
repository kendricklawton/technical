# PE Domain 8: System Design — Scenario Quiz 1

Test your knowledge on estimation, control vs data plane decoupling, and static stability.

---

### Question 1
During a system design interview, you are asked to design an Internal Developer Platform for 500 engineers deploying 2,000 microservices. What is the fundamental rule regarding the decoupling of the **Control Plane** and the **Data Plane**?

A) The control plane and data plane must always run inside the same process.  
B) Running applications in the Data Plane must continue serving production user traffic with zero degradation even if the Control Plane (API server, CI/CD runners, GitOps controller, Portal) suffers a complete outage.  
C) Control plane downtime should automatically terminate all data plane pods.  
D) Data plane traffic must route through the developer portal.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** B  
**Explanation:** Control and data planes have distinct availability objectives. If the control plane crashes, developers cannot deploy new code or modify configs, but existing running workloads in the data plane must continue processing customer requests uninterrupted (static stability).
</details>

---

### Question 2
You are sizing a Prometheus monitoring cluster ingesting 2,000,000 active time series. Scrapes occur every 15 seconds. Using Gorilla compression (~1.5 bytes per sample) and a 2-hour in-memory Head chunk window, roughly how much RAM is required just to store active samples in memory?

A) ~1 TB  
B) ~150 MB  
C) ~1.44 GB  
D) ~50 GB  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** C  
**Explanation:** 
- Scrapes per hour per series = $3600 / 15 = 240$ samples.
- Samples in 2 hours = 480 samples per series.
- Total samples = $2,000,000 \times 480 = 960,000,000$ samples.
- Total RAM = $960,000,000 \times 1.5 \text{ bytes} \approx 1,440,000,000 \text{ bytes} \approx 1.44 \text{ GB}$ (plus index overhead).
</details>

---

### Question 3
What does the distributed systems principle of **Static Stability** mean?

A) A system continues operating normally and serving traffic using pre-provisioned capacity and local caches without needing to make control plane calls or dynamically allocate new resources during a failure.  
B) The system runs on static HTML files.  
C) The database is locked in read-only mode permanently.  
D) The system never accepts any configuration changes.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** A  
**Explanation:** Static stability means systems don't rely on reactive control plane actions during a crisis (e.g., needing to spin up EC2 instances during a regional network failure). Capacity is pre-warmed ($N/(N-1)$ across AZs), and clients cache last-known-good configurations locally.
</details>

---

### Question 4
In multi-tenant distributed systems, what is the primary benefit of **Shuffle Sharding** over standard dedicated worker sharding?

A) It shuffles data randomly on disk to prevent fragmentation.  
B) By assigning each tenant a unique mathematical combination (subset) of workers rather than a single worker or all workers, the probability that a toxic/poison-pill tenant impacts another specific tenant drops exponentially.  
C) It encrypts network traffic using a deck of cards algorithm.  
D) It eliminates the need for load balancers.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** B  
**Explanation:** With 8 workers assigned in pairs, there are $\binom{8}{2} = 28$ unique shards. If Tenant A sends a poison-pill request that crashes its 2 assigned workers, 26 of the other 27 tenant pairings remain completely unaffected.
</details>

---

### Question 5
What is the risk of a **Bimodal Fallback** in resilient platform architecture?

A) The system alternates between two different font styles.  
B) A database that supports both SQL and NoSQL.  
C) Running two identical microservices.  
D) A system has a "fast path" that relies on a cache and a "slow path" fallback that queries the database directly; when the cache fails, the sudden 100% direct load on the slow path instantly overwhelms and crashes the database.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** D  
**Explanation:** Bimodal systems run fine until the primary mode fails, exposing the un-tested, un-scaled fallback mode to catastrophic overload. Systems should prefer "constant work" architectures where failure paths do not create massive load spikes.
</details>

---

### Question 6
You are designing a CI/CD runner fleet. The organization generates an average of 600 pipeline jobs per hour. The average job duration is 10 minutes. According to **Little's Law** ($L = \lambda \times W$), how many concurrent runner instances are required to handle this steady-state load without queue backlog?

A) 60 runners  
B) 10 runners  
C) 600 runners  
D) 100 runners  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** D  
**Explanation:** 
- Arrival rate $\lambda = 600 \text{ jobs/hour} = 10 \text{ jobs/minute}$.
- Average duration $W = 10 \text{ minutes}$.
- Concurrency $L = \lambda \times W = 10 \times 10 = 100$ concurrent runners.
</details>

---

### Question 7
In Cell-Based Architectures (such as AWS Cells), what is the primary architectural goal of partitioning infrastructure into independent "Cells"?

A) To establish hard blast-radius boundaries where any catastrophic outage, corrupted deployment, or poison-pill payload is strictly contained within a single cell, protecting the remaining 95% of users.  
B) To avoid using cloud load balancers.  
C) To run different programming languages in each cell.  
D) To minimize cell phone bills.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** A  
**Explanation:** Cells are self-contained, complete instances of an entire platform stack sized to a maximum capacity threshold. Outages in Cell 4 cannot spread to Cell 1, 2, or 3.
</details>

---

### Question 8
What is the mathematical availability of a system that has 3 critical components in a serial dependency chain, where each component has an individual availability of 99.9%?

A) 99.999%  
B) 99.7%  
C) 99.9%  
D) 100%  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** B  
**Explanation:** Serial availability multiplies: $A_{\text{total}} = A_1 \times A_2 \times A_3 = 0.999 \times 0.999 \times 0.999 \approx 0.997$ (99.7%). Serial dependencies compound downtime.
</details>

---

### Question 9
Why must every mutating platform API enforce **Idempotency**?

A) To encrypt database connections.  
B) Idempotent APIs consume zero CPU.  
C) Because non-idempotent APIs are rejected by HTTP/2.  
D) Because network retries are inevitable in distributed systems; if a client retries a `POST /v1/clusters` request due to a transient timeout, an idempotent API guarantees that only one cluster is created rather than duplicates.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** D  
**Explanation:** In distributed systems, network packets time out even when the server processed the write. Clients retry with an `Idempotency-Key`. The server checks the key in an atomic database transaction, returning the cached result rather than executing duplicate mutations.
</details>

---

### Question 10
What does the **PACELC Theorem** state regarding distributed storage systems?

A) Packets Are Checked Every Loop Cycle.  
B) Passwords And Credentials Expire Last Century.  
C) If there is a Partition (P), trade off Availability (A) and Consistency (C); Else (E), trade off Latency (L) and Consistency (C).  
D) Performance and Cost are equal to Compute.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** C  
**Explanation:** PACELC extends the CAP theorem: even when the network is healthy and there is no partition (Else), a distributed system must still choose between low Latency (returning local data immediately) or strong Consistency (waiting for cross-node replication).
</details>
