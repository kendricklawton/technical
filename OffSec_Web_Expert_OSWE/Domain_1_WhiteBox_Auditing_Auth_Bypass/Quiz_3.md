# Domain 1 — Quiz 3: Cryptographic Auth Flaws (JWT, Padding Oracle, PRNG)

> White-box scenarios. Answer before expanding.

---

**Q1.** Source generates a password-reset token with `token = String.format("%d", new java.util.Random(System.currentTimeMillis()).nextLong())`. Why is this exploitable?

- A. `java.util.Random` is a predictable LCG seeded by current time, so the token can be reproduced.
- B. `String.format` leaks the token into logs, exposing it directly.
- C. `nextLong()` always returns the same value regardless of seed.
- D. The token is stored in plaintext, which is the only real issue.

<details><summary>Show Answer & Explanation</summary>

**A.** `java.util.Random` is a 48-bit LCG; a time-based seed is brute-forceable, letting you reproduce the exact token. Fix: `SecureRandom`.

- **B** is a secondary concern, not why the token is predictable.
- **C** is wrong: output depends on the seed.
- **D** is wrong: predictability, not storage, is the exploit here.
</details>

---

**Q2.** A JWT verifier accepts the token's `alg` header to choose the algorithm. You hold the server's RSA public key. Which attack applies?

- A. RS256-to-HS256 confusion: sign an HS256 token using the public key as the HMAC secret.
- B. Padding oracle: flip ciphertext bytes to change the payload.
- C. Length extension: append claims to the existing signature.
- D. `kid` traversal: point `kid` at `/etc/passwd` to read it.

<details><summary>Show Answer & Explanation</summary>

**A.** If the verifier trusts the token's `alg` and you switch RS256→HS256, it HMAC-verifies with the RSA *public* key (which you know) as the secret — so you forge valid tokens.

- **B**/**C** are different crypto attacks not matching a JWT alg-confusion setup.
- **D** is a separate `kid` attack, not enabled merely by holding the public key.
</details>

---

**Q3.** A JWT library accepts `{"alg":"none"}` tokens. What is the forgery, and the fix?

- A. Drop the signature and set arbitrary claims; fix by pinning allowed algorithms server-side.
- B. Brute-force the HMAC secret; fix by lengthening the secret.
- C. Replay an old admin token; fix by shortening token lifetime.
- D. Steal the private key from the JWKS endpoint; fix by rotating keys.

<details><summary>Show Answer & Explanation</summary>

**A.** `alg:none` means "unsigned"; a naive verifier accepts forged claims with no signature. Fix: never accept `none`; pin the expected algorithm.

- **B** is a different attack (weak HMAC secret), not `alg:none`.
- **C** is replay, not forgery via `none`.
- **D** is wrong: JWKS exposes public keys, not private.
</details>

---

**Q4.** An app decrypts a CBC-mode session cookie and returns HTTP 500 on bad padding but 200 on valid padding (even if the plaintext is nonsense). What have you found?

- A. A padding oracle that lets you decrypt and forge the cookie byte-by-byte without the key.
- B. A timing oracle revealing the key length only.
- C. A length-extension primitive against the cookie's MAC.
- D. A harmless error-handling difference with no security impact.

<details><summary>Show Answer & Explanation</summary>

**A.** The distinguishable valid/invalid-padding response is a padding oracle; it enables full plaintext recovery and ciphertext forgery. Fix: authenticated encryption (AES-GCM / encrypt-then-MAC).

- **B** is wrong: this oracle yields plaintext, not just key length.
- **C** is wrong: length extension targets Merkle–Damgård MACs, not CBC padding.
- **D** is wrong: the difference is directly exploitable.
</details>

---

**Q5.** A MAC is computed as `md5(secret + message)` and the server trusts `message` + MAC. Why is this forgeable, and what should replace it?

- A. MD5 is Merkle–Damgård, so length extension forges a MAC for `message||extra` without the secret; use HMAC.
- B. MD5 is slow, so the MAC can be brute-forced online; use SHA-256 with the same construction.
- C. The secret is prepended, which is always safe; no change needed.
- D. MD5 collisions let you swap the whole message freely; use base64 encoding.

<details><summary>Show Answer & Explanation</summary>

**A.** `hash(secret||msg)` with a Merkle–Damgård hash is length-extendable: knowing the hash and `len(secret)` lets you append data and compute a valid MAC. HMAC is the correct construction.

- **B** misstates the issue; the construction is flawed regardless of hash speed, and SHA-256 with the same `secret||msg` is still extendable.
- **C** is exactly the vulnerable pattern.
- **D** conflates collisions with length extension and offers a non-fix.
</details>

---

**Q6.** The reset token's PRNG is `mt_rand()` seeded once at process start. You can trigger many token generations. What is the realistic attack path?

- A. Recover the Mersenne Twister state from observed outputs, then predict subsequent tokens.
- B. Overflow `mt_rand()` to force it to return zero for all tokens.
- C. Read the seed from the database where PHP stores it.
- D. Nothing; `mt_rand()` is cryptographically secure when seeded once.

<details><summary>Show Answer & Explanation</summary>

**A.** With enough outputs, the Mersenne Twister internal state is recoverable (tools exist), allowing prediction of future `mt_rand()` values — including the next reset token.

- **B** is wrong: there is no such overflow behaviour.
- **C** is wrong: PHP does not store the seed in the DB.
- **D** is wrong: `mt_rand()` is not a CSPRNG.
</details>

---

**Q7.** For your exploit script, how do you handle a padding-oracle decryption reliably within the single-command requirement?

- A. Implement (or call) the oracle attack in the script, feeding each guess and reading the valid/invalid signal automatically.
- B. Pause and ask the operator to decrypt the cookie manually, then resume.
- C. Hardcode the decrypted value from a prior manual run.
- D. Skip decryption and brute-force the plaintext with a wordlist.

<details><summary>Show Answer & Explanation</summary>

**A.** The script must drive the oracle itself — sending guesses and parsing the server's padding signal — so one command performs the full decryption/forgery.

- **B** violates single-command automation.
- **C** is non-general and fails on a fresh cookie.
- **D** misunderstands the attack (the oracle *is* how you recover plaintext).
</details>

---

**Q8.** Which remediation set correctly addresses the crypto flaws in this quiz?

- A. CSPRNGs for tokens, pinned-algorithm JWT verification, authenticated encryption, and HMAC for MACs.
- B. Longer MD5 hashes, client-side token validation, and base64-wrapped cookies.
- C. Disable HTTPS so tokens are visible for debugging, then re-enable later.
- D. Store all secrets in the JWT payload so the server need not keep state.

<details><summary>Show Answer & Explanation</summary>

**A.** Each flaw maps to a fix: CSPRNG (predictable tokens), pinned-alg verification (JWT), AES-GCM/encrypt-then-MAC (padding oracle), HMAC (length extension).

- **B** is non-fixes (MD5 length does not help; client validation is bypassable).
- **C** is dangerous and irrelevant.
- **D** exposes secrets to the client — the opposite of secure.
</details>
