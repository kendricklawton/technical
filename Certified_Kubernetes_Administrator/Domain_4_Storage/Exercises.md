# Domain 4: Storage — Production Hands-On Exercises & Solutions

## Exercise 1: Static PersistentVolume and PVC 1-to-1 Binding
### Scenario
A legacy analytics application requires 5Gi of persistent local storage mounted at `/var/data`. The storage must be retained even if the claim is deleted.

### Complete Solution (`storage-static.yaml`)
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: analytics-pv-01
spec:
  capacity:
    storage: 5Gi
  volumeMode: Filesystem
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: local-manual
  hostPath:
    path: /opt/analytics-data
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: analytics-pvc-01
  namespace: analytics
spec:
  accessModes:
  - ReadWriteOnce
  storageClassName: local-manual
  resources:
    requests:
      storage: 5Gi
---
apiVersion: v1
kind: Pod
metadata:
  name: analytics-worker
  namespace: analytics
spec:
  containers:
  - name: worker
    image: alpine:3.18
    command: ["sh", "-c", "echo 'storage test' >> /var/data/log.txt && sleep 3600"]
    volumeMounts:
    - name: data-vol
      mountPath: /var/data
  volumes:
  - name: data-vol
    persistentVolumeClaim:
      claimName: analytics-pvc-01
```

### Verification
```bash
kubectl apply -f storage-static.yaml
kubectl get pv,pvc -n analytics
# Verify PV status is 'Bound' to analytics/analytics-pvc-01
```

---

## Exercise 2: Dynamic Provisioning with Topology Awareness (`WaitForFirstConsumer`)
### Scenario
Create a StorageClass that delays volume binding until pod scheduling to avoid Multi-AZ EBS volume attachment errors.

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: topology-gp3
provisioner: ebs.csi.aws.com
volumeBindingMode: WaitForFirstConsumer
allowVolumeExpansion: true
parameters:
  type: gp3
```

---

## Exercise 3: Online Dynamic Volume Expansion
### Scenario
A PVC named `db-storage` in namespace `db` is at 90% capacity and needs to be resized from 10Gi to 30Gi without downtime.

```bash
# Verify StorageClass allows expansion
kubectl get sc $(kubectl get pvc db-storage -n db -o jsonpath='{.spec.storageClassName}') -o jsonpath='{.allowVolumeExpansion}'

# Patch PVC size
kubectl patch pvc db-storage -n db -p '{"spec":{"resources":{"requests":{"storage":"30Gi"}}}}'

# Watch filesystem expansion
kubectl describe pvc db-storage -n db
```

---

## Exercise 4: Local PersistentVolume with NodeAffinity
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: fast-local-ssd
spec:
  capacity:
    storage: 100Gi
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: local-nvme
  local:
    path: /mnt/disks/ssd1
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - worker-node-02
```

---

## Exercise 5: Reclaiming a Retained PV for a New Claim
1. Delete PVC: `kubectl delete pvc analytics-pvc-01 -n analytics`.
2. Notice PV status is `Released`.
3. Clear claim reference:
   ```bash
   kubectl patch pv analytics-pv-01 -p '{"spec":{"claimRef": null}}'
   ```
4. PV returns to `Available` status and can be bound to new PVCs.

---

## Exercise 6: VolumeSnapshot & Snapshot Restoration
Create a snapshot of `orders-pvc` and restore it into `orders-restored-pvc`.
