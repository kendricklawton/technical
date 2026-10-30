# CKA Full-Length Performance Practice Exam 1

**Time Limit:** 120 Minutes  
**Passing Score:** 66% (Score Target: $\ge 80\%$)  
**Format:** 17 Real-World Performance Tasks across Multi-Node Clusters

---

### Task 1: etcd Database Snapshot Backup and Restore (Weight: 7%)
**Context:** `kubectl config use-context k8s`  
**Scenario:** Take an etcd snapshot of the running cluster and restore it.
1. Save an etcd snapshot using `etcdctl` to `/opt/backup/etcd-snapshot.db` using certificates located in `/etc/kubernetes/pki/etcd/`.
2. Restore the snapshot to a new data directory `/var/lib/etcd-backup-restored`.
3. Update `/etc/kubernetes/manifests/etcd.yaml` to point to `/var/lib/etcd-backup-restored` and verify the control plane recovers.

**Exact Commands:**
```bash
ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  snapshot save /opt/backup/etcd-snapshot.db

ETCDCTL_API=3 etcdctl snapshot restore /opt/backup/etcd-snapshot.db \
  --data-dir=/var/lib/etcd-backup-restored
```

---

### Task 2: Control Plane & Worker Node Kubeadm Upgrade (Weight: 8%)
**Context:** `kubectl config use-context k8s`  
**Scenario:** Upgrade the control plane node `cp-node` from Kubernetes version `1.30.0` to `1.31.0`.
1. Drain the node: `kubectl drain cp-node --ignore-daemonsets`.
2. Upgrade `kubeadm`: `apt-mark unhold kubeadm && apt-get update && apt-get install -y kubeadm=1.31.0-1.1 && apt-mark hold kubeadm`.
3. Plan and apply: `kubeadm upgrade plan` -> `kubeadm upgrade apply v1.31.0 -y`.
4. Upgrade `kubelet` and `kubectl`: `apt-mark unhold kubelet kubectl && apt-get install -y kubelet=1.31.0-1.1 kubectl=1.31.0-1.1 && apt-mark hold kubelet kubectl`.
5. Restart kubelet: `systemctl daemon-reload && systemctl restart kubelet`.
6. Uncordon the node: `kubectl uncordon cp-node`.

---

### Task 3: RBAC Role & RoleBinding Configuration (Weight: 5%)
**Context:** `kubectl config use-context k8s`  
**Scenario:** Create a Role named `developer-role` in namespace `development` granting permissions to `get`, `list`, `watch`, `create`, and `update` on `pods` and `deployments`. Bind this role to user `dev-user` via a RoleBinding named `dev-user-binding`.

---

### Task 4: Multi-Container Pod with Native Sidecar (KEP-753) (Weight: 5%)
**Context:** `kubectl config use-context k8s`  
**Scenario:** Create a Pod `app-with-sidecar` in namespace `default` containing an application container running `nginx` and a native sidecar container running `busybox` configured inside `initContainers` with `restartPolicy: Always`.

---

### Task 5: Gateway API HTTPRoute Deployment (Weight: 6%)
**Context:** `kubectl config use-context k8s`  
**Scenario:** Deploy an `HTTPRoute` named `catalog-route` in namespace `shop` attached to Gateway `main-gw`. Route traffic with path prefix `/catalog` to Service `catalog-svc` on port 80.

---

### Task 6: NetworkPolicy for Intra-Namespace Isolation (Weight: 6%)
**Context:** `kubectl config use-context k8s`  
**Scenario:** In namespace `secure`, create a NetworkPolicy `db-netpol` that allows incoming TCP connections to pod `database` on port 5432 ONLY from pods labeled `role: api-worker`. Block all other ingress.

---

### Task 7: Storage: PersistentVolume & StorageClass with WaitForFirstConsumer (Weight: 6%)
**Context:** `kubectl config use-context k8s`  
**Scenario:** Create a StorageClass `local-sc` with provisioner `kubernetes.io/no-provisioner` and `volumeBindingMode: WaitForFirstConsumer`. Create a PV `local-pv` of size 5Gi using `local-sc` and a hostPath `/mnt/data`.

---

### Task 8: PVC Online Volume Resizing (Weight: 5%)
**Context:** `kubectl config use-context k8s`  
**Scenario:** Increase the storage capacity of an existing PVC `analytics-pvc` from 10Gi to 20Gi online without taking the consumer application offline.

---

### Task 9: Troubleshooting a NotReady Worker Node (Weight: 8%)
**Context:** `kubectl config use-context k8s`  
**Scenario:** Worker node `node02` is in status `NotReady`. SSH to `node02`, inspect `journalctl -u kubelet`, repair the crashed container runtime service (`systemctl restart containerd`), restart the kubelet, and verify `kubectl get nodes` shows `Ready`.

---

### Task 10: Triage CrashLoopBackOff Pod with Ephemeral Containers (Weight: 7%)
**Context:** `kubectl config use-context k8s`  
**Scenario:** Pod `order-processor` in namespace `orders` is in `CrashLoopBackOff`. Inspect previous logs with `kubectl logs order-processor --previous`, attach an ephemeral container using `kubectl debug`, fix the environment variable in the deployment, and verify healthy status.

---

### Task 11: Pod Scheduling with NodeAffinity & Taints (Weight: 6%)
**Context:** `kubectl config use-context k8s`  
**Scenario:** Taint node `node01` with `tier=gpu:NoSchedule`. Configure pod `ml-worker` with a toleration for this taint and a `nodeAffinity` targeting nodes labeled `accelerator=nvidia`.

---

### Task 12: Ingress Controller Path-Based Routing (Weight: 6%)
**Context:** `kubectl config use-context k8s`  
**Scenario:** Configure an Ingress `web-ingress` with paths:
- `/api` -> Service `api-svc:8080`
- `/` -> Service `frontend-svc:80`
Enforce TLS using secret `tls-secret`.

---

### Task 13: CoreDNS Corefile Custom Upstream Forwarding (Weight: 5%)
**Context:** `kubectl config use-context k8s`  
**Scenario:** Edit the `coredns` ConfigMap in namespace `kube-system` to conditionally forward queries for domain `corp.internal` to DNS server `10.50.0.10:53`.

---

### Task 14: Pod PriorityClass & Preemption (Weight: 5%)
**Context:** `kubectl config use-context k8s`  
**Scenario:** Create a PriorityClass `high-priority-workload` with value `1000000`. Deploy a pod using this PriorityClass on a saturated cluster to verify it preempts lower-priority pods.

---

### Task 15: PodDisruptionBudget (PDB) Maintenance Safeguard (Weight: 5%)
**Context:** `kubectl config use-context k8s`  
**Scenario:** Create a `PodDisruptionBudget` for deployment `redis-cluster` ensuring `minAvailable: 2` during node maintenance.

---

### Task 16: Custom Resource Definitions (CRD) & Operator Status (Weight: 5%)
**Context:** `kubectl config use-context k8s`  
**Scenario:** Inspect an unhealthy Custom Resource `DatabaseCluster` in namespace `db`. Inspect operator logs, fix the missing backing Secret, and confirm status becomes `Ready`.

---

### Task 17: Imperative JSONPath Queries (Weight: 5%)
**Context:** `kubectl config use-context k8s`  
**Scenario:** Using `kubectl` and `-o jsonpath`, extract the node names and internal IP addresses of all ready nodes, formatted as `<node-name> <internal-ip>`, and write to `/opt/node-ips.txt`.

**Command:**
```bash
kubectl get nodes -o jsonpath='{range .items[*]}{.metadata.name}{" "}{.status.addresses[?(@.type=="InternalIP")].address}{"\n"}{end}' > /opt/node-ips.txt
```
