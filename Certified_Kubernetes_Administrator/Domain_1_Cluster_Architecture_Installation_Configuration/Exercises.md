# Domain 1: Cluster Architecture, Installation & Configuration — Hands-On Lab Exercises

This lab guide contains **6 production-grade, exam-realistic CKA scenarios** covering RBAC, `kubeadm` cluster upgrades, `etcd` snapshot backup & restore, Helm release management, Kustomize overlays/patches, and Custom Resource Definitions (CRDs).

---

## Lab 1: Multi-Namespace RBAC with `ClusterRole`, `RoleBinding`, and `ServiceAccount`

### Scenario & Requirements
Your security team requires a dedicated monitoring automation account that can inspect Pods, Pod logs, Deployments, and StatefulSets in the `payments` and `inventory` namespaces, plus list all cluster Nodes, without having permission to modify or delete workloads.

1. Create namespaces `payments` and `inventory` if they do not exist.
2. Create a `ServiceAccount` named `ops-inspector` in the `payments` namespace.
3. Create a `ClusterRole` named `workload-auditor` that grants `get`, `list`, and `watch` on:
   - `pods` and `pods/log` (core API group)
   - `deployments` and `statefulsets` (`apps` API group)
4. Create a second `ClusterRole` named `node-viewer` that grants `get`, `list`, and `watch` on `nodes`.
5. Bind `workload-auditor` to `ops-inspector` (from namespace `payments`) inside **both** the `payments` and `inventory` namespaces using namespaced `RoleBindings` (`auditor-payments-rb` and `auditor-inventory-rb`).
6. Bind `node-viewer` to `ops-inspector` cluster-wide using a `ClusterRoleBinding` named `ops-node-viewer-crb`.
7. Create a Pod named `inspector-pod` in namespace `payments` running `bitnami/kubectl:latest` (command `sleep 3600`) configured to use the `ops-inspector` `ServiceAccount`.

### Step-by-Step Solution

#### 1. Create Namespaces & ServiceAccount
```bash
kubectl create namespace payments
kubectl create namespace inventory
kubectl create serviceaccount ops-inspector -n payments
```

#### 2. Create the `ClusterRoles`
```bash
kubectl create clusterrole workload-auditor \
  --verb=get,list,watch \
  --resource=pods,pods/log,deployments.apps,statefulsets.apps

kubectl create clusterrole node-viewer \
  --verb=get,list,watch \
  --resource=nodes
```

#### 3. Create the `RoleBindings` and `ClusterRoleBinding`
```bash
# Bind ClusterRole workload-auditor to ops-inspector in 'payments' namespace
kubectl create rolebinding auditor-payments-rb \
  --clusterrole=workload-auditor \
  --serviceaccount=payments:ops-inspector \
  -n payments

# Bind ClusterRole workload-auditor to ops-inspector in 'inventory' namespace
kubectl create rolebinding auditor-inventory-rb \
  --clusterrole=workload-auditor \
  --serviceaccount=payments:ops-inspector \
  -n inventory

# Bind ClusterRole node-viewer cluster-wide
kubectl create clusterrolebinding ops-node-viewer-crb \
  --clusterrole=node-viewer \
  --serviceaccount=payments:ops-inspector
```

#### 4. Deploy `inspector-pod` with the ServiceAccount
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: inspector-pod
  namespace: payments
spec:
  serviceAccountName: ops-inspector
  containers:
  - name: kubectl
    image: bitnami/kubectl:latest
    command: ["/bin/sh", "-c", "sleep 3600"]
```
```bash
kubectl apply -f inspector-pod.yaml
```

### Verification Commands
```bash
# Should return "yes":
kubectl auth can-i get pods/log -n payments --as=system:serviceaccount:payments:ops-inspector
kubectl auth can-i list statefulsets.apps -n inventory --as=system:serviceaccount:payments:ops-inspector
kubectl auth can-i get nodes --as=system:serviceaccount:payments:ops-inspector

# Should return "no" (scoped out of kube-system and no delete verb):
kubectl auth can-i get pods -n kube-system --as=system:serviceaccount:payments:ops-inspector
kubectl auth can-i delete deployments -n payments --as=system:serviceaccount:payments:ops-inspector
```

---

## Lab 2: Zero-Downtime Control Plane & Worker Node Minor Version Upgrade with `kubeadm`

### Scenario & Requirements
You are managing a 2-node cluster (`cka-cp1` and `cka-worker1`) currently running Kubernetes `v1.31.0`. Upgrade the entire cluster to `v1.31.2` with zero workload downtime:
1. Drain the control plane node (`cka-cp1`) safely before upgrading.
2. Upgrade `kubeadm`, the control plane components (**including `etcd`**), `kubelet`, and `kubectl` on `cka-cp1` to `1.31.2`, then uncordon `cka-cp1`.
3. Drain `cka-worker1`, upgrade `kubeadm`, local kubelet configuration, `kubelet`, and `kubectl` on `cka-worker1` to `1.31.2`, and uncordon `cka-worker1`.

### Step-by-Step Solution

#### Phase A: Upgrade the Control Plane Node (`cka-cp1`)
```bash
# 1. Cordon and drain cka-cp1 from the jump host
kubectl drain cka-cp1 --ignore-daemonsets

# 2. SSH into cka-cp1
ssh cka-cp1
sudo apt-get update

# 3. Check exact package version string in apt cache
sudo apt-cache madison kubeadm | grep 1.31.2
# Output: kubeadm | 1.31.2-1.1 | https://pkgs.k8s.io/core:/stable:/v1.31/deb/ Packages

# 4. Unhold, upgrade, and re-hold kubeadm
sudo apt-mark unhold kubeadm
sudo apt-get install -y kubeadm=1.31.2-1.1
sudo apt-mark hold kubeadm

# 5. Verify kubeadm version and run upgrade plan
sudo kubeadm version
sudo kubeadm upgrade plan

# 6. Apply the upgrade to the control plane
sudo kubeadm upgrade apply v1.31.2 -y

# 7. Unhold, upgrade, and re-hold kubelet and kubectl
sudo apt-mark unhold kubelet kubectl
sudo apt-get install -y kubelet=1.31.2-1.1 kubectl=1.31.2-1.1
sudo apt-mark hold kubelet kubectl

# 8. Reload systemd daemon and restart kubelet
sudo systemctl daemon-reload
sudo systemctl restart kubelet
exit

# 9. Uncordon cka-cp1
kubectl uncordon cka-cp1
```

#### Phase B: Upgrade the Worker Node (`cka-worker1`)
```bash
# 1. Drain cka-worker1 from the jump host
kubectl drain cka-worker1 --ignore-daemonsets --delete-emptydir-data

# 2. SSH into cka-worker1
ssh cka-worker1
sudo apt-get update

# 3. Upgrade kubeadm on worker node
sudo apt-mark unhold kubeadm
sudo apt-get install -y kubeadm=1.31.2-1.1
sudo apt-mark hold kubeadm

# 4. Upgrade node kubelet configuration (Notice: `upgrade node`, NOT `upgrade apply`!)
sudo kubeadm upgrade node

# 5. Upgrade kubelet and kubectl on worker node
sudo apt-mark unhold kubelet kubectl
sudo apt-get install -y kubelet=1.31.2-1.1 kubectl=1.31.2-1.1
sudo apt-mark hold kubelet kubectl

# 6. Restart kubelet service
sudo systemctl daemon-reload
sudo systemctl restart kubelet
exit

# 7. Uncordon cka-worker1
kubectl uncordon cka-worker1
```

### Verification Commands
```bash
kubectl get nodes -o wide
# Both cka-cp1 and cka-worker1 must show STATUS=Ready and VERSION=v1.31.2
```

---

## Lab 3: `etcd` Snapshot Backup and Disaster Recovery Restore

### Scenario & Requirements
1. Take a snapshot backup of the running `etcd` instance on `cka-cp1` and save it to `/var/backups/etcd-snapshot-pre-change.db`.
2. Simulate a catastrophic accidental deletion by creating a test namespace `critical-prod` with a Secret `api-keys` BEFORE the backup, then deleting the `critical-prod` namespace AFTER the backup.
3. Restore the `etcd` snapshot into `/var/lib/etcd-restored` and update the `etcd` Static Pod manifest so the cluster recovers the deleted `critical-prod` namespace and `api-keys` Secret.

### Step-by-Step Solution

#### 1. Create Test Workload & Inspect `etcd` Static Pod Certificates
```bash
kubectl create ns critical-prod
kubectl create secret generic api-keys --from-literal=STRIPE_KEY=sk_live_987654321 -n critical-prod

# Inspect /etc/kubernetes/manifests/etcd.yaml on cka-cp1
ssh cka-cp1
sudo grep -E "cert-file|key-file|trusted-ca-file|listen-client-urls" /etc/kubernetes/manifests/etcd.yaml
```

#### 2. Save the `etcd` Snapshot
```bash
sudo mkdir -p /var/backups
ETCDCTL_API=3 sudo etcdctl snapshot save /var/backups/etcd-snapshot-pre-change.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key

# Verify the snapshot status
ETCDCTL_API=3 sudo etcdctl snapshot status /var/backups/etcd-snapshot-pre-change.db --write-out=table
```

#### 3. Simulate Accidental Deletion
```bash
kubectl delete ns critical-prod
kubectl get ns critical-prod
# Error from server (NotFound): namespaces "critical-prod" not found
```

#### 4. Restore the Snapshot to `/var/lib/etcd-restored`
```bash
ETCDCTL_API=3 sudo etcdctl snapshot restore /var/backups/etcd-snapshot-pre-change.db \
  --data-dir=/var/lib/etcd-restored

# Verify the restored directory exists and contains member/snap/db
sudo ls -la /var/lib/etcd-restored/member
```

#### 5. Point `/etc/kubernetes/manifests/etcd.yaml` to `/var/lib/etcd-restored`
Edit `/etc/kubernetes/manifests/etcd.yaml` and change the `etcd-data` `hostPath` volume path:
```yaml
  volumes:
  - hostPath:
      path: /etc/kubernetes/pki/etcd
      type: DirectoryOrCreate
    name: etcd-certs
  - hostPath:
      path: /var/lib/etcd-restored   # <-- Updated from /var/lib/etcd
      type: DirectoryOrCreate
    name: etcd-data
```

### Verification Commands
```bash
# Wait 15-30 seconds for kubelet to restart the etcd static pod
sudo crictl ps | grep etcd
kubectl get ns critical-prod
kubectl get secret api-keys -n critical-prod -o jsonpath='{.data.STRIPE_KEY}' | base64 -d ; echo
# Output: sk_live_987654321
```

---

## Lab 4: Managing Releases and Customizing Chart Values with Helm

### Scenario & Requirements
1. Add the official Bitnami Helm repository (`https://charts.bitnami.com/bitnami`) under the name `bitnami` and update your local repo index.
2. Search for the `bitnami/nginx` chart and extract its default `values.yaml` to `/root/nginx-default-values.yaml`.
3. Install a Helm release named `edge-gateway` in the `ingress-edge` namespace (create the namespace automatically) using `bitnami/nginx` with:
   - `replicaCount=2`
   - `service.type=NodePort`
   - `service.nodePorts.http=30088`
4. Upgrade the `edge-gateway` release to scale `replicaCount=3` while keeping all previously configured custom values intact.
5. Roll back the `edge-gateway` release to revision `1` and verify replicas return to `2`.

### Step-by-Step Solution

```bash
# 1. Add and update repository
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update

# 2. Save default chart values
helm show values bitnami/nginx > /root/nginx-default-values.yaml

# 3. Install release with custom parameters
helm install edge-gateway bitnami/nginx \
  --namespace ingress-edge \
  --create-namespace \
  --set replicaCount=2 \
  --set service.type=NodePort \
  --set service.nodePorts.http=30088

# 4. Upgrade release to 3 replicas while reusing existing values
helm upgrade edge-gateway bitnami/nginx \
  --namespace ingress-edge \
  --reuse-values \
  --set replicaCount=3

# Verify revision 2 has 3 replicas
helm history edge-gateway -n ingress-edge
kubectl get deploy -n ingress-edge

# 5. Rollback to revision 1
helm rollback edge-gateway 1 -n ingress-edge
```

### Verification Commands
```bash
helm list -n ingress-edge
helm history edge-gateway -n ingress-edge
kubectl get deploy,svc -n ingress-edge
# Deployment shows 2/2 Ready; Service shows NodePort 80:30088/TCP (Revision 3 active after rollback to 1)
```

---

## Lab 5: Multi-Environment Deployment with Kustomize Overlays & Patches

### Scenario & Requirements
You are given a base application directory `/root/k8s-kustomize/base` containing a `Deployment` (`order-api`, `replicas: 1`, image `nginx:1.25`) and a `Service` (`order-api`).
Create a `production` overlay in `/root/k8s-kustomize/overlays/production` that:
1. Sets the target namespace to `prod-orders` and adds a `namePrefix` of `prod-`.
2. Adds common labels `environment: production` and `managed-by: kustomize`.
3. Overrides the container image tag from `nginx:1.25` to `nginx:1.27-alpine`.
4. Uses a **Strategic Merge Patch** to set `replicas: 4` and add memory limits (`256Mi`) and requests (`128Mi`) to the `order-api` container.
5. Uses a **JSON 6902 Patch** to add the annotation `prometheus.io/scrape: "true"` to the `Service`.
6. Render the final YAML to `/root/prod-orders-rendered.yaml` and apply the overlay to the cluster.

### Step-by-Step Solution

#### 1. Create the Base Manifests (`/root/k8s-kustomize/base`)
```bash
mkdir -p /root/k8s-kustomize/base /root/k8s-kustomize/overlays/production
kubectl create ns prod-orders
```

```yaml
# /root/k8s-kustomize/base/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: order-api
spec:
  replicas: 1
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
        image: nginx:1.25
        ports:
        - containerPort: 80
---
# /root/k8s-kustomize/base/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: order-api
  annotations:
    service.beta.kubernetes.io/topology: "auto"
spec:
  selector:
    app: order-api
  ports:
  - port: 80
    targetPort: 80
```

```yaml
# /root/k8s-kustomize/base/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - deployment.yaml
  - service.yaml
```

#### 2. Create the Production Overlay (`/root/k8s-kustomize/overlays/production/kustomization.yaml`)
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

namespace: prod-orders
namePrefix: prod-

resources:
  - ../../base

labels:
  - pairs:
      environment: production
      managed-by: kustomize
    includeSelectors: false

images:
  - name: nginx
    newTag: 1.27-alpine

patches:
  # Strategic Merge Patch for Deployment replicas and resources
  - patch: |-
      apiVersion: apps/v1
      kind: Deployment
      metadata:
        name: order-api
      spec:
        replicas: 4
        template:
          spec:
            containers:
            - name: order-api
              resources:
                requests:
                  memory: "128Mi"
                  cpu: "100m"
                limits:
                  memory: "256Mi"
                  cpu: "250m"
  # JSON 6902 Patch for Service annotation
  - target:
      version: v1
      kind: Service
      name: order-api
    patch: |-
      - op: add
        path: /metadata/annotations/prometheus.io~1scrape
        value: "true"
```
*(Note: In JSON Pointer RFC 6901, a forward slash `/` inside a key name like `prometheus.io/scrape` is escaped as `~1`!)*

#### 3. Render and Apply
```bash
kubectl kustomize /root/k8s-kustomize/overlays/production > /root/prod-orders-rendered.yaml
kubectl apply -k /root/k8s-kustomize/overlays/production
```

### Verification Commands
```bash
kubectl get deploy,svc -n prod-orders
kubectl get deploy prod-order-api -n prod-orders -o jsonpath='{.spec.template.spec.containers[0].image}{"\n"}'
# Output: nginx:1.27-alpine
kubectl get svc prod-order-api -n prod-orders -o jsonpath='{.metadata.annotations.prometheus\.io/scrape}{"\n"}'
# Output: true
```

---

## Lab 6: Creating a Custom Resource Definition (CRD) with OpenAPI v3 Schema Validation

### Scenario & Requirements
1. Create a Namespaced `CustomResourceDefinition` named `databases.platform.example.com` in API group `platform.example.com`, version `v1`, Kind `Database`, singular `database`, plural `databases`, and shortName `db`.
2. Enforce an OpenAPI v3 schema on `spec` requiring:
   - `engine` (`string`, enum restricted to `"postgres"`, `"mysql"`, `"redis"`)
   - `version` (`string`)
   - `storageGB` (`integer`, minimum `10`, maximum `2000`)
   - `highAvailability` (`boolean`, default `false`)
3. Create a valid `Database` custom resource named `orders-postgres` in the `default` namespace with `engine: postgres`, `version: "16.4"`, `storageGB: 100`, `highAvailability: true`.
4. Verify that attempting to create a `Database` with `storageGB: 5` is rejected by `kube-apiserver` schema validation.

### Step-by-Step Solution

#### 1. Create the CRD Manifest (`crd-database.yaml`)
```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: databases.platform.example.com
spec:
  group: platform.example.com
  scope: Namespaced
  names:
    plural: databases
    singular: database
    kind: Database
    shortNames:
    - db
  versions:
  - name: v1
    served: true
    storage: true
    schema:
      openAPIV3Schema:
        type: object
        properties:
          spec:
            type: object
            required: ["engine", "version", "storageGB"]
            properties:
              engine:
                type: string
                enum: ["postgres", "mysql", "redis"]
              version:
                type: string
              storageGB:
                type: integer
                minimum: 10
                maximum: 2000
              highAvailability:
                type: boolean
                default: false
    additionalPrinterColumns:
    - name: Engine
      type: string
      jsonPath: .spec.engine
    - name: Version
      type: string
      jsonPath: .spec.version
    - name: Storage(GB)
      type: integer
      jsonPath: .spec.storageGB
    - name: HA
      type: boolean
      jsonPath: .spec.highAvailability
```

#### 2. Apply the CRD and Create the Custom Resource (`orders-postgres.yaml`)
```bash
kubectl apply -f crd-database.yaml
```

```yaml
apiVersion: platform.example.com/v1
kind: Database
metadata:
  name: orders-postgres
  namespace: default
spec:
  engine: postgres
  version: "16.4"
  storageGB: 100
  highAvailability: true
```
```bash
kubectl apply -f orders-postgres.yaml
```

### Verification Commands
```bash
kubectl get db
# NAME              ENGINE     VERSION   STORAGE(GB)   HA
# orders-postgres   postgres   16.4      100           true

kubectl explain db.spec

# Test invalid storageGB (5 < minimum 10):
cat <<EOF | kubectl apply -f -
apiVersion: platform.example.com/v1
kind: Database
metadata:
  name: invalid-db
spec:
  engine: redis
  version: "7.2"
  storageGB: 5
EOF
# Expected: The Database "invalid-db" is invalid: spec.storageGB: Invalid value: 5: spec.storageGB in body should be greater than or equal to 10
```
