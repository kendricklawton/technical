# Domain 4: Minimize Microservice Vulnerabilities — Quiz 1 (Pod Security Standards & Admission)

**Exam:** Certified Kubernetes Security Specialist (CKS)  
**Format:** 10 Performance Scenario & Multiple Choice Security Questions with Explanations

---

### Question 1: Pod Security Standards (PSS) Three Profile Levels
Kubernetes defines three cumulative Pod Security Standard (PSS) levels. Which ordering correctly ranks them from least restrictive to most restrictive?
- A. `privileged` (unrestricted) $\to$ `baseline` (blocks known privilege escalations like `hostNetwork`, `privileged`, `hostPath`) $\to$ `restricted` (enforces pod hardening best practices like `runAsNonRoot: true`, `allowPrivilegeEscalation: false`, `seccompProfile: RuntimeDefault/Localhost`, `capabilities.drop: ["ALL"]`)
- B. `baseline` $\to$ `privileged` $\to$ `restricted`
- C. `unconfined` $\to$ `default` $\to$ `strict`
- D. `audit` $\to$ `warn` $\to$ `enforce`

---

### Question 2: Pod Security Admission (PSA) Namespace Labels (`enforce`, `audit`, `warn`)
You need to configure namespace `payments` so that any Pod violating the `restricted` profile is immediately rejected by `kube-apiserver`, while also logging a warning for `latest` version rules. Which label command configures this?
- A. `kubectl label namespace payments pod-security.kubernetes.io/enforce=restricted pod-security.kubernetes.io/enforce-version=latest pod-security.kubernetes.io/warn=restricted`
- B. `kubectl annotate namespace payments pss.k8s.io/level=restricted`
- C. `kubectl label namespace payments security=restricted`
- D. Create a `PodSecurityPolicy` in `payments`.

---

### Question 3: PSA Behavior on Existing `Deployment` When `enforce=restricted` Is Applied
A namespace `legacy` already contains a running `Deployment` whose Pod template lacks `runAsNonRoot: true`. You apply `pod-security.kubernetes.io/enforce=restricted` to namespace `legacy`. What happens immediately to the running Pods, and what happens if a Pod is deleted and the ReplicaSet tries to recreate it?
- A. Existing running Pods are **not** evicted, and `kubectl label ns` prints a warning about existing violating workloads; however, when a Pod is deleted, the ReplicaSet's attempt to create the replacement Pod is **blocked** (`403 Forbidden: violates PodSecurity "restricted:latest"`).
- B. All existing Pods in `legacy` are immediately terminated.
- C. The label command fails until all Deployments are deleted.
- D. The ReplicaSet bypasses PSA because it uses a system ServiceAccount.

---

### Question 4: Capabilities Permitted Under PSS `restricted` Profile
Under the `restricted` Pod Security Standard, a container MUST drop `ALL` capabilities (`securityContext.capabilities.drop: ["ALL"]`). Which single capability is the **only** capability permitted in `capabilities.add` under the `restricted` profile?
- A. `NET_BIND_SERVICE`
- B. `NET_RAW`
- C. `SYS_CHROOT`
- D. `CHOWN`

---

### Question 5: Volume Types Permitted Under PSS `restricted` Profile
Which of the following volume types is **prohibited** under the `restricted` (and `baseline`) Pod Security Standard?
- A. `hostPath` (only safe ephemeral/persistent volume types are permitted in `restricted`: `configMap`, `csi`, `downwardAPI`, `emptyDir`, `ephemeral`, `persistentVolumeClaim`, `projected`, `secret`).
- B. `persistentVolumeClaim`
- C. `emptyDir`
- D. `projected`

---

### Question 6: Cluster-Wide Default Pod Security Admission Configuration (`AdmissionConfiguration`)
Instead of labeling every namespace manually, an administrator wants `kube-apiserver` to apply `baseline` enforcement by default across all namespaces except `kube-system`. How is this configured on the control plane?
- A. Pass `--admission-control-config-file=/etc/kubernetes/psa/admission-config.yaml` to `kube-apiserver` containing an `AdmissionConfiguration` with plugin `PodSecurity` (`pod-security.admission.config.k8s.io/v1`) specifying `defaults: { enforce: "baseline" }` and `exemptions: { namespaces: ["kube-system"] }`.
- B. Edit `/var/lib/kubelet/config.yaml`.
- C. Add an annotation to the `default` ServiceAccount.
- D. Edit `CoreDNS`.

---

### Question 7: Mandatory Fields for a Pod to Pass `restricted` PSS Validation
A developer submits the following Pod to a `restricted` namespace, and it is rejected:
```yaml
spec:
  securityContext:
    runAsNonRoot: true
    seccompProfile:
      type: RuntimeDefault
  containers:
  - name: web
    image: nginx:1.27
```
Which two container-level `securityContext` fields are still missing to satisfy `restricted:latest`?
- A. `allowPrivilegeEscalation: false` and `capabilities: { drop: ["ALL"] }`
- B. `readOnlyRootFilesystem: true` and `runAsUser: 65534`
- C. `privileged: false` and `hostNetwork: false`
- D. `fsGroup: 1000` and `runAsGroup: 1000`

---

### Question 8: Isolating Host Namespaces (`hostNetwork`, `hostPID`, `hostIPC`)
What risk does setting `hostPID: true` and `hostIPC: true` on a Pod introduce, and which PSS profiles forbid them?
- A. `hostPID: true` allows the container to see all processes running on the worker node in `/proc` (including environment variables and command-line arguments of other containers/daemons), and `hostIPC: true` allows reading/writing shared memory segments on the host; both `baseline` and `restricted` forbid them.
- B. Only `restricted` forbids them; `baseline` allows `hostPID: true`.
- C. They only affect network latency.
- D. They are required for `emptyDir` volumes.

---

### Question 9: `procMount` Restriction in PSS
In container security, what does `/proc` masking protect against, and what value must `securityContext.procMount` have in `baseline` and `restricted` pods?
- A. By default (`procMount: Default`), the container runtime masks sensitive `/proc` paths (such as `/proc/kcore`, `/proc/keys`, `/proc/timer_list`, `/proc/acpi`) with `/dev/null` or read-only tmpfs mounts to prevent kernel information leaks and writes; `Unmasked` is forbidden unless the pod is `privileged` (or inside a User Namespace in 1.30+).
- B. `procMount: Unmasked` is the default.
- C. `/proc` is not mounted inside containers.
- D. `procMount` controls `cgroup` CPU limits.

---

### Question 10: Pre-Testing Namespace PSA Enforcement with `--dry-run=server`
Before changing a production namespace from `warn=restricted` to `enforce=restricted`, how can you test whether all existing workloads in namespace `prod` would pass `enforce=restricted` without actually modifying the namespace in etcd?
- A. `kubectl label --dry-run=server --overwrite ns prod pod-security.kubernetes.io/enforce=restricted` (which returns server-side warnings listing every existing Pod in `prod` that violates `restricted`).
- B. `kubectl get pods -n prod --validate=true`
- C. `kubectl auth can-i create pods -n prod`
- D. `kubeadm config validate`

---

## Answer Key & Detailed Explanations

1. **A** — `privileged` has no restrictions, `baseline` prevents known privilege escalations, and `restricted` enforces current pod hardening best practices.
2. **A** — Pod Security Admission uses `pod-security.kubernetes.io/<mode>=<level>` and `pod-security.kubernetes.io/<mode>-version=<version>` labels on Namespace objects.
3. **A** — Admission controllers only intercept API requests (`CREATE`/`UPDATE`); existing running Pods are untouched until a controller attempts to `CREATE` a new Pod.
4. **A** — Only `NET_BIND_SERVICE` may be added back after dropping `ALL` capabilities in the `restricted` profile.
5. **A** — `hostPath` volumes allow trivial escape to the underlying worker node filesystem and are blocked in both `baseline` and `restricted`.
6. **A** — The built-in `PodSecurity` admission plugin accepts a `PodSecurityConfiguration` via `--admission-control-config-file` on `kube-apiserver`.
7. **A** — `restricted` strictly requires four properties: `runAsNonRoot: true`, `seccompProfile: RuntimeDefault/Localhost`, `allowPrivilegeEscalation: false`, and `capabilities.drop: ["ALL"]`.
8. **A** — Host namespace sharing (`hostNetwork`, `hostPID`, `hostIPC`) breaks container isolation and is forbidden in both `baseline` and `restricted`.
9. **A** — `procMount: Default` maintains OCI runtime masking of dangerous `/proc` subtrees.
10. **A** — Running `kubectl label --dry-run=server` triggers the `PodSecurity` admission plugin's audit of existing Pods in that namespace and outputs any violations without persisting the label change.
