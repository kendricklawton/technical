# CKS Full-Length Performance Practice Exam 1

**Time Limit:** 120 Minutes  
**Passing Score:** 67% (Score Target: $\ge 75\%$)  
**Topology:** DigitalOcean Droplet Multi-Node Simulation (`cks-cp1` / `controlplane`, `cks-worker1` / `node01`, `cks-worker2` / `node02`)  
**Format:** 16 Real-World Hands-On Security Tasks with Full Solution Manifests & Verification Steps

---

### Task 1: Encrypt Secrets at Rest in etcd (Weight: 7%)
**Context:** `kubectl config use-context k8s-security`  
**Scenario:** An audit revealed that Kubernetes Secrets stored in etcd are currently in plaintext. Configure the Kubernetes API server to encrypt secrets at rest using the `aescbc` provider.

**Requirements:**
1. Generate a 32-byte base64-encoded key: `head -c 32 /dev/urandom | base64`.
2. Create an `EncryptionConfiguration` file at `/etc/kubernetes/enc/enc.yaml` with the `aescbc` provider as the primary provider, followed by the `identity` provider as fallback.
3. Update `/etc/kubernetes/manifests/kube-apiserver.yaml` to enable `--encryption-provider-config=/etc/kubernetes/enc/enc.yaml` and configure required host volume mounts.
4. Ensure `kube-apiserver` restarts cleanly and re-encrypt all existing secrets across all namespaces: `kubectl get secrets --all-namespaces -o json | kubectl replace -f -`.

**Verification:**
Query etcd directly with `ETCDCTL_API=3 etcdctl get /registry/secrets/default/<secret-name>` and verify the prefix `k8s:enc:aescbc:v1:key1`.

---

### Task 2: CIS Benchmark Audit & Kube-Bench Remediation (Weight: 6%)
**Context:** `kubectl config use-context k8s-security`  
**Scenario:** Run `kube-bench` on the master control plane node (`cks-cp1`) and remediate the following failures:
1. Ensure the `--profiling` argument is set to `false` on `kube-controller-manager`.
2. Ensure the `--authorization-mode` argument on `kube-apiserver` includes `NodeRestriction`.
3. Ensure file permissions on `/etc/kubernetes/manifests/kube-apiserver.yaml` are set to `600` and owned by `root:root`.

**Verification:**
Run `kube-bench run --targets master` and verify tests pass with `[PASS]`.

---

### Task 3: Restrict Pod Security with Pod Security Standards (PSA) (Weight: 6%)
**Context:** `kubectl config use-context k8s-security`  
**Scenario:** Configure namespace `finance-apps` to enforce the `restricted` Pod Security Standard. Warn and audit on violations as well.

**Requirements:**
1. Label the namespace:
   ```bash
   kubectl label --overwrite namespace finance-apps \
     pod-security.kubernetes.io/enforce=restricted \
     pod-security.kubernetes.io/enforce-version=latest \
     pod-security.kubernetes.io/warn=restricted \
     pod-security.kubernetes.io/warn-version=latest \
     pod-security.kubernetes.io/audit=restricted \
     pod-security.kubernetes.io/audit-version=latest
   ```

**Verification:**
Attempt to launch a privileged pod: `kubectl run test-priv --image=nginx -n finance-apps --privileged`. The API server must reject the pod creation with a PodSecurity violation error.

---

### Task 4: In-Kernel Runtime Threat Detection with Falco (Weight: 7%)
**Context:** `ssh cks-worker1` (`node01`)  
**Scenario:** On worker Droplet `cks-worker1` (`node01`), configure Falco to trigger a `WARNING` alert whenever any process inside a container opens `/etc/shadow` or attempts to modify `/etc/sudoers`.

**Requirements:**
1. Edit `/etc/falco/falco_rules.local.yaml`.
2. Add a rule named `Sensitive File Modification in Container` with condition `open_write and container and (fd.name in (/etc/shadow, /etc/sudoers))`.
3. Set priority to `WARNING`.
4. Restart Falco via `systemctl restart falco-modern-bpf` (or `systemctl restart falco`) and verify status with `journalctl -u falco-modern-bpf -e`.

**Verification:**
Spawn a container modifying `/etc/shadow` and verify alert output in `/var/log/syslog`.

---

### Task 5: Mitigate DigitalOcean / Cloud Metadata Attack (SSRF Prevention) (Weight: 6%)
**Context:** `kubectl config use-context k8s-security`  
**Scenario:** Prevent pods running in namespace `workloads` from accessing the DigitalOcean Droplet / cloud provider instance metadata service at `169.254.169.254/32` (`http://169.254.169.254/metadata/v1.json`). All other egress traffic (including DNS and internal pods) must remain allowed.

**Requirements:**
Apply a NetworkPolicy in namespace `workloads`:
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: block-cloud-metadata
  namespace: workloads
spec:
  podSelector: {}
  policyTypes:
  - Egress
  egress:
  - to:
    - ipBlock:
        cidr: 0.0.0.0/0
        except:
        - 169.254.169.254/32
```

**Verification:**
Exec into any pod in `workloads` and test `curl --connect-timeout 2 http://169.254.169.254/metadata/v1.json`. The connection must time out.

---

### Task 6: Audit Log Policy Configuration (Weight: 7%)
**Context:** `kubectl config use-context k8s-security`  
**Scenario:** Configure Kubernetes API Server auditing with the following policy:
1. File location: `/etc/kubernetes/audit/audit-policy.yaml`.
2. Rules:
   - Log `RequestResponse` for `secrets` and `configmaps` in all namespaces.
   - Log `Metadata` for `pods` and `deployments`.
   - Log `None` for `system:kube-proxy` and endpoints.
3. Add API server flags:
   - `--audit-policy-file=/etc/kubernetes/audit/audit-policy.yaml`
   - `--audit-log-path=/var/log/kubernetes/audit/audit.log`
   - `--audit-log-maxbackup=10`
   - `--audit-log-maxsize=100`
4. Configure volume mounts and host volumes in `/etc/kubernetes/manifests/kube-apiserver.yaml`.

**Verification:**
Create a secret `kubectl create secret generic test-sec --from-literal=k=v` and verify a `RequestResponse` audit record in `/var/log/kubernetes/audit/audit.log`.

---

### Task 7: AppArmor Profile Enforcement (Weight: 6%)
**Context:** `ssh cks-worker1` (`node01`)  
**Scenario:** A custom AppArmor profile `k8s-deny-write` is provided at `/etc/apparmor.d/k8s-deny-write`.

**Requirements:**
1. Load the profile on `cks-worker1` (`node01`): `apparmor_parser -q /etc/apparmor.d/k8s-deny-write`.
2. Confirm active status: `aa-status | grep k8s-deny-write`.
3. Create a Pod `secure-writer` in namespace `default` with image `busybox:1.36` configured with `securityContext.appArmorProfile.type: Localhost` and `localhostProfile: k8s-deny-write`.

**Verification:**
Exec into `secure-writer` and attempt to write a file in `/`: `touch /test.txt`. The kernel must return `Operation not permitted`.

---

### Task 8: Seccomp Profile Implementation (Weight: 6%)
**Context:** `kubectl config use-context k8s-security`  
**Scenario:** A custom seccomp profile `fine-grained.json` is stored in `/var/lib/kubelet/seccomp/profiles/fine-grained.json` across all worker Droplets.

**Requirements:**
Configure the Deployment `secure-nginx` in namespace `web` to use this seccomp profile via `securityContext.seccompProfile.type: Localhost` with `localhostProfile: profiles/fine-grained.json`.

**Verification:**
Inspect the deployed pod: `kubectl get pod -n web -o yaml | grep -A 3 seccompProfile`.

---

### Task 9: Container Image CVE Scanning with Trivy (Weight: 7%)
**Context:** `kubectl config use-context k8s-security`  
**Scenario:** Scan container images running in namespace `staging` for known vulnerabilities.

**Requirements:**
1. Identify all pods in `staging` using images with `CRITICAL` severity CVEs.
2. For image `python:3.7-slim`, output a filtered vulnerability report to `/opt/vuln-report.txt` containing only fixed vulnerabilities (`--ignore-unfixed`).
3. Delete any pod in `staging` that uses an image with $>5$ Critical CVEs.

**Verification:**
Run `trivy image --severity CRITICAL python:3.7-slim` and verify output file exists.

---

### Task 10: Cryptographic Image Verification with Cosign (Weight: 6%)
**Context:** `kubectl config use-context k8s-security`  
**Scenario:** A company enforces that all production images must be cryptographically signed by corporate release keys.

**Requirements:**
1. Public key is at `/etc/cosign/release-key.pub`.
2. Verify image `myregistry.internal/apps/payment:v2.1.0`.
3. If signature verification fails, remove the corresponding deployment `payment-service` from namespace `prod`.

**Verification:**
Run `cosign verify --key /etc/cosign/release-key.pub myregistry.internal/apps/payment:v2.1.0`.

---

### Task 11: Declarative Admission with ValidatingAdmissionPolicy (Weight: 7%)
**Context:** `kubectl config use-context k8s-security`  
**Scenario:** Create an in-tree Common Expression Language (CEL) admission policy named `enforce-resource-limits`.

**Requirements:**
1. Policy must reject any Pod creation where any container does not specify `resources.limits.memory`.
2. Bind the policy to namespace `dev-team-a` with validation action `Deny`.

**Verification:**
Attempt to launch a pod in `dev-team-a` without memory limits. The API server must return `Forbidden` with your custom message.

---

### Task 12: Kubelet Port 10250 Authentication & Authorization (Weight: 6%)
**Context:** `ssh cks-worker1` (`node01`)  
**Scenario:** Harden worker Droplet `cks-worker1`'s Kubelet configuration at `/var/lib/kubelet/config.yaml`.

**Requirements:**
1. Disable anonymous authentication: `authentication.anonymous.enabled: false`.
2. Set authorization mode to `Webhook`: `authorization.mode: Webhook`.
3. Disable read-only port: `readOnlyPort: 0`.
4. Restart the kubelet service and ensure it reaches `active (running)`.

**Verification:**
From the control plane (`cks-cp1`), attempt an unauthenticated curl to `https://10.116.0.11:10250/pods`. The server must return `Unauthorized`.

---

### Task 13: ServiceAccount Hardening & Token Mounting (Weight: 6%)
**Context:** `kubectl config use-context k8s-security`  
**Scenario:** Harden namespace `internal-tools` against token theft.

**Requirements:**
1. Update the default ServiceAccount in `internal-tools` to disable automatic token mounting: `automountServiceAccountToken: false`.
2. For Deployment `cron-worker` in `internal-tools`, ensure `spec.template.spec.automountServiceAccountToken: false` is configured.

**Verification:**
Exec into `cron-worker` pod and verify `/var/run/secrets/kubernetes.io/serviceaccount` does not exist.

---

### Task 14: Defense-in-Depth Pod SecurityContext (Weight: 6%)
**Context:** `kubectl config use-context k8s-security`  
**Scenario:** Harden Pod `api-gateway` in namespace `prod`.

**Requirements:**
1. Set `runAsNonRoot: true` and `runAsUser: 10001`.
2. Set `allowPrivilegeEscalation: false`.
3. Set `readOnlyRootFilesystem: true`.
4. Drop all capabilities: `capabilities.drop: ["ALL"]`.
5. Mount an `emptyDir` volume at `/tmp` for writable cache files.

**Verification:**
`kubectl exec -it -n prod api-gateway -- touch /etc/test` must fail with Read-only file system.

---

### Task 15: Incident Response & Compromised Pod Isolation (Weight: 7%)
**Context:** `kubectl config use-context k8s-security`  
**Scenario:** Intrusion detection alerts that pod `legacy-app` in namespace `legacy` has been compromised and is executing a port scan.

**Requirements:**
1. Create a NetworkPolicy `isolate-legacy` in namespace `legacy` matching pod label `app: legacy-app` that denies all inbound and outbound traffic.
2. Capture process trees of the running container to `/tmp/forensics-proc.txt` using `crictl inspect`.
3. Do NOT delete the pod.

**Verification:**
Test that `legacy-app` cannot send packets or receive packets while remaining in `Running` state.

---

### Task 16: Tetragon eBPF Process Termination (Weight: 7%)
**Context:** `kubectl config use-context k8s-security`  
**Scenario:** Deploy a Tetragon `TracingPolicy` named `kill-namespace-tampering`.

**Requirements:**
1. Detect any execution of `setns` or `unshare` syscalls by non-root containers.
2. Configure action `Sigkill` to terminate the offending process instantly at the Linux kernel level.

**Verification:**
Inspect Tetragon event stream via `tetra getevents` and verify blocked process events.
