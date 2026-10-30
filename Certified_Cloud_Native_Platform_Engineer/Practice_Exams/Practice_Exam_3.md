# CNPE Full-Length Performance Practice Exam 3

**Time Limit:** 120 Minutes  
**Passing Score:** 70% (Score Target: $\ge 80\%$)  
**Topology:** Kubernetes v1.31+, Argo CD, Argo Rollouts, Crossplane, Kyverno, ESO, OpenTelemetry, KEDA, Gateway API  
**Format:** 15 Real-World Hands-On Platform Engineering Tasks with Full Solution Manifests & Verification Commands

---

### Task 1: Argo CD Multi-Cluster Disaster Recovery Failover (Weight: 7%)
**Scenario:** Region `us-east-1` experiences a catastrophic control plane failure. Reconfigure the Argo CD ApplicationSet `global-services` to fail over all traffic to region `us-west-2` by adjusting cluster selectors.

**Solution:**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: global-services
  namespace: argocd
spec:
  generators:
    - clusters:
        selector:
          matchLabels:
            region: us-west-2
            status: active
  template:
    metadata:
      name: '{{name}}-failover'
    spec:
      project: default
      source:
        repoURL: https://github.com/enterprise/prod-manifests.git
        targetRevision: main
        path: apps/core
      destination:
        server: '{{server}}'
        namespace: default
      syncPolicy:
        automated:
          prune: true
          selfHeal: true
```

---

### Task 2: Crossplane Function Pipeline Composition (v1.16+) (Weight: 8%)
**Scenario:** Author a modern Crossplane Composition utilizing Composition Functions (`function-patch-and-transform` and `function-auto-ready`) rather than deprecated native P&T.

**Solution:**
```yaml
apiVersion: apiextensions.crossplane.io/v1
kind: Composition
metadata:
  name: modern-storage-composition
spec:
  compositeTypeRef:
    apiVersion: storage.platform.io/v1alpha1
    kind: XCloudBucket
  mode: Pipeline
  pipeline:
    - step: patch-and-transform
      functionRef:
        name: function-patch-and-transform
      input:
        apiVersion: pt.fn.crossplane.io/v1beta1
        kind: Resources
        resources:
          - name: cloud-bucket
            base:
              apiVersion: s3.aws.upbound.io/v1beta1
              kind: Bucket
              spec:
                forProvider:
                  region: us-east-1
            patches:
              - type: FromCompositeFieldPath
                fromFieldPath: spec.name
                toFieldPath: metadata.annotations[crossplane.io/external-name]
    - step: auto-ready
      functionRef:
        name: function-auto-ready
```

---

### Task 3: OpenTelemetry OTTL PII Data Redaction (Weight: 7%)
**Scenario:** Use the OpenTelemetry Transformation Language (OTTL) inside the OTel Collector `transform` processor to scrub authorization tokens and credit card numbers from span attributes.

**Solution:**
```yaml
processors:
  transform:
    error_mode: ignore
    trace_statements:
      - context: span
        statements:
          - delete_key(attributes, "http.request.header.authorization")
          - replace_pattern(attributes["customer.card"], "^[0-9]{12}([0-9]{4})$", "XXXX-XXXX-XXXX-$$1") where attributes["customer.card"] != nil
```

---

### Task 4: Argo Rollouts Dynamic Weight Promotion via Gateway API (Weight: 7%)
**Scenario:** Configure an Argo Rollouts resource using the Kubernetes Gateway API plugin to dynamically shift traffic from stable to canary across 4 phases: 10%, 25%, 50%, 100%.

**Solution:**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: frontend-rollout
  namespace: web
spec:
  replicas: 4
  strategy:
    canary:
      trafficRouting:
        gatewayAPI:
          httpRoute: frontend-route
          namespace: web
      steps:
        - setWeight: 10
        - pause: { duration: 10m }
        - setWeight: 25
        - pause: { duration: 15m }
        - setWeight: 50
        - pause: { duration: 20m }
        - setWeight: 100
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
        - name: web
          image: ghcr.io/enterprise/frontend:v4.0.0
          ports:
            - containerPort: 80
```

---

### Task 5: Kyverno Image Tag Immobility Policy (Weight: 6%)
**Scenario:** Prevent developers from using the `:latest` tag in any container image deployed to production namespaces.

**Solution:**
```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: disallow-latest-tag
spec:
  validationFailureAction: Enforce
  rules:
    - name: validate-image-tag
      match:
        any:
          - resources:
              kinds: [Pod]
              namespaces: ["prod-*"]
      validate:
        message: "Using ':latest' image tag is prohibited in production namespaces."
        pattern:
          spec:
            containers:
              - image: "!*:latest"
```

---

### Task 6: KEDA Scaling on Prometheus Metric Query (Weight: 7%)
**Scenario:** Scale the `search-indexer` deployment between 2 and 20 pods based on the total active Elasticsearch unassigned shards reported by Prometheus.

**Solution:**
```yaml
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
  name: elasticsearch-shard-scaler
  namespace: search
spec:
  scaleTargetRef:
    name: search-indexer
  minReplicaCount: 2
  maxReplicaCount: 20
  triggers:
    - type: prometheus
      metadata:
        serverAddress: http://prometheus-operated.monitoring.svc:9090
        metricName: es_unassigned_shards
        query: sum(elasticsearch_cluster_health_unassigned_shards)
        threshold: "5"
```

---

### Task 7: Multi-Window Multi-Burn-Rate PrometheusRule (Weight: 7%)
**Scenario:** Implement a Google SRE-style $14.4\times$ burn-rate alert for a 99.9% SLO service. The alert must verify both a 1-hour long window and a 5-minute short window before firing.

**Solution:**
```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: slo-burnrate-alerts
  namespace: core
spec:
  groups:
    - name: slo.burnrate
      rules:
        - alert: ServiceBurnRateUrgent
          expr: |
            (
              sum(rate(http_requests_total{job="core-api",status=~"5.."}[1h]))
              /
              sum(rate(http_requests_total{job="core-api"}[1h]))
              > 14.4 * 0.001
            )
            and
            (
              sum(rate(http_requests_total{job="core-api",status=~"5.."}[5m]))
              /
              sum(rate(http_requests_total{job="core-api"}[5m]))
              > 14.4 * 0.001
            )
          labels:
            severity: page
          annotations:
            summary: "Urgent: 2% of monthly error budget consumed within 1 hour"
```

---

### Task 8: Karpenter Inter-Node Consolidation Disruption Window (Weight: 6%)
**Scenario:** Configure Karpenter to never consolidate worker nodes during peak trading hours between 08:00 and 18:00 UTC Monday through Friday.

**Solution:**
```yaml
apiVersion: karpenter.sh/v1
kind: NodePool
metadata:
  name: trading-compute
spec:
  template:
    spec:
      requirements:
        - key: "karpenter.sh/capacity-type"
          operator: In
          values: ["on-demand"]
      nodeClassRef:
        group: karpenter.k8s.aws
        kind: EC2NodeClass
        name: default
  disruption:
    consolidationPolicy: WhenEmptyOrUnderutilized
    budgets:
      - schedule: "0 8 * * 1-5"
        duration: 10h
        nodes: "0"
```

---

### Task 9: External Secrets Operator PushSecret Synchronization (Weight: 6%)
**Scenario:** Push a freshly generated Kubernetes TLS certificate secret `wildcard-tls` in namespace `ingress` to an AWS Secrets Manager secret named `corp/certificates/wildcard-tls`.

**Solution:**
```yaml
apiVersion: external-secrets.io/v1alpha1
kind: PushSecret
metadata:
  name: push-wildcard-tls
  namespace: ingress
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: aws-secrets-store
    kind: ClusterSecretStore
  selector:
    secret:
      name: wildcard-tls
  data:
    - match:
        secretKey: tls.crt
        remoteRef:
          remoteKey: corp/certificates/wildcard-tls
          property: certificate
    - match:
        secretKey: tls.key
        remoteRef:
          remoteKey: corp/certificates/wildcard-tls
          property: privateKey
```

---

### Task 10: Cilium ClusterMesh Multi-Cluster Service Discovery (Weight: 7%)
**Scenario:** Expose a database service `analytics-db` in cluster `alpha` so that pods running in cluster `beta` can route to it via Cilium ClusterMesh global services.

**Solution:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: analytics-db
  namespace: data
  annotations:
    service.cilium.io/global: "true"
    service.cilium.io/shared: "true"
spec:
  type: ClusterIP
  ports:
    - port: 5432
      targetPort: 5432
  selector:
    app: postgresql
```

---

### Task 11: Kyverno Generating Resource Quotas on Namespace Creation (Weight: 6%)
**Scenario:** Create a Kyverno policy that automatically injects a `ResourceQuota` limiting pods to 50 and CPU to 20 cores whenever a new non-system namespace is created.

**Solution:**
```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: generate-tenant-quotas
spec:
  rules:
    - name: inject-quota
      match:
        any:
          - resources:
              kinds: [Namespace]
              names:
                - "!kube-*"
                - "!default"
      generate:
        apiVersion: v1
        kind: ResourceQuota
        name: tenant-quota
        namespace: "{{request.object.metadata.name}}"
        synchronize: true
        data:
          spec:
            hard:
              pods: "50"
              requests.cpu: "20"
              requests.memory: "40Gi"
```

---

### Task 12: Argo CD Sync Windows Maintenance Freeze (Weight: 6%)
**Scenario:** Prevent Argo CD from executing any automated or manual sync operations for the `payment-system` app between 23:00 and 01:00 UTC daily.

**Solution:**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: payment-project
  namespace: argocd
spec:
  syncWindows:
    - kind: deny
      schedule: "0 23 * * *"
      duration: 2h
      applications:
        - "payment-system"
      manualSync: false
```

---

### Task 13: Gateway API URL Redirection (Weight: 6%)
**Scenario:** Configure an `HTTPRoute` rule permanently redirecting all legacy HTTP requests from `/old-api/*` to `/v2/api/*` with HTTP status code 301.

**Solution:**
```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: legacy-redirect
  namespace: default
spec:
  parentRefs:
    - name: shared-gateway
  rules:
    - matches:
        - path:
            type: PathPrefix
            value: /old-api
      filters:
        - type: RequestRedirect
          requestRedirect:
            path:
              type: ReplacePrefixMatch
              replacePrefixMatch: /v2/api
            statusCode: 301
```

---

### Task 14: Prometheus Operator Probe Synthetic Monitoring (Weight: 7%)
**Scenario:** Configure a `Probe` custom resource using Blackbox Exporter to perform synthetic HTTPS GET checks against external partner API `https://api.partner.com/v1/ping` every 30 seconds.

**Solution:**
```yaml
apiVersion: monitoring.coreos.com/v1
kind: Probe
metadata:
  name: partner-api-synthetic-check
  namespace: monitoring
  labels:
    release: prometheus
spec:
  interval: 30s
  prober:
    url: blackbox-exporter.monitoring.svc:9115
  targets:
    staticConfig:
      static:
        - "https://api.partner.com/v1/ping"
  module: http_2xx
```

---

### Task 15: Crossplane Composite Resource Deletion Protection (Weight: 7%)
**Scenario:** Ensure that deleting a composite claim `PostgreSQLInstance` does not delete the physical RDS database in the cloud, setting the deletion policy to `Orphan`.

**Solution:**
```yaml
apiVersion: database.platform.io/v1alpha1
kind: PostgreSQLInstance
metadata:
  name: enterprise-db
  namespace: prod
spec:
  parameters:
    storageGB: 100
    engineVersion: "16"
  compositionRef:
    name: rds-postgres-composition
  # Prevents deletion of the cloud managed resource when claim is deleted
  deletionPolicy: Orphan
```
**Verification:**
```bash
kubectl apply -f protected-claim.yaml
kubectl get postgresqlinstances -n prod
```
