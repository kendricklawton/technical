# PE Domain 9: Coding & Automation — Scenario Quiz 2

Test your knowledge on rate limiting algorithms, graph sorting, and Kubernetes client-go architecture.

---

### Question 1
You are building an API rate limiter. The requirement specifies that users are allowed an average of 100 requests per minute, but short, transient bursts of up to 20 requests in a single second should be accommodated without rejection. Which algorithm should you implement?

A) Token Bucket  
B) Leaky Bucket  
C) Fixed Window Counter  
D) FIFO Queue  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** A  
**Explanation:** The Token Bucket algorithm naturally accommodates bursts up to the bucket capacity while maintaining a steady long-term refill rate. Leaky bucket smooths traffic to a rigid constant rate without burst support.
</details>

---

### Question 2
What is the "Boundary Problem" in the **Fixed Window Counter** rate limiting algorithm?

A) It only works on Linux.  
B) Floating-point numbers cannot be divided.  
C) A user can send their entire quota at the end of Window 1 (e.g., 10:00:59) and another full quota at the start of Window 2 (10:01:01), effectively sending 2x the allowed rate limit within a 2-second burst without being blocked.  
D) It consumes all system RAM.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** C  
**Explanation:** Fixed windows reset counters on boundary edges. A tenant can concentrate requests around the boundary, doubling the allowed burst rate. Sliding window algorithms eliminate this vulnerability.
</details>

---

### Question 3
You have 10 microservices with interdependent deployment dependencies. Which algorithm is required to calculate the valid deployment order and detect circular dependency deadlocks?

A) Dijkstra's Shortest Path  
B) Binary Search  
C) Topological Sort (using Kahn's algorithm or DFS)  
D) QuickSort  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** C  
**Explanation:** Service dependencies form a Directed Acyclic Graph (DAG). Topological sort linearizes nodes so every dependency appears before its dependent. If in-degrees cannot be reduced to zero, a cycle exists.
</details>

---

### Question 4
In Kubernetes `client-go`, what does a **WorkQueue** provide that a standard Go channel lacks?

A) It bypasses RBAC.  
B) It provides rate-limiting, deduplication (ensuring a key is not processed concurrently by multiple workers), and failure re-queueing with exponential backoff.  
C) It stores data on tape.  
D) It converts JSON to YAML.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** B  
**Explanation:** `workqueue.RateLimitingInterface` coalesces duplicate events for the same resource key, prevents parallel reconciliation of the same item, and retries failed reconciles with exponential backoff.
</details>

---

### Question 5
What is the time complexity of Topological Sort using Kahn's algorithm on a graph with $V$ vertices and $E$ edges?

A) $O(V + E)$  
B) $O(V^2)$  
C) $O(\log V)$  
D) $O(V \times E)$  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** A  
**Explanation:** Kahn's algorithm visits every vertex once and decrements in-degrees for every edge once, achieving optimal linear time complexity $O(V + E)$.
</details>

---

### Question 6
Why should systems automation tools use `json.Unmarshal` into strongly typed structs rather than `map[string]interface{}` whenever possible?

A) Strongly typed structs provide compile-time type safety, avoid runtime type assertion panics, consume less memory, and validate that required schema fields conform to specifications.  
B) Structs run 100x faster than maps.  
C) Maps cannot hold strings.  
D) Go does not support JSON parsing with maps.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** A  
**Explanation:** Parsing into `map[string]interface{}` requires verbose, dangerous type assertions (`m["key"].(string)`) that panic if the field is missing or typed differently. Structs guarantee safety.
</details>

---

### Question 7
In the Sliding Window Log rate limiter, what is the primary operational trade-off compared to the Token Bucket algorithm?

A) Sliding Window Log cannot run in Redis.  
B) Token bucket requires GPUs.  
C) Sliding Window Log is less accurate.  
D) Sliding Window Log is 100% accurate because it stores the exact timestamp of every request, but its memory footprint is $O(N)$ requests, making it expensive under high throughput compared to Token Bucket's $O(1)$ memory.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** D  
**Explanation:** Storing millions of individual timestamps in a sorted set (ZSET in Redis) consumes significant memory. Token bucket only stores two numbers (token count and timestamp), achieving $O(1)$ memory.
</details>

---

### Question 8
What is the function of the **Cobra** library in Go platform tooling?

A) It compiles Python code to Go.  
B) It provides cryptographic encryption.  
C) It routes network packets.  
D) It is the industry standard framework for building modern command-line interfaces (CLIs) with nested subcommands, POSIX flags, and auto-generated help documentation (used by `kubectl`, `helm`, `hugo`).  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** D  
**Explanation:** Cobra powers almost all major cloud-native CLIs. It organizes applications into commands, arguments, and flags with built-in validation.
</details>

---

### Question 9
In Kubernetes controller design, what does `reconcile.Result{RequeueAfter: 5 * time.Minute}` do?

A) It deletes the resource after 5 minutes.  
B) It reboots the pod.  
C) It pauses the controller process for 5 minutes.  
D) It schedules the current resource key to be re-enqueued into the work queue after 5 minutes, allowing periodic health checks without blocking worker threads.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** D  
**Explanation:** `RequeueAfter` is non-blocking: worker threads continue processing other keys, and the runtime enqueues the item again once the timer fires.
</details>

---

### Question 10
What is the purpose of an **Exponential Backoff with Full Jitter** retry algorithm in network automation scripts?

A) It logs network errors to disk.  
B) It spreads out retried requests randomly over an exponentially expanding time range, preventing synchronized waves of client requests from overwhelming a recovering service.  
C) It doubles the network bandwidth on each retry.  
D) It compresses HTTP headers.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** B  
**Explanation:** Full jitter randomizes the sleep duration: $t = \text{rand}(0, \text{base} \times 2^{\text{attempt}})$, ensuring requests disperse evenly across time.
</details>
