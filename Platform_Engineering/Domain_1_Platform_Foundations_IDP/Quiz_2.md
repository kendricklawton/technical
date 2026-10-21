# PE Domain 1: Platform Foundations & IDPs — Scenario Quiz 2

Deepen your understanding of IDP reference architectures, Backstage operational realities, and delivery planes.

---

### Question 1
In the CNCF 5-plane IDP reference architecture, which plane is responsible for binding declarative application resource requests (e.g., "I need an RDS Postgres database with 50GB storage") to physical cloud infrastructure?

A) Integration & Delivery Plane  
B) Resource Orchestration Plane  
C) Developer Control Plane  
D) Security & Governance Plane  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** B  
**Explanation:** The Resource Orchestration Plane (using engines like Crossplane, Terraform Controller, or custom Kubernetes Operators) accepts declarative application specifications and orchestrates the creation, lifecycle, and teardown of cloud resources.
</details>

---

### Question 2
Your engineering organization (120 developers) is considering adopting Spotify's Backstage as an internal developer portal. Which of the following is the most critical operational factor your team must plan for?

A) Backstage replaces Kubernetes, eliminating the need for container runtimes.  
B) Backstage automatically migrates legacy Jenkins pipelines to GitHub Actions without manual intervention.  
C) Backstage is a TypeScript/React and Node.js SDK requiring dedicated software engineers to build, customize plugins, and maintain frequent dependency updates.  
D) Backstage is a pre-compiled binary that can be deployed via a single Helm command without code changes.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** C  
**Explanation:** Backstage is an extensible framework/SDK rather than an off-the-shelf product. Organizations must allocate ongoing frontend and backend engineering bandwidth to maintain the code, update npm packages, and develop custom integrations.
</details>

---

### Question 3
A microservice repository contains a `catalog-info.yaml` file declaring:
```yaml
apiVersion: backstage.io/v1alpha1
kind: Component
metadata:
  name: order-aggregator
spec:
  type: service
  lifecycle: experimental
  owner: group:checkout-squad
```
What is the primary role of this file within the Internal Developer Platform?

A) It instructs the Linux kernel how to allocate cgroups.  
B) It acts as a Kubernetes HorizontalPodAutoscaler configuration.  
C) It replaces Dockerfile container build instructions.  
D) It serves as a metadata descriptor for the centralized Software Catalog to track service ownership, lifecycle, and dependencies.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** D  
**Explanation:** `catalog-info.yaml` is the Backstage entity descriptor format. It registers the service in the central Software Catalog, establishing ownership (`checkout-squad`), lifecycle state (`experimental`), and system relationships.
</details>

---

### Question 4
A platform team creates 50 microservices by cloning a standard template. One year later, a critical zero-day vulnerability is discovered in the Alpine base image used by the template. What fundamental limitation of static scaffolding templates does this scenario expose?

A) The Escape Hatch Anti-Pattern  
B) The Inverse Conway Dilemma  
C) Template Rot / Day-2 Drift  
D) The Monolithic Abstraction Trap  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** C  
**Explanation:** Static templates suffer from "Day-2 Drift": once generated, the repositories become independent forks. Updates to the master template do not automatically propagate to downstream repos. Solving this requires automated dependency bots (e.g., Renovate) or centralized reusable workflows.
</details>

---

### Question 5
Which plane of the IDP reference architecture is responsible for ensuring that all container images deployed to production are cryptographically signed and have an accompanying Software Bill of Materials (SBOM)?

A) Security & Governance Plane  
B) Developer Control Plane  
C) Infrastructure Plane  
D) Resource Orchestration Plane  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** A  
**Explanation:** The Security & Governance Plane handles policy enforcement (e.g., Kyverno, OPA), image signature verification (Sigstore/Cosign), workload identity (SPIRE), and compliance controls.
</details>

---

### Question 6
You are designing an internal CLI (`pctl`) for your company's platform. Which design choice best supports developer adoption and scriptability?

A) Store user authentication tokens unencrypted in `/tmp/auth.txt`.  
B) Support structured JSON and YAML output flags (e.g., `-o json`), follow POSIX exit codes, and provide clear error messages with actionable remediation URLs.  
C) Require interactive terminal prompts for every command without flags.  
D) Require developers to write custom bash aliases to invoke the tool.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** B  
**Explanation:** Great developer tooling offers structured output for CI/CD scripting, standard exit codes, non-interactive execution flags, and actionable remediation links when errors occur.
</details>

---

### Question 7
What is the function of **TechDocs** in the Backstage ecosystem?

A) It automatically generates API code from Jira tickets.  
B) It serves as a replacement for Confluence and Microsoft Office across the entire enterprise.  
C) It monitors server CPU and RAM utilization in real time.  
D) It provides a "Docs-like-Code" engine that compiles Markdown stored directly in application repositories (via MkDocs) and renders it within Backstage.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** D  
**Explanation:** TechDocs enables docs-like-code: developers write markdown files inside their code repository, which are automatically built and rendered centrally in Backstage, keeping documentation synchronized with code.
</details>

---

### Question 8
A platform team wants to provide a self-service database provisioning mechanism. They decide between:
Option 1: A Backstage form that executes a Python script running raw AWS CLI commands.
Option 2: A Kubernetes Custom Resource (`kind: PostgresInstance`) managed by Crossplane with GitOps synchronization via Argo CD.
Why is Option 2 architecturally superior for a resilient IDP?

A) Option 2 provides declarative state, continuous reconciliation against drift, auditability in Git, and automated lifecycle management.  
B) Option 2 is faster to implement in an afternoon than Option 1.  
C) Option 1 has higher security because it relies on imperative bash execution.  
D) Option 2 requires no Kubernetes knowledge.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** A  
**Explanation:** Declarative state with reconciliation loops (Crossplane/GitOps) prevents configuration drift, maintains a complete audit trail in Git, and provides self-healing infrastructure, unlike imperative scripts which lack state tracking and drift correction.
</details>

---

### Question 9
In an organization with 50 engineers, the Head of Infrastructure proposes spending 8 months building a custom, ground-up web developer portal from scratch. Why would a seasoned Platform Architect push back on this proposal?

A) Building custom software is legally prohibited under CNCF guidelines.  
B) 50 engineers is too large an organization for any platform tooling.  
C) It violates the Thinnest Viable Platform (TVP) principle; with 50 engineers, a combination of GitHub templates, reusable workflows, and clear markdown documentation provides immediate ROI with minimal maintenance overhead.  
D) Developers refuse to read web pages under any circumstances.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** C  
**Explanation:** A 50-engineer organization needs a Thinnest Viable Platform (TVP). Allocating substantial engineering capital to build a custom web portal before establishing robust baseline CI/CD and deployment automation is an expensive mistake.
</details>

---

### Question 10
What role do Service Scorecards play within an Internal Developer Platform?

A) They rank individual engineers by the number of lines of code they commit each week.  
B) They calculate the monthly bonus for product managers.  
C) They determine which services should be instantly deleted if traffic drops.  
D) They provide automated visibility into whether services adhere to organizational standards (security patches, test coverage, SLO definitions, ownership metadata).  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** D  
**Explanation:** Service Scorecards gamify and clarify quality standards by measuring compliance against operational excellence criteria (e.g., "has a valid PagerDuty on-call rota", "base image is updated within 30 days", "has open-telemetry tracing enabled").
</details>
