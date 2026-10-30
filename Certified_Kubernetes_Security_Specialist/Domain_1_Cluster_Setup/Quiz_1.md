# CKS Domain 1: Cluster Setup — Quiz 1 (NetworkPolicies & Cloud Metadata Protection)

Test your mastery of Kubernetes `NetworkPolicy` evaluation logic, selector semantics, DNS egress, and Cloud Metadata Server protection.

---

### Question 1
You apply the following `NetworkPolicy` in the `payments` namespace:
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend
  namespace: payments
spec:
  podSelector:
    matchLabels:
      tier: backend
  policyTypes:
    - Ingress
  ingress:
    - from:
        - namespaceSelector:
            matchLabels:
              env: prod
        - podSelector:
            matchLabels:
              tier: frontend
```
Which Pods are allowed to initiate connections to `tier: backend` Pods in the `payments` namespace?

- A) Only Pods labeled `tier: frontend` that are running inside namespaces labeled `env: prod`.
- B) All Pods in any namespace labeled `env: prod`, PLUS any Pod in the `payments` namespace labeled `tier: frontend`.
- C) All Pods labeled `tier: frontend` in any namespace across the cluster.
- D) No Pods, because `ports` is omitted from the `ingress` rule.

<details>
<summary>Reveal Answer & Explanation</summary>

**Correct Answer: B**

**Explanation:**
Because `namespaceSelector` and `podSelector` are two separate YAML list items (each prefixed with a `-` under `from:`), Kubernetes evaluates them with a **logical OR**:
1. `- namespaceSelector: {matchLabels: {env: prod}}` matches **all Pods** in any namespace labeled `env=prod`.
2. `- podSelector: {matchLabels: {tier: frontend}}` matches Pods labeled `tier=frontend` in the **policy's own namespace** (`payments`).
Also, omitting `ports:` in an ingress rule allows traffic on **all ports** from the matched sources. To enforce a logical **AND**, remove the second `-` so both selectors belong to the same list item.
</details>

---

### Question 2
A cluster administrator applies a default-deny policy in the `checkout` namespace with `podSelector: {}` and `policyTypes: ["Ingress", "Egress"]`. Immediately afterward, an application developer adds a second `NetworkPolicy` allowing `app: cart` Pods to connect to `app: redis` Pods on TCP port `6379`. However, `cart` Pods still fail to connect to `redis-service.checkout.svc.cluster.local:6379` with `Could not resolve host: redis-service`. Why?

- A) Default-deny NetworkPolicies take precedence over allow NetworkPolicies unless a higher priority weight is set.
- B) `cart` Pods lack an Egress rule permitting UDP/TCP port `53` traffic to the `kube-dns` / CoreDNS Pods in `kube-system`.
- C) `redis` Pods must also have an Egress rule allowing return packets from TCP port `6379` back to `cart` Pods.
- D) Kubernetes Services (`ClusterIP`) cannot be used when Egress NetworkPolicies are active; Pods must connect by Pod IP.

<details>
<summary>Reveal Answer & Explanation</summary>

**Correct Answer: B**

**Explanation:**
When `default-deny-all` isolated all Pods in `checkout` for `Egress`, DNS queries from `cart` Pods to CoreDNS (`kube-dns` in `kube-system` on UDP/TCP port 53) were also blocked. Because `NetworkPolicy` rules are additive (union), you must add an Egress rule allowing UDP/TCP port `53` to `kube-system` (`k8s-app: kube-dns`). Return packets on established connections (Choice C) are handled statefully by the CNI and do not require a separate rule.
</details>

---

### Question 3
Which `NetworkPolicy` configuration properly blocks Pods labeled `role: worker` from accessing the cloud provider instance metadata endpoint (`169.254.169.254`) while permitting all other outbound traffic?

- A) Set `policyTypes: ["Egress"]` with `egress: [{to: [{ipBlock: {cidr: 169.254.169.254/32, action: Deny}}]}]`.
- B) Set `policyTypes: ["Egress"]` with `egress: [{to: [{ipBlock: {cidr: 0.0.0.0/0, except: ["169.254.169.254/32"]}}]}]`.
- C) Set `policyTypes: ["Ingress"]` with `ingress: [{from: [{ipBlock: {cidr: 0.0.0.0/0, except: ["169.254.169.254/32"]}}]}]`.
- D) Add annotation `networking.k8s.io/block-metadata: "true"` to the `worker` Deployment.

<details>
<summary>Reveal Answer & Explanation</summary>

**Correct Answer: B**

**Explanation:**
Standard Kubernetes `NetworkPolicy` objects do not have a `Deny` action field. Instead, once a Pod is selected for `Egress`, all outbound traffic is denied except what matches the `to:` rules. By allowing `cidr: 0.0.0.0/0` with `except: ["169.254.169.254/32"]`, all IPv4 destinations are permitted *except* `169.254.169.254/32`.
</details>

---

### Question 4
You need to write a `NetworkPolicy` in the `database` namespace that allows Ingress on TCP port `3306` to `app: mysql` Pods from Pods labeled `app: api` located in **any** namespace across the entire cluster. Which `from:` block achieves this?

- A)
```yaml
from:
  - podSelector:
      matchLabels:
        app: api
```
- B)
```yaml
from:
  - namespaceSelector: {}
    podSelector:
      matchLabels:
        app: api
```
- C)
```yaml
from:
  - namespaceSelector: {}
  - podSelector:
      matchLabels:
        app: api
```
- D)
```yaml
from:
  - namespaceSelector:
      matchLabels:
        app: api
```

<details>
<summary>Reveal Answer & Explanation</summary>

**Correct Answer: B**

**Explanation:**
In a single list item (logical AND), `namespaceSelector: {}` matches all namespaces in the cluster, and `podSelector: {matchLabels: {app: api}}` restricts the match to only Pods labeled `app: api` within those namespaces. Choice A only matches `app: api` Pods inside the `database` namespace itself. Choice C uses two list items (OR), which would allow *all* Pods in *all* namespaces!
</details>

---

### Question 5
Which of the following Container Network Interface (CNI) plugins does **NOT** natively enforce Kubernetes `NetworkPolicy` objects when deployed standalone?

- A) Calico
- B) Cilium
- C) Flannel
- D) Antrea

<details>
<summary>Reveal Answer & Explanation</summary>

**Correct Answer: C**

**Explanation:**
Standalone **Flannel** only provides Layer-3 overlay packet forwarding (VXLAN/host-gw) and silently ignores `NetworkPolicy` resources. To enforce NetworkPolicies, a cluster must use a policy-capable CNI such as Calico, Cilium, Weave Net, Antrea, or Canal (Flannel + Calico).
</details>

---

### Question 6
Look at the following `NetworkPolicy`:
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: MysteryPolicy
  namespace: default
spec:
  podSelector: {}
  policyTypes:
    - Ingress
  ingress:
    - {}
```
What is the effect of `MysteryPolicy` on the `default` namespace?

- A) It denies all Ingress traffic to all Pods in the `default` namespace.
- B) It allows all Ingress traffic from any source (all Pods and external IPs) on all ports to all Pods in the `default` namespace.
- C) It allows Ingress traffic only between Pods within the `default` namespace.
- D) It is rejected by the API server as invalid YAML.

<details>
<summary>Reveal Answer & Explanation</summary>

**Correct Answer: B**

**Explanation:**
Look closely at `ingress: - {}`. Unlike `ingress: []` (or omitting `ingress:` entirely), which has zero allow rules and therefore **denies all ingress**, `ingress: - {}` contains an empty rule object `{}` with no `from:` and no `ports:` restrictions. An empty `{}` rule matches **all sources and all ports**, meaning it explicitly **allows all Ingress traffic**!
</details>

---

### Question 7
Since Kubernetes v1.21+, which built-in label is automatically applied by the control plane to every Namespace, making it easy to target a specific Namespace by name in a `namespaceSelector`?

- A) `namespace.kubernetes.io/name`
- B) `kubernetes.io/metadata.name`
- C) `k8s-app/namespace`
- D) `metadata.name`

<details>
<summary>Reveal Answer & Explanation</summary>

**Correct Answer: B**

**Explanation:**
Kubernetes automatically sets the immutable label `kubernetes.io/metadata.name: <namespace-name>` on every Namespace.
</details>

---

### Question 8
In a Cilium-enabled cluster, you create a `CiliumNetworkPolicy` that uses `toFQDNs` to allow Egress to `matchPattern: "*.stripe.com"` on TCP port `443`. Why must the same policy (or another policy selecting the Pod) also include an Egress rule to `kube-dns` with `rules: dns: [{matchPattern: "*"}]`?

- A) Because `toFQDNs` is deprecated in Cilium v1.14+.
- B) Because Cilium's DNS Proxy intercepts the Pod's DNS lookup response from `kube-dns` to dynamically learn which IP addresses correspond to `*.stripe.com`.
- C) Because `toFQDNs` only works for UDP traffic unless `rules: dns` is specified.
- D) Because `kube-dns` performs TLS termination for `*.stripe.com`.

<details>
<summary>Reveal Answer & Explanation</summary>

**Correct Answer: B**

**Explanation:**
Cilium enforces `toFQDNs` at the eBPF socket/packet level by intercepting DNS responses via its L7 DNS Proxy (`rules: dns: [{matchPattern: "*"}]`). When the Pod queries `api.stripe.com`, Cilium sees the DNS A/AAAA record IPs returned by `kube-dns` and dynamically populates its eBPF policy map for that Pod before the Pod initiates the TCP 443 connection.
</details>

---

### Question 9
Pod A (`app: web`) in namespace `ns-a` sends a TCP request to Pod B (`app: api`) on port `8080` in namespace `ns-b`. Both namespaces have a `default-deny-all` (Ingress + Egress) policy applied. What is the **minimum** set of additional rules required for this TCP connection to succeed (assuming Pod A connects directly to Pod B's IP)?

- A) Only an Egress rule in `ns-a` allowing Pod A to reach Pod B on TCP 8080.
- B) Only an Ingress rule in `ns-b` allowing Pod B to receive traffic from Pod A on TCP 8080.
- C) An Egress rule in `ns-a` allowing Pod A to send to Pod B on TCP 8080, AND an Ingress rule in `ns-b` allowing Pod B to receive from Pod A on TCP 8080.
- D) Egress and Ingress rules on both TCP 8080 and ephemeral ports `32768-60999` in both namespaces.

<details>
<summary>Reveal Answer & Explanation</summary>

**Correct Answer: C**

**Explanation:**
When both the source Pod (`Pod A`) is isolated for Egress and the destination Pod (`Pod B`) is isolated for Ingress, **both** sides must permit the flow: `Pod A` needs an Egress rule to `Pod B:8080`, and `Pod B` needs an Ingress rule from `Pod A` on port `8080`. Ephemeral return ports are tracked statefully by conntrack/eBPF and do not need explicit rules.
</details>

---

### Question 10
Which field in a standard `networking.k8s.io/v1` `NetworkPolicy` allows you to specify a continuous range of TCP ports (for example, ports `30000` through `32767`) in a single port entry?

- A) `portRange: "30000-32767"`
- B) `port: 30000` combined with `endPort: 32767`
- C) `minPort: 30000` combined with `maxPort: 32767`
- D) Standard `NetworkPolicy` does not support port ranges; every port must be listed individually.

<details>
<summary>Reveal Answer & Explanation</summary>

**Correct Answer: B**

**Explanation:**
Kubernetes `NetworkPolicyPort` supports the `endPort` field alongside `port` (e.g., `port: 30000`, `endPort: 32767`) to target a contiguous range of ports without listing each port separately.
</details>
