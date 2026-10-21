# Domain 3: Kubernetes Platform Engineering — Essential Technical Documentation & Guide

> **Core Focus:** Kubernetes API Machinery, Custom Resource Definitions (CRDs), Operator Pattern, Controller-Runtime Reconcilers, Multi-Tenancy Architecture (Namespaces vs vCluster vs Fleets), Dynamic Admission Policies, and Autoscaling (Karpenter, KEDA).

---

## 1. Authoritative Architecture References & Documentation

### 1.1 [Kubernetes API Machinery Architecture](https://kubernetes.io/docs/concepts/overview/kubernetes-api/)
* **Canonical Scope:** Internal architecture of `kube-apiserver`, etcd storage serialization, resource versions, and optimistic concurrency control (`metadata.resourceVersion`).
* **Informers & Workqueues:**
  * ListWatch API using HTTP chunked transfer encoding and etcd streaming watches.
  * SharedInformerFactory: local client cache preventing direct etcd polling.
  * RateLimitingWorkqueue: deduplication and exponential backoff retry semantics.

### 1.2 [Kubernetes Multi-Tenancy Working Group (MTWG) Best Practices](https://github.com/kubernetes-sigs/multi-tenancy)
* **Comparative Multi-Tenancy Architecture:**
  | Model | Isolation Level | Blast Radius | Overhead | Best Suited For |
  | :--- | :--- | :--- | :--- | :--- |
  | **Soft Multi-Tenancy (Namespaces)** | Logical (RBAC, Quotas, NetPol) | Shared control plane & node kernel | Lowest | Trusted internal teams within one company |
  | **Virtual Clusters (vCluster)** | Dedicated control plane; Shared nodes | Control plane isolated; node kernel shared | Low | Complex dev environments needing CRDs/ClusterRoles |
  | **Hard Multi-Tenancy (Multi-Cluster)** | Physical / Cloud instance | Complete isolation | Highest | Untrusted external tenants, PCI/HIPAA compliance |

### 1.3 [Karpenter Architecture & Node Auto-Provisioning](https://karpenter.sh/docs/)
* **Canonical Scope:** Flexible, group-less node autoscaler provisioning right-sized EC2 compute directly based on unschedulable pod requirements.
* **Core Primitives:**
  * `NodePool`: Defines compute constraints (architectures, families, Spot vs On-Demand, capacity limits).
  * `EC2NodeClass`: Defines cloud provider details (subnets, security groups, AMIs, IAM instance profiles).
  * Disruption & Consolidation: Automatically replaces underutilized or expensive nodes with smaller or spot nodes.

### 1.4 [KEDA: Kubernetes Event-Driven Autoscaling](https://keda.sh/docs/)
* **Canonical Scope:** Fine-grained application autoscaling driven by external metrics (SQS queue depth, Kafka consumer lag, Redis list length, Prometheus queries).
* **Architecture:** `ScaledObject` exposing external metrics to Kubernetes HPA controller via an External Metrics API server.

---

## 2. Operator Development & Admission Policies

### 2.1 Operator Pattern & `controller-runtime`
* **Core Tenets:** Level-triggered reconciliation loop (`Reconcile(ctx, req)`) converging actual state to desired state.
* **Idempotency:** Reconcilers must be completely idempotent; events can be re-delivered out of order or multiple times.

### 2.2 [ValidatingAdmissionPolicy & CEL](https://kubernetes.io/docs/reference/access-authn-authz/validating-admission-policy/)
* In-tree declarative admission control using Common Expression Language (CEL). Replaces complex, brittle webhook microservices with high-performance native API server evaluations.

---

## 3. Staff-Level Interview Angle & Architectural Reflexes

1. **"One Big Cluster vs Many Small Clusters":**
   * *Trade-offs:* Large clusters offer high bin-packing efficiency and simpler cross-service networking, but suffer from large blast radiuses, etcd scaling limits (max 8 GB), and difficult Kubernetes version upgrades.
   * *Staff Standard:* Separate clusters by environment (Dev vs Prod) and compliance domain, using Fleet Management tooling (Argo CD, Cluster API) to treat clusters like cattle.
2. **Custom Controller Event Storms:**
   * Reconciler updates the object status without updating the resourceVersion or triggers a continuous infinite reconcile loop.
   * *Mitigation:* Use status subresources (`/status`) and filter events using predicates (`predicate.GenerationChangedPredicate`).

---

## 4. Reading & Study Checklist

- [ ] Read *Kubernetes API Machinery Architecture*
- [ ] Read *Kubernetes MTWG Multi-Tenancy Best Practices*
- [ ] Review Karpenter NodePool consolidation and interruption handling
- [ ] Study KEDA ScaledObject integration with Kafka/SQS
- [ ] Master ValidatingAdmissionPolicy CEL syntax
