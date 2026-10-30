# Domain 3: System Hardening — Quiz 2 (Linux Capabilities, SecurityContext & Privilege Escalation)

**Exam:** Certified Kubernetes Security Specialist (CKS)  
**Format:** 10 Performance Scenario & Multiple Choice Security Questions with Explanations

---

### Question 1: Dropping All Default Linux Capabilities in a Container
By default, the OCI container runtime (`runc` / `containerd`) grants containers a default set of 14 Linux capabilities (including `CAP_NET_RAW`, `CAP_CHOWN`, `CAP_SETUID`, `CAP_SETGID`). An NGINX container only needs to bind to port 80 (`CAP_NET_BIND_SERVICE`). What is the best-practice `securityContext.capabilities` configuration?
- A. `drop: ["ALL"]` and `add: ["NET_BIND_SERVICE"]`
- B. `drop: ["CAP_SYS_ADMIN"]`
- C. `privileged: true`
- D. `add: ["ALL"]`

---

### Question 2: Note on `CAP_` Prefix in Kubernetes YAML Manifests
When specifying Linux capabilities under `containers[].securityContext.capabilities.add` or `drop` in a Kubernetes Pod manifest, how must the capability name be formatted?
- A. Without the `CAP_` prefix (e.g., `NET_ADMIN`, `SYS_TIME`, `NET_RAW`, `CHOWN`).
- B. With the lowercase `cap_` prefix (`cap_net_admin`).
- C. Using numeric bitmask IDs only.
- D. With the `LINUX_` prefix.

---

### Question 3: Preventing `setuid` Privilege Escalation (`allowPrivilegeEscalation: false`)
What kernel flag does `securityContext.allowPrivilegeEscalation: false` enable on the container process, and what threat does it mitigate?
- A. It sets the Linux kernel `PR_SET_NO_NEW_PRIVS` bit (`no_new_privs`) on the process, preventing child processes from gaining more privileges than their parent via `setuid`/`setgid` binaries or file capabilities.
- B. It disables network access inside the container.
- C. It mounts `/` as read-only.
- D. It disables `kubectl exec`.

---

### Question 4: Interaction Between `privileged: true` and `allowPrivilegeEscalation`
An engineer sets `privileged: true` and `allowPrivilegeEscalation: false` in the same container `securityContext`. What happens when the Pod is submitted to `kube-apiserver`?
- A. The API server rejects the Pod with validation error: `Invalid value: false: cannot set allowPrivilegeEscalation to false and privileged to true` because `privileged: true` grants all capabilities and forces `allowPrivilegeEscalation: true`.
- B. `allowPrivilegeEscalation: false` overrides `privileged: true`.
- C. Both settings are ignored.
- D. The container runs as UID 65534.

---

### Question 5: Inspecting Capabilities of a Running Process via `/proc/<pid>/status`
You SSH into a worker node and find a suspicious container process with host PID `19450`. You inspect `grep Cap /proc/19450/status` and see `CapEff: 000001ffffffffff`. How do you decode that hexadecimal bitmask into human-readable Linux capability names on the node?
- A. `capsh --decode=000001ffffffffff` (or `getpcaps 19450`)
- B. `base64 -d`
- C. `aa-status`
- D. `crictl inspectp`

---

### Question 6: Danger of `CAP_SYS_ADMIN` vs `privileged: true`
Why is granting `CAP_SYS_ADMIN` to a container almost equivalent to running the container with `privileged: true`?
- A. `CAP_SYS_ADMIN` permits mounting filesystems (`mount`), configuring kernel namespaces (`setns`, `unshare`), loading BPF programs, and manipulating block devices, enabling trivial container escape to the host OS.
- B. `CAP_SYS_ADMIN` only allows changing the system clock.
- C. `CAP_SYS_ADMIN` only allows pinging external hosts.
- D. `CAP_SYS_ADMIN` is included in the default OCI capability set.

---

### Question 7: Mitigating ICMP / ARP Spoofing Inside the Cluster (`CAP_NET_RAW`)
Why does the CIS Kubernetes Benchmark recommend dropping `NET_RAW` from all untrusted application containers?
- A. `CAP_NET_RAW` allows a process to open raw and packet sockets (`AF_PACKET`, `SOCK_RAW`), which an attacker inside a compromised container can use to craft spoofed ARP replies, ICMP redirects, or port-scan the pod network.
- B. `CAP_NET_RAW` is required for standard TCP and HTTP connections.
- C. `CAP_NET_RAW` allows mounting host disks.
- D. `CAP_NET_RAW` modifies `iptables` rules (which actually requires `CAP_NET_ADMIN`).

---

### Question 8: Enforcing `readOnlyRootFilesystem: true` with Application Temp Directories
You harden a Python FastAPI container by setting `securityContext.readOnlyRootFilesystem: true`. On startup, the container crashes because Gunicorn attempts to write temporary worker heartbeat files to `/tmp` and `/var/run`. How do you fix the Pod without setting `readOnlyRootFilesystem: false`?
- A. Mount `emptyDir: {}` volumes (optionally with `medium: Memory` and `sizeLimit: 64Mi`) at `/tmp` and `/var/run` while keeping `readOnlyRootFilesystem: true`.
- B. Mount the host's `/tmp` via `hostPath`.
- C. Grant `CAP_DAC_OVERRIDE`.
- D. Run the container as `runAsUser: 0`.

---

### Question 9: Pod-Level vs Container-Level `securityContext` Precedence
A Pod defines `spec.securityContext.runAsUser: 1000`, while container `sidecar` inside that Pod defines `spec.containers[1].securityContext.runAsUser: 2000`. Under which UID does `sidecar` execute, and where must `readOnlyRootFilesystem` and `capabilities` be defined?
- A. `sidecar` executes as UID `2000` (container-level overrides pod-level); furthermore, `readOnlyRootFilesystem`, `capabilities`, `privileged`, and `allowPrivilegeEscalation` are valid **only** at the container-level `securityContext` (not pod-level).
- B. `sidecar` executes as UID `1000`; `capabilities` can be set at the pod level.
- C. The API server rejects the Pod due to conflicting UIDs.
- D. `sidecar` executes as UID `0`.

---

### Question 10: Kubernetes User Namespaces (`hostUsers: false`)
In Kubernetes 1.30+ (beta/GA feature), what security guarantee does setting `spec.hostUsers: false` on a Pod provide?
- A. It isolates the Pod in a Linux User Namespace (`CLONE_NEWUSER`), mapping container UID `0` (root inside the container) to an unprivileged high UID (e.g., `165536`) on the worker node host OS, neutralizing host privilege escalation even if a container breaks out.
- B. It deletes `/etc/passwd` inside the container.
- C. It disables ServiceAccount token mounting.
- D. It blocks SSH access to the worker node.

---

## Answer Key & Detailed Explanations

1. **A** — Dropping `ALL` capabilities first and explicitly adding back only the minimal capability (`NET_BIND_SERVICE`) enforces strict least privilege.
2. **A** — Kubernetes capability lists omit the `CAP_` prefix (`NET_ADMIN`, `SYS_PTRACE`, `NET_BIND_SERVICE`).
3. **A** — `allowPrivilegeEscalation: false` sets `no_new_privs` in the kernel task struct, blocking `setuid` root binaries from elevating privileges.
4. **A** — Kubernetes validation forbids combining `privileged: true` with `allowPrivilegeEscalation: false`.
5. **A** — `capsh --decode=<hex>` translates a hexadecimal capability bitmap from `/proc/<pid>/status` into capability names (`000001ffffffffff` represents all capabilities, i.e., a privileged process).
6. **A** — `CAP_SYS_ADMIN` is the catch-all Linux kernel admin capability and offers numerous well-documented container escape vectors.
7. **A** — `CAP_NET_RAW` is granted by default by Docker/containerd purely so `ping` works, but is unnecessary for TCP/UDP applications and enables raw packet spoofing.
8. **A** — Mounting `emptyDir` volumes at specific writable directories (`/tmp`, `/var/cache/nginx`, `/var/run`) preserves an immutable container root filesystem.
9. **A** — Container-level `securityContext` fields take precedence over pod-level fields, and fields like `capabilities`, `readOnlyRootFilesystem`, `privileged`, and `allowPrivilegeEscalation` only exist on `containers[].securityContext`.
10. **A** — `hostUsers: false` activates Linux User Namespaces so root inside the container has zero privileges on the host kernel.
