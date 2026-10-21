# CKS Domain 1: Cluster Setup — Quiz 3 (TLS Ingress, Binary Verification & Cilium L7)

Test your understanding of TLS Ingress configuration, X.509 certificate Secrets, platform binary verification (`sha512sum`), and Cilium L7 policies.

---

### Question 1
You have generated a TLS certificate (`tls.crt`) and private key (`tls.key`) for `api.secure.cks`. Which `kubectl` command creates the proper Secret type required by an `Ingress` resource's `spec.tls.secretName` field?

- A) `kubectl create secret generic api-tls --from-file=tls.crt --from-file=tls.key`
- B) `kubectl create secret tls api-tls --cert=tls.crt --key=tls.key`
- C) `kubectl create secret docker-registry api-tls --cert=tls.crt --key=tls.key`
- D) `kubectl create configmap api-tls --from-file=tls.crt --from-file=tls.key`

<details>
<summary>Reveal Answer & Explanation</summary>

**Correct Answer: B**

**Explanation:**
`kubectl create secret tls <name> --cert=<path/to/cert> --key=<path/to/key>` creates a Secret of type `kubernetes.io/tls` containing the mandatory data keys `tls.crt` and `tls.key`. (While a `generic` secret with `--from-file=tls.crt=... --from-file=tls.key=...` has the same keys, its type defaults to `Opaque` rather than `kubernetes.io/tls`).
</details>

---

### Question 2
An `Ingress` resource named `checkout-ing` is deployed in the `storefront` namespace. Where must its referenced TLS Secret `checkout-tls-secret` be created?

- A) In the `ingress-nginx` namespace where the Ingress Controller Pods run.
- B) In the `kube-system` namespace.
- C) In the `storefront` namespace (the exact same namespace as the `Ingress` resource).
- D) In the `default` namespace.

<details>
<summary>Reveal Answer & Explanation</summary>

**Correct Answer: C**

**Explanation:**
In Kubernetes `networking.k8s.io/v1` `Ingress`, `spec.tls[].secretName` is a local object reference within the **same namespace** as the `Ingress` resource (`storefront`). Cross-namespace Secret references are not permitted in standard `Ingress` specs for security reasons (preventing one namespace from hijacking another namespace's TLS private key).
</details>

---

### Question 3
Which NGINX Ingress Controller annotation ensures that any client making an unencrypted HTTP request on port `80` is automatically redirected to HTTPS on port `443`, even if TLS is terminated upstream or `X-Forwarded-Proto` is manipulated?

- A) `nginx.ingress.kubernetes.io/backend-protocol: "HTTPS"`
- B) `nginx.ingress.kubernetes.io/force-ssl-redirect: "true"`
- C) `nginx.ingress.kubernetes.io/ssl-passthrough: "true"`
- D) `nginx.ingress.kubernetes.io/rewrite-target: "https://$host$request_uri"`

<details>
<summary>Reveal Answer & Explanation</summary>

**Correct Answer: B**

**Explanation:**
`nginx.ingress.kubernetes.io/ssl-redirect: "true"` redirects HTTP to HTTPS when the Ingress has a `spec.tls` section, and `nginx.ingress.kubernetes.io/force-ssl-redirect: "true"` forces the HTTP-to-HTTPS redirect unconditionally. Note that `backend-protocol: "HTTPS"` (Choice A) controls whether NGINX uses HTTPS when speaking to the upstream Pod, not client redirection.
</details>

---

### Question 4
During a CKS task, you are asked to verify whether `/usr/bin/kubectl` on `node01` matches the official Kubernetes v1.31.0 release binary. You download `kubectl.sha512` from `https://dl.k8s.io/release/v1.31.0/bin/linux/amd64/kubectl.sha512`, which contains only a 128-character hexadecimal string without a filename. Which command verifies the binary against `kubectl.sha512`?

- A) `sha512sum -c kubectl.sha512 /usr/bin/kubectl`
- B) `echo "$(cat kubectl.sha512)  /usr/bin/kubectl" | sha512sum --check`
- C) `md5sum /usr/bin/kubectl | diff - kubectl.sha512`
- D) `kubectl version --verify-checksum=kubectl.sha512`

<details>
<summary>Reveal Answer & Explanation</summary>

**Correct Answer: B**

**Explanation:**
The `.sha512` files hosted on `dl.k8s.io` contain only the raw SHA-512 hash without the `<hash>  <filename>` format expected by `sha512sum --check`. Either compare `cat kubectl.sha512` and `sha512sum /usr/bin/kubectl` visually, or format it as `echo "$(cat kubectl.sha512)  /usr/bin/kubectl" | sha512sum --check`.
</details>

---

### Question 5
You suspect that the `kube-apiserver` binary inside the running `kube-apiserver-controlplane` container has been replaced by a rootkit. Because the container uses a minimal `distroless` image with no shell (`sh` or `sha512sum` does not exist inside the container), how can you compute the SHA-512 hash of the `kube-apiserver` binary directly from the host OS?

- A) Run `kubectl exec -n kube-system kube-apiserver-controlplane -- sha512sum /usr/local/bin/kube-apiserver`.
- B) Find the container's host PID via `PID=$(crictl inspect $(crictl ps --name kube-apiserver -q) | jq .info.pid)` and run `sha512sum /proc/$PID/root/usr/local/bin/kube-apiserver` on the host.
- C) Run `sha512sum /etc/kubernetes/manifests/kube-apiserver.yaml`.
- D) It is impossible to hash a binary inside a `distroless` container without rebuilding the image.

<details>
<summary>Reveal Answer & Explanation</summary>

**Correct Answer: B**

**Explanation:**
Every Linux process exposes its mount namespace root filesystem on the host at `/proc/<PID>/root/`. By getting the container's host PID with `crictl inspect <container-id> | jq .info.pid`, you can use the host's own `/usr/bin/sha512sum` binary to hash `/proc/$PID/root/usr/local/bin/kube-apiserver` without executing anything inside the container!
</details>

---

### Question 6
You want to generate a new 2048-bit RSA private key (`server.key`) and a self-signed certificate (`server.crt`) valid for 365 days for `CN=internal.cks.io` without being prompted for a passphrase. Which `openssl` command is correct?

- A) `openssl genrsa -out server.key 2048 && openssl x509 -req -in server.key -out server.crt`
- B) `openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout server.key -out server.crt -subj "/CN=internal.cks.io"`
- C) `openssl req -new -days 365 -keyout server.key -out server.crt -cn "internal.cks.io"`
- D) `openssl tls -create -days 365 -key server.key -cert server.crt -subj "CN=internal.cks.io"`

<details>
<summary>Reveal Answer & Explanation</summary>

**Correct Answer: B**

**Explanation:**
* `-x509` outputs a self-signed certificate instead of a Certificate Signing Request (CSR).
* `-nodes` ("no DES") skips encrypting the private key with a passphrase (required so Kubernetes Ingress controllers can read `tls.key` non-interactively).
* `-days 365` sets the validity period.
* `-newkey rsa:2048` generates a new 2048-bit RSA key simultaneously.
* `-subj "/CN=internal.cks.io"` sets the subject non-interactively.
</details>

---

### Question 7
In a `CiliumNetworkPolicy`, what is the difference between `toFQDNs[].matchName` and `toFQDNs[].matchPattern`?

- A) `matchName` matches exact domain names (e.g., `api.example.com`), whereas `matchPattern` supports wildcard matching (e.g., `*.example.com`).
- B) `matchName` matches Kubernetes Service names, whereas `matchPattern` matches external DNS names.
- C) `matchName` works at Layer 3, whereas `matchPattern` works at Layer 7.
- D) There is no difference; they are aliases.

<details>
<summary>Reveal Answer & Explanation</summary>

**Correct Answer: A**

**Explanation:**
In Cilium's `toFQDNs` rule, `matchName` requires an exact literal match of the queried FQDN, while `matchPattern` allows `*` wildcard patterns such as `*.trusted.org`.
</details>

---

### Question 8
Which command allows you to test an HTTPS Ingress resource configured for host `app.cks.internal` against the Ingress Controller's IP `10.105.22.40` when `app.cks.internal` is not in `/etc/hosts` or DNS?

- A) `curl -k https://10.105.22.40/ -H "Host: app.cks.internal"`
- B) `curl -kv --resolve app.cks.internal:443:10.105.22.40 https://app.cks.internal/`
- C) `curl --dns-servers 10.105.22.40 https://app.cks.internal/`
- D) `wget --no-check-certificate https://app.cks.internal:10.105.22.40/`

<details>
<summary>Reveal Answer & Explanation</summary>

**Correct Answer: B**

**Explanation:**
Why is Choice B better than Choice A for TLS Ingress? While `-H "Host: app.cks.internal"` sets the HTTP `Host` header *after* the TLS handshake, `curl https://10.105.22.40` does **not** send the TLS **SNI (Server Name Indication)** extension for `app.cks.internal` during the TLS ClientHello! Using `--resolve app.cks.internal:443:10.105.22.40` makes `curl` send both the proper TLS SNI hostname and the HTTP `Host` header, ensuring the Ingress Controller serves the `app.cks.internal` TLS certificate rather than the default fake certificate.
</details>

---

### Question 9
You inspect a Kubernetes Dashboard Deployment in the `kubernetes-dashboard` namespace and find the container args include `--enable-skip-login` and `--insecure-port=9090`, and a `ClusterRoleBinding` binds the `kubernetes-dashboard` ServiceAccount to `cluster-admin`. How should this be remediated?

- A) Keep `--enable-skip-login` but change the Service type from `NodePort` to `ClusterIP`.
- B) Remove `--enable-skip-login` and `--insecure-port=9090`, ensure `--auto-generate-certificates` (HTTPS) is used, and delete the `cluster-admin` `ClusterRoleBinding` so users must authenticate with least-privilege credentials.
- C) Add a `NetworkPolicy` that allows all Pods to access port `9090`.
- D) Set `privileged: true` on the Dashboard container.

<details>
<summary>Reveal Answer & Explanation</summary>

**Correct Answer: B**

**Explanation:**
Passing `--enable-skip-login` allows anyone who reaches the Dashboard UI to click "Skip" and immediately assume the permissions of the Dashboard's ServiceAccount. When combined with a `cluster-admin` binding, any user gets full root control of the cluster without logging in! Removing `--enable-skip-login`, serving only HTTPS, and removing the `cluster-admin` binding remedies this critical vulnerability.
</details>

---

### Question 10
In a `CiliumNetworkPolicy`, which section is used to restrict a Pod from sending or receiving **ICMP** (ping) packets?

- A) `icmps:`
- B) `l3Protocols:`
- C) `pingPolicy:`
- D) Cilium cannot filter ICMP packets.

<details>
<summary>Reveal Answer & Explanation</summary>

**Correct Answer: A**

**Explanation:**
`CiliumNetworkPolicy` supports an `icmps:` block inside `ingress` / `egress` rules (specifying `family: IPv4` or `IPv6` and `type: 8` / `"EchoRequest"`) to explicitly control ICMP traffic.
</details>
