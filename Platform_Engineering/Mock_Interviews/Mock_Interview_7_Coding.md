# Mock Interview 7: Systems & Automation Coding

Simulates a 60-minute live systems programming interview in Python or Go. Tests algorithms, concurrency, system call interactions, and Kubernetes controller paradigms.

---

## Candidate Brief (Safe to read in advance)

- **Duration:** 60 minutes.
- **Language:** Go or Python 3.
- **Format:** Live coding session. You will be asked to write clean, production-ready code with proper error handling, unit tests, and optimal time/space complexity.
- **Tools:** Standard IDE / code editor. Standard library preferred; third-party packages allowed only if specified.

---

## Interviewer Packet (Confidential — Do not read beforehand)

### Variant A: Distributed Token Bucket Rate Limiter with Sliding Window
- **Task:** Implement a thread-safe, high-performance Token Bucket rate limiter in Go or Python.
- **Requirements:**
  1. Initialize with `capacity` (max tokens) and `refill_rate` (tokens per second).
  2. Implement `Allow(tokens int) bool` that returns `true` if sufficient tokens exist, atomically deducting them.
  3. Must NOT run a background timer thread per bucket (must calculate refill lazily on-demand using timestamp deltas).
  4. Must support concurrent access safely without race conditions.
  5. *Follow-up / Part 2:* Extend to a distributed Redis sliding-window log or counter script in Lua.
- **Edge Cases to Watch:**
  - Clock skew or non-monotonic clocks (use monotonic clock).
  - Integer overflow when calculating token additions after prolonged idle time.
  - Lock contention under high concurrency.

### Variant B: Kubernetes Custom Reconciler Loop Engine
- **Task:** Implement a miniature level-triggered reconciliation engine modeling `client-go` controller runtime.
- **Requirements:**
  1. Maintain a desired state store and an actual state store for virtual resources.
  2. Implement an exponential backoff retry queue (`MaxRetries: 5`, base delay 100ms, max delay 10s with full jitter).
  3. Implement `Reconcile(key string) (Result, error)`:
     - If desired exists but actual does not: Create resource.
     - If desired differs from actual: Update resource.
     - If desired is nil but actual exists: Delete resource (respecting simulated finalizers).
  4. Ensure idempotent reconciliation: running `Reconcile` multiple times on identical state produces no side-effects.

### Variant C: Pipeline DAG Execution Engine with Worker Pools
- **Task:** Build a concurrent Directed Acyclic Graph (DAG) task runner for CI/CD pipeline steps.
- **Requirements:**
  1. Given a set of tasks where each task specifies `Dependencies: []string` and `Execute() error`.
  2. Detect circular dependencies before execution and return a descriptive cycle error.
  3. Execute independent tasks concurrently using a bounded worker pool (e.g., max 4 concurrent tasks).
  4. If any task fails, gracefully abort dependent downstream tasks while allowing un-related in-flight tasks to complete.
  5. Return total pipeline execution summary and error traces.

---

## Hint Ladder
- *Level 1 (Nudge):* "How can we avoid spinning up a background ticker goroutine that wastes CPU while idle?"
- *Level 2 (Guiding Question):* "How can we calculate how many tokens accumulated between `last_refill_time` and `now`?"
- *Level 3 (Explicit Cue):* "`tokens_to_add = (now - last_refill_time) * refill_rate`. Clamp the result at `capacity`."

---

## Scoring Rubric

| Score | Anchor | Behavioral Criteria |
| :---: | :--- | :--- |
| **1** | Strong No Hire | Syntax errors; fails to write working code; fails to implement concurrency safety (data races). |
| **2** | No Hire | Working code but inefficient (uses polling sleep loops); misses edge cases or fails cycle detection in DAG. |
| **3** | Hire | Clean, idiomatic code; correct synchronization (mutexes, atomics, channels); handles errors and edge cases cleanly. |
| **4** | Strong Hire | Production-grade implementation in under 35 minutes; writes comprehensive table-driven tests; flawlessly implements distributed extension. |

---

## Self-Debrief Template
- **Problem Chosen:** Token Bucket / Reconciler Loop / DAG Scheduler
- **Time to Working Solution:** ___ min
- **Concurrency / Edge Cases Missed:**
- **Action Items:**
