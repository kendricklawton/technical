# Practice Exam 1 — Java Spring Auth-Bypass-to-RCE Chain

> One Spring Boot application. Find the authentication bypass by reading source, chain it to remote code execution, and automate the full chain in a single Python command. Snippets below are *illustrative blueprints* of the vulnerable patterns — build an equivalent to attempt hands-on.

---

## Application Blueprint

A Spring Boot REST app (`corp-portal`) with JWT auth, an admin reporting module, and an XML import feature.

### Auth layer (the bypass lives here)

```java
// JwtFilter.java  — verification trusts the token's own algorithm header
Claims parse(String token) {
    // BUG: algorithm is selected from the token, enabling RS256→HS256 confusion
    return Jwts.parser()
               .setSigningKey(publicKeyPem)   // public key reused as HMAC secret if alg=HS256
               .parseClaimsJws(token).getBody();
}
// SecurityConfig.java
http.authorizeRequests()
    .antMatchers("/api/admin/**").hasRole("ADMIN")
    .antMatchers("/api/import").authenticated();
```

### RCE sink (reachable once you are admin)

```java
// ReportController.java  (role ADMIN)
@PostMapping("/api/admin/report")
String build(@RequestBody byte[] body) {
    // BUG: deserialises untrusted bytes; Commons-Collections is on the classpath
    ObjectInputStream ois = new ObjectInputStream(new ByteArrayInputStream(body));
    Object o = ois.readObject();   // ← gadget-chain sink
    return render(o);
}
```

### Secondary sink (alternative path)

```java
// ImportController.java  (authenticated)
@PostMapping("/api/import")
void importXml(@RequestBody String xml) throws Exception {
    DocumentBuilderFactory dbf = DocumentBuilderFactory.newInstance(); // BUG: defaults allow XXE
    dbf.newDocumentBuilder().parse(new InputSource(new StringReader(xml)));
}
```

---

## Intended Chain (one viable path)

1. **Auth bypass:** the JWT filter picks the algorithm from the token. Forge an **HS256** token signed with the known **RSA public key** as the HMAC secret → become `ROLE_ADMIN` (RS256→HS256 confusion).
2. **RCE:** as admin, POST a **ysoserial CommonsCollections** payload to `/api/admin/report` (the `readObject` sink) → command execution → reverse shell.
3. **Alternative:** if the public key is unavailable, use the authenticated **XXE** at `/api/import` for file read (recover the key/secret), then return to step 1–2.
4. **Automate:** one script forges the token, generates+sends the gadget, hosts anything needed, and catches the shell.

**Pass milestones:** auth bypass (admin), then RCE, demonstrated by the exploit.

---

## 12 Scenario Questions

**Q1.** The JWT filter selects the algorithm from the token header. Which forgery works given the RSA public key?

- A. Craft an HS256 token signed with the public key as the HMAC secret.
- B. Craft an `alg:none` token, which this verifier always rejects.
- C. Brute-force the RSA private key from the public key.
- D. Replay the admin's existing RS256 token without changes.

<details><summary>Show Answer & Explanation</summary>

**A.** When the verifier trusts the token's `alg` and you switch RS256→HS256, it HMAC-verifies with the public key (known) as the secret, so you forge admin tokens. `alg:none` is not indicated to be accepted (B); RSA keys are not brute-forceable (C); replay needs a token you do not have (D).
</details>

---

**Q2.** Where in the source did the bypass originate, for the report?

- A. `JwtFilter.parse` trusting the token-supplied algorithm during verification.
- B. `SecurityConfig` mapping `/api/admin/**` to `hasRole("ADMIN")`.
- C. `ReportController` deserialising bytes.
- D. `ImportController` parsing XML.

<details><summary>Show Answer & Explanation</summary>

**A.** The algorithm-confusion flaw is in `JwtFilter.parse`. `SecurityConfig` (B) is the correct-looking authorisation rule the bypass defeats; C and D are the *RCE* sinks, not the auth bypass.
</details>

---

**Q3.** Having reached admin, which endpoint is the most direct RCE sink?

- A. `/api/admin/report`, which calls `readObject` on request bytes.
- B. `/api/import`, which only reads files via XXE.
- C. `/api/admin/report`, which only renders a string.
- D. `SecurityConfig`, which executes the gadget.

<details><summary>Show Answer & Explanation</summary>

**A.** `readObject` on untrusted bytes with Commons-Collections present is the deserialization RCE. `/api/import` gives file read (useful but not direct RCE) (B); the render is downstream of the sink (C); `SecurityConfig` executes nothing (D).
</details>

---

**Q4.** Which fact confirms the deserialization sink is exploitable?

- A. Commons-Collections (a gadget library) is on the classpath and the bytes are attacker-controlled.
- B. The endpoint returns JSON.
- C. The server runs on Linux.
- D. The method is annotated `@PostMapping`.

<details><summary>Show Answer & Explanation</summary>

**A.** Untrusted input into `readObject` plus an available gadget library makes it exploitable; you match a ysoserial chain to the library. Response type (B), OS (C), and the annotation (D) do not determine exploitability.
</details>

---

**Q5.** Your exploit should generate the gadget payload how?

- A. Invoke ysoserial (or embed the bytes) in the script and POST them to the sink.
- B. Ask the grader to run ysoserial and paste the output.
- C. Generate random bytes until deserialization succeeds.
- D. Use sqlmap to build the Java object.

<details><summary>Show Answer & Explanation</summary>

**A.** The single-command script produces the payload (subprocess or embedded) and sends it. Manual paste (B) breaks automation; randomness (C) will not deserialise; sqlmap (D) is unrelated.
</details>

---

**Q6.** If the public key is not exposed, how does the XXE help?

- A. Read a server file (config/keystore) via an external entity to recover the key or secret.
- B. Execute commands directly through the XML parser.
- C. Bypass the JWT filter without any key.
- D. Deserialise the report object remotely.

<details><summary>Show Answer & Explanation</summary>

**A.** The default `DocumentBuilderFactory` allows XXE file read; recover the key material, then perform the JWT forgery. XXE here is file read, not direct exec (B); it does not itself bypass JWT (C) or deserialise (D).
</details>

---

**Q7.** For blind XXE (no reflected output), what must your exploit stand up?

- A. An HTTP server thread hosting a malicious DTD and receiving the exfiltrated data.
- B. A MySQL server for the parser to connect to.
- C. A second Spring instance to proxy the XML.
- D. A browser to render the XML response.

<details><summary>Show Answer & Explanation</summary>

**A.** Blind XXE uses a hosted DTD with parameter entities and your listener to receive data. A DB (B), second app (C), and browser (D) are unnecessary.
</details>

---

**Q8.** Which debugger approach best confirms the chain before you finalise the exploit?

- A. Attach via JDWP and breakpoint `JwtFilter.parse` and `ReportController.build` to watch the forged token and payload.
- B. Recompile the app with print statements the grader must add.
- C. Use dnSpy to debug the JVM.
- D. Rely only on static reading; debugging is not allowed.

<details><summary>Show Answer & Explanation</summary>

**A.** JDWP breakpoints confirm the forged token is accepted and the payload reaches `readObject`. Editing the app (B) is intrusive/unnecessary; dnSpy (C) is for .NET; debugging *is* allowed and encouraged (D is false).
</details>

---

**Q9.** The RS256→HS256 forgery requires the public key in which form for the HMAC step?

- A. The exact public-key bytes the server uses to verify, used verbatim as the HMAC secret.
- B. The private key, derived from the public key.
- C. A base64 of any RSA key, since the server ignores the value.
- D. The server's TLS certificate chain.

<details><summary>Show Answer & Explanation</summary>

**A.** You must HMAC with the precise public-key representation the server verifies against (often the PEM bytes). You do not need/cannot get the private key (B); the value is not ignored (C); the TLS chain (D) is unrelated to JWT signing.
</details>

---

**Q10.** Which remediation set correctly addresses this application?

- A. Pin the JWT algorithm server-side, avoid `readObject` on untrusted input (or use allow-list filters), and disable XXE in the parser.
- B. Rotate the RSA keys weekly and keep the algorithm-from-token behaviour.
- C. Add a WAF rule blocking the word "CommonsCollections".
- D. Switch the report endpoint to GET.

<details><summary>Show Answer & Explanation</summary>

**A.** Fix each link: pin the alg (bypass), remove/guard deserialization (RCE), disable external entities (XXE). Rotating keys while trusting the token alg (B) still allows confusion; WAF keyword blocks (C) are bypassable; HTTP method (D) is irrelevant.
</details>

---

**Q11.** Your exploit works on your pre-authenticated session but fails from a clean checkout. Likely cause?

- A. The script assumes state (a prior token/cookie) instead of performing the bypass itself each run.
- B. Python is too slow to forge JWTs.
- C. ysoserial cannot run twice on the same host.
- D. The target only accepts one exploit attempt ever.

<details><summary>Show Answer & Explanation</summary>

**A.** A non-reproducible exploit usually leans on pre-existing state; it must perform the forgery and chain from scratch every run. Speed (B), ysoserial reuse (C), and a one-shot target (D) are not the issue.
</details>

---

**Q12.** For full marks, the single-command exploit must do all of the following except:

- A. Require the operator to manually paste the admin JWT at runtime.
- B. Forge the token and authenticate automatically.
- C. Generate and deliver the gadget payload automatically.
- D. Catch and let you interact with the reverse shell.

<details><summary>Show Answer & Explanation</summary>

**A.** Manual token paste violates the single-command rule — the script must forge it itself. B, C, and D are exactly what the deliverable must do automatically.
</details>

---

## Debrief

Score the two milestones and write a retro against the [README](./README.md) rubric. If the bypass took too long, drill route/auth mapping; if the exploit was flaky, rebuild on a proven scaffold.
