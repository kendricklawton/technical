# PE Domain 10: Behavioral & Career — Workbook

## How to Use This Workbook

The other domains have labs that run on a kind cluster. This domain's "labs" produce the artifacts that decide behavioral and hiring-manager rounds: a story bank, a pitch, resume bullets, product-sense answers, a 30-60-90 day plan, and a portfolio project. Each exercise has the same shape:

- **Goal:** what you will have when you finish.
- **Instructions:** the steps, in order.
- **Template:** a fill-in block. Copy it into your own notes and fill it in there.
- **Worked Example:** a model answer. Read it *after* your first draft, not before, or you will anchor on it.
- **Self-Review Checklist:** score your draft honestly before you rehearse it.

> [!WARNING]
> Every worked example is fictional and illustrative: the companies, numbers, and people are invented to show the *shape* of a strong answer. Replace every detail with your own experience. Interviewers recognize recited stories, and drill-down questions will expose any detail you cannot explain from memory.

> [!IMPORTANT]
> Keep your filled-in templates outside this repository (a private notes folder or document). They will contain employer names, internal metrics, and colleagues' names that do not belong in a shared learning repo.

| Exercise | Time | Output | Where you will use it |
| :--- | :--- | :--- | :--- |
| 1. Build your story bank | 3–4 hours over a week | 15 stories, 2 fully written, follow-ups prepared | Every behavioral round |
| 2. 90-second pitch | 1 hour plus rehearsal | A pitch per role family you target | Recruiter screen, hiring manager, every "tell me about yourself" |
| 3. Resume bullet clinic | 2 hours | 10 rewritten bullets | Resume, LinkedIn, recruiter screens |
| 4. Product-sense drill | 90 minutes | Three written answers | Hiring manager, IDP system design |
| 5. 30-60-90 day plan | 1 hour | A plan you can talk through in two minutes | Hiring manager; your actual first quarter |
| 6. Homelab IDP portfolio spec | 1 hour to spec, weeks to build | A project plan and demo script | Take-home presentations, portfolio, deep dives |

Work through the [Study Guide](Study_Guide.md) first, practice with the [Interview Question Bank](Interview_Questions.md), and test yourself under realistic conditions with [Mock Interview 8: Behavioral and Hiring Manager](../Mock_Interviews/Mock_Interview_8_Behavioral_Hiring_Manager.md).

---

## Exercise 1: Build Your Story Bank

### Goal

A bank of 15 stories mapped to the 14 platform-engineer competencies (Study Guide section 3), with two stories written out in full STAR+L form, a 60-second version of each, and prepared answers to the drill-down follow-ups interviewers are most likely to ask.

### Instructions

1. Block 90 minutes. Open your calendar for the last two or three years, your pull request history, your incident tracker, and any performance reviews or promotion packets.
2. Brain-dump at least 25 candidate moments using the prompts in Study Guide section 4.2 (incidents, migrations, automation wins, cost savings, launches, decisions, people, pressure, mistakes, ambiguity). Do not filter yet.
3. Pick the 15 that give the best coverage and fill in the bank table below. Each story should cover two to four competencies.
4. Mark the coverage matrix in Study Guide 4.3 for your own stories. Every competency needs at least two stories. Mine more moments for any gap.
5. Pick your two strongest stories and write them in full using the story card. Read each aloud with a timer; target 1:45–2:30.
6. Write a 60-second version of each: headline, one action, one number, one learning.
7. For each full story, write answers to the five follow-ups in the follow-up template.
8. Record yourself telling both stories. Play the recordings back and note where you rambled, hedged, or said "we" for something you did.
9. Over the following week, write the remaining 13 stories as story cards (bullets are fine) and rehearse them aloud.

### Template

**Story bank**

| # | Working title | Year | My role (one sentence) | Key metric: before → after (source) | Competencies (2–4) | Amazon LPs | 60 s ready | 2 min ready |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :---: | :---: |
| 1 | | | | | | | [ ] | [ ] |
| 2 | | | | | | | [ ] | [ ] |
| 3 | | | | | | | [ ] | [ ] |
| 4 | | | | | | | [ ] | [ ] |
| 5 | | | | | | | [ ] | [ ] |
| 6 | | | | | | | [ ] | [ ] |
| 7 | | | | | | | [ ] | [ ] |
| 8 | | | | | | | [ ] | [ ] |
| 9 | | | | | | | [ ] | [ ] |
| 10 | | | | | | | [ ] | [ ] |
| 11 | | | | | | | [ ] | [ ] |
| 12 | | | | | | | [ ] | [ ] |
| 13 | | | | | | | [ ] | [ ] |
| 14 | | | | | | | [ ] | [ ] |
| 15 | | | | | | | [ ] | [ ] |

**Story card (STAR+L)**

```text
TITLE:                                                   YEAR:
COMPETENCIES:                                            AMAZON LPs:
HEADLINE (one sentence: what happened and why it matters):

SITUATION (2-3 sentences; scale, constraint, what was broken):

TASK (my specific responsibility; who else owned what):

ACTIONS (3-5; each with the WHY):
  1.
  2.
  3.
  4.
SETBACK (one thing that went wrong midway and what I did):

RESULT (baseline -> outcome; how measured; one non-numeric outcome):

LEARNING (what I do differently now; where I have applied it since):

60-SECOND VERSION:
```

**Follow-up preparation**

```text
1. What exactly did YOU do versus the team?
2. Why that approach? What two alternatives did you reject, and why?
3. How did you measure the result? Over what period? What else could explain it?
4. Who disagreed or resisted, and how did you handle it?
5. What would you do differently?
```

### Worked Example

Two complete stories from the fictional engineer whose bank appears in Study Guide 4.3: a senior DevOps engineer at a SaaS company with about 400 engineers running on AWS and EKS.

#### Story A: Migrating 120 services from Jenkins to GitHub Actions and Argo CD

**Competencies:** large migrations, driving adoption, handling resistance, technical decision-making, cross-team collaboration. **Amazon LPs:** Deliver Results, Invent and Simplify, Earn Trust.

**Headline:** "I led the migration of 120 services across 14 teams from Jenkins to GitHub Actions and Argo CD in nine months, without a mandate, and we shut Jenkins down two weeks ahead of plan."

**Situation:** "We ran six Jenkins controllers with roughly 300 plugins and about 40 always-on build agents. I counted 37 distinct Jenkinsfile patterns, mostly copy-paste variants. Plugin upgrades had caused two Sev-2 incidents in the previous two quarters, deploys ran `helm upgrade` from Jenkins with cluster-admin credentials stored in Jenkins (an open audit finding), and median merge-to-production time was about two days, mostly waiting on a manual change ticket for production."

**Task:** "My manager asked me to lead the migration, and leadership wanted Jenkins gone within the fiscal year. I owned the plan, the migration tooling, and engagement with the product teams. Three other platform engineers worked on it part-time: two built the runner infrastructure and one owned the Argo CD setup."

**Actions:**

1. "I started with an inventory instead of a design. I wrote a script that parsed every Jenkinsfile and job config, and found five archetypes (JVM service, Node service, Python worker, scheduled job, library) that covered about 85% of services. That told me tooling could do most of the work, and that the remaining 15% would need people."
2. "I built the paved road before asking anyone to move: reusable GitHub Actions workflows per archetype that build, test, scan, and push to ECR using OIDC federation, so there were no long-lived AWS keys in CI, and an Argo CD ApplicationSet that generated an application per service per environment from the config repo layout. Production promotion became a pull request to the environment config, and I worked with the change-management group for six weeks to get that pre-approved as a standard change, which is what actually removed the two-day wait."
3. "Wave one stalled: teams agreed in principle, then never found the time. So I wrote a converter that generated the workflow file and the Argo CD manifests from each Jenkinsfile for the five archetypes and opened the pull request in each repo. Teams went from 'migrate your pipeline' to 'review and merge this PR'. Merge rates went up immediately."
4. "The payments team refused. They were worried about losing their manual production gate for PCI reasons. I met with their tech lead and their security partner, and we kept a human gate in a different place: CODEOWNERS and branch protection on their production config, plus Argo CD sync windows that only allow production syncs during staffed hours. They migrated in wave three and later presented the setup at our internal engineering forum."
5. "I made progress public: a weekly burndown in the engineering channel and twice-weekly office hours. Each service's Jenkins job stayed runnable with triggers disabled for 30 days after cutover, so rollback was a single click."

**Setback:** "In wave one, JVM build times regressed by about 30% because hosted runners did not have the warm Gradle cache our Jenkins agents had. Teams noticed immediately. We added dependency caching to the reusable workflows and moved heavy builds to autoscaling self-hosted runners on EKS, which brought times back below the Jenkins baseline within two weeks."

**Result:** "118 of 120 services migrated in nine months; the other two were decommissioned instead. Median merge-to-production time went from about two days to about 45 minutes, measured from merge commit to the production sync in Argo CD over four weeks. We shut down six controllers and 40 agents, roughly $11k a month in EC2, closed the audit finding because CI no longer holds cluster credentials, and we've had no pipeline-platform incidents since. CI/CD satisfaction in the quarterly survey went from 2.9 to 4.1 out of 5."

**Learning:** "Migrations are adoption problems more than technical problems. The converter that opened pull requests did more than any amount of documentation, and I built it a month too late, after wave one stalled. On our secrets migration the following year I built the converter before the first wave, and that wave finished in half the time."

**60-second version:** "I led our move of 120 services from Jenkins to GitHub Actions and Argo CD in nine months. The key decision was to treat it as an adoption problem: after the first wave stalled, I wrote a converter that opened a ready-to-merge pull request in every repo, so teams reviewed instead of rewrote. I also got GitOps pull requests pre-approved as standard changes, which took median merge-to-production from two days to about 45 minutes. We shut Jenkins down two weeks early. The lesson I took away: build the migration tooling before the first wave, not after it stalls."

**Follow-ups prepared:**

| Follow-up | Prepared answer (summary) |
| :--- | :--- |
| What did you do versus the team? | Inventory script, wave plan, converter, the standard-change negotiation, office hours, the payments-team agreement. Colleagues built the runners and the ApplicationSet templates. |
| Why GitHub Actions and Argo CD? | Code already lived in GitHub; reusable workflows and OIDC removed stored credentials. Tekton was considered but would have recreated the self-hosted burden we were removing. Argo CD gave pull-based delivery, drift detection, and a Git audit trail without cluster credentials in CI. |
| How did you measure lead time? | Merge commit timestamp to the Argo CD production sync of that image tag, median over four weeks before and after, computed with a small script over Git and Argo CD notification data. |
| Who resisted? | The payments team (PCI and the manual gate); solved with CODEOWNERS, branch protection, and sync windows rather than an exception. |
| What would you do differently? | Build the converter first, and involve change management in week one instead of week eight. |

#### Story B: Leading a Sev-1 caused by an expired certificate

**Competencies:** ownership and incident leadership, delivering under pressure, raising the bar. **Amazon LPs:** Ownership, Bias for Action, Earn Trust, Insist on the Highest Standards.

**Headline:** "I was incident commander for a Sev-1 in which an expired TLS certificate took our partner API down for 38 minutes, and I led the follow-up that removed certificate expiry as a failure mode across our AWS estate."

**Situation:** "Our partner API, which about 200 business customers integrate with, sat behind an Application Load Balancer using a certificate from a commercial CA that had been imported into ACM years earlier. ACM does not renew imported certificates. The expiry reminder emails went to a distribution list owned by a team that had been reorganized eight months before. At 09:12 UTC on a Tuesday, our synthetic checks started failing TLS handshakes."

**Task:** "I was the platform on-call. I declared the incident and took the incident commander role."

**Actions:**

1. "I declared a Sev-1 within four minutes, because partner traffic was failing completely, and assigned roles: an engineer from the API team as operations lead, and our support manager as communications lead with a status-page update every 15 minutes. I made myself stay out of the terminal so I could keep the bridge focused."
2. "The operations lead confirmed the cause in about six minutes with `openssl s_client -connect` piped to `openssl x509 -noout -enddate`, which showed a `notAfter` date earlier that morning."
3. "We had two mitigation paths: an emergency reissue from the commercial CA, whose turnaround we couldn't predict, or a new ACM-issued certificate validated through our existing Route 53 zone, attached to the listener. I chose the ACM path because it was faster and easily reversible, and asked a second engineer to start the CA reissue in parallel as a fallback. ACM issued the certificate in about 12 minutes, we swapped it on the HTTPS listener, and handshakes recovered at 09:50."
4. "In the blameless postmortem, I pushed the root cause past 'someone missed an email' to 'a production dependency had an expiry date and no owner or alert'. I took the action items: an inventory script across all 31 accounts that listed ACM certificates and in-cluster cert-manager certificates (212 in total, nine expiring within 60 days); routing of ACM's approaching-expiration events to the owning team's pager using a mandatory owner tag; the AWS Config rule `acm-certificate-expiration-check` for an organization-wide compliance view; moving most imported certificates to ACM-issued certificates that renew automatically; and adding expiry checks to our production readiness review."

**Result:** "Customer impact lasted 38 minutes. Over the next quarter we moved 80% of imported certificates to auto-renewing ACM certificates, and every remaining one has an owner and an alert at 30 days. We have had no certificate-expiry incidents in the 18 months since, and the inventory now runs daily."

**Learning:** "Ownership metadata is a reliability control: the certificate expired because nobody owned it, not because someone was careless. Anything with an expiry date (certificates, tokens, keys, domain registrations) now needs an owner and an alert before I consider it production-ready. As incident commander, I also learned to name a communications lead in the first five minutes; on an earlier incident I tried to do comms myself and it slowed diagnosis."

**60-second version:** "An imported certificate on our partner API's load balancer expired and took the API down. I was on call, declared a Sev-1, assigned an ops lead and a comms lead, and when we had two mitigation paths I chose a new ACM-issued certificate because it was fast and reversible, while a colleague pursued a reissue in parallel. We recovered in 38 minutes. In the postmortem I reframed the cause as 'no owner, no alert', inventoried 212 certificates across 31 accounts, and routed expiry alerts by owner tag. There have been no expiry incidents in 18 months."

**Follow-ups prepared:**

| Follow-up | Prepared answer (summary) |
| :--- | :--- |
| Why not just renew with the CA? | Turnaround was unpredictable; the ACM path was faster and reversible. We ran both in parallel. |
| Whose fault was it? | Nobody's individually. The system lacked ownership and alerting; the engineer who had set it up had moved teams. The postmortem said so without naming anyone. |
| How did you get teams to own their certificates? | Made it near-zero effort: alerts arrived automatically via the owner tag, and the inventory pre-filled who owned what for them to confirm. |
| How did you measure "no recurrence"? | Incident tracker query on the expiry cause category, plus the daily inventory showing no certificate under 30 days without an acknowledged owner. |
| What would you do differently? | Name the comms lead even faster, and add an external TLS-expiry check to synthetic monitoring, which would have warned us weeks earlier. |

### Self-Review Checklist

- [ ] The bank has 15 stories, and every competency in Study Guide section 3 has at least two.
- [ ] It includes at least two incidents, two adoption or influence stories, one failure, one conflict with a peer, and one with a manager.
- [ ] Every story has a baseline-to-outcome number, and I know where each number came from.
- [ ] I can say my personal role in each story in one sentence.
- [ ] Each two-minute version runs between 1:45 and 2:30 out loud; each 60-second version is under 70 seconds.
- [ ] Each learning is specific, and for at least half I can say where I have applied it since.
- [ ] Each story names at least one alternative I rejected and why.
- [ ] No story depends on internal names or acronyms the interviewer would not know.
- [ ] No confidential figures; I use ratios where exact numbers are sensitive.
- [ ] I recorded myself and fixed the three worst habits I heard.

---

## Exercise 2: The 90-Second "Tell Me About Yourself" Pitch

### Goal

A rehearsed pitch of about 200–230 words (roughly 90 seconds at a natural pace) that frames you as a platform engineer, includes two quantified highlights, and ends with a hook that invites the interviewer to talk about their platform. Build one version per role family you target (for example, platform engineer at a product company versus at an infrastructure vendor).

### Instructions

1. Re-read the job description and highlight the three things the team most needs (for example: self-service infrastructure, CI/CD at scale, Kubernetes multi-tenancy).
2. Pick the two stories from your bank that best match those needs and have the strongest numbers.
3. Fill in the template. Write it out in full first, then trim.
4. Check the word count (`wc -w pitch.txt`) and aim for 200–230 words.
5. Read it aloud with a timer. If it runs over 100 seconds, cut the context, not the numbers.
6. Rehearse until you can deliver it conversationally from the four headings alone. Do not memorize it word for word; recited pitches sound recited.
7. Prepare a 30-second version for recruiter calls and networking.

### Template

```text
PRESENT (~15 s, ~35 words)
I'm a <role> at <type of company> with <scale: engineers, cloud, platform>.
I own <the platform-shaped part of your job>.

PAST HIGHLIGHTS (~40 s, ~100 words)
Highlight 1: I <action> <scope> using <technology>; <metric before -> after>.
Highlight 2: I <action> <scope>; <metric>, because <the product-minded reason it worked>.

THE THREAD (~20 s, ~45 words)
The pattern across my work is <building things other engineers use / making the right
thing the default / turning expert-only infrastructure into self-service>.
That's platform engineering, and I want to <do it as my core job / do it at larger scale>.

FUTURE / WHY THIS ROLE (~15 s, ~35 words)
This role stood out because <one specific fact about their platform or team>.
<A hook question: "I'd love to hear how you measure success today.">
```

### Worked Example

**Example 1: DevOps engineer moving to platform engineering (204 words)**

> "I'm a senior DevOps engineer at a B2B SaaS company with about 400 engineers on AWS and EKS. I own our delivery platform: CI, GitOps, and the templates teams use to create new services.
>
> Two things I'm proudest of. I led the migration of 120 services from Jenkins to GitHub Actions and Argo CD. Median time from merge to production went from about two days to under an hour, and we shut down six Jenkins controllers. I also built our golden-path service template, and adoption went from 13% to 69% of services in two quarters, mostly because I treated the teams as customers: I interviewed them, fixed what they hated, and made the paved road the easiest road.
>
> The thread through my career is that the work I enjoy most is building things other engineers depend on, and then measuring whether it actually made their week better. That's platform engineering, and I want to do it as my core job rather than squeezed in between tickets.
>
> That's why this role caught my eye. You're building an internal developer platform for a fast-growing engineering org, and the posting talks about running it as a product. I'd love to hear how you measure its success today."

**Example 2: SRE moving to platform engineering (202 words)**

> "I'm a site reliability engineer at an e-commerce company with about 150 engineers, on the team that runs our Kubernetes platform and observability stack on AWS across two regions.
>
> Two highlights. I was incident commander for most of our major incidents last year, and after watching the same failure patterns repeat, I built an SLO framework into our service template: every new service gets dashboards, burn-rate alerts, and a runbook skeleton by default. Today 85 services have SLOs, and median time to restore for high-severity incidents dropped from about 70 minutes to about 30. I also trained six engineers as incident commanders, so major incidents no longer depend on the same three people.
>
> What I've learned is that the best reliability work is the kind teams adopt without being asked, because it's built into the path they already use. That's what pulls me toward platform engineering: preventing incidents by making the right thing the default, instead of responding after the fact.
>
> Your platform team owns both the developer platform and observability, and you're expanding into new regions next year, so reliability by default is going to matter a lot. I'd like to understand how your product teams engage with the platform today."

**Why these work:** each opens with role and scale, uses two highlights with baselines and outcomes, names the product-minded reason for success (treating teams as customers; defaults over mandates), connects the past to platform engineering explicitly, and ends with a question that hands the conversation back on the candidate's strongest ground.

### Self-Review Checklist

- [ ] 200–230 words; 80–100 seconds out loud.
- [ ] The first sentence states role, company type, and scale.
- [ ] Two highlights, each with a baseline and an outcome.
- [ ] At least one highlight shows product thinking (users, adoption, defaults), not just technical execution.
- [ ] The word "platform" appears in the thread, tied to evidence rather than enthusiasm.
- [ ] The "why this role" line contains a fact specific to this company.
- [ ] No chronological career walk, no list of tools, no certifications (they are on the resume).
- [ ] It ends with a natural hand-off question.
- [ ] I can deliver it from the four headings without reading.

---

## Exercise 3: Resume Bullet Clinic

### Goal

Ten resume bullets rewritten from duties into evidence, using the formula **action verb + scope + technology + quantified outcome**, each one defensible under drill-down.

### Instructions

1. Copy your ten most important current bullets into the template's "Original" column.
2. For each, fill in the four columns: the strongest honest action verb, the scope (services, teams, engineers, accounts, clusters), the technology, and the outcome with a baseline.
3. If you do not have a number, reconstruct one using Study Guide 4.4 (ticket counts, Git history, billing data), or use an honest qualitative outcome ("eliminated the manual approval step").
4. Rewrite each bullet in one or two lines. Lead with the verb; put the outcome where the eye lands.
5. Read each bullet and ask, "Could I talk about this for five minutes, including how I measured it?" If not, soften it or cut it.
6. Order bullets within each job by relevance to platform roles, not chronology.

Strong platform verbs: led, designed, built, migrated, automated, reduced, replaced, introduced, standardized, scaled, consolidated, launched, deprecated, mentored.

### Template

| # | Original | Action verb | Scope | Technology | Outcome (baseline → result) | Rewritten bullet |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | | | | | | |
| 2 | | | | | | |
| 3 | | | | | | |
| 4 | | | | | | |
| 5 | | | | | | |
| 6 | | | | | | |
| 7 | | | | | | |
| 8 | | | | | | |
| 9 | | | | | | |
| 10 | | | | | | |

### Worked Example

1. **Before:** "Managed Kubernetes clusters."
   **After:** "Operated 9 EKS clusters (about 1,400 pods across 3 regions) and ran quarterly version upgrades with zero customer-facing incidents over 6 upgrade cycles, using Pluto for deprecated-API detection and blue/green node groups."
   *What changed:* a scope number, a quality outcome, and the mechanism that made it safe.

2. **Before:** "Wrote Terraform."
   **After:** "Built a versioned Terraform module library (VPC, EKS, RDS, IAM) consumed by 30+ teams through a private registry, replacing 40+ copy-pasted stacks and cutting new-environment setup from 2 weeks to 2 days."
   *What changed:* "wrote" became a product with consumers, and the outcome is time to value.

3. **Before:** "Improved CI/CD pipelines."
   **After:** "Cut median PR pipeline time from 22 to 9 minutes for 120 engineers by adding remote build caching and autoscaled self-hosted runners; slow CI fell from the #1 to the #5 complaint in the developer survey."
   *What changed:* a baseline, the population affected, and a developer-experience signal.

4. **Before:** "Set up monitoring and alerting."
   **After:** "Introduced SLOs with multi-window burn-rate alerts for 85 services via the service template, reducing pages by about 40% and median time to restore for Sev-1/Sev-2 incidents from ~70 to ~30 minutes."
   *What changed:* it describes a default that teams inherit, and two outcome metrics.

5. **Before:** "Worked on CI security."
   **After:** "Removed long-lived AWS access keys from CI for 120 repositories by moving to GitHub Actions OIDC federation with per-repository IAM roles, closing a high-severity audit finding two months early."
   *What changed:* a concrete risk removed, the scope, and an outcome the business cares about.

6. **Before:** "Helped reduce cloud costs."
   **After:** "Reduced EKS compute spend by about a third (~$61k/month) through Karpenter consolidation, Spot for stateless tiers, and moving 70 services to Graviton, with no SLO regressions."
   *What changed:* "helped" became ownership, and the guardrail (no SLO regressions) pre-empts the obvious follow-up.

7. **Before:** "Built a developer portal."
   **After:** "Launched a Backstage portal with a catalog of 140 services and 3 golden-path templates, cutting time from new-service request to first production deploy from 9 days to 1."
   *What changed:* the outcome is developer time to value, not the existence of a portal.

8. **Before:** "Handled infrastructure tickets."
   **After:** "Replaced ticket-based namespace and AWS account provisioning with self-service (Backstage template, Terraform, Argo CD), cutting provisioning from ~3 days to 15 minutes and removing ~25 tickets per week."
   *What changed:* the work moved from doing tickets to eliminating them, which is the core platform signal.

9. **Before:** "Participated in on-call and incident response."
   **After:** "Served as incident commander for 12 Sev-1/Sev-2 incidents; wrote the IC playbook and started a shadow rotation that grew the incident commander pool from 3 to 9 engineers."
   *What changed:* leadership and a multiplier effect instead of participation.

10. **Before:** "Migrated applications to Kubernetes."
    **After:** "Migrated 60 services from EC2 Auto Scaling groups to EKS in 3 waves over 6 months using a shared Helm library chart, cutting deploy time from ~40 to ~8 minutes and raising average node utilization from ~25% to ~60%."
    *What changed:* the migration method (waves, shared chart), plus speed and efficiency outcomes.

### Self-Review Checklist

- [ ] Every bullet starts with a strong past-tense verb; none says "responsible for", "helped", or "worked on".
- [ ] Every bullet has a scope (how many services, teams, engineers, accounts).
- [ ] Every bullet has an outcome with a baseline, or an honest qualitative outcome.
- [ ] I can explain how each number was measured.
- [ ] Technologies appear in service of outcomes, not as a list.
- [ ] At least three bullets show platform signals: self-service, adoption, or eliminating toil for other teams.
- [ ] Bullets are one or two lines each.
- [ ] Numbers match what I say in my stories and on LinkedIn.

---

## Exercise 4: Product-Sense Drill

### Goal

Three written product-sense answers built on one reusable skeleton, each deliverable out loud in four to six minutes, and the habit of clarifying and gathering evidence before proposing a solution.

### Instructions

1. Give yourself 20 minutes per prompt. Write your own answer before reading the model answer.
2. Use the skeleton in the template: clarify, users and jobs, evidence, options, prioritize, measure, rollout and risks, what you would not do.
3. Compare your answer with the model and list what you missed.
4. Deliver each answer out loud in four to six minutes, then have a partner (or an AI interviewer) ask three follow-ups.
5. Re-attempt any prompt you struggled with within 48 hours. Study Guide section 5 covers the underlying tools (JTBD, journey maps, RICE, success metrics).

### Template

```text
PROMPT:
1. CLARIFY: 2-3 questions I'd ask; the assumptions I'll state if there's no answer
2. USERS AND JOBS: segments; "When I ..., I want to ..., so I can ..."
3. EVIDENCE: how I'd find out (interviews, tickets, telemetry, survey, friction log)
4. OPTIONS: 2-3, including a lighter-weight option and "buy" where relevant
5. PRIORITIZE: criteria; what goes first and why; the separate must-do lane
6. MEASURE: baseline; one outcome, one adoption, one satisfaction, one reliability metric
7. ROLLOUT AND RISKS: pilot, design partners, migration path, rollback, long tail
8. WHAT I WOULD NOT DO: and why
```

### Worked Example

#### Prompt A: "Leadership wants a developer portal because 'everyone has Backstage now'. You're the senior engineer on a six-person platform team at a company with 300 engineers. How do you decide what to build in the first two quarters?"

**Clarify.** "What does leadership expect the portal to fix: onboarding speed, ownership visibility during incidents, or platform ticket load? Is there a deadline or an executive commitment behind it? How many services and teams are there, and does any catalog exist, even a spreadsheet? If I can't get answers, I'll assume the goal is faster delivery and fewer platform tickets."

**Users and jobs.** "Three segments: product engineers creating and running services, on-call engineers during incidents, and engineering managers who need ownership and compliance views. The jobs I'd test first are 'When I start a new service, I want a working repo, pipeline, and staging deploy, so I can ship this week without filing tickets' and 'When I'm paged about a dependency, I want to find its owner and runbook in under a minute.'"

**Evidence.** "Two weeks of discovery before building anything: interviews with eight to ten engineers across new and tenured staff and different kinds of teams, a categorization of the last 90 days of platform tickets, a measurement of time from new-service request to first production deploy, and a five-question survey. Discovery often shows that the portal itself isn't the bottleneck; a multi-day wait for a database or a namespace is."

**Options.** "One: full Backstage with catalog, templates, and TechDocs. Two: the catalog plus a single golden-path template. Three: automate the top ticket category first, then add a thin portal as the front door. Four: a commercial portal if we lack the frontend skills to maintain Backstage, which is a real ongoing cost for a six-person team."

**Prioritize.** "Quarter one is the thinnest valuable slice: a catalog with ownership populated automatically from repository metadata, plus one golden-path template for the most common service type, wired into our existing CI and Argo CD. Quarter two is self-service for the top ticket category discovery found, exposed through the portal. The must-do lane stays separate: if a Kubernetes upgrade is due, it gets staffed regardless."

**Measure.** "Capture baselines before launch, then report monthly: time from new-service request to first production deploy (from days to under a day), share of new services created from the template, ownership coverage in the catalog (target 95% of services with a valid owner), weekly tickets in the automated category, and one satisfaction question in the quarterly survey."

**Rollout and risks.** "Two design-partner teams build with us from week one. The biggest risk with a portal is a stale catalog, so ownership data comes from files in each repository, validated in CI, never from manual entry. The second risk is maintenance cost; I'd budget ongoing engineering time explicitly and tell leadership up front."

**What I would not do.** "Install dozens of plugins in quarter one, build custom UI before the catalog is trustworthy, or mandate portal usage. If the portal is useful, usage will show it."

#### Prompt B: "Your golden-path service template launched six months ago, and only 35% of new services use it. What do you do?"

**Clarify.** "Is that 35% of new services or of all services? Which teams use it and which don't? Is the goal 100%, or are some service types, such as data pipelines, intentionally out of scope?"

**Evidence.** "Segment the non-adopters by team, service type, and language. Interview five teams that didn't use it and two that did, and watch one engineer try to create a service with the template while narrating a friction log. The usual causes are a missing capability (no gRPC or worker variant), copy-pasting an existing repo being easier, teams not knowing it exists, or a bad early experience that spread by word of mouth."

**Options, matched to the cause.**

| Finding | Response |
| :--- | :--- |
| Missing capability for common service types | Add the top two variants; accept contributions with a review SLA |
| Copying an existing repo is easier | Make the template faster than copying: one form or one command, and a working deploy within minutes |
| Teams don't know it exists | Put it where engineers start: the portal's create page, onboarding docs, the new-repo checklist |
| A bad early experience | Fix the bug, then personally go back to the teams who hit it |
| Fear of lock-in | Document escape hatches and an upgrade path, so it isn't a one-way door |

**Prioritize.** "Fix whatever cause accounts for the most non-adopters first; that's usually a missing service type or speed. Then make the paved road the easiest road: services created from the template get dashboards, SLO alerts, autoscaling, and pre-approved production deploys without extra work, and the create page says so."

**Measure.** "Weekly template share of new services, by segment; time to first deploy for template versus non-template services; and how many manual changes teams make to the template output in the first month, as a proxy for fit. Target 70% of eligible new services within two quarters, with out-of-scope types excluded from the denominator."

**Rollout and risks.** "Ship the fixes to the interviewed teams first and ask them to try again. The risk is chasing every edge case; I'd cap the variants and keep a supported escape hatch for the rest."

**What I would not do.** "Mandate the template before fixing its gaps. That converts a product signal into a compliance problem and hides why people avoid it."

#### Prompt C: "We need to retire 'shipit', a homegrown deploy tool that 90 services depend on, within two quarters. Plan it."

**Clarify.** "Why two quarters: a security problem, the maintainer leaving, or a platform migration? What replaces it, and is the replacement ready? Who owns shipit today, and what happens to services that miss the date?"

**Evidence.** "Inventory from telemetry rather than a registry alone: shipit's logs show which services deployed in the last 90 days and which features they used. Expect three segments: services on the standard flow (most), services using rare features such as custom hooks, and dormant services that haven't deployed in months, which are decommissioning candidates."

**Options.** "Big-bang cutover (too risky with 90 services and unknown dependencies), team-by-team self-migration with docs (too slow; teams deprioritize it), or platform-led waves with generated migration pull requests. I'd choose the third."

**Plan.**

| Weeks | Milestone |
| :--- | :--- |
| 0–2 | Inventory and segmentation; confirm the replacement (Argo CD) covers the standard flow; design equivalents for rare features or decide explicitly not to support them |
| 2–4 | Announce the reason, the date, the migration guide, office hours, and the owner; block new adopters |
| 4–8 | Tooling that generates the Argo CD application and opens a pull request for each service; pilot with three volunteer teams |
| 8–18 | Waves by segment and risk; a weekly published burndown; platform engineers migrate the dormant long tail themselves |
| 18–22 | Scheduled brownouts (shipit disabled for one hour, then four) to flush out hidden dependencies; an exceptions process with director sign-off |
| 22–26 | Remove at zero usage; archive the code; delete its credentials and infrastructure; confirm with telemetry |

**Risks.** "Hidden consumers such as cron jobs or other tools calling shipit's API, which brownouts expose; teams in launch freezes, which we schedule around; and deadline creep. Every exception needs an owner, an end date, and leadership visibility."

**Measure.** "The burndown of services still on shipit, deploy success rate before and after migration, and engineering time per migration, which tells us whether the tooling is doing its job."

**What I would not do.** "Start the deadline clock before the replacement and the tooling are ready, or delete shipit while any production service still deploys through it."

### Self-Review Checklist

- [ ] I asked clarifying questions before proposing anything, and stated my assumptions when there were no answers.
- [ ] I named user segments and wrote at least one job statement.
- [ ] I said how I would gather evidence, with specific sources.
- [ ] I compared at least two options, including a lighter-weight one.
- [ ] I prioritized explicitly and kept a separate must-do lane.
- [ ] I defined success metrics with baselines, including adoption, and avoided vanity metrics.
- [ ] I described a pilot and rollout, including the long tail.
- [ ] I said what I would not do, and why.
- [ ] Out loud, each answer fits in four to six minutes.

---

## Exercise 5: 30-60-90 Day Plan for Joining a Platform Team

### Goal

A plan for your first 90 days as a senior engineer on a platform team that you can summarize in two minutes when a hiring manager asks "What would your first 90 days look like?", and that you can actually follow after you join.

### Instructions

1. Gather what you learned in interviews: the team's charter, its customers, the current pain points, and the roadmap.
2. Fill in the template for three phases: learn (days 1–30), contribute (days 31–60), and lead (days 61–90).
3. For each phase, specify what you will learn, what you will ship, which relationships you will build, and how you will know it is working.
4. Include one small, real, visible win in the first 30 days. Do not propose big changes before day 30.
5. Prepare a two-minute spoken version.
6. After you join, revise the plan with your manager in week one and use it as the agenda for your 30-, 60-, and 90-day check-ins.

### Template

| Phase | Learn | Ship | Relationships | Success signal |
| :--- | :--- | :--- | :--- | :--- |
| Days 1–30: Learn | | | | |
| Days 31–60: Contribute | | | | |
| Days 61–90: Lead | | | | |

```text
TWO-MINUTE VERSION
"In the first month I'd focus on ______, and I'd ship ______ so I learn the systems by changing them.
By day 60 I'd ______. By day 90 I'd expect to be leading ______, measured by ______.
What I'd avoid is ______."
```

### Worked Example

Context: a senior platform engineer joining a seven-person platform team that supports about 250 engineers on AWS and EKS. The team runs CI, Argo CD, the cluster fleet, and a young Backstage catalog. The interviews revealed high ticket load and a mostly reactive roadmap.

| Phase | Learn | Ship | Relationships | Success signal |
| :--- | :--- | :--- | :--- | :--- |
| **Days 1–30: Learn** | Architecture walkthroughs; the last ten postmortems and existing ADRs; one shadow on-call rotation; a developer journey map built with three product teams | One real papercut from the ticket queue fixed end to end (for example, automated namespace creation); one improved runbook; small pull requests to each core repository | 1:1s with every platform teammate; a listening tour with 8–10 tech leads from product teams; introductions to the security and finance partners | I can explain how a commit reaches production; I've shadowed on-call; my pull requests get routine reviews |
| **Days 31–60: Contribute** | A categorization of the last 90 days of tickets; baselines for time to first deploy, tickets per week, CI p50 and p90, and deploy success rate | Join the on-call rotation; one medium improvement aimed at the top ticket category; my first ADR | Agree with my manager on the problem I will own; recruit two design-partner teams for it | Baselines are published; at least two teams use the improvement; the ADR is reviewed and merged |
| **Days 61–90: Lead** | A deep dive into the area I will own, with its dependencies and risks | A one-page proposal for my next-quarter initiative with success metrics; the first milestone shipped with the design partners | Present at the platform review and an engineering-wide forum; start mentoring one teammate | The proposal is on the roadmap; the first improvement's metric is moving; product teams bring platform questions to me directly |

**Two-minute spoken version:** "In the first month I'd learn by doing: shadowing on-call, reading the last ten postmortems, and talking to eight to ten tech leads on product teams about where the platform slows them down. I'd also fix one real papercut from the ticket queue end to end, because changing a system is the fastest way to learn it. In the second month I'd join on-call, publish baselines for a few metrics such as time to first deploy and tickets per week, and ship one improvement aimed at the biggest ticket category. By day 90 I'd expect to own a roadmap initiative with two design-partner teams, a one-page proposal with success metrics, and its first milestone shipped. What I'd avoid is proposing big changes before I understand why things are the way they are; most platform decisions had reasons that were never written down."

### Self-Review Checklist

- [ ] The first 30 days emphasize learning, relationships, and one small real win, not a rewrite.
- [ ] Each phase has concrete deliverables and observable success signals.
- [ ] The plan includes conversations with product teams (the customers), not only platform-internal work.
- [ ] Baselines are captured before any improvement is claimed.
- [ ] On-call participation is part of the plan.
- [ ] The plan names relationships outside the team (security, finance, product tech leads).
- [ ] The spoken version runs about two minutes and ends with what I would avoid.

---
