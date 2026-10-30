# Domain 4 — Quiz 2: XXE & SSRF

> White-box scenarios. Answer before expanding.

---

**Q1.** A Java service parses user XML with a default `DocumentBuilderFactory`. Which single configuration would have prevented XXE?

- A. Setting `disallow-doctype-decl` (and disabling external entities) to true.
- B. Enabling schema validation against an XSD.
- C. Increasing the parser's entity-expansion limit.
- D. Switching the response Content-Type to application/json.

<details><summary>Show Answer & Explanation</summary>

**A.** Disabling DOCTYPE/external entities is the direct XXE fix. Schema validation (B) does not stop entity resolution; expansion limits (C) address billion-laughs DoS, not XXE file reads; the response type (D) is irrelevant to parsing.
</details>

---

**Q2.** You have XXE but the response reflects nothing. Which technique exfiltrates a file?

- A. An out-of-band parameter-entity DTD hosted on your server that sends the file contents to you.
- B. A `{{7*7}}` probe to confirm template evaluation.
- C. A `gopher://` payload to Redis.
- D. A `$ne` operator to bypass the parser.

<details><summary>Show Answer & Explanation</summary>

**A.** Blind XXE uses a remote DTD with parameter entities to exfiltrate data to your listener. SSTI probes (B), gopher/Redis (C), and NoSQL operators (D) are unrelated primitives.
</details>

---

**Q3.** To read a PHP source file that contains characters breaking XML, which entity payload works?

- A. `php://filter/convert.base64-encode/resource=config.php` as the SYSTEM entity target.
- B. `file:///config.php` directly, since XML tolerates any content.
- C. `expect://cat config.php` to run a command.
- D. `data://text/plain,<?php` to inline the file.

<details><summary>Show Answer & Explanation</summary>

**A.** The base64 filter wrapper encodes the source so it does not break the XML document. A raw `file://` read (B) breaks on `<`/`&`; `expect://` (C) is a different (often-unavailable) wrapper for commands, not safe source read; `data://` (D) does not read the target file.
</details>

---

**Q4.** An app fetches a user-supplied URL after checking it is not in an internal range. Which bypass exploits a parser differential?

- A. A URL like `http://expected-host@internal-host/` where validator and client disagree on the host.
- B. A URL longer than 2048 characters to overflow the validator.
- C. A URL with a valid TLS certificate for the internal host.
- D. A URL using HTTP/2 to evade inspection.

<details><summary>Show Answer & Explanation</summary>

**A.** Credentials/`@`/`#`/backslash tricks cause the validator and HTTP client to parse different hosts, so a "safe-looking" URL fetches internally. Length (B), TLS certs (C), and HTTP version (D) do not create the host differential.
</details>

---

**Q5.** Why does `gopher://` elevate an SSRF from "fetch a URL" to potential RCE?

- A. It lets the server send arbitrary bytes over TCP, so you can speak protocols like Redis or FastCGI to internal services.
- B. It decrypts internal TLS automatically.
- C. It grants the server root on the internal host.
- D. It bypasses the need for the service to be running.

<details><summary>Show Answer & Explanation</summary>

**A.** `gopher://` smuggles raw bytes, turning SSRF into a raw-TCP client against text protocols (Redis `SET`/cron, FastCGI PHP exec). It does not decrypt TLS (B), grant root (C), or work against a down service (D).
</details>

---

**Q6.** DNS rebinding defeats which specific defence, and how?

- A. TOCTOU IP validation: the hostname resolves safe at check time and internal at fetch time.
- B. TLS certificate pinning, by forging a certificate.
- C. An allow-list of exact URLs, by appending query parameters.
- D. Output encoding, by changing the charset.

<details><summary>Show Answer & Explanation</summary>

**A.** Rebinding exploits the gap between validation and fetch by flipping the DNS answer to an internal IP after the check. It is not about TLS pinning (B), URL allow-lists (C), or encoding (D).
</details>

---

**Q7.** For a blind-XXE exploit script, what infrastructure must the script stand up itself?

- A. An HTTP server thread to serve the malicious DTD and receive the exfiltrated data.
- B. A full DNS server to resolve the target's domain.
- C. A database to store the parsed XML.
- D. A browser instance to render the response.

<details><summary>Show Answer & Explanation</summary>

**A.** The script hosts the DTD and catches the exfiltration on its own HTTP listener thread — self-contained automation. A DNS server (B), database (C), and browser (D) are unnecessary for OOB XXE.
</details>

---

**Q8.** Which remediation set correctly addresses the XXE and SSRF findings?

- A. Disable DOCTYPE/external entities in parsers; for SSRF, validate by resolved IP at fetch time, block internal ranges, and disable gopher/file schemes.
- B. Rate-limit XML uploads and add a CAPTCHA to the fetch endpoint.
- C. Base64-encode all XML and URLs before processing them.
- D. Move the XML parser and URL fetcher to separate servers.

<details><summary>Show Answer & Explanation</summary>

**A.** Disable external entities (XXE) and fix SSRF with fetch-time IP validation, internal-range blocking, and scheme restrictions. Rate limits/CAPTCHA (B) do not stop a single malicious request; encoding (C) does not disable entity resolution; separating servers (D) does not remove the primitives.
</details>
