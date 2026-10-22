# Staff SRE & Platform Engineering — Production Incident War Room & Post-Mortems

A collection of five production-grade incident post-mortems based on real-world large-scale distributed systems outages. Each case study documents the end-to-end incident lifecycle: initial trigger, telemetry signals, hypothesis testing, immediate mitigation, five whys root-cause analysis, and permanent architectural remediation.

---

## Post-Mortem 1: Conntrack Table Saturation & DNS Flapping Outage
* **Severity:** SEV-1 (Global User-Facing Outage)  
* **Duration:** 42 Minutes  
* **Impact:** 68% of API requests returning HTTP 504 Gateway Timeout; global checkout pipeline stalled.

### 1. Architectural Topology
- 120-node Kubernetes cluster running across 3 AWS Availability Zones.
- Microservices communicating over ClusterIP Services proxied via iptables kube-proxy.
- CoreDNS deployed as a 4-replica Deployment using standard UDP 53 resolution.

### 2. Timeline of Events
- **14:02 UTC:** A marketing push notification triggers a 6x surge in mobile app checkouts (traffic jumps from 15,000 to 95,000 req/sec).
- **14:05 UTC:** Checkout API p99 latency degrades from 45 ms to 2,800 ms. CloudWatch reports spikes in 504 Gateway Timeouts at the Application Load Balancer.
- **14:08 UTC:** On-call SRE paged for `CheckoutServiceErrorBudgetBurnRateHigh` (14.4x burn rate over 1h window).
- **14:12 UTC:** Initial hypothesis: Database write connection saturation. SRE checks RDS CPU and connection pools; Aurora PostgreSQL CPU is at 28%, connection pool has 85% headroom. Hypothesis falsified.
- **14:18 UTC:** SRE inspects pod logs: `dial tcp: lookup auth.prod.svc.cluster.local on 10.96.0.10:53: read udp 10.244.3.18:41235->10.96.0.10:53: i/o timeout`.
- **14:23 UTC:** CoreDNS pods checked: CoreDNS CPU is under 40%, memory stable. However, `kubectl top nodes` reveals worker nodes dropping packets.
- **14:26 UTC:** SRE SSHs into worker node `worker-prod-az1-42` and runs `dmesg -T`:
  ```text
  [14:24:12.102341] nf_conntrack: table full, dropping packet
  [14:24:12.103512] nf_conntrack: table full, dropping packet
  ```
- **14:28 UTC (Mitigation):** SRE runs emergency kernel parameter tuning across all worker nodes via Systems Manager Run Command:
  ```bash
  sysctl -w net.netfilter.nf_conntrack_max=1048576
  sysctl -w net.netfilter.nf_conntrack_tcp_timeout_established=600
  ```
- **14:32 UTC:** Packet drops immediately halt. DNS resolution recovers. P99 latency returns to 42 ms.
- **14:44 UTC:** Incident declared resolved.

### 3. Root Cause Analysis (5 Whys)
1. *Why did checkout requests time out?*  
   Microservice pods could not resolve internal DNS names to reach downstream auth and payment endpoints.
2. *Why did DNS requests time out?*  
   UDP packets between worker node pods and CoreDNS were dropped silently by the Linux kernel.
3. *Why did the kernel drop packets?*  
   The Netfilter connection tracking table (`nf_conntrack`) reached its hard limit of 262,144 entries.
4. *Why was the conntrack table saturated?*  
   Each DNS lookup created a new UDP tracking entry, and glibc's `ndots:5` search path generated 5 sequential DNS queries for every single internal microservice call, multiplying DNS connection entries by 5x under traffic surge.
5. *Why was NodeLocal DNSCache not present?*  
   Cluster was built using baseline kubeadm defaults without configuring local DNS caching daemons.

### 4. Permanent Architectural Remediations
1. **Deploy NodeLocal DNSCache:** Deploy `node-local-dns` DaemonSet on every node to cache DNS queries locally via a link-local IP (`169.254.20.10`), keeping DNS queries off the iptables conntrack engine.
2. **Optimize `ndots`:** Update deployment pod specs with `dnsConfig.options: [{name: "ndots", value: "2"}]` to eliminate recursive search domain traversal.
3. **Automated Conntrack Headroom Alert:** Deploy Prometheus alert triggering when `node_nf_conntrack_entries / node_nf_conntrack_entries_limit > 0.75`.

---

## Post-Mortem 2: etcd Disk Fsync Latency Starvation & Split-Brain Quorum Loss
* **Severity:** SEV-1 (Total Control Plane Failure)  
* **Duration:** 58 Minutes  
* **Impact:** Kubernetes API server completely unresponsive cluster-wide; no deployments, rollouts, or autoscaling possible.

### 1. Architectural Topology
- 3-node stacked etcd control plane deployed on AWS EC2 `m5.2xlarge` instances across 3 AZs.
- etcd data directory `/var/lib/etcd` hosted on standard General Purpose SSD (`gp2`) EBS volumes.

### 2. Timeline of Events
- **09:15 UTC:** Scheduled cron job triggers 80 concurrent database backup snapshots and Docker image builds on control plane nodes.
- **09:18 UTC:** `kubectl` CLI commands begin hanging with `ETIMEDOUT`.
- **09:22 UTC:** Kubelet nodes begin flapping between `Ready` and `NotReady`. Controller manager reports leader election failures.
- **09:28 UTC:** SRE inspects etcd logs on control plane node 1:
  ```text
  wal: sync duration of 2.152341253s was too high, expected < 10ms
  etcdserver: failed to send out heartbeat on time (exceeded 100ms)
  raft: 8a7c293 lost leader 4f128bc, starting new election
  ```
- **09:35 UTC:** Due to fsync disk starvation, all 3 etcd nodes simultaneously timed out on Raft heartbeats, continually electing new leaders and dropping active client connections before any transaction could commit.
- **09:42 UTC (Mitigation):** SRE stopped the heavy backup cron jobs.
- **09:48 UTC:** SRE detached the EBS volumes and upgraded volume types from `gp2` to `gp3` with provisioned IOPS = 10,000 and Throughput = 500 MB/s.
- **10:13 UTC:** etcd WAL sync latency dropped to 1.8 ms. Raft leader stabilized. API server recovered.

### 3. Root Cause Analysis
etcd is extremely sensitive to disk write latency because every write transaction must be committed to the Write-Ahead Log (WAL) via `fsync()` before being committed to memory. Running noisy-neighbor disk I/O on the same volume starved etcd of write IOPS, driving fsync latency from $<5$ ms to $>2,000$ ms, triggering catastrophic Raft leader churn.

### 4. Permanent Architectural Remediations
1. **Dedicated High-Speed NVMe Storage:** Mount `/var/lib/etcd` on dedicated, local NVMe SSDs or high-IOPS EBS `io2` / `gp3` volumes with guaranteed minimum 3,000 IOPS and dedicated controllers.
2. **Process Priority:** Set systemd nice value `Nice=-20` and IO scheduling priority `IONice=realtime` for the etcd service.
3. **Strict WAL Latency Alert:** Set Prometheus alert on `histogram_quantile(0.99, rate(etcd_disk_wal_fsync_duration_seconds_bucket[5m])) > 0.01` (alert if p99 fsync exceeds 10 ms).

---

## Post-Mortem 3: CFS CPU Bandwidth Quota Throttling & Tail Latency Explosion
* **Severity:** SEV-2 (Severe Latency Degradation)  
* **Duration:** 1 Hour 15 Minutes  
* **Impact:** P99 API latency spiked from 38 ms to 1,450 ms; average CPU utilization was only 12%.

### 1. Architectural Topology
- Java 21 Spring Boot microservice deployed on Kubernetes with Cgroups v2.
- Resource configuration: `requests: cpu: 500m, memory: 2Gi`, `limits: cpu: 1000m, memory: 4Gi`.

### 2. Timeline of Events
- **11:00 UTC:** Feature release deployed. Immediately, P99 latency spikes by 35x.
- **11:15 UTC:** SRE examines CPU metrics: Datadog shows average CPU utilization across all pods is only 12%. Autoscaler does not scale out because CPU target (70%) is not met.
- **11:28 UTC:** SRE inspects `/sys/fs/cgroup/cpu.stat` inside the container:
  ```text
  nr_periods 18542
  nr_throttled 12841
  throttled_usec 481920512
  ```
  Over 69% of all execution periods were throttled!
- **11:35 UTC (Mitigation):** SRE removes `limits.cpu` from the deployment manifest, leaving only `requests.cpu: 1000m`.
- **11:42 UTC:** Throttling drops to 0%. P99 latency immediately collapses back to 36 ms.

### 3. Root Cause Analysis
The Linux Completely Fair Scheduler (CFS) enforces CPU limits over a default period of 100 ms (`cfs_period_us = 100000`). With `limits.cpu = 1000m` (1 core), the process is allocated 100 ms of CPU runtime per 100 ms period. When Java multi-threaded garbage collection or request threads burst across 8 physical host cores, they consume 100 ms of cumulative CPU within the first 12 ms of the period. The Linux kernel throttles the container for the remaining 88 ms, causing dramatic latency spikes despite low average CPU usage.

### 4. Permanent Architectural Remediations
1. **Remove Hard CPU Limits on Latency-Sensitive Services:** Enforce accurate CPU requests (for bin-packing and scheduling) while omitting hard CPU limits, allowing pods to burst into unallocated host capacity without kernel throttling.
2. **Tune CFS Quota Period:** For environments requiring limits, decrease `cpu.cfs_period_us` to 10 ms (or 20 ms) to reduce throttle sleep durations.

---

## Post-Mortem 4: Thundering Herd & Cascading Retry Storm
* **Severity:** SEV-1 (Total Database Outage)  
* **Duration:** 35 Minutes  
* **Impact:** 100% database CPU lockup; all application instances crashing.

### 1. Architectural Topology
- 400 microservice worker pods communicating with Amazon Aurora PostgreSQL.
- Worker pods configured with standard connection pool of 20 connections per pod (8,000 total connections).

### 2. Timeline of Events
- **08:30 UTC:** Aurora primary instance undergoes an automated minor patch failover. Failover completes in 32 seconds.
- **08:31 UTC:** All 400 worker pods discover broken TCP connections simultaneously and immediately initiate reconnection attempts.
- **08:32 UTC:** 8,000 concurrent TLS handshakes and connection authentication requests hit the newly promoted primary database simultaneously.
- **08:34 UTC:** Aurora CPU hits 100% purely handling TLS handshakes and process fork overhead.
- **08:36 UTC:** Worker connection attempts time out after 5 seconds, and client retry logic immediately fires without delay, creating a continuous retry storm.
- **08:45 UTC (Mitigation):** SRE scaled worker deployments to 0 replicas to eliminate traffic, allowed database CPU to stabilize at 0%, and scaled workers back up in increments of 50 pods with exponential backoff.

### 3. Root Cause Analysis
The failure was caused by two classic anti-patterns:
1. **Lack of Exponential Backoff & Jitter:** Client retry code retried every 2 seconds on a fixed interval, hammering the recovering database.
2. **Lack of Connection Pooling Proxy:** 8,000 direct connections overwhelmed the database process table.

### 4. Permanent Architectural Remediations
1. **Deploy Amazon RDS Proxy:** Insert RDS Proxy between workers and database to multiplex client requests over a fixed pool of 300 database connections.
2. **Enforce Full Jitter Algorithm:** Mandate AWS Full Jitter in all client SDKs: `sleep = random(0, min(max_backoff, base * 2 ^ attempt))`.

---

## Post-Mortem 5: BGP Route Flapping & Asymmetric Inspection Routing
* **Severity:** SEV-1 (Hybrid Cloud Disconnect)  
* **Duration:** 48 Minutes  
* **Impact:** All traffic between on-premises headquarters and AWS cloud dropped.

### 1. Architectural Topology
- Dual 10G AWS Direct Connect links connected to AWS Transit Gateway via Direct Connect Gateway.
- Centralized Inspection VPC with Gateway Load Balancer (GWLB) and stateful third-party firewalls.

### 2. Timeline of Events
- **16:10 UTC:** On-premises network engineer modifies BGP MED on primary router.
- **16:12 UTC:** Return packets from AWS begin routing through Location B while ingress packets enter Location A.
- **16:15 UTC:** Firewalls in Inspection VPC begin dropping all TCP traffic with `TCP Reset: Invalid Sequence Number / State Not Found`.
- **16:28 UTC:** SRE discovers Transit Gateway Appliance Mode was disabled on the Inspection VPC attachment.
- **16:35 UTC (Mitigation):** Enabled Transit Gateway Appliance Mode via CLI:
  ```bash
  aws ec2 modify-transit-gateway-vpc-attachment --transit-gateway-attachment-id tgw-attach-xyz --appliance-mode-support enable
  ```
- **16:40 UTC:** Symmetric flow routing restored. Network traffic recovered immediately.

### 3. Permanent Architectural Remediations
1. **Enforce Appliance Mode via Infrastructure as Code:** Codify `appliance_mode_support = "enable"` in Terraform module definitions for all inspection VPC attachments.
2. **BGP Community Standard:** Standardize on BGP Communities (`7224:7300` / `7224:7100`) rather than MED for deterministic route selection.

---

## Post-Mortem 6: eBPF BPF Map Saturation & Pod-to-Pod Packet Blackhole
* **Severity:** SEV-1 (Internal Service Discovery & Mesh Failure)  
* **Duration:** 36 Minutes  
* **Impact:** 45% of inter-pod RPC calls failing with `connect: connection timed out` across 60 worker nodes; automated payment processing halted.

### 1. Architectural Topology
- 150-node Kubernetes cluster utilizing Cilium eBPF v1.15 in kube-proxy replacement mode.
- High-churn microservice architecture deploying 4,000+ ephemeral batch pods per hour.
- Default Cilium BPF conntrack table size: 524,288 entries (`bpf-ct-global-any-max: 524288`).

### 2. Timeline of Events
- **09:14 UTC:** A distributed data pipeline batch run initiates 12,000 parallel short-lived TCP connection requests to cache endpoints.
- **09:18 UTC:** Core payment service reports p99 internal gRPC latency climbing from 4 ms to 15,000 ms. Inter-service HTTP requests fail with connection timeouts.
- **09:22 UTC:** On-call SRE paged for `PodCommunicationDegraded`. Initial check of node CPU and RAM shows all nodes below 45% utilization.
- **09:27 UTC:** SRE runs `cilium status` on an affected node:
  ```text
  BPF Maps:       bpf_ct_global_any: 524288/524288 (100% full) [ERROR]
  BPF Errors:     Map full, dropping packets
  ```
- **09:31 UTC (Immediate Mitigation):** SRE dynamically flushes closed/stale TCP connections and expands the BPF conntrack map limit live:
  ```bash
  cilium-dbg bpf ct flush
  helm upgrade cilium cilium/cilium -n kube-system --reuse-values --set bpf.ctGlobalAnyMax=2097152
  ```
- **09:38 UTC:** BPF map usage drops to 24%. Internal RPC packet drops cease immediately; services recover.

### 3. Permanent Architectural Remediations
1. **Dynamic BPF Map Auto-Sizing:** Reconfigure Cilium to scale BPF maps dynamically based on total node memory (`bpf.mapDynamicSizeRatio: 0.005`).
2. **Aggressive TCP Conntrack GC Tuning:** Reduce `bpf-ct-timeout-regular-tcp` from 24h to 6h and set `bpf-ct-timeout-service-close` to 10 seconds.
3. **Proactive Alerting:** Deploy a Prometheus alerting rule firing at 75% BPF map capacity:
   `cilium_bpf_map_pressure{map_name="cilium_ct_any"} > 0.75`.

---

## Post-Mortem 7: Mutating Admission Webhook Deadlock & Cluster Lockout
* **Severity:** SEV-1 (Total Control Plane Deployment Freeze)  
* **Duration:** 52 Minutes  
* **Impact:** All rolling deployments, pod autoscaling (HPA), and administrative pod creations blocked across entire production cluster with HTTP 500 Internal Server Error.

### 1. Architectural Topology
- Multi-tenant Kubernetes cluster running Kyverno and custom mutating admission webhooks.
- Webhook configuration applied with `failurePolicy: Fail` and `timeoutSeconds: 10`.
- Webhook pod running inside namespace `security-system` on the same worker nodes as application workloads.

### 2. Timeline of Events
- **11:00 UTC:** Routine Kubernetes node OS security patching initiates a rolling node drain.
- **11:04 UTC:** The node hosting all 3 replicas of the custom webhook pod is drained. Due to missing `podAntiAffinity`, all webhook pods were collocated on that single node.
- **11:06 UTC:** The webhook pods transition to `Pending` because remaining nodes lack capacity.
- **11:08 UTC:** Application HPA triggers scale-out requests. Kubernetes API server intercepts pod creation requests, attempts to contact the admission webhook at `https://webhook.security-system.svc:443`, and times out after 10 seconds.
- **11:10 UTC:** Because `failurePolicy: Fail` was configured without namespace exclusion, the API server rejects ALL pod creation requests cluster-wide—including the webhook pods trying to schedule!
- **11:18 UTC:** SRE attempts to deploy an emergency recovery pod; `kubectl run` fails with `Internal error occurred: failed calling webhook`.
- **11:24 UTC (Immediate Mitigation):** SRE uses emergency administrative privileges to delete the `MutatingWebhookConfiguration`:
  ```bash
  kubectl delete mutatingwebhookconfiguration enterprise-security-webhook
  ```
- **11:28 UTC:** API server immediately unblocks pod admissions. Webhook pods schedule and start running.

### 3. Permanent Architectural Remediations
1. **Exclude Critical System Namespaces:** Update all admission webhook configurations with explicit namespace selectors:
   ```yaml
   namespaceSelector:
     matchExpressions:
       - key: kubernetes.io/metadata.name
         operator: NotIn
         values: ["kube-system", "kube-node-lease", "security-system"]
   ```
2. **Mandatory Pod Anti-Affinity & PDB:** Require all admission controllers to run at least 3 replicas across separate Availability Zones with a `PodDisruptionBudget` (`minAvailable: 2`).
3. **Audit FailurePolicy:** Switch non-blocking security checks to `failurePolicy: Ignore` and enforce compliance through asynchronous background audit scans.
