# Interviewer Protocol — Platform Engineering Mock Interviews

Rules for whoever plays the interviewer: an AI assistant (Claude, Gemini) or a human peer. **The candidate only needs Section 1.** Sections 2–6 are for the interviewer.

---

## 1. For the Candidate: Starting a Mock

Run these from the repo root (`~/repos/learning`) in Claude Code or Gemini CLI.

| Goal | Prompt |
| :--- | :--- |
| Standard mock | `Run Mock Interview 3, Variant A, senior level, strict mode. Follow Platform_Engineering/Mock_Interviews/Interviewer_Protocol.md.` |
| Let the log decide | `Read my Mock_Interview_Log.md and run the mock I should do next, following the Interviewer Protocol.` |
| Troubleshooting | `Run Mock Interview 2, Variant B. Simulate the terminal; I'll ask for commands.` |
| Weak-area drill | `Lightning round: 10 Tier 2 questions from Domain 3's Interview_Questions.md, one at a time. Score each 1-4. No model answers until the end.` |
| Design sprint | `Run Domain 8 Design Drill 4 as a live 45-minute interview.` |
| Practice mode | `Run Mock Interview 5 in practice mode: hints allowed, coach me after each segment.` |

**In-session controls** (type these at any time):

| Command | Effect |
| :--- | :--- |
| `pause` / `resume` | Stop and restart the clock |
| `hint` | Get the next hint on the current item's ladder. Hints are recorded and cap the score for that item. |
| `skip` | Move on; the item is scored as not answered |
| `time?` | Get elapsed and remaining time |
| `out of character: <question>` | Ask a meta question without ending the mock |
| `end interview` | Stop and go straight to scoring |

> [!TIP]
> **Answer the way you would speak.** Real rounds are spoken. Voice dictation gets you closest; if you type, write what you would *say* (1–3 minutes of speech), not an essay. Thinking out loud in text, e.g. "First I'd check X because...", earns credit just as it does in a live interview.

---

## 2. Interviewer Setup (Before the First Question)

1. **Confirm parameters in one message.** If the candidate just says "go", use the defaults.
   - Mock number (1–9) and variant (A/B/C). *Default: the lowest variant not yet recorded for that mock in [Mock_Interview_Log.md](Mock_Interview_Log.md).*
   - Target level: mid / senior / staff. *Default: senior.*
   - Mode: **strict** (real-interview conditions: hints only through the ladder, no feedback until the end) or **practice** (hints on request, brief coaching after each segment). *Default: strict.*
2. **Prepare silently.** Read the entire mock file, including the Interviewer Packet. Read the log and note earlier gaps so you can probe them harder. Optionally skim the linked domain `Interview_Questions.md` files for extra probes.
3. **Protect the packet.** Never show or paraphrase questions before you ask them, and never reveal hidden fact sheets, evidence banks, root causes, model answers or rubric anchors until the debrief.
4. **Start the clock.** If you can run shell commands, record the start time with `date +%s` and check elapsed time at each segment transition. Otherwise, ask the candidate to start a timer and report it with `time?` checks.
5. **Open like a real interviewer.** Use a short persona ("I'm Sam, a staff engineer on the platform team"), describe the format and length, ask the candidate to think out loud, then ask the first question.

---

## 3. Conduct Rules (All Rounds)

- **One prompt at a time.** Never batch questions. Wait for the answer.
- **Stay neutral in strict mode.** Use "Okay.", "Got it.", "Let's go a level deeper on that." Never say "Great answer!" or "That's wrong" mid-interview.
- **Find the ceiling, then move on.** Probe with the mock's *Probe deeper with* list until the candidate's depth limit is clear. The goal is calibration, not stumping them indefinitely.
- **Universal probes** (use when the packet runs out):
  - Why that choice over the obvious alternative?
  - What breaks first at 10x?
  - How would you know it's broken — what signal, alert or dashboard?
  - What's the blast radius, and how do you contain it?
  - How does a developer experience this day to day?
  - How do you roll it out, and how do you roll it back?
  - What did *you* personally do? What was the measurable result?
- **Hints follow the ladder.** Give hints only from the item's hint ladder, and only when the candidate asks, says they're stuck, or has clearly stalled (about 2 minutes live, or two non-progressing replies in text). Record every hint.
- **Keep the timeline.** If a segment overruns by more than 3 minutes, move on politely: "In the interest of time, let's switch gears."
- **Stay in character** until `end interview` or the timeline ends. Answer `out of character:` questions briefly, then resume.

### Round-Specific Rules

| Round | Rules |
| :--- | :--- |
| **1. Phone Screen** | Brisk pace and broad coverage. Expect 1–3 minute answers and one or two probes per question. Don't let any single topic consume more than about 6 minutes. |
| **2. Troubleshooting** | You are the terminal and the environment. Present the alert, then reveal evidence **only** when the candidate asks for a specific command, query, dashboard or fact. Render output exactly as the real tool prints it (headers, columns, event reasons, timestamps). Stay consistent with the hidden root cause and its red herrings. If asked for something not in the evidence bank, improvise output consistent with the root cause. Never volunteer or confirm the root cause; let the evidence do that. At least once per scenario, ask: *"What's your mitigation while you keep investigating?"* and *"What would you tell stakeholders right now?"* End a scenario when the candidate gives root cause, mitigation and prevention, or when the time box runs out. |
| **3–4. Deep Dives** | Use the drill-down ladders and record the deepest level reached per topic. For code-review artifacts, show the artifact verbatim and let the candidate list issues. For missed issues, ask about *areas* ("anything about how the provider is configured?"), never the issue itself. |
| **5–6. System Design** | Reveal fact-sheet items only when the candidate asks a relevant clarifying question. If they skip clarification and start designing, let them, and score it. Inject curveballs at the times the packet specifies. Ask the candidate to describe diagrams as components and arrows in text, or to draw in Excalidraw or a file and walk you through it. |
| **7. Coding** | Ask for the approach before any code. The candidate writes and runs code in a scratch directory (suggest `/tmp/mock_coding/`), and you may read their files. Never fix their bugs or write code for them. Ask for tests, edge cases and complexity. Use the follow-ups if they finish early. |
| **8. Behavioral** | Push for specifics: what did *you* do, what exactly did you say, what was the number before and after, who disagreed, what would you do differently. Note overuse of "we", vague outcomes, and stories with no conflict or stakes. |
| **9. Take-Home Presentation** | The candidate points you to their repo or pastes the README; read it before the presentation. Let them demo uninterrupted for the first 10–15 minutes, then work through the probing Q&A. Challenge at least two design decisions. |

---

## 4. Scoring & Debrief

After the interview ends:

1. **Score each competency** in the mock's *Scoring Rubric* on a 1–4 scale, and cite evidence for every score (a short quote or a paraphrase of what the candidate said or did).
2. **Apply hint caps** per item: one hint caps that item at Hire-level evidence (3); two or more hints cap it at No Hire-level evidence (2). An item answered only after the full answer was revealed counts as 1.
3. **Decide** using the overall decision guide:

   | Condition | Decision |
   | :--- | :--- |
   | Any competency scored 1 | **No Hire** |
   | No 1s, average below 3.0 | **No Hire** (borderline) |
   | No 1s, average ≥ 3.0 | **Hire** |
   | Average ≥ 3.5 with at least two 4s | **Strong Hire** |

4. **Debrief in this order:**
   1. Decision plus a per-competency score table.
   2. Top 3 strengths, with evidence.
   3. Top 3 gaps. Make them specific ("you never mentioned connection draining, so the load balancer kept routing to terminating pods"), not generic ("add more detail").
   4. **Model-answer deltas:** for the 2–3 weakest answers, what a Strong Hire answer would have included. You may now draw on the interviewer notes and the domain model answers.
   5. **Study prescription:** exact files and sections to review, for example "[Domain 3 Study Guide, §6 Autoscaling](../Domain_3_Kubernetes_Platform_Engineering/Study_Guide.md)", "redo [Domain 6 Quiz 2](../Domain_6_Observability_SRE/Quiz_2.md)", and labs to repeat.
   6. **Next mock:** which round and variant, and when (usually 5–7 days later, after the gaps are addressed).
5. **Log it.** Append a row to the Scoreboard and a detailed entry to [Mock_Interview_Log.md](Mock_Interview_Log.md), and update the Readiness Gate Tracker. If you cannot write files, output the entry so the candidate can paste it.

---

## 5. Level Calibration

When the target level is **staff**, raise the bar using the Staff column: an answer that would earn a 3 at senior earns a 2 at staff. When the target is **mid**, an answer that would earn a 2 at senior can earn a 3.

| Dimension | Mid | Senior | Staff |
| :--- | :--- | :--- | :--- |
| **Scope** | Owns components and services within a team | Owns a platform capability used by many teams | Owns platform direction across an org; shapes multi-quarter strategy |
| **Technical depth** | Uses tools correctly; knows the common failure modes | Explains internals and failure modes; anticipates second-order effects | Sets standards; spots systemic risks others miss; knows where the abstractions leak |
| **Design** | Produces a workable design with prompting | Drives the design; weighs trade-offs unprompted; covers failure modes and migration | Reframes the problem, challenges requirements, sequences the evolution, and accounts for team structure and cost |
| **Troubleshooting** | Follows a checklist to the cause | Hypothesis-driven and mitigates first; names the systemic fix | Also fixes the process or org gap that let it happen; improves detection fleet-wide |
| **Communication** | Clear and correct | Structured and concise; adapts to the audience | Aligns stakeholders; can turn the discussion into a strategy doc |
| **Behavioral impact** | Within the team | Across teams, through influence without authority | Across the org; multiplies other engineers |

---

## 6. Ad-Hoc Drills (Shorter Than a Full Mock)

| Drill | Length | How It Runs |
| :--- | :---: | :--- |
| **Lightning round** | 15 min | 8–10 questions from one domain's `Interview_Questions.md`, one at a time. Score each 1–4 and reveal model answers at the end. |
| **Explain it back** | 10 min | The candidate explains a concept to a "new hire." The interviewer asks naive but sharp questions ("but why can't we just...?"). |
| **Incident sprint** | 20 min | One troubleshooting scenario from Mock 2's variants, run under the full simulation rules. |
| **Design sprint** | 25 min | First half of a design round only: clarification, requirements, estimates and the high-level design. |
| **Story drill** | 15 min | Three behavioral questions with full drill-down probing, scored on the Mock 8 rubric. |
| **Quiz blitz** | 10 min | Ten random questions across the domain quizzes. The interviewer reads each question; the candidate answers before the reveal. |
