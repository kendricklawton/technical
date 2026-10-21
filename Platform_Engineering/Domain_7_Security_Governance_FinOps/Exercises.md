# PE Domain 7: Security, Governance & FinOps — Hands-On Labs

Six production-grade labs drilling External Secrets Operator, Kyverno security policies, EKS Pod Identity, Istio mTLS, and OpenCost financial optimization.

---

## Lab 1: Configuring External Secrets Operator (ESO)

### Objective
Deploy and configure the External Secrets Operator in Kubernetes to fetch a sensitive database password from an external mock secret store and generate a native Kubernetes Secret dynamically.

### Step-by-Step Implementation

1. **Define the ClusterSecretStore (`cluster-secret-store.yaml`):**
   ```yaml
   apiVersion: external-secrets.io/v1beta1
   kind: ClusterSecretStore
   metadata:
     name: aws-secrets-manager
   spec:
     provider:
       aws:
         service: SecretsManager
         region: us-east-1
         auth:
           jwt:
             serviceAccountRef:
               name: eso-sa
               namespace: external-secrets-system
   ```

2. **Define the Application ExternalSecret (`external-secret.yaml`):**
   ```yaml
   apiVersion: external-secrets.io/v1beta1
   kind: ExternalSecret
   metadata:
     name: database-credentials
     namespace: default
   spec:
     refreshInterval: 1h
     secretStoreRef:
       name: aws-secrets-manager
       kind: ClusterSecretStore
     target:
       name: app-db-secret
       creationPolicy: Owner
     data:
       - secretKey: password
         remoteRef:
           key: production/database/master
           property: password
   ```

---

## Lab 2: Enforcing Zero-Trust Pod Security with Kyverno

### Objective
Deploy a Kyverno `ClusterPolicy` that blocks any container from mounting host paths, running as root, or lacking CPU/memory limits.

### Implementation Manifest (`kyverno-security-policy.yaml`)
```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: enforce-pod-hardening
spec:
  validationFailureAction: Enforce
  background: true
  rules:
    - name: block-root-execution
      match:
        any:
          - resources:
              kinds: [Pod]
      validate:
        message: "Running as root is forbidden. Set securityContext.runAsNonRoot: true."
        pattern:
          spec:
            containers:
              - securityContext:
                  runAsNonRoot: true

    - name: block-hostpath-mounts
      match:
        any:
          - resources:
              kinds: [Pod]
      validate:
        message: "hostPath volumes are strictly forbidden on this platform."
        pattern:
          spec:
            =(volumes):
              - X(hostPath): "null"
```

---

## Lab 3: Configuring EKS Pod Identity for AWS IAM Authentication

### Objective
Create a Kubernetes ServiceAccount and link it to an AWS IAM Role using EKS Pod Identity Association.

### Step-by-Step Implementation

1. **Create Kubernetes ServiceAccount (`sa.yaml`):**
   ```yaml
   apiVersion: v1
   kind: ServiceAccount
   metadata:
     name: payment-processor-sa
     namespace: default
   ```

2. **Associate with AWS IAM Role using AWS CLI:**
   ```bash
   aws eks create-pod-identity-association      --cluster-name platform-prod      --namespace default      --service-account payment-processor-sa      --role-arn arn:aws:iam::123456789012:role/PaymentS3AccessRole
   ```

3. **Verify Inside a Test Pod:**
   Inside the pod, query the AWS container credentials endpoint:
   ```bash
   curl $AWS_CONTAINER_CREDENTIALS_FULL_URI
   ```
   Observe the temporary security credentials provided directly by the local agent!

---

## Lab 4: Enforcing Strict mTLS and L7 Authorization with Istio

### Objective
Configure Istio `PeerAuthentication` to enforce STRICT mutual TLS across a namespace and author an `AuthorizationPolicy` restricting access to `/v1/admin` routes.

### Implementation Manifest (`istio-security.yaml`)
```yaml
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
  namespace: checkout
spec:
  mtls:
    mode: STRICT
---
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: restrict-admin-route
  namespace: checkout
spec:
  selector:
    matchLabels:
      app: checkout-service
  action: ALLOW
  rules:
    - from:
        - source:
            principals: ["cluster.local/ns/platform-system/sa/admin-gateway-sa"]
      to:
        - operation:
            paths: ["/v1/admin/*"]
            methods: ["POST", "DELETE"]
```

---

## Lab 5: Deploying OpenCost and Tracking Pod Cost Allocation

### Objective
Install OpenCost in a Kubernetes cluster, configure cloud provider spot/on-demand pricing rates, and query the OpenCost API to calculate hourly cost per microservice namespace.

### Step-by-Step Implementation

1. **Deploy OpenCost via Helm:**
   ```bash
   helm repo add opencost https://opencost.github.io/opencost-helm-chart
   helm repo update
   helm install opencost opencost/opencost --namespace opencost --create-namespace
   ```

2. **Query Real-time Namespace Cost Allocation:**
   ```bash
   kubectl port-forward --namespace opencost service/opencost 9003:9003 &
   curl -s "http://localhost:9003/allocation/compute?window=1d&aggregate=namespace" | jq .
   ```

---

## Lab 6: Building an Automated SOC 2 Compliance Audit Script

### Objective
Write an automated Python audit script that queries the Kubernetes cluster for security compliance indicators (non-root pods, default deny network policies, encrypted secrets) and outputs a formatted compliance report for SOC 2 auditors.

### Implementation (`soc2_audit.py`)
```python
#!/usr/bin/env python3
import json
import subprocess
import sys

def audit_pod_security():
    cmd = ["kubectl", "get", "pods", "-A", "-o", "json"]
    res = subprocess.run(cmd, capture_output=True, text=True)
    if res.returncode != 0:
        print("Failed to query Kubernetes pods")
        sys.exit(1)

    data = json.loads(res.stdout)
    compliant = 0
    non_compliant = 0

    for pod in data.get("items", []):
        name = pod["metadata"]["name"]
        ns = pod["metadata"]["namespace"]
        if ns in ["kube-system", "kube-node-lease"]:
            continue

        pod_safe = True
        for c in pod["spec"].get("containers", []):
            sc = c.get("securityContext", {})
            if not sc.get("runAsNonRoot", False) or sc.get("privileged", False):
                pod_safe = False
                break

        if pod_safe:
            compliant += 1
        else:
            non_compliant += 1
            print(f"[FAIL] Non-compliant pod: {ns}/{name}")

    total = compliant + non_compliant
    score = (compliant / total * 100) if total > 0 else 100
    print(f"\n--- SOC 2 Pod Hardening Audit ---")
    print(f"Compliant: {compliant} | Non-Compliant: {non_compliant}")
    print(f"Compliance Score: {score:.1f}%")

if __name__ == "__main__":
    audit_pod_security()
```
