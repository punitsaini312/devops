Here's my take: **Focusing heavily on production incident response, network debugging, and standard diagnostic workflows is exactly what top-tier interviewers look for.**

While commands like `df -h` or `free -h` are good first checks, senior interviewers want to see **structured troubleshooting logic**. They want to know *why* you run a command, *what* you expect to see, and *how* you trace a failure down the stack—from the network boundary down to the application socket.

I've structured this comprehensive document focusing on **Production Down Emergency Playbooks**, **Deep Network Debugging**, and **Firewall & Traffic Inspection**.

---

# Linux Production Incident Response & Advanced Network Debugging

## Table of Contents

1. [Emergency Incident Response Playbook](https://www.google.com/search?q=%231-emergency-incident-response-playbook)
2. [Network Debugging & Connectivity Deep Dive](https://www.google.com/search?q=%232-network-debugging--connectivity-deep-dive)
3. [Firewalls, Traffic Filtering & Packet Analysis](https://www.google.com/search?q=%233-firewalls-traffic-filtering--packet-analysis)
4. [Real-World Scenario Walkthroughs](https://www.google.com/search?q=%234-real-world-scenario-walkthroughs)
5. [Interview Questions & Expected Answers](https://www.google.com/search?q=%235-interview-questions--expected-answers)
6. [One-Minute Production Cheatsheet](https://www.google.com/search?q=%236-one-minute-production-cheatsheet)

---

# 1. Emergency Incident Response Playbook

When an alert fires or an application drops offline in production, do not run random commands. Follow a structured 4-phase triage pipeline: **System Health → Process & Port → Logs → Network Connectivity**.

```text
               +----------------------------------+
               |  Phase 1: System Level Health    |
               |  (top, free -h, df -h, uptime)   |
               +----------------+-----------------+
                                |
                                v
               +----------------------------------+
               |  Phase 2: Process & Socket Check |
               |  (ps aux, ss -tulpn, lsof)       |
               +----------------+-----------------+
                                |
                                v
               +----------------------------------+
               |  Phase 3: Application Log Check  |
               |  (journalctl, tail -f, dmesg)    |
               +----------------+-----------------+
                                |
                                v
               +----------------------------------+
               |  Phase 4: Network & Firewall     |
               |  (nc, dig, traceroute, iptables) |
               +----------------------------------+

```

---

## Phase 1: High-Level System Health (First 60 Seconds)

Run these four commands immediately upon SSHing into a failing server:

```bash
# 1. Check CPU load average and overall process status
uptime

# 2. Check real-time CPU & process resource hogs
top -b -n 1 | head -n 20
# or interactively: htop

# 3. Check memory availability and swap paging
free -h

# 4. Check disk storage and inode capacity
df -h
df -i

```

### What to look for:

* **`uptime`**: Load averages (1, 5, 15 mins). If load average is significantly higher than the total CPU core count, the system is overloaded.
* **`free -h`**: Is available memory near `0MB`? Is `swap` heavily used? (High swap usage severely degrades performance).
* **`df -h` & `df -i**`: Is any partition at `100%`? Is root (`/`) or `/var/log` full?

---

## Phase 2: Process & Socket Verification

If the system has capacity, check if the application process is running and bound to its assigned port.

```bash
# Verify if application process is active (e.g., node, python, java, nginx)
pgrep -a python3
# or
ps aux | grep java

# Verify if target port (e.g., 8080) is bound and actively listening
sudo ss -tulpn | grep :8080

# Identify exact process owning the port
sudo lsof -i :8080

```

---

## Phase 3: Application & System Log Analysis

If the process is stopped or throwing 5xx errors, inspect recent logs:

```bash
# View last 100 lines and follow live logs for a systemd service
sudo journalctl -u myapp.service -n 100 -f

# Check for kernel level crashes or OOM (Out-Of-Memory) events
sudo dmesg -T | grep -i -E "oom|kill|fail"

# Inspect application error logs directly
tail -n 100 /var/log/nginx/error.log

```

---

# 2. Network Debugging & Connectivity Deep Dive

Network issues usually fall into three categories: **DNS resolution**, **Routing/Path issues**, or **Port accessibility**.

---

## Testing Layer 3 (IP) vs Layer 4 (Port) Connectivity

### 1. `ping` (ICMP Protocol)

`ping` tests basic IP-level reachability (Layer 3).

```bash
# Send 4 ICMP echo requests to target host
ping -c 4 10.0.1.50

```

> ⚠️ **Production Gotcha**: Modern cloud environments (AWS Security Groups, Azure NSGs) frequently block ICMP traffic by default. **A failing `ping` DOES NOT mean the web application is down.** Always test at Layer 4 (TCP).

---

### 2. `nc` (Netcat) — The Swiss Army Knife

`nc` tests actual TCP/UDP port connectivity (Layer 4) without sending HTTP payloads.

```bash
# Test if remote host is listening on port 5432 (PostgreSQL) with timeout
nc -zv -w 5 192.168.1.100 5432

# Breakdown of flags:
# -z : Zero-I/O mode (Scan for listening daemons without sending data)
# -v : Verbose output
# -w 5 : Connection timeout of 5 seconds

```

#### Output Interpretation:

* `Connection to 192.168.1.100 5432 port [tcp/postgresql] succeeded!` → Port is **open** and reachable.
* `nc: connect to 192.168.1.100 port 5432 (tcp) failed: Connection refused` → Server reached, but **no process is listening** on port 5432.
* `nc: connect to 192.168.1.100 port 5432 (tcp) timed out` → **Firewall blocking traffic** or packet dropped.

---

### 3. Tracing Route & Network Hops (`traceroute` & `mtr`)

When traffic drops intermittently or suffers high latency, trace the network path.

```bash
# Standard UDP traceroute
traceroute example.com

# TCP Traceroute (Bypasses ICMP/UDP blocks on enterprise firewalls)
traceroute -T -p 443 example.com

# Modern interactive route analysis (Combines ping + traceroute)
mtr --report --report-cycles 10 example.com

```

---

## DNS Troubleshooting (`dig` & `nslookup`)

When applications fail with `Name or service not known`:

```bash
# Query DNS A record
dig api.example.com

# Short answer mode
dig api.example.com +short

# Query using a specific DNS resolver (e.g., Google 8.8.8.8) to bypass local cache
dig @8.8.8.8 api.example.com

# Inspect reverse DNS lookup (PTR record)
dig -x 192.168.1.100 +short

# Trace full DNS resolution hierarchy from root servers down
dig api.example.com +trace

```

---

# 3. Firewalls, Traffic Filtering & Packet Analysis

## Local Firewall Debugging

### UFW (Debian / Ubuntu)

```bash
# Check status and listed rules
sudo ufw status numbered

# Temporarily allow incoming traffic on port 8080
sudo ufw allow 8080/tcp

# Delete a specific rule by number
sudo ufw delete 3

```

### Firewalld (RHEL / Rocky / CentOS)

```bash
# Check active zones and rules
sudo firewall-cmd --get-active-zones
sudo firewall-cmd --list-all

# Add temporary port rule
sudo firewall-cmd --add-port=8080/tcp

```

### Native IPTables

```bash
# List all active rules with line numbers and packet counters
sudo iptables -L -n -v --line-numbers

# Check if a specific port is being dropped
sudo iptables -L INPUT -n -v | grep 8080

```

---

## Packet Inspection with `tcpdump`

When services communicate but payloads fail, inspect raw network packets.

```bash
# Capture packets on eth0 interface filtering for port 80 or 443
sudo tcpdump -i eth0 port 80 or port 443 -n

# Capture traffic between local machine and specific remote IP
sudo tcpdump -i any host 10.0.1.50 -n

# Capture and print HTTP packet contents in ASCII mode
sudo tcpdump -i eth0 -A -s 0 'tcp port 80 and (((ip[20:2] - ((ip[0]&0xf)<<2)) - ((tcp[12]&0xf0)>>2)) != 0)'

# Save output to PCAP file for inspection in Wireshark
sudo tcpdump -i eth0 -w /tmp/traffic.pcap port 443

```

---

# 4. Real-World Scenario Walkthroughs

### Scenario 1: "App is down" (502 Bad Gateway on Nginx)

**Goal**: Determine if Nginx is broken or if the upstream application (e.g., Python/Node on port 5000) crashed.

1. **Check Nginx status**:
```bash
sudo systemctl status nginx

```


2. **Inspect Nginx error log**:
```bash
sudo tail -n 20 /var/log/nginx/error.log

```


*Log output*: `connect() failed (111: Connection refused) while connecting to upstream`
3. **Verify upstream process**:
```bash
ps aux | grep app.py

```


4. **Verify if port 5000 is listening**:
```bash
sudo ss -tulpn | grep :5000

```


5. **Resolution**: If process died, check application logs (`journalctl -u myapp`) and restart service.

---

### Scenario 2: "Database Connection Timeout"

**Goal**: Troubleshoot why App Server A cannot connect to DB Server B (10.0.2.20:5432).

1. **Test basic IP reachability**:
```bash
ping -c 2 10.0.2.20

```


2. **Test TCP Port reachability**:
```bash
nc -zv -w 3 10.0.2.20 5432

```


3. **Analyze `nc` output**:
* **Scenario A (`Connection refused`)**: DB Server B is reached, but PostgreSQL daemon is down or bound only to `127.0.0.1` instead of `0.0.0.0`.
* **Scenario B (`Connection timed out`)**: Network firewall, AWS Security Group, or local `iptables` on DB Server B is dropping incoming packets on port 5432.


4. **Check route path**:
```bash
traceroute -T -p 5432 10.0.2.20

```



---

# 5. Interview Questions & Expected Answers

### Q1: What steps do you take when a production server is completely non-responsive to HTTP requests?

**Answer**:

1. Check cloud monitoring/load balancer metrics to confirm if health checks are failing.
2. Attempt SSH access. If SSH succeeds:
* Run system health triage: `uptime`, `top`, `free -h`, `df -h`.
* Check if root or log partition is at 100% capacity.
* Check if process is running via `ps aux` and listening via `ss -tulpn`.


3. If SSH times out, check cloud security groups/firewalls, or use serial console/KVM to view kernel panic or OOM output.

---

### Q2: Why does `nc -zv host port` time out while `ping host` succeeds?

**Answer**: ICMP traffic (used by `ping`) is routed at Layer 3 (IP Level). `nc` operates at Layer 4 (Transport Level). A timeout in `nc` indicates that while the IP address is routable, a firewall (cloud security group, `iptables`, `ufw`) is actively blocking or dropping TCP packets on that specific port, or a router along the path is dropping traffic.

---

### Q3: How do you verify if an application is listening on IPv4 vs IPv6?

**Answer**: Inspect `ss -tulpn`.

* An address of `0.0.0.0:80` indicates IPv4 socket binding.
* An address of `:::80` or `[::]:80` indicates IPv6 binding. If dual-stack isn't configured properly in kernel, clients connecting over IPv4 may fail.

---

### Q4: How do you test if a local DNS resolution issue is caused by local configuration (`/etc/resolv.conf`) or the upstream DNS server?

**Answer**:

1. Query default DNS configuration: `dig example.com` (uses `/etc/resolv.conf`).
2. Bypass local configuration and query upstream explicitly: `dig @8.8.8.8 example.com`.
3. If explicit IP query succeeds but default query fails, the issue lies in local `/etc/resolv.conf`, systemd-resolved, or local network configuration.

---

### Q5: What command captures raw HTTP traffic on interface `eth0` for port `8080`?

**Answer**:

```bash
sudo tcpdump -i eth0 port 8080 -A -n

```

---

# 6. One-Minute Production Cheatsheet

✓ **Emergency Initial Checks**: `uptime` → `free -h` → `df -h` → `top`.

✓ **Process Check**: `pgrep -a <name>` or `ps aux | grep <name>`.

✓ **Port Check**: `sudo ss -tulpn | grep :<port>` or `sudo lsof -i :<port>`.

✓ **Layer 4 Port Test**: `nc -zv -w 3 <IP> <PORT>`.

✓ **Timeout vs Refused**:

* `Connection refused` = Reached host, but **no service is listening**.
* `Connection timed out` = **Firewall/Security Group is blocking traffic**.

✓ **DNS Query**: `dig <domain> +short` or `dig @<dns-ip> <domain>`.

✓ **Trace TCP Path**: `traceroute -T -p <port> <destination>`.

✓ **Packet Capture**: `sudo tcpdump -i any port <port> -n`.
