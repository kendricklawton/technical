# PE Domain 2: Linux & Networking — Scenario Quiz 3

Test your understanding of DNS resolution in cloud-native systems, iptables/eBPF routing, and TLS.

---

### Question 1
A microservice running inside a Kubernetes cluster makes 1,000 HTTP requests per second to `https://api.stripe.com`. The application experiences high outbound latency. A packet capture reveals that for every single outbound request, 4 failed DNS queries are sent before the successful query resolves. What is causing this behavior?

A) The Kubernetes `/etc/resolv.conf` defaults to `ndots:5`; because `api.stripe.com` contains 2 dots (<5), the resolver queries the cluster search domains (`.namespace.svc.cluster.local`, `.svc.cluster.local`, etc.) before querying the public name.  
B) Stripe's DNS servers are malfunctioning.  
C) CoreDNS is rejecting all external domains.  
D) The pod has run out of memory.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** A  
**Explanation:** The glibc resolver tests search domains whenever the queried domain contains fewer dots than the `ndots` threshold (defaulting to 5 in Kubernetes). Appending a trailing dot (`api.stripe.com.`) or lowering `ndots` to 2 in the pod's `dnsConfig` eliminates these 4 wasted lookups.
</details>

---

### Question 2
What architectural advantage does deploying **NodeLocal DNSCache** provide in large Kubernetes clusters?

A) It automatically encrypts all HTTP traffic with mTLS.  
B) It runs a lightweight DNS caching agent on every node listening on a local loopback IP (`169.254.20.10`), eliminating cross-node network hops for DNS queries, caching responses locally, and bypassing Netfilter conntrack race conditions by using TCP to upstream CoreDNS.  
C) It stores DNS records in Git.  
D) It replaces external DNS registrars.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** B  
**Explanation:** NodeLocal DNSCache runs as a DaemonSet. Pods query the local node's cache via a link-local address, resolving most requests locally with sub-millisecond latency while preventing conntrack race conditions that historically plagued UDP-based DNS lookups in high-throughput clusters.
</details>

---

### Question 3
Why does **Cilium (eBPF)** scale significantly better than **kube-proxy (iptables)** in Kubernetes clusters with thousands of Services and Endpoints?

A) iptables evaluates rules sequentially ($O(N)$ complexity) and requires full table locking on every change, whereas Cilium uses eBPF hash tables ($O(1)$ complexity) attached directly to socket/tc hooks in the Linux kernel without table locks.  
B) iptables does not support TCP traffic.  
C) Cilium disables all firewall security.  
D) iptables was written in Python; Cilium is written in Rust.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** A  
**Explanation:** In large clusters (thousands of services), iptables rules grow to tens of thousands of lines. Kube-proxy must flush and reload the entire chain on changes, causing high kernel CPU consumption. Cilium leverages eBPF BPF maps, allowing $O(1)$ constant-time lookups and lockless incremental updates.
</details>

---

### Question 4
In TLS 1.3, what significant architectural change was introduced compared to TLS 1.2 to reduce network latency?

A) TLS 1.3 only operates over UDP.  
B) TLS 1.3 dropped encryption entirely.  
C) TLS 1.3 streamlined the handshake from 2 round-trips (2-RTT) down to 1 round-trip (1-RTT) by combining cryptographic parameter negotiation and key exchange into the initial `ClientHello` and `ServerHello` messages.  
D) TLS 1.3 requires dedicated hardware cryptographic accelerators.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** C  
**Explanation:** TLS 1.3 deprecated obsolete cipher suites and mandates Diffie-Hellman key exchange. The client speculatively sends its public key share in the `ClientHello`, allowing the server to compute the shared secret and respond with encrypted parameters in the `ServerHello`, cutting connection establishment latency in half (1-RTT).
</details>

---

### Question 5
What is the function of the **Server Name Indication (SNI)** extension during a TLS handshake?

A) It allows a server to report its internal CPU utilization to the client.  
B) It prevents DNS cache poisoning.  
C) It authenticates the client's credit card credentials.  
D) It enables the client to specify the target hostname (e.g., `api.example.com`) in plaintext within the initial `ClientHello` message, allowing a reverse proxy hosting multiple certificates on a single IP address to present the correct TLS certificate.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** D  
**Explanation:** Because TLS negotiation happens before HTTP headers are sent, a server hosting multiple virtual domains on a single IP address wouldn't know which certificate to present. SNI passes the desired hostname in the `ClientHello`, enabling virtual hosting over TLS.
</details>

---

### Question 6
What is the difference between an **L4 (Transport)** load balancer and an **L7 (Application)** load balancer?

A) L4 is slower than L7.  
B) L4 is software; L7 is always physical hardware.  
C) L4 routes packets based on IP address and TCP/UDP port without decrypting payload; L7 terminates TLS, inspects HTTP headers, URLs, and cookies, and can perform path-based routing and intelligent retries.  
D) L7 only works with email protocols.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** C  
**Explanation:** L4 operates at the transport layer (TCP/UDP), forwarding packets with maximum throughput and minimal processing. L7 terminates connections, parses application payloads (HTTP/2, gRPC), and enables complex capabilities like canary routing, header manipulation, and circuit breaking.
</details>

---

### Question 7
In Kubernetes pod networking, what is the role of a **veth (Virtual Ethernet) pair**?

A) It connects two physical servers via an underwater cable.  
B) It synchronizes clock times between pods.  
C) It encrypts pod memory storage.  
D) It acts as a bidirectional virtual wire connecting the isolated Network Namespace of a Pod (`eth0`) to the root network namespace of the host node.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** D  
**Explanation:** A veth pair is a kernel networking primitive. Whatever packet enters one end emerges immediately from the peer end. The CNI places one end inside the pod namespace as `eth0` and keeps the other in the host root namespace to interface with bridges, routing tables, or eBPF hooks.
</details>

---

### Question 8
What happens if the MTU (Maximum Transmission Unit) configured on an overlay network (such as VXLAN or Geneve) is identical to the physical network interface MTU (e.g., 1500 bytes)?

A) All IP addresses are automatically reassigned.  
B) Packets exceeding 1500 bytes after encapsulation headers (50-byte VXLAN overhead) are either fragmented or dropped by routers with the Don't Fragment (DF) bit set, causing silent connection hangs during large data transfers.  
C) Network throughput increases by 500%.  
D) The kernel switches to token ring networking.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** B  
**Explanation:** Overlay networks encapsulate inner IP packets inside outer UDP packets. This encapsulation adds 50 bytes of overhead. If the overlay MTU is not set smaller than the underlying physical network MTU (e.g., 1450 vs 1500), packets will exceed the physical MTU, causing severe fragmentation or silent black-hole packet drops.
</details>

---

### Question 9
What does the `ALPN (Application-Layer Protocol Negotiation)` extension accomplish during a TLS handshake?

A) It allows the client and server to negotiate whether to speak HTTP/1.1 or HTTP/2 inside the encrypted session without incurring an extra network round-trip.  
B) It updates the Linux kernel.  
C) It calculates the geographic distance between data centers.  
D) It reloads NGINX configuration files.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** A  
**Explanation:** ALPN enables the client to list supported application protocols (e.g., `h2`, `http/1.1`) inside the `ClientHello`. The server selects the protocol in the `ServerHello`, ensuring that encrypted data transfer begins immediately using the optimal protocol without additional round-trips.
</details>

---

### Question 10
When an application container built with **Alpine Linux (musl libc)** executes DNS lookups, why does it occasionally behave differently than a container built with **Debian (glibc)**?

A) glibc only supports IPv6.  
B) Alpine does not support networking.  
C) Musl libc automatically blocks all outbound internet access.  
D) Musl libc historical versions do not support TCP fallback for DNS queries exceeding 512 bytes, send A and AAAA queries concurrently over the same socket (triggering conntrack drops in early kernels), and handle search domains differently than glibc.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** D  
**Explanation:** Musl libc is a lightweight C standard library. Historically, musl's resolver sent parallel A and AAAA lookups on the same socket port, triggering netfilter conntrack race conditions in Linux, and lacked standard glibc features like `single-request-reopen` and TCP fallback.
</details>
