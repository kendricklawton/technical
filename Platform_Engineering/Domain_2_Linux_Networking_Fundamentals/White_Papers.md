# Domain 2: Linux & Networking Fundamentals — Essential Technical Documentation & RFC Guide

> **Core Focus:** Linux Kernel Internals, Namespaces, Cgroups v2, Memory Management & OOM Killer, CPU CFS Throttling, Socket Buffers, TCP State Machine, TLS 1.3 Handshake, DNS Architecture, and eBPF.

---

## 1. Authoritative RFCs & Linux Kernel Documentation

### 1.1 [Control Groups v2 (cgroups-v2) Kernel Documentation](https://www.kernel.org/doc/Documentation/cgroup-v2.txt)
* **Canonical Scope:** Single unified hierarchy for system resource isolation.
* **Core Controllers:**
  * `cpu.max`: Enforces CFS bandwidth quota and period (e.g. `100000 100000` = 1 core). Exceeding quota triggers throttling without killing processes.
  * `memory.max`: Hard ceiling; exceeding triggers the kernel Out-Of-Memory (OOM) killer.
  * `memory.high`: Throttle threshold; processes slow down and aggressively reclaim page cache before hitting `memory.max`.
  * `memory.oom.group`: Kills the entire cgroup container if any single process inside causes an OOM event.

### 1.2 [Linux Namespaces Architecture](https://man7.org/linux/man-pages/man7/namespaces.7.html)
* **Canonical Scope:** Seven kernel primitives creating the container isolation illusion:
  * `pid` (Process IDs), `net` (Network devices, routing tables, iptables), `mnt` (Mount points), `ipc` (Inter-process communication), `uts` (Hostname), `user` (UID/GID mapping), `cgroup` (Virtual cgroup root).

### 1.3 Core Networking RFCs:
* **[RFC 793 (TCP Specification)](https://www.rfc-editor.org/rfc/rfc793):** 3-way handshake (`SYN` -> `SYN-ACK` -> `ACK`), sliding window flow control, Congestion Control (Tahoe, Reno, BBR), and connection teardown (`FIN-WAIT-1`, `FIN-WAIT-2`, `TIME-WAIT`).
* **[RFC 8446 (TLS 1.3 Specification)](https://www.rfc-editor.org/rfc/rfc8446):** 1-RTT handshake (0-RTT with PSK resumption), Diffie-Hellman ephemeral key exchange (ECDHE), mandatory forward secrecy, and removal of insecure ciphers.
* **[RFC 1035 (Domain Names Implementation)](https://www.rfc-editor.org/rfc/rfc1035):** UDP 53 query protocol, A/AAAA records, CNAME resolution chains, TTL expiration, and recursive vs iterative lookup semantics.

---

## 2. Production Performance & Diagnostic Primitives

### 2.1 [Brendan Gregg: Systems Performance & The USE Method](https://www.brendangregg.com/usemethod.html)
* **Utilization, Saturation, and Errors (USE):**
  * *CPU:* Check utilization (`mpstat`), saturation (`vmstat` run queue, load averages), errors (`dmesg`).
  * *Memory:* Check page faults, swap activity, and `/proc/vmstat`.
  * *Network:* Check dropped packets, ring buffer overflows (`ethtool -S`), and TCP retransmits (`ss --tcp -i`).

### 2.2 Linux Conntrack & iptables / nftables:
* Netfilter connection tracking (`nf_conntrack`).
* When `nf_conntrack_max` table is saturated, kernel drops new incoming SYN packets silently with `nf_conntrack: table full, dropping packet`.

---

## 3. Staff-Level Interview Angle & Architectural Reflexes

1. **"The curl https://example.com Walkthrough":**
   * Walk down every layer: libc `getaddrinfo` -> `/etc/nsswitch.conf` -> `/etc/resolv.conf` (ndots:5 traversal) -> local CoreDNS -> recursive resolver -> socket creation (`socket(AF_INET, SOCK_STREAM)`) -> ARP cache lookup -> TCP SYN packet -> iptables/eBPF DNAT -> TLS 1.3 ServerHello -> HTTP GET -> HTTP 200 response -> TCP FIN.
2. **CPU Throttling in Kubernetes:**
   * Symptoms: P99 latency spikes despite low average CPU usage.
   * *Mechanism:* Completely Fair Scheduler (CFS) quota period (100ms). If a thread bursts and uses all 100ms of CPU in the first 20ms, it is throttled for the remaining 80ms!
   * *Staff Solution:* Remove CPU limits on latency-sensitive workloads while keeping CPU requests accurate, or tune CFS quota period.

---

## 4. Reading & Study Checklist

- [ ] Read *Control Groups v2 Kernel Documentation*
- [ ] Read Brendan Gregg's *The USE Method*
- [ ] Study TCP state transitions, specifically `TIME-WAIT` and socket reuse
- [ ] Review TLS 1.3 vs TLS 1.2 handshake message exchanges
- [ ] Trace the complete lifecycle of a container network packet through veth pairs
