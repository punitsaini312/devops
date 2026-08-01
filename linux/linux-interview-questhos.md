Q1: What is Linux?

Ans: **Linux is an open-source operating system that manages computer hardware and software resources. It's widely used for its stability, security, and flexibility. Many servers, supercomputers, and embedded systems run on Linux. It's popular among developers because they can modify and distribute their own versions.**

Q2: What is Kernal?

Ans: **A kernel is the core part of an operating system. It acts as a bridge between applications and the computer's hardware. The kernel manages system resources, such as the CPU, memory, and devices, ensuring everything runs smoothly and efficiently. It handles tasks like executing processes, managing memory, and facilitating communication between hardware and software.**

Q3: what is the shell?

Ans: **In Linux, a "shell" is a program that allows you to interact with the operating system by typing commands. It's like a translator between you and the computer, interpreting what you type and then telling the computer what to do.**

In Linux, five Shells are used:

•csh (C Shell): This shell offers job control and spell checking and is similar to C syntax.]

•ksh (Korn Shell): A high-level shell for programming languages.

•ssh (Z Shell): This shell has a unique nature, such as closing comments, startup files, file name generating, and observing logout/login watching.

•bash (Bourne Again Shell): This is the default shell for Linux.

•Fish (Friendly Interactive Shell): This shell provides auto-suggestion, web-based configuration, etc

Q4. What is a root account?

Ans: **The root is like the user’s name or system administrator account in Linux. The root account provides complete system control, which an ordinary user cannot do.**

Q5. What is softlink and hardlink?

Ans. ### Soft Links (Symbolic Links)

1. **Definition**: A soft link, also known as a symbolic link, is a type of file that points to another file or directory.

2. **Behavior**: If you delete the original file, the soft link becomes broken because it references a non-existent file.

3. **Usage**: Useful for creating shortcuts and linking files across different filesystems.

4. **Creation**: Created using the `ln -s` command. Example: `ln -s target_file link_name`.

### Hard Links1. 

**Definition**: A hard link is an additional directory entry for an existing file, effectively creating another name for the same file data

2.**Behavior**: Both the original and hard link entries point to the same inode. Deleting one does not affect the other; the data remains until all hard links are deleted.

3. **Usage**: Useful for ensuring data availability and reducing redundancy within the same filesystem.

4. **Creation**: Created using the `ln` command. Example: `ln target_file link_name`.

### Key Differences

1. **Inode Sharing**: Hard links share the same inode as the original file, while soft links have a different inode and just point to the original file's name.

2. **Filesystem**: Hard links cannot span different filesystems; soft links can.

3. **Directory Linking**: Soft links can link to directories; hard links typically do not.

Q .5 What do you understand about the standard streams?

Ans:  Standard streams are pre-connected input and output communication channels between a program and its environment:
1. **Standard Input (stdin)**: The default source of input for the program, typically from the keyboard. Accessed in code via `stdin` (e.g., `scanf` in C, `input()` in Python).

2. **Standard Output (stdout)**: The default destination for program output, typically the terminal or console. Used for displaying output to the user (e.g., `printf` in C, `print()` in Python).

3. **Standard Error (stderr)**: The default destination for error messages and diagnostics, also typically the terminal. Separates error messages from regular output to avoid confusion (e.g., `fprintf(stderr, "error message")

Q6. What is the find command, and how do you use it?

Ans. The find command searches for files based on different factors such as name, size, permissions, etc. 

Here is the basic command:

find <directory> <file>

For example, let’s find a Linux.txt file located in the Downloads directory through the below command:

find ~/Downloads -name Linux.txt

Once you run the above command, the find command will start finding the Linux.txt in the Downloads directory and subdirectories.

Q7. What is a Linux virtual memory system?

Ans. Virtual memory is a great memory management utility in any OS. You can use the virtual memory system as **secondary memory**. This memory is used by both software and hardware in Linux so that your system can cope with the lack of physical memory. Moreover, virtual memory is also used to compensate for the RAM usage by transferring the data temporarily from RAM to disk storage.

Q8. What is the init process in Linux?

Ans. The init or also called the initialization process is the first process that begins during the system boot. It is responsible for initializing and processing the system in its functional state. Hence, init works as the parent process because its process ID is 1. Originally Linux systems used to have SysV init, but now it is developed as the systemd init (an improved version of SysV).

Q9. What is LVM in Linux?

Ans. The full form of LVM is Logical Volume Manager, which provides an advanced disk management approach in Linux. It is a subsystem that allows a user to efficiently allocate the disk space on the physical storage device. You can use the LVM to create the logical volume for easy storage management through various features like resizing, volume mirroring, and snapshots. LVM is a powerful utility for disk management where you need dynamic storage allocations.

Q10. What is the difference between UDP and TCP?

Ans. TCP (Transmission Control Protocol)

Connection-oriented: Establishes a connection between sender and receiver before data transmission, ensuring a reliable communication channel.

Reliability: Provides error checking, acknowledgment of data receipt, and retransmission of lost packets, guaranteeing data integrity and order.

Flow Control: Uses mechanisms like windowing to control the flow of data, preventing network congestion and ensuring efficient data transfer.

Use Cases: Ideal for applications where reliability and data integrity are crucial, such as web browsing (HTTP/HTTPS), email (SMTP, IMAP, POP3), and file transfers (FTP).

**UDP (User Datagram Protocol)**

Connectionless: Sends data without establishing a connection, leading to faster communication but without guaranteed delivery.

No Reliability: Lacks error checking, acknowledgments, and retransmission mechanisms, making it suitable for applications where speed is more critical than reliability.

Lower Overhead: Minimal protocol overhead, resulting in reduced latency and faster data transmission.

Use Cases: Suitable for real-time applications where occasional data loss is acceptable, such as video streaming, online gaming, and VoIP (Voice over IP).

Q11. What is /etc/resolv.conf file?

Ans. The /etc/resolv.conf is the config file used for the DNS server resolution process. This config file is used to specify the DNS server, set up the search directive for domains, and configure the resolver options.

Q12. What is the difference between absolute and relative paths in Linux?

Ans. Absolute path = It specifies the exact location of a file or directory from the root directory (“/”). We will notice that they always start with a forward slash (“/”).For Example: `/home/user/jayesh/geeksforgeeks.txt`

Relative paths = It specifies the location relative to the current working directory. In this we do not start with a forward slash (“/”).For Example: `documents/file.txt`

Q13. What is the grep command used for in Linux?
Ans. The grep command is used to search for specific patterns within files or input
streams. It allows us to find and print lines that we give to match the pattern.
For example: If we want to search `test` in a text file name “file.txt”. We use the
following command
grep "test" file.txt
This command will search for the word `test` in the file named “file.txt” and print the
matching lines.

Q14. How do you check the status of a service or daemon in Linux?

Ans. Systemctl status httpd, systemctl start httpd, systemctl enable —now httpd.

Q15. What is the difference between /etc/passwd and /etc/shadow files?

Ans. The /etc/passwd file stores essential user information like usernames, user IDs, home directories, and default shells. Each line in the file represents a user account.

The /etc/shadow file contains encrypted passwords and other security-related information. It is only accessible by the root user or privileged processes.

Q16.  What is the difference between a process and a daemon in Linux?

Ans. A process is an executing instance of a program. It can be a foreground process that interacts with the user or a background process started by a user or another process.

A daemon is a background process that runs independently of user sessions. It is typically started at system boot time and performs system tasks or provides services. Daemons often have no user interaction and continue running even when users log out.

Q17. How do you schedule recurring tasks in Linux?
Ans. We can use `crontab` command for performing recurring tasks in Linux. By adding
entries to the crontab file, we can specify when and how frequently a command or
script should be executed
For Example: If we want to execute a script name “geeks.sh” every day at 3:30 AM.
We use the following command.

crontab -e
This command opens the crontab file in an editor.
30 3 * * * /path/to/geeks.sh

Q18. What is the sed command used for in Linux?
Ans. The sed command is used to perform text transformations on files. It can search for
specific patterns and replace them with desired text.
For Example:
sed `s/foo/bar/g` file.txt
This command replaces all occurrences of “foo” with “bar” in the file name “file.txt”

Q19.  What is sudo in Linux?

Ans. The word “sudo” is the short form of “Superuser Do” that allows you to run the command with system privileges. With this command, you can get the system’s administrative access to perform various tasks. The sudo command requires a password before the execution to verify the user’s authorization.

Q20. What is umask?

Ans. It is used for user file creation mode. When a user creates any file, then it has default file permission. Umask specifies restrictions for these permissions on the file, i.e., controls the permissions.

Q21. How to find and kill a process in Linux?

Ans. You can use different commands to kill a process, but first, you must find the PID of that specific process. So, please run the below command:ps aux | grep <process>Once you get the PID of the process then run the kill command to end it:kill <PID>

If you don’t want to find the PID, then you can use the pkill command to kill a process by its name:pkill <process>The pkill command sends a signal (by default, SIGTERM) to the matched processes, causing them to terminate.

Q22. What is SELinux?

Ans. SELinux or also known as Security-Enhanced Linux, is the security framework. It offers an additional layer of security to improve access control and strengthen security. SELinux was developed to improve the security policies to prevent unauthorized access and exploitation. However, learning about SELinux is essential before working on it can create serious security issues.

Q23.What is the purpose of the SSH protocol in Linux, and how do you securely connect to a remote server using SSH?

Ans. The Secure Shell (SSH) is a protocol in Linux that is used to establish a secure encrypted connection between a local and remote machine. It allows you to securely access and manage remote servers. If we want to connect to a remote server using SSH. We can use the following command.ssh username@remote_ipHere replace the `username` with the desired username of the remote server and replace the `remote_ip` with the IP address of the remote server.

Q24.What is the purpose of the sudoers file in Linux, and how do you configure sudo access for users?

Ans. The sudoers file in Linux controls the sudo access permissions for users. It determines which users are allowed to run commands with superuser (root) privileges. To configure sudo access, you can edit the sudoers file using the visudo command. For example, sudo visudoNow add this line anywhere in the file. For instance, if we want to grant a user full sudo access.user_name ALL=(ALL) ALL

Q25.What is the purpose of the netstat command in Linux, and how do
Do you view network connections and listening ports?
Ans. The netstat command in Linux is used to display active network connections, routing
tables, and listening ports. To view network connections and listening ports, use the
netstat command with appropriate options.
For example: If we want to display all listening TCP ports, we can use the following
command.

Q26.How do you configure a DNS server in Linux?
Ans. DNS server configuration involves editing the ‘/etc/named.conf’ (BIND) or
‘/etc/named/named.conf.options’ (ISC BIND) file to specify the server’s zone
information, name resolution options, and defining forwarders or root hints.

!Screenshot_2024-07-16-22-23-09-487_com.linkedin.android.jpg
