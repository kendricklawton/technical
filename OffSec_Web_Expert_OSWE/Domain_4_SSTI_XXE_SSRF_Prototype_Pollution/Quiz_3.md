# Domain 4 — Quiz 3: Prototype Pollution to RCE

> White-box scenarios. Answer before expanding.

---

**Q1.** Which Node pattern is the prototype-pollution sink?

- A. A recursive deep-merge of a parsed request body that copies `__proto__` keys without guarding them.
- B. `JSON.parse(req.body)` assigned to a local variable and never merged.
- C. `Object.freeze(config)` applied to a static object.
- D. `const x = {...req.query}` spread into a new object once.

<details><summary>Show Answer & Explanation</summary>

**A.** An unguarded recursive merge copies `__proto__`/`constructor.prototype` onto the shared prototype — the sink. `JSON.parse` alone (B) produces inert data; `Object.freeze` (C) prevents mutation; a shallow spread (D) does not recurse into `__proto__`.
</details>

---

**Q2.** You send `{"__proto__":{"polluted":"yes"}}` and afterwards `({}).polluted === "yes"`. What have you proven?

- A. Prototype pollution: every object now inherits the attacker-set property.
- B. That the server executed your JSON as code.
- C. That you have achieved RCE already.
- D. That the database was modified.

<details><summary>Show Answer & Explanation</summary>

**A.** A fresh object inheriting `polluted` proves the prototype was polluted. It is not code execution (B), not yet RCE (C) — pollution is a primitive needing a gadget — and unrelated to the DB (D).
</details>

---

**Q3.** Pollution alone is not RCE. What is required to escalate it?

- A. A gadget: later code that reads the polluted property as configuration (e.g. EJS `outputFunctionName`).
- B. A second pollution request to overwrite the first.
- C. A larger JSON body exceeding the parser limit.
- D. Admin privileges on the application.

<details><summary>Show Answer & Explanation</summary>

**A.** RCE needs a gadget sink that consumes the polluted property (EJS option, `NODE_OPTIONS`, child-process `shell`). Repeated pollution (B), body size (C), and admin rights (D) are not what bridges to execution.
</details>

---

**Q4.** Which gadget turns pollution into command execution via a spawned process?

- A. Polluting `NODE_OPTIONS` (or `shell`/`env`) read by a later `child_process.spawn`/`fork`.
- B. Polluting `Array.length` to overflow a buffer.
- C. Polluting `Math.random` to predict tokens.
- D. Polluting `console.log` to leak memory.

<details><summary>Show Answer & Explanation</summary>

**A.** If a later `spawn`/`fork` reads a polluted `NODE_OPTIONS`/`shell`/`env`, you control child-process execution (e.g. `--require`/`--eval`). The others do not reach process execution.
</details>

---

**Q5.** Why is a vulnerable deep-merge the usual culprit rather than a simple assignment?

- A. A recursive merge descends into nested keys including `__proto__`, writing onto the prototype; a shallow assignment does not.
- B. Assignments always pollute the prototype; merges never do.
- C. Merges encrypt the object, hiding the pollution.
- D. Only merges run on the server; assignments run client-side.

<details><summary>Show Answer & Explanation</summary>

**A.** Recursion into `__proto__` is what reaches the prototype; a shallow copy treats `__proto__` as an own key or ignores it. B inverts reality; C and D are false.
</details>

---

**Q6.** During review, how do you confirm a prototype-pollution finding is exploitable to RCE?

- A. Verify an unguarded recursive merge of untrusted input AND locate a gadget sink that reads a polluted property.
- B. Verify the server has Node 18 or newer.
- C. Verify the request uses POST rather than GET.
- D. Verify the JSON is minified.

<details><summary>Show Answer & Explanation</summary>

**A.** Exploitability needs both the pollution sink and a reachable gadget. Node version (B), HTTP method (C), and minification (D) are irrelevant to the chain.
</details>

---

**Q7.** For the one-command exploit, how are the pollution and gadget steps ordered?

- A. Send the pollution request first, then trigger the gadget request that reads the polluted property, then catch the shell.
- B. Send both in a single request, since order does not matter.
- C. Trigger the gadget first so the pollution has somewhere to land.
- D. Require the grader to restart the server between steps.

<details><summary>Show Answer & Explanation</summary>

**A.** Pollute first (set the shared property), then hit the gadget that consumes it, then receive the shell — all in one script. Order matters (B is wrong); the gadget must run after pollution (C inverts it); no restart is needed (D).
</details>

---

**Q8.** Which remediation set correctly addresses prototype pollution?

- A. Guard merges against `__proto__`/`constructor`/`prototype`, use null-prototype objects or Map, validate schemas, and freeze prototypes.
- B. Rename `__proto__` references in the client code only.
- C. Increase the JSON body size limit to absorb payloads.
- D. Disable POST requests to the merge endpoint.

<details><summary>Show Answer & Explanation</summary>

**A.** Block the dangerous keys in merges, use `Object.create(null)`/`Map`, validate input schemas, and freeze prototypes. Client-side renaming (B) is bypassable; body limits (C) do not stop a small payload; blocking POST (D) does not address the merge flaw.
</details>
