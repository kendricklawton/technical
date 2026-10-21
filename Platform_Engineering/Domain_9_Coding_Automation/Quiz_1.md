# PE Domain 9: Coding & Automation — Scenario Quiz 1

Test your knowledge on Go concurrency, channels, and context cancellation.

---

### Question 1
In Go, what happens if a goroutine attempts to write to an unbuffered channel when there is no active receiving goroutine ready?

A) The sending goroutine blocks execution indefinitely until another goroutine initiates a read on the channel (or causes a deadlock panic if all goroutines are asleep).  
B) The data is stored in an in-memory temporary queue.  
C) The data is silently discarded.  
D) The channel automatically converts into a buffered channel.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** A  
**Explanation:** Unbuffered channels are synchronous. A send operation blocks until another goroutine executes a receive, facilitating rendezvous synchronization. If no receiver arrives, the goroutine leaks.
</details>

---

### Question 2
What is the primary operational consequence of omitting `defer cancel()` when creating a context with a timeout: `ctx, cancel := context.WithTimeout(parent, 5*time.Second)`?

A) Goroutines run 10x slower.  
B) The program crashes with a segmentation fault.  
C) The network interface drops packets.  
D) The internal timer associated with the context remains active until the 5-second deadline expires, retaining references in memory and leaking resources even if the parent function finished in 2 milliseconds.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** D  
**Explanation:** `WithTimeout` schedules a timer in the Go runtime. Calling `cancel()` releases the timer resources immediately. Omitting it leaks the timer until it expires naturally, wasting memory under high call volumes.
</details>

---

### Question 3
Why should shared state in multi-threaded Go programs be protected with `sync.RWMutex` rather than a standard `sync.Mutex` when read operations heavily outnumber write operations?

A) `RWMutex` permits multiple concurrent readers to hold the lock simultaneously (`RLock()`), blocking only when an exclusive writer (`Lock()`) is active, drastically reducing lock contention in read-heavy caches.  
B) Standard `Mutex` does not work with pointers.  
C) `RWMutex` consumes less disk space.  
D) `RWMutex` automatically detects deadlocks.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** A  
**Explanation:** In read-heavy systems (e.g., 99% reads, 1% writes), a standard `Mutex` serializes all reads, crippling multi-core performance. `RWMutex` allows concurrent read access.
</details>

---

### Question 4
In Go, what does a `select` statement with a `default` case accomplish?

A) It catches runtime panics.  
B) It enables non-blocking channel communication: if none of the channel cases are ready to communicate, the `default` case executes immediately rather than blocking the goroutine.  
C) It converts the channel into a buffered channel.  
D) It reboots the application.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** B  
**Explanation:** A `select` without `default` blocks until one channel is ready. Adding a `default` case makes the select non-blocking, allowing goroutines to poll or shed work if a channel is full.
</details>

---

### Question 5
What is the danger of closing a Go channel from a receiver goroutine when multiple concurrent producer goroutines are writing to it?

A) The data is encrypted.  
B) Memory usage drops to zero.  
C) The channel buffer overflows.  
D) Attempting to send on a closed channel causes an immediate, unrecoverable runtime panic (`panic: send on closed channel`).  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** D  
**Explanation:** In Go, the channel owner (the sender) should be the one to close the channel. Sending on a closed channel triggers a panic. Only close channels when you are certain no more writes will occur.
</details>

---

### Question 6
What is the function of `sync.Once` in Go?

A) It limits a loop to 1 iteration.  
B) It creates a single-use variable.  
C) It guarantees that a specific initialization function is executed exactly once across multiple concurrent goroutines, performing atomic synchronization safely without manual lock management.  
D) It restricts execution to a single CPU core.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** C  
**Explanation:** `sync.Once` is used for singleton initialization (e.g., setting up database connections or compiling regular expressions once on startup).
</details>

---

### Question 7
How does `GOMAXPROCS` affect Go runtime execution?

A) It limits the number of operating system threads that can execute user-level Go code simultaneously, defaulting to the number of logical CPU cores visible to the process.  
B) It limits the number of goroutines to 100.  
C) It limits the maximum number of network sockets.  
D) It sets the maximum memory limit.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** A  
**Explanation:** `GOMAXPROCS` controls the number of OS threads executing goroutines concurrently. In containerized environments with CPU limits, failing to tune `GOMAXPROCS` to match container quotas can lead to CFS throttling.
</details>

---

### Question 8
What does the `sync.Pool` type provide in high-throughput Go services?

A) A database connection pool.  
B) A thread pool for worker tasks.  
C) A list of active servers.  
D) A cache of temporary, reusable objects (e.g., byte buffers) that can be shared across goroutines to reduce allocation overhead and minimize garbage collection (GC) pressure.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** D  
**Explanation:** `sync.Pool` recycles memory objects. Instead of allocating a new 64KB buffer for every incoming HTTP request, the application retrieves a buffer from `sync.Pool` and returns it when finished, sparing the GC from allocating and freeing millions of small objects.
</details>

---

### Question 9
In Go, how do you detect race conditions during automated test execution?

A) By running tests inside Docker.  
B) By executing `go test -race ./...`, which compiles the code with the ThreadSanitizer instrumentation to detect unsynchronized concurrent memory access at runtime.  
C) By reading the test logs manually.  
D) By disabling all goroutines.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** B  
**Explanation:** The Go race detector (`-race`) instruments memory reads and writes, immediately flagging any concurrent read/write or write/write access to shared memory lacking proper synchronization.
</details>

---

### Question 10
What is the result of reading from a closed channel in Go?

A) It returns an EOF error.  
B) The goroutine blocks indefinitely.  
C) It returns the zero value for the channel's type immediately (along with `ok == false` if using comma-ok syntax: `val, ok := <-ch`), draining any remaining buffered items first.  
D) It triggers an immediate runtime panic.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** C  
**Explanation:** Reading from a closed channel does not panic. If buffered items remain, it drains them; once empty, it yields zero values immediately with `ok = false`.
</details>
