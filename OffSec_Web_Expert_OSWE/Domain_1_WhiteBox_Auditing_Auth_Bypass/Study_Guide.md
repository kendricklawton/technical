# Domain 1: White-Box Source Review, Type Juggling & Cryptographic Auth Bypass — Study Guide

> **Exam weight:** the *entry* milestone on every OSWE application — you must reach a privileged context before you can pursue RCE. This domain is also the methodology that governs the entire exam: read the code, trace input, prove reachability.

---

## 0. The White-Box Mindset

Black-box testing guesses at behaviour; white-box *reads* it. You have the source — your job is to trace untrusted input from where it enters (a **source**) to where it does something dangerous (a **sink**), and to decide whether any check in between actually stops you.

```text
   [ SOURCE ]                      [ SINK ]
   request param ──► controller ──► auth check ──► query/deserialise/render/exec
   (attacker)        (routing)      (bypass?)       (impact: bypass or RCE)

   Two reading directions:
     Sources → Sinks : start at entry points, follow taint forward (thorough).
     Sinks → Sources : grep dangerous functions, trace backward (fast on big code).
```

Use **sinks→sources** to triage a large codebase quickly (grep the dangerous-function list), then **sources→sinks** to confirm a specific tainted path and build the exploit.

---

## 1. Mapping the Application

Before hunting bugs, build a mental model:

- **Routing & middleware** — how requests reach handlers, and *where authentication/authorisation is enforced*. The highest-value question: **which routes skip the auth middleware?**
  - **Java Spring:** `@RestController`/`@RequestMapping`; security via `WebSecurityConfigurerAdapter`/`SecurityFilterChain`; `@PreAuthorize`. Servlets: `web.xml` `<security-constraint>`.
  - **ASP.NET Core/MVC:** `[Authorize]`/`[AllowAnonymous]` attributes; middleware order in `Startup`/`Program.cs`; routing attributes.
  - **PHP:** front controller + router; often auth checks are per-file `include` guards (easy to miss one).
  - **Node.js Express:** `app.use()` middleware order; route-level vs app-level auth.
  - **Python Flask/Django:** decorators (`@login_required`), Django middleware and `urls.py`.
- **Authorisation model** — how is a role/admin decided? Anything derived from client-controlled data (a header, a cookie field, a JSON property) is suspect.
- **Data/exec layers** — note every raw query, deserialiser, template render, file write, and `exec`. These are your RCE candidates for later domains.

---

## 2. Decompilation & Live Debugging

You can read bytecode-compiled apps and step through execution — do both.

- **Java:** decompile `.jar`/`.war` with **CFR**, **Fernflower**, **Procyon**, or **JD-GUI**. For dynamic analysis, start the target with a **JDWP** agent (`-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=*:8000`) and attach your IDE to set breakpoints.
- **.NET:** open `.dll`/`.exe` in **ILSpy** or **dnSpy**; dnSpy can also *debug* (breakpoints, edit-and-continue) a running .NET process.
- **PHP:** step-debug with **Xdebug** (set `xdebug.mode=debug`, point `client_host` at your IDE). Breakpoint the sink and inspect the exact string reaching it.
- **Node.js:** `node --inspect app.js`, then attach Chrome DevTools (`chrome://inspect`) to set breakpoints and watch variables.

**Why debugging matters on OSWE:** reading code tells you a path *might* be reachable; a breakpoint at the sink proves it *is*, and shows the precise payload shape (encoding, escaping, type) you must send — which is what makes your final exploit deterministic.

---

## 3. PHP Type Juggling

PHP's loose comparison (`==`) coerces types before comparing; this breaks authentication logic that uses `==` where it should use `===` (strict).

### 3.1 Magic hashes (`0e...`)

When two hashes are compared loosely and both look like `0e<digits>`, PHP interprets them as **0 × 10^n = 0**, so they compare *equal*:

```php
// VULNERABLE: loose compare of hashes
if (md5($_POST['pass']) == $stored_hash) { /* auth */ }
// If $stored_hash is "0e830400451993494058024219903391"
// and md5($_POST['pass']) is also "0e..." (a known "magic hash" input),
// then "0e..." == "0e..."  →  0 == 0  →  TRUE  (bypass)
```
Fix: `===` (strict) and `hash_equals()` for constant-time comparison.

### 3.2 `strcmp()` with arrays

Passing an array where `strcmp()` expects a string historically returned `NULL`, and `NULL == 0` is true:

```php
// VULNERABLE
if (strcmp($_POST['pass'], $real) == 0) { /* auth */ }
// Request pass[]=x  →  strcmp(array, string) returns NULL  →  NULL == 0  →  TRUE
```

### 3.3 `in_array()` loose mode

`in_array($needle, $haystack)` without the third `true` argument does loose comparison, so `"1abc"` can match `1`, and `0` can match a string starting non-numerically.

**Audit rule:** every authentication/authorisation comparison must be **strict**. Grep for `==` near `password`, `token`, `hash`, `strcmp`, `in_array`.

---

## 4. Node.js Type/Operator Injection

Express apps that pass `req.body`/`req.query` straight into queries or comparisons inherit the request's JSON types:

```js
// VULNERABLE (NoSQL): user controls the TYPE, not just the value
db.users.findOne({ user: req.body.user, pass: req.body.pass });
// POST {"user":"admin","pass":{"$ne":null}}  → matches admin with any password
```

- **Array/object where a string is expected** can bypass `===` checks, crash validators open, or alter query semantics.
- The root cause is trusting the *type* of client JSON. Fix: validate/coerce types server-side (schema validation), use parameterised queries, and cast to `String()` before comparison.

---

## 5. Cryptographic Authentication Flaws

### 5.1 Predictable PRNG for reset/session tokens

If a "random" password-reset token or session ID comes from a **non-cryptographic PRNG seeded predictably**, you can reproduce it:

- `mt_rand()`/`rand()` (PHP), `java.util.Random` (Java), `random.random()` (`random.seed()`, Python) are **not** cryptographically secure. If the seed is time-based or leaked, an attacker reproduces the sequence and predicts the next token.
- **Audit rule:** reset/session/CSRF tokens must use a CSPRNG (`random_bytes`, `SecureRandom`, `secrets`). Finding `mt_rand()` generating a reset token is an exploitable flaw.

### 5.2 JWT weaknesses

- **`alg: none`** — if the server accepts an unsigned token, forge any claims (set `alg` to `none`, drop the signature).
- **RS256 → HS256 key confusion** — if the server verifies with a key-by-algorithm the attacker controls, sign an HS256 token using the *public* RSA key (which is known) as the HMAC secret; a naive verifier that trusts the token's `alg` accepts it.
- **`kid` injection** — the `kid` header selects the verification key; if it is used in a SQL query or file path unsanitised, inject SQLi or path traversal to control which key verifies (e.g. point at a predictable file).

### 5.3 CBC padding oracle

If the app decrypts attacker-supplied CBC ciphertext and its response differs on *valid vs invalid padding*, that difference is an **oracle** that lets you decrypt (and forge) ciphertext byte-by-byte without the key — classically used to tamper with encrypted auth tokens/cookies.

### 5.4 Hash length-extension

With a MAC built as `hash(secret || message)` (e.g. naive `md5(secret+data)`), knowing the hash and the length of `secret` lets you append data and compute a valid MAC for the extended message — forging authenticated parameters. Fix: use HMAC, not `hash(secret||msg)`.

---

## 6. Host Header Password-Reset Poisoning

If the app builds the password-reset link from the incoming `Host` header, an attacker requests a reset for a victim while sending `Host: attacker.com`; the victim receives a link pointing at the attacker, who then captures the token:

```php
// VULNERABLE
$reset_link = "https://" . $_SERVER['HTTP_HOST'] . "/reset?token=" . $token;
mail($victim, "Reset", $reset_link);
```
Fix: build URLs from server-side config (a fixed canonical host), never from the request header.

---

## 7. From Bypass to the Rest of the Chain

Domain 1 gets you *in*. The authenticated/admin context you reach is the launchpad for:
- SQLi-to-RCE (Domain 2),
- deserialisation gadgets (Domain 3),
- SSTI/XXE/SSRF/prototype pollution (Domain 4),
- XSS-driven admin actions (Domain 5).

Always note, at the moment of bypass, *what new functionality* the privileged context unlocks — that is where the RCE sink usually lives.

---

## 8. Defensive & Reporting Takeaways

- **Comparisons:** strict typing (`===`), constant-time compares (`hash_equals`), server-side type validation for JSON.
- **Tokens:** CSPRNGs only; sign JWTs with vetted libraries, pin the algorithm, validate `kid`.
- **Crypto:** authenticated encryption (AES-GCM) to kill padding oracles; HMAC instead of `hash(secret||msg)`.
- **Reset flows:** canonical host from config; single-use, short-lived, unpredictable tokens.
- **Auth placement:** enforce authz centrally; deny-by-default routing so a forgotten route is not public.

---

## Key Audit Commands

```bash
# triage sinks fast
grep -rniE "unserialize|eval\(|system\(|exec\(|readObject|BinaryFormatter|pickle.loads|child_process" src/
# auth-logic smells
grep -rniE "==[^=]|strcmp|in_array|md5\(|mt_rand|java.util.Random|alg.*none" src/
# decompile / debug
procyon -jar app.jar -o src/ ; node --inspect app.js ; # dnSpy app.dll ; xdebug
```

➡️ **Next:** [Exercises.md](./Exercises.md), then [Quiz_1](./Quiz_1.md) (methodology/decompilation), [Quiz_2](./Quiz_2.md) (type juggling/logic), [Quiz_3](./Quiz_3.md) (crypto auth).
