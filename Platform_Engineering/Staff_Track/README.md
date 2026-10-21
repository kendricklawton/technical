# Staff Track: October 2026 → October 2029

The rest of this folder prepares you for a platform interview. This track covers the three years after it: growing from the role you land into a staff-level skill set by October 2029.

Staff is a scope of work, not a level of knowledge. Staff engineers are judged on what changed because of them: problems they framed that nobody owned, decisions that teams they don't lead adopted, systems that became more reliable or cheaper, and engineers who grew. Quizzes can't show any of that, so this track measures progress with **evidence**: artifacts you can point to.

> [!IMPORTANT]
> **The biggest lever isn't in this folder.** You can only practise staff-level scope where staff-level problems exist. When you pick your next job, favour a growing platform org with real scale and staff engineers you can learn from (see [§7](#7-choosing-the-next-job)). If by mid-2028 no multi-team problem is open to you, treat that as the signal to change teams or companies.

---

## Staff SRE & Platform Engineering Core Playbooks

To bridge the gap between theoretical knowledge and real production operating depth, review and reference these production-grade staff artifacts:

| Staff Artifact | Focus & Depth | Why Staff Engineers Need It |
| :--- | :--- | :--- |
| **[Staff SRE Production Playbook](Staff_SRE_Playbook.md)** | Linux memory/OOM triage, CFS CPU throttling, TCP socket buffers, eBPF tracing one-liners, and Google MWMBR PromQL alerting rules | Definitive systems engineering reference for kernel-level debugging, latency mitigation, and statistical alerting math. |
| **[Incident War Room Postmortems](../Incident_War_Room_Postmortems.md)** | Five comprehensive SEV-1 production incident post-mortems (Conntrack table saturation, etcd fsync starvation, CFS CPU throttling, DB thundering herd retry storms, BGP asymmetric routing) | Learn how to lead high-stakes war rooms, isolate subtle systemic failure modes, and formulate blameless postmortems with high-impact action items. |
| **[Architecture Decision Records (ADRs)](../Architecture_Decision_Records.md)** | Five production ADRs (Backstage vs Port, vCluster vs Mega-clusters, Crossplane vs OpenTofu, Gateway API vs Lattice, OTel Tail-Based Sampling vs Direct Ingestion) | Master how to structure organization-wide technical proposals, weigh architectural trade-offs, and steer cross-team consensus. |

---

## 1. How Progress Is Measured

Seven dimensions, extending the Mid / Senior / Staff table in [Domain 10 §7.1](../Domain_10_Behavioral_Career/Study_Guide.md). Score each one quarterly: **1** = mid, **2** = senior, **3** = staff. A score only counts if you can link the artifact behind it.

| Dimension | What counts as evidence | Staff-level example |
| :--- | :--- | :--- |
| **Scope** | The largest problem you owned end to end, with its outcome metric | A fleet upgrade program across 40 clusters, finished with zero customer-facing incidents |
| **Ambiguity** | A problem you framed before anyone asked you to | A problem statement showing that CI queue time, not test time, drives lead time |
| **Influence** | A decision adopted by teams you don't lead | An RFC for a new deployment standard, adopted by 12 of 15 product teams |
| **Depth** | Something in your specialty that only you could design or debug | The root cause of a kernel-level networking stall, plus the upstream fix |
| **Writing** | Documents that changed a decision | A strategy doc that moved a quarter's roadmap |
| **Multiplying others** | People who now do without you what used to need you | A mentee who now runs the incident process |
| **Operating** | Incidents commanded, postmortems written, SLOs owned | Incident commander for a Sev-1, with a postmortem whose action items all closed |

**Quarterly review (1 hour, last week of each quarter):**
1. Add an entry to the **[Evidence Log](Evidence_Log.md)** and score all seven dimensions.
2. Check the current year's exit gate (below). Anything at risk?
3. Choose at most **three** goals for next quarter. More than three means none get done.

---

## 2. The Three Years at a Glance

| Year | Dates | Theme | Exit gate (summary) |
| :---: | :--- | :--- | :--- |
| 1 | Oct 2026 – Sep 2027 | Get hired, then own something | One capability shipped end to end; specialty chosen |
| 2 | Oct 2027 – Sep 2028 | Multi-team scope | A finished cross-team initiative; a strategy doc that changed a roadmap |
| 3 | Oct 2028 – Sep 2029 | Org-level direction | A staff packet: three org-scope stories, each with an artifact and a metric |

---

## 3. Year by Year

### 3.1 Year 1: Get Hired, Then Own Something (Oct 2026 – Sep 2027)

| Quarter | Focus | Do this |
| :--- | :--- | :--- |
| **Q1** (Oct 2026 – Jan 2027) | The 12-week plan | Run the [README plan](../README.md#12-week-study-plan--progress-tracker) through mid-January. Alongside it, start the long-running platform (§4.1) with the [Mock 9 take-home](../Mock_Interviews/Mock_Interview_9_Take_Home_Presentation.md) as its seed, move the Domain 9 problems into real tested code (§4.2), and start the [Evidence Log](Evidence_Log.md). Apply once the readiness gates are met. |
| **Q2** (Jan – Mar 2027) | Interview and choose well | Use the job criteria in §7. Keep the platform running. Start the writing cadence (§4.3): one document a month from now on. |
| **Q3** (Apr – Jun 2027) | The first 90 days | Learn the system before changing it. Join on-call, fix three paper cuts other teams complain about, and write a "state of the platform as I found it" doc for your manager. **Choose your specialty by the end of June** (§5). |
| **Q4** (Jul – Sep 2027) | Own a capability | Take one platform capability end to end (a golden path, the CI system, the cluster upgrade process) and ship a measurable improvement. Run your first staff drill (§4.4). |

**Year 1 exit gate (September 2027):**
- [ ] One capability shipped end to end, with a before/after metric
- [ ] Two design docs adopted at work
- [ ] Incident commander for at least two incidents, with the postmortems written
- [ ] One person mentored in something you're good at
- [ ] Specialty chosen, with one deep project started in it

### 3.2 Year 2: Multi-Team Scope (Oct 2027 – Sep 2028)

- **Lead one initiative across three or more teams.** A migration or deprecation is ideal because it forces influence without authority, for example a CI migration, a fleet upgrade program, or Ingress → Gateway API (see §6.2).
- **Write a technical strategy for one area** of the platform, using diagnosis → guiding policy → coherent actions (§6.3).
- **Go public in your specialty:** get one upstream contribution merged, and give one talk or publish one post (a local meetup counts).
- **Mentor two engineers**, and join or run your team's design review.
- **Have the staff conversation.** Ask your manager what staff means at your company and what evidence the promotion committee expects. Get the answer in writing.

**Year 2 exit gate (September 2028):**
- [ ] A finished cross-team initiative with adoption metrics ("launched" doesn't count)
- [ ] A strategy doc that changed a roadmap
- [ ] Known as the go-to person for your specialty outside your own team
- [ ] Two mentees, one talk or post, one merged upstream contribution
- [ ] Your manager agrees in writing on the gap between you and staff

### 3.3 Year 3: Org-Level Direction (Oct 2028 – Sep 2029)

- **Set multi-quarter technical direction** for an area of the platform org, and get it adopted by teams you don't lead.
- **Find a problem nobody owns** (cost, reliability, developer experience), frame it, and turn it into a funded initiative.
- **Sponsor, don't just mentor:** put a senior engineer forward for bigger scope, and make sure they get credit.
- **Assemble your staff packet:** three or four stories at org scope, each following *problem framed → direction set → others' behavior changed → outcome measured*, each with a written artifact. Use it for a promotion case or a staff-level interview loop.

**Year 3 exit gate (September 2029):**
- [ ] A staff promotion, or staff-level offers from external loops
- [ ] A packet with three org-scope stories, each backed by an artifact and a metric

---

## 4. Workstreams (All Three Years)

### 4.1 The Long-Running Platform

The labs in this folder are build-and-delete: you never operate anything. Staff judgment comes from running systems over time, through upgrades, outages and your own past decisions. So run one small platform continuously.

| Phase | Add |
| :--- | :--- |
| **Year 1** | k3s or Talos (one to three nodes), GitOps (Argo CD or Flux), OpenTofu or Terraform for everything outside the cluster, Prometheus + Grafana + OpenTelemetry Collector, SLOs with burn-rate alerts that reach your phone, and one or two real workloads (your own services count) |
| **Year 2** | Admission policy (Kyverno or ValidatingAdmissionPolicy), secrets (External Secrets with OpenBao or Vault), signed images and SBOMs (cosign, Syft), cost visibility (OpenCost), backups with a quarterly restore drill, a second cluster |
| **Year 3** | Fleet management across clusters, monthly game days, and a thin developer interface on top (a CLI or a portal page) |

**Operating rules:**
- Upgrade Kubernetes every minor release (three a year), and write down what broke.
- Every outage gets a postmortem using the [Domain 6 Lab 6 template](../Domain_6_Observability_SRE/Exercises.md). These become interview stories.
- Keep a `CHANGELOG.md` for the platform and an ADR for every non-obvious choice.
- **Hardware:** the laptop can't stay up 24/7. A small always-on machine or a capped cloud budget both work. Don't overbuild: three nodes is plenty.

### 4.2 Real Code

| When | What |
| :--- | :--- |
| Q1 2027 | Move the eight [Domain 9 problems](../Domain_9_Coding_Automation/Exercises.md) into a Go module and a pytest project with real tests, run by `make test`. |
| Year 1 | A Kubernetes operator built with kubebuilder and tested with envtest, managing something real on your platform. |
| Year 2 | A CLI your platform's users would actually want, with versioned releases. |
| Years 2–3 | Upstream contributions in your specialty (Kubernetes SIGs, OpenTelemetry, Argo, Crossplane, Cilium, and so on). |

### 4.3 Writing

Staff engineers are judged mostly on written work. **Write one document a month**, rotating through these types:

| Type | Use it for | Template |
| :--- | :--- | :--- |
| Problem statement | Framing something nobody owns yet | One page: symptom, evidence, cost of doing nothing, who is affected |
| Design doc / RFC | A change other teams must review | [Domain 1 §9.2](../Domain_1_Platform_Foundations_IDP/Study_Guide.md) |
| ADR | Recording a decision and its context | [Domain 1 §9.2](../Domain_1_Platform_Foundations_IDP/Study_Guide.md) & [ADR Repository](../Architecture_Decision_Records.md) |
| Postmortem | Learning from an outage | [Domain 6 Lab 6](../Domain_6_Observability_SRE/Exercises.md) & [War Room Postmortems](../Incident_War_Room_Postmortems.md) |
| Migration plan | Moving many teams off something | §6.2 |
| Strategy | Choosing what *not* to do for a year | §6.3 |
| Executive memo | Asking for money or headcount | [Domain 10 §10.1](../Domain_10_Behavioral_Career/Study_Guide.md) |

**Review rubric** (score each 1–3 before you share the doc):
1. The problem and context are clear to someone outside your team.
2. At least two real options are compared, including one you'd defend even though you didn't pick it.
3. The decision and its reasons are explicit.
4. Risks, rollback and what you're *not* doing are stated.
5. Success is measurable.
6. The length fits the reader: an executive gets one page.

### 4.4 Staff Drills

Mock rounds aimed at staff-level loops. Run one per quarter from Q4 2027, and more before a staff interview. To start one, tell the AI interviewer: *"Run Staff Drill 2 from Platform_Engineering/Staff_Track/README.md at staff level, strict mode. Follow Platform_Engineering/Mock_Interviews/Interviewer_Protocol.md."* Log results in the [Mock Interview Log](../Mock_Interviews/Mock_Interview_Log.md).

| Drill | Length | Prompt | What's scored |
| :--- | :---: | :--- | :--- |
| **1. Project deep dive** | 60 min | "Walk me through the most complex platform project you led." The interviewer drills five levels down: why this problem, what alternatives, what went wrong, what you'd change, how you measured it. | Ownership, depth, honesty about mistakes, measured outcomes |
| **2. Ambiguous problem** | 60 min | "Lead time for changes has doubled over two quarters and nobody knows why. You have read access to everything. What do you do in your first 30 days?" | Framing, data before opinions, a stakeholder map, a decision, a communication plan |
| **3. Design-doc critique** | 45 min | The interviewer writes a two-page design doc with five planted problems. Review it as you would at work. | What you flag first, tone, how actionable the feedback is |
| **4. Strategy defense** | 45 min | Present a strategy doc you wrote. The interviewer plays a skeptical VP, then a peer staff engineer. | Quality of diagnosis, trade-offs, handling pushback without caving or digging in |

---

## 5. Choose a Depth Specialty (by June 2027)

Staff means deep in one place and broad everywhere else. Pick the specialty that (1) your new job has real problems in, (2) will matter more in 2029 than today, and (3) you're happy to debug at 2 a.m.

| Specialty | Why it's a good bet | First deep project |
| :--- | :--- | :--- |
| **AI/ML infrastructure** | Most platform orgs will run GPU fleets and inference by 2029, and few platform engineers understand both scheduling and serving | Serve an open model on your platform with vLLM or KServe behind the Gateway API Inference Extension; model GPU sharing with Dynamic Resource Allocation (DRA) |
| **Networking and eBPF** | The hardest outages end up here, and deep skills are rare | Replace kube-proxy with Cilium on your platform; write a small eBPF tool that answers a question you actually had |
| **Delivery and supply chain** | Every company needs it, and regulation (such as the EU Cyber Resilience Act) keeps raising the bar | Signed build provenance for your platform's images, verified at admission |
| **Reliability and observability** | Direct line to business outcomes; the natural fit for the staff "solver" archetype | An SLO program for your platform with an error-budget policy and end-to-end tracing |
| **Control planes and platform APIs** | Platforms are becoming APIs, and this is where self-service gets built | A platform API (a Crossplane v2 composition or your own operator) that provisions a database end to end |

---

## 6. Subjects Beyond the Ten Domains

### 6.1 Distributed Systems Depth

The ten domains touch these topics. Staff "solvers" need them cold: consensus and quorum (etcd), consistency models, queueing (Little's law, why latency explodes near full utilization), backpressure and load shedding, and metastable failures.

- **Read:** *Designing Data-Intensive Applications* (2nd ed., Kleppmann & Riccomini, 2026), plus one paper a month in Years 1–2:
  - "Large-scale cluster management at Google with Borg" (Verma et al., 2015)
  - "Borg, Omega, and Kubernetes" (Burns et al., 2016)
  - "In Search of an Understandable Consensus Algorithm" (Raft; Ongaro & Ousterhout, 2014)
  - "The Tail at Scale" (Dean & Barroso, 2013)
  - "On Designing and Deploying Internet-Scale Services" (Hamilton, 2007)
  - "How Complex Systems Fail" (Cook, 1998)
  - "Metastable Failures in Distributed Systems" (Bronson et al., 2021)
- **Practice:** for each paper, add a short note to the Evidence Log: the claim, what it means for your platform, and one thing you'd change because of it.

### 6.2 Large-Scale Migrations

Migrations are the purest test of influence without authority, and every platform org has several running.

- **Playbook:** inventory and segment users → build the paved path first → automate the move (codemods, bot PRs) → lighthouse teams → a published deprecation calendar → brownouts → chase the long tail → define and measure "done". Domain 1 §7 and the migration questions in the Domain 4 and 5 interview banks work through examples.
- **Read:** Will Larson's essay "Migrations: the sole scalable fix to tech debt" (also in *An Elegant Puzzle*).
- **Practice:** write a migration plan for your platform. Ingress → Gateway API is a timely choice: ingress-nginx was retired in March 2026, so many companies are doing exactly this migration now.

### 6.3 Technical Strategy and Influence

- **Read:** *Good Strategy Bad Strategy* (Rumelt, 2011), *Crafting Engineering Strategy* (Larson, 2025), *An Elegant Puzzle* (Larson, 2019), and Part III of *The Staff Engineer's Path*.
- **Practice:** the Year 2 strategy doc. Use Rumelt's kernel: a **diagnosis** of what's really going on, a **guiding policy** that rules options out, and **coherent actions** that reinforce each other.

### 6.4 AI/ML Infrastructure Literacy

Worth knowing even if it isn't your specialty:
- GPU scheduling and sharing (DRA, MIG, time-slicing)
- Batch queueing (Kueue) and model serving (vLLM, KServe)
- Inference-aware routing (Gateway API Inference Extension)
- LLM gateways with per-team quotas and cost controls
- How to measure whether AI coding tools actually help your developers

**Read:** *AI Engineering* (Chip Huyen, 2025) for the application side, plus the Kubernetes docs for DRA and Kueue. **Practice:** serve a small quantized model on your platform. CPU inference is fine for learning.

### 6.5 Platform Product and People

- **Read:** *Platform Engineering* (Camille Fournier & Ian Nowland, 2024), *The Manager's Path* (Fournier, 2017; to understand the partner you'll work with most), and *Thinking in Systems* (Meadows, 2008).
- **Practice:** interview five of your platform's users each half-year, and join your company's hiring loop as an interviewer in Year 1 or 2.

---

## 7. Choosing the Next Job

Your next job decides how much staff-shaped work you can get. In the Q2 2027 interviews, ask:

| Ask | Good sign | Red flag |
| :--- | :--- | :--- |
| Who uses the platform? | 100+ engineers as internal customers | The platform team works a ticket queue |
| Are there staff+ engineers near the platform? | Yes, and you can talk to one during the loop | No one above senior, or staff granted for tenure |
| What does staff mean here, and how many platform engineers got there in the last two years? | A written ladder and recent examples | "We don't really do levels" |
| What's changing in the platform this year? | New capabilities or migrations underway | Maintenance mode |
| How are decisions made? | Design docs and reviews | Whoever argues longest |
| Is there on-call? | Yes, with a sane rotation | No production exposure at all |

---

## 8. Using This Track with Claude

| Goal | Prompt |
| :--- | :--- |
| Quarterly review | "Read Platform_Engineering/Staff_Track/Evidence_Log.md and run my quarterly review using §1 of Staff_Track/README.md." |
| Doc critique | "Critique this design doc as a skeptical staff engineer, using the rubric in Staff_Track/README.md §4.3." |
| Staff drill | "Run Staff Drill 2 at staff level, strict mode, following Mock_Interviews/Interviewer_Protocol.md." |
| Paper discussion | "I read 'The Tail at Scale'. Quiz me on it, then ask how it applies to my platform." |
