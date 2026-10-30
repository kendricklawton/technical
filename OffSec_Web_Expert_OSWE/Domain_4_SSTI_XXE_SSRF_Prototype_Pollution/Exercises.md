# Domain 4 — Hands-On Lab Exercises

> **Lab safety & authorisation.** Deploy each vulnerable app yourself (Docker on localhost) plus any internal services (Redis, PHP-FPM) you stand up for SSRF practice on your own network. Keep exploit scripts separate from any downloaded sample. Never point SSRF/XXE/SSTI payloads at infrastructure you do not own.

---

## Exercise 4.1 — Jinja2 SSTI to RCE

**Objective:** escape the Jinja2 sandbox to command execution.

**Setup:** a Flask app that concatenates user input into `render_template_string`.

**Tasks:**
1. Confirm SSTI with `{{7*7}}`.
2. Walk `__class__`/`__mro__`/`__subclasses__` (or `__globals__`) to reach `os.popen`.
3. Upgrade to a reverse shell.

**Success criteria:** RCE via Jinja2, with the object-graph traversal explained step by step.

---

## Exercise 4.2 — Multi-Engine Fingerprinting

**Objective:** distinguish template engines by probe behaviour.

**Setup:** three small apps (Jinja2, Freemarker, a Node engine).

**Tasks:**
1. Send `{{7*7}}`, `${7*7}`, `#{7*7}` to each and record which evaluates.
2. Reach a known-good RCE payload for each identified engine.

**Success criteria:** correct engine identification for all three plus one working RCE each.

---

## Exercise 4.3 — In-Band XXE File Read

**Objective:** read a server file via external entity.

**Setup:** an app parsing user XML with external entities enabled.

**Tasks:**
1. Confirm the parser resolves a `SYSTEM` entity.
2. Read `/etc/passwd` in-band.
3. Use the PHP base64 wrapper to read a config file that would otherwise break XML.

**Success criteria:** both a plain file and a base64-wrapped source file recovered.

---

## Exercise 4.4 — Blind OOB XXE

**Objective:** exfiltrate a file when no output is reflected.

**Setup:** an app that parses XML but returns nothing useful.

**Tasks:**
1. Host a malicious DTD on your own HTTP server.
2. Use parameter entities to exfiltrate a file to your server.
3. (Stretch) Trigger an error-based variant to leak via the error message.

**Success criteria:** file contents received out-of-band at your listener.

---

## Exercise 4.5 — XXE to SSRF

**Objective:** reach an internal endpoint through XXE.

**Setup:** an XXE-vulnerable app with an internal-only HTTP service.

**Tasks:**
1. Point a `SYSTEM` entity at the internal URL.
2. Retrieve the internal response (in-band or OOB).

**Success criteria:** internal service content retrieved via the XXE entity.

---

## Exercise 4.6 — SSRF Allow-List Bypass

**Objective:** defeat a URL validator.

**Setup:** an app that fetches a user URL after an allow-list/blocklist check.

**Tasks:**
1. Read the validator and identify a parser differential (credentials, `#`, `\`, alternate encodings).
2. Craft a URL that passes validation but resolves/fetches internally.
3. Confirm the internal fetch.

**Success criteria:** an internal request made despite the validator, with the differential explained.

---

## Exercise 4.7 — SSRF via gopher to Redis

**Objective:** turn SSRF into code execution against internal Redis.

**Setup:** an internal Redis reachable only from the app host, plus an SSRF with `gopher://` support.

**Tasks:**
1. Construct a `gopher://` payload writing a cron/webshell via Redis.
2. Deliver it through the SSRF.
3. Confirm execution.

**Success criteria:** RCE via gopher-smuggled Redis commands, documented safely in your own lab.

---

## Exercise 4.8 — DNS Rebinding

**Objective:** bypass TOCTOU IP validation.

**Setup:** an SSRF that resolves and validates the hostname before fetching.

**Tasks:**
1. Set up a rebinding host that returns a safe IP then an internal IP.
2. Time the flip so validation sees safe and fetch hits internal.
3. Confirm the internal fetch.

**Success criteria:** internal access via rebinding, with the check/use gap explained.

---

## Exercise 4.9 — Prototype Pollution to RCE

**Objective:** chain pollution to code execution.

**Setup:** a Node app with a vulnerable deep-merge and an EJS (or `child_process`) gadget.

**Tasks:**
1. Confirm pollution with `{"__proto__":{"polluted":"yes"}}`.
2. Pollute the gadget property (`outputFunctionName` / `NODE_OPTIONS`).
3. Trigger the gadget sink to run code.

**Success criteria:** RCE via pollution, with both the merge sink and the gadget sink identified.

---

## Exercise 4.10 — One-Command Chain

**Objective:** automate a full Domain-4 chain.

**Setup:** any app chaining one of these to RCE.

**Tasks:**
1. Host any required DTD/payload from the script's HTTP server thread.
2. Send the pollution/SSTI/XXE/SSRF request(s) in order.
3. Catch the shell automatically.

**Success criteria:** `python3 exploit.py http://target --lhost .. --lport ..` reaches a shell unattended.

---

## Consolidation Challenge — Classify and Chain

On an unfamiliar app, identify which of the four classes is present by source review alone, then build a reliable one-command exploit. Record the sink, the input path, and the gadget/escalation used. This is a complete OSWE RCE milestone.
