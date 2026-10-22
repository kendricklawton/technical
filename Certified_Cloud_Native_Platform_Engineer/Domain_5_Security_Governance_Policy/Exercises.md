# Domain 5: Security, Governance & Policy Enforcement — Performance Labs & Drills

---

## Lab 1: Enforce Read-Only Root Filesystem with Kyverno ClusterPolicy

### Objective
Create a Kyverno `ClusterPolicy` in `Enforce` mode that mandates every container in non-system namespaces must specify `readOnlyRootFilesystem: true` within its `securityContext`.

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: require-readonly-rootfs
  annotations:
    policies.kyverno.io/category: Pod Security Standards (Restricted)
    policies.kyverno.io/severity: medium
spec:
  validationFailureAction: Enforce
  background: true
  rules:
    - name: check-readonly-rootfs
      match:
        any:
          - resources:
              kinds:
                - Pod
              namespaces:
                - "!kube-system"
                - "!kube-public"
                - "!kyverno"
      validate:
        message: "Root filesystem must be mounted read-only (readOnlyRootFilesystem: true). Use emptyDir volumes for writable scratch directories."
        pattern:
          spec:
            containers:
              - securityContext:
                  readOnlyRootFilesystem: true
```

### Verification
```bash
kubectl apply -f require-readonly-rootfs.yaml
# Test non-compliant pod (must be blocked)
kubectl run bad-pod --image=nginx --restart=Never
# Test compliant pod (must succeed)
kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: good-pod
spec:
  containers:
    - name: app
      image: busybox:1.36
      command: ["sh", "-c", "sleep 3600"]
      securityContext:
        readOnlyRootFilesystem: true
EOF
```

---

## Lab 2: Synchronize Cloud Secrets using External Secrets Operator (ESO)

### Objective
Configure an `ExternalSecret` resource that pulls a secret key from an existing `ClusterSecretStore` and generates a native Kubernetes Secret.

```yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: api-gateway-credentials
  namespace: default
spec:
  refreshInterval: "15m"
  secretStoreRef:
    name: vault-backend
    kind: ClusterSecretStore
  target:
    name: synced-gateway-secret
    creationPolicy: Owner
  data:
    - secretKey: CLIENT_ID
      remoteRef:
        key: kv/production/apigw
        property: clientId
    - secretKey: CLIENT_SECRET
      remoteRef:
        key: kv/production/apigw
        property: clientSecret
```

### Verification
```bash
kubectl apply -f external-secret.yaml
kubectl get externalsecrets -n default
kubectl describe externalsecret api-gateway-credentials -n default
kubectl get secret synced-gateway-secret -n default -o yaml
```

---

## Lab 3: Cryptographic Image Signature Verification Policy

### Objective
Enforce a Kyverno image verification rule requiring all images from `ghcr.io/my-org/*` to be cryptographically signed by Cosign using a public key stored in a Kubernetes Secret.

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: verify-cosign-signature
spec:
  validationFailureAction: Enforce
  webhookTimeoutSeconds: 30
  rules:
    - name: verify-image-sig
      match:
        any:
          - resources:
              kinds:
                - Pod
      verifyImages:
        - imageReferences:
            - "ghcr.io/my-org/*"
          attestors:
            - entries:
                - keys:
                    secret:
                      name: cosign-public-key
                      namespace: kyverno
```

### Verification
```bash
kubectl apply -f verify-image-sig.yaml
kubectl get clusterpolicies
```
