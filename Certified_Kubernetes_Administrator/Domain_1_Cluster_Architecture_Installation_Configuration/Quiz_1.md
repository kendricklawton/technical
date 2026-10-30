# Domain 1 — Quiz 1: Control Plane Architecture, PKI Certificates & RBAC

Test your mastery of Kubernetes Control Plane internals, authentication/authorization pipelines, PKI certificates, and Role-Based Access Control (RBAC).

---

### Question 1
During an audit of a `kubeadm`-provisioned cluster, you want to verify which component is directly responsible for reading and writing state to `etcd`. Which of the following components communicates directly with the `etcd` datastore over port `2379`?

A. `kube-apiserver`, `kube-scheduler`, and `kube-controller-manager`  
B. Only `kube-apiserver`  
C. `kube-apiserver` and `kubelet` on control plane nodes  
D. All control plane Static Pods in `/etc/kubernetes/manifests/`

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer: B**

**Explanation:**  
In Kubernetes, **`kube-apiserver` is the only component that communicates directly with `etcd`**. Every other component (`kube-scheduler`, `kube-controller-manager`, `kubelet`, `kube-proxy`, `coredns`) interacts exclusively with `kube-apiserver` over HTTPS (`:6443`), which validates, authorizes, and persists state changes into `etcd`.
</details>

---

### Question 2
You want to grant the `ServiceAccount` `ci-runner` in namespace `build` permission to read (`get`, `list`, `watch`) Pods and view container logs (`kubectl logs`) in the `staging` namespace, without granting permissions in any other namespace. You want to reuse an existing `ClusterRole` named `pod-log-reader` that already defines `get`, `list`, `watch` on `pods` and `pods/log`. What is the most appropriate RBAC configuration?

A. Create a `ClusterRoleBinding` referencing `ClusterRole/pod-log-reader` and subject `ServiceAccount` `build:ci-runner`.  
B. Create a `RoleBinding` in namespace `build` referencing `ClusterRole/pod-log-reader` and subject `ServiceAccount` `build:ci-runner`.  
C. Create a `RoleBinding` in namespace `staging` referencing `ClusterRole/pod-log-reader` and subject `ServiceAccount` `ci-runner` (with `namespace: build`).  
D. A `RoleBinding` cannot reference a `ClusterRole`; you must recreate `pod-log-reader` as a namespaced `Role` in `staging`.

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer: C**

**Explanation:**  
- A **namespaced `RoleBinding` CAN reference a `ClusterRole`**, scoping the permissions of that `ClusterRole` solely to the namespace where the `RoleBinding` resides (`staging`).
- The `ServiceAccount` lives in `build`, while the target resources live in `staging`. Therefore, the `RoleBinding` must be created in `-n staging`, and its `--serviceaccount` subject must specify `build:ci-runner`:
  ```bash
  kubectl create rolebinding ci-staging-logs \
    --clusterrole=pod-log-reader \
    --serviceaccount=build:ci-runner \
    -n staging
  ```
</details>

---

### Question 3
A junior administrator created a `Role` with `resources: ["pods"]` and `verbs: ["get", "list", "watch"]` for user `dev-user`. However, when `dev-user` runs `kubectl logs my-pod -n dev`, they receive `Error from server (Forbidden): pods "my-pod" is forbidden: User "dev-user" cannot get resource "pods/log" in API group "" in the namespace "dev"`. Why did this happen?

A. `pods/log` is in the `metrics.k8s.io` API group, not the core `""` API group.  
B. `log` is a **subresource** of `pods` and must be explicitly listed in `resources: ["pods", "pods/log"]`.  
C. Viewing logs requires the `watch` verb on `nodes/proxy`.  
D. `kubectl logs` requires a `ClusterRole` rather than a `Role`.

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer: B**

**Explanation:**  
In Kubernetes RBAC, subresources such as `pods/log`, `pods/exec`, `pods/portforward`, `pods/status`, and `deployments/scale` are authorized separately from the parent resource. Granting `get` on `pods` allows viewing the Pod YAML/metadata, but viewing container stdout/stderr via `kubectl logs` requires `get` on `pods/log`.
</details>

---

### Question 4
Which command accurately tests whether the `ServiceAccount` `telemetry-agent` in the `monitoring` namespace is allowed to list `daemonsets` in the `kube-system` namespace?

A. `kubectl auth can-i list daemonsets -n kube-system --as=telemetry-agent`  
B. `kubectl auth can-i list daemonsets -n kube-system --as=system:serviceaccount:monitoring:telemetry-agent`  
C. `kubectl auth can-i list daemonsets -n monitoring --as=system:serviceaccount:kube-system:telemetry-agent`  
D. `kubectl get daemonsets -n kube-system --serviceaccount=monitoring:telemetry-agent`

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer: B**

**Explanation:**  
When impersonating a `ServiceAccount` with `--as`, you must use its full Kubernetes username format: `system:serviceaccount:<serviceaccount-namespace>:<serviceaccount-name>`, and specify `-n <target-namespace>` for the namespace where the action is being evaluated.
</details>

---

### Question 5
How does a `kubeadm`-initialized control plane node start and manage the `kube-apiserver`, `etcd`, `kube-controller-manager`, and `kube-scheduler` pods when `kube-apiserver` itself is not yet running?

A. `systemd` runs four separate unit files (`kube-apiserver.service`, `etcd.service`, etc.).  
B. `containerd` reads `/etc/containerd/manifests/` directly at boot.  
C. The `kubelet` systemd service reads Static Pod manifests from `staticPodPath: /etc/kubernetes/manifests/` (configured in `/var/lib/kubelet/config.yaml`) and instructs `containerd` via CRI to start them.  
D. `kube-proxy` bootstraps the control plane containers via `iptables`.

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer: C**

**Explanation:**  
On a `kubeadm` node, only `containerd` and `kubelet` run as OS-level `systemd` services. `kubelet` is configured via `/var/lib/kubelet/config.yaml` with `staticPodPath: /etc/kubernetes/manifests`. `kubelet` watches this directory directly on the local filesystem and starts `etcd`, `kube-apiserver`, `kube-controller-manager`, and `kube-scheduler` as Static Pods via CRI without needing an active API server.
</details>

---

### Question 6
You want to create a new custom `ClusterRole` named `crd-viewer` so that its permissions are **automatically inherited** by every user or ServiceAccount bound to the built-in `view` `ClusterRole`. How do you achieve this without editing the default `view` `ClusterRole` directly?

A. Name the new `ClusterRole` `system:view:crd-viewer`.  
B. Add the label `rbac.authorization.k8s.io/aggregate-to-view: "true"` to the metadata of `crd-viewer`.  
C. Create a `ClusterRoleBinding` that binds `crd-viewer` to `ClusterRole/view`.  
D. Add `parentRole: view` under `spec` in `crd-viewer`.

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer: B**

**Explanation:**  
The built-in `view`, `edit`, and `admin` `ClusterRoles` use `aggregationRule.clusterRoleSelectors`. Any `ClusterRole` labeled with `rbac.authorization.k8s.io/aggregate-to-view: "true"` has its `rules[]` automatically aggregated into the `view` `ClusterRole` by the `ClusterRoleAggregation` controller in `kube-controller-manager`.
</details>

---

### Question 7
In the `kube-apiserver` request processing lifecycle, in what exact order are incoming API requests processed before being written to `etcd`?

A. Authorization $\to$ Authentication $\to$ Validating Admission $\to$ Mutating Admission $\to$ `etcd`  
B. Authentication $\to$ Authorization $\to$ Mutating Admission $\to$ Schema Validation $\to$ Validating Admission $\to$ `etcd`  
C. Authentication $\to$ Mutating Admission $\to$ Authorization $\to$ Validating Admission $\to$ `etcd`  
D. Mutating Admission $\to$ Validating Admission $\to$ Authentication $\to$ Authorization $\to$ `etcd`

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer: B**

**Explanation:**  
1. **Authentication (AuthN)** verifies identity.  
2. **Authorization (AuthZ)** checks RBAC/Node permissions.  
3. **Mutating Admission** runs first because it may inject defaults or sidecars that need to be validated afterwards.  
4. **Object Schema Validation** validates against OpenAPI v3 schemas.  
5. **Validating Admission** performs final policy/quota validation on the mutated object before it is persisted to **`etcd`**.
</details>

---

### Question 8
Where does `kubeadm` store the client certificate and private key used by `kube-apiserver` to authenticate itself as a client when connecting to `etcd`?

A. `/etc/kubernetes/pki/etcd/server.crt` and `/etc/kubernetes/pki/etcd/server.key`  
B. `/etc/kubernetes/pki/apiserver-etcd-client.crt` and `/etc/kubernetes/pki/apiserver-etcd-client.key`  
C. `/etc/kubernetes/pki/apiserver.crt` and `/etc/kubernetes/pki/apiserver.key`  
D. `/var/lib/kubelet/pki/kubelet-client-current.pem`

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer: B**

**Explanation:**  
- `/etc/kubernetes/pki/apiserver-etcd-client.crt` (and `.key`) is signed by the `etcd` CA (`/etc/kubernetes/pki/etcd/ca.crt`) and is passed to `kube-apiserver` via `--etcd-certfile` and `--etcd-keyfile`.
- `/etc/kubernetes/pki/etcd/server.crt` is the server certificate presented by `etcd` itself.
</details>

---

### Question 9
A developer needs a `Role` in namespace `app-prod` that allows them to `get` and `update` **only** a single existing `ConfigMap` named `feature-flags`, while preventing them from reading or updating any other `ConfigMap` in `app-prod`. Which field in the `Role`'s `rules[]` block accomplishes this?

A. `labelSelector: { "metadata.name": "feature-flags" }`  
B. `resourceNames: ["feature-flags"]`  
C. `fieldSelector: "metadata.name=feature-flags"`  
D. `subresources: ["configmaps/feature-flags"]`

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer: B**

**Explanation:**  
RBAC rules do not support label or field selectors, but they DO support `resourceNames: ["<name1>", "<name2>"]` to restrict `get`, `update`, `patch`, and `delete` requests to specific named instances of a resource. (Note: `list`, `watch`, and `create` cannot be restricted by `resourceNames` because `create` validates before the resource exists and `list` operates on the collection.)
</details>

---

### Question 10
When onboarding a new user `sarah` using an X.509 client certificate signed by the cluster's Certificate Authority (`/etc/kubernetes/pki/ca.crt` and `/etc/kubernetes/pki/ca.key`), how does `kube-apiserver` determine the user's **Username** and **Group** memberships from the certificate's Subject field?

A. Username is in `OU` (Organizational Unit); Groups are in `CN` (Common Name).  
B. Username is in `CN` (Common Name); Groups are in `O` (Organization).  
C. Username is in `SAN` (Subject Alternative Name); Groups are in `DC` (Domain Component).  
D. A `User` custom resource must first be created in `etcd` via `kubectl create user sarah`.

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer: B**

**Explanation:**  
Kubernetes does not have a `User` API object stored in `etcd`. For X.509 client certificate authentication, `kube-apiserver` extracts the **Common Name (`CN`)** as the Username (e.g., `/CN=sarah`) and each **Organization (`O`)** field as a Group membership (e.g., `/O=system:masters` or `/O=sre-team`).
</details>

---

### Question 11
Why do `kube-scheduler` and `kube-controller-manager` enable `--leader-elect=true` by default while `kube-apiserver` does not?

A. `kube-apiserver` is stateless and horizontally scalable behind a load balancer, whereas running multiple active schedulers or controller-managers simultaneously without leader election would cause conflicting scheduling binds and duplicate reconciliation loops.  
B. `kube-apiserver` uses Raft consensus internally.  
C. Only one `kube-apiserver` instance is allowed in a Kubernetes cluster.  
D. `kube-scheduler` stores cluster state locally on disk.

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer: A**

**Explanation:**  
In a multi-control-plane HA cluster, all `kube-apiserver` instances are active simultaneously behind a TCP load balancer because state consistency is enforced by `etcd`. However, if 3 `kube-controller-manager` instances or 3 `kube-scheduler` instances acted simultaneously, they would race to create ReplicaSet pods or bind Pods to nodes. They use `Lease` objects in `kube-system` so only 1 leader is active at a time.
</details>

---

### Question 12
You configured a new Linux node with `containerd` and `kubelet`, where `kubelet` uses `cgroupDriver: systemd` (the default in `kubeadm`). However, in `/etc/containerd/config.toml`, `SystemdCgroup` is left at `false`. What is the architectural consequence of this mismatch?

A. `containerd` automatically detects `kubelet`'s setting and overrides `config.toml`.  
B. The node will have two separate cgroup hierarchies (`cgroupfs` and `systemd`) managing resource limits, causing instability, resource accounting errors, and `kubelet` / control-plane pods failing or crashlooping under load.  
C. `kube-proxy` will fail to program `iptables` rules.  
D. Only `hostNetwork: true` pods will be affected.

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer: B**

**Explanation:**  
Both the container runtime (`containerd`) and `kubelet` must use the exact same cgroup driver (`systemd` on systemd-based Linux distros). Setting `SystemdCgroup = true` under `[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]` in `/etc/containerd/config.toml` and restarting `containerd` is a critical prerequisite for stable node operation.
</details>
