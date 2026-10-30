# Domain 1: GitOps & Continuous Delivery — Quiz 3

**Exam:** Certified Cloud Native Platform Engineer (CNPE)  
**Format:** 10 Performance & Troubleshooting Scenario Questions

---

### Question 1: Argo CD Out of Memory (OOM) on Repo-Server
During a cluster-wide sync involving 400 Helm charts, `argocd-repo-server` crashes repeatedly with exit code 137 (OOMKilled). What configuration changes resolve this bottleneck?
- A. Scale `argocd-repo-server` replicas and increase its container memory request/limit in the deployment.
- B. Delete all Helm charts.
- C. Run `kubectl delete all --all`.
- D. Disable Kubernetes cgroups.

---

### Question 2: Resolving CRD Schema Validation Sync Failures
An Argo CD application sync fails because a Custom Resource Definition (CRD) and its corresponding Custom Resource (CR) are deployed in the same Git commit, but the API server rejects the CR because the CRD has not yet registered. How is this resolved?
- A. Place the CRD in sync wave `0` (`argocd.argoproj.io/sync-wave: "0"`) and the CR in sync wave `1` (`argocd.argoproj.io/sync-wave: "1"`).
- B. Run `terraform init`.
- C. Delete the CRD.
- D. Use `create_before_destroy`.

---

### Question 3: Progressive Delivery Rollout Abort Command
An automated canary deployment is failing in production, but the engineer needs to abort the canary manually before the Prometheus analysis step finishes. Which CLI command immediately aborts the rollout and restores stable traffic?
- A. `kubectl argo rollouts abort <rollout-name> -n <namespace>`
- B. `kubectl delete pod --all`
- C. `kubectl drain node`
- D. `kubectl rollout undo`

---

### Question 4: Setting AnalysisTemplate Consecutive Failures
In an `AnalysisTemplate`, what does `consecutiveErrorLimit: 3` mean?
- A. The analysis fails only if 3 consecutive metric evaluations return an error (such as a timeout connecting to Prometheus), preventing transient network blips from triggering rollbacks.
- B. It retries 3 times after a pod crash.
- C. It allows 3 bad deployments per day.
- D. It restarts Prometheus 3 times.

---

### Question 5: Multi-Cluster Registration in Argo CD
When registering an external workload cluster with Argo CD:
- A. Argo CD creates a ServiceAccount with appropriate RBAC in the remote cluster and stores its kubeconfig credentials as a labeled Secret (`argocd.argoproj.io/secret-type: cluster`) in the `argocd` namespace.
- B. Argo CD installs an entire copy of its server in the remote cluster.
- C. It uses plaintext passwords stored in ConfigMaps.
- D. It requires an SSH tunnel on port 22.

---

### Question 6: GitOps Operator Controller Reconcile Loop Tuning
In a massive enterprise cluster with 10,000 resources, the GitOps controller CPU usage spikes. What architectural tuning is recommended?
- A. Increase the reconcile interval (polling frequency), configure Git webhook triggers to replace aggressive polling, and increase controller worker concurrency flags.
- B. Migrate to manual bash scripts.
- C. Delete the Git repository.
- D. Disable all health checks.

---

### Question 7: Handling Deleted Files in Git (Pruning)
If a developer deletes a Kubernetes Service manifest from Git, but `spec.syncPolicy.automated.prune` is set to `false`, what happens in the live cluster?
- A. The live Service is automatically deleted.
- B. The live Service remains running in the cluster as an orphan, and Argo CD flags it as a pruning candidate in the UI.
- C. The cluster crashes.
- D. Argo CD commits the file back to Git.

---

### Question 8: Flux v2 Kustomization Health Checks
How does Flux v2 determine that a set of deployed resources is healthy before reporting a successful reconciliation?
- A. By specifying `spec.healthChecks` in the `Kustomization` resource pointing to target Deployments, DaemonSets, or Custom Resources.
- B. By pinging the node IP.
- C. By checking if Git is online.
- D. Flux does not check health.

---

### Question 9: GitOps Webhook Triggers
Why are Git webhooks preferred over periodic polling in high-scale GitOps platforms?
- A. Webhooks trigger immediate synchronization within seconds of a commit without subjecting Git providers (GitHub/GitLab) to aggressive API rate limiting.
- B. Polling is blocked by firewalls.
- C. Webhooks encrypt the Git repository.
- D. Polling consumes 100% of network bandwidth.

---

### Question 10: Rollout Pause Duration Invariants
In an Argo Rollout step:
```yaml
steps:
  - setWeight: 20
  - pause: {}
```
What is the effect of `pause: {}` without a `duration` field?
- A. It pauses for 10 minutes.
- B. It pauses indefinitely until a human or external automation issues `kubectl argo rollouts promote <name>`.
- C. It skips the pause immediately.
- D. It aborts the rollout.

---

## Answer Key & Explanations

1. **A** — `argocd-repo-server` is CPU/memory intensive when parsing large volumes of Helm and Kustomize templates.
2. **A** — Sync waves sequence resource creation so CRDs are established before custom resources are instantiated.
3. **A** — `kubectl argo rollouts abort` immediately diverts all traffic back to stable pods.
4. **A** — Consecutive error thresholds prevent false-positive rollbacks caused by temporary metrics outages.
5. **A** — Argo CD uses standard Kubernetes Secrets to store remote cluster API endpoints and credentials.
6. **A** — Webhooks eliminate wasteful polling loops while providing sub-second reaction times.
7. **B** — Without `prune: true`, deleted Git manifests leave orphaned live resources in the cluster.
8. **A** — Flux `healthChecks` inspects workload readiness conditions before marking the release ready.
9. **A** — Webhooks eliminate polling delays and stay well within API rate limits.
10. **B** — An empty pause block represents a manual promotion gate.
