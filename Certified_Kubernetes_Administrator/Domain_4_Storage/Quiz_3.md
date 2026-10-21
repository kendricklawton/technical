# Domain 4: Storage — Quiz 3 (StatefulSets, Mount Permissions & Storage Troubleshooting)

**Exam:** Certified Kubernetes Administrator (CKA)  
**Format:** 10 Scenario & Performance Questions with Detailed Explanations

---

### Question 1: StatefulSet `volumeClaimTemplates` Retention on Scale-Down
A StatefulSet `kafka` has `replicas: 3` and uses `volumeClaimTemplates` to create `data-kafka-0`, `data-kafka-1`, and `data-kafka-2`. You scale the StatefulSet down to `replicas: 1`. What happens to `data-kafka-1` and `data-kafka-2`?
- A. Both PVCs are automatically deleted when `kafka-1` and `kafka-2` terminate.
- B. By default (`persistentVolumeClaimRetentionPolicy.whenScaled: Retain`), `data-kafka-1` and `data-kafka-2` are retained and remain `Bound` to preserve stateful data; if the StatefulSet is later scaled back up to 3, `kafka-1` and `kafka-2` automatically reattach to their existing PVCs.
- C. The PVCs transition to `Released`.
- D. The data is merged into `data-kafka-0`.

---

### Question 2: Automated PVC Cleanup on StatefulSet Deletion
In Kubernetes 1.27+, you want PVCs created by a StatefulSet's `volumeClaimTemplates` to be automatically deleted when the StatefulSet itself is deleted, but retained when scaled down. Which `spec.persistentVolumeClaimRetentionPolicy` configuration achieves this?
- A. `whenDeleted: Delete` and `whenScaled: Retain`
- B. `whenDeleted: Retain` and `whenScaled: Delete`
- C. `reclaimPolicy: AutoDelete`
- D. `cascadeDelete: true`

---

### Question 3: Resolving Volume Permission Denied Errors with `fsGroup`
A Pod runs as non-root user `runAsUser: 10001` and mounts an `ext4` CSI block volume at `/var/lib/postgresql/data`. The container fails to start with `initdb: could not change permissions of directory "/var/lib/postgresql/data": Permission denied` because the newly formatted `ext4` volume is owned by `root:root`. What Pod `securityContext` setting causes Kubernetes to recursively `chown` and set group-write permissions on the mounted volume?
- A. `runAsGroup: 10001`
- B. `fsGroup: 10001`
- C. `allowPrivilegeEscalation: true`
- D. `readOnlyRootFilesystem: false`

---

### Question 4: Speeding Up Mount Time on Millions of Files (`fsGroupChangePolicy`)
A StatefulSet mounts a `2Ti` persistent volume containing 15 million small files. Pod startup takes 18 minutes because the kubelet recursively walks the entire directory tree executing `chown`/`chmod` on every mount. How do you optimize this in the Pod's `securityContext`?
- A. Remove `fsGroup` completely.
- B. Set `fsGroupChangePolicy: "OnRootMismatch"` so the kubelet only performs recursive ownership changes if the top-level root directory's ownership/permissions do not already match `fsGroup`.
- C. Set `volumeBindingMode: Immediate`.
- D. Use `subPath`.

---

### Question 5: Diagnosing `Multi-Attach error for volume`
During a rolling update of a Deployment using an AWS EBS (`ReadWriteOnce`) PVC, the new Pod on `worker-2` hangs in `ContainerCreating` with event: `Multi-Attach error for volume "pvc-89a1..." Volume is already exclusively attached to one node and can't be attached to another`. How do you prevent this during Deployment rollouts of single-replica RWO workloads?
- A. Change the Deployment `spec.strategy.type` from `RollingUpdate` to `Recreate` so the old Pod is terminated and its RWO volume detached before the new Pod is scheduled.
- B. Increase `maxSurge` to `100%`.
- C. Set `persistentVolumeReclaimPolicy: Recycle`.
- D. Add `terminationGracePeriodSeconds: 300`.

---

### Question 6: Stuck PVC Deletion (`Terminating` State)
An administrator ran `kubectl delete pvc data-old-0` and also force-deleted the Pod, or a finalizer from a defunct backup operator is stuck. The PVC remains indefinitely in `Terminating`. After verifying no Pod is mounting the volume, what command removes the finalizer to complete PVC deletion?
- A. `kubectl patch pvc data-old-0 -p '{"metadata":{"finalizers":null}}'`
- B. `kubectl delete pvc data-old-0 --force --grace-period=0`
- C. `kubectl taint pvc data-old-0 finalizer-`
- D. `kubectl cordon pvc data-old-0`

---

### Question 7: Generic Ephemeral Volumes (`ephemeral` Volume Source)
A batch analytics job requires a `200Gi` scratch disk backed by a high-IOPS CSI StorageClass (`io2-nvme`), which exceeds the worker node's root disk capacity (`emptyDir`), and must be automatically created with the Pod and destroyed when the Pod finishes. Which volume type should be used?
- A. `hostPath`
- B. Generic `ephemeral` volume containing a `volumeClaimTemplate` referencing `storageClassName: io2-nvme`
- C. `configMap`
- D. Static `PersistentVolume` with `Retain` policy

---

### Question 8: Mount Propagation (`MountPropagation`) for CSI Node Plugins
Why must a CSI Node DaemonSet container configure `mountPropagation: Bidirectional` on its `/var/lib/kubelet` volumeMount?
- A. To encrypt data in transit.
- B. So that filesystem mounts created inside the CSI Node container are propagated back to the host OS kernel mount table where the kubelet and application containers can access them.
- C. To enable `ReadWriteMany` across nodes.
- D. To bypass NetworkPolicies.

---

### Question 9: NFS Mount Failures (`mount.nfs: No such file or directory` / Missing Client Utils)
A Pod mounting an NFSv4 PersistentVolume fails in `ContainerCreating` with `Output: mount: /var/lib/kubelet/pods/...: bad option; for several filesystems (e.g. nfs, cifs) you might need a /sbin/mount.<type> helper program`. Where must the `nfs-common` / `nfs-utils` package be installed?
- A. Inside the application container image.
- B. On the host OS of every Kubernetes worker node that schedules Pods mounting NFS volumes.
- C. Only on the `kube-apiserver` control-plane node.
- D. Inside the `coredns` pod.

---

### Question 10: Eviction Signals for Node `ephemeral-storage` (`imagefs` vs `nodefs`)
A Pod without storage limits writes 45 GB of debug logs into an `emptyDir` volume, causing the worker node's root filesystem to cross `nodefs.available<10%`. What action does the kubelet take?
- A. It expands the node's root disk automatically.
- B. It taints the node with `node.kubernetes.io/disk-pressure:NoSchedule` and evicts Pods exceeding their ephemeral-storage requests (starting with `BestEffort` and Pods writing the most to `emptyDir`/container writable layers).
- C. It deletes all PersistentVolumes in the cluster.
- D. It restarts `kube-proxy`.

---

## Answer Key & Detailed Explanations

1. **B** — StatefulSets prioritize data safety: scaling down a StatefulSet leaves the `volumeClaimTemplates` PVCs intact (`Bound`) so scaling back up reattaches the exact same state.
2. **A** — `persistentVolumeClaimRetentionPolicy` supports `whenDeleted` and `whenScaled` independently (`Retain` or `Delete`).
3. **B** — Setting `securityContext.fsGroup: 10001` at the Pod level tells the kubelet to change the group ownership of all files on the volume to GID `10001` with the `setgid` bit enabled.
4. **B** — `fsGroupChangePolicy: "OnRootMismatch"` skips the expensive recursive `chown`/`chmod` walk if the root directory of the volume already has the expected group ownership and permissions.
5. **A** — Because `RollingUpdate` starts the new Pod before terminating the old Pod, scheduling the new Pod onto a different node triggers a `Multi-Attach error` on `ReadWriteOnce` volumes. `Recreate` terminates the old Pod first.
6. **A** — Patching `metadata.finalizers` to `null` (or `[]`) clears stuck finalizers on a `Terminating` resource. (`--force --grace-period=0` does not bypass finalizers).
7. **B** — Generic ephemeral volumes (`volumes[].ephemeral.volumeClaimTemplate`) dynamically provision a CSI volume tied directly to the Pod's lifecycle.
8. **B** — `Bidirectional` mount propagation allows mounts performed by the containerized CSI driver to appear in the host's mount namespace.
9. **B** — Because the kubelet (or CSI node plugin in the host mount namespace) executes the `mount` syscall on the worker node, `/sbin/mount.nfs` (`nfs-common` on Debian/Ubuntu, `nfs-utils` on RHEL) must be installed on the worker nodes.
10. **B** — When `nodefs` or `imagefs` crosses the kubelet's eviction threshold, the node reports `DiskPressure`, blocks new scheduling, and evicts pods to reclaim disk space.
