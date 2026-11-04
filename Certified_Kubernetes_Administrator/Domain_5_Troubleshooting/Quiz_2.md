# Domain 5: Troubleshooting — Quiz 2 (Worker Nodes, Kubelet, CRI & Node Conditions)

**Exam:** Certified Kubernetes Administrator (CKA)  
**Format:** 10 Scenario & Performance Questions with Detailed Explanations

---

### Question 1: Worker Node in `NotReady` State (`kubelet` Stopped)
`kubectl get nodes` reports `worker-2` as `NotReady`. You SSH into `worker-2` and run `systemctl status kubelet`, which shows `Active: inactive (dead) (Result: exit-code)` and `Unit kubelet.service is masked` or disabled. Which command sequence unmasks, enables, and starts the kubelet?
- A. `crictl start kubelet`
- B. `kubeadm reset`
- C. `systemctl unmask kubelet && systemctl enable --now kubelet`
- D. `kubectl uncordon worker-2`

---

### Question 2: Wrong `kubelet` Binary Path in Systemd Drop-In
On `worker-1`, `systemctl status kubelet` shows `Main PID: 4120 (code=exited, status=203/EXEC)`. `journalctl -u kubelet` logs `Failed to execute command: No such file or directory: /usr/local/bin/kubelet`. Where do you fix the `ExecStart` path?
- A. In `/var/lib/kubelet/config.yaml`.
- B. In `/etc/systemd/system/kubelet.service.d/10-kubeadm.conf` (or `/lib/systemd/system/kubelet.service`), point `ExecStart` to `/usr/bin/kubelet` (verified via `which kubelet`), then run `systemctl daemon-reload && systemctl restart kubelet`.
- C. In `/etc/kubernetes/kubelet.conf`.
- D. In `/etc/containerd/config.toml`.

---

### Question 3: Container Runtime (`containerd`) Socket Unreachable
`journalctl -u kubelet` on `worker-3` floods with `remote runtime service failed: rpc error: code = Unavailable desc = connection error: desc = "transport: Error while dialing dial unix /run/containerd/containerd.sock: connect: no such file or directory"`. What is the root cause and fix?
- A. The `containerd` systemd service is stopped or crashed; run `systemctl enable --now containerd` and verify `/run/containerd/containerd.sock` exists.
- B. `kube-proxy` is stopped.
- C. `etcd` is unreachable.
- D. The node has a `NoSchedule` taint.

---

### Question 4: Swap Enabled on Worker Node (`failSwapOn: true`)
After rebooting a worker node, `kubelet` refuses to start with log message: `running with swap on is not supported, please disable swap! or set --fail-swap-on=false`. Which commands disable swap immediately AND persist the change across reboots?
- A. `kubectl taint node worker-1 swap=false:NoSchedule`
- B. `swapoff -a` AND comment out or remove any `swap` entries in `/etc/fstab` (and mask any `.swap` systemd units).
- C. `sysctl -w vm.swappiness=0`
- D. `free -m`

---

### Question 5: Cgroup Driver Mismatch (`cgroupfs` vs `systemd`)
After upgrading `containerd`, pods on `worker-2` repeatedly restart and `kubelet` logs errors creating cgroup slices. You inspect `/var/lib/kubelet/config.yaml` and see `cgroupDriver: systemd`, while `/etc/containerd/config.toml` has `SystemdCgroup = false`. How do you resolve this?
- A. Change `kube-proxy` mode to `ipvs`.
- B. Reinstall the Linux kernel.
- C. Set `SystemdCgroup = true` under `[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]` in `/etc/containerd/config.toml` and run `systemctl restart containerd kubelet`.
- D. Delete `/sys/fs/cgroup`.

---

### Question 6: Certificate Rotation / Wrong API Server IP in `/etc/kubernetes/kubelet.conf`
`worker-1` is `NotReady` and `journalctl -u kubelet` shows `Unable to register node "worker-1" with API server: Post "https://10.0.1.99:6443/api/v1/nodes": dial tcp 10.0.1.99:6443: connect: connection refused`. The actual control-plane IP is `10.0.1.10`. Where do you fix the API server endpoint for the kubelet?
- A. Edit `/var/lib/kubelet/config.yaml`.
- B. Run `kubectl config set-cluster`.
- C. Edit `server: https://10.0.1.10:6443` in `/etc/kubernetes/kubelet.conf` and restart `kubelet` (`systemctl restart kubelet`).
- D. Edit `/etc/hosts`.

---

### Question 7: Diagnosing Node `MemoryPressure` & Kernel OOM Kills
A node intermittently drops to `MemoryPressure` and terminates high-memory pods with `OOMKilled` (`exitCode: 137`). Which command on the worker node inspects kernel-level OOM killer invocations and victim PIDs?
- A. `crictl images`
- B. `kubectl top nodes`
- C. `df -h`
- D. `dmesg -T | grep -i -E "oom|killed process"` (or `journalctl -k`)

---

### Question 8: Diagnosing Node `DiskPressure` from Inode Exhaustion
`kubectl describe node worker-1` reports `DiskPressure = True`, yet `df -h` shows only 42% of byte capacity used on `/`. Which command reveals the hidden cause of `DiskPressure`?
- A. `fdisk -l`
- B. `mount`
- C. `lsblk`
- D. `df -i` (checking inode utilization, as millions of tiny files can exhaust 100% of filesystem inodes (`IFree = 0`) even when gigabytes of block space remain free).

---

### Question 9: Node `PIDPressure` Condition
A misbehaving fork-bomb container spawns thousands of child threads on `worker-2`, triggering `PIDPressure` and preventing even SSH commands (`-bash: fork: retry: Resource temporarily unavailable`). Which kubelet setting caps the maximum number of processes a single Pod can spawn?
- A. `maxPods` in `/var/lib/kubelet/config.yaml`.
- B. `podPidsLimit` in `/var/lib/kubelet/config.yaml` (backed by the Linux `pids.max` cgroup controller).
- C. `cpu.cfs_quota_us`.
- D. `net.ipv4.ip_local_port_range`.

---

### Question 10: Node Stuck with `node.kubernetes.io/unschedulable:NoSchedule`
After maintenance, `kubectl get nodes` shows `worker-2` as `Ready,SchedulingDisabled`. No new pods are being placed on `worker-2`. What command restores scheduling on `worker-2`?
- A. `kubectl uncordon worker-2`
- B. `systemctl restart kubelet`
- C. `kubectl label node worker-2 schedulable=true`
- D. `kubectl drain worker-2`

---

## Answer Key & Detailed Explanations

1. **C** — When a systemd unit is masked (symlinked to `/dev/null`), `systemctl unmask kubelet` removes the mask, and `systemctl enable --now kubelet` enables it at boot and starts it immediately.
2. **B** — Exit code `203/EXEC` in systemd indicates the binary specified in `ExecStart` does not exist or is not executable. Updating the drop-in file `/etc/systemd/system/kubelet.service.d/10-kubeadm.conf` and running `systemctl daemon-reload && systemctl restart kubelet` fixes it.
3. **A** — The kubelet communicates with the container runtime via CRI over `/run/containerd/containerd.sock`. If `containerd` is stopped, the socket is unavailable.
4. **B** — `swapoff -a` disables swap in the running kernel, and commenting out the swap line in `/etc/fstab` prevents systemd from re-mounting swap on reboot.
5. **C** — Both the kubelet and containerd must use the exact same cgroup driver (`systemd` on modern systemd-based Linux distributions).
6. **C** — `/etc/kubernetes/kubelet.conf` is the kubeconfig file used by the kubelet to authenticate to and reach `kube-apiserver`.
7. **D** — The Linux kernel OOM killer logs its scoring heuristics and the terminated process PID/name to the kernel ring buffer (`dmesg -T` or `journalctl -k`).
8. **D** — The kubelet monitors both `nodefs.available` (bytes) and `nodefs.inodesFree` (inodes). `df -i` displays inode usage across mounted filesystems.
9. **B** — `podPidsLimit` configures the `pids` cgroup controller for every Pod to prevent PID exhaustion on the node.
10. **A** — `SchedulingDisabled` (`spec.unschedulable: true`) is set by `kubectl cordon` or `kubectl drain` and cleared by `kubectl uncordon <node>`.
