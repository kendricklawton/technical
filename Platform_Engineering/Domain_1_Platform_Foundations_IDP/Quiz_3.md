# PE Domain 1: Platform Foundations & IDPs — Scenario Quiz 3

Master platform measurement frameworks (DORA, SPACE, DevEx), migration campaigns, and anti-pattern recovery.

---

### Question 1
A product engineering department measures success by counting the number of Pull Requests merged per developer per week. According to the **SPACE framework**, what is the fatal flaw with this metric?

A) Pull request counts only measure a single dimension (Activity) and incentivize developers to split small, meaningless changes to inflate numbers, while ignoring Quality, Satisfaction, and Efficiency.  
B) SPACE recommends measuring lines of code committed instead.  
C) Activity metrics are strictly illegal under DORA standards.  
D) Pull request numbers cannot be calculated in Git.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** A  
**Explanation:** The SPACE framework emphasizes that productivity cannot be measured by a single metric. Focusing exclusively on Activity (PR counts) creates perverse incentives (splitting changes into tiny PRs) while ignoring Satisfaction, Performance, Communication, and Efficiency.
</details>

---

### Question 2
Your engineering organization's DORA metrics show:
- Deployment Frequency: Multiple times per day (Elite)
- Lead Time for Changes: 45 minutes (Elite)
- Change Failure Rate: 52% (Low Performer)
- Failed Deployment Recovery Time: 18 hours (Low Performer)
What is the primary diagnostic conclusion for this platform team?

A) The team should immediately switch from Kubernetes to bare-metal servers.  
B) The delivery pipeline moves fast but lacks pre-production validation, automated testing guardrails, and rapid rollback mechanisms.  
C) Developers are shipping code too slowly and need tighter deadlines.  
D) The DORA framework is fundamentally broken and should be discarded.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** B  
**Explanation:** High deployment frequency paired with a high Change Failure Rate (52%) and slow recovery (18h) indicates that teams are pushing broken code quickly into production without automated tests, canary analysis, or instant 1-click rollback capabilities. The platform team must introduce progressive delivery (Argo Rollouts) and automated quality gates.
</details>

---

### Question 3
The DevEx framework emphasizes three core dimensions: Feedback Loops, Cognitive Load, and Flow State. Which platform initiative directly improves **Feedback Loops**?

A) Renaming all Git branches from master to main.  
B) Adding a mandatory manual approval step before staging deployments.  
C) Reducing local Docker/unit test feedback time from 15 minutes to 30 seconds and providing fast ephemeral CI environments.  
D) Writing a 50-page architecture document on cloud networking.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** C  
**Explanation:** Feedback loops are about the speed and quality of responses developers receive after taking an action. Sub-minute local builds, instant test execution, and rapid CI runs dramatically tighten feedback loops.
</details>

---

### Question 4
You need to migrate 300 microservices from an unmaintained, in-house deployment script to a modern GitOps pipeline using Argo CD. What is the most effective migration pattern?

A) Delete the legacy script immediately to force teams to figure out GitOps on their own.  
B) Employ the Strangler Fig pattern: partner with 2 lighthouse teams to validate the GitOps pipeline, then create automated fleet PRs to migrate the remaining services in staged waves.  
C) Schedule a company-wide "Code Freeze Weekend" and manually cut over all 300 services simultaneously.  
D) Allow product teams to migrate whenever they want without deadlines or tracking.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** B  
**Explanation:** Big-bang migrations carry catastrophic risk. The Strangler Fig pattern allows incremental validation with trusted design partners, followed by automated batch PRs to migrate remaining services systematically without business disruption.
</details>

---

### Question 5
What is a **"Brownout Exercise"** during a platform deprecation campaign?

A) Permanently deleting all legacy databases without prior warning.  
B) A performance test where servers are starved of electrical power.  
C) A dark-themed styling update applied to the developer portal.  
D) A scheduled, temporary, and announced short outage of the deprecated tool during business hours to uncover hidden, undocumented dependencies before permanent retirement.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** D  
**Explanation:** Brownouts are intentional, controlled, temporary shutdowns of a deprecated capability (e.g., turning off the legacy artifact repository for 2 hours on a Wednesday). It surfaces unmigrated services while engineers are at work to fix them, avoiding surprises when the system is decommissioned permanently.
</details>

---

### Question 6
Under DORA definitions, what constitutes a "Failed Deployment Recovery Time" (formerly MTTR)?

A) The time required to build a fresh Docker image on a developer laptop.  
B) The time spent waiting for hardware replacement in an on-premises data center.  
C) The time elapsed from the moment an incident or deployment failure occurs in production until service is fully restored to normal operations (e.g., via rollback, fix-forward, or failover).  
D) The time it takes for a developer to write a postmortem document.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** C  
**Explanation:** Recovery time measures the duration from the start of a production degradation or failure until the service successfully returns to a healthy operating state.
</details>

---

### Question 7
A platform team is flooded with over 40 Slack messages a day in `#ask-platform`, asking basic questions about how to configure database connection pools. How should the platform lead address this?

A) Ignore all messages and focus exclusively on future roadmap features.  
B) Institute a rotating "Platform Support Shield" role, treat support requests as user research data, and improve the Golden Path defaults and documentation to eliminate the root causes.  
C) Complain to the VP of Engineering that developers are incompetent.  
D) Delete the Slack channel and force engineers to send emails.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** B  
**Explanation:** Repeated support questions are an invaluable research signal showing gaps in documentation or suboptimal defaults. A dedicated rotating support role protects teammates from interruptions while feeding findings directly into the platform improvement backlog.
</details>

---

### Question 8
Which metric is the best indicator of genuine **voluntary adoption** of a Golden Path?

A) The overall size of the platform team's Jira backlog.  
B) The number of hours the platform team spent writing documentation.  
C) The percentage of product teams forced to use the template by executive mandate.  
D) The ratio of new services scaffolded using the Golden Path versus manual setups, along with high developer retention on that path over a 6-month period.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** D  
**Explanation:** Voluntary adoption is reflected in real developer choices: when teams consistently choose the Golden Path for new services and continue using it over time without managerial coercion.
</details>

---

### Question 9
In the context of developer productivity, what is **"Flow State"**?

A) A mental state of deep immersion, focus, and peak productivity where developers can solve complex problems without disruptive context switches or tool failures.  
B) The rate at which Jira tickets move across a Kanban board.  
C) A Git workflow where all developers commit directly to the `main` branch.  
D) The speed of packets flowing across a 100Gbps network switch.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** A  
**Explanation:** Flow state is the psychological state of energized focus and immersion. High-performing engineering cultures protect flow state by minimizing unnecessary interruptions, meeting overload, and frustrating tooling snags.
</details>

---

### Question 10
Which of the following books is the primary industry reference for organizing teams for fast flow, defining platform team interactions, and managing organizational cognitive load?

A) *Team Topologies* by Matthew Skelton and Manuel Pais  
B) *Design Patterns* by the Gang of Four  
C) *Introduction to Algorithms* by Cormen et al.  
D) *Clean Code* by Robert C. Martin  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** A  
**Explanation:** *Team Topologies* (Skelton & Pais) is the definitive book that introduced stream-aligned teams, platform teams, enabling teams, complicated-subsystem teams, interaction modes, and cognitive load management to modern platform engineering.
</details>
