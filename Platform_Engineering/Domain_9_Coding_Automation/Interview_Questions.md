# PE Domain 9: Coding & Automation — 25 Core Interview Questions

Master these questions across Coding rounds (Mock 7), Take-Home reviews, and Staff Architecture loops.

---

## Tier 1: Core / Screening Questions (Questions 1–10)

### Q1: What is the difference between buffered and unbuffered channels in Go?
- **Short Answer:** Unbuffered channels are synchronous: sending blocks until a receiver is ready. Buffered channels are asynchronous up to their capacity: sending only blocks when the buffer is full.
- **Deep Dive:** Unbuffered channels guarantee synchronization and delivery. Buffered channels decouple producers from consumers, smoothing bursty traffic.
- **Trade-offs & Realities:** Over-buffering channels hides concurrency bugs and delays backpressure propagation.
- **Follow-up Probes:** "What is the capacity of an unbuffered channel?" (Zero).

### Q2: How does context cancellation work in Go?
- **Short Answer:** Calling `cancel()` closes the context's internal `Done` channel (`<-ctx.Done()`). All goroutines listening to this channel unblock and terminate gracefully.
- **Deep Dive:** Contexts form a tree: cancelling a parent context cancels all descendant child contexts automatically, preventing orphaned goroutines.
- **Trade-offs & Realities:** Always pass `context.Context` as the first argument in functions performing I/O. Never store contexts inside structs.
- **Follow-up Probes:** "What error does `ctx.Err()` return after cancellation?" (`context.Canceled` or `context.DeadlineExceeded`).

### Q3: What is a Worker Pool, and why is it used?
- **Short Answer:** A concurrency pattern that allocates a fixed number of worker threads/goroutines to process tasks from a shared queue, preventing unbounded concurrency from exhausting CPU, memory, or file descriptors.
- **Deep Dive:** Under sudden load spikes (e.g., 10,000 tasks arriving at once), a naive system spawning 10,000 goroutines crashes. A worker pool with 20 workers processes tasks steadily while maintaining predictable memory usage.
- **Trade-offs & Realities:** When the task queue is full, the system must either block the producer or apply backpressure (e.g., returning HTTP 429).
- **Follow-up Probes:** "How do you size a worker pool?" (CPU-bound: $N = \text{CPU cores}$; I/O-bound: higher, e.g., $2 \times \text{cores}$ to $10 \times \text{cores}$).

### Q4: Explain the Token Bucket rate limiting algorithm.
- **Short Answer:** A bucket holds up to $C$ tokens and is refilled at a fixed rate of $R$ tokens/second. A request consumes 1 token. If tokens are available, the request is allowed; if empty, it is rejected.
- **Deep Dive:** Allows bursts up to capacity $C$ while enforcing an average rate limit of $R$. Implemented efficiently in $O(1)$ memory by calculating elapsed time since the last refill rather than running background tickers.
- **Trade-offs & Realities:** Ideal for public API endpoints that need to tolerate bursty developer behavior.
- **Follow-up Probes:** "How is it implemented in Go without background goroutines?" (Calculate `tokens += elapsed * rate` on each incoming request).

### Q5: What is Topological Sort, and where is it used in platform engineering?
- **Short Answer:** An algorithm that linearizes nodes in a Directed Acyclic Graph (DAG) such that for every directed edge $U \to V$, $U$ appears before $V$.
- **Deep Dive:** Essential for resolving deployment dependencies (deploying databases before microservices), Terraform graph evaluation, and CI build DAGs. If in-degrees cannot be resolved, a circular dependency cycle is detected.
- **Trade-offs & Realities:** Only works on acyclic graphs; cycles must be broken before sorting.
- **Follow-up Probes:** "What algorithm implements topological sort in $O(V+E)$ time?" (Kahn's algorithm using in-degrees and queues, or DFS).

### Q6: Why should production HTTP clients avoid `http.DefaultClient` in Go?
- **Short Answer:** `http.DefaultClient` has zero timeout (`Timeout: 0`) and defaults to `MaxIdleConnsPerHost: 2`.
- **Deep Dive:** An unresponsive external server can cause requests to hang indefinitely, leaking goroutines and file descriptors until the service crashes. The low connection pool limit forces sockets into `TIME_WAIT` under concurrent load.
- **Trade-offs & Realities:** Always instantiate custom `http.Client` with explicit connect, TLS, and request timeouts.
- **Follow-up Probes:** "What is an appropriate connection pool size for high-throughput clients?" (`MaxIdleConnsPerHost: 100`).

### Q7: What is the purpose of `sync.WaitGroup`?
- **Short Answer:** A synchronization primitive that blocks execution until a collection of concurrent goroutines finishes executing.
- **Deep Dive:** `wg.Add(n)` increments the counter; each worker calls `defer wg.Done()` to decrement; the main thread calls `wg.Wait()` to block until the counter hits zero.
- **Trade-offs & Realities:** Calling `Add()` with negative numbers or calling `Done()` more times than `Add()` triggers a runtime panic.
- **Follow-up Probes:** "Can a WaitGroup be reused?" (Yes, but only after `Wait()` has completed).

### Q8: What does `defer` do in Go, and in what order do deferred statements execute?
- **Short Answer:** `defer` schedules a function call to execute immediately before the surrounding function returns. Multiple deferred calls execute in Last-In, First-Out (LIFO) order.
- **Deep Dive:** Commonly used for cleanups: unlocking mutexes (`defer mu.Unlock()`), closing files, and closing response bodies.
- **Trade-offs & Realities:** Arguments to deferred functions are evaluated immediately when the `defer` line is evaluated, not when the function returns.
- **Follow-up Probes:** "What happens if a panic occurs inside a function with deferred calls?" (Deferred calls still execute, allowing `recover()` to catch panics).

### Q9: How do you prevent shell injection when running subcommands in Python?
- **Short Answer:** Use `subprocess.run(["cmd", "arg1", "arg2"], shell=False)`. Never pass untrusted user input to `shell=True` or format strings.
- **Deep Dive:** Passing a list of arguments directly invokes the operating system's `execve` system call without launching a shell interpreter (`/bin/sh`), rendering shell metacharacters (`;`, `|`, `&&`) inert.
- **Trade-offs & Realities:** `shell=False` requires arguments to be split into individual list items.
- **Follow-up Probes:** "What library safely splits shell command strings into argument lists in Python?" (`shlex.split()`).

### Q10: What is the difference between a goroutine leak and a memory leak?
- **Short Answer:** A memory leak is un-freed heap memory. A goroutine leak occurs when a goroutine is launched but blocks indefinitely on an unclosed channel or network I/O, retaining its entire call stack in memory forever.
- **Deep Dive:** Leaked goroutines accumulate over time, eventually consuming gigabytes of RAM and causing the OS to kill the process with an OOM error.
- **Trade-offs & Realities:** Avoid goroutine leaks by always bounding network calls with timeouts and passing cancellable contexts.
- **Follow-up Probes:** "What profiling tool detects goroutine leaks in Go?" (`pprof` via `/debug/pprof/goroutine`).

---

## Tier 2: Deep Dive & Mid-Senior Questions (Questions 11–20)

### Q11: Walk through the implementation of Kahn's Algorithm for Topological Sorting.
- **Short Answer:** Compute the in-degree of every vertex; enqueue all vertices with in-degree 0; while the queue is non-empty, dequeue a vertex, append it to the sorted list, and decrement in-degrees of its neighbors; if any neighbor hits 0, enqueue it; if the sorted list length does not match total vertices, a cycle exists.
- **Deep Dive:**
  - Time: $O(V + E)$ where $V$ is vertices and $E$ is edges.
  - Space: $O(V + E)$ to store graph adjacency lists and in-degree maps.
- **Trade-offs & Realities:** Ideal for build dependency resolution because it identifies exactly which nodes caused the cycle.
- **Follow-up Probes:** "How do you print the cycle path if one is detected?" (Use Depth-First Search with recursion stack tracking).

### Q12: How do you build an atomic, distributed rate limiter in Redis?
- **Short Answer:** Use a Lua script to execute the Token Bucket or Sliding Window algorithm atomically within Redis, avoiding multi-step race conditions.
- **Deep Dive:**
  - Sliding Window Counter Lua script:
    1. Key: `ratelimit:{tenant_id}:{window_timestamp}`.
    2. Uses `INCR` to increment current counter.
    3. Fetches previous window counter; computes weighted average: `rate = prev * (1 - fraction) + current`.
    4. Sets TTL on keys (e.g., 2 minutes).
    5. Returns 1 if `rate <= limit`, else 0.
- **Trade-offs & Realities:** Atomic Lua scripts run single-threaded in Redis, guaranteeing zero race conditions with sub-millisecond execution.
- **Follow-up Probes:** "What happens if the Redis cluster experiences a network partition?" (Configure client fallback: fail-open to preserve availability, or fail-closed to protect backend).

### Q13: Explain how Kubernetes client-go uses WorkQueues and Informers together.
- **Short Answer:** Informers listen to watch streams and update local memory caches; when events occur, ResourceEventHandlers extract the key (`namespace/name`) and add it to the WorkQueue; worker goroutines pop keys and reconcile.
- **Deep Dive:** The WorkQueue decouples event reception from processing. It provides rate-limited retries: if `Reconcile()` returns an error, calling `queue.AddRateLimited(key)` re-enqueues the item with exponential backoff (e.g., 5ms, 10ms, 20ms) up to a max limit, preventing API server flooding.
- **Trade-offs & Realities:** The queue coalesces duplicates: if 10 update events occur for `checkout/order-1` while a worker is busy, the key is only enqueued once.
- **Follow-up Probes:** "What is the difference between `Add()`, `AddRateLimited()`, and `AddAfter()`?" (`Add` enqueues immediately; `AddRateLimited` enqueues with backoff; `AddAfter` enqueues after a fixed delay).

### Q14: How do you implement a non-blocking Graceful Shutdown in a Go microservice?
- **Short Answer:** Capture `SIGTERM` and `SIGINT` on an OS signal channel; on signal arrival, call `server.Shutdown(ctx)` with a timeout; drain worker pools; close database connections; exit with code 0.
- **Deep Dive:**
  ```go
  stop := make(chan os.Signal, 1)
  signal.Notify(stop, syscall.SIGTERM, syscall.SIGINT)
  <-stop // Wait for signal

  ctx, cancel := context.WithTimeout(context.Background(), 25*time.Second)
  defer cancel()
  if err := server.Shutdown(ctx); err != nil {
      log.Fatalf("Server forced to shutdown: %v", err)
  }
  ```
- **Trade-offs & Realities:** Must shut down within the Kubernetes `terminationGracePeriodSeconds` (default 30s) before the kernel forcibly terminates the container with `SIGKILL`.
- **Follow-up Probes:** "Why should you stop accepting new traffic before draining connections?" (To ensure in-flight requests finish while new traffic routes to other pods).

### Q15: How do you safely parse and mutate YAML files in Python without losing formatting or comments?
- **Short Answer:** Use `ruamel.yaml` in round-trip mode (`typ="rt"`).
- **Deep Dive:** Standard `PyYAML` discards comments, changes indentation, and sorts keys alphabetically, corrupting human-formatted GitOps repositories. `ruamel.yaml` parses into an annotated syntax tree, allowing targeted updates (e.g., updating an image tag) while preserving original comments and formatting byte-for-byte.
- **Trade-offs & Realities:** `ruamel.yaml` has a steeper API learning curve than simple `json.loads()`.
- **Follow-up Probes:** "How do you preserve block sequence styles in ruamel.yaml?" (Configure `yaml.preserve_quotes = True`).

### Q16: How do you implement Full Jitter in an exponential backoff algorithm, and why is it superior to Equal Jitter?
- **Short Answer:** Full jitter randomizes the sleep duration across the entire backoff range: $t = \text{rand}(0, \text{base} \times 2^{\text{attempt}})$.
- **Deep Dive:** Equal jitter keeps half the delay deterministic: $t = \frac{\text{cap}}{2} + \text{rand}(0, \frac{\text{cap}}{2})$. Amazon's architecture research proved that Full Jitter produces the lowest aggregate client completion times and minimizes server queue depths under severe contention.
- **Trade-offs & Realities:** Full jitter can produce very small initial sleeps (near 0), so establish a reasonable minimum base delay (e.g., 50ms).
- **Follow-up Probes:** "What is Decorrelated Jitter?" (A variant where the next sleep is calculated based on the previous sleep duration rather than attempt count).

### Q17: What are the trade-offs of using Go's `sync.Map` versus a standard `map` with a `sync.RWMutex`?
- **Short Answer:** Use a standard `map` with `RWMutex` for 95% of use cases. Use `sync.Map` only when the map has disjoint keys across goroutines or is strictly write-once, read-many.
- **Deep Dive:** `sync.Map` uses two internal maps (read and dirty) with atomic pointer swapping. Under heavy write contention with changing keys, `sync.Map` locks frequently and suffers from cache misses, performing worse than a standard mutex-protected map.
- **Trade-offs & Realities:** `sync.Map` is untyped (`interface{}` keys and values), sacrificing compile-time type safety.
- **Follow-up Probes:** "When does `sync.Map` outperform `RWMutex`?" (When multiple goroutines read, write, and overwrite a stable set of keys concurrently).

### Q18: How do you design an IP CIDR subnet allocation algorithm in Python?
- **Short Answer:** Use the built-in `ipaddress` module. Parse the root network; maintain an ordered list of available blocks; when allocating a prefix, find the first block with prefix $\le$ target; if smaller, recursively split the block using `block.subnets(new_prefix)` and return the first partition.
- **Deep Dive:** (Refer to Problem 8 in Exercises). Binary tree partitioning (Buddy Allocation) ensures non-overlapping CIDRs and optimal address space utilization.
- **Trade-offs & Realities:** Address fragmentation: allocating small `/28` blocks can fragment large `/20` contiguous blocks needed for future VPCs.
- **Follow-up Probes:** "How do you calculate the broadcast address of `192.168.1.0/24`?" (`192.168.1.255`).

### Q19: Explain the memory allocation behavior of Go slices and how to avoid memory leaks with large slices.
- **Short Answer:** A slice is a 24-byte header: a pointer to an underlying array, a length, and a capacity. Slicing a large slice (`small := large[:2]`) retains a reference to the entire underlying array in memory, preventing garbage collection.
- **Deep Dive:** If you read a 10MB file into memory and extract a 10-byte substring via slicing, the entire 10MB array remains allocated in RAM. Solution: allocate a fresh slice and copy:
  ```go
  small := make([]byte, 2)
  copy(small, large[:2])
  ```
- **Trade-offs & Realities:** Understanding slice capacity doubling: slices grow by $2\times$ up to 256 elements, then grow by $(len + 768) / 4$ to conserve memory.
- **Follow-up Probes:** "What does `append()` do when length exceeds capacity?" (Allocates a new, larger array and copies elements).

### Q20: How do you write a thread-safe Singleton pattern in Go?
- **Short Answer:** Use `sync.Once`.
- **Deep Dive:**
  ```go
  var (
      instance *DatabaseConnection
      once     sync.Once
  )

  func GetInstance() *DatabaseConnection {
      once.Do(func() {
          instance = &DatabaseConnection{}
      })
      return instance
  }
  ```
  `once.Do()` uses fast atomic loads to check if initialization has occurred, avoiding expensive mutex acquisition on subsequent calls.
- **Trade-offs & Realities:** If the initialization function panics, `sync.Once` considers itself completed and will not re-attempt execution.
- **Follow-up Probes:** "Why is naive double-checked locking unsafe in Go without atomic primitives?" (Compiler and CPU memory reordering can allow readers to observe a non-nil pointer before the struct fields are fully written).

---

## Tier 3: Staff & Principal Architect Scenarios (Questions 21–25)

### Q21: Architect a high-performance Kubernetes Custom Controller in Go that manages 100,000 Custom Resources with sub-second reconciliation latency.
- **Candidate Framework:**
  - *Shared Informer Factory:* Use a single Informer cache for common resources; filter watches using label selectors to avoid caching irrelevant objects.
  - *Worker Pool Tuning:* Run 50–100 concurrent worker goroutines popping from the `RateLimitingWorkQueue`.
  - *Batching & Coalescing:* Let the workqueue coalesce rapid duplicate updates for the same key.
  - *Client Rate Limiting:* Tune client-go QPS and Burst (`config.QPS = 100`, `config.Burst = 200`) to prevent client-side throttling.
  - *Optimistic Locking:* Use `Update()` with `ResourceVersion`; handle `409 Conflict` by re-enqueuing with jitter rather than panicking.
  - *Memory Management:* Profile with `pprof`; reuse allocations using `sync.Pool`.
- **Red Flags:** Spawning an unconstrained goroutine per reconcile event or making synchronous API calls without Informer caches.

### Q22: You need to build a distributed Rate Limiter for an enterprise API Gateway handling 2,000,000 RPS across 3 global regions. How do you design it?
- **Candidate Framework:**
  - *Multi-Tier Rate Limiting:*
    - *Tier 1 (In-Memory Local Token Bucket on Envoy):* Enforces 90% of coarse limits locally in memory with zero network latency.
    - *Tier 2 (Regional Redis Cluster):* Envoy proxies query regional Redis clusters via atomic Lua scripts for accurate cross-instance limits.
    - *Tier 3 (Asynchronous Global Synchronization):* Regional limit counters synchronize asynchronously via Kafka/CRDTs; global limits tolerate slight eventual consistency (soft limits) to avoid cross-region network latency on every request.
  - *Fail-Open vs Fail-Closed:* If regional Redis fails, fail open (allow traffic) to preserve user availability while emitting metrics.
- **Red Flags:** Suggesting synchronous cross-region database calls on every incoming HTTP request (adds 150ms+ latency).

### Q23: Design a resilient, automated schema migration engine for a multi-tenant microservice fleet.
- **Candidate Framework:**
  - *Expand-and-Contract Architecture:* Enforce that all schema changes are backward-compatible.
  - *PreSync Migration Jobs:* Execute database schema migrations as Kubernetes `Job` resources via Argo CD PreSync hooks before new application pods are scheduled.
  - *Distributed Advisory Locks:* Migration runner acquires a database advisory lock (`pg_advisory_lock(key)`) to ensure that only one instance of the migration job executes even if multiple pods deploy simultaneously.
  - *Rollback Safety:* Never drop columns in the same release that updates code; deprecate in release $N$, stop reading in $N+1$, drop in $N+2$.
- **Red Flags:** Running database migrations directly inside application container `init()` functions (causes concurrent migration race conditions).

### Q24: How would you design a high-throughput log anomaly detection pipeline processing 500,000 log events/sec in Python or Go?
- **Candidate Framework:**
  - *Ingestion:* Ingest log streams via Apache Kafka partitioned by service domain.
  - *Processing Engine:* Author worker daemon in Go (or Python with PyPy/C-extensions).
  - *Data Structures:* Maintain circular ring buffers or sliding window counters per service; track baseline error rates using EWMA (Exponentially Weighted Moving Average).
  - *Threshold Evaluation:* If current 1-minute error rate exceeds $3\sigma$ above the EWMA baseline, emit an alert event to Slack/PagerDuty.
  - *Memory Protection:* Bound maximum tracked services; evict inactive streams with LRU caches to prevent memory leaks.
- **Red Flags:** Attempting to store every raw log string in Python memory or querying relational databases for sliding window calculations.

### Q25: How do you design an enterprise CLI tool (`pctl`) with plugin architecture and self-updating capabilities?
- **Candidate Framework:**
  - *CLI Framework:* Use Cobra for commands, Viper for configuration management, and pflag for POSIX flags.
  - *Plugin Architecture:*
    - Git-style executable discovery: Any executable on the system PATH matching `pctl-*` is treated as a native subcommand (e.g., executing `pctl-database` when user runs `pctl database`).
    - gRPC HashiCorp go-plugin: For high-performance plugins, run plugins as independent subprocesses communicating over local Unix domain sockets with mutual TLS.
  - *Self-Updating:* Embed semantic version and Git commit SHA; `pctl update` queries GitHub releases API, verifies the SHA256 checksum and Cosign signature of the new binary, and replaces the running executable atomically via `rename()` system call.
- **Red Flags:** Storing unversioned bash scripts or requiring root permissions for standard CLI executions.
