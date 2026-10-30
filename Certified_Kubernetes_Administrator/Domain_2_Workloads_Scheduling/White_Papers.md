# Domain 2: Workloads & Scheduling — Essential Documentation & Technical Reference Guide

> **Blueprint Weight:** 15% of Certified Kubernetes Administrator (CKA)  
> **Core Objective:** Manage Pod lifecycle, Deployments, DaemonSets, StatefulSets, Jobs/CronJobs; configure advanced scheduling via node affinity, taints/tolerations, priority classes, and enforce workload resilience with Pod Disruption Budgets (PDBs).

---

## 1. Official Documentation & Authoritative References

### 1.1 [Pod Lifecycle & Native Sidecars](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/)
* **Canonical Scope:** Container startup, probes, lifecycle hooks, and native sidecar containers (KEP-753).
* **Native Sidecar Containers:**
  * Defined inside `initContainers` with `restartPolicy: Always`.
  * Starts before application containers and stays running for the lifetime of the Pod (ideal for service meshes, log shippers, and proxy sidecars).

### 1.2 [Assigning Pods to Nodes (Affinity, Anti-Affinity, NodeSelector)](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/)
* **Canonical Scope:** Directing pod placement onto specific physical or virtual nodes.
* **Affinity Rules:**
  * `nodeSelector`: Simple key-value matching.
  * `nodeAffinity`:
    * `requiredDuringSchedulingIgnoredDuringExecution` (Hard constraint — pod won't schedule unless matched).
    * `preferredDuringSchedulingIgnoredDuringExecution` (Soft constraint — scheduler tries best effort with weighted preferences).
  * `podAffinity` / `podAntiAffinity`: Scheduling pods relative to other pods (e.g., co-locating cache pods or spreading replicas across failure domains using `topologyKey: topology.kubernetes.io/zone`).

### 1.3 [Taints and Tolerations](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/)
* **Canonical Scope:** Allowing nodes to repel sets of pods.
* **Taint Effects:**
  * `NoSchedule`: New pods without toleration will not schedule; existing pods remain unaffected.
  * `PreferNoSchedule`: Scheduler attempts to avoid placing pod on tainted node, but will if necessary.
  * `NoExecute`: New pods without toleration will not schedule, AND existing pods without matching toleration are immediately evicted.

### 1.4 [Pod Priority, Preemption & Pod Disruption Budgets](https://kubernetes.io/docs/concepts/scheduling-eviction/pod-priority-preemption/)
* **Canonical Scope:** Resource preemption under starvation and availability guarantees during maintenance.
* **Mechanisms:**
  * **PriorityClass:** Defines integer `value` (e.g., 1,000,000). Higher-priority pods preempt lower-priority pods when cluster is out of CPU/Memory.
  * **PodDisruptionBudget (PDB):** Enforces `minAvailable` or `maxUnavailable` during voluntary disruptions (`kubectl drain`).

---

## 2. High-Frequency Exam Traps & Practical Reflexes

1. **`kubectl drain` Blocked by Local Storage or DaemonSets:**
   * Node drain fails with errors regarding DaemonSets or emptyDir volumes.
   * *Resolution:* Run `kubectl drain <node> --ignore-daemonsets --delete-emptydir-data --force`.
2. **StatefulSet Headless Service:**
   * StatefulSet pods fail to achieve stable network identities (`pod-0.service.namespace.svc.cluster.local`).
   * *Resolution:* Verify the `serviceName:` field in the StatefulSet spec matches the name of a Headless Service (`clusterIP: None`).
3. **CronJob Concurrency Policies:**
   * Long-running jobs overlap and exhaust cluster resources.
   * *Resolution:* Configure `concurrencyPolicy: Forbid` or `Replace` in the CronJob spec.

---

## 3. Reading & Study Checklist

- [ ] Read *Pod Lifecycle & Native Sidecars*
- [ ] Read *Assigning Pods to Nodes*
- [ ] Master NodeAffinity (Hard vs Soft) and PodAntiAffinity with topologyKey
- [ ] Practice applying and tolerating taints (`kubectl taint nodes <node> key=value:NoSchedule`)
- [ ] Configure PodDisruptionBudgets (PDB) and test with node drains
