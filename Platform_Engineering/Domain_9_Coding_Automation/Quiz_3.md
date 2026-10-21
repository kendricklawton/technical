# PE Domain 9: Coding & Automation — Scenario Quiz 3

Test your mastery of HTTP client tuning, connection pools, and CIDR subnet slicing.

---

### Question 1
A Go microservice makes 500 outbound HTTP requests per second to a backend payment gateway using `http.DefaultClient`. Under load, the service exhausts all ephemeral ports and throws `bind: address already in use` errors. What is the root cause in `http.DefaultClient`?

A) `http.DefaultClient` uses a default transport where `MaxIdleConnsPerHost` is set to only **2**; subsequent concurrent connections cannot be reused from the pool and are closed immediately, leaving thousands of sockets in the `TIME_WAIT` state and exhausting ports.  
B) Go does not support HTTP/1.1.  
C) DNS queries timed out.  
D) The client did not use HTTPS.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** A  
**Explanation:** In Go's `http.DefaultTransport`, `MaxIdleConnsPerHost` defaults to 2. If you execute 500 concurrent requests, 498 connections are aggressively closed after use, creating massive `TIME_WAIT` socket storms. Solution: create a custom transport with `MaxIdleConnsPerHost: 100`.
</details>

---

### Question 2
Why must an engineer always read and close the HTTP response body in Go: `defer resp.Body.Close(); io.Copy(io.Discard, resp.Body)`?

A) To free local disk space.  
B) To prevent memory leaks in the browser.  
C) It is only required for POST requests.  
D) If the response body is not fully read to EOF and closed, the underlying TCP connection cannot be reused by the HTTP transport connection pool and is closed, defeating connection pooling.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** D  
**Explanation:** For the Go HTTP transport to return a TCP connection to the idle pool, the connection must be clean (read to EOF). Neglecting to drain and close the body forces the transport to tear down the connection.
</details>

---

### Question 3
In network automation, given an IPv4 network `10.0.0.0/16`, how many `/24` subnets can be carved out of this block?

A) 16 subnets  
B) 65,536 subnets  
C) 256 subnets  
D) 8 subnets  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** C  
**Explanation:** Prefix difference = $24 - 16 = 8$ bits. Total subnets = $2^8 = 256$ subnets (each containing 256 IP addresses: $256 \times 256 = 65,536$ total IPs).
</details>

---

### Question 4
In Python, what is the advantage of using `sys.exit(0)` vs raising an unhandled exception in an automation CLI tool?

A) Unhandled exceptions run faster.  
B) `sys.exit(0)` sets the proper POSIX process exit status code (0 for success, non-zero for error), allowing shell scripts and CI/CD pipelines to evaluate success/failure reliably without printing messy tracebacks.  
C) `sys.exit(0)` reboots the machine.  
D) Unhandled exceptions automatically retry.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** B  
**Explanation:** Automation tools must conform to POSIX exit standards. Exit code 0 signals success; non-zero signals failure, enabling shell `set -e` and CI runners to detect errors deterministically.
</details>

---

### Question 5
What is the primary risk of using `time.Sleep()` inside a multi-threaded or asynchronous worker loop?

A) It crashes the operating system.  
B) In synchronous threads, `time.Sleep()` blocks the thread from performing any other work; in asynchronous event loops (like Python `asyncio`), calling synchronous `time.sleep()` blocks the entire single-threaded event loop, freezing all concurrent coroutines.  
C) It cancels the context.  
D) It uses 100% CPU.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** B  
**Explanation:** In async code, you must use `await asyncio.sleep()`. Calling synchronous `time.sleep()` halts the thread, preventing the event loop from scheduling other pending I/O tasks.
</details>

---

### Question 6
When manipulating YAML configuration files programmatically in Python, why is **`ruamel.yaml`** preferred over standard `PyYAML`?

A) `ruamel.yaml` encrypts all passwords.  
B) `ruamel.yaml` is written in C++.  
C) PyYAML cannot parse Kubernetes manifests.  
D) `ruamel.yaml` is a round-trip YAML parser that preserves original comments, formatting, and key order when modifying values, preventing automated tools from stripping human comments from Git manifests.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** D  
**Explanation:** PyYAML strips all comments and reorders keys during dump. `ruamel.yaml` preserves the full AST (comments, quotes, indentations), making it ideal for automated GitOps configuration updates.
</details>

---

### Question 7
In Go, why should `time.After()` be avoided inside a long-running `select` loop?

A) `time.After()` creates a new `time.Timer` on every loop iteration that cannot be garbage-collected until the duration expires; in rapid loops, this leaks thousands of timers in memory.  
B) It crashes the garbage collector.  
C) It reboots the computer.  
D) `time.After()` cannot be formatted.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** A  
**Explanation:** `time.After()` allocates memory that lives until the timer fires. Inside a fast loop, use `time.NewTimer()` and reset it manually to prevent timer leaks.
</details>

---

### Question 8
What does the `context.WithCancel()` function return in Go?

A) A boolean value.  
B) A derived child `Context` and a `CancelFunc`; invoking `cancel()` closes the context's `.Done()` channel, notifying all descendant goroutines to abort work immediately.  
C) An HTTP response.  
D) A network socket.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** B  
**Explanation:** Context cancellation allows graceful cascading aborts across trees of goroutines when an operation is cancelled or times out.
</details>

---

### Question 9
In IP networking, how many usable host addresses are available in a standard `/28` IPv4 subnet?

A) 16 hosts  
B) 28 hosts  
C) 14 hosts (or 11 in AWS)  
D) 256 hosts  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** C  
**Explanation:** Total IPs in $/28 = 2^{32-28} = 2^4 = 16$. In standard networking, 2 are reserved (network address and broadcast address), leaving **14 usable**. In AWS VPCs, AWS reserves 5 IPs (network, router, DNS, future use, broadcast), leaving **11 usable**.
</details>

---

### Question 10
What tool can be used to compile Go code into ultra-minimal, single-binary container images with zero operating system dependencies?

A) Ubuntu Desktop  
B) Docker compose  
C) `CGO_ENABLED=0 GOOS=linux go build -ldflags="-w -s"` combined with `FROM scratch`  
D) Python pip  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** C  
**Explanation:** Disabling CGO produces a statically linked binary with zero C library dependencies, allowing it to execute inside an empty `scratch` or `distroless` container image with a total size under 15MB.
</details>
