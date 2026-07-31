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

====================================================================================================================================================================================================================================================================================

Excellent! This is the point where most people get confused. The key is:

umask is NOT normal subtraction.

It removes permission bits, not arithmetic values.

Let's use your example.

Step 1: You ran
umask 077
Meaning:

Owner  : 0
Group  : 7
Others : 7
Now think of each digit separately.

Each permission digit consists of:

Read    = 4
Write   = 2
Execute = 1
So:

7 = 4 + 2 + 1
  = rwx
Therefore:

umask 077

Owner  -> remove nothing
Group  -> remove rwx
Others -> remove rwx
Example 1: Creating a file
Linux first creates a file with

666

Owner  rw-
Group  rw-
Others rw-
Now apply the mask.

Default file

Owner   rw-
Group   rw-
Others  rw-

umask 077

Owner   remove nothing
Group   remove rwx
Others  remove rwx
Result:

Owner   rw-
Group   ---
Others  ---
which is

600
Exactly what you got:

-rw-------
Example 2: Creating a directory
Directories start with

777

Owner  rwx
Group  rwx
Others rwx
Apply the mask

Owner   keep rwx

Group   remove rwx

Others  remove rwx
Result

700
Exactly what you got:

drwx------
Visual Table
Files
Default

Owner   rw-
Group   rw-
Others  rw-

umask 077

Owner   keep
Group   remove rw-
Others  remove rw-

Result

Owner   rw-
Group   ---
Others  ---
→ 600

Directories
Default

Owner   rwx
Group   rwx
Others  rwx

umask 077

Owner   keep
Group   remove rwx
Others  remove rwx

Result

Owner   rwx
Group   ---
Others  ---
→ 700

Why isn't it arithmetic subtraction?
Suppose you do

umask 033
If it were arithmetic:

666 - 033 = 633
That isn't how Linux thinks.

Instead Linux removes permission bits.

Default file

rw- rw- rw-

Mask

-wx -wx

Remove those permissions

Result

rw- r-- r--
which is

644
So umask is really a permission mask, not a calculator.

The easiest way to remember
Imagine this conversation:

Linux:

"I'll create every new file as 666."

umask 022:

"Wait! Remove write from Group and Others."

Result:

644
Linux:

"I'll create every new directory as 777."

umask 077:

"Remove everything from Group and Others."

Result:

700
A rule that DevOps engineers remember
Never think:

666 - 022
Instead think:

Default permissions
        ↓
Remove the bits specified by umask
        ↓
Final permissions
This "remove permission bits" mental model will help you understand every umask value, including 022, 027, 002, and 077.
