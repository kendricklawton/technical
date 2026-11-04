# Domain 2: Self-Service Platform APIs & Automation — Quiz 2

**Exam:** Certified Cloud Native Platform Engineer (CNPE)  
**Format:** 10 Advanced Control Plane & Operator Architecture Questions

---

### Question 1: Connection Secret Publishing in Crossplane
When Crossplane provisions a cloud database, how are the resulting connection credentials (host, port, username, password) safely returned to the developer?
- A. By emailing the developer.
- B. By saving credentials to `/tmp`.
- C. By writing credentials into the Pod logs.
- D. By configuring `writeConnectionSecretToRef` in the Composition and Claim, which causes Crossplane to emit a Kubernetes `Secret` containing connection details into the developer's namespace.

---

### Question 2: Validating Admission Webhooks in Operators
Why should an operator author include a Validating Admission Webhook alongside their CRD schema?
- A. To disable TLS encryption.
- B. Webhooks make the cluster faster.
- C. Webhooks are required by Git.
- D. OpenAPI v3 schemas cannot validate dynamic constraints or cross-field dependencies (e.g. `if engine == "postgres", then version >= 14`), whereas a webhook can reject invalid requests synchronously before they are stored in etcd.

---

### Question 3: Mutating Admission Webhooks for Default Injection
What is the primary role of a Mutating Admission Webhook in a platform engineering control plane?
- A. To scan Docker images.
- B. To automatically inject organizational defaults, sidecars, or security labels (e.g. injecting standard node selectors or tags) into incoming resource manifests before validation.
- C. To format HCL code.
- D. To delete failed pods.

---

### Question 4: Composition Rejection & Selector Matching
A developer submits a `DatabaseClaim` requesting `storageGB: 50` and labels `provider: digitalocean`. However, the claim stays in `Pending` with reason: `No matching composition found`. What is the root cause?
- A. Crossplane is disabled.
- B. No Composition resource exists that satisfies both `compositeTypeRef` and the claim's `compositionSelector` label match rules.
- C. DigitalOcean is offline.
- D. The cluster is out of disk space.

---

### Question 5: Environment Configs in Crossplane
What is the purpose of `EnvironmentConfig` in modern Crossplane?
- A. To provide shared, in-memory configuration maps (such as network VPC IDs, region mappings, or account numbers) that can be queried and patched into Compositions without hardcoding values.
- B. To manage Docker containers.
- C. To store environment variables for Linux.
- D. To configure developer laptops.

---

### Question 6: Finalizers in Kubernetes Operators
Why does an operator attach a `metadata.finalizer` to a custom resource when it is created?
- A. To speed up pod scheduling.
- B. Finalizers are only used by Docker.
- C. To prevent the resource from being updated.
- D. To block the Kubernetes API server from deleting the object from etcd until the operator has safely de-provisioned all backing external cloud resources (e.g. deleting cloud load balancers or disks).

---

### Question 7: Backstage Catalog Relations
In Backstage, how do you model that a `Component` (e.g. `order-service`) consumes an `API` (e.g. `payment-api`)?
- A. By linking them in Git.
- B. By creating an AWS Route 53 record.
- C. By specifying `consumesApis: [payment-api]` in the component's `catalog-info.yaml` spec.
- D. By setting environment variables.

---

### Question 8: Reconcile Rate Limiting in Controller-Runtime
Why does controller-runtime use an exponential backoff rate limiter (`workqueue.NewItemExponentialFailureRateLimiter`) when a reconciliation returns an error?
- A. To prevent a crashing operator or failing cloud API call from hammering the API server and exhausting CPU or remote API rate limits.
- B. To save memory.
- C. To delay cluster upgrades.
- D. To simulate network jitter.

---

### Question 9: Composite Resource Ownership
In Crossplane, which resource is the Kubernetes `OwnerReference` parent of the underlying Managed Resources?
- A. The developer's laptop.
- B. The Composite Resource (XR).
- C. The worker node.
- D. The Kubernetes namespace.

---

### Question 10: Platform as a Product (PaaP) Principles
According to the CNCF Platforms White Paper, what is the primary metric of success for an Internal Developer Platform (IDP)?
- A. Enforcing mandatory CLI tools for all engineers.
- B. The number of tools installed in the cluster.
- C. Voluntary internal adoption, reduced developer cognitive load, and accelerated lead time for changes (DORA metrics).
- D. The total number of Git commits per day.

---

## Answer Key & Explanations

1. **D** — `writeConnectionSecretToRef` creates namespaced Kubernetes Secrets for database consumption.
2. **D** — Validating webhooks enforce complex business logic and cross-attribute invariants before commit to etcd.
3. **B** — Mutating webhooks inject automated platform standards and security configurations transparently.
4. **B** — A claim requires a matching Composition satisfying both its type and selector labels.
5. **A** — `EnvironmentConfig` decouples global platform environment settings from individual composition definitions.
6. **D** — Finalizers ensure asynchronous cleanup of external cloud infrastructure before etcd removes the metadata.
7. **C** — Backstage `catalog-info.yaml` defines explicit relationships between software components, APIs, and systems.
8. **A** — Exponential backoff prevents denial-of-service loops against Kubernetes and cloud provider APIs.
9. **B** — The XR acts as the parent controller managing the lifecycle of its child managed resources.
10. **C** — Internal platforms must be treated as products focused on user enablement and cognitive load reduction.
