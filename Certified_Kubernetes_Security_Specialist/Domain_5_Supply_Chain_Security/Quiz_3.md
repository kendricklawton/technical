# Domain 5: Supply Chain Security — Quiz 3 (Cosign Image Signing & ImagePolicyWebhook)

**Exam:** Certified Kubernetes Security Specialist (CKS)  
**Format:** 10 Performance Scenario & Multiple Choice Security Questions with Explanations

---

### Question 1: Generating a Key Pair and Signing a Container Image with `cosign`
You need to generate a local key pair (`cosign.key` and `cosign.pub`) and sign container image `registry.local:5000/payments@sha256:9f86d08...` using Sigstore `cosign`. Which command sequence performs this?
- A. `cosign generate-key-pair` followed by `cosign sign --key cosign.key registry.local:5000/payments@sha256:9f86d08...`
- B. `trivy sign --key cosign.key`
- C. `cosign init` followed by `cosign push`
- D. `openssl req -x509` followed by `docker sign`

---

### Question 2: Verifying a Signed Container Image with `cosign verify`
How do you manually verify on a command line that `registry.local:5000/payments@sha256:9f86d08...` has a valid signature created by `cosign.key`?
- A. `kubectl verify image registry.local:5000/payments`
- B. `cosign verify --key cosign.pub registry.local:5000/payments@sha256:9f86d08...`
- C. `crictl verify cosign.pub`
- D. `cosign check --private-key cosign.key registry.local:5000/payments@sha256:9f86d08...`

---

### Question 3: Keyless Signing with Sigstore Fulcio and Rekor
How does Sigstore `cosign` "keyless" signing work in CI/CD pipelines (such as GitHub Actions or GitLab CI) without managing long-lived private keys?
- A. `cosign` obtains a short-lived OIDC identity token from the CI provider, exchanges it with the **Fulcio** Certificate Authority for an ephemeral short-lived X.509 code-signing certificate (valid for ~10 minutes), signs the image digest, records the signature and certificate in the immutable **Rekor** transparency log, and discards the ephemeral private key from memory.
- B. It signs images using a shared hardcoded password.
- C. It stores private keys inside a public Git repository.
- D. It skips cryptographic verification.

---

### Question 4: Enabling the Built-In `ImagePolicyWebhook` Admission Controller
On the CKS exam, you are asked to configure `kube-apiserver` to consult an external image scanner webhook before allowing any Pod to be created. Which admission plugin name must be added to `--enable-admission-plugins` on `kube-apiserver`?
- A. `RegistryWebhook`
- B. `VerifyImageSignature`
- C. `AlwaysPullImages`
- D. `ImagePolicyWebhook`

---

### Question 5: `ImagePolicyWebhook` Configuration Files Structure
When configuring `ImagePolicyWebhook` via `--admission-control-config-file=/etc/kubernetes/epconfig/admission_configuration.yaml`, which three components are linked together?
- A. `/var/lib/kubelet/config.yaml` and `/etc/containerd/config.toml`.
- B. Only a single `ConfigMap` in `kube-system`.
- C. (1) `AdmissionConfiguration` referencing the `ImagePolicyWebhook` plugin configuration file, (2) `imagePolicy:` (`kubeConfigFile`, `allowTTL`, `denyTTL`, `retryBackoff`, `defaultAllow`), and (3) a standard `kubeconfig` file defining the HTTPS `server:` URL of the external webhook backend and mTLS client certificates (`client-certificate`, `client-key`, `certificate-authority`).
- D. `Corefile` and `etcd.yaml`.

---

### Question 6: Fail-Closed Security in `ImagePolicyWebhook` (`defaultAllow: false`)
In the `ImagePolicyWebhook` configuration, what is the critical difference between `defaultAllow: true` and `defaultAllow: false`?
- A. `defaultAllow` controls `NetworkPolicy` default behavior.
- B. `defaultAllow: true` blocks `kube-system` pods.
- C. `defaultAllow: false` enforces **fail-closed** security: if the external image scanning webhook server is unreachable or returns an error, `kube-apiserver` **rejects** all Pod creations; `defaultAllow: true` fails open and allows Pod creations if the webhook fails.
- D. `defaultAllow: false` blocks all images even when the webhook approves them.

---

### Question 7: Mounting the Webhook Config Directory into `kube-apiserver` Static Pod
You created `/etc/kubernetes/epconfig/admission_configuration.yaml`, `imagepolicywebhook_config.yaml`, and `kubeconfig.yaml` on the control-plane host, and added `--enable-admission-plugins=NodeRestriction,ImagePolicyWebhook` and `--admission-control-config-file=/etc/kubernetes/epconfig/admission_configuration.yaml` to `/etc/kubernetes/manifests/kube-apiserver.yaml`. However, `kube-apiserver` crashes on startup with `open /etc/kubernetes/epconfig/admission_configuration.yaml: no such file or directory`. Why?
- A. Because `etcd` also needs the volume mount.
- B. Because `chmod 777` is required.
- C. Because `kube-apiserver` runs inside a container! Any file path referenced inside `admission_configuration.yaml` and `kubeconfig.yaml` must exist **inside the `kube-apiserver` container**, requiring a `hostPath` `volume` and `volumeMount` mounting `/etc/kubernetes/epconfig` into `/etc/kubernetes/epconfig` inside `kube-apiserver.yaml`.
- D. Because `kube-apiserver` cannot read YAML files.

---

### Question 8: Enforcing Image Signature Verification with Kyverno (`verifyImages`) or Sigstore `policy-controller`
How does a Kyverno `verifyImages` rule or Sigstore `ClusterImagePolicy` protect the cluster at admission time when a developer deploys `image: registry.corp.io/app:v1.2`?
- A. It disables TLS on the registry.
- B. It replaces `kube-scheduler`.
- C. It scans the node's hard drive after the Pod terminates.
- D. During the Mutating/Validating admission phase, the webhook queries the OCI registry for the `.sig` Cosign signature artifact associated with `registry.corp.io/app:v1.2`, verifies the cryptographic signature against the configured public key (`cosign.pub`) or Fulcio OIDC identity, blocks the Pod if invalid, and mutates the Pod image reference to its immutable `@sha256:...` digest to prevent Time-of-Check to Time-of-Use (TOCTOU) attacks.

---

### Question 9: Preventing TOCTOU (Time-of-Check to Time-of-Use) Between Admission and Kubelet Pull
Why do image verification admission controllers mutate `image: app:v1` to `image: app@sha256:abcd...` upon admission approval?
- A. To bypass `AlwaysPullImages`.
- B. Without digest mutation, an attacker could push a signed benign image to `:v1` to pass the admission webhook check, and milliseconds later overwrite `:v1` in the registry with an unsigned malicious image before the worker node `kubelet` pulls it.
- C. To reduce network bandwidth.
- D. Because `containerd` cannot pull tags.

---

### Question 10: Restricting Allowed Image Registries via OPA Gatekeeper or CEL
On the CKS exam, a task requires blocking any Pod in namespace `prod` that uses an image from `docker.io` or `gcr.io`, allowing only images starting with `10.96.0.50:5000/`. Which two container arrays in a Pod specification must your policy inspect to prevent bypass?
- A. `spec.containers` AND `spec.initContainers` (plus `spec.ephemeralContainers` for `pods/ephemeralcontainers` subresource updates).
- B. Only `spec.volumes`.
- C. `status.containerStatuses`.
- D. Only `spec.containers[0]`.

---

## Answer Key & Detailed Explanations

1. **A** — `cosign generate-key-pair` creates `cosign.key` and `cosign.pub`; `cosign sign --key cosign.key <image-digest>` signs the image and pushes the `.sig` OCI artifact to the registry.
2. **B** — `cosign verify --key cosign.pub <image>` downloads the signature from the registry and verifies it against the public key.
3. **A** — Sigstore's keyless architecture combines OIDC identity providers, Fulcio (short-lived certs), and Rekor (transparency log).
4. **D** — `ImagePolicyWebhook` is the built-in Kubernetes admission controller for external image policy backends.
5. **C** — `ImagePolicyWebhook` requires an `AdmissionConfiguration` file pointing to the webhook configuration (TTL, `defaultAllow`) and a `kubeconfig` file pointing to the HTTPS webhook endpoint.
6. **C** — CKS exam tasks consistently require `defaultAllow: false` so unverified pods are blocked even if the webhook errors or rejects them.
7. **C** — This is the #1 mistake on `ImagePolicyWebhook` tasks: forgetting to mount the host directory containing the admission config, kubeconfig, and TLS certs into the `kube-apiserver` static pod container.
8. **D** — `verifyImages` in Kyverno and `ClusterImagePolicy` in Sigstore Policy Controller verify Cosign signatures and resolve mutable tags to immutable `@sha256` digests.
9. **B** — Mutating the tag to `@sha256:<digest>` at admission ensures the kubelet pulls the exact artifact whose signature was verified.
10. **A** — Attackers frequently hide malicious images inside `initContainers` or `ephemeralContainers` when admission policies only inspect `spec.containers`.
