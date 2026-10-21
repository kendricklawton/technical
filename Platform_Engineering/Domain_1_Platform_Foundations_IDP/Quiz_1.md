# PE Domain 1: Platform Foundations & IDPs — Scenario Quiz 1

Test your knowledge on platform fundamentals, Team Topologies, and product management principles.

---

### Question 1
Your company has 200 developers across 25 product teams. Currently, when a team needs a new PostgreSQL database, they submit a Jira ticket to the Database Administration (DBA) team, who manually executes a Terraform script. The median time to receive a database is 6 business days. As the new Platform Lead, what is your most appropriate first initiative?

A) Expose a declarative Custom Resource Definition (CRD) or self-service Terraform module integrated with GitOps that stream-aligned teams can define directly in their application repo.  
B) Mandate that all teams switch to SQLite so that database provisioning is no longer a centralized bottleneck.  
C) Build a custom Backstage plugin with a React GUI that allows developers to request a database, which sends a webhook to create a Jira ticket.  
D) Hire two additional DBAs to clear the ticket backlog and lower the SLA to 2 days.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** A  
**Explanation:** Platform engineering aims to eliminate manual ticket-based handoffs by providing self-service interfaces. Providing a declarative API (CRD via Crossplane/operator or a tested Terraform module via PR automation) enables developers to consume databases X-as-a-Service. Option D preserves the ticket silo. Option C is "lipstick on a ticket system" (portal without a platform). Option B is unrealistic for enterprise production systems.
</details>

---

### Question 2
According to *Team Topologies*, which interaction mode should a platform team use when initially researching and prototyping a brand new capability (such as an ephemeral preview environment) with a stream-aligned team?

A) Outsourcing  
B) Facilitating  
C) X-as-a-Service  
D) Collaboration  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** D  
**Explanation:** Collaboration is a temporary, high-touch interaction mode where two teams work closely together to co-create and discover the boundary of a new capability. Once the capability is proven and hardened, the platform team transitions it into an X-as-a-Service model. Facilitating is used by enabling teams to coach.
</details>

---

### Question 3
A platform team discovers that 6 months after launching an internal developer portal, fewer than 15% of product engineers log into it weekly. When surveyed, engineers say they prefer using the command line and finding information directly in GitHub. How should the platform team interpret this signal?

A) The platform team should ask executive leadership to mandate portal usage for all engineering check-ins.  
B) The developers lack discipline and need mandatory training sessions.  
C) The platform team fell into the "Portal as the Platform" trap; the portal is merely an interface, and the team should meet developers where they work (CLI / Git).  
D) The team should immediately rewrite the portal in a different web framework.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** C  
**Explanation:** A developer portal is just one interface to a platform. Forcing developers onto an unwanted web portal violates the principle of "developers as customers" and "voluntary adoption". The platform team should offer robust CLI tools and Git-based interfaces that integrate seamlessly with existing developer habits.
</details>

---

### Question 4
In John Sweller's Cognitive Load Theory (as applied in *Team Topologies*), which type of cognitive load is the primary target for reduction or elimination by the platform team?

A) Extraneous cognitive load  
B) Intrinsic cognitive load  
C) Germane cognitive load  
D) Biological cognitive load  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** A  
**Explanation:** Extraneous cognitive load represents effort wasted on tasks that add no business value (e.g., figuring out complex IAM syntax, fixing broken CI boilerplate, navigating labyrinthine documentation). Intrinsic load is the difficulty inherent in the business problem (e.g., calculating mortgage rates), while germane load is the effort dedicated to learning and mastering the core domain.
</details>

---

### Question 5
What is the core distinction between a "Golden Path" and a "Golden Cage"?

A) A Golden Path is designed for junior developers; a Golden Cage is for senior staff engineers.  
B) A Golden Path offers an opinionated, supported path of least resistance with permitted escape hatches; a Golden Cage strictly mandates tooling with no deviations allowed.  
C) A Golden Path uses open-source tools; a Golden Cage uses proprietary enterprise software.  
D) A Golden Path is maintained by product teams; a Golden Cage is maintained by platform teams.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** B  
**Explanation:** A Golden Path is an accelerated, well-supported highway that encourages voluntary adoption while allowing teams with specialized needs to off-road (with the caveat that they maintain their custom solution). A Golden Cage enforces strict conformity via bureaucracy, stifling innovation and generating developer resentment.
</details>

---

### Question 6
Which of the following is considered an "Enabling Team" in the Team Topologies framework?

A) A team that owns and operates the company's central Kubernetes clusters and VPC network.  
B) A specialized team of security coaches that embeds with product teams for two weeks to teach threat modeling, then rotates away.  
C) A cross-functional squad that delivers the payment processing user journey.  
D) A team that maintains the specialized C++ video rendering engine.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** B  
**Explanation:** An Enabling Team consists of specialists who actively coach and spread knowledge across other teams, purposefully avoiding long-term operational ownership so they can step away once the skill is transferred. Option A is a Platform team; Option C is a Stream-aligned team; Option D is a Complicated-subsystem team.
</details>

---

### Question 7
The platform engineering team wants to track its internal operational performance. Which of the following is an appropriate Service Level Indicator (SLI) for a Golden Path Scaffolding capability?

A) The number of pull requests opened by product engineers each month.  
B) The percentage of automated scaffolder executions that complete successfully and generate a passing initial CI build within 3 minutes.  
C) The total number of microservices currently running in production.  
D) The overall cloud bill for the entire engineering department.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** B  
**Explanation:** A platform SLI must measure the reliability and responsiveness of the platform capability from the user's perspective. Tracking the success rate and latency of generating a working, validated service repo directly reflects scaffolder service quality.
</details>

---

### Question 8
When prioritizing platform backlog items using the RICE scoring model, what does the "C" represent, and how is it quantified?

A) Confidence — expressed as a percentage (e.g., 100% = high confidence, 50% = speculative) to discount uncertain assumptions.  
B) Complexity — scored on a Fibonacci scale from 1 to 21.  
C) Customers — the total number of developers who requested the feature.  
D) Cost — in dollars required to purchase enterprise licenses.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** A  
**Explanation:** In RICE prioritization (`Score = [Reach × Impact × Confidence] / Effort`), Confidence is a percentage (typically 100% for solid data, 80% for medium evidence, 50% for low confidence) used to prevent speculative, unvalidated ideas from scoring artificially high.
</details>

---

### Question 9
Conway's Law states that "organizations which design systems are constrained to produce designs which are copies of the communication structures of these organizations." How does a platform team leverage the **Inverse Conway Maneuver**?

A) By eliminating all cross-team communication and enforcing code reviews exclusively via anonymous AI bots.  
B) By allowing application architectures to dictate organizational reporting structures organically.  
C) By restructuring teams and communication boundaries first to drive the desired decoupled software architecture.  
D) By forcing all developers to report to the same engineering director.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** C  
**Explanation:** The Inverse Conway Maneuver intentionally designs the organizational team structures and interfaces to mirror the target system architecture (e.g., creating independent stream-aligned teams consuming decoupled X-as-a-Service platforms to foster loosely-coupled microservice architectures).
</details>

---

### Question 10
What is the primary danger of operating a platform team with an "Extrinsic Push" (mandatory adoption) model rather than an "Intrinsic Pull" model?

A) Product teams will release software too frequently, overwhelming customer support.  
B) Cloud costs will immediately drop to zero.  
C) The platform team will run out of backlog items to work on.  
D) Developers will develop workarounds, build unmonitored shadow IT systems, and experience severe morale decline.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** D  
**Explanation:** When adoption is mandated without developer buy-in or product empathy, developers find ways to bypass the platform (shadow infrastructure, unapproved SaaS tools, manual hacks) and engineering morale drops sharply. Platforms should be adopted voluntarily because they are demonstrably superior to manual alternatives.
</details>
