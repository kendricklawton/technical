# Mock Interview 2: Live Troubleshooting & Incident Response

Simulates a 60-minute interactive live troubleshooting incident. The candidate asks for commands, metrics, and logs; the interviewer provides terminal output.

---

## Candidate Brief (Safe to read in advance)

- **Duration:** 60 minutes.
- **Format:** Simulated production outage. You are the on-call platform engineer. You must diagnose, mitigate, and resolve a high-severity production incident by proposing commands and interpreting terminal outputs.
- **Core Principles:**
  1. *Mitigate First:* Stop customer bleeding before spending hours diagnosing root causes.
  2. *Hypothesis-Driven:* State *why* you are running a command and what you expect to observe.
  3. *Calm Communication:* Keep stakeholders updated; avoid erratic panic commands.

---

## Interviewer Packet (Confidential — Do not read beforehand)

### Variant A: The Intermittent 502 Outage
- **Scenario:** At 14:00 UTC, the customer checkout API starts throwing intermittent HTTP 502 Bad Gateway errors (15% error rate).
- **Hidden System State:**
  - Ingress: AWS ALB -> Envoy Ingress -> Service `checkout-api` (Go app).
  - ALB idle timeout is set to 60 seconds.
  - The Go backend `http.Server` idle keepalive timeout is set to 55 seconds.
  - When the backend closes an idle TCP connection right as the ALB forwards a request, the ALB encounters a closed socket and returns 502.
- **Terminal Responses to Provide:**
  - `kubectl get pods -n checkout`: All 10 pods Running, 0 restarts.
  - `kubectl logs -n checkout deployment/checkout-api`: No panics, normal 200 OK logs.
  - `kubectl top pods -n checkout`: CPU 15%, Memory 40% (no throttling).
  - `alb-access-logs`: `elb_status_code=502`, `target_status_code=-`, `request_processing_time=0.001`, `target_processing_time=-1`.
- **Winning Mitigation:** Increase Go server keepalive timeout to 65s (strictly higher than ALB 60s) or enable connection retries in Envoy.

### Variant B: The Silent Pod CrashLoopBackOff & OOM
- **Scenario:** A core payment batch processing deployment crashes with `CrashLoopBackOff` during peak traffic.
- **Hidden System State:**
  - Pod has `limits.memory: 500Mi`.
  - Application caches incoming transaction batches in an unbounded in-memory map without eviction.
  - Kernel OOM killer terminates the pod with exit code 137.
- **Terminal Responses to Provide:**
  - `kubectl describe pod <pod>`: `Last State: Terminated, Reason: OOMKilled, Exit Code: 137`.
  - `dmesg -T`: `Memory cgroup out of memory: Killed process 2411 (payment-batch) total-vm:812MB, anon-rss:498MB`.
- **Winning Mitigation:** Increase memory limit to 2Gi immediately to restore production batch processing, followed by deploying an LRU cache with eviction limits in code.

### Variant C: The DNS Resolution Blackout
- **Scenario:** Suddenly, all services across an entire EKS cluster report timeout errors when connecting to AWS RDS databases and external payment APIs.
- **Hidden System State:**
  - A developer applied a malformed CoreDNS ConfigMap with a syntax error, causing CoreDNS pods to enter `CrashLoopBackOff`.
  - Pods cannot resolve internal or external domain names.
- **Terminal Responses to Provide:**
  - `kubectl get pods -n kube-system -l k8s-app=kube-dns`: 2/2 pods in `CrashLoopBackOff`.
  - `kubectl logs -n kube-system -l k8s-app=kube-dns`: `plugin/errors: Corefile:12: unknown option 'caching_ttl'`.
- **Winning Mitigation:** Roll back the CoreDNS ConfigMap (`kubectl rollout undo deployment/coredns -n kube-system`) within 5 minutes.

---

## Scoring Rubric

| Score | Anchor | Behavioral Criteria |
| :---: | :--- | :--- |
| **1** | Strong No Hire | Random trial-and-error commands; ignores mitigation; panics. |
| **2** | No Hire | Identifies the bug eventually but takes 50 minutes; never proposes temporary mitigation. |
| **3** | Hire | Structured hypothesis-driven debugging; mitigates customer impact in <20 min; articulates root cause. |
| **4** | Strong Hire | Flawless calm leadership; requests exact diagnostic commands; formulates long-term preventative guardrails. |

---

## Self-Debrief Template
- **Elapsed Time to Mitigation:** ___ min
- **Elapsed Time to Root Cause:** ___ min
- **Commands Proposed:**
- **Key Learnings:**
