# CKA Full-Length Performance Practice Exam 4

**Time Limit:** 120 Minutes  
**Passing Score:** 66% (Score Target: $\ge 85\%$)  
**Topology:** DigitalOcean Droplet Multi-Node Simulation (`cka-cp1`, `cka-worker1`, `cka-worker2`) on Private VPC (`eth1`)  
**Format:** 17 Real-World Hands-On Performance Tasks with Full Solutions & Verification Commands

---

### Task 1: Kubelet TLS Bootstrap & CSR Approval Triage (Weight: 7%)
**Context:** `kubectl config use-context cka-cluster`  
**Scenario:** A newly joined DigitalOcean worker node `cka-worker2` has not appeared in `kubectl get nodes`. Inspection reveals the node submitted a Certificate Signing Request (CSR) that is currently pending approval.
**Requirements:**
1. List all CSRs: `kubectl get csr`.
2. Inspect the pending CSR for `cka-worker2`.
3. Approve the CSR manually using `kubectl certificate approve <csr-name>`.
4. Verify the node transitions to `Ready` status.

**Solution:**
```bash
kubectl get csr
# Approve pending CSR:
CSR_NAME=$(kubectl get csr --no-headers | grep Pending | awk '{print $1}')
kubectl certificate approve "$CSR_NAME"
kubectl get nodes -o wide
```

---

### Task 2: DigitalOcean Private VPC Route Fix for Cilium CNI (Weight: 8%)
**Context:** `ssh root@cka-worker1`  
**Scenario:** Pods on `cka-worker1` cannot reach pods on `cka-worker2`. Cilium was configured to encapsulate traffic over the public interface `eth0` instead of the internal DigitalOcean VPC interface `eth1` (`10.108.0.0/20`).
**Requirements:**
1. Inspect network interfaces on worker nodes: `ip addr show`.
2. Reconfigure Cilium Helm values or ConfigMap `cilium-config` in `kube-system` to set `devices: eth1` and tunnel protocol to `vxlan`.
3. Restart Cilium daemonset pods: `kubectl rollout restart ds/cilium -n kube-system`.
4. Validate pod-to-pod cross-node ping.

**Solution:**
```bash
kubectl -n kube-system edit cm cilium-config
# Update:
# devices: eth1
kubectl -n kube-system rollout restart ds cilium
kubectl -n kube-system rollout status ds cilium
# Test cross-node ping
kubectl run test-w1 --image=busybox --restart=Never --overrides='{"spec":{"nodeName":"cka-worker1"}}' -- ping -c 3 <pod-ip-on-w2>
```

---

### Task 3: CoreDNS Performance Tuning & Autoscaling (Weight: 6%)
**Context:** `kubectl config use-context cka-cluster`  
**Scenario:** Applications in the cluster are experiencing intermittent DNS lookup timeouts under heavy concurrency.
**Requirements:**
1. Scale the `coredns` deployment in `kube-system` to 3 replicas.
2. Edit the `coredns` ConfigMap to enable the `autopath` plugin before `kubernetes`.
3. Configure `cache` with a 30-second TTL.
4. Verify DNS resolution from a client pod.

**Solution:**
```bash
kubectl scale deployment coredns -n kube-system --replicas=3
kubectl edit cm coredns -n kube-system
# Inside Corefile:
# cache 30
kubectl rollout restart deployment coredns -n kube-system
```

---

### Task 4: PersistentVolume Reclaim Policy & Manual Data Recovery (Weight: 7%)
**Context:** `kubectl config use-context cka-cluster`  
**Scenario:** A developer accidentally deleted a PersistentVolumeClaim `db-data-pvc` in namespace `finance`. The associated PersistentVolume `pv-database` transitioned to `Released` status. The volume reclaim policy is currently `Delete`, putting the data at risk of total destruction.
**Requirements:**
1. Immediately patch the PV `pv-database` reclaim policy to `Retain`.
2. Clear the `spec.claimRef` UID on `pv-database` so it can be re-bound to a new PVC.
3. Create a replacement PVC named `db-data-recovered-pvc` in namespace `finance` requesting 10Gi, binding back to `pv-database`.

**Solution:**
```bash
kubectl patch pv pv-database -p '{"spec":{"persistentVolumeReclaimPolicy":"Retain"}}'
kubectl patch pv pv-database -p '{"spec":{"claimRef": null}}'

cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: db-data-recovered-pvc
  namespace: finance
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  volumeName: pv-database
EOF
```

---

### Task 5: Troubleshoot Broken Containerd Systemd Cgroup (Weight: 7%)
**Context:** `ssh root@cka-worker2`  
**Scenario:** After an OS package upgrade on `cka-worker2`, the kubelet fails to start with error: `misconfiguration: cgroup driver: cannot execute systemd`.
**Requirements:**
1. Edit `/etc/containerd/config.toml`.
2. Ensure `[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]` has `SystemdCgroup = true`.
3. Restart `containerd` and `kubelet`.
4. Verify node transitions back to `Ready`.

**Solution:**
```bash
sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml
systemctl restart containerd
systemctl restart kubelet
systemctl status kubelet
```

---

### Task 6: Deploy Multi-Container Pod with Logging Sidecar (Weight: 6%)
**Context:** `kubectl config use-context cka-cluster`  
**Scenario:** Deploy a pod named `app-logger` in namespace `production`.
**Requirements:**
1. Container 1 (`main`): image `busybox:1.36`, writing the current timestamp to `/var/log/app.log` every 5 seconds.
2. Container 2 (`sidecar`): image `busybox:1.36`, streaming `/var/log/app.log` to stdout (`tail -f /var/log/app.log`).
3. Both containers must share an `emptyDir` volume mounted at `/var/log`.

**Solution:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-logger
  namespace: production
spec:
  volumes:
    - name: shared-logs
      emptyDir: {}
  containers:
    - name: main
      image: busybox:1.36
      command: ["sh", "-c", "while true; do date >> /var/log/app.log; sleep 5; done"]
      volumeMounts:
        - name: shared-logs
          mountPath: /var/log
    - name: sidecar
      image: busybox:1.36
      command: ["sh", "-c", "tail -f /var/log/app.log"]
      volumeMounts:
        - name: shared-logs
          mountPath: /var/log
```

---

### Task 7: Configure Node Taints, Tolerations & Dedicated Scheduling (Weight: 6%)
**Context:** `kubectl config use-context cka-cluster`  
**Scenario:** Dedicate worker node `cka-worker1` exclusively to GPU batch processing jobs.
**Requirements:**
1. Apply taint: `workload=gpu:NoSchedule` to `cka-worker1`.
2. Create deployment `gpu-renderer` in namespace `batch` with 2 replicas, image `nginx:alpine`, and a toleration for the taint.
3. Verify that non-GPU pods are not scheduled on `cka-worker1`.

**Solution:**
```bash
kubectl taint nodes cka-worker1 workload=gpu:NoSchedule --overwrite
```
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: gpu-renderer
  namespace: batch
spec:
  replicas: 2
  selector:
    matchLabels:
      app: gpu-renderer
  template:
    metadata:
      labels:
        app: gpu-renderer
    spec:
      tolerations:
        - key: "workload"
          operator: "Equal"
          value: "gpu"
          effect: "NoSchedule"
      containers:
        - name: renderer
          image: nginx:alpine
```

---

### Task 8: Ingress Path-Based & Canary Traffic Routing (Weight: 6%)
**Context:** `kubectl config use-context cka-cluster`  
**Scenario:** Configure an Ingress resource `store-ingress` in namespace `ecommerce` exposing:
- `/api/v1` $\rightarrow$ service `store-v1` on port 80.
- `/api/v2` $\rightarrow$ service `store-v2` on port 80.
- Host: `store.company.internal`.

**Solution:**
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: store-ingress
  namespace: ecommerce
spec:
  ingressClassName: nginx
  rules:
    - host: store.company.internal
      http:
        paths:
          - path: /api/v1
            pathType: Prefix
            backend:
              service:
                name: store-v1
                port:
                  number: 80
          - path: /api/v2
            pathType: Prefix
            backend:
              service:
                name: store-v2
                port:
                  number: 80
```

---

### Task 9: JSONPath Advanced Querying & Node IP Extraction (Weight: 5%)
**Context:** `kubectl config use-context cka-cluster`  
**Scenario:** Query all nodes in the cluster and output their internal IP addresses and node names formatted as `<NODE_NAME>:<INTERNAL_IP>` sorted by node name, saving to `/opt/cka/node_ips.txt`.

**Solution:**
```bash
kubectl get nodes -o jsonpath='{range .items[*]}{.metadata.name}{":"}{range .status.addresses[?(@.type=="InternalIP")]}{.address}{"\n"}{end}{end}' | sort > /opt/cka/node_ips.txt
cat /opt/cka/node_ips.txt
```

---

### Task 10: Multi-Master etcd TLS Backup with Environment Tokens (Weight: 7%)
**Context:** `ssh root@cka-cp1`  
**Scenario:** Take an automated snapshot of etcd using `etcdctl` over TLS.
**Requirements:**
1. Endpoint: `https://127.0.0.1:2379`.
2. CACert: `/etc/kubernetes/pki/etcd/ca.crt`.
3. Cert: `/etc/kubernetes/pki/etcd/server.crt`.
4. Key: `/etc/kubernetes/pki/etcd/server.key`.
5. Destination: `/var/backups/etcd-backup-droplet.db`.
6. Verify snapshot status.

**Solution:**
```bash
ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  snapshot save /var/backups/etcd-backup-droplet.db

ETCDCTL_API=3 etcdctl --write-out=table snapshot status /var/backups/etcd-backup-droplet.db
```

---

### Task 11: Pod Disruption Budget with Rolling Deployment (Weight: 6%)
**Context:** `kubectl config use-context cka-cluster`  
**Scenario:** Protect a critical stateful payment deployment `payment-processor` in namespace `finance` with 4 replicas. Configure a `PodDisruptionBudget` ensuring at least 3 replicas are always available.

**Solution:**
```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: payment-pdb
  namespace: finance
spec:
  minAvailable: 3
  selector:
    matchLabels:
      app: payment-processor
```

---

### Task 12: NetworkPolicy Egress Isolation to External API (Weight: 6%)
**Context:** `kubectl config use-context cka-cluster`  
**Scenario:** In namespace `analytics`, restrict pod `report-generator` so it can only resolve DNS (UDP port 53) and communicate via HTTPS (TCP port 443) to IP CIDR `198.51.100.0/24`. All other egress and ingress must be denied.

**Solution:**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: restrict-report-generator
  namespace: analytics
spec:
  podSelector:
    matchLabels:
      app: report-generator
  policyTypes:
    - Ingress
    - Egress
  ingress: []
  egress:
    - to:
        - namespaceSelector: {}
          podSelector:
            matchLabels:
              k8s-app: kube-dns
      ports:
        - protocol: UDP
          port: 53
    - to:
        - ipBlock:
            cidr: 198.51.100.0/24
      ports:
        - protocol: TCP
          port: 443
```

---

### Task 13: Configure Static Pod on Specific Worker Node (Weight: 5%)
**Context:** `ssh root@cka-worker1`  
**Scenario:** Configure a static pod named `node-monitor` running image `nginx:1.25-alpine` directly on worker node `cka-worker1`.
**Requirements:**
1. Check kubelet configuration for `staticPodPath` (typically `/etc/kubernetes/manifests`).
2. Create manifest `node-monitor.yaml` in the static pod directory.
3. Ensure the pod appears in `kubectl get pods -A` with suffix `-cka-worker1`.

**Solution:**
```bash
grep -i staticPod /var/lib/kubelet/config.yaml
# Path: /etc/kubernetes/manifests

cat <<EOF > /etc/kubernetes/manifests/node-monitor.yaml
apiVersion: v1
kind: Pod
metadata:
  name: node-monitor
  namespace: kube-system
spec:
  containers:
    - name: monitor
      image: nginx:1.25-alpine
      ports:
        - containerPort: 80
EOF
```

---

### Task 14: Kube-Proxy IPVS Mode Migration (Weight: 6%)
**Context:** `kubectl config use-context cka-cluster`  
**Scenario:** The cluster network team requests migrating `kube-proxy` from legacy `iptables` mode to high-performance `ipvs` mode.
**Requirements:**
1. Edit ConfigMap `kube-proxy` in `kube-system`.
2. Change `mode: ""` to `mode: "ipvs"`.
3. Restart `kube-proxy` daemonset: `kubectl rollout restart ds/kube-proxy -n kube-system`.
4. Verify kube-proxy logs display `Using ipvs Proxier`.

**Solution:**
```bash
kubectl -n kube-system edit cm kube-proxy
# Set: mode: "ipvs"
kubectl -n kube-system rollout restart ds kube-proxy
kubectl -n kube-system rollout status ds kube-proxy
kubectl -n kube-system logs -l k8s-app=kube-proxy --tail=20 | grep -i ipvs
```

---

### Task 15: Troubleshoot CrashLooping Control Plane Static Pod (Weight: 7%)
**Context:** `ssh root@cka-cp1`  
**Scenario:** `kube-scheduler` static pod fails to start after an admin modified `/etc/kubernetes/manifests/kube-scheduler.yaml`.
**Requirements:**
1. Check static pod logs: `crictl ps -a` and `crictl logs <container-id>`.
2. Fix typo in flag: `--leader-elect=tru` $\rightarrow$ `--leader-elect=true`.
3. Ensure `kube-scheduler` pod restarts and becomes healthy.

**Solution:**
```bash
crictl ps -a | grep scheduler
sed -i 's/--leader-elect=tru$/--leader-elect=true/' /etc/kubernetes/manifests/kube-scheduler.yaml
sleep 5
kubectl get pods -n kube-system -l component=kube-scheduler
```

---

### Task 16: Configure Horizontal Pod Autoscaler (HPA v2) (Weight: 5%)
**Context:** `kubectl config use-context cka-cluster`  
**Scenario:** Auto-scale deployment `frontend-web` in namespace `web` between 2 and 10 replicas when average CPU utilization exceeds 70%.

**Solution:**
```bash
kubectl autoscale deployment frontend-web -n web --cpu-percent=70 --min=2 --max=10
kubectl get hpa -n web
```

---

### Task 17: Drain Worker Node for OS Kernel Patching (Weight: 5%)
**Context:** `kubectl config use-context cka-cluster`  
**Scenario:** Cordon and drain worker node `cka-worker2` for emergency maintenance. Ignore daemonsets and delete any emptyDir data if necessary. Once maintenance is done, uncordon the node.

**Solution:**
```bash
kubectl drain cka-worker2 --ignore-daemonsets --delete-emptydir-data --force
# Verify all non-daemonset pods moved:
kubectl get pods -A -o wide | grep cka-worker2
# Uncordon node:
kubectl uncordon cka-worker2
kubectl get nodes
```
