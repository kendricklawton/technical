# Domain 4: Storage — Production Hands-On Exercises & DigitalOcean Droplet Workbook

**Domain Weight:** 10% of the CKA Exam  
**Lab Environment:** 3-Node DigitalOcean Droplet Cluster (`cka-cp1`, `cka-worker1`, `cka-worker2` over DO VPC `10.116.0.0/20`)

---

## Exercise 1: Static PersistentVolume and PVC 1-to-1 Binding on Droplets

### Scenario
A legacy analytics application requires `5Gi` of persistent storage mounted at `/var/data` on `cka-worker1`. The storage must be retained even if the claim is deleted.

### Complete Solution (`storage-static.yaml`)
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: analytics
---
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
    type: DirectoryOrCreate
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
  nodeName: cka-worker1
  containers:
  - name: worker
    image: alpine:3.20
    command: ["sh", "-c", "echo 'DigitalOcean droplet storage test' >> /var/data/log.txt && sleep 3600"]
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
kubectl exec -n analytics analytics-worker -- cat /var/data/log.txt
```

---

## Exercise 2: Dynamic Provisioning with Topology Awareness (`WaitForFirstConsumer`)

### Scenario
Create two `StorageClasses` on your DigitalOcean cluster:
1. `do-block-delayed`: Uses the DigitalOcean Block Storage CSI driver (`dobs.csi.digitalocean.com`) with `WaitForFirstConsumer` and `allowVolumeExpansion: true`.
2. `droplet-local-delayed`: Uses `rancher.io/local-path` on your Droplets' NVMe disks, marked as the cluster's **default** StorageClass (`storageclass.kubernetes.io/is-default-class: "true"`).

### Complete Solution (`storageclasses.yaml`)
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: do-block-delayed
provisioner: dobs.csi.digitalocean.com
volumeBindingMode: WaitForFirstConsumer
allowVolumeExpansion: true
reclaimPolicy: Delete
parameters:
  fstype: ext4
---
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: droplet-local-delayed
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
provisioner: rancher.io/local-path
volumeBindingMode: WaitForFirstConsumer
reclaimPolicy: Delete
```

### Verification
```bash
kubectl apply -f storageclasses.yaml
kubectl get sc
# Confirm droplet-local-delayed shows "(default)"
```

---

## Exercise 3: Online Dynamic Volume Expansion

### Scenario
A PVC named `db-storage` in namespace `db` is at 90% capacity and needs to be resized from `10Gi` to `30Gi` without pod downtime.

### Execution Steps
```bash
# 1. Verify the StorageClass has allowVolumeExpansion: true (patch it if false)
SC_NAME=$(kubectl get pvc db-storage -n db -o jsonpath='{.spec.storageClassName}')
kubectl patch sc "$SC_NAME" -p '{"allowVolumeExpansion": true}'

# 2. Patch the PVC requested storage to 30Gi
kubectl patch pvc db-storage -n db -p '{"spec":{"resources":{"requests":{"storage":"30Gi"}}}}'

# 3. Verify resize completion in PVC events and status
kubectl describe pvc db-storage -n db
kubectl get pvc db-storage -n db
```

---

## Exercise 4: Local PersistentVolume with NodeAffinity on a Specific Droplet (`cka-worker2`)

### Scenario
You attached a DigitalOcean Block Storage Volume (or created a mount point `/mnt/disks/do-vol1` on `cka-worker2`). Create a `local` PersistentVolume `fast-droplet-ssd` of size `20Gi` that forces the Kubernetes scheduler to place any Pod claiming it strictly onto `cka-worker2`.

### Complete Solution (`local-pv.yaml`)
```bash
# Prepare directory on cka-worker2 first:
ssh cka-worker2 "sudo mkdir -p /mnt/disks/do-vol1"
```

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: fast-droplet-ssd
spec:
  capacity:
    storage: 20Gi
  volumeMode: Filesystem
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: local-nvme
  local:
    path: /mnt/disks/do-vol1
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - cka-worker2
```

---

## Exercise 5: Shared `ReadWriteMany` (`RWX`) NFS Volume Across Droplets Over `eth1` VPC & Reclaiming `Released` PVs

### Scenario
1. Configure a PersistentVolume `shared-assets-pv` (`ReadWriteMany`) backed by the NFSv4 server running on `cka-cp1`'s private VPC IP (`10.116.0.10:/srv/nfs/kubedata`).
2. Bind it to PVC `shared-assets-pvc` and mount it across a 3-replica Deployment spanning `cka-worker1` and `cka-worker2`.
3. Delete the PVC, observe the PV enter `Released`, and transition the PV back to `Available` without losing data.

### Complete Solution (`nfs-rwx.yaml`)
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: shared-assets-pv
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  storageClassName: nfs-vpc
  nfs:
    server: 10.116.0.10
    path: /srv/nfs/kubedata
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: shared-assets-pvc
  namespace: default
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: nfs-vpc
  resources:
    requests:
      storage: 10Gi
```

### Reclaiming the `Released` PV
```bash
# 1. Delete the PVC
kubectl delete pvc shared-assets-pvc -n default
kubectl get pv shared-assets-pv
# STATUS shows: Released

# 2. Clear spec.claimRef so the PV transitions back to Available
kubectl patch pv shared-assets-pv -p '{"spec":{"claimRef": null}}'
kubectl get pv shared-assets-pv
# STATUS shows: Available
```

---

## Exercise 6: CSI `VolumeSnapshot` & Snapshot Restoration (`dobs.csi.digitalocean.com`)

### Scenario
Create a `VolumeSnapshot` named `orders-snap-01` of an existing PVC `orders-pvc` in namespace `commerce` using `VolumeSnapshotClass` `dobs-snapclass`, and restore a new `20Gi` PVC named `orders-restored-pvc` initialized from that snapshot.

### Complete Solution (`snapshot-restore.yaml`)
```yaml
apiVersion: snapshot.storage.k8s.io/v1
kind: VolumeSnapshotClass
metadata:
  name: dobs-snapclass
driver: dobs.csi.digitalocean.com
deletionPolicy: Delete
---
apiVersion: snapshot.storage.k8s.io/v1
kind: VolumeSnapshot
metadata:
  name: orders-snap-01
  namespace: commerce
spec:
  volumeSnapshotClassName: dobs-snapclass
  source:
    persistentVolumeClaimName: orders-pvc
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: orders-restored-pvc
  namespace: commerce
spec:
  storageClassName: do-block-delayed
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi
  dataSource:
    name: orders-snap-01
    kind: VolumeSnapshot
    apiGroup: snapshot.storage.k8s.io
```

### Verification
```bash
kubectl get volumesnapshot -n commerce orders-snap-01
kubectl get pvc -n commerce orders-restored-pvc
```
