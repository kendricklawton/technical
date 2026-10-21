# PE Domain 3: Kubernetes Platform Engineering — Scenario Quiz 2

Test your knowledge on admission controllers, multi-tenancy, and cluster fleet management.

---

### Question 1
A mutating admission webhook with `failurePolicy: Fail` crashes due to a panic. What is the immediate impact on the Kubernetes cluster?

A) The cluster automatically downgrades to Kubernetes 1.20.  
B) The API server automatically fixes the webhook's Go code.  
C) Only warning messages are printed to the logs; deployments continue normally.  
D) All API requests matching the webhook's intercept rules (e.g., all Pod creation requests) will be rejected by the API server with HTTP 500 errors, completely blocking new deployments.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** D  
**Explanation:** When `failurePolicy` is set to `Fail`, the API server treats webhook unavailability as a fatal error. Any matching request is rejected, protecting security policies at the cost of total deployment availability if the webhook fails.
</details>

---

### Question 2
Why does Kubernetes 1.30+ introduce `ValidatingAdmissionPolicy` to replace dynamic admission webhooks?

A) To force all applications to run in AWS.  
B) Because webhooks cannot be written in Python.  
C) To eliminate external network hops, TLS overhead, and the operational failure modes of webhook pods by running compiled CEL expressions directly in `kube-apiserver` memory.  
D) Because webhooks are incompatible with Docker.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** C  
**Explanation:** Dynamic webhooks suffer from network latency and catastrophic cluster outages when webhook pods crash. `ValidatingAdmissionPolicy` executes in-process using CEL, providing maximum performance and high availability.
</details>

---

### Question 3
In a multi-tenant Kubernetes cluster, two teams (Team Checkout and Team Analytics) share worker nodes. Which combination of primitives provides baseline namespace multi-tenancy?

A) Only Kubernetes RBAC.  
B) Setting node labels.  
C) NetworkPolicies (default deny cross-namespace traffic), ResourceQuotas (CPU/RAM/PVC caps), LimitRanges (default pod requests/limits), and Pod Security Standards (enforcing `restricted` profile).  
D) Giving both teams cluster-admin permissions.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** C  
**Explanation:** Multi-tenancy on shared clusters requires multi-layered defense: NetworkPolicies isolate network traffic, ResourceQuotas prevent "noisy neighbor" resource starvation, LimitRanges prevent unbound allocations, and Pod Security Standards prevent privilege escalation.
</details>

---

### Question 4
What is the core architectural difference between **vCluster** and standard Kubernetes namespaces?

A) Namespaces have higher security than separate virtual clusters.  
B) vCluster runs a dedicated, lightweight control plane (API server, etcd, controller manager) inside a pod for each tenant, giving tenants their own isolated API server, custom CRDs, and admin rights while sharing underlying host worker nodes.  
C) vCluster replaces the Linux kernel.  
D) vCluster requires physical bare-metal servers for each tenant.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** B  
**Explanation:** vCluster creates a virtual Kubernetes cluster within a namespace of the host cluster. Tenants interact with their own dedicated virtual API server and can install custom CRDs, namespaces, and RBAC without impacting the host cluster or other tenants.
</details>

---

### Question 5
When performing an in-place rolling upgrade of worker nodes in an EKS cluster, what role does a **PodDisruptionBudget (PDB)** play?

A) It encrypts node hard drives.  
B) It automatically purchases AWS Spot instances.  
C) It specifies the minimum number or percentage of pods that must remain available during voluntary disruptions (`minAvailable` or `maxUnavailable`), preventing `kubectl drain` from evicting all replicas simultaneously.  
D) It reboots the control plane.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** C  
**Explanation:** PDBs protect application availability during planned maintenance (such as node drains for AMI upgrades). The eviction subresource honors PDB constraints; if evicting a pod would violate the PDB, the drain operation pauses until new replicas are healthy.
</details>

---

### Question 6
What is the primary operational trade-off between **Blue/Green Cluster Upgrades** and **In-Place Rolling Node Upgrades**?

A) Rolling upgrades are impossible on Kubernetes 1.25+.  
B) Blue/Green upgrades only work on on-premises data centers.  
C) Blue/Green requires no cloud spend; rolling upgrades cost millions.  
D) Blue/Green creates a completely fresh, parallel cluster with zero impact on the running cluster and enables instant rollback, but requires routing migration and temporarily doubles cluster infrastructure costs; in-place rolling upgrades are cost-effective but risk downtime if stateful components fail.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** D  
**Explanation:** Blue/Green cluster upgrades provide maximum safety: if an upgrade breaks network CNI or ingress, traffic simply routes back to the old cluster. However, running dual clusters temporarily doubles infrastructure spend and requires external DNS/traffic management.
</details>

---

### Question 7
How does a mutating webhook differ from a validating webhook in execution order within `kube-apiserver`?

A) Mutating webhooks execute before Schema Validation and Validating webhooks; changes made by mutating webhooks are subsequently validated by validating webhooks.  
B) Validating webhooks run first; mutating webhooks run last.  
C) Mutating webhooks only run during node reboots.  
D) They execute concurrently in parallel threads.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** A  
**Explanation:** The API server runs Mutating webhooks first to allow defaults or sidecars to be injected. The modified object then passes through Schema Validation and Validating webhooks to ensure the final mutated object complies with all security policies.
</details>

---

### Question 8
What is the purpose of Cluster API (CAPI) in enterprise fleet management?

A) It monitors CPU temperatures in physical data centers.  
B) It generates marketing websites for platforms.  
C) It replaces `kubectl` with an AI assistant.  
D) It treats Kubernetes clusters as declarative custom resources (`kind: Cluster`, `kind: MachineDeployment`), enabling platform teams to provision, upgrade, and operate fleets of clusters across clouds using Kubernetes-native tooling.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** D  
**Explanation:** Cluster API brings declarative GitOps management to cluster infrastructure itself: creating or upgrading an EKS, GKE, or bare-metal cluster is achieved simply by applying a Kubernetes CRD to a management cluster.
</details>

---

### Question 9
In Kubernetes Pod Security Standards, what does the **`restricted`** profile enforce?

A) It prevents pods from accessing external websites.  
B) It enforces current pod hardening best practices: non-root execution, dropping all capabilities except `NET_BIND_SERVICE`, read-only root filesystems, and forbidding privilege escalation.  
C) It limits pod runtime to 1 hour.  
D) It disables all container logging.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** B  
**Explanation:** The `restricted` profile represents the highest level of security isolation in Kubernetes Pod Security Admission, strictly prohibiting privileged containers, root execution, host namespaces, and unnecessary Linux capabilities.
</details>

---

### Question 10
Why is sharing a single Kubernetes cluster between un-isolated, mutually hostile untrusted external tenants considered dangerous?

A) Containers share the host Linux kernel; a container escape exploit (kernel zero-day) or shared control plane exhaustion allows an attacker to compromise the entire host node and access co-located tenant workloads.  
B) The Kubernetes API server can only support 10 users.  
C) Kubernetes worker nodes lack CPU caches.  
D) Kubernetes does not support TLS encryption.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** A  
**Explanation:** Containers provide process-level isolation, not hardware virtualization. Shared kernel vulnerabilities, shared etcd contention, and node-level file access mean hard multi-tenancy for untrusted hostile tenants requires separate clusters or hypervisor-isolated microVM runtimes (like Kata Containers).
</details>
