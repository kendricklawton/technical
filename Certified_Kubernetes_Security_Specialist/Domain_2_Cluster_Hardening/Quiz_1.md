# CKS Domain 2: Cluster Hardening — Quiz 1 (API Server Security & NodeRestriction)

Test your understanding of `kube-apiserver` authentication, authorization modes, X.509 certificate groups, and the `NodeRestriction` admission plugin.

---

### Question 1
A security engineer notices that an external user with a client certificate `CN=ci-deployer, O=system:masters` is able to delete any Namespace and read all Secrets in the cluster, even after the engineer deletes the `cluster-admin` `ClusterRoleBinding`. Why does `ci-deployer` still have full cluster root access, and how can this access be revoked?

- A) Because `kube-apiserver` caches RBAC bindings for 24 hours; restart `kube-apiserver` to revoke access.
- B) Because the `system:masters` group is hardcoded into `kube-apiserver`'s superuser authorizer and completely bypasses RBAC. Since Kubernetes does not support X.509 CRLs natively for `--client-ca-file`, you must rotate the cluster CA (`/etc/kubernetes/pki/ca.crt`) or remove that client's access to the certificate/key.
- C) Because `CN=ci-deployer` matches a built-in `Node` authorizer rule.
- D) Because `--anonymous-auth=true` is enabled on `kube-apiserver`.

<details>
<summary>Reveal Answer & Explanation</summary>

**Correct Answer: B**

**Explanation:**
In Kubernetes, any client certificate signed by the cluster CA with Organization `O=system:masters` is authenticated into the built-in `system:masters` superuser group, which bypasses RBAC checks altogether. Deleting RBAC `ClusterRoleBindings` has zero effect on `system:masters`. That is why CIS Benchmark (1.1.20) forbids using `system:masters` for anything other than break-glass `/etc/kubernetes/admin.conf`.
</details>

---

### Question 2
Which `kube-apiserver` configuration ensures that `kubelet` credentials (`system:node:<nodeName>`) can only read Secrets and ConfigMaps mounted by Pods actually scheduled onto that specific node, and can only modify their own `Node` API object?

- A) `--authorization-mode=RBAC` and `--enable-admission-plugins=NamespaceLifecycle`
- B) `--authorization-mode=Node,RBAC` and `--enable-admission-plugins=NodeRestriction`
- C) `--authorization-mode=AlwaysAllow` and `--enable-admission-plugins=NodeRestriction`
- D) `--authorization-mode=Webhook` and `--anonymous-auth=false`

<details>
<summary>Reveal Answer & Explanation</summary>

**Correct Answer: B**

**Explanation:**
The **Node Authorizer** (`--authorization-mode=Node,RBAC`) restricts *read* access of a `kubelet` to only the Secrets, ConfigMaps, and PVs attached to Pods bound to that node. The **`NodeRestriction` Admission Plugin** (`--enable-admission-plugins=NodeRestriction`) restricts *write* operations of a `kubelet` so it can only modify its own `Node` object and `Pod` objects bound to its node, and cannot modify `node-restriction.kubernetes.io/*` labels.
</details>

---

### Question 3
Suppose a compromised worker node `node02` attempts to attract high-value PCI-DSS Pods by adding a label to its own `Node` object using its kubelet client certificate (`/etc/kubernetes/kubelet.conf`). Assuming `NodeRestriction` is enabled on `kube-apiserver`, which label key prefix prevents `node02` from adding or modifying the label?

- A) `security.kubernetes.io/`
- B) `node-restriction.kubernetes.io/`
- C) `kubelet.kubernetes.io/`
- D) `restricted.k8s.io/`

<details>
<summary>Reveal Answer & Explanation</summary>

**Correct Answer: B**

**Explanation:**
The `NodeRestriction` admission plugin specifically blocks `kubelet` users (`system:nodes`) from adding, updating, or removing any `Node` label with the prefix `node-restriction.kubernetes.io/` (as well as `kubernetes.io/` core security labels). Administrators can set `node-restriction.kubernetes.io/pci="true"` using `cluster-admin` credentials and reference it in Pod `nodeSelector` or `nodeAffinity`.
</details>

---

### Question 4
What happens when `--anonymous-auth=false` is configured on `kube-apiserver` and a client sends an HTTPS `GET /api/v1/pods` request without a Bearer Token or client certificate?

- A) The request is mapped to `system:anonymous` and evaluated by RBAC, which returns `403 Forbidden`.
- B) The request is immediately rejected at the Authentication stage with HTTP `401 Unauthorized` before reaching the Authorization or Admission stages.
- C) The TCP connection is reset by `etcd`.
- D) The request succeeds if the Pod is in the `default` namespace.

<details>
<summary>Reveal Answer & Explanation</summary>

**Correct Answer: B**

**Explanation:**
With `--anonymous-auth=false`, the API server rejects unauthenticated requests at the Authentication layer (`401 Unauthorized`) rather than assigning them the `system:anonymous` identity and forwarding them to RBAC (`403 Forbidden`).
</details>

---

### Question 5
In `/etc/kubernetes/manifests/kube-apiserver.yaml`, you see:
```yaml
- --authorization-mode=AlwaysAllow,RBAC
```
How does `kube-apiserver` evaluate an incoming request from a restricted ServiceAccount?

- A) It denies the request unless RBAC also allows it, because all authorizers must agree.
- B) It approves the request immediately! Authorizers in `--authorization-mode` are evaluated in order with logical OR semantics: as soon as `AlwaysAllow` approves the request, `RBAC` is never consulted.
- C) `kube-apiserver` fails to start because `AlwaysAllow` cannot be combined with `RBAC`.
- D) It evaluates `RBAC` first because `RBAC` has higher priority than `AlwaysAllow`.

<details>
<summary>Reveal Answer & Explanation</summary>

**Correct Answer: B**

**Explanation:**
Multiple authorizers configured in `--authorization-mode` are evaluated sequentially from left to right. If an authorizer returns **Allow** or **Deny**, that decision is final; the next authorizer is only consulted if the previous one returns **NoOpinion**. Since `AlwaysAllow` returns **Allow** for every request, placing `AlwaysAllow` anywhere in `--authorization-mode` completely neutralizes `RBAC`!
</details>

---

### Question 6
When an X.509 client certificate is created for user `alice` belonging to team `secops` and `developers`, how must the certificate's Subject fields be formatted for `kube-apiserver` to recognize her username and groups?

- A) `-subj "/UID=alice/GID=secops,developers"`
- B) `-subj "/CN=alice/O=secops/O=developers"`
- C) `-subj "/O=alice/CN=secops/CN=developers"`
- D) `-subj "/CN=user:alice/OU=group:secops"`

<details>
<summary>Reveal Answer & Explanation</summary>

**Correct Answer: B**

**Explanation:**
Kubernetes X.509 client certificate authentication extracts the **Common Name (`CN`)** as the Username (`alice`) and each **Organization (`O`)** field as a Group (`secops` and `developers`).
</details>

---

### Question 7
Which Kubernetes API resource allows an administrator to submit an X.509 Certificate Signing Request (`.csr`) directly to the cluster and approve it via `kubectl certificate approve <csr-name>`?

- A) `certificates.k8s.io/v1` `CertificateSigningRequest`
- B) `cert-manager.io/v1` `Certificate`
- C) `rbac.authorization.k8s.io/v1` `CertificateApproval`
- D) `v1` `Secret` of type `kubernetes.io/csr`

<details>
<summary>Reveal Answer & Explanation</summary>

**Correct Answer: A**

**Explanation:**
The built-in `certificates.k8s.io/v1` `CertificateSigningRequest` (`csr`) API allows submitting a Base64-encoded PKCS#10 CSR (`spec.request`) with `signerName: kubernetes.io/kube-apiserver-client` and approving it via `kubectl certificate approve`.
</details>

---

### Question 8
Why is granting RBAC permissions (`approve` on `certificatesigningrequests/approval` or `update` on `certificatesigningrequests/status`) considered equivalent to granting `cluster-admin` if `signerName: kubernetes.io/kube-apiserver-client` is allowed?

- A) Because CSRs are stored unencrypted in `etcd`.
- B) Because the user can submit a CSR with `O=system:masters` in the Subject, approve their own CSR, and obtain a valid client certificate with unrestricted cluster superuser privileges!
- C) Because approving a CSR restarts `kube-apiserver`.
- D) Because CSR approval disables NetworkPolicies.

<details>
<summary>Reveal Answer & Explanation</summary>

**Correct Answer: B**

**Explanation:**
Wait—in recent Kubernetes versions, the built-in `kube-controller-manager` CSR signer blocks signing CSRs with `O=system:masters`, *unless* the signer is custom or the user requests another privileged group/username like `system:admin` or an RBAC-privileged user! Therefore, `certificatesigningrequests/approval` is a critical privilege escalation vector audited in Domain 2.
</details>

---

### Question 9
Where is the cluster administrator's kubeconfig file located by default on a `kubeadm` control plane node, and what file permissions does CIS Benchmark require for it?

- A) `/etc/kubernetes/manifests/admin.yaml` with `chmod 755`
- B) `/etc/kubernetes/admin.conf` with `chmod 600` and ownership `root:root`
- C) `/var/lib/kubelet/admin.conf` with `chmod 644`
- D) `/etc/kubernetes/pki/admin.key` with `chmod 644`

<details>
<summary>Reveal Answer & Explanation</summary>

**Correct Answer: B**

**Explanation:**
`kubeadm init` generates `/etc/kubernetes/admin.conf` containing the base64-encoded client certificate and private key for `default-admin` (`O=system:masters`). CIS Benchmark 1.1.13/1.1.14 requires `chmod 600` and `chown root:root` on `/etc/kubernetes/admin.conf`.
</details>

---

### Question 10
When `kube-apiserver` processes an incoming API call that modifies a Pod, in what order do Mutating Admission Controllers and Validating Admission Controllers run?

- A) Validating Admission runs first, then Mutating Admission runs second.
- B) Mutating Admission runs first (and can modify the object), followed by Object Schema Validation, and then Validating Admission runs last on the final mutated object.
- C) Both run in parallel at the same time.
- D) Admission Controllers run before Authentication and Authorization.

<details>
<summary>Reveal Answer & Explanation</summary>

**Correct Answer: B**

**Explanation:**
Admission Control always runs **after** Authentication and Authorization. Within Admission Control, **Mutating** controllers run first (e.g., injecting sidecar containers or default values) because a mutation might affect whether the final object complies with **Validating** controllers (e.g., Pod Security Admission, OPA Gatekeeper, `NodeRestriction`).
</details>
