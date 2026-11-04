# Domain 4: Incident Management & SRE Postmortems — Quiz 3

**Exam:** Google Cloud Certified: Professional Cloud DevOps Engineer  
**Format:** 10 Scenario-Based Multiple-Choice Questions with In-Depth Technical Explanations

---

### Question 1
A retail company experiences an outage during cyber Monday where their cloud database CPU reaches 100% due to an unexpected flood of scraper bots. The team must shed load immediately to restore service for legitimate shoppers. What is the most effective immediate mitigation?
- A. Enable Cloud Armor rate-limiting and bot-management security policies at the Global Load Balancer edge to block abusive IP ranges and throttle unauthenticated requests.
- B. Provision a new database in a different cloud provider.
- C. Re-index all database tables in production.
- D. Increase pod replica count to 500.

---

### Question 2
In the aftermath of an incident, the team identifies that the alerting system failed to page the on-call engineer because the alert threshold was set to an absolute CPU percentage rather than an SLO error budget burn rate. In the postmortem, how should this be categorized?
- A. User Error
- B. Hardware Failure
- C. Monitoring / Detection Gap
- D. Third-Party Vendor Outage

---

### Question 3
An SRE team wants to simulate real-world failure modes by randomly terminating GKE pods in production during business hours while the team is in the office. What is this practice called?
- A. Chaos Monkey / Fault Injection Testing
- B. Denial of Service attack
- C. Blue-Green Switchover
- D. Forensic Auditing

---

### Question 4
What should an Incident Commander do if an incident is taking much longer to mitigate than expected and hypotheses are proving false?
- A. Panic and hand over command to an intern.
- B. Keep repeating the same failed command.
- C. Turn off monitoring to hide the graphs.
- D. Step back, pause current actions, summarize verified facts versus unverified assumptions, and solicit alternative hypotheses from the team.

---

### Question 5
When is a postmortem considered "closed" according to Google SRE operational standards?
- A. When the incident call ends.
- B. After 1 year.
- C. When the postmortem is reviewed, blameless root causes are documented, and all high-priority (P0/P1) corrective action items are implemented or tracked with assigned owners and deadlines.
- D. As soon as the draft document is written.

---

### Question 6
During an incident, the Operations Lead suspects a corrupted persistent disk volume is causing pod crash loops on GKE. What command sequence allows the responder to inspect the disk without risking production data loss?
- A. Format the disk with `mkfs.ext4`.
- B. Restart the cluster master node.
- C. `rm -rf /mnt/data`
- D. Take a compute disk snapshot via `gcloud compute disks snapshot`, attach the snapshot as read-only to a debug VM, and inspect the filesystem out of band.

---

### Question 7
How does Google SRE distinguish between an "incident mitigation" and an "incident resolution"?
- A. They are synonyms with no operational difference.
- B. Mitigation stops the customer pain and restores SLIs to acceptable thresholds; resolution fixes the underlying bug and eliminates the root failure permanently.
- C. Mitigation happens in staging; resolution happens in production.
- D. Mitigation is done by management; resolution is done by software engineers.

---

### Question 8
What is the primary danger of having an Incident Commander who also attempts to act as the primary hands-on keyboard debugger during a critical SEV-1 outage?
- A. They lose situational awareness, become hyper-focused on one technical detail, and fail to coordinate overall response, resulting in prolonged downtime.
- B. It violates Google Cloud billing terms.
- C. It prevents Git commits from signing.
- D. They get double pay.

---

### Question 9
In a blameless postmortem review meeting, an engineering manager repeatedly asks "Who pushed that commit without testing it?" How should the postmortem facilitator intervene?
- A. Allow the manager to berate the developer.
- B. Refocus the inquiry from "who" made the mistake to "why" the CI/CD pipeline and automated guardrails permitted an untested commit to reach production without detection.
- C. Cancel the meeting immediately.
- D. Escalate to Human Resources.

---

### Question 10
An organization wants to reduce its Mean Time to Mitigate (MTTM) for recurring transient microservice deadlocks. What automation strategy provides the fastest recovery?
- A. Waiting for morning standup.
- B. Sending emails to developers when deadlocks occur.
- C. Implementing automated self-healing readiness and liveness probes in Kubernetes, backed by automated rollback triggers in Cloud Deploy.
- D. Manual SSH into VMs to run `systemctl restart`.

---

## Answer Key & Rationales

1. **A** — Dropping abusive bot traffic at the global edge via Cloud Armor prevents malicious queries from consuming database connections and protects legitimate users.
2. **C** — When monitoring fails to detect user-impacting regressions in a timely manner, it is classified as a detection/observability gap.
3. **A** — Chaos Monkey / Fault Injection tests system resilience by proactively injecting failures in a controlled, observable environment.
4. **D** — Periodically resetting situational awareness, reviewing assumptions, and soliciting alternative viewpoints prevents tunnel vision during complex outages.
5. **C** — Postmortems remain active until systemic improvements and high-priority preventive actions are completed.
6. **D** — Taking a disk snapshot and mounting it read-only on an isolated debug instance enables non-destructive forensic analysis.
7. **B** — Mitigation stops active degradation (e.g. rollback, failover); resolution eradicates the underlying bug completely.
8. **A** — The IC must remain detached from hands-on debugging to maintain high-level situational awareness, delegate tasks, and direct strategy.
9. **B** — Blameless culture shifts focus from human attribution to systemic engineering improvements that make catastrophic mistakes impossible.
10. **C** — Automated Kubernetes health probes and CI/CD automated rollback policies provide sub-minute self-healing without human intervention.
