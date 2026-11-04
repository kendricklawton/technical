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
- A. The policy is syntactically invalid.
- B. Logical AND: Traffic is allowed only from pods matching `role: frontend` inside namespaces matching `env: prod`.
- C. Kubernetes denies all traffic.
- D. Logical OR: Traffic is allowed from ANY pod in a namespace labeled `env: prod` OR from any pod in the NetworkPolicy's own namespace labeled `role: frontend`.

---

### Question 2: Writing a Logical AND Selector in NetworkPolicy
How do you modify the YAML in Question 1 so traffic is allowed **only** from pods labeled `role: frontend` that reside inside namespaces labeled `env: prod`?
- A. Add `operator: AND` to the NetworkPolicy spec.
- B. Create two separate NetworkPolicies.
- C. Remove the second dash (`-`) before `podSelector` so both `namespaceSelector` and `podSelector` are sibling keys inside the exact same `from` list element:
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
- D. Use `ipBlock` instead.

---

### Question 3: Default Deny All Ingress AND Egress Traffic
You need to lock down namespace `pci-dss` so no pod can receive any inbound traffic or initiate any outbound traffic unless explicitly allowed by another NetworkPolicy. What does the Default Deny All NetworkPolicy spec look like?
- A. `ingress: [{}]` and `egress: [{}]`.
- B. `podSelector: {}` and `policyTypes: ["Ingress", "Egress"]` with no `ingress:` or `egress:` rules defined.
- C. `podSelector: { matchLabels: { all: "true" } }` and `policyTypes: ["None"]`.
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
- A. `metadata.namespace`
- B. `k8s.io/namespace`
- C. `kubernetes.io/metadata.name`
- D. `name`

---

### Question 6: `ipBlock` with `except` CIDR Ranges
A NetworkPolicy needs to allow egress traffic to any external IP in `172.16.0.0/12` **except** the restricted management subnet `172.16.100.0/24`. Which `from`/`to` peer syntax achieves this?
- A. `ipBlock: { cidr: 172.16.0.0/12, except: ["172.16.100.0/24"] }`
- B. NetworkPolicy cannot filter by IP CIDR blocks.
- C. `cidrSelector: { allow: 172.16.0.0/12, deny: 172.16.100.0/24 }`
- D. `podSelector: { ip: "!172.16.100.0/24" }`

---

### Question 7: Kubernetes FQDN Format for Services and Pods
A Pod in namespace `web` wants to send an HTTP request to a Service named `orders` in namespace `commerce` on a cluster configured with the default cluster domain `cluster.local`. What is the complete FQDN of the Service, and what would the Pod IP DNS record look like for a Pod with IP `10.244.2.15` in `commerce`?
- A. Service FQDN: `orders.commerce.pod.cluster.local`; Pod DNS record: `10.244.2.15.commerce.svc.cluster.local`
- B. Service FQDN: `commerce.orders.svc.local`
- C. Service FQDN: `orders.svc.commerce.cluster.local`
- D. Service FQDN: `orders.commerce.svc.cluster.local`; Pod DNS record: `10-244-2-15.commerce.pod.cluster.local`

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
- B. It drops the query because it has fewer than 5 dots.
- C. It queries `/etc/hosts` only.
- D. It queries `api.stripe.com.` immediately on the first packet.

---

### Question 9: Pod `dnsPolicy` Modes (`ClusterFirst` vs `Default` vs `ClusterFirstWithHostNet` vs `None`)
A DaemonSet pod runs with `hostNetwork: true` so it can bind directly to the node's network interface, but it ALSO needs to resolve internal Kubernetes Service names (`*.svc.cluster.local`) via CoreDNS. Which `dnsPolicy` MUST be set on the Pod?
- A. `ClusterFirst`
- B. `None`
- C. `Default`
- D. `ClusterFirstWithHostNet` (because when `hostNetwork: true` is set, a Pod with default `dnsPolicy: ClusterFirst` automatically falls back to the node's `/etc/resolv.conf` unless `ClusterFirstWithHostNet` is explicitly specified).

---

### Question 10: Customizing CoreDNS via `Corefile` (`hosts` / `rewrite` / `forward`)
An exam task asks you to configure CoreDNS so that any pod in the cluster resolving `legacy-db.corp.internal` receives IP `192.168.50.100`, and queries for `*.consul.local` are forwarded to `10.10.0.53`. Which resource do you edit?
- A. Create an `ExternalName` Service in `kube-system`.
- B. Edit the `coredns` ConfigMap in namespace `kube-system` (`kubectl edit cm coredns -n kube-system`), add a `hosts` block (or `rewrite` rule) and a `consul.local:53 { forward . 10.10.0.53 }` server block to the `Corefile`, and run `kubectl rollout restart deploy coredns -n kube-system`.
- C. Edit `kube-proxy` ConfigMap.
- D. Edit `/etc/hosts` on the control-plane node.

---

## Answer Key & Detailed Explanations

1. **D** — Separate list items (`-`) under `from` or `to` are evaluated with logical **OR**.
2. **C** — Combining `namespaceSelector` and `podSelector` under a single list item (`-`) evaluates them with logical **AND**.
3. **B** — An empty `podSelector: {}` selects all pods in the namespace, and listing `policyTypes: ["Ingress", "Egress"]` with no allow rules blocks all ingress and egress traffic.
4. **A** — NetworkPolicy egress rules apply to all outbound traffic, including UDP/TCP port 53 DNS resolution to CoreDNS.
5. **C** — Kubernetes automatically sets `kubernetes.io/metadata.name: <namespace-name>` on every Namespace object.
6. **A** — `ipBlock` supports a primary `cidr` range and an array of `except` sub-ranges.
7. **D** — Services use `<svc>.<ns>.svc.cluster.local`, while Pod IP records replace dots with dashes (`<ip-with-dashes>.<ns>.pod.cluster.local`).
8. **A** — Any query name with fewer than `ndots` dots is tried against every entry in the `search` list first. Lowering `ndots` to `2` via `spec.dnsConfig.options` or appending a trailing dot (`api.stripe.com.`) avoids search-path amplification.
9. **D** — `ClusterFirstWithHostNet` forces pods using `hostNetwork: true` to still use the cluster's CoreDNS `nameserver` instead of the host's `/etc/resolv.conf`.
10. **B** — CoreDNS configuration lives in the `Corefile` key of the `coredns` ConfigMap in `kube-system`.
