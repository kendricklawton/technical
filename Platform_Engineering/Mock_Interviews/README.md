# Platform Engineering Mock Interview Loop

Nine rounds that together simulate a full senior Platform Engineer interview loop. Each mock file contains a **Candidate Brief** (safe to read beforehand), an **Interviewer Packet** (questions, hidden scenario details, answer keys, hint ladders), a **Scoring Rubric** with behavioral anchors, at least two **re-run variants**, and a **Self-Debrief Template**.

> [!IMPORTANT]
> **Don't read the Interviewer Packets in advance.** A mock is only useful once. If you've seen the answers, move to the next variant. Each file has at least three variants (A, B, C), so every round can be run three times.

---

## Round Catalog

| # | Round | Duration | Format | Primary Prep |
| :---: | :--- | :---: | :--- | :--- |
| 1 | [Technical Phone Screen](Mock_Interview_1_Phone_Screen.md) | 45 min | Rapid-fire breadth with depth probes | [Domain 2](../Domain_2_Linux_Networking_Fundamentals/Interview_Questions.md), [3](../Domain_3_Kubernetes_Platform_Engineering/Interview_Questions.md), [4](../Domain_4_Infrastructure_as_Code/Interview_Questions.md), [5](../Domain_5_CICD_GitOps_Delivery/Interview_Questions.md), [6](../Domain_6_Observability_SRE/Interview_Questions.md) |
| 2 | [Live Troubleshooting](Mock_Interview_2_Troubleshooting.md) | 60 min | Simulated incidents: you ask for commands, the interviewer returns output | [Domain 2](../Domain_2_Linux_Networking_Fundamentals/Study_Guide.md), [3](../Domain_3_Kubernetes_Platform_Engineering/Study_Guide.md), [6](../Domain_6_Observability_SRE/Study_Guide.md) |
| 3 | [Kubernetes & Platform Deep Dive](Mock_Interview_3_Kubernetes_Deep_Dive.md) | 60 min | Drill-down ladders (L1 → L4) | [Domain 3](../Domain_3_Kubernetes_Platform_Engineering/Study_Guide.md) |
| 4 | [IaC, CI/CD & GitOps Deep Dive](Mock_Interview_4_IaC_CICD_Deep_Dive.md) | 60 min | Code review of flawed artifacts + design questions | [Domain 4](../Domain_4_Infrastructure_as_Code/Study_Guide.md), [5](../Domain_5_CICD_GitOps_Delivery/Study_Guide.md) |
| 5 | [System Design: Internal Developer Platform](Mock_Interview_5_System_Design_IDP.md) | 60 min | Open-ended design with a hidden company fact sheet and curveballs | [Domain 1](../Domain_1_Platform_Foundations_IDP/Study_Guide.md), [8](../Domain_8_System_Design/Study_Guide.md) |
| 6 | [System Design: Infrastructure](Mock_Interview_6_System_Design_Infrastructure.md) | 60 min | Distributed-systems-heavy design | [Domain 8](../Domain_8_System_Design/Study_Guide.md) |
| 7 | [Coding](Mock_Interview_7_Coding.md) | 60 min | Two problems: practical data processing + systems-flavored | [Domain 9](../Domain_9_Coding_Automation/Exercises.md) |
| 8 | [Behavioral & Hiring Manager](Mock_Interview_8_Behavioral_Hiring_Manager.md) | 45–60 min | Story drill-downs + product sense + motivation | [Domain 10](../Domain_10_Behavioral_Career/Interview_Questions.md), [Domain 1](../Domain_1_Platform_Foundations_IDP/Study_Guide.md) |
| 9 | [Take-Home & Presentation](Mock_Interview_9_Take_Home_Presentation.md) | ~6 h + 45 min | Build a mini platform, then defend it | Domains [1](../Domain_1_Platform_Foundations_IDP/Study_Guide.md), [3](../Domain_3_Kubernetes_Platform_Engineering/Exercises.md), [4](../Domain_4_Infrastructure_as_Code/Exercises.md), [5](../Domain_5_CICD_GitOps_Delivery/Exercises.md), [6](../Domain_6_Observability_SRE/Exercises.md) |

---

## Three Ways to Run a Mock

### 1. With an AI Interviewer (Best for Frequency)

From the repo root in Claude Code or Gemini CLI:

```text
Run Mock Interview 2, Variant A, senior level, strict mode.
Follow Platform_Engineering/Mock_Interviews/Interviewer_Protocol.md and log the result.
```

The AI reads the interviewer packet and asks one question at a time. It simulates terminal output in troubleshooting rounds, holds back design facts until you ask clarifying questions, scores you against the rubric, and appends the debrief to the [log](Mock_Interview_Log.md). See [Interviewer_Protocol.md](Interviewer_Protocol.md) for more prompts and in-session controls (`hint`, `pause`, `time?`, `end interview`).

### 2. With a Peer (Best for Realism)

Send them the mock file and the [protocol](Interviewer_Protocol.md). They read the Interviewer Packet beforehand; you don't. Use a video call, Excalidraw for design rounds, and a shared editor for coding. Return the favor: interviewing someone else is one of the fastest ways to learn what "strong" sounds like.

### 3. Solo (Fine for Mocks 1, 3, 4, 8)

Start a timer. Read **one prompt at a time**, answer out loud (record yourself with a phone or OBS), then open the interviewer notes and self-score honestly against the rubric. Troubleshooting and system design rounds (Mocks 2, 5, 6) work much better with an interviewer who can hold back information. Use an AI or a peer for those.

---

## Scoring

Every competency is scored 1–4:

| Score | Meaning |
| :---: | :--- |
| **1** | Strong No Hire: missing fundamentals, or a serious red flag |
| **2** | No Hire: partially correct, needs heavy prompting, shallow under probing |
| **3** | Hire: correct, structured, solid depth, handles probes |
| **4** | Strong Hire: goes beyond the question, names trade-offs unprompted, teaches the interviewer something |

**Overall decision:** any competency at 1 → **No Hire**. No 1s but an average below 3.0 → **No Hire**. Average ≥ 3.0 with no 1s → **Hire**. Average ≥ 3.5 with at least two 4s → **Strong Hire**. Hints cap the score for the item they were used on (see [protocol §4](Interviewer_Protocol.md#4-scoring--debrief)).

---

## Recommended Cadence

The schedule follows the [12-week plan](../README.md#12-week-study-plan--progress-tracker):

| Week | Mock |
| :---: | :--- |
| 1 | Mock 1, Variant A, taken **cold** as a baseline |
| 4 | Mock 3, Variant A |
| 6 | Mock 4, Variant A |
| 7 | Mock 2, Variant A |
| 8 | Mock 1, Variant B (progress check against the baseline) |
| 9 | Mock 5, Variant A |
| 10 | Mock 6, Variant A · Mock 7, Variant A |
| 11 | Mock 8, Variant A · build the Mock 9 take-home |
| 12 | Full-loop simulation over two "onsite days" using Variants B/C, plus the Mock 9 presentation |

**Rules of thumb:**
- Fix your top 3 gaps before the next mock. A mock without follow-up study is just practice at being nervous.
- Wait at least two weeks before re-running a round, and always use a new variant.
- Mix modes: AI for volume, a peer at least two or three times for realism.

---

## Variant Rotation Tracker

| Mock | Variant A | Variant B | Variant C |
| :--- | :---: | :---: | :---: |
| 1. Phone Screen | [ ] | [ ] | [ ] |
| 2. Troubleshooting | [ ] | [ ] | [ ] |
| 3. Kubernetes Deep Dive | [ ] | [ ] | [ ] |
| 4. IaC / CI/CD Deep Dive | [ ] | [ ] | [ ] |
| 5. System Design: IDP | [ ] | [ ] | [ ] |
| 6. System Design: Infrastructure | [ ] | [ ] | [ ] |
| 7. Coding | [ ] | [ ] | [ ] |
| 8. Behavioral & Hiring Manager | [ ] | [ ] | [ ] |
| 9. Take-Home & Presentation | [ ] | [ ] | [ ] |

---

## Files in This Folder

- [Interviewer_Protocol.md](Interviewer_Protocol.md): rules for AI and human interviewers, scoring, level calibration, ad-hoc drills
- [Mock_Interview_Log.md](Mock_Interview_Log.md): scoreboard, readiness gates, detailed debriefs
- `Mock_Interview_1`–`9_*.md`: the nine rounds
