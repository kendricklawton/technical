# Domain 4: Incident Management & SRE Postmortems — Quiz 1

**Exam:** Google Cloud Certified: Professional Cloud DevOps Engineer  
**Format:** 10 Scenario-Based Multiple-Choice Questions with In-Depth Technical Explanations

---

### Question 1
During a SEV-1 outage of a tier-1 customer authentication service on GKE, four different software engineers, a customer support director, and a product VP join the emergency bridge simultaneously and start proposing conflicting database configuration tweaks. Communication is chaotic and uncoordinated. According to the Incident Command System (ICS) framework, what should happen immediately?
- A. Every engineer tests their proposed configuration directly on production pods.
- B. The product VP takes over because they hold the highest executive authority.
- C. Disband the call and continue discussions on an asynchronous email thread.
- D. A designated Incident Commander (IC) takes control of the bridge, asserts command, delegates specific technical roles (Operations Lead, Communications Lead), and mutes unauthorized cross-talk.

---

### Question 2
While triaging an active outage where an e-commerce checkout API is failing with HTTP 504 Gateway Timeouts, the on-call engineer notices a memory leak in the latest microservice release. Management wants to understand the exact line of code causing the leak. What is the fundamental SRE principle for incident response?
- A. Wait for the cloud provider support team to respond to a priority ticket.
- B. Keep the service down until a patch is written, tested, and reviewed.
- C. Mitigate the customer impact first (e.g., rollback release, route traffic to secondary region, or restart pods), and defer deep root cause debugging until after service restoration.
- D. Debug the root cause thoroughly on production before making any changes.

---

### Question 3
An engineering team is writing a postmortem following a major data pipeline failure that led to 6 hours of missing analytics data. The preliminary draft states: "The outage occurred because Engineer Bob forgot to update the service account permissions in production after the deployment." How should this statement be revised to align with Google's blameless culture?
- A. "The outage occurred because Bob was working overtime."
- B. "The outage occurred because the deployment workflow relied on manual IAM configuration steps and lacked automated least-privilege verification before traffic promotion."
- C. "The outage occurred because Engineer Bob failed to read the deployment checklist."
- D. Remove the incident from the records completely.

---

### Question 4
What is the primary role of the Communications Lead in the Incident Command System (ICS) during a major incident?
- A. Writing hotfixes and executing `kubectl rollout undo`.
- B. Approving code commits to the main Git repository.
- C. Handling all inbound and outbound stakeholder communications, updating status pages, and insulating the Incident Commander and technical responders from external distractions.
- D. Calculating developer bonuses.

---

### Question 5
An organization holds a quarterly "Disaster Recovery Testing" exercise where engineers simulate the loss of an entire Google Cloud region (`us-central1`). Which SRE concept describes the practice of intentionally introducing controlled failures into production and staging systems to build confidence in system resilience?
- A. Blue-Green Deployment
- B. Canary Verification
- C. Feature Flagging
- D. Chaos Engineering / Game Days

---

### Question 6
During an incident, the Operations Lead applies an emergency traffic-shedding policy using Cloud Armor rate limiting to save a backend database from crashing. What must the Incident Commander ensure happens before declaring the incident resolved?
- A. Verify that all temporary mitigations are documented, traffic has stabilized, service SLIs have returned to normal bounds, and an owner is assigned to remove or tune the temporary rate limits.
- B. Fire the engineer who deployed the code that caused the traffic surge.
- C. Immediately publish a public apology on social media.
- D. Disable Cloud Armor completely.

---

### Question 7
In a postmortem, action items are categorized to prevent recurrence. Which of the following is considered an antipattern for postmortem action items?
- A. "Add automated integration tests in Cloud Build to validate database migration timeouts."
- B. "Remind developers in next Monday's standup to be more careful when modifying production configs."
- C. "Implement rate-limiting filters at the Cloud Load Balancer level."
- D. "Configure a multi-window burn rate alert for HTTP 5xx errors."

---

### Question 8
A SaaS company categorizes incidents into severity levels. A SEV-1 incident is declared when:
- A. A new feature request is rejected by marketing.
- B. An internal developer documentation page has a broken hyperlink.
- C. A non-critical batch job fails and will be retried automatically tonight.
- D. A core customer-facing service is completely down or severely degraded for a large percentage of users with active SLO budget burn.

---

### Question 9
Why does Google SRE advocate for postmortems to be publicly searchable and readable across the entire engineering organization?
- A. To comply with federal tax audit requirements.
- B. To automatically generate customer invoices.
- C. To facilitate institutional learning, allow teams to recognize shared failure patterns, and prevent identical architectural bugs across different projects.
- D. To publicly name and shame teams that introduce regressions.

---

### Question 10
An Incident Commander notices that responders are suffering from cognitive overload and fatigue during an incident that has lasted 14 hours. What action should the IC take?
- A. Formally transition the Incident Commander and Responder roles to a fresh shift of engineers via a structured handoff briefing.
- B. Disband the response team and leave the outage unmonitored.
- C. Escalate to the CEO.
- D. Order the team to drink caffeine and push through until resolution.

---

## Answer Key & Rationales

1. **D** — Under ICS, clear chain-of-command is essential. The Incident Commander establishes authority, assigns structured roles, and prevents disorganized chaos on the bridge.
2. **C** — The core principle of emergency response is *mitigate first, debug later*. Customer impact must be removed as fast as possible before investigating root causes.
3. **B** — Blameless postmortems focus on systemic flaws and lack of automated guardrails rather than individual human error.
4. **C** — The Comms Lead shields responders from executive and customer inquiries, publishing regular status updates so engineers can focus on mitigation.
5. **D** — Chaos Engineering and Game Days test resilience by deliberately inducing faults into controlled environments.
6. **A** — Resolution requires verified SLI restoration, cleanup or tracking of temporary emergency patches, and transition to post-incident review.
7. **B** — "Telling engineers to be careful" is an ineffective human-dependent action item. Postmortems must result in systemic, automated, technical safeguards.
8. **D** — SEV-1 represents catastrophic, revenue-impacting outages with active severe user degradation.
9. **C** — Organization-wide visibility turns isolated failures into collective institutional knowledge, preventing repeated mistakes across departments.
10. **A** — Responders become error-prone after extended hours. Structured handoffs to fresh rotations maintain response efficacy during prolonged events.
