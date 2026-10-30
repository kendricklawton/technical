# Domain 1 — Quiz 1: Source Review Methodology & Decompilation

> White-box scenarios. Answer before expanding.

---

**Q1.** You are handed a 40k-line Java codebase and need to find the RCE sink quickly. Which review direction is most efficient as a first pass?

- A. Sinks-to-sources: grep dangerous functions, then trace backward to user input.
- B. Sources-to-sinks: enumerate every request parameter and follow each one forward.
- C. Line-by-line from the first file alphabetically until a bug appears.
- D. Run the app and fuzz endpoints until one errors, then read that code.

<details><summary>Show Answer & Explanation</summary>

**A.** On a large codebase, grepping sinks (`readObject`, `exec`, `Statement` concat) and tracing backward quickly narrows thousands of lines to a handful of candidates.

- **B** is thorough but slow as a *first* pass on 40k lines; better once you have a candidate.
- **C** is hopelessly inefficient.
- **D** is black-box fuzzing — it ignores the source advantage OSWE is built on.
</details>

---

**Q2.** In a Spring app, which artifact most directly tells you *which routes skip authentication*?

- A. The security filter chain / `WebSecurityConfigurerAdapter` configuration.
- B. The `pom.xml` dependency list.
- C. The log4j configuration file.
- D. The database migration scripts.

<details><summary>Show Answer & Explanation</summary>

**A.** Spring Security's filter-chain config (`http.authorizeRequests()`/`SecurityFilterChain`) declares which paths are permitted anonymously — the direct answer to "what skips auth?".

- **B** lists libraries (useful for known-CVE gadgets, not route auth).
- **C** is logging config.
- **D** is schema, not access control.
</details>

---

**Q3.** You decompiled a `.jar` but one class shows garbled control flow. What is the pragmatic next step?

- A. Try a different decompiler (CFR/Procyon/Fernflower) since each handles constructs differently.
- B. Conclude the class is encrypted and unreadable.
- C. Rewrite the class from scratch by guessing its behaviour.
- D. Skip all of the app's logic and fuzz blindly.

<details><summary>Show Answer & Explanation</summary>

**A.** Decompilers vary; a construct that defeats one (lambdas, switch-on-string) often decompiles cleanly in another. Cross-check.

- **B** is usually wrong: garbled output means decompiler limitations, not encryption.
- **C** wastes time and is error-prone.
- **D** abandons the white-box advantage.
</details>

---

**Q4.** Why set a breakpoint at a sink rather than trusting your read of the code?

- A. It proves reachability and reveals the exact payload shape (encoding, escaping, type) reaching the sink.
- B. It automatically patches the vulnerability for the report.
- C. It is the only way to decompile the surrounding method.
- D. It bypasses authentication so you can skip the bypass step.

<details><summary>Show Answer & Explanation</summary>

**A.** A breakpoint confirms the path executes and shows the precise value at the sink — the information that makes your exploit deterministic.

- **B** is wrong: debugging observes, it does not patch.
- **C** is wrong: decompilation is static; debugging is dynamic.
- **D** is wrong: a breakpoint does not bypass auth.
</details>

---

**Q5.** An ASP.NET Core controller action has no `[Authorize]` attribute, but the class does. What determines whether the action is protected?

- A. The class-level `[Authorize]` applies to all actions unless an action has `[AllowAnonymous]`.
- B. Only action-level attributes count; class-level attributes are ignored.
- C. Neither applies without middleware explicitly re-declaring each route.
- D. Authorisation is decided solely by the database role table at runtime.

<details><summary>Show Answer & Explanation</summary>

**A.** Class-level `[Authorize]` covers every action; an action opts out only with `[AllowAnonymous]`. So the action *is* protected here.

- **B** is wrong: class-level attributes do apply.
- **C** is wrong: the attribute model works without per-route re-declaration.
- **D** is wrong: the attribute gates access before any DB role check.
</details>

---

**Q6.** You find `node --inspect` is enabled on a dev build of the target. For your review, what is its value?

- A. Attach a debugger to set breakpoints and watch variables as requests execute.
- B. It grants shell access to the host automatically.
- C. It decompiles minified client-side JavaScript into original source.
- D. It disables authentication on all routes while active.

<details><summary>Show Answer & Explanation</summary>

**A.** `--inspect` exposes the V8 debugger so you can breakpoint server code and inspect runtime values — dynamic confirmation of a path.

- **B** overstates (though an exposed inspector on a reachable interface is itself a serious risk, it is a debugger, not an auto-shell).
- **C** is wrong: it debugs server JS, not client minification.
- **D** is wrong: it does not alter route auth.
</details>

---

**Q7.** Which grep best triages serialization and command sinks across a mixed codebase in one pass?

- A. `grep -rniE "unserialize|readObject|pickle.loads|BinaryFormatter|child_process|exec\("`
- B. `grep -rni "password"`
- C. `grep -rni "TODO"`
- D. `grep -rni "import"`

<details><summary>Show Answer & Explanation</summary>

**A.** A single alternation across the known dangerous functions surfaces deserialisation and exec sinks in every language at once.

- **B** finds credential handling (useful separately) but not sinks.
- **C** finds developer notes.
- **D** lists imports — far too broad.
</details>

---

**Q8.** After mapping the app, what should your notes let you state before you write any exploit?

- A. Which routes skip auth, how admin is decided, and where each dangerous sink lives.
- B. The exact CVSS score of every finding.
- C. The server's CPU and memory specifications.
- D. The commit history of every source file.

<details><summary>Show Answer & Explanation</summary>

**A.** The three things that drive the exploit: unauthenticated surface, the authorisation decision, and the sink inventory for RCE.

- **B** is report polish, not what drives exploitation.
- **C** is irrelevant to the chain.
- **D** is rarely necessary and not the review's goal.
</details>
