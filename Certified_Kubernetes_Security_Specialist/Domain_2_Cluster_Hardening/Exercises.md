# CKS Domain 2: Cluster Hardening — Hands-On Security Labs

These 6 hands-on labs cover the high-probability scenarios tested in **Domain 2 (Cluster Hardening - 15%)** of the CKS exam.

---

## Lab 1: Hardening `kube-apiserver` Authentication, Authorization & Admission

### Scenario
Inspect the control plane node (`controlplane`) and harden `/etc/kubernetes/manifests/kube-apiserver.yaml` according to the following security requirements:
1. Anonymous authentication to the API server must be disabled.
2. Any unauthenticated `system:anonymous` `ClusterRoleBinding` that was left behind in the cluster (`system:anonymous-admin`) must be deleted.
3. The API server's authorization mode must be set to `Node,RBAC` (remove `AlwaysAllow` if present).
4. The `NodeRestriction` admission controller plugin must be enabled without removing existing admission plugins.

### Step-by-Step Solution

#### Step 1: Audit and Delete the Rogue `ClusterRoleBinding`
```bash
# Find ClusterRoleBindings involving system:anonymous
kubectl get clusterrolebindings -o wide | grep anonymous
kubectl delete clusterrolebinding system:anonymous-admin
```

#### Step 2: Backup and Edit `/etc/kubernetes/manifests/kube-apiserver.yaml`
```bash
mkdir -p /root/backups
cp /etc/kubernetes/manifests/kube-apiserver.yaml /root/backups/kube-apiserver.yaml.bak
vi /etc/kubernetes/manifests/kube-apiserver.yaml
```
Update the flags under `spec.containers[0].command`:
```yaml
    - command:
        - kube-apiserver
        - --anonymous-auth=false
        - --authorization-mode=Node,RBAC
        - --enable-admission-plugins=NodeRestriction
```

#### Step 3: Verify API Server Restart and Anonymous Rejection
```bash
# Wait 15-30 seconds for the kubelet to recreate kube-apiserver
crictl ps | grep kube-apiserver
kubectl get nodes

# Test an unauthenticated curl request to the API server: must return 401 Unauthorized!
curl -k https://127.0.0.1:6443/api/v1/namespaces
```

---

## Lab 2: RBAC Least-Privilege Audit, Wildcard Removal & Scope Reduction

### Scenario
In the `inventory` namespace, a Deployment named `stock-tracker` runs using ServiceAccount `stock-sa`.
1. Currently, a `ClusterRoleBinding` named `stock-global-binding` binds `stock-sa` to a `ClusterRole` named `stock-overprivileged-role` which grants `verbs: ["*"]` on `resources: ["*"]`.
2. Inspect the `stock-tracker` pod logs (`kubectl logs -n inventory deploy/stock-tracker`) and determine what API calls the application actually makes. You observe it only runs `GET` and `LIST` requests against `pods` and `services` inside the `inventory` namespace.
3. Delete `stock-global-binding` and `stock-overprivileged-role`.
4. Create a namespace-scoped `Role` named `stock-least-priv-role` in `inventory` that grants **only** `get` and `list` verbs on `pods` and `services` in the core API group (`""`).
5. Bind `stock-least-priv-role` to `stock-sa` in `inventory` using a `RoleBinding` named `stock-least-priv-binding`.

### Step-by-Step Solution

```bash
# 1. Inspect existing binding, role, and pod logs
kubectl get clusterrolebinding stock-global-binding -o yaml
kubectl get clusterrole stock-overprivileged-role -o yaml
kubectl logs -n inventory deploy/stock-tracker

# 2. Delete the overprivileged ClusterRoleBinding and ClusterRole
kubectl delete clusterrolebinding stock-global-binding
kubectl delete clusterrole stock-overprivileged-role

# 3. Create the least-privilege Role and RoleBinding in 'inventory'
kubectl -n inventory create role stock-least-priv-role \
  --verb=get,list \
  --resource=pods,services

kubectl -n inventory create rolebinding stock-least-priv-binding \
  --role=stock-least-priv-role \
  --serviceaccount=inventory:stock-sa
```

#### Verification
```bash
# Must return 'yes':
kubectl auth can-i get pods --as=system:serviceaccount:inventory:stock-sa -n inventory
kubectl auth can-i list services --as=system:serviceaccount:inventory:stock-sa -n inventory

# Must return 'no':
kubectl auth can-i get secrets --as=system:serviceaccount:inventory:stock-sa -n inventory
kubectl auth can-i delete pods --as=system:serviceaccount:inventory:stock-sa -n inventory
kubectl auth can-i get pods --as=system:serviceaccount:inventory:stock-sa -n kube-system
```

---

## Lab 3: ServiceAccount Token Hardening & Projected Bound Token Volumes

### Scenario
In the `billing` namespace:
1. Modify the existing `default` ServiceAccount so that it does **not** automatically mount API credentials (`automountServiceAccountToken: false`).
2. Create a new dedicated ServiceAccount named `invoice-sa` in `billing` with `automountServiceAccountToken: false`.
3. Update the existing Deployment `invoice-worker` in `billing` (manifest saved at `/root/invoice-worker.yaml`) so that:
   * It uses ServiceAccount `invoice-sa`.
   * It mounts a projected bound `serviceAccountToken` volume named `vault-token-vol` at `/var/run/secrets/vault` (read-only) with token file path `token`, audience `https://vault.billing.svc`, and an expiration time of `7200` seconds (2 hours).

### Step-by-Step Solution

#### Step 1: Harden `default` SA and Create `invoice-sa`
```bash
kubectl -n billing patch serviceaccount default -p '{"automountServiceAccountToken": false}'

cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: ServiceAccount
metadata:
  name: invoice-sa
  namespace: billing
automountServiceAccountToken: false
EOF
```

#### Step 2: Update `/root/invoice-worker.yaml`
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: invoice-worker
  namespace: billing
spec:
  replicas: 1
  selector:
    matchLabels:
      app: invoice-worker
  template:
    metadata:
      labels:
        app: invoice-worker
    spec:
      serviceAccountName: invoice-sa
      automountServiceAccountToken: false
      containers:
        - name: worker
          image: busybox:1.36
          command: ["sh", "-c", "sleep 3600"]
          volumeMounts:
            - name: vault-token-vol
              mountPath: /var/run/secrets/vault
              readOnly: true
      volumes:
        - name: vault-token-vol
          projected:
            sources:
              - serviceAccountToken:
                  path: token
                  expirationSeconds: 7200
                  audience: https://vault.billing.svc
```

Apply and verify:
```bash
kubectl apply -f /root/invoice-worker.yaml
kubectl -n billing rollout status deploy/invoice-worker

# Verify default token path does NOT exist, and projected token DOES exist:
kubectl -n billing exec deploy/invoice-worker -- ls -la /var/run/secrets/kubernetes.io/serviceaccount
# Expected: ls: /var/run/secrets/kubernetes.io/serviceaccount: No such file or directory

kubectl -n billing exec deploy/invoice-worker -- ls -la /var/run/secrets/vault/token
```

---

## Lab 4: Restricting Secret Access to Specific `resourceNames`

### Scenario
In the `auth-system` namespace, three Secrets exist: `oauth-client-secret`, `jwt-signing-key`, and `root-db-password`.
A Pod named `oauth-proxy` runs with ServiceAccount `oauth-sa`. Currently, `Role` `secret-reader` in `auth-system` allows `get`, `list`, and `watch` on all `secrets`.
Modify `Role` `secret-reader` so that `oauth-sa` can **only** `get` the single Secret named `oauth-client-secret` and cannot `list`, `watch`, or read any other Secret in the namespace.

### Step-by-Step Solution

```yaml
# /root/secret-reader-role.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: secret-reader
  namespace: auth-system
rules:
  - apiGroups: [""]
    resources: ["secrets"]
    verbs: ["get"]
    resourceNames: ["oauth-client-secret"]
```

Apply and verify:
```bash
kubectl apply -f /root/secret-reader-role.yaml

# Must return 'yes':
kubectl auth can-i get secret/oauth-client-secret --as=system:serviceaccount:auth-system:oauth-sa -n auth-system

# Must return 'no':
kubectl auth can-i get secret/root-db-password --as=system:serviceaccount:auth-system:oauth-sa -n auth-system
kubectl auth can-i list secrets --as=system:serviceaccount:auth-system:oauth-sa -n auth-system
```

---

## Lab 5: Protecting Dedicated High-Security Nodes with `NodeRestriction`

### Scenario
Your cluster has a worker node `node01` dedicated to PCI-DSS workloads.
1. Ensure the `NodeRestriction` admission plugin is enabled in `/etc/kubernetes/manifests/kube-apiserver.yaml`.
2. Label `node01` with `node-restriction.kubernetes.io/pci-compliance=true` so that even if the `kubelet` on `node02` is compromised, `node02` cannot self-assign this label via its `system:node:node02` credentials.

### Step-by-Step Solution

```bash
# 1. Verify NodeRestriction is in --enable-admission-plugins in /etc/kubernetes/manifests/kube-apiserver.yaml
grep enable-admission-plugins /etc/kubernetes/manifests/kube-apiserver.yaml

# 2. Apply the protected node-restriction.kubernetes.io/* label as cluster-admin
kubectl label node node01 node-restriction.kubernetes.io/pci-compliance=true

# 3. Verify that node01 cannot modify or delete its own node-restriction.kubernetes.io/* label when impersonating system:node:node01
kubectl label node node01 node-restriction.kubernetes.io/pci-compliance=false --overwrite \
  --as=system:node:node01 --as-group=system:nodes
# Expected error: nodes "node01" is forbidden: is not allowed to modify labels: node-restriction.kubernetes.io/pci-compliance
```

---

## Lab 6: Zero-Downtime Worker Node Security Upgrade (`kubeadm`)

### Scenario
A critical CVE has been disclosed in Kubernetes `v1.31.0`. The control plane (`controlplane`) has already been upgraded to `v1.31.1`, while worker node `node01` is still running `v1.31.0`.
1. Safely drain `node01` of all workloads (ignoring DaemonSets and deleting local emptyDir data).
2. Upgrade `kubeadm`, the node configuration, `kubelet`, and `kubectl` on `node01` to `v1.31.1` (`1.31.1-1.1`).
3. Restart `kubelet` and uncordon `node01` so it is `Ready` and schedulable.

### Step-by-Step Solution

```bash
# 1. From controlplane: drain node01
kubectl drain node01 --ignore-daemonsets --delete-emptydir-data

# 2. SSH to node01 and upgrade packages
ssh node01 << 'EOF'
apt-get update
apt-mark unhold kubeadm kubelet kubectl
apt-get install -y kubeadm=1.31.1-1.1
kubeadm upgrade node
apt-get install -y kubelet=1.31.1-1.1 kubectl=1.31.1-1.1
apt-mark hold kubeadm kubelet kubectl
systemctl daemon-reload
systemctl restart kubelet
EOF

# 3. From controlplane: uncordon node01 and verify version
kubectl uncordon node01
kubectl get nodes
```
