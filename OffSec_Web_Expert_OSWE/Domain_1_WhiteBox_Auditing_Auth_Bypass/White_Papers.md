# Domain 1 — Reference White Papers & Deep Dives

> Mechanism-level background for white-box review and authentication bypass. The recurring theme: authentication breaks when a comparison, a token, or a trust boundary is built on an assumption the attacker controls.

---

## 1. Taint Analysis: Sources, Sinks, and Sanitisers

- A **source** is where untrusted data enters (request params, headers, body, uploaded files). A **sink** is a sensitive operation (query, deserialise, render, exec, file write). A vulnerability exists when a source reaches a sink without an effective **sanitiser**.
- Manual review mirrors what static analysers formalise; the human advantage is understanding *semantic* sanitisers (a check that looks protective but isn't — e.g. a blacklist, or an `==` that should be `===`).

**Sources:** OWASP Code Review Guide; "Static Taint Analysis" literature; Semgrep/CodeQL rule docs (to see what tools grep for).

---

## 2. PHP's Type System and Loose Comparison

- PHP `==` performs type juggling: strings that look numeric are compared as numbers, so `"0e123" == "0e456"` is `0 == 0`. `strcmp`/`in_array`/`switch` have historically surprising behaviour with non-string types.
- The design intent (developer convenience) becomes an auth bypass whenever a security decision uses loose comparison. PHP 8 tightened some of this, but legacy code — exactly what white-box exams model — is rife with it.

**Sources:** PHP manual "Comparison Operators" and "Type Juggling"; PHP 8 migration notes; the classic "PHP Magic Hashes" research.

---

## 3. Trusting Client-Controlled Types (Node.js/NoSQL)

- Express parses JSON into native types, so `req.body.x` can be an object/array, not a string. Passed into a MongoDB query, `{"$ne":null}` becomes an *operator*, not a value — changing query semantics. This is operator injection, a cousin of SQL injection at the document-DB layer.

**Sources:** OWASP NoSQL Injection; MongoDB query-operator docs; Express body-parser behaviour.

---

## 4. Randomness and Token Unpredictability

- Security tokens must be unpredictable. Non-CSPRNGs (`mt_rand`, `java.util.Random`, Python `random`) are deterministic given the seed; `java.util.Random` is a 48-bit LCG whose internal state can be recovered from a couple of outputs. Time-based seeds are brute-forceable.
- CSPRNGs (`random_bytes`, `SecureRandom`, `secrets`) are designed so observing outputs does not reveal future ones.

**Sources:** "Predicting java.util.Random" write-ups; CWE-330/CWE-338; language CSPRNG docs.

---

## 5. JWT Verification Pitfalls

- JWTs are `header.payload.signature`. The vulnerabilities stem from the verifier trusting attacker-controlled header fields: `alg:none` (no signature), algorithm confusion (RS256 public key reused as an HS256 secret), and `kid` used unsafely to locate keys.
- The fix is to pin the expected algorithm and key server-side and use a hardened library.

**Sources:** RFC 7519 (JWT), RFC 7518 (JWA); Auth0 "Critical vulnerabilities in JSON Web Token libraries"; PortSwigger JWT labs.

---

## 6. CBC Padding Oracles and Length Extension

- **Padding oracle:** CBC decryption + a distinguishable "bad padding" response lets an attacker recover plaintext and forge ciphertext (Vaudenay's attack). The fix is authenticated encryption (encrypt-then-MAC, or AES-GCM), which rejects tampered ciphertext before padding is ever checked.
- **Length extension:** Merkle–Damgård hashes (MD5/SHA-1/SHA-256) allow computing `hash(secret||msg||padding||extra)` from `hash(secret||msg)` without the secret. HMAC is immune.

**Sources:** Vaudenay (2002) padding-oracle paper; "Flickr API signature forgery" (length extension); NIST crypto guidance.

---

## 7. Host Header Trust

- The `Host` header is attacker-controlled; any security-relevant URL (password reset, absolute links) built from it is poisonable. Canonicalise from configuration.

**Sources:** PortSwigger "Host header attacks"; OWASP password-reset guidance.

---

## 8. Primary Reference Index

- **OWASP Code Review Guide** and **WSTG** (authentication testing).
- **PortSwigger Web Security Academy** — JWT, host header, authentication labs.
- **CWE** entries: 697 (incorrect comparison), 330 (insufficient randomness), 347 (improper signature verification), 294 (auth bypass by capture-replay).
- Decompiler/debugger docs: **CFR/Procyon/JD-GUI**, **dnSpy/ILSpy**, **Xdebug**, **Node Inspector**.

> **Study tip:** for every bypass you learn, write the *vulnerable code pattern* and the *one-line grep* that finds it. Your exam speed comes from recognising the pattern on sight.
