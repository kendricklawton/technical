# CKS Domain 1: Cluster Setup — Hands-On Security Labs

These 6 realistic, performance-based labs simulate the exact scenarios tested in **Domain 1 (Cluster Setup - 10%)** of the Certified Kubernetes Security Specialist (CKS) exam on your **DigitalOcean Droplet** cluster (`cks-cp1`, `cks-worker1`, `cks-worker2`).

---

## Lab 1: Multi-Tier Zero-Trust NetworkPolicies & CoreDNS Egress

### Scenario
Your organization runs a three-tier payment processing application in the `payments` namespace consisting of:
* `frontend` Pods (`app: frontend`) listening on TCP port `8080`
* `processor` Pods (`app: processor`) listening on TCP port `8443`
* `ledger-db` Pods (`app: ledger-db`) listening on TCP port `5432`

Additionally, an `ingress-nginx` controller runs in the `ingress-nginx` namespace with Pods labeled `app.kubernetes.io/name: ingress-nginx`.

Currently, no NetworkPolicies exist in the `payments` namespace. Implement a strict **Zero-Trust** network architecture in `payments`:
1. Create a default-deny policy named `default-deny-all` in `payments` that blocks **all** Ingress and Egress traffic for all Pods in the namespace.
2. Create a policy named `allow-dns-egress` in `payments` that allows **all** Pods in `payments` to send DNS queries (`UDP` and `TCP` port `53`) only to Pods labeled `k8s-app: kube-dns` inside the `kube-system` namespace.
3. Create a policy named `processor-netpol` in `payments` for `app: processor` Pods that:
   * Allows **Ingress** on TCP port `8443` **only** from `app: frontend` Pods in the `payments` namespace.
   * Allows **Egress** on TCP port `5432` **only** to `app: ledger-db` Pods in the `payments` namespace.

### Step-by-Step Solution

#### Step 1: Inspect Namespace and Pod Labels
Always verify the exact labels on namespaces and CoreDNS pods before writing policies:
```bash
kubectl get ns payments ingress-nginx kube-system --show-labels
kubectl get pods -n payments --show-labels
kubectl get pods -n kube-system -l k8s-app=kube-dns --show-labels
```

#### Step 2: Create the Zero-Trust Manifests
Create `/root/payments-netpol.yaml`:
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
  namespace: payments
spec:
  podSelector: {}
  policyTypes:
    - Ingress
    - Egress
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-dns-egress
  namespace: payments
spec:
  podSelector: {}
  policyTypes:
    - Egress
  egress:
    - to:
        - namespaceSelector:
            matchLabels:
              kubernetes.io/metadata.name: kube-system
          podSelector:
            matchLabels:
              k8s-app: kube-dns
      ports:
        - protocol: UDP
          port: 53
        - protocol: TCP
          port: 53
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: processor-netpol
  namespace: payments
spec:
  podSelector:
    matchLabels:
      app: processor
  policyTypes:
    - Ingress
    - Egress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: frontend
      ports:
        - protocol: TCP
          port: 8443
  egress:
    - to:
        - podSelector:
            matchLabels:
              app: ledger-db
      ports:
        - protocol: TCP
          port: 5432
```

Apply and verify:
```bash
kubectl apply -f /root/payments-netpol.yaml
kubectl describe netpol -n payments processor-netpol
```

#### Step 3: Verification Commands
```bash
# 1. Verify DNS resolution still works from processor pod (permitted by allow-dns-egress)
kubectl -n payments exec deploy/processor -- nslookup kubernetes.default.svc.cluster.local

# 2. Verify frontend -> processor:8443 succeeds (once frontend egress is also allowed)
kubectl -n payments exec deploy/frontend -- nc -zv processor-svc 8443

# 3. Verify frontend -> ledger-db:5432 is BLOCKED (timeout)
kubectl -n payments exec deploy/frontend -- nc -zv -w 2 ledger-db-svc 5432
```

---

## Lab 2: Protecting DigitalOcean Droplet & Cloud Metadata Server Endpoints (`169.254.169.254/32`)

### Scenario
In the `public-workers` namespace, untrusted tenant workloads run on DigitalOcean Droplets that expose the DigitalOcean Metadata API at `http://169.254.169.254/metadata/v1.json` (and `/metadata/v1/user-data`).
1. Create a `NetworkPolicy` named `restrict-metadata-server` in the `public-workers` namespace.
2. It must apply to all Pods labeled `trust-level: untrusted`.
3. Those Pods must be allowed to make outbound connections to any IP address (`0.0.0.0/0`), **except** the DigitalOcean / cloud metadata server IP `169.254.169.254/32` and the internal management subnet `10.96.0.0/12`.

### Step-by-Step Solution

Create `/root/metadata-netpol.yaml`:
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: restrict-metadata-server
  namespace: public-workers
spec:
  podSelector:
    matchLabels:
      trust-level: untrusted
  policyTypes:
    - Egress
  egress:
    - to:
        - ipBlock:
            cidr: 0.0.0.0/0
            except:
              - 169.254.169.254/32
              - 10.96.0.0/12
```

Apply and verify on your DigitalOcean Droplet:
```bash
kubectl apply -f /root/metadata-netpol.yaml
kubectl get netpol -n public-workers restrict-metadata-server -o yaml

# Test from an untrusted pod on your DigitalOcean Droplet: connection to 169.254.169.254 must time out!
kubectl -n public-workers exec pod/worker-1 -- curl -m 2 http://169.254.169.254/metadata/v1.json
```

---

## Lab 3: CIS Kubernetes Benchmark Remediation with `kube-bench`

### Scenario
Run `kube-bench` on `cks-cp1` (`controlplane`) and `cks-worker1` (`node01`) and remediate the following CIS Benchmark security findings:
1. **Control Plane (`kube-apiserver`)**:
   * Disable anonymous authentication.
   * Ensure authorization mode includes `Node,RBAC` and does **not** include `AlwaysAllow`.
   * Enable the `NodeRestriction` admission plugin.
   * Disable profiling.
2. **Control Plane (`etcd`)**:
   * Ensure client certificate authentication is enabled (`--client-cert-auth=true`).
   * Ensure the `/var/lib/etcd` data directory has permissions `700` and is owned by `etcd:etcd`.
3. **Worker Node (`cks-worker1` / `node01` `kubelet`)**:
   * Disable anonymous authentication on the kubelet.
   * Change kubelet authorization mode from `AlwaysAllow` to `Webhook`.
   * Ensure the read-only port (`10255`) is disabled (`readOnlyPort: 0`).

### Step-by-Step Solution

#### Step 1: Audit with `kube-bench` and Backup Static Pod Manifests
```bash
mkdir -p /root/backups
cp /etc/kubernetes/manifests/kube-apiserver.yaml /root/backups/
cp /etc/kubernetes/manifests/etcd.yaml /root/backups/

kube-bench run --targets master,etcd
```

#### Step 2: Remediate `/etc/kubernetes/manifests/kube-apiserver.yaml`
Edit `/etc/kubernetes/manifests/kube-apiserver.yaml` and update/add these flags under `spec.containers[0].command`:
```yaml
    - command:
        - kube-apiserver
        - --anonymous-auth=false
        - --authorization-mode=Node,RBAC
        - --enable-admission-plugins=NodeRestriction
        - --profiling=false
```

#### Step 3: Remediate `/etc/kubernetes/manifests/etcd.yaml` and `/var/lib/etcd`
Verify `--client-cert-auth=true` in `/etc/kubernetes/manifests/etcd.yaml`, then fix directory ownership and permissions:
```bash
chmod 700 /var/lib/etcd
chown -R etcd:etcd /var/lib/etcd
stat -c "%a %U:%G %n" /var/lib/etcd
```

#### Step 4: Remediate `kubelet` on `cks-worker1` (`node01`)
SSH to `cks-worker1` (`10.116.0.11`) and edit `/var/lib/kubelet/config.yaml`:
```bash
ssh cks-worker1
cp /var/lib/kubelet/config.yaml /root/kubelet-config.yaml.bak
vi /var/lib/kubelet/config.yaml
```
Ensure the following fields are set in `/var/lib/kubelet/config.yaml`:
```yaml
authentication:
  anonymous:
    enabled: false
  webhook:
    enabled: true
authorization:
  mode: Webhook
readOnlyPort: 0
```
Restart `kubelet` and verify:
```bash
systemctl daemon-reload
systemctl restart kubelet
systemctl is-active kubelet
kube-bench run --targets node --check 4.2.1,4.2.2,4.2.4
```

---

## Lab 4: TLS Ingress with HTTPS Redirect & Certificate Creation

### Scenario
A microservice named `vault-ui-svc` runs in the `secops` namespace on port `8200`. Expose it securely via an NGINX Ingress resource:
1. Generate a 2048-bit RSA private key (`/root/vault-ui.key`) and self-signed certificate (`/root/vault-ui.crt`) valid for 365 days with Common Name (`CN`) and Subject Alternative Name (`DNS`) set to `vault.secops.local`.
2. Create a `kubernetes.io/tls` Secret named `vault-tls-cert` in the `secops` namespace using those files.
3. Create an `Ingress` resource named `vault-ingress` in `secops` with `ingressClassName: nginx` that routes traffic for host `vault.secops.local` (`pathType: Prefix`, path `/`) to `vault-ui-svc:8200` using TLS Secret `vault-tls-cert`, and automatically redirects all HTTP requests to HTTPS.

### Step-by-Step Solution

```bash
# 1. Generate the TLS private key and certificate
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout /root/vault-ui.key \
  -out /root/vault-ui.crt \
  -subj "/CN=vault.secops.local" \
  -addext "subjectAltName = DNS:vault.secops.local"

# 2. Create the TLS Secret in 'secops'
kubectl -n secops create secret tls vault-tls-cert \
  --cert=/root/vault-ui.crt \
  --key=/root/vault-ui.key
```

Create `/root/vault-ingress.yaml`:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: vault-ingress
  namespace: secops
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
spec:
  ingressClassName: nginx
  tls:
    - hosts:
        - vault.secops.local
      secretName: vault-tls-cert
  rules:
    - host: vault.secops.local
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: vault-ui-svc
                port:
                  number: 8200
```

Apply and verify:
```bash
kubectl apply -f /root/vault-ingress.yaml
kubectl -n secops describe ingress vault-ingress
```

---

## Lab 5: Verifying Kubernetes Platform Binaries via `sha512sum`

### Scenario
A security alert indicates that one or more Kubernetes binaries staged in `/opt/candidate/k8s-v1.31.0/` (`kubectl`, `kubeadm`, `kubelet`, `kube-apiserver`) may have been tampered with before node provisioning.
1. Compare the SHA-512 checksum of each binary in `/opt/candidate/k8s-v1.31.0/` against the official reference file `/opt/candidate/official-sha512.txt`.
2. Write the names of any modified/corrupted binaries (one per line, filename only) to `/opt/candidate/tampered-binaries.txt` and delete the tampered binary files from `/opt/candidate/k8s-v1.31.0/`.

### Step-by-Step Solution

```bash
# 1. View the official reference SHA-512 hashes
cat /opt/candidate/official-sha512.txt

# 2. Compute actual SHA-512 hashes of all binaries in /opt/candidate/k8s-v1.31.0/
sha512sum /opt/candidate/k8s-v1.31.0/*

# 3. Run automated verification if official-sha512.txt is in standard sha512sum format
cd /opt/candidate/k8s-v1.31.0/
sha512sum --check /opt/candidate/official-sha512.txt

# 4. Suppose 'kubelet' and 'kubeadm' report FAILED:
cat <<EOF > /opt/candidate/tampered-binaries.txt
kubeadm
kubelet
EOF

# 5. Remove the tampered binaries
rm -f /opt/candidate/k8s-v1.31.0/kubeadm /opt/candidate/k8s-v1.31.0/kubelet
```

---

## Lab 6: Cilium Layer-7 HTTP & FQDN Egress Policy (`CiliumNetworkPolicy`)

### Scenario
Your cluster uses Cilium as its CNI on DigitalOcean VPC interface `eth1`. In the `analytics` namespace, Pods labeled `app: webhook-sender` need to:
1. Resolve DNS names via `kube-dns` in `kube-system`.
2. Make outbound HTTPS (`TCP 443`) requests **only** to `hooks.slack.com` and any subdomain matching `*.datadoghq.com`.
3. Receive inbound HTTP (`TCP 8080`) requests from Pods labeled `app: aggregator` in the `analytics` namespace, **only** for `POST /api/v2/events` when HTTP header `X-Api-Version: 2` is present.

Create a `CiliumNetworkPolicy` named `webhook-l7-policy` in the `analytics` namespace implementing these rules.

### Step-by-Step Solution

Create `/root/webhook-l7-policy.yaml`:
```yaml
apiVersion: "cilium.io/v2"
kind: CiliumNetworkPolicy
metadata:
  name: webhook-l7-policy
  namespace: analytics
spec:
  endpointSelector:
    matchLabels:
      app: webhook-sender
  ingress:
    - fromEndpoints:
        - matchLabels:
            app: aggregator
      toPorts:
        - ports:
            - port: "8080"
              protocol: TCP
          rules:
            http:
              - method: "POST"
                path: "/api/v2/events"
                headers:
                  - "X-Api-Version: 2"
  egress:
    - toEndpoints:
        - matchLabels:
            "k8s:io.kubernetes.pod.namespace": kube-system
            "k8s:k8s-app": kube-dns
      toPorts:
        - ports:
            - port: "53"
              protocol: ANY
          rules:
            dns:
              - matchPattern: "*"
    - toFQDNs:
        - matchName: "hooks.slack.com"
        - matchPattern: "*.datadoghq.com"
      toPorts:
        - ports:
            - port: "443"
              protocol: TCP
```

Apply and verify:
```bash
kubectl apply -f /root/webhook-l7-policy.yaml
kubectl get cnp -n analytics webhook-l7-policy -o yaml
```
