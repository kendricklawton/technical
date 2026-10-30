# Domain 3: Services & Networking — Essential Documentation & Technical Reference Guide

> **Blueprint Weight:** 20% of Certified Kubernetes Administrator (CKA)  
> **Core Objective:** Understand Kubernetes networking models, configure CoreDNS, manage Services (ClusterIP, NodePort, LoadBalancer, Headless), implement ingress routing, deploy Gateway API resources, and enforce granular network policies.

---

## 1. Official Documentation & Authoritative References

### 1.1 [Cluster Networking Specification](https://kubernetes.io/docs/concepts/cluster-administration/networking/)
* **Canonical Scope:** The fundamental rules governing Kubernetes communication.
* **The Four Fundamental Network Invariants:**
  1. All Pods can communicate with all other Pods on all nodes without NAT.
  2. All agents on a node (e.g. kubelet) can communicate with all Pods on that node.
  3. Pods on the host network can communicate with all Pods on all nodes without NAT.
  4. The IP that a Pod sees itself as is the same IP that other Pods see it as.

### 1.2 [Services & EndpointSlices](https://kubernetes.io/docs/concepts/services-networking/service/)
* **Canonical Scope:** Service abstraction, proxying, and service discovery.
* **Service Types:**
  * `ClusterIP`: Internal virtual IP (default); load-balanced across endpoints via iptables or IPVS / eBPF (kube-proxy).
  * `NodePort`: Exposes service on every node's IP at a static port (default range: 30000–32767).
  * `LoadBalancer`: Provisions cloud load balancer pointing to NodePorts.
  * `Headless (None)`: Does not allocate a ClusterIP; DNS returns direct Pod A/AAAA records.

### 1.3 [Gateway API Specification](https://gateway-api.sigs.k8s.io/)
* **Canonical Scope:** The next-generation evolution of Ingress for service mesh and edge routing.
* **Core Resource Hierarchy:**
  * `GatewayClass`: Infrastructure provider controller definition (e.g., Envoy, Cilium, Istio).
  * `Gateway`: Declares listening ports, protocols, and TLS certificates at the edge.
  * `HTTPRoute` / `GRPCRoute`: Defines routing rules, path matches, header rewrites, and canary traffic splits pointing to Services.

### 1.4 [Network Policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/)
* **Canonical Scope:** Granular L3/L4 firewall rules controlling pod ingress and egress traffic.
* **Critical Rule:**
  * By default, pods are **non-isolated** (all traffic accepted).
  * Once a `NetworkPolicy` selects a pod via `podSelector`, the pod becomes **isolated**, and all traffic not explicitly allowed is **dropped**.

---

## 2. High-Frequency Exam Traps & Practical Reflexes

1. **NetworkPolicy Default Deny All:**
   * Isolate all pods in a namespace completely:
     ```yaml
     apiVersion: networking.k8s.io/v1
     kind: NetworkPolicy
     metadata:
       name: default-deny-all
       namespace: production
     spec:
       podSelector: {}
       policyTypes:
       - Ingress
       - Egress
     ```
2. **Missing CNI Plugin:**
   * Newly joined nodes remain in `NotReady` with `NetworkPluginNotReady: cni plugin not initialized`.
   * *Resolution:* Install Calico or Cilium manifest, or verify CNI bridge configuration in `/etc/cni/net.d/`.
3. **CoreDNS Service Resolution Traps:**
   * Pod cannot resolve external domains or cluster services.
   * *Resolution:* Inspect `/etc/resolv.conf` in pod, test with `nslookup <service>.<namespace>.svc.cluster.local`, and check CoreDNS deployment logs in `kube-system`.

---

## 3. Reading & Study Checklist

- [ ] Read *Cluster Networking Specification*
- [ ] Read *Services & EndpointSlices*
- [ ] Read *Gateway API Specification*
- [ ] Master NetworkPolicy syntax (podSelector, namespaceSelector, ipBlock, ports)
- [ ] Practice CoreDNS Corefile configuration troubleshooting
