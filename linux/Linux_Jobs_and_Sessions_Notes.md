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

# Interview Questions

## Beginner

1.  What is a foreground process?
2.  What is a background process?
3.  What does `&` do?
4.  What is the difference between a Job ID and a PID?
5.  How do you list background jobs?
6.  How do you bring a job to the foreground?
7.  How do you move a stopped job to the background?
8.  What is the difference between `Ctrl+C` and `Ctrl+Z`?

## Intermediate

1.  What happens if you close the terminal after running
    `./deploy.sh &`?
2.  What problem does `nohup` solve?
3.  Explain:

``` bash
./deploy.sh > deploy.log 2>&1 &
```

4.  What does `2>&1` mean?
5.  How do you monitor a log file in real time?
6.  Difference between `jobs` and `ps`?
7.  Difference between a shell job and a Linux process?

## Scenario-Based

**Q1**

Your deployment takes 20 minutes.

What command would you use?

Expected answer:

``` bash
nohup ./deploy.sh > deploy.log 2>&1 &
```

------------------------------------------------------------------------

**Q2**

How would you monitor deployment progress?

``` bash
tail -f deploy.log
```

------------------------------------------------------------------------

**Q3**

Deployment is stuck.

How do you find and stop it?

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

# Key Takeaways

-   `&` → Run in background.
-   `jobs` → List shell jobs.
-   `fg` → Bring job to foreground.
-   `bg` → Resume stopped job in background.
-   `nohup` → Keep process alive after terminal closes.
-   `tail -f` → Watch logs live.
-   Redirect output to log files for long-running deployments.
