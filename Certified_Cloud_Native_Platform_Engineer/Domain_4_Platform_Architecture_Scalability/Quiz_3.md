# Domain 4: Platform Architecture & Scalability — Quiz 3

**Exam:** Certified Cloud Native Platform Engineer (CNPE)  
**Format:** 10 Hands-On Scenario & Performance Questions with Detailed Rationales

---

### Question 1: Gateway API Header-Based Canary Routing
You need to route beta testers whose HTTP request includes the header `X-Beta-Tester: true` to the `payment-canary` service, while all other traffic goes to `payment-stable`. Which Gateway API resource and match type configures this behavior?
- A. `NetworkPolicy` with `egress.ports`
- B. `HTTPRoute` with `matches.headers` specifying `name: X-Beta-Tester` and `value: "true"`
- C. `Ingress` with standard nginx annotation
- D. `ConfigMap` in `kube-system`

---

### Question 2: Multi-Tenant Network Isolation Default Deny
When establishing baseline network isolation for a multi-tenant namespace, what is the recommended security posture?
- A. Allow all ingress and egress traffic by default.
- B. Apply a default-deny `NetworkPolicy` blocking all ingress and egress traffic, then selectively whitelist permitted DNS egress (port 53) and namespace-internal communication.
- C. Delete `kube-proxy`.
- D. Run pods with `hostNetwork: true`.

---

### Question 3: Karpenter NodePool Expiration Policy
How can a platform engineer enforce that no worker node runs longer than 14 days to comply with enterprise AMI patching and compliance rotation guidelines?
- A. By running a daily cron job that kills random EC2 instances.
- B. By setting `disruption.expireAfter: 336h` (14 days) in the Karpenter `NodePool` specification.
- C. By setting `ttlSecondsAfterFinished: 1209600`.
- D. By configuring an AWS CloudWatch alarm to terminate instances.

---

### Question 4: KEDA Scaler Authentication via IAM Roles (IRSA)
To allow KEDA to authenticate to AWS SQS without embedding hardcoded AWS credentials in Kubernetes Secrets, which KEDA resource is used?
- A. `ClusterRoleBinding`
- B. `TriggerAuthentication` referencing a ServiceAccount annotated with the AWS IAM Role ARN (`eks.amazonaws.com/role-arn`)
- C. `KubeletConfiguration`
- D. `ConfigMap`

---

### Question 5: vCluster Performance vs Full Cluster
Why does a vCluster consume substantially fewer infrastructure resources than running a standalone Kubernetes cluster?
- A. Because vCluster skips kernel system calls entirely.
- B. Because vCluster reuses the host cluster's worker nodes, kubelets, CNI network plugins, and storage drivers, running only a lightweight virtual API server and syncer.
- C. Because vCluster runs inside Docker-in-Docker.
- D. Because vCluster cannot run containers.

---

### Question 6: HPA v2 Scaling Policies & Stabilization Window
An application experiences sudden spikes in traffic that cause pod replicas to double, but within 60 seconds the traffic subsides, causing pods to terminate prematurely and thrash under intermittent loads. Which HPA v2 section configures scale-down dampening?
- A. `behavior.scaleDown.stabilizationWindowSeconds`
- B. `spec.minReplicas`
- C. `spec.targetCPUUtilizationPercentage`
- D. `metadata.generation`

---

### Question 7: Cilium ClusterMesh Multi-Cluster Networking
When connecting multiple Kubernetes clusters across different cloud regions or data centers using Cilium ClusterMesh, what is a mandatory networking prerequisite?
- A. All clusters must have identical overlapping Pod CIDR ranges.
- B. Pod IP CIDRs and Node IP ranges across all interconnected clusters must be strictly non-overlapping and routable.
- C. Only TCP port 80 may be open between nodes.
- D. Clusters must run on the exact same physical server.

---

### Question 8: Gateway API GRPCRoute Capabilities
What operational advantage does `GRPCRoute` provide over generic `HTTPRoute` when routing microservices traffic?
- A. `GRPCRoute` can inspect and match on gRPC service names and gRPC methods directly, as well as handle gRPC status codes and streaming trailers.
- B. `GRPCRoute` does not require TCP.
- C. `GRPCRoute` only works on port 53.
- D. `GRPCRoute` replaces the application binary.

---

### Question 9: Karpenter Inter-Pod Anti-Affinity Handling
When a deployment defines `podAntiAffinity` requiring pods to be spread across distinct availability zones (`topologyKey: topology.kubernetes.io/zone`), how does Karpenter handle node provisioning?
- A. Karpenter fails to schedule the pods.
- B. Karpenter evaluates the anti-affinity topology constraint and provisions instances in separate availability zones to satisfy the scheduling rule.
- C. Karpenter ignores pod anti-affinity.
- D. Karpenter schedules all pods on a single node.

---

### Question 10: Hierarchical Namespaces Subnamespace Lifecycle
In HNC, what happens to child subnamespaces when a parent namespace is deleted, assuming default cascading deletion is enabled?
- A. Child subnamespaces remain orphaned forever.
- B. Child subnamespaces and all contained workloads are safely cascaded and deleted in synchronization with the parent.
- C. The physical cluster is deleted.
- D. The nodes are rebooted.

---

## Answer Key & Rationales

1. **B** — `HTTPRoute` supports rich L7 matching criteria, including HTTP headers (`matches.headers`), query parameters, and URL prefixes.
2. **B** — A zero-trust platform baseline mandates default-deny ingress and egress NetworkPolicies, opening egress to kube-dns port 53 and strictly required endpoints.
3. **B** — Karpenter v1.0 `disruption.expireAfter` sets an automated lifetime for nodes, cordoning and replacing them after the specified period (e.g. `336h` = 14 days).
4. **B** — KEDA `TriggerAuthentication` can delegate pod identity to an AWS IAM Role for Service Accounts (IRSA) via `podIdentity: { provider: aws }`.
5. **B** — vCluster only runs a virtual API server (k3s or k8s) and syncer; physical compute execution, storage provisioning, and CNI networking are handled by the host cluster.
6. **A** — `behavior.scaleDown.stabilizationWindowSeconds` (e.g., 300s) prevents premature scale-down actions during fluctuating load.
7. **B** — Cilium ClusterMesh requires non-overlapping Pod and Service CIDR ranges between clusters so that eBPF programs can route packets across clusters without NAT conflicts.
8. **A** — `GRPCRoute` provides native first-class constructs for matching gRPC services and methods (`matches.method.service` and `matches.method.method`).
9. **B** — Karpenter natively understands Kubernetes scheduling constraints (node affinity, pod anti-affinity, topology spread) and dynamically launches nodes across matching zones.
10. **B** — HNC cascades namespace deletion down the hierarchy tree when subnamespaces are managed as subresources.
