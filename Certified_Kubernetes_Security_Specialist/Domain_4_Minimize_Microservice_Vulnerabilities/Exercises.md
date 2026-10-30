# Domain 4: Minimize Microservice Vulnerabilities — Production Hands-On Labs & CKS Workbook

**Domain Weight:** 20% of the CKS Exam  
**Format:** 6 Performance-Based Security Labs with End-to-End Manifests, Control-Plane Configs, and Verification Steps.

---

## Lab 1: Encrypting Kubernetes `Secrets` at Rest in `etcd` (`EncryptionConfiguration`) & Key Rotation

### 1.1 Scenario
Currently, `Secrets` in the cluster are stored in plaintext (`identity: {}`) inside `etcd`. You must:
1. Generate a 32-byte random key and encode it in base64.
2. Create `/etc/kubernetes/enc/encryption-config.yaml` using `aescbc` (key `key-2026-10`) as the first provider and `identity: {}` as the fallback provider.
3. Configure `kube-apiserver` to use `/etc/kubernetes/enc/encryption-config.yaml` and mount `/etc/kubernetes/enc` read-only into the static pod container.
4. Verify a new Secret `payment-key` is stored encrypted in `etcd` with prefix `k8s:enc:aescbc:v1:key-2026-10:`, and re-encrypt all existing Secrets across all namespaces.

### 1.2 Execution Steps
```bash
# Step 1: Create directory and generate 32-byte base64 key on control-plane node
sudo mkdir -p /etc/kubernetes/enc
ENC_KEY=$(head -c 32 /dev/urandom | base64)

# Step 2: Write EncryptionConfiguration manifest with restricted 0600 permissions
cat << EOF | sudo tee /etc/kubernetes/enc/encryption-config.yaml
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
  - resources:
      - secrets
    providers:
      - aescbc:
          keys:
            - name: key-2026-10
              secret: ${ENC_KEY}
      - identity: {}
EOF
sudo chmod 0600 /etc/kubernetes/enc/encryption-config.yaml
```

Update `/etc/kubernetes/manifests/kube-apiserver.yaml`:
1. Under `spec.containers[0].command`, add:
   ```yaml
   - --encryption-provider-config=/etc/kubernetes/enc/encryption-config.yaml
   ```
2. Under `spec.containers[0].volumeMounts`, add:
   ```yaml
   - mountPath: /etc/kubernetes/enc
     name: enc-config
     readOnly: true
   ```
3. Under `spec.volumes`, add:
   ```yaml
   - hostPath:
       path: /etc/kubernetes/enc
       type: DirectoryOrCreate
     name: enc-config
   ```

### 1.3 Verification & Full Cluster Re-Encryption
```bash
# Wait for kube-apiserver to restart and become Ready
kubectl get --raw='/readyz'

# Create a test Secret and inspect raw etcd bytes via etcdctl
kubectl create secret generic payment-key -n default --from-literal=api_key=super_secret_token_99

sudo ETCDCTL_API=3 etcdctl \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  get /registry/secrets/default/payment-key | hexdump -C | head -n 10
# Confirm output shows: k8s:enc:aescbc:v1:key-2026-10:

# Re-encrypt all pre-existing Secrets in the cluster
kubectl get secrets --all-namespaces -o json | kubectl replace -f -
```

---

## Lab 2: Enforcing Pod Security Admission (`restricted`) & Remediating Non-Compliant Deployments

### 2.1 Scenario
You must enforce the `restricted` Pod Security Standard on namespace `pci-vault`. Currently, Deployment `vault-processor` in `pci-vault` fails `restricted` admission validation when scaled or restarted. You must:
1. Pre-test the namespace against `restricted` using `--dry-run=server`.
2. Apply `enforce=restricted`, `warn=restricted`, and `audit=restricted` labels to `pci-vault`.
3. Remediate the `vault-processor` Deployment template so its ReplicaSet successfully creates compliant Pods.

### 2.2 Execution Steps
```bash
# Step 1: Dry-run label to inspect violations on existing pods
kubectl label --dry-run=server --overwrite ns pci-vault \
  pod-security.kubernetes.io/enforce=restricted \
  pod-security.kubernetes.io/enforce-version=latest

# Step 2: Apply PSA labels to namespace pci-vault
kubectl label --overwrite ns pci-vault \
  pod-security.kubernetes.io/enforce=restricted \
  pod-security.kubernetes.io/enforce-version=latest \
  pod-security.kubernetes.io/warn=restricted \
  pod-security.kubernetes.io/audit=restricted
```

Update Deployment `vault-processor` in `pci-vault` to satisfy all four `restricted` requirements (`runAsNonRoot`, `seccompProfile`, `allowPrivilegeEscalation: false`, `capabilities.drop: ["ALL"]`) and remove any forbidden `hostPath` volumes:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: vault-processor
  namespace: pci-vault
spec:
  replicas: 2
  selector:
    matchLabels:
      app: vault-processor
  template:
    metadata:
      labels:
        app: vault-processor
    spec:
      automountServiceAccountToken: false
      securityContext:
        runAsNonRoot: true
        runAsUser: 65532
        runAsGroup: 65532
        fsGroup: 65532
        seccompProfile:
          type: RuntimeDefault
      containers:
      - name: processor
        image: busybox:1.36
        command: ["sh", "-c", "sleep 3600"]
        securityContext:
          allowPrivilegeEscalation: false
          readOnlyRootFilesystem: true
          capabilities:
            drop:
              - ALL
        volumeMounts:
        - name: scratch
          mountPath: /tmp
      volumes:
      - name: scratch
        emptyDir:
          medium: Memory
          sizeLimit: 64Mi
```

### 2.3 Verification
```bash
kubectl rollout status deploy/vault-processor -n pci-vault
kubectl get pods -n pci-vault
```

---

## Lab 3: Container Sandboxing with `RuntimeClass` (gVisor `runsc` & Kata Containers)

### 3.1 Scenario
Worker node `worker-01` has been labeled `sandbox.security.io/runtime=gvisor` and has `containerd` configured with the `runsc` handler. You must:
1. Create a `RuntimeClass` named `gvisor-sandbox` that maps to handler `runsc`, schedules pods exclusively onto nodes labeled `sandbox.security.io/runtime=gvisor`, and accounts for `64Mi` memory and `100m` CPU fixed pod overhead.
2. Migrate all Pods in namespace `untrusted-tenants` to execute inside `gvisor-sandbox`.

### 3.2 Execution Steps
```yaml
apiVersion: node.k8s.io/v1
kind: RuntimeClass
metadata:
  name: gvisor-sandbox
handler: runsc
overhead:
  podFixed:
    memory: "64Mi"
    cpu: "100m"
scheduling:
  nodeSelector:
    sandbox.security.io/runtime: gvisor
  tolerations:
  - key: sandbox
    operator: Equal
    value: gvisor
    effect: NoSchedule
```

Patch Deployments in `untrusted-tenants` to use `runtimeClassName: gvisor-sandbox`:
```bash
kubectl patch deploy webhook-runner -n untrusted-tenants \
  -p '{"spec":{"template":{"spec":{"runtimeClassName":"gvisor-sandbox"}}}}'
```

### 3.3 Verification
```bash
# Confirm Pod is running inside the gVisor Sentry kernel rather than the host Linux kernel
kubectl exec -n untrusted-tenants deploy/webhook-runner -- dmesg | head -n 5
# Expected output: [ 0.000000] Starting gVisor...
```

---

## Lab 4: Custom Admission Policy with OPA Gatekeeper (`ConstraintTemplate` & `Constraint`)

### 4.1 Scenario
Using OPA Gatekeeper, enforce a cluster policy that blocks any Pod in namespaces labeled `environment=production` if:
1. Any container image uses the `:latest` tag (or omits a tag), OR
2. Any container omits `resources.limits.memory`.

### 4.2 Execution Steps
```yaml
apiVersion: templates.gatekeeper.sh/v1
kind: ConstraintTemplate
metadata:
  name: k8sprodcontainerstandards
spec:
  crd:
    spec:
      names:
        kind: K8sProdContainerStandards
  targets:
    - target: admission.k8s.gatekeeper.sh
      rego: |
        package k8sprodcontainerstandards

        violation[{"msg": msg}] {
          container := input.review.object.spec.containers[_]
          endswith(container.image, ":latest")
          msg := sprintf("Container %v uses forbidden :latest tag (%v)", [container.name, container.image])
        }

        violation[{"msg": msg}] {
          container := input.review.object.spec.containers[_]
          not contains(container.image, ":")
          msg := sprintf("Container %v must specify an explicit version tag (%v)", [container.name, container.image])
        }

        violation[{"msg": msg}] {
          container := input.review.object.spec.containers[_]
          not container.resources.limits.memory
          msg := sprintf("Container %v is missing required resources.limits.memory", [container.name])
        }
---
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sProdContainerStandards
metadata:
  name: prod-container-standards
spec:
  enforcementAction: deny
  match:
    kinds:
      - apiGroups: [""]
        kinds: ["Pod"]
    namespaceSelector:
      matchLabels:
        environment: production
```

### 4.3 Verification
```bash
# Attempt to create a Pod with :latest in a production namespace
kubectl run test-bad -n prod --image=nginx:latest
# Expected: Error from server (Forbidden): admission webhook "validation.gatekeeper.sh" denied the request: [prod-container-standards] Container test-bad uses forbidden :latest tag (nginx:latest)
```

---

## Lab 5: Native Kubernetes CEL `ValidatingAdmissionPolicy` (Zero-Webhook Admission Control)

### 5.1 Scenario
Without deploying any external webhook servers, configure a native Kubernetes `ValidatingAdmissionPolicy` (`admissionregistration.k8s.io/v1`) named `deny-hostpath-and-root` bound to namespace `tenant-a` that rejects any Pod that defines `spec.volumes[].hostPath` or fails to set `automountServiceAccountToken: false`.

### 5.2 Execution Steps
```yaml
apiVersion: admissionregistration.k8s.io/v1
kind: ValidatingAdmissionPolicy
metadata:
  name: deny-hostpath-and-sa-token
spec:
  failurePolicy: Fail
  matchConstraints:
    resourceRules:
    - apiGroups:   [""]
      apiVersions: ["v1"]
      operations:  ["CREATE", "UPDATE"]
      resources:   ["pods"]
  validations:
    - expression: "!has(object.spec.volumes) || object.spec.volumes.all(v, !has(v.hostPath))"
      message: "Mounting hostPath volumes is strictly prohibited by security policy."
    - expression: "has(object.spec.automountServiceAccountToken) && object.spec.automountServiceAccountToken == false"
      message: "Pods must explicitly set spec.automountServiceAccountToken: false."
---
apiVersion: admissionregistration.k8s.io/v1
kind: ValidatingAdmissionPolicyBinding
metadata:
  name: deny-hostpath-and-sa-token-binding
spec:
  policyName: deny-hostpath-and-sa-token
  validationActions: [Deny]
  matchResources:
    namespaceSelector:
      matchLabels:
        kubernetes.io/metadata.name: tenant-a
```

### 5.3 Verification
```bash
kubectl run bad-pod -n tenant-a --image=busybox:1.36 -- sleep 3600
# Expected: Forbidden: ValidatingAdmissionPolicy 'deny-hostpath-and-sa-token' with binding 'deny-hostpath-and-sa-token-binding' denied request: Pods must explicitly set spec.automountServiceAccountToken: false.
```

---

## Lab 6: Zero-Trust Pod-to-Pod Mutual TLS (mTLS) with Istio `PeerAuthentication` & `AuthorizationPolicy`

### 6.1 Scenario
Namespace `banking` runs microservices with Istio sidecar injection enabled (`istio-injection=enabled`). You must:
1. Enforce strict mutual TLS (`STRICT`) for all workloads in `banking` so any plaintext connection from non-mesh pods is rejected.
2. Create an L7 `AuthorizationPolicy` allowing only Pods using ServiceAccount `cluster.local/ns/banking/sa/checkout-sa` to send `POST` requests to path `/api/v1/charge` on `payment-service` Pods (`app: payment-service`).

### 6.2 Execution Steps
```yaml
apiVersion: security.istio.io/v1
kind: PeerAuthentication
metadata:
  name: default-strict-mtls
  namespace: banking
spec:
  mtls:
    mode: STRICT
---
apiVersion: security.istio.io/v1
kind: AuthorizationPolicy
metadata:
  name: payment-service-rbac
  namespace: banking
spec:
  selector:
    matchLabels:
      app: payment-service
  action: ALLOW
  rules:
  - from:
    - source:
        principals: ["cluster.local/ns/banking/sa/checkout-sa"]
    to:
    - operation:
        methods: ["POST"]
        paths: ["/api/v1/charge"]
```

### 6.3 Verification
```bash
# 1. Verify plaintext curl from a non-mesh namespace fails TLS handshake
kubectl run non-mesh-test -n default --image=curlimages/curl --rm -it --restart=Never -- \
  curl -v http://payment-service.banking.svc.cluster.local:8080/api/v1/charge
# Expected: Recv failure: Connection reset by peer

# 2. Verify mesh pod with checkout-sa succeeds on POST /api/v1/charge but gets 403 RBAC: access denied on DELETE
kubectl exec -n banking deploy/checkout -- curl -i -X DELETE http://payment-service.banking.svc.cluster.local:8080/api/v1/charge
# Expected: HTTP/1.1 403 Forbidden (RBAC: access denied)
```
