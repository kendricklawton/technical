# Domain 4: Platform Architecture & Scalability — Quiz 1

**Exam:** Certified Cloud Native Platform Engineer (CNPE)  
**Format:** 10 Hands-On Scenario & Performance Questions with Detailed Rationales

---

### Question 1: Soft vs Hard Multi-Tenancy Boundary
A software engineering team requires the ability to install custom third-party operators and define cluster-scoped CustomResourceDefinitions (CRDs) for testing. They share a physical Kubernetes cluster with critical production services. Which isolation mechanism allows this without giving them cluster-admin access to the physical host cluster?
- A. Hierarchical Namespaces (HNC)
- B. Virtual Clusters (vCluster)
- C. ResourceQuota and LimitRange
- D. Cilium NetworkPolicies

---

### Question 2: KEDA Scale-To-Zero Capabilities
Why can KEDA scale a Deployment down to 0 replicas while standard Kubernetes Horizontal Pod Autoscaler (HPA) v2 cannot do so natively based on CPU utilization?
- A. CPU utilization is an undefined percentage when replicas are 0 (division by zero), whereas KEDA evaluates absolute external event metrics (such as queue message count) via an external metrics server.
- B. HPA is deprecated in Kubernetes v1.31.
- C. KEDA deletes the Deployment manifest from Git.
- D. KEDA only works with Docker containers.

---

### Question 3: Karpenter vs Cluster Autoscaler Architecture
What is the core architectural differentiator in how Karpenter provisions compute compared to Cluster Autoscaler?
- A. Karpenter manages static cloud auto-scaling node groups (ASGs) by incrementing node group desired capacity.
- B. Karpenter bypasses node groups entirely, evaluating pending pod scheduling constraints directly against the cloud provider's compute API to launch right-sized VMs within seconds.
- C. Karpenter requires manual intervention before provisioning instances.
- D. Cluster Autoscaler launches instances faster than Karpenter.

---

### Question 4: Gateway API ReferenceGrant Security
Under Kubernetes Gateway API, why is the `ReferenceGrant` resource essential when an `HTTPRoute` in the `app-team` namespace attempts to bind to a `Gateway` in the `infra-team` namespace?
- A. To speed up DNS queries.
- B. It enforces explicit administrative cross-namespace trust, preventing unauthorized tenants from claiming domain routing or accessing services/secrets across namespace boundaries.
- C. It replaces TLS certificates.
- D. It enables SSH access to pods.

---

### Question 5: Karpenter Consolidation Disruption Budgets
During a major black-friday sales event, the platform team wants to prevent Karpenter from terminating or consolidating any underutilized nodes to avoid unnecessary pod rescheduling risk. Which Karpenter `NodePool` configuration achieves this?
- A. Setting `limits.cpu: 0`
- B. Configuring a disruption budget with `schedule: "0 0 * * *"` and `nodes: "0"` or setting `consolidationPolicy: WhenEmpty`
- C. Deleting the Karpenter deployment controller
- D. Uninstalling Cilium

---

### Question 6: Hierarchical Namespace Controller (HNC)
In an enterprise organization with many sub-teams, what operational advantage does the Hierarchical Namespace Controller (HNC) provide over flat namespace architectures?
- A. It creates nested Linux namespaces inside the kernel.
- B. It allows parent namespaces to cascade common RBAC roles, ResourceQuotas, and NetworkPolicies downward to child subnamespaces automatically.
- C. It eliminates the need for Kubernetes API servers.
- D. It allows pods to run without container images.

---

### Question 7: KEDA ScaledJob vs ScaledObject
When should a platform engineer use a KEDA `ScaledJob` instead of a `ScaledObject`?
- A. When scaling long-running web servers.
- B. For batch, event-driven processing where each message or batch of messages should spawn a run-to-completion Kubernetes `Job` rather than continuously running pods.
- C. Only when deploying daemonsets.
- D. When the cluster is offline.

---

### Question 8: Gateway API Role Separation
Which Gateway API resource is managed exclusively by the Infrastructure / Cluster Provider team rather than application developers?
- A. `HTTPRoute`
- B. `GRPCRoute`
- C. `GatewayClass`
- D. `BackendRef`

---

### Question 9: Cluster API (CAPI) Reconcile Pattern
In declarative Kubernetes cluster lifecycle management using Cluster API (CAPI), which resource defines the desired Kubernetes control plane version and etcd topology?
- A. `MachineDeployment`
- B. `KubeadmControlPlane`
- C. `ConfigMap`
- D. `ClusterIssuer`

---

### Question 10: Horizontal vs Vertical Pod Autoscaling Conflict
A platform engineer deploys both HPA (Horizontal Pod Autoscaler) and VPA (Vertical Pod Autoscaler) on the same microservice deployment targeting CPU utilization. What dangerous failure mode will occur?
- A. The pods will be deleted permanently.
- B. A race condition / oscillation loop: high CPU causes HPA to add replicas, dropping per-pod CPU, which causes VPA to downsize pod CPU requests, triggering further oscillations.
- C. Kubernetes will disable the Linux kernel scheduler.
- D. etcd database will become read-only.

---

## Answer Key & Rationales

1. **B** — Virtual Clusters (`vCluster`) create a separate virtual control plane (API server and controller manager) inside a host namespace, granting tenants virtual cluster-admin rights and isolated CRD management.
2. **A** — CPU utilization percentage cannot be computed when zero pods are running ($\text{CPU} / 0$); KEDA reads queue lengths from external systems, allowing it to scale workloads down to 0 and back to 1.
3. **B** — Karpenter communicates directly with cloud APIs (e.g., AWS EC2, Azure VMs) to dynamically select and launch individual VM instances tailored specifically to unscheduled pod constraints.
4. **B** — `ReferenceGrant` prevents cross-namespace attack vectors where a rogue tenant routes incoming traffic through an unauthorized Gateway or references secrets across namespace boundaries.
5. **B** — Disruption budgets in Karpenter v1.0 allow platform teams to define time-based schedules that freeze node consolidation by setting `nodes: "0"`.
6. **B** — HNC establishes parent-child relationships between namespaces and automatically propagates policies, labels, and role bindings down the hierarchy.
7. **B** — `ScaledJob` launches finite, run-to-completion `Job` resources per queue message, saving resources for non-daemon workloads.
8. **C** — `GatewayClass` defines the controller implementation (e.g. Cilium, Istio, Envoy) and is cluster-scoped, managed strictly by platform infrastructure admins.
9. **B** — In CAPI, `KubeadmControlPlane` reconciles control plane machine count, Kubernetes version, kubeadm init/join configurations, and etcd cluster state.
10. **B** — Running HPA and VPA simultaneously on the same metric induces unstable positive feedback loops and thrashing between pod replica counts and container resource limits.
