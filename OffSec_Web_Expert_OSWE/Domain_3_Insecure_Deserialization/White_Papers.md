# Domain 3 — Reference White Papers & Deep Dives

> Mechanism-level background for insecure deserialization. The core insight: deserialization is *implicit code execution* whenever rebuilding an object runs attacker-influenced logic.

---

## 1. Serialization as an Execution Surface

- Deserialisers rebuild object graphs and, in doing so, invoke lifecycle hooks: constructors, magic methods (PHP), `readObject`/`readResolve` (Java), `__reduce__`/`__setstate__` (Python), type handlers (.NET/Jackson). A **gadget** is an existing class whose hook does something useful to an attacker; a **chain** composes gadgets so deserialisation alone reaches `exec`.
- The attacker supplies only *data*; the vulnerability is that the deserialiser treats structure/type as trustworthy.

**Sources:** "Marshalling Pickles" (Frohoff & Lawrence, the foundational Java deserialization talk); OWASP Deserialization Cheat Sheet.

---

## 2. PHP Magic Methods and POP Chains

- `__wakeup`, `__destruct`, `__toString`, `__call`, `__get` run automatically in specific lifecycle moments. Property-Oriented Programming wires object properties so these fire in sequence toward a sink.
- **Phar:** PHAR metadata is PHP-serialized; filesystem functions on a `phar://` path deserialise it *without* `unserialize`, which is why phar expands the attack surface to any path-taking filesystem call.

**Sources:** PHP manual (magic methods, phar); Sam Thomas, "It's a PHP unserialization vulnerability Jim, but not as we know it" (phar research).

---

## 3. Java Gadget Chains and JNDI

- `ObjectInputStream.readObject` will instantiate any serialisable class on the classpath; chains like CommonsCollections abuse reflective "transformers" to reach `Runtime.exec`. `TemplatesImpl` carries compiled bytecode executed on instantiation.
- Polymorphic deserialisers (Jackson default typing, SnakeYAML, XStream, XMLDecoder) let the input choose the type, broadening the gadget space.
- **JNDI injection:** `lookup()` on an attacker URL fetches a remote object factory (LDAP/RMI) — the mechanism behind Log4Shell-class RCE.

**Sources:** ysoserial project; "Exploiting JNDI Injections in Java" (Veracode/BlackHat); Oracle JEP 290 (serialization filtering).

---

## 4. .NET Formatters and ViewState

- `BinaryFormatter`/`SoapFormatter`/`NetDataContractSerializer`/`LosFormatter` reconstruct typed graphs; `ObjectDataProvider` is a convenient gadget that invokes arbitrary methods (`Process.Start`).
- **ViewState** is `LosFormatter`-serialised and, when MAC-protected, requires the `machineKey`. A leaked `validationKey`/`decryptionKey` (e.g. a `web.config` disclosed via LFI/SQLi) lets an attacker forge a signed, malicious ViewState.
- **Json.NET** `TypeNameHandling` other than `None` embeds `$type`, enabling attacker-chosen instantiation.

**Sources:** ysoserial.net; Microsoft `BinaryFormatter` security guidance (obsoletion); "Exploiting ViewState Deserialization" (Soroush Dalili / Microsoft advisories).

---

## 5. Python and Node.js

- **Pickle** is documented as unsafe for untrusted data: `__reduce__` returns a callable executed on load. **PyYAML** `load` instantiates arbitrary objects unless `SafeLoader` is used.
- **Node:** `node-serialize`/`funcster`/`cryo` can reconstruct *functions*; a self-invoking function in the payload runs on deserialisation.

**Sources:** Python `pickle` docs (explicit security warning); PyYAML docs; "Exploiting Node.js deserialization in node-serialize" (Ajin Abraham).

---

## 6. Primary Reference Index

- **OWASP Deserialization Cheat Sheet** and **WSTG** deserialization tests.
- **ysoserial** / **ysoserial.net** repos — gadget catalogues; read which library each chain needs.
- **PortSwigger Web Security Academy** — insecure deserialization labs (PHP/Java).
- **Language docs** — pickle/PyYAML/`BinaryFormatter` security notes are primary sources.

> **Study tip:** keep a sink→gadget map per language. On the exam, you find the sink by review, then the only remaining question is "which gadget is available here?" — answered from the dependency list / class inventory.
