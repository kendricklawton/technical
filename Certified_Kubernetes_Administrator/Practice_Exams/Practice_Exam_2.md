# CKA Mock Practice Exam 2

**Duration:** 120 Minutes  
**Passing Score:** 66%  
**Format:** 17 Hands-on Performance Tasks  

---

### Task 1: Cluster Upgrade with kubeadm (Weight: 8%)
**Context:** `kubectl config use-context cluster1`  
Upgrade the control plane node `cp-node` from version `1.30.0` to `1.30.1`.
Drain the node, upgrade kubeadm, apply the upgrade plan, upgrade kubelet and kubectl, and uncordon the node.

#### Solution:
```bash
kubectl drain cp-node --ignore-daemonsets
sudo apt-mark unhold kubeadm && sudo apt-get update && sudo apt-get install -y kubeadm=1.30.1-1.1
sudo kubeadm upgrade plan
sudo kubeadm upgrade apply v1.30.1 -y
sudo apt-mark unhold kubelet kubectl && sudo apt-get install -y kubelet=1.30.1-1.1 kubectl=1.30.1-1.1
sudo apt-mark hold kubeadm kubelet kubectl
sudo systemctl daemon-reload && sudo systemctl restart kubelet
kubectl uncordon cp-node
```

---

### Task 2: etcd Backup and Restore (Weight: 7%)
**Context:** `kubectl config use-context cluster1`  
Take a snapshot of the etcd database located on control plane at `https://127.0.0.1:2379` using certificates at `/etc/kubernetes/pki/etcd/` and save to `/opt/backup/etcd-snapshot.db`.

#### Solution:
```bash
ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379   --cacert=/etc/kubernetes/pki/etcd/ca.crt   --cert=/etc/kubernetes/pki/etcd/server.crt   --key=/etc/kubernetes/pki/etcd/server.key   snapshot save /opt/backup/etcd-snapshot.db
```

---

### Task 3: Multi-Container Pod with Sidecar Log Agent (Weight: 5%)
Create a pod `app-logger` in namespace `logging` with an app container writing to `/var/log/app.log` and a sidecar container streaming it to stdout.
