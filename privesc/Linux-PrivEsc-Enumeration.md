# Linux Privilege Escalation – OS Enumeration

OS enumeration is one of the first steps in Linux privilege escalation. It helps identify the target's operating system, kernel version, running processes, scheduled tasks, and installed packages. This information can reveal potential privilege escalation opportunities.

### Hostname

The `hostname` command displays the name assigned to the target machine.

```bash
hostname
```

The hostname may sometimes reveal the machine's role within a network. For example, `SQL-PROD-01` could indicate a production database server.

### Kernel Information

The `uname` command provides information about the system and its running kernel.

```bash
uname -a
```

Example output:

```text
Linux home 6.8.0-41-generic
```

- `home`: Hostname.
- `6.8.0-41-generic`: Kernel version.

The kernel version is important when checking for known vulnerabilities that could allow local privilege escalation.

Other useful options:

```bash
uname -r    # Kernel version
uname -m    # System architecture
```

### /proc/version

The `/proc/version` file provides additional information about the running kernel and how it was built.

```bash
cat /proc/version
```

The output may include the kernel version, build machine, GCC compiler version, and linker information.

### OS Version

The `/etc/issue` file usually contains information about the installed Linux distribution and its release.

```bash
cat /etc/issue
```

For example, the output might identify the system as Ubuntu 22.04.1 LTS.

This file can be modified, so it should not be treated as the only source of OS information.

### Running Processes

The `ps` command displays processes currently running on the system.

```bash
ps
ps aux
ps axjf
```

- `ps aux`: Displays processes across all users, including those without a controlling terminal.
- `ps axjf`: Displays processes in a tree-like structure, showing parent and child relationships.

Processes running with elevated privileges or executing unusual commands may be worth investigating.

### Cron Jobs

Cron is a time-based job scheduler used to execute commands and scripts automatically.

```bash
cat /etc/crontab
ls -la /etc/cron.d/
ls -la /var/spool/cron/
```

Example entry:

```text
*/5 * * * * root /home/ubuntu/cleanup.sh
```

This runs `cleanup.sh` every five minutes as root.

Cron jobs can present privilege escalation opportunities when scripts or their dependencies are writable by lower-privileged users. Checking file permissions is therefore important.

### Installed Packages – dpkg

On Debian-based distributions such as Ubuntu, `dpkg` can list installed packages and their versions.

```bash
dpkg -l
```

To search for a specific package:

```bash
dpkg -l | grep apparmor
```

To view its installed version and package details:

```bash
dpkg -s apparmor
```

Package versions can be useful when checking for outdated software and known vulnerabilities.

---

**Key takeaway:** OS enumeration provides an overview of the target system and helps identify potential weaknesses in its kernel, installed packages, processes, and scheduled tasks.



# Linux Privilege Escalation – User Enumeration

User enumeration helps identify accounts on the target system, their group memberships, privileges, and potentially exposed information. This can reveal users with elevated permissions or opportunities to gain additional access.

### User & Group Information – id

The `id` command displays information about the current user, including their UID, GID, and group memberships.

```bash
id
```

It can also be used to check another user's details:

```bash
id luke
```

Group memberships are particularly useful because some groups may grant access to sensitive files or administrative functions.

### Environment Variables – env

The `env` command displays environment variables associated with the current session.

```bash
env
```

Pay attention to variables such as `PATH`, `HOME`, `USER`, and `LANG`.

The `PATH` variable is especially interesting because it determines where the system searches for executable commands. Unusual or writable directories in the PATH may introduce security weaknesses.

### Command History – history

The `history` command displays previously executed commands.

```bash
history
```

Command history may expose useful information about the system, previous administrative activity, usernames, or accidentally entered credentials and flags.

History files can also be checked directly:

```bash
cat ~/.bash_history
```

### Sudo Permissions – sudo -l

The `sudo -l` command shows which commands the current user is allowed to execute with elevated privileges.

```bash
sudo -l
```

Some configurations allow specific commands to run as root without requiring a password. These permissions should be examined carefully, as certain allowed commands may provide a path to privilege escalation.

### Local Users – /etc/passwd

The `/etc/passwd` file contains account information for users on the system.

```bash
cat /etc/passwd
```

Each entry contains several fields separated by colons:

```text
username:password:UID:GID:comment:home_directory:login_shell
```

The password field usually contains `x`, indicating that password hashes are stored separately in `/etc/shadow`.

To display only usernames:

```bash
cat /etc/passwd | cut -d ":" -f 1
```

To narrow the results to accounts with home directories under `/home`:

```bash
grep "/home" /etc/passwd
```

This helps identify potential interactive users, although not every legitimate user necessarily has a home directory under `/home`.

---

**Key takeaway:** User enumeration helps establish who is on the system, what privileges they have, and whether environment variables, command history, sudo permissions, or account information expose useful opportunities.



# Linux Privilege Escalation – Network Enumeration

Network enumeration helps identify the target's network interfaces, IP addresses, and active connections. This is useful for understanding the machine's network position and identifying services that may not be accessible externally.

### Network Interfaces – ifconfig

The `ifconfig` command displays information about the network interfaces on the system.

```bash
ifconfig
```

The output can reveal:
- Interface names.
- IP addresses and subnet masks.
- MAC addresses.
- Network activity.

For example, an interface named `docker0` may indicate that Docker is running on the host. This is worth investigating because the machine could have access to additional internal networks.

On modern Linux distributions, `ifconfig` has largely been replaced by the `ip` command.

```bash
ip addr
```

### Network Connections – netstat

The `netstat` command provides information about network connections, listening ports, and network statistics.

Some useful options include:

```bash
netstat -a      # All connections and listening ports
netstat -at     # TCP connections
netstat -au     # UDP connections
netstat -l      # Listening ports
netstat -lt     # Listening TCP ports
```

Listening ports are particularly interesting because they reveal services running on the machine.

For example:

```bash
netstat -tuln
```

Displays listening TCP and UDP ports using numeric addresses and port numbers.

### Identifying Services & Processes

The `-p` option displays the PID and program name associated with a connection or listening port.

```bash
netstat -tpln
```

This is useful for identifying which process is responsible for a listening TCP port. Root privileges may be required to display processes owned by other users.

The `-n` option prevents name resolution, displaying numeric IP addresses and port numbers instead.

### Network Statistics & Interfaces

Network statistics can be viewed with:

```bash
netstat -s
```

This displays statistics grouped by network protocol.

To view network interface statistics:

```bash
netstat -i
```

This shows information such as received and transmitted packets, errors, and dropped packets.

### Understanding netstat -ano

A commonly encountered command is:

```bash
netstat -ano
```

Its options are:

- `-a`: Display all sockets.
- `-n`: Display numeric addresses and ports.
- `-o`: Display connection timers.

### Modern Alternative – ss

On modern Linux distributions, `netstat` has largely been replaced by `ss`.

For example:

```bash
ss -tulnp
```

This displays listening TCP and UDP sockets, numeric addresses and ports, and associated processes where permissions allow.

---

**Key takeaway:** Network enumeration provides an overview of the target's network interfaces and active services. Particular attention should be given to additional interfaces, listening ports, and services bound to localhost, as these may reveal internal services that are not directly exposed to external systems.

# Linux Privilege Escalation – File Enumeration

File enumeration involves searching the filesystem for hidden files, sensitive information, unusual permissions, and files that may provide opportunities for privilege escalation.

### Listing Files – ls

The `ls` command is commonly used to list files and directories. During enumeration, `ls -la` is particularly useful because it also displays hidden files (those beginning with a dot).

```bash
ls
ls -l
ls -la
```

For example, a file named `.secret.txt` will not appear when using a normal `ls` command, but will be visible with `ls -la`.

File permissions and ownership are also important, as they help identify files that can be modified or accessed by other users.

### Searching for Files – find

The `find` command searches for files and directories based on specific criteria.

Some useful examples:

```bash
find . -name "flag.txt"
find /home -name "flag.txt"
find / -type d -name "config" 2>/dev/null
```

- Search the current directory and subdirectories.
- Search within `/home`.
- Find directories named `config` across the filesystem.

### Searching by Permissions & Ownership

Files with unusual permissions or ownership may be worth investigating.

```bash
find / -type f -perm 0777 2>/dev/null
find / -perm -a=x 2>/dev/null
find /home -user frank 2>/dev/null
```

To locate writable directories:

```bash
find / -writable -type d 2>/dev/null
```

Writable directories can be interesting when privileged processes execute scripts or access files stored within them.

### Searching by Date & File Size

Files can also be located based on when they were accessed or modified, or how large they are.

```bash
find / -mtime -10 2>/dev/null
find / -atime -10 2>/dev/null
find / -cmin -60 2>/dev/null
find / -amin -60 2>/dev/null
find / -size +50M 2>/dev/null
```

- `-mtime`: Modified within the specified number of days.
- `-atime`: Accessed within the specified number of days.
- `-cmin`: Metadata changed within the specified number of minutes.
- `-amin`: Accessed within the specified number of minutes.
- `-size`: Search based on file size.

For example:

```bash
find / -size +100M 2>/dev/null
```

Finds files larger than 100 MB.

The `2>/dev/null` part suppresses permission errors and other error messages, making the output easier to read.

### Understanding File Permissions with find

The `-perm` option can be used to search for files and directories with specific permissions.

For example:

```bash
find / -perm -o=w -type d 2>/dev/null
```

Finds directories where the owner, group, or others? 

**Correction:** `-o=w` specifically checks for write permission granted to others (world-writable directories).

Other useful examples:

```bash
find / -perm -u=x -type f 2>/dev/null
find / -perm -u=s -type f 2>/dev/null
find / -perm -g=s -type f 2>/dev/null
```

- `-u=x`: Files with owner execute permission.
- `-u=s`: Files with the SUID bit set.
- `-g=s`: Files with the SGID bit set.

SUID files are particularly interesting because they execute with the privileges of their file owner. A misconfigured or exploitable SUID binary could provide a privilege escalation path.

### Searching for Development Tools

Installed interpreters and compilers can be useful during privilege escalation, particularly when executing scripts or compiling exploits.

```bash
find / -name "perl*" 2>/dev/null
find / -name "python*" 2>/dev/null
find / -name "gcc*" 2>/dev/null
```

### Searching with Wildcards

Wildcards help locate files when the exact filename is unknown.

For example:

```bash
find / -name "pass*.txt" 2>/dev/null
```

This can return filenames such as `pass.txt`, `password.txt`, and `passwords.txt`.

---

**Key takeaway:** File enumeration helps uncover hidden files, sensitive information, unusual permissions, and potentially vulnerable binaries. Pay particular attention to writable files, SUID/SGID binaries, and files that may contain credentials.

