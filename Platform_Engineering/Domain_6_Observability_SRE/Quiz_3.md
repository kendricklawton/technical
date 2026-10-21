# PE Domain 6: Observability & SRE — Scenario Quiz 3

Test your mastery of SRE mathematics (SLOs, burn rates), incident command, and blameless postmortems.

---

### Question 1
A service has a 30-day rolling Service Level Objective (SLO) of **99.9% availability**. The service receives 10,000,000 requests per month. What is the total allowable **Error Budget** in failed requests?

A) 100,000 requests  
B) 1,000 requests  
C) 0 requests  
D) 10,000 requests  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** D  
**Explanation:** Error Budget is $1 - \text{SLO} = 1 - 0.999 = 0.001$ (0.1%). Allowable failed requests = $10,000,000 \times 0.001 = 10,000$ requests.
</details>

---

### Question 2
In the Google SRE multi-window multi-burn-rate alerting framework, what does a **14.4x burn rate** signify?

A) The service is consuming its 30-day error budget at a rate that will completely exhaust the entire monthly budget in roughly 50 hours (and consumes 2% of the total monthly budget in just 1 hour), justifying an immediate high-priority page to the on-call engineer.  
B) Network latency has increased by 14.4ms.  
C) The CPU utilization is at 144%.  
D) The service has crashed 14 times.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** A  
**Explanation:** A 1x burn rate consumes 100% of the budget in 30 days (720 hours). A 14.4x burn rate consumes the budget 14.4 times faster: $720 / 14.4 = 50$ hours. In 1 hour, it burns $1 / 50 = 2\%$ of the monthly budget.
</details>

---

### Question 3
Why does a multi-window burn rate alert evaluate **both a long window (e.g., 1 hour) AND a short window (e.g., 5 minutes)** before firing a page?

A) It reduces Prometheus memory by 50%.  
B) The long window ensures that a significant fraction of budget has actually been burned; the short window ensures that the error condition is **actively ongoing right now**, preventing on-call engineers from being woken up for transient spikes that resolved 10 minutes ago.  
C) Because Prometheus cannot evaluate single windows.  
D) It reboots the alerting daemon.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** B  
**Explanation:** Dual-window alerting eliminates the #1 cause of alert fatigue: pages for errors that have already stopped. If a blip lasted 3 minutes and ended, the 1-hour rate might still be elevated, but the 5-minute rate drops back to zero, immediately suppressing the page.
</details>

---

### Question 4
During a Sev-1 outage, what is the primary responsibility of the **Incident Commander (IC)**?

A) Opening an IDE and debugging the Java source code directly.  
B) Writing the press release.  
C) Calling the company's legal counsel.  
D) Leading the incident response process, assigning roles (Ops Lead, Comms Lead), coordinating hypotheses, facilitating decisions, and maintaining focus without touching technical keyboards or getting trapped in weeds.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** D  
**Explanation:** The Incident Commander owns the process, not the technical keyboard. If the IC begins debugging code, they lose high-level situational awareness. The IC directs traffic, removes blockers, and ensures operational tasks are delegated cleanly.
</details>

---

### Question 5
What is the core principle of a **Blameless Postmortem**?

A) Blaming external cloud vendors for all outages.  
B) Assuming that engineers make decisions in good faith based on the information available to them at the time, and focusing on systemic, architectural, and procedural flaws rather than punishing individuals for human errors.  
C) Deleting all logs related to the failure.  
D) Rewarding developers who caused the outage.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** B  
**Explanation:** Punishing individuals drives failures underground: engineers hide mistakes and stop experimenting. Blameless culture recognizes that humans will make mistakes; the system must have guardrails (canaries, policies, tests) to prevent single human actions from causing catastrophic outages.
</details>

---

### Question 6
What is the "Five Whys" technique in root cause analysis?

A) A method for estimating cloud budgets.  
B) A five-step checklist for rebooting servers.  
C) An iterative interrogation technique that repeatedly asks "Why did this occur?" to drill past superficial symptoms and expose the fundamental systemic and organizational root cause of an incident.  
D) Asking five random engineers for their opinion.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** C  
**Explanation:** The Five Whys moves from symptom to systemic cause: "Why did service crash? OOM. Why OOM? Memory leak in cache. Why leak? Unbounded map. Why unbounded? No code review guidelines. Why no guidelines? Onboarding lacked training."
</details>

---

### Question 7
What is an **Error Budget Policy** in an SRE organization?

A) A policy allowing developers to make 10 mistakes per sprint.  
B) An insurance policy purchased from an underwriter.  
C) A formal agreement between engineering and product management dictating actions when the error budget is exhausted (e.g., pausing new feature releases to prioritize reliability, refactoring, and automated testing until the budget recovers).  
D) A tax deduction for broken software.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** C  
**Explanation:** An SLO is meaningless without consequences. The Error Budget Policy establishes explicit rules: when budget is healthy, ship fast; when budget is exhausted, reliability and platform work take priority over new product features.
</details>

---

### Question 8
In Chaos Engineering, what is a **GameDay**?

A) A day with no meetings.  
B) An engineering video game tournament.  
C) A structured, planned exercise where engineering teams intentionally inject failure scenarios (e.g., killing an entire database replica or severing a network zone) into a staging or production environment to observe system resilience and practice incident response procedures.  
D) The launch day of a new mobile app.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** C  
**Explanation:** GameDays validate resilience hypotheses under controlled conditions. Teams test if automatic failovers truly work and train engineers in calm, structured incident response before real catastrophes occur.
</details>

---

### Question 9
What does **Mean Time to Detect (MTTD)** measure in incident management?

A) The duration from the exact moment an outage or degradation begins in production until the engineering team is formally alerted and aware of the issue.  
B) The time spent conducting interviews.  
C) The time spent writing postmortems.  
D) The time it takes to reboot a server.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** A  
**Explanation:** MTTD measures detection efficacy. Low MTTD is achieved via fast, accurate synthetic monitoring, real-time telemetry, and multi-burn-rate SLO alerting.
</details>

---

### Question 10
Why must postmortem Action Items be SMART (Specific, Measurable, Achievable, Relevant, Time-bound)?

A) It reduces the length of the postmortem document.  
B) Because vague action items like "improve monitoring" or "fix documentation" are never prioritized, leaving the same systemic vulnerabilities open to trigger identical outages in the future.  
C) It converts postmortems into patents.  
D) SMART is an acronym required by the Linux kernel.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** B  
**Explanation:** Vague action items without specific owners or deadlines languish in backlogs. Action items must be concrete engineering tasks (e.g., "Add 100ms connection timeout to payment client by Nov 1 — Owner: Alice") to guarantee systemic remediation.
</details>
