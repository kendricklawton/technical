# PE Domain 2: Linux & Networking — Scenario Quiz 2

Test your knowledge on CPU CFS throttling, TCP state machines, and socket management.

---

### Question 1
A multi-threaded Java application running in Kubernetes with `resources.limits.cpu: "2"` has an average CPU usage of only 35%. However, developers report severe p99 response latency spikes exceeding 100ms. Examining `/sys/fs/cgroup/cpu/cpu.stat` reveals:
```text
nr_periods 1000
nr_throttled 650
throttled_time 48000000000
```
What is causing the application's latency spikes?

A) The JVM threads burst concurrently across multiple host CPU cores, exhausting the cgroup's 200ms quota early in the 100ms CFS period, causing the Linux kernel to throttle (freeze) execution for the remainder of the period.  
B) The node is experiencing network interface packet drops.  
C) The Java heap size is too small, causing frequent GC pauses.  
D) The kernel routing table has a circular reference.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** A  
**Explanation:** In the Linux Completely Fair Scheduler (CFS), a CPU limit of 2 cores grants 200ms of CPU time per 100ms period. When a multi-threaded application spawns multiple threads that run concurrently on physical cores, it consumes the quota rapidly within the first 10–20ms. The kernel then throttles the container for the remaining 80ms of the period, creating severe latency spikes. Solutions include increasing CPU limits, removing CPU limits in favor of requests, or sizing thread pools to match limits.
</details>

---

### Question 2
When inspecting network connections on an API gateway using `netstat` or `ss`, you observe thousands of sockets in the `TIME_WAIT` state. What is the fundamental purpose of the `TIME_WAIT` state in the TCP protocol?

A) It keeps the TLS certificate cached in memory.  
B) It indicates that the remote client crashed without sending a FIN.  
C) It allows the server to buffer unread application data.  
D) It ensures that delayed or duplicated packets from the terminated connection dissipate in the network and cannot be mistakenly delivered to a new connection reusing the same 4-tuple.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** D  
**Explanation:** `TIME_WAIT` lasts for $2 \times \text{MSL}$ (typically 60 seconds). It guarantees that late-arriving segments from the closed connection do not collide with new connections using the same local and remote IP/port pair, and ensures the remote peer received the final ACK.
</details>

---

### Question 3
An engineer notices 5,000 sockets on a production backend server stuck in the `CLOSE_WAIT` state. Which of the following is the true root cause of a `CLOSE_WAIT` buildup?

A) The Linux kernel TCP buffer is full.  
B) The network switch has dropped packets.  
C) The remote client has closed its side of the connection (sent FIN), but the local application code has failed to call `close()` on the socket descriptor due to a bug or thread deadlock.  
D) DNS resolution has failed.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** C  
**Explanation:** `CLOSE_WAIT` is a passive state. It means the OS kernel received a FIN from the remote peer and acknowledged it, but the local application process has not invoked `close()` on the socket file descriptor. A buildup of `CLOSE_WAIT` is **always an application-level leak**.
</details>

---

### Question 4
What is the function of the Linux kernel setting `net.ipv4.tcp_tw_reuse = 1`?

A) It forces all connections to use UDP instead of TCP.  
B) It increases the maximum MTU from 1500 to 9000 bytes.  
C) It completely disables all TCP security checks.  
D) It allows the kernel to safely reuse outgoing sockets in the `TIME_WAIT` state for new outgoing connections when the timestamp indicates the new connection's sequence numbers are strictly newer.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** D  
**Explanation:** `tcp_tw_reuse` allows client systems (like reverse proxies making high-volume outbound connections) to reuse sockets in `TIME_WAIT` if the TCP timestamps extension is enabled and sequence numbers are safe, avoiding ephemeral port exhaustion.
</details>

---

### Question 5
When a client sends a TCP SYN packet to a server whose application listening queue is completely full (exceeding `somaxconn`), and `tcp_abort_on_overflow` is set to `0` (default), what does the Linux kernel do with the incoming packet?

A) It crashes the operating system.  
B) It silently drops the packet, relying on the client's exponential backoff and TCP retransmission timer to retry in hopes the queue clears.  
C) It buffers the packet indefinitely on disk.  
D) It sends a TCP RST packet to terminate the client immediately.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** B  
**Explanation:** When the listen queue is full, Linux by default silently drops incoming SYN/ACK packets. This allows the client to retry (via TCP retransmission). Dropping the packet is preferable to sending an immediate RST because transient spikes might clear before the client's retransmission arrives.
</details>

---

### Question 6
What is the role of **SYN Cookies** (`net.ipv4.tcp_syncookies = 1`) in defending against SYN flood DoS attacks?

A) It forces the client to solve a Proof-of-Work puzzle before establishing a connection.  
B) When the SYN backlog queue overflows, the server encodes the connection state cryptographically into the SYN-ACK's Initial Sequence Number (ISN) instead of allocating kernel memory, verifying the state only when the client returns a valid ACK.  
C) It blocks all incoming traffic from unknown IP addresses.  
D) It installs an HTTP cookie in the client's browser.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** B  
**Explanation:** In a SYN flood, an attacker sends thousands of SYNs with spoofed IPs, exhausting server memory. SYN Cookies bypass queue storage: the server computes a cryptographic hash (using secret key, timestamp, IPs, and ports) and embeds it in the sequence number. Memory is only allocated when the client sends the final ACK containing the valid sequence number.
</details>

---

### Question 7
How does the Linux kernel Completely Fair Scheduler enforce CPU bandwidth control?

A) By dropping 50% of incoming network packets.  
B) By limiting the physical clock frequency of the CPU core (thermal throttling).  
C) By tracking cgroup runtime using `cpu.cfs_quota_us` and `cpu.cfs_period_us`, descheduling and putting all processes in the cgroup to sleep when their allocated quota within the current period is exhausted.  
D) By killing the lowest priority thread.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** C  
**Explanation:** CFS bandwidth control tracks cumulative CPU runtime consumed across all threads in a cgroup. Once usage hits `cfs_quota_us` within `cfs_period_us`, the kernel suspends all tasks in that cgroup until the timer expires and a new period begins.
</details>

---

### Question 8
What is the primary difference between TCP and UDP regarding connection orientation?

A) TCP is connection-oriented, providing reliable, ordered byte streams via sequence numbers, ACKs, and window flow control; UDP is connectionless, sending individual datagrams with zero delivery or ordering guarantees.  
B) UDP cannot be routed across the internet.  
C) UDP is encrypted; TCP is always plaintext.  
D) TCP only works on fiber-optic cables.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** A  
**Explanation:** TCP establishes an explicit 3-way handshake state, tracks sequence numbers, retransmits lost packets, and manages flow/congestion control. UDP is a lightweight, stateless wrapper providing port multiplexing and checksums without delivery guarantees.
</details>

---

### Question 9
What does a TCP **Window Update** segment communicate?

A) The amount of receive buffer space (in bytes) the sending endpoint currently has available for incoming data, enabling flow control to prevent buffer overflows.  
B) A request to change the IP address.  
C) That the operating system needs to restart for updates.  
D) A change in the physical display resolution of the server.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** A  
**Explanation:** Flow control in TCP uses sliding windows. A receiver advertises its available receive buffer size (`win=...`). A zero-window probe indicates the receiver is temporarily full and the sender must pause transmission.
</details>

---

### Question 10
What tool should an engineer run to inspect Linux socket statistics (states, timer details, and memory buffers) faster and more efficiently than `netstat`?

A) `ping`  
B) `ss` (Socket Statistics)  
C) `traceroute`  
D) `htop`  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** B  
**Explanation:** `ss` directly queries kernel Netlink socket diag interfaces, making it orders of magnitude faster and less resource-intensive than legacy `netstat` (which parsed thousands of lines in `/proc/net/tcp`).
</details>
