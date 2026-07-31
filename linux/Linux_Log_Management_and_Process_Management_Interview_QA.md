# Linux Log Management & Process Management (DevOps Notes + Interview Q&A)

# 1. Why Logs Matter

Logs tell you **what happened, when it happened, and why it happened**.

As a DevOps engineer, logs are usually the first place you look when: -
Deployment fails - Pod keeps restarting - Application throws errors -
Server becomes slow - Service won't start

------------------------------------------------------------------------

# Common Log Locations

  Log              Location
  ---------------- ------------------------------------------
  System logs      `/var/log/`
  Authentication   `/var/log/auth.log` (Ubuntu)
  Syslog           `/var/log/syslog`
  Kernel           `/var/log/kern.log`
  Nginx            `/var/log/nginx/access.log`, `error.log`
  Apache           `/var/log/apache2/`

------------------------------------------------------------------------

# Useful Commands

## View entire file

``` bash
cat app.log
```

## Read page by page

``` bash
less app.log
```

## Last 20 lines

``` bash
tail -20 app.log
```

## Watch live logs

``` bash
tail -f app.log
```

**DevOps Example**

``` bash
tail -f deploy.log
```

while a deployment is running.

------------------------------------------------------------------------

## Search logs

``` bash
grep ERROR app.log
grep -i timeout app.log
grep -n Exception app.log
```

Example:

``` bash
kubectl logs my-pod | grep ERROR
```

------------------------------------------------------------------------

## Filter today's logs

``` bash
grep "2026-08-01" app.log
```

------------------------------------------------------------------------

# journalctl (systemd)

Show logs for a service

``` bash
journalctl -u nginx
```

Latest logs

``` bash
journalctl -u docker -f
```

------------------------------------------------------------------------

# Kubernetes Logs

Single Pod

``` bash
kubectl logs my-pod
```

Follow

``` bash
kubectl logs -f my-pod
```

Previous crashed container

``` bash
kubectl logs --previous my-pod
```

------------------------------------------------------------------------

# Real DevOps Scenarios

## Scenario 1 - Deployment Failed

``` bash
./deploy.sh > deploy.log 2>&1 &
tail -f deploy.log
```

Read errors without cluttering terminal.

------------------------------------------------------------------------

## Scenario 2 - Pod CrashLoopBackOff

``` bash
kubectl logs my-pod
kubectl describe pod my-pod
```

Look for: - Image pull errors - Missing env vars - Database connection
failure

------------------------------------------------------------------------

## Scenario 3 - Nginx Returning 502

Check

``` bash
tail -f /var/log/nginx/error.log
```

------------------------------------------------------------------------

# Log Rotation

Without rotation:

    app.log

keeps growing.

Linux uses **logrotate** to:

-   Rotate logs
-   Compress old logs
-   Delete old logs

Common location

``` bash
/etc/logrotate.conf
/etc/logrotate.d/
```

------------------------------------------------------------------------

# Process Management

A process is a running program.

Example

``` bash
python app.py
```

creates a process.

------------------------------------------------------------------------

# Find Processes

``` bash
ps
```

All processes

``` bash
ps -ef
```

Search

``` bash
ps -ef | grep nginx
```

------------------------------------------------------------------------

# Real-time Monitoring

``` bash
top
```

Better UI

``` bash
htop
```

Useful columns

-   PID
-   CPU %
-   Memory %
-   Command

------------------------------------------------------------------------

# Kill Processes

Graceful

``` bash
kill PID
```

Force

``` bash
kill -9 PID
```

Kill by name

``` bash
pkill nginx
```

------------------------------------------------------------------------

# Background Processes

``` bash
./deploy.sh &
jobs
fg %1
bg %1
```

------------------------------------------------------------------------

# Find Process Using a Port

``` bash
lsof -i :8080
```

or

``` bash
ss -tulpn
```

Example

Port 8080 already in use.

Find PID

Kill process

Restart application.

------------------------------------------------------------------------

# DevOps Scenario

Application won't start.

Error:

    Address already in use

Solution

``` bash
lsof -i :8080
kill PID
```

Restart application.

------------------------------------------------------------------------

# Interview Questions with Answers

## Q1 Why are logs important?

Answer: Logs help identify application errors, deployment failures,
security issues and performance problems.

------------------------------------------------------------------------

## Q2 Difference between stdout and stderr?

Answer:

stdout = normal output

stderr = error messages

Both can be redirected:

``` bash
command > out.log 2>&1
```

------------------------------------------------------------------------

## Q3 Difference between tail and tail -f?

Answer:

tail = last lines once

tail -f = continuously follow new log entries.

------------------------------------------------------------------------

## Q4 How do you check logs of a Kubernetes Pod?

Answer

``` bash
kubectl logs my-pod
```

Follow

``` bash
kubectl logs -f my-pod
```

------------------------------------------------------------------------

## Q5 How do you find a running process?

Answer

``` bash
ps -ef
```

or

``` bash
pgrep nginx
```

------------------------------------------------------------------------

## Q6 Difference between kill and kill -9?

Answer

kill sends SIGTERM allowing graceful shutdown.

kill -9 sends SIGKILL forcing immediate termination.

Use SIGKILL only if the process refuses to stop.

------------------------------------------------------------------------

## Q7 How do you find which process is using port 8080?

Answer

``` bash
lsof -i :8080
```

------------------------------------------------------------------------

## Q8 Your deployment script has been running for 15 minutes. How do you check progress?

Answer

``` bash
tail -f deploy.log
```

------------------------------------------------------------------------

## Q9 Nginx service won't start. First steps?

Answer

1.  Check service status

``` bash
systemctl status nginx
```

2.  Check logs

``` bash
journalctl -u nginx
```

3.  Check if another process is using port 80

``` bash
lsof -i :80
```

------------------------------------------------------------------------

## Q10 Pod keeps restarting.

Answer

``` bash
kubectl describe pod POD_NAME
kubectl logs POD_NAME
kubectl logs --previous POD_NAME
```

------------------------------------------------------------------------

# Quick Revision

-   cat = display file
-   less = read large logs
-   tail = last lines
-   tail -f = follow logs
-   grep = search logs
-   journalctl = systemd logs
-   ps -ef = list processes
-   top/htop = monitor resources
-   kill = graceful stop
-   kill -9 = force stop
-   lsof -i :PORT = process using a port
-   logrotate = rotate logs
