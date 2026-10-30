# Domain 3 — Hands-On Lab Exercises

> **Lab safety & authorisation.** Deploy each vulnerable app yourself in Docker on localhost. Good targets: small apps you write around each sink, plus deliberately-vulnerable projects for Java (a `readObject` endpoint), .NET (`BinaryFormatter`/ViewState lab), and Node (`node-serialize`). Keep exploit code in a directory separate from any downloaded/extracted sample. Never fire gadget payloads at systems you do not own.

---

## Exercise 3.1 — PHP POP Chain

**Objective:** craft a property-oriented program chain to RCE.

**Setup:** a PHP app with `unserialize($_COOKIE['data'])` and a class whose `__destruct` performs a dangerous action.

**Tasks:**
1. Identify the sink and the magic methods on available classes.
2. Instantiate the classes locally, set properties, and `serialize()` the chain.
3. Send it to the sink and achieve code execution.

**Success criteria:** command execution via a crafted POP chain, with each magic-method hop explained.

---

## Exercise 3.2 — PHAR Deserialization

**Objective:** trigger deserialization without an explicit `unserialize`.

**Setup:** a PHP app that calls `file_exists()`/`md5_file()` on a user-supplied path and accepts image uploads.

**Tasks:**
1. Build a polyglot JPEG+PHAR whose metadata is your POP chain.
2. Upload it and reference it via `phar://` through the filesystem-function sink.
3. Confirm the chain fires.

**Success criteria:** RCE via phar metadata deserialization, with the filesystem-function trigger identified.

---

## Exercise 3.3 — Python Pickle `__reduce__`

**Objective:** RCE through a pickle sink.

**Setup:** a Python service that `pickle.loads()` a request field.

**Tasks:**
1. Write a class whose `__reduce__` returns `(os.system, ('id',))`.
2. `pickle.dumps` it, deliver to the sink (base64 as needed).
3. Upgrade the command to a reverse shell.

**Success criteria:** a reverse shell via a malicious pickle, with `__reduce__` semantics explained.

---

## Exercise 3.4 — PyYAML `yaml.load`

**Objective:** exploit unsafe YAML loading.

**Setup:** a Python app using `yaml.load(user_input)` without `SafeLoader`.

**Tasks:**
1. Craft `!!python/object/apply:os.system ['id']`.
2. Deliver it and confirm execution.
3. Patch to `yaml.safe_load` and confirm the payload no longer runs.

**Success criteria:** execution via unsafe YAML, plus a verified `safe_load` fix.

---

## Exercise 3.5 — Java `readObject` with ysoserial

**Objective:** RCE through a Java deserialization endpoint.

**Setup:** a Java app with a `readObject` sink and Commons-Collections on the classpath.

**Tasks:**
1. Confirm the vulnerable library version from the dependencies.
2. Generate a matching `ysoserial` payload (e.g. CommonsCollections6).
3. Deliver it to the sink and catch a shell.

**Success criteria:** a shell via the matched gadget chain, with the library-to-chain reasoning documented.

---

## Exercise 3.6 — Jackson/SnakeYAML Typing

**Objective:** exploit polymorphic typing.

**Setup:** a Java app with Jackson `enableDefaultTyping()` or SnakeYAML `Yaml.load()`.

**Tasks:**
1. Identify where untrusted JSON/YAML is deserialised with type handling on.
2. Supply a payload specifying a dangerous type to instantiate.
3. Reach code execution (or confirm a JNDI lookup to your listener).

**Success criteria:** instantiation of an attacker-chosen type leading to execution, documented.

---

## Exercise 3.7 — .NET Json.NET TypeNameHandling

**Objective:** RCE via polymorphic JSON.

**Setup:** a .NET app deserialising request JSON with `TypeNameHandling.Auto/All`.

**Tasks:**
1. Confirm the setting in source.
2. Use `ysoserial.net` (`ObjectDataProvider`, `Json.Net`) to generate a `Process.Start` payload.
3. Deliver it and confirm execution.

**Success criteria:** command execution via `$type`-driven instantiation, with the setting flagged as the root cause.

---

## Exercise 3.8 — ViewState Forging

**Objective:** forge a malicious `__VIEWSTATE` using leaked keys.

**Setup:** an ASP.NET lab with a `machineKey` you obtain via a simulated LFI/SQLi reading `web.config`.

**Tasks:**
1. Recover `validationKey`/`decryptionKey`.
2. Generate a signed ViewState payload with `ysoserial.net` using those keys.
3. Submit it and achieve RCE.

**Success criteria:** RCE via a forged, correctly-MAC'd ViewState, with the key-leak dependency explained.

---

## Exercise 3.9 — Node.js node-serialize

**Objective:** RCE via function-capable deserialization.

**Setup:** an Express app calling `nodeSerialize.unserialize(req.cookies.profile)`.

**Tasks:**
1. Confirm the sink.
2. Craft the `_$$ND_FUNC$$_...()` self-invoking payload running `child_process.exec`.
3. Deliver it and catch a shell.

**Success criteria:** a shell via node-serialize, with the trailing-`()` self-invocation explained.

---

## Exercise 3.10 — One-Command Deserialization Exploit

**Objective:** automate payload generation + delivery + shell.

**Setup:** any sink from 3.1–3.9.

**Tasks:**
1. Build or invoke the payload generator inside your Python script.
2. Deliver with correct encoding to the sink.
3. Start a listener thread and catch the shell automatically.

**Success criteria:** `python3 deser_rce.py http://target --lhost .. --lport ..` yields a shell unattended.

---

## Consolidation Challenge — Find the Sink Cold

On an unfamiliar app in a language of your choice, locate the deserialization sink by source review, determine the available gadget/magic methods, and build a reliable one-command exploit. Note how you confirmed the gadget was present (classpath/dependencies/classes). This is a full OSWE RCE milestone.
