# Domain 9: Coding & Automation — Essential Technical Reference & Architecture Guide

> **Core Focus:** Production Systems Programming in Go and Python, Concurrent Programming (Goroutines, Channels, Asyncio), Resilient HTTP Clients, Backoff & Jitter, Kubernetes Controllers (`client-go`), and Twelve-Factor App Principles.

---

## 1. Authoritative References & Architectural Standards

### 1.1 [Writing Kubernetes Controllers with controller-runtime and client-go](https://github.com/kubernetes-sigs/controller-runtime)
* **Canonical Scope:** Standard framework for building production Kubernetes operators in Go.
* **Core Mechanics:**
  * `Manager`: Manages shared dependencies (clients, caches, schemes, webhooks).
  * `Reconciler`: Implements business logic; must be strictly idempotent and return `reconcile.Result{RequeueAfter: ...}` on transient errors.
  * `Informer Cache`: Local read-only cache backed by etcd watches; writes go through direct API client (`client.Client`).

### 1.2 [Exponential Backoff And Jitter (Amazon Builders' Library)](https://aws.amazon.com/builders-library/exponential-backoff-and-jitter/)
* **Canonical Scope:** Eliminating the "thundering herd" problem in distributed systems.
* **Algorithms:**
  * Full Jitter formula:
    $$t = \text{random}(0, \min(M, B \times 2^{\text{attempt}}))$$
  * *Why Jitter is Mandatory:* Without jitter, all failing clients retry at the exact same synchronized intervals, hammering an already failing service and preventing recovery.

### 1.3 [The Twelve-Factor App Methodology](https://12factor.net/)
* **Canonical Scope:** Software engineering methodology for building cloud-native, scalable applications.
* **Core Factors:** Codebase in VCS, Explicit dependencies, Config in environment, Backing services as attached resources, Strict build/release/run separation, Stateless processes, Port binding, Concurrency via process model, Fast startup and graceful shutdown (`SIGTERM`), Dev/prod parity, Logs as event streams.

---

## 2. Systems Programming Patterns (Go & Python)

### 2.1 Resilient Go Primitives
* **Context Propagation:** Passing `context.Context` to all network and database operations to propagate cancellation signals and deadlines.
* **Worker Pools:** Limiting concurrent goroutine execution using buffered channels to prevent memory exhaustion.
* **Graceful Shutdown:** Intercepting `SIGTERM` / `SIGINT`, stopping listeners, waiting for active requests to finish with `sync.WaitGroup`, and closing database connection pools cleanly.

### 2.2 Resilient Python Primitives
* Using `asyncio` event loops for high-throughput I/O bound workloads.
* Typing with `mypy` and structured validation with `Pydantic`.

---

## 3. Staff-Level Interview Angle & Architectural Reflexes

1. **"Deadlock in Go Channels":**
   * *Common Bug:* Writing to an unbuffered channel when there is no active receiver goroutine blocks indefinitely.
   * *Reflex:* Always pair channel sends/receives with `select` statements incorporating `ctx.Done()` or timeouts.
2. **Graceful Shutdown in Kubernetes:**
   * A pod receives `SIGTERM`. If the application exits immediately, clients receive HTTP 502 Bad Gateway errors because kube-proxy and cloud load balancers haven't removed the pod IP from iptables yet!
   * *Resolution:* Catch `SIGTERM`, sleep for 5–10 seconds to allow endpoint propagation, stop accepting new requests, drain existing connections, and exit 0.

---

## 4. Reading & Study Checklist

- [ ] Read *Exponential Backoff And Jitter (Builders' Library)*
- [ ] Review *Twelve-Factor App Methodology*
- [ ] Master Go concurrency patterns: worker pools, errgroup, and context cancellation
- [ ] Study `client-go` Informer and Reconcile loop architecture
- [ ] Implement a production-grade graceful HTTP server shutdown in Go and Python
