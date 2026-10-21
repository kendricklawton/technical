# Domain 4: Storage — Quiz 2 (StorageClasses, CSI & Dynamic Provisioning)

**Exam:** Certified Kubernetes Administrator (CKA)  
**Format:** 10 Scenario & Performance Questions with Detailed Explanations

---

### Question 1: `WaitForFirstConsumer` vs `Immediate` Volume Binding Mode
A Kubernetes cluster spans multiple DigitalOcean Droplets (`cka-worker1` and `cka-worker2`) and failure domains. An engineer creates a PVC backed by a local NVMe or regional block CSI StorageClass (`dobs.csi.digitalocean.com` / `rancher.io/local-path`) configured with `volumeBindingMode: Immediate`. When a Pod is later created with a `nodeSelector` for `cka-worker2`, the Pod remains stuck in `Pending` with `VolumeNodeAffinityConflict`. Why?
- A. Block volumes cannot be used on `cka-worker2`.
- B. `Immediate` binding provisioned/bound the volume as soon as the PVC was created (selecting `cka-worker1` / a different topology domain) before knowing which Droplet the Pod would be scheduled onto; changing the StorageClass to `WaitForFirstConsumer` delays volume provisioning until the scheduler selects the target node.
- C. The PVC needs `ReadWriteMany`.
- D. The CSI driver does not support multi-node clusters.

---

### Question 2: Setting a Default StorageClass
A cluster currently has two StorageClasses: `standard` and `do-block-storage-fast`. Neither is marked as default. What annotation marks `do-block-storage-fast` as the cluster's default StorageClass?
- A. `storageclass.kubernetes.io/is-default-class: "true"`
- B. `volume.beta.kubernetes.io/default-sc: "true"`
- C. `csi.storage.k8s.io/default: "enabled"`
- D. `kubernetes.io/storage-default: "1"`

---

### Question 3: Online Volume Expansion (`allowVolumeExpansion: true`)
A stateful workload is running out of disk space on its `20Gi` PVC. You edit the PVC and change `spec.resources.requests.storage` to `50Gi`. Which two requirements must be met for the volume to expand without pod downtime?
- A. `persistentVolumeReclaimPolicy` must be `Delete` and `volumeMode` must be `Block`.
- B. The StorageClass must have `allowVolumeExpansion: true` AND the underlying CSI driver (`dobs.csi.digitalocean.com`) must support `ExpandVolume` (including `NodeExpandVolume` for filesystem resize).
- C. The PVC must be deleted and recreated.
- D. The worker node must be cordoned and drained.

---

### Question 4: `FileSystemResizePending` Condition on a PVC
After increasing a PVC request from `10Gi` to `40Gi`, `kubectl describe pvc app-data` shows condition `FileSystemResizePending: Waiting for user to (re-)start a pod to finish file system resize of volume on node`. Why is the resize waiting?
- A. The underlying cloud disk has been expanded at the storage control plane, and it will complete filesystem expansion as soon as a Pod mounts the PVC (or if the CSI driver only supports offline filesystem expansion, when the Pod is restarted).
- B. The StorageClass has `allowVolumeExpansion: false`.
- C. The node is out of memory.
- D. The PVC must be manually formatted with `mkfs.ext4`.

---

### Question 5: Raw Block Volumes (`volumeMode: Block`)
A high-performance database manages raw disk blocks directly without an OS filesystem layer (`ext4`/`xfs`). How must the PVC and Pod specification be configured?
- A. Set `volumeMode: Block` in both the PV/PVC and use `volumeDevices` (with `devicePath: /dev/xvda`) instead of `volumeMounts` in the Pod spec.
- B. Set `fsType: raw` in the StorageClass.
- C. Mount the PVC at `/dev/sda` using `volumeMounts`.
- D. Use `hostPath` with `type: BlockDevice`.

---

### Question 6: Protecting Active PVs and PVCs (`storage-object-in-use-protection`)
An engineer runs `kubectl delete pvc prod-db-pvc` while the `prod-db-0` Pod is still actively running and mounting that PVC. What happens?
- A. The PVC is immediately deleted and the Pod crashes with I/O errors.
- B. The PVC enters a `Terminating` state, but deletion is blocked by the `kubernetes.io/pvc-protection` finalizer until all Pods mounting the PVC are terminated.
- C. The `prod-db-0` Pod is automatically deleted along with the PVC.
- D. The API server returns `403 Forbidden`.

---

### Question 7: CSI Architecture Components (`external-provisioner`, `external-attacher`, `node-driver-registrar`)
In a standard Container Storage Interface (CSI) deployment (such as `dobs.csi.digitalocean.com`), how are the controller plugin and node plugin typically deployed in Kubernetes?
- A. Both run inside `kube-controller-manager`.
- B. The CSI Controller plugin runs as a Deployment/StatefulSet (interacting with cloud APIs via sidecars like `csi-provisioner`, `csi-attacher`, `csi-resizer`), while the CSI Node plugin runs as a DaemonSet on every worker node (registering with kubelet via unix domain sockets and performing `NodeStageVolume` / `NodePublishVolume`).
- C. Both must be compiled into the `kubelet` binary.
- D. CSI drivers run only on control-plane nodes.

---

### Question 8: CSI Volume Snapshots (`VolumeSnapshot`, `VolumeSnapshotContent`, `VolumeSnapshotClass`)
You want to create a point-in-time snapshot of a CSI-backed PVC named `mysql-pvc`. Which API resource do you create in the workload namespace?
- A. `PersistentVolumeSnapshot`
- B. `VolumeSnapshot` (in API group `snapshot.storage.k8s.io/v1`) referencing `source.persistentVolumeClaimName: mysql-pvc` and a `volumeSnapshotClassName`
- C. `CSISnapshot`
- D. `StorageSnapshot`

---

### Question 9: Restoring a PVC from a `VolumeSnapshot`
Once a `VolumeSnapshot` named `mysql-snap-2026` shows `readyToUse: true`, how do you provision a new PersistentVolumeClaim pre-populated with that snapshot's data?
- A. Run `kubectl restore pvc mysql-pvc --from-snapshot=mysql-snap-2026`.
- B. Create a new `PersistentVolumeClaim` with a `spec.dataSource` block setting `kind: VolumeSnapshot`, `name: mysql-snap-2026`, and `apiGroup: snapshot.storage.k8s.io`.
- C. Edit the existing `VolumeSnapshot` and change its target PVC name.
- D. Copy `/var/lib/kubelet/pods` manually.

---

### Question 10: Local PersistentVolumes (`local` volume type)
Unlike `hostPath` volumes, what does a `local` PersistentVolume require in its specification to prevent the scheduler from scheduling pods onto the wrong worker Droplet?
- A. `spec.nodeAffinity` defining the specific node (`kubernetes.io/hostname`) where the local disk resides, combined with a StorageClass using `volumeBindingMode: WaitForFirstConsumer`.
- B. `persistentVolumeReclaimPolicy: Delete`
- C. `accessModes: [ReadWriteMany]`
- D. A `daemonSetSelector` field.

---

## Answer Key & Detailed Explanations

1. **B** — `WaitForFirstConsumer` binds and provisions the volume only after the Pod scheduler evaluates Pod affinity, taints/tolerations, and `nodeSelector` constraints to choose a target node/topology.
2. **A** — Adding annotation `storageclass.kubernetes.io/is-default-class: "true"` designates a StorageClass as default. Ensure only one StorageClass has this annotation set to `"true"`.
3. **B** — Dynamic volume expansion requires `allowVolumeExpansion: true` on the StorageClass and CSI driver support for controller and node expansion operations.
4. **A** — `FileSystemResizePending` indicates the cloud block storage size has increased, and the kubelet/CSI node plugin will expand the filesystem on the next mount or online once mounted.
5. **A** — `volumeMode: Block` exposes the raw block device directly inside the container at `devicePath` under `volumeDevices` without formatting a filesystem.
6. **B** — The `StorageObjectInUseProtection` admission plugin adds `kubernetes.io/pvc-protection` to PVCs and `kubernetes.io/pv-protection` to PVs, preventing deletion while actively used by a Pod or bound to a PVC.
7. **B** — CSI decouples storage control-plane operations (Deployment) from node-level mount/format operations (DaemonSet communicating with kubelet over `/var/lib/kubelet/plugins_registry`).
8. **B** — `VolumeSnapshot` is the namespaced resource (analogous to `PersistentVolumeClaim`), while `VolumeSnapshotContent` is the cluster-scoped resource (analogous to `PersistentVolume`).
9. **B** — Specifying `spec.dataSource` with `kind: VolumeSnapshot` instructs the CSI provisioner to hydrate the new volume from the snapshot.
10. **A** — `local` volumes require `spec.nodeAffinity` on the PV so the Kubernetes scheduler understands which node owns the physical disk.
