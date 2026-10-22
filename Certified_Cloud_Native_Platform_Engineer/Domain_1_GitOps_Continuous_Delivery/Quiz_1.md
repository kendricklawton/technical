# Domain 1: GitOps & Continuous Delivery — Quiz 1

**Exam:** Certified Cloud Native Platform Engineer (CNPE)  
**Format:** 10 Hands-On Scenario & Performance Questions with Detailed Rationales

---

### Question 1: Argo CD Auto-Sync with HPA Conflict
An application uses Horizontal Pod Autoscaler (HPA) to scale its Deployment replicas between 3 and 20 based on CPU utilization. Whenever HPA scales the deployment up to 12 replicas, Argo CD marks the Application as `OutOfSync` and immediately reverts replicas back to `3`. How must the Application manifest be configured to stop this conflict?
- A. By disabling automated sync entirely.
- B. By adding `/spec/replicas` under `spec.ignoreDifferences` for that Deployment resource in the Application manifest.
- C. By deleting the HPA controller.
- D. By increasing CPU requests in the pod template.

---

### Question 2: Cascading Deletion in Argo CD
When an Argo CD `Application` resource is deleted via `kubectl delete app payment-service -n argocd`, the live Kubernetes resources in the `payments` namespace are NOT deleted. What configuration was missing?
- A. The finalizer `resources-finalizer.argocd.argoproj.io`.
- B. `prune: true`.
- C. `cascade: true` flag.
- D. An AppProject binding.

---

### Question 3: Matrix Generator Mechanics
An ApplicationSet uses a Matrix Generator combining a Cluster generator (2 clusters: `us-east` and `eu-west`) with a Git generator (3 directories: `auth`, `billing`, `cart`). How many Argo CD `Application` resources will be dynamically generated?
- A. 3
- B. 5
- C. 6 (2 clusters $\times$ 3 applications)
- D. 1

---

### Question 4: Automated Canary Rollback
In an Argo Rollouts `AnalysisTemplate`, what occurs when Prometheus metrics evaluate `result[0] >= 0.05` (5% error rate) and breach the specified `failureLimit`?
- A. The rollout is paused indefinitely waiting for human intervention.
- B. The rollout is immediately aborted, traffic is instantly reverted 100% back to the stable revision, and the rollout phase transitions to `Degraded`.
- C. The rollout terminates all pods.
- D. The cluster restarts.

---

### Question 5: Flux v2 Controller Responsibilities
Which Flux v2 controller is responsible for cloning a Git repository, verifying PGP/GPG commit signatures, and packaging the workspace artifact into a tarball?
- A. `kustomize-controller`
- B. `source-controller`
- C. `helm-controller`
- D. `notification-controller`

---

### Question 6: Sync Waves Ordering
In Argo CD, how do you ensure that a database migration Job runs and reaches `Complete` before an API Deployment begins updating?
- A. By assigning `argocd.argoproj.io/sync-wave: "1"` to the Job and `argocd.argoproj.io/sync-wave: "2"` to the Deployment, combined with `argocd.argoproj.io/hook: Sync`.
- B. By using `sleep 60` in the deployment entrypoint.
- C. By configuring network policies.
- D. Sync waves do not support Jobs.

---

### Question 7: PrunePropagationPolicy in Argo CD
Why is `syncOptions: ["PrunePropagationPolicy=foreground"]` recommended over `background` when deleting StatefulSets or complex workloads?
- A. It prevents child Pods from continuing to run orphaned while the parent controller is deleted.
- B. It runs 10x faster.
- C. It bypasses RBAC.
- D. It does not use the Kubernetes API.

---

### Question 8: Flux v2 Image Automation
How does Flux v2 achieve automated image promotion without running external CI push pipelines?
- A. The `image-reflector-controller` scans the OCI registry for new semver tags, and the `image-automation-controller` automatically updates the Git repository with a commit containing the new tag.
- B. By polling Docker Hub directly from the kubelet.
- C. Through an API gateway webhook.
- D. By executing `docker pull` on nodes.

---

### Question 9: RespectIgnoreDifferences Setting
When external mutating webhooks (such as Linkerd/Istio sidecar injection or Kyverno) inject annotations and containers into a pod, Argo CD may constantly report drift. What setting must be enabled in `syncOptions`?
- A. `RespectIgnoreDifferences=true`
- B. `IgnoreAll=true`
- C. `BypassWebhooks=true`
- D. `ForceSync=true`

---

### Question 10: Argo CD AppProject Isolation
How do you restrict a development team in Argo CD so they can deploy applications ONLY to namespaces matching `dev-*` on the `staging-cluster`?
- A. In the `AppProject` resource, configure `destinations` specifying `server: "https://staging-k8s..."` and `namespace: "dev-*"`.
- B. By deleting their kubeconfig.
- C. By disabling the Argo CD web UI.
- D. By blocking port 443 with an AWS security group.

---

## Answer Key & Explanations

1. **B** — `ignoreDifferences` prevents Argo CD from fighting external controllers like HPA that mutate the live replica count.
2. **A** — Without the `resources-finalizer`, deleting an Application manifest only removes the tracking record, leaving live resources running.
3. **C** — The Matrix Generator performs a cartesian product calculation ($2 \times 3 = 6$).
4. **B** — Failed canary analysis triggers an automated rollback to protect production users from bad code releases.
5. **B** — `source-controller` is the foundational artifact acquisition engine in Flux v2.
6. **A** — Sync waves execute incrementally; Wave 1 must reach healthy/complete before Wave 2 starts.
7. **A** — Foreground propagation guarantees children are terminated before parent resources disappear.
8. **A** — The Flux image automation controllers implement native write-back GitOps.
9. **A** — `RespectIgnoreDifferences=true` instructs Argo's automated sync engine to honor configured diff exclusions.
10. **A** — `AppProject` defines multi-tenant destination whitelists and allowed source repositories.
