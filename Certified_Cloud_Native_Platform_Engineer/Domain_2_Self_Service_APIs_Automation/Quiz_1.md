# Domain 2: Self-Service Platform APIs & Automation — Quiz 1

**Exam:** Certified Cloud Native Platform Engineer (CNPE)  
**Format:** 10 Scenario & Performance Questions with Detailed Rationales

---

### Question 1: Crossplane XRD vs Composition
In Crossplane, what is the exact architectural difference between a CompositeResourceDefinition (XRD) and a Composition?
- A. XRD runs in the cloud; Composition runs locally.
- B. They are interchangeable aliases.
- C. XRD creates virtual machines; Composition creates Docker containers.
- D. XRD defines the OpenAPI schema and type of the custom API; Composition defines the concrete implementation blueprint specifying which underlying Managed Resources to instantiate and patch.

---

### Question 2: Scope of Composite Resource Claims (XRC)
Why does Crossplane support namespaced Composite Resource Claims (XRC) alongside cluster-scoped Composite Resources (XR)?
- A. Claims do not use memory.
- B. To allow developers working inside isolated namespaces to self-serve infrastructure using standard Kubernetes RBAC without requiring cluster-admin privileges.
- C. To bypass Kubernetes security checks.
- D. Because claims run faster.

---

### Question 3: Operator Reconcile Loop Idempotence
Why must the `Reconcile(ctx, req)` function in a Kubernetes operator always be idempotent?
- A. Because network latency requires multiple calls.
- B. Because CRDs cannot store state.
- C. Because Kubernetes can enqueue duplicate reconciliation events at any time; running `Reconcile()` repeatedly for the same state must produce the exact same outcome without creating duplicate cloud resources or corrupting data.
- D. To reduce Go binary compilation time.

---

### Question 4: Patching in Crossplane Compositions
In a Crossplane Composition, which patch type is used to copy an input attribute from the developer's claim spec into the cloud provider's managed resource spec?
- A. `CombineFromComposite`
- B. `FromCompositeFieldPath`
- C. `ToCompositeFieldPath`
- D. `TransformPatch`

---

### Question 5: Backstage Software Catalog Entity Model
In the Backstage Catalog, what entity type represents an individual deployed piece of software (e.g. a microservice, frontend website, or cron daemon)?
- A. `Component`
- B. `Domain`
- C. `Resource`
- D. `System`

---

### Question 6: Provider Authentication in Crossplane
How should a Crossplane Provider (such as `provider-aws` or `provider-digitalocean`) authenticate to remote cloud APIs in a hardened production environment?
- A. Providers do not require authentication.
- B. SSH directly into the cloud provider.
- C. Use short-lived Workload Identity / IAM Roles for Service Accounts (IRSA) via the ProviderConfig resource, avoiding long-lived static secrets.
- D. Hardcode root API tokens in the provider Deployment YAML.

---

### Question 7: Status Subresource in Custom Resource Definitions
Why is the `/status` subresource enabled on CRDs when authoring Kubernetes operators?
- A. It separates modifications to `spec` (developer desired state) from `status` (controller observed state), allowing RBAC policies to restrict who can edit `spec` while permitting the operator to write status updates without incrementing `metadata.generation`.
- B. It reduces etcd disk size by 50%.
- C. It accelerates DNS resolution.
- D. To encrypt the spec.

---

### Question 8: Crossplane Composition Functions
Why did Crossplane introduce Composition Functions (e.g. `function-patch-and-transform`, Python/Go/KCL functions) in v1.14+?
- A. To eliminate the need for Kubernetes.
- B. To replace rigid, deeply nested YAML templates with expressive, testable programming paradigms supporting complex loops, conditionals, and validation logic.
- C. To disable cloud APIs.
- D. To compile HCL directly into CRDs.

---

### Question 9: Scaffolder Actions in Backstage
In a Backstage Software Template (`template.yaml`), what action publishes the newly rendered skeleton files to a GitHub repository?
- A. `fetch:template`
- B. `git:commit`
- C. `catalog:register`
- D. `publish:github`

---

### Question 10: Deleting a Composite Resource Claim
When a developer deletes their `PostgresInstance` claim in a tenant namespace, what happens to the underlying cloud database managed by Crossplane?
- A. Nothing happens.
- B. The cluster crashes.
- C. The cloud database is immediately backed up to S3.
- D. If `deletionPolicy: Delete` is set (default), Crossplane deletes the underlying cloud resource; if `deletionPolicy: Orphan` is set, Crossplane deletes the Kubernetes records but leaves the live cloud database running.

---

## Answer Key & Explanations

1. **D** — XRD specifies the API contract; Composition specifies the orchestration implementation.
2. **B** — XRCs bring multi-tenant namespace isolation to external cloud infrastructure requests.
3. **C** — Idempotence is the primary requirement for level-triggered control loops.
4. **B** — `FromCompositeFieldPath` maps values from the top-level XR down to managed resources.
5. **A** — A `Component` in Backstage models a single microservice, website, or library.
6. **C** — OIDC / IRSA provides temporary, least-privilege cloud credentials without static tokens.
7. **A** — The `/status` subresource prevents reconcile loops from triggering unnecessary spec mutation events.
8. **B** — Composition Functions bring true programming power and unit testability to control plane authoring.
9. **D** — `publish:github` creates and configures the target Git repository with rendered code.
10. **D** — `deletionPolicy: Orphan` protects critical stateful data from accidental claim deletions.
