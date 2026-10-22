# CKS Full-Length Performance Practice Exam 5

**Time Limit:** 120 Minutes  
**Passing Score:** 67% (Score Target: $\ge 80\%$)  
**Topology:** DigitalOcean Droplet Multi-Node Simulation (`cks-cp1`, `cks-worker1`, `cks-worker2`) on Private VPC (`eth1`)  
**Format:** 16 Real-World Hands-On Security Tasks with Full Solution Manifests & Verification Commands

---

### Task 1: Falco Rule: Detect Sensitive File Reads in /etc/shadow (Weight: 7%)
**Context:** `ssh root@cks-worker2`  
**Scenario:** Author a Falco rule alerting whenever any container reads `/etc/shadow` or `/etc/security`.
**Requirements:**
1. Rule name: `Read Sensitive Shadow File`.
2. Condition: `open_read and container and fd.name startswith /etc/shadow`.
3. Output: `Sensitive credential read detected (user=%user.name pod=%k8s.pod.name file=%fd.name)`.
4. Priority: `CRITICAL`.
5. Apply and verify rule in `/etc/falco/falco_rules.local.yaml`.

**Solution:**
```yaml
- rule: Read Sensitive Shadow File
  desc: Detect access to /etc/shadow inside containers
  condition: open_read and container and (fd.name startswith /etc/shadow or fd.name startswith /etc/security)
  output: "Sensitive credential read detected (user=%user.name pod=%k8s.pod.name file=%fd.name)"
  priority: CRITICAL
  tags: [credentials, k8s, cks]
```
```bash
systemctl restart falco
# Test rule
kubectl run sec-test2 --image=busybox --restart=Never -it -- cat /etc/shadow
journalctl -u falco -n 20 --no-pager | grep "Sensitive credential read detected"
```

---

### Task 2: Tetragon eBPF Process Execution Enforcement (Weight: 8%)
**Context:** `kubectl config use-context cks-cluster`  
**Scenario:** Deploy a Tetragon `TracingPolicy` that blocks (via `SIGKILL`) any process attempting to execute `nc`, `ncat`, or `netcat` inside namespace `finance`.

**Solution:**
```yaml
apiVersion: cilium.io/v1alpha1
kind: TracingPolicy
metadata:
  name: block-netcat-exec
  namespace: finance
spec:
  kprobes:
    - call: "sys_execve"
      syscall: true
      args:
        - index: 0
          type: "string"
      selectors:
        - matchArgs:
            - index: 0
              operator: "Prefix"
              values:
                - "/bin/nc"
                - "/usr/bin/nc"
                - "/bin/netcat"
          matchActions:
            - action: Sigkill
```

---

### Task 3: Kube-Bench Master & Worker Automated Remediation (Weight: 7%)
**Context:** `ssh root@cks-cp1`  
**Scenario:** Run `kube-bench run --targets master` and fix all FAIL items:
1. Ensure `--profiling=false` on `kube-controller-manager` and `kube-scheduler`.
2. Ensure `--protect-kernel-defaults=true` on `kubelet`.
3. Fix ownership on `/etc/kubernetes/pki` to `root:root` and permissions to `0700`.

**Solution:**
```bash
sed -i 's/--profiling=true/--profiling=false/' /etc/kubernetes/manifests/kube-controller-manager.yaml
sed -i 's/--profiling=true/--profiling=false/' /etc/kubernetes/manifests/kube-scheduler.yaml
echo "protectKernelDefaults: true" >> /var/lib/kubelet/config.yaml
chmod -R 0700 /etc/kubernetes/pki
systemctl restart kubelet
kube-bench run --targets master
```

---

### Task 4: Cosign Software Bill of Materials (SBOM) Attestation Verification (Weight: 7%)
**Context:** `kubectl config use-context cks-cluster`  
**Scenario:** Verify the SBOM attestation of container image `ghcr.io/enterprise/core-app:v1.0.0` using `cosign verify-attestation` against public key `/opt/cosign/cosign.pub`.

**Solution:**
```bash
cosign verify-attestation \
  --key /opt/cosign/cosign.pub \
  --type spdxjson \
  ghcr.io/enterprise/core-app:v1.0.0
```

---

### Task 5: Pod Security Admission Audit Reporting (Weight: 6%)
**Context:** `kubectl config use-context cks-cluster`  
**Scenario:** In namespace `audit-staging`, configure PSA to audit against `restricted` without blocking deployments, and view generated audit logs in `/var/log/kubernetes/audit.log`.

**Solution:**
```bash
kubectl label --overwrite namespace audit-staging \
  pod-security.kubernetes.io/audit=restricted \
  pod-security.kubernetes.io/audit-version=latest
```

---

### Task 6: Secure etcd TLS Client Authentication Configuration (Weight: 7%)
**Context:** `ssh root@cks-cp1`  
**Scenario:** Verify that etcd is strictly configured with `--client-cert-auth=true` in `/etc/kubernetes/manifests/etcd.yaml`.

**Solution:**
```bash
grep -i "client-cert-auth" /etc/kubernetes/manifests/etcd.yaml
```

---

### Task 7: Cilium DNS-Based Egress Filtering Policy (Weight: 7%)
**Context:** `kubectl config use-context cks-cluster`  
**Scenario:** Author a `CiliumNetworkPolicy` for `auth-service` pods in namespace `auth` permitting egress only to `auth.okta.com` over HTTPS port 443, plus kube-dns port 53.

**Solution:**
```yaml
apiVersion: "cilium.io/v2"
kind: CiliumNetworkPolicy
metadata:
  name: okta-egress-only
  namespace: auth
spec:
  endpointSelector:
    matchLabels:
      app: auth-service
  egress:
    - toEndpoints:
        - matchLabels:
            k8s:io.kubernetes.pod.namespace: kube-system
            k8s-app: kube-dns
      toPorts:
        - ports:
            - port: "53"
              protocol: UDP
          rules:
            dns:
              - matchPattern: "*"
    - toFQDNs:
        - matchName: "auth.okta.com"
      toPorts:
        - ports:
            - port: "443"
              protocol: TCP
```

---

### Task 8: Dockerfile & Base Image Hardening Audit (Weight: 6%)
**Context:** `kubectl config use-context cks-cluster`  
**Scenario:** Inspect the developer Dockerfile at `/opt/src/Dockerfile` and remediate security vulnerabilities:
1. Replace `FROM ubuntu:latest` with non-root distroless base: `gcr.io/distroless/static:nonroot`.
2. Remove package managers (`apt-get`).
3. Ensure `USER 65532:65532` is explicitly declared.

**Solution:**
```dockerfile
# /opt/src/Dockerfile
FROM golang:1.22-alpine AS builder
WORKDIR /app
COPY . .
RUN CGO_ENABLED=0 go build -o server .

FROM gcr.io/distroless/static:nonroot
WORKDIR /
COPY --from=builder /app/server /server
USER 65532:65532
ENTRYPOINT ["/server"]
```

---

### Task 9: ServiceAccount Token Projection & Audience Restriction (Weight: 5%)
**Context:** `kubectl config use-context cks-cluster`  
**Scenario:** Configure a pod `vault-agent` in namespace `vault` that projects a service account token with `audience: https://vault.corp.internal` and expiration of 15 minutes (900 seconds).

**Solution:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: vault-agent
  namespace: vault
spec:
  containers:
    - name: agent
      image: busybox:1.36
      command: ["sleep", "3600"]
      volumeMounts:
        - name: vault-token
          mountPath: /var/run/secrets/tokens
  volumes:
    - name: vault-token
      projected:
        sources:
          - serviceAccountToken:
              path: vault-token
              expirationSeconds: 900
              audience: https://vault.corp.internal
```

---

### Task 10: Kubernetes RBAC ClusterRole Aggregation (Weight: 6%)
**Context:** `kubectl config use-context cks-cluster`  
**Scenario:** Create an aggregated `ClusterRole` named `platform-auditor` that dynamically aggregates all roles with label `rbac.platform.io/aggregate-to-auditor: "true"`.

**Solution:**
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: platform-auditor
aggregationRule:
  clusterRoleSelectors:
    - matchLabels:
        rbac.platform.io/aggregate-to-auditor: "true"
rules: []
```

---

### Task 11: Securing Kubelet Read-Only Port 10255 (Weight: 5%)
**Context:** `ssh root@cks-worker2`  
**Scenario:** Ensure the unauthenticated read-only Kubelet port `10255` is completely disabled.

**Solution:**
```bash
grep -q "readOnlyPort: 0" /var/lib/kubelet/config.yaml || echo "readOnlyPort: 0" >> /var/lib/kubelet/config.yaml
systemctl restart kubelet
ss -tulpn | grep 10255 (Must return nothing)
```

---

### Task 12: NetworkPolicy Default-Deny with Selective Ingress (Weight: 6%)
**Context:** `kubectl config use-context cks-cluster`  
**Scenario:** In namespace `secure-data`, create a default-deny NetworkPolicy that isolates all pods, and a secondary policy allowing ingress on TCP port `3306` only from pods labeled `role: web` in namespace `frontend`.

**Solution:**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
  namespace: secure-data
spec:
  podSelector: {}
  policyTypes:
    - Ingress
    - Egress
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-to-db
  namespace: secure-data
spec:
  podSelector:
    matchLabels:
      app: mysql
  policyTypes:
    - Ingress
  ingress:
    - from:
        - namespaceSelector:
            matchLabels:
              kubernetes.io/metadata.name: frontend
          podSelector:
            matchLabels:
              role: web
      ports:
        - protocol: TCP
          port: 3306
```

---

### Task 13: Kyverno Disallowing HostPath Mounts (Weight: 6%)
**Context:** `kubectl config use-context cks-cluster`  
**Scenario:** Create a Kyverno `ClusterPolicy` blocking any pod that attempts to mount a `hostPath` volume.

**Solution:**
```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: disallow-hostpath
spec:
  validationFailureAction: Enforce
  rules:
    - name: validate-no-hostpath
      match:
        any:
          - resources:
              kinds: [Pod]
              namespaces: ["!kube-system"]
      validate:
        message: "hostPath volumes are prohibited."
        pattern:
          spec:
            =(volumes):
              - =(hostPath): "!*"
```

---

### Task 14: Restricting Linux Syscalls with Seccomp RuntimeDefault (Weight: 5%)
**Context:** `kubectl config use-context cks-cluster`  
**Scenario:** Deploy a pod `hardened-web` in namespace `default` that enables the `RuntimeDefault` seccomp profile across all containers.

**Solution:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hardened-web
  namespace: default
spec:
  securityContext:
    seccompProfile:
      type: RuntimeDefault
  containers:
    - name: web
      image: nginx:alpine
```

---

### Task 15: Auditing Open Network Ports on Worker Nodes (Weight: 5%)
**Context:** `ssh root@cks-worker1`  
**Scenario:** Audit all listening network ports on worker node `cks-worker1` using `ss` and terminate any unauthorized process listening on port `8080`.

**Solution:**
```bash
ss -tulpn | grep 8080
PID=$(ss -tulpn | grep 8080 | awk '{print $NF}' | sed 's/.*pid=\([0-9]*\).*/\1/')
kill -9 "$PID"
```

---

### Task 16: Verify API Server TLS Cipher Suites (Weight: 5%)
**Context:** `ssh root@cks-cp1`  
**Scenario:** Ensure `kube-apiserver` only uses strong TLS 1.3 ciphers (`--tls-cipher-suites=TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256`) and `--tls-min-version=VersionTLS12`.

**Solution:**
```bash
grep -i "tls-min-version" /etc/kubernetes/manifests/kube-apiserver.yaml
grep -i "tls-cipher-suites" /etc/kubernetes/manifests/kube-apiserver.yaml
```
