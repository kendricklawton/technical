# Domain 3: System Hardening — Quiz 1 (Host Hardening, Network Ports & Kernel Modules)

**Exam:** Certified Kubernetes Security Specialist (CKS)  
**Format:** 10 Performance Scenario & Multiple Choice Security Questions with Explanations

---

### Question 1: Identifying Listening Network Processes on a Worker Node
During a CKS host-hardening task, you are instructed to identify and stop a rogue service listening on TCP port `9099` on `worker-1`. Which command displays the exact process name and PID bound to TCP port `9099`?
- A. `ss -tulpn | grep :9099` (or `netstat -tulpn | grep :9099` / `lsof -i TCP:9099`)
- B. `ps aux | grep 9099`
- C. `kubectl get svc --all-namespaces`
- D. `ip route show`

---

### Question 2: Permanently Stopping and Disabling an Unnecessary Systemd Daemon
After identifying that PID `1842` bound to port `9099` belongs to the `snmpd.service` systemd unit, which command stops the running daemon immediately, prevents it from starting at boot, and masks it so no other unit can activate it?
- A. `systemctl disable --now snmpd && systemctl mask snmpd`
- B. `kill -9 1842`
- C. `systemctl reload snmpd`
- D. `chmod 000 /etc/init.d/snmpd`

---

### Question 3: Blocking Unauthenticated Kubelet Read-Only Port (`10255`)
A security scan flags worker node `worker-2` for exposing pod metadata over TCP port `10255` without authentication. Which setting in `/var/lib/kubelet/config.yaml` disables this unauthenticated read-only endpoint?
- A. `readOnlyPort: 0` (combined with `authentication.anonymous.enabled: false` and `authorization.mode: Webhook`)
- B. `port: 0`
- C. `enableDebuggingHandlers: false`
- D. `protectKernelDefaults: false`

---

### Question 4: Blacklisting Vulnerable Linux Kernel Modules (`dccp`, `sctp`, `rds`, `tipc`)
A CKS task requires you to unload the `sctp` kernel module from a worker node immediately and prevent the Linux kernel from ever dynamically loading `sctp` again on boot or via socket creation. What is the complete procedure?
- A. Run `modprobe -r sctp` (or `rmmod sctp`) and create `/etc/modprobe.d/blacklist-sctp.conf` containing `install sctp /bin/true` and `blacklist sctp`, then run `update-initramfs -u`.
- B. Delete `/proc/modules`.
- C. Set `sysctl -w net.sctp.enabled=0`.
- D. Add `sctp: false` to `/var/lib/kubelet/config.yaml`.

---

### Question 5: Hardening Worker Node Host Firewall with `ufw`
On an Ubuntu Kubernetes worker node (`worker-1`), you must enable `ufw` default deny incoming while ensuring SSH (`22/tcp`), Kubelet API (`10250/tcp`), NodePort services (`30000:32767/tcp`), and Calico BGP (`179/tcp`) remain reachable. Which sequence of commands configures this safely?
- A. `ufw default deny incoming && ufw default allow outgoing && ufw allow 22/tcp && ufw allow 10250/tcp && ufw allow 179/tcp && ufw allow 30000:32767/tcp && ufw --force enable`
- B. `ufw enable && ufw deny all`
- C. `iptables -P INPUT DROP`
- D. `ufw reset`

---

### Question 6: Required Control-Plane Node Inbound Ports
According to official Kubernetes documentation, which four inbound TCP ports must be open on a `kubeadm` control-plane node for cluster operation?
- A. `6443` (`kube-apiserver`), `2379-2380` (`etcd` client and peer API), `10250` (`kubelet` API), `10259` (`kube-scheduler`), and `10257` (`kube-controller-manager`)
- B. `80`, `443`, `8080`, `3306`
- C. `2375`, `2376`, `4243`, `5000`
- D. `9090`, `9093`, `3000`, `6379`

---

### Question 7: Auditing SUID and SGID Binaries on Worker Nodes
Attackers frequently exploit setuid (`SUID`) root binaries on Linux hosts or inside container images to escalate privileges. Which `find` command locates all binaries on the root filesystem (`/`) with either the `SUID` (`-4000`) or `SGID` (`-2000`) permission bit set?
- A. `find / -xdev -type f \( -perm -4000 -o -perm -2000 \) -exec ls -la {} + 2>/dev/null`
- B. `find / -name "*suid*"`
- C. `ls -l /usr/bin | grep root`
- D. `chmod -R -s /`

---

### Question 8: Kubelet `protectKernelDefaults: true` Behavior
When you set `protectKernelDefaults: true` in `/var/lib/kubelet/config.yaml` and restart `kubelet`, the kubelet immediately exits with a fatal error if certain host kernel `sysctl` parameters do not match its expectations. Which kernel `sysctl` values must be configured in `/etc/sysctl.d/99-kubernetes.conf` for `protectKernelDefaults: true` to succeed?
- A. `vm.overcommit_memory=1`, `vm.panic_on_oom=0`, `kernel.panic=10`, `kernel.panic_on_oops=1`, and `kernel.keys.root_maxkeys=1000000` (and `kernel.keys.root_maxbytes=25000000`)
- B. `net.ipv4.ip_forward=0`
- C. `vm.swappiness=100`
- D. `fs.file-max=1024`

---

### Question 9: Restricting `/etc/kubernetes/pki` and Admin Kubeconfig File Permissions
CIS Kubernetes Benchmark requires strict POSIX file permissions and ownership on `/etc/kubernetes/manifests/*.yaml`, `/etc/kubernetes/admin.conf`, and `/etc/kubernetes/pki/*.key`. What are the required permissions and ownership for `/etc/kubernetes/admin.conf` and `/etc/kubernetes/pki/ca.key`?
- A. Ownership `root:root` (`chown root:root`) and permissions `0600` (`chmod 600`, or `0644` for public `.crt` files, `0600` for `.key` and `.conf` files).
- B. Ownership `nobody:nogroup` and permissions `0777`.
- C. Permissions `0444` so all users on the host can run `kubectl`.
- D. Ownership `kubelet:docker` and permissions `0750`.

---

### Question 10: Linux Kernel ASLR and `kptr_restrict` Hardening
Which two `sysctl` settings hide kernel symbol addresses in `/proc/kallsyms` from unprivileged containers and enforce full Address Space Layout Randomization (ASLR) on a Kubernetes node?
- A. `kernel.kptr_restrict=2` and `kernel.randomize_va_space=2`
- B. `kernel.kptr_restrict=0` and `kernel.randomize_va_space=0`
- C. `kernel.dmesg_restrict=0`
- D. `kernel.unprivileged_bpf_disabled=0`

---

## Answer Key & Detailed Explanations

1. **A** — `ss -tulpn` lists TCP (`-t`) and UDP (`-u`) listening (`-l`) sockets with process/PID (`-p`) and numeric port numbers (`-n`).
2. **A** — `systemctl disable --now <unit>` stops the service and removes boot symlinks; `systemctl mask <unit>` symlinks the unit to `/dev/null` so it cannot be started manually or via socket/D-Bus activation.
3. **A** — Setting `readOnlyPort: 0` in KubeletConfiguration disables the unauthenticated HTTP `:10255` metrics/pod-list endpoint.
4. **A** — `modprobe -r <module>` unloads the active module, and `install <module> /bin/true` in `/etc/modprobe.d/*.conf` prevents both direct and dependency-triggered loading.
5. **A** — Always allow SSH (`22/tcp`) and required Kubernetes ports (`10250`, CNI ports, NodePorts) before enabling `ufw` to avoid locking yourself out or breaking cluster networking.
6. **A** — Control-plane nodes require `6443` (API server), `2379-2380` (etcd), `10250` (kubelet), `10259` (kube-scheduler), and `10257` (kube-controller-manager).
7. **A** — `-perm -4000` matches the SUID bit and `-perm -2000` matches the SGID bit; `-xdev` prevents descending into `/proc` or other virtual filesystems.
8. **A** — When `protectKernelDefaults: true` is set, the kubelet refuses to modify host sysctls and instead exits if those 6 kernel parameters do not match.
9. **A** — Private keys (`*.key`) and kubeconfig files containing client certificates (`admin.conf`, `controller-manager.conf`, `scheduler.conf`) must be `0600` owned by `root:root`.
10. **A** — `kernel.kptr_restrict=2` zeroes out kernel pointers in `/proc/` regardless of UID, and `kernel.randomize_va_space=2` enables full ASLR for stack, heap, VDSO, and shared libraries.
