# PE Domain 2: Linux & Networking — Scenario Quiz 1

Test your mastery of processes, signals, and Linux memory management.

---

### Question 1
A containerized Node.js application is deployed to Kubernetes. When `kubectl delete pod` is executed, the pod takes exactly 30 seconds to terminate, causing rolling updates to proceed very slowly. The Dockerfile contains:
```dockerfile
FROM node:20
WORKDIR /app
COPY . .
ENTRYPOINT npm start
```
What is the root cause of the 30-second termination delay?

A) Kubernetes node network latency.  
B) Node.js applications are immune to Linux signals.  
C) The shell form of `ENTRYPOINT` spawned `/bin/sh -c "npm start"` as PID 1, which does not forward `SIGTERM` to the child Node.js process; Kubernetes eventually timed out and sent `SIGKILL`.  
D) The application has a disk I/O bottleneck during shutdown.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** C  
**Explanation:** When using the shell form (`ENTRYPOINT command`), the container runtime runs `/bin/sh -c`, making `/bin/sh` PID 1. Standard shells do not forward signals to child processes. When Kubernetes sends `SIGTERM`, the shell ignores it, the 30-second `terminationGracePeriodSeconds` expires, and Kubernetes forcibly terminates the container with `SIGKILL`. The solution is using the JSON exec array syntax (`ENTRYPOINT ["npm", "start"]`) or an init wrapper like `tini`.
</details>

---

### Question 2
Under Linux virtual memory management, what is the primary difference between **Anonymous Memory** and **File-Backed Memory** during physical memory pressure?

A) Anonymous memory can be discarded immediately without writing to disk; file-backed memory cannot.  
B) Anonymous memory is only used by root processes.  
C) File-backed memory is never counted towards container cgroup memory limits.  
D) File-backed memory (page cache, binary executables) can be dropped directly from RAM because it can be re-read from disk; anonymous memory (heap, stack) cannot be dropped unless swapped out.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** D  
**Explanation:** File-backed pages reflect data stored on non-volatile disks (such as executable code or cached files). If the kernel needs RAM, it can discard clean file-backed pages immediately because it can reload them from disk. In contrast, anonymous memory has no underlying file storage, so it must stay in RAM unless swap space is active.
</details>

---

### Question 3
A Kubernetes worker node has 16GB of RAM. Three Pods are running on the node:
- Pod A: `Guaranteed` QoS (`requests.memory = 4Gi`, `limits.memory = 4Gi`)
- Pod B: `Burstable` QoS (`requests.memory = 2Gi`, `limits.memory = 8Gi`, currently using 7.5Gi)
- Pod C: `BestEffort` QoS (no requests or limits specified, currently using 500Mi)
If the node suddenly encounters severe physical memory exhaustion, which Pod will the Linux kernel OOM killer terminate first?

A) Pod A  
B) Pod B  
C) All three simultaneously  
D) Pod C  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** D  
**Explanation:** Kubernetes assigns `/proc/[pid]/oom_score_adj` based on QoS class. `BestEffort` pods receive `oom_score_adj = 1000`, making their OOM score maximum (~1000), meaning the kernel terminates them first. `Burstable` pods have dynamically calculated scores based on memory usage relative to limits, and `Guaranteed` pods receive `-997`, making them protected.
</details>

---

### Question 4
A container exits unexpectedly. Running `kubectl describe pod` reveals:
```text
State:          Terminated
  Reason:       OOMKilled
  Exit Code:    137
```
What does the exit code `137` specifically signify?

A) The process crashed with a segmentation fault.  
B) The process exited cleanly with a custom return code.  
C) The process received `SIGKILL` ($128 + 9 = 137$), typically sent by the Linux kernel OOM killer when the container exceeded its cgroup memory limit.  
D) Port 137 (NetBIOS) was blocked by a firewall.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** C  
**Explanation:** In Unix, when a process is killed by an unhandled signal, its exit status is $128 + \text{Signal Number}$. Signal 9 is `SIGKILL`, resulting in $128 + 9 = 137$.
</details>

---

### Question 5
What is the purpose of the `Copy-On-Write (COW)` mechanism during a Linux `fork()` system call?

A) To duplicate all physical RAM pages immediately to ensure complete isolation.  
B) To share physical memory pages between parent and child read-only, allocating a new physical page only when one of the processes attempts to write to it, making process creation fast and memory-efficient.  
C) To compress application data before writing to disk.  
D) To prevent multi-threaded applications from using CPU caches.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** B  
**Explanation:** COW optimizes process creation by duplicating only page table pointers rather than physical memory. Physical memory pages are marked read-only; only when either the parent or child writes to a page does the CPU trigger a page fault, prompting the kernel to allocate a private copy of that page.
</details>

---

### Question 6
You run `ps aux` inside a Linux container and notice several processes listed with state `Z` (`[defunct]`). What are these processes, and why do they exist?

A) They are processes stopped by `SIGSTOP`.  
B) They are sleeping threads waiting for disk I/O.  
C) They are malicious malware processes encrypting files.  
D) They are zombie processes that have terminated execution, but their parent process has failed to call `waitpid()` to read their exit status and allow the kernel to free their process table entry.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** D  
**Explanation:** A zombie process has already finished executing and released its memory, but its entry remains in the OS process table so its parent can inspect its exit code via `wait()` or `waitpid()`. If the parent neglects to reap them, zombies accumulate, eventually exhausting the kernel's PID table (`/proc/sys/kernel/pid_max`).
</details>

---

### Question 7
Which system call is the fundamental primitive used by container runtimes to create a new process with isolated Linux namespaces (such as `CLONE_NEWPID` and `CLONE_NEWNET`)?

A) `clone()`  
B) `ptrace()`  
C) `kill()`  
D) `execve()`  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** A  
**Explanation:** The `clone()` system call extends `fork()` by accepting flags (e.g., `CLONE_NEWPID`, `CLONE_NEWNET`, `CLONE_NEWNS`, `CLONE_NEWIPC`) that instruct the Linux kernel to create new namespaces for the child process.
</details>

---

### Question 8
What happens if a process attempts to register an explicit signal handler for `SIGKILL` (signal 9) or `SIGSTOP` (signal 19)?

A) The signal is converted into a `SIGHUP`.  
B) The process immediately core dumps.  
C) The kernel rejects the attempt; `SIGKILL` and `SIGSTOP` cannot be caught, blocked, or ignored.  
D) The kernel accepts the handler and allows the process to intercept the signal.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** C  
**Explanation:** By design in POSIX and the Linux kernel, `SIGKILL` and `SIGSTOP` can never be intercepted, caught, or ignored, guaranteeing that system administrators can unconditionally terminate or suspend rogue processes.
</details>

---

### Question 9
In Linux, what is the significance of the `PR_SET_CHILD_SUBREAPER` prctl option?

A) It limits process memory usage to 1GB.  
B) It designates a process as a "subreaper", so that when child processes in its subtree become orphaned, they are reparented to this process rather than PID 1.  
C) It grants a process root privileges.  
D) It prevents a process from opening network sockets.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** B  
**Explanation:** `PR_SET_CHILD_SUBREAPER` allows process supervisors (like systemd, tmux, or container managers) to adopt orphaned descendant processes instead of letting them reparent all the way to PID 1, keeping zombie reaping localized.
</details>

---

### Question 10
Under cgroups v2, what does **Pressure Stall Information (PSI)** provide that was missing in cgroups v1?

A) It provides real-time metrics (`some` and `full` stall time) quantifying the percentage of time tasks are delayed waiting for CPU, memory, or I/O, allowing platforms to detect resource starvation *before* an OOM kill occurs.  
B) It encrypts network traffic between cgroups.  
C) It replaces the Completely Fair Scheduler with a FIFO queue.  
D) It automatically resizes AWS EC2 instances.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** A  
**Explanation:** PSI is a landmark cgroups v2 feature that measures resource contention. It reports the percentage of wall-clock time that processes were stalled waiting for CPU, memory page reclamation, or disk I/O, enabling proactive auto-scaling or pod eviction before catastrophic OOM kills.
</details>
