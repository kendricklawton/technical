# Domain 3: Services & Networking — Quiz 1 (Core Networking, CNI & Services)

**Exam:** Certified Kubernetes Administrator (CKA)  
**Format:** 10 Performance Scenario & Multiple Choice Questions with Detailed Explanations

---

### Question 1: Default Pod-to-Pod Traffic Behavior
A new namespace `marketing` has 5 pods running. No NetworkPolicy exists in the namespace. Can pods from another namespace `sales` communicate with pods in `marketing`?
- A. Only if a Service of type NodePort is created.
- B. Only if an Ingress resource is defined.
- C. Yes, by default all pods in a Kubernetes cluster can communicate with all other pods across all namespaces without NAT.
- D. No, namespaces isolate traffic by default in Kubernetes.

---

### Question 2: Service `ClusterIP` vs `NodePort` Default Range
You expose a Deployment using `kubectl expose deployment api --type=NodePort --port=80 --target-port=8080` without specifying a `nodePort` number. From what default port range does `kube-apiserver` allocate the NodePort?
- A. `30000-32767`
- B. `8000-9000`
- C. `1024-32767`
- D. `49152-65535`

---

### Question 3: Headless Service IP Allocation & DNS Records
When creating a Headless Service with `clusterIP: None` that selects 3 ready StatefulSet pods (`db-0`, `db-1`, `db-2`), what does `kubectl get svc` report for the ClusterIP column, and what does CoreDNS return for an `A` record lookup on the Service name?
- A. The node IP address; CoreDNS returns a single CNAME.
- B. `None`; CoreDNS returns the individual Pod IP addresses of all Ready pods matching the selector, plus individual `<pod-name>.<svc-name>.<ns>.svc.cluster.local` A records when pods set `hostname`/`subdomain` (or belong to a StatefulSet).
- C. `Pending`.
- D. A random IP from `--service-cluster-ip-range`.

---

### Question 4: Preserving Client Source IP (`externalTrafficPolicy: Local`)
An external load balancer forwards client traffic to a Kubernetes `NodePort` / `LoadBalancer` Service. By default (`externalTrafficPolicy: Cluster`), `kube-proxy` performs SNAT when forwarding traffic to a Pod on another node, replacing the client's original IP with the node's IP. Which Service setting preserves the client's original source IP and eliminates extra cross-node hops?
- A. `spec.publishNotReadyAddresses: true`
- B. `spec.sessionAffinity: ClientIP`
- C. `spec.internalTrafficPolicy: Local`
- D. `spec.externalTrafficPolicy: Local`

---

### Question 5: Trade-Off of `externalTrafficPolicy: Local`
When `externalTrafficPolicy: Local` is configured on a `NodePort` Service, what happens if an external packet arrives at `worker-3:30080`, which currently runs **zero** pods for that Service?
- A. `worker-3` forwards the packet to `worker-1` without SNAT.
- B. The packet is dropped at `worker-3` (and `kube-proxy`'s `healthCheckNodePort` returns HTTP `503` so external load balancers stop sending traffic to `worker-3`).
- C. `kube-scheduler` immediately starts a Pod on `worker-3`.
- D. The Service falls back to `Cluster` mode automatically.

---

### Question 6: `kube-proxy` Modes (`iptables` vs `ipvs` vs `nftables`)
In a 2,000-node cluster with 15,000 Services, `kube-proxy` in `iptables` mode experiences high CPU usage and slow rule synchronization because `iptables` evaluates chains sequentially ($O(N)$). Which `kube-proxy` mode uses Linux kernel netfilter hash tables for $O(1)$ lookup latency and supports load-balancing algorithms like Least Connections (`lc`) and Round Robin (`rr`)?
- A. `bridge`
- B. `userspace`
- C. `ebpf-host`
- D. `ipvs`

---

### Question 7: CNI Plugin Configuration & Binary Paths
During CKA troubleshooting, a newly provisioned cluster has nodes in `NotReady` with `NetworkPluginNotReady: cni plugin not initialized`. What are the two standard filesystem paths on Linux nodes where the kubelet/container runtime looks for CNI configuration files and CNI executable binaries?
- A. `/var/lib/cni/conf` and `/var/lib/kubelet/cni`
- B. `/etc/kubernetes/cni` and `/usr/local/bin/cni`
- C. `/etc/cni/net.d` (configurations) and `/opt/cni/bin` (binaries)
- D. `/run/cni` and `/sbin/cni`

---

### Question 8: `EndpointSlices` vs Legacy `Endpoints` Scalability
Why did Kubernetes introduce `EndpointSlice` (`discovery.k8s.io/v1`) to replace the monolithic `Endpoints` resource for Services with thousands of backend Pods?
- A. `Endpoints` did not support TCP ports.
- B. `EndpointSlices` encrypt Pod traffic with mTLS.
- C. `EndpointSlices` replace `CoreDNS`.
- D. Legacy `Endpoints` stored all Pod IPs in a single object (which hit etcd's 1.5MB object size limit and caused massive $O(N^2)$ network watch payloads on every single pod readiness change), whereas `EndpointSlices` shard endpoints into chunks of up to 100 endpoints per slice by default.

---

### Question 9: `Service` `sessionAffinity: ClientIP`
A stateful legacy web application stores user sessions in local container memory instead of Redis. How can you configure a Kubernetes `Service` so repeated requests from the same client IP are routed to the same backend Pod for 1 hour (3600 seconds)?
- A. Set `spec.sessionAffinity: ClientIP` and `spec.sessionAffinityConfig.clientIP.timeoutSeconds: 3600`.
- B. Set `spec.clusterIP: None`.
- C. Set `spec.type: ExternalName`.
- D. Add annotation `service.kubernetes.io/sticky: "true"`.

---

### Question 10: Dual-Stack Networking (`ipFamilyPolicy`)
You want a Service to allocate both an IPv4 `ClusterIP` and an IPv6 `ClusterIP` in a dual-stack enabled cluster. Which fields in the Service spec configure this?
- A. `spec.ipFamilyPolicy: RequireDualStack` (or `PreferDualStack`) and `spec.ipFamilies: [IPv4, IPv6]`
- B. `spec.clusterIPs: [auto, auto]`
- C. `spec.enableIPv6: true`
- D. `spec.dualStack: true`

---

## Answer Key & Detailed Explanations

1. **C** — By default, Kubernetes pods are non-isolated and can communicate across all namespaces without NAT.
2. **A** — The default NodePort range in Kubernetes is `30000-32767` (configurable on `kube-apiserver` via `--service-node-port-range`).
3. **B** — Headless services (`clusterIP: None`) do not allocate a virtual IP; CoreDNS returns individual Pod A/AAAA records directly.
4. **D** — `externalTrafficPolicy: Local` forces `kube-proxy` to route traffic only to node-local endpoints, eliminating cross-node SNAT and preserving the original client IP.
5. **B** — With `externalTrafficPolicy: Local`, nodes without local pods drop direct NodePort traffic and report unhealthy on their `healthCheckNodePort`.
6. **D** — `ipvs` mode uses kernel hash tables with $O(1)$ complexity regardless of Service count.
7. **C** — `/etc/cni/net.d` holds the lexicographically sorted CNI JSON/conflist files, and `/opt/cni/bin` holds the executable CNI plugins (`bridge`, `host-local`, `loopback`, `calico`, `cilium-cni`).
8. **D** — By default, each `EndpointSlice` holds a maximum of 100 endpoints, dramatically reducing API server and `kube-proxy` watch overhead.
9. **A** — `sessionAffinity: ClientIP` configures client-IP stickiness with a configurable `timeoutSeconds` (default `10800` seconds = 3 hours).
10. **A** — `ipFamilyPolicy` (`SingleStack`, `PreferDualStack`, `RequireDualStack`) and `ipFamilies` (`[IPv4, IPv6]` or `[IPv6, IPv4]`) control dual-stack Service IP allocation.
