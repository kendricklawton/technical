# Platform Engineering Master Cheat Sheet — Interview Night-Before Reference

The one file to reread the night before a Platform Engineer loop and to skim between rounds. It covers definitions you can say out loud, the frameworks interviewers name-drop, numbers worth memorizing, real commands, a system design scaffold, troubleshooting trees, behavioral structure, and day-of logistics. Every section links to the domain material for depth.

> [!IMPORTANT]
> **How to use this sheet**
> - **Night before:** read sections 1, 2, 3, 8 and 9 once. Say the section 1 definitions and your 90-second pitch (7.3) out loud, then stop. Sleep is worth more than one more pass.
> - **Between rounds:** open only the section for the next round (map below). Spend five minutes, not twenty.
> - **Facts checked 2026-10-19** against primary sources: dora.dev, slsa.dev, sre.google, kubernetes.io, the CNCF maturity model, opengitops.dev, finops.org, and the Helm and Argo CD docs. Commands assume kubectl 1.37, Helm 4.3, Terraform 1.15, OpenTofu 1.12, Argo CD 3.4.

| Next round | Skim here | Go deeper |
|---|---|---|
| Recruiter / phone screen | §1, §2, §8 | [Mock 1: Phone Screen](Mock_Interviews/Mock_Interview_1_Phone_Screen.md) |
| Live troubleshooting | §3.5–3.8, §4.1–4.3, §6 | [Domain 2](Domain_2_Linux_Networking_Fundamentals/Study_Guide.md), [Mock 2: Troubleshooting](Mock_Interviews/Mock_Interview_2_Troubleshooting.md) |
| Kubernetes deep dive | §3.4, §4.3, §6.1–6.5 | [Domain 3](Domain_3_Kubernetes_Platform_Engineering/Study_Guide.md), [Mock 3: Kubernetes](Mock_Interviews/Mock_Interview_3_Kubernetes_Deep_Dive.md) |
| IaC / CI/CD deep dive | §2.7–2.8, §4.4–4.6, §6.7–6.8 | [Domain 4](Domain_4_Infrastructure_as_Code/Study_Guide.md), [Domain 5](Domain_5_CICD_GitOps_Delivery/Study_Guide.md), [Mock 4: IaC and CI/CD](Mock_Interviews/Mock_Interview_4_IaC_CICD_Deep_Dive.md) |
| System design (IDP or infrastructure) | §5, §3, §2 | [Domain 8](Domain_8_System_Design/Study_Guide.md), [Mock 5: IDP](Mock_Interviews/Mock_Interview_5_System_Design_IDP.md), [Mock 6: Infrastructure](Mock_Interviews/Mock_Interview_6_System_Design_Infrastructure.md) |
| Coding / automation | §4, reflexes 26–27 in §8 | [Domain 9](Domain_9_Coding_Automation/Study_Guide.md), [Mock 7: Coding](Mock_Interviews/Mock_Interview_7_Coding.md) |
| Behavioral / hiring manager | §7, §8 | [Domain 10](Domain_10_Behavioral_Career/Study_Guide.md), [Mock 8: Hiring Manager](Mock_Interviews/Mock_Interview_8_Behavioral_Hiring_Manager.md) |
| Take-home presentation | §5.2, §7 | [Mock 9: Take-Home](Mock_Interviews/Mock_Interview_9_Take_Home_Presentation.md) |

---

## 1. Thirty-Second Definitions You Can Say Out Loud

Lead with "Say this", then add one senior nuance. Each should take under 30 seconds.

| Term | Say this | Senior add-on |
|---|---|---|
| **Platform engineering** | Building and running an internal platform of self-service capabilities and golden paths that lets product teams build, ship, and operate software with less cognitive load, run as a product for internal developers. | It turns tickets into APIs. Judge it by outcomes (lead time, onboarding time, adoption, developer satisfaction), not tool count. |
| **IDP (Internal Developer Platform)** | The integrated set of self-service APIs, templates, pipelines, and infrastructure automation that a platform team provides, so developers can provision and deploy within guardrails without filing tickets. | "IDP" also gets used for Internal Developer *Portal*. Ask which one they mean, then cover both. |
| **Developer portal vs platform** | The portal (for example Backstage) is the front door: catalog, templates, docs, scorecards. The platform is the engine behind it: APIs, controllers, pipelines, infrastructure. | A portal in front of manual processes is just a catalog of ticket forms. You can ship a platform with only Git and a CLI, so build the backend capability first. |
| **Golden path / paved road** | An opinionated, fully supported route for a common job (new service to production). It is the easiest way, not the only way. "Golden path" is Spotify's term; "paved road" is Netflix's. | Optional but compelling. Teams can go off-path if they own the consequences. Track the share of services on the path. |
| **Thinnest Viable Platform (TVP)** | From Team Topologies: the smallest set of APIs, docs, and tools that actually speeds teams up. Sometimes it's just a wiki page of conventions. | Stops you building a platform nobody asked for. Start thin, grow on demand, and delete parts nobody uses. |
| **Platform as a product** | Developers are customers. That means product management, user research, a roadmap, docs, SLOs, a deprecation policy, and adoption metrics. The platform has to earn adoption. | Mention discovery (interviews, journey mapping), surveys plus telemetry, and the discipline to say no. |
| **Cognitive load** | The mental effort a team has to spend. Team Topologies borrows Sweller's three types: intrinsic (the problem itself), extraneous (friction from tooling and process), germane (domain learning). | A platform cuts extraneous load so teams can spend effort on germane load. That is the core justification for a platform team. |
| **GitOps** | Desired state is declared, versioned and immutable in Git, pulled automatically by agents, and continuously reconciled (the four OpenGitOps principles). | Pull-based means CI holds no cluster credentials, Git history is the audit log, and rollback is a revert. Watch secrets, ordering, and drift during incidents. |
| **Reconciliation loop** | A controller observes actual state, diffs it against desired state, acts to converge, and repeats. It is level-triggered: it acts on current state, so missed events self-correct. | Actions must be idempotent. Requeue with backoff, and report progress through `status.conditions` and `observedGeneration`. |
| **Operator** | A Kubernetes controller plus CRDs that encodes operational knowledge for one application: install, upgrade, backup, failover. | Writing one means owning a distributed system's lifecycle. Look for a managed service or a mature operator first. |
| **SLI / SLO / SLA** | SLI: the ratio of good events to valid events (for example, requests under 300 ms that aren't 5xx). SLO: a target for an SLI over a window (99.9% over 30 days). SLA: a contract with penalties, set looser than the SLO. | Measure SLIs as close to the user as you can, keep only a few per service, and pair each SLO with an error budget policy. |
| **Error budget** | 1 − SLO: the unreliability you're allowed to spend in the window. 99.9% over 30 days is 43.2 minutes of full-outage equivalent. Once it's spent, priority shifts to reliability work. | Without a policy that product and engineering leadership have signed, the budget is only a number. |
| **Toil** | Work tied to running a production service that is manual, repetitive, automatable, tactical, has no enduring value, and scales linearly with the service (Google SRE book). Google SRE aims to keep it under 50% of each engineer's time. | Measure it (tickets, pages, hours) and automate the biggest sources first. Not all operational work is toil. |
| **Blast radius** | How far damage spreads when a component fails or a change is bad: which users, tenants, regions, and services. | Shrink it with cells, shuffle sharding, staged rollouts (one cell or region first), per-tenant limits, and narrowly scoped credentials. |
| **Cell-based architecture** | Split the system into independent full-stack copies (cells), each serving a subset of customers behind a thin routing layer. A failure or bad deploy then hits only one cell. | The costs: the router becomes critical, cross-cell operations get harder, and you need tooling for cell sizing and tenant migration. Source: the AWS whitepaper "Reducing the Scope of Impact with Cell-Based Architecture". |
| **Static stability** | The system keeps working in its pre-failure mode when a dependency fails, without having to change anything. Capacity is already provisioned in every AZ, and the data plane keeps serving while the control plane is down. | AWS Builders' Library term. Design data planes so they never need a control-plane call to keep serving. |
| **Drift** | Live state has diverged from declared state (Git or IaC), through manual changes, out-of-band automation, or provider-side defaults. | Detect it (`terraform plan -refresh-only`, Argo CD OutOfSync), then revert it or codify it. Repeated drift usually means a missing self-service path. |
| **Idempotency** | Running an operation several times has the same effect as running it once. | Required for retries, reconcilers, and at-least-once delivery. Put idempotency keys on mutating APIs. |
| **Progressive delivery** | Release gradually (canary, blue/green, feature flags), with automated analysis gates and automatic rollback when SLIs regress. | Decouple deploy from release. Tools: Argo Rollouts, Flagger, feature-flag services. |
| **SBOM / provenance / SLSA** | SBOM: an inventory of an artifact's components (SPDX or CycloneDX). Provenance: a verifiable attestation of how, where, and from what source it was built (in-toto format). SLSA: graded levels of supply-chain integrity. v1.2 has Build L0–L3 and Source L1–L4. | The SBOM answers "what's in it?" and provenance answers "where did it come from?". Sign both (Sigstore cosign) and verify them at admission time. |
| **Workload identity** | Workloads get short-lived, automatically rotated credentials bound to their identity instead of static secrets. Examples: EKS Pod Identity or IRSA, GKE Workload Identity Federation, Microsoft Entra Workload ID, SPIFFE/SPIRE. | Removes long-lived keys. CI uses the same idea: GitHub Actions OIDC calling `sts:AssumeRoleWithWebIdentity`. |
| **Showback vs chargeback** | Showback reports each team's cost without billing them. Chargeback moves the cost into their budget. | Start with showback and allocation (tags, labels). Chargeback needs high allocation accuracy and agreed rules for shared costs, such as splitting a shared cluster by requests. |

**Bonus terms that come up in the same breath**

| Term | Say this |
|---|---|
| **Control plane vs data plane** | The control plane decides and configures (APIs, schedulers, reconcilers). The data plane does the work (serving traffic, running pods). The data plane should keep working while the control plane is down. |
| **Shuffle sharding** | Give each tenant a small random subset of workers, so two tenants rarely share all of theirs and one bad tenant can't take everyone down (AWS Builders' Library). |
| **Inner loop vs outer loop** | Inner loop: code, build, and test locally, in seconds to minutes. Outer loop: PR, CI, review, deploy, operate. Platforms speed up both, but the outer loop is where tickets hide. |
| **Guardrails vs gates** | Guardrails are automated, always-on limits that keep you safe without anyone asking (policy as code, quotas). Gates are manual approvals. Mature platforms swap gates for guardrails. |

Deeper: [Domain 1 Study Guide](Domain_1_Platform_Foundations_IDP/Study_Guide.md) · [Domain 1 Interview Questions](Domain_1_Platform_Foundations_IDP/Interview_Questions.md)

---

## 2. Frameworks at a Glance

### 2.1 Team Topologies (Skelton and Pais)

| Team type | Purpose | Where platform engineering fits |
|---|---|---|
| **Stream-aligned** | Owns a flow of work end to end (product, service, user journey). The default team type | Your customers |
| **Platform** | Provides internal services that reduce stream-aligned teams' cognitive load, consumed as a service | You |
| **Enabling** | Specialists who coach teams through adopting a new capability, then step away | Partner with them to drive adoption (SRE practices, security tooling) |
| **Complicated-subsystem** | Owns a component that needs deep specialist knowledge (codec, ML ranking, pricing engine) | Keeps scarce expertise from being spread thin |

| Interaction mode | Use it when | Smell if it lasts too long |
|---|---|---|
| **Collaboration** | Two teams are discovering a new capability or boundary together. High bandwidth, time-boxed | Ownership blurs, and the platform team turns into an extension of one product team |
| **X-as-a-Service** | The capability is well defined, and consumers self-serve with minimal coordination | The service stops fitting real needs because nobody acts on feedback |
| **Facilitating** | Coaching and unblocking another team (the enabling team's main mode) | The facilitator ends up doing the work |

> [!TIP]
> **Interview Angle:** Say that the platform team's steady-state mode with stream-aligned teams is X-as-a-Service, with short collaboration phases to discover new capabilities. Team boundaries should follow cognitive load. Conway's Law cuts both ways: org design shapes the architecture you get.

### 2.2 DORA Software Delivery Metrics (five since the 2024 report)

| Metric | Group | Definition (dora.dev) | Platform levers |
|---|---|---|---|
| **Change lead time** | Throughput | Time for a change to go from committed to version control to deployed in production | Fast CI, trunk-based development, paved pipelines, automated approvals |
| **Deployment frequency** | Throughput | Number of deployments over a period, or the time between deployments | Small batches, self-service deploys, progressive delivery |
| **Failed deployment recovery time** | Throughput | Time to recover from a deployment that fails and requires immediate intervention | One-click rollback, feature flags, good telemetry |
| **Change fail rate** | Instability | Ratio of deployments that require immediate intervention following a deployment | Automated tests, canary analysis, policy checks |
| **Deployment rework rate** | Instability | Ratio of deployments that are unplanned but happen as a result of an incident in production | Quality gates, smaller changes, better pre-production signal |

> [!CAUTION]
> **Dated answers that cost points:** "the four keys" and "MTTR". In 2023, MTTR was redefined as **failed deployment recovery time**, which counts only failures caused by a change. In 2024, **deployment rework rate** was added and recovery time moved into the throughput group. Benchmarks are re-derived from each year's survey, so describe top performers directionally (on-demand deploys, lead time and recovery measured in hours, low failure rates) rather than quoting thresholds from memory. Never use DORA metrics to rank teams or individuals.

### 2.3 SPACE (Forsgren, Storey, Maddila, Zimmermann, Houck, Butler, 2021)

| Dimension | Example signals |
|---|---|
| **S**atisfaction and well-being | Satisfaction surveys, burnout risk, how likely developers are to recommend the tooling |
| **P**erformance | Outcomes such as quality, reliability, and customer impact (not individual output) |
| **A**ctivity | Counts of PRs, builds, deploys (never meaningful on their own) |
| **C**ommunication and collaboration | Review turnaround, onboarding time, how easily people find docs and owners |
| **E**fficiency and flow | Handoffs, wait time, interruptions, time in flow |

Rule of thumb: measure at least three dimensions, and mix survey (perception) data with system (workflow) data.

### 2.4 DevEx (Noda, Storey, Forsgren, Greiler, 2023)

| Dimension | What it means | Measure with |
|---|---|---|
| **Feedback loops** | How fast and how clearly the system responds to developer actions | CI duration, review latency, deploy time, local build time, plus perceived satisfaction with each |
| **Cognitive load** | How much mental processing the work demands | Perceived complexity of tools and docs, tool count for a common task, time to find answers |
| **Flow state** | Whether people can focus without interruption | Interruptions, meeting load, unplanned work, perceived focus time |

### 2.5 CNCF Platform Engineering Maturity Model (TAG App Delivery)

| Aspect | L1 Provisional | L2 Operational | L3 Scalable | L4 Optimizing |
|---|---|---|---|---|
| **Investment** | Voluntary or temporary | Dedicated team | As product | Enabled ecosystem |
| **Adoption** | Erratic | Extrinsic push | Intrinsic pull | Participatory |
| **Interfaces** | Custom processes | Standard tooling | Self-service solutions | Integrated services |
| **Operations** | By request | Centrally tracked | Centrally enabled | Managed services |
| **Measurement** | Ad hoc | Consistent collection | Insights | Quantitative and qualitative |

Use it as a profile across aspects, not a single score. For example: "Investment is at L3, Adoption at L2 because it's driven by mandates, Measurement at L1, so I'd fix measurement first."

### 2.6 Golden Signals, RED, USE

| Method | Signals | Apply to | Origin |
|---|---|---|---|
| **Four golden signals** | Latency, Traffic, Errors, Saturation | User-facing services | Google SRE book, "Monitoring Distributed Systems" |
| **RED** | Rate, Errors, Duration | Request-driven services, per endpoint | Tom Wilkie |
| **USE** | Utilization, Saturation, Errors | Every resource: CPU, memory, disk, NIC, connection pools, queues | Brendan Gregg |

Nuances: track error latency separately, because fast failures make p99 look good. Saturation is the leading indicator. Use RED for services and USE for the resources they run on.

### 2.7 OpenGitOps Principles (v1.0.0)

1. **Declarative:** a system managed by GitOps must have its desired state expressed declaratively.
2. **Versioned and Immutable:** desired state is stored in a way that enforces immutability and versioning, and retains a complete version history.
3. **Pulled Automatically:** software agents automatically pull the desired state declarations from the source.
4. **Continuously Reconciled:** software agents continuously observe actual system state and attempt to apply the desired state.

### 2.8 SLSA v1.2 (current spec)

| Build track | Name | What it guarantees |
|---|---|---|
| **Build L0** | No guarantees | Nothing. L0 means SLSA isn't in place |
| **Build L1** | Provenance exists | Provenance shows how the package was built. Catches mistakes, not tampering |
| **Build L2** | Hosted build platform | A hosted platform generates and signs the provenance, so forging it takes an explicit attack |
| **Build L3** | Hardened builds | Builds are isolated and signing material is out of reach of build steps, so forging provenance takes exploiting a vulnerability |

| Source track | Name |
|---|---|
| **Source L1** | Version controlled |
| **Source L2** | History and provenance: history is continuous and retained, and the source control system issues provenance for each revision |
| **Source L3** | Continuous technical controls, enforced on named references such as protected branches |
| **Source L4** | Two-party review: two trusted people review every change to a protected branch |

> [!WARNING]
> SLSA v0.1 used Levels 1–4 for everything. If someone says "SLSA 4 requires two-person review", they mean the retired v0.1 model. In v1.x the Build track stops at L3, and two-party review is Source L4.

### 2.9 FinOps Lifecycle Phases (FinOps Foundation)

| Phase | Goal | Platform examples |
|---|---|---|
| **Inform** | Visibility and allocation: who spends what, and on what | Mandatory tags and labels baked into templates, cost per namespace and team, showback dashboards, unit costs |
| **Optimize** | Find and rank efficiency opportunities | Rightsizing requests, autoscaling and bin packing, Spot for stateless work, storage tiers, commitments (Savings Plans) once usage is stable |
| **Operate** | Make it continuous through governance, policy, and culture | Budgets and anomaly alerts, policy as code for tagging and instance types, cost as a standing item in architecture reviews |

The phases repeat in a cycle, and practices mature from crawl to walk to run one capability at a time.

Deeper: [Domain 6 Study Guide](Domain_6_Observability_SRE/Study_Guide.md) · [Domain 7 Study Guide](Domain_7_Security_Governance_FinOps/Study_Guide.md)

---

## 3. Numbers to Know

### 3.1 Latency Numbers (the classic Dean/Norvig list, circa 2010: learn the orders of magnitude)

| Operation | Approx. time | So what |
|---|---|---|
| L1 cache reference | 0.5 ns | |
| Branch mispredict | 5 ns | |
| L2 cache reference | 7 ns | |
| Mutex lock/unlock | 25 ns | |
| Main memory reference | 100 ns | RAM is about 5,000x faster than a datacenter round trip |
| Send 1 KB over a 1 Gbps network | 10 µs | |
| Read 4 KB randomly from SSD | 150 µs | Modern local NVMe is faster (tens of µs) |
| Read 1 MB sequentially from memory | 250 µs | |
| Round trip within one datacenter | 500 µs | Every synchronous hop costs roughly this |
| Read 1 MB sequentially from SSD | 1 ms | |
| HDD seek | 10 ms | |
| Read 1 MB sequentially from HDD | 20 ms | |
| Packet from California to the Netherlands and back | 150 ms | No cache gets you past physics. Only moving closer does |

**Cloud rules of thumb** (measure your own with `ping`, `mtr`, or `curl -w`):
- Light in fiber travels about 200,000 km/s, so each 100 km of fiber path adds about 1 ms of round trip. Real routes are longer than the straight line.
- Same AZ: well under 1 ms RTT. Across AZs in a region: around 1 ms or a bit more (AWS keeps a Region's AZs within about 100 km of each other). Across regions on one continent: tens of ms. Between continents: roughly 70–250 ms.
- EBS: sub-millisecond per I/O (io2 Block Express) up to single-digit milliseconds (gp3). Local instance-store NVMe is faster.
- Human perception (Nielsen's limits): 0.1 s feels instant, 1 s keeps the user's flow of thought, 10 s loses their attention.

### 3.2 Availability Nines (computed with year = 365 days, month = 30 days)

| SLO | Downtime / year | / 30 days | / week | / day |
|---|---|---|---|---|
| 99% | 3.65 days (87.6 h) | 7 h 12 min | 1 h 40 min 48 s | 14 min 24 s |
| 99.5% | 1.83 days (43.8 h) | 3 h 36 min | 50 min 24 s | 7 min 12 s |
| 99.9% | 8 h 45 min 36 s | 43 min 12 s | 10 min 4.8 s | 1 min 26.4 s |
| 99.95% | 4 h 22 min 48 s | 21 min 36 s | 5 min 2.4 s | 43.2 s |
| 99.99% | 52 min 33.6 s | 4 min 19.2 s | 1 min 0.5 s | 8.64 s |
| 99.999% | 5 min 15.4 s | 25.9 s | 6.05 s | 0.86 s |

- With a 30.44-day average month the numbers come out slightly larger (99.9% gives 43.8 min). Say which window you mean. A rolling 30 days is the usual SLO window.
- **Serial dependencies multiply:** two hard dependencies at 99.9% cap you at 99.9% × 99.9% ≈ 99.8%.
- **Redundancy adds nines only when failures are independent:** two replicas at 99% each give 1 − 0.01² = 99.99%. A shared region, config, or deploy pipeline breaks that assumption.
- Each extra nine cuts allowed downtime by 10x and usually raises cost and slows change. Choose the SLO from user needs.

### 3.3 Multi-Window, Multi-Burn-Rate Alerts (Google SRE Workbook, Table 5-8: 99.9% SLO, 30-day window)

| Severity | Long window | Short window | Burn rate | Budget spent in the long window | Error-ratio threshold at 99.9% | Whole budget gone in |
|---|---|---|---|---|---|---|
| Page | 1 h | 5 min | 14.4 | 2% | 1.44% | 50 h |
| Page | 6 h | 30 min | 6 | 5% | 0.6% | 5 days |
| Ticket | 3 days | 6 h | 1 | 10% | 0.1% | 30 days |

- **Burn rate** = observed error ratio ÷ (1 − SLO). A burn rate of 1 spends exactly the whole budget over the SLO window.
- **Budget spent** = burn rate × long window ÷ SLO window. For example, 14.4 × 1 h ÷ 720 h = 2%.
- **The short window is 1/12 of the long window.** Both windows must be over the threshold: the long one proves the problem is significant, and the short one proves it's still happening, so the alert resets soon after recovery.
- Many SLO tools add a fourth row: a ticket at burn rate 3 over 1 day with a 2 h short window (also 10% of the budget). The workbook's own example expression uses it.

> [!TIP]
> **Interview Angle:** If asked "why not just alert when errors exceed 1%?", say that a fixed threshold is either too noisy for short blips or too slow for a slow burn. Burn rates tie the alert to how much of the error budget is actually at risk, and the two-window check keeps both precision and fast reset.

### 3.4 Kubernetes Numbers

| Item | Value |
|---|---|
| Scalability envelope (v1.37 docs) | No more than 5,000 nodes, 110 pods per node, 150,000 total pods, and 300,000 total containers, all at once. These are ceilings, not targets |
| kubelet `maxPods` default | 110. On EKS with the VPC CNI and no prefix delegation: ENIs × (IPv4 addresses per ENI − 1) + 2 |
| Pod DNS `ndots` | 5, with search path `<ns>.svc.cluster.local svc.cluster.local cluster.local` plus the node's search domains |
| NodePort range | 30000–32767 |
| Service CIDR (kubeadm default) / pod CIDR per node (IPv4 default) | 10.96.0.0/12 / one /24 per node |
| `terminationGracePeriodSeconds` | 30 s from SIGTERM to SIGKILL |
| Probe defaults | `periodSeconds: 10`, `timeoutSeconds: 1`, `failureThreshold: 3`, `successThreshold: 1` |
| Eviction from a NotReady or unreachable node | Pods tolerate `not-ready` and `unreachable` taints for 300 s by default (DefaultTolerationSeconds) |
| Kubelet hard eviction defaults (Linux) | `memory.available<100Mi`, `nodefs.available<10%`, `imagefs.available<15%`, `nodefs.inodesFree<5%` |
| HPA | 15 s sync period, 10% tolerance, 300 s scale-down stabilization window |
| ConfigMap / Secret size | 1 MiB each |
| etcd | 1.5 MiB max request size, 2 GiB default backend quota, 8 GiB suggested maximum |
| Version skew | kubelet can be up to 3 minor versions older than kube-apiserver, never newer. kubectl must be within one minor version either way |
| CrashLoopBackOff | Exponential restart back-off, classically starting at 10 s and doubling to a 5 min cap. Recent releases have been tuning this, so check your version |
| Argo CD reconciliation | Every 120 s plus up to 60 s jitter (`timeout.reconciliation` in `argocd-cm`) |
| Helm 4 upgrade defaults | `--timeout 5m0s`, `--history-max 10` |

### 3.5 Ports Worth Knowing

| Port | Service | Port | Service |
|---|---|---|---|
| 22 | SSH | 6443 | kube-apiserver |
| 53 (UDP and TCP) | DNS | 2379 / 2380 | etcd client / peer |
| 80 / 443 | HTTP / HTTPS (HTTP/3 runs on UDP 443) | 10250 | kubelet API |
| 123 (UDP) | NTP | 10256 | kube-proxy health |
| 179 | BGP (Calico, MetalLB BGP mode) | 10257 / 10259 | kube-controller-manager / kube-scheduler (HTTPS) |
| 389 / 636 | LDAP / LDAPS | 30000–32767 | NodePort range |
| 2049 | NFS (Amazon EFS) | 9153 | CoreDNS metrics |
| 3306 | MySQL / Aurora MySQL | 9090 / 9093 | Prometheus / Alertmanager |
| 5432 | PostgreSQL | 9100 | node_exporter |
| 6379 | Redis / Valkey | 3000 | Grafana |
| 9092 | Kafka | 3100 | Loki |
| 8200 | Vault API | 4317 / 4318 | OTLP over gRPC / OTLP over HTTP |
| 4789 (UDP) | VXLAN (IANA) | 8472 (UDP) | VXLAN, Linux default (Flannel, Cilium) |
| 51820 (UDP) | WireGuard | 16686 | Jaeger UI |

### 3.6 Connection Setup Cost: TLS 1.2 vs 1.3

| New connection | Round trips before the request can be sent | Round trips to the first response byte |
|---|---|---|
| TCP + TLS 1.2, full handshake | 1 (TCP) + 2 (TLS) = 3 | 4 |
| TCP + TLS 1.2, session resumption | 1 + 1 = 2 | 3 |
| TCP + TLS 1.3, full handshake | 1 + 1 = 2 | 3 |
| TCP + TLS 1.3, 0-RTT resumption | 1 + 0 = 1 | 2 |
| QUIC (HTTP/3), new connection | 1 (transport and TLS 1.3 combined) | 2 |
| QUIC, 0-RTT resumption | 0 | 1 |
| Reused keep-alive or HTTP/2 connection | 0 | 1 |

At 80 ms RTT, a fresh TLS 1.2 connection costs 320 ms before any server time, TLS 1.3 costs 240 ms, and a pooled connection costs 80 ms. 0-RTT early data can be replayed, so servers should accept it only for idempotent requests. Connection reuse beats any handshake tuning.

### 3.7 Exit Codes

| Code | Meaning | What it usually means in a container |
|---|---|---|
| 0 | Success | The process finished. Under `restartPolicy: Always`, exiting 0 still loops into CrashLoopBackOff (wrong workload type, or the process daemonized) |
| 1 | General error | Application failure. Read `kubectl logs --previous` |
| 126 | Found but not executable | Missing `+x`, a `noexec` mount, or a wrong-architecture binary run from a shell |
| 127 | Command not found | A shell or entrypoint script calls a binary that isn't in the image or on `PATH`. If the runtime can't exec the entrypoint at all, you usually see `StartError` or `RunContainerError` instead |
| 128 + n | Killed by signal n | |
| 130 | 128 + 2, SIGINT | Ctrl-C |
| 137 | 128 + 9, SIGKILL | `OOMKilled` if the `reason` field says so. Otherwise it was killed after the grace period, or by `kill -9` |
| 139 | 128 + 11, SIGSEGV | Segfault: a native code bug, or an ABI or architecture mismatch |
| 143 | 128 + 15, SIGTERM | Asked to stop by a rollout, scale-in, eviction, or failed liveness probe. Normal if the app shut down cleanly |

### 3.8 Ephemeral Ports and Connection Limits

- The Linux default `net.ipv4.ip_local_port_range` is **32768–60999**, which gives **28,232** source ports per (source IP, destination IP, destination port, protocol).
- `TIME_WAIT` lasts 60 s on Linux. A client that actively closes every connection to one destination therefore tops out around 28,232 ÷ 60 ≈ 470 new connections per second per source IP.
- Pods behind one SNAT address (the node IP or a NAT gateway) share that budget. An AWS NAT gateway supports up to 55,000 simultaneous connections to each unique destination.
- Fixes, in order: connection pooling and keep-alive, more source IPs, a wider port range, then `net.ipv4.tcp_tw_reuse` for outbound connections.

Deeper: [Domain 2 Study Guide](Domain_2_Linux_Networking_Fundamentals/Study_Guide.md) · [Domain 3 Study Guide](Domain_3_Kubernetes_Platform_Engineering/Study_Guide.md) · [Domain 6 Study Guide](Domain_6_Observability_SRE/Study_Guide.md)

---

## 4. Command Quick Reference

### 4.1 Linux: The 60-Second Triage (Brendan Gregg, Netflix)

```bash
uptime                 # load averages 1/5/15 min: rising or falling? (Linux load includes D-state I/O waits)
dmesg -T | tail        # OOM kills, hardware errors, TCP drops, "nf_conntrack: table full"
vmstat 1               # r > CPU count = CPU saturation; si/so = swapping; wa = I/O wait; st = steal
mpstat -P ALL 1        # per-CPU balance: one hot CPU means a single-threaded bottleneck
pidstat 1              # per-process CPU over time (rolling, unlike top)
iostat -xz 1           # r/s, w/s, await (latency), aqu-sz (queue depth), %util per device
free -m                # read "available", not "free"; buff/cache is reclaimable
sar -n DEV 1           # rxkB/s and txkB/s per interface vs link capacity
sar -n TCP,ETCP 1      # active/s (outbound), passive/s (inbound), retrans/s
top                    # overview, then drill into the suspicious process
```

`mpstat`, `pidstat`, `iostat`, and `sar` come from `sysstat` (`sudo pacman -S sysstat`). Next steps:

```bash
systemctl --failed
journalctl -u kubelet --since "15 min ago" --no-pager | tail -50
df -h; df -i                                        # running out of inodes also looks like "disk full"
ps -eo pid,ppid,stat,%cpu,%mem,etime,cmd --sort=-%cpu | head
cat /proc/<pid>/limits; ls /proc/<pid>/fd | wc -l   # open-file limit vs actual usage
sudo strace -f -tt -p <pid> -e trace=network        # which syscalls it is stuck in (adds overhead)
sudo perf top                                       # where CPU time goes, live
```

### 4.2 Networking

```bash
# Sockets
ss -tlnp                                          # listening TCP sockets and the owning process
ss -tnp state established '( dport = :5432 )'     # who is connected to Postgres
ss -s                                             # totals, including TIME-WAIT
ss -ti dst 10.0.3.7                               # per-connection rtt, cwnd, retransmits

# Interfaces and routing
ip -br addr                                       # interfaces and IPs, one line each
ip route get 10.20.0.15                           # the route, interface, and source IP actually used
ip -s link show eth0                              # RX/TX errors and drops
ip neigh                                          # ARP/NDP cache
ip rule; ip route show table all                  # policy routing (CNI, VPN)

# DNS
dig +short api.example.com
dig @10.96.0.10 api.payments.svc.cluster.local +short   # ask cluster DNS directly
dig +trace example.com                            # follow delegation down from the root
dig api.example.com +search                       # apply the resolv.conf search list (dig skips it by default)
getent hosts api                                  # resolve the way applications do (nsswitch, /etc/hosts)

# HTTP timing: where did the time go?
curl -sS -o /dev/null -w 'dns=%{time_namelookup} connect=%{time_connect} tls=%{time_appconnect} ttfb=%{time_starttransfer} total=%{time_total} code=%{http_code} ip=%{remote_ip}\n' https://api.example.com/healthz
curl -sv --resolve api.example.com:443:203.0.113.10 https://api.example.com/   # pin an IP, keep SNI and Host
curl -sS --connect-timeout 3 --max-time 10 http://api.payments.svc:8080/healthz

# Packets
sudo tcpdump -i any -nn port 53                                    # DNS queries and answers
sudo tcpdump -i any -nn 'host 10.0.3.7 and tcp port 443'
sudo tcpdump -i any -nn 'tcp[tcpflags] & (tcp-syn|tcp-rst) != 0'   # SYNs and resets only
sudo tcpdump -i any -nn -s 0 -c 2000 -w /tmp/cap.pcap port 8080    # save for Wireshark

# TLS
openssl s_client -connect api.example.com:443 -servername api.example.com -showcerts </dev/null
openssl s_client -connect api.example.com:443 -servername api.example.com </dev/null 2>/dev/null \
  | openssl x509 -noout -subject -issuer -dates -ext subjectAltName
openssl s_client -connect api.example.com:443 -tls1_3 -alpn h2 </dev/null   # protocol and ALPN check
```

The `curl -w` timers are cumulative from the start. TLS time is `time_appconnect − time_connect`, and server time is roughly `time_starttransfer − time_appconnect`.

### 4.3 Kubernetes Debugging (kubectl 1.37)

```bash
# Orientation
kubectl config current-context && kubectl auth whoami
kubectl get nodes -o wide -L topology.kubernetes.io/zone,node.kubernetes.io/instance-type
kubectl get pods -A --field-selector=status.phase!=Running,status.phase!=Succeeded

# The big four: describe, logs, previous logs, events
kubectl describe pod api-7d9c -n payments                    # Events, Last State, Exit Code, Reason
kubectl logs api-7d9c -n payments -c api --previous          # logs from the crashed instance
kubectl logs deploy/api -n payments --all-containers --since=15m --timestamps
kubectl events -n payments --for=pod/api-7d9c --types=Warning
kubectl get events -A --sort-by=.metadata.creationTimestamp | tail -30

# Capacity (needs metrics-server)
kubectl top nodes
kubectl top pods -n payments --containers --sort-by=memory
kubectl describe node ip-10-0-1-23 | grep -A 12 'Allocated resources'

# Rollouts
kubectl rollout status deploy/api -n payments --timeout=120s
kubectl rollout history deploy/api -n payments
kubectl rollout undo deploy/api -n payments

# Service wiring: selector -> EndpointSlices -> ready pods
kubectl get svc api -n payments -o wide
kubectl get endpointslices -n payments -l kubernetes.io/service-name=api
kubectl port-forward -n payments svc/api 8080:80

# Debug containers (--profile: general (default), baseline, restricted, netadmin, sysadmin)
kubectl debug -it pod/api-7d9c -n payments --image=busybox:1.36 --target=api
kubectl debug -it pod/api-7d9c -n payments --image=nicolaka/netshoot --profile=netadmin -- tcpdump -i any -nn port 5432
kubectl debug -it pod/api-7d9c -n payments --copy-to=api-debug --container=api -- sh   # copy of the pod, shell instead of the crashing entrypoint
kubectl debug node/ip-10-0-1-23 -it --image=busybox:1.36   # host filesystem mounted at /host; then: chroot /host

# RBAC
kubectl auth can-i create deployments -n payments --as=system:serviceaccount:payments:ci
kubectl auth can-i --list -n payments --as=jane@example.com

# jsonpath, custom-columns, jq
kubectl get pods -n payments -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.status.containerStatuses[*].lastState.terminated.reason}{"\n"}{end}'
kubectl get pods -A -o custom-columns='NS:.metadata.namespace,POD:.metadata.name,NODE:.spec.nodeName,CPU_REQ:.spec.containers[*].resources.requests.cpu,MEM_LIM:.spec.containers[*].resources.limits.memory'
kubectl get pods -A -o json | jq -r '.items[] | select(any(.status.containerStatuses[]?; .restartCount > 5)) | "\(.metadata.namespace)/\(.metadata.name)"'

# Control plane health and deprecated API usage (check before upgrades)
kubectl get --raw='/readyz?verbose'
kubectl get --raw=/metrics | grep apiserver_requested_deprecated_apis
```

For core cluster operations, review Domain 2 and Domain 3 study guides.

### 4.4 Terraform 1.15 / OpenTofu 1.12

```bash
terraform init -upgrade                           # newest providers the constraints allow; updates the lock file
terraform fmt -recursive -check && terraform validate
terraform plan -out=tfplan                        # save exactly what was reviewed
terraform show -json tfplan | jq -r '.resource_changes[] | select(.change.actions != ["no-op"]) | "\(.change.actions | join(",")) \(.address)"'
terraform apply tfplan                            # applies that plan and nothing else
terraform plan -detailed-exitcode                 # exit 0 = no changes, 1 = error, 2 = changes (drift jobs)
terraform plan -refresh-only                      # drift only; `terraform apply -refresh-only` accepts it into state
terraform plan -replace='aws_instance.bastion'    # force one replacement (successor to `taint`)
terraform plan -lock-timeout=5m
terraform plan -generate-config-out=generated.tf  # draft HCL for resources targeted by import blocks

terraform state list | grep aws_s3
terraform state show 'module.vpc.aws_subnet.private["us-east-1a"]'
terraform state mv 'aws_instance.web' 'module.web.aws_instance.this'   # prefer a moved block
terraform state rm 'aws_s3_bucket.legacy'                             # prefer a removed block
terraform state pull > "backup-$(date +%F).tfstate"
terraform force-unlock <LOCK_ID>                  # only after proving no run holds the lock

terraform test                                    # runs *.tftest.hcl from the root and tests/ directories
terraform providers lock -platform=linux_amd64 -platform=darwin_arm64
```

```hcl
import {                     # adopt existing infrastructure (Terraform 1.5+; for_each since 1.7)
  to = aws_s3_bucket.logs
  id = "acme-prod-logs"
}

moved {                      # refactor addresses without destroy/create (1.1+)
  from = aws_instance.web
  to   = module.web.aws_instance.this
}

removed {                    # stop managing without destroying (1.7+)
  from = aws_s3_bucket.legacy
  lifecycle {
    destroy = false
  }
}
```

```hcl
# tests/bucket.tftest.hcl
run "bucket_name_has_org_prefix" {
  command = plan
  assert {
    condition     = startswith(aws_s3_bucket.logs.bucket, "acme-")
    error_message = "Bucket names must start with acme-."
  }
}
```

OpenTofu (`tofu`) accepts the same subcommands and blocks, and adds client-side state encryption (1.7+), early evaluation of variables in backend and module-source configuration (1.8+), and `-exclude` plus provider `for_each` (1.9+).

### 4.5 Helm 4 and Kustomize

> [!CAUTION]
> Helm 4 renamed two flags people type from Helm 3 muscle memory. `--atomic` is now `--rollback-on-failure`, and `--force` is now `--force-replace` (with `--force-conflicts` for server-side apply conflicts). `--wait` takes `watcher` (which a bare `--wait` means), `hookOnly` (the default when the flag is omitted), or `legacy`. `--dry-run` takes `client`, `server`, or `none`.

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts && helm repo update
helm search repo prometheus-community/kube-prometheus-stack --versions | head
helm show values prometheus-community/kube-prometheus-stack --version <pinned> > values.yaml
helm lint ./charts/api --strict
helm template api ./charts/api -n payments -f values-prod.yaml | kubectl diff -n payments -f -   # preview the change
helm upgrade --install api ./charts/api -n payments --create-namespace -f values-prod.yaml --dry-run=server
helm upgrade --install api ./charts/api -n payments -f values-prod.yaml --rollback-on-failure --timeout 10m
helm upgrade --install api oci://registry.example.com/charts/api --version 1.4.2 -n payments
helm list -A --failed; helm list -A --pending            # failed or stuck releases
helm status api -n payments; helm history api -n payments
helm get values api -n payments --all                    # computed values actually applied
helm get manifest api -n payments
helm rollback api 7 -n payments
```

```bash
kustomize build overlays/prod | less                     # or: kubectl kustomize overlays/prod
kubectl diff -k overlays/prod
kubectl apply -k overlays/prod --server-side
(cd overlays/prod && kustomize edit set image api=registry.example.com/payments/api:1.4.2)
kustomize build --enable-helm overlays/prod              # inflate helmCharts entries
```

```yaml
# overlays/prod/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
namespace: payments
resources:
  - ../../base
images:
  - name: api
    newName: registry.example.com/payments/api
    newTag: "1.4.2"
patches:
  - path: replicas.yaml
    target:
      kind: Deployment
      name: api
```

### 4.6 Argo CD 3.x CLI

```bash
argocd login argocd.example.com --sso --grpc-web
argocd app list -p payments -l team=payments
argocd app get payments-api --refresh             # --hard-refresh also clears the manifest cache
argocd app diff payments-api                      # exit 0 = no diff, 1 = diff, 2 = error
argocd app diff payments-api --local ./overlays/prod --server-side-generate
argocd app sync payments-api --prune --timeout 300
argocd app sync payments-api --resource apps:Deployment:api   # sync a single resource
argocd app wait payments-api --sync --health --timeout 600
argocd app history payments-api
argocd app set payments-api --sync-policy manual  # rollback is refused while automated sync is on
argocd app rollback payments-api 42               # then fix Git before turning auto-sync back on
argocd app terminate-op payments-api              # stop a stuck sync operation
argocd app manifests payments-api --source live
argocd app resources payments-api --orphaned
```

### 4.7 PromQL Snippets

```promql
# Error ratio per service (5xx / all)
sum by (service) (rate(http_requests_total{code=~"5.."}[5m]))
  / sum by (service) (rate(http_requests_total[5m]))

# p99 latency from classic histograms (keep le in the by clause)
histogram_quantile(0.99, sum by (le, service) (rate(http_request_duration_seconds_bucket[5m])))

# p99 latency from native histograms (if enabled)
histogram_quantile(0.99, sum by (service) (rate(http_request_duration_seconds[5m])))

# Saturation: share of CFS periods throttled, per container
sum by (namespace, pod, container) (rate(container_cpu_cfs_throttled_periods_total[5m]))
  / sum by (namespace, pod, container) (rate(container_cpu_cfs_periods_total[5m]))

# Saturation: memory working set as a fraction of the limit
sum by (namespace, pod, container) (container_memory_working_set_bytes{container!=""})
  / sum by (namespace, pod, container) (kube_pod_container_resource_limits{resource="memory"})

# Node CPU utilization, and a filesystem that will be full within 4 hours at the current trend
1 - avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[5m]))
predict_linear(node_filesystem_avail_bytes{fstype!~"tmpfs|overlay"}[6h], 4 * 3600) < 0

# Burn-rate page for a 99.9% SLO: both windows above 14.4 (use recording rules in production)
(
  sum(rate(http_requests_total{code=~"5.."}[1h])) / sum(rate(http_requests_total[1h]))
) / 0.001 > 14.4
and
(
  sum(rate(http_requests_total{code=~"5.."}[5m])) / sum(rate(http_requests_total[5m]))
) / 0.001 > 14.4

# Restart storms and missing scrape targets
increase(kube_pod_container_status_restarts_total[1h]) > 3
up == 0
absent(up{job="api"})
```

### 4.8 LogQL (Loki)

- Stream selector first, then cheap line filters: `{namespace="payments", app="api"} |= "error" != "healthz"`
- Regex line filter: `{app="api"} |~ "timeout|deadline exceeded"`
- Parse JSON, filter on a field, reformat: `{app="api"} | json | status >= 500 | line_format "{{.method}} {{.path}} {{.status}}"`
- Filter on a parsed duration: `{app="api"} | logfmt | duration > 500ms`
- Drop lines that failed to parse: `{app="api"} | json | __error__ = ""`
- Log lines per status: `sum by (status) (count_over_time({app="api"} | json [5m]))`
- Error-line ratio: `sum(rate({app="api"} |= "error" [5m])) / sum(rate({app="api"} [5m]))`
- p99 from a logged field: `quantile_over_time(0.99, {app="api"} | json | unwrap latency_ms [5m]) by (route)`
- Noisiest pods: `topk(5, sum by (pod) (rate({namespace="payments"} |= "panic" [10m])))`

Loki indexes only labels, so keep stream labels low-cardinality (namespace, app, cluster) and never use user or request IDs as labels. Put the most selective line filter first, and parse (`json`, `logfmt`) only after filtering.

Deeper: [Domain 4 Study Guide](Domain_4_Infrastructure_as_Code/Study_Guide.md) · [Domain 5 Study Guide](Domain_5_CICD_GitOps_Delivery/Study_Guide.md) · [Domain 6 Study Guide](Domain_6_Observability_SRE/Study_Guide.md)

---

## 5. System Design Template

### 5.1 Step Framework and Time Boxes

| Step | 45-min round | 60-min round | What ends up on the board |
|---|---|---|---|
| 0. Restate the prompt | 0–2 | 0–3 | One-sentence problem statement |
| 1. Requirements: users and top journeys, functional scope, NFRs (scale, SLOs, tenancy, compliance), what's out of scope | 2–8 | 3–10 | Bullets plus 2–3 stated assumptions |
| 2. Estimates that change the design | 8–11 | 10–14 | 3–5 numbers: RPS, data size, tenants, clusters |
| 3. Interfaces: platform API or CRD, developer journey, data model | 11–15 | 14–20 | API sketch or the golden path step by step |
| 4. High-level architecture: control plane vs data plane, main flows | 15–25 | 20–32 | Box-and-arrow diagram |
| 5. Deep dive on 1–2 components (let the interviewer choose) | 25–35 | 32–45 | Component detail plus a trade-off table |
| 6. Failure modes, blast radius, security, cost, operations | 35–40 | 45–52 | Failure table with mitigations |
| 7. Rollout and migration plan, success metrics, summary | 40–43 | 52–56 | Phases plus 3 metrics |
| 8. Buffer and your questions | 43–45 | 56–60 | |

If the interviewer steers, follow them: the template is a fallback, not a script. State the plan up front ("about five minutes on requirements, then a high-level design, then wherever you'd like to go deep") and checkpoint every 10 minutes.

### 5.2 Design Checklist

| Area | Ask yourself out loud | A platform-grade answer includes |
|---|---|---|
| **Tenancy** | Who shares what? Soft or hard isolation? What about noisy neighbors? | Namespace vs cluster vs account per tenant tier; quotas, NetworkPolicy, per-tenant rate limits; cost attribution |
| **Blast radius** | What is the worst single change or failure, and how far does it spread? | Cells or shards; rollout order (canary cell, then one region, then all); scoped credentials; separate control and data planes |
| **Failure modes** | What happens when each dependency is slow, down, or wrong? | Timeouts, retries with budgets, circuit breakers, static stability, graceful degradation, DR tier (RTO/RPO) |
| **Migration** | How do we get from today to there without a big bang? | Strangler pattern or parallel run, backfill, feature flags, a lighthouse team, rollback plan, deprecation dates |
| **Metrics** | How do we know it works and gets adopted? | SLOs for the platform itself, DORA and DevEx for outcomes, adoption share, time to first deploy |
| **Cost** | What drives cost, and who pays? | Unit cost (per tenant, request, or build minute), autoscaling, Spot, retention tiers, showback |
| **Security** | Identity, secrets, supply chain, policy, audit? | Workload identity, least privilege, signed artifacts verified at admission, policy as code, audit logs |
| **DX** | What does a developer actually do, step by step? | Golden path, one command or one PR to self-serve, clear errors, docs, escape hatches, a feedback channel |

### 5.3 Building Blocks (AWS first, with equivalents)

| Capability | AWS | GCP | Azure |
|---|---|---|---|
| Tenant / blast-radius boundary | Accounts in AWS Organizations + SCPs | Projects and folders + Organization Policy | Subscriptions and management groups + Azure Policy |
| Managed Kubernetes | EKS | GKE | AKS |
| Workload identity | EKS Pod Identity / IRSA | Workload Identity Federation for GKE | Microsoft Entra Workload ID |
| Secrets | Secrets Manager | Secret Manager | Key Vault |
| Image registry | ECR | Artifact Registry | Azure Container Registry |
| L7 load balancing | ALB | Cloud Load Balancing (Application Load Balancer) | Application Gateway / Front Door |
| DNS | Route 53 | Cloud DNS | Azure DNS |
| Managed Prometheus | Amazon Managed Service for Prometheus | Google Cloud Managed Service for Prometheus | Azure Monitor managed service for Prometheus |
| Cost data | Cost Explorer, Data Exports (CUR 2.0) | Cloud Billing export to BigQuery | Cost Management exports |

Cloud-neutral platform pieces: Backstage (portal), Argo CD or Flux (GitOps), Crossplane or Terraform automation (infrastructure APIs), Kyverno, Gatekeeper, or ValidatingAdmissionPolicy (policy), External Secrets Operator, cert-manager, external-dns, Karpenter or Cluster Autoscaler, OpenTelemetry Collector.

### 5.4 Estimation Shortcuts

| Shortcut | Value |
|---|---|
| Seconds per day / 30 days / year | 86,400 (≈ 10^5) / 2.59 M / 31.5 M |
| Requests per day to RPS | 1 M/day ≈ 12 RPS average; 100 M/day ≈ 1,160 RPS; 1 B/day ≈ 11,600 RPS. State your peak-to-average assumption explicitly |
| Little's Law | In-flight = arrival rate × latency. 1,000 RPS × 200 ms = 200 concurrent requests, which sizes pools, threads, and pods |
| Replicas | Peak RPS ÷ per-pod RPS at about 60–70% utilization, then add enough that losing one AZ still covers peak |
| Bandwidth | 1 Gbps ≈ 125 MB/s; 10 Gbps ≈ 1.25 GB/s |
| Logs | 1,000 RPS × 1 KB per line ≈ 1 MB/s ≈ 86 GB/day ≈ 2.6 TB per 30 days, before compression and replication |
| Metric cardinality | Series = product of label values. 100 pods × 20 routes × 5 status codes × 12 histogram series (10 buckets + sum + count) = 120,000 series for one metric |
| Availability | Serial: multiply. Independent parallel: 1 − (1 − a)ⁿ |
| Subnets | /16 = 65,536; /20 = 4,096; /24 = 256 addresses. AWS reserves 5 per subnet |
| Powers of two | 2^10 ≈ 10^3, 2^20 ≈ 10^6, 2^30 ≈ 10^9, 2^40 ≈ 10^12 |
| Error budget in requests | 99.9% over 30 days at 1,000 RPS ≈ 2.6 M failed requests allowed |

Deeper: [Domain 8 Study Guide](Domain_8_System_Design/Study_Guide.md) · [Domain 8 Interview Questions](Domain_8_System_Design/Interview_Questions.md)

---

## 6. Troubleshooting Trees

Every tree runs the same way: state the symptom and its scope, run the first three commands, branch on what you see, mitigate before you root-cause, and finish with the guardrail that stops a repeat. Interviewers score the last column as much as the fix. General-purpose commands are in §4; these are the tree-specific ones.

### 6.1 Pod Stuck Pending

**Symptom:** `Pending` with no node assigned. (`ContainerCreating` means it *was* scheduled; look at volumes, image pulls, or the CNI instead.)

```bash
kubectl get pod <pod> -n <ns> -o jsonpath='{.status.conditions[?(@.type=="PodScheduled")].message}{"\n"}'
kubectl get events -n <ns> --field-selector involvedObject.name=<pod>,reason=FailedScheduling
kubectl get nodes -o custom-columns='NODE:.metadata.name,CPU:.status.allocatable.cpu,MEM:.status.allocatable.memory,TAINTS:.spec.taints[*].key'
```

| Scheduler says | Likely cause | Fix | Guardrail |
|---|---|---|---|
| `Insufficient cpu` / `Insufficient memory` | Requests exceed free allocatable, and the autoscaler can't add nodes (NodePool `limits`, no capacity for the instance types, group max size) | Right-size requests; read the Karpenter logs (`kubectl get nodeclaims`) or the `cluster-autoscaler-status` ConfigMap | LimitRange defaults, VPA recommendations, alert on pods Pending > 5 min |
| `untolerated taint` | Dedicated pool (GPU, system, Spot) without a matching toleration | Add the toleration, or target the right pool | Workload classes in the golden path set selectors and tolerations |
| `didn't match Pod's node affinity/selector` | Label typo, or pinned to a zone with no capacity | `kubectl get nodes -L <key>`, then fix the selector | Allowlist node labels in policy |
| `unbound immediate PersistentVolumeClaims` / `volume node affinity conflict` | No default StorageClass, or a zonal volume (EBS) in a different AZ from the free capacity | `volumeBindingMode: WaitForFirstConsumer`; schedule into the volume's zone | Default StorageClass uses WaitForFirstConsumer |
| `Too many pods` | `maxPods` reached (VPC CNI IP limits, §3.4) | Prefix delegation, or larger instances | Track pod and IP headroom per node |
| `didn't satisfy existing pods anti-affinity rules` / `topology spread constraints` | Hard rules (required anti-affinity, `DoNotSchedule`) with too few nodes or zones | Soften to preferred / `ScheduleAnyway`, or add capacity | Templates default to soft spreading |
| STATUS `SchedulingGated` | A controller hasn't removed its `spec.schedulingGates` entry (for example a batch queue such as Kueue waiting for quota) | Check the controller that owns the gate | Alert on long-gated pods |

No pod at all? The controller never created it: `kubectl describe rs <rs> -n <ns>` shows `FailedCreate` from a ResourceQuota, a Pod Security Admission violation, or a webhook denial.

### 6.2 CrashLoopBackOff, OOMKilled and Exit Codes

**Symptom:** `RESTARTS` climbing. `CrashLoopBackOff` is the wait between restarts, not a cause. Run the big four from §4.3 first (`logs --previous` above all), and decode the exit code with §3.7.

```bash
kubectl get pod <pod> -n <ns> -o jsonpath='{range .status.containerStatuses[*]}{.name}: reason={.lastState.terminated.reason} exit={.lastState.terminated.exitCode} restarts={.restartCount}{"\n"}{end}'
kubectl get pod <pod> -n <ns> -o jsonpath='{range .spec.containers[*]}{.name}: {.resources}{"\n"}  liveness={.livenessProbe}{"\n"}{end}'
kubectl get events -n <ns> --field-selector involvedObject.name=<pod>,reason=Unhealthy   # probe failures
```

| You see | Likely cause | Fix | Guardrail |
|---|---|---|---|
| `reason=OOMKilled`, exit 137 | The working set hit the memory limit: limit too low, a leak, or a runtime heap not sized to the container | Raise the limit to the observed peak plus headroom; `-XX:MaxRAMPercentage` for the JVM, `GOMEMLIMIT` for Go; then hunt the leak | VPA recommendations; alert at 90% of the limit (§4.7); base images set runtime memory flags |
| `Unhealthy` events, then `failed liveness probe, will be restarted` (exit 137 or 143) | Liveness probe too strict: slow start, the 1 s default timeout, or it checks dependencies | Add a `startupProbe`; liveness checks only the process itself; raise `timeoutSeconds` | Probe templates in the golden path; lint that liveness never calls downstreams |
| Exit 1, stack trace or config error in `--previous` logs | Missing env var, Secret key, or flag; a dependency down at boot; a failing migration | Fix the config; retry dependencies with backoff instead of exiting | Validate config in CI; alert on External Secrets sync failures |
| `CreateContainerConfigError` | A referenced Secret or ConfigMap key doesn't exist | Create it, or fix the key name | Render and validate manifests in CI |
| `exec format error`, exit 126/127, `StartError` | Wrong architecture (arm64 image on amd64), a missing binary, or no shell in a distroless image | Build multi-arch images; fix the entrypoint | CI smoke-runs the image on every target architecture |
| Exit 0 and still restarting | The process daemonized or finished under `restartPolicy: Always` | Run in the foreground; use a Job for one-shot work | Workload-type guidance in templates |
| `ImagePullBackOff`, no restarts yet | Tag doesn't exist, registry auth (pull secret, node role for ECR), or a registry rate limit | `kubectl describe pod` quotes the registry's error verbatim | Pull-through cache; admission policy requiring digests from approved registries |

### 6.3 Service Unreachable

**Symptom:** calls to `<svc>.<ns>:<port>` time out, are refused, or don't resolve. Walk the chain in order: selector → EndpointSlices → kube-proxy/CNI → DNS. The basic wiring commands are in §4.3.

```bash
kubectl get pods -n <ns> -o wide -l "$(kubectl get svc <svc> -n <ns> -o jsonpath='{.spec.selector}' | jq -r 'to_entries | map("\(.key)=\(.value)") | join(",")')"
kubectl get endpointslices -n <ns> -l kubernetes.io/service-name=<svc> -o jsonpath='{range .items[*].endpoints[*]}{.addresses[0]} ready={.conditions.ready}{"\n"}{end}'
kubectl run nettest -n <ns> -it --rm --restart=Never --image=nicolaka/netshoot -- sh -c 'cat /etc/resolv.conf; dig +search <svc>; curl -sv --max-time 3 http://<svc>:<port>/'
```

| You see | Likely cause | Fix | Guardrail |
|---|---|---|---|
| The selector matches no pods | Label mismatch between Service and pod template (a typo, or a `version` label in the selector) | Align the labels; keep version labels out of selectors | Generate Service and workload labels from one source; CI checks every selector matches a pod template |
| Pods match, endpoints `ready=false` | Readiness probe failing | Fix readiness (§6.2) | Readiness checks the real serving path |
| Endpoints ready, `connection refused` through the Service | `targetPort` doesn't match the container port, or the app listens on 127.0.0.1 | Named ports (`targetPort: http`); bind 0.0.0.0 | Named ports in templates |
| Pod IP works, ClusterIP times out | Dataplane: kube-proxy not syncing (`iptables` is still the default mode; `nftables` is GA since 1.33), an unhealthy CNI or eBPF replacement, or a NetworkPolicy dropping traffic | `kubectl -n kube-system logs ds/kube-proxy`; CNI agent status; `kubectl get networkpolicy -n <ns>` | Default-deny plus explicit allows, tested in CI; synthetic cross-namespace probes |
| `NXDOMAIN`, or lookups take about 5 s | Short name used from another namespace; CoreDNS overloaded; `ndots:5` multiplying external lookups; conntrack race on parallel A/AAAA over UDP | Use `<svc>.<ns>.svc.cluster.local`, and a trailing dot for external names; scale CoreDNS | NodeLocal DNSCache by default; a CoreDNS latency SLO |
| Intermittent drops under load, `nf_conntrack: table full` in `dmesg` | Conntrack table exhausted by short-lived connections | Raise `nf_conntrack_max`; pool connections | Alert when `node_nf_conntrack_entries / node_nf_conntrack_entries_limit > 0.8` |
| Works in-cluster, fails from outside | Gateway or Ingress layer: route not attached, load balancer health checks failing | Read `status.parents[].conditions` (`Accepted`, `ResolvedRefs`) on the HTTPRoute | Gateway API by default; ingress-nginx was retired in March 2026, so treat any remaining install as a migration item |

### 6.4 Node NotReady and Pressure Evictions

**Symptom:** a node is `NotReady`, pods are `Evicted` or stuck `Terminating`, or a wave of rescheduling hits. Pods on an unreachable node are evicted after the 300 s default toleration (§3.4).

```bash
kubectl get node <node> -o jsonpath='{range .status.conditions[*]}{.type}={.status} ({.reason}) {.message}{"\n"}{end}'
kubectl get pods -A -o wide --field-selector spec.nodeName=<node>,status.phase=Failed   # evicted pods end up Failed
# on the node: kubectl debug node/<node> (§4.3), then chroot /host
journalctl -u kubelet -u containerd --since "30 min ago" --no-pager | grep -iE 'evict|pressure|PLEG|x509' | tail -40
```

| You see | Likely cause | Fix | Guardrail |
|---|---|---|---|
| `Ready=Unknown`, "Kubelet stopped posting node status" | Instance dead or partitioned, kubelet crashed, or its client certificate expired (`x509` in the log) | Check the instance in the cloud console; restart the kubelet; rotate the certificate | Node auto-repair (GKE auto-repair, EKS node auto repair); NotReady > 5 min alert; certificate expiry monitoring |
| `Ready=False`, "PLEG is not healthy" or runtime errors | containerd hung or overloaded, or the disk stalled | Restart containerd; cordon, drain, replace | Cap pods per node; alert on runtime operation latency |
| `MemoryPressure=True`, pods `Evicted` | Below the hard eviction threshold (`memory.available<100Mi`, §3.4). Pods using more than their requests go first, then by priority | Set requests to real usage; reserve memory for the system (`kubeReserved`, `systemReserved`) | LimitRange defaults; node templates with reservations; VPA recommendations |
| `DiskPressure=True` | Image bloat, unrotated container logs, or `emptyDir` growth | Image garbage collection; `ephemeral-storage` requests and limits; `containerLogMaxSize` | Ephemeral-storage limits in templates; an image-size budget in CI |
| `PIDPressure=True` | A fork bomb or thread leak in one pod | Kill or fix that pod | `podPidsLimit` in the kubelet config |
| `NetworkUnavailable=True` | CNI agent not ready, or the subnet is out of IPs | Fix the CNI DaemonSet; add subnet capacity | Subnet IP alerts; prefix delegation |
| StatefulSet pods stuck `Terminating` on a dead node | Nobody can confirm the pod stopped, so the replacement and the volume detach wait | Confirm the machine is really off, then taint the node `node.kubernetes.io/out-of-service=nodeshutdown:NoExecute` | A runbook, then automation, for non-graceful node shutdown |

### 6.5 Latency Spike, CPU Throttling, Noisy Neighbor

**Symptom:** p99 is up and errors are flat or creeping. Scope it first: every pod (a dependency, traffic, a deploy) or a few (node, neighbor, limits)? If it started with a rollout, roll back before you investigate.

```promql
# 1. Is the spike in every pod or only a few?
topk(5, histogram_quantile(0.99, sum by (le, pod) (rate(http_request_duration_seconds_bucket{service="api"}[5m]))))
```

```bash
# 2. Throttling and CPU pressure inside a slow pod (cgroup v2; PSI must be enabled in the kernel)
kubectl exec <pod> -n <ns> -c <container> -- cat /sys/fs/cgroup/cpu.stat /sys/fs/cgroup/cpu.pressure
# 3. Who shares its node?
kubectl get pods -A --field-selector spec.nodeName=<node> -o custom-columns='NS:.metadata.namespace,POD:.metadata.name,QOS:.status.qosClass,CPU_REQ:.spec.containers[*].resources.requests.cpu,CPU_LIM:.spec.containers[*].resources.limits.cpu'
```

| You see | Likely cause | Fix | Guardrail |
|---|---|---|---|
| `nr_throttled` climbing in step with p99 | CPU limit too low for bursty or multi-threaded work: the quota runs out early in each 100 ms period | Raise or drop the CPU limit and keep the request; match `GOMAXPROCS` to the limit (automatic from Go 1.25) and size JVM GC threads | Latency-tier policy: requests without tight CPU limits; throttling-ratio alert (§4.7) |
| High `some` in `cpu.pressure`, no throttling | CPU contention on the node: neighbors bursting above their requests | Raise requests to real usage; move to a dedicated pool; static CPU Manager policy for Guaranteed pods | Node pools per workload class; alert on node PSI (`node_pressure_cpu_waiting_seconds_total`) |
| Only pods on one node or zone are slow | A bad node: steal time (`st` in `vmstat`, §4.1), disk or network errors | Cordon, drain, replace | Node problem detection with automatic cordon |
| Every pod slow, CPU normal | A slow dependency, connection-pool waits, GC pauses, or traffic that outran the autoscaler | Traces show the slow span; scale out; check pool wait time | Per-dependency SLOs; scale on RPS or queue depth (KEDA) instead of CPU |
| Started with a deploy | A regression | Roll back, then investigate | Canary analysis on latency SLIs (Argo Rollouts) |

### 6.6 Linux and Network: TLS, Timeouts vs Refused, MTU

**Symptom:** a client can't connect, or connects and then hangs. The exact error names the layer to start at. Socket, DNS, `curl -w`, and certificate commands are in §4.2.

```bash
nc -vz -w 3 <host> 443          # refused (instant) or timed out (silence)?
openssl s_client -connect <host>:443 -servername <host> -verify_return_error </dev/null 2>&1 | grep -E 'verify|Verify|subject=|issuer='
tracepath -n <host>             # path MTU per hop; or: ping -M do -s 1472 -c 3 <host>
```

| You see | Likely cause | Fix | Guardrail |
|---|---|---|---|
| `Connection refused`, instantly | Host reachable, nothing listening on that port: process down, bound to 127.0.0.1, or the wrong port | `ss -tlnp` on the target (§4.2) | Readiness probes on the serving port; port conventions in templates |
| Timeout, no answer | Packets dropped: security group, NACL, NetworkPolicy, host firewall, routing, or a full SYN backlog | `tcpdump` on both ends to find where the SYNs stop | Policy tests; synthetic reachability checks; AWS VPC Reachability Analyzer |
| `certificate has expired`, `unknown authority`, or a hostname mismatch | Expired cert, missing intermediate, a private CA that isn't trusted, or the wrong SNI | Renew (cert-manager); serve the full chain; distribute the CA bundle (trust-manager); fix SNI | Alert when `certmanager_certificate_expiration_timestamp_seconds - time()` drops below 14 days |
| `handshake failure` or `protocol version` | TLS version or cipher mismatch, a missing client certificate for mTLS, or an ALPN mismatch | Align the minimum version; present the client cert | One TLS policy, set at the gateway or mesh |
| Small requests work, large ones hang | MTU black hole: overlay overhead (VXLAN 50 bytes, WireGuard 60 bytes over IPv4) plus blocked ICMP "fragmentation needed" | Set the CNI MTU to fit the underlay (AWS VPCs allow 9001-byte jumbo frames inside the VPC); allow ICMP type 3 code 4; clamp MSS | MTU set in cluster templates; large-payload synthetic checks |
| `Too many open files` or `cannot assign requested address` | File-descriptor limit, or ephemeral port exhaustion (§3.8) | Raise `nofile`; pool connections | Alerts on fd usage and `TIME_WAIT` counts |

### 6.7 Terraform / OpenTofu Failures

**Symptom:** `plan` or `apply` errors, or a plan shows changes nobody made. Pause auto-apply while you investigate. State, import, and refactoring commands are in §4.4.

```bash
TF_LOG=DEBUG TF_LOG_PATH=./tf-debug.log terraform plan -no-color   # every provider API call and error
terraform plan -refresh=false   # diff disappears = real-world drift; diff stays = code, state, or provider normalization
aws sts get-caller-identity     # the identity the provider will actually use
```

| You see | Likely cause | Fix | Guardrail |
|---|---|---|---|
| `Error acquiring the state lock` | A run is in progress, or a crashed run left the lock behind | Read the lock info (who, when, which operation); wait, or `force-unlock` once you've proved nothing holds it (§4.4) | One queue per state (Atlantis, HCP Terraform, CI concurrency groups); on the S3 backend, native locking (`use_lockfile = true`) replaces the DynamoDB table |
| Changes nobody made | Drift: console edits, a second tool owning the same resource, an autoscaler changing counts | Apply to revert, or codify the change; `lifecycle { ignore_changes = [...] }` for fields another system owns | Scheduled drift detection (`-detailed-exitcode`, §4.4); deny console writes in production |
| The same diff after every apply | Provider normalization (JSON policy ordering, case), inline rules mixed with standalone rule resources, `default_tags` overlap | `jsonencode()` or `aws_iam_policy_document`; manage rules one way only; upgrade the provider | A post-apply check that fails when `plan -detailed-exitcode` returns 2 |
| `No valid credential sources`, `ExpiredToken`, or `AccessDenied` | Wrong profile or role, an expired SSO session, or a CI OIDC trust policy that doesn't match the repo, branch, or environment | Check the identity; fix the role's trust conditions | OIDC only, no static keys; one least-privilege role per state |
| Plan destroys and recreates after a refactor | The address changed (rename, module move, `count` to `for_each`) | `moved` blocks, one per instance when going from `count` to `for_each` | Policy on the plan JSON that blocks deleting stateful resources; `prevent_destroy` on databases |
| `already exists` on create | Created out of band, or a failed apply lost track of it | An `import` block plus `-generate-config-out` (§4.4) | Everything goes through the pipeline; `managed-by` tags |
| Lock file checksum mismatch in CI | `.terraform.lock.hcl` lacks hashes for the CI platform | `terraform providers lock` for every platform (§4.4) | Commit the lock file; Renovate opens provider bumps |

### 6.8 CI/CD and GitOps

**Symptom:** an app stuck `OutOfSync` or `Degraded`, a sync that hangs, a canary that aborts, or a pipeline that fails and then passes on retry. The Argo CD CLI is in §4.6.

```bash
kubectl get application <app> -n argocd -o jsonpath='{.status.sync.status} / {.status.health.status}{"\n"}{.status.operationState.message}{"\n"}{range .status.conditions[*]}{.type}: {.message}{"\n"}{end}'
kubectl argo rollouts get rollout <rollout> -n <ns>   # canary step, pause reason, AnalysisRun result
gh run view <run-id> --log-failed                     # only the failed steps' logs
```

| You see | Likely cause | Fix | Guardrail |
|---|---|---|---|
| `OutOfSync` again right after every sync | Something else mutates the live object: an HPA owning `replicas`, a mutating webhook, server-side defaults | Drop `replicas` from Git when an HPA owns it; `ignoreDifferences` plus `RespectIgnoreDifferences=true`; server-side diff | Lint: workloads with an HPA don't set `spec.replicas` |
| `ComparisonError`, or manifest generation failed | A Helm or Kustomize render error, missing values, or repo credentials | Render locally (`argocd app diff --local`, §4.6); fix the credentials | CI renders and validates every app (`kustomize build`, `helm template`, kubeconform) before merge |
| Sync fails with `could not find the requested resource` | A custom resource synced before its CRD | Sync waves (`argocd.argoproj.io/sync-wave`: CRDs and namespaces first) or `SkipDryRunOnMissingResource=true` | CRDs live in their own, earlier Application |
| Sync stuck `Running` | A PreSync hook Job (often a migration) hanging or retrying | `kubectl logs job/<hook> -n <ns>`; `argocd app terminate-op` (§4.6) | Hook Jobs must be idempotent and set `activeDeadlineSeconds` and `backoffLimit` |
| `Synced` but `Degraded`, or `Progressing` forever | Unhealthy workload (§6.1–6.2), `progressDeadlineSeconds` exceeded, or a custom resource with no health check | Fix the pods; add a Lua health check (`resource.customizations.health.<group>_<kind>` in `argocd-cm`) | Every platform CRD ships with a health check |
| Canary aborted, AnalysisRun `Failed` | A real regression, so the gate worked | Leave it rolled back; fix forward through Git | Shared, tested AnalysisTemplates in the golden path |
| AnalysisRun `Error` or `Inconclusive` | The metric query is broken or returns no data | Run the query against the stable version; fix the template; `kubectl argo rollouts retry rollout <rollout>` | Analysis queries tested in CI like any other code |
| Pipeline fails, passes on retry | A flaky test: shared state, ordering, real network calls, starved runners, time-dependent logic | Quarantine it with an owner and a deadline; make it hermetic | Track flake rate per test; no silent pipeline-level retries |

Deeper: [Domain 2 Study Guide](Domain_2_Linux_Networking_Fundamentals/Study_Guide.md) · [Domain 3 Study Guide](Domain_3_Kubernetes_Platform_Engineering/Study_Guide.md) · [Domain 4 Study Guide](Domain_4_Infrastructure_as_Code/Study_Guide.md) · [Domain 5 Study Guide](Domain_5_CICD_GitOps_Delivery/Study_Guide.md) · [Mock 2: Troubleshooting](Mock_Interviews/Mock_Interview_2_Troubleshooting.md)

---

## 7. Behavioral Structure

### 7.1 STAR+L in One Table

| Part | Time (about 2 min) | What goes in it | What the interviewer listens for |
|---|---|---|---|
| **Headline** | 5 s | One sentence: what you did and why it mattered | That you lead with the point |
| **Situation** | 15–20 s | Scale, constraint, what was broken. No org-chart tour | Stakes, in numbers |
| **Task** | 10 s | Your specific responsibility, and what wasn't yours | Where your ownership started and stopped |
| **Action** | 60–75 s | Three or four actions, each with the *why* | Judgment and trade-offs, in "I" |
| **Result** | 15–20 s | Numbers against a baseline, plus one change that lasted | A measured, durable outcome |
| **Learning** | 10–15 s | One specific thing you now do differently, with proof | Self-awareness, not a platitude |

STAR+L is the default. Use CARL (Context, Action, Result, Learning) for short phone-screen answers where the task is obvious.

### 7.2 Competency to Story Map

Have at least one story per row. One strong story can cover two rows; don't stretch it to more than three.

| Competency | Typical prompt | The story must show | Platform example |
|---|---|---|---|
| **Ownership** | "Tell me about something you owned end to end." | You took it on, finished it, and did the unglamorous parts (docs, on-call, deprecation) | Shipped a golden path and ran its first incidents |
| **Influence without authority** | "How did you get teams to adopt something you didn't control?" | Data, a pilot, making the right thing the easy thing, without a mandate | Moved 14 teams off Jenkins with templates and generated migration PRs |
| **Conflict** | "Tell me about a disagreement with another team." | You understood their constraint, argued on the merits, and committed to the outcome | Security wanted manual approvals; you shipped policy as code instead |
| **Failure** | "Tell me about a time you failed." | A real failure you caused, the fix, and the mechanism that prevents a repeat | A rushed cluster upgrade broke ingress; you introduced canary clusters |
| **Ambiguity** | "Tell me about a problem with no clear owner or answer." | How you framed it, what you measured first, and what you chose not to do | Turned "CI is slow" into a measured, prioritized plan |
| **Mentoring** | "How have you grown other engineers?" | Someone else's behavior or level changed, and you let go of the work | Coached a teammate to run on-call and lead postmortems |
| **Product sense** | "How did you decide what the platform should build?" | User research, adoption data, and saying no | Interviews and survey data killed a portal feature and funded faster CI |

### 7.3 The 90-Second Pitch

About 200–230 words. Rehearse it out loud until it sounds natural, not memorized.

| Part | Time | Fill in |
|---|---|---|
| Present | ~15 s | "I'm a ___ at ___, where I own ___ for about ___ engineers." |
| Two highlights | ~40 s | "Two highlights. First, I ___, which took ___ from ___ to ___. Second, ___." |
| The thread | ~20 s | "What connects those is ___. Platform engineering is that work as a discipline." |
| Why this role | ~15 s | "That's why this role stood out: you're ___, and the posting says ___. I'd like to hear ___." |

Leave out the chronological résumé walk and the tool list. Domain 10 §6.2 has a worked example, and the [Domain 10 workbook](Domain_10_Behavioral_Career/Exercises.md) has versions for DevOps and SRE backgrounds.

### 7.4 Level Signals

| Signal | Mid | Senior | Staff |
|---|---|---|---|
| Story scope | A task or feature inside your team | A capability across several teams | An org-wide problem spanning quarters |
| Starting point | A clear problem | A fuzzy goal you turned into a plan | A problem nobody owned, which you found and framed |
| Language | "I implemented..." | "I decided X because Y, at the cost of Z" | "I set the direction, and these teams changed how they work" |
| Proof of impact | It works | Metrics against a baseline | Other people's behavior changed and kept changing without you |
| Multiplier | Helps teammates | Mentors; raises the team's standards | Grows senior engineers; shapes standards and hiring |

Down-leveling usually comes from story scope, not technical depth (Domain 10 §7).

### 7.5 Presenting a Take-Home

| Minutes (of about 45) | Cover | Tip |
|---|---|---|
| 0–3 | The problem, the constraints, and what you scoped out and why | Lead with the problem, not the tooling |
| 3–10 | Demo the golden path: one command or one PR to a running service | Keep a recorded backup in case the live demo fails |
| 10–20 | The architecture and the 2–3 decisions that mattered, with your ADRs | For each: the options, your choice, and what it cost |
| 20–25 | Operations: SLOs, failure modes, security, cost | Show one failure and how you'd detect it |
| 25–30 | What you'd do with another week, and what changes at 10x | Prioritized, with reasons |
| 30–45 | Questions | Concede gaps plainly and say how you'd find out |

Use the §5.2 checklist as the slide skeleton. Mock 9 has the rubric.

### 7.6 Questions to Ask

Ask two or three per round, matched to the interviewer. Your questions are scored too.

| Interviewer | Ask |
|---|---|
| Hiring manager | "How do you measure whether the platform is succeeding?" · "What would make this hire a clear success after a year?" · "What's on the roadmap that you decided not to do?" |
| Peer engineer | "Roughly what share of your week goes to tickets and interrupts?" · "What does on-call look like: pages per week, rotation size?" · "What changed after your last major incident?" |
| Skip-level or director | "How is platform work weighed against product work in planning?" · "What would make you invest more in the platform, or less?" |
| A platform customer (product engineer) | "What's the most annoying part of shipping here?" · "When the platform breaks, how do you find out, and who helps?" |

What healthy and red-flag answers sound like: Domain 10 §6.6–6.7.

Deeper: [Domain 10 Study Guide](Domain_10_Behavioral_Career/Study_Guide.md) · [Domain 10 Workbook](Domain_10_Behavioral_Career/Exercises.md) · [Mock 8: Hiring Manager](Mock_Interviews/Mock_Interview_8_Behavioral_Hiring_Manager.md) · [Mock 9: Take-Home](Mock_Interviews/Mock_Interview_9_Take_Home_Presentation.md)

---

## 8. Interview Reflexes

Habits to run on autopilot. The Golden Interview Rules in the [README](README.md) are the short version; these are the round-by-round reflexes.

**Phone screen**

1. Answer in one sentence, then offer depth: "Want me to go into how the scheduler scores nodes?"
2. Use current terms and show you know the old ones: failed deployment recovery time (not MTTR), SLSA Build L3 (not "SLSA 4"), Gateway API (Ingress is frozen).
3. If you don't know, say so, reason from first principles, and say how you'd verify.
4. Tie every tool back to the problem it solves. A tool name is not an answer.
5. Keep each answer to about 90 seconds unless they ask for more.

**Troubleshooting**

6. Before running anything, ask for the symptom, the scope (one pod, node, zone, or everyone?) and what changed recently.
7. If it started with a change, roll back first. Root cause comes after the bleeding stops.
8. State the hypothesis before each command, and what result would rule it out.
9. Read the exact error text out loud. It usually names the layer.
10. Follow the request path or go bottom-up, and say which one you chose.
11. Give a status update every 15–30 minutes. Communication is part of the score.
12. Finish with the guardrail and the alert that would have caught it earlier.

**System design**

13. Spend the first five minutes on users, scale, SLOs, tenancy, and compliance, and write the assumptions down.
14. Draw the developer's journey before drawing boxes.
15. Separate control plane from data plane, and say what keeps working while the control plane is down.
16. For each component, say what breaks first at 10x and how you'd know.
17. State the blast radius and the rollout order: canary cell, one region, then everything.
18. Make the build-vs-buy call explicitly, and the team size it implies.
19. Include the migration from today's state. Nobody gets a greenfield.
20. Put a number on cost and say who pays (showback).
21. Close with three metrics that would prove it worked.

**Coding**

22. Restate the problem and confirm inputs, outputs, and constraints before typing.
23. Write the signature and one worked example first.
24. Get a simple correct version working, then optimize, and say which phase you're in.
25. State time and space complexity without being asked.
26. Before calling it done, trace an edge case by hand: empty input, duplicates, a timeout, a malformed record.
27. In concurrent code, say who closes the channel, how cancellation propagates (`context`), how parallelism is bounded, and what happens on partial failure.
28. Handle every error explicitly and log it in a structured way. Platform code runs unattended at 3 a.m.

**Behavioral**

29. Pick the story for the competency being probed, not your favorite story.
30. Say "I" for your decisions and "we" for the team's results.
31. Quantify against a baseline, and name one outcome that outlasted your involvement.
32. Bring a real failure, plus the mechanism you put in place afterwards.
33. When they drill down, add detail instead of repeating the headline.

Deeper: [README](README.md) · [Mock Interview Guide](Mock_Interviews/README.md) · [Interviewer Protocol](Mock_Interviews/Interviewer_Protocol.md)

---

## 9. Day-of Logistics

### 9.1 Setup Checklist (the Day Before)

- [ ] Camera at eye level, light in front of you, plain background.
- [ ] Headset or external mic tested in the interview's exact tool (Zoom, Meet, Teams, or the coding platform's built-in call).
- [ ] On Arch with Wayland, screen sharing tested in that same browser or app. It depends on PipeWire and `xdg-desktop-portal`, and it can fail silently.
- [ ] IDE ready with Go and Python, a scratch file per language, and tests that run with one keystroke. AI completion off unless the interviewer allows it.
- [ ] Whiteboard tool open and practiced (Excalidraw, Miro, or the company's own), with the §5.1 steps ready as headings.
- [ ] Notifications off, other apps closed, laptop plugged in, phone hotspot ready as backup internet.
- [ ] Water, a pen, paper, and your pitch and story list printed or open in one document.
- [ ] Interviewer names and roles from the schedule, and the job description open.

### 9.2 Ten Minutes Before Each Round

1. Re-read the interviewer's name and role, and find the matching row in the map at the top of this sheet.
2. Skim that section for five minutes, no more.
3. Pick the two or three stories most likely to come up.
4. Water, bathroom, close everything else, and join two minutes early.
5. In the first 30 seconds, confirm the format and the time box, then start.

### 9.3 Between Rounds

- Write two lines on what was asked and how it went, then let it go. One weak round doesn't automatically sink a loop, but carrying it into the next round can.
- Stand up, eat something, drink water.
- Open only the next round's section.

### 9.4 After the Loop

- Within the hour, write down every question, your answer, and what you'd change. Memory fades fast.
- Log it in the [Mock Interview Log](Mock_Interviews/Mock_Interview_Log.md) in the same format as a mock, so real loops and mocks feed one list of gaps.
- Send thank-you notes within 24 hours, through the recruiter if you don't have direct emails. Keep them short and mention one thing you enjoyed discussing.
- Ask the recruiter for the decision timeline, and don't negotiate on that call. For offers, see Domain 10 §9.

Deeper: [Domain 10 Study Guide](Domain_10_Behavioral_Career/Study_Guide.md) · [Mock Interview Log](Mock_Interviews/Mock_Interview_Log.md)

---
