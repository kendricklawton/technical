# PE Domain 1: Platform Foundations & IDPs — 25 Core Interview Questions

Master these questions across Screening, Technical Deep Dive, and Staff-level Architecture rounds.

---

## Tier 1: Core / Screening Questions (Questions 1–10)

### Q1: What is Platform Engineering, and how does it differ from DevOps?
- **Short Answer:** DevOps is an operational philosophy emphasizing shared ownership and rapid feedback ("you build it, you run it"). Platform Engineering is the product discipline of building self-service internal developer platforms (IDPs) that make that DevOps model sustainable at scale by removing extraneous cognitive load from stream-aligned teams.
- **Deep Dive:** In early DevOps implementations, every product squad had to master cloud architecture, Kubernetes manifests, IAM policies, and CI/CD pipelines. This created "shadow ops" developers and massive cognitive overload. A platform team treats internal developers as customers, providing curated golden paths and APIs so product teams retain operational ownership without reinventing the infrastructure wheel.
- **Trade-offs & Realities:** If a platform team operates via manual tickets, it is just a rebranded sysadmin silo. The platform must offer self-service capabilities (X-as-a-Service).
- **Follow-up Probes:** "Can an organization have DevOps without Platform Engineering?" (Yes, at small scale, ~10–30 engineers. Beyond ~100 engineers, unassisted DevOps usually degrades into cognitive chaos).

### Q2: What are the 4 fundamental team types in Team Topologies?
- **Short Answer:** Stream-aligned teams, Platform teams, Enabling teams, and Complicated-subsystem teams.
- **Deep Dive:** Stream-aligned teams deliver business value directly to users. Platform teams provide underlying capabilities via self-service APIs to reduce cognitive load. Enabling teams are coaches that cross-pollinate new technologies and practices. Complicated-subsystem teams own components requiring deep, rare technical specializations (e.g., cryptographic engines).
- **Trade-offs & Realities:** A common pitfall is letting an Enabling team act like a Platform team, or having a Platform team accept tickets like an operations helpdesk.
- **Follow-up Probes:** "What are the 3 interaction modes?" (Collaboration, X-as-a-Service, Facilitating).

### Q3: What is a Golden Path, and why is it not a mandate?
- **Short Answer:** A Golden Path is an opinionated, supported, and automated path of least resistance for building and deploying software. It is not an enforced straightjacket; teams may deviate ("off-road") if their requirements demand it, provided they own their operational maintenance.
- **Deep Dive:** Coined at Netflix (Paved Road) and Spotify, Golden Paths provide pre-configured templates, standardized CI/CD pipelines, and runtime baselines. By making the Golden Path dramatically faster and safer than manual alternatives, adoption occurs through intrinsic pull rather than executive mandate.
- **Trade-offs & Realities:** Mandating tools leads to developer resentment and shadow IT. However, the *outcomes* (e.g., security scanning, container signing, observability standards) can be mandatory, while the *tooling path* remains optional.
- **Follow-up Probes:** "What happens when a team decides to off-road?" (They accept the operational pager for their custom components, adhering to the basic platform contract like health probes and security standards).

### Q4: Explain the difference between an Internal Developer Platform (IDP) and an Internal Developer Portal.
- **Short Answer:** An IDP is the comprehensive collection of capabilities, APIs, infrastructure orchestration, and delivery pipelines. A Portal (like Backstage or Port) is merely a single graphical interface into that platform.
- **Deep Dive:** An IDP consists of 5 planes: Developer Control, Integration & Delivery, Resource Orchestration, Security & Governance, and Infrastructure. A portal sits in the Developer Control plane. If you install Backstage without automated underlying provisioning APIs, you have built a facade.
- **Trade-offs & Realities:** Many teams waste months configuring Backstage when a simple internal CLI, GitHub Actions, and Terraform modules would deliver far more value.
- **Follow-up Probes:** "When is a portal premature?" (When the team has fewer than 100 engineers and the underlying infrastructure provisioning is still ticket-based).

### Q5: What are the three types of cognitive load, and which one does a platform target?
- **Short Answer:** Intrinsic, Extraneous, and Germane. The platform's mission is to eliminate **Extraneous** cognitive load.
- **Deep Dive:** Intrinsic load is the fundamental difficulty of the business domain (e.g., loan calculations). Germane load is constructive energy spent mastering that domain. Extraneous load is non-value-add friction (e.g., debugging IAM syntax, wrestling with 6 YAML dialects, finding endpoints).
- **Trade-offs & Realities:** You cannot eliminate intrinsic load with tooling; attempting to over-abstract business logic creates brittle, leaky abstractions.
- **Follow-up Probes:** "How do you measure extraneous cognitive load?" (Through friction logs, developer surveys, onboarding ramp time, and support ticket clustering).

### Q6: What are the 4 core DORA metrics?
- **Short Answer:** Deployment Frequency, Lead Time for Changes, Change Failure Rate, and Failed Deployment Recovery Time (MTTR).
- **Deep Dive:** Developed by DevOps Research & Assessment, these metrics measure software delivery throughput and stability. Elite teams deploy on-demand multiple times a day with lead times under an hour, low change failure rates (<15%), and fast recovery (<1 hour).
- **Trade-offs & Realities:** Optimizing for Deployment Frequency without automated test guardrails causes Change Failure Rate to spike. Stability and throughput must be balanced.
- **Follow-up Probes:** "How does the platform team influence Lead Time for Changes?" (By caching CI builds, automating security scans, and eliminating manual approval gates).

### Q7: What is Conway's Law, and how does it relate to Platform Engineering?
- **Short Answer:** Conway's Law states that system architectures mirror the communication structures of the organizations that design them.
- **Deep Dive:** If an organization has separate siloed teams for Networking, Storage, Databases, and Security, applications will require multi-hop approvals and fragmented architectures. The **Inverse Conway Maneuver** reorganizes teams into autonomous stream-aligned squads supported by a self-service platform to drive decoupled microservice architectures.
- **Trade-offs & Realities:** Reorganizing teams without changing their interfaces and tooling only moves the bottleneck to a new department.
- **Follow-up Probes:** "Give an example of the Inverse Conway Maneuver in action." (Creating a dedicated Platform API team to replace inter-departmental ticket handoffs).

### Q8: What is a Thinnest Viable Platform (TVP)?
- **Short Answer:** The smallest possible set of documentation, APIs, and tools that measurably accelerates stream-aligned teams.
- **Deep Dive:** Instead of embarking on a multi-year project to build a sprawling custom portal, a TVP starts with a clear wiki guide, a shared CI/CD template, or an automated Terraform module. It evolves strictly in response to verified developer demand.
- **Trade-offs & Realities:** Building too much too early leads to abandoned platforms and wasted capital.
- **Follow-up Probes:** "What does a Day-1 TVP look like?" (A curated GitHub repository template with built-in linting, CI workflow, and a clear runbook).

### Q9: Why is "Build it and they will come" an anti-pattern in platform engineering?
- **Short Answer:** Because internal developers have real deadlines and established habits. If a platform is built without user research, it will fail to solve actual developer pain points and will be ignored or bypassed.
- **Deep Dive:** Platform engineering is a product discipline. Platforms require dedicated user discovery, continuous feedback, developer advocacy, documentation, and design partners to achieve voluntary adoption.
- **Trade-offs & Realities:** An ignored platform often results in leadership issuing heavy-handed mandates, which breeds resentment and employee attrition.
- **Follow-up Probes:** "How do you validate demand before building a feature?" (Conduct friction logs, shadow developer onboarding, and analyze the top themes in `#platform-help`).

### Q10: What is a Friction Log?
- **Short Answer:** A step-by-step document created by an engineer (or observer) performing a real workflow, recording every snag, delay, confusion, and timestamp encountered along the way.
- **Deep Dive:** Friction logs capture qualitative developer experience. By logging actions like "trying to create a new database" or "running integration tests locally", the team identifies exact friction points, missing docs, and broken scripts.
- **Trade-offs & Realities:** Engineers who have been at a company for years become immune to friction because they know all the workarounds. Friction logs are most effective when run with new hires.
- **Follow-up Probes:** "What are the typical columns in a friction log?" (Timestamp, Step, Expected Outcome, Actual Experience, Severity, Time Lost).

---

## Tier 2: Technical Deep Dive & Mid-Senior Questions (Questions 11–20)

### Q11: How do you address "Day-2 Drift" and template rot across hundreds of microservices?
- **Short Answer:** Replace static template copies with centralized reusable CI workflows, package shared logic into managed libraries, and use automated bot PR campaigns (Renovate/Dependabot) to keep dependencies and base images updated across the fleet.
- **Deep Dive:** Static scaffolding creates hard forks. To maintain consistency, decouple the template:
  1. *CI Pipelines:* Use callable workflows (`uses: org/workflows/deploy.yml@v2`).
  2. *Base Images:* Build centralized, distroless multi-architecture images and distribute automated bump PRs.
  3. *Scorecards:* Surface catalog drift scores to motivate teams to merge dependency updates.
- **Trade-offs & Realities:** Auto-merging dependency PRs requires comprehensive automated integration test suites; without them, automated PRs risk breaking production.
- **Follow-up Probes:** "How do you handle a breaking API change across 200 services?" (Provide automated migration codemods/scripts via bot PRs and maintain deprecation brownout windows).

### Q12: Walk through the architectural components of Backstage.
- **Short Answer:** Backstage comprises the Software Catalog, Software Templates (Scaffolder), TechDocs, and a modular Plugin architecture supported by a Node.js backend and React frontend.
- **Deep Dive:**
  - *Catalog:* Ingests `catalog-info.yaml` files to build a graph of Components, APIs, Systems, and Domains.
  - *Scaffolder:* Executes declarative multi-step recipes to fetch skeletons, substitute parameters, push to Git, and register in the catalog.
  - *TechDocs:* Compiles markdown within repos via MkDocs and renders centralized documentation.
  - *Plugins:* Independent frontend/backend extensions communicating via REST/GraphQL.
- **Trade-offs & Realities:** Backstage requires substantial maintenance overhead. Version upgrades can be painful due to yarn dependency trees.
- **Follow-up Probes:** "What database does Backstage use?" (PostgreSQL or SQLite for development).

### Q13: How do you establish Service Level Objectives (SLOs) for a Platform Team?
- **Short Answer:** Define user-centric SLIs reflecting availability, latency, and correctness for platform capabilities (CI runner queue time, Scaffolder success rate, GitOps sync latency), and establish an error budget policy.
- **Deep Dive:** Examples:
  - *CI Runner Queue Time:* 99% of pipeline jobs start within 45 seconds over a rolling 28-day window.
  - *Platform API Availability:* 99.9% successful HTTP responses for self-service endpoints.
  - *Scaffolding Reliability:* 98% of template generation runs produce an immediately green CI pipeline.
- **Trade-offs & Realities:** When the platform burns its error budget, feature development pauses to prioritize platform stability and developer enablement.
- **Follow-up Probes:** "What happens if a product team blames the platform for their failed SLO?" (Use distributed tracing and platform telemetry to isolate platform failure vs application error).

### Q14: How does the SPACE framework improve upon traditional productivity metrics?
- **Short Answer:** SPACE evaluates productivity holistically across Satisfaction, Performance, Activity, Communication, and Efficiency, preventing the distortion that occurs when optimizing for a single metric like commit count.
- **Deep Dive:** Developed by GitHub, Microsoft, and researchers, SPACE recognizes that high performance requires a balance. A team might have high Activity (many PRs) but terrible Efficiency (blocked on reviews) and low Satisfaction (burnout).
- **Trade-offs & Realities:** You cannot measure all dimensions simultaneously with automated tools; Satisfaction requires qualitative surveys, while Activity and Efficiency leverage telemetry.
- **Follow-up Probes:** "How do you measure Efficiency without spying on engineers?" (Track pipeline execution wait times, PR review turnaround latency, and context-switching frequency).

### Q15: Explain how Crossplane fits into an IDP Resource Orchestration Plane compared to Terraform.
- **Short Answer:** Terraform is typically executed imperatively via CI pipelines or CLI. Crossplane extends the Kubernetes control plane to manage external cloud resources declaratively with continuous reconciliation and drift detection.
- **Deep Dive:** Crossplane allows platform teams to define Custom Resource Definitions (Compositions and Composite Resource Definitions - XRDs). A product team declares a simple `kind: Database`, and Crossplane reconciles the underlying VPC peering, RDS instance, security groups, and IAM roles directly in AWS/GCP, healing any out-of-band drift automatically.
- **Trade-offs & Realities:** Crossplane introduces Kubernetes control plane complexity and high resource consumption at scale. Terraform is more widely known and has a broader ecosystem of providers.
- **Follow-up Probes:** "Can Terraform and Crossplane coexist?" (Yes; Terraform for foundational VPC/cluster landing zones, Crossplane for developer self-service workload resources).

### Q16: How do you run a successful deprecation and migration campaign across an enterprise?
- **Short Answer:** Use a published timeline (Announce → Warn → Brownout → Sunset), provide automated migration PRs, track progress in the software catalog, and conduct brownout drills.
- **Deep Dive:**
  1. *Announce:* 90-day advance notice with clear technical rationales and migration guides.
  2. *In-Tool Warnings:* Emit non-breaking warnings in CI build logs and catalog dashboards.
  3. *Automated Tooling:* Provide a CLI script or bot PR that migrates 80% of boilerplate automatically.
  4. *Brownouts:* Execute controlled 2-hour outages during working hours to flush out undocumented consumers.
  5. *Decommission:* Archive the legacy system cleanly.
- **Trade-offs & Realities:** Without executive support and clear migration tooling, deprecations drag on indefinitely, forcing the platform team to maintain dual stacks.
- **Follow-up Probes:** "What do you do when a critical service refuses to migrate before the deadline?" (Escalate with cost and risk data to their engineering director; offer pairing support).

### Q17: What is an "Escape Hatch" in platform design, and how is it implemented?
- **Short Answer:** A structured mechanism allowing teams with specialized technical needs to bypass standard Golden Path constraints without breaking foundational security and operational contracts.
- **Deep Dive:** Rather than blocking an innovative project because it doesn't use the approved Go/Java stack, the platform defines a **Platform Contract**:
  - Expose `/healthz` and `/readyz` probes.
  - Emit JSON logs to `stdout`.
  - Propagate W3C OpenTelemetry headers.
  - Run as non-root user.
  - Adhere to network and compute resource quotas.
  If the custom service fulfills the contract, it can run on the cluster, but the product team owns its internal runtime troubleshooting.
- **Trade-offs & Realities:** Unrestricted escape hatches lead to organizational fragmentation. Teams must understand that stepping off the paved road increases their operational burden.
- **Follow-up Probes:** "When should an off-road pattern be paved into an official Golden Path?" (When 3+ teams independently build similar custom solutions).

### Q18: How do you prioritize platform features using RICE and Cost of Delay?
- **Short Answer:** RICE ranks projects by `(Reach × Impact × Confidence) / Effort`. Cost of Delay (CD3) divides the economic cost of postponing a capability by the time required to build it, ensuring urgent deadlines are respected.
- **Deep Dive:** RICE is excellent for comparing feature ideas (e.g., Backstage portal vs. automated database provisioning). However, RICE misses time-sensitive external constraints (e.g., an upcoming cloud contract renewal or security audit). CD3 prioritizes tasks with severe cost of delay (e.g., deprecating an end-of-life Kubernetes version).
- **Trade-offs & Realities:** Scoring inputs can be subjective. Use developer surveys and ticket metrics to ground Reach and Impact in data.
- **Follow-up Probes:** "How much platform team capacity should be reserved for unplanned toil and maintenance?" (Typically 20–30%).

### Q19: What constitutes an effective Developer Journey Map?
- **Short Answer:** A visual mapping of every sequential step a developer takes to accomplish an outcome (e.g., create service, deploy change, triage incident), highlighting tools used, handoffs, wait times, and emotional friction.
- **Deep Dive:** Stages: Discover → Scaffold → Develop Locally → Commit/PR → CI Build & Test → Deploy Staging → Verify → Deploy Production → Monitor. By documenting the "time spent waiting" versus "time spent coding", the platform team exposes the biggest bottlenecks in the delivery pipeline.
- **Trade-offs & Realities:** A journey map is useless if treated as a static artifact; it must be updated quarterly to reflect platform improvements.
- **Follow-up Probes:** "What is Flow Efficiency in journey mapping?" (`Active Time / Total Cycle Time × 100%`).

### Q20: How do you balance platform standardization with developer autonomy?
- **Short Answer:** Standardize on outcomes and interfaces rather than implementation details. Pave the path for the 80% common use case, while providing documented escape hatches for the 20% outliers.
- **Deep Dive:** Enforce compliance at the boundary: automated security scans, workload identity, and ingress policies must be met by all workloads. Inside the container, give teams autonomy over framework and architecture, provided they meet operational baselines.
- **Trade-offs & Realities:** Too much standardization creates a rigid bureaucracy; too much autonomy creates unmaintainable technology sprawl.
- **Follow-up Probes:** "How do you handle a team that insists on using Haskell for a simple CRUD service?" (Present the total cost of ownership: hiring, onboarding, maintenance, and lack of platform tooling support).

---

## Tier 3: Staff & Principal Architect Scenarios (Questions 21–25)

### Q21: Design the technical and organizational strategy for introducing an IDP to a 500-engineer enterprise currently mired in manual tickets.
- **Candidate Framework:**
  1. *Discovery (Weeks 1–4):* Perform developer friction audits across 5 business units. Identify the 2 biggest bottlenecks (typically environment provisioning and CI build flakiness).
  2. *Thinnest Viable Platform (Weeks 5–12):* Do not start with Backstage. Build automated self-service GitHub Actions workflows and Crossplane/Terraform modules for the top requested resource.
  3. *Lighthouse Partnership:* Embed with 2 high-impact stream-aligned teams in *Collaboration* mode. Solve their delivery pain completely.
  4. *Showcase & Adoption:* Present the lighthouse team's 10x lead time reduction at engineering all-hands. Roll out self-service capabilities X-as-a-Service to the rest of the org.
  5. *Expansion (Months 6–12):* Introduce a software catalog (Backstage or SaaS) only once the underlying APIs are automated.
- **Red Flags:** Proposing a big-bang rewrite or mandating Backstage on day one.

### Q22: You inherit an internal platform that developers actively hate and bypass. How do you turn this around?
- **Candidate Framework:**
  1. *Stop Defending the Tool:* Acknowledge developer frustration openly. Conduct a listening tour with key tech leads and critics.
  2. *Audit the Workarounds:* Study the shadow IT systems developers built; their workarounds represent the true user requirements the official platform failed to meet.
  3. *Deprecate Broken Features:* Remove broken, bureaucratic guardrails that cause friction without providing security.
  4. *Fix the Core:* Focus on the top 3 developer papercuts (e.g., CI runner speed, local dev reliability).
  5. *Rebuild Trust:* Establish transparent SLAs, open `#platform-help`, and ship visible, daily improvements.
- **Red Flags:** Blaming developers for being uncooperative or asking leadership for stricter mandates.

### Q23: How do you design an IDP Developer Control Plane that serves both junior frontend developers and senior distributed systems engineers?
- **Candidate Framework:**
  - *Layered Abstractions:*
    - *Layer 1 (Opinionated Portal/GUI):* Simple forms in Backstage/Port for junior engineers and standard web services (generate repo, choose DB size, click deploy).
    - *Layer 2 (CLI & Declarative Manifests):* `pctl` CLI and high-level Kubernetes CRDs for experienced engineers wanting scriptability and GitOps automation.
    - *Layer 3 (Raw Cloud & Escape Hatch):* Direct access to underlying cloud primitives governed by policy-as-code (Kyverno/OPA) for staff architects building specialized distributed systems.
  - All layers write to the same underlying declarative state (GitOps / Crossplane).
- **Red Flags:** Building a GUI that hides everything with no way to customize, or exposing raw AWS primitives with zero abstractions.

### Q24: A CTO demands: "We need an IDP to cut our cloud bill by 40% and enforce zero-trust security by next quarter." How do you respond?
- **Candidate Framework:**
  - *Expectation Management:* Clarify that an IDP's primary value is developer velocity and operational consistency; while FinOps and security guardrails are essential planes of an IDP, cutting 40% of cloud spend in 3 months requires direct workload rightsizing and architectural decommissioning, not just a portal.
  - *Phased Architecture:*
    1. *FinOps Integration:* Embed Kubecost/OpenCost and AWS Cost Allocation tags into Golden Path templates so teams have immediate cost visibility per service in their catalog.
    2. *Automated Idle Teardown:* Implement ephemeral environment auto-deletion (Karpenter/Kube-downscaler) for dev/staging clusters.
    3. *Zero-Trust Baselines:* Enforce SPIFFE/SPIRE or EKS Pod Identity and Kyverno admission controls transparently inside the Golden Path without requiring developers to write IAM logic.
- **Red Flags:** Promising to achieve all goals simultaneously without clarifying the trade-offs on developer velocity.

### Q25: What is the architectural role of AI agents and LLMs within the future of Internal Developer Platforms?
- **Candidate Framework:**
  - *Contextual Knowledge Retrieval:* Natural language querying of the software catalog, architecture decision records (ADRs), runbooks, and telemetry (e.g., "Which service owns the `/v2/charges` route and who is on call?").
  - *Automated Remediation & Scaffolding:* AI assistants that generate boilerplate code and Kubernetes manifests aligned with the company's private Golden Path libraries.
  - *Intelligent Triage:* Automated initial response in `#platform-help`, diagnosing common pipeline failures and pointing engineers to exact runbooks.
  - *Guardrails:* AI agents must operate within strict deterministic policy boundaries (Kyverno, OPA, GitOps validation) so generated configurations cannot compromise production security.
- **Red Flags:** Believing AI replaces the need for robust platform APIs, or allowing unconstrained AI agents to deploy unreviewed code directly to production.
