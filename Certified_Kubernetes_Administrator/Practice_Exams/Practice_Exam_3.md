# CKA Full-Length Performance Practice Exam 3

**Time Limit:** 120 Minutes  
**Passing Score:** 66% (Score Target: $\ge 90\%$)  
**Format:** 17 Hard/Challenge Mode Performance Tasks Across Broken Clusters

---

### Task 1: Complete Cluster Recovery from Cold etcd Snapshot (Weight: 8%)
**Context:** `kubectl config use-context k8s`  
**Scenario:** A catastrophic disaster wiped the etcd data directory on `cp-1`. Restore the entire cluster state from an uncompressed snapshot `/backup/etcd-snapshot-final.db` into `/var/lib/etcd-production-restored`, reconfigure static pod manifests, and verify all worker nodes reconnect.

---

### Task 2: Multi-Node Control Plane High Availability Triage (Weight: 8%)
**Context:** `kubectl config use-context k8s`  
**Scenario:** On a 3-node HA control plane cluster, `cp-2` has lost API server connectivity. Diagnose load balancer health check failures, renew expired etcd peer certificates, and restore 3/3 HA quorum.

---

### Task 3: Kubeadm Minor Version Upgrade under Pod Disruption Constraints (Weight: 7%)
**Context:** `kubectl config use-context k8s`  
**Scenario:** Upgrade worker node `node01` while strictly respecting an active `PodDisruptionBudget` for a critical production deployment without causing downtime.

---

### Task 4: Gateway API HTTPRoute Header Rewriting & Mirroring (Weight: 6%)
**Context:** `kubectl config use-context k8s`  
**Scenario:** Configure an `HTTPRoute` with request header modification (injecting `X-Forwarded-Host`) and request mirroring (copying 10% of traffic to a shadow service `shadow-svc`).

---

### Task 5: Diagnose Broken Container Runtime & Cgroup Driver Conflict (Weight: 7%)
**Context:** `ssh node02`  
**Scenario:** Worker node `node02` fails to register with error `failed to run Kubelet: misconfiguration: cgroup driver: cannot execute systemd`. Reconcile `containerd` `/etc/containerd/config.toml` (`SystemdCgroup = true`) with kubelet config, restart services, and restore node readiness.

---

### Task 6: NetworkPolicy Multi-Namespace Interconnect (Weight: 6%)
**Context:** `kubectl config use-context k8s`  
**Scenario:** In namespace `analytics`, allow ingress traffic on port 9092 ONLY from namespaces labeled `env: prod` AND pods labeled `role: ingest`. Block all other traffic.

---

### Task 7: Dynamic CSI Provisioning with Encrypted Volume Claims (Weight: 6%)
**Context:** `kubectl config use-context k8s`  
**Scenario:** Create a StorageClass specifying KMS encryption parameters for cloud EBS volumes. Create a PVC claiming 50Gi and verify binding.

---

### Task 8: Triage Pending Pods due to Resource Request Exhaustion (Weight: 6%)
**Context:** `kubectl config use-context k8s`  
**Scenario:** Multiple pods in namespace `batch` are stuck in `Pending`. Analyze cluster resource allocatable capacity, identify CPU/Memory oversubscription, and adjust pod resource requests to allow scheduling.

---

### Task 9: Repair Misconfigured CoreDNS Service & Endpoints (Weight: 6%)
**Context:** `kubectl config use-context k8s`  
**Scenario:** Internal service discovery has stopped working cluster-wide. Triage the `kube-dns` service in `kube-system`, repair the mismatched selector (`k8s-app: kube-dns`), and verify DNS lookups succeed from a test pod.

---

### Task 10: Pod Anti-Affinity for Zone-Level Resilience (Weight: 6%)
**Context:** `kubectl config use-context k8s`  
**Scenario:** Configure a Deployment with 3 replicas to guarantee that no two pods are scheduled in the same Availability Zone using `podAntiAffinity` and `topologyKey: topology.kubernetes.io/zone`.

---

### Task 11: StatefulSet Volume Expansion Drill (Weight: 6%)
**Context:** `kubectl config use-context k8s`  
**Scenario:** Expand the persistent volume claims of a running 3-replica StatefulSet from 5Gi to 15Gi without deleting the pods or losing data.

---

### Task 12: RBAC Aggregated ClusterRoles (Weight: 5%)
**Context:** `kubectl config use-context k8s`  
**Scenario:** Create an aggregated `ClusterRole` named `view-all-custom` using label selector `rbac.example.com/aggregate-to-view: "true"`. Verify that permissions from secondary ClusterRoles are automatically dynamically inherited.

---

### Task 13: Troubleshoot Kube-Proxy Failure & Iptables Stalling (Weight: 6%)
**Context:** `kubectl config use-context k8s`  
**Scenario:** ClusterIP service traffic is failing on `node03`. Check `kube-proxy` daemonset logs, repair the corrupted IPVS/iptables ruleset, and verify proxy traffic routing.

---

### Task 14: Init Container Database Schema Migration (Weight: 5%)
**Context:** `kubectl config use-context k8s`  
**Scenario:** Configure a Pod with an `initContainer` that waits for a PostgreSQL database service port 5432 to become reachable before starting the primary application container.

---

### Task 15: Node Maintenance: Graceful Cordon, Drain, and Uncordon (Weight: 5%)
**Context:** `kubectl config use-context k8s`  
**Scenario:** Execute a safe maintenance procedure on `node01`: cordon the node, drain all workloads while preserving local storage, simulate hardware replacement, and uncordon.

---

### Task 16: VolumeSnapshot & Snapshot Restoration (Weight: 6%)
**Context:** `kubectl config use-context k8s`  
**Scenario:** Take a point-in-time `VolumeSnapshot` of a 20Gi data PVC named `db-data-pvc`. Create a new PVC `db-data-restored` restored from the snapshot.

---

### Task 17: Advanced JSONPath Filtering with Regular Expressions & Custom Columns (Weight: 5%)
**Context:** `kubectl config use-context k8s`  
**Scenario:** Using `kubectl`, generate a table displaying `POD_NAME`, `NODE_NAME`, and `STATUS` of all pods in namespace `kube-system` using `-o custom-columns` and save to `/opt/system-pods.txt`.
