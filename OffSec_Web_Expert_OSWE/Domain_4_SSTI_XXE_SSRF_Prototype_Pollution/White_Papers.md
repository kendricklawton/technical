# Domain 4 — Reference White Papers & Deep Dives

> Mechanism-level background for SSTI, XXE, SSRF, and prototype pollution. Each is a *context confusion*: data crosses into code, markup, request, or object-property context.

---

## 1. SSTI: Data Crossing into Template Code

- Template engines compile expressions at render time. When user input is concatenated into the template *source*, the engine compiles it — so input becomes code. Bound variables are safe because they are data in a pre-compiled template.
- **Jinja2 escape** is Python object-graph navigation: `__class__`→`__mro__`→`__subclasses__()` reaches a class whose `__init__.__globals__`/constructor exposes `os`, giving `popen`/`system`. The "sandbox" only restricts attribute access patterns, which the MRO walk circumvents.

**Sources:** James Kettle, "Server-Side Template Injection" (PortSwigger/Black Hat); engine docs (Jinja2 sandbox, Freemarker `Execute`).

---

## 2. XXE: The XML Entity Mechanism

- XML DTDs can declare entities, including **external** ones that the parser fetches (`SYSTEM`). Parameter entities (`%`) allow building entities dynamically, which enables blind out-of-band exfiltration via a remote DTD.
- The vulnerability is entirely about parser configuration: resolving external entities (and DTDs) on untrusted input. Secure defaults disable DOCTYPE/external entities.

**Sources:** OWASP XXE Prevention Cheat Sheet; W3C XML spec (entities); language parser docs (DocumentBuilderFactory, libxml, XmlReaderSettings, lxml).

---

## 3. SSRF: Trust in Outbound Requests and URL Parsing

- SSRF exploits that the *server* is trusted on the internal network. Defences fail on **parser differentials** (validator vs HTTP client disagree on the host) and **TOCTOU** (DNS resolves differently at check vs fetch — rebinding).
- **gopher://** is pivotal: it sends arbitrary bytes, converting an HTTP-only SSRF into a raw-TCP client against text protocols (Redis, FastCGI, SMTP, Postgres), which is how SSRF becomes RCE.

**Sources:** OWASP SSRF Prevention Cheat Sheet; Orange Tsai, "A New Era of SSRF" (URL parser differentials); PortSwigger SSRF labs; "SSRF to RCE via gopher" write-ups.

---

## 4. Prototype Pollution: Polluting the Object Graph

- JavaScript objects inherit from `Object.prototype`. A merge/clone that copies a `__proto__` or `constructor.prototype` key writes onto that shared prototype, so *all* objects gain the property. Pollution is a *primitive*; RCE requires a **gadget** — later code that reads the polluted property as configuration (EJS `outputFunctionName`, child-process `shell`/`NODE_OPTIONS`).

**Sources:** Olivier Arteau, "Prototype pollution attacks in NodeJS" (original research); PortSwigger prototype-pollution labs; Node.js security docs.

---

## 5. Why White-Box Finds These Reliably

- Each class has a crisp source signature: a template built by concatenation, a parser with entities enabled, a server-side fetch of a user URL, and an unguarded deep-merge. Reading the code turns "does this exist?" into a grep plus a reachability check — far faster and more certain than black-box probing.

**Sources:** OWASP Code Review Guide; the per-class cheat sheets above.

---

## 6. Primary Reference Index

- **PortSwigger Web Security Academy** — SSTI, XXE, SSRF, prototype-pollution labs (free, hands-on).
- **OWASP Cheat Sheets** — XXE Prevention, SSRF Prevention.
- **Orange Tsai** SSRF research; **James Kettle** SSTI research; **Olivier Arteau** prototype-pollution paper.
- **PayloadsAllTheThings** — SSTI/XXE/SSRF/Prototype-Pollution payload references (understand before use).

> **Study tip:** for each class, memorise the *source signature* and the *fingerprint probe*. On the exam you confirm the class in seconds, then spend your time on the engine/gadget-specific escalation.
