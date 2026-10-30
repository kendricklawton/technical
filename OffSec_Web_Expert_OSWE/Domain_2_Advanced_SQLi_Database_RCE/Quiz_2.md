# Domain 2 — Quiz 2: Blind Extraction Automation

> White-box scenarios. Answer before expanding.

---

**Q1.** Your injection yields two distinguishable pages (true/false) but no data. Roughly how many requests does binary search need per printable ASCII character?

- A. About 7, since log2(~95) rounds up to 7.
- B. About 95, one per possible character.
- C. Exactly 1, since the page reveals the character directly.
- D. About 26, one per lowercase letter.

<details><summary>Show Answer & Explanation</summary>

**A.** Binary search over ~95 printable values needs ⌈log2(95)⌉ ≈ 7 comparisons per character. 95 (B) is linear search; 1 (C) assumes in-band output; 26 (D) ignores digits/symbols and is not how binary search scales.
</details>

---

**Q2.** Why recover the target value's length before extracting characters?

- A. Knowing the length lets you fan out one thread per index for parallel extraction.
- B. The length is the password, so no further extraction is needed.
- C. Databases refuse `SUBSTRING` without a known length.
- D. Length reveals the hashing algorithm used.

<details><summary>Show Answer & Explanation</summary>

**A.** With a known length you can parallelise across indices with a thread pool. The length is not the secret (B); `SUBSTRING` works without pre-known length (C); length hints at but does not reveal the algorithm as "needed" here (D).
</details>

---

**Q3.** Converting a boolean extractor to time-based, what is the core change?

- A. Replace the page-content check with a response-time measurement against a delay threshold.
- B. Replace `SUBSTRING` with `SLEEP` to read characters directly from the timer.
- C. Remove the binary search, since timing reveals the whole value at once.
- D. Switch from GET to POST, since timing only works over POST.

<details><summary>Show Answer & Explanation</summary>

**A.** The oracle becomes "did the response take longer than the threshold?" using `IF(cond,SLEEP(t),0)`/`pg_sleep`/`WAITFOR`. `SLEEP` does not emit characters (B); binary search is still needed (C); HTTP method is irrelevant (D).
</details>

---

**Q4.** Your time-based extractor occasionally flips bits on a jittery network. Best mitigation?

- A. Increase the sleep delay and/or confirm each bit with a repeat probe above the threshold.
- B. Remove threading entirely and accept it will be slow but is otherwise unfixable.
- C. Switch to UNION-based extraction, which always works regardless of output.
- D. Lower the delay to zero so jitter cannot affect the measurement.

<details><summary>Show Answer & Explanation</summary>

**A.** A larger delay widens the signal/noise gap, and re-probing borderline bits adds reliability. Dropping threads (B) does not fix jitter; UNION (C) needs in-band output you do not have; a zero delay (D) destroys the signal.
</details>

---

**Q5.** Why is a hand-written threaded extractor acceptable on the OSWE when sqlmap-style automation is discouraged on some OffSec exams?

- A. It is your own code demonstrating understanding; OSWE expects you to build reliable exploit tooling.
- B. sqlmap is required on the OSWE and your script merely wraps it.
- C. Extractors are banned entirely; you must extract manually by hand.
- D. The OSWE forbids any Python, so only shell one-liners are allowed.

<details><summary>Show Answer & Explanation</summary>

**A.** OSWE's deliverable is *your* exploit; a custom extractor shows mastery and is expected. It does not wrap sqlmap (B); extractors are not banned (C); Python is the norm (D is false).
</details>

---

**Q6.** A `$regex` NoSQL field lets you test prefixes. How do you extract a secret character-by-character?

- A. Query `{"$regex":"^<known>X"}` for each candidate X, keeping the prefix that matches.
- B. Query `{"$ne":null}` repeatedly until the value appears in the response.
- C. Request `UNION SELECT` through the regex to dump the collection.
- D. Use `$where:"1=1"` to print the field directly.

<details><summary>Show Answer & Explanation</summary>

**A.** Anchor `^` and extend the known prefix one character at a time, testing candidates — the NoSQL analogue of boolean-blind. `$ne` (B) is a bypass, not extraction; `UNION` (C) is SQL; `$where:"1=1"` (D) does not print the field.
</details>

---

**Q7.** Your extractor must run as a single command for the exam. Which design meets that?

- A. `argparse` for the target URL, internal functions for length + character extraction, printing progress and the final value.
- B. A script that pauses to ask the operator for each character's value.
- C. A Jupyter notebook the grader runs cell by cell.
- D. A set of curl commands the grader pastes one at a time.

<details><summary>Show Answer & Explanation</summary>

**A.** A self-contained script taking the target and extracting unattended satisfies the single-command rule. B, C, and D all require manual intervention.
</details>

---

**Q8.** What proves your extractor is reliable enough for exam submission?

- A. It produces the same correct value across repeated runs without manual retries.
- B. It works once out of several attempts if you are patient.
- C. It extracts most characters and you fill the rest in by guessing.
- D. It runs quickly even if the output sometimes differs.

<details><summary>Show Answer & Explanation</summary>

**A.** Determinism is the bar: repeated runs yield the same correct value unattended. Flaky (B), partial (C), or inconsistent-but-fast (D) extraction loses marks.
</details>
