# Practice Exam 3 — C# .NET ViewState & Python Deserialization Simulation

> Two applications (one ASP.NET, one Python). Each needs an auth-bypass-to-RCE chain and a single-command Python exploit. Snippets are illustrative blueprints of the vulnerable patterns.

---

## Application A — ASP.NET (`billing-web`)

### Auth bypass (predictable reset token) + info leak

```csharp
// ResetController.cs — non-CSPRNG token
var rnd = new Random(Environment.TickCount);     // BUG: predictable seed
string token = rnd.Next().ToString("x8");         // reset token an attacker can reproduce

// ConfigController.cs — path traversal reads web.config (leaks machineKey)
string p = Request.Query["file"];                 // BUG: no sanitisation
return System.IO.File.ReadAllText("/app/" + p);   // ?file=../web.config
```

### RCE sink (ViewState)

```xml
<!-- web.config — MAC enabled, but keys are leakable via the traversal above -->
<machineKey validationKey="..." decryptionKey="..." validation="SHA1" />
```
```csharp
// Page lifecycle deserialises __VIEWSTATE with ObjectStateFormatter (LosFormatter)
// With the leaked machineKey, a forged, correctly-MAC'd ViewState → RCE.
```

---

## Application B — Python Flask (`data-lab`)

### Auth bypass (host-header reset poisoning)

```python
# reset.py — reset link built from the request host
link = f"https://{request.headers['Host']}/reset?t={token}"  # BUG: attacker-controlled host
send_mail(victim_email, link)
```

### RCE sink (unsafe deserialization)

```python
# import_view.py (authenticated) — pickle on request data
import pickle, base64
obj = pickle.loads(base64.b64decode(request.form['blob']))   # BUG: __reduce__ RCE
```

---

## Intended Chains (one viable path each)

- **App A:** read `web.config` via path traversal to leak `validationKey`/`decryptionKey` → forge a malicious `__VIEWSTATE` with `ysoserial.net` (TypeConfuseDelegate) and the keys → RCE. (Alternative entry: predict the reset token to seize an account.)
- **App B:** poison the reset link via the `Host` header to take over an account → as that user, POST a malicious pickle (`__reduce__` → `os.system`) to `/import` → reverse shell.
- **Automate:** one script per app performing bypass/leak → RCE → shell.

**Pass milestones:** each app contributes an auth-bypass/leak milestone and an RCE milestone.

---

## 12 Scenario Questions

**Q1.** Why is App A's reset token predictable?

- A. `new Random(Environment.TickCount)` is a non-CSPRNG seeded by a guessable value, so the sequence is reproducible.
- B. `Random.Next()` always returns the same number.
- C. The token is stored in plaintext, which makes it predictable.
- D. `ToString("x8")` weakens an otherwise-secure value.

<details><summary>Show Answer & Explanation</summary>

**A.** `System.Random` is not cryptographically secure and a `TickCount` seed is guessable, so tokens can be reproduced. `Next()` varies with state (B); storage format (C) and the hex formatting (D) are not the predictability cause.
</details>

---

**Q2.** The ViewState is MAC-protected. What makes forging it possible here?

- A. The `machineKey` (validation/decryption keys) leaks via the path-traversal file read.
- B. ViewState MAC can always be forged without any key.
- C. The TLS private key signs ViewState.
- D. The reset token doubles as the ViewState key.

<details><summary>Show Answer & Explanation</summary>

**A.** Leaking `web.config` exposes the keys needed to compute a valid MAC for a malicious ViewState. MAC cannot be forged without the key (B); TLS keys (C) and the reset token (D) do not sign ViewState.
</details>

---

**Q3.** Which tool forges the malicious ViewState given the keys?

- A. `ysoserial.net` with the validation/decryption keys and a gadget like TypeConfuseDelegate.
- B. `ysoserial` (Java), using CommonsCollections.
- C. `pickle` with a `__reduce__` payload.
- D. `node-serialize` with an IIFE.

<details><summary>Show Answer & Explanation</summary>

**A.** `ysoserial.net` generates signed ViewState payloads from the keys. Java ysoserial (B), pickle (C), and node-serialize (D) target other runtimes.
</details>

---

**Q4.** App A's path traversal is significant because it:

- A. Leaks `web.config` secrets that unlock the ViewState RCE — a chaining enabler, not RCE itself.
- B. Directly executes commands on the server.
- C. Bypasses the ViewState MAC without keys.
- D. Is only a low-impact information disclosure with no chain value.

<details><summary>Show Answer & Explanation</summary>

**A.** The traversal leaks the keys that enable the forged-ViewState RCE — the pivotal chain link. It is not itself exec (B), does not bypass MAC without keys (C), and is far from low-impact given the chain (D).
</details>

---

**Q5.** App B's reset poisoning works because:

- A. The reset link is built from the attacker-controlled `Host` header, so the victim's link points to the attacker.
- B. Flask encrypts the Host header, exposing the token.
- C. The token is sent to the attacker by email automatically.
- D. The victim's password is included in the link.

<details><summary>Show Answer & Explanation</summary>

**A.** Building the link from `request.headers['Host']` lets the attacker set the host so the victim's reset link (and token, when clicked/fetched) goes to the attacker. Flask does not encrypt the header (B); the mail goes to the victim, not attacker (C); the password is not in the link (D).
</details>

---

**Q6.** App B's pickle sink yields RCE via:

- A. A class whose `__reduce__` returns `(os.system, ('cmd',))`, executed on `pickle.loads`.
- B. A YAML `!!python/object` tag in the blob.
- C. A node-serialize IIFE in the blob.
- D. A magic-hash collision in the blob.

<details><summary>Show Answer & Explanation</summary>

**A.** Pickle executes the callable from `__reduce__` on load. YAML tags (B) are for `yaml.load`; IIFE (C) is node-serialize; magic hash (D) is PHP comparison.
</details>

---

**Q7.** For App A, what must your exploit do before forging ViewState?

- A. Fetch `web.config` via the traversal and parse out the validation/decryption keys.
- B. Brute-force the MAC offline without keys.
- C. Disable ViewState MAC remotely.
- D. Predict the reset token to derive the keys.

<details><summary>Show Answer & Explanation</summary>

**A.** The script first leaks the keys via traversal, then uses them to forge. MAC brute force (B) is infeasible; you cannot disable MAC remotely (C); the reset token is unrelated to the keys (D).
</details>

---

**Q8.** Which debugger suits each app?

- A. dnSpy for the .NET app; a Python debugger / `pdb` for the Flask app.
- B. JDWP for both.
- C. Xdebug for both.
- D. No debugger is permitted.

<details><summary>Show Answer & Explanation</summary>

**A.** dnSpy debugs .NET; `pdb`/IDE debuggers suit Python. JDWP (B) is Java; Xdebug (C) is PHP; debugging is allowed (D is false).
</details>

---

**Q9.** App B's pickle endpoint requires authentication. How does your single-command exploit reach it?

- A. First perform the host-header reset poisoning to take over an account, then submit the pickle in that session.
- B. Call the pickle endpoint anonymously, since pickle ignores auth.
- C. Brute-force the login before sending the pickle.
- D. Use the .NET ViewState token to authenticate to Flask.

<details><summary>Show Answer & Explanation</summary>

**A.** The chain is bypass → authenticated RCE: take over an account via reset poisoning, then hit the authenticated pickle sink. The endpoint requires auth (B); brute force (C) is unnecessary given the bypass; a .NET token (D) does not authenticate to Flask.
</details>

---

**Q10.** Which remediation set addresses both apps?

- A. CSPRNG tokens + fix path traversal + protect/rotate machineKey (A); canonical reset host + never pickle untrusted data (B).
- B. Base64-encode ViewState and the pickle blob.
- C. Add CAPTCHAs to the reset flows of both apps.
- D. Rename `web.config` and the import endpoint.

<details><summary>Show Answer & Explanation</summary>

**A.** Each flaw gets its fix: secure randomness, stop traversal, protect keys (ASP.NET); build reset URLs from config and avoid `pickle` on untrusted input (Flask). Encoding (B), CAPTCHAs (C), and renaming (D) do not remediate the root causes.
</details>

---

**Q11.** Your App A exploit works only when you manually supply the leaked keys. What must change for full marks?

- A. The script must perform the traversal and extract the keys itself, then forge ViewState — no manual input.
- B. Nothing; manual key entry is acceptable.
- C. It should hardcode one set of keys for all targets.
- D. It should ask the grader to run ysoserial.net separately.

<details><summary>Show Answer & Explanation</summary>

**A.** Single-command automation means leaking the keys and forging within the script. Manual entry (B, D) and hardcoded keys (C) break reproducibility/automation.
</details>

---

**Q12.** Across both apps, which habit most ensures reproducible submissions?

- A. Each exploit performs its full chain from a clean checkout against a fresh instance and lands an interactive shell.
- B. Testing once on already-compromised instances.
- C. Submitting annotated code without running it.
- D. Relying on artifacts left over from manual exploitation.

<details><summary>Show Answer & Explanation</summary>

**A.** Clean-checkout, fresh-instance, end-to-end runs prove the exploits. A single dirty run (B), unrun code (C), and leftover-state dependence (D) all risk failing reproduction.
</details>

---

## Debrief

Two apps, two full chains, two scripts. Score four milestones and retro against the [README](./README.md). Then re-attempt whichever chain you found slowest under the clock — the real exam rewards fluency in exactly these patterns.
