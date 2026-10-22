# Domain 2: Self-Service Platform APIs & Automation — Quiz 3

**Exam:** Certified Cloud Native Platform Engineer (CNPE)  
**Format:** 10 Performance & Troubleshooting Scenario Questions

---

### Question 1: Crossplane Provider Drift Detection
A developer logs into the AWS console and manually increases the allocated storage on an RDS instance managed by Crossplane from 50 GB to 100 GB. How does Crossplane react?
- A. It crashes.
- B. During the next reconciliation loop, Crossplane queries the AWS API, detects the drift against the Managed Resource spec, and emits an event or reconciles based on provider policy.
- C. Crossplane deletes the database.
- D. AWS blocks the change.

---

### Question 2: Resolving CRD Storage Version Migration
An operator upgrades an internal CRD from version `v1alpha1` to `v1beta1`. Both versions are served, but existing instances in etcd are stored as `v1alpha1`. What Kubernetes tool or API migrates stored objects to `v1beta1` (`storage: true`)?
- A. `kube-storage-migrator` or `kubectl get <resource> -o json | kubectl replace -f -`.
- B. `terraform init`.
- C. `helm rollback`.
- D. `crictl stop`.

---

### Question 3: Backstage Scaffolder Parameter Validation
How do you ensure in a Backstage Software Template that a developer cannot enter uppercase letters or spaces in an application name?
- A. Using JSON Schema pattern validation: `pattern: "^[a-z0-9-]+$"`.
- B. By validating it in a shell script after creation.
- C. Backstage does not support validation.
- D. By sending a Slack notification.

---

### Question 4: Compositions with Multiple Providers
Can a single Crossplane Composition instantiate Managed Resources across two different providers (e.g. creating an S3 bucket in AWS and a DNS record in Cloudflare)?
- A. No, Compositions can only communicate with a single provider.
- B. Yes; a Composition can include resources spanning any number of installed Crossplane providers.
- C. Only if using Docker Swarm.
- D. Only if both providers run on the same node.

---

### Question 5: Operator Generation Mismatch
Why do production operators check `if instance.Generation != instance.Status.ObservedGeneration` at the start of a reconciliation loop?
- A. To verify if the resource's `spec` has been modified since the controller last reconciled it, avoiding unnecessary actions if only `status` changed.
- B. To check if the node is online.
- C. To measure network latency.
- D. To rotate TLS certificates.

---

### Question 6: Crossplane Package Management
How are Crossplane Providers and Configurations packaged and distributed?
- A. As standard OCI container images stored in standard container registries (e.g. Docker Hub, GitHub Packages, Amazon ECR).
- B. As zip files uploaded via FTP.
- C. As Debian packages.
- D. Through Windows installers.

---

### Question 7: Backstage Catalog Registration via GitOps
How can platform engineers automatically register all microservices in an enterprise GitHub organization into the Backstage Catalog without manual UI clicks?
- A. By configuring the `catalog.providers.github` entity provider in `app-config.yaml` to automatically discover and ingest `catalog-info.yaml` files across all repos matching organizational filters.
- B. By running a nightly bash script with `curl`.
- C. By emailing the Backstage team.
- D. Entity registration cannot be automated.

---

### Question 8: Crossplane Resource Pausing
You need to perform maintenance on an AWS RDS database directly in AWS without Crossplane constantly fighting and reconciling changes. What annotation pauses Crossplane reconciliation?
- A. `crossplane.io/paused: "true"`
- B. `k8s.io/stop: "true"`
- C. `reconciliation: disabled`
- D. `ignore: true`

---

### Question 9: Handling Orphaned Claims
If an engineer runs `kubectl delete composition xdatabase.digitalocean` while 10 live `DatabaseClaim` instances are actively using it:
- A. Crossplane immediately blocks deletion using finalizers until all dependent Composite Resources are migrated or deleted.
- B. All 10 databases are instantly terminated.
- C. etcd is corrupted.
- D. The cluster reboots.

---

### Question 10: Validating Webhook FailurePolicy
In an enterprise admission webhook, why is `failurePolicy: Fail` preferred for security policies while `failurePolicy: Ignore` is preferred for non-critical observability labeling?
- A. `Fail` rejects incoming API requests if the webhook service is offline (enforcing security fail-closed), while `Ignore` permits workloads to deploy even if a non-critical sidecar injector is temporarily down.
- B. `Fail` causes fewer CPU cycles.
- C. `Ignore` is deprecated.
- D. Kubernetes does not support `Fail`.

---

## Answer Key & Explanations

1. **B** — Crossplane's continuous reconciliation loop continuously audits observed cloud state against the declared spec.
2. **A** — Storage version migration ensures stored etcd JSON bytes are updated to the active storage version.
3. **A** — Standard JSON Schema regex patterns validate developer inputs client-side in the Backstage form.
4. **B** — Compositions are multi-provider orchestration engines that bind heterogeneous cloud services into unified APIs.
5. **A** — `metadata.generation` increments on spec changes; comparing against `status.observedGeneration` optimizes controller execution.
6. **A** — Crossplane leverages OCI images for versioned, verifiable package distribution.
7. **A** — Automated entity providers continually scan version control systems to keep the catalog synchronized.
8. **A** — The `crossplane.io/paused: "true"` annotation freezes reconciliation on individual resources.
9. **A** — Finalizers protect active infrastructure topologies from dangling dependency deletions.
10. **A** — `failurePolicy: Fail` guarantees that security admission checks fail-closed.
