# Domain 3 — Quiz 3: Node.js Deserialization & Sink Hunting

> White-box scenarios. Answer before expanding.

---

**Q1.** An Express app calls `serialize.unserialize(req.cookies.profile)` using the `node-serialize` package. Which payload yields command execution?

- A. A value using `_$$ND_FUNC$$_function(){require('child_process').exec('id')}()` with the trailing `()` to self-invoke.
- B. A plain JSON object `{"cmd":"id"}` that node-serialize runs as a command.
- C. A pickle blob base64-encoded into the cookie.
- D. A Java serialized object delivered in the cookie.

<details><summary>Show Answer & Explanation</summary>

**A.** `node-serialize` reconstructs functions; the trailing `()` makes the function self-invoke on deserialisation, running `child_process.exec`. Plain JSON (B) is not executed; pickle (C) and Java objects (D) are the wrong runtimes.
</details>

---

**Q2.** What specifically causes the node-serialize payload to execute *during* deserialization rather than later?

- A. The immediately-invoked function expression — the trailing `()` runs it as it is reconstructed.
- B. A buffer overflow in the JSON parser.
- C. The cookie being marked HttpOnly.
- D. Express automatically executing all cookies on receipt.

<details><summary>Show Answer & Explanation</summary>

**A.** The IIFE pattern (`function(){...}()`) invokes the reconstructed function at deserialisation time. It is not an overflow (B); HttpOnly (C) is a cookie flag unrelated to execution; Express does not execute cookies (D).
</details>

---

**Q3.** Besides `node-serialize`, which other packages reconstruct functions and are therefore deserialization RCE candidates?

- A. `funcster` and `cryo`.
- B. `express` and `body-parser`.
- C. `lodash` and `moment`.
- D. `axios` and `node-fetch`.

<details><summary>Show Answer & Explanation</summary>

**A.** `funcster` and `cryo` can reconstruct functions/objects from untrusted input. The others are routing/parsing (B), utility/date (C), and HTTP client (D) libraries without that deserialization sink.
</details>

---

**Q4.** Which grep most directly surfaces Node deserialization sinks across a repo?

- A. `grep -rniE "node-serialize|unserialize\(|funcster|cryo"`
- B. `grep -rni "require"`
- C. `grep -rni "function"`
- D. `grep -rni "json"`

<details><summary>Show Answer & Explanation</summary>

**A.** Targeting the specific packages/calls pinpoints the sinks. `require` (B), `function` (C), and `json` (D) are far too broad to be useful.
</details>

---

**Q5.** You find `JSON.parse(req.body.data)` (standard library) rather than a function-capable deserialiser. Is this a deserialization RCE sink?

- A. No; `JSON.parse` builds only plain data (no functions), so it is not an RCE deserialization sink by itself.
- B. Yes; `JSON.parse` executes any function in the JSON.
- C. Yes; `JSON.parse` is identical to `node-serialize`.
- D. No; `JSON.parse` cannot parse attacker input at all.

<details><summary>Show Answer & Explanation</summary>

**A.** `JSON.parse` produces inert data structures, not functions, so it is not a deserialization-RCE sink (though parsed data can still feed *other* sinks like prototype pollution). It does not execute functions (B), is not node-serialize (C), and does parse input (D is false).
</details>

---

**Q6.** During sink hunting you must decide quickly whether a deserialization finding is exploitable. What confirms it fastest in white-box?

- A. Confirm the sink receives untrusted input and that a function/gadget-capable deserialiser or gadget class is present.
- B. Confirm the server has more than 4 GB of RAM.
- C. Confirm the app uses HTTPS end to end.
- D. Confirm the cookie is larger than 512 bytes.

<details><summary>Show Answer & Explanation</summary>

**A.** Exploitability hinges on untrusted input reaching a deserialiser capable of execution (functions/gadgets). RAM (B), HTTPS (C), and cookie size (D) are irrelevant to the primitive.
</details>

---

**Q7.** For the one-command exploit against a node-serialize sink, what must the script do end to end?

- A. Build the IIFE payload with the operator's LHOST/LPORT, deliver it in the cookie, and catch the reverse shell on a listener thread.
- B. Print the payload and ask the grader to send it manually.
- C. Start a Node process on the grader's machine to run the payload locally.
- D. Require the target to have a debugger attached first.

<details><summary>Show Answer & Explanation</summary>

**A.** The script assembles the payload (parameterised by LHOST/LPORT), sends it to the sink, and catches the shell automatically — single command, unattended. Manual send (B), local execution (C), and a required debugger (D) all break the deliverable.
</details>

---

**Q8.** Which remediation correctly addresses Node.js deserialization RCE?

- A. Do not use function-capable deserialisers on untrusted input; prefer `JSON.parse` on validated schemas.
- B. Minify the server code so attackers cannot read the sink.
- C. Set all cookies to SameSite=Strict, which blocks deserialization.
- D. Upgrade Node to the latest LTS, which removes `child_process`.

<details><summary>Show Answer & Explanation</summary>

**A.** Avoid `node-serialize`/`funcster`/`cryo` on request data; use plain JSON with schema validation. Minification (B) is obscurity; SameSite (C) addresses CSRF, not deserialization; Node does not remove `child_process` (D is false).
</details>
