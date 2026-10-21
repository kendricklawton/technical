# PE Domain 7: Security, Governance & FinOps — Scenario Quiz 3

Test your mastery of Cloud FinOps, Kubernetes cost allocation, and resource optimization.

---

### Question 1
An engineering director reviews the company's AWS EKS bill and notices that the compute cost for a 100-node cluster is $30,000/month. However, Prometheus metrics reveal that average cluster CPU utilization is only 12%. What is the primary cause of this massive financial waste?

A) Kubernetes requires 80% CPU overhead just to run the kernel.  
B) Spot instances were accidentally purchased.  
C) The "Idle Capacity Gap": developers configured excessively high CPU `requests` (e.g., requesting 4 cores when the app only uses 0.2 cores), causing `kube-scheduler` to pack fewer pods per node and forcing autoscalers to provision dozens of unnecessary nodes.  
D) AWS is overcharging for network egress.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** C  
**Explanation:** In Kubernetes, nodes are provisioned based on aggregate **CPU Requests**, not real CPU utilization. When teams over-request resources, nodes appear full to the scheduler even though they are mostly idle. Sizing requests to match actual p95 usage eliminates this waste.
</details>

---

### Question 2
What is the primary difference between **CPU Requests** and **CPU Limits** from a FinOps perspective?

A) Requests can only be set in integers.  
B) You are billed for CPU limits, not requests.  
C) CPU Requests dictate the physical node capacity that must be purchased and provisioned by autoscalers (direct financial cost); CPU Limits merely set the ceiling at which the Linux CFS throttles execution.  
D) CPU Limits determine cluster node counts.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** C  
**Explanation:** Financial spend is driven by Requests: `kube-scheduler` guarantees requests, so a cluster with 1,000 cores of aggregate requests requires 1,000 cores of EC2 instances. Limits do not allocate capacity; they simply constrain runtime bursts.
</details>

---

### Question 3
How does **OpenCost / Kubecost** calculate real-time cost allocation per Kubernetes namespace or pod?

A) By dividing the total AWS bill equally among all namespaces.  
B) By combining real-time node instance hourly pricing (from cloud billing APIs or Spot pricing feeds) with each container's CPU, memory, and storage resource requests and usage, proportionally allocating node costs down to individual pods and namespaces.  
C) By counting lines of code committed to Git.  
D) By inspecting application HTTP response codes.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** B  
**Explanation:** OpenCost maps cloud infrastructure pricing to Kubernetes primitives. If a node costs $1.00/hour and Pod A reserves 50% of the node's CPU and memory, OpenCost allocates $0.50/hour to Pod A's owning team.
</details>

---

### Question 4
Under the **FinOps Framework**, what are the three iterative phases of cloud financial management?

A) Build, Measure, Learn  
B) Inform, Optimize, Operate  
C) Plan, Code, Test  
D) Buy, Sell, Hold  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** B  
**Explanation:** FinOps operates continuously through:
1. *Inform:* Provide visibility, allocation, and showback/chargeback.
2. *Optimize:* Eliminate waste (rightsizing, Karpenter consolidation) and optimize rates (Savings Plans, Spot).
3. *Operate:* Embed cost metrics into platform scorecards and business unit KPIs.
</details>

---

### Question 5
How does **Karpenter Node Consolidation** directly reduce Kubernetes infrastructure costs?

A) It compresses container filesystems on disk.  
B) It lowers developer salaries.  
C) It continuously monitors cluster resource utilization; when workloads on underutilized nodes can fit onto fewer or smaller instances, it provisions the cheaper replacement, drains the old nodes, and terminates them automatically.  
D) It cancels all AWS contracts.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** C  
**Explanation:** Traditional autoscalers only scale down when a node is 100% empty. If 5 nodes each run 1 tiny pod, Cluster Autoscaler leaves all 5 nodes running. Karpenter consolidates those 5 pods onto a single node and terminates the 4 redundant instances.
</details>

---

### Question 6
What is the primary operational risk of utilizing **AWS Spot Instances** for production Kubernetes workloads, and how is it mitigated?

A) Spot instances cannot run Linux.  
B) Spot instances do not support network traffic.  
C) Spot instances cost 5x more than On-Demand instances.  
D) AWS can reclaim Spot instances with only a 2-minute notice during capacity shortages; mitigated by deploying diverse instance types across multiple AZs, maintaining PodDisruptionBudgets, and using Karpenter to handle termination warnings preemptively.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** D  
**Explanation:** Spot instances offer up to 70–90% savings over On-Demand, but can be reclaimed at any time. Resilient platforms handle this by diversifying across dozens of instance types, listening to 2-minute interruption warnings via SQS, and restricting Spot to stateless, fault-tolerant workloads.
</details>

---

### Question 7
In cloud FinOps, what is **Unit Economics**?

A) The hourly wage of a platform engineer.  
B) The price of a single server rack.  
C) The currency exchange rate between USD and Euros.  
D) Translating raw cloud costs into meaningful business metrics (e.g., "Infrastructure cost per checkout transaction", "Cost per active user", or "Cost per gigabyte processed") to measure whether architectural scaling is economically efficient.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** D  
**Explanation:** Looking at total cloud spend alone is misleading: a company's bill might double because its customer base grew 10x. Unit economics measures efficiency: if gross revenue grew 10x while infrastructure cost per transaction dropped 20%, the platform is highly successful.
</details>

---

### Question 8
What tool can be deployed in Kubernetes to analyze historical resource utilization and provide automated recommendations for rightsizing CPU and memory requests?

A) Vertical Pod Autoscaler (VPA) in `recommendation` mode (or Goldilocks)  
B) kube-proxy  
C) Docker compose  
D) CoreDNS  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** A  
**Explanation:** VPA in `Off` mode continuously analyzes historical resource metrics and writes rightsizing recommendations directly to its status field, allowing platform teams (or Goldilocks dashboards) to identify over-provisioned pods without restarting them.
</details>

---

### Question 9
What is the difference between a **Showback** model and a **Chargeback** model in internal platform financial governance?

A) Showback provides financial visibility and reports cloud spend to team leads without cross-charging internal budgets (raising awareness); Chargeback actually bills and transfers financial funds directly from product team departmental cost centers.  
B) Showback is for hardware; Chargeback is for software.  
C) Showback is illegal under GAAP accounting rules.  
D) Chargeback does not require cost allocation tags.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** A  
**Explanation:** Showback informs teams of their cost footprint to encourage voluntary optimization. Chargeback enforces financial accountability by directly debiting product team profit-and-loss (P&L) budgets based on their measured consumption.
</details>

---

### Question 10
Why is setting `requests.cpu = limits.cpu` (Guaranteed QoS) on batch processing or non-latency-critical microservices considered a FinOps anti-pattern?

A) It causes the pod to crash immediately.  
B) It prevents the pod from bursting to consume idle CPU cycles available on the node, forcing teams to set high requests that lock up capacity and prevent cost-effective node bin-packing.  
C) It is unsupported by Kubernetes 1.30.  
D) It increases the cost of AWS S3 storage.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** B  
**Explanation:** Equal requests and limits (Guaranteed QoS) provide predictability for latency-critical services, but for general workloads, it prevents opportunistic bursting into unallocated node CPU cycles. Setting modest requests with higher (or unbounded) limits allows denser bin-packing.
</details>
