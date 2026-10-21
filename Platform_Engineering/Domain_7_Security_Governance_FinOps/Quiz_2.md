# PE Domain 7: Security, Governance & FinOps — Scenario Quiz 2

Test your knowledge on Zero-Trust Service Mesh, Kyverno guardrails, and SOC 2 compliance automation.

---

### Question 1
What does configuring **`mtls: mode: STRICT`** in an Istio `PeerAuthentication` policy enforce across a Kubernetes namespace?

A) It blocks all outbound internet access.  
B) It encrypts all network packets with DES-56.  
C) It enforces password logins for all curl requests.  
D) It strictly requires that all incoming TCP/HTTP connections to workloads in that namespace must use mutual TLS (mTLS); any unencrypted plaintext connections from outside or inside the cluster are rejected immediately.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** D  
**Explanation:** `STRICT` mode eliminates plaintext communication. Workloads only accept connections that successfully complete an mTLS handshake presenting a valid SPIFFE x509 certificate issued by Istio's Citadel CA. In contrast, `PERMISSIVE` mode accepts both plaintext and mTLS.
</details>

---

### Question 2
Why is **Kyverno** frequently preferred over **OPA Gatekeeper** for platform engineering teams standardizing on Kubernetes-native tooling?

A) Kyverno policies are authored in standard, declarative Kubernetes YAML, requiring no specialized domain-specific language (DSL) like Rego, and supports native resource mutation and image verification out of the box.  
B) Kyverno was written by Microsoft.  
C) Kyverno disables all Linux capabilities.  
D) OPA Gatekeeper cannot run on Kubernetes.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** A  
**Explanation:** Gatekeeper requires writing complex Rego code. Kyverno uses pure Kubernetes YAML patterns that any platform engineer can read and author immediately, significantly reducing policy authoring friction and maintenance overhead.
</details>

---

### Question 3
An engineer attempts to deploy a pod with the following manifest:
```yaml
spec:
  containers:
    - name: agent
      image: alpine:latest
      securityContext:
        privileged: true
```
If an admission controller (Kyverno/OPA) enforces standard **Pod Security Standards (restricted)**, what happens?

A) The node running the pod reboots.  
B) The API server rejects the pod creation request with an HTTP 403 / 422 error, printing an explicit policy violation message indicating that privileged containers are forbidden.  
C) The image is deleted from Docker Hub.  
D) The pod is created and granted root permissions.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** B  
**Explanation:** Pod Security Standards (`restricted`) strictly disallow `privileged: true`. The admission controller intercepts the request during the validation phase and rejects it, preventing container escape vectors from being introduced.
</details>

---

### Question 4
In a **Zero-Trust Network Architecture**, why is relying on IP-based firewall rules (Security Groups, CIDR blocks) considered insufficient inside a multi-tenant Kubernetes cluster?

A) Pod IP addresses in Kubernetes are ephemeral, constantly reallocated, and shared across nodes; relying on IPs cannot establish workload identity, whereas cryptographic identities (SPIFFE/mTLS) authenticate the exact service and namespace regardless of IP address.  
B) IP addresses cannot be routed in Kubernetes.  
C) Firewall rules do not support IPv4.  
D) Cloud providers charge per IP address rule.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** A  
**Explanation:** Pods churn and reschedule continuously, meaning an IP assigned to `checkout` today may belong to an untrusted batch worker tomorrow. Zero-Trust verifies identity cryptographically at L7 via mTLS certificates rather than trusting transient L3 IP addresses.
</details>

---

### Question 5
What is the purpose of an **Istio Ambient Mesh (Sidecarless)** architecture compared to classic Istio?

A) It splits service mesh processing into two layers: an L4 shared node proxy (`ztunnel`) for transparent zero-trust mTLS, and an optional L7 waypoint proxy for HTTP routing, eliminating the massive CPU and memory footprint of running an Envoy sidecar container in every single pod.  
B) It disables encryption entirely.  
C) It only runs on Windows Server.  
D) It replaces Kubernetes with virtual machines.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** A  
**Explanation:** Classic sidecar meshes suffer from high operational cost: 1,000 pods require 1,000 Envoy sidecars consuming gigabytes of RAM. Ambient mesh provides transparent L4 mTLS at the node level via `ztunnel`, dramatically reducing resource overhead and simplifying pod lifecycle management.
</details>

---

### Question 6
During a SOC 2 Type II audit, what is the primary difference between **Type I** and **Type II** compliance?

A) Type I is for small startups; Type II is only for Fortune 500 banks.  
B) Type I audits financial statements; Type II audits code syntax.  
C) Type I requires hardware encryption; Type II does not.  
D) Type I evaluates whether security controls are designed properly at a specific point in time (snapshot); Type II evaluates the operational effectiveness of those security controls over an extended period (typically 6 to 12 months).  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** D  
**Explanation:** A Type I report tests control design on a single date. A Type II report evaluates continuous operational adherence over months, proving through audit logs and telemetry that policies were consistently enforced without unauthorized exceptions.
</details>

---

### Question 7
What is the principle of **Just-In-Time (JIT) Privileged Access** in modern platform governance?

A) Firing developers who make mistakes on time.  
B) Scheduling deployments only during lunch hours.  
C) Eliminating permanent, standing administrative credentials; engineers request temporary elevated privileges (e.g., for production incident triage) via an automated workflow that expires automatically after a short duration and records all activity.  
D) Giving all developers root access immediately upon hire.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** C  
**Explanation:** Standing administrative privileges represent high-risk attack surfaces. JIT access grants temporary access (e.g., 2 hours) upon peer approval, logging all session actions for audit trails and automatically revoking access upon expiration.
</details>

---

### Question 8
What does the Kyverno `validationFailureAction: Enforce` setting specify?

A) The policy restarts the Kubernetes API server.  
B) The policy actively blocks and rejects any non-compliant API request that violates the policy rules.  
C) The policy automatically edits the developer's source code.  
D) The policy only generates warning log messages without blocking deployments.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** B  
**Explanation:** In `Enforce` mode, non-compliant resources are strictly rejected. In `Audit` mode, the resource is admitted, but a policy report entry is created to notify platform engineers of the violation without interrupting deployments.
</details>

---

### Question 9
In Kubernetes Pod Security, why is setting `readOnlyRootFilesystem: true` considered a critical defense-in-depth measure?

A) It prevents the pod from reading database records.  
B) It makes the pod boot in 1 millisecond.  
C) It disables container logs.  
D) It prevents attackers who achieve remote code execution (RCE) from downloading, compiling, or executing malicious binaries, scripts, or rootkits into the container's filesystem.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** D  
**Explanation:** Attackers exploiting vulnerabilities (e.g., command injection) typically attempt to download malware or tools (like `nmap` or crypto-miners) into `/tmp` or `/bin`. A read-only root filesystem blocks writes to disk, crippling the attacker's ability to establish persistence.
</details>

---

### Question 10
What role does **AWS CloudTrail** play in continuous compliance automation?

A) It compiles Java code.  
B) It compresses S3 buckets.  
C) It records an immutable, cryptographically verifiable audit log of every API call made across an AWS account (who called what, from where, and when), providing the definitive evidence log required for SOC 2 and ISO 27001 compliance.  
D) It routes web traffic across AWS regions.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** C  
**Explanation:** CloudTrail captures every administrative action, console login, and API invocation. Compliance platforms ingest CloudTrail logs to prove that security boundaries and change management procedures were honored.
</details>
