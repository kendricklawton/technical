# OSWE Practice Exams — Full-Length White-Box Simulations

> Three simulations that reproduce the OSWE's defining loop: **read source → find an auth bypass → chain it to RCE → write one script that automates the whole thing.** Each exam provides vulnerable-code *blueprints* (illustrative multi-file snippets) and scenario questions that rehearse the decisions you make under a 48-hour clock.

---

## How to Use These

Each exam describes two or three white-box applications with the *shape* of their vulnerabilities shown as short, representative code snippets. Build an equivalent to practise against, or study the snippets as a pure code-review drill. Equivalent targets:

- **OffSec's own OSWE lab machines** (the closest analogue) if you are enrolled.
- **Deliberately-vulnerable apps** you deploy: write small Spring/Express/Flask/PHP/.NET apps that embody each pattern, or adapt open vulnerable projects.
- **PortSwigger Web Security Academy** labs for individual techniques.

For each app: find the bypass by reading, chain to RCE, and write a single-command Python exploit — then run it from a clean checkout against a fresh instance.

---

## Exam Rules to Rehearse (match the real thing)

- **Scoring:** 100 points, **85 to pass**. Each application contributes an **auth-bypass** milestone and an **RCE** milestone.
- **Time:** **47h45m** to exploit + **24h** to report. White-box: source is provided. Debuggers are allowed and encouraged.
- **The script matters:** full marks require a **working, deterministic, single-command** exploit per objective. A manual walkthrough without automation loses points.
- **Proof:** retrieve the required proof values AND demonstrate the vulnerability via your script.

---

## A Sane 48-Hour Strategy (rehearse it)

```text
   Hour 0-2     Read ALL source. Map routes/auth/sinks for every app. No exploiting yet.
   Hour 2-10    App #1: find the auth bypass, then the RCE sink; confirm in a debugger.
   Hour 10-16   App #1: write + harden the single-command exploit. Bank it.
   Hour 16-20   Sleep. Seriously.
   Hour 20-34   App #2 (and #3): repeat find → chain → automate.
   Hour 34-44   Fill gaps; make every exploit deterministic from a clean checkout.
   Hour 44-48   STOP. Verify proofs, save every artifact, confirm scripts reproduce.
   +24h         Write the report: annotated code, the chain, and the exploit listing.
```

**The golden rule:** read before you type. The fastest OSWE candidates spend the first two hours purely mapping source; the bypass usually reveals itself in the route/auth map.

---

## Self-Assessment Rubric

- Did I find the bypass by **reading source**, not black-box fuzzing?
- Did I **confirm the sink in a debugger** before building the exploit?
- Is each exploit **single-command and deterministic** from a clean checkout?
- Can I **reproduce and explain** every link of the chain in the report?
- Did I **annotate the vulnerable code** (file:line) for the write-up?

---

## Index

- [Practice Exam 1 — Java Spring Auth-Bypass-to-RCE Chain](./Practice_Exam_1.md)
- [Practice Exam 2 — PHP & Node.js Multi-App White-Box Simulation](./Practice_Exam_2.md)
- [Practice Exam 3 — C# .NET ViewState & Python Deserialization Simulation](./Practice_Exam_3.md)
