# CKS Full-Length Performance Practice Exam 4

**Time Limit:** 120 Minutes  
**Passing Score:** 67% (Score Target: $\ge 80\%$)  
**Topology:** DigitalOcean Droplet Multi-Node Simulation (`cks-cp1`, `cks-worker1`, `cks-worker2`) on Private VPC (`eth1`)  
**Format:** 16 Real-World Hands-On Security Tasks with Full Solution Manifests & Verification Commands

---

### Task 1: Falco Custom Rule: Detect Interactive Shells in Production Pods (Weight: 7%)
**Context:** `ssh root@cks-worker1`  
**Scenario:** Create a custom Falco rule to alert whenever an interactive shell (`bash`, `sh`, `zsh`) is spawned inside a container running in the `production` namespace.
**Requirements:**
1. Append rule to `/etc/falco/falco_rules.local.yaml`.
2. Rule name: `Terminal Shell in Production Pod`.
3. Conditions: `spawn_process`, `k8s.ns.name = "production"`, and `proc.name in (bash, sh, zsh)`.
4. Output format: `Shell spawned in production pod (user=%user.name pod=%k8s.pod.name ns=%k8s.ns.name command=%proc.cmdline)`.
5. Priority: `WARNING`.
6. Restart Falco: `systemctl restart falco`.

**Solution:**
```yaml
# /etc/falco/falco_rules.local.yaml
- rule: Terminal Shell in Production Pod
  desc: Detect interactive shell execution in production pods
  condition: >
    spawned_process and
    k8s.ns.name = "production" and
    proc.name in (bash, sh, zsh, dash)
  output: "Shell spawned in production pod (user=%user.name pod=%k8s.pod.name ns=%k8s.ns.name command=%proc.cmdline)"
  priority: WARNING
  tags: [security, k8s, production]
```
```bash
systemctl restart falco
systemctl status falco
# Test rule:
kubectl run sec-test --image=busybox -n production --restart=Never -it -- sh -c "echo Test Shell"
journalctl -u falco -n 20 --no-pager | grep "Shell spawned in production pod"
```

---

### Task 2: Custom AppArmor Profile Creation & Pod Enforcement (Weight: 7%)
**Context:** `ssh root@cks-worker2`  
**Scenario:** Create and load an AppArmor profile preventing file writes to `/var/log` and blocking raw network sockets.
**Requirements:**
1. Profile file: `/etc/apparmor.d/k8s-deny-write`.
2. Profile name: `k8s-deny-write`.
3. Deny write access to `/var/log/**`.
4. Load profile into the Linux kernel: `apparmor_parser -r /etc/apparmor.d/k8s-deny-write`.
5. Deploy pod `apparmor-pod` in namespace `default` with image `busybox` and enforcing the profile via `securityContext.appArmorProfile.type: Localhost` and `localhostProfile: k8s-deny-write`.

**Solution:**
```bash
cat <<EOF > /etc/apparmor.d/k8s-deny-write
#include <tunables/global>

profile k8s-deny-write flags=(attach_disconnected,mediate_deleted) {
  #include <abstractions/base>

  file,
  deny /var/log/** w,
}
EOF

apparmor_parser -r /etc/apparmor.d/k8s-deny-write
aa-status | grep k8s-deny-write
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: apparmor-pod
  namespace: default
spec:
  securityContext:
    appArmorProfile:
      type: Localhost
      localhostProfile: k8s-deny-write
  containers:
    - name: test
      image: busybox:1.36
      command: ["sh", "-c", "touch /var/log/test.txt || sleep 3600"]
```

---

### Task 3: Seccomp Profile Creation & Pod Hardening (Weight: 6%)
**Context:** `ssh root@cks-worker1`  
**Scenario:** Configure a custom Seccomp profile on worker node `cks-worker1` that denies `chmod`, `chown`, and `mount` syscalls with `SCMP_ACT_ERRNO`.
**Requirements:**
1. Create directory `/var/lib/kubelet/seccomp/profiles`.
2. Save profile to `/var/lib/kubelet/seccomp/profiles/deny-chmod.json`.
3. Default action: `SCMP_ACT_ALLOW`.
4. Architectures: `SCMP_ARCH_X86_64`.
5. Syscalls denied: `chmod`, `fchmod`, `fchmodat`, `chown`, `mount`.
6. Enforce profile on pod `seccomp-hardened-pod` via `securityContext.seccompProfile.type: Localhost` and `localhostProfile: profiles/deny-chmod.json`.

**Solution:**
```json
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
        "chown",
        "fchown",
        "mount"
      ],
      "action": "SCMP_ACT_ERRNO"
    }
  ]
}
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: seccomp-hardened-pod
  namespace: default
spec:
  securityContext:
    seccompProfile:
      type: Localhost
      localhostProfile: profiles/deny-chmod.json
  containers:
    - name: app
      image: busybox:1.36
      command: ["sleep", "3600"]
```

---

### Task 4: gVisor Sandboxed RuntimeClass Configuration (Weight: 6%)
**Context:** `kubectl config use-context cks-cluster`  
**Scenario:** Configure a Kubernetes `RuntimeClass` for gVisor (`runsc`) to execute untrusted user-submitted code in an isolated kernel sandbox.
**Requirements:**
1. Create a `RuntimeClass` named `gvisor` with `handler: runsc`.
2. Deploy pod `untrusted-runner` in namespace `sandbox` using `runtimeClassName: gvisor`.
3. Verify that the pod runs inside the gVisor sandbox by inspecting `dmesg`.

**Solution:**
```yaml
apiVersion: node.k8s.io/v1
kind: RuntimeClass
metadata:
  name: gvisor
handler: runsc
---
apiVersion: v1
kind: Pod
metadata:
  name: untrusted-runner
  namespace: sandbox
spec:
  runtimeClassName: gvisor
  containers:
    - name: runner
      image: busybox:1.36
      command: ["sh", "-c", "dmesg | head -n 5; sleep 3600"]
```
**Verification:**
```bash
kubectl apply -f runtimeclass.yaml
kubectl logs untrusted-runner -n sandbox
# Output must indicate "Starting gVisor..."
```

---

### Task 5: Kyverno Image Signature Verification via Cosign (Weight: 7%)
**Context:** `kubectl config use-context cks-cluster`  
**Scenario:** Block all pods in namespace `payments` unless their container images are cryptographically signed by the company's Cosign public key.

**Solution:**
```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: verify-payments-signatures
spec:
  validationFailureAction: Enforce
  webhookTimeoutSeconds: 30
  rules:
    - name: verify-cosign-key
      match:
        any:
          - resources:
              kinds: [Pod]
              namespaces: [payments]
      verifyImages:
        - imageReferences:
            - "ghcr.io/enterprise/*"
          attestors:
            - entries:
                - keys:
                    secret:
                      name: cosign-public-key
                      namespace: kyverno
```

---

### Task 6: etcd Encryption Key Rotation (Weight: 8%)
**Context:** `ssh root@cks-cp1`  
**Scenario:** Rotate the encryption key used to encrypt Secrets in etcd.
**Requirements:**
1. Generate new 32-byte key: `head -c 32 /dev/urandom | base64`.
2. Edit `/etc/kubernetes/enc/enc.yaml`. Add `key2` as the FIRST provider under `aescbc`, keeping `key1` as secondary.
3. Restart `kube-apiserver`.
4. Re-encrypt all existing secrets: `kubectl get secrets -A -o json | kubectl replace -f -`.
5. Remove `key1` once re-encryption completes.

**Solution:**
```yaml
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
  - resources:
      - secrets
    providers:
      - aescbc:
          keys:
            - name: key2
              secret: <NEW_BASE64_KEY>
            - name: key1
              secret: <OLD_BASE64_KEY>
      - identity: {}
```
```bash
# Wait for API server reload
sleep 10
kubectl get secrets --all-namespaces -o json | kubectl replace -f -
```

---

### Task 7: Kubernetes Audit Logging with Metadata Stage Filter (Weight: 7%)
**Context:** `ssh root@cks-cp1`  
**Scenario:** Configure the Kubernetes API server audit logging policy to capture:
- Requests to `secrets` and `configmaps` at `Metadata` level (avoiding token leakage).
- All cluster-admin actions in `kube-system` at `RequestResponse` level.
- Drop all `RequestReceived` stage events.
- Path: `/etc/kubernetes/audit/audit-policy.yaml`. Log destination: `/var/log/kubernetes/audit.log`.

**Solution:**
```yaml
apiVersion: audit.k8s.io/v1
kind: Policy
omitStages:
  - "RequestReceived"
rules:
  - level: RequestResponse
    namespaces: ["kube-system"]
    verbs: ["create", "update", "patch", "delete"]
  - level: Metadata
    resources:
      - group: ""
        resources: ["secrets", "configmaps"]
  - level: None
    users: ["system:kube-scheduler", "system:kube-proxy"]
  - level: Metadata
```

---

### Task 8: Trivy CI Container Image Vulnerability Gating (Weight: 6%)
**Context:** `kubectl config use-context cks-cluster`  
**Scenario:** Scan container image `nginx:1.14.0` using Trivy and generate a report `/opt/cks/nginx-scan.txt` failing with exit code 1 if any CRITICAL severity vulnerabilities exist.

**Solution:**
```bash
trivy image --severity CRITICAL --exit-code 1 --output /opt/cks/nginx-scan.txt nginx:1.14.0
cat /opt/cks/nginx-scan.txt
```

---

### Task 9: CIS Benchmark Kubelet Hardening (Weight: 6%)
**Context:** `ssh root@cks-worker1`  
**Scenario:** Remediate CIS Benchmark failures on worker node kubelet configuration `/var/lib/kubelet/config.yaml`:
1. Ensure `anonymous.enabled: false`.
2. Ensure `authorization.mode: Webhook`.
3. Set `readOnlyPort: 0`.
4. Restart kubelet: `systemctl restart kubelet`.

**Solution:**
```bash
sed -i 's/enabled: true/enabled: false/' /var/lib/kubelet/config.yaml
# Ensure readOnlyPort is 0:
grep -q "readOnlyPort" /var/lib/kubelet/config.yaml && sed -i 's/readOnlyPort:.*/readOnlyPort: 0/' /var/lib/kubelet/config.yaml || echo "readOnlyPort: 0" >> /var/lib/kubelet/config.yaml
systemctl restart kubelet
systemctl status kubelet
```

---

### Task 10: Ingress Mutual TLS (mTLS) Authentication (Weight: 6%)
**Context:** `kubectl config use-context cks-cluster`  
**Scenario:** Secure the administrative ingress `admin-ingress` in namespace `management` requiring client certificate verification using CA secret `client-ca-secret`.

**Solution:**
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: admin-ingress
  namespace: management
  annotations:
    nginx.ingress.kubernetes.io/auth-tls-verify-client: "on"
    nginx.ingress.kubernetes.io/auth-tls-secret: "management/client-ca-secret"
    nginx.ingress.kubernetes.io/auth-tls-verify-depth: "2"
spec:
  ingressClassName: nginx
  rules:
    - host: admin.internal.corp
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: admin-service
                port:
                  number: 8443
```

---

### Task 11: Pod Security Standards Restricted Admission (Weight: 6%)
**Context:** `kubectl config use-context cks-cluster`  
**Scenario:** Configure namespace `secure-finance` with Pod Security Admission enforcing the `restricted` profile and warning/auditing on violations.

**Solution:**
```bash
kubectl label --overwrite namespace secure-finance \
  pod-security.kubernetes.io/enforce=restricted \
  pod-security.kubernetes.io/enforce-version=latest \
  pod-security.kubernetes.io/warn=restricted \
  pod-security.kubernetes.io/warn-version=latest \
  pod-security.kubernetes.io/audit=restricted \
  pod-security.kubernetes.io/audit-version=latest
```

---

### Task 12: NetworkPolicy Egress Lock Down to Internal DNS Only (Weight: 5%)
**Context:** `kubectl config use-context cks-cluster`  
**Scenario:** Create a NetworkPolicy `lockdown-egress` in namespace `isolated` that permits egress ONLY to CoreDNS on port 53 (UDP/TCP). Block all other egress completely.

**Solution:**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: lockdown-egress
  namespace: isolated
spec:
  podSelector: {}
  policyTypes:
    - Egress
  egress:
    - to:
        - namespaceSelector: {}
          podSelector:
            matchLabels:
              k8s-app: kube-dns
      ports:
        - protocol: UDP
          port: 53
        - protocol: TCP
          port: 53
```

---

### Task 13: ServiceAccount RBAC Least Privilege Audit (Weight: 6%)
**Context:** `kubectl config use-context cks-cluster`  
**Scenario:** An audit detected that ServiceAccount `dashboard-sa` in namespace `kube-system` has `cluster-admin` privileges. Create a scoped `Role` and `RoleBinding` in namespace `production` granting read-only (`get`, `list`, `watch`) on `pods` and `services`, and delete the dangerous `ClusterRoleBinding`.

**Solution:**
```bash
kubectl delete clusterrolebinding dashboard-admin-binding
```
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-service-reader
  namespace: production
rules:
  - apiGroups: [""]
    resources: ["pods", "services"]
    verbs: ["get", "list", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: dashboard-read-binding
  namespace: production
subjects:
  - kind: ServiceAccount
    name: dashboard-sa
    namespace: kube-system
roleRef:
  kind: Role
  name: pod-service-reader
  apiGroup: rbac.authorization.k8s.io
```

---

### Task 14: Restrict Linux Capabilities in Pod Specification (Weight: 5%)
**Context:** `kubectl config use-context cks-cluster`  
**Scenario:** Deploy a pod `network-sniffer` in namespace `default` that drops `ALL` capabilities and adds only `NET_RAW`. Ensure root file system is read-only.

**Solution:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: network-sniffer
  namespace: default
spec:
  containers:
    - name: sniffer
      image: busybox:1.36
      command: ["sleep", "3600"]
      securityContext:
        readOnlyRootFilesystem: true
        capabilities:
          drop: ["ALL"]
          add: ["NET_RAW"]
```

---

### Task 15: File Integrity Monitoring with AIDE (Weight: 5%)
**Context:** `ssh root@cks-worker1`  
**Scenario:** Initialize the AIDE (Advanced Intrusion Detection Environment) database on worker node `cks-worker1` and run a check to verify system binary integrity.

**Solution:**
```bash
aideinit
cp /var/lib/aide/aide.db.new /var/lib/aide/aide.db
aide --check
```

---

### Task 16: NodeRestriction Admission Controller Verification (Weight: 5%)
**Context:** `ssh root@cks-cp1`  
**Scenario:** Verify that the `NodeRestriction` admission controller is enabled on `kube-apiserver` in `/etc/kubernetes/manifests/kube-apiserver.yaml`.

**Solution:**
```bash
grep -i "enable-admission-plugins" /etc/kubernetes/manifests/kube-apiserver.yaml | grep NodeRestriction
```
