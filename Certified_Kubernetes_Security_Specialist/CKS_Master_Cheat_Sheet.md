# CKS Master Cheat Sheet — Exam-Day Rapid Reference

This master reference contains every critical CLI command, static pod configuration flag, and YAML template required for the **Certified Kubernetes Security Specialist (CKS)** exam.

---

## 0. Terminal Setup & Control Plane Safety

```bash
# 1. Shell & Vim setup (usually pre-configured, verify at start of exam)
alias k=kubectl
export do="--dry-run=client -o yaml"
export now="--force --grace-period=0"
cat <<EOF >> ~/.vimrc
set tabstop=2 shiftwidth=2 expandtab autoindent smartindent
EOF

# 2. GOLDEN RULE: Backup static pod manifests OUTSIDE /etc/kubernetes/manifests/
mkdir -p /root/backups
cp /etc/kubernetes/manifests/kube-apiserver.yaml /root/backups/kube-apiserver.yaml.bak
cp /etc/kubernetes/manifests/etcd.yaml /root/backups/etcd.yaml.bak

# 3. Debug kube-apiserver when kubectl cannot connect
crictl ps -a | grep kube-apiserver
crictl logs $(crictl ps -a --name kube-apiserver -q | head -n 1)
tail -n 40 /var/log/pods/kube-system_kube-apiserver-*/kube-apiserver/*.log
journalctl -u kubelet -n 50 --no-pager
```

---

## 1. Domain 1: Cluster Setup (10%)

### 1.1 Default-Deny & Multi-Tier NetworkPolicies

```yaml
# Default Deny ALL Ingress + Egress in a Namespace
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
  namespace: prod
spec:
  podSelector: {}
  policyTypes:
    - Ingress
    - Egress
---
# Allow Egress DNS (Port 53 UDP/TCP) + Multi-Tier Ingress/Egress + Block Metadata IP
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: backend-netpol
  namespace: prod
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
    - Ingress
    - Egress
  ingress:
    - from:
        # Single list item with BOTH namespaceSelector and podSelector = logical AND!
        - namespaceSelector:
            matchLabels:
              kubernetes.io/metadata.name: frontend-ns
          podSelector:
            matchLabels:
              role: web
      ports:
        - protocol: TCP
          port: 8080
  egress:
    # Rule 1: Allow DNS resolution to kube-system
    - to:
        - namespaceSelector:
            matchLabels:
              kubernetes.io/metadata.name: kube-system
          podSelector:
            matchLabels:
              k8s-app: kube-dns
      ports:
        - protocol: UDP
          port: 53
        - protocol: TCP
          port: 53
    # Rule 2: Allow Egress to external CIDRs EXCEPT Cloud Metadata Server (169.254.169.254/32)
    - to:
        - ipBlock:
            cidr: 0.0.0.0/0
            except:
              - 169.254.169.254/32
```

### 1.2 Cilium L7 & DNS-Aware NetworkPolicy (`CiliumNetworkPolicy`)

```yaml
apiVersion: "cilium.io/v2"
kind: CiliumNetworkPolicy
metadata:
  name: l7-fqdn-policy
  namespace: prod
spec:
  endpointSelector:
    matchLabels:
      app: crawler
  egress:
    - toEndpoints:
        - matchLabels:
            "k8s:io.kubernetes.pod.namespace": kube-system
            "k8s:k8s-app": kube-dns
      toPorts:
        - ports:
            - port: "53"
              protocol: ANY
          rules:
            dns:
              - matchPattern: "*"
    - toFQDNs:
        - matchName: "api.github.com"
        - matchPattern: "*.trusted-partner.io"
      toPorts:
        - ports:
            - port: "443"
              protocol: TCP
```

### 1.3 CIS Benchmark (`kube-bench`) Remediation Cheat Table

```bash
# Run kube-bench for all targets or specific target
kube-bench run --targets master,node,etcd
kube-bench run --targets master --check 1.2.1,1.2.6,1.2.18
```

| Component | Config File | Common CIS Remediation Settings |
| :--- | :--- | :--- |
| **kube-apiserver** | `/etc/kubernetes/manifests/kube-apiserver.yaml` | `--anonymous-auth=false`<br>`--authorization-mode=Node,RBAC` (never `AlwaysAllow`)<br>`--enable-admission-plugins=NodeRestriction,ImagePolicyWebhook`<br>`--profiling=false`<br>`--kubelet-certificate-authority=/etc/kubernetes/pki/ca.crt`<br>Remove `--insecure-port` (deprecated/forbidden) |
| **controller-manager** | `/etc/kubernetes/manifests/kube-controller-manager.yaml` | `--profiling=false`<br>`--terminated-pod-gc-threshold=1000`<br>`--use-service-account-credentials=true`<br>`--bind-address=127.0.0.1` |
| **kube-scheduler** | `/etc/kubernetes/manifests/kube-scheduler.yaml` | `--profiling=false`<br>`--bind-address=127.0.0.1` |
| **etcd** | `/etc/kubernetes/manifests/etcd.yaml` | `--client-cert-auth=true`<br>`--peer-client-cert-auth=true`<br>`--auto-tls=false` & `--peer-auto-tls=false`<br>`chmod 700 /var/lib/etcd && chown -R etcd:etcd /var/lib/etcd`<br>`chmod 600 /etc/kubernetes/pki/etcd/*.crt /etc/kubernetes/pki/etcd/*.key` |
| **kubelet** | `/var/lib/kubelet/config.yaml` | `authentication.anonymous.enabled: false`<br>`authentication.webhook.enabled: true`<br>`authorization.mode: Webhook` (never `AlwaysAllow`)<br>`readOnlyPort: 0`<br>`protectKernelDefaults: true`<br>`rotateCertificates: true`<br>Then run: `systemctl daemon-reload && systemctl restart kubelet` |

### 1.4 Ingress with TLS & HTTPS Redirect

```bash
# Generate self-signed TLS cert & key if not provided
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout tls.key -out tls.crt -subj "/CN=secure.example.com"

# Create kubernetes.io/tls Secret
kubectl -n prod create secret tls web-tls-secret --cert=tls.crt --key=tls.key
```

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: secure-ingress
  namespace: prod
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
spec:
  ingressClassName: nginx
  tls:
    - hosts:
        - secure.example.com
      secretName: web-tls-secret
  rules:
    - host: secure.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: web-svc
                port:
                  number: 80
```

### 1.5 Verifying Platform Binaries

```bash
# Compare binary SHA512 checksum against official release hash
K8S_VER=$(kubectl version -o json | jq -r .serverVersion.gitVersion)
curl -sLO "https://dl.k8s.io/release/${K8S_VER}/bin/linux/amd64/kubectl.sha512"
echo "$(cat kubectl.sha512)  /usr/bin/kubectl" | sha512sum --check
# Manual comparison:
sha512sum /usr/bin/kubelet /usr/bin/kubectl /usr/bin/kubeadm
```

---

## 2. Domain 2: Cluster Hardening (15%)

### 2.1 RBAC Auditing & Least Privilege

```bash
# List all ClusterRoleBindings and RoleBindings in a namespace
kubectl get clusterrolebindings -o wide | grep -E "system:anonymous|system:unauthenticated|system:serviceaccounts"
kubectl get role,rolebinding -n prod -o wide

# Check effective permissions of a ServiceAccount
kubectl auth can-i --list --as=system:serviceaccount:prod:app-sa -n prod
kubectl auth can-i get secrets --as=system:serviceaccount:prod:app-sa -n prod

# Create least-privilege Role & RoleBinding
kubectl -n prod create role pod-reader --verb=get,list,watch --resource=pods
kubectl -n prod create rolebinding app-pod-reader --role=pod-reader --serviceaccount=prod:app-sa
```

### 2.2 ServiceAccount Hardening & Projected Bound Tokens

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: hardened-sa
  namespace: prod
automountServiceAccountToken: false
---
apiVersion: v1
kind: Pod
metadata:
  name: secure-app
  namespace: prod
spec:
  serviceAccountName: hardened-sa
  automountServiceAccountToken: false # Overrides SA setting if both exist
  containers:
    - name: app
      image: nginx:1.27-alpine
      volumeMounts:
        - name: token-vol
          mountPath: /var/run/secrets/tokens
          readOnly: true
  volumes:
    - name: token-vol
      projected:
        sources:
          - serviceAccountToken:
              path: vault-token
              expirationSeconds: 3600
              audience: "https://vault.prod.svc"
```

### 2.3 Upgrading a Kubernetes Cluster (`kubeadm`)

```bash
# Control Plane Node
apt-mark unhold kubeadm && apt-get update && apt-get install -y kubeadm=1.31.1-1.1 && apt-mark hold kubeadm
kubeadm upgrade plan
kubeadm upgrade apply v1.31.1 -y
kubectl drain controlplane --ignore-daemonsets
apt-mark unhold kubelet kubectl && apt-get install -y kubelet=1.31.1-1.1 kubectl=1.31.1-1.1 && apt-mark hold kubelet kubectl
systemctl daemon-reload && systemctl restart kubelet
kubectl uncordon controlplane

# Worker Node (run drain/uncordon from controlplane, kubeadm/apt on worker)
kubectl drain node01 --ignore-daemonsets --delete-emptydir-data
ssh node01 "apt-mark unhold kubeadm kubelet kubectl && apt-get update && apt-get install -y kubeadm=1.31.1-1.1 kubelet=1.31.1-1.1 kubectl=1.31.1-1.1 && apt-mark hold kubeadm kubelet kubectl && kubeadm upgrade node && systemctl daemon-reload && systemctl restart kubelet"
kubectl uncordon node01
```

---

## 3. Domain 3: System Hardening (15%)

### 3.1 Host Port Auditing, Service Removal & UFW Firewall

```bash
# Find open listening ports and the owning PID/binary
ss -tulpn
lsof -i :9090
ps -fp <PID>

# Stop, disable, and mask a rogue systemd service or purge its package
systemctl stop <service-name>
systemctl disable --now <service-name>
systemctl mask <service-name>
apt-get purge -y <package-name>

# UFW Host Firewall on Control Plane
ufw default deny incoming
ufw default allow outgoing
ufw allow 22/tcp
ufw allow 6443/tcp       # kube-apiserver
ufw allow 2379:2380/tcp  # etcd
ufw allow 10250/tcp      # kubelet
ufw allow 10259/tcp      # kube-scheduler
ufw allow 10257/tcp      # kube-controller-manager
ufw allow 30000:32767/tcp # NodePort range
ufw --force enable
ufw status numbered
```

### 3.2 AppArmor Profile Management & Pod Configuration

```bash
# 1. Load an AppArmor profile onto the worker node where the Pod will run!
apparmor_parser -q /etc/apparmor.d/k8s-deny-write
# Put profile into complain mode (instead of enforce):
aa-complain /etc/apparmor.d/k8s-deny-write
# Put profile into enforce mode:
aa-enforce /etc/apparmor.d/k8s-deny-write

# 2. Verify the profile name (note: Kubernetes uses the profile NAME inside the file, NOT the filename!)
aa-status | grep k8s-deny-write
```

Sample AppArmor Profile (`/etc/apparmor.d/k8s-deny-write`):
```text
#include <tunables/global>
profile k8s-deny-write flags=(attach_disconnected,mediate_deleted) {
  #include <abstractions/base>
  file,
  deny /** w,
}
```

Applying AppArmor to a Pod (Kubernetes v1.30+ GA `securityContext` + legacy annotation):
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: apparmor-pod
  # Legacy annotation (pre-v1.30 / exam backwards-compatibility):
  # annotations:
  #   container.apparmor.security.beta.kubernetes.io/app: localhost/k8s-deny-write
spec:
  securityContext:
    appArmorProfile:
      type: Localhost
      localhostProfile: k8s-deny-write
  containers:
    - name: app
      image: busybox:1.36
      command: ["sh", "-c", "sleep 3600"]
```

### 3.3 Seccomp Profiles & Linux Capabilities

```bash
# Copy custom seccomp profile into kubelet's default seccomp root directory on the worker node:
mkdir -p /var/lib/kubelet/seccomp/profiles
cp /root/audit.json /var/lib/kubelet/seccomp/profiles/audit.json
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hardened-syscall-pod
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 10001
    runAsGroup: 10001
    fsGroup: 2000
    seccompProfile:
      type: Localhost # Or RuntimeDefault
      localhostProfile: profiles/audit.json # Relative to /var/lib/kubelet/seccomp/
  containers:
    - name: web
      image: nginxinc/nginx-unprivileged:1.27-alpine
      securityContext:
        allowPrivilegeEscalation: false
        readOnlyRootFilesystem: true
        capabilities:
          drop:
            - ALL
          add:
            - NET_BIND_SERVICE
      volumeMounts:
        - name: tmp-vol
          mountPath: /tmp
  volumes:
    - name: tmp-vol
      emptyDir: {}
```

---

## 4. Domain 4: Minimize Microservice Vulnerabilities (20%)

### 4.1 Pod Security Admission (PSA) Namespace Labels

```bash
# Enforce 'restricted' Pod Security Standard on a namespace
kubectl label namespace prod \
  pod-security.kubernetes.io/enforce=restricted \
  pod-security.kubernetes.io/enforce-version=latest \
  pod-security.kubernetes.io/warn=restricted \
  pod-security.kubernetes.io/audit=restricted --overwrite

# Dry-run test existing workloads in a namespace against restricted profile
kubectl label --dry-run=server --overwrite ns prod pod-security.kubernetes.io/enforce=restricted
```

### 4.2 Container Sandbox Isolation (`RuntimeClass` — gVisor / Kata)

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
  name: sandboxed-pod
spec:
  runtimeClassName: gvisor
  containers:
    - name: untrusted
      image: nginx:1.27-alpine
```

### 4.3 Encrypting Secret Data at Rest (`EncryptionConfiguration`)

```bash
# Generate a 32-byte random key and base64-encode it
head -c 32 /dev/urandom | base64
```

`/etc/kubernetes/enc/enc.yaml`:
```yaml
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
  - resources:
      - secrets
    providers:
      # First provider is used to ENCRYPT new/updated secrets in etcd
      - aescbc:
          keys:
            - name: key1
              secret: <BASE64_ENCODED_32_BYTE_KEY>
      # Fallback identity provider allows READING existing unencrypted secrets
      - identity: {}
```

Add flag and volume mount in `/etc/kubernetes/manifests/kube-apiserver.yaml`:
```yaml
spec:
  containers:
    - name: kube-apiserver
      command:
        - kube-apiserver
        - --encryption-provider-config=/etc/kubernetes/enc/enc.yaml
      volumeMounts:
        - name: enc-vol
          mountPath: /etc/kubernetes/enc
          readOnly: true
  volumes:
    - name: enc-vol
      hostPath:
        path: /etc/kubernetes/enc
        type: DirectoryOrCreate
```

Re-encrypt all existing Secrets & verify in etcd:
```bash
# Re-encrypt all secrets across all namespaces
kubectl get secrets --all-namespaces -o json | kubectl replace -f -

# Verify secret is encrypted in etcd (starts with k8s:enc:aescbc:v1:key1)
ETCDCTL_API=3 etcdctl \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  get /registry/secrets/prod/my-secret | hexdump -C
```

### 4.4 OPA Gatekeeper (`ConstraintTemplate` & `Constraint`)

```yaml
apiVersion: templates.gatekeeper.sh/v1
kind: ConstraintTemplate
metadata:
  name: k8srequiredlabels
spec:
  crd:
    spec:
      names:
        kind: K8sRequiredLabels
      validation:
        openAPIV3Schema:
          type: object
          properties:
            labels:
              type: array
              items:
                type: string
  targets:
    - target: admission.k8s.gatekeeper.sh
      rego: |
        package k8srequiredlabels
        violation[{"msg": msg}] {
          provided := {label | input.review.object.metadata.labels[label]}
          required := {label | label := input.parameters.labels[_]}
          missing := required - provided
          count(missing) > 0
          msg := sprintf("you must provide labels: %v", [missing])
        }
---
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sRequiredLabels
metadata:
  name: ns-must-have-sec-tier
spec:
  match:
    kinds:
      - apiGroups: [""]
        kinds: ["Namespace"]
  parameters:
    labels: ["security-tier"]
```

### 4.5 Pod-to-Pod mTLS with Istio (`PeerAuthentication`)

```yaml
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default-strict-mtls
  namespace: prod
spec:
  mtls:
    mode: STRICT
```

---

## 5. Domain 5: Supply Chain Security (20%)

### 5.1 Hardened Multi-Stage Dockerfile Template

```dockerfile
# Stage 1: Build statically linked binary
FROM golang:1.27-alpine@sha256:b8c9b82affbbb82a020bbcf998dad2ba9516944ac7e883524642a481a9505e9c AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY main.go ./
RUN CGO_ENABLED=0 GOOS=linux go build -ldflags="-w -s" -o /app/server main.go

# Stage 2: Minimal distroless runtime image running as non-root numeric UID
FROM gcr.io/distroless/static-debian12:nonroot@sha256:62abc193a8490b3effb6d9f1b61d14b26f901a379595ff3d21a8e43952ab68b3
WORKDIR /
COPY --from=builder /app/server /server
USER 65532:65532
ENTRYPOINT ["/server"]
```

### 5.2 Trivy Vulnerability Scanning, Static Analysis & SBOMs

```bash
# Scan an image for HIGH and CRITICAL CVEs (skipping unfixed if requested)
trivy image --severity HIGH,CRITICAL nginx:1.19
trivy image --severity CRITICAL --ignore-unfixed python:3.9-slim

# Scan all images running in a namespace
for img in $(kubectl -n prod get pods -o jsonpath='{..image}' | tr -s '[[:space:]]' '\n' | sort -u); do
  echo "=== Scanning $img ==="
  trivy image --quiet --severity CRITICAL "$img"
done

# Generate SPDX and CycloneDX SBOMs using trivy and bom
trivy image --format spdx-json --output /root/sbom.spdx.json alpine:3.19
trivy image --format cyclonedx --output /root/sbom.cdx.json alpine:3.19
bom generate --image alpine:3.19 --format json --output /root/alpine.spdx.json

# Static analysis of Kubernetes manifests
kubesec scan /root/deployment.yaml
conftest test /root/deployment.yaml -p /root/policy/
```

### 5.3 `ImagePolicyWebhook` Admission Configuration

1. `/etc/kubernetes/epconfig/admission_configuration.yaml`:
```yaml
apiVersion: apiserver.config.k8s.io/v1
kind: AdmissionConfiguration
plugins:
  - name: ImagePolicyWebhook
    configuration:
      imagePolicy:
        kubeConfigFile: /etc/kubernetes/epconfig/kubeconfig.yaml
        allowTTL: 50
        denyTTL: 50
        retryBackoff: 500
        defaultAllow: false # Fail-closed if webhook server is unreachable or rejects!
```

2. `/etc/kubernetes/epconfig/kubeconfig.yaml`:
```yaml
apiVersion: v1
kind: Config
clusters:
  - name: image-bouncer-webhook
    cluster:
      certificate-authority: /etc/kubernetes/epconfig/webhook-ca.crt
      server: https://image-bouncer-webhook.default.svc:1323/image_policy
contexts:
  - name: webhook-context
    context:
      cluster: image-bouncer-webhook
      user: api-server
current-context: webhook-context
users:
  - name: api-server
    user:
      client-certificate: /etc/kubernetes/epconfig/apiserver-client.crt
      client-key: /etc/kubernetes/epconfig/apiserver-client.key
```

3. Add flags and volume mount to `/etc/kubernetes/manifests/kube-apiserver.yaml`:
```yaml
      command:
        - kube-apiserver
        - --enable-admission-plugins=NodeRestriction,ImagePolicyWebhook
        - --admission-control-config-file=/etc/kubernetes/epconfig/admission_configuration.yaml
      volumeMounts:
        - name: epconfig
          mountPath: /etc/kubernetes/epconfig
          readOnly: true
  volumes:
    - name: epconfig
      hostPath:
        path: /etc/kubernetes/epconfig
        type: DirectoryOrCreate
```

---

## 6. Domain 6: Monitoring, Logging and Runtime Security (20%)

### 6.1 Kubernetes API Audit Logging (`audit.k8s.io/v1`)

1. `/etc/kubernetes/audit/policy.yaml` (Remember: **First matching rule wins!**):
```yaml
apiVersion: audit.k8s.io/v1
kind: Policy
# Do not log RequestReceived stage globally to avoid duplicate entries
omitStages:
  - "RequestReceived"
rules:
  # 1. Log Secret changes at Metadata level ONLY (never RequestResponse to avoid leaking secret values in logs!)
  - level: Metadata
    resources:
      - group: ""
        resources: ["secrets"]

  # 2. Log Pod & Deployment mutations in 'prod' namespace at RequestResponse level
  - level: RequestResponse
    verbs: ["create", "update", "patch", "delete"]
    namespaces: ["prod"]
    resources:
      - group: ""
        resources: ["pods"]
      - group: "apps"
        resources: ["deployments"]

  # 3. Don't log read-only requests by system:nodes or kube-proxy
  - level: None
    users: ["system:kube-proxy"]
    verbs: ["watch", "get", "list"]

  # 4. Catch-all rule at the very bottom: log everything else at Metadata level
  - level: Metadata
```

2. `kube-apiserver.yaml` flags & mounts (BOTH policy file AND log output directory must be mounted!):
```yaml
      command:
        - kube-apiserver
        - --audit-policy-file=/etc/kubernetes/audit/policy.yaml
        - --audit-log-path=/var/log/kubernetes/audit/audit.log
        - --audit-log-maxage=30
        - --audit-log-maxbackup=10
        - --audit-log-maxsize=100
      volumeMounts:
        - name: audit-policy
          mountPath: /etc/kubernetes/audit/policy.yaml
          readOnly: true
        - name: audit-logs
          mountPath: /var/log/kubernetes/audit
          readOnly: false
  volumes:
    - name: audit-policy
      hostPath:
        path: /etc/kubernetes/audit/policy.yaml
        type: File
    - name: audit-logs
      hostPath:
        path: /var/log/kubernetes/audit
        type: DirectoryOrCreate
```

### 6.2 Falco Runtime Rules (`/etc/falco/falco_rules.local.yaml`)

Always place custom rules or overrides in `/etc/falco/falco_rules.local.yaml`, then run `systemctl restart falco`:

```yaml
- rule: Detect Interactive Shell in Container
  desc: Alert when a shell (sh, bash, ash, zsh) is spawned inside a container
  condition: >
    spawned_process and container and
    proc.name in (sh, bash, ash, zsh)
  output: >
    Shell spawned in container (time=%evt.time user=%user.name
    container_id=%container.id container_name=%container.name
    image=%container.image.repository cmdline=%proc.cmdline)
  priority: WARNING
  tags: [container, shell, mitre_execution]

# Overriding only the output and priority of an existing rule from /etc/falco/falco_rules.yaml:
- rule: Read sensitive file untrusted
  output: >
    Sensitive file opened for reading (user=%user.name command=%proc.cmdline
    file=%fd.name container_id=%container.id image=%container.image.repository)
  priority: CRITICAL
  override:
    output: replace
    priority: replace
```

### 6.3 Cilium Tetragon eBPF Enforcement (`TracingPolicy`)

```yaml
apiVersion: cilium.io/v1alpha1
kind: TracingPolicy
metadata:
  name: block-curl-execution
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
              operator: "Postfix"
              values:
                - "/curl"
                - "/wget"
          matchActions:
            - action: Sigkill
```

### 6.4 Live Container Incident Forensics

```bash
# 1. Locate the worker node and container ID of the suspicious pod
kubectl get pod -n prod compromised-pod -o wide
crictl ps | grep compromised-pod
CID=$(crictl ps --name compromised-pod -q)

# 2. Find the host PID of the container process
PID=$(crictl inspect "$CID" | jq '.info.pid')

# 3. Inspect process filesystem, open files, environment, and network sockets from the host
ls -la /proc/$PID/root/tmp
cat /proc/$PID/environ | tr '\0' '\n'
ls -l /proc/$PID/fd
nsenter -t $PID -n ss -tulpn
```
