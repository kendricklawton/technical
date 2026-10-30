# Domain 3 — Quiz 1: PHP & Python Deserialization

> White-box scenarios. Answer before expanding.

---

**Q1.** Source: `$o = unserialize($_COOKIE['session']);` and a class `Logger` with `__destruct(){ file_put_contents($this->file, $this->data); }`. How do you reach a webshell?

- A. Serialize a `Logger` with `file` set to a webroot path and `data` set to PHP, then send it in the cookie.
- B. Base64-decode the cookie to read the admin password directly.
- C. Call `unserialize` twice to double-free the object into a shell.
- D. Overwrite `$_COOKIE` to disable the destructor entirely.

<details><summary>Show Answer & Explanation</summary>

**A.** The `__destruct` writes `data` to `file`; craft a `Logger` whose properties point at the webroot with PHP content, and on destruction it writes your webshell. B, C, and D misunderstand the primitive (a controllable file write via the destructor).
</details>

---

**Q2.** Which PHP magic method fires automatically when a deserialised object is destroyed, making it a common POP-chain entry point?

- A. `__destruct()`, which runs on object destruction at end of request.
- B. `__construct()`, which runs when the object is deserialised.
- C. `__invoke()`, which runs on object destruction automatically.
- D. `__set()`, which runs whenever the object is deserialised.

<details><summary>Show Answer & Explanation</summary>

**A.** `__destruct()` runs on destruction, a reliable trigger. `__construct` is *not* called by `unserialize` (B); `__invoke` fires when an object is called as a function (C); `__set` fires on inaccessible-property writes (D).
</details>

---

**Q3.** An app never calls `unserialize`, but it runs `md5_file($_GET['avatar'])` and accepts uploads. Why is deserialization still possible?

- A. A `phar://` path to an uploaded polyglot triggers PHAR metadata deserialization inside `md5_file`.
- B. `md5_file` internally calls `unserialize` on every file it hashes.
- C. MD5 collisions let you inject a serialized object into the hash.
- D. It is not possible without an explicit `unserialize` call.

<details><summary>Show Answer & Explanation</summary>

**A.** Filesystem functions acting on a `phar://` path auto-deserialise the archive's metadata; upload a JPEG+PHAR polyglot and reference it via `phar://` through `md5_file`. `md5_file` does not call `unserialize` on normal files (B); collisions are irrelevant (C); phar makes it possible without an explicit call (D is false).
</details>

---

**Q4.** A Python service runs `pickle.loads(base64.b64decode(req.data))`. Which object yields code execution on load?

- A. A class whose `__reduce__` returns `(os.system, ('id',))`.
- B. A dictionary containing the key `exec` mapped to a command string.
- C. A list whose first element is the shell command to run.
- D. A string beginning with `!!python/object` naming a callable.

<details><summary>Show Answer & Explanation</summary>

**A.** Pickle calls the callable returned by `__reduce__` with its args on load, so `(os.system, ('id',))` runs `id`. A plain dict/list (B, C) carries no execution semantics in pickle; `!!python/object` (D) is YAML syntax, not pickle.
</details>

---

**Q5.** Source uses `yaml.load(user_input)` (no loader argument, older PyYAML). Which payload executes a command, and what is the fix?

- A. `!!python/object/apply:os.system ['id']`; fix by using `yaml.safe_load`.
- B. `{"__reduce__": "os.system('id')"}`; fix by validating the JSON schema.
- C. `<?php system('id'); ?>`; fix by disabling PHP in YAML.
- D. `_$$ND_FUNC$$_function(){...}()`; fix by upgrading Node.

<details><summary>Show Answer & Explanation</summary>

**A.** Unsafe `yaml.load` instantiates/calls arbitrary Python via `!!python/object/apply`; `yaml.safe_load` (or `Loader=SafeLoader`) blocks it. B is pickle-flavoured JSON (wrong); C is PHP; D is node-serialize.
</details>

---

**Q6.** Why is building a PHP POP chain a *white-box* task rather than guesswork?

- A. You read the available classes and their magic methods to compose property values that cascade to a sink.
- B. You must brute-force serialized strings until one triggers a shell.
- C. The chain is identical for every PHP app regardless of its code.
- D. Only the PHP interpreter version determines the chain, not the code.

<details><summary>Show Answer & Explanation</summary>

**A.** POP chains are assembled from the *specific* classes present; reading source reveals the magic methods and reachable sinks. Brute force (B) is impractical; chains are app-specific (C); the code, not just the version, defines the gadgets (D).
</details>

---

**Q7.** For your exploit script, how do you generate a PHP POP-chain payload from within a Python exploit?

- A. Shell out to `php -r` (or embed the serialized string) to produce the exact blob, then send it with requests.
- B. Ask the operator to paste the serialized string at runtime.
- C. Hardcode a random string and hope it deserialises.
- D. Use sqlmap to generate the serialized object.

<details><summary>Show Answer & Explanation</summary>

**A.** Generate the serialized chain deterministically (e.g. `php -r` as a subprocess or a known-good literal), then deliver via `requests`. Operator paste (B) breaks automation; random strings (C) will not deserialise; sqlmap (D) is unrelated.
</details>

---

**Q8.** Which remediation set correctly addresses the PHP and Python sinks here?

- A. Avoid deserialising untrusted input; use JSON/`safe_load`, never `pickle` untrusted data, and be wary of phar triggers.
- B. Base64-encode all serialized data so attackers cannot read it.
- C. Add a WAF rule blocking the string `O:` and `!!python` only.
- D. Rename the vulnerable classes so POP chains cannot reference them.

<details><summary>Show Answer & Explanation</summary>

**A.** The fix is not to deserialise untrusted data with dangerous deserialisers: use data-only formats, `yaml.safe_load`, avoid `pickle` on user input, and recognise phar triggers. Encoding (B) hides nothing; narrow WAF rules (C) are bypassable; renaming classes (D) is obscurity.
</details>
