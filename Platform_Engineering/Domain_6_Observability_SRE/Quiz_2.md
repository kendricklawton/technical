# PE Domain 6: Observability & SRE — Scenario Quiz 2

Test your knowledge on OpenTelemetry Collector pipelines, distributed tracing context, and logging architectures.

---

### Question 1
Why MUST the `memory_limiter` processor be placed as the very first processor in an OpenTelemetry Collector pipeline?

A) It acts as a safety valve, continuously checking the Collector's memory usage and dropping data or returning backpressure *before* expensive decoding or batching operations occur, preventing the Collector pod from crashing with an Out-Of-Memory (OOM) kill.  
B) It speeds up network interfaces.  
C) It formats logs as JSON.  
D) It decrypts incoming TLS packets.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** A  
**Explanation:** If heavy processors (like batch or transform) run before the memory limiter, memory usage can spike out of control before the limiter can intervene. Placing `memory_limiter` first guarantees immediate shedding or backpressure under high memory pressure.
</details>

---

### Question 2
In the W3C Trace Context standard, what does the following `traceparent` header value represent?
`00-4bf92f3577b34da6a3ce929d0e0e4736-00f067aa0ba902b7-01`

A) A Kubernetes service account token.  
B) Version `00`, Trace ID `4bf92f3577b34da6a3ce929d0e0e4736` (identifying the entire distributed transaction), Parent Span ID `00f067aa0ba902b7`, and Trace Flags `01` (indicating the trace was sampled).  
C) An encrypted customer password.  
D) An IPv6 address and MAC address.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** B  
**Explanation:** W3C Trace Context standardizes header propagation across diverse languages and vendors: `version (2 hex) - trace_id (32 hex) - parent_id (16 hex) - trace_flags (2 hex)`. Flag `01` signifies the trace was recorded.
</details>

---

### Question 3
What is the primary operational advantage of **Tail-Based Sampling** over **Head-Based Sampling** in distributed tracing?

A) Head-based sampling makes random sampling decisions at the root before errors happen, often discarding failed traces; Tail-based sampling buffers spans until the trace completes, guaranteeing that 100% of traces containing HTTP 5xx errors or high latency are retained while boring successful traces are sampled down.  
B) Tail-based sampling requires no collector.  
C) Head-based sampling is incompatible with OpenTelemetry.  
D) Tail-based sampling consumes zero CPU.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** A  
**Explanation:** With head-based sampling, an error occurring on hop 4 of a 5-hop request will be missed if the root chose not to sample that request. Tail-based sampling evaluates the complete trace outcome before deciding whether to persist or drop it.
</details>

---

### Question 4
An engineering team logs user activity to Grafana Loki. A developer adds `user_id` as an extracted stream label in the logging daemon configuration. Shortly after, Loki crashes with OOM and queries fail. What happened?

A) The developer accidentally deleted the Loki database.  
B) Loki does not support strings.  
C) Loki indexes streams by unique label combinations; adding `user_id` introduced massive cardinality, generating millions of independent streams and exhausting Loki's in-memory index.  
D) The log message exceeded 1KB.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** C  
**Explanation:** Loki is intentionally designed as an index-free log engine: it only indexes metadata streams, keeping costs low. High-cardinality fields must never be extracted as stream labels; they should remain inside the un-indexed log line text and be searched using line filter queries.
</details>

---

### Question 5
What is the function of the **Batch Processor** in an OpenTelemetry Collector?

A) It reboots worker nodes in batches.  
B) It deletes old log files from disk.  
C) It compiles Go code into binaries.  
D) It groups individual spans, metrics, or log records into larger network batches before exporting, reducing HTTP/gRPC connection overhead and significantly increasing throughput.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** D  
**Explanation:** Sending individual telemetry records over the wire generates massive network overhead. The batch processor buffers records until either `send_batch_size` or `timeout` is reached, exporting them in bulk.
</details>

---

### Question 6
In distributed tracing, what is **Baggage**?

A) Physical storage attached to server racks.  
B) Corrupted trace spans that cannot be parsed.  
C) Deprecated code left behind in repositories.  
D) A mechanism in OpenTelemetry to propagate user-defined key-value pairs (e.g., `account_tier: enterprise`) across process boundaries within the in-band trace context, making the metadata accessible to all downstream microservices.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** D  
**Explanation:** Unlike trace attributes (which attach to a single span), Baggage items travel along with the request context across HTTP/gRPC boundaries, allowing downstream services to read context without querying external databases.
</details>

---

### Question 7
What is the primary difference between **Structured Logging (JSON)** and unstructured plaintext logging?

A) Structured logs cannot be printed to terminal.  
B) Unstructured logging is encrypted.  
C) Structured logging encodes logs as key-value pairs (JSON), allowing log aggregators (Loki, Elasticsearch, ClickHouse) to parse, filter, and extract fields efficiently without brittle regular expressions.  
D) Plaintext logs consume zero bytes on disk.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** C  
**Explanation:** Plaintext strings (`"User 123 failed login from IP 1.2.3.4"`) require complex regex parsing. Structured JSON (`{"event": "login_failed", "user_id": 123, "ip": "1.2.3.4"}`) allows log aggregation engines to filter and aggregate metrics instantly.
</details>

---

### Question 8
What is the purpose of the **OTLP (OpenTelemetry Protocol)**?

A) A modern, vendor-neutral standard for transmitting metrics, logs, and traces using Protocol Buffers over gRPC (port 4317) or HTTP (port 4318) with built-in compression and backpressure.  
B) A legacy proprietary protocol owned by Oracle.  
C) A protocol for streaming video games.  
D) An operating system file system.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** A  
**Explanation:** OTLP provides a single unified open standard for telemetry data. Applications instrumented with OpenTelemetry export via OTLP, allowing platform teams to switch backend vendors without altering application code.
</details>

---

### Question 9
In Kubernetes logging, why is **Vector** or **Fluentbit** deployed as a DaemonSet rather than running as a sidecar container in every single application Pod?

A) DaemonSets can only run on Linux.  
B) A DaemonSet runs one agent per worker node, reading log files directly from `/var/log/pods/`, consuming minimal node memory; deploying a logging sidecar in every pod duplicates agents thousands of times, wasting gigabytes of RAM.  
C) Kubernetes does not allow sidecar containers.  
D) Sidecars cannot read log files.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** B  
**Explanation:** DaemonSets are orders of magnitude more resource-efficient: 1 agent per 64-core node vs 50 sidecars on the same node. Node-level agents stream container stdout directly from host filesystem mounts.
</details>

---

### Question 10
What does the OpenTelemetry **Transform Processor** allow platform engineers to accomplish?

A) Transform Python code into Rust.  
B) Convert Prometheus metrics into audio waves.  
C) Mutate, filter, enrich, or redact telemetry attributes in transit (e.g., masking credit card numbers or removing sensitive headers) using declarative OpenTelemetry Transformation Language (OTTL) statements.  
D) Automatically resize Kubernetes clusters.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** C  
**Explanation:** The transform processor uses OTTL to perform in-flight manipulation of telemetry data: dropping PII attributes, renaming metrics, or synthesizing new labels before data reaches external storage.
</details>
