# Domain 3: Services & Networking — Quiz 1 (Core Networking & Services)

**Exam:** Certified Kubernetes Administrator (CKA)
**Format:** Performance Scenario & Multiple Choice Questions with Explanations

---

### Question 1: Default Ingress Traffic Behavior

A new namespace `marketing` has 5 pods running. No NetworkPolicy exists in the namespace. Can pods from another namespace `sales` communicate with pods in `marketing`?

- A. No, namespaces isolate traffic by default in Kubernetes.
- B. Yes, by default all pods in a Kubernetes cluster can communicate with all other pods across all namespaces without NAT.
- C. Only if a Service of type NodePort is created.
- D. Only if an Ingress resource is defined.

---

### Question 2: NetworkPolicy AND vs OR List Semantics

An administrator writes a NetworkPolicy ingress rule with `- from:` containing two separate list items: one `namespaceSelector` and one `podSelector`. How does Kubernetes interpret this rule?

- A. Logical AND: Traffic is allowed only from pods matching the podSelector inside namespaces matching the namespaceSelector.
- B. Logical OR: Traffic is allowed from ANY pod in the matching namespace OR from any pod in the current namespace matching the podSelector.
- C. The policy is syntactically invalid.
- D. Kubernetes denies all traffic.

---

### Question 3: Headless Service IP Allocation

When creating a Headless Service with `clusterIP: None`, what IP does `kubectl get svc` report for the ClusterIP column?

- A. The node IP address.
- B. `None`
- C. A random IP from the `service-cluster-ip-range`.
- D. `Pending`

---

## Answer Key & Detailed Explanations

### Question 1: Default Ingress Traffic Behavior
**Answer:** B
**Explanation:** In the Kubernetes networking model, network isolation is NOT enabled by default. Pods are non-isolated until selected by at least one NetworkPolicy.

---

### Question 2: NetworkPolicy AND vs OR List Semantics
**Answer:** B
**Explanation:** In YAML, separate dash `-` items inside `from` represent an OR relationship. To specify an AND relationship, the `podSelector` must be placed directly under the same list element as `namespaceSelector`.

---

### Question 3: Headless Service IP Allocation
**Answer:** B
**Explanation:** Headless services do not allocate a virtual ClusterIP; `kubectl get svc` displays `None`. DNS queries directly return the A records of the matching pods.

---

