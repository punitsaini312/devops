Here is the complete content for the **Linux System Administration & DevOps Operations Reference** guide formatted as a clean Markdown document.

---

# Linux System Administration & DevOps Operations Reference

## Table of Contents

1. [Process Management](https://www.google.com/search?q=%231-process-management)
2. [Jobs & Session Management](https://www.google.com/search?q=%232-jobs--session-management)
3. [Log Management & Systemd Journal](https://www.google.com/search?q=%233-log-management--systemd-journal)
4. [Task Scheduling with Cron & Crontab](https://www.google.com/search?q=%234-task-scheduling-with-cron--crontab)
5. [Archiving, Compression & Tar Backups](https://www.google.com/search?q=%235-archiving-compression--tar-backups)
6. [SSH Administration & Hardening](https://www.google.com/search?q=%236-ssh-administration--hardening)
7. [Comprehensive Interview & Scenario Guide](https://www.google.com/search?q=%237-comprehensive-interview--scenario-guide)
8. [One-Minute Cheatsheet](https://www.google.com/search?q=%238-one-minute-cheatsheet)

---

# 1. Process Management

## Overview

A process is an instance of a running program in Linux. Every process is assigned a unique **PID** (Process ID) and runs under a specific environment, user context, and priority level. DevOps engineers must manage processes to optimize application performance, debug stuck deployments, and maintain cluster node health.

---

## Important Process Concepts

| Concept | Description |
| --- | --- |
| **PID** | Process Identification Number (Unique per process). |
| **PPID** | Parent Process ID (The process that launched this process). |
| **Daemon** | Background process running without user interaction (e.g., `sshd`, `dockerd`). |
| **Zombie Process** | Terminated process whose exit code hasn't been read by its parent (`Z` state). |
| **Orphan Process** | Process whose parent died; adopted automatically by PID 1 (`systemd` or `init`). |
| **Nice Value** | Priority weighting ranging from `-20` (highest priority) to `19` (lowest priority). |

---

## Process Monitoring Commands

### 1. `ps` (Process Status Snapshot)

Displays static snapshots of active processes.

```bash
# Standard DevOps syntax: List all processes in full-format detail
ps -ef

# BSD syntax: Show detailed resource usage (CPU, Memory, TTY, State)
ps aux

# Show process hierarchy as a tree
ps aux --forest

# Sort processes by highest CPU usage
ps aux --sort=-%cpu | head -n 10

# Sort processes by highest Memory usage
ps aux --sort=-%mem | head -n 10

```

### 2. Interactive Real-Time Monitors

```bash
# Standard real-time system process monitor
top

# Interactive, user-friendly real-time process viewer
htop

# Disk I/O process monitor (requires root/sudo)
sudo iotop

```

---

## Managing Process Execution & Signals

Signals are asynchronous notifications sent by the OS or user to a process to force state changes.

### Essential Signals Table

| Signal | Number | Keyword | Behavior |
| --- | --- | --- | --- |
| **SIGHUP** | `1` | Hangup | Reload configuration without stopping the daemon. |
| **SIGINT** | `2` | Interrupt | Gracefully stop process from terminal (`Ctrl + C`). |
| **SIGQUIT** | `3` | Quit | Terminate process and dump core. |
| **SIGKILL** | `9` | Kill | **Forcefully & immediately kill** process (Cannot be caught or ignored). |
| **SIGTERM** | `15` | Terminate | Default graceful shutdown request (Allows cleanup). |
| **SIGSTOP** | `19` | Stop | Pause process execution (`Ctrl + Z`). |
| **SIGCONT** | `18` | Continue | Resume execution of a paused process. |

### Process Signal Commands

```bash
# Gracefully terminate a process by PID
kill 1234

# Forcefully kill a runaway process by PID
kill -9 1234

# Send reload signal to Nginx master process
sudo kill -1 $(pgrep nginx | head -n 1)

# Terminate process by name
pkill nginx

# Forcefully terminate all instances of a application name
killall -9 python3

```

---

## Priority and Niceness

Linux processes default to a niceness value of `0`. Lower niceness means higher execution priority.

```bash
# Start a CPU-intensive backup script with lowest priority (nice = 19)
nice -n 19 ./backup.sh

# Increase priority of a database process (requires sudo)
sudo renice -n -10 -p 1234

```

---

# 2. Jobs & Session Management

## Overview

In Linux shells, a **job** is a process started interactively within a terminal session. Session management tools like `tmux` prevent jobs from terminating when SSH sessions disconnect or timeout.

---

## Foreground vs Background Jobs

```bash
# Run a long-running process in the background by appending '&'
python3 app.py &

# Suspend current foreground process and send to background
# Press: Ctrl + Z

# Display all background jobs in current shell session
jobs -l

# Bring background job #1 into the foreground
fg %1

# Resume job #2 in the background without bringing it to foreground
bg %2

```

---

## Terminal Persistence with `nohup` & `disown`

When an SSH session closes, a `SIGHUP` (Signal 1) is sent to all sub-processes, killing them.

```bash
# Execute script immune to hangups, redirecting logs to nohup.out
nohup ./deploy_pipeline.sh &

# Detach a running background process from current shell session
./run_app.sh &
disown -h %1

```

---

## Session Persistence with `tmux`

`tmux` creates a terminal multiplexer session independent of SSH connections.

```bash
# Start a new named session
tmux new -s devops_session

# Detach from current session (Inside tmux)
# Press: Ctrl + B, then press D

# List active tmux sessions
tmux ls

# Reattach to an existing session
tmux attach -t devops_session

# Kill a specific session
tmux kill-session -t devops_session

```

---

# 3. Log Management & Systemd Journal

## Overview

Log management is critical for system auditing, performance debugging, and security analysis. Modern Linux distros (Ubuntu, RHEL, Debian) use `systemd-journald` alongside `rsyslog`.

---

## Core Log File Directory Locations (`/var/log`)

| Log File | Purpose |
| --- | --- |
| `/var/log/syslog` or `/var/log/messages` | Central system & operational logs. |
| `/var/log/auth.log` or `/var/log/secure` | Authentication logs (SSH logins, sudo usage, failures). |
| `/var/log/dmesg` | Kernel ring buffer logs (Hardware, boot errors, driver info). |
| `/var/log/nginx/` / `/var/log/httpd/` | Web application server access and error logs. |

---

## Querying Logs with `journalctl`

`journalctl` queries logs collected by `systemd-journald`.

```bash
# View all system logs (paginated)
journalctl

# Follow live log stream in real time (equivalent to tail -f)
journalctl -f

# View logs for a specific systemd service (e.g., Docker, Nginx)
journalctl -u docker.service -f

# View logs for current boot only
journalctl -b

# View error level logs and above (Priority 3)
journalctl -p err..emerg

# Filter logs by time window
journalctl --since "2026-07-30 10:00:00" --until "2026-07-31 12:00:00"

# Check log storage consumption
journalctl --disk-usage

# Clean logs older than 7 days
sudo journalctl --vacuum-time=7d

```

---

## Log Rotation with `logrotate`

Logrotate prevents disks from filling up by compressing, rotating, and removing old logs automatically.

* **Main Configuration File**: `/etc/logrotate.conf`
* **Service Configurations**: `/etc/logrotate.d/`

### Example Configuration: `/etc/logrotate.d/myapp`

```text
/var/log/myapp/*.log {
    daily
    missingok
    rotate 7
    compress
    delaycompress
    notifempty
    create 0640 appuser appuser
    sharedscripts
    postrotate
        /usr/bin/systemctl reload myapp.service > /dev/null 2>&1 || true
    endscript
}

```

```bash
# Force a dry-run test of logrotate configuration
sudo logrotate -d /etc/logrotate.d/myapp

```

---

# 4. Task Scheduling with Cron & Crontab

## Overview

Cron is a time-based job scheduler in Unix-like operating systems. It allows tasks (backup scripts, cleanup jobs, health checks) to run automatically at specific intervals.

---

## Crontab Syntax Guide

```text
.---------------- minute (0 - 59)
|  .------------- hour (0 - 23)
|  |  .---------- day of month (1 - 31)
|  |  |  .------- month (1 - 12)
|  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7)
|  |  |  |  |
*  *  *  *  * command_to_execute

```

---

## Cron Operators

| Operator | Meaning | Example |
| --- | --- | --- |
| `*` | Any value / Every interval | `* * * * *` (Every minute) |
| `,` | Value list separator | `0 8,12,18 * * *` (At 8:00, 12:00, and 18:00) |
| `-` | Range of values | `0 9-17 * * *` (Hourly from 9 AM to 5 PM) |
| `/` | Step values | `*/15 * * * *` (Every 15 minutes) |

---

## Common Schedule Examples

| Cron Expression | Schedule Description |
| --- | --- |
| `*/5 * * * *` | Every 5 minutes |
| `0 * * * *` | Every hour on the hour |
| `0 2 * * *` | Daily at 2:00 AM |
| `0 0 * * 0` | Weekly on Sunday at midnight |
| `0 0 1 * *` | First day of every month at midnight |
| `@reboot` | Run once at system startup |

---

## Crontab Management Commands

```bash
# Edit current user's crontab file
crontab -e

# View active cron jobs for current user
crontab -l

# Remove all cron jobs for current user
crontab -r

# View cron jobs for a specific user (requires root)
sudo crontab -u www-data -l

```

> ⚠️ **Production Best Practice**: Always specify full absolute paths for commands inside cron jobs (e.g., `/usr/bin/python3` instead of `python3`) and redirect output/errors to log files.

```bash
# Correct Production Cron Pattern
0 3 * * * /usr/local/bin/backup.sh >> /var/log/backup.log 2>&1

```

---

# 5. Archiving, Compression & Tar Backups

## Overview

`tar` (Tape Archive) bundles multiple files and directories into a single archive file, which can then be compressed using algorithms like `gzip`, `bzip2`, or `xz`.

---

## Tar Flag Reference

| Flag | Meaning |
| --- | --- |
| `-c` | **Create** a new archive file. |
| `-x` | **Extract** files from an archive. |
| `-t` | **List** contents of an archive without extracting. |
| `-v` | **Verbose** output (displays processed files). |
| `-f` | Specifies target archive **filename**. |
| `-z` | Filter through **gzip** (`.tar.gz` or `.tgz`). |
| `-j` | Filter through **bzip2** (`.tar.bz2`). |
| `-J` | Filter through **xz** (`.tar.xz`). |
| `-C` | Change target **directory** before extracting. |
| `-p` | **Preserve** file permissions and attributes. |

---

## Common `tar` Usage Examples

```bash
# 1. Create a compressed tar.gz archive of a directory
tar -czvf app_backup.tar.gz /var/www/app

# 2. Extract tar.gz archive to current directory
tar -xzvf app_backup.tar.gz

# 3. Extract archive into a specific destination directory
tar -xzvf app_backup.tar.gz -C /opt/deployments/

# 4. List contents of tar file without extracting
tar -tzvf app_backup.tar.gz

# 5. Create an archive excluding specific folders (e.g., node_modules, logs)
tar --exclude='app/node_modules' --exclude='app/logs' -czvf app.tar.gz /var/www/app

```

---

# 6. SSH Administration & Hardening

## Overview

Secure Shell (SSH) provides encrypted communication over insecure networks. Proper SSH configuration and key management are essential for securing server infrastructure.

---

## Key Files & Paths

| Path | Purpose |
| --- | --- |
| `~/.ssh/id_rsa` | Client Private Key (**Keep strictly secure! Permissions `600**`). |
| `~/.ssh/id_rsa.pub` | Client Public Key (Copied to remote servers). |
| `~/.ssh/authorized_keys` | Remote server file containing authorized public keys (Permissions `600`). |
| `~/.ssh/known_hosts` | Fingerprints of remote servers previously connected to. |
| `~/.ssh/config` | Client-side SSH configuration file for host aliases. |
| `/etc/ssh/sshd_config` | Server-side daemon configuration file (**Hardening location**). |

---

## SSH Key Management & Setup

```bash
# Generate a modern, highly secure ED25519 key pair
ssh-keygen -t ed25519 -C "admin@company.com"

# Copy public key to remote server for passwordless authentication
ssh-copy-id -i ~/.ssh/id_ed25519.pub devops@192.168.1.50

```

---

## Client-Side SSH Config File (`~/.ssh/config`)

Avoid typing long IP addresses, ports, and key paths repeatedly by creating aliases.

```text
Host prod-db
    HostName 10.0.2.15
    User ubuntu
    Port 2222
    IdentityFile ~/.ssh/prod_id_ed25519
    ServerAliveInterval 60

```

*Usage*: Connect instantly with `ssh prod-db`

---

## Server Hardening Best Practices (`/etc/ssh/sshd_config`)

Apply these parameters in `/etc/ssh/sshd_config` to secure production servers:

```ini
# Disable root account direct login
PermitRootLogin no

# Disable password authentication (Force SSH Key authentication only)
PasswordAuthentication no

# Change default port away from 22
Port 2222

# Restrict maximum authentication attempts to block brute-force attacks
MaxAuthTries 3

# Disable empty passwords
PermitEmptyPasswords no

# Disable X11 graphical forwarding
X11Forwarding no

# Set idle session timeout (300 seconds = 5 minutes)
ClientAliveInterval 300
ClientAliveCountMax 0

# Limit SSH access to specific users or groups
AllowUsers devops appadmin
AllowGroups devops-team

```

```bash
# Validate configuration syntax before restarting daemon
sudo sshd -t

# Apply configuration changes
sudo systemctl reload sshd

```

---

# 7. Comprehensive Interview & Scenario Guide

### Q1: Difference between SIGKILL (9) and SIGTERM (15)?

**Answer**: `SIGTERM` (15) asks a process to terminate gracefully, giving it time to release file locks, close database connections, and clear temporary files. `SIGKILL` (9) immediately terminates the process at the kernel level; the process cannot handle or ignore `SIGKILL`, which may leave temporary files or lock files behind.

---

### Q2: What causes a Zombie Process, and how do you resolve it?

**Answer**: A zombie process (`Z` state in `ps aux`) occurs when a child process terminates, but its parent process fails to read its exit status using the `wait()` system call. Zombie processes consume no memory or CPU, only a PID table entry. You cannot kill a zombie process using `kill -9`. To resolve it, kill the parent process, causing the zombie to be adopted by PID 1 (`systemd`), which automatically reaps it.

---

### Q3: How do you keep a process running after closing your SSH terminal?

**Answer**: Use one of the following:

1. **`nohup`**: `nohup ./script.sh &` ignores `SIGHUP` signals.
2. **`disown`**: Detaches running background jobs using `disown -h %jobid`.
3. **`tmux`**: Run the process inside a persistent session and detach using `Ctrl + B, D`.
4. **`systemd`**: Run the program as an underlying background system service.

---

### Q4: How do you tail real-time logs for a specific systemd unit?

**Answer**:

```bash
journalctl -u nginx.service -f

```

---

### Q5: What permissions should be applied to SSH keys and directories?

**Answer**:

* `~/.ssh` directory: `700` (`drwx------`)
* `~/.ssh/authorized_keys`: `600` (`-rw-------`)
* `~/.ssh/id_rsa` (Private Key): `600` (`-rw-------`)
* `~/.ssh/id_rsa.pub` (Public Key): `644` (`-rw-r--r--`)

---

### Q6: What happens if you forget `-f` when running `tar -czv archive.tar.gz /data`?

**Answer**: `tar` expects the filename immediately following the `-f` flag. If `-f` is omitted, `tar` attempts to write to standard archive devices (like magnetic tapes) or fail with argument parsing errors.

---

### Scenario 1: High CPU Utilization on Server

**Problem**: An unknown process is causing 100% CPU usage on a production server.
**Resolution Steps**:

1. Run `htop` or `ps aux --sort=-%cpu | head -n 5` to locate the culprit process and note its PID and user.
2. Check process origin using `ls -l /proc/<PID>/exe` and `pwdx <PID>`.
3. Send a graceful shutdown signal: `kill -15 <PID>`.
4. If unresponsive after 10 seconds, force-kill: `kill -9 <PID>`.

---

### Scenario 2: Log Partition Disk Space Emergency

**Problem**: The root partition is 100% full because `/var/log` ran out of space.
**Resolution Steps**:

1. Identify large log files using `du -sh /var/log/* | sort -rh | head -n 10`.
2. Vacuum old journalctl logs: `sudo journalctl --vacuum-size=500M`.
3. Truncate active log files safely without breaking process handles:
```bash
sudo truncate -s 0 /var/log/app/huge_output.log

```


4. Verify `/etc/logrotate.d/` configurations to ensure log rotation is functioning correctly.

---

# 8. One-Minute Cheatsheet

✓ **Processes**: View processes with `ps aux`, monitor with `htop`, kill gracefully with `kill -15 <PID>`, force kill with `kill -9 <PID>`.

✓ **Priority**: Lower `nice` value = higher priority. Scale runs from `-20` (highest) to `19` (lowest).

✓ **Sessions**: Suspend foreground job using `Ctrl + Z`, run in background with `bg`, bring back with `fg`. Protect long-running tasks using `nohup` or `tmux`.

✓ **Logs**: Read systemd logs using `journalctl -u <service> -f`. Configure automatic log management via `/etc/logrotate.d/`.

✓ **Cron**: Format: `Min Hour Day Month DayOfWeek Command`. Edit with `crontab -e`, list with `crontab -l`.

✓ **Tar**: Pack archive with `tar -czvf archive.tar.gz /path`, unpack with `tar -xzvf archive.tar.gz -C /destination`.

✓ **SSH Security**: Set `PermitRootLogin no` and `PasswordAuthentication no` in `/etc/ssh/sshd_config`. Enforce strict `600` permissions on private keys.
