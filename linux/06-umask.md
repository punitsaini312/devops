Imagine you are creating new files
When you run:

touch app.log
mkdir project
You didn't specify permissions.

So Linux asks:

"What permissions should I give these new objects?"

That's where umask comes in.

Step 1: Linux starts with maximum permissions
For files

666
rw-rw-rw-
Why not 777?

Because making every new file executable would be dangerous.

For directories

777
rwxrwxrwx
Directories need execute (x) to allow users to enter them.

Step 2: Apply umask
Suppose

umask 022
Think of it as saying:

"Remove write permission from Group and Others."

022

Owner : 0 -> remove nothing
Group : 2 -> remove write
Others: 2 -> remove write
So

Files

666
022
----
644
Directories

777
022
----
755
That's why almost every Linux machine creates

-rw-r--r--
drwxr-xr-x
by default.

Practical Example 1 (Developer)
Suppose you create

touch config.yml
Without umask, everyone could modify it.

With

umask 022
Result

-rw-r--r--
Others can read it.

Only you can modify it.

Perfect for source code.

Practical Example 2 (Private files)
Suppose you're storing SSH keys.

You don't want anyone reading them.

Use

umask 077
Now

touch id_rsa
becomes

-rw-------
Nobody else can even read it.

Practical Example 3 (Shared Team)
Imagine

Three developers

Alice
Bob
Charlie
Working on the same project.

Administrator sets

umask 002
Now Alice creates

touch app.py
Permissions become

-rw-rw-r--
Notice

Group has write permission.

Bob can edit the file immediately.

This is common in shared development environments.

Practical Example 4 (CI/CD Server)
Jenkins creates

build.log
artifact.zip
If Jenkins used

umask 000
Everyone could modify build artifacts.

Not good.

Instead administrators usually use

022
or

027
to protect them.

Practical Example 5 (Web Server)
Suppose Nginx creates

access.log
Administrator wants

nginx can write
developers can read
public cannot read
So service may start with

umask 027
Result

-rw-r-----
Much safer.

Why DevOps Engineers Care
When a developer says:

"Why is my newly created file 600 instead of 644?"

A DevOps engineer immediately checks

umask
When Jenkins creates files with strange permissions

↓

Check

umask
When Docker containers create files owned by root with unexpected permissions

↓

Check

umask
When shared directories don't let teammates edit files

↓

Check

umask
It's often one of the first things to investigate when default permissions aren't what you expect.

Try It Yourself
Run these commands:

umask
touch file1
mkdir dir1

ls -ld file1 dir1
Then change the mask:

umask 077

touch file2
mkdir dir2

ls -ld file2 dir2
Finally:

umask 002

touch file3
mkdir dir3

ls -ld file3 dir3
Compare the permissions of file1, file2, file3, and the directories. Seeing the differences firsthand makes the concept much easier to remember.

One thing to remember
A common misconception is that umask sets permissions. It doesn't.

It removes permissions from the defaults:

Files start at 666, then umask removes bits.
Directories start at 777, then umask removes bits.
That's why it's called a mask—it masks (hides/removes) permission bits rather than granting them.
