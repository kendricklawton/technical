# CKS Domain 1: Cluster Setup — Quiz 2 (CIS Benchmark & `kube-bench` Remediation)

Test your knowledge of CIS Kubernetes Benchmark checks, `kube-bench` usage, control plane static pods, `etcd` permissions, and `kubelet` hardening.

---

### Question 1
You run `kube-bench run --targets master` on the control plane node and see the following failure:
```text
[FAIL] 1.2.1 Ensure that the --anonymous-auth argument is set to false (Automated)
[FAIL] 1.2.6 Ensure that the --authorization-mode argument is not set to AlwaysAllow (Automated)
```
On a `kubeadm`-provisioned cluster, which file should you edit to remediate these findings, and how do you apply the changes?

- A) Edit `/var/lib/kubelet/config.yaml` and run `systemctl restart kubelet`.
- B) Edit `/etc/kubernetes/manifests/kube-apiserver.yaml` and wait for the `kubelet` to automatically recreate the `kube-apiserver` static Pod.
- C) Edit `/etc/systemd/system/kube-apiserver.service` and run `systemctl daemon-reload && systemctl restart kube-apiserver`.
- D) Run `kubectl edit configmap -n kube-system kube-apiserver-config`.

<details>
<summary>Reveal Answer & Explanation</summary>

**Correct Answer: B**

**Explanation:**
On `kubeadm` clusters, `kube-apiserver`, `kube-controller-manager`, `kube-scheduler`, and `etcd` run as static Pods defined in `/etc/kubernetes/manifests/`. The `kubelet` watches this directory via inotify and automatically restarts the static Pod when `/etc/kubernetes/manifests/kube-apiserver.yaml` is saved.
</details>

---

### Question 2
Before editing `/etc/kubernetes/manifests/kube-apiserver.yaml` during the CKS exam, a candidate runs:
```bash
cp /etc/kubernetes/manifests/kube-apiserver.yaml /etc/kubernetes/manifests/kube-apiserver.yaml.bak
```
What harmful side effect does this command cause?

- A) The `kubelet` deletes both files because duplicate filenames are prohibited.
- B) The `kubelet` attempts to parse and run `kube-apiserver.yaml.bak` as a second static Pod, causing port conflicts on TCP port `6443`.
- C) `kube-bench` fails check 1.1.1 because `.bak` extensions are not allowed in `/etc/kubernetes`.
- D) `etcd` rejects connections from the API server due to duplicate client certificates.

<details>
<summary>Reveal Answer & Explanation</summary>

**Correct Answer: B**

**Explanation:**
The `kubelet` attempts to start a static Pod for **every** valid manifest file placed inside `/etc/kubernetes/manifests/`, regardless of its file extension (`.yaml`, `.bak`, `.orig`). Because `kube-apiserver` uses `hostNetwork: true` and binds to port `6443`, the duplicate static Pod will fight for port `6443` and cause instability. Always save backups outside `/etc/kubernetes/manifests/` (e.g., `/root/backups/`).
</details>

---

### Question 3
You run `kube-bench run --targets etcd` and observe the following two failures:
```text
[FAIL] 1.1.11 Ensure that the etcd data directory permissions are set to 700 or more restrictive
[FAIL] 1.1.12 Ensure that the etcd data directory ownership is set to etcd:etcd
```
Where is the default `etcd` data directory located on a `kubeadm` control plane node, and which commands fix both issues?

- A) `/etc/kubernetes/etcd`; `chmod 600 /etc/kubernetes/etcd && chown root:root /etc/kubernetes/etcd`
- B) `/var/lib/etcd`; `chmod 700 /var/lib/etcd && chown -R etcd:etcd /var/lib/etcd`
- C) `/var/lib/kubelet/etcd`; `chmod 755 /var/lib/kubelet/etcd && chown etcd:etcd /var/lib/kubelet/etcd`
- D) `/etc/etcd/data`; `chmod 700 /etc/etcd/data && chown -R kube:kube /etc/etcd/data`

<details>
<summary>Reveal Answer & Explanation</summary>

**Correct Answer: B**

**Explanation:**
By default, `etcd` stores its database (`member/snap/db`) in `/var/lib/etcd` on the host. CIS 1.1.11 and 1.1.12 require `chmod 700 /var/lib/etcd` and `chown -R etcd:etcd /var/lib/etcd`.
</details>

---

### Question 4
On worker node `node01`, `kube-bench run --targets node` reports:
```text
[FAIL] 4.2.1 Ensure that the --anonymous-auth argument is set to false
[FAIL] 4.2.2 Ensure that the --authorization-mode argument is not set to AlwaysAllow
[FAIL] 4.2.4 Verify that the --read-only-port argument is set to 0
```
Which settings in `/var/lib/kubelet/config.yaml` remediate all three failures?

- A)
```yaml
anonymousAuth: false
authorizationMode: RBAC
readOnlyPort: 0
```
- B)
```yaml
authentication:
  anonymous:
    enabled: false
authorization:
  mode: Webhook
readOnlyPort: 0
```
- C)
```yaml
authentication:
  anonymous:
    enabled: false
authorization:
  mode: Node,RBAC
readOnlyPort: false
```
- D)
```yaml
authentication:
  webhook:
    enabled: false
authorization:
  mode: Webhook
readOnlyPort: 10255
```

<details>
<summary>Reveal Answer & Explanation</summary>

**Correct Answer: B**

**Explanation:**
In `/var/lib/kubelet/config.yaml` (`KubeletConfiguration`), anonymous authentication is disabled via `authentication.anonymous.enabled: false`, authorization uses `authorization.mode: Webhook` (the kubelet delegates authorization checks to `kube-apiserver` via `SubjectAccessReview`), and the unauthenticated read-only port is disabled with `readOnlyPort: 0`.
</details>

---

### Question 5
In `kube-bench` output, CIS check 1.2.18 states: *"Ensure that the `--insecure-port` argument is set to `0`"*. When you add `- --insecure-port=0` to `/etc/kubernetes/manifests/kube-apiserver.yaml` on a Kubernetes v1.31 cluster, the `kube-apiserver` Pod immediately crashes and `kubectl` stops working. Why?

- A) `--insecure-port=0` requires `--insecure-bind-address=127.0.0.1` to also be set.
- B) The `--insecure-port` flag was completely removed in Kubernetes v1.24+; passing it causes `kube-apiserver` to exit with `Error: unknown flag: --insecure-port`.
- C) Port `0` is reserved by `etcd`.
- D) You must run `systemctl restart kube-apiserver` whenever changing port flags.

<details>
<summary>Reveal Answer & Explanation</summary>

**Correct Answer: B**

**Explanation:**
The `--insecure-port` and `--insecure-bind-address` flags were deprecated in Kubernetes v1.20 and **completely removed** in v1.24. If you add `--insecure-port=0` to a modern `kube-apiserver.yaml` manifest, the API server binary refuses to parse the flag and exits immediately. On modern Kubernetes, the insecure port is permanently disabled by default, so you should **never** add `--insecure-port` (and if a broken manifest has `--insecure-port`, delete the line).
</details>

---

### Question 6
What is the security purpose of setting `--use-service-account-credentials=true` in `/etc/kubernetes/manifests/kube-controller-manager.yaml` (CIS Benchmark 1.3.3)?

- A) It forces Pods created by Deployments to mount a ServiceAccount token.
- B) It runs each control loop inside `kube-controller-manager` using a separate, dedicated ServiceAccount (`system:serviceaccount:kube-system:<controller-name>`) governed by RBAC, rather than a single monolithic superuser credential.
- C) It encrypts ServiceAccount tokens stored in `etcd`.
- D) It allows `kube-controller-manager` to authenticate to `etcd` directly.

<details>
<summary>Reveal Answer & Explanation</summary>

**Correct Answer: B**

**Explanation:**
When `--use-service-account-credentials=true` is enabled, `kube-controller-manager` runs each individual controller (e.g., `deployment-controller`, `replicaset-controller`, `cronjob-controller`) with its own least-privilege ServiceAccount and matching `system:controller:*` ClusterRole.
</details>

---

### Question 7
Why does CIS Benchmark recommend setting `--profiling=false` on `kube-apiserver`, `kube-controller-manager`, and `kube-scheduler`?

- A) Profiling disables audit logging when active.
- B) Profiling exposes Go `pprof` runtime debugging data over `/debug/pprof/`, which can leak sensitive memory/program state and be abused to cause Denial-of-Service (CPU/memory exhaustion).
- C) Profiling allows containers to bypass Seccomp profiles.
- D) Profiling enables anonymous authentication on port `10250`.

<details>
<summary>Reveal Answer & Explanation</summary>

**Correct Answer: B**

**Explanation:**
Go's `pprof` profiler endpoint exposes stack traces, heap allocations, and CPU profiles. Disabling it (`--profiling=false`) reduces the attack surface and prevents resource-intensive profiling queries against control plane components.
</details>

---

### Question 8
Which `etcd` flag ensures that only clients presenting a valid X.509 certificate signed by the trusted `etcd` CA (`--trusted-ca-file`) are allowed to read or write cluster state in `etcd` (CIS 2.2)?

- A) `--auto-tls=true`
- B) `--client-cert-auth=true`
- C) `--peer-auto-tls=true`
- D) `--anonymous-auth=false`

<details>
<summary>Reveal Answer & Explanation</summary>

**Correct Answer: B**

**Explanation:**
In `etcd`, `--client-cert-auth=true` enforces mutual TLS client authentication for all client connections on port `2379`. Conversely, `--auto-tls` and `--peer-auto-tls` must be set to `false` so `etcd` does not accept self-generated untrusted certificates.
</details>

---

### Question 9
What is the purpose of setting `protectKernelDefaults: true` in `/var/lib/kubelet/config.yaml` (CIS 4.2.6)?

- A) It prevents containers from loading kernel modules via `modprobe`.
- B) It causes the `kubelet` to error out during startup if underlying host kernel `sysctl` parameters (such as `vm.overcommit_memory` or `kernel.panic`) differ from the `kubelet`'s expected safe defaults, rather than silently modifying host `sysctl` settings.
- C) It automatically enables AppArmor and Seccomp for all Pods on the node.
- D) It upgrades the Linux kernel automatically when CVEs are detected.

<details>
<summary>Reveal Answer & Explanation</summary>

**Correct Answer: B**

**Explanation:**
When `protectKernelDefaults: true` is set, the `kubelet` verifies that kernel tunables match its expectations and refuses to start if they do not, preventing the `kubelet` from overwriting host OS kernel security settings hardened by the system administrator.
</details>

---

### Question 10
When `kube-apiserver` initiates outbound connections to a `kubelet` (for example, to serve `kubectl exec`, `kubectl logs`, or `kubectl port-forward`), which `kube-apiserver` flag ensures that the API server verifies the `kubelet`'s serving TLS certificate against the cluster CA (CIS 1.2.5)?

- A) `--client-ca-file=/etc/kubernetes/pki/ca.crt`
- B) `--kubelet-certificate-authority=/etc/kubernetes/pki/ca.crt`
- C) `--tls-cert-file=/etc/kubernetes/pki/apiserver.crt`
- D) `--kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt`

<details>
<summary>Reveal Answer & Explanation</summary>

**Correct Answer: B**

**Explanation:**
By default, if `--kubelet-certificate-authority` is omitted, `kube-apiserver` does **not** verify the `kubelet`'s serving certificate when connecting to port `10250`, leaving `kubectl exec` and `kubectl logs` streams vulnerable to Man-in-the-Middle (MitM) attacks on the node network. Setting `--kubelet-certificate-authority=/etc/kubernetes/pki/ca.crt` enforces server certificate verification.
</details>
