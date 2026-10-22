# Domain 4: Platform Architecture & Scalability — Performance Labs & Drills

---

## Lab 1: Deploy KEDA ScaledObject with RabbitMQ Trigger

### Objective
Deploy a KEDA `ScaledObject` that scales a worker deployment from 0 to 20 replicas based on queue length in RabbitMQ.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: queue-worker
  namespace: messaging
spec:
  replicas: 0
  selector:
    matchLabels:
      app: queue-worker
  template:
    metadata:
      labels:
        app: queue-worker
    spec:
      containers:
        - name: worker
          image: busybox:1.36
          command: ["sh", "-c", "echo Processing queue; sleep 10"]
          resources:
            requests:
              cpu: "100m"
              memory: "128Mi"
            limits:
              cpu: "250m"
              memory: "256Mi"
---
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
  name: rabbitmq-queue-scaler
  namespace: messaging
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: queue-worker
  minReplicaCount: 0
  maxReplicaCount: 20
  cooldownPeriod: 60
  pollingInterval: 10
  triggers:
    - type: rabbitmq
      metadata:
        protocol: auto
        queueName: order-tasks
        mode: QueueLength
        value: "25"
      authenticationRef:
        name: rabbitmq-auth
---
apiVersion: keda.sh/v1alpha1
kind: TriggerAuthentication
metadata:
  name: rabbitmq-auth
  namespace: messaging
spec:
  secretTargetRef:
    - parameter: host
      name: rabbitmq-secret
      key: connection-string
```

### Verification
```bash
kubectl apply -f keda-rabbitmq-scaler.yaml
kubectl get scaledobjects -n messaging
kubectl describe scaledobject rabbitmq-queue-scaler -n messaging
```

---

## Lab 2: Modern Node Autoscaling with Karpenter v1.0 NodePool

### Objective
Create a Karpenter `NodePool` targeting AMD64 and ARM64 Spot compute with an active disruption consolidation budget.

```yaml
apiVersion: karpenter.sh/v1
kind: NodePool
metadata:
  name: platform-spot-compute
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
        - key: "karpenter.k8s.aws/instance-family"
          operator: In
          values: ["c6g", "c7g", "m6i", "m7i"]
      nodeClassRef:
        group: karpenter.k8s.aws
        kind: EC2NodeClass
        name: default-ec2-nodeclass
  limits:
    cpu: "500"
    memory: 1000Gi
  disruption:
    consolidationPolicy: WhenEmptyOrUnderutilized
    consolidateAfter: 30s
    budgets:
      - nodes: 20%
```

### Verification
```bash
kubectl apply -f karpenter-nodepool.yaml
kubectl get nodepools
kubectl describe nodepool platform-spot-compute
```

---

## Lab 3: Kubernetes Gateway API & Cross-Namespace HTTPRoute

### Objective
Deploy a centralized Gateway in `platform-gateway` and an `HTTPRoute` in the `billing` namespace routing `/v1/invoices` traffic to an internal service, validated with `ReferenceGrant`.

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: shared-edge-gateway
  namespace: platform-gateway
spec:
  gatewayClassName: cilium
  listeners:
    - name: http
      protocol: HTTP
      port: 80
      allowedRoutes:
        namespaces:
          from: All
---
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: billing-route
  namespace: billing
spec:
  parentRefs:
    - name: shared-edge-gateway
      namespace: platform-gateway
  hostnames:
    - "api.company.internal"
  rules:
    - matches:
        - path:
            type: PathPrefix
            value: /v1/invoices
      backendRefs:
        - name: invoice-service
          port: 8080
---
apiVersion: gateway.networking.k8s.io/v1beta1
kind: ReferenceGrant
metadata:
  name: allow-platform-gateway
  namespace: billing
spec:
  from:
    - group: gateway.networking.k8s.io
      kind: Gateway
      namespace: platform-gateway
  to:
    - group: ""
      kind: Service
      name: invoice-service
```

### Verification
```bash
kubectl apply -f gateway-cross-namespace.yaml
kubectl get gateways -n platform-gateway
kubectl get httproutes -n billing
```
