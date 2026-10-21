# Domain 4: Storage — Essential Documentation & Technical Reference Guide

> **Blueprint Weight:** 10% of Certified Kubernetes Administrator (CKA)  
> **Core Objective:** Configure and manage PersistentVolumes (PV), PersistentVolumeClaims (PVC), StorageClasses, dynamic provisioning, volume expansion, access modes, reclaim policies, and VolumeSnapshots on DigitalOcean Droplets.

---

## 1. Official Documentation & Authoritative References

### 1.1 [Persistent Volumes, PVCs, and StorageClasses](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)
* **Canonical Scope:** Decoupling storage lifecycle from Pod and Node lifecycles.
* **Volume Lifecycle & Access Modes:**
  * **Access Modes:**
    * `ReadWriteOnce` (RWO): Read-write by a single node (e.g., DigitalOcean Block Storage `dobs.csi.digitalocean.com`, `rancher.io/local-path`).
    * `ReadOnlyMany` (ROX): Read-only by many nodes.
    * `ReadWriteMany` (RWX): Read-write by many nodes (NFS over DigitalOcean VPC `eth1`, CephFS).
    * `ReadWriteOncePod` (RWOP): Read-write by a single pod across the entire cluster.
  * **Reclaim Policies:**
    * `Retain`: Manual reclamation; PV remains in `Released` status after PVC deletion.
    * `Delete`: Deletes the backing cloud storage volume automatically upon PVC deletion.

### 1.2 [Dynamic Volume Provisioning & StorageClasses](https://kubernetes.io/docs/concepts/storage/dynamic-provisioning/)
* **Canonical Scope:** On-demand volume allocation triggered by PVC creation.
* **Key StorageClass Parameters:**
  * `provisioner`: The CSI driver plugin (e.g., `dobs.csi.digitalocean.com` for DigitalOcean Block Storage, `rancher.io/local-path` for local Droplet NVMe storage, `nfs.csi.k8s.io` for NFS).
  * `volumeBindingMode`:
    * `Immediate`: Volume created as soon as PVC is created (can lead to topology/node mismatches).
    * `WaitForFirstConsumer`: Delays volume creation until the Pod using the PVC is scheduled onto a Droplet, guaranteeing storage is provisioned in the matching region/node (`nyc3` / `cka-worker1`).
  * `allowVolumeExpansion: true`: Enables online resizing of PVCs.

### 1.3 [Volume Snapshots & Restore](https://kubernetes.io/docs/concepts/storage/volume-snapshots/)
* **Canonical Scope:** Standardized point-in-time storage snapshots via CSI (`dobs.csi.digitalocean.com` or `csi-driver-host-path`).
* **Resources:** `VolumeSnapshotClass`, `VolumeSnapshot`, and restoring via `dataSource` in PVC.

---

## 2. High-Frequency Exam Traps & Practical Reflexes

1. **PVC Remains in `Pending`:**
   * Cause 1: `volumeBindingMode: WaitForFirstConsumer` is enabled — the PVC will stay Pending until a Pod referencing it is created.
   * Cause 2: StorageClass does not exist, or requested storage capacity exceeds available PV capacity.
2. **Online Volume Expansion:**
   * To resize an active volume: edit PVC `spec.resources.requests.storage: 20Gi`. If `allowVolumeExpansion: true` is set on the StorageClass, the filesystem expands automatically without restarting the pod.
3. **PV Deletion Cascade Trap:**
   * Deleting a PVC bound to a PV with `persistentVolumeReclaimPolicy: Retain` will release the claim, but the PV status will become `Released`. It cannot be bound by another PVC until an administrator manually scrubs the `spec.claimRef`.

---

## 3. Reading & Study Checklist

- [ ] Read *Persistent Volumes, PVCs, and StorageClasses*
- [ ] Read *Dynamic Volume Provisioning & StorageClasses*
- [ ] Practice creating PV/PVC pairings with RWO, RWX, and Retain policies on your DigitalOcean Droplets
- [ ] Test `volumeBindingMode: WaitForFirstConsumer` scheduling behavior
- [ ] Perform online PVC volume expansion drill
