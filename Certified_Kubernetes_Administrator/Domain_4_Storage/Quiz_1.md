# Domain 4: Storage — Quiz 1 (Volumes, PVs, PVCs & Access Modes)

**Exam:** Certified Kubernetes Administrator (CKA)  
**Format:** 10 Scenario & Performance Questions with Detailed Explanations

---

### Question 1: PersistentVolume Access Modes (`RWO` vs `ROX` vs `RWX`)
A deployment with 4 replicas dispersed across 3 worker nodes requires all 4 pods to mount the same PersistentVolumeClaim concurrently with read and write permissions. Which AccessMode MUST the underlying PersistentVolume and StorageClass support?
- A. `ReadWriteOnce (RWO)`
- B. `ReadOnlyMany (ROX)`
- C. `ReadWriteMany (RWX)`
- D. `ReadWriteOncePod (RWOP)`

---

### Question 2: `ReadWriteOncePod` (RWOP) Enforcement
A financial database pod requires strict guarantee that no other pod—even on the exact same worker node—can simultaneously mount its CSI block volume. Which access mode enforces single-pod exclusive attachment across the entire cluster?
- A. `ReadWriteOnce`
- B. `ReadWriteOncePod`
- C. `ReadOnlyMany`
- D. `ExclusiveMount`

---

### Question 3: PersistentVolume Reclaim Policy (`Retain` vs `Delete`)
A production PostgreSQL StatefulSet uses a PersistentVolume configured with `persistentVolumeReclaimPolicy: Retain`. An operator accidentally deletes the PersistentVolumeClaim (`kubectl delete pvc pg-data-0`). What happens to the PersistentVolume and its underlying data?
- A. Both the PV and the underlying storage volume are immediately deleted.
- B. The PV transitions to the `Available` phase and can be immediately bound by any new PVC.
- C. The PV transitions to the `Released` phase, the data remains intact on the backing storage, and the PV cannot be bound to a new PVC until an administrator manually clears `spec.claimRef`.
- D. Kubernetes blocks the deletion of the PVC until `persistentVolumeReclaimPolicy` is changed.

---

### Question 4: Transitioning a `Released` PersistentVolume Back to `Available`
An administrator wants to reuse a `Released` PersistentVolume (`pv-analytics`) without losing the existing data on the volume. Which action makes `pv-analytics` `Available` for re-binding?
- A. Delete and recreate the StorageClass.
- B. Edit the PV (`kubectl edit pv pv-analytics`) and remove the `spec.claimRef` block.
- C. Run `kubectl rollout restart pv pv-analytics`.
- D. Annotate the PV with `pv.kubernetes.io/bind-completed: "false"`.

---

### Question 5: `emptyDir` Memory-Backed Volumes (`tmpfs`)
A high-throughput caching sidecar requires an ephemeral scratch directory backed by node RAM (`tmpfs`) rather than the worker node's root disk, capped at `512Mi`. How should the volume be configured?
- A. `emptyDir: { medium: Memory, sizeLimit: 512Mi }`
- B. `hostPath: { path: /dev/shm, type: Memory }`
- C. `configMap: { memory: true, limit: 512Mi }`
- D. `ephemeral: { storageClass: ramdisk }`

---

### Question 6: `subPath` Volume Mount Behavior with ConfigMaps
A pod mounts a single configuration file from a ConfigMap into `/etc/nginx/nginx.conf` using `subPath: nginx.conf` so that other files in `/etc/nginx` are preserved. The administrator updates the ConfigMap via `kubectl edit cm nginx-config`. What happens inside the running container?
- A. `/etc/nginx/nginx.conf` is updated within 60 seconds (kubelet sync period).
- B. `/etc/nginx/nginx.conf` is NEVER automatically updated because volumes mounted with `subPath` do not receive projected ConfigMap/Secret updates; the pod must be restarted or reload via another mechanism.
- C. The container restarts automatically.
- D. The kubelet unmounts and remounts `/etc/nginx/nginx.conf` immediately.

---

### Question 7: Static Binding with `storageClassName: ""`
An administrator manually created a static PersistentVolume with `storageClassName: ""` and capacity `10Gi`. However, when creating a PersistentVolumeClaim requesting `10Gi` without specifying `storageClassName`, the PVC dynamically provisions a new volume from the cluster's default StorageClass instead of binding to the static PV. Why?
- A. Static PVs cannot be bound if a default StorageClass exists.
- B. Omitting `storageClassName` in a PVC causes the `DefaultStorageClass` admission controller to inject the cluster's default StorageClass name; to bind to a PV with no StorageClass, the PVC must explicitly set `storageClassName: ""`.
- C. The PVC must specify `volumeMode: Block`.
- D. Static PVs require `ReadWriteMany`.

---

### Question 8: Pre-Binding a Specific PVC to a Specific PV
Two teams are creating `50Gi` PVCs concurrently. You need to guarantee that `pvc-finance` in namespace `finance` binds exclusively to `pv-encrypted-01` and no other PVC can claim `pv-encrypted-01`. How do you enforce this two-way reservation?
- A. Add a label `team: finance` to the node.
- B. Set `spec.volumeName: pv-encrypted-01` on `pvc-finance` AND/OR pre-populate `spec.claimRef` (`name: pvc-finance`, `namespace: finance`) on `pv-encrypted-01`.
- C. Set `volumeBindingMode: WaitForFirstConsumer`.
- D. Use `hostPath` instead of PersistentVolumes.

---

### Question 9: `hostPath` Type Verification (`DirectoryOrCreate` vs `File`)
A DaemonSet pod mounts `/var/log/app-audit.log` from the worker node using `hostPath` with `type: File`. On a newly provisioned node where `/var/log/app-audit.log` does not yet exist, what happens when the pod is scheduled?
- A. The kubelet automatically creates an empty file `/var/log/app-audit.log`.
- B. The kubelet creates a directory named `/var/log/app-audit.log`.
- C. Pod startup fails at the `MountVolume` stage because `type: File` requires the file to exist prior to mounting (unlike `FileOrCreate`).
- D. The pod runs without mounting the volume.

---

### Question 10: Projected Volumes (`serviceAccountToken`, `downwardAPI`, `configMap`)
An application requires the pod's namespace, a rotated ServiceAccount token with a 1-hour expiration, and a CA bundle from a ConfigMap all mounted into a single unified directory `/var/run/secrets/app`. Which volume source accomplishes this without mounting three separate volumes?
- A. `emptyDir`
- B. `projected` volume with `sources` combining `downwardAPI`, `serviceAccountToken`, and `configMap`
- C. `csi` inline volume
- D. `hostPath`

---

## Answer Key & Detailed Explanations

1. **C** — `ReadWriteMany (RWX)` allows multiple nodes and pods to mount the volume simultaneously for read/write (e.g., NFS, CephFS, AWS EFS). `ReadWriteOnce (RWO)` only permits mounting by a single node.
2. **B** — Introduced in Kubernetes 1.22 and GA in 1.29, `ReadWriteOncePod (RWOP)` guarantees that only a single pod across the entire cluster can mount the volume.
3. **C** — Under the `Retain` reclaim policy, deleting a PVC transitions the PV to `Released`. The data remains intact, and an administrator must manually remove `spec.claimRef` before it becomes `Available` again.
4. **B** — Deleting the `spec.claimRef` stanza from a `Released` PV clears the previous claim binding and immediately transitions the PV back to `Available`.
5. **A** — Setting `emptyDir.medium: Memory` mounts a `tmpfs` RAM-backed filesystem. Note that memory consumed by a tmpfs `emptyDir` counts against the pod's memory cgroup limit.
6. **B** — Kubernetes documentation explicitly warns that a container using a ConfigMap or Secret as a `subPath` volume mount will not receive automated updates when the ConfigMap/Secret is modified.
7. **B** — When a default StorageClass exists in the cluster, omitting `storageClassName` on a PVC causes the admission controller to assign the default StorageClass. Explicitly specifying `storageClassName: ""` disables dynamic provisioning and matches PVs with an empty storageClassName.
8. **B** — Setting `spec.volumeName` in the PVC and `spec.claimRef` in the PV establishes a bidirectional binding contract between that exact PV and PVC.
9. **C** — `hostPath.type: File` validates that the file already exists on the host filesystem before mounting. To auto-create missing files, `FileOrCreate` must be used.
10. **B** — A `projected` volume maps multiple volume sources (`secret`, `configMap`, `downwardAPI`, `serviceAccountToken`, `clusterTrustBundle`) into a single directory with atomic symlink projection.
