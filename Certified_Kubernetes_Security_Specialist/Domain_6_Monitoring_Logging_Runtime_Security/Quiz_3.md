# Domain 6: Monitoring, Logging & Runtime Security — Quiz 3 (Immutable Containers & Live Forensics)

**Exam:** Certified Kubernetes Security Specialist (CKS)  
**Format:** 10 Performance Scenario & Multiple Choice Security Questions with Explanations

---

### Question 1: Live Container Forensics Without Alerting the Attacker (`kubectl exec` Avoided)
A security alert indicates Pod `web-frontend-7d9c` on `worker-2` has been compromised and is running an in-memory crypto-miner. Why should an incident responder **avoid** running `kubectl exec -it web-frontend-7d9c -- /bin/sh` to investigate, and what should they do instead on `worker-2`?
- A. Running `kubectl exec` spawns a new process inside the container's namespaces (which the attacker can detect) and relies on potentially trojaned binaries (`ps`, `ls`, `netstat`) inside the container; instead, SSH to `worker-2`, find the container's host PID via `crictl inspect <container-id> | jq .info.pid`, and inspect `/proc/<pid>/root`, `/proc/<pid>/exe`, `/proc/<pid>/fd`, and `/proc/<pid>/environ` directly from the trusted host OS.
- B. `kubectl exec` deletes the container's memory.
- C. `kubectl exec` restarts the Pod.
- D. `/proc` is not accessible from the worker node root user.

---

### Question 2: Inspecting Modified/Added Files in a Running Container (`OverlayFS` `upperdir`)
Containerd mounts container filesystems using Linux `overlayfs`, which merges read-only image layers (`lowerdir`) with a writable container layer (`upperdir`). How can you inspect **only** the files that were created or modified at runtime inside a compromised container from the worker node host?
- A. Run `mount | grep <container-id>` (or `crictl inspect <container-id>`) to locate the `upperdir=` path on the host filesystem (e.g., `/run/containerd/io.containerd.runtime.v2.task/k8s.io/<id>/rootfs` / `/var/lib/containerd/io.containerd.snapshotter.v1.overlayfs/snapshots/<n>/fs`) and inspect the files inside `upperdir`.
- B. Run `kubectl describe pod`.
- C. Check `/etc/fstab`.
- D. Run `crictl rmi`.

---

### Question 3: Isolating a Compromised Pod for Forensics Instead of Deleting It
An active intrusion is detected in Pod `billing-api-4f8a` belonging to Deployment `billing-api` (which selects `app: billing`). If you run `kubectl delete pod billing-api-4f8a`, all volatile memory and `/proc` evidence is destroyed. How do you quarantine `billing-api-4f8a` off the network while keeping it running for forensic memory capture AND allowing the Deployment to spin up a clean replacement Pod?
- A. Change the label on `billing-api-4f8a` (`kubectl label pod billing-api-4f8a app=quarantine --overwrite`) so the Deployment/Service detaches from it and spawns a fresh replacement Pod, and apply a strict Deny-All Ingress/Egress `NetworkPolicy` selecting `app: quarantine`.
- B. Scale the Deployment to `0`.
- C. Drain the worker node.
- D. Delete the namespace.

---

### Question 4: Capturing Container Checkpoints (`kubelet` Forensic Container Checkpointing)
Kubernetes 1.30+ supports the Forensic Container Checkpointing feature (`ContainerCheckpoint`), allowing administrators to snapshot a running container's memory and process state to disk (`checkpoint-<pod>_<ns>-<container>-<timestamp>.tar`) without terminating it. What kubelet API endpoint triggers a container checkpoint on a worker node?
- A. `curl -X POST "https://localhost:10250/checkpoint/<namespace>/<pod>/<container>" --cacert ... --cert ... --key ...`
- B. `kubectl checkpoint pod <pod>`
- C. `etcdctl snapshot save`
- D. `crictl stop <container>`

---

### Question 5: Enforcing Container Immutability at Runtime
Which four Pod/Container settings work together to make a container completely immutable at runtime so an attacker who achieves Remote Code Execution (RCE) cannot modify binaries, install packages, escalate privileges, or persist files on the root filesystem?
- A. `readOnlyRootFilesystem: true`, `runAsNonRoot: true` (`runAsUser: 65532`), `allowPrivilegeEscalation: false`, and `capabilities: { drop: ["ALL"] }` (combined with a distroless image containing no shell or package manager).
- B. `privileged: true`, `hostNetwork: true`, `imagePullPolicy: Always`.
- C. `restartPolicy: Never` and `terminationGracePeriodSeconds: 0`.
- D. `automountServiceAccountToken: true`.

---

### Question 6: Recovering a Deleted Binary That Is Still Executing in Memory (`/proc/<pid>/exe`)
An attacker compromises a container, launches a malicious binary `/tmp/backdoor`, and immediately runs `rm -f /tmp/backdoor` so the file no longer appears in `ls /tmp`. While the process (host PID `28410`) is still running in memory, how can you recover the exact ELF binary from the worker node for malware analysis?
- A. Copy `/proc/28410/exe` (`cp /proc/28410/exe /root/forensics/backdoor.elf`), because the Linux kernel keeps the inode and executable pages referenced via the `/proc/<pid>/exe` magic symlink until the process exits.
- B. Run `git checkout /tmp/backdoor`.
- C. It is impossible once `rm -f` is run.
- D. Check `/var/log/lastlog`.

---

### Question 7: Inspecting Open Network Connections of a Compromised Container from the Host (`nsenter`)
A distroless container (host PID `28410`) has no `ss`, `netstat`, or `lsof` binary inside its image. How can you run the worker node's own trusted `ss -tapn` binary **inside** the container's network namespace without entering its mount or PID namespace?
- A. `nsenter -t 28410 -n ss -tapn` (where `-n` enters only the target PID's network namespace while executing the host's trusted `/usr/bin/ss` binary).
- B. `chroot /proc/28410/root ss -tapn`
- C. `kubectl port-forward`
- D. `iptables -L`

---

### Question 8: Detecting Malicious `ConfigMap` or `Secret` Mount Tampering via Audit & Falco
A CKS scenario describes a compromised pod attempting to read `/var/run/secrets/kubernetes.io/serviceaccount/token` and connect to the Kubernetes API server (`10.96.0.1:443`). Which two controls prevent and detect this?
- A. Prevention: Set `automountServiceAccountToken: false` on the Pod/ServiceAccount; Detection: Configure a Falco rule triggering on processes other than expected SDKs opening `/var/run/secrets/kubernetes.io/serviceaccount/token` or initiating outbound connections to `10.96.0.1:443`, plus an Audit Policy logging all `system:serviceaccount:*` calls.
- B. Delete `coredns`.
- C. Change the Service type to `NodePort`.
- D. Disable `etcd`.

---

### Question 9: Auditing Sysdig / Falco Event Drops Under High Syscall Load
If an attacker floods a worker node with millions of meaningless system calls per second to blind the runtime security sensor (ring buffer overflow), which Falco config option monitors and alerts on dropped kernel events?
- A. `syscall_event_drops` settings in `/etc/falco/falco.yaml` (`actions: [log, alert]`, `threshold: 0.1`).
- B. `priority: DEBUG`
- C. `json_output: false`
- D. `buffered_outputs: true`

---

### Question 10: Eliminating Mutable Configuration Drift with GitOps & Admission Controllers
During an incident postmortem, engineers discover someone manually ran `kubectl edit deployment payment-api` in production to disable `readOnlyRootFilesystem`. How do you prevent manual out-of-band mutations in a hardened production cluster?
- A. Remove `update`/`patch` RBAC permissions on workloads from human user roles (restricting write access solely to the GitOps controller ServiceAccount like Argo CD/Flux with drift self-healing enabled) AND enforce a `ValidatingAdmissionPolicy` / `PodSecurity` `enforce=restricted` policy that rejects any Deployment missing `readOnlyRootFilesystem: true`.
- B. Ask engineers not to use `kubectl edit`.
- C. Disable `kube-controller-manager`.
- D. Set `replicas: 1`.

---

## Answer Key & Detailed Explanations

1. **A** — Host-level inspection via `/proc/<pid>/` avoids alerting the attacker and avoids executing untrusted binaries inside the compromised container.
2. **A** — In `overlayfs`, the `upperdir` directory on the host contains every file created or modified since the container started, as well as character device whiteout (`0,0`) files for deleted files.
3. **A** — Relabeling a compromised Pod detaches it from the ReplicaSet and Service selector (so it stops receiving traffic and isn't deleted on scale/rollout), while a Quarantine `NetworkPolicy` isolates it on the network.
4. **A** — Kubelet's `/checkpoint/<ns>/<pod>/<container>` API creates a CRI checkpoint archive (`tar`) of the running container's memory and state for offline forensic analysis (e.g., with `checkpointctl` or `criu`).
5. **A** — Combining `readOnlyRootFilesystem: true`, `runAsNonRoot: true`, `allowPrivilegeEscalation: false`, `capabilities.drop: ["ALL"]`, and a shell-less distroless image makes runtime persistence and post-exploitation virtually impossible.
6. **A** — `/proc/<pid>/exe` is a direct kernel link to the open executable inode in memory even after the directory entry is unlinked (`deleted`).
7. **A** — `nsenter -t <pid> -n <host-command>` switches only the network namespace (`CLONE_NEWNET`) and executes the trusted binary from the host's root filesystem.
8. **A** — Combining `automountServiceAccountToken: false` with Falco file-read detection on `/var/run/secrets/kubernetes.io/serviceaccount/token` and API Audit logging provides defense-in-depth.
9. **A** — Falco's `syscall_event_drops` configuration detects when the eBPF/kmod ring buffer drops events under load.
10. **A** — Restricting human RBAC write access + admission enforcement ensures even cluster-admins or automated tools cannot deploy a non-compliant container.
