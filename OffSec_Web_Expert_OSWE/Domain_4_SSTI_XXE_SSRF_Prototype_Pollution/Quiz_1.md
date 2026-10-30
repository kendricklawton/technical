# Domain 4 — Quiz 1: Server-Side Template Injection

> White-box scenarios. Answer before expanding.

---

**Q1.** Which line is the SSTI sink?

- A. `return render_template_string("Hi " + request.args['name'])`
- B. `return render_template("hi.html", name=request.args['name'])`
- C. `return render_template_string("Hi {{ name }}", name=request.args['name'])`
- D. `return "Hi " + escape(request.args['name'])`

<details><summary>Show Answer & Explanation</summary>

**A.** Concatenating user input into the template *source* makes it compile as template code. B and C pass `name` as a bound variable (data); D escapes and returns a plain string (XSS concern, not SSTI).
</details>

---

**Q2.** `{{7*7}}` returns `49` and `${7*7}` is echoed literally. Which engine family, and what does that tell you?

- A. A `{{ }}` engine (Jinja2/Twig); pursue its specific RCE escalation.
- B. A `${ }` engine (Freemarker); pursue `Execute`.
- C. A NoSQL injection point, not a template at all.
- D. A reflected XSS sink only.

<details><summary>Show Answer & Explanation</summary>

**A.** `{{7*7}}` evaluating while `${7*7}` is literal indicates the `{{ }}` family (Jinja2/Twig). It is not Freemarker (B), NoSQL (C), or mere XSS (D) — server-side arithmetic evaluated.
</details>

---

**Q3.** In Jinja2, why does `''.__class__.__mro__[1].__subclasses__()` help reach RCE?

- A. It walks to `object` and lists loaded subclasses, letting you pick one that exposes OS functionality.
- B. It decrypts the Flask secret key stored in every string.
- C. It overflows the template compiler to run shellcode.
- D. It disables autoescaping so payloads run as commands.

<details><summary>Show Answer & Explanation</summary>

**A.** `__class__`→`__mro__`→`__subclasses__()` enumerates classes; one typically exposes `os` (via constructor or `__globals__`) for `popen`/`system`. It is not key decryption (B), an overflow (C), or autoescape (D).
</details>

---

**Q4.** A Java app evaluates `${7*7}` to 49 in a Freemarker template built from user input. Which utility reaches command execution?

- A. `freemarker.template.utility.Execute`, invoked to run an OS command.
- B. `java.util.Random`, seeded to predict a token.
- C. `ObjectInputStream.readObject`, to deserialise a gadget.
- D. `DocumentBuilderFactory`, to parse an external entity.

<details><summary>Show Answer & Explanation</summary>

**A.** Freemarker's `Execute` utility runs OS commands from a template. The others are PRNG (B), deserialization (C), and XML parsing (D) — different classes entirely.
</details>

---

**Q5.** Why is identifying the SSTI sink a white-box advantage over black-box fuzzing?

- A. You can see exactly where user input is concatenated into a template and which engine compiles it.
- B. Source code automatically disables the template sandbox.
- C. Black-box testing cannot send `{{7*7}}`.
- D. Reading source changes the server's rendering behaviour.

<details><summary>Show Answer & Explanation</summary>

**A.** The source pinpoints the concatenation sink and the engine, so you skip guessing. Source does not disable sandboxes (B); black-box can send probes (C) but cannot see the sink; reading source is passive (D).
</details>

---

**Q6.** A template passes user input as a bound variable but *also* has an admin feature that stores a user-editable template string later rendered. What should you check?

- A. Whether the stored template string is attacker-controllable, making it a (possibly second-order) SSTI.
- B. Nothing; bound variables guarantee the whole app is SSTI-free.
- C. Only the client-side rendering, since server templates are safe.
- D. The database encryption of the stored template.

<details><summary>Show Answer & Explanation</summary>

**A.** A stored, user-editable template later rendered is SSTI if an attacker can influence it — trace stored values into render calls. Bound variables elsewhere do not make the app safe (B); server rendering is the risk (C); encryption (D) is irrelevant to the injection.
</details>

---

**Q7.** For the exploit script, how should the SSTI payload be delivered?

- A. Send the engine-specific RCE expression with correct encoding, parameterised by target, and catch the shell.
- B. Print the payload and have the grader paste it into the app.
- C. Randomly mutate `{{ }}` payloads until one executes.
- D. Rely on the browser to render the payload client-side.

<details><summary>Show Answer & Explanation</summary>

**A.** The single-command script sends the known-good, engine-specific payload and catches the shell. Manual paste (B) and random mutation (C) break the deliverable; SSTI is server-side, not browser-rendered (D).
</details>

---

**Q8.** Which remediation correctly addresses SSTI?

- A. Never build templates from user input; pass data as bound context and use logic-less/sandboxed templates.
- B. HTML-encode the output so the template cannot evaluate input.
- C. Deploy a WAF blocking `{{` and `}}` as the primary control.
- D. Upgrade the template engine to the latest version.

<details><summary>Show Answer & Explanation</summary>

**A.** The root cause is compiling user input as a template; bind data and avoid user-authored templates. Output encoding (B) addresses XSS after evaluation; a WAF (C) is bypassable mitigation; upgrading (D) does not stop user-controlled template compilation.
</details>
