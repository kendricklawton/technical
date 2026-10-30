# Domain 2 — Reference White Papers & Deep Dives

> Mechanism-level background for white-box SQLi and database-to-RCE. The unifying idea: injection is a *parsing* failure (data parsed as code), and DB-to-RCE abuses legitimate database *features*.

---

## 1. Why Parameterisation Actually Works

- A prepared statement sends the SQL *template* to the server first; the server parses and plans it, then binds parameter *values* separately. Because parsing happens before the values arrive, a value can never change the parse tree — injection is structurally impossible.
- String concatenation (including ORM "raw" escape hatches like HQL string queries and `FromSqlRaw` interpolation) merges code and data before parsing, reintroducing the flaw. This is why "an ORM" is not automatically safe.

**Sources:** OWASP SQL Injection Prevention Cheat Sheet; database driver docs (PDO, JDBC, EF Core, node-postgres).

---

## 2. Blind Extraction as Information Theory

- Each boolean/time request yields one bit about the target. Binary search over the ASCII range recovers a character in ~⌈log2(95)⌉ ≈ 7 requests. Parallelising across character indices (known length) turns an O(n·7) sequential process into near-O(7) wall-clock with enough workers — the reason threaded extractors matter on a timed exam.
- Time-based extraction trades reliability for universality: it works with no response difference at all, but network jitter forces conservative thresholds.

**Sources:** PortSwigger blind SQLi labs; classic "blind SQL injection" papers; `concurrent.futures` docs.

---

## 3. PostgreSQL's OS Surface

- `COPY ... PROGRAM` exists for data pipelines but runs shell commands as the server account. Large objects (`lo_*`) are a legitimate binary-storage feature repurposed to write files (UDF shared objects). C-language UDFs are an extension mechanism that, once you can write a `.so`, becomes arbitrary native code.
- Each is gated by role privileges — which is exactly why least-privilege DB accounts are the remediation.

**Sources:** PostgreSQL docs (`COPY`, large objects, `CREATE FUNCTION ... C`); "SQLi to RCE in PostgreSQL" research.

---

## 4. MySQL and MSSQL OS Surfaces

- **MySQL:** `INTO OUTFILE/DUMPFILE` + `FILE` privilege writes files (webshell or UDF `.so` in the plugin dir); `general_log` redirection writes query text to an attacker-chosen path. `secure_file_priv` constrains where writes may go.
- **MSSQL:** `xp_cmdshell` (gated by `sp_configure`), `sp_OACreate` OLE automation, and user CLR assemblies are all designed extensibility features that become RCE for a sufficiently-privileged login.

**Sources:** MySQL `secure_file_priv`/UDF docs; MSSQL `xp_cmdshell`/`sp_OACreate`/CLR integration docs.

---

## 5. Second-Order and Non-HTTP Channels

- **Second-order** injection decouples storage from execution; the fix is to parameterise the *consuming* query and treat all stored data as untrusted. White-box review is the most effective way to find it because the two sites may be far apart.
- **WebSocket/other channels:** injection is transport-agnostic; the parsing flaw is in the query builder, not the protocol. Your tooling must speak the channel.

**Sources:** OWASP Testing Guide (second-order); PortSwigger WebSocket security.

---

## 6. NoSQL Injection

- Document stores accept query *operators* as data (`$ne`, `$gt`, `$regex`) and sometimes server-side JavaScript (`$where`). Trusting client-supplied structure/types yields logic bypass, blind extraction (`$regex` prefix matching), or JS execution.

**Sources:** OWASP NoSQL Injection; MongoDB operator and `$where` docs.

---

## 7. Primary Reference Index

- **OWASP SQL Injection Prevention** and **Query Parameterization** cheat sheets.
- **PortSwigger Web Security Academy** — SQLi (incl. blind) and NoSQL labs.
- **Database vendor docs** — the authoritative source for each RCE primitive and its privilege gate.
- **PayloadsAllTheThings — SQL Injection / NoSQL Injection** (payload reference; understand before use).

> **Study tip:** maintain a per-engine table in your notes: version probe, blind syntax, file read, file write, command exec, and the privilege each needs. On the exam you identify the engine, then execute the matching row.
