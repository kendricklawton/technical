# PE Domain 7: Security, Governance & FinOps — 25 Core Interview Questions

Master these questions across Technical Screening, Security/FinOps Deep Dives, and Staff Architecture rounds.

---

## Tier 1: Core / Screening Questions (Questions 1–10)

### Q1: Why are static IAM access keys an anti-pattern in containerized platforms?
- **Short Answer:** Static keys do not expire, can be leaked via Git, container image layers, or environment variables, cannot be easily rotated without downtime, and lack fine-grained per-pod audit trails.
- **Deep Dive:** Workload identity (AWS EKS Pod Identity / IRSA, SPIRE) eliminates static credentials: the container runtime presents an ephemeral cryptographic token to AWS STS, receiving temporary credentials valid for 1 hour that rotate automatically.
- **Trade-offs & Realities:** Static keys are easy for developers to copy-paste during prototyping, which is why platform teams must provide frictionless workload identity in Golden Path templates.
- **Follow-up Probes:** "How do you detect leaked access keys across your organization?" (Tools like TruffleHog, GitGuardian, and AWS IAM credential reports).

### Q2: What is the difference between External Secrets Operator (ESO) and SealedSecrets?
- **Short Answer:** ESO fetches secrets from an external cloud provider (Vault, AWS Secrets Manager, GCP) and creates native Kubernetes Secrets. SealedSecrets encrypts secret values using a cluster public key and decrypts them in-cluster, storing encrypted ciphertext directly in Git.
- **Deep Dive:** ESO is the modern enterprise standard because it decouples secret lifecycle from Git, supports dynamic secret rotation, and integrates with corporate identity providers. SealedSecrets couples secrets to a specific cluster's private key, making multi-cluster disaster recovery difficult.
- **Trade-offs & Realities:** ESO introduces external API dependencies; SealedSecrets works completely offline inside the cluster.
- **Follow-up Probes:** "What happens if you lose the private key in a SealedSecrets cluster?" (All encrypted secrets in Git become permanently unrecoverable).

### Q3: What is mTLS, and how does it implement Zero-Trust networking?
- **Short Answer:** Mutual TLS authenticates both the client and the server cryptographically using x509 certificates before establishing an encrypted session, ensuring that all communications inside the cluster perimeter are encrypted and identity-verified.
- **Deep Dive:** In perimeter security, anything inside the firewall is trusted. In Zero-Trust, the internal network is assumed to be compromised. Workloads verify each other's SPIFFE identities at L7, preventing lateral movement by attackers who breach a single container.
- **Trade-offs & Realities:** Traditional sidecar mTLS introduces CPU and memory overhead; modern ambient/eBPF meshes provide transparent mTLS at the node level.
- **Follow-up Probes:** "What component issues mTLS certificates in Istio?" (`istiod` / Citadel).

### Q4: Explain the difference between Kyverno and OPA Gatekeeper.
- **Short Answer:** Kyverno uses declarative, native Kubernetes YAML and supports validation, mutation, and image signing. OPA Gatekeeper uses the Rego query language and is a general-purpose policy engine across clouds and systems.
- **Deep Dive:** Kyverno is easier for platform teams to adopt because it requires no new programming language. Gatekeeper is powerful for complex cross-system policies and enterprise-wide Rego standardization.
- **Trade-offs & Realities:** Gatekeeper requires learning Rego; Kyverno is specialized strictly for Kubernetes.
- **Follow-up Probes:** "Can Kyverno mutate incoming pods?" (Yes, via `mutate` rules that automatically inject labels, annotations, or security contexts).

### Q5: What is the FinOps lifecycle, and why is it important?
- **Short Answer:** Inform (visibility and allocation), Optimize (rate and usage reduction), and Operate (continuous tracking and KPI integration).
- **Deep Dive:** Cloud spend is variable and decentralized: every engineer's code change impacts the bill. FinOps creates financial accountability, shifting cost awareness to the teams making architectural decisions.
- **Trade-offs & Realities:** FinOps is not about cutting costs blindly; it is about maximizing business value and understanding unit economics.
- **Follow-up Probes:** "What is Showback vs Chargeback?" (Showback reports costs to teams for awareness; Chargeback debits costs from their actual department budget).

### Q6: What is the "Idle Capacity Gap" in Kubernetes cost optimization?
- **Short Answer:** The financial waste resulting from the difference between the CPU/RAM capacity **requested** by pods (which dictates node provisioning) and the actual CPU/RAM **utilized** by running workloads.
- **Deep Dive:** Autoscalers provision nodes based on requests. If pods request 4 cores but only use 0.2 cores, 3.8 cores sit idle on the node, wasting cloud spend.
- **Trade-offs & Realities:** Rightsizing requests reduces costs, but setting requests too close to average usage risks throttling or OOM kills during traffic spikes.
- **Follow-up Probes:** "What tool provides rightsizing recommendations?" (Vertical Pod Autoscaler or Goldilocks).

### Q7: What is the difference between SOC 2 Type I and Type II?
- **Short Answer:** Type I audits security control design at a single point in time. Type II audits the operational effectiveness of those controls over an extended evaluation period (6–12 months).
- **Deep Dive:** Enterprise B2B customers demand Type II reports because they provide proof through immutable audit logs (CloudTrail, Git history) that security policies were continuously enforced.
- **Trade-offs & Realities:** Type II audits require substantial continuous evidence gathering and strict change management discipline.
- **Follow-up Probes:** "What does a SOC 2 auditor check regarding Git access?" (Proof that all production code was peer-reviewed and that branch protection rules were enforced).

### Q8: What does `securityContext.runAsNonRoot: true` accomplish?
- **Short Answer:** It instructs the container runtime to verify that the container process UID is not 0 (root); if the image specifies UID 0, the pod is rejected and fails to start.
- **Deep Dive:** Running as root inside a container grants elevated privileges within the namespace. In the event of a container breakout (kernel vulnerability), a root container process can compromise the host kernel.
- **Trade-offs & Realities:** Legacy images hardcoded to root must be refactored to specify a non-root user (e.g., `USER 65532:65532`).
- **Follow-up Probes:** "What is the difference between `runAsNonRoot` and `runAsUser`?" (`runAsNonRoot` is a boolean check; `runAsUser` specifies the exact numeric UID).

### Q9: What is OpenCost?
- **Short Answer:** An open-source, vendor-neutral CNCF project that measures real-time Kubernetes cost allocation down to the pod, namespace, deployment, and label level.
- **Deep Dive:** OpenCost combines live cloud provider pricing APIs (AWS, Azure, GCP) with Kubernetes resource requests and utilization metrics to calculate hourly infrastructure spend per microservice.
- **Trade-offs & Realities:** OpenCost provides real-time allocation; reconciling it with final monthly consolidated cloud bills requires factoring in enterprise discounts and EDP commitments.
- **Follow-up Probes:** "What is the commercial distribution of OpenCost?" (Kubecost).

### Q10: How do Spot Instances reduce cloud infrastructure costs?
- **Short Answer:** By allowing cloud consumers to bid on spare, unused cloud compute capacity at discounts of up to 70–90% compared to standard On-Demand pricing.
- **Deep Dive:** Spot instances can be reclaimed by the cloud provider with a 2-minute interruption notice. They are ideal for stateless microservices, batch jobs, and CI/CD runner fleets managed by Karpenter.
- **Trade-offs & Realities:** Never run single-replica stateful databases on Spot instances without multi-region replication.
- **Follow-up Probes:** "How does Karpenter handle Spot interruptions?" (It listens to AWS SQS termination events and cordons/drains nodes gracefully before eviction).

---

## Tier 2: Deep Dive & Mid-Senior Questions (Questions 11–20)

### Q11: Walk through the internal token exchange flow of AWS EKS Pod Identity.
- **Short Answer:** The container runtime mounts an AWS container credentials URI; the AWS SDK queries the local EKS Pod Identity agent; the agent invokes the EKS Auth API to assume the IAM role and returns temporary credentials to the pod.
- **Deep Dive:**
  1. EKS mutating webhook injects `AWS_CONTAINER_CREDENTIALS_FULL_URI` pointing to link-local IP `169.254.170.23` and a signed JWT service account token.
  2. The AWS SDK in the application sends the token to the local node agent.
  3. The local Pod Identity agent calls the EKS Auth API (`AssumeRoleForPodIdentity`).
  4. EKS validates the pod's identity and returns temporary STS credentials.
- **Trade-offs & Realities:** Faster and more scalable than IRSA; eliminates OIDC discovery endpoint management.
- **Follow-up Probes:** "Does EKS Pod Identity require modifying application code?" (No; standard modern AWS SDKs support the container credentials endpoint natively).

### Q12: How do you design an automated, least-privilege secrets rotation architecture using HashiCorp Vault?
- **Short Answer:** Leverage Vault Dynamic Secrets Engines. Applications request temporary database credentials via Vault API; Vault generates a unique database role with a 1-hour lease and automatically drops the role upon expiration.
- **Deep Dive:** Applications renew their lease periodically. If a compromised credential is leaked, it expires within 60 minutes automatically. Platform teams can revoke all active credentials for a compromised service with a single API call (`vault lease revoke -prefix database/creds/payments`).
- **Trade-offs & Realities:** Requires applications to integrate with Vault SDK or use a sidecar agent/ESO to refresh secrets.
- **Follow-up Probes:** "What is the difference between static secret rotation and dynamic secrets?" (Static rotation modifies a shared password in place; dynamic secrets create and destroy unique credentials per caller).

### Q13: Explain how Istio AuthorizationPolicies enforce L7 Zero-Trust access control.
- **Short Answer:** `AuthorizationPolicy` resources define fine-grained rules evaluating the client's cryptographically authenticated SPIFFE identity (`source.principals`), HTTP paths, and HTTP methods.
- **Deep Dive:** Example:
  ```yaml
  apiVersion: security.istio.io/v1beta1
  kind: AuthorizationPolicy
  metadata:
    name: allow-billing
    namespace: payments
  spec:
    selector:
      matchLabels:
        app: payments-service
    action: ALLOW
    rules:
      - from:
          - source:
              principals: ["cluster.local/ns/checkout/sa/checkout-sa"]
        to:
          - operation:
              methods: ["POST"]
              paths: ["/v1/charge"]
  ```
  Only pods authenticated with the `checkout-sa` identity can execute `POST /v1/charge`. All other requests receive HTTP 403 Forbidden.
- **Trade-offs & Realities:** Requires mTLS `STRICT` mode so client principals are cryptographically verified.
- **Follow-up Probes:** "What happens if no `AuthorizationPolicy` is defined in a namespace?" (By default, all authenticated traffic is allowed).

### Q14: How do you implement Just-In-Time (JIT) privileged access in an enterprise Kubernetes fleet?
- **Short Answer:** Eliminate standing cluster-admin bindings. Engineers request temporary access via Slack/CLI; an automated approval workflow evaluates the request, binds a temporary `ClusterRoleBinding` with a 2-hour TTL, and records all session audit logs.
- **Deep Dive:** Tools like Teleport, Okta Access Requests, or custom Kubernetes controllers integrate with Slack:
  1. Engineer runs `tsh request --roles admin --reason "INC-104 triage"`.
  2. PagerDuty on-call lead approves via Slack button.
  3. Short-lived x509 client certificate issued, expiring in 120 minutes.
  4. Kubernetes API server audit logs every command executed by the engineer to CloudWatch/S3 for SOC 2 compliance.
- **Trade-offs & Realities:** Adds a 2-minute friction delay during emergencies; mitigated by pre-authorized break-glass roles for on-call leads.
- **Follow-up Probes:** "What is Break-Glass access?" (A pre-configured, heavily audited emergency path allowing immediate root access when central identity providers fail).

### Q15: How do you design an automated Kyverno policy testing pipeline in CI?
- **Short Answer:** Use the `kyverno` CLI in GitHub Actions to validate policies and test them against sample Kubernetes resource manifests using declarative test files before deploying policies to the cluster.
- **Deep Dive:** Kyverno CLI command: `kyverno test ./policies`. Test files (`kyverno-test.yaml`) declare expected results:
  ```yaml
  apiVersion: cli.kyverno.io/v1alpha1
  kind: Test
  metadata:
    name: test-disallow-root
  policies:
    - disallow-root.yaml
  resources:
    - bad-pod.yaml
  results:
    - policy: disallow-root
      rule: autogen-check-runasnonroot
      resource: bad-pod
      result: fail
  ```
- **Trade-offs & Realities:** Prevents deploying buggy admission policies that accidentally block all cluster deployments.
- **Follow-up Probes:** "What is Kyverno `mutate` rule testing?" (Testing that incoming manifests are correctly modified with default labels/contexts).

### Q16: How do you calculate Unit Economics for a cloud-native platform?
- **Short Answer:** Divide total allocated infrastructure cost by a core business volume metric over the same time window:
  $$\text{Unit Cost} = \frac{\text{Total Cloud & Platform Spend}}{\text{Business Volume (e.g., Transactions, Active Users)}}$$
- **Deep Dive:** Example: A payment platform spends $50,000/month on AWS and processes 5,000,000 transactions. Unit cost = $0.01 per transaction. If next quarter spend rises to $75,000 but transactions rise to 10,000,000, unit cost drops to $0.0075, proving the platform gained economic efficiency at scale.
- **Trade-offs & Realities:** Selecting the wrong unit metric (e.g., lines of code) distorts architectural priorities; must align with executive business KPIs.
- **Follow-up Probes:** "How do you allocate shared platform costs (e.g., Kubernetes control planes, observability) to unit cost?" (Distribute proportionally based on tenant compute usage).

### Q17: What are the security risks of mounting the host Docker socket (`/var/run/docker.sock`) inside a container?
- **Short Answer:** It grants the container full root control over the host Docker daemon, allowing anyone inside the container to escape, spin up privileged containers, mount the host root filesystem, and take complete control of the host node.
- **Deep Dive:** Communicating with the Docker daemon API bypasses all Kubernetes namespace, cgroup, and RBAC restrictions. An unprivileged user with Docker socket access is effectively root on the host.
- **Trade-offs & Realities:** Historically used for "Docker-in-Docker" image builds; modern platforms use daemonless, rootless builders instead (rootless BuildKit, Buildah). Kaniko was the common choice, but Google archived it in June 2025, and only community forks are still maintained.
- **Follow-up Probes:** "How do you block Docker socket mounts in Kubernetes?" (Kyverno or ValidatingAdmissionPolicy blocking `hostPath: /var/run/docker.sock`).

### Q18: How does Karpenter optimize cost allocation compared to static AWS Auto Scaling Groups?
- **Short Answer:** Karpenter analyzes unscheduled pod requests and provisions the exact instance type and pricing model (Spot vs On-Demand) required, continuously consolidating pods onto fewer, smaller nodes as workloads terminate.
- **Deep Dive:** ASGs force homogeneous instance sizes (e.g., all `m5.large`). Karpenter selects from hundreds of instance types (e.g., mixing `c6g.xlarge`, `r6i.2xlarge`, Spot). Karpenter's consolidation engine terminates nodes that are less than 50% utilized, saving 30–50% on compute spend.
- **Trade-offs & Realities:** Consolidation evicts pods, which can cause transient disruptions if applications lack proper PDBs and graceful shutdown handlers.
- **Follow-up Probes:** "What Karpenter setting limits disruption during consolidation?" (`NodePool.spec.disruption.budgets`).

### Q19: Explain the concept and implementation of "Compliance as Code".
- **Short Answer:** Codifying compliance and regulatory controls (SOC 2, PCI-DSS) into automated scripts, policy-as-code rules, and tests that continuously verify infrastructure state in CI/CD and production.
- **Deep Dive:** Instead of an annual manual audit where engineers take screenshots of the AWS console, automated tools (Trivy, Steampipe, CloudQuery, Kyverno) continuously query cloud APIs and emit compliance reports daily.
- **Trade-offs & Realities:** Auditors must be educated on how to trust automated evidence over manual screenshots.
- **Follow-up Probes:** "Name an open-source tool for querying cloud infrastructure using SQL." (Steampipe or CloudQuery).

### Q20: How do you handle emergency security patching of a critical zero-day vulnerability (e.g., Log4Shell) across 500 microservices?
- **Short Answer:**
  1. *Identify:* Query the central Software Catalog and SBOM database to instantly list all services running the vulnerable library version.
  2. *Mitigate:* Apply a cluster-wide Kyverno mutation policy or WAF rule to block exploit payloads at the network ingress boundary.
  3. *Remediate:* Update the core Golden Path base image/template, and use automated bot PR campaigns (Renovate) to submit 500 upgrade PRs with automated testing.
  4. *Enforce:* After 48 hours, switch admission policies to `Enforce` mode, rejecting any pod running the unpatched library.
- **Trade-offs & Realities:** In extreme emergencies, platform teams may use emergency change-approval procedures to auto-merge patch PRs.
- **Follow-up Probes:** "Why is having an SBOM critical in this scenario?" (Without an SBOM, identifying affected microservices takes weeks of manual code searches).

---

## Tier 3: Staff & Principal Architect Scenarios (Questions 21–25)

### Q21: Architect an enterprise Zero-Trust security and workload identity framework for a hybrid multi-cloud platform (AWS, GCP, on-prem bare-metal).
- **Candidate Framework:**
  - *Identity Backbone:* Deploy SPIRE in a multi-region federation topology. SPIRE Server acts as the universal Certificate Authority, issuing short-lived SPIFFE SVIDs to all workloads across clouds.
  - *Data Plane mTLS:* Deploy Istio Ambient Mesh or Cilium Service Mesh. Nodes use `ztunnel` / eBPF to negotiate transparent mTLS handshakes using SPIRE SVIDs, eliminating sidecar overhead.
  - *Cloud Authentication:* Use SPIFFE JWT tokens with AWS STS and GCP Workload Identity to eliminate all static cloud credentials across all clouds.
  - *Policy & Guardrails:* Deploy Kyverno and in-tree ValidatingAdmissionPolicies enforcing Pod Security Standards (`restricted`), signed container verification (Cosign), and mandatory non-root execution.
  - *Governance & Audit:* Centralize all access through a JIT privileged access proxy (Teleport) with automated session recording and CloudTrail streaming to a dedicated security account.
- **Red Flags:** Recommending static IP whitelisting across cloud boundaries or relying on shared root credentials.

### Q22: You inherit an organization spending $500,000/month on AWS with zero cost allocation. How do you lead a FinOps transformation to cut spend by 30% in 90 days?
- **Candidate Framework:**
  - *Month 1: Visibility & Allocation (Inform)*
    - Enforce mandatory AWS Cost Allocation Tags (`Environment`, `TeamOwner`, `Service`) via Kyverno and Terraform guardrails.
    - Deploy OpenCost/Kubecost across all Kubernetes clusters.
    - Publish weekly cost showback dashboards to engineering directors.
  - *Month 2: Waste Reduction (Optimize)*
    - Deploy Karpenter with automated node consolidation, replacing rigid ASGs and eliminating idle node capacity.
    - Rightsize CPU/RAM requests on the top 20 most expensive microservices using VPA historical data.
    - Migrate stateless workloads and CI runner fleets to AWS Spot Instances (saving up to 70%).
    - Delete orphaned EBS volumes, unattached Elastic IPs, and obsolete staging RDS snapshots.
  - *Month 3: Rate Optimization & Operationalization (Operate)*
    - Analyze stable compute baseline; purchase 1-year / 3-year AWS Compute Savings Plans for predictable workloads.
    - Embed cost metrics into developer portal service scorecards.
- **Red Flags:** Demanding that developers rewrite code before tackling low-hanging infrastructure waste (over-provisioned requests, unattached disks, lack of Spot).

### Q23: How do you design an isolated Cardholder Data Environment (CDE) for PCI-DSS Level 1 compliance on a shared Kubernetes platform?
- **Candidate Framework:**
  - *Network Segmentation:* Deploy the CDE in a dedicated, isolated AWS VPC or separate EKS cluster to strictly minimize the compliance scope.
  - *Zero-Trust Ingress:* Terminate all traffic at an external L7 gateway; enforce strict Istio mTLS and `AuthorizationPolicy` allowing only encrypted tokenized payload transit.
  - *Database & Storage Encryption:* Encrypt all payment storage volumes with customer-managed KMS keys with annual automatic rotation; enforce TLS 1.3 in transit.
  - *Immutable Audit Logging:* Stream all container stdout, kube-apiserver audit logs, and network flow logs to an isolated, write-once-read-many (WORM) S3 bucket in a dedicated Security Account with Object Lock enabled.
  - *Vulnerability Management:* Enforce container image signing via Cosign, block unapproved base images via Kyverno, and run automated quarterly container vulnerability scans.
- **Red Flags:** Co-locating PCI payment databases on the same Kubernetes worker nodes as unvetted general internal apps without hypervisor-level isolation.

### Q24: What is the architectural role of eBPF in modern cloud-native security (e.g., Cilium Tetragon, Falco)?
- **Candidate Framework:**
  - *Limitation of Userspace Agents:* Traditional security scanners inspect containers statically or query logs after an attack has already succeeded.
  - *eBPF Runtime Enforcement:* eBPF attaches directly to kernel tracepoints, kprobes, and Linux Security Modules (LSM) hooks.
  - *Real-Time Behavioral Monitoring:* Tools like Tetragon monitor kernel events in real time: namespace changes, privilege escalations, binary executions in `/tmp`, and unauthorized socket creation.
  - *In-Kernel Prevention:* Unlike passive logging tools, eBPF can synchronously terminate a rogue process at the kernel level (`SIGKILL`) before a malicious system call completes, blocking zero-day exploits in microseconds.
- **Red Flags:** Confusing static image scanning with real-time kernel runtime security.

### Q25: How do you design an automated, self-service secret rotation engine for third-party API keys (e.g., Stripe, Twilio) across 200 services?
- **Candidate Framework:**
  - *External Secrets Integration:* All microservices consume secrets via Kubernetes `ExternalSecret` custom resources mapped to HashiCorp Vault.
  - *Orchestration Controller:* A centralized Platform Secret Rotator controller monitors secret expiration dates recorded in Vault metadata.
  - *Two-Phase Rotation Pattern:*
    1. *Generate New Key:* Controller calls third-party vendor API (e.g., Stripe API) to generate a secondary API key.
    2. *Inject Dual Keys:* Controller updates Vault with both `PRIMARY_KEY` and `SECONDARY_KEY`. ESO updates Kubernetes secrets.
    3. *Rolling Update:* Microservices seamlessly transition to the new key without downtime.
    4. *Revoke Old Key:* After a 24-hour verification window, controller calls third-party API to revoke the old primary key.
  - *Audit Logging:* Every rotation phase emits structured events to Slack and compliance telemetry.
- **Red Flags:** Attempting single-phase instant key replacement (causes downtime if applications make requests while keys update).
