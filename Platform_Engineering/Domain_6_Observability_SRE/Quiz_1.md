# PE Domain 6: Observability & SRE — Scenario Quiz 1

Test your mastery of Prometheus architecture, PromQL calculations, and TSDB storage.

---

### Question 1
When computing the 99th percentile request latency using `histogram_quantile(0.99, ...)` in PromQL, why is omitting `by (le)` considered a catastrophic mathematical bug?

A) It increases query execution time by 5000%.  
B) Prometheus will delete the underlying metric.  
C) The query will fail with a syntax error.  
D) Without `by (le)`, the `sum()` function collapses and strips the `le` (less-than-or-equal) bucket label from the time series, leaving the `histogram_quantile()` function with no bucket boundaries to interpolate across, producing garbage or empty results.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** D  
**Explanation:** `histogram_quantile()` relies on the `le` label to construct the cumulative distribution function. If you aggregate multiple series with `sum(...)` without preserving `by (le)`, all bucket tags are wiped out, making accurate quantile interpolation impossible.
</details>

---

### Question 2
Why does the Google SRE handbook strictly recommend using `rate()` rather than `irate()` when writing alerting rules and SLO burn rate calculations in Prometheus?

A) `irate()` does not work with counters.  
B) `irate()` is deprecated in Prometheus 2.0+.  
C) `rate()` uses less disk space.  
D) `irate()` calculates instant rate between the last two data points, making it hypersensitive to transient, single-second spikes that immediately disappear; `rate()` calculates the average rate over the entire specified window, providing smooth, statistically reliable alerts.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** D  
**Explanation:** `irate()` is designed for zooming in on high-resolution graphs during active troubleshooting. For alerts and SLOs, `irate()` generates false alarms on isolated blips. `rate()` averages over the window (e.g., 5m), dampening noise and correctly handling counter resets.
</details>

---

### Question 3
An engineer restarts an application pod. The Prometheus counter metric `http_requests_total` resets from 5,000,000 back to 0. How does the PromQL `rate(http_requests_total[5m])` function handle this counter reset?

A) It drops the metric permanently.  
B) It automatically detects that the counter value decreased, assumes a process restart occurred, and adjusts its calculation to account for the reset without generating negative numbers or anomalies.  
C) It reports a massive negative rate spike.  
D) It crashes the Prometheus server.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** B  
**Explanation:** The `rate()` function is counter-reset aware. Whenever an incoming sample value is strictly lower than the previous sample, `rate()` treats it as a counter reset (monotonicity violation) and calculates the increase accordingly.
</details>

---

### Question 4
In Prometheus TSDB storage, what is the role of the **Write-Ahead Log (WAL)**?

A) It compresses old metrics into zip files.  
B) It records user login attempts in a text file.  
C) It records raw incoming samples to disk synchronously before they are compressed into memory chunks, ensuring that if the Prometheus server process crashes, uncommitted in-memory data can be reconstructed completely on restart.  
D) It routes alerts to Alertmanager.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** C  
**Explanation:** The WAL ensures crash durability. Because Prometheus buffers active samples in the in-memory Head chunk for up to 2 hours before compaction, a sudden crash would lose that data without the WAL to replay recent writes upon reboot.
</details>

---

### Question 5
What is the primary danger of adding a label with high cardinality (such as `user_id` or `uuid`) to a Prometheus metric?

A) The metric will be converted to uppercase.  
B) It slows down application database queries.  
C) Prometheus will refuse to scrape the endpoint.  
D) It creates an independent time series for every unique value, causing exponential growth in time series count, consuming gigabytes of RAM in the TSDB inverted index, degrading query latency, and eventually causing Prometheus to crash with Out-Of-Memory (OOM).  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** D  
**Explanation:** High cardinality is the archenemy of Prometheus. Prometheus stores every unique combination of label keys and values as a separate time series. Adding 100,000 user IDs creates 100,000 time series, overwhelming TSDB memory and query performance.
</details>

---

### Question 6
You have two metric vectors:
- `http_requests_total{job="api", instance="10.0.1.5", handler="/checkout"}`
- `node_metadata{instance="10.0.1.5", datacenter="us-east-1"}`
Which PromQL keyword is required to join these vectors on the `instance` label and copy the `datacenter` label into the resulting time series?

A) `outer_join`  
B) `on(instance) group_left(datacenter)`  
C) `join_left`  
D) `merge_by(instance)`  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** B  
**Explanation:** Many-to-one matching in PromQL uses `group_left` (or `group_right` for one-to-many). `on(instance) group_left(datacenter)` matches on the shared `instance` label and copies the `datacenter` metadata label from the right-hand vector into the result.
</details>

---

### Question 7
In Prometheus histograms, what does the `+Inf` bucket represent?

A) An error indicating that the metric has overflowed.  
B) Infinite mathematical latency.  
C) The cumulative bucket that captures all observed events, regardless of their duration, ensuring the total event count is preserved.  
D) A bucket reserved for timeouts.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** C  
**Explanation:** Histograms are cumulative. The `+Inf` bucket (`le="+Inf"`) represents the total number of events recorded across all durations, matching the value of the companion `<metric_name>_count` counter.
</details>

---

### Question 8
What is the function of a Prometheus **Recording Rule**?

A) It precomputes frequently needed or computationally expensive PromQL expressions (e.g., 5-minute error rates across 1,000 pods) and saves the result as a new time series, making dashboards and alerts execute near-instantly.  
B) It generates video recordings of outages.  
C) It archives metrics to tape storage.  
D) It records audio logs of developers.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** A  
**Explanation:** Complex aggregations across thousands of series can take 10+ seconds to evaluate. Recording rules run periodically in the background, computing and storing the aggregated value so dashboard panels and alerting rules query a single precomputed metric in milliseconds.
</details>

---

### Question 9
In Prometheus, how does a **Gauge** differ from a **Counter**?

A) A Counter can go up and down; a Gauge can only increase.  
B) A Counter is a monotonically increasing cumulative value that only resets to zero on restart; a Gauge represents a single numerical value that can arbitrarily go up and down (e.g., temperature, current memory usage, concurrent connections).  
C) A Gauge cannot be plotted on a graph.  
D) Counters are only used for hardware monitoring.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** B  
**Explanation:** Counters track accumulated totals (requests served, errors encountered) and are analyzed with `rate()`. Gauges capture snapshot state (memory in use, thread count) and are analyzed directly or with smoothing functions like `avg_over_time()`.
</details>

---

### Question 10
What tool should platform engineers use to lint, validate, and unit-test Prometheus alerting and recording rules before deploying them to production?

A) `promtool`  
B) `kubectl lint`  
C) `docker test`  
D) `grep`  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** A  
**Explanation:** `promtool check rules` verifies the syntactic correctness of rule YAML files, while `promtool test rules` executes automated unit tests against synthetic metric series to verify alerting logic before deployment.
</details>
