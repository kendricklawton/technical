# Domain 4: Minimize Microservice Vulnerabilities — Hands-On Exercises & Solutions

## Exercise 1: Configure etcd Secret Encryption at Rest (End-to-End)
### Scenario
The security auditor mandates that all Kubernetes secrets in etcd must be encrypted with AES-CBC.

### Step 1: Generate 32-Byte Base64 Key
```bash
head -c 32 /dev/urandom | base64
# Example output: yQv1Z...==
```

### Step 2: Create Encryption Configuration (`/etc/kubernetes/enc/enc.yaml`)
```yaml
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
  - resources:
      - secrets
    providers:
      - aescbc:
          keys:
            - name: key1
              secret: yQv1Z09f87KlpMNbvCxZaWq1234567890abcdefgh=
      - identity: {}
```

### Step 3: Update `kube-apiserver.yaml`
Add flag `--encryption-provider-config=/etc/kubernetes/enc/enc.yaml` and hostPath mounts.

### Step 4: Re-Encrypt Existing Secrets
```bash
kubectl get secrets -A -o json | kubectl replace -f -
```

---

## Exercise 2: Enforce Pod Security Standards (Restricted) on a Namespace
```bash
kubectl create ns secure-finance
kubectl label ns secure-finance   pod-security.kubernetes.io/enforce=restricted   pod-security.kubernetes.io/enforce-version=latest   pod-security.kubernetes.io/warn=restricted   pod-security.kubernetes.io/audit=restricted
```

---

## Exercise 3: Deploy Workload in gVisor Sandboxed Runtime (`runsc`)
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
  name: secure-sandbox
  namespace: default
spec:
  runtimeClassName: gvisor
  containers:
  - name: untrusted-app
    image: nginx:alpine
```
