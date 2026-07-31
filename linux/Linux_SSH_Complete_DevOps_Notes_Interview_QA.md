# Linux SSH - Complete DevOps Notes + Interview Questions & Answers

# 1. What is SSH?

SSH (Secure Shell) is a secure protocol used to remotely access Linux
servers over an encrypted connection.

Default port:

``` text
22
```

Common DevOps uses: - Login to EC2/VMs - Deploy applications - Copy
files - Execute remote commands - Git over SSH - Ansible automation

------------------------------------------------------------------------

# 2. Basic SSH Connection

``` bash
ssh user@server_ip
```

Example:

``` bash
ssh ubuntu@192.168.1.20
```

Custom port:

``` bash
ssh -p 2222 ubuntu@192.168.1.20
```

------------------------------------------------------------------------

# 3. Password-based Authentication

## Server Side

Install OpenSSH server:

``` bash
sudo apt install openssh-server
```

Check service:

``` bash
sudo systemctl status ssh
```

Edit config:

``` bash
sudo vim /etc/ssh/sshd_config
```

Important settings:

``` text
PasswordAuthentication yes
PermitRootLogin no
PubkeyAuthentication yes
```

Restart SSH:

``` bash
sudo systemctl restart ssh
```

Now users can login using username + password.

------------------------------------------------------------------------

# 4. SSH Key-Based (Password-less) Authentication

## Step 1 Generate Keys

``` bash
ssh-keygen -t ed25519
```

or

``` bash
ssh-keygen -t rsa -b 4096
```

Files created:

``` text
~/.ssh/id_ed25519
~/.ssh/id_ed25519.pub
```

Private key = NEVER share

Public key = Copy to server

------------------------------------------------------------------------

## Step 2 Copy Public Key

Recommended:

``` bash
ssh-copy-id user@server
```

Manual:

Append public key into:

``` text
~/.ssh/authorized_keys
```

Correct permissions:

``` bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

------------------------------------------------------------------------

## Step 3 Login

``` bash
ssh user@server
```

No password is required (unless your private key has a passphrase).

------------------------------------------------------------------------

# 5. Disable Password Login (Production)

After verifying key login works:

``` text
PasswordAuthentication no
PubkeyAuthentication yes
```

Restart:

``` bash
systemctl restart ssh
```

Always test a second SSH session before closing the first.

------------------------------------------------------------------------

# 6. SSH Config File (Client)

Location:

``` text
~/.ssh/config
```

Example:

``` text
Host prod
    HostName 10.0.0.10
    User ubuntu
    IdentityFile ~/.ssh/prod_key
    Port 22

Host staging
    HostName 10.0.0.20
    User ubuntu
    IdentityFile ~/.ssh/staging_key
```

Now simply run:

``` bash
ssh prod
ssh staging
```

Useful options:

-   Host
-   HostName
-   User
-   Port
-   IdentityFile
-   ForwardAgent
-   ServerAliveInterval

------------------------------------------------------------------------

# 7. SSH Server Config

Location:

``` text
/etc/ssh/sshd_config
```

Important directives:

``` text
Port 22
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
PermitEmptyPasswords no
MaxAuthTries 3
AllowUsers ubuntu devops
```

After changes:

``` bash
sudo sshd -t
sudo systemctl restart ssh
```

`sshd -t` validates configuration before restarting.

------------------------------------------------------------------------

# 8. Copy Files

Upload:

``` bash
scp app.jar ubuntu@server:/opt/apps/
```

Download:

``` bash
scp ubuntu@server:/tmp/app.log .
```

Copy directory:

``` bash
scp -r project ubuntu@server:/opt/
```

------------------------------------------------------------------------

# 9. Remote Command Execution

``` bash
ssh ubuntu@server "hostname"
```

Example:

``` bash
ssh ubuntu@server "kubectl get pods"
```

Useful in CI/CD pipelines.

------------------------------------------------------------------------

# 10. Troubleshooting

Connection refused

-   SSH service stopped
-   Wrong port
-   Firewall
-   Security group (cloud)

Permission denied (publickey)

-   Wrong private key
-   Public key missing
-   Wrong permissions on \~/.ssh
-   Wrong username

Host key verification failed

Remove old entry:

``` bash
ssh-keygen -R server_ip
```

------------------------------------------------------------------------

# 11. DevOps Scenarios

## EC2 Login

``` bash
ssh -i aws.pem ubuntu@PUBLIC_IP
```

------------------------------------------------------------------------

## GitHub

Generate key

``` bash
ssh-keygen -t ed25519
```

Add public key to GitHub.

Clone:

``` bash
git clone git@github.com:user/repo.git
```

------------------------------------------------------------------------

## Jenkins Deployment

Jenkins stores deployment key.

Pipeline runs:

``` bash
ssh prod "cd /app && ./deploy.sh"
```

------------------------------------------------------------------------

# Security Best Practices

-   Disable root login
-   Disable password authentication after key setup
-   Protect private key (`chmod 600`)
-   Use passphrases
-   Rotate keys
-   Restrict users with AllowUsers
-   Use non-root accounts with sudo
-   Test new SSH config in another terminal before closing current
    session

------------------------------------------------------------------------

# Interview Questions & Answers

## 1. Difference between password and key authentication?

Password authentication uses a username/password.

Key authentication uses a private/public key pair and is more secure.

------------------------------------------------------------------------

## 2. Which key should be copied to the server?

Only the **public key** (`*.pub`).

Never copy the private key.

------------------------------------------------------------------------

## 3. Where is the server SSH configuration?

``` text
/etc/ssh/sshd_config
```

------------------------------------------------------------------------

## 4. Where is the client SSH configuration?

``` text
~/.ssh/config
```

------------------------------------------------------------------------

## 5. How do you generate an SSH key?

``` bash
ssh-keygen -t ed25519
```

------------------------------------------------------------------------

## 6. Where are public keys stored on the server?

``` text
~/.ssh/authorized_keys
```

------------------------------------------------------------------------

## 7. Why does "Permission denied (publickey)" occur?

-   Wrong username
-   Wrong key
-   Public key not installed
-   Incorrect file permissions
-   SSH server not allowing public key auth

------------------------------------------------------------------------

## 8. Why disable root login?

Reduces attack surface and encourages login as a normal user followed by
sudo.

------------------------------------------------------------------------

## 9. What permissions should \~/.ssh have?

``` text
~/.ssh                700
authorized_keys       600
Private key           600
Public key            644 (or 600)
```

------------------------------------------------------------------------

## 10. Why run `sshd -t`?

To validate SSH configuration before restarting the SSH service.

------------------------------------------------------------------------

## 11. How would you troubleshoot "Connection refused"?

1.  Check SSH service:

``` bash
systemctl status ssh
```

2.  Check listening port:

``` bash
ss -tulpn | grep ssh
```

3.  Check firewall/security groups.
4.  Verify the port in `sshd_config`.

------------------------------------------------------------------------

# Quick Revision

-   ssh = remote login
-   sshd = SSH server daemon
-   ssh-keygen = generate keys
-   ssh-copy-id = install public key
-   authorized_keys = trusted public keys
-   \~/.ssh/config = client shortcuts
-   /etc/ssh/sshd_config = server configuration
-   scp = secure copy
-   PasswordAuthentication = enable/disable passwords
-   PubkeyAuthentication = enable key login
-   PermitRootLogin = control root SSH access
