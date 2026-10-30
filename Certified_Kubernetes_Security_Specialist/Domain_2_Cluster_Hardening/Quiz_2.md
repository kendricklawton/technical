# CKS Domain 2: Cluster Hardening — Quiz 2 (RBAC Least Privilege & Wildcard Auditing)

Test your mastery of Kubernetes RBAC scoping (`Role`, `ClusterRole`, `RoleBinding`, `ClusterRoleBinding`), `resourceNames`, Secret protection, and privilege escalation vectors.

---

### Question 1
You have a `ClusterRole` named `secret-reader` that allows `get`, `list`, and `watch` on `secrets`. You create a namespace-scoped **`RoleBinding`** named `dev-secret-binding` inside the `dev` namespace that references `ClusterRole/secret-reader` and binds it to `ServiceAccount/dev:app-sa`. Which Secrets can `dev:app-sa` read?

- A) Secrets in **all** namespaces across the cluster, because `secret-reader` is a `ClusterRole`.
- B) Only Secrets inside the **`dev` namespace**, because a `RoleBinding` restricts the scope of a referenced `ClusterRole` to the `RoleBinding`'s own namespace.
- C) None; a `RoleBinding` cannot reference a `ClusterRole`.
- D) Only Secrets in the `default` namespace.

<details>
<summary>Reveal Answer & Explanation</summary>

**Correct Answer: B**

**Explanation:**
When a namespace-scoped `RoleBinding` references a `ClusterRole`, the subject receives those permissions **only within the `RoleBinding`'s namespace** (`dev`). This is a powerful Kubernetes pattern that lets you define reusable `ClusterRole` templates without granting cluster-wide access.
</details>

---

### Question 2
A developer asks for RBAC permission for their Pod (`ServiceAccount` `web-sa` in namespace `prod`) to read **only** a single Secret named `stripe-api-key` and no other Secrets in `prod`. You write the following `Role`:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: single-secret-role
  namespace: prod
rules:
  - apiGroups: [""]
    resources: ["secrets"]
    verbs: ["get", "list"]
    resourceNames: ["stripe-api-key"]
```
When the developer runs `kubectl get secrets -n prod --as=system:serviceaccount:prod:web-sa`, it fails with `Forbidden`, even though `list` is in `verbs`. Why, and what is the security implication of `list` on `secrets`?

- A) Kubernetes RBAC **cannot** filter `list` (or `watch` or `create`) requests by `resourceNames`. Furthermore, granting `list` on `secrets` without `resourceNames` returns the full secret `data` of **every** Secret in the namespace! The developer should use `kubectl get secret stripe-api-key` with `verbs: ["get"]` and `resourceNames: ["stripe-api-key"]`.
- B) `secrets` belongs to the `security.k8s.io` API group, not `""`.
- C) `resourceNames` only works on `ClusterRole`, not `Role`.
- D) `list` requires `watch` to also be specified.

<details>
<summary>Reveal Answer & Explanation</summary>

**Correct Answer: A**

**Explanation:**
In Kubernetes RBAC, `resourceNames` only applies to requests that target a specific named resource in the URL path (`get`, `update`, `patch`, `delete` on `/api/v1/namespaces/prod/secrets/stripe-api-key`). Collection verbs like `list`, `watch`, and `create` operate on `/api/v1/namespaces/prod/secrets` (no resource name in path) and **cannot** be restricted by `resourceNames`. More importantly, granting `list` on `secrets` lets a client retrieve the full plaintext contents of **all** Secrets in the namespace! Always restrict single-Secret access to `verbs: ["get"]` + `resourceNames: ["<secret-name>"]`.
</details>

---

### Question 3
Which `kubectl` command accurately checks whether the ServiceAccount `ci-runner` in namespace `cicd` has permission to create `deployments` in the `production` namespace?

- A) `kubectl auth can-i create deployments -n production --as=ci-runner`
- B) `kubectl auth can-i create deployments -n production --as=system:serviceaccount:cicd:ci-runner`
- C) `kubectl auth can-i create deployments -n cicd --as=system:serviceaccount:production:ci-runner`
- D) `kubectl can-i create deployments --serviceaccount=cicd:ci-runner -n production`

<details>
<summary>Reveal Answer & Explanation</summary>

**Correct Answer: B**

**Explanation:**
When impersonating a ServiceAccount with `--as`, you must use its full Kubernetes username format: `system:serviceaccount:<sa-namespace>:<sa-name>` (`system:serviceaccount:cicd:ci-runner`), and set `-n <target-namespace>` (`-n production`) to specify where the action takes place.
</details>

---

### Question 4
During an RBAC audit, you find a `RoleBinding` in the `production` namespace with the following `subjects` block:
```yaml
subjects:
  - kind: Group
    name: system:serviceaccounts
    apiGroup: rbac.authorization.k8s.io
```
Which ServiceAccounts are granted the permissions of this `RoleBinding`?

- A) Only the `default` ServiceAccount in the `production` namespace.
- B) All ServiceAccounts inside the `production` namespace only.
- C) **Every ServiceAccount in every namespace across the entire cluster** (allow-all ServiceAccounts to access `production`)!
- D) Only ServiceAccounts in `kube-system`.

<details>
<summary>Reveal Answer & Explanation</summary>

**Correct Answer: C**

**Explanation:**
* `Group: system:serviceaccounts` matches **all ServiceAccounts in all namespaces** cluster-wide! Even if placed inside a namespace-scoped `RoleBinding` in `production`, it grants every Pod in `dev`, `staging`, or `default` access to resources inside `production`!
* `Group: system:serviceaccounts:production` matches all ServiceAccounts in the `production` namespace.
* `Kind: ServiceAccount, name: my-sa, namespace: production` matches only that single ServiceAccount.
</details>

---

### Question 5
Which `apiGroups` value must be specified in a `Role` rule that grants `get`, `list`, and `watch` permissions on `deployments` and `statefulsets`?

- A) `apiGroups: [""]`
- B) `apiGroups: ["apps"]`
- C) `apiGroups: ["extensions"]`
- D) `apiGroups: ["batch"]`

<details>
<summary>Reveal Answer & Explanation</summary>

**Correct Answer: B**

**Explanation:**
* Core resources (`pods`, `services`, `secrets`, `configmaps`, `persistentvolumeclaims`, `namespaces`, `nodes`) belong to the core API group `""`.
* `deployments`, `statefulsets`, `daemonsets`, and `replicasets` belong to `apiGroups: ["apps"]`.
* `jobs` and `cronjobs` belong to `apiGroups: ["batch"]`.
* `networkpolicies` and `ingresses` belong to `apiGroups: ["networking.k8s.io"]`.
If you are ever unsure on the exam, run `kubectl api-resources | grep deployments`.
</details>

---

### Question 6
Why does Kubernetes prevent a user who has `create` and `update` permissions on `roles` and `rolebindings` in a namespace from creating a `Role` that grants `secrets` access if the user does not *already* possess `secrets` access themselves (unless they have the `escalate` verb)?

- A) Because `Role` objects are immutable once created.
- B) Built-in **RBAC Privilege Escalation Prevention**: Kubernetes blocks a user from creating or updating a `Role` or `RoleBinding` with permissions exceeding the user's own current permissions, unless the user is explicitly granted the `escalate` or `bind` verb on those `roles`/`clusterroles`.
- C) Because `secrets` can only be bound via `ClusterRoleBinding`.
- D) Because `Role` creation requires `O=system:masters`.

<details>
<summary>Reveal Answer & Explanation</summary>

**Correct Answer: B**

**Explanation:**
Kubernetes RBAC enforces privilege escalation prevention at the API server level: you cannot grant away permissions you do not already hold, unless you have the `escalate` verb (on `roles`/`clusterroles`) or `bind` verb (allowing you to bind specific existing `roles`/`clusterroles`).
</details>

---

### Question 7
An attacker compromises a container whose ServiceAccount has **only** the following RBAC rule in the `production` namespace:
```yaml
rules:
  - apiGroups: [""]
    resources: ["pods/exec"]
    verbs: ["create"]
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["get", "list"]
```
How can the attacker escalate privileges within the `production` namespace using this permission?

- A) `pods/exec` cannot be used for privilege escalation because `exec` uses `GET`, not `CREATE`.
- B) The attacker can run `kubectl exec -n production <any-pod> -- cat /var/run/secrets/kubernetes.io/serviceaccount/token` (or inspect environment variables/mounted secrets) inside any other Pod in `production`, stealing higher-privileged ServiceAccount tokens or database credentials!
- C) The attacker can modify the `Node` object directly.
- D) The attacker can read `/etc/kubernetes/admin.conf` on the control plane.

<details>
<summary>Reveal Answer & Explanation</summary>

**Correct Answer: B**

**Explanation:**
In Kubernetes API versions, `kubectl exec` initiates a WebSocket/SPDY subresource connection using HTTP `POST` (`create` verb on `pods/exec`). Anyone with `create` on `pods/exec` can execute arbitrary shell commands inside **any Pod** in that namespace and steal its mounted Secrets or ServiceAccount token!
</details>

---

### Question 8
What is `ClusterRole` aggregation (`aggregationRule.clusterRoleSelectors`) used for in Kubernetes RBAC?

- A) Combining multiple `ServiceAccounts` into a single Pod.
- B) Dynamically combining the rules of multiple label-matched `ClusterRoles` into a single parent `ClusterRole` (such as the built-in `admin`, `edit`, and `view` roles automatically picking up CRD permissions).
- C) Encrypting RBAC rules in `etcd`.
- D) Converting `ClusterRoleBindings` into `RoleBindings`.

<details>
<summary>Reveal Answer & Explanation</summary>

**Correct Answer: B**

**Explanation:**
Aggregated `ClusterRoles` use `aggregationRule.clusterRoleSelectors` so that when you install a CustomResourceDefinition (CRD) and create a `ClusterRole` labeled `rbac.authorization.k8s.io/aggregate-to-view: "true"`, the controller manager automatically merges those rules into the built-in `view` `ClusterRole`.
</details>

---

### Question 9
You want to create a `RoleBinding` named `read-pods-rb` in namespace `audit-ns` that binds the built-in `ClusterRole` `view` to ServiceAccount `auditor-sa` in namespace `audit-ns`. Which imperative `kubectl` command is correct?

- A) `kubectl -n audit-ns create rolebinding read-pods-rb --role=view --serviceaccount=audit-ns:auditor-sa`
- B) `kubectl -n audit-ns create rolebinding read-pods-rb --clusterrole=view --serviceaccount=audit-ns:auditor-sa`
- C) `kubectl create clusterrolebinding read-pods-rb --clusterrole=view --serviceaccount=audit-ns:auditor-sa -n audit-ns`
- D) `kubectl -n audit-ns bind clusterrole view --to=auditor-sa`

<details>
<summary>Reveal Answer & Explanation</summary>

**Correct Answer: B**

**Explanation:**
When creating a `RoleBinding` that references a `ClusterRole`, you must pass `--clusterrole=view` (not `--role=view`, which sets `roleRef.kind: Role` instead of `roleRef.kind: ClusterRole`).
</details>

---

### Question 10
Which built-in `ClusterRole` is intended for read-only inspection of most objects in a namespace, and intentionally **excludes** reading `secrets`, `roles`, and `rolebindings` to prevent credential leakage and privilege escalation?

- A) `cluster-admin`
- B) `admin`
- C) `edit`
- D) `view`

<details>
<summary>Reveal Answer & Explanation</summary>

**Correct Answer: D**

**Explanation:**
The built-in `view` `ClusterRole` allows `get`, `list`, and `watch` on non-sensitive namespace resources (`pods`, `services`, `deployments`, `configmaps`), but explicitly **excludes** `secrets` (to prevent credential theft) and `roles`/`rolebindings`.
</details>
