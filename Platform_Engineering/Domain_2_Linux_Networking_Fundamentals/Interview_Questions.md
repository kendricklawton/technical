# PE Domain 2: Linux & Networking — 25 Core Interview Questions

Master these questions across Technical Screening, Live Troubleshooting, and Senior Systems Architecture rounds.

---

## Tier 1: Core / Screening Questions (Questions 1–10)

### Q1: What happens under the hood when a Linux process is killed by the OOM killer?
- **Short Answer:** When available RAM and swap are exhausted and cannot be reclaimed from caches, the kernel calls `out_of_memory()`, computes an `oom_score` for all processes, selects the process with the highest score, and terminates it unconditionally with `SIGKILL` (signal 9).
- **Deep Dive:** The kernel ranks processes using memory consumption proportional to total RAM, adjusted by `/proc/[pid]/oom_score_adj`. In Kubernetes, `BestEffort` pods receive `+1000`, `Burstable` receive proportional scores, and `Guaranteed` receive `-997`. The killed process exits with code 137 ($128 + 9$).
- **Trade-offs & Realities:** Increasing memory limits masks memory leaks. Platform engineers must identify whether the issue is an organic growth pattern or a code leak using heap dumps and metrics before blindly increasing limits.
- **Follow-up Probes:** "Can a process intercept the OOM kill signal?" (No, `SIGKILL` cannot be caught or handled).

### Q2: What is the difference between a process and a thread in the Linux kernel?
- **Short Answer:** To the Linux kernel scheduler, both are `task_struct` instances. A thread is simply a process that shares its virtual memory space, file descriptor table, and signal handlers with its parent via flags passed to `clone()`.
- **Deep Dive:** Linux uses the 1:1 threading model (NPTL). System calls like `fork()` clone tasks with separate address spaces, while `pthread_create()` invokes `clone()` with flags `CLONE_VM`, `CLONE_FS`, `CLONE_FILES`, and `CLONE_SIGHAND`.
- **Trade-offs & Realities:** Threads are cheaper to context switch because they share page tables (avoiding TLB flushes), but a memory corruption bug in one thread crashes the entire process.
- **Follow-up Probes:** "How do you view threads of a process in Linux?" (`ps -T -p <pid>` or `top -H -p <pid>`).

### Q3: Explain the Completely Fair Scheduler (CFS) CPU quota and period mechanics.
- **Short Answer:** CFS limits container CPU bandwidth using `cpu.cfs_quota_us` within a recurring time window `cpu.cfs_period_us` (typically 100ms). When a container consumes its quota, the kernel throttles (freezes) it until the next period.
- **Deep Dive:** A limit of `1.5 CPUs` sets `quota = 150000us` and `period = 100000us`. If multiple threads run simultaneously across multiple cores, they exhaust the quota in a fraction of the period, leading to latency spikes.
- **Trade-offs & Realities:** Many organizations remove CPU limits on latency-critical microservices, relying strictly on CPU `requests` with node autoscaling to avoid CFS throttling bugs.
- **Follow-up Probes:** "Where do you check if a container is being throttled?" (`/sys/fs/cgroup/cpu/cpu.stat` -> `nr_throttled`).

### Q4: What is the difference between `SIGTERM` and `SIGKILL`?
- **Short Answer:** `SIGTERM` (signal 15) is a polite termination request that can be caught and handled by the process to perform graceful shutdown. `SIGKILL` (signal 9) is an unconditional kernel termination that cannot be caught, blocked, or ignored.
- **Deep Dive:** During graceful shutdown, a process catches `SIGTERM`, stops accepting new network connections, drains in-flight requests, commits database transactions, closes open file handles, and exits with code 0.
- **Trade-offs & Realities:** If an application ignores `SIGTERM`, Kubernetes waits for `terminationGracePeriodSeconds` (default 30s) before issuing `SIGKILL`, slowing down deployments.
- **Follow-up Probes:** "Why should containers handle `SIGTERM`?" (To prevent data loss, dropped client requests, and slow rollouts).

### Q5: What are the primary Linux namespaces used to implement containers?
- **Short Answer:** PID (processes), NET (networking), MNT (filesystems), IPC (inter-process communication), UTS (hostnames), USER (user/group IDs), CGROUP (cgroup view), and TIME (clocks).
- **Deep Dive:** Namespaces partition global kernel resources. When a process queries its network interfaces or process tree, it only sees the subset allocated to its namespace.
- **Trade-offs & Realities:** Namespaces provide logical isolation, not security virtualization. Kernel exploits in shared kernel subsystems (like eBPF or io_uring) can breach namespaces.
- **Follow-up Probes:** "What tool lets you execute commands inside another container's namespace?" (`nsenter -t <pid> -n -m /bin/sh`).

### Q6: What does the `TIME_WAIT` TCP socket state mean, and why is it important?
- **Short Answer:** It represents a socket closed by the local endpoint that remains open for $2 \times \text{MSL}$ (60 seconds) to ensure all delayed packets in transit die before the same 4-tuple is reused.
- **Deep Dive:** `TIME_WAIT` prevents old delayed duplicate packets from corrupting new connections. It also ensures the remote peer receives the final ACK; if the final ACK was lost, the remote peer will retransmit its FIN, which can only be acknowledged if the local socket is still in `TIME_WAIT`.
- **Trade-offs & Realities:** High connection churn on proxies can cause `TIME_WAIT` storms, leading to ephemeral port exhaustion. Fixed via HTTP connection pooling and `net.ipv4.tcp_tw_reuse = 1`.
- **Follow-up Probes:** "What is the difference between `TIME_WAIT` and `CLOSE_WAIT`?" (`TIME_WAIT` is active close and normal; `CLOSE_WAIT` is passive close and indicates an application descriptor leak).

### Q7: What is the difference between Resident Set Size (RSS) and Virtual Memory (VIRT)?
- **Short Answer:** VIRT is the total memory space mapped by the process (including files, shared libraries, and uncommitted allocations). RSS is the actual physical RAM currently occupied by the process.
- **Deep Dive:** A process can map a 10GB file using `mmap()`, causing VIRT to jump to 10GB, but if it only reads 5MB, RSS remains 5MB. Cgroup memory limits and OOM killers evaluate RSS and page cache, not VIRT.
- **Trade-offs & Realities:** Developers often panic seeing high VIRT in Java or Go applications; platform engineers look at RSS to determine actual node memory impact.
- **Follow-up Probes:** "What is SHR in `top`?" (Shared memory used by shared libraries like `libc.so`).

### Q8: What causes the `ndots:5` DNS performance penalty in Kubernetes?
- **Short Answer:** Kubernetes sets `ndots:5` in `/etc/resolv.conf`. If an external domain has fewer than 5 dots (e.g., `api.stripe.com` has 2), the resolver queries all internal search domains first, generating up to 4 failed lookups before resolving the external domain.
- **Deep Dive:** Each failed lookup queries the cluster CoreDNS servers, adding 10–50ms of network latency per lookup and overloading DNS pods.
- **Trade-offs & Realities:** Resolved by adding a trailing dot in application URLs (`api.stripe.com.`), setting `ndots:2` in pod specs, or using NodeLocal DNSCache.
- **Follow-up Probes:** "Why does Kubernetes set `ndots:5` by default?" (To allow short internal service queries like `service.namespace`).

### Q9: What is a veth pair in container networking?
- **Short Answer:** A virtual ethernet (veth) pair is a virtual cable connecting two Linux network namespaces. Packets transmitted on one interface emerge immediately from the other.
- **Deep Dive:** The CNI creates the pair, places one interface inside the pod as `eth0`, and keeps the peer interface in the host root namespace connected to a bridge or routed via eBPF.
- **Trade-offs & Realities:** Moving packets through veth pairs involves kernel context switches. Technologies like Cilium eBPF bypass parts of the network stack to optimize throughput.
- **Follow-up Probes:** "How do you trace packets on a pod's veth interface from the host?" (`tcpdump -i vethXXXX`).

### Q10: What is the difference between an L4 and an L7 proxy?
- **Short Answer:** L4 operates at the transport layer (TCP/UDP), forwarding packets without terminating TLS or inspecting application payloads. L7 operates at the application layer (HTTP/gRPC), terminating TLS and parsing headers, paths, and cookies.
- **Deep Dive:** L4 proxies (AWS NLB, IPVS) provide raw speed and massive concurrency with low CPU usage. L7 proxies (Envoy, NGINX) allow advanced routing, canary deployments, header transformations, and circuit breaking.
- **Trade-offs & Realities:** L7 proxies incur higher CPU and latency overhead due to TLS termination and HTTP parsing.
- **Follow-up Probes:** "Can an L4 proxy inspect HTTP request paths?" (No; L4 has no awareness of HTTP protocols).

---

## Tier 2: Deep Dive & Mid-Senior Questions (Questions 11–20)

### Q11: Walk through the 18 steps of executing `curl https://api.internal.company.com/v1/health` from inside a Kubernetes Pod.
- **Short Answer:** (Refer to Section 9 of the Study Guide for the complete breakdown).
- **Key Points Interviewer Looks For:**
  1. Syscall `socket()` and `getaddrinfo()`.
  2. Resolv.conf evaluation and `ndots` search paths.
  3. CoreDNS resolution to ClusterIP.
  4. TCP SYN creation in Pod netns.
  5. Veth pair traversal to host netns.
  6. Iptables / eBPF DNAT from ClusterIP to backend Pod IP.
  7. Routing table lookup and CNI encapsulation (VXLAN/Geneve or direct VPC route).
  8. Physical network transit across switches/routers.
  9. Node B decapsulation and target Pod socket ingress.
  10. TCP 3-way handshake completion.
  11. TLS 1.3 handshake (ClientHello with SNI, ServerHello with key share, encrypted certificate).
  12. HTTP GET request execution, application processing, 200 OK response, and teardown.

### Q12: Why does `CLOSE_WAIT` build up on a server, and how do you fix it?
- **Short Answer:** `CLOSE_WAIT` indicates the remote client closed the connection (sent FIN), the OS acknowledged it, but the local application process has not invoked `close()` on the socket descriptor.
- **Deep Dive:** This is exclusively an application-level bug: unhandled exceptions in worker threads, leaked connection pool handles, or deadlocks in event loops. The OS cannot close the socket because the application might still want to send data.
- **Trade-offs & Realities:** Tuning kernel sysctls will not fix `CLOSE_WAIT`. You must fix the application code or restart the process.
- **Follow-up Probes:** "What tool identifies which process owns the leaking socket?" (`ss -tap | grep CLOSE_WAIT` or `lsof -i :<port>`).

### Q13: Compare kube-proxy in `iptables` mode vs `ipvs` mode vs Cilium `eBPF`.
- **Short Answer:** `iptables` evaluates rules sequentially ($O(N)$) and suffers from lock contention during updates. `IPVS` uses kernel hash tables ($O(1)$) with better scaling up to 50k services. `nftables` mode (GA since Kubernetes 1.33) uses verdict maps for $O(1)$ lookups with incremental updates, and is the intended successor to `iptables` mode, which is still the default. `Cilium eBPF` bypasses Netfilter entirely, executing bytecode programs at socket and tc layers with sub-microsecond latency and $O(1)$ scaling.
- **Deep Dive:** At enterprise scale (10,000+ endpoints), kube-proxy iptables rule flushes cause noticeable CPU spikes and dropped packets. Cilium replaces kube-proxy with eBPF maps that update atomically without table locks.
- **Trade-offs & Realities:** Cilium requires modern Linux kernels (5.4+) and eBPF debugging skills.
- **Follow-up Probes:** "What is XDP in eBPF networking?" (eXpress Data Path: executes eBPF bytecode directly at the network card driver layer before kernel packet allocation).

### Q14: Explain the Linux OOM score calculation and how Kubernetes manipulates it.
- **Short Answer:** The kernel calculates `oom_score` based on memory usage as a fraction of total RAM (0–1000) and adds `/proc/[pid]/oom_score_adj` (-1000 to +1000). Kubernetes assigns `oom_score_adj` according to QoS: Guaranteed (-997), Burstable (calculated based on request/limit ratio), BestEffort (+1000).
- **Deep Dive:** In node-level memory pressure, the kernel kills processes with the highest combined score. System daemons (kubelet, sshd) are set to `-1000` to prevent them from ever being killed.
- **Trade-offs & Realities:** If a Burstable pod consumes nearly 100% of its memory limit, its score approaches 1000, making it an immediate target.
- **Follow-up Probes:** "Why is Guaranteed set to -997 instead of -1000?" (To preserve -1000 exclusively for critical host-level daemons like kubelet and containerd).

### Q15: How does TLS 1.3 achieve 1-RTT handshake compared to TLS 1.2's 2-RTT?
- **Short Answer:** TLS 1.3 dropped legacy cipher negotiation and mandates ephemeral Diffie-Hellman (ECDHE). The client speculatively includes its public key share in the `ClientHello`, allowing the server to calculate the shared encryption key and reply with encrypted certificates in the `ServerHello`.
- **Deep Dive:** In TLS 1.2, round-trip 1 negotiates ciphers, and round-trip 2 performs key exchange. TLS 1.3 collapses this into a single exchange. TLS 1.3 also supports 0-RTT resumption using pre-shared keys.
- **Trade-offs & Realities:** 0-RTT data is vulnerable to replay attacks; it should only be used for idempotent HTTP GET requests.
- **Follow-up Probes:** "What is the security risk of TLS 1.2 RSA key exchange?" (Lack of Forward Secrecy: if the server's private key is ever stolen, all historical recorded traffic can be decrypted).

### Q16: How do you debug packet drops in the Linux kernel?
- **Short Answer:** Check network interface drops with `ip -s link`, inspect protocol-level drops with `netstat -s` or `nstat`, examine Netfilter drops with `iptables -nvL`, and use eBPF tools like `dropwatch` or `pwru` (Packet Where aRe yoU).
- **Deep Dive:** `pwru` leverages eBPF kprobes to trace an individual packet through every kernel function (e.g., `ip_rcv`, `nf_hook_slow`, `tcp_v4_rcv`), highlighting the exact line of kernel code that returned `NET_RX_DROP`.
- **Trade-offs & Realities:** Running raw tcpdump under 40Gbps load can cause CPU spikes and packet loss; eBPF filters allow targeted kernel tracing with minimal overhead.
- **Follow-up Probes:** "What does `net.core.somaxconn` control?" (The maximum length of the listen queue for fully established TCP sockets waiting for `accept()`).

### Q17: What is MTU black-holing in overlay networks, and how do you resolve it?
- **Short Answer:** When overlay networks (VXLAN/Geneve) add encapsulation headers (typically 50 bytes) to an already maximum-sized packet (1500 bytes), the packet exceeds physical MTU. If routers drop it with the Don't Fragment (DF) bit set and Path MTU Discovery (PMTUD) fails, connections hang silently.
- **Deep Dive:** Symptoms: TCP handshakes and small curls succeed, but large HTTP payloads or file transfers hang indefinitely.
- **Trade-offs & Realities:** Resolved by configuring CNI MTU to 1450 (or enabling Jumbo Frames on physical networks: MTU 9000) and enabling TCP MSS clamping in iptables (`-j TCPMSS --clamp-mss-to-pmtu`).
- **Follow-up Probes:** "What is MSS in TCP?" (Maximum Segment Size: `MTU - 40 bytes [IP header + TCP header]`).

### Q18: What is cgroups v2 Pressure Stall Information (PSI), and how does it improve observability?
- **Short Answer:** PSI measures the exact percentage of wall-clock time that processes are stalled waiting for CPU, memory, or I/O resources, distinguishing between minor contention and debilitating starvation.
- **Deep Dive:** Metric types: `some` (at least one thread is stalled) and `full` (all threads are stalled, meaning productive work has stopped completely). In Kubernetes, high memory `full` PSI warns of imminent OOM kills before they occur.
- **Trade-offs & Realities:** PSI requires cgroups v2 and Linux kernel 4.20+.
- **Follow-up Probes:** "Where is PSI data located?" (`/proc/pressure/cpu`, `/proc/pressure/memory`, `/proc/pressure/io`).

### Q19: Explain the difference between SYN cookies and the standard SYN backlog queue.
- **Short Answer:** Standard connections allocate a `request_sock` struct in the SYN backlog queue upon receiving a SYN. Under a SYN flood attack, this queue exhausts RAM. SYN cookies disable the queue, encoding state into the SYN-ACK Initial Sequence Number (ISN) and allocating memory only when a valid final ACK is returned.
- **Deep Dive:** When SYN cookies are active, advanced TCP options (like large window scales and selective ACKs) may be restricted unless TCP timestamps are enabled.
- **Trade-offs & Realities:** SYN cookies add minor CPU hashing overhead but provide robust DDoS resilience.
- **Follow-up Probes:** "What sysctl controls SYN cookies?" (`net.ipv4.tcp_syncookies = 1`).

### Q20: What is eBPF, and how is it transforming platform engineering?
- **Short Answer:** Extended Berkeley Packet Filter (eBPF) allows engineers to run sandboxed, verified bytecode programs inside the Linux kernel at runtime without modifying kernel source code or loading unstable kernel modules.
- **Deep Dive:** eBPF programs attach to kprobes, tracepoints, socket buffers, and XDP hooks. They enable $O(1)$ Kubernetes networking (Cilium), deep kernel observability without sampling (Pixie, Parca), and real-time runtime security enforcement (Tetragon, Falco).
- **Trade-offs & Realities:** Writing eBPF code in C requires understanding kernel data structures. The in-kernel verifier strictly rejects programs with unverified memory safety or infinite loops.
- **Follow-up Probes:** "What does the eBPF verifier guarantee?" (Termination, no infinite loops, no out-of-bounds memory access, no uninitialized variable reads).

---

## Tier 3: Staff & Principal Architect Scenarios (Questions 21–25)

### Q21: Architect a high-throughput, low-latency Kubernetes networking layer for 2,000 nodes serving 500,000 RPS.
- **Candidate Framework:**
  - *CNI Architecture:* Replace kube-proxy and standard overlay CNI with Cilium running in native routing mode (eBPF host routing) without VXLAN encapsulation.
  - *Hardware & MTU:* Enable Jumbo Frames (MTU 9000) across AWS/bare-metal network fabrics to maximize throughput and minimize packet processing overhead.
  - *DNS Scaling:* Deploy NodeLocal DNSCache on every node listening on a link-local IP, forcing TCP to upstream CoreDNS clusters sized with auto-scaling based on core count.
  - *Kernel Sysctls:* Tune `net.core.somaxconn = 32768`, `net.ipv4.tcp_max_syn_backlog = 16384`, `net.ipv4.tcp_tw_reuse = 1`, and expand ephemeral port range (`1024-65535`).
  - *Hardware Offload:* Leverage SR-IOV or AWS ENA Express for high-bandwidth workloads.
- **Red Flags:** Recommending default Flannel/iptables kube-proxy or neglecting DNS caching.

### Q22: You observe widespread, intermittent 502 Bad Gateway errors across an entire EKS cluster during peak traffic. How do you isolate the root cause?
- **Candidate Framework:**
  - *Isolate the Boundary:* Determine if 502s originate from the external ALB, the ingress controller (Envoy/NGINX), or the application pods.
  - *Check Connection Pooling & Keepalives:* A classic cause of 502s is a keepalive timeout mismatch where the backend pod closes an idle TCP connection (e.g., 60s timeout) right as the ingress proxy sends a request (ALB idle timeout 65s). Solution: Ensure backend keepalive timeout is strictly higher than upstream proxy timeout.
  - *Check Node Conntrack Exhaustion:* Inspect `dmesg` for `nf_conntrack: table full, dropping packet`.
  - *Check CFS Throttling:* Verify if ingress controller or application pods are hitting CPU limits.
- **Red Flags:** Guessing randomly without checking upstream vs downstream HTTP response headers and logs.

### Q23: How would you design a multi-tenant container runtime security architecture capable of preventing container breakouts?
- **Candidate Framework:**
  - *Kernel Hardening:* Enforce seccomp profiles (blocking dangerous syscalls like `unshare`, `ptrace`, `bpf`), apparmor/SELinux, and drop all default Linux capabilities except `NET_BIND_SERVICE`.
  - *Rootless & User Namespaces:* Run all containers as non-root UIDs, enabling `UserNamespaces` to map container UID 0 to an unprivileged host UID.
  - *Sandboxed Runtimes:* For untrusted or multi-tenant code execution, employ hardware-isolated runtimes like Kata Containers (lightweight microVMs via QEMU/Firecracker) or gVisor (user-space kernel interception).
  - *Runtime Behavioral Tracing:* Deploy Cilium Tetragon or Falco to monitor eBPF events for namespace modifications, binary execution in `/tmp`, or unexpected outbound socket connections.
- **Red Flags:** Assuming Docker namespaces provide foolproof multi-tenant isolation against zero-day kernel exploits.

### Q24: A financial API experiences sporadic 1-second latency spikes every 15 minutes. How do you isolate whether it's GC pauses, kernel memory reclamation, or network drops?
- **Candidate Framework:**
  - *Correlate Telemetry:* Compare application p99 latency timestamps against host-level metrics.
  - *Inspect Garbage Collection:* Check JVM/Go GC logs for Stop-the-World pauses.
  - *Inspect Kernel Memory Stall (PSI):* Check `/proc/pressure/memory` for `full` stall spikes. If background memory compaction (`kswapd`) is scanning for free pages, direct reclamation freezes user threads.
  - *Inspect Disk I/O Writeback:* If a cron job executes `sync` or generates dirty page writebacks every 15 minutes, inspect `vm.dirty_ratio` and iowait.
  - *eBPF Tracing:* Deploy `runqlat` (Brendan Gregg's BCC tool) to measure CPU scheduler run queue latency to prove whether threads were waiting for CPU allocation.
- **Red Flags:** Blaming the network without capturing TCP metrics or packet traces.

### Q25: What are the trade-offs between implementing Service Mesh mTLS at L7 (Envoy sidecars) vs L4 (Ambient / eBPF)?
- **Candidate Framework:**
  - *Sidecar Model (Istio / Linkerd classic):*
    - *Pros:* Full L7 inspection, rich per-pod authorization policies, distributed tracing injection, local circuit breaking.
    - *Cons:* Huge resource footprint (1 Envoy per Pod = gigabytes of wasted RAM), extra network hops (App -> Sidecar -> Host -> Network), slow container startup.
  - *Ambient / Node-Level Proxy with eBPF (Istio Ambient, Cilium Service Mesh):*
    - *Pros:* Zero sidecar memory overhead, transparent mTLS at L4 (via ztunnel or eBPF IPsec/WireGuard), fast pod startup, simplified application lifecycle.
    - *Cons:* Shared node proxy handles traffic for multiple tenants (blast radius concerns), complex L7 authorization delegation.
- **Red Flags:** Assuming sidecars are the only way to implement mTLS in modern Kubernetes.
