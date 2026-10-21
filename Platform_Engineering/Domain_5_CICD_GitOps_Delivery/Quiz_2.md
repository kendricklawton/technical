# PE Domain 5: CI/CD, GitOps & Delivery — Scenario Quiz 2

Test your knowledge on GitOps engines (Argo CD vs Flux), reconciliation loops, and Kustomize overlays.

---

### Question 1
An engineer uses `kubectl edit` to manually scale a production deployment from 5 to 10 replicas in an emergency. The cluster is managed by **Argo CD** with `automated.selfHeal: true`. What happens within 3 minutes of the engineer's manual change?

A) Argo CD sends an email and leaves the 10 replicas running.  
B) Argo CD updates the Git repository automatically.  
C) The Kubernetes cluster shuts down.  
D) Argo CD's reconciliation loop detects that the live cluster state (10 replicas) diverges from the desired state declared in Git (5 replicas), and automatically scales the deployment back down to 5 replicas.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** D  
**Explanation:** Self-healing is a fundamental pillar of GitOps. The Git repository is the absolute source of truth. When manual out-of-band changes occur, Argo CD detects the drift and forcefully overwrites the live state to match Git.
</details>

---

### Question 2
What is the primary architectural difference between **Argo CD** and **Flux v2**?

A) Argo CD does not support Helm.  
B) Flux does not support Git repositories.  
C) Argo CD features a centralized server, rich graphical web UI, and built-in SSO/RBAC; Flux is a collection of decentralized, headless, composable Kubernetes controllers without an official built-in UI, relying on native Kubernetes RBAC.  
D) Argo CD only runs on AWS; Flux only runs on Azure.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** C  
**Explanation:** Argo CD provides a batteries-included platform with an intuitive web dashboard, multi-tenant projects, and audit logs. Flux adopts the Unix philosophy: a set of independent, lightweight controllers (Source Controller, Kustomize Controller, Helm Controller) designed for headless GitOps automation.
</details>

---

### Question 3
An engineering team adopts a "Branch-per-Environment" GitOps strategy (`dev`, `staging`, `prod` branches). Over 6 months, developers frequently merge bugfixes into `prod` without backporting them to `dev`, and cherry-pick commits across branches. What problem does this team inevitably encounter?

A) Severe Git drift, merge conflicts, and un-reproducible environments; configurations in `dev` and `prod` diverge permanently, defeating the core GitOps promise of consistency.  
B) Git repositories cannot hold more than 2 branches.  
C) Argo CD will refuse to sync.  
D) Cloud costs increase by 1000%.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** A  
**Explanation:** Branch-per-environment is a notorious anti-pattern. Branches naturally diverge over time. The industry best practice is **Directory-per-Environment** on a single `main` trunk, using tools like Kustomize or Helm values to clearly visualize differences between environments.
</details>

---

### Question 4
In Kustomize, what is the role of the `base/` directory?

A) It stores database backups.  
B) It contains bash deployment scripts.  
C) It contains the common, environment-agnostic Kubernetes manifests (Deployment, Service, ServiceAccount) that are inherited and patched by environment-specific `overlays/`.  
D) It stores binary executables.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** C  
**Explanation:** Kustomize separates shared structural templates (`base/`) from environment-specific customizations (`overlays/dev/`, `overlays/prod/`), allowing clean inheritance without copy-pasting YAML boilerplate.
</details>

---

### Question 5
What is the **App-of-Apps** pattern in Argo CD?

A) A tool that converts Dockerfiles into Helm charts.  
B) A design pattern where a single "root" Argo CD `Application` points to a repository containing manifests for dozens of other child `Application` custom resources, enabling automated, declarative bootstrapping of entire cluster fleets.  
C) A mobile app for iOS.  
D) A Kubernetes backup script.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** B  
**Explanation:** App-of-Apps solves the bootstrapping challenge: instead of manually executing `kubectl apply` for 40 different microservice applications, you apply a single root application in Argo CD, which automatically provisions and manages all downstream applications.
</details>

---

### Question 6
Why does pull-based GitOps offer superior security compared to push-based CI deployment pipelines?

A) Push-based pipelines cannot deploy containers.  
B) In pull-based GitOps, the CI runner requires zero cloud or cluster credentials; the GitOps controller runs inside the cluster firewall, pulling declarations outward from Git, eliminating the risk of cluster compromise if CI is breached.  
C) Pull-based GitOps runs without Linux permissions.  
D) Pull-based GitOps encrypts all files on disk.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** B  
**Explanation:** In push models, the CI server holds cluster-admin `kubeconfig` credentials. If an attacker breaches CI (or a malicious dependency runs in a build), the entire Kubernetes cluster is compromised. In GitOps, CI only pushes to Git; the cluster agent pulls changes safely from within.
</details>

---

### Question 7
What does `prune: true` accomplish in an Argo CD sync policy?

A) It instructs Argo CD to automatically delete real Kubernetes resources from the cluster when their corresponding manifest files are removed from the Git repository.  
B) It clears the browser cache.  
C) It deletes old Git commits.  
D) It stops running pods.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** A  
**Explanation:** Without `prune: true`, deleting a manifest in Git leaves the orphan resource running in the cluster forever. Pruning ensures the live cluster accurately reflects deletions made in the repository.
</details>

---

### Question 8
What is the function of the **Argo CD Image Updater**?

A) It deletes old Docker images from the host disk.  
B) It writes new Go code.  
C) It converts JPEG images to PNG.  
D) A companion controller that monitors container registries for new image tags and automatically commits updated image tags directly back to the GitOps repository, triggering an automated cluster sync.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** D  
**Explanation:** Argo CD Image Updater closes the loop between CI and CD: when CI pushes `app:v1.2.3` to Harbor or ECR, the updater detects the new tag and pushes a Git commit updating `kustomization.yaml`, allowing Git to remain the audit log for image bumps.
</details>

---

### Question 9
In Helm, what is the role of `Chart.lock`?

A) It records the exact versions and integrity hashes of dependent subcharts listed in `Chart.yaml`, guaranteeing reproducible chart builds.  
B) It encrypts Helm secrets.  
C) It prevents junior engineers from modifying values.  
D) It locks the cluster during deployments.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** A  
**Explanation:** Similar to lock files in package managers (npm, pip), `Chart.lock` pins dependency subchart versions, ensuring that running `helm dependency build` always downloads identical upstream charts.
</details>

---

### Question 10
Why is Kustomize often preferred over Helm for managing internal, proprietary microservices?

A) Kustomize is faster than Go.  
B) Kustomize preserves pure, valid, readable Kubernetes YAML without complex templating logic or syntax errors, making diffs transparent and debugging straightforward.  
C) Helm cannot deploy to Kubernetes 1.28+.  
D) Kustomize requires no installation.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** B  
**Explanation:** Helm templates replace valid YAML with Go template tags (`{{ if ... }}`), making raw manifests un-parseable by standard linters. Kustomize uses pure declarative YAML overlays, making code review and manifest generation clean and intuitive.
</details>
