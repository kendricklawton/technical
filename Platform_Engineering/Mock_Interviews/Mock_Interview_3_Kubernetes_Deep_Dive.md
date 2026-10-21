# Mock Interview 3: Kubernetes & Platform Deep Dive

Simulates a 60-minute technical deep dive using 4-tier drill-down ladders (L1 Recall -> L2 Mechanism -> L3 Trade-off -> L4 Systemic Design).

---

## Candidate Brief (Safe to read in advance)

- **Duration:** 60 minutes.
- **Format:** In-depth technical interview. The interviewer will pick 3–4 topics and drill down through 4 levels of depth on each topic until hitting your technical boundary.
- **Topics Covered (pool across all variants):** API machinery, CRDs & CEL, custom controllers, admission pipelines, pod networking & Service routing, Gateway API, storage & stateful workloads, multi-tenancy, cluster upgrades & etcd operations, and autoscaling.

---

## Interviewer Packet (Confidential — Do not read beforehand)

### Variant A: API Machinery, Controllers & Admission

#### Topic 1: Kubernetes API Machinery & Consensus
- **L1 (Recall):** What happens when you execute `kubectl apply -f pod.yaml`?
- **L2 (Mechanism):** Walk through `kube-apiserver` internal request processing: AuthN, AuthZ, Mutating Webhooks, Schema Validation, Validating Webhooks, etcd write.
- **L3 (Trade-off):** Why does etcd require an odd number of nodes? What happens to writes when a 5-node cluster loses 2 nodes vs 3 nodes?
- **L4 (Systemic Design):** How does Optimistic Concurrency Control (`resourceVersion`) prevent concurrent write corruption? How would you design a custom controller handling high-concurrency conflicts?

#### Topic 2: Controller Runtime & Operator Patterns
- **L1 (Recall):** What is the core loop of a Kubernetes Operator?
- **L2 (Mechanism):** Explain the difference between edge-triggered and level-triggered systems. How do `client-go` Informers and Indexer caches prevent API server overload?
- **L3 (Trade-off):** What are Finalizers, and what happens when an operator crashes while deleting an external cloud resource?
- **L4 (Systemic Design):** Design a custom controller that manages external AWS RDS databases with automated disaster recovery failover.

#### Topic 3: Admission Control & Policy Guardrails
- **L1 (Recall):** What is an admission webhook?
- **L2 (Mechanism):** How does `failurePolicy: Fail` vs `Ignore` affect cluster stability during webhook pod outages?
- **L3 (Trade-off):** Why does Kubernetes 1.30+ introduce in-tree `ValidatingAdmissionPolicy` using CEL to replace dynamic webhooks?
- **L4 (Systemic Design):** Author a native CEL policy that strictly forbids privileged containers and requires non-root execution.

### Variant B: Networking, Gateway API & Stateful Storage

#### Topic 1: Pod Networking & Service Routing
- **L1 (Recall):** What does the CNI plugin do when a pod starts, and what does kube-proxy do?
- **L2 (Mechanism):** Trace a request from a pod to a ClusterIP Service whose backend runs on another node: the DNS lookup, the virtual IP, the DNAT to a pod IP, and how the packet crosses nodes (overlay encapsulation vs. routed pod IPs).
- **L3 (Trade-off):** Compare kube-proxy's iptables, IPVS and nftables modes (nftables has been GA since 1.33) with an eBPF replacement such as Cilium. What happens to rule-update time and per-packet cost as a cluster grows past 5,000 Services, and what do you lose when kube-proxy is removed entirely?
- **L4 (Systemic Design):** An EKS cluster is heading to 2,000 nodes and 60,000 pods. Design the pod networking: IP address management (VPC CNI address exhaustion, prefix delegation, or an overlay), the Service datapath, and NetworkPolicy enforcement. What would you measure to prove the design holds up?

#### Topic 2: Gateway API & North-South Traffic
- **L1 (Recall):** Name the core Gateway API resources and which role (infrastructure provider, cluster operator, application developer) owns each.
- **L2 (Mechanism):** How does an HTTPRoute in a team namespace attach to a shared Gateway? Explain `parentRefs`, a listener's `allowedRoutes`, and when you need a `ReferenceGrant`.
- **L3 (Trade-off):** ingress-nginx was retired in March 2026. You own 400 Ingress objects, many relying on `nginx.ingress.kubernetes.io/*` annotations (rewrites, external auth, rate limits, configuration snippets). How do you plan the migration, and what doesn't translate cleanly to Gateway API?
- **L4 (Systemic Design):** Design the edge for 50 teams: shared vs. dedicated Gateways, certificate ownership (cert-manager), safe self-service routes, protection against one team claiming another team's hostname, and upgrading the gateway implementation without a fleet-wide outage.

#### Topic 3: Storage & Stateful Workloads
- **L1 (Recall):** How do a StorageClass, a PersistentVolumeClaim and a PersistentVolume relate, and what does the CSI driver do?
- **L2 (Mechanism):** Explain `volumeBindingMode: WaitForFirstConsumer`. What goes wrong with `Immediate` binding on a multi-zone cluster backed by zonal block storage?
- **L3 (Trade-off):** A StatefulSet pod's node in `us-east-1a` was terminated, and the replacement pod is stuck `Pending` with a volume node affinity conflict. Why, and what are your options: capacity in that zone, the `node.kubernetes.io/out-of-service` taint for a non-graceful shutdown, replicated storage, application-level replication, or snapshot and restore?
- **L4 (Systemic Design):** Should the platform offer databases on Kubernetes through an operator (for example CloudNativePG), or only managed services such as RDS? Lay out the decision framework and the backup and restore guarantees (RPO, RTO, restore drills, who gets paged) you would require before offering either.

### Variant C: Multi-Tenancy, Upgrades & Autoscaling

#### Topic 1: Multi-Tenancy & Isolation Boundaries
- **L1 (Recall):** What does a namespace isolate, and what does it not isolate?
- **L2 (Mechanism):** Walk through the baseline controls for a tenant namespace: RBAC, ResourceQuota and LimitRange, a default-deny NetworkPolicy, and Pod Security Admission at `restricted`. Which of these silently fail open when misconfigured?
- **L3 (Trade-off):** Compare namespace-per-tenant, virtual clusters (vCluster) and cluster-per-tenant on isolation, cost, operational load, and the API server as a shared failure domain. Where would you add sandboxed runtimes (gVisor, Kata Containers) or user namespaces?
- **L4 (Systemic Design):** One tenant's CI jobs hammer the API server with uncached LIST calls, and every team's deploys slow down. Design the protections: API Priority and Fairness (FlowSchemas and PriorityLevelConfigurations), client guidance (informers instead of polling), object-count quotas, and a tenancy model that bounds the blast radius.

#### Topic 2: Cluster Upgrades & etcd Operations
- **L1 (Recall):** In what order do you upgrade a cluster's components, and what does the version skew policy allow between the API server and kubelets?
- **L2 (Mechanism):** How do you find workloads that will break because the target release removes an API version they use? Think about the `apiserver_requested_deprecated_apis` metric, audit logs, manifests stored in Helm release secrets, and tools that scan for removed APIs.
- **L3 (Trade-off):** For a 40-cluster fleet, compare in-place rolling upgrades with blue/green cluster replacement. What does each cost, what does each protect against, and where do PodDisruptionBudgets and surge capacity fit?
- **L4 (Systemic Design):** A cluster's etcd hits its space quota and raises the `NOSPACE` alarm (`mvcc: database space exceeded`), so writes fail. Walk through recovery (compaction, defragmenting each member, disarming the alarm), then design etcd operations for the fleet: snapshot cadence, restore drills, size alerts, and what tenants must never be allowed to store in etcd. What changes on a managed control plane?

#### Topic 3: Autoscaling & Capacity
- **L1 (Recall):** What are the jobs of the HPA, the VPA and a node autoscaler (Cluster Autoscaler or Karpenter)?
- **L2 (Mechanism):** Walk through the HPA calculation, `desiredReplicas = ceil(currentReplicas * currentMetricValue / desiredMetricValue)`. Why do the tolerance band and the scale-down stabilization window exist?
- **L3 (Trade-off):** Karpenter consolidation keeps evicting a latency-sensitive service during business hours. Which controls do you reach for (NodePool disruption budgets with schedules, `consolidationPolicy`, `consolidateAfter`, the `karpenter.sh/do-not-disrupt` annotation, PodDisruptionBudgets), and what does each cost in spend?
- **L4 (Systemic Design):** Design autoscaling for a mixed fleet: request-driven web services, queue workers that must scale to zero (KEDA), and a GPU inference service with multi-minute cold starts. Which signal drives each scaler, and how do you manage scarce GPU capacity (dedicated node pools, Dynamic Resource Allocation, warm capacity)?

---

## Scoring Rubric

| Score | Anchor | Behavioral Criteria |
| :---: | :--- | :--- |
| **1** | Strong No Hire | Cannot explain the variant's fundamentals: API server stages and basic etcd consensus (A), how a packet reaches a pod or how a PVC binds (B), or what separates one tenant from another (C). |
| **2** | No Hire | Solid on L1 and L2, but struggles on L3 trade-offs and fails L4 architectural design. |
| **3** | Hire | Easily reaches L3 on all topics; provides strong, functional designs for L4. |
| **4** | Strong Hire | Masters L4 effortlessly; demonstrates deep knowledge of internals (`client-go`, the kernel datapath, etcd) and how they interact. |

---

## Self-Debrief Template
- **Variant:** A / B / C
- **Highest Level Reached (Topic 1):** L1 / L2 / L3 / L4
- **Highest Level Reached (Topic 2):** L1 / L2 / L3 / L4
- **Highest Level Reached (Topic 3):** L1 / L2 / L3 / L4
- **Technical Gaps Identified:**
