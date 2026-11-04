# Domain 1: GitOps & Continuous Delivery — Quiz 1

**Exam:** Certified Cloud Native Platform Engineer (CNPE)  
**Format:** 10 Hands-On Scenario & Performance Questions with Detailed Rationales

---

### Question 1: Argo CD Auto-Sync with HPA Conflict
An application uses Horizontal Pod Autoscaler (HPA) to scale its Deployment replicas between 3 and 20 based on CPU utilization. Whenever HPA scales the deployment up to 12 replicas, Argo CD marks the Application as `OutOfSync` and immediately reverts replicas back to `3`. How must the Application manifest be configured to stop this conflict?
- A. By deleting the HPA controller.
- B. By disabling automated sync entirely.
- C. By increasing CPU requests in the pod template.
- D. By adding `/spec/replicas` under `spec.ignoreDifferences` for that Deployment resource in the Application manifest.

---

### Question 2: Cascading Deletion in Argo CD
When an Argo CD `Application` resource is deleted via `kubectl delete app payment-service -n argocd`, the live Kubernetes resources in the `payments` namespace are NOT deleted. What configuration was missing?
- A. An AppProject binding.
- B. `cascade: true` flag.
- C. `prune: true`.
- D. The finalizer `resources-finalizer.argocd.argoproj.io`.

---

### Question 3: Matrix Generator Mechanics
An ApplicationSet uses a Matrix Generator combining a Cluster generator (2 clusters: `us-east` and `eu-west`) with a Git generator (3 directories: `auth`, `billing`, `cart`). How many Argo CD `Application` resources will be dynamically generated?
- A. 6 (2 clusters $\times$ 3 applications)
- B. 5
- C. 1
- D. 3

---

### Question 4: Automated Canary Rollback
In an Argo Rollouts `AnalysisTemplate`, what occurs when Prometheus metrics evaluate `result[0] >= 0.05` (5% error rate) and breach the specified `failureLimit`?
- A. The cluster restarts.
- B. The rollout terminates all pods.
- C. The rollout is immediately aborted, traffic is instantly reverted 100% back to the stable revision, and the rollout phase transitions to `Degraded`.
- D. The rollout is paused indefinitely waiting for human intervention.

---

### Question 5: Flux v2 Controller Responsibilities
Which Flux v2 controller is responsible for cloning a Git repository, verifying PGP/GPG commit signatures, and packaging the workspace artifact into a tarball?
- A. `source-controller`
- B. `notification-controller`
- C. `kustomize-controller`
- D. `helm-controller`

---

### Question 6: Sync Waves Ordering
In Argo CD, how do you ensure that a database migration Job runs and reaches `Complete` before an API Deployment begins updating?
- A. By configuring network policies.
- B. Sync waves do not support Jobs.
- C. By assigning `argocd.argoproj.io/sync-wave: "1"` to the Job and `argocd.argoproj.io/sync-wave: "2"` to the Deployment, combined with `argocd.argoproj.io/hook: Sync`.
- D. By using `sleep 60` in the deployment entrypoint.

---

### Question 7: PrunePropagationPolicy in Argo CD
Why is `syncOptions: ["PrunePropagationPolicy=foreground"]` recommended over `background` when deleting StatefulSets or complex workloads?
- A. It does not use the Kubernetes API.
- B. It bypasses RBAC.
- C. It runs 10x faster.
- D. It prevents child Pods from continuing to run orphaned while the parent controller is deleted.

---

### Question 8: Flux v2 Image Automation
How does Flux v2 achieve automated image promotion without running external CI push pipelines?
- A. The `image-reflector-controller` scans the OCI registry for new semver tags, and the `image-automation-controller` automatically updates the Git repository with a commit containing the new tag.
- B. Through an API gateway webhook.
- C. By executing `docker pull` on nodes.
- D. By polling Docker Hub directly from the kubelet.

---

### Question 9: RespectIgnoreDifferences Setting
When external mutating webhooks (such as Linkerd/Istio sidecar injection or Kyverno) inject annotations and containers into a pod, Argo CD may constantly report drift. What setting must be enabled in `syncOptions`?
- A. `BypassWebhooks=true`
- B. `RespectIgnoreDifferences=true`
- C. `IgnoreAll=true`
- D. `ForceSync=true`

---

### Question 10: Argo CD AppProject Isolation
How do you restrict a development team in Argo CD so they can deploy applications ONLY to namespaces matching `dev-*` on the `staging-cluster`?
- A. By disabling the Argo CD web UI.
- B. In the `AppProject` resource, configure `destinations` specifying `server: "https://staging-k8s..."` and `namespace: "dev-*"`.
- C. By deleting their kubeconfig.
- D. By blocking port 443 with an AWS security group.

---

## Answer Key & Explanations

1. **D** — `ignoreDifferences` prevents Argo CD from fighting external controllers like HPA that mutate the live replica count.
2. **D** — Without the `resources-finalizer`, deleting an Application manifest only removes the tracking record, leaving live resources running.
3. **A** — The Matrix Generator performs a cartesian product calculation ($2 \times 3 = 6$).
4. **C** — Failed canary analysis triggers an automated rollback to protect production users from bad code releases.
5. **A** — `source-controller` is the foundational artifact acquisition engine in Flux v2.
6. **C** — Sync waves execute incrementally; Wave 1 must reach healthy/complete before Wave 2 starts.
7. **D** — Foreground propagation guarantees children are terminated before parent resources disappear.
8. **A** — The Flux image automation controllers implement native write-back GitOps.
9. **B** — `RespectIgnoreDifferences=true` instructs Argo's automated sync engine to honor configured diff exclusions.
10. **B** — `AppProject` defines multi-tenant destination whitelists and allowed source repositories.
