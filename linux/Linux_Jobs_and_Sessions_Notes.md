# Linux Jobs and Sessions Notes (DevOps Interview)

## What is a Job?

A **job** is a process started from your current shell.

Example:

``` bash
./deploy.sh
```

The shell creates a process and waits until it finishes.

------------------------------------------------------------------------

## Foreground Job

Runs in the current terminal and blocks the prompt.

``` bash
./deploy.sh
```

Use when the task is short.

------------------------------------------------------------------------

## Background Job

Run a command with `&`:

``` bash
./deploy.sh &
```

Output example:

``` text
[1] 12345
```

-   `1` → Job ID
-   `12345` → Process ID (PID)

The prompt returns immediately, allowing you to continue using the
terminal.

Check running jobs:

``` bash
jobs
```

Example:

``` text
[1]+ Running    ./deploy.sh &
```

Bring a job back:

``` bash
fg %1
```

Move a stopped job to background:

``` bash
bg %1
```

Stop a running foreground job:

Press `Ctrl + Z`

Kill a job:

``` bash
kill %1
```

or

``` bash
kill <PID>
```

------------------------------------------------------------------------

# Why use `&` in DevOps?

Example deployment:

``` bash
./deploy.sh &
```

While deployment runs, you can execute:

``` bash
kubectl get pods
kubectl logs deployment/my-app -f
helm list
```

without opening another terminal.

------------------------------------------------------------------------

# Problem with `./deploy.sh &`

If the script prints many lines:

``` text
Building image...
Pushing image...
Deploying...
```

they still appear on your terminal and make it messy.

------------------------------------------------------------------------

# Better way

``` bash
./deploy.sh > deploy.log 2>&1 &
```

Meaning:

-   `>` → Redirect standard output (stdout)
-   `2>&1` → Redirect standard error (stderr) to the same file
-   `&` → Run in background

Monitor logs:

``` bash
tail -f deploy.log
```

Stop monitoring:

Press `Ctrl + C`

The deployment continues running.

------------------------------------------------------------------------

# What is `nohup`?

If you close the terminal after:

``` bash
./deploy.sh &
```

the process usually receives **SIGHUP** and may terminate.

Use:

``` bash
nohup ./deploy.sh > deploy.log 2>&1 &
```

Now the process continues even if:

-   SSH disconnects
-   Terminal closes

------------------------------------------------------------------------

# Jobs vs Processes

  Jobs                       Processes
  -------------------------- -------------------------------
  Managed by current shell   Managed by kernel
  Seen with `jobs`           Seen with `ps`, `top`, `htop`
  Have Job IDs               Have PIDs

------------------------------------------------------------------------

# Commands to Remember

``` bash
jobs
fg %1
bg %1
kill %1
kill <PID>
ps -ef
top
htop
nohup command &
tail -f deploy.log
```

------------------------------------------------------------------------

# Typical DevOps Workflow

Start deployment:

``` bash
nohup ./deploy.sh > deploy.log 2>&1 &
```

Check logs:

``` bash
tail -f deploy.log
```

Check Kubernetes:

``` bash
kubectl get pods -w
```

------------------------------------------------------------------------

# Linux Jobs and Sessions - Notes + Interview Questions with Answers

## Interview Questions with Answers

### 1. What is a foreground process?

**Answer:** A foreground process runs in the current terminal and
occupies the shell until it finishes. You cannot use that terminal for
other commands unless you suspend or stop the process.

Example:

``` bash
./deploy.sh
```

------------------------------------------------------------------------

### 2. What is a background process?

**Answer:** A background process runs independently of the shell prompt,
allowing you to continue using the terminal.

Example:

``` bash
./deploy.sh &
```

------------------------------------------------------------------------

### 3. What does `&` do?

**Answer:** It tells the shell to start the command as a background job
and immediately returns the prompt.

------------------------------------------------------------------------

### 4. What is the difference between a Job ID and a PID?

  Job ID                         PID
  ------------------------------ -------------------------------
  Created by the shell           Created by the kernel
  Used with `jobs`, `fg`, `bg`   Used with `ps`, `kill`, `top`
  Example: `%1`                  Example: `12345`

------------------------------------------------------------------------

### 5. How do you list background jobs?

**Answer:**

``` bash
jobs
```

------------------------------------------------------------------------

### 6. How do you bring a background job to the foreground?

**Answer:**

``` bash
fg %1
```

------------------------------------------------------------------------

### 7. How do you resume a stopped job in the background?

**Answer:**

``` bash
bg %1
```

------------------------------------------------------------------------

### 8. Difference between Ctrl+C and Ctrl+Z?

**Ctrl+C** - Sends SIGINT - Terminates the process

**Ctrl+Z** - Sends SIGTSTP - Suspends (pauses) the process - Can later
be resumed using `bg` or `fg`

------------------------------------------------------------------------

### 9. What happens if you close the terminal after running `./deploy.sh &`?

**Answer:** Normally the process receives a **SIGHUP** signal and may
terminate.

To prevent this:

``` bash
nohup ./deploy.sh > deploy.log 2>&1 &
```

------------------------------------------------------------------------

### 10. What problem does `nohup` solve?

**Answer:** It keeps the process running even after the terminal or SSH
session closes.

------------------------------------------------------------------------

### 11. Explain this command.

``` bash
./deploy.sh > deploy.log 2>&1 &
```

**Answer:**

-   `./deploy.sh` → Run deployment script
-   `>` → Redirect stdout to `deploy.log`
-   `2>&1` → Redirect stderr to the same file
-   `&` → Run in the background

------------------------------------------------------------------------

### 12. What does `2>&1` mean?

**Answer:**

-   File descriptor 1 = Standard Output (stdout)
-   File descriptor 2 = Standard Error (stderr)

`2>&1` means:

> Redirect stderr to wherever stdout is currently going.

------------------------------------------------------------------------

### 13. How do you monitor deployment logs?

**Answer:**

``` bash
tail -f deploy.log
```

------------------------------------------------------------------------

### 14. Difference between `jobs` and `ps`?

**jobs** - Shows jobs started from the current shell.

**ps** - Shows system processes (or all processes with options like
`ps -ef`).

------------------------------------------------------------------------

### 15. Difference between a shell job and a Linux process?

**Answer:**

A shell job is managed by the shell for job control.

A process is managed by the Linux kernel.

Every job is a process, but not every process is a shell job.

------------------------------------------------------------------------

# Scenario-Based Questions

## Q1

Your deployment takes 20 minutes. What would you do?

**Answer**

``` bash
nohup ./deploy.sh > deploy.log 2>&1 &
```

Then monitor:

``` bash
tail -f deploy.log
```

------------------------------------------------------------------------

## Q2

Deployment output is cluttering your terminal.

**Answer**

Redirect output:

``` bash
./deploy.sh > deploy.log 2>&1 &
```

------------------------------------------------------------------------

## Q3

How do you stop the deployment?

**Answer**

``` bash
jobs
kill %1
```

or

``` bash
ps -ef | grep deploy
kill <PID>
```

------------------------------------------------------------------------

## Q4

How do you temporarily pause a running deployment?

**Answer**

Press:

``` text
Ctrl+Z
```

Resume:

``` bash
bg %1
```

or

``` bash
fg %1
```

------------------------------------------------------------------------

# Quick Revision

-   `&` → Background execution
-   `jobs` → List shell jobs
-   `fg` → Foreground
-   `bg` → Background
-   `Ctrl+C` → Kill
-   `Ctrl+Z` → Suspend
-   `nohup` → Survive terminal close
-   `tail -f` → Watch logs live
-   `2>&1` → Merge stderr into stdout


------------------------------------------------------------------------

# Key Takeaways

-   `&` → Run in background.
-   `jobs` → List shell jobs.
-   `fg` → Bring job to foreground.
-   `bg` → Resume stopped job in background.
-   `nohup` → Keep process alive after terminal closes.
-   `tail -f` → Watch logs live.
-   Redirect output to log files for long-running deployments.
