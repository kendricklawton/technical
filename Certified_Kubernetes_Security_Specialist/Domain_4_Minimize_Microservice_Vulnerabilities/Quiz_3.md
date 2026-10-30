# Domain 4: Minimize Microservice Vulnerabilities — Quiz 3 (Container Sandboxes, Secrets Encryption & mTLS)

**Exam:** Certified Kubernetes Security Specialist (CKS)  
**Format:** 10 Performance Scenario & Multiple Choice Security Questions with Explanations

---

### Question 1: Threat Model of Standard OCI (`runc`) Containers vs Sandboxed Runtimes
Why do standard `runc` containers share a single point of failure when executing untrusted third-party or multi-tenant code?
- A. Standard `runc` containers isolated by namespaces and cgroups still make system calls directly to the **shared host Linux kernel**; a single kernel zero-day vulnerability in a syscall handler allows an attacker to escape the container onto the worker node.
- B. `runc` containers do not use cgroups.
- C. `runc` runs every container in a separate VM.
- D. `runc` disables network isolation.

---

### Question 2: gVisor (`runsc`) vs Kata Containers (`kata-runtime`) Architecture
How do **gVisor** (`runsc`) and **Kata Containers** (`kata-runtime`) differ in how they isolate container system calls from the host Linux kernel?
- A. **gVisor (`runsc`)** implements an application kernel (the **Sentry**) in memory-safe Go userspace that intercepts and services guest application syscalls without passing them to the host kernel (using minimal host syscalls via the Gofer for file I/O), whereas **Kata Containers** launches each Pod inside a lightweight hardware-virtualized microVM (using QEMU/Cloud-Hypervisor/Firecracker) with its own dedicated guest Linux kernel.
- B. gVisor uses hardware VMs, while Kata Containers uses AppArmor profiles.
- C. Both use `chroot` only.
- D. Neither integrates with `containerd`.

---

### Question 3: Creating a Kubernetes `RuntimeClass` and Scheduling a Sandboxed Pod
A worker node has `containerd` configured with a runtime handler named `runsc` (`[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runsc]`). How do you expose this in Kubernetes as a `RuntimeClass` named `gvisor` and configure a Pod to use it?
- A. Create a cluster-scoped `RuntimeClass` (`node.k8s.io/v1`) with `metadata.name: gvisor` and `handler: runsc`, then set `spec.runtimeClassName: gvisor` on the Pod.
- B. Set `securityContext.runtime: runsc` on the Pod.
- C. Annotate the Pod with `runtime.kubernetes.io/handler: gvisor`.
- D. Add `runtimeClass: gvisor` to `ServiceAccount`.

---

### Question 4: Verifying That a Running Pod Is Actually Executing Inside gVisor (`dmesg` / `uname`)
After deploying a Pod with `runtimeClassName: gvisor`, you run `kubectl exec <pod> -- dmesg` (or `uname -a`). What output confirms the container is isolated by the gVisor Sentry kernel rather than the host Linux kernel?
- A. `dmesg` prints `Starting gVisor...` (with witty gVisor boot messages) and `uname -r` reports the virtualized gVisor release (`4.4.0`), completely hiding the host's kernel ring buffer.
- B. `dmesg` prints the worker node's hardware PCI bus logs.
- C. `uname -a` prints `Kata-VM`.
- D. `kubectl exec` does not work with gVisor.

---

### Question 5: `RuntimeClass` `scheduling` and `overhead` Fields
In a heterogeneous cluster where only nodes labeled `sandbox=gvisor` have the `runsc` binary installed, and each gVisor Sentry process consumes an extra `64Mi` of RAM outside the container, which two `RuntimeClass` fields ensure Pods with `runtimeClassName: gvisor` are placed only on capable nodes and have accurate resource accounting?
- A. `scheduling.nodeSelector` (and `tolerations`) to target nodes with `sandbox=gvisor`, and `overhead.podFixed` (`memory: 64Mi`) to add runtime memory overhead to the Pod's scheduling and cgroup limits.
- B. `spec.affinity` and `spec.limits`.
- C. `spec.hostSelector` and `spec.buffer`.
- D. `RuntimeClass` cannot influence scheduling.

---

### Question 6: Encrypting Kubernetes `Secrets` at Rest in `etcd` (`EncryptionConfiguration`)
By default, Kubernetes stores `Secret` objects in `etcd` as unencrypted base64-encoded plaintext. When configuring an `EncryptionConfiguration` (`apiserver.config.k8s.io/v1`) passed to `kube-apiserver` via `--encryption-provider-config`, what does the **order** of providers in the `providers:` list determine?
- A. The **first** provider in the `providers:` list is used to **encrypt** all new/updated `Secret` writes to `etcd`, while **all** listed providers are tried in order to **decrypt** existing data read from `etcd`.
- B. The last provider in the list is used for encryption.
- C. All providers encrypt the data simultaneously in layers.
- D. The order has no effect.

---

### Question 7: Pitfall of Placing `identity: {}` First in `EncryptionConfiguration`
An administrator writes the following `EncryptionConfiguration`:
```yaml
resources:
  - resources: ["secrets"]
    providers:
      - identity: {}
      - aescbc:
          keys:
            - name: key1
              secret: <32-byte-base64-key>
```
Why are newly created `Secrets` still stored in plaintext in `etcd`?
- A. Because `identity: {}` (plaintext no-op provider) is listed **first**; `identity: {}` must be placed **last** so `aescbc` (or `kms` / `aesgcm`) encrypts new writes while `identity: {}` allows reading pre-existing unencrypted Secrets until they are re-encrypted.
- B. Because `aescbc` does not support Secrets.
- C. Because `etcd` must be restarted before encryption works.
- D. Because base64 keys must be 128 bytes.

---

### Question 8: Re-Encrypting All Existing Secrets in `etcd` After Enabling Encryption
After enabling `--encryption-provider-config` on `kube-apiserver` with `aescbc` first and `identity: {}` second, existing Secrets created prior to the change remain unencrypted on disk in `etcd`. Which command forces `kube-apiserver` to re-write and encrypt every existing `Secret` across all namespaces?
- A. `kubectl get secrets --all-namespaces -o json | kubectl replace -f -`
- B. `etcdctl encrypt --all`
- C. `kubectl rollout restart secrets -A`
- D. `kubeadm certs renew all`

---

### Question 9: Verifying That a Secret Is Encrypted in `etcd` Using `etcdctl`
Which command directly inspects the raw value stored in `etcd` for Secret `db-creds` in namespace `default` to verify it starts with the encrypted prefix `k8s:enc:aescbc:v1:key1:` instead of plaintext?
- A. `ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key get /registry/secrets/default/db-creds | hexdump -C`
- B. `kubectl get secret db-creds -o yaml`
- C. `kubectl describe secret db-creds`
- D. `crictl inspect etcd`

---

### Question 10: Enforcing Strict Pod-to-Pod mTLS with Istio (`PeerAuthentication`) or Cilium
In a cluster running Istio service mesh, you must require strict mutual TLS (mTLS) for all inbound traffic to every workload in namespace `finance`, rejecting any plaintext connections. Which resource accomplishes this?
- A. An Istio `PeerAuthentication` (`security.istio.io/v1beta1`) in namespace `finance` with `spec.mtls.mode: STRICT`.
- B. A Kubernetes `Ingress` with `tls: true`.
- C. A `Service` of type `ExternalName`.
- D. A `Secret` of type `kubernetes.io/tls`.

---

## Answer Key & Detailed Explanations

1. **A** — Because all standard containers share the host OS kernel, kernel syscall vulnerabilities (`Dirty Pipe`, `Dirty COW`, `io_uring` bugs) can lead to host compromise.
2. **A** — gVisor intercepts syscalls via a userspace kernel (`runsc`), while Kata Containers isolates pods inside lightweight hardware-virtualized VMs.
3. **A** — `RuntimeClass` maps a Kubernetes object name (`gvisor`) to the CRI runtime handler (`runsc`) configured in `/etc/containerd/config.toml`.
4. **A** — Running `dmesg` inside a gVisor pod outputs the gVisor Sentry startup banner rather than the host Linux kernel ring buffer.
5. **A** — `RuntimeClass.scheduling` appends `nodeSelector` and `tolerations` to matching Pods, and `RuntimeClass.overhead` accounts for the sandbox hypervisor/Sentry memory and CPU footprint.
6. **A** — The first provider in `EncryptionConfiguration` encrypts all writes; subsequent providers are fallback decryptors for historical data.
7. **A** — `identity: {}` performs no encryption. Listing it first leaves all writes unencrypted.
8. **A** — Piping `kubectl get secrets -A -o json` into `kubectl replace -f -` triggers an `UPDATE` write for every Secret, causing `kube-apiserver` to encrypt each one using the first provider.
9. **A** — `kubectl get secret` always returns decrypted data because `kube-apiserver` decrypts it on read; only querying `/registry/secrets/<ns>/<name>` directly via `etcdctl` confirms at-rest encryption (`k8s:enc:...`).
10. **A** — Istio's `PeerAuthentication` with `mtls.mode: STRICT` rejects non-mTLS plaintext TCP connections between pods.
