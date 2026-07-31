1. What does ssh-copy-id do?
Suppose you have:

Your laptop (client):

~/.ssh/
├── id_ed25519
├── id_ed25519.pub
You want to log in to a remote VM:

ssh ubuntu@192.168.1.20
For password-less login to work, your public key must be present on the VM in:

/home/ubuntu/.ssh/authorized_keys
You could do this manually:

scp ~/.ssh/id_ed25519.pub ubuntu@192.168.1.20:/tmp
Login to the VM:

ssh ubuntu@192.168.1.20
Then:

mkdir -p ~/.ssh
cat /tmp/id_ed25519.pub >> ~/.ssh/authorized_keys
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
That's a lot of work.

ssh-copy-id automates all of this
Simply run:

ssh-copy-id ubuntu@192.168.1.20
It will:

✅ Ask for the user's password one last time.

Then automatically:

Create ~/.ssh if it doesn't exist.
Create authorized_keys if needed.
Copy your public key (*.pub) into authorized_keys.
Set the correct permissions.
Now you can log in with:

ssh ubuntu@192.168.1.20
without entering the account password.

Before
Your laptop

id_ed25519
id_ed25519.pub
↓

VM

authorized_keys

(empty)
Run

ssh-copy-id ubuntu@192.168.1.20
↓

VM

authorized_keys

ssh-ed25519 AAAAC3Nza...
Done.

2. What is ~/.ssh/config?
This is NOT on the VM.

It is on your machine (the SSH client).

Imagine this setup:

Your Laptop
        |
        |
        +----------------------+
        |                      |
        v                      v
    AWS Server             Azure Server
Your laptop has

~/.ssh/config
Each server has

/etc/ssh/sshd_config
These are different files with different purposes.

Client
Your Laptop

~/.ssh/config
This file tells your SSH client:

"When I type ssh prod, connect to this server using this user and this key."

Example:

Host prod
    HostName 10.0.0.20
    User ubuntu
    IdentityFile ~/.ssh/prod.pem
Now instead of typing:

ssh -i ~/.ssh/prod.pem ubuntu@10.0.0.20
you simply type:

ssh prod
Very convenient when you manage many servers.

Server
On the VM:

/etc/ssh/sshd_config
This file configures the SSH server.

Example:

PasswordAuthentication no
PermitRootLogin no
Port 22
PubkeyAuthentication yes
These settings determine:

Can users log in with passwords?
Can root log in?
Which port does SSH listen on?
Are public keys accepted?
Easy way to remember
Client side (your laptop)
~/.ssh/config
Meaning:

"How should I connect to servers?"

Server side (VM)
/etc/ssh/sshd_config
Meaning:

"How should I allow others to connect to me?"

Real DevOps Example
Suppose you manage three servers:

AWS Production
AWS Staging
Azure VM
Without ~/.ssh/config:

ssh -i ~/.ssh/prod.pem ubuntu@18.23.45.67
ssh -i ~/.ssh/staging.pem ubuntu@18.22.11.33
ssh -i ~/.ssh/azure.pem azureuser@20.10.5.6
With ~/.ssh/config:

Host prod
    HostName 18.23.45.67
    User ubuntu
    IdentityFile ~/.ssh/prod.pem

Host staging
    HostName 18.22.11.33
    User ubuntu
    IdentityFile ~/.ssh/staging.pem

Host azure
    HostName 20.10.5.6
    User azureuser
    IdentityFile ~/.ssh/azure.pem
Now you simply use:

ssh prod
ssh staging
ssh azure
This is why DevOps engineers almost always use ~/.ssh/config—it makes managing multiple servers much simpler.

