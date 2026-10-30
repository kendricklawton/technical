# OSCP Practice Exams — Full-Length Mock Simulations

> Three full-length simulations that reproduce the **structure, time pressure, and decision-making** of the real OSCP, so you train the exam *meta-skills* (triage, time-boxing, note discipline, knowing when to walk away from a box) rather than just individual techniques.

---

## How to Use These

Each practice exam defines a **target architecture** — one 3-machine Active Directory set (40 points) plus three standalone machines (20 points each) — and 20 exam-style scenario questions. Because you must practise on machines you are authorised to use, build or source equivalents from:

- **OffSec Proving Grounds Practice** (closest analogue; filter for OSCP-like boxes).
- **HackTheBox** (retired boxes + the "OSCP-like" community lists; Pro Labs for AD).
- **Self-built labs** — GOAD for the AD set; VulnHub boxes for standalones.

Map the blueprint in each exam to equivalent boxes, then attempt the set under exam conditions.

---

## Exam Rules to Rehearse (match the real thing)

- **Scoring:** 70/100 to pass. AD set = 40 (with partial credit for foothold/lateral steps); each standalone = 10 (`local.txt`) + 10 (`proof.txt`).
- **Time:** 23h45m hacking + 24h to submit the report. Practise *both* halves — the report is where unprepared candidates fail.
- **Tool restrictions:** no `sqlmap`/autopwn/automated vuln scanners; Metasploit `exploit/post/meterpreter` on **one** machine only; `msfvenom` and `multi/handler` unlimited. Rehearse within these limits so exam day holds no surprises.
- **Proof hygiene:** screenshot each `local.txt`/`proof.txt` with `ipconfig`/`ip a` and `whoami`/`id` in frame. Lost proof = lost points.

---

## A Sane Time-Box Strategy (rehearse it)

```text
   Hour 0-1    Kick off AutoRecon on ALL targets in parallel; read everything.
   Hour 1-6    AD set: foothold → enumerate → aim for lateral movement.
   Hour 6-9    Standalone #1 (full chain), then bank the points.
   Hour 9-12   Standalone #2. Eat. Sleep is a weapon — schedule it.
   Hour 12-18  Standalone #3 + push AD set to the DC.
   Hour 18-22  Fill gaps; re-enumerate anything stuck; chase partial credit.
   Hour 22-24  STOP hacking early. Verify every screenshot and note.
   +24h        Write the report methodically from your notes.
```

**The golden rule:** if a box has consumed ~3 hours with no progress, switch. Tunnel vision is the #1 exam killer. Re-enumerate — the answer is almost always something you walked past.

---

## Self-Assessment Rubric

After each mock, score yourself honestly and write a short retro:

- Did I **enumerate fully** before exploiting (all ports, UDP, every service, every vhost)?
- Did I **time-box** and switch when stuck, or tunnel-vision?
- Can I **reproduce every step** from my notes alone?
- Is my **proof** complete (IP + identity in every screenshot)?
- Could I **write the report** right now without going back to a box?

Track these across all three mocks — the trend matters more than any single score.

---

## Index

- [Practice Exam 1 — Complete 24-Hour Enterprise AD + 3 Standalone Simulation](./Practice_Exam_1.md)
- [Practice Exam 2 — Multi-Pivot Hybrid AD Forest & Hardened Standalone Simulation](./Practice_Exam_2.md)
- [Practice Exam 3 — Web-Heavy Initial Access, ADCS Escalation & Container Escape Simulation](./Practice_Exam_3.md)
