
---

# SetUID (SUID)

## Why do we need SetUID?

Suppose we have two users.

```text
User A (root)
User B (punit)
```

User A creates a program called `change-password`.

```text
Owner : root
Permission : -rwxr-xr-x
```

Now User B runs the program.

Normally, a program runs with the permissions of the user who executes it.

```text
User B executes program

↓

Program runs as User B
```

But User B is a normal user.

The program tries to modify

```text
/etc/shadow
```

Only **root** can modify this file.

So the program fails.

---

## Normal Solution?

Give write permission to everyone.

```bash
chmod 777 /etc/shadow
```

❌ Very dangerous.

Everyone can modify passwords.

---

## Change owner?

```bash
chown punit /etc/shadow
```

❌ Even worse.

Root no longer owns the password file.

---

## Best Solution

Use **SetUID**.

```bash
chmod u+s change-password
```

Now check:

```bash
ls -l change-password
```

Output

```text
-rwsr-xr-x
```

Notice

```text
rws
```

instead of

```text
rwx
```

The **s** means SetUID.

---

## What happens now?

```text
User B executes program

↓

Program temporarily runs as OWNER

↓

Owner is root

↓

Program can modify /etc/shadow

↓

Program finishes

↓

Back to User B
```

The user **does not become root**.

Only that program runs with the owner's privileges.

---

## Real Example

```bash
passwd
```

Check it

```bash
ls -l /usr/bin/passwd
```

Output

```text
-rwsr-xr-x
```

`passwd` is owned by root.

That's why a normal user can change **their own password** even though `/etc/shadow` is owned by root.

---

## Memory Trick

```text
SetUID

↓

Run as OWNER
```

---

# SetGID (SGID)

There are **two uses** of SetGID.

---

# 1. SetGID on File

Imagine

```text
Owner : root

Group : developers
```

Program permission

```text
-rwxr-xr-x
```

Normally

```text
User runs program

↓

Program runs with user's group
```

Enable SGID

```bash
chmod g+s program
```

Now

```bash
ls -l
```

Output

```text
-rwxr-sr-x
```

Notice

```text
r-s
```

Group execute becomes **s**.

Now

```text
Program runs with FILE'S GROUP

↓

developers
```

This is less common than SetUID.

---

# 2. SetGID on Directory (Very Important)

This is what you'll see in DevOps.

Suppose

```text
project/
```

belongs to

```text
Owner : root

Group : developers
```

Team members

```text
John

Alice

Bob
```

All belong to

```text
developers
```

---

John creates

```text
app.py
```

Normally

```text
Owner : John

Group : John's primary group
```

Now Alice creates

```text
config.yaml
```

```text
Owner : Alice

Group : Alice's primary group
```

Everyone ends up with different groups.

This causes permission problems.

---

## Best Solution

Enable SetGID.

```bash
chmod g+s project
```

Now

```bash
ls -ld project
```

Output

```text
drwxr-sr-x
```

Notice

```text
r-s
```

---

Now every new file automatically gets

```text
Group = developers
```

No matter who creates it.

John creates

```text
app.py
```

↓

Group

```text
developers
```

Alice creates

```text
config.yaml
```

↓

Group

```text
developers
```

Very useful for

* Shared project directories
* Team repositories
* Shared log directories

---

## Memory Trick

```text
SetGID

↓

Run as GROUP

Directory

↓

New files inherit directory group
```

---

# Sticky Bit

This is the easiest one.

---

Suppose we have

```text
/tmp
```

Everyone can write there.

User A

creates

```text
abc.txt
```

User B also has write permission.

Without Sticky Bit

User B can delete

```text
abc.txt
```

Even though User B didn't create it.

---

## Normal Solution

Remove write permission.

```bash
chmod o-w /tmp
```

❌ Bad.

Now nobody can create temporary files.

---

## Better Solution

Sticky Bit.

```bash
chmod +t /tmp
```

Now

```bash
ls -ld /tmp
```

Output

```text
drwxrwxrwt
```

Notice

```text
t
```

instead of

```text
x
```

---

Now

User A creates

```text
abc.txt
```

User B tries

```bash
rm abc.txt
```

Result

```text
Permission denied
```

Only these users can delete the file.

* File owner
* Directory owner
* Root

---

## Real Example

```text
/tmp
```

Every Linux system uses Sticky Bit.

Check

```bash
ls -ld /tmp
```

You'll almost always see

```text
drwxrwxrwt
```

---

## Memory Trick

```text
Sticky Bit

↓

Everyone can create

↓

Only OWNER can delete
```

---

# Final Memory Trick

```text
SUID

↓

Run as OWNER

----------------------

SGID

↓

Run as GROUP

----------------------

Sticky Bit

↓

Only OWNER can DELETE
```

---

# Interview Question

**Q. Difference between SetUID, SetGID, and Sticky Bit?**

| Feature      | SetUID                    | SetGID                                              | Sticky Bit                              |
| ------------ | ------------------------- | --------------------------------------------------- | --------------------------------------- |
| Works on     | Files                     | Files & Directories                                 | Directories                             |
| Purpose      | Run program as file owner | Run program as file group / inherit directory group | Prevent other users from deleting files |
| Symbol       | `s` (user field)          | `s` (group field)                                   | `t`                                     |
| Command      | `chmod u+s file`          | `chmod g+s file`                                    | `chmod +t directory`                    |
| Real Example | `/usr/bin/passwd`         | Shared project directory                            | `/tmp`                                  |

---

## One-Minute Revision

```text
Problem

Need root permission?

↓

SetUID

------------------------

Need same group for all new files?

↓

SetGID

------------------------

Need everyone to create files but stop them deleting each other's files?

↓

Sticky Bit
```

This is exactly how I'd include it in your GitHub notes. It teaches the **problem**, shows why normal permission changes are not the right solution, introduces the special permission as the correct solution, and finishes with the real-world DevOps use case. That's much easier to remember than memorizing definitions.
