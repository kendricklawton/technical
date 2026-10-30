# Domain 1: Cluster Architecture, Installation & Configuration — Essential Documentation & Technical Reference Guide

> **Blueprint Weight:** 25% of Certified Kubernetes Administrator (CKA)  
> **Core Objective:** Design, provision, upgrade, and secure production Kubernetes clusters; implement role-based access control (RBAC), perform etcd snapshot backup/restore, manage Custom Resource Definitions (CRDs), and deploy workloads via Helm and Kustomize.

---

## 1. Official Documentation & Authoritative References

### 1.1 [Creating a Cluster with kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/)
* **Canonical Scope:** Standard procedure for bootstrapping multi-node production-grade Kubernetes control planes and worker nodes.
* **Essential Commands & Flags:**
  * `kubeadm init --pod-network-cidr=192.168.0.0/16 --control-plane-endpoint="k8s-lb:6443" --upload-certs`
  * Control Plane static pod manifests located in `/etc/kubernetes/manifests/` (`kube-apiserver.yaml`, `kube-controller-manager.yaml`, `kube-scheduler.yaml`, `etcd.yaml`).
  * Worker node join workflow: `kubeadm token create --print-join-command`.

### 1.2 [Operating etcd Clusters for Kubernetes](https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/)
* **Canonical Scope:** Disaster recovery, consistency, and maintenance of the core Kubernetes state store.
* **Backup & Restore Mechanics:**
  * **Snapshot Save:**
    ```bash
    ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379       --cacert=/etc/kubernetes/pki/etcd/ca.crt       --cert=/etc/kubernetes/pki/etcd/server.crt       --key=/etc/kubernetes/pki/etcd/server.key       snapshot save /var/lib/etcd-snapshot.db
    ```
  * **Snapshot Restore:**
    ```bash
    ETCDCTL_API=3 etcdctl snapshot restore /var/lib/etcd-snapshot.db       --data-dir=/var/lib/etcd-restored
    ```
  * **Critical Exam Step:** Update host volume path in `/etc/kubernetes/manifests/etcd.yaml` to point to the newly restored `--data-dir=/var/lib/etcd-restored` directory.

### 1.3 [Upgrading kubeadm Clusters](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/)
* **Canonical Scope:** Safely executing zero-downtime control plane and worker node version upgrades.
* **Strict Sequence:**
  1. Upgrade control plane:
     * `apt-mark unhold kubeadm && apt-get update && apt-get install -y kubeadm=1.31.x-1.1 && apt-mark hold kubeadm`
     * `kubeadm upgrade plan` -> `kubeadm upgrade apply v1.31.x`
     * Drain control plane: `kubectl drain <cp-node> --ignore-daemonsets`
     * Upgrade kubelet/kubectl: `apt-mark unhold kubelet kubectl && apt-get install -y kubelet=1.31.x-1.1 kubectl=1.31.x-1.1 && apt-mark hold kubelet kubectl`
     * `systemctl daemon-reload && systemctl restart kubelet`
     * `kubectl uncordon <cp-node>`
  2. Upgrade worker nodes:
     * Drain: `kubectl drain <worker-node> --ignore-daemonsets --delete-emptydir-data`
     * Upgrade kubeadm -> `kubeadm upgrade node` -> Upgrade kubelet/kubectl -> Restart kubelet -> Uncordon.

### 1.4 [Using RBAC Authorization](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)
* **Canonical Scope:** Role, ClusterRole, RoleBinding, and ClusterRoleBinding architecture.
* **Key Patterns:**
  * Non-namespaced resources (Nodes, PVs, StorageClasses, Namespaces, CRDs) require `ClusterRole` and `ClusterRoleBinding`.
  * Namespaced access for users or `ServiceAccounts` using `Role` and `RoleBinding`.

---

## 2. High-Frequency Exam Traps & Practical Reflexes

1. **etcdctl API Version Trap:**
   * If `ETCDCTL_API=3` is not exported, `etcdctl` defaults to v2 API on older tooling, failing silently or reporting syntax errors on `snapshot save`.
2. **Kubelet Fails After Upgrade:**
   * Node stays in `NotReady` after upgrade. Check `systemctl status kubelet` and `journalctl -u kubelet -e`. Common causes: mismatched cgroup driver (`systemd` vs `cgroupfs`), or expired kubelet client certificate in `/var/lib/kubelet/pki/`.
3. **CRDs & Custom Resources:**
   * Understanding API Group, Version, and Kind hierarchy. Deleting a CRD immediately cascades and deletes all instantiated custom resources across all namespaces.

---

## 3. Reading & Study Checklist

- [ ] Read *Creating a Cluster with kubeadm*
- [ ] Read *Operating etcd Clusters for Kubernetes*
- [ ] Practice etcd backup and restore drills 3x from memory
- [ ] Practice kubeadm control-plane and worker-node upgrade sequence
- [ ] Memorize imperative RBAC creation commands (`kubectl create role`, `kubectl create clusterrolebinding`)
