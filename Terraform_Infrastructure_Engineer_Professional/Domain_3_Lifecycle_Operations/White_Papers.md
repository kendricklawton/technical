# Domain 3: Lifecycle Operations & Declarative Refactoring — Reference White Papers & Architectural Deep Dives

## 1. Declarative State Evolution: `moved`, `removed`, and Config-Driven `import`
* **Primary Reference:** *HashiCorp Engineering Design Document — Declarative Refactoring in GitOps Pipelines*
* **Core Architectural Takeaways:**
  * Imperative state manipulation (`terraform state mv/rm`, `terraform import`) breaks the fundamental invariant of GitOps because state mutations happen out-of-band before code review.
  * By embedding `moved`, `removed`, and `import` blocks directly into `.tf` files, refactoring actions appear explicitly inside the `terraform plan` output attached to the Pull Request and execute atomically when merged.
  * Published module authors can chain historical `moved` blocks across minor versions so module consumers upgrade transparently without state breakage.

---

## 2. Continuous Infrastructure Validation Architecture (`check` vs. `precondition` / `postcondition`)
* **Primary Reference:** *HashiCorp Continuous Validation & Health Assessments Whitepaper*
* **Core Architectural Takeaways:**
  * **Defense-in-Depth Validation Timeline:**
    1. **Input Variable `validation`:** Static contract verification prior to graph walk.
    2. **Resource/Data `precondition`:** Pre-flight dependency verification before invoking cloud provider write APIs.
    3. **Resource/Data `postcondition`:** Immediate post-operation verification of `self.*` provider return values (blocks downstream graph nodes if violated).
    4. **Top-Level `check` Block:** Post-execution operational health probing (executes scoped `data` sources at the end of the run and reports warnings without failing the apply).
