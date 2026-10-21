# PE Domain 3: Kubernetes Platform Engineering — 25 Core Interview Questions

Master these questions across Technical Screening, Kubernetes Deep Dive (Mock 3), and Staff Architect rounds.

---

## Tier 1: Core / Screening Questions (Questions 1–10)

### Q1: Walk through the lifecycle of a request entering `kube-apiserver`.
- **Short Answer:** The request passes through Authentication (AuthN), Authorization (AuthZ), Mutating Admission Webhooks, Schema Validation (OpenAPI & CEL), Validating Admission Webhooks / Policies, and is finally committed to etcd via Raft consensus.
- **Deep Dive:** If any stage fails (e.g., token invalid in AuthN, RBAC forbidden in AuthZ, or CEL rule violation in validation), the pipeline terminates immediately with an appropriate HTTP error (401, 403, 422).
- **Trade-offs & Realities:** Slow mutating or validating webhooks directly degrade cluster-wide API latency, as every write must wait for external webhook responses.
- **Follow-up Probes:** "What is the difference between Mutating and Validating webhooks?" (Mutating runs first and can modify the object; Validating runs last and can only accept or reject).

### Q2: What is the purpose of `metadata.resourceVersion` in Kubernetes?
- **Short Answer:** It is an internal revision identifier managed by etcd that enables Optimistic Concurrency Control and streaming watches.
- **Deep Dive:** When updating an object, the client must submit the current `resourceVersion`. If another write occurred in the interim, the API server rejects the update with `409 Conflict`, preventing concurrent write overwrites.
- **Trade-offs & Realities:** Clients must never invent or mutate `resourceVersion`; it should be treated as an opaque string returned by the API server.
- **Follow-up Probes:** "What happens if you omit `resourceVersion` on an update?" (The API server rejects the update or forces a replace depending on whether it's a PUT or PATCH).

### Q3: What is the difference between a Custom Resource Definition (CRD) and an aggregated API server?
- **Short Answer:** A CRD extends `kube-apiserver` declaratively without writing Go code, storing data in the primary etcd. An aggregated API server is a separate, standalone binary that implements the Kubernetes API specification, storing data in its own independent storage.
- **Deep Dive:** CRDs are suitable for 95% of use cases. Aggregated API servers are used when you require custom storage backends (e.g., metrics-server storing data in RAM) or custom subresources not supported by CRDs.
- **Trade-offs & Realities:** Aggregated API servers add substantial operational overhead (managing separate binaries, TLS certificates, and high-availability setups).
- **Follow-up Probes:** "Give an example of an aggregated API server." (`metrics-server` for the `metrics.k8s.io` API).

### Q4: What is a Finalizer in Kubernetes?
- **Short Answer:** A pre-deletion hook in `metadata.finalizers` that prevents physical deletion of a resource from etcd until external cleanup tasks are completed.
- **Deep Dive:** When a resource with finalizers is deleted, the API server sets `metadata.deletionTimestamp`. The owning controller detects this timestamp, destroys external assets (e.g., AWS RDS database or cloud load balancer), removes its finalizer string, and updates the object, allowing etcd to purge the record.
- **Trade-offs & Realities:** If a controller crashes or has a bug, the resource hangs in `Terminating` state indefinitely.
- **Follow-up Probes:** "How do you forcefully delete a stuck resource with a finalizer?" (`kubectl patch resource name -p '{"metadata":{"finalizers":[]}}' --type=merge`).

### Q5: What is the difference between soft and hard multi-tenancy in Kubernetes?
- **Short Answer:** Soft multi-tenancy relies on logical isolation (namespaces, RBAC, NetworkPolicies, ResourceQuotas) within a single shared cluster. Hard multi-tenancy provides physical or virtualization isolation (separate clusters or microVMs) to prevent cross-tenant kernel exploitation.
- **Deep Dive:** Soft multi-tenancy is suitable for trusted teams within the same company. Hard multi-tenancy is required for untrusted external customers or strictly regulated compliance environments (PCI, HIPAA).
- **Trade-offs & Realities:** Hard multi-tenancy increases infrastructure costs and management overhead; soft multi-tenancy carries blast-radius risks.
- **Follow-up Probes:** "Where does vCluster fit?" (vCluster provides high multi-tenancy by isolating control planes while sharing compute nodes).

### Q6: How does Karpenter differ from the Kubernetes Cluster Autoscaler?
- **Short Answer:** Cluster Autoscaler scales by modifying AWS Auto Scaling Groups (ASGs). Karpenter bypasses ASGs, evaluating unscheduled pod constraints directly and provisioning optimal EC2 instances via the EC2 Fleet API in seconds.
- **Deep Dive:** Karpenter supports group-less auto-provisioning, automated node consolidation, and native Spot interruption handling, cutting provisioning latency from minutes to ~30 seconds.
- **Trade-offs & Realities:** Karpenter requires AWS IAM permissions and careful configuration of NodePool disruption budgets.
- **Follow-up Probes:** "What is Karpenter Node Consolidation?" (Automated rescheduling of workloads onto fewer or smaller nodes to reduce idle spend).

### Q7: What are the three core personas in the Gateway API?
- **Short Answer:** Infrastructure Provider (`GatewayClass`), Platform Operator (`Gateway`), and Application Developer (`HTTPRoute`).
- **Deep Dive:** This separation of concerns allows cluster administrators to manage networking ports, TLS certificates, and IP addresses without coordinating with product teams who only manage routing rules.
- **Trade-offs & Realities:** The multi-resource model has a steeper learning curve than simple legacy Ingress.
- **Follow-up Probes:** "What is a `ReferenceGrant`?" (A security resource explicitly permitting cross-namespace references between Gateways and Routes/Secrets).

### Q8: What does `failurePolicy: Fail` mean in an admission webhook?
- **Short Answer:** It instructs `kube-apiserver` to reject API requests if the webhook endpoint is unreachable, times out, or returns a 500 error.
- **Deep Dive:** `Fail` guarantees security policy enforcement, but creates a single point of failure: if webhook pods crash, deployments across the cluster are blocked.
- **Trade-offs & Realities:** `failurePolicy: Ignore` prevents cluster outages during webhook incidents, but allows non-compliant pods to slip into production.
- **Follow-up Probes:** "How does `ValidatingAdmissionPolicy` solve this?" (It runs CEL rules in-process within `kube-apiserver`, eliminating external webhook downtime risks).

### Q9: What is the function of a PodDisruptionBudget (PDB)?
- **Short Answer:** A PDB limits the number of pods of a replicated application that can be down simultaneously during voluntary disruptions (e.g., node drain, cluster upgrade).
- **Deep Dive:** Configured via `minAvailable` or `maxUnavailable`. When `kubectl drain` runs, the eviction API checks the PDB; if evicting the pod violates the budget, eviction pauses until new pods become healthy.
- **Trade-offs & Realities:** A misconfigured PDB (e.g., `minAvailable: 100%` on a 1-replica deployment) can permanently block node upgrades.
- **Follow-up Probes:** "Does a PDB protect against hardware node crashes?" (No; PDBs only govern *voluntary* disruptions, not hardware or kernel crashes).

### Q10: How does KEDA enable scale-to-zero?
- **Short Answer:** KEDA acts as an external metrics server and controller that monitors event sources (Kafka lag, SQS depth, RabbitMQ queues). When the queue is empty, it scales the deployment down to 0 replicas, and activates it when new events arrive.
- **Deep Dive:** Traditional HPA cannot scale to zero based on CPU/RAM because an inactive pod consumes 0% CPU. KEDA directly manages the scale subresource when transitioning between 0 and 1, delegating to HPA once active.
- **Trade-offs & Realities:** Scaling from 0 incurs cold-start latency as new containers are pulled and booted.
- **Follow-up Probes:** "What happens if KEDA crashes?" (The deployment remains at its last known replica count).

---

## Tier 2: Deep Dive & Mid-Senior Questions (Questions 11–20)

### Q11: Explain the internal architecture of a `client-go` Informer and why it prevents API server overload.
- **Short Answer:** An Informer coordinates a Reflector, a DeltaFIFO queue, an Indexer (in-memory cache), and a WorkQueue.
- **Deep Dive:** The Reflector streams changes from `kube-apiserver` via HTTP/2 watches into `DeltaFIFO`. The Informer consumes the FIFO, updates the local Indexer cache, and enqueues resource keys into a rate-limiting work queue for the reconcile loop. Controllers read strictly from local RAM cache, reducing API server load from $O(N)$ polling to zero.
- **Trade-offs & Realities:** Informer caches consume controller memory. In large clusters (100k pods), controllers watching all pods require significant RAM.
- **Follow-up Probes:** "What is ResyncPeriod in an Informer?" (A periodic trigger that re-enqueues all cached items into the work queue to reconcile against potential out-of-band external drift).

### Q12: How do In-Tree CEL Validations in CRDs compare to Validating Admission Policies?
- **Short Answer:** CEL validations inside a CRD schema validate an individual Custom Resource against its own fields. ValidatingAdmissionPolicies can validate standard built-in Kubernetes resources (like Pods, Deployments) across the entire cluster.
- **Deep Dive:** CRD CEL is scoped strictly to `openAPIV3Schema.properties.x-kubernetes-validations`. `ValidatingAdmissionPolicy` is a cluster-wide admission plugin that inspects operations (`CREATE`, `UPDATE`), matches namespaces, and evaluates complex multi-object rules.
- **Trade-offs & Realities:** Both run in-process within `kube-apiserver` memory, avoiding external webhook latency.
- **Follow-up Probes:** "Can CRD CEL validation compare fields across multiple resources?" (No; CRD CEL can only evaluate `self` within the resource being validated).

### Q13: How do you design a zero-downtime cluster upgrade strategy for a multi-tenant fleet?
- **Short Answer:** Use either Blue/Green cluster replacement or automated node-by-node rolling drains governed by PDBs, topology spread constraints, and pre-warmed replacement nodes.
- **Deep Dive:**
  1. *Blue/Green Cluster:* Stand up a parallel cluster running Kubernetes $N+1$, deploy workloads via GitOps (Argo CD), validate via synthetic canary traffic, shift DNS/ingress traffic gradually, and decommission the old cluster after 48 hours.
  2. *Rolling Node Upgrade:* Cordon and drain nodes sequentially using Karpenter or managed node groups, ensuring PodDisruptionBudgets (`maxUnavailable: 1`) and readiness probes prevent service interruption.
- **Trade-offs & Realities:** Blue/Green eliminates control plane downtime risks but requires DNS cutovers and doubles infrastructure costs during migrations.
- **Follow-up Probes:** "How do you handle API deprecations before upgrading?" (Use tools like `pluto` or audit API server metrics `apiserver_requested_deprecated_apis`).

### Q14: Explain how vCluster works under the hood.
- **Short Answer:** vCluster runs an API server, etcd (or SQLite), and a lightweight syncer controller inside a single pod in a host cluster namespace.
- **Deep Dive:** The tenant interacts with the virtual API server. When the tenant creates a Pod, the vCluster syncer controller intercepts it and creates a matching "shadow pod" in the host cluster namespace. The host cluster's kubelet and CNI schedule and run the actual container. Service accounts, CRDs, and namespaces remain entirely virtualized inside the tenant's virtual API server.
- **Trade-offs & Realities:** Low memory overhead (~200MB per tenant control plane) and near-instant provisioning compared to spinning up complete EKS clusters.
- **Follow-up Probes:** "Can a tenant in vCluster modify host node sysctls?" (No; vCluster does not grant host-level privileges).

### Q15: How does the Kubernetes Horizontal Pod Autoscaler handle flapping (rapid scaling oscillations)?
- **Short Answer:** Through the `behavior` field, which configures stabilization windows and rate-limiting policies for scaling up and down independently.
- **Deep Dive:** By default, HPA uses a 300-second (5-minute) stabilization window for scale-down. It inspects all calculated replica counts across the last 5 minutes and selects the maximum value, preventing a temporary drop in traffic from prematurely killing replicas.
- **Trade-offs & Realities:** A long stabilization window prevents flapping but increases cloud costs by retaining excess capacity longer than strictly necessary.
- **Follow-up Probes:** "How do you configure aggressive scale-up but cautious scale-down?" (Set `scaleUp.stabilizationWindowSeconds: 0` and `scaleDown.stabilizationWindowSeconds: 300`).

### Q16: What happens when an etcd cluster loses quorum?
- **Short Answer:** etcd enters read-only mode (or fails health checks entirely) and rejects all write operations. Running workloads continue executing, but the cluster cannot schedule new pods, update statuses, or deploy changes.
- **Deep Dive:** If a 3-node etcd cluster loses 2 nodes, Raft consensus cannot be reached ($\lfloor 3/2 \rfloor + 1 = 2$). The API server returns HTTP 500 errors for all mutating requests.
- **Trade-offs & Realities:** Static stability: running data-plane containers continue serving traffic because worker nodes and kubelet rely on local state.
- **Follow-up Probes:** "How do you recover an etcd cluster that permanently lost quorum?" (Restore from the latest snapshot using `etcdctl snapshot restore`).

### Q17: Compare Kubernetes Ingress vs Gateway API for canary deployments.
- **Short Answer:** Legacy Ingress has no standard API for canary routing, forcing teams to rely on vendor-specific annotations (`nginx.ingress.kubernetes.io/canary-weight`). Gateway API provides first-class, vendor-neutral `weight` fields directly in `HTTPRoute.spec.rules.backendRefs`.
- **Deep Dive:** Gateway API allows portable canary traffic splitting across any conforming implementation (Envoy Gateway, Cilium, Istio, AWS VPC Lattice) without rewriting manifests.
- **Trade-offs & Realities:** Gateway API requires installing CRDs and learning a multi-layered resource model.
- **Follow-up Probes:** "Can Gateway API route traffic based on HTTP headers?" (Yes, via `matches.headers` in the `HTTPRoute` rule).

### Q18: What is the purpose of `topologySpreadConstraints` in Kubernetes pod scheduling?
- **Short Answer:** It controls how pods are distributed across failure domains (availability zones, regions, or nodes) to achieve high availability and prevent single-point-of-failure outages.
- **Deep Dive:** Defines `maxSkew` (the maximum allowed difference in pod count between any two zones), `topologyKey` (`topology.kubernetes.io/zone`), and `whenUnsatisfiable` (`DoNotSchedule` or `ScheduleAnyway`).
- **Trade-offs & Realities:** Setting `whenUnsatisfiable: DoNotSchedule` can cause pods to remain unscheduled if an entire AZ runs out of capacity.
- **Follow-up Probes:** "How does it compare to `podAntiAffinity`?" (Topology spread provides balanced distribution across multiple zones, whereas anti-affinity is binary).

### Q19: Explain the difference between `StorageClass`, `PersistentVolume` (PV), and `PersistentVolumeClaim` (PVC).
- **Short Answer:** `StorageClass` defines the storage provisioner and parameters (e.g., AWS EBS `gp3`). A `PVC` is a developer's request for storage. A `PV` is the actual provisioned storage volume bound to the PVC.
- **Deep Dive:** When a PVC is created, the CSI driver dynamically provisions the cloud disk and creates a matching PV with `volumeBindingMode: WaitForFirstConsumer` to ensure the disk is created in the same Availability Zone as the scheduled pod.
- **Trade-offs & Realities:** EBS volumes are zone-locked: a pod bound to an EBS volume in `us-east-1a` cannot run on a node in `us-east-1b`.
- **Follow-up Probes:** "What storage class setting allows multi-pod read-write access?" (`ReadWriteMany` using EFS, NFS, or Ceph).

### Q20: What are conversion webhooks in CRD multi-versioning?
- **Short Answer:** When a CRD supports multiple API versions (e.g., `v1alpha1` and `v1`), a conversion webhook translates resources between different schema versions on the fly when requested by clients.
- **Deep Dive:** etcd stores the object in the designated `storage: true` version. If a legacy client requests `v1alpha1`, `kube-apiserver` calls the conversion webhook to convert the stored `v1` representation into `v1alpha1` before responding.
- **Trade-offs & Realities:** Conversion webhooks must be highly available and backward compatible; a bug in conversion breaks API access for that resource type.
- **Follow-up Probes:** "Can CRD conversion be handled without webhooks?" (Only if versions have identical structural schemas; semantic transformations require a webhook).

---

## Tier 3: Staff & Principal Architect Scenarios (Questions 21–25)

### Q21: Architect an enterprise Kubernetes platform hosting 5,000 microservices across 50 autonomous business units.
- **Candidate Framework:**
  - *Cluster Topology:* Multi-cluster fleet partitioned by business unit domain and compliance tier (e.g., Tier 1 Core Banking on dedicated PCI clusters; Tier 2 Internal Services on shared multi-tenant clusters managed via vCluster).
  - *Fleet Management:* Use Cluster API (CAPI) and Argo CD ApplicationSets to manage cluster lifecycle and deploy baseline configurations declaratively.
  - *Control Plane Hardening:* Deploy ValidatingAdmissionPolicies (in-tree CEL) for guardrails; eliminate external admission webhooks.
  - *Compute Autoscaling:* Standardize on Karpenter across all EKS clusters with Spot/On-Demand consolidation and multi-AZ topology spread.
  - *Networking & Routing:* Deploy Gateway API with Cilium eBPF CNI in native routing mode, providing cross-namespace routing and mTLS without sidecar overhead.
- **Red Flags:** Proposing a single massive 5,000-service cluster or manually managing 50 separate clusters with bespoke scripts.

### Q22: A developer reports: "Our deployment rolling update is failing; half the pods are crashing, but rollback is blocked." How do you lead incident response?
- **Candidate Framework:**
  - *Mitigate First:* Immediately halt the rollout (`kubectl rollout pause deployment/<name>`). Force traffic to the surviving healthy pods or route ingress traffic to the previous known good deployment.
  - *Diagnose:* Inspect `kubectl rollout status`, check `dmesg` / events for OOMKills (Exit 137), check Readiness Probes, inspect container logs for failed DB migrations.
  - *Unblock Rollback:* If rollback is blocked due to PDB constraints or bad image tags, temporarily scale the replica count or patch the image explicitly (`kubectl rollout undo`).
  - *Prevent Recurrence:* Add automated canary analysis (Argo Rollouts) with Prometheus analysis templates to abort deployments automatically before impacting >10% of users.
- **Red Flags:** Spending 30 minutes reading application code while production is down instead of pausing the rollout and mitigating user impact.

### Q23: How do you solve the "Zone-Locked EBS Volume" challenge when autoscaling stateful workloads across multiple Availability Zones?
- **Candidate Framework:**
  - *Understand Constraint:* AWS EBS volumes exist in a single AZ and cannot attach to an EC2 instance in a different AZ.
  - *StorageClass Configuration:* Use `volumeBindingMode: WaitForFirstConsumer` so the PV is not created until the pod is assigned to a node in a specific AZ.
  - *Workload Architecture:* Deploy StatefulSets with `volumeClaimTemplates` and assign pods with `topologySpreadConstraints` so replicas are evenly distributed.
  - *Karpenter Integration:* Karpenter automatically detects the volume's AZ affinity and provisions the replacement node in the exact AZ where the PV resides.
  - *Alternative Shared Storage:* For multi-AZ read-write requirements, migrate to AWS EFS (using EFS CSI Driver) or distributed software-defined storage (Ceph/Rook).
- **Red Flags:** Suggesting manual volume snapshots and EBS migrations during pod failover.

### Q24: What are the trade-offs of using eBPF-based host routing (Cilium) versus AWS VPC CNI on EKS?
- **Candidate Framework:**
  - *AWS VPC CNI:*
    - *Pros:* Pods receive native AWS VPC IP addresses; first-class AWS Security Groups per Pod; native AWS VPC flow logs.
    - *Cons:* Heavy ENI/IP exhaustion per node; high AWS API call volume for IP allocation; relies on kube-proxy iptables unless tuned.
  - *Cilium eBPF Host Routing:*
    - *Pros:* Ultra-low latency ($O(1)$ BPF map lookups); bypasses netfilter/iptables; native L7 network policies and transparent encryption (WireGuard/IPsec); scales to 100k services.
    - *Cons:* Overlay encapsulation mode masks pod IPs in VPC flow logs (unless running in ENI mode); requires advanced Linux kernel and eBPF operational expertise.
- **Red Flags:** Failing to mention AWS ENI allocation limits per instance type.

### Q25: How would you design a self-service Kubernetes operator that manages external cloud databases with disaster recovery failover?
- **Candidate Framework:**
  - *CRD Design:* Define `kind: ManagedDatabase` with `.spec.engine`, `.spec.storage`, `.spec.multiRegion`, and `/status` subresource.
  - *Reconciliation Architecture:* Use Kubebuilder controller. The reconcile loop calls AWS SDK to provision primary RDS and cross-region read replica.
  - *Health Monitoring:* Reconciler periodically verifies database availability and replication lag, posting condition updates (`type: Ready`, `status: True`).
  - *Automated Failover:* If primary region health check fails for >3 consecutive periods, controller initiates promotion of the read replica, updates Route53 CNAME, and sets `.status.endpoint` to the new primary.
  - *Finalizer Protection:* Attach `platform.example.com/rds-cleanup` to ensure cloud databases are cleanly snapshotted before resource deletion.
- **Red Flags:** Hardcoding AWS credentials in the controller or lacking finalizer teardown logic.
