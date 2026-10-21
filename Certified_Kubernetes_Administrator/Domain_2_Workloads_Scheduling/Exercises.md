# Domain 2: Workloads & Scheduling — Hands-On Lab Exercises

This lab guide contains **6 production-grade, exam-realistic CKA scenarios** covering Rolling Updates/Rollbacks, Native Sidecar Containers (`restartPolicy: Always`), Taints/Tolerations with Node Affinity, Pod Anti-Affinity with `PodDisruptionBudget` & `PriorityClass`, Multi-Probe Pods with ConfigMap/Secret injection, and QoS/`LimitRange`/`ResourceQuota`/HPA v2.

---

## Lab 1: Zero-Downtime Rolling Updates, Strategy Tuning & Rollbacks

### Scenario & Requirements
1. Create a namespace `ecommerce`.
2. Create a Deployment `catalog-web` in `ecommerce` with `6` replicas running `nginx:1.25.3`, labeled `app: catalog-web, tier: frontend`.
3. Configure the `RollingUpdate` strategy so that during any update:
   - **Zero** Pods can be unavailable below the desired replica count (`maxUnavailable: 0`).
   - At most **2** extra Pods can be created above the desired replica count (`maxSurge: 2`).
   - Set `revisionHistoryLimit: 5`.
4. Perform a rolling update of the container image to `nginx:1.26.2` and verify the rollout completes.
5. Simulate a broken deployment by updating the image to `nginx:1.999-nonexistent`, verify that the rollout stalls without taking down existing healthy replicas (thanks to `maxUnavailable: 0`), and roll back `catalog-web` to the working `nginx:1.26.2` revision.

### Step-by-Step Solution

#### 1. Create Namespace & Deployment Manifest (`catalog-web.yaml`)
```bash
kubectl create ns ecommerce
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: catalog-web
  namespace: ecommerce
spec:
  replicas: 6
  revisionHistoryLimit: 5
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 2
      maxUnavailable: 0
  selector:
    matchLabels:
      app: catalog-web
      tier: frontend
  template:
    metadata:
      labels:
        app: catalog-web
        tier: frontend
    spec:
      containers:
      - name: nginx
        image: nginx:1.25.3
        ports:
        - containerPort: 80
```
```bash
kubectl apply -f catalog-web.yaml
kubectl rollout status deploy/catalog-web -n ecommerce
```

#### 2. Upgrade to `nginx:1.26.2`
```bash
kubectl set image deploy/catalog-web nginx=nginx:1.26.2 -n ecommerce --record
kubectl rollout status deploy/catalog-web -n ecommerce
```

#### 3. Introduce Broken Tag `nginx:1.999-nonexistent` and Observe Zero-Downtime Protection
```bash
kubectl set image deploy/catalog-web nginx=nginx:1.999-nonexistent -n ecommerce --record
kubectl get pods -n ecommerce
# Notice: All 6 v1.26.2 Pods remain Running while 2 new Surge Pods show ImagePullBackOff!
```

#### 4. Roll Back to Revision 2 (`nginx:1.26.2`)
```bash
kubectl rollout history deploy/catalog-web -n ecommerce
kubectl rollout undo deploy/catalog-web -n ecommerce --to-revision=2
kubectl rollout status deploy/catalog-web -n ecommerce
```

### Verification Commands
```bash
kubectl get deploy catalog-web -n ecommerce -o wide
# READY: 6/6, UP-TO-DATE: 6, AVAILABLE: 6, IMAGES: nginx:1.26.2
```

---

## Lab 2: Native Sidecar Container (`restartPolicy: Always`) & Init Container with Shared Volume

### Scenario & Requirements
An existing legacy application writes log files to `/var/log/legacy/events.log` inside its container rather than `stdout`.
Create a Pod named `legacy-processor` in the `default` namespace that meets all of the following requirements:
1. Mounts an `emptyDir` volume named `shared-logs` into all containers at `/var/log/legacy`.
2. Includes a standard `initContainer` named `log-dir-init` (`busybox:1.36`) that creates `/var/log/legacy/events.log` and writes `"=== Log Initialized ==="` into it before exiting `0`.
3. Includes a **Kubernetes Native Sidecar Container** named `log-streamer` (`busybox:1.36`) defined inside `spec.initContainers[]` with `restartPolicy: Always` that runs `sh -c "tail -n+1 -F /var/log/legacy/events.log"`.
4. Runs the main application container named `processor-app` (`busybox:1.36`) that appends a timestamped event line to `/var/log/legacy/events.log` every 3 seconds.

### Step-by-Step Solution

#### Create and Apply `legacy-processor.yaml`
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: legacy-processor
  namespace: default
spec:
  initContainers:
  # 1. Standard Init Container (runs once to completion)
  - name: log-dir-init
    image: busybox:1.36
    command:
    - /bin/sh
    - -c
    - "echo '=== Log Initialized ===' > /var/log/legacy/events.log"
    volumeMounts:
    - name: shared-logs
      mountPath: /var/log/legacy

  # 2. Native Sidecar Container (runs continuously alongside main container)
  - name: log-streamer
    image: busybox:1.36
    restartPolicy: Always
    command:
    - /bin/sh
    - -c
    - "tail -n+1 -F /var/log/legacy/events.log"
    volumeMounts:
    - name: shared-logs
      mountPath: /var/log/legacy

  containers:
  # 3. Main Application Container
  - name: processor-app
    image: busybox:1.36
    command:
    - /bin/sh
    - -c
    - "while true; do echo \"$(date -Iseconds) [INFO] Transaction processed\" >> /var/log/legacy/events.log; sleep 3; done"
    volumeMounts:
    - name: shared-logs
      mountPath: /var/log/legacy

  volumes:
  - name: shared-logs
    emptyDir: {}
```
```bash
kubectl apply -f legacy-processor.yaml
```

### Verification Commands
```bash
kubectl get pod legacy-processor
# NAME               READY   STATUS    RESTARTS   AGE
# legacy-processor   2/2     Running   0          25s

# Verify logs streamed by the native sidecar container:
kubectl logs legacy-processor -c log-streamer
# === Log Initialized ===
# 2026-10-16T19:20:01+00:00 [INFO] Transaction processed
```

---

## Lab 3: Dedicated Node Isolation with Taints, Tolerations & Hard Node Affinity

### Scenario & Requirements
You need to dedicate worker node `worker-01` (or your first worker node) exclusively to AI inference workloads:
1. Add a taint `workload=ai-inference:NoExecute` to `worker-01` so existing non-tolerating Pods are immediately evicted and no normal Pods can schedule onto it.
2. Label `worker-01` with `hardware=gpu-a100` and `disk=nvme`.
3. Create a Deployment `llm-inference` (`2` replicas, image `nginx:1.27`) in namespace `default` that:
   - Tolerates the taint `workload=ai-inference:NoExecute`.
   - Uses **hard `nodeAffinity`** (`requiredDuringSchedulingIgnoredDuringExecution`) requiring `hardware In [gpu-a100, gpu-h100]` AND `disk Exists`.
   - Uses **soft `nodeAffinity`** (`preferredDuringSchedulingIgnoredDuringExecution`) with `weight: 50` preferring `topology.kubernetes.io/zone In [us-west-2a]`.

### Step-by-Step Solution

#### 1. Taint and Label the Worker Node
```bash
kubectl taint nodes worker-01 workload=ai-inference:NoExecute
kubectl label nodes worker-01 hardware=gpu-a100 disk=nvme --overwrite
```

#### 2. Create the Deployment Manifest (`llm-inference.yaml`)
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: llm-inference
  namespace: default
spec:
  replicas: 2
  selector:
    matchLabels:
      app: llm-inference
  template:
    metadata:
      labels:
        app: llm-inference
    spec:
      tolerations:
      - key: "workload"
        operator: "Equal"
        value: "ai-inference"
        effect: "NoExecute"
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: hardware
                operator: In
                values:
                - gpu-a100
                - gpu-h100
              - key: disk
                operator: Exists
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 50
            preference:
              matchExpressions:
              - key: topology.kubernetes.io/zone
                operator: In
                values:
                - us-west-2a
      containers:
      - name: inference-server
        image: nginx:1.27
```
```bash
kubectl apply -f llm-inference.yaml
```

### Verification Commands
```bash
kubectl get pods -l app=llm-inference -o wide
# Both Pods must be Running on worker-01!
```

---

## Lab 4: High-Availability Pod Anti-Affinity, `PriorityClass` & `PodDisruptionBudget`

### Scenario & Requirements
1. Create a `PriorityClass` named `tier-0-critical` with `value: 1000000`, `globalDefault: false`, and description `"Critical payment gateway workloads"`.
2. Create a Deployment named `payment-core` with `2` replicas (`nginx:1.27`) in namespace `payments` that:
   - Uses `priorityClassName: tier-0-critical`.
   - Uses `podAntiAffinity` (`requiredDuringSchedulingIgnoredDuringExecution`) with `topologyKey: kubernetes.io/hostname` so that two replicas of `payment-core` are **never** co-located on the same node.
3. Create a `PodDisruptionBudget` named `payment-core-pdb` in `payments` ensuring at least `1` Pod (`minAvailable: 1`) of `app=payment-core` remains available during node drains.

### Step-by-Step Solution

#### 1. Create the `PriorityClass`
```bash
kubectl create priorityclass tier-0-critical \
  --value=1000000 \
  --description="Critical payment gateway workloads"
```

#### 2. Create the Deployment with Hard Pod Anti-Affinity (`payment-core.yaml`)
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: payment-core
  namespace: payments
spec:
  replicas: 2
  selector:
    matchLabels:
      app: payment-core
  template:
    metadata:
      labels:
        app: payment-core
    spec:
      priorityClassName: tier-0-critical
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - payment-core
            topologyKey: "kubernetes.io/hostname"
      containers:
      - name: payment-core
        image: nginx:1.27
```
```bash
kubectl apply -f payment-core.yaml
```

#### 3. Create the `PodDisruptionBudget`
```bash
kubectl create pdb payment-core-pdb \
  --selector=app=payment-core \
  --min-available=1 \
  -n payments
```

### Verification Commands
```bash
kubectl get pods -n payments -l app=payment-core -o wide
# Verify the 2 replicas are scheduled on 2 DIFFERENT nodes!
kubectl get pdb payment-core-pdb -n payments
# NAME               MIN AVAILABLE   MAX UNAVAILABLE   ALLOWED DISRUPTIONS   AGE
# payment-core-pdb   1               N/A               1                     15s
```

---

## Lab 5: Multi-Probe Configuration (`startupProbe`, `livenessProbe`, `readinessProbe`) & Secret/ConfigMap Mounts

### Scenario & Requirements
1. In namespace `default`, create a `ConfigMap` `webapp-settings` with literals `APP_MODE=production` and `PORT=80`, and a `ConfigMap` `webapp-html` with key `index.html` containing `"Welcome to CKA Production Portal"`.
2. Create a `Secret` `webapp-auth` with literal `API_TOKEN=tok_prod_998877`.
3. Create a Pod `resilient-webapp` (`nginx:1.27`) that:
   - Injects all keys from `webapp-settings` via `envFrom`.
   - Injects `API_TOKEN` from `webapp-auth` into environment variable `SECRET_API_TOKEN`.
   - Mounts `webapp-html` at `/usr/share/nginx/html` (read-only, `defaultMode: 0444`).
   - Configures a **`startupProbe`**: `httpGet` on `/index.html` port `80`, `periodSeconds: 3`, `failureThreshold: 10`.
   - Configures a **`livenessProbe`**: `tcpSocket` on port `80`, `initialDelaySeconds: 5`, `periodSeconds: 10`, `failureThreshold: 3`.
   - Configures a **`readinessProbe`**: `exec` running `["sh", "-c", "grep -q CKA /usr/share/nginx/html/index.html"]`, `periodSeconds: 5`.

### Step-by-Step Solution

#### 1. Create the ConfigMaps and Secret
```bash
kubectl create cm webapp-settings --from-literal=APP_MODE=production --from-literal=PORT=80
kubectl create cm webapp-html --from-literal=index.html="Welcome to CKA Production Portal"
kubectl create secret generic webapp-auth --from-literal=API_TOKEN=tok_prod_998877
```

#### 2. Create `resilient-webapp.yaml`
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: resilient-webapp
  namespace: default
spec:
  containers:
  - name: nginx
    image: nginx:1.27
    ports:
    - containerPort: 80
    envFrom:
    - configMapRef:
        name: webapp-settings
    env:
    - name: SECRET_API_TOKEN
      valueFrom:
        secretKeyRef:
          name: webapp-auth
          key: API_TOKEN
    volumeMounts:
    - name: html-vol
      mountPath: /usr/share/nginx/html
      readOnly: true
    startupProbe:
      httpGet:
        path: /index.html
        port: 80
      periodSeconds: 3
      failureThreshold: 10
    livenessProbe:
      tcpSocket:
        port: 80
      initialDelaySeconds: 5
      periodSeconds: 10
      failureThreshold: 3
    readinessProbe:
      exec:
        command:
        - sh
        - -c
        - "grep -q CKA /usr/share/nginx/html/index.html"
      periodSeconds: 5
  volumes:
  - name: html-vol
    configMap:
      name: webapp-html
      defaultMode: 0444
```
```bash
kubectl apply -f resilient-webapp.yaml
```

### Verification Commands
```bash
kubectl get pod resilient-webapp
kubectl exec resilient-webapp -- env | grep -E "APP_MODE|SECRET_API_TOKEN"
kubectl exec resilient-webapp -- curl -s http://localhost/index.html
```

---

## Lab 6: QoS `Guaranteed` Pods, `LimitRange`, `ResourceQuota` & HPA v2

### Scenario & Requirements
1. Create a namespace `quota-lab`.
2. Create a `ResourceQuota` `compute-quota` in `quota-lab` enforcing:
   - `requests.cpu: "2"`, `limits.cpu: "4"`
   - `requests.memory: 2Gi`, `limits.memory: 4Gi`
   - `pods: "6"`
3. Create a `LimitRange` `default-limits` in `quota-lab` for `type: Container` that sets:
   - `default` (limits): `cpu: 200m`, `memory: 256Mi`
   - `defaultRequest` (requests): `cpu: 100m`, `memory: 128Mi`
   - `max`: `cpu: "1"`, `memory: 1Gi`
4. Create a Pod `guaranteed-db` (`redis:7.2`) in `quota-lab` that achieves a **`Guaranteed`** QoS class with `cpu: 500m` and `memory: 512Mi`.
5. Create a Deployment `autoscale-api` (`nginx:1.27`, `1` replica) in `quota-lab` (relying on `LimitRange` for its default requests/limits) and configure an `autoscaling/v2` `HorizontalPodAutoscaler` `autoscale-api-hpa` with `minReplicas: 1`, `maxReplicas: 5`, targeting `65%` average CPU utilization and a scale-down stabilization window of `60` seconds.

### Step-by-Step Solution

#### 1. Create Namespace, `ResourceQuota`, and `LimitRange`
```bash
kubectl create ns quota-lab

kubectl create quota compute-quota -n quota-lab \
  --hard=requests.cpu=2,limits.cpu=4,requests.memory=2Gi,limits.memory=4Gi,pods=6
```

```yaml
# limitrange.yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: default-limits
  namespace: quota-lab
spec:
  limits:
  - type: Container
    default:
      cpu: 200m
      memory: 256Mi
    defaultRequest:
      cpu: 100m
      memory: 128Mi
    max:
      cpu: "1"
      memory: 1Gi
```
```bash
kubectl apply -f limitrange.yaml
```

#### 2. Create `guaranteed-db` Pod (`Guaranteed` QoS)
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: guaranteed-db
  namespace: quota-lab
spec:
  containers:
  - name: redis
    image: redis:7.2
    resources:
      requests:
        cpu: 500m
        memory: 512Mi
      limits:
        cpu: 500m
        memory: 512Mi
```
```bash
kubectl apply -f guaranteed-db.yaml
```

#### 3. Create Deployment and `autoscaling/v2` HPA (`hpa.yaml`)
```bash
kubectl create deploy autoscale-api --image=nginx:1.27 --replicas=1 -n quota-lab
```

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: autoscale-api-hpa
  namespace: quota-lab
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: autoscale-api
  minReplicas: 1
  maxReplicas: 5
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 65
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 60
```
```bash
kubectl apply -f hpa.yaml
```

### Verification Commands
```bash
# Verify QoS class of guaranteed-db is Guaranteed:
kubectl get pod guaranteed-db -n quota-lab -o jsonpath='{.status.qosClass}{"\n"}'
# Output: Guaranteed

# Verify autoscale-api pods inherited Burstable QoS from LimitRange defaults:
kubectl get pods -n quota-lab -l app=autoscale-api -o jsonpath='{.items[0].status.qosClass}{"\n"}'
# Output: Burstable

# Inspect ResourceQuota usage and HPA:
kubectl describe quota compute-quota -n quota-lab
kubectl get hpa autoscale-api-hpa -n quota-lab
```
