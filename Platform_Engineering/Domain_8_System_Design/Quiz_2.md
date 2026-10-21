# PE Domain 8: System Design — Scenario Quiz 2

Test your mastery of multi-tenant isolation, noisy neighbor mitigation, and migration patterns.

---

### Question 1
In a multi-tenant platform API, Tenant A submits 10,000 requests/sec, consuming 100% of the worker thread pool and causing requests from Tenant B and Tenant C to time out. What architectural pattern prevents this "noisy neighbor" starvation?

A) Fair Queueing / Weighted Fair Queuing: maintain per-tenant request queues with concurrency caps and weighted round-robin dequeueing, ensuring no single tenant can starve others.  
B) Rebooting the server every 10 minutes.  
C) Banning Tenant A permanently from the company.  
D) Converting all APIs to synchronous GraphQL.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** A  
**Explanation:** Fair queueing isolates tenant workloads: incoming requests are enqueued into tenant-specific queues. Workers pull round-robin from each queue up to defined concurrency limits, isolating the noisy tenant's blast radius to its own queue.
</details>

---

### Question 2
When migrating a legacy monolithic deployment system to a modern GitOps platform without downtime, which architectural pattern should be employed?

A) Big-Bang Cutover  
B) Strangler Fig Pattern: incrementally migrate small, non-critical services to the new GitOps path while routing legacy services through the old path, gradually strangling the legacy system until 100% of services run on GitOps.  
C) Deleting the legacy system on a Friday night.  
D) Renaming the Git repository.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** B  
**Explanation:** The Strangler Fig pattern replaces pieces of a legacy system incrementally behind an abstraction or facade, validating stability in production with minimal business risk until the old system can be cleanly retired.
</details>

---

### Question 3
Why is naive **Dual-Writing** (an application writing to both the old database and the new database simultaneously) considered dangerous during platform database migrations?

A) Databases do not allow multiple connections.  
B) Dual writing consumes 100% of network cables.  
C) Network partitions or partial failures will inevitably cause writes to succeed on one database and fail on the other, creating silent, inconsistent data divergence without distributed transactions (which introduce severe latency and lock contention).  
D) It violates HTTP standards.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** C  
**Explanation:** Dual-writing without two-phase commit leads to split state. Best practice uses **Change Data Capture (CDC)** (e.g., Debezium querying the database transaction log) or an Outbox pattern to replicate data asynchronously and reliably.
</details>

---

### Question 4
What is the **Expand and Contract** pattern in database schema migrations?

A) Compressing database tables with gzip.  
B) Expanding disk volumes when full.  
C) Backing up databases to tape.  
D) A multi-phase migration pattern: Phase 1 (Expand) adds new columns/tables while maintaining backward compatibility with old code; Phase 2 updates application code to write to both; Phase 3 (Contract) removes old deprecated columns after all services have transitioned.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** D  
**Explanation:** Expand and contract decouples code deployments from database changes. It ensures that running application instances (v1) and newly deployed instances (v2) can operate concurrently without crashing during rolling updates.
</details>

---

### Question 5
What is the difference between returning an **HTTP 429 Too Many Requests** vs an **HTTP 503 Service Unavailable** under high load?

A) There is no difference.  
B) 429 deletes the client's token.  
C) 429 indicates client-specific rate limit quota exhaustion (client should back off and not retry immediately); 503 indicates server-side system overload or maintenance (client should retry with exponential backoff and jitter).  
D) 429 is for GET requests; 503 is for POST requests.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** C  
**Explanation:** 429 tells a specific tenant they breached their contractual quota. 503 signals global server distress, prompting well-behaved clients to back off.
</details>

---

### Question 6
Why must distributed retry mechanisms always incorporate **Exponential Backoff with Jitter**?

A) Without jitter, all retrying clients execute on synchronized intervals (the "Thundering Herd" problem), repeatedly slamming the recovering server with synchronized waves of retry spikes and preventing recovery.  
B) To comply with RFC 2616.  
C) Jitter encrypts the retry payload.  
D) Backoff speeds up the network.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** A  
**Explanation:** Jitter adds randomized delay to retry intervals: $t = \text{base} \times 2^{\text{attempt}} + \text{rand}(0, \text{jitter})$. This desynchronizes client retries across a time continuum, allowing overwhelmed servers to recover smoothly.
</details>

---

### Question 7
In a large-scale Kubernetes cluster, what is the pod density limit on AWS EKS using the standard AWS VPC CNI?

A) Unlimited pods per node.  
B) Governed by the size of the hard drive.  
C) Exactly 10 pods per core.  
D) Bound strictly by the number of Elastic Network Interfaces (ENIs) and secondary IPv4 addresses supported by the physical EC2 instance type: $\text{Max Pods} = \text{ENIs} \times (\text{IPs per ENI} - 1) + 2$.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** D  
**Explanation:** The AWS VPC CNI assigns native VPC IP addresses to pods. Because AWS hardware limits how many ENIs and IPs can attach to an instance, pod density is strictly constrained unless CNI prefix delegation (`/28`) is enabled.
</details>

---

### Question 8
What is the purpose of an **Outbox Pattern** in microservice event publishing?

A) It solves distributed dual-write inconsistency: the application writes its business entity update and an "outbox event" in the same atomic local database transaction; a background CDC process tails the outbox table and reliably publishes events to Kafka.  
B) A pattern for deploying containers.  
C) A tool for sending Slack notifications.  
D) A folder where deleted emails are stored.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** A  
**Explanation:** If an app updates a database and then attempts to send an HTTP/Kafka event, a network failure or crash leaves the message un-sent. Storing the event inside the database transaction guarantees at-least-once message delivery.
</details>

---

### Question 9
In distributed locking (e.g., using Redis or ZooKeeper), why are **Fencing Tokens** required to prevent data corruption?

A) They prevent DDoS attacks.  
B) They restart hung containers.  
C) If a process holding a lock experiences a long Stop-the-World garbage collection pause, its lock lease expires in the lock server; another process acquires the lock; fencing tokens (monotonically increasing numbers) allow storage to reject stale writes from the first process.  
D) They encrypt lock tokens with AES.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** C  
**Explanation:** Martin Kleppmann showed that simple time-based locks are unsafe in asynchronous networks. A process paused by GC can wake up believing it still owns the lock. Fencing tokens increment on every lock grant; storage rejects any write presenting a token lower than the highest seen token.
</details>

---

### Question 10
What is a **Circuit Breaker** in microservice communication?

A) A tool that kills runaway CPU threads.  
B) A software pattern that monitors calls to a downstream dependency; when failure rates exceed a threshold, it "trips" open, failing fast immediately without calling the dependency, allowing the downstream system to recover and preventing cascading failures.  
C) A Kubernetes network policy.  
D) A physical fuse on a server rack.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** B  
**Explanation:** Circuit breakers prevent cascading system collapse. Instead of letting 1,000 threads hang waiting for a slow downstream database (exhausting connection pools), the breaker opens and returns immediate errors or cached fallbacks.
</details>
