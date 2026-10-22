# Domain 4: Incident Management & SRE Postmortems — Quiz 2

**Exam:** Google Cloud Certified: Professional Cloud DevOps Engineer  
**Format:** 10 Scenario-Based Multiple-Choice Questions with In-Depth Technical Explanations

---

### Question 1
An SRE team conducts a postmortem following a cascading failure in an interconnected microservices mesh. They discover that when Service B experienced a 100ms latency increase, Service A aggressively retried every failed request with exponential backoff disabled, quickly multiplying total network requests by a factor of 10. What design pattern should the team implement to prevent this failure mode?
- A. Infinite linear retries
- B. Circuit Breaker with Exponential Backoff and Jitter
- C. Disabling load balancing
- D. Increasing pod memory requests by 500%

---

### Question 2
What is the primary objective of a "Pre-Mortem" exercise conducted prior to the launch of a high-visibility new banking service on Google Cloud?
- A. To draft the final billing invoice.
- B. To assume the launch has already suffered a catastrophic failure and work backward to identify architectural weaknesses, operational blind spots, and missing playbooks.
- C. To celebrate the project completion ahead of time.
- D. To audit developer timesheets.

---

### Question 3
An incident document contains an executive summary, timeline of events, root cause analysis, impact assessment, and action items. In which section should the timeline specifically record the "Time to Detect (TTD)" and "Time to Mitigate (TTM)"?
- A. Action Items
- B. Incident Timeline & Metrics
- C. Appendix
- D. Glossary

---

### Question 4
An on-call engineer is paged at 3:00 AM. The alert notification links directly to an outdated, ambiguous wiki page with no operational commands, leading to a 45-minute delay while the engineer figures out how to restart the worker pool. What artifact was missing or inadequate?
- A. Architecture Diagram
- B. Service Playbook / Runbook
- C. GitHub README
- D. Company Handbook

---

### Question 5
During an incident on Cloud Run, the operations lead rolls back a faulty revision. However, traffic continues to route to the bad revision for 15 minutes because CDN cache headers were set to `Cache-Control: public, max-age=3600`. What mitigation step should be executed immediately?
- A. Wait 45 minutes for the cache to expire naturally.
- B. Issue a Cloud CDN cache invalidation command for the affected URL paths.
- C. Delete the Cloud Storage backend.
- D. Terminate the domain registration.

---

### Question 6
Which metric measures the average time elapsed between when a customer-facing degradation begins and when the alerting system alerts the on-call engineer?
- A. MTBF (Mean Time Between Failures)
- B. MTTD (Mean Time to Detect)
- C. MTTR (Mean Time to Resolve)
- D. TCO (Total Cost of Ownership)

---

### Question 7
In Google SRE, what is "Operational Debt"?
- A. Unpaid invoices to Google Cloud.
- B. The accumulated backlog of unaddressed bugs, manual workarounds, outdated playbooks, and brittle infrastructure shortcuts that degrade long-term system reliability.
- C. The total number of open Jira tickets across the engineering org.
- D. The cost of running multi-region compute instances.

---

### Question 8
During an incident, how should the Incident Commander manage status updates to non-technical stakeholders (e.g. Sales, Marketing, C-Suite)?
- A. Invite all 50 stakeholders into the operational debugging voice bridge.
- B. Establish an asynchronous status page / Slack broadcast channel where the Communications Lead posts standardized, clear, non-jargon updates at scheduled intervals (e.g. every 20 minutes).
- C. Ignore them until the incident is fully resolved next week.
- D. Hand off on-call credentials to the sales lead.

---

### Question 9
A team realizes that during an outage, their primary monitoring tool hosted in the same GKE cluster also crashed, leaving them blind to pod statuses. What architectural principle was violated?
- A. Infrastructure as Code
- B. Out-of-band / Decoupled Observability Plane
- C. Microservices Decoupling
- D. Container immutability

---

### Question 10
What makes an action item "actionable" in a postmortem?
- A. Assigning it to an entire team with no single owner or due date.
- B. Assigning a specific single owner, a clear definition of done, a priority level (P0/P1/P2), and tracking it in the primary issue tracker.
- C. Vaguely suggesting that "we should improve logging someday".
- D. Sending an email thanking everyone for their hard work.

---

## Answer Key & Rationales

1. **B** — Circuit breakers cut off traffic to failing downstreams, and exponential backoff with jitter prevents thundering herd retry storms.
2. **B** — A pre-mortem hypothetically assumes total failure to unearth unmitigated risks, failure domains, and gaps in playbooks before going live.
3. **B** — The Incident Timeline tracks precise timestamps for Detection (TTD), Triage, Escalation, Mitigation (TTM), and Resolution.
4. **B** — High-quality SRE runbooks/playbooks provide copy-paste diagnostic and mitigation steps linked directly from alerting notifications.
5. **B** — Invalidating Cloud CDN caches immediately purges poisoned or stale responses from edge locations worldwide.
6. **B** — Mean Time to Detect (MTTD) measures the latency between incident inception and monitoring system notification.
7. **B** — Operational debt is the accumulation of brittle hacks, manual toil, and deferred maintenance that increases risk of outages.
8. **B** — Dedicated, scheduled stakeholder broadcasts run by the Comms Lead protect responders from operational disruption while maintaining transparency.
9. **B** — Observability systems must not share failure domains with the services they monitor; otherwise an outage blinds the responders.
10. **B** — Postmortem action items must be SMART: specific, measurable, prioritized, assigned to a single DRI, and trackable.
