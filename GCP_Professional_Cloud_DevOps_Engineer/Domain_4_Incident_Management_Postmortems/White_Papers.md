# Domain 4: Managing Service Incidents — Authoritative White Papers & SRE Standards

> **Exam Focus:** Incident Response Best Practices, Blameless Culture, and Effective On-Call Rotations.

---

## 1. Authoritative References & Standards

- **[Google SRE Book — Managing Incidents](https://sre.google/sre-book/managing-incidents/):** Incident command structure, delegation, and communication protocols.
- **[Google SRE Book — Postmortem Culture: Learning from Failure](https://sre.google/sre-book/postmortem-culture/):** Blameless philosophy, psychological safety, and action-oriented remediation.
- **[Google SRE Workbook — Being On-Call](https://sre.google/workbook/on-call/):** Paging etiquette, operational hygiene, and preventing burnout.
- **[Google Cloud Architecture Framework — Incident Response](https://cloud.google.com/architecture/framework/operational-excellence/incident-response):** Automating alerts and communication channels.

---

## 2. Staff SRE Architectural Patterns

### 2.1 The "5 Whys" Root Cause Analysis
During an incident postmortem, engineers must dig deeply into underlying systems vulnerabilities:
- *Symptom:* The web service crashed at 14:02.
  - *Why 1:* Memory exhausted on worker nodes.
  - *Why 2:* Pods were handling 10x normal request size.
  - *Why 3:* A batch client sent uncompressed 50MB payloads to `/v1/upload`.
  - *Why 4:* The API Gateway lacked max payload size limits.
  - *Why 5:* Payload size constraints were never documented in the API specification.
- *Remediation:* Enforce max request payload body limit (10MB) in Cloud Armor / API Gateway and configure automated contract testing in CI.

### 2.2 Effective On-Call Rotations
- **Two In-Rotation Minimum:** Maintain a primary and secondary on-call engineer for every rotation.
- **Compensation & Recovery:** Provide compensatory time off after grueling nighttime pages.
- **Follow-the-Sun:** In global organizations, hand off paging shifts across time zones (e.g. US, EMEA, APAC) to minimize nighttime pages.

---

## 3. Production Failure Modes & Anti-Patterns

| Anti-Pattern | Operational Risk | Production Resolution |
| :--- | :--- | :--- |
| **Finger-Pointing & Blame** | Punishing engineers who trigger outages breeds secrecy, fear, and prevents team members from admitting mistakes. | Enforce strict blameless postmortem guidelines. Celebrate near-miss reports and reward proactive disclosures of vulnerabilities. |
| **Debugging During Active Outages** | Responders spend 60 minutes running gdb or inspecting code while customers suffer downtime. | Prioritize mitigation: rollback, restart, shed load, or divert traffic first. Debug the root cause offline in a staging sandbox. |
| **Orphaned Action Items** | Postmortem documents are written and forgotten, leaving the same bugs to trigger repeat outages. | Track postmortem action items as high-priority P0/P1 JIRA tickets with strict SLA due dates reviewed at weekly executive reliability meetings. |
