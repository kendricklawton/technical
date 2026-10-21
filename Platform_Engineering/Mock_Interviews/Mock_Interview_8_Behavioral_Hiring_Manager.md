# Mock Interview 8: Behavioral & Platform Leadership

Simulates a 45–60 minute behavioral interview with an Engineering Director or VP of Infrastructure. Tests executive communication, influence without authority, incident ownership, and platform product mindset.

---

## Candidate Brief (Safe to read in advance)

- **Duration:** 45–60 minutes.
- **Format:** Behavioral questions evaluated using the STAR method (Situation, Task, Action, Result).
- **Core Competencies Evaluated:**
  1. Influence without authority & developer empathy.
  2. Navigating organizational resistance and shadow ops.
  3. Crisis leadership & blameless postmortem culture.
  4. Strategic platform roadmapping & technical debt negotiation.

---

## Interviewer Packet (Confidential — Do not read beforehand)

### Variant A: Driving Platform Adoption Without Mandates
- **Core Question:** "Tell me about a time when you built a new platform tool or Golden Path, but stream-aligned product engineering teams refused to adopt it and continued using legacy manual workflows. How did you diagnose the problem and turn adoption around?"
- **Interviewer Probing Ladder:**
  - *Probe 1:* "Did you ever consider asking leadership to issue an executive mandate?"
    - *Look for:* Strong candidates resist mandates; they treat developers as customers, conduct user research, find friction points, and build buy-in.
  - *Probe 2:* "How did you measure adoption quantitatively and qualitatively?"
  - *Probe 3:* "What did you do with the 10% of teams that still refused after the product improved?"

### Variant B: High-Severity Incident & Systemic Cultural Reform
- **Core Question:** "Describe the most critical production outage or security breach you personally managed. Walk me through your actions during the incident, communication with stakeholders, and the lasting organizational changes you instituted afterward."
- **Interviewer Probing Ladder:**
  - *Probe 1:* "When leadership demanded to know which engineer pushed the bad commit, how did you respond?" (Tests blameless culture and psychological safety).
  - *Probe 2:* "What technical guardrail did you put in place to ensure that exact failure mode is mathematically impossible in the future?"
  - *Probe 3:* "How did you balance fixing the postmortem action items against product delivery pressure?"

### Variant C: Managing Technical Debt & Decommissioning Legacy Systems
- **Core Question:** "Platform teams often inherit fragile, legacy systems (e.g., self-hosted Jenkins clusters or unmaintained Kubernetes 1.18 clusters). Tell me about how you successfully migrated hundreds of engineers off a legacy system and safely decommissioned it."
- **Interviewer Probing Ladder:**
  - *Probe 1:* "How did you prevent the migration from stalling at the 'long tail' (the last 10% of stubborn services)?"
  - *Probe 2:* "How did you calculate and communicate the ROI of this migration to the CFO / non-technical executives?"

---

## Scoring Rubric (STAR Alignment)

| Score | Anchor | Behavioral Criteria |
| :---: | :--- | :--- |
| **1** | Strong No Hire | Blames other teams or junior engineers; advocates executive mandates to force adoption; uses vague "we" statements without owning actions. |
| **2** | No Hire | Answers lack structure; describes technical actions without business context or developer empathy; struggles with conflict resolution. |
| **3** | Hire | Crisp STAR responses; clearly articulates personal leadership actions; demonstrates high empathy for product engineers; embraces blameless SRE culture. |
| **4** | Strong Hire | Executive-level maturity; quantifies impact using business and DORA metrics; demonstrates proven playbook for cultural transformation and platform evangelism. |

---

## Self-Debrief Template
- **Stories Prepared:** Adoption / Incident / Tech Debt
- **STAR Crispness Score (1-4):**
- **Was Impact Quantified? (Metrics/DORA):** Yes / No
- **Action Items:**
