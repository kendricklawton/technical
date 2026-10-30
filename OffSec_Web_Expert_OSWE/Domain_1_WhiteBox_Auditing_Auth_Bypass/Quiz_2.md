# Domain 1 — Quiz 2: Type Juggling & Logic Flaws

> White-box scenarios. Answer before expanding.

---

**Q1.** Source shows `if (md5($_POST['p']) == $stored)` where `$stored` is `"0e68..."`. Why is this bypassable?

- A. Loose `==` treats two `0e`-prefixed strings as `0 == 0`, so a magic-hash input authenticates.
- B. MD5 is reversible, so the password is recovered directly from the stored hash.
- C. `$_POST` values are always arrays, so the comparison is skipped.
- D. `md5()` returns an integer that overflows the comparison.

<details><summary>Show Answer & Explanation</summary>

**A.** Loose comparison coerces `"0e..."` to scientific-notation zero; any input whose MD5 is also `0e...` compares equal. Fix: `===`/`hash_equals`.

- **B** is wrong: the attack does not reverse MD5; it exploits the comparison.
- **C** is wrong: `$_POST['p']` is a string here, not forced to an array.
- **D** is wrong: `md5()` returns a hex string, not an integer.
</details>

---

**Q2.** A check reads `if (strcmp($_POST['token'], $secret) == 0)`. Which request bypasses it, and why?

- A. `token[]=x`, because `strcmp(array, string)` returns NULL and `NULL == 0` is true.
- B. `token=0`, because any zero value equals the secret.
- C. `token=%00`, because the null byte terminates the secret.
- D. `token=true`, because booleans always satisfy `== 0`.

<details><summary>Show Answer & Explanation</summary>

**A.** Passing an array makes `strcmp` return NULL (with a warning), and `NULL == 0` is true under loose comparison, bypassing the check.

- **B** is wrong: `token=0` compares the string "0" to the secret, not equal unless the secret is "0".
- **C** is wrong: PHP strings are not null-terminated like C here; `%00` does not truncate the secret in this compare.
- **D** is wrong: `"true"` is just a string; it does not satisfy the compare.
</details>

---

**Q3.** An Express login runs `User.findOne({username: req.body.username, password: req.body.password})`. Which body logs in as admin without the password?

- A. `{"username":"admin","password":{"$ne":null}}`
- B. `{"username":"admin","password":""}`
- C. `{"username":"admin'--","password":"x"}`
- D. `{"username":["admin"],"password":["admin"]}`

<details><summary>Show Answer & Explanation</summary>

**A.** `{"$ne":null}` is a Mongo operator meaning "not equal to null" — it matches admin's real (non-null) password, bypassing the check via type/operator injection.

- **B** only works if admin's password is literally empty.
- **C** is SQLi syntax, irrelevant to a Mongo query object.
- **D** arrays may error or not match; `$ne` is the reliable operator-injection payload.
</details>

---

**Q4.** Source: `if ($user['role'] == 'admin' or $user['id'] = 1)`. What is the critical bug beyond style?

- A. `=` is assignment, not comparison, so the condition is always truthy — an unconditional bypass.
- B. `or` has the wrong precedence, so the check never runs.
- C. The single quotes make `admin` a variable, causing an error.
- D. `$user['id']` is undefined, so PHP throws a fatal error.

<details><summary>Show Answer & Explanation</summary>

**A.** `$user['id'] = 1` assigns 1 (truthy) rather than comparing, so the whole `or` is always true — everyone is treated as admin. A classic logic-flaw bypass.

- **B** misdiagnoses: precedence is not the core issue; the assignment is.
- **C** is wrong: single-quoted `'admin'` is a string literal.
- **D** is wrong: the assignment defines it; no fatal error.
</details>

---

**Q5.** A role check compares a client-supplied JWT claim `isAdmin` directly. Why is this a logic flaw even if the signature is valid?

- A. If users can obtain a token where they control `isAdmin` (e.g. self-service profile), trusting it grants privilege escalation.
- B. JWT claims cannot be read server-side, so the check never executes.
- C. Signature validity guarantees the claim is server-authored, so there is no flaw.
- D. `isAdmin` is a reserved claim that JWT libraries always reject.

<details><summary>Show Answer & Explanation</summary>

**A.** The flaw is trusting a claim whose *value* the user can influence during issuance; a valid signature only proves integrity, not that the value is authoritative. Authorisation must be derived server-side.

- **B** is wrong: claims are readable server-side.
- **C** is the misconception being tested: a valid signature does not mean the value is trustworthy.
- **D** is wrong: `isAdmin` is a custom claim, not reserved/rejected.
</details>

---

**Q6.** You see `switch ($_GET['action']) { case 0: adminReset(); }` with no type handling. Why might `action=anything` trigger `case 0`?

- A. Loose `switch` compares `"anything" == 0`, which is true for non-numeric strings in older PHP.
- B. `switch` always executes the first case when no break is present.
- C. `$_GET` values default to 0 when unset.
- D. `case 0` is a wildcard matching any string.

<details><summary>Show Answer & Explanation</summary>

**A.** `switch` uses loose comparison; in older PHP a non-numeric string `== 0` is true, so arbitrary input hits `case 0`. (PHP 8 changed this.)

- **B** is wrong: switch jumps to the matching case, not always the first.
- **C** is wrong: the value is set to "anything".
- **D** is wrong: `case 0` is not a wildcard; it is the loose compare causing the match.
</details>

---

**Q7.** Which fix set correctly hardens the comparisons in this quiz?

- A. Use strict `===`/`hash_equals`, validate/coerce input types server-side, and derive authorisation server-side.
- B. Switch all hashes from MD5 to SHA-1 and keep loose comparison.
- C. Add client-side JavaScript validation of all form fields.
- D. Wrap every comparison in a try/catch to swallow type errors.

<details><summary>Show Answer & Explanation</summary>

**A.** Strict comparison, constant-time compares, server-side type validation, and server-derived roles address every flaw shown.

- **B** is wrong: SHA-1 with loose `==` is still magic-hash-vulnerable.
- **C** is wrong: client-side validation is trivially bypassed.
- **D** is wrong: swallowing errors hides the flaw without fixing the logic.
</details>

---

**Q8.** For the exam exploit script, how should you encode a magic-hash or operator-injection bypass?

- A. As a deterministic request with the exact payload, parameterised by target URL via argparse.
- B. As a manual step the grader performs after running your script.
- C. As a random payload generator that tries values until one works.
- D. As a comment describing the bypass, without implementing it.

<details><summary>Show Answer & Explanation</summary>

**A.** The deliverable is a single-command script that sends the known-good payload deterministically against the supplied target.

- **B** violates the single-command requirement.
- **C** is non-deterministic and unreliable.
- **D** is not an exploit.
</details>
