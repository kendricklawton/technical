# CKA Full-Length Performance Practice Exam 5

**Time Limit:** 120 Minutes  
**Passing Score:** 66% (Score Target: $\ge 85\%$)  
**Topology:** DigitalOcean Droplet Multi-Node Simulation (`cka-cp1`, `cka-worker1`, `cka-worker2`) on Private VPC (`eth1`)  
**Format:** 17 Real-World Hands-On Performance Tasks with Full Solutions & Verification Commands

---

### Task 1: Complete Cluster Certificate Expiration & Renewal (Weight: 8%)
**Context:** `ssh root@cka-cp1`  
**Scenario:** Control plane certificates have expired, causing `kubectl` commands to fail with `x509: certificate has expired or is not yet valid`.
**Requirements:**
1. Check certificate status using `kubeadm certs check-expiration`.
2. Renew all certificates using `kubeadm certs renew all`.
3. Restart control plane components (`kube-apiserver`, `kube-controller-manager`, `kube-scheduler`).
4. Update admin kubeconfig file `~/.kube/config` and verify cluster access.

**Solution:**
```bash
kubeadm certs check-expiration
kubeadm certs renew all
# Restart static pods by moving or touching manifests:
kill -s SIGHUP $(pgrep kube-apiserver) || docker/crictl restart static pods
# Copy renewed admin credentials:
cp /etc/kubernetes/admin.conf ~/.kube/config
chown $(id -u):$(id -g) ~/.kube/config
kubectl get nodes
```

---

### Task 2: Rebuilding etcd Data from Raw Snapshot on Fresh Volume (Weight: 8%)
**Context:** `ssh root@cka-cp1`  
**Scenario:** The etcd data directory `/var/lib/etcd` was corrupted by a disk failure. Restore the cluster from snapshot `/backups/etcd-production.db` into a new data directory `/var/lib/etcd-restored`.
**Requirements:**
1. Restore using `etcdctl snapshot restore --data-dir=/var/lib/etcd-restored`.
2. Update `/etc/kubernetes/manifests/etcd.yaml` hostPath volume to point to `/var/lib/etcd-restored`.
3. Verify `etcd` restarts and API server communicates successfully.

**Solution:**
```bash
ETCDCTL_API=3 etcdctl snapshot restore /backups/etcd-production.db \
  --data-dir=/var/lib/etcd-restored

sed -i 's|/var/lib/etcd|/var/lib/etcd-restored|g' /etc/kubernetes/manifests/etcd.yaml
sleep 10
kubectl get nodes
```

---

### Task 3: Ingress Controller TLS Termination (Weight: 6%)
**Context:** `kubectl config use-context cka-cluster`  
**Scenario:** Secure the customer portal `portal.company.org` with TLS.
**Requirements:**
1. Create a TLS Secret `portal-tls-secret` in namespace `portal` using existing cert `/opt/certs/portal.crt` and key `/opt/certs/portal.key`.
2. Configure an Ingress `portal-ingress` terminating TLS with this secret and forwarding traffic to service `portal-service` on port 80.

**Solution:**
```bash
kubectl create secret tls portal-tls-secret -n portal \
  --cert=/opt/certs/portal.crt \
  --key=/opt/certs/portal.key
```
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: portal-ingress
  namespace: portal
spec:
  ingressClassName: nginx
  tls:
    - hosts:
        - portal.company.org
      secretName: portal-tls-secret
  rules:
    - host: portal.company.org
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: portal-service
                port:
                  number: 80
```

---

### Task 4: Troubleshoot Broken Worker Node Kubelet Token (Weight: 7%)
**Context:** `ssh root@cka-worker1`  
**Scenario:** Worker node `cka-worker1` is in `NotReady` status. Kubelet logs report `bootstrap token expired or invalid`.
**Requirements:**
1. On `cka-cp1`, generate a fresh join token: `kubeadm token create --print-join-command`.
2. On `cka-worker1`, reset kubelet: `kubeadm reset -f`.
3. Re-join the node to the cluster using the generated command.
4. Verify the node returns to `Ready`.

**Solution:**
```bash
# On cp1:
kubeadm token create --print-join-command
# Output: kubeadm join 10.108.0.10:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash>

# On worker1:
kubeadm reset -f
kubeadm join 10.108.0.10:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash>
```

---

### Task 5: Pod PriorityClass & Preemption Configuration (Weight: 6%)
**Context:** `kubectl config use-context cka-cluster`  
**Scenario:** Create a high-priority class for critical platform pods so they can preempt lower-priority batch workloads when nodes are full.
**Requirements:**
1. Create a `PriorityClass` named `mission-critical` with value `1000000`, preemptionPolicy `PreemptLowerPriority`, and description `"Mission critical system workloads"`.
2. Deploy pod `critical-watchdog` in namespace `production` using this priority class.

**Solution:**
```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: mission-critical
value: 1000000
preemptionPolicy: PreemptLowerPriority
globalDefault: false
description: "Mission critical system workloads"
---
apiVersion: v1
kind: Pod
metadata:
  name: critical-watchdog
  namespace: production
spec:
  priorityClassName: mission-critical
  containers:
    - name: watchdog
      image: busybox:1.36
      command: ["sleep", "3600"]
```

---

### Task 6: Custom Metrics Server Installation & Verification (Weight: 6%)
**Context:** `kubectl config use-context cka-cluster`  
**Scenario:** `kubectl top nodes` and `kubectl top pods` fail with `error: Metrics API not available`.
**Requirements:**
1. Deploy Metrics Server: `kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml`.
2. Edit deployment `metrics-server` in `kube-system` to add `--kubelet-insecure-tls` to container args.
3. Verify `kubectl top nodes` successfully reports CPU and memory metrics.

**Solution:**
```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
kubectl patch deployment metrics-server -n kube-system --type='json' -p='[{"op":"add","path":"/spec/template/spec/containers/0/args/-","value":"--kubelet-insecure-tls"}]'
kubectl rollout status deployment metrics-server -n kube-system
kubectl top nodes
```

---

### Task 7: DigitalOcean CSI Block Storage Volume Provisioning (Weight: 6%)
**Context:** `kubectl config use-context cka-cluster`  
**Scenario:** Create a dynamic storage class `do-block-storage` using provisioner `dobs.csi.digitalocean.com` with `volumeBindingMode: WaitForFirstConsumer` and reclaim policy `Delete`.

**Solution:**
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: do-block-storage
provisioner: dobs.csi.digitalocean.com
volumeBindingMode: WaitForFirstConsumer
reclaimPolicy: Delete
allowVolumeExpansion: true
```

---

### Task 8: ServiceAccount & Token Projection Security (Weight: 5%)
**Context:** `kubectl config use-context cka-cluster`  
**Scenario:** Deploy a pod `api-client` in namespace `security` with `automountServiceAccountToken: false` in the pod spec, mounting a projected short-lived token volume with expiration of 1 hour (3600 seconds) instead.

**Solution:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: api-client
  namespace: security
spec:
  automountServiceAccountToken: false
  containers:
    - name: client
      image: busybox:1.36
      command: ["sleep", "3600"]
      volumeMounts:
        - name: token-vol
          mountPath: /var/run/secrets/tokens
  volumes:
    - name: token-vol
      projected:
        sources:
          - serviceAccountToken:
              path: token
              expirationSeconds: 3600
              audience: api-server
```

---

### Task 9: CoreDNS Custom Stub Domain Forwarding (Weight: 6%)
**Context:** `kubectl config use-context cka-cluster`  
**Scenario:** Reconfigure CoreDNS in `kube-system` to forward all DNS queries ending in `.corp.internal` to corporate DNS resolver `10.100.0.53:53`.

**Solution:**
```bash
kubectl edit cm coredns -n kube-system
# Add block before the default .:53 block:
# corp.internal:53 {
#     forward . 10.100.0.53
#     cache 30
# }
kubectl rollout restart deployment coredns -n kube-system
```

---

### Task 10: Multi-Container Pod with Ephemeral Debug Container (Weight: 5%)
**Context:** `kubectl config use-context cka-cluster`  
**Scenario:** A distroless production pod `distroless-app` in namespace `prod` has no shell (`/bin/sh` does not exist). Attach an ephemeral debugging container with image `busybox` to inspect its network namespace and process table.

**Solution:**
```bash
kubectl debug -it distroless-app -n prod \
  --image=busybox \
  --target=app \
  --share-processes
```

---

### Task 11: StatefulSet Headless Service & Stable Network Identifiers (Weight: 6%)
**Context:** `kubectl config use-context cka-cluster`  
**Scenario:** Deploy a 3-replica StatefulSet `elastic-node` in namespace `search` with headless service `elastic-svc` (clusterIP: None) ensuring pods get predictable DNS names `elastic-node-0.elastic-svc.search.svc.cluster.local`.

**Solution:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: elastic-svc
  namespace: search
spec:
  clusterIP: None
  selector:
    app: elastic
  ports:
    - port: 9200
      name: http
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: elastic-node
  namespace: search
spec:
  serviceName: "elastic-svc"
  replicas: 3
  selector:
    matchLabels:
      app: elastic
  template:
    metadata:
      labels:
        app: elastic
    spec:
      containers:
        - name: elastic
          image: busybox:1.36
          command: ["sleep", "3600"]
          ports:
            - containerPort: 9200
```

---

### Task 12: NetworkPolicy Ingress with Named Ports & IPBlocks (Weight: 6%)
**Context:** `kubectl config use-context cka-cluster`  
**Scenario:** In namespace `database`, restrict PostgreSQL pods (`app: postgres`) to only allow ingress traffic on named port `postgres` (5432) originating from pods labeled `role: api-backend` in namespace `api-gateway`.

**Solution:**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-api-to-postgres
  namespace: database
spec:
  podSelector:
    matchLabels:
      app: postgres
  policyTypes:
    - Ingress
  ingress:
    - from:
        - namespaceSelector:
            matchLabels:
              kubernetes.io/metadata.name: api-gateway
          podSelector:
            matchLabels:
              role: api-backend
      ports:
        - protocol: TCP
          port: 5432
```

---

### Task 13: Cordoning, Draining & Node Taint Elimination (Weight: 5%)
**Context:** `kubectl config use-context cka-cluster`  
**Scenario:** Worker node `cka-worker1` was marked `NoSchedule` with taint `maintenance=true:NoSchedule`. Once maintenance is verified complete, remove this taint and ensure pending pods schedule on it.

**Solution:**
```bash
kubectl taint nodes cka-worker1 maintenance=true:NoSchedule-
kubectl uncordon cka-worker1
kubectl get nodes
```

---

### Task 14: Recovering Misconfigured Kube-Controller-Manager Static Pod (Weight: 7%)
**Context:** `ssh root@cka-cp1`  
**Scenario:** `kube-controller-manager` fails to boot because an administrator incorrectly formatted the `--cluster-cidr` flag.
**Requirements:**
1. Check logs: `crictl ps -a | grep controller-manager`.
2. Inspect `/var/log/pods` or use `crictl logs`.
3. Correct flag to `--cluster-cidr=10.244.0.0/16` in `/etc/kubernetes/manifests/kube-controller-manager.yaml`.
4. Verify the pod starts and reports Running.

**Solution:**
```bash
sed -i 's/--cluster-cidr=.*$/--cluster-cidr=10.244.0.0\/16/' /etc/kubernetes/manifests/kube-controller-manager.yaml
sleep 5
kubectl get pods -n kube-system -l component=kube-controller-manager
```

---

### Task 15: DaemonSet Rolling Update Strategy (Weight: 5%)
**Context:** `kubectl config use-context cka-cluster`  
**Scenario:** Configure the DaemonSet `fluentd-logger` in namespace `logging` with `updateStrategy: RollingUpdate` and `maxUnavailable: 1`. Trigger an image update to `fluentd:v1.16` and monitor the rollout.

**Solution:**
```bash
kubectl patch ds fluentd-logger -n logging -p '{"spec":{"updateStrategy":{"type":"RollingUpdate","rollingUpdate":{"maxUnavailable":1}}}}'
kubectl set image ds/fluentd-logger fluentd=fluentd:v1.16 -n logging
kubectl rollout status ds/fluentd-logger -n logging
```

---

### Task 16: CronJob Concurrency Policy & History Limits (Weight: 5%)
**Context:** `kubectl config use-context cka-cluster`  
**Scenario:** Create a CronJob `db-indexer` in namespace `batch` running every 10 minutes (`*/10 * * * *`). Set `concurrencyPolicy: Forbid`, `successfulJobsHistoryLimit: 3`, and `failedJobsHistoryLimit: 1`.

**Solution:**
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: db-indexer
  namespace: batch
spec:
  schedule: "*/10 * * * *"
  concurrencyPolicy: Forbid
  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 1
  jobTemplate:
    spec:
      template:
        spec:
          restartPolicy: OnFailure
          containers:
            - name: indexer
              image: busybox:1.36
              command: ["sh", "-c", "echo Indexing complete"]
```

---

### Task 17: Linux crictl Container & Image Inspection (Weight: 5%)
**Context:** `ssh root@cka-worker2`  
**Scenario:** Using `crictl`, list all non-running stopped containers on the node and remove all unused container images to free disk space.

**Solution:**
```bash
crictl ps --state Exited
crictl rmi --prune
```
