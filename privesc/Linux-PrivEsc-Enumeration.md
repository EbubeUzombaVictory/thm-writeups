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
