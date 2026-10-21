# PE Domain 8: System Design — Scenario Quiz 3

Test your mastery of global artifact distribution, canary analysis, and capacity planning.

---

### Question 1
You are designing a global artifact distribution system for container images across US, EU, and APAC data centers. What is the primary operational trade-off between **Synchronous Cross-Region Replication** and **Asynchronous Replication**?

A) Synchronous replication guarantees instant consistency across all regions but severely increases CI build latency by making `docker push` wait for trans-oceanic network round-trips; Asynchronous replication allows fast local pushes but introduces replication lag.  
B) Asynchronous replication corrupts container layers.  
C) Synchronous replication is free; asynchronous costs money.  
D) Synchronous replication only works with Docker.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** A  
**Explanation:** Trans-oceanic latency (e.g., US to APAC is 150ms+ RTT) makes synchronous writes intolerable for fast CI pipelines. Asynchronous replication finishes local push fast and streams layers across regions in the background.
</details>

---

### Question 2
When planning etcd capacity for a Kubernetes cluster supporting 5,000 worker nodes and 150,000 pods, what is the recommended maximum database size limit for etcd before severe performance degradation occurs?

A) 100 MB  
B) 10 TB  
C) 500 GB  
D) 8 GB  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** D  
**Explanation:** etcd is an in-memory B-tree with copy-on-write MVCC storage. The default quota is 2GB, configurable up to 8GB. Operating beyond 8GB leads to excessive memory usage, slow compactions, and disk sync latencies that threaten Raft heartbeats and cause leader elections.
</details>

---

### Question 3
How does **Peer-to-Peer (P2P) Image Distribution** (e.g., Uber Kraken, CNCF Dragonfly) prevent registry failure during massive concurrent cluster rollouts?

A) Instead of 1,000 worker nodes all downloading identical 2GB image layers from a single central registry simultaneously (saturating the registry's NIC and incurring high egress costs), nodes download chunks from each other over the local cluster network.  
B) It converts container images into zip files.  
C) It downloads images from BitTorrent sites.  
D) It deletes all base images.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** A  
**Explanation:** In large clusters, simultaneous deployments create a burst of hundreds of gigabits per second against the registry. P2P distribution transforms nodes into seeders and leechers, shifting traffic to local top-of-rack switches.
</details>

---

### Question 4
In system design estimation, what is the approximate latency of an un-cached disk SSD read compared to an in-memory L1 cache reference?

A) SSDs are faster than memory.  
B) SSD reads take 10 minutes.  
C) They have identical latency.  
D) Memory access takes ~100 nanoseconds; SSD random reads take ~100 microseconds (roughly 1,000x to 10,000x slower).  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** D  
**Explanation:** Understanding latency numbers every programmer should know (Jeff Dean) is essential in system design: L1 cache (~0.5ns), RAM (~100ns), SSD read (~100us), cross-datacenter RTT (~150ms).
</details>

---

### Question 5
What is the primary architectural function of an **Application Recovery Controller (Route53 ARC)** in multi-region active-active architectures?

A) It backs up databases to tape.  
B) It manages high-reliability regional routing controls and automated health checks, allowing platform teams to execute a 100% cross-region traffic shift in under 60 seconds with deterministic consistency.  
C) It optimizes CSS styles.  
D) It reboots failed application pods.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** B  
**Explanation:** Route53 ARC provides highly available routing controls distributed across 5 independent regional endpoints, ensuring that even if an entire AWS region collapses, routing controls can shift traffic to healthy regions without depending on the impacted region.
</details>

---

### Question 6
Why is storing stateful database files on local NVMe ephemeral storage faster, but operationally riskier, than using network-attached cloud block storage (AWS EBS)?

A) NVMe drives only work on Linux.  
B) NVMe drives cannot store SQL data.  
C) Local NVMe provides millions of IOPS with sub-millisecond bus latency, but data is lost permanently if the host EC2 instance is stopped or suffers hardware failure; EBS provides network durability and automated snapshots, but with higher latency.  
D) EBS storage cannot be encrypted.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** C  
**Explanation:** Local NVMe attaches directly to the PCIe bus, delivering maximum performance for high-throughput databases (Cassandra, Kafka). However, durability requires distributed application-level replication, as instance termination erases ephemeral disks.
</details>

---

### Question 7
In a distributed event streaming platform, what does **At-Least-Once Delivery** guarantee, and what does it require from consumers?

A) Messages are guaranteed to arrive once and only once.  
B) Messages are guaranteed never to be lost, but network retries may cause duplicate messages to be delivered; consumers must be **idempotent** to handle duplicates safely.  
C) Messages are delivered within 1 millisecond.  
D) Messages are encrypted with RSA.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** B  
**Explanation:** True exactly-once delivery across distributed networks is impossible without heavy two-phase coordination. At-least-once guarantees zero data loss, delegating deduplication and idempotency to consumers.
</details>

---

### Question 8
What is the primary symptom of a **Metastable Failure State** in a distributed system?

A) A bug in the CPU microcode.  
B) A database that has zero records.  
C) A failure state where an initial transient trigger (like a temporary network spike) pushes the system into a sustaining feedback loop of overload (e.g., retries, cache misses, GC thrashing) that persists indefinitely even *after* the initial trigger has disappeared.  
D) The server temperature drops below freezing.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** C  
**Explanation:** Metastability (Brooker, SRE) occurs when self-reinforcing recovery mechanisms become destructive: retrying clients overload a slow server, preventing it from catching up. Recovering requires dramatic intervention: shedding 80% of traffic, pausing retries, or restarting with cold caches.
</details>

---

### Question 9
How does **Dead Letter Queue (DLQ)** pattern improve system resilience?

A) It archives emails of employees who left the company.  
B) When a message repeatedly fails processing (poison pill) after a maximum retry threshold, it is moved to a separate queue (DLQ) for asynchronous inspection, preventing it from blocking the main processing pipeline or triggering crash loops.  
C) It reboots worker pods.  
D) It deletes all unread messages.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** B  
**Explanation:** A poison pill message that crashes worker threads will block a queue indefinitely if retried endlessly. Moving failed messages to a DLQ unblocks downstream processing and preserves the payload for debugging.
</details>

---

### Question 10
In multi-region architectures, what is **Write Amplification** caused by synchronous cross-region database replication?

A) Writing to two different text editors.  
B) Writing files in uppercase letters.  
C) For every single logical write executed by the application, multiple physical network packets and disk writes must be duplicated across distant data centers, compounding write latency and consuming network bandwidth.  
D) Hard drive audio speakers playing too loudly.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** C  
**Explanation:** Cross-region synchronous replication forces every transaction commit to wait for light to travel across fiber-optic cables, turning 1ms local disk writes into 100ms+ network-bound latency bottlenecks.
</details>
