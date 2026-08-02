# Linux User Administration

## Overview

Linux is a multi-user operating system. Every user has a unique User ID (UID), belongs to one or more groups, and has specific permissions that control access to files, directories, and system resources.

As a DevOps engineer, user administration is frequently used for:
- Creating application users
- Managing SSH access
- Running services with least privilege
- Managing Docker permissions
- Setting password policies
- Giving sudo access
- Troubleshooting permission issues

---

# Important Files

| File | Purpose |
|------|---------|
| /etc/passwd | User information |
| /etc/shadow | Encrypted passwords |
| /etc/group | Group information |
| /etc/gshadow | Secure group information |
| /home/ | User home directories |
| /etc/login.defs | Default password policies |

---

# User Management Commands

## Create User

```bash
useradd username
And if you don't to run command for attaching shell,set password,add group, Run

adduser username
```

Example

```bash
sudo useradd devops
```

---

Create user with home directory

```bash
useradd -m username
```

---

Create user with custom home directory

```bash
useradd -d /data/devops devops
```

---

Create user with specific shell

```bash
useradd -s /bin/bash devops
```

---

Create user with UID

```bash
useradd -u 2001 devops
```

---

Set Password

```bash
passwd username
```

Example

```bash
passwd devops
```

---

Delete User

```bash
userdel username
```

Delete user with home directory

```bash
userdel -r username
```

---

Modify User

```bash
usermod
```

Examples

Change username

```bash
usermod -l newname oldname
```

Lock account

```bash
usermod -L username
```

Unlock account

```bash
usermod -U username
```

Expire account

```bash
usermod -e 2026-12-31 username
```

Change login shell

```bash
usermod -s /bin/bash username
```

---

# Group Management

Create group

```bash
groupadd developers
```

Delete group

```bash
groupdel developers
```

Rename group

```bash
groupmod -n devops developers
```

---

Add user to supplementary group

```bash
usermod -aG docker ubuntu
```

Explanation

-a = append

-G = supplementary groups

Never use

```bash
usermod -G docker ubuntu
```

because it removes existing supplementary groups.

---

Remove user from group

```bash
gpasswd -d username docker
```

---

Display groups

```bash
groups username
```

or

```bash
id username
```

---

Current logged in user

```bash
whoami
```

Current user details

```bash
id
```

---

Switch User

```bash
su username
```

Login shell

```bash
su - username
```

Root login

```bash
sudo su -
su - root
su -
```

---

Password Aging

Display password policy

```bash
chage -l username
```

Force password change

```bash
chage -d 0 username
```

Maximum password age

```bash
chage -M 90 username
```

Minimum password age

```bash
chage -m 7 username
```

Warning days

```bash
chage -W 7 username
```

Expire account

```bash
chage -E 2026-12-31 username
```

---

# Sudo Access

Add user to sudo group

Ubuntu

```bash
usermod -aG sudo username
```

RHEL

```bash
usermod -aG wheel username
```

---

# Docker Example

Problem

Normal user cannot run Docker commands.

Error

permission denied while trying to connect to Docker daemon

Solution

```bash
sudo usermod -aG docker ubuntu
```

Apply changes

```bash
newgrp docker
```

or logout/login again.

---

# Useful Commands

```bash
cat /etc/passwd

cat /etc/shadow

cat /etc/group

id

groups

whoami

who

w

last

finger username

passwd

chage

getent passwd

getent group
```

---

# Production Notes

- Never use root for applications.
- Create dedicated service users.
- Give minimum required permissions.
- Use groups instead of giving permissions individually.
- Never edit /etc/shadow manually.
- Always use usermod instead of editing configuration files.
- Use sudo instead of logging in as root.

---

# Interview Questions

### 1. What is UID?

Answer:
UID (User ID) uniquely identifies a user in Linux. Root has UID 0.

---

### 2. What is GID?

Answer:
GID identifies a group.

---

### 3. Difference between useradd and adduser?

Answer:

useradd
- Low-level command
- Available on most Linux distributions
- Requires more manual configuration

adduser
- Interactive script
- User-friendly
- Creates home directory automatically (on Debian/Ubuntu)

---

### 4. Difference between su and su - ?

Answer:

su
Switches user but keeps current environment.

su -
Loads target user's environment completely.

---

### 5. Difference between sudo and su?

Answer:

sudo executes a single command with elevated privileges.

su switches to another user account.

---

### 6. What does usermod -aG mean?

Answer:

-a appends groups.

-G specifies supplementary groups.

---

### 7. Why shouldn't we use usermod -G?

Answer:

It replaces existing supplementary groups.

---

### 8. Where is user information stored?

Answer:

/etc/passwd

---

### 9. Where are passwords stored?

Answer:

/etc/shadow

---

### 10. Which file stores groups?

Answer:

/etc/group

---

### 11. What is /etc/gshadow?

Answer:

Stores secure group information including encrypted group passwords and administrators.

---

### 12. What command displays user information?

Answer:

id

---

### 13. How do you list groups of a user?

Answer:

groups username

---

### 14. How do you lock a user account?

Answer:

usermod -L username

---

### 15. How do you unlock a user account?

Answer:

usermod -U username

---

### 16. How do you force a password change?

Answer:

chage -d 0 username

---

### 17. What command shows password expiry?

Answer:

chage -l username

---

### 18. How do you delete a user and home directory?

Answer:

userdel -r username

---

### 19. Difference between primary and supplementary groups?

Answer:

Every user has one primary group.

A user can belong to multiple supplementary groups.

---

### 20. Why should applications run as non-root users?

Answer:

For better security and least privilege.

---

### 21. What command shows the current logged-in user?

Answer:

whoami

---

### 22. What is the difference between who, w, and last?

Answer:

- `who`: shows who is currently logged in.
- `w`: shows logged-in users and what they are doing.
- `last`: displays login history.

---

### 23. What is the purpose of `newgrp`?

Answer:

It starts a new shell with updated group membership, allowing new group permissions (such as the `docker` group) without logging out.

---

### 24. How can you check whether a user exists?

Answer:

Use:

```bash
id username
```

or

```bash
getent passwd username
```

---

### 25. Why is UID 0 special?

Answer:

UID 0 belongs to the root user, which has unrestricted administrative privileges.

---

# Scenario-Based Questions

### Scenario 1

A developer cannot execute Docker commands without sudo.

How will you fix it?

Answer

```bash
sudo usermod -aG docker developer

newgrp docker
```

or ask the user to log out and log back in.

---

### Scenario 2

Developer forgot password.

Answer

```bash
sudo passwd developer
```

---

### Scenario 3

Employee left the company.

Answer

```bash
sudo userdel -r username
```

or lock the account first if it may need to be restored:

```bash
sudo usermod -L username
```

---

### Scenario 4

User account should expire after contractor period.

Answer

```bash
sudo chage -E 2026-12-31 contractor
```

---

### Scenario 5

Application should not run as root.

Answer

Create a dedicated service account.

```bash
sudo useradd -r appuser
```

Run the application as that user.

---

### Scenario 6

You accidentally ran:

```bash
usermod -G docker ubuntu
```

What happened?

Answer

All existing supplementary groups were replaced by only the `docker` group. You need to add the missing groups back.

---

### Scenario 7

A user reports "Permission denied" after being added to the `docker` group.

Answer

Verify group membership with `id username`, then ask the user to log out and back in or run:

```bash
newgrp docker
```

---

### Scenario 8

How would you verify whether a password has expired?

Answer

```bash
chage -l username
```

---

### Scenario 9

A service must always run under the same user after a reboot.

Answer

Create a dedicated service account and configure the service (for example, in a systemd unit) to run as that user.

---

### Scenario 10

How do you find all users with Bash as their login shell?

Answer

```bash
grep "/bin/bash" /etc/passwd
```

---

# One-Minute Revision

✓ UID identifies a user.

✓ GID identifies a group.

✓ Root UID = 0.

✓ User info → /etc/passwd

✓ Passwords → /etc/shadow

✓ Groups → /etc/group

✓ Create user → useradd

✓ Delete user → userdel -r

✓ Modify user → usermod

✓ Password policy → chage

✓ User groups → groups, id

✓ Current user → whoami

✓ Add Docker access → usermod -aG docker username

✓ Use `-aG`, never just `-G` when adding supplementary groups.

✓ Prefer `su -` over `su` when switching users.
