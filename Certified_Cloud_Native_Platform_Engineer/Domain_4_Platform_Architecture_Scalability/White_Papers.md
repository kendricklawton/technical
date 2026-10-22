# Domain 4: Platform Architecture & Scalability — Authoritative White Papers & CNCF Standards

> **CNPE Exam Weight:** 15% of Certified Cloud Native Platform Engineer

---

## 1. Authoritative References & Standards

- **[CNCF Multi-Tenancy Working Group (WG)](https://github.com/kubernetes-sigs/multi-tenancy):** Benchmarks for tenant isolation, Hierarchical Namespaces, and virtual control planes.
- **[Kubernetes Gateway API Specification](https://gateway-api.sigs.k8s.io/):** The official successor to Ingress; role-oriented networking, `GatewayClass`, `Gateway`, `HTTPRoute`, `GRPCRoute`, and `ReferenceGrant`.
- **[KEDA Architecture & Scaler Specifications](https://keda.sh/docs/latest/concepts/):** Event-driven horizontal autoscaling, metric serving, and zero-replica scaling.
- **[Karpenter v1.0 Architectural Design](https://karpenter.sh/docs/concepts/):** Node autoscaling, JIT compute provisioning, NodePool disruption budgets, and spot instance lifecycle management.
- **[Cluster API (CAPI) Core Specification](https://cluster-api.sigs.k8s.io/):** Declarative lifecycle management for Kubernetes clusters as first-class Kubernetes objects.

---

## 2. Staff SRE Architecture Patterns

### 2.1 Multi-Tenant Isolation Spectrum
When architecting platform environments, platform engineers evaluate the multi-tenancy trade-off matrix:

| Isolation Model | Blast Radius | Compute Overhead | Management Complexity | Best Suited For |
| :--- | :--- | :--- | :--- | :--- |
| **Namespace per Tenant** | High (shared control plane, shared kernel) | Near Zero | Low | Trusted internal microservices |
| **Hierarchical Namespaces (HNC)** | Moderate (cascading RBAC / Quotas) | Very Low | Low-Medium | Complex engineering teams with sub-teams |
| **Virtual Clusters (vCluster)** | Low (isolated API server, isolated CRDs) | Low (~0.1 CPU, ~150MB RAM per tenant) | Medium | Untrusted tenants, tenant-managed CRDs/operators |
| **Cluster per Tenant (CAPI)** | Minimal (dedicated VMs/nodes & control plane) | High (duplicated control planes) | High | Regulated environments (PCI-DSS, FedRAMP, HIPAA) |

### 2.2 Pod Disruptions & Karpenter Consolidation
Aggressive cost optimization can cause service instability if node disruption budgets are ignored. Platform engineers must enforce:
1. **Pod Disruption Budgets (PDB):** Every production deployment must define `minAvailable: 1` or `maxUnavailable: 25%`. Karpenter will respect PDBs during consolidation.
2. **Karpenter Disruption Budgets:**
   ```yaml
   disruption:
     consolidationPolicy: WhenEmptyOrUnderutilized
     consolidateAfter: 30s
     budgets:
       - nodes: "20%"
       - schedule: "0 9 * * 1-5" # Business hours freeze
         duration: 8h
         nodes: "0" # Do not disrupt nodes during peak trading hours
   ```

### 2.3 Gateway API RBAC Separation
Under traditional Ingress, application developers frequently need cluster-wide Ingress editing privileges or break shared ingress controllers by committing conflicting path rules. Gateway API solves this through RBAC segregation:
- **Infrastructure Team:** Manages `GatewayClass` and `Gateway` objects in a centralized `network-infra` namespace.
- **Application Teams:** Only manage `HTTPRoute` objects in their local namespaces, attaching to the central Gateway via `parentRefs`.

---

## 3. Production Failure Modes & Anti-Patterns

| Failure Mode | Root Cause | Engineering Remediation |
| :--- | :--- | :--- |
| **HPA and VPA Infinite Oscillations** | Configuring both HPA and VPA on the identical resource metric (e.g., CPU) causes competing scaling actions. | Restrict VPA to `mode: "Off"` (recommendation only) or use VPA exclusively for memory while HPA scales replicas on CPU or KEDA queue depth. |
| **KEDA Scale-to-Zero Connection Drops** | Scaling HTTP workloads to 0 replicas causes initial incoming HTTP requests to fail with connection refused before pods spin up. | Pair KEDA with an HTTP add-on / queue buffer proxy (such as KEDA HTTP Addon or Knative Queue Proxy) to hold requests during cold start. |
| **Cross-Namespace Route Hijacking** | Malicious or misconfigured tenant attaches an `HTTPRoute` claiming the host `api.company.com` and path `/` ahead of another service. | Use Gateway `allowedRoutes.namespaces` filters or require `ReferenceGrant` objects to explicitly approve attached routes. |
| **Spot Instance Termination Storm** | Relying 100% on Spot instances without diversification or Pod Disruption Budgets during cloud provider capacity evictions. | Enforce multi-AZ and multi-instance-type requirements in `NodePool` specs and maintain a baseline percentage of On-Demand capacity. |
