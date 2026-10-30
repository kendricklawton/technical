# Domain 1 — Quiz 2: `kubeadm` Installation, Upgrades, HA Topologies & `etcd` Backup/Restore

Test your mastery of cluster installation, High Availability (HA) control plane design, `kubeadm` minor version upgrades, and `etcd` disaster recovery.

---

### Question 1
You are designing a High-Availability (HA) Kubernetes control plane using the **Stacked `etcd`** topology. Your organization wants to tolerate the simultaneous failure of **2 control plane nodes** without losing cluster write availability (`etcd` quorum). What is the **minimum** number of control plane nodes required?

A. 3 nodes  
B. 4 nodes  
C. 5 nodes  
D. 6 nodes

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer: C**

**Explanation:**  
`etcd` uses the Raft consensus algorithm, which requires a quorum of $\lfloor N/2 \rfloor + 1$ members.  
- With **3 nodes**, quorum is $\lfloor 3/2 \rfloor + 1 = 2$ (tolerates $3 - 2 = 1$ node failure).  
- With **4 nodes**, quorum is $\lfloor 4/2 \rfloor + 1 = 3$ (still tolerates only $4 - 3 = 1$ node failure!).  
- With **5 nodes**, quorum is $\lfloor 5/2 \rfloor + 1 = 3$ (tolerates $5 - 3 = 2$ simultaneous node failures).
</details>

---

### Question 2
A cluster is currently running Kubernetes `v1.29.4`. Management asks you to upgrade the cluster to `v1.31.2` using `kubeadm`. According to the official Kubernetes Version Skew and Upgrade Policy, what is the supported procedure?

A. Run `kubeadm upgrade apply v1.31.2` directly on the control plane node, then upgrade the worker nodes directly to `v1.31.2`.  
B. Upgrade the worker nodes to `v1.31.2` first, then upgrade the control plane node to `v1.31.2`.  
C. Perform two sequential minor version upgrades: first upgrade the control plane and worker nodes from `v1.29.x` to `v1.30.x`, and then upgrade from `v1.30.x` to `v1.31.2`.  
D. Backup `etcd`, run `kubeadm reset`, and initialize a fresh `v1.31.2` cluster.

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer: C**

**Explanation:**  
`kubeadm` and the Kubernetes Version Skew Policy **do not support skipping minor versions** during an in-place upgrade. You must upgrade one minor release at a time (`1.29` $\to$ `1.30` $\to$ `1.31`), always upgrading the Control Plane node(s) before the Worker nodes.
</details>

---

### Question 3
During a `kubeadm` cluster upgrade from `v1.31.0` to `v1.31.1`, you have already upgraded the control plane node (`cp-01`). You are now logged into a worker node (`worker-02`) after draining it and upgrading the `kubeadm` package to `1.31.1-1.1`. Which `kubeadm` command should you run on `worker-02` before upgrading the `kubelet` package?

A. `sudo kubeadm upgrade apply v1.31.1`  
B. `sudo kubeadm upgrade node`  
C. `sudo kubeadm upgrade plan`  
D. `sudo kubeadm init phase kubelet-start`

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer: B**

**Explanation:**  
- `kubeadm upgrade apply <version>` is run **only once** on the primary control plane node.
- On all **worker nodes** (and secondary control plane nodes), you run `sudo kubeadm upgrade node`, which fetches the updated `kubelet` `ConfigMap` from `kube-apiserver` and updates the local `/var/lib/kubelet/config.yaml` file.
</details>

---

### Question 4
You need to join a new worker node (`worker-03`) to an existing `kubeadm` cluster that was initialized 3 months ago. The original `kubeadm init` output was lost, and the initial bootstrap token has expired (default TTL is 24 hours). Which single command on the control plane node generates a new valid token and outputs the complete `kubeadm join` command including the `--discovery-token-ca-cert-hash`?

A. `kubeadm token generate --with-ca-hash`  
B. `kubeadm token create --print-join-command`  
C. `kubeadm init phase bootstrap-token`  
D. `kubectl create token default --duration=24h`

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer: B**

**Explanation:**  
`kubeadm token create --print-join-command` creates a new 24-hour bootstrap token in the `kube-system` namespace, calculates the SHA-256 hash of `/etc/kubernetes/pki/ca.crt`, and prints the exact ready-to-run `kubeadm join <api-endpoint> --token <token> --discovery-token-ca-cert-hash sha256:<hash>` command.
</details>

---

### Question 5
You are taking a snapshot backup of a `kubeadm`-managed stacked `etcd` instance from the control plane node. Which combination of flags is required for `ETCDCTL_API=3 etcdctl snapshot save /backup/etcd.db` to authenticate properly?

A. `--endpoints=http://127.0.0.1:2380 --cacert=/etc/kubernetes/pki/ca.crt`  
B. `--endpoints=https://127.0.0.1:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key`  
C. `--endpoints=https://127.0.0.1:6443 --kubeconfig=/etc/kubernetes/admin.conf`  
D. `--data-dir=/var/lib/etcd`

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer: B**

**Explanation:**  
- `etcd` listens for client connections over HTTPS on port `2379` (`2380` is for peer-to-peer Raft traffic).
- `etcd` uses its own dedicated CA (`/etc/kubernetes/pki/etcd/ca.crt`) and accepts client connections authenticated with `/etc/kubernetes/pki/etcd/server.crt` (or `healthcheck-client.crt`) and its corresponding private key.
</details>

---

### Question 6
You are restoring an `etcd` snapshot `/backup/snapshot.db` on a single-node `kubeadm` control plane. After running:
```bash
ETCDCTL_API=3 sudo etcdctl snapshot restore /backup/snapshot.db --data-dir=/var/lib/etcd-from-backup
```
What must you do next so the cluster uses the restored data?

A. Run `systemctl restart etcd`.  
B. Copy `/var/lib/etcd-from-backup/*` into `/etc/kubernetes/pki/etcd/`.  
C. Edit `/etc/kubernetes/manifests/etcd.yaml` and update the `etcd-data` volume's `hostPath.path` from `/var/lib/etcd` to `/var/lib/etcd-from-backup`.  
D. Run `kubeadm init phase etcd local`.

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer: C**

**Explanation:**  
On a `kubeadm` cluster, `etcd` runs as a Static Pod defined in `/etc/kubernetes/manifests/etcd.yaml`. Internally, the `etcd` container mounts the `etcd-data` volume at `/var/lib/etcd` inside the container, mapped from `volumes[].hostPath.path` on the host. Updating `volumes[].hostPath.path` to `/var/lib/etcd-from-backup` causes `kubelet` to automatically recreate the `etcd` Static Pod mounting the newly restored data directory.
</details>

---

### Question 7
Why do we run `sudo apt-mark hold kubeadm kubelet kubectl` after installing or upgrading Kubernetes packages on Debian/Ubuntu nodes?

A. To prevent `kubelet` from restarting automatically when a node reboots.  
B. To lock the container images in `containerd` so garbage collection does not delete them.  
C. To prevent unattended OS `apt-get upgrade` operations from accidentally upgrading Kubernetes binaries out of order or violating version skew policies.  
D. To pin the Linux kernel version.

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer: C**

**Explanation:**  
Kubernetes requires a strict, orchestrated upgrade sequence (`kubeadm` $\to$ `kubeadm upgrade apply/node` $\to$ `kubelet` & `kubectl`). If a standard `apt-get upgrade` upgraded `kubelet` on a worker node to a version newer than `kube-apiserver`, it would violate the Kubernetes Version Skew Policy and risk breaking the cluster.
</details>

---

### Question 8
When draining a node for maintenance or a `kubeadm` upgrade using `kubectl drain worker-01`, the command fails with two errors:
1. `cannot delete DaemonSet-managed Pods (use --ignore-daemonsets to ignore)`
2. `cannot delete Pods with local storage (use --delete-emptydir-data to override)`

Why does `kubectl drain` require `--ignore-daemonsets` on almost every Kubernetes node?

A. DaemonSet Pods store `etcd` data on worker nodes.  
B. If `kubectl drain` evicted a DaemonSet Pod (like `kube-proxy` or `calico-node`), the DaemonSet controller would immediately recreate a replacement Pod on the same node (since DaemonSets tolerate `node.kubernetes.io/unschedulable`), causing an infinite drain loop.  
C. DaemonSets cannot be terminated by `kubelet`.  
D. `--ignore-daemonsets` migrates DaemonSet Pods to the control plane node.

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer: B**

**Explanation:**  
DaemonSet Pods are designed to run on every node (including cordoned/`unschedulable` nodes) to provide node-critical services like CNI networking (`calico-node`, `cilium`) and `kube-proxy`. Therefore, `kubectl drain` must ignore DaemonSet pods via `--ignore-daemonsets`.
</details>

---

### Question 9
Compare the **Stacked `etcd`** HA topology with the **External `etcd`** HA topology. Which statement is TRUE?

A. Stacked `etcd` requires a minimum of 6 hosts (3 control plane + 3 etcd), whereas External `etcd` requires only 3 hosts.  
B. External `etcd` decouples the control plane and datastore failure domains so losing a control plane instance does not impact `etcd` quorum, but it requires twice as many hosts (minimum 6 hosts: 3 control plane + 3 dedicated `etcd` hosts).  
C. `kubeadm` does not support External `etcd` clusters.  
D. Stacked `etcd` stores data in memory only.

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer: B**

**Explanation:**  
- **Stacked `etcd`:** Minimum **3 hosts** (each runs `kube-apiserver`, `kube-controller-manager`, `kube-scheduler`, and `etcd`). Simpler to manage, but a single host failure impacts both compute control plane capacity and an `etcd` quorum member.
- **External `etcd`:** Minimum **6 hosts** (3 control plane nodes + 3 dedicated `etcd` nodes). Provides higher fault isolation because control plane failures and `etcd` member failures are completely decoupled.
</details>

---

### Question 10
During the CKA exam, a task instructs you: *"Upgrade the control plane node `cluster1-cp1` from `v1.31.0` to `v1.31.2`, but **do NOT upgrade `etcd`**."* Which command satisfies this requirement?

A. `sudo kubeadm upgrade apply v1.31.2 --etcd-upgrade=false`  
B. `sudo kubeadm upgrade node --skip-etcd`  
C. Move `/etc/kubernetes/manifests/etcd.yaml` to `/tmp` before running `kubeadm upgrade apply v1.31.2`.  
D. `sudo kubeadm upgrade apply v1.31.2 --ignore-preflight-errors=etcd`

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer: A**

**Explanation:**  
Both `kubeadm upgrade apply` and `kubeadm upgrade node` accept the flag `--etcd-upgrade=false` (default is `true`), which upgrades the Kubernetes control plane components (`kube-apiserver`, `kube-controller-manager`, `kube-scheduler`) while leaving the `etcd` Static Pod at its current version.
</details>

---

### Question 11
Before running `kubeadm init` on a newly provisioned Ubuntu Linux server, you verify that `br_netfilter` is loaded, `swap` is disabled, and `containerd` is running. However, `kubeadm init` preflight checks fail with:
`[ERROR FileContent--proc-sys-net-ipv4-ip_forward]: /proc/sys/net/ipv4/ip_forward contents are not set to 1`
How do you fix this so that it persists across node reboots?

A. Run `iptables -P FORWARD ACCEPT`  
B. Set `net.ipv4.ip_forward = 1` in `/etc/sysctl.d/k8s.conf` and run `sudo sysctl --system`  
C. Add `--ip-forward=true` to `/var/lib/kubelet/config.yaml`  
D. Restart `systemd-networkd`

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer: B**

**Explanation:**  
Linux kernel IPv4 packet forwarding (`/proc/sys/net/ipv4/ip_forward = 1`) is required for Pod-to-Pod and Service routing. Writing `net.ipv4.ip_forward = 1` to `/etc/sysctl.d/k8s.conf` and running `sudo sysctl --system` applies it immediately and persists it across reboots (unlike `echo 1 > /proc/sys/net/ipv4/ip_forward`, which is lost on reboot).
</details>

---

### Question 12
You suspect that the TLS certificates on an older `kubeadm` control plane node are close to expiring or have already expired. Which two `kubeadm` commands allow you to inspect the expiration dates of all PKI certificates in `/etc/kubernetes/pki/` and renew them all at once?

A. `kubeadm certs list` and `kubeadm certs regenerate`  
B. `kubeadm certs check-expiration` and `kubeadm certs renew all`  
C. `kubeadm alpha certs check` and `kubeadm init phase certs all`  
D. `kubectl get csr` and `kubectl certificate approve --all`

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer: B**

**Explanation:**  
- `sudo kubeadm certs check-expiration` displays the residual validity time of all control plane certificates and CAs in `/etc/kubernetes/pki/`.
- `sudo kubeadm certs renew all` renews all control plane certificates signed by the existing CA for another 1 year (after which you should restart the control plane Static Pods so they reload the new certs).
</details>
