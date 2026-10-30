# Domain 2 — Hands-On Lab Exercises

> **Lab safety & authorisation.** Deploy vulnerable DB-backed apps yourself (Docker on localhost) — a small PDO/Express/Spring app you write with a deliberately concatenated query, plus local PostgreSQL/MySQL/MSSQL containers. Never run extractors or RCE primitives against databases you do not own.

---

## Exercise 2.1 — Spot the Injectable Query

**Objective:** distinguish parameterised from concatenated across four stacks.

**Setup:** four tiny apps (PDO, Hibernate/JDBC, EF `FromSqlRaw`, Node `pg`) each with one safe and one unsafe query.

**Tasks:**
1. For each app, identify the injectable site by reading source.
2. Explain why the "safe" one is safe (bound parameter) and the other is not.
3. Confirm the injectable one with a boolean probe.

**Success criteria:** correct classification of all eight queries with a one-line reason each.

---

## Exercise 2.2 — Boolean-Blind Extractor

**Objective:** build a fast threaded extractor.

**Setup:** an app with a boolean-blind injection (page differs on true/false).

**Tasks:**
1. Recover the target value's length first.
2. Implement binary-search character extraction with `concurrent.futures`.
3. Extract the admin password hash and compare single- vs multi-threaded runtime.

**Success criteria:** a working `extract.py` that recovers the value and a timing note showing the threading speedup.

---

## Exercise 2.3 — Time-Based Blind Extractor

**Objective:** extract when there is no page difference at all.

**Setup:** the same app, output suppressed, with a working `SLEEP`/`pg_sleep`/`WAITFOR`.

**Tasks:**
1. Convert your oracle to measure `response.elapsed` against a threshold.
2. Tune the delay and threshold for your network to avoid false bits.
3. Extract the same value and verify it matches 2.2.

**Success criteria:** reliable time-based extraction matching the boolean result.

---

## Exercise 2.4 — PostgreSQL to RCE

**Objective:** reach command execution three ways.

**Setup:** a local PostgreSQL reachable via a stacked-query injection, superuser role.

**Tasks:**
1. `COPY ... FROM PROGRAM` to run a command and read its output.
2. Use large objects (`lo_*`) to write a file to disk.
3. (Stretch) Write and `CREATE FUNCTION` a C UDF, then call it.

**Success criteria:** command output via `COPY PROGRAM` and a file written via large objects, documented.

---

## Exercise 2.5 — MySQL to Webshell

**Objective:** write a webshell via the database.

**Setup:** MySQL with `FILE` and permissive `secure_file_priv`, plus a writable webroot.

**Tasks:**
1. `INTO OUTFILE` a PHP webshell into the webroot and execute it.
2. Separately, poison `general_log` to write PHP into the webroot and trigger it.
3. Note which preconditions each requires.

**Success criteria:** RCE via both `INTO OUTFILE` and `general_log` poisoning.

---

## Exercise 2.6 — MSSQL to RCE

**Objective:** execute commands when `xp_cmdshell` is blocked.

**Setup:** a local SQL Server with `xp_cmdshell` disabled but OLE automation available.

**Tasks:**
1. Re-enable and use `xp_cmdshell` where permitted.
2. Where blocked, use `sp_OACreate` with `WScript.Shell` to run a command.
3. (Stretch) Load a hex-encoded CLR assembly and call its procedure.

**Success criteria:** command execution via at least two of the three MSSQL paths.

---

## Exercise 2.7 — Second-Order SQLi

**Objective:** exploit a stored-then-triggered injection.

**Setup:** an app that parameterises registration but concatenates the stored username in a later admin report query.

**Tasks:**
1. Trace the stored value from write to its unsafe re-use.
2. Register a username carrying a payload.
3. Trigger the second query and extract data.

**Success criteria:** data extracted via the second-order path, with both query sites identified.

---

## Exercise 2.8 — WebSocket SQLi

**Objective:** extract over a WebSocket channel.

**Setup:** an app that sends queries via WebSocket messages, injectable.

**Tasks:**
1. Read the WS message handler to find the injectable field.
2. Adapt your extractor to use a WebSocket client instead of `requests`.
3. Extract a value over WS.

**Success criteria:** blind extraction working over WebSocket, reusing your binary-search logic.

---

## Exercise 2.9 — NoSQL Blind Extraction

**Objective:** extract a secret from MongoDB via `$regex`.

**Setup:** an Express+Mongo app with a `$regex`-injectable field.

**Tasks:**
1. Confirm `{"$regex":"^a"}`-style prefix matching.
2. Automate character-by-character extraction of the admin password.
3. (Stretch) If `$where` is present, evaluate JS to confirm server-side execution.

**Success criteria:** the admin secret extracted via `$regex`, with the automation reused from 2.2.

---

## Exercise 2.10 — One-Command SQLi Exploit

**Objective:** package extraction + RCE into a single script.

**Setup:** one app chaining a blind SQLi to a webshell/`PROGRAM` RCE.

**Tasks:**
1. `argparse` for target URL and `--lhost/--lport`.
2. Extract the credential/flag, then trigger the DB-to-RCE primitive.
3. Catch a reverse shell automatically in the same script.

**Success criteria:** `python3 sqli_rce.py http://target --lhost .. --lport ..` goes from injection to shell unattended.

---

## Consolidation Challenge — Blind-to-Shell in 90 Minutes

On an unfamiliar vulnerable app, find a SQLi by source review, write a reliable extractor, identify the engine, and reach RCE — then wrap it all in one script. Record which engine primitive you used and why. This is a complete OSWE RCE milestone.
