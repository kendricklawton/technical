# CNPE Full-Length Performance Practice Exam 2

**Time Limit:** 120 Minutes  
**Passing Score:** 70% (Score Target: $\ge 80\%$)  
**Topology:** Kubernetes v1.31+, Argo CD, Argo Rollouts, Crossplane, Kyverno, ESO, OpenTelemetry, KEDA  
**Format:** 15 Real-World Hands-On Platform Engineering Tasks with Full Solution Manifests & Verification Commands

---

### Task 1: Crossplane Composition with Managed S3 Bucket (Weight: 7%)
**Scenario:** Build a Crossplane Composition that binds the composite resource `XObjectStorage` to an AWS S3 `Bucket` managed resource.
**Requirements:**
1. Define a `Composition` named `s3-storage-composition`.
2. Composite Type Ref: group `storage.platform.io`, kind `XObjectStorage`.
3. Provision managed resource `Bucket` from `s3.aws.upbound.io/v1beta1`.
4. Patch `spec.bucketName` from the composite resource to the S3 bucket's external name.

**Solution:**
```yaml
apiVersion: apiextensions.crossplane.io/v1
kind: Composition
metadata:
  name: s3-storage-composition
  labels:
    provider: aws
spec:
  compositeTypeRef:
    apiVersion: storage.platform.io/v1alpha1
    kind: XObjectStorage
  resources:
    - name: s3-bucket
      base:
        apiVersion: s3.aws.upbound.io/v1beta1
        kind: Bucket
        spec:
          forProvider:
            region: us-east-1
      patches:
        - type: FromCompositeFieldPath
          fromFieldPath: spec.bucketName
          toFieldPath: metadata.annotations[crossplane.io/external-name]
```

---

### Task 2: Argo Rollouts Blue-Green Deployment with Pre-Promotion Analysis (Weight: 7%)
**Scenario:** Deploy a Blue-Green Rollout for `inventory-service` in namespace `inventory`.
**Requirements:**
1. Active Service: `inventory-active`, Preview Service: `inventory-preview`.
2. Configure `autoPromotionEnabled: false` so that human approval or automated analysis is required before promotion.
3. Add a pre-promotion analysis template `smoke-test` executing an HTTP synthetic check against `inventory-preview`.

**Solution:**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: inventory-service
  namespace: inventory
spec:
  replicas: 3
  strategy:
    blueGreen:
      activeService: inventory-active
      previewService: inventory-preview
      autoPromotionEnabled: false
      prePromotionAnalysis:
        templates:
          - templateName: smoke-test
  selector:
    matchLabels:
      app: inventory-service
  template:
    metadata:
      labels:
        app: inventory-service
    spec:
      containers:
        - name: app
          image: ghcr.io/enterprise/inventory:v3.0.0
          ports:
            - containerPort: 8080
---
apiVersion: argoproj.io/v1alpha1
kind: AnalysisTemplate
metadata:
  name: smoke-test
  namespace: inventory
spec:
  metrics:
    - name: http-health
      interval: 10s
      count: 3
      successCondition: result == "200"
      provider:
        web:
          url: "http://inventory-preview.inventory.svc:8080/healthz"
```

---

### Task 3: Kyverno Mutating Policy Injecting Security Context (Weight: 7%)
**Scenario:** Automatically inject a hardened `securityContext` (`runAsNonRoot: true`, `allowPrivilegeEscalation: false`) into all pods deployed in namespace `staging` if the developer omitted them.

**Solution:**
```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: mutate-pod-security
spec:
  rules:
    - name: inject-security-context
      match:
        any:
          - resources:
              kinds: [Pod]
              namespaces: [staging]
      mutate:
        patchStrategicMerge:
          spec:
            securityContext:
              +(runAsNonRoot): true
            containers:
              - (name): "*"
                securityContext:
                  +(allowPrivilegeEscalation): false
```

---

### Task 4: KEDA ScaledJob for RabbitMQ Batch Workers (Weight: 7%)
**Scenario:** Configure a KEDA `ScaledJob` that triggers batch jobs to process image resize tasks from RabbitMQ queue `resize-tasks`.
**Requirements:**
1. Scale up to a maximum of 10 concurrent Jobs.
2. Queue threshold: 1 Job per 10 messages.

**Solution:**
```yaml
apiVersion: keda.sh/v1alpha1
kind: ScaledJob
metadata:
  name: image-resizer-scaler
  namespace: media
spec:
  jobTargetRef:
    template:
      spec:
        restartPolicy: OnFailure
        containers:
          - name: resizer
            image: enterprise/resizer:v1.0
            command: ["process-batch"]
  maxReplicaCount: 10
  successfulJobsHistoryLimit: 5
  failedJobsHistoryLimit: 5
  triggers:
    - type: rabbitmq
      metadata:
        protocol: auto
        queueName: resize-tasks
        mode: QueueLength
        value: "10"
      authenticationRef:
        name: rabbitmq-auth
```

---

### Task 5: External Secrets Operator Multi-Tenant SecretStore (Weight: 6%)
**Scenario:** Create a namespace-scoped `SecretStore` in namespace `logistics` that connects to AWS Secrets Manager using IAM Roles for Service Accounts (IRSA).

**Solution:**
```yaml
apiVersion: external-secrets.io/v1beta1
kind: SecretStore
metadata:
  name: aws-secrets-store
  namespace: logistics
spec:
  provider:
    aws:
      service: SecretsManager
      region: us-east-1
      auth:
        jwt:
          serviceAccountRef:
            name: logistics-secrets-sa
```

---

### Task 6: Argo CD Project Multi-Tenant Guardrails (Weight: 7%)
**Scenario:** Create an `AppProject` named `team-analytics` restricting deployments strictly to namespace `analytics` on the in-cluster destination and whitelisting only git repository `https://github.com/enterprise/analytics-apps.git`.

**Solution:**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: team-analytics
  namespace: argocd
spec:
  description: "Restricted project for analytics workloads"
  sourceRepos:
    - "https://github.com/enterprise/analytics-apps.git"
  destinations:
    - namespace: analytics
      server: https://kubernetes.default.svc
  clusterResourceWhitelist: []
  namespaceResourceWhitelist:
    - group: "apps"
      kind: "Deployment"
    - group: ""
      kind: "Service"
    - group: ""
      kind: "ConfigMap"
```

---

### Task 7: OpenTelemetry Collector Spanmetrics RED Telemetry (Weight: 7%)
**Scenario:** Enable the `spanmetrics` connector in the OTel Collector pipeline so that trace spans automatically generate Prometheus request rate, error rate, and duration metrics.

**Solution:**
```yaml
apiVersion: opentelemetry.io/v1alpha1
kind: OpenTelemetryCollector
metadata:
  name: otel-spanmetrics
  namespace: observability
spec:
  config: |
    receivers:
      otlp:
        protocols:
          grpc:
            endpoint: 0.0.0.0:4317
    connectors:
      spanmetrics:
        histogram:
          explicit:
            buckets: [10ms, 50ms, 100ms, 250ms, 500ms, 1s, 5s]
        dimensions:
          - name: http.status_code
          - name: http.method
    exporters:
      prometheus:
        endpoint: 0.0.0.0:8889
      otlp/tempo:
        endpoint: tempo:4317
        tls: { insecure: true }
    service:
      pipelines:
        traces:
          receivers: [otlp]
          exporters: [otlp/tempo, spanmetrics]
        metrics:
          receivers: [spanmetrics]
          exporters: [prometheus]
```

---

### Task 8: Hierarchical Namespace Controller (HNC) Subnamespace (Weight: 6%)
**Scenario:** Using HNC, create a subnamespace `eng-testing` under parent namespace `engineering` and verify that RBAC rolebindings cascade automatically.

**Solution:**
```yaml
apiVersion: hnc.x-k8s.io/v1alpha2
kind: SubnamespaceAnchor
metadata:
  name: eng-testing
  namespace: engineering
```
**Verification:**
```bash
kubectl get subnamespaceanchors -n engineering
kubectl get namespace eng-testing
kubectl hns tree engineering
```

---

### Task 9: Karpenter Inter-Zone Topology Spread Constraints (Weight: 7%)
**Scenario:** Ensure Karpenter respects a topology spread constraint requiring the `checkout-api` pods to be evenly spread across availability zones with `maxSkew: 1`.

**Solution:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: checkout-api
  namespace: checkout
spec:
  replicas: 6
  selector:
    matchLabels:
      app: checkout-api
  template:
    metadata:
      labels:
        app: checkout-api
    spec:
      topologySpreadConstraints:
        - maxSkew: 1
          topologyKey: topology.kubernetes.io/zone
          whenUnsatisfiable: DoNotSchedule
          labelSelector:
            matchLabels:
              app: checkout-api
      containers:
        - name: checkout
          image: ghcr.io/enterprise/checkout:v1.2.0
          resources:
            requests:
              cpu: "1"
              memory: "2Gi"
```

---

### Task 10: Kyverno Generating NetworkPolicies for New Namespaces (Weight: 7%)
**Scenario:** Whenever any developer creates a new namespace, Kyverno must automatically generate a default-deny ingress `NetworkPolicy` inside that new namespace.

**Solution:**
```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: generate-default-deny-netpol
spec:
  rules:
    - name: create-default-deny
      match:
        any:
          - resources:
              kinds: [Namespace]
      generate:
        apiVersion: networking.k8s.io/v1
        kind: NetworkPolicy
        name: default-deny-ingress
        namespace: "{{request.object.metadata.name}}"
        synchronize: true
        data:
          spec:
            podSelector: {}
            policyTypes:
              - Ingress
```

---

### Task 11: Alertmanager Notification Silencing (Weight: 6%)
**Scenario:** Create an automated silence rule in Alertmanager silencing all `PodCrashLoopBackOff` alerts for namespace `sandbox` during scheduled maintenance from 22:00 to 23:00.

**Solution:**
```bash
amtool silence add \
  --alertmanager.url=http://alertmanager-operated.monitoring.svc:9093 \
  --comment="Scheduled maintenance on sandbox namespace" \
  --duration=1h \
  alertname=PodCrashLoopBackOff namespace=sandbox
```

---

### Task 12: Gateway API HTTPRoute Header Matching (Weight: 6%)
**Scenario:** Route requests containing HTTP header `X-Canary: beta` to service `web-canary` on port 80, while all other requests route to `web-stable`.

**Solution:**
```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: web-route
  namespace: frontend
spec:
  parentRefs:
    - name: shared-gateway
      namespace: platform-network
  rules:
    - matches:
        - headers:
            - name: X-Canary
              value: beta
      backendRefs:
        - name: web-canary
          port: 80
    - backendRefs:
        - name: web-stable
          port: 80
```

---

### Task 13: Argo CD Synced Resource Health Check Custom Lua Script (Weight: 7%)
**Scenario:** Argo CD reports custom resource `CRDJob` as `Healthy` even while the CRD status is `Processing`. Write a custom Lua health check in Argo CD ConfigMap `argocd-cm` to accurately report health.

**Solution:**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-cm
  namespace: argocd
data:
  resource.customizations.health.batch.platform.io_CRDJob: |
    hs = {}
    if obj.status ~= nil then
      if obj.status.phase == "Complete" then
        hs.status = "Healthy"
        hs.message = "Job finished successfully"
        return hs
      end
      if obj.status.phase == "Failed" then
        hs.status = "Degraded"
        hs.message = "Job failed"
        return hs
      end
    end
    hs.status = "Progressing"
    hs.message = "Job is running"
    return hs
```

---

### Task 14: Pod Disruption Budget (PDB) Safeguards (Weight: 6%)
**Scenario:** Ensure the `order-database` statefulset with 3 replicas never drops below 2 available pods during node draining, Karpenter consolidation, or Kubernetes upgrades.

**Solution:**
```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: order-db-pdb
  namespace: orders
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: order-database
```

---

### Task 15: Crossplane Managed Resource ProviderConfig (Weight: 6%)
**Scenario:** Configure a Crossplane `ProviderConfig` for AWS referencing an in-cluster Kubernetes secret containing the AWS credentials.

**Solution:**
```yaml
apiVersion: aws.upbound.io/v1beta1
kind: ProviderConfig
metadata:
  name: default
spec:
  credentials:
    source: Secret
    secretRef:
      namespace: crossplane-system
      name: aws-creds
      key: credentials
```
