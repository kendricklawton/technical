# Domain 3: Services & Networking — Quiz 3 (CoreDNS & NetworkPolicies)

**Exam:** Certified Kubernetes Administrator (CKA)  
**Format:** 10 Performance Scenario & Multiple Choice Questions with Detailed Explanations

---

### Question 1: NetworkPolicy AND vs OR YAML List Semantics
An administrator writes a NetworkPolicy `ingress` rule with `- from:` containing two separate YAML list items (`-`): one `namespaceSelector` and one `podSelector`:
```yaml
ingress:
- from:
  - namespaceSelector:
      matchLabels:
        env: prod
  - podSelector:
      matchLabels:
        role: frontend
```
How does Kubernetes interpret this rule?
- A. Logical AND: Traffic is allowed only from pods matching `role: frontend` inside namespaces matching `env: prod`.
- B. Logical OR: Traffic is allowed from ANY pod in a namespace labeled `env: prod` OR from any pod in the NetworkPolicy's own namespace labeled `role: frontend`.
- C. The policy is syntactically invalid.
- D. Kubernetes denies all traffic.

---

### Question 2: Writing a Logical AND Selector in NetworkPolicy
How do you modify the YAML in Question 1 so traffic is allowed **only** from pods labeled `role: frontend` that reside inside namespaces labeled `env: prod`?
- A. Remove the second dash (`-`) before `podSelector` so both `namespaceSelector` and `podSelector` are sibling keys inside the exact same `from` list element:
  ```yaml
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          env: prod
      podSelector:
        matchLabels:
          role: frontend
  ```
- B. Add `operator: AND` to the NetworkPolicy spec.
- C. Create two separate NetworkPolicies.
- D. Use `ipBlock` instead.

---

### Question 3: Default Deny All Ingress AND Egress Traffic
You need to lock down namespace `pci-dss` so no pod can receive any inbound traffic or initiate any outbound traffic unless explicitly allowed by another NetworkPolicy. What does the Default Deny All NetworkPolicy spec look like?
- A. `podSelector: {}` and `policyTypes: ["Ingress", "Egress"]` with no `ingress:` or `egress:` rules defined.
- B. `podSelector: { matchLabels: { all: "true" } }` and `policyTypes: ["None"]`.
- C. `ingress: [{}]` and `egress: [{}]`.
- D. Delete `kube-proxy` on the worker nodes.

---

### Question 4: Allowing CoreDNS Egress When `policyTypes: ["Egress"]` Is Enabled
After applying a Default Deny Egress NetworkPolicy to namespace `backend`, you add an egress rule allowing TCP port `5432` to `postgres`. However, pods in `backend` now fail to connect to `postgres.backend.svc.cluster.local` with `Temporary failure in name resolution`. Why?
- A. Enabling `Egress` isolation blocks outbound DNS queries to CoreDNS unless you explicitly add an egress rule permitting UDP and TCP port `53` (to `kube-system` / CoreDNS pods).
- B. `postgres` uses UDP port `5432`.
- C. NetworkPolicies block `localhost` traffic.
- D. CoreDNS only supports IPv6.

---

### Question 5: Targeting a Specific Namespace by Name (`kubernetes.io/metadata.name`)
In Kubernetes 1.21+, every namespace automatically receives an immutable system label containing its namespace name. What is the key of this label, which makes writing `namespaceSelector` rules in NetworkPolicies straightforward without manually labeling namespaces?
- A. `kubernetes.io/metadata.name`
- B. `name`
- C. `k8s.io/namespace`
- D. `metadata.namespace`

---

### Question 6: `ipBlock` with `except` CIDR Ranges
A NetworkPolicy needs to allow egress traffic to any external IP in `172.16.0.0/12` **except** the restricted management subnet `172.16.100.0/24`. Which `from`/`to` peer syntax achieves this?
- A. `ipBlock: { cidr: 172.16.0.0/12, except: ["172.16.100.0/24"] }`
- B. `cidrSelector: { allow: 172.16.0.0/12, deny: 172.16.100.0/24 }`
- C. `podSelector: { ip: "!172.16.100.0/24" }`
- D. NetworkPolicy cannot filter by IP CIDR blocks.

---

### Question 7: Kubernetes FQDN Format for Services and Pods
A Pod in namespace `web` wants to send an HTTP request to a Service named `orders` in namespace `commerce` on a cluster configured with the default cluster domain `cluster.local`. What is the complete FQDN of the Service, and what would the Pod IP DNS record look like for a Pod with IP `10.244.2.15` in `commerce`?
- A. Service FQDN: `orders.commerce.svc.cluster.local`; Pod DNS record: `10-244-2-15.commerce.pod.cluster.local`
- B. Service FQDN: `orders.commerce.pod.cluster.local`; Pod DNS record: `10.244.2.15.commerce.svc.cluster.local`
- C. Service FQDN: `commerce.orders.svc.local`
- D. Service FQDN: `orders.svc.commerce.cluster.local`

---

### Question 8: Pod `/etc/resolv.conf` `ndots:5` Search Path Mechanics
In a default Kubernetes Pod, `/etc/resolv.conf` contains:
```text
search default.svc.cluster.local svc.cluster.local cluster.local
nameserver 10.96.0.10
options ndots:5
```
When an application inside the Pod queries `api.stripe.com` (which contains 2 dots, fewer than `ndots:5`), what DNS queries does the C library resolver emit before querying `api.stripe.com.` as an absolute domain?
- A. It first appends each search domain (`api.stripe.com.default.svc.cluster.local`, `api.stripe.com.svc.cluster.local`, `api.stripe.com.cluster.local` for both A and AAAA records—6 wasted queries) before finally querying `api.stripe.com.`.
- B. It queries `api.stripe.com.` immediately on the first packet.
- C. It drops the query because it has fewer than 5 dots.
- D. It queries `/etc/hosts` only.

---

### Question 9: Pod `dnsPolicy` Modes (`ClusterFirst` vs `Default` vs `ClusterFirstWithHostNet` vs `None`)
A DaemonSet pod runs with `hostNetwork: true` so it can bind directly to the node's network interface, but it ALSO needs to resolve internal Kubernetes Service names (`*.svc.cluster.local`) via CoreDNS. Which `dnsPolicy` MUST be set on the Pod?
- A. `ClusterFirstWithHostNet` (because when `hostNetwork: true` is set, a Pod with default `dnsPolicy: ClusterFirst` automatically falls back to the node's `/etc/resolv.conf` unless `ClusterFirstWithHostNet` is explicitly specified).
- B. `ClusterFirst`
- C. `Default`
- D. `None`

---

### Question 10: Customizing CoreDNS via `Corefile` (`hosts` / `rewrite` / `forward`)
An exam task asks you to configure CoreDNS so that any pod in the cluster resolving `legacy-db.corp.internal` receives IP `192.168.50.100`, and queries for `*.consul.local` are forwarded to `10.10.0.53`. Which resource do you edit?
- A. Edit the `coredns` ConfigMap in namespace `kube-system` (`kubectl edit cm coredns -n kube-system`), add a `hosts` block (or `rewrite` rule) and a `consul.local:53 { forward . 10.10.0.53 }` server block to the `Corefile`, and run `kubectl rollout restart deploy coredns -n kube-system`.
- B. Edit `/etc/hosts` on the control-plane node.
- C. Edit `kube-proxy` ConfigMap.
- D. Create an `ExternalName` Service in `kube-system`.

---

## Answer Key & Detailed Explanations

1. **B** — Separate list items (`-`) under `from` or `to` are evaluated with logical **OR**.
2. **A** — Combining `namespaceSelector` and `podSelector` under a single list item (`-`) evaluates them with logical **AND**.
3. **A** — An empty `podSelector: {}` selects all pods in the namespace, and listing `policyTypes: ["Ingress", "Egress"]` with no allow rules blocks all ingress and egress traffic.
4. **A** — NetworkPolicy egress rules apply to all outbound traffic, including UDP/TCP port 53 DNS resolution to CoreDNS.
5. **A** — Kubernetes automatically sets `kubernetes.io/metadata.name: <namespace-name>` on every Namespace object.
6. **A** — `ipBlock` supports a primary `cidr` range and an array of `except` sub-ranges.
7. **A** — Services use `<svc>.<ns>.svc.cluster.local`, while Pod IP records replace dots with dashes (`<ip-with-dashes>.<ns>.pod.cluster.local`).
8. **A** — Any query name with fewer than `ndots` dots is tried against every entry in the `search` list first. Lowering `ndots` to `2` via `spec.dnsConfig.options` or appending a trailing dot (`api.stripe.com.`) avoids search-path amplification.
9. **A** — `ClusterFirstWithHostNet` forces pods using `hostNetwork: true` to still use the cluster's CoreDNS `nameserver` instead of the host's `/etc/resolv.conf`.
10. **A** — CoreDNS configuration lives in the `Corefile` key of the `coredns` ConfigMap in `kube-system`.
