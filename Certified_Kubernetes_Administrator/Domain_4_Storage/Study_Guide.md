# Domain 4: Storage — In-Depth Master Study Guide

**Exam Weight:** 10% of CKA  
**Target Mastery:** PersistentVolumes (PV), PersistentVolumeClaims (PVC), StorageClasses (`dobs.csi.digitalocean.com`, `rancher.io/local-path`, NFS), CSI Dynamic Provisioning, Access Modes (`RWO`, `ROX`, `RWX`, `RWOP`), Reclaim Policies (`Retain`, `Delete`), Online PVC Volume Expansion, Local Persistent Volumes with NodeAffinity on Droplets, VolumeSnapshots, and VolumeMounts.

---

## 1. Kubernetes Storage Architecture & PV/PVC Lifecycle

Kubernetes decouples storage infrastructure configuration (managed by cluster administrators) from storage consumption (requested by application developers) through PersistentVolumes and PersistentVolumeClaims.

```mermaid
flowchart LR
    Admin["Cluster Admin / DigitalOcean CSI"] -->|Provisions| StorageClass["StorageClass<br/>(do-block-storage, local-path, nfs)"]
    StorageClass -->|Dynamic Provisioning| PV["PersistentVolume (PV)<br/>Cluster-Scoped"]
    Dev["Developer"] -->|Submits| PVC["PersistentVolumeClaim (PVC)<br/>Namespace-Scoped"]
    PVC -->|Binds 1-to-1| PV
    Pod["Workload Pod on Droplet"] -->|Mounts PVC| Container["Container Filesystem<br/>/var/lib/data"]
```

### The 4 Phases of the PV/PVC Lifecycle
1. **Provisioning:**
   - **Static Provisioning:** An administrator manually creates PersistentVolume objects pointing to attached DigitalOcean Block Storage Volumes (`/dev/disk/by-id/scsi-0DO_Volume_*`), NFS shares over the Droplet `eth1` VPC network, or local NVMe directories.
   - **Dynamic Provisioning:** A PVC references a `StorageClass` (`dobs.csi.digitalocean.com` or `rancher.io/local-path`). The Container Storage Interface (CSI) provisioner automatically provisions the backend volume and creates a matching PV.
2. **Binding:**
   - The master control-plane (`pv-controller`) watches for unbound PVCs and finds a matching PV by checking:
     - Storage capacity (PV capacity $\ge$ PVC requested capacity).
     - AccessModes compatibility.
     - StorageClassName match (`spec.storageClassName`).
     - VolumeMode (`Filesystem` vs `Block`).
   - The binding is strictly **exclusive and 1-to-1**. A PV bound to one PVC cannot bind to another, even if excess storage remains!
3. **Using:**
   - Pod manifests reference the claim: `volumes: [persistentVolumeClaim: {claimName: my-pvc}]`.
   - The `kubelet` attaches the volume (via CSI Node plugin) and mounts the filesystem into the container target directory.
4. **Reclaiming:**
   - When a user deletes a PVC, the bound PV transitions to the `Released` status.
   - The `persistentVolumeReclaimPolicy` governs what happens next.

---

## 2. Access Modes & Reclaim Policies

### Access Modes Deep Dive

| Access Mode | CLI Shorthand | Mount Capability | DigitalOcean Droplet & Cloud Backends |
|---|---|---|---|
| **ReadWriteOnce** | `RWO` | Mounted as read-write by a **single node**. Multiple pods on the *same Droplet* can access it simultaneously. | DigitalOcean Block Storage Volumes (`dobs.csi.digitalocean.com`), `local-path` on Droplet NVMe, iSCSI. |
| **ReadOnlyMany** | `ROX` | Mounted as read-only by **many nodes** simultaneously. | NFS Server over DO VPC `eth1`, CephFS, Read-only Object Storage mount. |
| **ReadWriteMany** | `RWX` | Mounted as read-write by **many nodes** simultaneously. | NFSv4 Server over DO VPC (`10.116.0.10`), CephFS / Rook. |
| **ReadWriteOncePod** | `RWOP` | Mounted as read-write by a **single Pod** across the entire cluster. | CSI drivers supporting `ReadWriteOncePod` (including `dobs.csi.digitalocean.com` on Kubernetes 1.27+ GA). |

> [!WARNING]
> **Exam Trap:** `ReadWriteOnce` means **Once per NODE**, not once per pod! Multiple pods scheduled on the *same* physical worker Droplet can mount an `RWO` volume concurrently. If you require single-pod isolation across the entire cluster, use `ReadWriteOncePod`.

### PersistentVolume Reclaim Policies
- **`Retain`:**
  - When the PVC is deleted, the PV is NOT deleted.
  - PV status changes from `Bound` to `Released`.
  - The underlying physical data remains completely untouched.
  - Another PVC **cannot** immediately bind to this PV because `spec.claimRef` still contains the UID of the deleted claim. An administrator must manually scrub data or edit the PV to clear `claimRef`.
- **`Delete` (Default for dynamically provisioned volumes):**
  - When the PVC is deleted, the Kubernetes PV object AND the underlying DigitalOcean Block Storage Volume are deleted.
- **`Recycle` (Deprecated):**
  - Performs an automated `rm -rf /volume/*` wipe and makes the PV available for rebinding.

---

## 3. StorageClasses & CSI Dynamic Provisioning on DigitalOcean

A `StorageClass` defines the "provisioner", volume parameters, and binding behavior.

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: do-block-storage
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
provisioner: dobs.csi.digitalocean.com
volumeBindingMode: WaitForFirstConsumer
allowVolumeExpansion: true
reclaimPolicy: Delete
parameters:
  fstype: ext4
```

### `volumeBindingMode` Deep Dive
- **`Immediate` (Default if omitted):**
  - The CSI provisioner immediately creates the physical volume and Kubernetes PV as soon as the PVC is submitted.
  - **The Topology / Region Scheduling Disaster:** When using node-local storage (`rancher.io/local-path`) or zonal cloud block storage (`dobs.csi.digitalocean.com`), `Immediate` binding selects a node/topology before knowing where the Pod will be scheduled. If the Pod has a `nodeSelector` or `nodeAffinity` for `cka-worker2` while the volume was created on `cka-worker1`, the pod gets stuck forever in `Pending` with error `VolumeNodeAffinityConflict`!
- **`WaitForFirstConsumer` (Best Practice):**
  - Delays PV provisioning and binding until a Pod referencing the PVC is scheduled.
  - The scheduler evaluates node taints, affinities, and capacity *first*, selects `cka-worker2`, and then signals the CSI driver / `local-path-provisioner` to provision and attach the volume on `cka-worker2`!

---

## 4. Online PVC Volume Expansion

Kubernetes supports dynamic expansion of file systems (`dobs.csi.digitalocean.com` supports online `NodeExpandVolume` for `ext4` and `xfs`) without restarting workloads if the StorageClass has `allowVolumeExpansion: true`.

### Step-by-Step Volume Expansion Workflow:
1. **Verify StorageClass allows expansion:**
   `allowVolumeExpansion: true` must be present on the StorageClass.
2. **Edit the PVC:**
   ```bash
   kubectl patch pvc data-claim -p '{"spec":{"resources":{"requests":{"storage":"50Gi"}}}}'
   ```
   *(Note: You can ONLY increase storage capacity; shrinking volumes is unsupported).*
3. **Verify Status:**
   - The PVC status shows `FileSystemResizePending` while the block device and `ext4`/`xfs` filesystem expand on the worker Droplet.
   - Once completed, `kubectl get pvc data-claim` reflects the new storage size (`50Gi`).

---

## 5. Local Persistent Volumes & Node Affinity on Droplets

For ultra-low latency workloads (or when attaching a dedicated DigitalOcean Volume at `/mnt/volume_nyc3_01` on `cka-worker2`), Kubernetes supports **Local Persistent Volumes** backed by local NVMe or attached block storage.

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: droplet-nvme-pv
spec:
  capacity:
    storage: 20Gi
  volumeMode: Filesystem
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: local-storage
  local:
    path: /mnt/disks/nvme1
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - cka-worker2
```

> [!IMPORTANT]
> A Local Persistent Volume **MUST** specify `nodeAffinity`. It binds storage to a specific Droplet hostname (`cka-worker2`) so the Kubernetes scheduler only schedules consuming pods onto that exact host.

---

## 6. VolumeSnapshots & Snapshot Controllers

The External Snapshotter CRDs (`snapshot.storage.k8s.io/v1`) allow point-in-time snapshotting and restoration of CSI volumes (`dobs.csi.digitalocean.com`).

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
  name: postgres-backup-snap
  namespace: databases
spec:
  volumeSnapshotClassName: dobs-snapclass
  source:
    persistentVolumeClaimName: postgres-data-pvc
---
# Restoring a PVC from VolumeSnapshot
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgres-restored-pvc
  namespace: databases
spec:
  storageClassName: do-block-storage
  dataSource:
    name: postgres-backup-snap
    kind: VolumeSnapshot
    apiGroup: snapshot.storage.k8s.io
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 50Gi
```

---

## 7. CKA Exam Traps & Troubleshooting Guide

1. **PVC Stuck in `Pending`:**
   - Case A: StorageClass uses `volumeBindingMode: WaitForFirstConsumer`. This is NORMAL behavior until a pod referencing the PVC is created.
   - Case B: No PV matches the PVC's requested size, `storageClassName`, or `accessModes`.
   - Case C: CSI plugin or `local-path-provisioner` pod is crashing or not running in `kube-system`/`local-path-storage`.
2. **Releasing Retained PV for Reuse:**
   - When a PV has status `Released` under `Retain` policy:
     `kubectl patch pv <pv-name> -p '{"spec":{"claimRef": null}}'` clears the claim reference and returns the PV to `Available` status.
3. **NFS Mount Failure on Worker Droplets (`bad option; might need /sbin/mount.nfs`):**
   - Ensure `nfs-common` is installed on every Ubuntu worker Droplet (`sudo apt-get install -y nfs-common`).
4. **Volume Mount `subPath` Gotcha:**
   - Using `subPath` to mount a single file from a ConfigMap or Secret prevents automated live-updates when the ConfigMap changes!
