# Domain 2 — Quiz 1: Prepared-Statement Auditing & Injection Discovery

> White-box scenarios. Answer before expanding.

---

**Q1.** Which PDO snippet is injectable?

- A. `$db->query("SELECT * FROM u WHERE id = " . $_GET['id']);`
- B. `$s=$db->prepare("SELECT * FROM u WHERE id = ?"); $s->execute([$_GET['id']]);`
- C. `$s=$db->prepare("SELECT * FROM u WHERE id = :id"); $s->execute([':id'=>$_GET['id']]);`
- D. `$s=$db->prepare("SELECT * FROM u WHERE id = ?"); $s->bindValue(1,$_GET['id'],PDO::PARAM_INT); $s->execute();`

<details><summary>Show Answer & Explanation</summary>

**A.** Concatenating `$_GET['id']` directly into the SQL string is classic injection. B, C, and D all bind the value as a parameter, so the input cannot alter the parse tree.
</details>

---

**Q2.** A Hibernate query reads `session.createQuery("from User where name='" + name + "'")`. Is ORM use protective here?

- A. No; HQL built by string concatenation is injectable just like raw SQL.
- B. Yes; Hibernate automatically parameterises all HQL strings.
- C. Yes; HQL cannot express injectable queries by design.
- D. No, but only because `createQuery` is deprecated, not due to injection.

<details><summary>Show Answer & Explanation</summary>

**A.** HQL injection is real — concatenating `name` into the query string is exploitable. The fix is `setParameter`. Hibernate does not auto-parameterise concatenated strings (B); HQL can absolutely be injectable (C); the issue is injection, not deprecation (D).
</details>

---

**Q3.** In EF Core, which call is safe from injection?

- A. `ctx.Users.FromSqlRaw("SELECT * FROM Users WHERE Name = {0}", name)`
- B. `ctx.Users.FromSqlRaw($"SELECT * FROM Users WHERE Name = '{name}'")`
- C. `ctx.Database.ExecuteSqlRaw("DELETE FROM Users WHERE Name = '" + name + "'")`
- D. `ctx.Users.FromSqlRaw("SELECT * FROM Users WHERE Name = '" + name + "'")`

<details><summary>Show Answer & Explanation</summary>

**A.** The `{0}` form is parameterised by EF — the value is bound. B uses string interpolation into `FromSqlRaw` (injectable; `FromSqlInterpolated` would parameterise, but `FromSqlRaw` with `$""` does not), and C/D concatenate directly.
</details>

---

**Q4.** A `node-postgres` handler runs `client.query("SELECT * FROM items WHERE cat = " + req.query.cat)`. The correct fix is:

- A. `client.query("SELECT * FROM items WHERE cat = $1", [req.query.cat])`
- B. `client.query("SELECT * FROM items WHERE cat = " + escape(req.query.cat))`
- C. Validate `req.query.cat` with a client-side regex before submitting.
- D. Switch to a `GET`-only route so SQL cannot be injected.

<details><summary>Show Answer & Explanation</summary>

**A.** Parameterised query with `$1` binds the value. Ad-hoc `escape()` (B) is error-prone and not the driver's parameterisation; client-side validation (C) is bypassable; the HTTP method (D) is irrelevant to injection.
</details>

---

**Q5.** You see a prepared statement that *still* concatenates a column name for `ORDER BY`: `prepare("... ORDER BY "+col)`. Why is this a finding?

- A. Column/identifier positions cannot be bound as parameters, so concatenating `col` is injectable.
- B. `ORDER BY` is never injectable, so it is a false positive.
- C. Prepared statements make all concatenation safe, so there is no issue.
- D. The bug is only cosmetic because `ORDER BY` cannot leak data.

<details><summary>Show Answer & Explanation</summary>

**A.** Parameters bind *values*, not identifiers; a concatenated `ORDER BY` column is a real injection point (often exploitable via subqueries/CASE for blind extraction). Allow-list the column instead.

- **B**/**D** are wrong: `ORDER BY` injection can drive boolean/error extraction.
- **C** is the misconception: binding elsewhere does not sanitise a concatenated identifier.
</details>

---

**Q6.** Which grep most efficiently surfaces raw query construction across a polyglot repo?

- A. `grep -rniE "createQuery|FromSqlRaw|ExecuteSqlRaw|\.query\(|Statement\b|rawQuery"`
- B. `grep -rni "SELECT"`
- C. `grep -rni "database"`
- D. `grep -rni "password"`

<details><summary>Show Answer & Explanation</summary>

**A.** Targeting the query-builder APIs finds construction sites directly. `SELECT` (B) floods with comments/strings; `database` (C) and `password` (D) do not pinpoint query construction.
</details>

---

**Q7.** Before writing an extractor, why confirm the injection with a benign boolean probe rather than a destructive payload?

- A. A boolean probe proves injectability without risking data/impact, matching professional and exam practice.
- B. Destructive payloads extract data faster, so they are preferred.
- C. Boolean probes are the only syntax databases accept.
- D. A benign probe automatically parameterises the query.

<details><summary>Show Answer & Explanation</summary>

**A.** Confirm with a safe `AND 1=1`/`AND 1=2` style probe; destructive actions (e.g. `DROP`) cause unauthorised impact. B is reckless; C/D are false.
</details>

---

**Q8.** For the report, what is the precise root cause and fix for an injectable concatenated query?

- A. User input was concatenated into SQL; use parameterised queries/prepared statements and least-privilege DB accounts.
- B. The database was unpatched; upgrading the DB engine removes the injection.
- C. The WAF failed; deploying a WAF fully remediates SQL injection.
- D. Verbose errors were shown; hiding errors removes the vulnerability.

<details><summary>Show Answer & Explanation</summary>

**A.** The root cause is code+data mixing; parameterisation fixes it, and least privilege limits blast radius. Patching the engine (B) does not fix app-level injection; a WAF (C) is mitigation, not a fix; hiding errors (D) only blinds you to error-based, leaving blind SQLi.
</details>
