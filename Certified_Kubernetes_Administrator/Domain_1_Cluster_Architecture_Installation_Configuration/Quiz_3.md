# Domain 1 — Quiz 3: Helm, Kustomize, Custom Resource Definitions (CRDs) & Operators

Test your mastery of the 2025/2026 CKA curriculum topics on Helm package management, Kustomize configuration overlays, CRDs, and the Operator pattern.

---

### Question 1
You are asked to check for existing Helm releases across all namespaces in a cluster. You run `helm list -A` and see only one `deployed` release in `kube-system`. However, when you try to install a release named `payment-gateway` in the `payments` namespace, Helm fails with:
`Error: INSTALLATION FAILED: cannot re-use a name that is still in use`
Which command will reveal why `payment-gateway` is blocking the installation?

A. `helm search repo payment-gateway`  
B. `helm list -A -a` (or `helm list -n payments --all` / `--pending`)  
C. `kubectl get configmaps -n kube-system`  
D. `helm repo update`

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer: B**

**Explanation:**  
By default, `helm list` (even with `-A`) only displays releases with status `deployed` or `failed`. If a previous release is stuck in `pending-install`, `pending-upgrade`, or `uninstalling` state, it is hidden unless you pass `-a` (`--all`) or `--pending`. Once visible, you can clean it up with `helm uninstall payment-gateway -n payments` or `helm rollback`.
</details>

---

### Question 2
Where does **Helm v3** store release state and revision history metadata inside a Kubernetes cluster by default?

A. Inside a `tiller` Pod in the `kube-system` namespace  
B. As Kubernetes `Secrets` (with `type: helm.sh/release.v1`) in the **same namespace** where the Helm release was installed  
C. In `CustomResourceDefinitions` under `helm.sh/v1`  
D. On the local filesystem in `~/.config/helm/releases/`

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer: B**

**Explanation:**  
Helm v3 removed the Tiller server from Helm v2. Instead, Helm v3 stores each release revision as a base64-encoded, gzipped Kubernetes `Secret` named `sh.helm.release.v1.<release-name>.v<revision>` directly inside the target namespace of the release.
</details>

---

### Question 3
A CKA task asks you to render the Kubernetes YAML manifests from a local Helm chart located at `/opt/charts/webapp` using release name `prod-web`, namespace `production`, and custom values file `/opt/charts/prod-values.yaml`, and save the rendered YAML to `/root/webapp-manifests.yaml` **without** installing anything into the cluster. Which command accomplishes this?

A. `helm install prod-web /opt/charts/webapp -n production -f /opt/charts/prod-values.yaml --output yaml > /root/webapp-manifests.yaml`  
B. `helm template prod-web /opt/charts/webapp --namespace production -f /opt/charts/prod-values.yaml > /root/webapp-manifests.yaml`  
C. `helm show all /opt/charts/webapp -f /opt/charts/prod-values.yaml > /root/webapp-manifests.yaml`  
D. `kubectl kustomize /opt/charts/webapp > /root/webapp-manifests.yaml`

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer: B**

**Explanation:**  
`helm template <release-name> <chart> --namespace <ns> -f <values-file>` renders all chart templates locally on the client without contacting the Kubernetes API server or installing the release, writing pure Kubernetes YAML to `stdout`.
</details>

---

### Question 4
You have an existing Helm release `redis-cache` in the `caching` namespace that was originally installed with several custom `--set` flags (`auth.enabled=true`, `replica.replicaCount=3`). You now want to upgrade `redis-cache` to change **only** `image.tag=7.2.5`, while ensuring your previous custom settings (`auth.enabled=true` and `replica.replicaCount=3`) are not reset to the chart defaults. Which flag should you pass to `helm upgrade`?

A. `--keep-history`  
B. `--reuse-values`  
C. `--atomic`  
D. `--force`

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer: B**

**Explanation:**  
`helm upgrade <release> <chart> --reuse-values --set image.tag=7.2.5` instructs Helm to reuse the last release's customized values and merge your new `--set` overrides on top of them.
</details>

---

### Question 5
You are working with a Kustomize directory `/root/app/overlays/staging` that contains a `kustomization.yaml` file. Which pair of `kubectl` commands lets you (1) preview the rendered YAML manifests in the terminal, and (2) apply the Kustomize overlay to the cluster?

A. `kubectl kustomize /root/app/overlays/staging` and `kubectl apply -k /root/app/overlays/staging`  
B. `kubectl apply -f /root/app/overlays/staging --dry-run=client` and `kubectl apply -f /root/app/overlays/staging`  
C. `kubectl render -k /root/app/overlays/staging` and `kubectl create -k /root/app/overlays/staging`  
D. `kustomize init` and `kubectl patch -k /root/app/overlays/staging`

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer: A**

**Explanation:**  
- `kubectl kustomize <dir>` builds the Kustomize target directory and prints the final merged YAML manifests to `stdout`.
- `kubectl apply -k <dir>` builds and applies the Kustomize directory directly to the cluster (note the `-k` flag, NOT `-f`!).
</details>

---

### Question 6
In a `kustomization.yaml` file, why does `configMapGenerator` (and `secretGenerator`) append a 10-character hash suffix (such as `app-config-9m4k2b7f8c`) to the generated `ConfigMap` name by default?

A. To prevent RBAC users from guessing the `ConfigMap` name.  
B. Because Kubernetes does not allow updating `ConfigMaps` in-place.  
C. So that whenever any key/value in the `ConfigMap` changes, a new `ConfigMap` name is generated and injected into referencing `Deployment` Pod templates, automatically triggering a rolling update of the Pods.  
D. To support multiple namespaces inside the same `ConfigMap`.

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer: C**

**Explanation:**  
Normally in Kubernetes, modifying a `ConfigMap` does **not** trigger a rolling restart of a `Deployment` whose Pods consume that `ConfigMap` via environment variables (or `subPath` volume mounts). By appending a content hash to the `ConfigMap` name and automatically updating the `Deployment`'s references, Kustomize ensures that any configuration change causes a `Deployment` `PodTemplateSpec` change and an immediate rolling update.
</details>

---

### Question 7
In Kustomize, what is the primary difference between a **Strategic Merge Patch** and a **JSON 6902 Patch**?

A. Strategic Merge Patches only work on `Services`, while JSON 6902 Patches only work on `Deployments`.  
B. A Strategic Merge Patch looks like a partial Kubernetes YAML manifest that merges fields based on Kubernetes schema merge strategies, whereas a JSON 6902 Patch uses explicit `op` (`add`, `remove`, `replace`), `path` (e.g., `/spec/replicas`), and `value` operations paired with an explicit `target` selector.  
C. JSON 6902 Patches cannot modify lists/arrays.  
D. Strategic Merge Patches require Helm to be installed.

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer: B**

**Explanation:**  
- **Strategic Merge Patch:** Partial YAML document containing `apiVersion`, `kind`, `metadata.name`, and the fields to modify/merge.
- **JSON 6902 Patch:** Explicit RFC 6902 operations (`op: add | remove | replace | move | copy | test`, `path: /spec/...`, `value: ...`) that require a `target:` block (`group`, `version`, `kind`, `name`) in `kustomization.yaml`.
</details>

---

### Question 8
You create a `CustomResourceDefinition` (CRD) with:
- `spec.group: networking.acme.io`
- `spec.names.plural: firewalls`
- `spec.names.singular: firewall`
- `spec.names.kind: Firewall`

What **MUST** the `metadata.name` of the `CustomResourceDefinition` object be for `kube-apiserver` to accept it?

A. ` firewall.networking.acme.io`  
B. `firewalls.networking.acme.io`  
C. `Firewall.networking.acme.io`  
D. Any arbitrary string (e.g., `acme-firewall-crd`)

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer: B**

**Explanation:**  
Kubernetes strictly enforces that a `CustomResourceDefinition`'s `metadata.name` must follow the exact format `<spec.names.plural>.<spec.group>`. Here, that is `firewalls.networking.acme.io`. If it does not match, `kube-apiserver` rejects the CRD creation request.
</details>

---

### Question 9
In a `CustomResourceDefinition` that defines multiple API versions (`v1alpha1`, `v1beta1`, and `v1`) under `spec.versions[]`, what is the rule regarding the `served` and `storage` boolean fields?

A. Only one version can have `served: true`, while multiple versions can have `storage: true`.  
B. Multiple versions can have `served: true`, but **exactly ONE** version must have `storage: true`.  
C. All versions must have both `served: true` and `storage: true`.  
D. Neither `served` nor `storage` is required in `apiextensions.k8s.io/v1`.

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer: B**

**Explanation:**  
- `served: true` enables the REST API endpoint for that version so clients can read/write using that version. Multiple versions can be served simultaneously during migrations.
- `storage: true` designates the single canonical version used when serializing and persisting the object in `etcd`. **Exactly one** version in `spec.versions[]` can have `storage: true`.
</details>

---

### Question 10
What happens when you apply a `CustomResourceDefinition` (CRD) for `BackupJob` to a cluster and then create a `BackupJob` custom resource YAML, **without** deploying a custom controller or Operator?

A. `kube-apiserver` rejects the `BackupJob` resource until a controller registers a webhook.  
B. `kube-apiserver` validates the `BackupJob` against the CRD's `openAPIV3Schema` and stores the object in `etcd`, and `kubectl get backupjobs` works, but no actual backup actions occur because there is no controller reconciliation loop watching the resource.  
C. `kube-controller-manager` automatically runs a generic backup job.  
D. The CRD remains in `Pending` state.

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer: B**

**Explanation:**  
A **CRD** only defines the API schema and storage endpoint in `kube-apiserver` / `etcd`. To make declarative custom resources actually perform work in the real world, you pair the CRD with a **Custom Controller** (together forming the **Operator Pattern**) that watches the custom resource and reconciles the desired state.
</details>

---

### Question 11
On the CKA exam, you encounter an unfamiliar Custom Resource `Certificate` (from `cert-manager.io/v1`) and need to find out the exact field structure under `spec.issuerRef` without leaving the terminal. Which command gives you the exact schema definition directly from the cluster's API server?

A. `kubectl describe crd certificates`  
B. `kubectl explain certificate.spec.issuerRef` (or `kubectl explain certificate.spec --recursive`)  
C. `kubectl api-versions | grep cert-manager`  
D. `helm show crds cert-manager`

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer: B**

**Explanation:**  
Because `apiextensions.k8s.io/v1` CRDs require an `openAPIV3Schema`, `kubectl explain <custom-resource>.spec --recursive` works on Custom Resources (`Gateway`, `HTTPRoute`, `Certificate`, `Prometheus`, etc.) just like built-in resources!
</details>

---

### Question 12
You delete a `CustomResourceDefinition` named `widgets.store.example.com` using `kubectl delete crd widgets.store.example.com`. What happens to the 25 existing `Widget` custom resources that were created across various namespaces?

A. They remain in `etcd` as orphaned objects.  
B. `kube-apiserver` blocks the CRD deletion until you manually delete each `Widget` resource.  
C. Kubernetes automatically deletes all existing `Widget` custom resources across all namespaces and removes the `/apis/store.example.com/...` REST endpoint.  
D. They are converted into `ConfigMaps`.

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer: C**

**Explanation:**  
Deleting a CRD is a destructive operation: the CustomResourceDefinition cleanup controller automatically garbage-collects and permanently deletes **every** Custom Resource instance of that CRD across the entire cluster (subject to any finalizers on those custom resources).
</details>
