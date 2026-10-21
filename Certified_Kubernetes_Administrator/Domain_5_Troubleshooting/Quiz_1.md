# Domain 5: Troubleshooting — Quiz 1 (Control Plane, API Server & etcd Failure Triage)

**Exam:** Certified Kubernetes Administrator (CKA)  
**Format:** 10 Scenario & Performance Questions with Detailed Explanations

---

### Question 1: Diagnosing `The connection to the server <ip>:6443 was refused`
You run `kubectl get nodes` and receive `The connection to the server 10.0.1.10:6443 was refused - did you specify the right host or port?`. Because `kube-apiserver` is down, `kubectl logs` cannot be used. Which command on the control-plane node inspects the crashed `kube-apiserver` container logs?
- A. `docker ps`
- B. `crictl ps -a | grep kube-apiserver` followed by `crictl logs <container-id>` (or inspecting `/var/log/pods/kube-system_kube-apiserver-*`)
- C. `kubectl describe pod kube-apiserver`
- D. `systemctl status kube-apiserver`

---

### Question 2: Static Pod Manifest Directory Mismatch
After editing `/etc/kubernetes/manifests/kube-scheduler.yaml`, you notice `kube-scheduler` is still not starting, and `crictl ps -a` shows no scheduler container attempts at all. You inspect `/var/lib/kubelet/config.yaml` and find `staticPodPath: /etc/kubernetes/static-manifests`. How do you fix this?
- A. Restart `containerd`.
- B. Update `staticPodPath: /etc/kubernetes/manifests` in `/var/lib/kubelet/config.yaml` and run `systemctl restart kubelet`.
- C. Run `kubeadm init` again.
- D. Delete `/etc/kubernetes/manifests`.

---

### Question 3: `kube-apiserver` Failing to Connect to `etcd` (Port / Cert Typo)
In `/var/log/pods/kube-system_kube-apiserver-*/kube-apiserver/*.log`, you see: `grpclog: addrConn.createTransport failed to connect to {Addr: "127.0.0.1:2380" ... tls: bad certificate}`. Why is `kube-apiserver` failing to connect to `etcd`?
- A. `etcd` does not use TLS.
- B. Port `2380` is `etcd`'s peer-to-peer cluster port (which uses peer certs), whereas `kube-apiserver` must connect to `etcd`'s client port `127.0.0.1:2379` (`--etcd-servers=https://127.0.0.1:2379`).
- C. `kube-apiserver` should connect over HTTP.
- D. `kube-proxy` is down.

---

### Question 4: Deployments Not Scaling Pods (`kube-controller-manager` Down)
You scale a Deployment (`kubectl scale deploy web --replicas=5`), and `kubectl get deploy web` shows `0/5` available, yet `kubectl get pods` shows zero ReplicaSet pods created. Scheduling of manually created pods (`kubectl run test --image=nginx`) works normally. Which control-plane component is failing?
- A. `kube-scheduler`
- B. `kube-controller-manager` (specifically the Deployment and ReplicaSet controller loops)
- C. `coredns`
- D. `kube-proxy`

---

### Question 5: All New Pods Stuck in `Pending` with Zero Events
Every newly created Pod across all namespaces remains in `Pending`, and `kubectl describe pod <name>` shows `<none>` under `Events` (no `Scheduled` or `FailedScheduling` events). Existing running pods continue serving traffic. What is the most likely root cause?
- A. `kube-scheduler` static pod is crashed, misconfigured, or pointing to an invalid `--kubeconfig=/etc/kubernetes/scheduler.conf`.
- B. `etcd` is corrupted.
- C. All worker nodes are out of disk space.
- D. The CNI plugin was deleted.

---

### Question 6: Expired Control-Plane Certificates (`kubeadm certs`)
`kubectl` commands fail with `Unable to connect to the server: x509: certificate has expired or is not yet valid`. Which pair of commands verifies certificate expiration dates across all kubeadm control-plane components and renews them?
- A. `kubeadm certs check-expiration` followed by `kubeadm certs renew all` and restarting the control-plane static pods.
- B. `openssl verify` followed by `kubeadm reset`.
- C. `kubectl certificate approve --all`.
- D. `systemctl restart ca-certificates`.

---

### Question 7: `etcd` Space Quota Exceeded (`mvcc: database space exceeded`)
All write operations (`kubectl apply`, `kubectl create`) fail with `rpc error: code = ResourceExhausted desc = etcdserver: mvcc: database space exceeded`. Which sequence of `etcdctl` operations resolves this alarm?
- A. Delete `/var/lib/etcd/member`.
- B. Get the current revision (`rev`), compact historical revisions (`etcdctl compact $rev`), defragment storage (`etcdctl defrag`), and disarm the NOSPACE alarm (`etcdctl alarm disarm`).
- C. Restart `kube-controller-manager`.
- D. Increase `maxPods` in kubelet config.

---

### Question 8: Corrupt YAML Indentation in Static Pod Manifest
You modified `/etc/kubernetes/manifests/kube-apiserver.yaml` to add an audit log flag, and `kube-apiserver` vanished. `crictl ps -a` shows no new `kube-apiserver` container was even created after the old one stopped. Where do you find the YAML parsing error?
- A. `journalctl -u kubelet -n 100 --no-pager` (since the `kubelet` parses `/etc/kubernetes/manifests/*.yaml` and logs syntax/unmarshaling errors directly to the systemd journal).
- B. `/var/log/apiserver-syntax.log`
- C. `kubectl get events`
- D. `dmesg`

---

### Question 9: Incorrect ServiceAccount Signing Key Mismatch
`kube-controller-manager` is running, and ReplicaSets create Pods, but ServiceAccount tokens are not being mounted or fail validation in `kube-apiserver` with `crypto/rsa: verification error`. Which flag pair between `kube-apiserver` and `kube-controller-manager` is mismatched?
- A. `--service-account-key-file` / `--service-account-signing-key-file` on `kube-apiserver` vs `--service-account-private-key-file` on `kube-controller-manager` (typically `/etc/kubernetes/pki/sa.pub` and `/etc/kubernetes/pki/sa.key`).
- B. `--tls-cert-file` and `--tls-private-key-file`.
- C. `--etcd-cafile` and `--etcd-certfile`.
- D. `--client-ca-file`.

---

### Question 10: Restoring `etcd` Snapshot Ownership Trap
After running `etcdutl snapshot restore /backup/snap.db --data-dir=/var/lib/etcd-restored` as `root`, you update `/etc/kubernetes/manifests/etcd.yaml` to mount `/var/lib/etcd-restored`. If the `etcd` pod runs under a specific user/group or checks directory permissions (`0700`), what should you verify on `/var/lib/etcd-restored` if `etcd` fails to open the BoltDB file?
- A. Verify directory permissions (`chmod 700 /var/lib/etcd-restored`) and ownership (`chown -R etcd:etcd` if running as non-root user) and check `crictl logs` for permission or lock errors.
- B. Format `/var/lib/etcd-restored` as `nfs`.
- C. Delete `snap.db`.
- D. Set `hostNetwork: false` on `etcd`.

---

## Answer Key & Detailed Explanations

1. **B** — When `kube-apiserver` is unreachable, use `crictl ps -a` and `crictl logs <id>` or read `/var/log/pods/kube-system_kube-apiserver-*/kube-apiserver/*.log` directly on the control-plane host.
2. **B** — The `kubelet` watches the directory configured in `staticPodPath` inside `/var/lib/kubelet/config.yaml`. Fixing that path and restarting `kubelet` loads all control-plane static pods.
3. **B** — `etcd` listens on `2379` for client connections (`kube-apiserver`) and `2380` for peer-to-peer Raft traffic.
4. **B** — `kube-controller-manager` runs the Deployment and ReplicaSet controllers responsible for reconciling `spec.replicas` into Pod objects.
5. **A** — When `kube-scheduler` is down, Pods remain in `Pending` with no `spec.nodeName` assigned and zero scheduling events.
6. **A** — `kubeadm certs check-expiration` audits all PKI certificates in `/etc/kubernetes/pki`, and `kubeadm certs renew all` re-signs them using the cluster CA.
7. **B** — When `etcd` hits its storage quota, it enters a maintenance alarm mode requiring compaction, defragmentation, and `etcdctl alarm disarm`.
8. **A** — If a static pod YAML has invalid syntax, `kubelet` cannot parse it into a Pod spec and logs the unmarshal error in `journalctl -u kubelet`.
9. **A** — `kube-controller-manager` signs tokens with `sa.key`, and `kube-apiserver` verifies them with `sa.pub` (and signs projected tokens with `sa.key`).
10. **A** — `etcd` requires `0700` permissions on its data directory and write access for the UID under which the `etcd` process executes.
