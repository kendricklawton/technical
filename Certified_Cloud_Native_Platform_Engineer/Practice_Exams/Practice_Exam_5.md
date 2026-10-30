# CNPE Full-Length Performance Practice Exam 5

**Time Limit:** 120 Minutes  
**Passing Score:** 70% (Score Target: $\ge 80\%$)  
**Topology:** Kubernetes v1.31+, Argo CD, Argo Rollouts, Crossplane, Kyverno, ESO, OpenTelemetry, KEDA, Gateway API, Cilium  
**Format:** 15 Real-World Hands-On Platform Engineering Tasks with Full Solution Manifests & Verification Commands

---

### Task 1: Crossplane XRD with Status Field Subresource & Conditions (Weight: 8%)
**Scenario:** Build an XRD for self-service Redis clusters that exposes status fields (`status.endpoint`, `status.port`, `status.ready`) and verifies condition evaluation.

**Solution:**
```yaml
apiVersion: apiextensions.crossplane.io/v1
kind: CompositeResourceDefinition
metadata:
  name: xredisclusters.cache.platform.io
spec:
  group: cache.platform.io
  names:
    kind: XRedisCluster
    plural: xredisclusters
  claimNames:
    kind: RedisCluster
    plural: redisclusters
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
              required: [nodeCount, memoryMB]
              properties:
                nodeCount:
                  type: integer
                memoryMB:
                  type: integer
            status:
              type: object
              properties:
                endpoint:
                  type: string
                port:
                  type: integer
```

---

### Task 2: Multi-Cluster ApplicationSet Git Directory Generator (Weight: 8%)
**Scenario:** Configure an Argo CD `ApplicationSet` that dynamically discovers tenant manifests located in `tenants/*` of a Git repository and deploys them to target clusters labeled `env: production`.

**Solution:**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: dynamic-tenant-fleet
  namespace: argocd
spec:
  generators:
    - matrix:
        generators:
          - git:
              repoURL: https://github.com/enterprise/tenant-fleet.git
              revision: HEAD
              directories:
                - path: tenants/*
          - clusters:
              selector:
                matchLabels:
                  env: production
  template:
    metadata:
      name: '{{name}}-{{path.basename}}'
    spec:
      project: default
      source:
        repoURL: https://github.com/enterprise/tenant-fleet.git
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

---

### Task 3: OpenTelemetry Collector Load-Balancing Exporter (Weight: 7%)
**Scenario:** Configure the OTel Collector to load-balance trace spans across 3 downstream Tempo collector replicas using trace-id consistent hashing.

**Solution:**
```yaml
exporters:
  loadbalancing:
    protocol:
      otlp:
        tls:
          insecure: true
    resolver:
      static:
        hostnames:
          - tempo-collector-0.tempo.svc:4317
          - tempo-collector-1.tempo.svc:4317
          - tempo-collector-2.tempo.svc:4317
service:
  pipelines:
    traces:
      receivers: [otlp]
      processors: [memory_limiter, batch]
      exporters: [loadbalancing]
```

---

### Task 4: Argo Rollouts Automated Traffic Mirroring (Weight: 7%)
**Scenario:** Configure an Argo Rollout with Nginx Ingress or Gateway API mirroring 100% of production traffic to a shadow canary service `catalog-shadow` without returning responses to clients.

**Solution:**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: catalog-service
  namespace: catalog
spec:
  replicas: 4
  strategy:
    canary:
      trafficRouting:
        nginx:
          stableIngress: catalog-ingress
          additionalIngressAnnotations:
            nginx.ingress.kubernetes.io/mirror-target: "http://catalog-shadow.catalog.svc:8080"
            nginx.ingress.kubernetes.io/mirror-request-body: "on"
      steps:
        - pause: { duration: 1h }
  selector:
    matchLabels:
      app: catalog
  template:
    metadata:
      labels:
        app: catalog
    spec:
      containers:
        - name: catalog
          image: ghcr.io/enterprise/catalog:v3.0.0
```

---

### Task 5: Kyverno Generating Synced ConfigMaps on Namespace Creation (Weight: 6%)
**Scenario:** When any new namespace is created, Kyverno must generate a standard corporate `ConfigMap` `corporate-ca-bundle` synced from the `default` namespace.

**Solution:**
```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: sync-ca-bundle
spec:
  rules:
    - name: sync-bundle
      match:
        any:
          - resources:
              kinds: [Namespace]
      generate:
        apiVersion: v1
        kind: ConfigMap
        name: corporate-ca-bundle
        namespace: "{{request.object.metadata.name}}"
        synchronize: true
        clone:
          namespace: default
          name: corporate-ca-bundle
```

---

### Task 6: KEDA Autoscaling on CloudWatch Metric (Weight: 7%)
**Scenario:** Scale worker deployment `sqs-processor` in namespace `worker` based on AWS SQS queue depth queried via the KEDA AWS CloudWatch scaler.

**Solution:**
```yaml
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
  name: aws-sqs-scaler
  namespace: worker
spec:
  scaleTargetRef:
    name: sqs-processor
  minReplicaCount: 1
  maxReplicaCount: 40
  triggers:
    - type: aws-cloudwatch
      metadata:
        namespace: AWS/SQS
        dimensionName: QueueName
        dimensionValue: incoming-orders-queue
        metricName: ApproximateNumberOfMessagesVisible
        targetMetricValue: "100"
        minMetricValue: "0"
        awsRegion: "us-east-1"
        identityOwner: operator
```

---

### Task 7: Cilium ClusterMesh Multi-Cluster Service Export (Weight: 7%)
**Scenario:** Export a Redis service in cluster `west` so that workloads in cluster `east` can seamlessly discover and connect to it.

**Solution:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: shared-redis
  namespace: cache
  annotations:
    service.cilium.io/global: "true"
    service.cilium.io/shared: "true"
spec:
  type: ClusterIP
  ports:
    - port: 6379
      targetPort: 6379
  selector:
    app: redis
```

---

### Task 8: Karpenter Custom UserData & Node Initialization (Weight: 6%)
**Scenario:** Configure an `EC2NodeClass` injecting custom bash UserData to mount an NVMe instance store disk to `/mnt/nvme-scratch` upon worker node boot.

**Solution:**
```yaml
apiVersion: karpenter.k8s.aws/v1
kind: EC2NodeClass
metadata:
  name: nvme-nodes
spec:
  amiFamily: AL2023
  subnetSelectorTerms:
    - tags:
        karpenter.sh/discovery: cluster-prod
  securityGroupSelectorTerms:
    - tags:
        karpenter.sh/discovery: cluster-prod
  userData: |
    MIME-Version: 1.0
    Content-Type: multipart/mixed; boundary="//"

    --//
    Content-Type: text/x-shellscript; charset="us-ascii"

    #!/bin/bash
    mkfs.ext4 -E nodiscard /dev/nvme1n1
    mkdir -p /mnt/nvme-scratch
    mount -o noatime /dev/nvme1n1 /mnt/nvme-scratch
    chmod 1777 /mnt/nvme-scratch
    --//--
```

---

### Task 9: External Secrets Operator Multi-Secret Store Merging (Weight: 6%)
**Scenario:** Combine secret keys from two separate Vault paths into a single output Kubernetes Secret `unified-app-secret` using ESO.

**Solution:**
```yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: merged-secret
  namespace: app
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: vault-backend
    kind: ClusterSecretStore
  target:
    name: unified-app-secret
    creationPolicy: Owner
  dataFrom:
    - extract:
        key: production/app/credentials
    - extract:
        key: production/app/api-keys
```

---

### Task 10: Gateway API Cross-Namespace Secret Reference via ReferenceGrant (Weight: 6%)
**Scenario:** Allow a Gateway in namespace `infra-edge` to reference a TLS certificate secret `wildcard-cert` located in namespace `security-certs`.

**Solution:**
```yaml
apiVersion: gateway.networking.k8s.io/v1beta1
kind: ReferenceGrant
metadata:
  name: allow-edge-to-certs
  namespace: security-certs
spec:
  from:
    - group: gateway.networking.k8s.io
      kind: Gateway
      namespace: infra-edge
  to:
    - group: ""
      kind: Secret
      name: wildcard-cert
```

---

### Task 11: Alertmanager Inhibit Rule for Cluster-Wide Storage Failure (Weight: 6%)
**Scenario:** Configure Alertmanager so that if `StorageClusterDegraded` is firing, all lower-priority `PVUsageWarning` alerts on that cluster are inhibited.

**Solution:**
```yaml
inhibit_rules:
  - source_match:
      alertname: 'StorageClusterDegraded'
    target_match:
      alertname: 'PVUsageWarning'
    equal: ['cluster', 'zone']
```

---

### Task 12: Argo CD Sync Windows Deny Window for Black Friday (Weight: 6%)
**Scenario:** Freeze all automated and manual sync operations across all production applications during an enterprise freeze period.

**Solution:**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: production-project
  namespace: argocd
spec:
  syncWindows:
    - kind: deny
      schedule: "0 0 25 11 *"
      duration: 72h
      applications:
        - "*"
      manualSync: false
```

---

### Task 13: Kyverno Disallow Privilege Escalation Mutation Policy (Weight: 6%)
**Scenario:** Mutate all containers in incoming pods to strictly enforce `allowPrivilegeEscalation: false` if not specified.

**Solution:**
```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: mutate-privilege-escalation
spec:
  rules:
    - name: set-allow-privilege-escalation-false
      match:
        any:
          - resources:
              kinds: [Pod]
      mutate:
        patchStrategicMerge:
          spec:
            containers:
              - (name): "*"
                securityContext:
                  +(allowPrivilegeEscalation): false
```

---

### Task 14: Automated eBPF Continuous Profiling Setup with Pyroscope (Weight: 7%)
**Scenario:** Deploy the Pyroscope eBPF agent DaemonSet to collect continuous CPU profiling flame graphs across all cluster nodes without modifying pod configurations.

**Solution:**
```bash
helm repo add pyroscope-io https://pyroscope-io.github.io/helm-chart
helm repo update
helm install pyroscope-ebpf pyroscope-io/pyroscope-ebpf \
  --namespace observability \
  --set serverAddress=http://pyroscope.observability.svc:4040
```

---

### Task 15: Crossplane Composite Resource Orphan Deletion Policy (Weight: 7%)
**Scenario:** Author a claim for an RDS PostgreSQL database with `deletionPolicy: Orphan` to guarantee that even if an engineer runs `kubectl delete postgresqlinstance db-claim`, the physical cloud database remains active and untouched.

**Solution:**
```yaml
apiVersion: database.platform.io/v1alpha1
kind: PostgreSQLInstance
metadata:
  name: mission-critical-db
  namespace: prod
spec:
  parameters:
    storageGB: 500
    engineVersion: "16"
  deletionPolicy: Orphan
```
**Verification:**
```bash
kubectl apply -f orphan-claim.yaml
kubectl get postgresqlinstances -n prod
```
