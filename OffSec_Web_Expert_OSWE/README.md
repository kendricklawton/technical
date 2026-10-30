# OffSec Web Expert (OSWE / WEB-300) — Complete Mastery Curriculum

## Overview & Exam Blueprint

The **OffSec Web Expert (OSWE)** certifies advanced **white-box** web application security. Unlike black-box testing, you are given the application's **source code** and must read it to discover vulnerabilities, chain an **authentication bypass into remote code execution**, and prove mastery by writing a **single-command automated exploit** that performs the entire chain unattended.

### Official Exam Structure (100 Points Total; 85 Points Required to Pass)
- **Format:** a **47-hour and 45-minute** hands-on exam window, followed by **24 hours** to submit a professional report.
- **Targets:** typically **two or three** independent web applications, each with source code provided (white-box).
- **Per-application objectives (two scored milestones each):**
  - **Authentication Bypass / access as a privileged user** — reach an authenticated or admin context you should not have.
  - **Remote Code Execution** — chain from that access to code execution on the server.
- **Proof:** retrieve the required `proof.txt`/flag values AND demonstrate the vulnerability via your own exploit.
- **The automation requirement:** for full marks you must supply a **standalone exploit script** (typically Python) that runs the complete chain with a single command — no manual steps.

### What Makes OSWE Different
- **Source code is the primary attack surface.** You trace user input from **sources** (request parameters) to dangerous **sinks** (query builders, deserialisers, template engines, file writes).
- **Debuggers are allowed and expected** (`jdwp`, `dnSpy`, `xdebug`, `node --inspect`) — set breakpoints, step through, and confirm reachability.
- **Tooling is lightweight.** No Metasploit-style frameworks are needed; the deliverable is *your* clean, reliable exploit code.
- **Reliability is graded.** An exploit that works once in ten is not acceptable; your script must be deterministic.

---

## Curriculum Architecture

- [OSWE Master Cheat Sheet](./OSWE_Master_Cheat_Sheet.md) — rapid reference for code-review methodology, per-language sinks, exploit-script scaffolding, and debugging.
- **Domain 1: White-Box Source Review, Type Juggling & Cryptographic Auth Bypass**
  - [Study Guide](./Domain_1_WhiteBox_Auditing_Auth_Bypass/Study_Guide.md)
  - [Hands-On Lab Exercises](./Domain_1_WhiteBox_Auditing_Auth_Bypass/Exercises.md)
  - [Reference White Papers](./Domain_1_WhiteBox_Auditing_Auth_Bypass/White_Papers.md)
  - [Quiz 1: Source Review Methodology & Decompilation](./Domain_1_WhiteBox_Auditing_Auth_Bypass/Quiz_1.md)
  - [Quiz 2: Type Juggling & Logic Flaws](./Domain_1_WhiteBox_Auditing_Auth_Bypass/Quiz_2.md)
  - [Quiz 3: Crypto Auth Flaws (JWT, Padding Oracle, PRNG)](./Domain_1_WhiteBox_Auditing_Auth_Bypass/Quiz_3.md)
- **Domain 2: Advanced SQL Injection & Database-to-RCE**
  - [Study Guide](./Domain_2_Advanced_SQLi_Database_RCE/Study_Guide.md)
  - [Hands-On Lab Exercises](./Domain_2_Advanced_SQLi_Database_RCE/Exercises.md)
  - [Reference White Papers](./Domain_2_Advanced_SQLi_Database_RCE/White_Papers.md)
  - [Quiz 1: Prepared-Statement Auditing & Injection Discovery](./Domain_2_Advanced_SQLi_Database_RCE/Quiz_1.md)
  - [Quiz 2: Blind Extraction Automation](./Domain_2_Advanced_SQLi_Database_RCE/Quiz_2.md)
  - [Quiz 3: SQLi-to-RCE per Engine](./Domain_2_Advanced_SQLi_Database_RCE/Quiz_3.md)
- **Domain 3: Insecure Deserialization & Gadget Chains**
  - [Study Guide](./Domain_3_Insecure_Deserialization/Study_Guide.md)
  - [Hands-On Lab Exercises](./Domain_3_Insecure_Deserialization/Exercises.md)
  - [Reference White Papers](./Domain_3_Insecure_Deserialization/White_Papers.md)
  - [Quiz 1: PHP & Python Deserialization](./Domain_3_Insecure_Deserialization/Quiz_1.md)
  - [Quiz 2: Java & .NET Gadget Chains](./Domain_3_Insecure_Deserialization/Quiz_2.md)
  - [Quiz 3: Node.js Deserialization & Sink Hunting](./Domain_3_Insecure_Deserialization/Quiz_3.md)
- **Domain 4: SSTI, XXE, SSRF & Prototype Pollution**
  - [Study Guide](./Domain_4_SSTI_XXE_SSRF_Prototype_Pollution/Study_Guide.md)
  - [Hands-On Lab Exercises](./Domain_4_SSTI_XXE_SSRF_Prototype_Pollution/Exercises.md)
  - [Reference White Papers](./Domain_4_SSTI_XXE_SSRF_Prototype_Pollution/White_Papers.md)
  - [Quiz 1: Server-Side Template Injection](./Domain_4_SSTI_XXE_SSRF_Prototype_Pollution/Quiz_1.md)
  - [Quiz 2: XXE & SSRF](./Domain_4_SSTI_XXE_SSRF_Prototype_Pollution/Quiz_2.md)
  - [Quiz 3: Prototype Pollution to RCE](./Domain_4_SSTI_XXE_SSRF_Prototype_Pollution/Quiz_3.md)
- **Domain 5: XSS-to-RCE Chains & Exploit Automation**
  - [Study Guide](./Domain_5_XSS_to_RCE_Exploit_Automation/Study_Guide.md)
  - [Hands-On Lab Exercises](./Domain_5_XSS_to_RCE_Exploit_Automation/Exercises.md)
  - [Reference White Papers](./Domain_5_XSS_to_RCE_Exploit_Automation/White_Papers.md)
  - [Quiz 1: DOM XSS, CSP Bypass & Session Riding](./Domain_5_XSS_to_RCE_Exploit_Automation/Quiz_1.md)
  - [Quiz 2: CORS & Race Conditions](./Domain_5_XSS_to_RCE_Exploit_Automation/Quiz_2.md)
  - [Quiz 3: Single-Command Python Exploit Automation](./Domain_5_XSS_to_RCE_Exploit_Automation/Quiz_3.md)
- **Full-Length Mock Exam Simulations**
  - [Practice Exams Overview](./Practice_Exams/README.md)
  - [Practice Exam 1: Java Spring Auth-Bypass-to-RCE Chain](./Practice_Exams/Practice_Exam_1.md)
  - [Practice Exam 2: PHP & Node.js Multi-App White-Box Simulation](./Practice_Exams/Practice_Exam_2.md)
  - [Practice Exam 3: C# .NET ViewState & Python Deserialization Simulation](./Practice_Exams/Practice_Exam_3.md)

---

## Study Sequence

1. **Domain 1 first** — code-review methodology is the lens for everything else.
2. **Domains 2–4** — the vulnerability classes, always approached *from the source*.
3. **Domain 5** — fold every technique into reliable, single-command automation.
4. **Practice Exams** — chain auth-bypass → RCE under time pressure, and write the exploit script.

> **Golden rule of OSWE:** you are not hunting payloads, you are reading code. Find the sink, trace the source, prove reachability in a debugger, then automate.
