# Domain 1: GitOps & Continuous Delivery — Quiz 2

**Exam:** Certified Cloud Native Platform Engineer (CNPE)  
**Format:** 10 Advanced Architecture & Multi-Cluster Scenario Questions

---

### Question 1: Argo CD in Any Namespace (Multi-Tenant Argo CD)
Starting in Argo CD v2.6+, an enterprise wants application teams to manage their own `Application` CRDs inside their dedicated tenant namespaces without granting access to the control plane `argocd` namespace. What feature must be enabled?
- A. Application in any namespace (`application.namespaces` parameter in `argocd-cmd-params-cm`).
- B. Running 50 separate Argo CD server installations.
- C. Granting `cluster-admin` to all developers.
- D. Committing kubeconfigs to Git.

---

### Question 2: Traffic Routing Plugins in Argo Rollouts
Which ingress and service mesh controllers are natively supported by Argo Rollouts for dynamic traffic splitting?
- A. NGINX Ingress Controller, AWS ALB Ingress, Istio, Linkerd, Ambassador, Envoy Gateway, and Kubernetes Gateway API.
- B. Only NGINX Ingress.
- C. Only Istio.
- D. Flannel only.

---

### Question 3: Self-Heal vs Manual Drift Remediation
If an administrator manually changes the image tag of a production deployment using `kubectl set image`, what does Argo CD do if `selfHeal: true` is configured?
- A. It sends an alert and waits for human approval.
- B. The `application-controller` detects the deviation from Git and immediately overrides the live container image back to the Git source of truth.
- C. It deletes the cluster.
- D. It disables Git synchronization.

---

### Question 4: AnalysisTemplate Metric Providers
Which of the following telemetry backends can be queried natively by an Argo Rollouts `AnalysisTemplate`?
- A. Prometheus, Datadog, New Relic, CloudWatch, Wavefront, Kayenta, and generic Webhooks.
- B. Only local text files.
- C. MySQL databases only.
- D. Kubernetes events only.

---

### Question 5: Ephemeral Review Environments with PR Generator
An engineering team wants an automated sandbox environment to spin up whenever a pull request is opened against their microservice repo, and be destroyed when the PR is merged. Which Argo CD component handles this?
- A. ApplicationSet with the Pull Request (`pullRequest`) generator.
- B. A cron job executing `kubectl apply`.
- C. A manual script in Jenkins.
- D. An S3 bucket trigger.

---

### Question 6: Blue/Green Deployment Active vs Preview Service
In an Argo Rollouts Blue/Green strategy:
- A. The `activeService` routes production traffic to the stable revision, while the `previewService` routes internal test traffic to the newly deployed revision before cutover.
- B. Both services route 50% of traffic.
- C. The preview service is exposed to the public internet.
- D. Active service deletes old pods immediately.

---

### Question 7: Sync Windows for Maintenance Windows
An enterprise requires that no automated GitOps deployments occur during Black Friday peak trading hours. How can this be scheduled declaratively in Argo CD?
- A. By shutting down the cluster.
- B. By configuring `syncWindows` inside the `AppProject` specifying `kind: deny`, a cron schedule, and a duration.
- C. By changing Git repository passwords.
- D. By scaling `argocd-server` to 0.

---

### Question 8: Flux v2 HelmRelease Dependency Chaining
In Flux v2, how do you ensure that the `redis-operator` Helm release is successfully installed and ready before the `redis-cluster` Helm release begins installation?
- A. Using the `dependsOn` field in the `HelmRelease` resource referencing `name: redis-operator`.
- B. By using alphabetical naming (`01-redis-op`, `02-redis-cl`).
- C. By running `sleep 120`.
- D. Flux does not support release dependencies.

---

### Question 9: Pre-Delete Hooks for Data Dumps
Before an application is uninstalled by Argo CD, you must take a final database dump. Which annotation attaches this lifecycle behavior?
- A. `argocd.argoproj.io/hook: PreDelete`
- B. `argocd.argoproj.io/hook: PostSync`
- C. `lifecycle.kubernetes.io/stop`
- D. `helm.sh/hook: post-install`

---

### Question 10: GitOps Secret Distribution Pattern
Why should raw base64-encoded Kubernetes Secret YAML files NEVER be committed to a public or private GitOps repository?
- A. Base64 is encoding, not encryption; anyone with read access to Git can decode the plaintext secret instantly.
- B. Git cannot store strings.
- C. Kubernetes rejects Secrets created by Git.
- D. Git repositories limit file sizes to 5 KB.

---

## Answer Key & Explanations

1. **A** — `application.namespaces` allows decentralized application management across namespaces.
2. **A** — Argo Rollouts supports a broad ecosystem of L7 traffic routers including standard Gateway API.
3. **B** — Self-healing actively enforces the Git-declared state against unauthorized live edits.
4. **A** — AnalysisTemplates natively connect to all major observability SaaS and open-source platforms.
5. **A** — The Pull Request generator watches GitHub/GitLab PRs to create ephemeral staging environments.
6. **A** — Blue/Green uses dual services to enable pre-promotion validation before switching production routing.
7. **B** — `syncWindows` enforces time-based freeze windows across projects.
8. **A** — `dependsOn` defines strict readiness dependencies between Flux custom resources.
9. **A** — `PreDelete` hooks run before managed resources are pruned from the cluster.
10. **A** — Plaintext secrets in Git violate core security compliance; External Secrets Operator or Sealed Secrets should be used instead.
