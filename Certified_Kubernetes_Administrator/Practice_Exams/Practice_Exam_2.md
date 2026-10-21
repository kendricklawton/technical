# CKA Full-Length Performance Practice Exam 2

**Time Limit:** 120 Minutes  
**Passing Score:** 66% (Score Target: $\ge 85\%$)  
**Format:** 17 Production-Grade Scenarios Across Multi-Node Topologies

---

### Task 1: Fix Crashed Control Plane after Static Pod Manifest Edit (Weight: 8%)
**Context:** `kubectl config use-context k8s`  
**Scenario:** A junior administrator edited `/etc/kubernetes/manifests/kube-apiserver.yaml`. The API server is down and `kubectl` commands fail.
1. Use `crictl` to inspect the failing static pod logs.
2. Correct the invalid command-line flag in the manifest.
3. Verify `kubectl get nodes` returns successfully.

---

### Task 2: Advanced etcd Snapshot on Clustered HA Topology (Weight: 7%)
**Context:** `kubectl config use-context k8s`  
**Scenario:** Connect to a 3-node stacked etcd cluster. Check cluster endpoint health using `etcdctl endpoint health`. Take an etcd snapshot from node `cp-1` and save it to `/opt/backup/etcd-ha.db`.

---

### Task 3: Ingress with TLS and Canary Annotations (Weight: 6%)
**Context:** `kubectl config use-context k8s`  
**Scenario:** Configure an NGINX Ingress resource with canary annotations directing 20% of traffic to service `app-v2` and 80% to `app-v1`.

---

### Task 4: StatefulSet with Headless Service & Persistent Storage (Weight: 7%)
**Context:** `kubectl config use-context k8s`  
**Scenario:** Create a StatefulSet named `kafka-broker` with 3 replicas. Each pod must mount a 5Gi persistent volume provisioned via `volumeClaimTemplates` and resolve DNS via a headless service named `kafka-headless`.

---

### Task 5: Gateway API Cross-Namespace Routing (Weight: 6%)
**Context:** `kubectl config use-context k8s`  
**Scenario:** Configure a `Gateway` in namespace `infra` that accepts `HTTPRoute` attachments from namespace `sales` using `allowedRoutes.namespaces.from: Selector`.

---

### Task 6: Triage OOMKilled Pod and Adjust cgroup Memory Limits (Weight: 6%)
**Context:** `kubectl config use-context k8s`  
**Scenario:** Pod `report-generator` terminated with `Exit Code 137 (OOMKilled)`. Adjust the deployment resource limits to double the memory allocation to prevent future eviction.

---

### Task 7: CSI StorageClass with Topology Constraints (Weight: 6%)
**Context:** `kubectl config use-context k8s`  
**Scenario:** Create a StorageClass `topology-aware-sc` with `allowedTopologies` restricting provisioning to zone `us-east-1a`.

---

### Task 8: ClusterRole & ServiceAccount Delegation (Weight: 5%)
**Context:** `kubectl config use-context k8s`  
**Scenario:** Create a ServiceAccount `monitoring-agent` in namespace `monitoring`. Create a ClusterRole granting read-only access to nodes, pods, and namespaces, and bind it to the ServiceAccount.

---

### Task 9: Drain Worker Node without Evicting DaemonSets (Weight: 6%)
**Context:** `kubectl config use-context k8s`  
**Scenario:** Drain worker node `node03` for OS maintenance, safely ignoring DaemonSets and deleting local `emptyDir` data.

---

### Task 10: Horizontal Pod Autoscaler (HPA) Tuning (Weight: 6%)
**Context:** `kubectl config use-context k8s`  
**Scenario:** Configure an HPA for deployment `frontend` targeting 60% average CPU utilization with min replicas 2 and max replicas 10.

---

### Task 11: NetworkPolicy Default-Deny-All with Explicit Egress (Weight: 6%)
**Context:** `kubectl config use-context k8s`  
**Scenario:** Create a default deny NetworkPolicy in namespace `restricted` that blocks all ingress and egress, except egress to port 53 (UDP/TCP) for CoreDNS.

---

### Task 12: Kubeadm Worker Node Join Drill (Weight: 6%)
**Context:** `kubectl config use-context k8s`  
**Scenario:** Generate a new join token on the control plane with `kubeadm token create --print-join-command`. SSH to a new node `node04` and join it to the cluster.

---

### Task 13: Debug Kubelet Certificate Expiration (Weight: 6%)
**Context:** `ssh node02`  
**Scenario:** Node `node02` is `NotReady`. Triage `/var/lib/kubelet/pki/`, regenerate the kubelet client certificate, and approve the CSR using `kubectl certificate approve`.

---

### Task 14: Projected Volume Deployment (Weight: 5%)
**Context:** `kubectl config use-context k8s`  
**Scenario:** Create a Pod that mounts a projected volume containing both a ConfigMap and a Secret into directory `/etc/projected`.

---

### Task 15: CronJob Troubleshooting & Concurrency Policy (Weight: 5%)
**Context:** `kubectl config use-context k8s`  
**Scenario:** A CronJob is failing to launch on schedule due to overlapping executions. Configure `concurrencyPolicy: Forbid` and adjust the schedule to run every 15 minutes.

---

### Task 16: Pod Affinity for Low-Latency Co-Location (Weight: 5%)
**Context:** `kubectl config use-context k8s`  
**Scenario:** Configure pod `web-cache` to be scheduled on the same node as pod `web-frontend` using `podAffinity` with `topologyKey: kubernetes.io/hostname`.

---

### Task 17: JSONPath Sorting by CPU and Creation Timestamp (Weight: 5%)
**Context:** `kubectl config use-context k8s`  
**Scenario:** Extract a sorted list of all persistent volumes sorted by capacity: `kubectl get pv --sort-by=.spec.capacity.storage`.
