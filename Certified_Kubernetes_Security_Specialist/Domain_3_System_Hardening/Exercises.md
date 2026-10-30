# Domain 3: System Hardening — Production Hands-On Labs & CKS Workbook

**Domain Weight:** 15% of the CKS Exam  
**Format:** 6 Performance-Based Security Labs with Host Commands, Kernel Profiles, Manifests, and Verification Steps on your **DigitalOcean Ubuntu 24.04 LTS Droplets** (`cks-cp1`, `cks-worker1`, `cks-worker2`).

---

## Lab 1: Authoring, Loading & Enforcing Custom AppArmor Profiles on Worker Droplets

### 1.1 Scenario
On worker Droplet `cks-worker1` (`worker-01`), a custom AppArmor profile exists at `/root/usr.sbin.nginx-hardened`. You must:
1. Inspect the profile name declared inside the file.
2. Copy it into `/etc/apparmor.d/` and load it into the kernel in **enforce** mode on `cks-worker1`.
3. Update Deployment `secure-nginx` in namespace `dmz` so its containers execute confined by this AppArmor profile using the Kubernetes 1.30+ GA `securityContext.appArmorProfile` syntax.

### 1.2 Execution Steps
```bash
# Step 1: SSH to cks-worker1 (10.116.0.11) and inspect the profile header
ssh cks-worker1
cat << 'EOF' | sudo tee /etc/apparmor.d/usr.sbin.nginx-hardened
#include <tunables/global>

profile k8s-nginx-no-write flags=(attach_disconnected,mediate_deleted) {
  #include <abstractions/base>
  network inet tcp,
  network inet udp,
  capability net_bind_service,
  capability setuid,
  capability setgid,
  capability chown,

  /usr/sbin/nginx mr,
  /etc/nginx/** r,
  /usr/share/nginx/** r,
  /var/log/nginx/** rw,
  /var/run/nginx.pid rw,
  /run/nginx.pid rw,
  /var/cache/nginx/** rw,

  # Explicitly deny writes to /tmp and /var/www/html
  deny /tmp/** w,
  deny /var/www/html/** w,
  deny /etc/passwd w,
  deny /etc/shadow rw,
}
EOF

# Step 2: Parse and load the profile into the kernel in enforce mode
sudo apparmor_parser -q -r /etc/apparmor.d/usr.sbin.nginx-hardened

# Step 3: Verify the internal profile name "k8s-nginx-no-write" is loaded in enforce mode
sudo aa-status | grep -B 2 -A 5 "k8s-nginx-no-write"
exit
```

Apply the Pod/Deployment manifest targeting `cks-worker1`:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: secure-nginx
  namespace: dmz
spec:
  replicas: 1
  selector:
    matchLabels:
      app: secure-nginx
  template:
    metadata:
      labels:
        app: secure-nginx
    spec:
      nodeName: cks-worker1
      securityContext:
        appArmorProfile:
          type: Localhost
          localhostProfile: k8s-nginx-no-write
      containers:
      - name: nginx
        image: nginx:1.27-alpine
        ports:
        - containerPort: 80
```

### 1.3 Verification
```bash
# Verify container is confined by k8s-nginx-no-write and cannot write to /tmp
kubectl exec -n dmz deploy/secure-nginx -- sh -c 'cat /proc/1/attr/current'
# Expected output: k8s-nginx-no-write (enforce)

kubectl exec -n dmz deploy/secure-nginx -- touch /tmp/exploit
# Expected output: touch: /tmp/exploit: Permission denied
```

---

## Lab 2: Custom Seccomp Syscall Filtering & Auditing (`SCMP_ACT_ERRNO` & `SCMP_ACT_LOG`)

### 2.1 Scenario
You must deploy two custom Seccomp profiles into the kubelet's default seccomp root directory (`/var/lib/kubelet/seccomp/profiles/`) on `cks-worker1`:
1. `audit-all.json`: Logs all system calls made by a container without blocking them.
2. `deny-mount-chmod.json`: Allows standard syscalls by default (`SCMP_ACT_ALLOW`), but blocks `chmod`, `fchmod`, `fchmodat`, `mount`, `umount2`, `reboot`, and `swapon` with `EPERM` (`SCMP_ACT_ERRNO`).

### 2.2 Execution Steps
```bash
ssh cks-worker1
sudo mkdir -p /var/lib/kubelet/seccomp/profiles

# 1. Create audit-all.json
cat << 'EOF' | sudo tee /var/lib/kubelet/seccomp/profiles/audit-all.json
{
  "defaultAction": "SCMP_ACT_LOG"
}
EOF

# 2. Create deny-mount-chmod.json
cat << 'EOF' | sudo tee /var/lib/kubelet/seccomp/profiles/deny-mount-chmod.json
{
  "defaultAction": "SCMP_ACT_ALLOW",
  "architectures": [
    "SCMP_ARCH_X86_64",
    "SCMP_ARCH_X86",
    "SCMP_ARCH_X32"
  ],
  "syscalls": [
    {
      "names": [
        "chmod",
        "fchmod",
        "fchmodat",
        "mount",
        "umount2",
        "reboot",
        "swapon"
      ],
      "action": "SCMP_ACT_ERRNO"
    }
  ]
}
EOF
exit
```

Deploy a Pod using `profiles/deny-mount-chmod.json`:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: seccomp-hardened
  namespace: default
spec:
  nodeName: cks-worker1
  securityContext:
    seccompProfile:
      type: Localhost
      localhostProfile: profiles/deny-mount-chmod.json
  containers:
  - name: app
    image: busybox:1.36
    command: ["sh", "-c", "sleep 3600"]
    securityContext:
      allowPrivilegeEscalation: false
      capabilities:
        drop: ["ALL"]
```

### 2.3 Verification
```bash
# Confirm Seccomp status is 2 (filter mode) inside /proc/1/status
kubectl exec seccomp-hardened -- grep Seccomp /proc/1/status
# Expected: Seccomp: 2

# Confirm chmod is blocked by the kernel seccomp filter
kubectl exec seccomp-hardened -- chmod 777 /tmp
# Expected: chmod: /tmp: Operation not permitted
```

---

## Lab 3: Linux Capabilities Least-Privilege Hardening & Decoding `/proc/<pid>/status`

### 3.1 Scenario
A Pod `net-agent` in namespace `ops` is currently running with `privileged: true`. It only needs to read raw network packets (`NET_RAW`) and bind to privileged ports $<1024$ (`NET_BIND_SERVICE`). You must:
1. Inspect its current effective capabilities (`CapEff`) on the worker node.
2. Recreate the Pod with `privileged: false`, `allowPrivilegeEscalation: false`, `readOnlyRootFilesystem: true`, `capabilities.drop: ["ALL"]`, and `capabilities.add: ["NET_BIND_SERVICE", "NET_RAW"]`.

### 3.2 Execution Steps
```bash
# Step 1: Inspect current privileged container's capability bitmask on cks-worker1
CID=$(sudo crictl ps --name net-agent -q)
PID=$(sudo crictl inspect "$CID" | jq '.info.pid')
grep Cap /proc/"$PID"/status
# CapEff: 000001ffffffffff
capsh --decode=000001ffffffffff
```

Replace the Pod with the hardened specification:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: net-agent
  namespace: ops
spec:
  containers:
  - name: net-agent
    image: alpine:3.20
    command: ["sh", "-c", "sleep 3600"]
    securityContext:
      privileged: false
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      capabilities:
        drop:
          - ALL
        add:
          - NET_BIND_SERVICE
          - NET_RAW
    volumeMounts:
    - name: tmp
      mountPath: /tmp
  volumes:
  - name: tmp
    emptyDir:
      medium: Memory
      sizeLimit: 32Mi
```

### 3.3 Verification
```bash
# Check decoded capabilities of the hardened container
CID=$(sudo crictl ps --name net-agent -q)
PID=$(sudo crictl inspect "$CID" | jq '.info.pid')
CAP_EFF=$(awk '/CapEff:/ {print $2}' /proc/"$PID"/status)
capsh --decode="$CAP_EFF"
# Expected output: 0x0000000000002400=cap_net_bind_service,cap_net_raw
```

---

## Lab 4: Host OS Network Port Audit, Daemon Removal & `ufw` Firewall Lockdown on DigitalOcean Droplets

### 4.1 Scenario
On `cks-worker1` (`worker-01`):
1. An unauthorized service is listening on TCP port `2121` (FTP) or `9091`. Find the PID and systemd unit, stop and mask it, and remove its package.
2. Configure `ufw` on `cks-worker1` to default deny incoming traffic while explicitly permitting SSH (`22/tcp` on `eth0`), internal DigitalOcean VPC traffic on `eth1` (`10.116.0.0/20`), Kubelet API (`10250/tcp`), CNI VXLAN (`8472/udp`) / Cilium (`4240/tcp`) / Calico BGP (`179/tcp`), and NodePort range (`30000:32767/tcp`).

### 4.2 Execution Steps
```bash
ssh cks-worker1

# 1. Identify listening process on port 9091
sudo ss -tulpn | grep :9091
# Example output: tcp LISTEN 0 128 0.0.0.0:9091 ... users:(("transmission-da",pid=2140,fd=10))

# 2. Identify owning systemd service and package
systemctl status 2140
dpkg -S $(readlink -f /proc/2140/exe)

# 3. Stop, disable, mask, and purge package
sudo systemctl disable --now transmission-daemon
sudo systemctl mask transmission-daemon
sudo apt-get purge -y transmission-daemon

# 4. Configure UFW host firewall safely across eth0 (Public WAN) and eth1 (DigitalOcean VPC)
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 22/tcp
sudo ufw allow in on eth1 from 10.116.0.0/20
sudo ufw allow 10250/tcp
sudo ufw allow 179/tcp
sudo ufw allow 4240/tcp
sudo ufw allow 8472/udp
sudo ufw allow 30000:32767/tcp
sudo ufw --force enable
sudo ufw status numbered
```

---

## Lab 5: Kernel Module Blacklisting (`sctp`, `dccp`, `rds`) & `sysctl` Hardening

### 5.1 Scenario
Harden `cks-worker1` by unloading and permanently blacklisting the `sctp` and `rds` kernel modules, and configuring kernel sysctls so `kubelet` runs with `protectKernelDefaults: true`.

### 5.2 Execution Steps
```bash
ssh cks-worker1

# 1. Unload kernel modules if currently loaded
sudo modprobe -r sctp rds || true

# 2. Blacklist modules in /etc/modprobe.d/
cat << 'EOF' | sudo tee /etc/modprobe.d/cks-blacklist.conf
install sctp /bin/true
blacklist sctp
install rds /bin/true
blacklist rds
install dccp /bin/true
blacklist dccp
EOF

# 3. Configure required kernel sysctls for protectKernelDefaults: true and ASLR/kptr hardening
cat << 'EOF' | sudo tee /etc/sysctl.d/99-cks-hardening.conf
vm.overcommit_memory = 1
vm.panic_on_oom = 0
kernel.panic = 10
kernel.panic_on_oops = 1
kernel.keys.root_maxkeys = 1000000
kernel.keys.root_maxbytes = 25000000
kernel.kptr_restrict = 2
kernel.dmesg_restrict = 1
kernel.randomize_va_space = 2
net.ipv4.conf.all.rp_filter = 1
EOF

sudo sysctl --system

# 4. Enable protectKernelDefaults and disable readOnlyPort in /var/lib/kubelet/config.yaml
sudo yq -i '.protectKernelDefaults = true | .readOnlyPort = 0 | .seccompDefault = true' /var/lib/kubelet/config.yaml
sudo systemctl restart kubelet
```

### 5.3 Verification
```bash
lsmod | grep -E "sctp|rds"
sudo sysctl vm.overcommit_memory kernel.kptr_restrict
systemctl is-active kubelet
```

---

## Lab 6: CIS Control-Plane File Permissions & SUID Binary Remediation

### 6.1 Scenario
A CIS `kube-bench` scan on `cks-cp1` (`cp-01`) failed because `/etc/kubernetes/manifests/*.yaml` and `/etc/kubernetes/admin.conf` have world-readable permissions (`0644`), and a container image `custom-runner` contains dangerous `SUID` bits on `/bin/mount` and `/usr/bin/passwd`.

### 6.2 Execution Steps
```bash
# Part A: Fix Control-Plane File Ownership and Permissions on cks-cp1
sudo chown root:root /etc/kubernetes/manifests/*.yaml /etc/kubernetes/*.conf
sudo chmod 0600 /etc/kubernetes/manifests/*.yaml /etc/kubernetes/*.conf
sudo chown -R root:root /etc/kubernetes/pki/
sudo chmod 0600 /etc/kubernetes/pki/*.key /etc/kubernetes/pki/etcd/*.key
sudo chmod 0644 /etc/kubernetes/pki/*.crt /etc/kubernetes/pki/etcd/*.crt

# Verify with kube-bench
kube-bench run --targets master --check 1.1.1,1.1.13,1.1.19

# Part B: Strip SUID/SGID bits inside a hardened multi-stage Dockerfile
cat << 'EOF' > Dockerfile.hardened
FROM ubuntu:24.04
# Find and remove all SUID (-4000) and SGID (-2000) bits across the image filesystem
RUN find / -xdev -type f \( -perm -4000 -o -perm -2000 \) -exec chmod a-s {} + 2>/dev/null || true \
    && groupadd -g 65532 appgroup \
    && useradd -u 65532 -g 65532 -s /sbin/nologin -M appuser
USER 65532:65532
ENTRYPOINT ["/bin/echo", "Hardened No-SUID Image"]
EOF
```
