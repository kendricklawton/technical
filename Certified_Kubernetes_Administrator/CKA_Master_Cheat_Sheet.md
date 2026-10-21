# CKA Master Cheat Sheet — High-Speed CLI & Exam Reference

This reference contains every imperative command, shell alias, JSONPath query, `etcdctl` operation, `kubeadm` upgrade workflow, and node debugging procedure needed to pass the **Certified Kubernetes Administrator (CKA)** exam with time to spare.

---

## 1. First 60 Seconds Exam Terminal Setup

Run this block immediately when your exam terminal opens:

```bash
# 1. Shell aliases & autocompletion
alias k=kubectl
complete -o default -F __start_kubectl k

# 2. High-speed environment variables
export do="--dry-run=client -o yaml"
export now="--force --grace-period=0"

# 3. Configure ~/.vimrc for clean YAML editing (2-space indent, expand tabs)
cat <<EOF > ~/.vimrc
set expandtab
set tabstop=2
set shiftwidth=2
set softtabstop=2
set autoindent
set smartindent
set number
set ruler
set paste
EOF

# 4. Persist aliases in ~/.bashrc in case you open a new shell or subshell
cat <<'EOF' >> ~/.bashrc
alias k=kubectl
complete -o default -F __start_kubectl k
export do="--dry-run=client -o yaml"
export now="--force --grace-period=0"
EOF
source ~/.bashrc
```

> [!TIP]
> **Using `kubectl explain` at Warp Speed:**
> Forget a field name? Don't scroll through browser docs!
> ```bash
> k explain pod.spec.containers.livenessProbe
> k explain deployment.spec.strategy.rollingUpdate
> k explain httproute.spec.rules --recursive
> k explain networkpolicy.spec.egress --recursive
> ```

---

## 2. Context, Namespace & Cluster Navigation

```bash
# List all kubeconfig contexts
k config get-contexts

# Switch active context (ALWAYS do this at the start of every exam task!)
k config use-context k8s-prod

# View current context
k config current-context

# Set default namespace for the current context (use with caution; reset to default after!)
k config set-context --current --namespace=finance

# Check cluster nodes and internal IPs
k get nodes -o wide

# View API resources, shortnames, API groups, and whether they are namespaced
k api-resources
k api-resources --namespaced=true
k api-resources --api-group=gateway.networking.k8s.io
```

---

## 3. Imperative Resource Generation (`kubectl run` & `kubectl create`)

### Pods
```bash
# Run a basic Nginx pod
k run web --image=nginx:1.27 --port=80

# Generate Pod YAML without creating it
k run web --image=nginx:1.27 --port=80 $do > web-pod.yaml

# Run a Pod with labels and environment variables
k run api --image=redis:7.2 -l tier=cache,env=prod --env="REDIS_PORT=6379"

# Run a Pod with custom command / arguments
k run sleeper --image=busybox:1.36 --command -- /bin/sh -c "sleep 3600"
k run sleeper --image=busybox:1.36 -- /bin/sh -c "while true; do date; sleep 5; done"

# Run a temporary ephemeral Pod to test DNS or curl an internal Service (auto-deletes on exit)
k run tmp-test --rm -it --image=busybox:1.36 --restart=Never -- nslookup my-service.default.svc.cluster.local
k run tmp-curl --rm -it --image=curlimages/curl:8.10.1 --restart=Never -- curl -sS http://10.96.45.12:8080/healthz

# Fast delete a stuck pod
k delete pod web $now
```

### Deployments, Scaling & Rollouts
```bash
# Create Deployment imperatively
k create deploy frontend --image=nginx:1.26 --replicas=3 --port=80

# Generate Deployment YAML to edit (e.g., to add affinity, volumes, or probes)
k create deploy frontend --image=nginx:1.26 --replicas=3 $do > deploy.yaml

# Scale a Deployment or StatefulSet
k scale deploy frontend --replicas=5

# Update container image and record revision annotation
k set image deploy/frontend nginx=nginx:1.27 --record

# Check rollout status & revision history
k rollout status deploy/frontend
k rollout history deploy/frontend
k rollout history deploy/frontend --revision=2

# Rollback to previous revision or specific revision
k rollout undo deploy/frontend
k rollout undo deploy/frontend --to-revision=1

# Pause and resume rollout (for batched changes)
k rollout pause deploy/frontend
k set resources deploy/frontend -c=nginx --limits=cpu=200m,memory=256Mi --requests=cpu=100m,memory=128Mi
k rollout resume deploy/frontend

# Restart all Pods in a Deployment (rolling restart)
k rollout restart deploy/frontend
```

### Services (`ClusterIP`, `NodePort`, Headless)
```bash
# Expose an existing Pod or Deployment as a ClusterIP Service
k expose pod web --name=web-svc --port=80 --target-port=80
k expose deploy frontend --name=frontend-svc --port=8080 --target-port=80 --type=ClusterIP

# Expose a Deployment as a NodePort Service
k expose deploy frontend --name=frontend-np --port=80 --target-port=80 --type=NodePort

# Create a Service directly (note: adjust selector labels if needed!)
k create service clusterip backend-svc --tcp=8080:8080
k create service nodeport ext-svc --tcp=80:80 --node-port=30080
k create service clusterip headless-svc --clusterip="None" $do > headless.yaml
```

### Jobs & CronJobs
```bash
# Create a one-off Job
k create job db-migrate --image=busybox:1.36 -- /bin/sh -c "echo Migrating DB && sleep 10"

# Generate a Job YAML to add completions: 5, parallelism: 2, backoffLimit: 4
k create job batch-worker --image=busybox:1.36 $do -- /bin/sh -c "sleep 5" > job.yaml

# Create a CronJob running every 5 minutes
k create cronjob log-cleaner --image=busybox:1.36 --schedule="*/5 * * * *" -- /bin/sh -c "date; echo cleaning"
```

### ConfigMaps & Secrets
```bash
# Create ConfigMap from literals, files, or env-files
k create cm app-config --from-literal=APP_ENV=production --from-literal=LOG_LEVEL=info
k create cm nginx-conf --from-file=/etc/nginx/nginx.conf
k create cm env-cm --from-env-file=app.env

# Create Generic Secret
k create secret generic db-secret --from-literal=DB_USER=admin --from-literal=DB_PASS='S3cr3tP@ss!'

# Create TLS Secret for Ingress or Gateway API
k create secret tls app-tls --cert=/path/to/tls.crt --key=/path/to/tls.key

# Create Docker Registry Secret for imagePullSecrets
k create secret docker-registry regcred \
  --docker-server=registry.example.com \
  --docker-username=myuser \
  --docker-password=mypass \
  --docker-email=myuser@example.com

# Decode a base64 Secret value quickly
k get secret db-secret -o jsonpath='{.data.DB_PASS}' | base64 -d ; echo
```

### RBAC (`ServiceAccount`, `Role`, `ClusterRole`, `RoleBinding`, `ClusterRoleBinding`)
```bash
# Create a ServiceAccount
k create sa cicd-bot -n dev

# Create a namespaced Role
k create role pod-reader --verb=get,list,watch --resource=pods,pods/log -n dev

# Create a RoleBinding binding a Role or ClusterRole to a User, Group, or ServiceAccount
k create rolebinding cicd-pod-reader \
  --role=pod-reader \
  --serviceaccount=dev:cicd-bot \
  -n dev

# Create a ClusterRole (cluster-scoped resources like nodes, persistentvolumes, or across all namespaces)
k create clusterrole node-admin --verb=get,list,watch,patch --resource=nodes
k create clusterrole deploy-scaler --verb=get,list,update,patch --resource=deployments,deployments/scale

# Create a ClusterRoleBinding
k create clusterrolebinding node-admin-binding \
  --clusterrole=node-admin \
  --user=jane \
  --group=sre-team \
  --serviceaccount=kube-system:node-controller

# Verify RBAC permissions using `auth can-i`
k auth can-i get pods -n dev --as=system:serviceaccount:dev:cicd-bot
k auth can-i delete nodes --as=jane
k auth can-i --list -n dev --as=system:serviceaccount:dev:cicd-bot
```

### Ingress, HPA, Quota & PDB Imperative Commands
```bash
# Create Ingress with host, path, and TLS
k create ingress web-ing \
  --class=nginx \
  --rule="app.example.com/api*=api-svc:8080,tls=app-tls" \
  --annotation nginx.ingress.kubernetes.io/rewrite-target=/

# Create HorizontalPodAutoscaler (autoscaling/v2)
k autoscale deploy frontend --cpu-percent=70 --min=2 --max=10

# Create PodDisruptionBudget
k create pdb frontend-pdb --selector=app=frontend --min-available=2
k create pdb backend-pdb --selector=app=backend --max-unavailable=25%

# Create PriorityClass
k create priorityclass high-priority --value=1000000 --description="Mission critical pods" --preemption-policy="PreemptLowerPriority"

# Create ResourceQuota
k create quota dev-quota --hard=cpu=4,memory=8Gi,pods=10,persistentvolumeclaims=5 -n dev
```

---

## 4. Labels, Annotations, Taints & Node Maintenance

```bash
# Label and unlabel resources (trailing minus `-` removes a label!)
k label node worker-01 disktype=nvme gpu=true
k label node worker-01 gpu-
k label pods -l app=v1 tier=frontend --overwrite

# Filter resources by label or field selector
k get pods -A -l env=prod,tier!=db
k get pods -A --field-selector=status.phase=Running
k get pods -A --field-selector=spec.nodeName=worker-01
k get events -A --sort-by='.metadata.creationTimestamp'

# Annotate resources
k annotate pod web description="Primary web frontend"

# Taint and untaint nodes (trailing minus `-` removes the taint!)
# Effects: NoSchedule | PreferNoSchedule | NoExecute
k taint nodes worker-01 dedicated=special-user:NoSchedule
k taint nodes worker-01 dedicated=special-user:NoExecute
k taint nodes worker-01 dedicated=special-user:NoSchedule-
k taint nodes worker-01 dedicated-   # Removes all taints with key 'dedicated'

# Inspect taints on all nodes
k describe nodes | grep -E "Name:|Taints:"

# Cordon, Drain, and Uncordon a Node (for maintenance or upgrades)
k cordon worker-01
k drain worker-01 --ignore-daemonsets --delete-emptydir-data
k uncordon worker-01
```

---

## 5. JSONPath, Custom Columns & Sorting Recipes

```bash
# 1. Get all Node names and their Internal IPs
k get nodes -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.status.addresses[?(@.type=="InternalIP")].address}{"\n"}{end}'

# 2. Custom columns output (often much easier to write than range JSONPath!)
k get nodes -o custom-columns=NODE:.metadata.name,IP:.status.addresses[0].address,KERNEL:.status.nodeInfo.kernelVersion,OS:.status.nodeInfo.osImage

# 3. List all Pods and their Container Images across the cluster
k get pods -A -o custom-columns=NAMESPACE:.metadata.namespace,POD:.metadata.name,IMAGES:.spec.containers[*].image

# 4. Sort PersistentVolumes by capacity
k get pv --sort-by=.spec.capacity.storage

# 5. Sort Pods by restart count
k get pods -A --sort-by='.status.containerStatuses[0].restartCount'

# 6. Find top CPU / Memory consuming Pods and Nodes
k top nodes --sort-by=cpu
k top nodes --sort-by=memory
k top pods -A --sort-by=cpu
k top pods -n prod -l app=payments --containers
```

---

## 6. `etcd` Backup & Restore Workflow (Domain 1 — High Yield!)

### Step 1: Extract `etcd` Flags from `/etc/kubernetes/manifests/etcd.yaml`
```bash
sudo grep -E "listen-client-urls|cert-file|key-file|trusted-ca-file|data-dir" /etc/kubernetes/manifests/etcd.yaml
```

### Step 2: Take an `etcd` Snapshot Backup
```bash
ETCDCTL_API=3 sudo etcdctl snapshot save /opt/etcd-backup.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key

# Verify the snapshot integrity
ETCDCTL_API=3 sudo etcdctl snapshot status /opt/etcd-backup.db --write-out=table
# (Or on newer distros with etcdutl):
sudo etcdutl snapshot status /opt/etcd-backup.db --write-out=table
```

### Step 3: Restore `etcd` Snapshot to a New Directory
```bash
# 1. Restore snapshot into a NEW host directory (e.g., /var/lib/etcd-from-backup)
ETCDCTL_API=3 sudo etcdctl snapshot restore /opt/etcd-backup.db \
  --data-dir=/var/lib/etcd-from-backup

# 2. Edit /etc/kubernetes/manifests/etcd.yaml so the volume `etcd-data` hostPath points to the new directory:
# Under volumes:
# - hostPath:
#     path: /var/lib/etcd-from-backup
#     type: DirectoryOrCreate
#   name: etcd-data
sudo vi /etc/kubernetes/manifests/etcd.yaml

# 3. Wait 15-30 seconds for kubelet to recreate the etcd static pod and verify:
sudo crictl ps | grep etcd
k get pods -n kube-system
```

---

## 7. `kubeadm` Cluster Minor Version Upgrade Checklist (e.g., `1.31.0` -> `1.31.1`)

### On the Control Plane Node:
```bash
# 1. Drain the control plane node
kubectl drain cp-node --ignore-daemonsets

# 2. SSH to the control plane node and update package index
ssh cp-node
sudo apt-get update

# 3. Check available versions in the Kubernetes apt repo
sudo apt-cache madison kubeadm

# 4. Upgrade kubeadm first
sudo apt-mark unhold kubeadm
sudo apt-get install -y kubeadm=1.31.1-1.1
sudo apt-mark hold kubeadm

# 5. Verify upgrade plan and apply the control plane upgrade
sudo kubeadm version
sudo kubeadm upgrade plan
sudo kubeadm upgrade apply v1.31.1   # Wait for "[upgrade/successful] SUCCESS!"
# (Note: Check if question says `--etcd-upgrade=false`!)

# 6. Upgrade kubelet and kubectl
sudo apt-mark unhold kubelet kubectl
sudo apt-get install -y kubelet=1.31.1-1.1 kubectl=1.31.1-1.1
sudo apt-mark hold kubelet kubectl

# 7. Reload systemd and restart kubelet
sudo systemctl daemon-reload
sudo systemctl restart kubelet
exit

# 8. Uncordon the control plane node
kubectl uncordon cp-node
```

### On a Worker Node:
```bash
# 1. From jump host: drain the worker node
kubectl drain worker-01 --ignore-daemonsets --delete-emptydir-data

# 2. SSH to worker-01 and upgrade kubeadm
ssh worker-01
sudo apt-get update
sudo apt-mark unhold kubeadm
sudo apt-get install -y kubeadm=1.31.1-1.1
sudo apt-mark hold kubeadm

# 3. Upgrade local kubelet configuration on the worker node (NOTE: `upgrade node`, NOT `upgrade apply`!)
sudo kubeadm upgrade node

# 4. Upgrade kubelet and kubectl, then restart kubelet
sudo apt-mark unhold kubelet kubectl
sudo apt-get install -y kubelet=1.31.1-1.1 kubectl=1.31.1-1.1
sudo apt-mark hold kubelet kubectl
sudo systemctl daemon-reload
sudo systemctl restart kubelet
exit

# 5. From jump host: uncordon the worker node and verify Ready + new version
kubectl uncordon worker-01
kubectl get nodes
```

---

## 8. Helm & Kustomize Essential Commands

### Helm
```bash
# Manage Helm repositories
helm repo list
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update

# Search charts and inspect values
helm search repo nginx --versions
helm show values bitnami/nginx > custom-values.yaml

# Install, upgrade, list, and rollback releases
helm list -A
helm list -n web --pending          # Show stuck pending-install / pending-upgrade releases!
helm install my-web bitnami/nginx -n web --create-namespace --version 18.1.0 -f custom-values.yaml --set replicaCount=3
helm upgrade my-web bitnami/nginx -n web --set image.tag=1.27.0
helm history my-web -n web
helm rollback my-web 1 -n web
helm uninstall my-web -n web

# Render chart templates locally without installing
helm template my-web bitnami/nginx -n web --set service.type=NodePort > rendered.yaml
```

### Kustomize
```bash
# Preview rendered manifests from a directory containing kustomization.yaml
k kustomize /path/to/overlays/production

# Apply Kustomize directory to cluster
k apply -k /path/to/overlays/production

# Delete resources managed by Kustomize directory
k delete -k /path/to/overlays/staging
```

---

## 9. Kubernetes Gateway API Quick Templates (`gateway.networking.k8s.io/v1`)

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: prod-gateway
  namespace: default
spec:
  gatewayClassName: nginx-gateway-class
  listeners:
  - name: http
    protocol: HTTP
    port: 80
    allowedRoutes:
      namespaces:
        from: Same
  - name: https
    protocol: HTTPS
    port: 443
    hostname: "shop.example.com"
    tls:
      mode: Terminate
      certificateRefs:
      - kind: Secret
        name: shop-tls-secret
---
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: shop-route
  namespace: default
spec:
  parentRefs:
  - name: prod-gateway
  hostnames:
  - "shop.example.com"
  rules:
  - matches:
    - path:
        type: PathPrefix
        value: /v2
      headers:
      - name: X-Canary
        value: "true"
    backendRefs:
    - name: shop-v2-svc
      port: 8080
      weight: 100
  - matches:
    - path:
        type: PathPrefix
        value: /
    backendRefs:
    - name: shop-v1-svc
      port: 8080
      weight: 80
    - name: shop-v2-svc
      port: 8080
      weight: 20
```

---

## 10. Control Plane & Node Troubleshooting Triage Commands

```bash
# 1. When kube-apiserver is DOWN (kubectl returns "connection refused" on port 6443):
sudo ls -la /etc/kubernetes/manifests/
sudo crictl ps -a                       # List all containers including Exited/Failed ones!
sudo crictl logs <container-id>         # Read why kube-apiserver or etcd crashed!
sudo tail -n 50 /var/log/pods/kube-system_kube-apiserver-*/*/*.log
sudo tail -n 50 /var/log/pods/kube-system_etcd-*/*/*.log

# 2. Check and renew expired control-plane certificates:
sudo kubeadm certs check-expiration
sudo kubeadm certs renew all

# 3. When a Worker Node is NotReady:
ssh worker-01
sudo systemctl status kubelet
sudo systemctl status containerd
sudo journalctl -u kubelet -xe --no-pager | tail -n 50
# Check common culprits:
# - Is kubelet stopped or disabled? -> sudo systemctl enable --now kubelet
# - Is containerd stopped? -> sudo systemctl enable --now containerd
# - Is swap enabled? -> free -m ; sudo swapoff -a ; sudo sed -i '/ swap / s/^/#/' /etc/fstab
# - Is kubelet binary path wrong in /etc/systemd/system/kubelet.service.d/10-kubeadm.conf? (check `which kubelet`)
# - Is server port/IP wrong in /etc/kubernetes/kubelet.conf? (should match kube-apiserver :6443)
# - Is clientCAFile path wrong in /var/lib/kubelet/config.yaml?
```

---

## 11. Crucial Filesystem Paths on Kubernetes Nodes

| File / Directory Path | Purpose |
| :--- | :--- |
| `/etc/kubernetes/manifests/` | Default `staticPodPath` watched by `kubelet` (`kube-apiserver.yaml`, `etcd.yaml`, `kube-controller-manager.yaml`, `kube-scheduler.yaml`) |
| `/etc/kubernetes/pki/` | Cluster PKI certificates & private keys (`ca.crt`, `apiserver.crt`, `etcd/ca.crt`, `etcd/server.crt`) |
| `/etc/kubernetes/admin.conf` | Superuser `kubeconfig` for cluster admin (`cp /etc/kubernetes/admin.conf ~/.kube/config`) |
| `/etc/kubernetes/kubelet.conf` | `kubelet` kubeconfig containing API server endpoint and kubelet client credentials |
| `/var/lib/kubelet/config.yaml` | Primary `KubeletConfiguration` file (`staticPodPath`, `cgroupDriver`, `clusterDNS`, `clientCAFile`) |
| `/var/lib/kubelet/pki/` | `kubelet` client/server certificates (`kubelet-client-current.pem`) |
| `/etc/systemd/system/kubelet.service.d/10-kubeadm.conf` | Systemd drop-in unit defining `kubelet` startup flags and binary location |
| `/var/lib/etcd/` | Default `etcd` data directory on stacked control plane nodes |
| `/etc/cni/net.d/` | CNI plugin configuration files (lexicographically first `.conf`/`.conflist` is loaded) |
| `/opt/cni/bin/` | CNI plugin executable binaries (`calico`, `flannel`, `loopback`, `host-local`, `portmap`) |
| `/etc/containerd/config.toml` | `containerd` CRI configuration file (`SystemdCgroup = true`, sandbox image) |
| `/var/log/pods/` & `/var/log/containers/` | Pod and container stdout/stderr log files on the node filesystem |
