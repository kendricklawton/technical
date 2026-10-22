# CNPE Full-Length Performance Practice Exam 1

**Time Limit:** 120 Minutes  
**Passing Score:** 70% (Score Target: $\ge 80\%$)  
**Topology:** Kubernetes v1.31+, Argo CD v2.12+, Crossplane v1.16+, Kyverno v1.12+, OTel Collector, KEDA, Cilium  
**Format:** 15 Real-World Hands-On Platform Engineering Tasks with Full Solution Manifests & Verification Commands

---

### Task 1: Argo CD Auto-Sync with HPA Conflict Resolution (Weight: 7%)
**Scenario:** The deployment `payment-processor` in namespace `payments` is managed by Argo CD and scales horizontally between 3 and 15 replicas using HPA. Every time HPA scales up the deployment, Argo CD flags the application as `OutOfSync` and attempts to revert replicas to 3.
**Requirements:**
1. Update the Argo CD Application manifest `/opt/cnpe/apps/payment-app.yaml`.
2. Configure `spec.ignoreDifferences` targeting group `apps`, kind `Deployment`, and json pointer path `/spec/replicas`.
3. Configure `spec.syncPolicy.automated` with `prune: true` and `selfHeal: true`.
4. Apply the updated manifest and trigger an Argo CD hard refresh.

**Solution:**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: payment-processor
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/enterprise/platform-apps.git
    targetRevision: HEAD
    path: apps/payments
  destination:
    server: https://kubernetes.default.svc
    namespace: payments
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
  ignoreDifferences:
    - group: apps
      kind: Deployment
      jsonPointers:
        - /spec/replicas
```
**Verification:**
```bash
kubectl apply -f /opt/cnpe/apps/payment-app.yaml
argocd app get payment-processor --refresh
```

---

### Task 2: Multi-Cluster ApplicationSet Matrix Generator (Weight: 7%)
**Scenario:** The platform team needs to dynamically deploy applications across multiple production clusters.
**Requirements:**
1. Create an `ApplicationSet` named `tenant-workloads` in namespace `argocd`.
2. Use a Matrix Generator combining:
   - A cluster generator matching clusters labeled `environment: production`.
   - A Git generator scanning directories `services/*` in repository `https://github.com/enterprise/microservices.git`.
3. Destination namespace must match `{{path.basename}}`.

**Solution:**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: tenant-workloads
  namespace: argocd
spec:
  generators:
    - matrix:
        generators:
          - clusters:
              selector:
                matchLabels:
                  environment: production
          - git:
              repoURL: https://github.com/enterprise/microservices.git
              revision: HEAD
              directories:
                - path: services/*
  template:
    metadata:
      name: '{{name}}-{{path.basename}}'
    spec:
      project: default
      source:
        repoURL: https://github.com/enterprise/microservices.git
        targetRevision: HEAD
        path: '{{path}}'
      destination:
        server: '{{server}}'
        namespace: '{{path.basename}}'
      syncPolicy:
        automated:
          prune: true
          selfHeal: true
        syncOptions:
          - CreateNamespace=true
```
**Verification:**
```bash
kubectl apply -f tenant-workloads.yaml
kubectl get applications -n argocd
```

---

### Task 3: Argo Rollouts Canary with Prometheus Analysis (Weight: 8%)
**Scenario:** Deploy a progressive delivery rollout for `order-api` in namespace `orders`.
**Requirements:**
1. Define an `AnalysisTemplate` named `success-rate` querying Prometheus.
2. The query must check that the 5xx error rate is below 1%:
   `sum(rate(http_requests_total{app="order-api",status=~"5.."}[2m])) / sum(rate(http_requests_total{app="order-api"}[2m])) <= 0.01`
3. Configure a `Rollout` with 5 replicas using steps:
   - Set weight 20%, pause 5m.
   - Run background analysis referencing `success-rate`.
   - Set weight 50%, pause 10m.
   - Set weight 100%.

**Solution:**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: AnalysisTemplate
metadata:
  name: success-rate
  namespace: orders
spec:
  metrics:
    - name: error-rate
      interval: 30s
      successCondition: result[0] <= 0.01
      failureLimit: 3
      provider:
        prometheus:
          address: http://prometheus-k8s.monitoring.svc:9090
          query: |
            sum(rate(http_requests_total{app="order-api",status=~"5.."}[2m]))
            /
            sum(rate(http_requests_total{app="order-api"}[2m]))
---
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: order-api
  namespace: orders
spec:
  replicas: 5
  strategy:
    canary:
      analysis:
        templates:
          - templateName: success-rate
      steps:
        - setWeight: 20
        - pause: { duration: 5m }
        - setWeight: 50
        - pause: { duration: 10m }
        - setWeight: 100
  selector:
    matchLabels:
      app: order-api
  template:
    metadata:
      labels:
        app: order-api
    spec:
      containers:
        - name: order-api
          image: ghcr.io/enterprise/order-api:v2.1.0
          ports:
            - containerPort: 8080
```
**Verification:**
```bash
kubectl apply -f order-rollout.yaml
kubectl argo rollouts get rollout order-api -n orders --watch
```

---

### Task 4: Crossplane Composite Resource Definition (XRD) (Weight: 7%)
**Scenario:** Build a self-service platform API allowing developers to request a PostgreSQL database without writing cloud-specific infrastructure code.
**Requirements:**
1. Define an XRD named `xpostgresqlinstances.database.platform.io`.
2. Group: `database.platform.io`, Claim Name: `PostgreSQLInstance`, Kind: `XPostgreSQLInstance`.
3. Accept spec fields: `storageGB` (integer) and `engineVersion` (string).

**Solution:**
```yaml
apiVersion: apiextensions.crossplane.io/v1
kind: CompositeResourceDefinition
metadata:
  name: xpostgresqlinstances.database.platform.io
spec:
  group: database.platform.io
  names:
    kind: XPostgreSQLInstance
    plural: xpostgresqlinstances
  claimNames:
    kind: PostgreSQLInstance
    plural: postgresqlinstances
  versions:
    - name: v1alpha1
      served: true
      referenceable: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              required: [storageGB, engineVersion]
              properties:
                storageGB:
                  type: integer
                  minimum: 10
                  maximum: 1000
                engineVersion:
                  type: string
                  default: "16"
```
**Verification:**
```bash
kubectl apply -f xrd.yaml
kubectl get xrd
```

---

### Task 5: Kyverno ClusterPolicy Disallowing Privileged Pods (Weight: 6%)
**Scenario:** Enforce CIS Kubernetes Benchmark compliance across the cluster.
**Requirements:**
1. Create a `ClusterPolicy` named `disallow-privileged`.
2. Block (`Enforce`) any Pod where `securityContext.privileged` is set to `true`.
3. Exclude the `kube-system` and `kyverno` namespaces from the policy.

**Solution:**
```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: disallow-privileged
spec:
  validationFailureAction: Enforce
  background: true
  rules:
    - name: check-privileged-containers
      match:
        any:
          - resources:
              kinds: [Pod]
              namespaces:
                - "!kube-system"
                - "!kyverno"
      validate:
        message: "Privileged containers are strictly prohibited."
        pattern:
          spec:
            containers:
              - securityContext:
                  privileged: "!true"
```
**Verification:**
```bash
kubectl apply -f kyverno-privileged.yaml
kubectl run test-fail --image=nginx --privileged=true (Must be rejected)
```

---

### Task 6: External Secrets Operator Sync from HashiCorp Vault (Weight: 7%)
**Scenario:** Sync database credentials from HashiCorp Vault into namespace `billing`.
**Requirements:**
1. Create an `ExternalSecret` named `billing-db-secret` referencing `vault-backend` ClusterSecretStore.
2. Target Secret name: `billing-db-credentials`.
3. Sync key `database/data/billing` property `password` to secret key `DB_PASSWORD`.

**Solution:**
```yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: billing-db-secret
  namespace: billing
spec:
  refreshInterval: 30m
  secretStoreRef:
    name: vault-backend
    kind: ClusterSecretStore
  target:
    name: billing-db-credentials
    creationPolicy: Owner
  data:
    - secretKey: DB_PASSWORD
      remoteRef:
        key: database/data/billing
        property: password
```
**Verification:**
```bash
kubectl apply -f billing-external-secret.yaml
kubectl get externalsecrets -n billing
kubectl get secret billing-db-credentials -n billing
```

---

### Task 7: OpenTelemetry Collector Memory Limiter & Tail-Sampling (Weight: 7%)
**Scenario:** The central OTel Collector is experiencing OOM kills during traffic spikes.
**Requirements:**
1. Configure `memory_limiter` processor with `check_interval: 1s`, `limit_percentage: 80`, `spike_limit_percentage: 20`.
2. Configure `tail_sampling` retaining 100% of error spans and 5% of OK spans.
3. Order the processors correctly in the traces pipeline: `[memory_limiter, tail_sampling, batch]`.

**Solution:**
```yaml
apiVersion: opentelemetry.io/v1alpha1
kind: OpenTelemetryCollector
metadata:
  name: otel-gateway
  namespace: observability
spec:
  config: |
    receivers:
      otlp:
        protocols:
          grpc:
            endpoint: 0.0.0.0:4317
    processors:
      memory_limiter:
        check_interval: 1s
        limit_percentage: 80
        spike_limit_percentage: 20
      tail_sampling:
        decision_wait: 10s
        expected_new_traces_per_sec: 1000
        policies:
          - name: error-spans
            type: status_code
            status_code: { status_codes: [ERROR] }
          - name: sample-normal
            type: probabilistic
            probabilistic: { sampling_percentage: 5.0 }
      batch:
        send_batch_size: 4096
        timeout: 5s
    exporters:
      otlp/tempo:
        endpoint: tempo:4317
        tls: { insecure: true }
    service:
      pipelines:
        traces:
          receivers: [otlp]
          processors: [memory_limiter, tail_sampling, batch]
          exporters: [otlp/tempo]
```

---

### Task 8: KEDA Kafka Event-Driven Autoscaling (Weight: 7%)
**Scenario:** Scale the `order-worker` deployment in namespace `orders` between 1 and 30 replicas based on consumer lag.
**Requirements:**
1. Target deployment: `order-worker`.
2. Trigger: `kafka` topic `orders-v1`, consumer group `order-worker-group`.
3. Lag threshold: `50` messages per pod.
4. Scale target ref: deployment `order-worker`.

**Solution:**
```yaml
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
  name: order-worker-scaler
  namespace: orders
spec:
  scaleTargetRef:
    name: order-worker
  minReplicaCount: 1
  maxReplicaCount: 30
  pollingInterval: 15
  cooldownPeriod: 120
  triggers:
    - type: kafka
      metadata:
        bootstrapServers: kafka.messaging.svc:9092
        topic: orders-v1
        consumerGroup: order-worker-group
        lagThreshold: "50"
```

---

### Task 9: Karpenter v1.0 NodePool with Consolidation (Weight: 7%)
**Scenario:** Deploy a Karpenter NodePool that provisions Spot compute instances and automatically consolidates underutilized nodes.
**Requirements:**
1. NodePool name: `spot-workers`.
2. Architecture: `amd64` or `arm64`, capacity-type: `spot`.
3. Disruption: `consolidationPolicy: WhenEmptyOrUnderutilized`, `consolidateAfter: 1m`.
4. Budget: max 15% of nodes disrupted at a time.

**Solution:**
```yaml
apiVersion: karpenter.sh/v1
kind: NodePool
metadata:
  name: spot-workers
spec:
  template:
    spec:
      requirements:
        - key: "karpenter.sh/capacity-type"
          operator: In
          values: ["spot"]
        - key: "kubernetes.io/arch"
          operator: In
          values: ["amd64", "arm64"]
      nodeClassRef:
        group: karpenter.k8s.aws
        kind: EC2NodeClass
        name: default
  limits:
    cpu: "200"
  disruption:
    consolidationPolicy: WhenEmptyOrUnderutilized
    consolidateAfter: 1m
    budgets:
      - nodes: 15%
```

---

### Task 10: Kubernetes Gateway API & ReferenceGrant (Weight: 7%)
**Scenario:** Expose a backend service `catalog-svc` in namespace `catalog` via a shared gateway in namespace `infra-gw`.
**Requirements:**
1. Create an `HTTPRoute` in namespace `catalog` matching `/products`.
2. Reference parent gateway `shared-gw` in namespace `infra-gw`.
3. Create a `ReferenceGrant` in `catalog` granting `infra-gw` permission to route to `catalog-svc`.

**Solution:**
```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: catalog-route
  namespace: catalog
spec:
  parentRefs:
    - name: shared-gw
      namespace: infra-gw
  rules:
    - matches:
        - path:
            type: PathPrefix
            value: /products
      backendRefs:
        - name: catalog-svc
          port: 8080
---
apiVersion: gateway.networking.k8s.io/v1beta1
kind: ReferenceGrant
metadata:
  name: allow-gw-to-catalog
  namespace: catalog
spec:
  from:
    - group: gateway.networking.k8s.io
      kind: Gateway
      namespace: infra-gw
  to:
    - group: ""
      kind: Service
      name: catalog-svc
```

---

### Task 11: Pod Security Admission (PSA) Configuration (Weight: 6%)
**Scenario:** Configure namespace `pci-compliance` to strictly block pods that do not meet the Restricted standard, while emitting warnings for any API calls that violate the latest draft standard.

**Solution:**
```bash
kubectl create namespace pci-compliance --dry-run=client -o yaml | kubectl apply -f -
kubectl label --overwrite namespace pci-compliance \
  pod-security.kubernetes.io/enforce=restricted \
  pod-security.kubernetes.io/enforce-version=v1.31 \
  pod-security.kubernetes.io/warn=restricted \
  pod-security.kubernetes.io/warn-version=latest \
  pod-security.kubernetes.io/audit=restricted \
  pod-security.kubernetes.io/audit-version=latest
```

---

### Task 12: Prometheus Operator ServiceMonitor & High Latency Alert (Weight: 6%)
**Scenario:** Configure monitoring for the `auth-svc` pods in namespace `auth`.
**Requirements:**
1. Create a `ServiceMonitor` matching label `app: auth-svc` on port `metrics`.
2. Create a `PrometheusRule` triggering `AuthServiceHighLatency` if the $P99$ latency exceeds 500ms for 3 minutes:
   `histogram_quantile(0.99, sum by (le) (rate(http_request_duration_seconds_bucket{job="auth-svc"}[3m]))) > 0.5`

**Solution:**
```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: auth-monitor
  namespace: auth
  labels:
    release: prometheus
spec:
  selector:
    matchLabels:
      app: auth-svc
  endpoints:
    - port: metrics
      interval: 15s
---
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: auth-latency-alert
  namespace: auth
  labels:
    role: alert-rules
    release: prometheus
spec:
  groups:
    - name: auth.alerts
      rules:
        - alert: AuthServiceHighLatency
          expr: histogram_quantile(0.99, sum by (le) (rate(http_request_duration_seconds_bucket{job="auth-svc"}[3m]))) > 0.5
          for: 3m
          labels:
            severity: warning
          annotations:
            summary: "Auth service P99 latency exceeds 500ms"
```

---

### Task 13: Cilium L7 FQDN Egress Security Policy (Weight: 7%)
**Scenario:** Enforce zero-trust egress on the `payment-gateway` pods in namespace `payments`.
**Requirements:**
1. Only allow DNS egress to kube-dns on UDP port 53.
2. Only allow HTTPS egress to `api.stripe.com` on TCP port 443.
3. Block all other egress.

**Solution:**
```yaml
apiVersion: "cilium.io/v2"
kind: CiliumNetworkPolicy
metadata:
  name: payment-egress-lockdown
  namespace: payments
spec:
  endpointSelector:
    matchLabels:
      app: payment-gateway
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
        - matchName: "api.stripe.com"
      toPorts:
        - ports:
            - port: "443"
              protocol: TCP
```

---

### Task 14: Sigstore Cosign Image Verification with Kyverno (Weight: 7%)
**Scenario:** Block all pods in namespace `secure-apps` whose images are not signed with the platform Cosign public key stored in Secret `cosign-pub` in namespace `kyverno`.

**Solution:**
```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: verify-app-signatures
spec:
  validationFailureAction: Enforce
  webhookTimeoutSeconds: 30
  rules:
    - name: verify-signature
      match:
        any:
          - resources:
              kinds: [Pod]
              namespaces: [secure-apps]
      verifyImages:
        - imageReferences:
            - "ghcr.io/enterprise/*"
          attestors:
            - entries:
                - keys:
                    secret:
                      name: cosign-pub
                      namespace: kyverno
```

---

### Task 15: vCluster Virtual Control Plane Isolation (Weight: 6%)
**Scenario:** Deploy a lightweight virtual cluster `dev-sandbox` inside namespace `team-sandbox` with isolated network policies and resource quotas enabled in its configuration.

**Solution:**
```bash
kubectl create namespace team-sandbox --dry-run=client -o yaml | kubectl apply -f -
cat <<EOF > /tmp/vcluster-values.yaml
isolation:
  enabled: true
  networkPolicy:
    enabled: true
  resourceQuota:
    enabled: true
sync:
  nodes:
    enabled: true
    syncAllNodes: false
EOF
vcluster create dev-sandbox -n team-sandbox -f /tmp/vcluster-values.yaml
```
**Verification:**
```bash
vcluster list
kubectl get pods -n team-sandbox
```
