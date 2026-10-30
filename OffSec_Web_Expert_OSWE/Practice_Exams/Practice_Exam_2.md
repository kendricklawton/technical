# Practice Exam 2 — PHP & Node.js Multi-App White-Box Simulation

> Two independent applications (one PHP, one Node.js). Each needs an auth-bypass-to-RCE chain and a single-command Python exploit. Snippets are illustrative blueprints of the vulnerable patterns.

---

## Application A — PHP (`legacy-crm`)

### Auth bypass (loose comparison)

```php
// auth.php — loose compare of a token hash
$calc = md5($_POST['api_token']);
if ($calc == $row['token_hash']) {          // BUG: == with a 0e-style stored hash
    $_SESSION['role'] = $row['role'];       // admin granted on magic-hash collision
}
```

### RCE sink (phar deserialization)

```php
// avatar.php — filesystem function on a user path triggers phar metadata deserialization
$path = $_GET['avatar'];                    // attacker: phar://uploads/polyglot.jpg
if (file_exists($path)) {                   // BUG: phar:// metadata is deserialised here
    echo getimagesize($path)[0];
}
// A class on the codebase: class Cache { function __destruct(){ file_put_contents($this->f,$this->d); } }
```

---

## Application B — Node.js (`team-chat`)

### Auth bypass (operator injection)

```js
// login.js — req.body used directly in the query
const u = await db.collection('users')
  .findOne({ user: req.body.user, pass: req.body.pass });  // BUG: {"pass":{"$ne":null}}
if (u) req.session.admin = u.isAdmin;
```

### RCE sink (node-serialize)

```js
// profile.js — function-capable deserialiser on a cookie
const serialize = require('node-serialize');
const profile = serialize.unserialize(req.cookies.profile); // BUG: IIFE payload runs on load
```

---

## Intended Chains (one viable path each)

- **App A:** magic-hash input so `md5(token) == '0e...'` grants admin → upload a JPEG+PHAR polyglot → reference it via `phar://` through `file_exists` → `Cache::__destruct` writes a webshell → RCE.
- **App B:** `{"user":"admin","pass":{"$ne":null}}` logs in as admin → set a `node-serialize` cookie with a self-invoking function running `child_process.exec` → reverse shell.
- **Automate:** one script per app performing bypass → RCE → shell.

**Pass milestones:** each app contributes an auth-bypass and an RCE milestone.

---

## 12 Scenario Questions

**Q1.** Why does App A's `if ($calc == $row['token_hash'])` enable an admin bypass?

- A. Loose `==` treats two `0e`-prefixed hashes as `0 == 0`, so a magic-hash token authenticates.
- B. MD5 is reversible, so the token is recovered from the hash.
- C. `$_POST` is always an array, so the compare is skipped.
- D. Sessions cannot store roles, so everyone is admin.

<details><summary>Show Answer & Explanation</summary>

**A.** Loose comparison and a `0e...` stored hash make a magic-hash input compare equal. Not MD5 reversal (B), not an array quirk here (C), and sessions do store roles (D).
</details>

---

**Q2.** App A never calls `unserialize`. How is deserialization reached?

- A. A `phar://` path through `file_exists` auto-deserialises the uploaded polyglot's metadata.
- B. `getimagesize` internally calls `unserialize` on all images.
- C. `md5` deserialises the token.
- D. It is not reachable without an explicit `unserialize`.

<details><summary>Show Answer & Explanation</summary>

**A.** Filesystem functions on a `phar://` path deserialise PHAR metadata; the polyglot carries the POP chain. `getimagesize`/`md5` do not deserialise (B, C); phar makes it reachable (D is false).
</details>

---

**Q3.** The POP chain in App A relies on which magic method?

- A. `Cache::__destruct`, which performs a controllable file write.
- B. `Cache::__construct`, called by phar on load.
- C. `Cache::__toString`, triggered by `file_exists`.
- D. `Cache::__invoke`, run on object creation.

<details><summary>Show Answer & Explanation</summary>

**A.** `__destruct` writes `d` to `f`; craft those properties to drop a webshell. `__construct` is not invoked by deserialization (B); `file_exists` does not stringify the object (C); `__invoke` is for calling objects as functions (D).
</details>

---

**Q4.** App B's login bypass uses which payload?

- A. `{"user":"admin","pass":{"$ne":null}}`
- B. `{"user":"admin","pass":"' OR 1=1--"}`
- C. `{"user":"admin","pass":"0e123"}`
- D. `user=admin&pass[]=x`

<details><summary>Show Answer & Explanation</summary>

**A.** `{"$ne":null}` matches admin's non-null password via operator injection in Mongo. SQL syntax (B) is wrong engine; magic hash (C) is PHP-specific; the array form (D) is a PHP `strcmp` trick, not this Mongo query.
</details>

---

**Q5.** App B's node-serialize payload executes during deserialization because:

- A. The function is immediately invoked via a trailing `()` as it is reconstructed.
- B. `node-serialize` compiles all JSON to native code.
- C. The cookie is marked Secure.
- D. Express executes cookies on receipt.

<details><summary>Show Answer & Explanation</summary>

**A.** The IIFE pattern self-invokes on load, running `child_process.exec`. node-serialize does not compile all JSON (B); Secure (C) is unrelated; Express does not execute cookies (D).
</details>

---

**Q6.** Mapping both apps first, which review output matters most for App B's bypass?

- A. That `req.body` fields flow untyped into the Mongo query.
- B. The Node.js version number.
- C. The CSS framework used.
- D. The number of routes in the app.

<details><summary>Show Answer & Explanation</summary>

**A.** The untyped `req.body` reaching the query is the bypass root cause. Node version (B), CSS (C), and route count (D) are not what enables operator injection.
</details>

---

**Q7.** For App A's exploit, how do you build the phar polyglot in a single-command script?

- A. Generate the JPEG+PHAR (serialised Cache metadata) within the script, upload it, then trigger via `phar://`.
- B. Ask the operator to craft the polyglot in a GUI tool and upload manually.
- C. Use a random file and hope phar parses it.
- D. Use sqlmap to produce the phar.

<details><summary>Show Answer & Explanation</summary>

**A.** The script builds the polyglot (e.g. via `php -r` or a known template), uploads it, and references it through the sink. Manual GUI steps (B) and randomness (C) break automation; sqlmap (D) is unrelated.
</details>

---

**Q8.** Which debugger fits each app for sink confirmation?

- A. Xdebug for the PHP app; `node --inspect` for the Node app.
- B. dnSpy for both.
- C. JDWP for both.
- D. No debugger; both must be done statically.

<details><summary>Show Answer & Explanation</summary>

**A.** Xdebug (PHP) and the Node inspector (Node) let you breakpoint the sinks. dnSpy (B) is .NET; JDWP (C) is Java; debugging is encouraged (D is false).
</details>

---

**Q9.** App B rejects your cookie with a parse error. Most likely fix?

- A. Ensure the serialized string matches node-serialize's exact format and encoding before sending.
- B. Switch to a Java ysoserial payload.
- C. Send the payload as a URL query instead of a cookie.
- D. Conclude the sink is unreachable and stop.

<details><summary>Show Answer & Explanation</summary>

**A.** The payload must conform to node-serialize's format/encoding (often URL-encoded JSON with the `_$$ND_FUNC$$_` marker). A Java payload (B) is wrong runtime; the sink reads the cookie specifically (C); the sink is reachable (D).
</details>

---

**Q10.** Which remediation set addresses both apps?

- A. Strict comparison in PHP + avoid phar-triggering calls on user paths; schema-validate Node input + drop function-capable deserialisers.
- B. Base64-encode cookies and tokens everywhere.
- C. Add CAPTCHAs to both login forms.
- D. Rename the `Cache` class and the profile cookie.

<details><summary>Show Answer & Explanation</summary>

**A.** Each flaw gets its real fix: `===`/`hash_equals` and no phar triggers (PHP), typed/validated input and safe deserialisation (Node). Encoding (B), CAPTCHAs (C), and renaming (D) do not fix the root causes.
</details>

---

**Q11.** You must submit two exploit scripts. What is the shared design?

- A. Each: argparse target + LHOST/LPORT, Session, payload builder/hoster, and a shell catcher — single command to shell.
- B. One combined script that requires manual mode-switching mid-run.
- C. A single script that only prints instructions for both apps.
- D. Two scripts that depend on your browser being logged in.

<details><summary>Show Answer & Explanation</summary>

**A.** Reuse the proven scaffold per app, each a deterministic single command. Manual switching (B), instruction-printing (C), and browser-session dependence (D) all fail the deliverable.
</details>

---

**Q12.** Which practice confirms both exploits will reproduce for the grader?

- A. Run each from a clean checkout against a freshly-reset app instance.
- B. Run each once on your already-exploited instances.
- C. Submit them untested with detailed comments.
- D. Provide a video instead of the scripts.

<details><summary>Show Answer & Explanation</summary>

**A.** Clean-checkout, fresh-instance runs prove reproducibility. A single dirty run (B), untested code (C), and a video (D) do not.
</details>

---

## Debrief

Two apps, two chains, two scripts. Score four milestones total and retro against the [README](./README.md). Note which language's review slowed you down and drill it.
