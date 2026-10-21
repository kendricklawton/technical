# CKS Full-Length Performance Practice Exam 2

**Time Limit:** 120 Minutes  
**Passing Score:** 67% (Score Target: $\ge 80\%$)  
**Topology:** Multi-Cluster Simulation (`controlplane`, `node01`)  
**Format:** 16 Advanced Performance Tasks Featuring Cilium L7 Policies, OPA Gatekeeper Rego, Istio mTLS, SBOMs, and Tetragon Enforcement

---

### Task 1: Cilium L7 HTTP Network Security Policy (Weight: 7%)
**Context:** `kubectl config use-context k8s-security`  
**Scenario:** Deploy a `CiliumNetworkPolicy` named `l7-api-guard` in namespace `payments`.
Restrict traffic to the `payment-gateway` service: only allow HTTP `POST` requests to path `/v1/charge`. Deny all other HTTP methods and paths.

**Requirements:**
```yaml
apiVersion: cilium.io/v2
kind: CiliumNetworkPolicy
metadata:
  name: l7-api-guard
  namespace: payments
spec:
  endpointSelector:
    matchLabels:
      app: payment-gateway
  ingress:
  - toPorts:
    - ports:
      - port: "8080"
        protocol: TCP
      rules:
        http:
        - method: POST
          path: "/v1/charge"
```

**Verification:**
Test with curl from another pod: `GET /v1/charge` returns HTTP 403 Forbidden; `POST /v1/charge` returns HTTP 200 OK.

---

### Task 2: Gatekeeper ConstraintTemplate & Rego Policy (Weight: 7%)
**Context:** `kubectl config use-context k8s-security`  
**Scenario:** Deploy an OPA Gatekeeper `ConstraintTemplate` named `K8sRequiredLabels` with Rego logic requiring all Namespaces to possess a `cost-center` label. Deploy a matching `Constraint` enforcing this across the cluster.

**Requirements:**
Create the ConstraintTemplate with Rego validation:
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
  targets:
    - target: admission.k8s.gatekeeper.sh
      rego: |
        package k8srequiredlabels
        violation[{"msg": msg}] {
          provided := {label | input.review.object.metadata.labels[label]}
          not provided["cost-center"]
          msg := "Namespace must contain cost-center label"
        }
```

---

### Task 3: Istio mTLS Strict Mode Enforcement (Weight: 6%)
**Context:** `kubectl config use-context k8s-security`  
**Scenario:** Apply a `PeerAuthentication` resource in namespace `finance` named `default` with mTLS mode `STRICT`. Ensure pods without sidecars cannot connect.

**Requirements:**
```yaml
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
  namespace: finance
spec:
  mtls:
    mode: STRICT
```

---

### Task 4: Software Bill of Materials (SBOM) Generation (Weight: 6%)
**Context:** `kubectl config use-context k8s-security`  
**Scenario:** Using `trivy`, generate a complete Software Bill of Materials (SBOM) in SPDX JSON format for container image `nginx:1.25-alpine` and save to `/opt/sbom/nginx.spdx.json`.

**Command:**
```bash
mkdir -p /opt/sbom
trivy image --format spdx-json --output /opt/sbom/nginx.spdx.json nginx:1.25-alpine
```

---

### Task 5: Recover Broken Control Plane from Malformed Audit Policy (Weight: 7%)
**Context:** `kubectl config use-context k8s-security`  
**Scenario:** The Kubernetes API server static pod is crashing in a loop after a junior engineer edited `/etc/kubernetes/manifests/kube-apiserver.yaml`. Diagnose the error using `crictl`, fix the syntax error in the audit configuration, and bring the API server back online.

**Steps:**
1. Check stopped containers: `crictl ps -a | grep apiserver`.
2. Inspect logs: `crictl logs <container-id>`.
3. Locate typo in `--audit-policy-file` path in `/etc/kubernetes/manifests/kube-apiserver.yaml`.
4. Fix typo and verify `kubectl get nodes` returns Ready.

---

### Task 6: Audit Kubeconfig Credentials & Rotate Compromised Keys (Weight: 6%)
**Context:** `kubectl config use-context k8s-security`  
**Scenario:** User `contractor-dev` had their client certificate compromised. Revoke access immediately by removing the user's `ClusterRoleBinding` and `RoleBinding` across the cluster.

---

### Task 7: Kernel Hardening with sysctl Parameters (Weight: 6%)
**Context:** `ssh node01`  
**Scenario:** On `node01`, configure Linux kernel parameters to disable IPv4 ICMP redirects (`net.ipv4.conf.all.accept_redirects = 0`), enable SYN cookies (`net.ipv4.tcp_syncookies = 1`), and reload via `sysctl -p`.

---

### Task 8: gVisor Sandboxed Container Deployment (Weight: 6%)
**Context:** `kubectl config use-context k8s-security`  
**Scenario:** Configure a `RuntimeClass` named `gvisor` with handler `runsc`. Deploy an untrusted pod `untrusted-calc` in namespace `sandbox` using `runtimeClassName: gvisor`.

**Requirements:**
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
  name: untrusted-calc
  namespace: sandbox
spec:
  runtimeClassName: gvisor
  containers:
  - name: calc
    image: busybox:1.36
    command: ["sh", "-c", "sleep 3600"]
```

---

### Task 9: Ingress TLS Hardening with Modern Ciphers (Weight: 6%)
**Context:** `kubectl config use-context k8s-security`  
**Scenario:** Update the NGINX Ingress controller configuration ConfigMap in namespace `ingress-nginx` to enforce TLS 1.3 only, disabling all legacy TLS 1.0 and 1.1 ciphers.

---

### Task 10: Secret Rotation Drill (Weight: 6%)
**Context:** `kubectl config use-context k8s-security`  
**Scenario:** Rotate the database password in secret `db-creds` in namespace `prod` from `OldPass123` to `NewSecretPass456!`. Perform a zero-downtime rolling restart of deployment `backend-api` to pick up the new secret.

---

### Task 11: Static Analysis of Kubernetes Manifests with Kubesec (Weight: 6%)
**Context:** `kubectl config use-context k8s-security`  
**Scenario:** Run `kubesec scan /opt/manifests/app.yaml`. Review the security score, remediate all reported critical vulnerabilities (privileged mode, root user, missing limits), and achieve a passing score $> 5$.

---

### Task 12: Ephemeral Debugging Container Forensics (Weight: 6%)
**Context:** `kubectl config use-context k8s-security`  
**Scenario:** Attach an ephemeral debug container with image `busybox` to a crashed, non-restarting pod `failing-worker` using `kubectl debug -it failing-worker --image=busybox --target=worker`. Inspect memory and write findings to `/tmp/debug-report.txt`.

---

### Task 13: Limit Worker Node Network Attack Surface (Weight: 6%)
**Context:** `ssh node01`  
**Scenario:** Ensure port 10250 is accessible ONLY from the control plane IP `192.168.1.10`, blocking all other node-to-node direct access to port 10250 using `iptables`.

---

### Task 14: Restrict Mounting of Sensitive Host Paths (Weight: 6%)
**Context:** `kubectl config use-context k8s-security`  
**Scenario:** Audit all pods in namespace `kube-system`. Any non-system pod mounting `/var/run/docker.sock` or `/etc/shadow` via `hostPath` must be terminated and reported.

---

### Task 15: Image Vulnerability Whitelisting (Weight: 6%)
**Context:** `kubectl config use-context k8s-security`  
**Scenario:** Configure Trivy with a `.trivyignore` file at `/opt/.trivyignore` to suppress approved false-positive CVE `CVE-2023-99999` while flagging all other High and Critical CVEs.

---

### Task 16: Supply Chain Multi-Stage Dockerfile Hardening (Weight: 6%)
**Context:** `kubectl config use-context k8s-security`  
**Scenario:** Refactor a bloated root-based Dockerfile at `/opt/build/Dockerfile` into a hardened multi-stage build using Google Distroless (`gcr.io/distroless/static:nonroot`) with `USER 65532:65532`.
