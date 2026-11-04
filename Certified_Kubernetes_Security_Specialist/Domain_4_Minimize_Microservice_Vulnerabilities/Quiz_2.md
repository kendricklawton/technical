# Domain 4: Minimize Microservice Vulnerabilities — Quiz 2 (Admission Controllers, OPA Gatekeeper, Kyverno & CEL)

**Exam:** Certified Kubernetes Security Specialist (CKS)  
**Format:** 10 Performance Scenario & Multiple Choice Security Questions with Explanations

---

### Question 1: Mutating vs Validating Admission Execution Order
During a `kubectl apply` request to `kube-apiserver`, in what exact sequence do Authentication, Authorization, Mutating Admission, Object Schema Validation, and Validating Admission execute prior to persisting the object in `etcd`?
- A. Validating Admission $\to$ Mutating Admission $\to$ RBAC $\to$ Authentication.
- B. Schema Validation $\to$ Authentication $\to$ RBAC.
- C. Authentication $\to$ Authorization (RBAC) $\to$ Mutating Admission Controllers $\to$ Object Schema Validation $\to$ Validating Admission Controllers $\to$ `etcd` persistence.
- D. `etcd` persistence $\to$ Mutating Admission $\to$ Validating Admission.

---

### Question 2: Why Mutating Admission Runs Before Validating Admission
Why is it architecturally mandatory for Mutating Admission Webhooks to run **before** Validating Admission Webhooks (and why can a Validating Admission Webhook never modify the incoming object)?
- A. Because RBAC runs after Validating Admission.
- B. Because Validating Webhooks run on the worker node kubelet.
- C. Because Mutating Webhooks do not use TLS.
- D. Because a Mutating Webhook may inject sidecars, default `securityContext` fields, or image digests that the subsequent Validating Webhook must inspect to verify the final mutated object still complies with cluster security policies before writing to `etcd`.

---

### Question 3: Webhook `failurePolicy: Fail` vs `Ignore` Deadlock Trap
An administrator deploys an external `ValidatingAdmissionWebhook` (such as OPA Gatekeeper or Kyverno) with `failurePolicy: Fail` and no `namespaceSelector` exclusions. When the cluster restarts or all Gatekeeper/Kyverno pods crash simultaneously, what happens when the ReplicaSet tries to recreate the Gatekeeper/Kyverno pods?
- A. `kubelet` starts the webhook pods directly without `kube-apiserver`.
- B. A cluster-wide deadlock occurs: `kube-apiserver` attempts to call the webhook to validate the creation of the webhook's own Pod, fails because the webhook Pod is down, and rejects the Pod creation (`failurePolicy: Fail`)—which is why `kube-system` and the webhook's own namespace must be excluded via `namespaceSelector` / `matchConditions`.
- C. `kube-apiserver` automatically switches to `Ignore`.
- D. `etcd` restores the pods.

---

### Question 4: Native Kubernetes `ValidatingAdmissionPolicy` (CEL — Kubernetes 1.30+ GA)
Which two native Kubernetes resources allow you to enforce custom admission rules (such as forbidding `:latest` image tags or requiring specific labels) directly inside `kube-apiserver` using Common Expression Language (CEL) **without** deploying an external webhook server?
- A. `ConstraintTemplate` and `Constraint`
- B. `PodSecurityPolicy` and `ClusterRoleBinding`
- C. `ValidatingAdmissionPolicy` and `ValidatingAdmissionPolicyBinding` (`admissionregistration.k8s.io/v1`)
- D. `NetworkPolicy` and `IngressClass`

---

### Question 5: Writing a CEL Expression in `ValidatingAdmissionPolicy`
You are writing a `ValidatingAdmissionPolicy` to block any Pod whose containers use an image from outside `registry.internal.corp/`. Which CEL expression under `spec.validations[].expression` enforces this for all standard containers in a Pod?
- A. `containers[*].image == "registry.internal.corp/*"`
- B. `object.spec.containers.all(c, c.image.startsWith('registry.internal.corp/'))`
- C. `SELECT * FROM containers WHERE image NOT LIKE 'registry.internal.corp/%'`
- D. `package kubernetes.admission; deny[msg] { ... }`

---

### Question 6: OPA Gatekeeper Architecture (`ConstraintTemplate` vs `Constraint`)
In OPA Gatekeeper, how are reusable Rego policy logic and specific enforcement instances separated across Custom Resource Definitions?
- A. `Constraint` holds the Rego code, and `ConstraintTemplate` holds the namespace list.
- B. OPA Gatekeeper uses YAML patterns instead of Rego.
- C. Both Rego and parameters live inside a `ConfigMap`.
- D. A `ConstraintTemplate` (`templates.gatekeeper.sh/v1`) defines the Rego source code (`violation[{"msg": msg}]`) and the OpenAPI parameter schema (which dynamically registers a new CRD), while a `Constraint` (an instance of that generated CRD under `constraints.gatekeeper.sh/v1beta1`) specifies the `match` target (`kinds`, `namespaces`) and `parameters`.

---

### Question 7: OPA Gatekeeper `enforcementAction` Modes (`deny`, `dryrun`, `warn`)
In an OPA Gatekeeper `Constraint` resource, you want to test a new Rego policy in production so that violating `kubectl apply` requests are **not** blocked, but an immediate warning is returned to the user's terminal AND violations are recorded in the Constraint's `status.violations`. Which `spec.enforcementAction` value should you set?
- A. `audit`
- B. `warn` (or `dryrun` for silent auditing without user terminal warnings; `deny` is the default that blocks the request).
- C. `ignore`
- D. `permissive`

---

### Question 8: Kyverno `ClusterPolicy` Rule Types (`validate`, `mutate`, `generate`, `verifyImages`)
Unlike OPA Gatekeeper, which uses Rego and focuses on validation and mutation, which four primary rule capabilities does a Kyverno `ClusterPolicy` provide natively in Kubernetes YAML?
- A. `route`, `proxy`, `trace`, `log`
- B. `compile`, `build`, `scan`, `deploy`
- C. `encrypt`, `decrypt`, `backup`, `restore`
- D. `validate` (enforce/audit schema patterns or CEL), `mutate` (JSON patch / strategic merge patch objects on creation), `generate` (automatically create companion resources like default `NetworkPolicy` or `ResourceQuota` when a new `Namespace` is created), and `verifyImages` (verify Cosign signatures and SLSA attestations).

---

### Question 9: `AlwaysPullImages` Admission Controller Security Purpose
In a multi-tenant Kubernetes cluster, Tenant A deploys a Pod using valid `imagePullSecrets` to pull private image `corp.registry.io/secret-algo:v1` onto `worker-1`. Five minutes later, Tenant B (who does **not** possess `imagePullSecrets`) schedules a Pod onto `worker-1` requesting `corp.registry.io/secret-algo:v1` with `imagePullPolicy: IfNotPresent`. How do you prevent Tenant B from launching a container from the locally cached private image on `worker-1` without credentials?
- A. Enable the `AlwaysPullImages` admission plugin in `--enable-admission-plugins` on `kube-apiserver`, which mutates every Pod's `imagePullPolicy` to `Always` so the kubelet always verifies registry pull credentials before starting the container.
- B. Delete `/var/lib/containerd`.
- C. Enable `NodeRestriction`.
- D. Set `readOnlyRootFilesystem: true`.

---

### Question 10: `NodeRestriction` Admission Controller Scope
What specific privilege escalation vector does the built-in `NodeRestriction` admission plugin block if a single worker node (`worker-2`) is compromised by an attacker who steals its `/etc/kubernetes/kubelet.conf` client certificate?
- A. It restricts the `system:node:<nodeName>` identity so the kubelet can only modify its **own** `Node` API object and `Pod` status objects bound to that specific node—and furthermore blocks the kubelet from adding/modifying `node-restriction.kubernetes.io/*` labels on its own `Node` object (preventing the compromised node from relabeling itself to attract high-security or control-plane workloads).
- B. It encrypts `etcd`.
- C. It blocks SSH to the node.
- D. It disables `hostPath`.

---

## Answer Key & Detailed Explanations

1. **C** — Every API write request passes through AuthN $\to$ AuthZ $\to$ Mutating Admission $\to$ Schema Validation $\to$ Validating Admission $\to$ etcd.
2. **D** — If Validating Admission ran before Mutating Admission, a Mutating Webhook could alter an already-validated Pod into an insecure state without re-validation.
3. **B** — Always exclude `kube-system` and the admission webhook's own namespace in `ValidatingWebhookConfiguration` / `MutatingWebhookConfiguration` when using `failurePolicy: Fail`.
4. **C** — `ValidatingAdmissionPolicy` + `ValidatingAdmissionPolicyBinding` compile and evaluate CEL expressions directly in-process inside `kube-apiserver` with zero network webhook latency.
5. **B** — CEL uses `.all(var, predicate)` on lists and `.startsWith()` on strings to validate all containers cleanly.
6. **D** — Gatekeeper's `ConstraintTemplate` defines the CRD schema and Rego logic; the `Constraint` instantiates that CRD with target selectors and parameters.
7. **B** — Gatekeeper supports `deny` (default), `warn` (returns K8s API warning header + audits), and `dryrun` (silent audit only).
8. **D** — Kyverno is Kubernetes-native and supports `validate`, `mutate`, `generate`, and `verifyImages`.
9. **A** — Without `AlwaysPullImages`, any tenant scheduled on the same node can run a cached private image using `IfNotPresent` or `Never` without authenticating to the registry.
10. **A** — `NodeRestriction` confines each kubelet's write permissions strictly to its own `Node` and bound `Pod` objects and prevents self-labeling with `node-restriction.kubernetes.io/*` keys.
