# PE Domain 2: Linux & Networking — Hands-On Labs & Exercises

Six comprehensive labs covering Linux kernel subsystems, container isolation primitives, packet tracing, and network troubleshooting.

---

## Lab 1: Triggering & Investigating the Linux OOM Killer with cgroups v2

### Objective
Create a dedicated cgroup v2 with strict memory limits, spawn an aggressive memory-allocating C/Python process, observe page cache depletion, capture the kernel OOM killer invocation in `dmesg`, and verify the exit code.

### Step-by-Step Implementation

1. **Verify cgroups v2 mount:**
   ```bash
   mount -t cgroup2
   ```

2. **Create a test cgroup directory:**
   ```bash
   sudo mkdir -p /sys/fs/cgroup/oom_test
   # Set memory max limit to 50MB
   echo "50M" | sudo tee /sys/fs/cgroup/oom_test/memory.max
   # Disable swap for this test to force immediate OOM
   echo "0" | sudo tee /sys/fs/cgroup/oom_test/memory.swap.max
   ```

3. **Create the Memory Hog Script (`memory_eater.py`):**
   ```python
   #!/usr/bin/env python3
   import time
   import sys

   print("Allocating memory rapidly...")
   blocks = []
   try:
       while True:
           # Allocate 10MB chunks and write data to force resident memory
           blocks.append(bytearray(10 * 1024 * 1024))
           print(f"Allocated {len(blocks) * 10} MB")
           time.sleep(0.1)
   except MemoryError:
       print("MemoryError caught!")
   ```

4. **Attach and Execute:**
   ```bash
   # Add current shell PID to the cgroup
   echo $$ | sudo tee /sys/fs/cgroup/oom_test/cgroup.procs
   python3 memory_eater.py
   echo "Exit status: $?"
   ```

### Verification & Analysis
```bash
# Check dmesg for kernel OOM kill log
sudo dmesg -T | grep -E -i "killed process|oom_score_adj|invoked oom-killer" | tail -n 15

# Clean up
sudo rmdir /sys/fs/cgroup/oom_test
```
Observe that the process terminates with status code `137` ($128 + 9 = \text{SIGKILL}$).

---

## Lab 2: Building Container Isolation from Scratch Using Linux Namespaces & Veth Pairs

### Objective
Construct a fully isolated container environment manually using `unshare`, mount namespaces, pivot_root, and configure network communication using a virtual ethernet (`veth`) pair and IP routing.

### Step-by-Step Implementation

1. **Create an isolated root filesystem:**
   ```bash
   mkdir -p /tmp/mycontainer/rootfs
   docker export $(docker create alpine:latest) | tar -C /tmp/mycontainer/rootfs -xvf -
   ```

2. **Launch an Isolated Namespace Session:**
   ```bash
   sudo unshare --mount --uts --ipc --net --pid --fork /bin/sh
   ```

3. **In a separate host terminal, create the veth pair:**
   ```bash
   CONTAINER_PID=$(pgrep -f "unshare.*mycontainer" | tail -n 1)
   
   # Create veth pair
   sudo ip link add veth-host type veth peer name veth-guest
   # Move guest end into container netns
   sudo ip link set veth-guest netns $CONTAINER_PID
   
   # Configure host end
   sudo ip addr add 10.200.1.1/24 dev veth-host
   sudo ip link set veth-host up
   ```

4. **Inside the container namespace:**
   ```bash
   ip link set lo up
   ip link set veth-guest name eth0
   ip addr add 10.200.1.2/24 dev eth0
   ip link set eth0 up
   ip route add default via 10.200.1.1
   
   # Ping the host
   ping -c 3 10.200.1.1
   ```

---

## Lab 3: Debugging the Kubernetes `ndots:5` DNS Latency Penalty

### Objective
Capture raw DNS queries using `tcpdump` while executing HTTP requests, measure the exact latency penalty caused by `ndots:5`, and resolve it via trailing dot FQDNs.

### Step-by-Step Implementation

1. **Start tcpdump monitoring DNS traffic (port 53):**
   ```bash
   sudo tcpdump -i any -nn -vvv port 53 -w /tmp/dns_trace.pcap &
   TCPDUMP_PID=$!
   ```

2. **Simulate a standard non-FQDN curl lookup:**
   ```bash
   curl -s -o /dev/null -w "Time DNS: %{time_namelookup}s\nTotal: %{time_total}s\n" https://api.github.com
   ```

3. **Simulate an explicit trailing-dot FQDN curl lookup:**
   ```bash
   curl -s -o /dev/null -w "Time DNS: %{time_namelookup}s\nTotal: %{time_total}s\n" https://api.github.com.
   ```

4. **Stop capture and inspect queries:**
   ```bash
   kill $TCPDUMP_PID
   tcpdump -r /tmp/dns_trace.pcap -nn
   ```

Observe how the first command issued multiple failed search-path queries, while the trailing-dot query resolved on the first packet.

---

## Lab 4: Diagnosing TCP Socket Exhaustion, TIME_WAIT, and CLOSE_WAIT Leaks

### Objective
Write a Python script that produces socket leaks (`CLOSE_WAIT` and `TIME_WAIT`), inspect the socket table using `ss`, and analyze kernel socket recycling parameters.

### Implementation Script (`socket_drill.py`)
```python
#!/usr/bin/env python3
import socket
import time
import threading

def run_server():
    server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    server.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
    server.bind(("127.0.0.1", 9999))
    server.listen(128)
    print("Server listening on 9999...")
    while True:
        conn, addr = server.accept()
        # Intentional CLOSE_WAIT leak: read data, but NEVER call conn.close()
        data = conn.recv(1024)
        print("Received request; leaving socket open to simulate CLOSE_WAIT")

def run_client():
    time.sleep(1)
    for i in range(5):
        s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        s.connect(("127.0.0.1", 9999))
        s.send(b"ping")
        s.close() # Client sends FIN, enters TIME_WAIT
        print(f"Client {i} closed connection")

if __name__ == "__main__":
    t = threading.Thread(target=run_server, daemon=True)
    t.start()
    run_client()
    time.sleep(2)
```

### Verification
Execute the script and inspect socket states:
```bash
python3 socket_drill.py &
PID=$!
sleep 3
# Inspect socket states
ss -tan '( sport = :9999 or dport = :9999 )'
kill $PID
```
Observe the presence of sockets in `CLOSE_WAIT` on the server and `TIME_WAIT` on the client.

---

## Lab 5: Tracing Kernel Packet Traversal & Drop Points Using `iptables-trace`

### Objective
Trace an inbound packet traversing Linux Netfilter chains (`PREROUTING`, `FORWARD`, `POSTROUTING`) using iptables `TRACE` targets and `dmesg`.

### Step-by-Step Implementation

1. **Enable Raw Trace Target:**
   ```bash
   sudo iptables -t raw -A PREROUTING -p icmp -j TRACE
   sudo iptables -t raw -A OUTPUT -p icmp -j TRACE
   ```

2. **Send test ICMP ping:**
   ```bash
   ping -c 1 127.0.0.1
   ```

3. **Inspect the trace log:**
   ```bash
   sudo dmesg | grep "TRACE:" | tail -n 20
   ```

4. **Clean up iptables rules:**
   ```bash
   sudo iptables -t raw -D PREROUTING -p icmp -j TRACE
   sudo iptables -t raw -D OUTPUT -p icmp -j TRACE
   ```

---

## Lab 6: Deep-Dive TLS Handshake Analysis with OpenSSL CLI

### Objective
Inspect the cryptographic parameters of a live TLS 1.3 connection, verify ALPN protocol negotiation (HTTP/2 vs HTTP/1.1), inspect certificate chains, and test cipher suite restrictions.

### Step-by-Step Implementation

1. **Perform TLS 1.3 Handshake Inspection:**
   ```bash
   openssl s_client -connect google.com:443 -tls1_3 -alpn h2,http/1.1 </dev/null
   ```

2. **Verify ALPN Negotiation:**
   Look for:
   `ALPN protocol: h2`
   `Protocol  : TLSv1.3`
   `Cipher    : TLS_AES_256_GCM_SHA384`

3. **Verify Certificate Chain Expiry Dates:**
   ```bash
   echo | openssl s_client -connect google.com:443 2>/dev/null | openssl x509 -noout -dates -subject -issuer
   ```
