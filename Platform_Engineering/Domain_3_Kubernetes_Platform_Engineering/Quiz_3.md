# PE Domain 3: Kubernetes Platform Engineering — Scenario Quiz 3

Test your mastery of advanced autoscaling (KEDA, Karpenter) and the Gateway API.

---

### Question 1
How does **Karpenter** achieve significantly faster node provisioning times (30–45 seconds) compared to the traditional Kubernetes **Cluster Autoscaler** (2–4 minutes)?

A) Karpenter bypasses AWS Auto Scaling Groups (ASGs) entirely; it evaluates unscheduled pod specifications directly and calls the AWS EC2 Fleet API to launch the exact matching instance type in a single API call.  
B) Karpenter compiles the Linux kernel into memory.  
C) Karpenter keeps hundreds of idle EC2 instances running constantly.  
D) Karpenter runs all pods on Raspberry Pis.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** A  
**Explanation:** Cluster Autoscaler operates by incrementally adjusting the desired capacity of AWS Auto Scaling Groups (ASGs), waiting for ASG lifecycle hooks and EC2 launch processes. Karpenter eliminates ASG abstractions: it evaluates unscheduled pod constraints directly and launches optimal compute via EC2 Fleet API in seconds.
</details>

---

### Question 2
What is Karpenter's **"Node Consolidation"** capability?

A) Compressing container images on disk.  
B) Deleting all pods with zero traffic.  
C) An automated optimization loop that continuously calculates whether pods on underutilized nodes can be rescheduled onto cheaper or fewer instances, automatically cordoning, draining, and terminating redundant nodes to minimize cloud spend.  
D) Merging multiple AWS accounts into one billing profile.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** C  
**Explanation:** Consolidation prevents cloud waste. Karpenter continuously calculates if the cluster can run on a more cost-effective instance mix. When savings are identified, Karpenter spins up the smaller replacement, drains workloads, and terminates the expensive node.
</details>

---

### Question 3
A message-processing worker pod consumes jobs from an Apache Kafka topic. The pod consumes zero CPU when idle, but when message spikes occur, queue lag increases. Why is traditional Horizontal Pod Autoscaler (HPA) CPU-based scaling ineffective for this workload, and how does **KEDA** solve it?

A) CPU utilization lags behind queue depth; by the time workers hit 80% CPU, thousands of messages are backed up. KEDA scales directly on external Kafka consumer lag metrics, and can scale replicas down to 0 when the topic is empty.  
B) KEDA rewrites the Kafka topic in Go.  
C) HPA cannot scale pods above 5 replicas.  
D) HPA does not support Linux containers.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** A  
**Explanation:** Metric lag is a common scaling failure. A consumer queue can be 100,000 messages behind while worker CPU is still low. KEDA exposes event-driven metrics directly to the HPA, allowing proactive scaling based on queue depth and enabling scale-to-zero.
</details>

---

### Question 4
In the Kubernetes **Gateway API**, what is the purpose of the **`ReferenceGrant`** resource?

A) It increases the cluster memory limit.  
B) It explicitly authorizes cross-namespace references, allowing a Gateway in the `platform-system` namespace to safely bind to an HTTPRoute or TLS Secret in the `payments` namespace without security risks.  
C) It reloads CoreDNS configurations.  
D) It grants academic research licenses.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** B  
**Explanation:** In legacy Ingress, any namespace could reference TLS secrets or services across namespaces, creating severe privilege escalation vulnerabilities. `ReferenceGrant` requires the owning namespace to explicitly grant permission before cross-namespace bindings are accepted.
</details>

---

### Question 5
Which Gateway API resource is owned and managed by the **Product / Application Developer** persona?

A) `Gateway`  
B) `GatewayClass`  
C) `NodePool`  
D) `HTTPRoute`  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** D  
**Explanation:** The Gateway API is role-oriented: Infrastructure Providers manage `GatewayClass`, Platform Administrators manage `Gateway` (listeners, ports, IP, certs), and Application Developers manage `HTTPRoute` (path routing, rules, canary splits).
</details>

---

### Question 6
An engineer wants to execute a canary deployment routing 80% of traffic to `service-v1` and 20% to `service-v2`. Using Gateway API, how is this configured?

A) In the `HTTPRoute.spec.rules.backendRefs` list, specify both services with `weight: 80` and `weight: 20`.  
B) By editing `/etc/hosts` on all worker nodes.  
C) By deploying two separate GatewayClasses.  
D) By killing 20% of the pods in `service-v1`.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** A  
**Explanation:** Gateway API has first-class native support for weighted traffic splitting: multiple `backendRefs` can be declared under a single rule with explicit integer weights, eliminating proprietary ingress annotations.
</details>

---

### Question 7
How does Karpenter handle AWS Spot instance terminations?

A) Karpenter converts Spot instances into On-Demand instances without rebooting.  
B) Karpenter listens to AWS 2-minute Spot Interruption Warnings via SQS/EventBridge, immediately cordoning the node, spinning up a replacement instance, and draining pods before the physical instance is terminated by AWS.  
C) Karpenter blocks AWS from ever terminating Spot instances.  
D) Karpenter crashes when Spot instances terminate.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** B  
**Explanation:** AWS issues a 2-minute warning before reclaiming Spot instances. Karpenter integrates natively with EventBridge/SQS to receive this event instantly, executing preemptive provisioning of replacement capacity and graceful pod draining.
</details>

---

### Question 8
What is the formula used by the Kubernetes Horizontal Pod Autoscaler (HPA) to determine desired replicas?

A) `Desired = Current Replicas × 2`  
B) `Desired = ceil[ Current Replicas × ( Current Metric Value / Target Metric Value ) ]`  
C) `Desired = Total Nodes + Total Namespaces`  
D) `Desired = (CPU Limits / RAM Requests)`  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** B  
**Explanation:** The HPA calculates target scale ratio: `ceil[ Current Replicas * (CurrentMetric / TargetMetric) ]`. If current usage is 80m and target is 40m with 2 replicas: `ceil[ 2 * (80/40) ] = 4 replicas`.
</details>

---

### Question 9
Why does the Gateway API deprecate and supersede the legacy Kubernetes `Ingress` resource?

A) Ingress cannot run on cloud providers.  
B) Ingress only supported UDP traffic.  
C) Ingress was written in Java.  
D) Ingress combined infrastructure and routing into a single monolithic schema, lacked native support for canary weights, header matching, and cross-namespace routing, forcing vendors to rely on fragmented, non-portable annotations.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** D  
**Explanation:** Legacy `Ingress` suffered from architectural limitations: it forced platform teams to use dozens of vendor-specific annotations (`nginx.ingress.kubernetes.io/...`) for basic features like rewrites and canaries. Gateway API provides clean, role-oriented, expressive primitives.
</details>

---

### Question 10
When using **Vertical Pod Autoscaler (VPA)** in Kubernetes, which mode provides automated recommendations without restarting or modifying running pods?

A) `Recreate` mode  
B) `Auto` mode  
C) `Off` mode  
D) `Initial` mode  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** C  
**Explanation:** In `Off` mode, VPA calculates and records recommended CPU and memory requests in its `.status.recommendation` field based on historical usage analysis, without modifying or restarting the running pods, making it ideal for non-disruptive capacity planning.
</details>
