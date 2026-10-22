# Domain 4: Platform Architecture & Scalability — Quiz 2

**Exam:** Certified Cloud Native Platform Engineer (CNPE)  
**Format:** 10 Hands-On Scenario & Performance Questions with Detailed Rationales

---

### Question 1: Karpenter Drift Detection Mechanics
How does Karpenter v1.0 detect and handle "drift" when a platform engineer updates the AMI ID or security group in an `EC2NodeClass`?
- A. It immediately deletes all active worker nodes without draining pods.
- B. It marks existing nodes provisioned under that NodeClass as drifted and gradually drains and replaces them according to disruption budgets.
- C. It rejects the update and returns an admission error.
- D. Drift detection is only supported in Cluster Autoscaler.

---

### Question 2: Gateway API HTTPRoute URL Rewriting
You need to configure an `HTTPRoute` rule so that client requests to `/api/v2/users` are forwarded to the backend service path `/users`. Which Gateway API filter must be added to the route rule?
- A. `URLRewrite` filter with `path: { type: ReplacePrefixMatch, replacePrefixMatch: "/users" }`
- B. `RequestHeaderModifier` filter
- C. `Redirect` filter
- D. `Mirror` filter

---

### Question 3: LimitRange Default Values
A namespace has a `LimitRange` configured with `defaultRequest.cpu: 250m` and `default.cpu: 500m`. A developer applies a deployment manifest without specifying any resource requests or limits. What happens to the created pods?
- A. The pods are rejected by the admission webhook.
- B. The admission controller automatically mutates the pod specification to set CPU requests to 250m and CPU limits to 500m.
- C. The pods run with unlimited CPU.
- D. The nodes immediately crash.

---

### Question 4: KEDA Cooldown Period Configuration
After processing a sudden surge of messages from Kafka, the queue drops back to 0. However, the worker pods remain running at 20 replicas for another 5 minutes before terminating. Which KEDA parameter governs this delay?
- A. `pollingInterval`
- B. `cooldownPeriod`
- C. `lagThreshold`
- D. `minReplicaCount`

---

### Question 5: vCluster Syncer Controller Responsibility
What is the primary operational role of the `vcluster-syncer` component running inside the host namespace?
- A. It compiles Go code into container images.
- B. It translates virtual cluster pod definitions and creates corresponding physical shadow pods in the host cluster namespace, while syncing pod statuses back to the virtual API server.
- C. It configures physical BGP routers in the data center.
- D. It manages SSL certificates for the cloud provider.

---

### Question 6: Karpenter Spot Instance Diversification
To minimize the impact of AWS EC2 Spot capacity evictions, how should a Karpenter `NodePool` requirements section be configured?
- A. By specifying a single instance type (e.g. `values: ["m5.large"]`).
- B. By specifying broad requirements with multiple instance families, generations, and sizes (e.g. `instance-category: In ["c", "m", "r"]`, `cpu: In ["4", "8", "16"]`).
- C. By setting `capacity-type: ["on-demand"]` only.
- D. By disabling consolidation.

---

### Question 7: ResourceQuota Scope Selectors
A platform team wants to restrict the total number of pods running in a namespace that do NOT have a `BestEffort` quality of service (QoS) class. Which field in `ResourceQuota` allows filtering quotas by QoS or priority class?
- A. `spec.hard.pods`
- B. `spec.scopes` (e.g. `NotBestEffort`)
- C. `metadata.annotations`
- D. `spec.selector`

---

### Question 8: Cilium Gateway API L7 Traffic Splitting
You want to implement a 90/10 canary traffic split across two service versions (`v1` and `v2`) using Gateway API. In which resource and section is this defined?
- A. In `GatewayClass` under `parametersRef`.
- B. In `HTTPRoute` under `spec.rules.backendRefs` using the `weight` attribute on each service backend.
- C. In `NetworkPolicy` under `egress`.
- D. In `ConfigMap` under `env.sh`.

---

### Question 9: Cluster API Infrastructure Providers
In Cluster API (CAPI), what is the function of an Infrastructure Provider (e.g., CAPA for AWS, CAPD for Docker, CAPDO for DigitalOcean)?
- A. It reconciles cloud infrastructure primitives such as VPCs, subnets, load balancers, and VM instances required to run the Kubernetes cluster.
- B. It instruments pods with OpenTelemetry.
- C. It runs security vulnerability scans on container images.
- D. It monitors Grafana dashboards.

---

### Question 10: Karpenter Consolidation vs Node Drain
When Karpenter decides to consolidate a node by replacing it with a cheaper instance, how does it ensure running pods are terminated safely?
- A. It executes `SIGKILL` on all processes immediately.
- B. It cordons the node, executes standard Kubernetes eviction respecting PodDisruptionBudgets and preStop lifecycle hooks, and waits for pods to terminate before deleting the cloud VM.
- C. It leaves the old node running forever.
- D. It reboots the physical hypervisor.

---

## Answer Key & Rationales

1. **B** — Karpenter v1.0 actively detects drift between live Node resources and the updated `NodePool`/`EC2NodeClass` CRDs, scheduling graceful eviction and replacement based on disruption budgets.
2. **A** — `URLRewrite` filters in Gateway API rewrite incoming request URLs before forwarding traffic to the designated backend service.
3. **B** — A Kubernetes `LimitRange` admission plugin mutates pods that lack explicit requests or limits, setting them to configured namespace defaults.
4. **B** — `cooldownPeriod` defines the duration (in seconds) KEDA waits before scaling down a target workload after all trigger metrics fall below threshold.
5. **B** — The vCluster syncer intercepts workloads created in the virtual API server and creates actual scheduled pods on the host cluster namespace.
6. **B** — Diversifying across multiple instance categories, families, and generations maximizes the availability of Spot compute pools and prevents widespread eviction.
7. **B** — `spec.scopes` allows quotas to target specific workload subsets, such as `BestEffort`, `NotBestEffort`, `Terminating`, or `NotTerminating`.
8. **B** — `HTTPRoute` defines canary weighting directly in `backendRefs` by assigning integer weights to target services (e.g., weight: 90 vs weight: 10).
9. **A** — Infrastructure providers in CAPI handle the creation, provisioning, and teardown of cloud-specific underlying VMs and networking infrastructure.
10. **B** — Karpenter cordons and drains nodes using the Kubernetes eviction API, strictly respecting PDBs and termination grace periods.
