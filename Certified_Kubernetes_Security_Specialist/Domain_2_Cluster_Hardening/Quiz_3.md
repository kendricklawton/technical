# CKS Domain 2: Cluster Hardening — Quiz 3 (ServiceAccount Hardening & Cluster Upgrades)

Test your knowledge of ServiceAccount token mounting precedence, Bound ServiceAccount Token Volumes (`projected`), `TokenRequest` API, and `kubeadm` cluster upgrades.

---

### Question 1
A `ServiceAccount` named `payment-sa` in the `checkout` namespace is configured with:
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: payment-sa
  namespace: checkout
automountServiceAccountToken: false
```
A Pod named `payment-pod` is created in `checkout` with the following `spec`:
```yaml
spec:
  serviceAccountName: payment-sa
  automountServiceAccountToken: true
  containers:
    - name: app
      image: nginx:1.27-alpine
```
Will Kubernetes mount the default ServiceAccount token at `/var/run/secrets/kubernetes.io/serviceaccount/token` inside `payment-pod`?

- A) No, because `automountServiceAccountToken: false` on the `ServiceAccount` cannot be overridden by a Pod.
- B) Yes, because `Pod.spec.automountServiceAccountToken` takes precedence over `ServiceAccount.automountServiceAccountToken`.
- C) The Pod will be rejected by the API server due to conflicting settings.
- D) The directory `/var/run/secrets/kubernetes.io/serviceaccount/` will be mounted empty.

<details>
<summary>Reveal Answer & Explanation</summary>

**Correct Answer: B**

**Explanation:**
In Kubernetes, **`Pod.spec.automountServiceAccountToken` always takes precedence** over `ServiceAccount.automountServiceAccountToken` when both are explicitly defined! If `Pod.spec.automountServiceAccountToken` is omitted (`nil`), Kubernetes falls back to the `ServiceAccount`'s setting. To prevent a Pod from mounting a token, ensure `automountServiceAccountToken: false` is set on the Pod spec (or not overridden with `true`).
</details>

---

### Question 2
How do modern Bound ServiceAccount Tokens (mounted via `projected` volumes / `TokenRequest` API in Kubernetes v1.22+) improve security compared to legacy Secret-based ServiceAccount tokens?

- A) Bound tokens are stored as plaintext ConfigMaps in `etcd`.
- B) Bound tokens have a finite lifetime (`expirationSeconds`), are bound to a specific `audience` and the specific `Pod` UID (becoming invalid immediately when the Pod is deleted), and are never persisted as `Secret` objects in `etcd`.
- C) Bound tokens automatically grant `cluster-admin` privileges for 10 minutes.
- D) Bound tokens do not use JWT format.

<details>
<summary>Reveal Answer & Explanation</summary>

**Correct Answer: B**

**Explanation:**
Legacy ServiceAccount tokens were stored in `etcd` as `Secret` objects (`kubernetes.io/service-account-token`) with **no expiration** and no Pod binding. Modern `TokenRequest`-issued bound tokens are dynamically signed in memory by `kube-apiserver`, expire automatically (e.g., 1 hour), are rotated automatically by the `kubelet`, are bound to the target `audience`, and are invalidated the instant the Pod or ServiceAccount is deleted.
</details>

---

### Question 3
You configure a `projected` volume in a Pod to mount a `serviceAccountToken` with `expirationSeconds: 120` (2 minutes). When you apply the Pod manifest, what happens?

- A) The token expires every 2 minutes and the `kubelet` rotates it every 96 seconds.
- B) The API server rejects the Pod (or enforces a minimum floor) because the minimum allowed `expirationSeconds` for a projected `serviceAccountToken` in Kubernetes is `600` seconds (10 minutes).
- C) The container restarts every 120 seconds.
- D) `automountServiceAccountToken` is automatically changed to `true`.

<details>
<summary>Reveal Answer & Explanation</summary>

**Correct Answer: B**

**Explanation:**
By default, `kube-apiserver` enforces a **minimum `expirationSeconds` of `600` seconds (10 minutes)** for projected `serviceAccountToken` volumes. Specifying a value less than `600` fails validation: `Invalid value: 120: may not specify a duration shorter than 10 minutes`.
</details>

---

### Question 4
Which `kubectl` command generates a temporary, short-lived Bearer Token (valid for 30 minutes) for ServiceAccount `jenkins-sa` in namespace `ci` without creating a `Secret` object in `etcd`?

- A) `kubectl -n ci create secret service-account-token jenkins-sa --ttl=30m`
- B) `kubectl -n ci create token jenkins-sa --duration=30m`
- C) `kubectl -n ci get sa jenkins-sa -o jsonpath='{.secrets[0].name}'`
- D) `kubectl auth token --serviceaccount=ci:jenkins-sa --expire=30m`

<details>
<summary>Reveal Answer & Explanation</summary>

**Correct Answer: B**

**Explanation:**
`kubectl create token <sa-name> --duration=30m` calls the `TokenRequest` subresource on the `ServiceAccount` and prints a time-bound JWT token directly to `stdout` without persisting any `Secret` in `etcd`.
</details>

---

### Question 5
You are upgrading a Kubernetes cluster from `v1.30.4` to `v1.31.1` using `kubeadm`. You have already upgraded the control plane node (`controlplane`). You now drain `node01`, SSH into `node01`, and upgrade the `kubeadm` package to `1.31.1-1.1`. Which `kubeadm` command should you run next on the **worker node** `node01`?

- A) `kubeadm upgrade apply v1.31.1`
- B) `kubeadm upgrade node`
- C) `kubeadm init phase kubelet-start`
- D) `kubeadm upgrade plan`

<details>
<summary>Reveal Answer & Explanation</summary>

**Correct Answer: B**

**Explanation:**
* `kubeadm upgrade plan` and `kubeadm upgrade apply v1.31.1` are run **only** on the primary control plane node.
* On worker nodes (and additional control plane nodes), you run **`kubeadm upgrade node`**, which downloads the updated `KubeletConfiguration` from the cluster `kubelet-config` ConfigMap and updates `/var/lib/kubelet/config.yaml`.
</details>

---

### Question 6
According to the official Kubernetes Version Skew Policy, if your cluster's `kube-apiserver` is running version **`v1.31.1`**, which of the following versions is **INVALID** (unsupported) for a worker node's `kubelet`?

- A) `v1.31.1`
- B) `v1.30.4`
- C) `v1.29.6`
- D) `v1.32.0`

<details>
<summary>Reveal Answer & Explanation</summary>

**Correct Answer: D**

**Explanation:**
The `kubelet` version **must never be newer** than `kube-apiserver`! A `v1.32.0` `kubelet` talking to a `v1.31.1` `kube-apiserver` violates the version skew policy. That is why you always upgrade the control plane (`kube-apiserver`) **before** upgrading worker node `kubelet` binaries.
</details>

---

### Question 7
When draining a worker node (`kubectl drain node01`) before upgrading its `kubelet` and OS kernel packages, the drain command fails with errors stating that `calico-node` (managed by a `DaemonSet`) and `redis-cache` (a Pod using an `emptyDir` volume) cannot be evicted. Which flags allow `kubectl drain` to proceed safely?

- A) `kubectl drain node01 --force --delete-namespaces`
- B) `kubectl drain node01 --ignore-daemonsets --delete-emptydir-data`
- C) `kubectl drain node01 --skip-daemonsets --preserve-emptydir`
- D) `kubectl cordon node01 --evict-all`

<details>
<summary>Reveal Answer & Explanation</summary>

**Correct Answer: B**

**Explanation:**
* `--ignore-daemonsets` skips evicting DaemonSet-managed Pods (since the DaemonSet controller would immediately reschedule them onto the same node anyway).
* `--delete-emptydir-data` acknowledges that ephemeral scratch data stored in `emptyDir` volumes on that node will be lost when the Pod is evicted and rescheduled elsewhere.
</details>

---

### Question 8
Why is it recommended to run `apt-mark hold kubeadm kubelet kubectl` after completing a Kubernetes node upgrade on Ubuntu/Debian?

- A) It prevents `systemctl` from stopping the `kubelet` service.
- B) It prevents unattended `apt-get upgrade` runs from accidentally upgrading `kubeadm`, `kubelet`, or `kubectl` out of sequence or violating the Kubernetes version skew policy.
- C) It locks `/etc/kubernetes/manifests/` as read-only.
- D) It pins the container image digests in `containerd`.

<details>
<summary>Reveal Answer & Explanation</summary>

**Correct Answer: B**

**Explanation:**
Holding the Kubernetes packages (`apt-mark hold kubeadm kubelet kubectl`) ensures that routine OS patching (`apt-get upgrade`) does not unexpectedly upgrade `kubelet` on a worker node ahead of `kube-apiserver` or skip required `kubeadm upgrade` steps.
</details>

---

### Question 9
After upgrading the `kubelet` binary on `node01` via `apt-get install -y kubelet=1.31.1-1.1`, `kubectl get nodes` still reports `node01` running the old version `v1.31.0`. What step was forgotten?

- A) Running `kubeadm reset` on `node01`.
- B) Running `systemctl daemon-reload && systemctl restart kubelet` on `node01`.
- C) Rebooting `controlplane`.
- D) Deleting the `Node` object from the API server.

<details>
<summary>Reveal Answer & Explanation</summary>

**Correct Answer: B**

**Explanation:**
Installing the new `kubelet` package replaces `/usr/bin/kubelet` on disk, but the running `kubelet` process continues executing the old binary in memory until you run `systemctl daemon-reload && systemctl restart kubelet`.
</details>

---

### Question 10
Look at the following `projected` volume snippet in a Pod specification:
```yaml
  volumes:
    - name: custom-token
      projected:
        sources:
          - serviceAccountToken:
              path: my-token
              expirationSeconds: 3600
              audience: vault
          - configMap:
              name: app-config
          - downwardAPI:
              items:
                - path: "pod-name"
                  fieldRef:
                    fieldPath: metadata.name
```
What happens when `custom-token` is mounted at `/var/run/app-context` inside the container?

- A) Kubernetes rejects the Pod because `projected` volumes can only contain a single source type.
- B) All three sources (`serviceAccountToken`, `configMap`, and `downwardAPI`) are merged into the single `/var/run/app-context` directory inside the container.
- C) Only the first source (`serviceAccountToken`) is mounted.
- D) Three separate mountPath entries are required.

<details>
<summary>Reveal Answer & Explanation</summary>

**Correct Answer: B**

**Explanation:**
A `projected` volume is specifically designed to project multiple volume sources (`serviceAccountToken`, `secret`, `configMap`, and `downwardAPI`) into a single unified directory inside the container.
</details>
