# 02 — Users, Permissions & Processes

> "Linux security is built on three pillars: every file has an owner, every process has an identity, and every action requires the right permissions."

---

## Table of Contents

1. [Users and Groups](#1-users-and-groups)
2. [Managing Users](#2-managing-users)
3. [Managing Groups](#3-managing-groups)
4. [File Permissions](#4-file-permissions)
5. [Changing Permissions and Ownership](#5-changing-permissions-and-ownership)
6. [Special Permissions — SUID, SGID, Sticky Bit](#6-special-permissions--suid-sgid-sticky-bit)
7. [sudo and the sudoers File](#7-sudo-and-the-sudoers-file)
8. [Process Management](#8-process-management)
9. [Signals](#9-signals)
10. [Background Jobs and Job Control](#10-background-jobs-and-job-control)
11. [Resource Limits and Priority](#11-resource-limits-and-priority)
12. [Summary & Cheatsheet](#12-summary--cheatsheet)

---

## 1. Users and Groups

### Core Concept

Every file, directory, and running process in Linux is owned by a user and a group. When you try to access something, the kernel checks: **Are you the owner? Are you in the group? Otherwise, what are the "other" permissions?**

### Key Files

```bash
cat /etc/passwd      # List of all users
# Format: username:x:UID:GID:comment:home:shell
# Example:
# root:x:0:0:root:/root:/bin/bash
# ajaz:x:1000:1000:Ajaz Beig:/home/ajaz:/bin/bash
# nginx:x:101:101:nginx user:/var/cache/nginx:/sbin/nologin
# nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin

# Fields explained:
# username       → login name
# x              → password stored in /etc/shadow
# UID            → User ID (0 = root, 1-999 = system users, 1000+ = humans)
# GID            → Primary group ID
# comment        → Full name / description
# home           → Home directory
# shell          → Login shell (/sbin/nologin = no login allowed)

cat /etc/shadow      # Hashed passwords (only root can read)
# Format: username:hashed_password:last_changed:min:max:warn:inactive:expire

cat /etc/group       # List of all groups
# Format: groupname:x:GID:members
# Example:
# sudo:x:27:ajaz
# docker:x:999:ajaz,ubuntu
# www-data:x:33:
```

### Who Am I?

```bash
whoami               # Current username
id                   # UID, GID, and all group memberships
# uid=1000(ajaz) gid=1000(ajaz) groups=1000(ajaz),27(sudo),999(docker)

id username          # Info about another user
groups               # List groups current user belongs to
groups username      # Groups for another user

w                    # Who is logged in and what are they doing
who                  # Who is logged in
last                 # Login history
last -n 20           # Last 20 logins
lastlog              # Last login time for every user
```

### UID Special Values

| UID Range | Purpose |
|-----------|---------|
| 0 | root — superuser, can do anything |
| 1–99 | Traditional static system accounts |
| 100–999 | Dynamically allocated system accounts (services) |
| 1000+ | Regular human users |
| 65534 | nobody (used for NFS, security sandboxing) |

---

## 2. Managing Users

### Creating Users

```bash
# useradd — low-level tool (use with many flags)
useradd username                           # Create user (minimal)
useradd -m -s /bin/bash -c "Full Name" username   # With home dir + shell + comment
useradd -m -G sudo,docker -s /bin/bash deployuser # Add to groups

# adduser — high-level wrapper (interactive, more user-friendly on Debian/Ubuntu)
adduser username                           # Interactive wizard

# Flags for useradd:
# -m          → Create home directory
# -s /bin/bash → Set login shell
# -c "comment" → Set GECOS (full name/description)
# -G group1,group2 → Add to supplementary groups
# -u UID      → Specify UID
# -d /path    → Set home directory path

# In DevOps — creating service accounts (no login allowed):
useradd -r -s /sbin/nologin -c "Prometheus" prometheus
# -r = system account (UID < 1000)
# -s /sbin/nologin = cannot log in interactively
```

### Managing Passwords

```bash
passwd username               # Set/change password for user (root only for others)
passwd                        # Change your own password
passwd -l username            # Lock account (prepend ! to hash in shadow)
passwd -u username            # Unlock account
passwd -e username            # Expire password — force change on next login
passwd -d username            # Delete password (no password required — insecure!)

# Check password status
passwd -S username
# ajaz P 2026-01-15 0 99999 7 -1
# Fields: user, status(P=set/L=locked/NP=none), last-change, min, max, warn, inactive
```

### Modifying and Deleting Users

```bash
usermod -s /bin/bash username        # Change shell
usermod -aG docker username          # Add to docker group (-a = append, don't replace)
usermod -G "" username               # Remove from all supplementary groups
usermod -d /new/home -m username     # Change and move home directory
usermod -l newname oldname           # Rename user
usermod -L username                  # Lock user
usermod -U username                  # Unlock user

userdel username                     # Delete user (keep home dir)
userdel -r username                  # Delete user AND home directory AND mail spool

# After adding user to a group, they must log out/in for it to take effect
# Or use: newgrp docker  (apply new group in current session)
```

---

## 3. Managing Groups

```bash
groupadd mygroup                     # Create group
groupadd -g 1500 mygroup             # Create with specific GID

groupmod -n newname mygroup          # Rename group
groupmod -g 1600 mygroup             # Change GID

groupdel mygroup                     # Delete group

# Add user to group
usermod -aG groupname username
# OR:
gpasswd -a username groupname        # Add to group
gpasswd -d username groupname        # Remove from group
gpasswd -A username groupname        # Make user group admin

# View group membership
getent group docker                  # Show docker group and all members
# docker:x:999:ajaz,ubuntu,jenkins
```

---

## 4. File Permissions

### Reading Permission Strings

Every file has 10 characters in the permission field:

```
-  r w x  r w x  r w x
^  ^^^^^  ^^^^^  ^^^^^
|  Owner  Group  Others
|
File type (- file, d dir, l symlink, etc.)
```

```bash
ls -l /etc/nginx/nginx.conf
# -rw-r--r-- 1 root root 2611 Mar 28 nginx.conf
#  ^^^  ^^^  ^
#  rw-  r--  r--
#  owner can read+write
#        group can only read
#               others can only read

ls -l /bin/passwd
# -rwsr-xr-x 1 root root 63960 /usr/bin/passwd
#    ^
#    s = SUID (run as owner, not caller)
```

### Permission Types

| Symbol | Value | Meaning on File | Meaning on Directory |
|--------|-------|----------------|---------------------|
| `r` | 4 | Read file content | List directory contents (`ls`) |
| `w` | 2 | Write/modify file | Create/delete files in dir |
| `x` | 1 | Execute file | Enter directory (`cd`) |
| `-` | 0 | No permission | No permission |

### Octal Notation

Each permission group (owner/group/other) is represented as a number 0–7:

```
r=4, w=2, x=1

rwx = 4+2+1 = 7
rw- = 4+2+0 = 6
r-x = 4+0+1 = 5
r-- = 4+0+0 = 4
-wx = 0+2+1 = 3
-w- = 0+2+0 = 2
--x = 0+0+1 = 1
--- = 0+0+0 = 0
```

Common permission values:

| Octal | Symbolic | Use Case |
|-------|----------|---------|
| `777` | `rwxrwxrwx` | Everyone can do everything (dangerous!) |
| `755` | `rwxr-xr-x` | Standard for executables, public dirs |
| `750` | `rwxr-x---` | Owner + group can access, others cannot |
| `644` | `rw-r--r--` | Standard for config files, web content |
| `640` | `rw-r-----` | Config with sensitive data (owner+group only) |
| `600` | `rw-------` | Private files (SSH keys, .env files) |
| `400` | `r--------` | Read-only, extra protected |
| `700` | `rwx------` | Private executable (only owner) |

---

## 5. Changing Permissions and Ownership

### `chmod` — Change Mode (Permissions)

```bash
# Numeric/octal syntax
chmod 755 script.sh          # Owner: rwx, Group: r-x, Others: r-x
chmod 644 config.conf        # Owner: rw-, Group: r--, Others: r--
chmod 600 ~/.ssh/id_rsa      # Owner: rw-, no one else (SSH key requirement)
chmod 400 secret.key         # Owner: r--, no one else

# Recursive
chmod -R 755 /var/www/html/  # Apply to directory and all contents

# Symbolic syntax (add/remove specific permissions)
chmod +x script.sh           # Add execute for all (owner, group, other)
chmod -x script.sh           # Remove execute for all
chmod u+x script.sh          # Add execute for user/owner only
chmod g-w file.txt           # Remove write from group
chmod o-rwx file.txt         # Remove all permissions from others
chmod u=rw,g=r,o= file.txt   # Set exactly: owner rw, group r, others nothing

# Common DevOps chmod operations:
chmod 755 /usr/local/bin/my-script.sh   # Make a script executable system-wide
chmod 644 /etc/nginx/nginx.conf          # Config readable by nginx user
chmod 700 /home/deploy/.ssh/             # SSH dir — private
chmod 600 /home/deploy/.ssh/authorized_keys  # SSH keys — private

# umask — sets default permissions for new files
umask                        # Show current umask (e.g., 022)
# umask 022 means new files get 644 (666-022) and dirs get 755 (777-022)
umask 027                    # More restrictive: files 640, dirs 750
```

### `chown` — Change Ownership

```bash
chown user file.txt              # Change owner only
chown user:group file.txt        # Change owner and group
chown :group file.txt            # Change group only (same as chgrp)
chown -R user:group directory/   # Recursive

# Practical examples:
chown -R www-data:www-data /var/www/html/    # Give nginx/apache ownership
chown deploy:deploy /opt/myapp/              # App owned by deploy user
chown root:root /usr/local/bin/mytool        # System tool owned by root

chgrp docker /var/run/docker.sock            # Change group on docker socket
```

---

## 6. Special Permissions — SUID, SGID, Sticky Bit

### SUID (Set User ID) — Bit 4

When set on an executable, it runs as the file's owner, not the user executing it.

```bash
ls -l /usr/bin/passwd
# -rwsr-xr-x 1 root root 63960 /usr/bin/passwd
#    ^
#    s = SUID

# passwd needs to write to /etc/shadow (owned by root)
# Without SUID, regular users couldn't change their own passwords
# With SUID, passwd runs as root even when run by ajaz

chmod u+s file           # Set SUID
chmod 4755 file          # Set SUID (4 prefix) + 755

# Security: Find all SUID files on your system
find / -perm -4000 -type f 2>/dev/null
# Any unexpected SUID files could be a security risk
```

### SGID (Set Group ID) — Bit 2

On a file: runs as the file's group.
On a directory: new files created inside inherit the directory's group.

```bash
ls -l /usr/bin/wall
# -rwxr-sr-x 1 root tty 27968 /usr/bin/wall
#       ^
#       s = SGID

# SGID on directories — great for shared project dirs:
mkdir /opt/projectteam
chown root:devteam /opt/projectteam
chmod 2775 /opt/projectteam    # 2 prefix = SGID

# Now any file created in /opt/projectteam belongs to group 'devteam'
# regardless of who creates it

chmod g+s directory/   # Set SGID on directory
chmod 2755 directory/  # Set SGID (2 prefix) + 755
```

### Sticky Bit — Bit 1

On a directory: users can only delete their own files, even if they have write permission.

```bash
ls -ld /tmp
# drwxrwxrwt 1 root root 4096 Mar 28 /tmp
#          ^
#          t = Sticky bit

# /tmp is world-writable BUT you can't delete other users' files
# This is the sticky bit in action

chmod +t /shared/uploads    # Set sticky bit
chmod 1777 /shared/uploads  # Set sticky bit (1 prefix) + 777

# Very common in DevOps for shared upload directories
```

---

## 7. sudo and the sudoers File

### What is sudo?

`sudo` (Super User Do) allows a permitted user to run commands as root (or another user) without knowing the root password. It logs every command — crucial for auditability.

```bash
sudo command                    # Run command as root
sudo -u username command        # Run as specific user
sudo -i                         # Interactive root shell (full environment)
sudo -s                         # Shell as root (keeps current environment)
sudo -l                         # List what sudo commands YOU can run
sudo !!                         # Re-run last command with sudo

# Check sudo logs
sudo grep sudo /var/log/auth.log    # On Debian/Ubuntu
sudo grep sudo /var/log/secure      # On RHEL/CentOS
```

### The `/etc/sudoers` File

**Never edit sudoers directly with `vi` — always use `visudo`!** It validates syntax before saving, preventing you from locking yourself out.

```bash
sudo visudo                        # Safe editor for sudoers
sudo visudo -f /etc/sudoers.d/myapp  # Edit a drop-in file
```

Understanding the sudoers format:

```
# Syntax: WHO WHERE = (AS_WHOM) WHAT

# Allow root to run anything
root    ALL=(ALL:ALL) ALL

# Allow sudo group to run anything with password
%sudo   ALL=(ALL:ALL) ALL

# Allow deploy user to restart nginx without password
deploy  ALL=(root) NOPASSWD: /bin/systemctl restart nginx, /bin/systemctl reload nginx

# Allow developers group to run docker commands
%developers ALL=(root) NOPASSWD: /usr/bin/docker

# Allow monitoring user to read logs
monitoring ALL=(root) NOPASSWD: /bin/cat /var/log/*, /usr/bin/tail /var/log/*

# Wildcards — be careful!  * is a wildcard for arguments
# This allows running any docker command:
deploy ALL=(root) NOPASSWD: /usr/bin/docker *
```

### Best Practices for sudo in DevOps

```bash
# 1. Create a sudoers drop-in file (safer than editing /etc/sudoers directly)
sudo visudo -f /etc/sudoers.d/deploy
# Add: deploy ALL=(root) NOPASSWD: /bin/systemctl

# 2. Never use NOPASSWD: ALL in production (audit nightmare)

# 3. Use specific command paths, not command names
# BAD:  deploy ALL=(root) NOPASSWD: systemctl
# GOOD: deploy ALL=(root) NOPASSWD: /bin/systemctl restart myapp

# 4. Check who has sudo access
grep -r "ALL=(ALL" /etc/sudoers /etc/sudoers.d/
getent group sudo
```

---

## 8. Process Management

### What is a Process?

Every running program is a process with:
- A **PID** (Process ID)
- A **PPID** (Parent Process ID — who spawned it)
- An **owner** (UID)
- A **state** (running, sleeping, zombie, stopped)

### Viewing Processes

```bash
ps                           # Your processes in current terminal
ps aux                       # ALL processes, all users, detailed
# a = all users
# u = user-oriented format
# x = include processes not attached to terminal

# ps aux output columns:
# USER  PID  %CPU  %MEM  VSZ   RSS  TTY   STAT  START  TIME  COMMAND
# root  1     0.0   0.1  168k  12m  ?     Ss   Mar27   0:03  /sbin/init

ps aux | grep nginx          # Find nginx processes
ps -ef                       # Alternative full listing
ps --forest                  # Show process tree
pstree                       # Visual process tree
pstree -p                    # Include PIDs in tree

# Process states in STAT column:
# R = Running
# S = Sleeping (interruptible)
# D = Sleeping (uninterruptible — usually waiting for I/O)
# Z = Zombie (finished but not collected by parent)
# T = Stopped
# s = Session leader
# + = Foreground process
# < = High priority
# N = Low priority
```

### `top` — Real-Time Process Monitor

```bash
top                          # Interactive process viewer
# Inside top:
# q          → Quit
# P          → Sort by CPU
# M          → Sort by Memory
# k          → Kill a process (enter PID)
# r          → Renice (change priority)
# 1          → Show per-CPU stats
# Shift+H    → Show threads

htop                         # Better version of top (may need: apt install htop)
# Arrow keys to navigate
# F6 = sort by column
# F9 = kill process
# F5 = tree view
# F4 = filter

# top output header explained:
# top - 14:23:01 up 5 days, 2:31, 2 users, load average: 0.08, 0.12, 0.10
# Tasks: 142 total, 1 running, 141 sleeping, 0 stopped, 0 zombie
# %Cpu(s):  2.3 us, 0.7 sy, 0.0 ni, 96.8 id, 0.2 wa, 0.0 hi, 0.0 si
# MiB Mem :  7841.2 total,  4291.0 free,  1823.4 used,  1726.8 buff/cache

# Load average: 0.08, 0.12, 0.10
# = 1-min, 5-min, 15-min averages
# On a 4-core system: 4.0 = fully loaded, >4.0 = overloaded
```

### Finding Processes

```bash
pgrep nginx                  # Find PIDs of processes named nginx
pgrep -l nginx               # PID + process name
pgrep -u www-data            # All processes owned by www-data
pgrep -a nginx               # Full command line

pidof nginx                  # Get PID(s) of nginx
cat /proc/$(pgrep nginx)/status  # Inspect a process via /proc
```

---

## 9. Signals

Signals are notifications sent to processes — the OS's way of telling a process to do something.

### Common Signals

| Signal | Number | Default Action | Purpose |
|--------|--------|----------------|---------|
| `SIGHUP` | 1 | Terminate | Hang up — reload config in many daemons |
| `SIGINT` | 2 | Terminate | Ctrl+C — polite interrupt |
| `SIGQUIT` | 3 | Core dump | Ctrl+\ — quit with core dump |
| `SIGKILL` | 9 | Kill | Forced kill — cannot be caught or ignored |
| `SIGTERM` | 15 | Terminate | Default kill — graceful shutdown |
| `SIGSTOP` | 19 | Stop | Pause process — cannot be ignored |
| `SIGCONT` | 18 | Continue | Resume stopped process |
| `SIGUSR1` | 10 | Terminate | Application-defined |
| `SIGUSR2` | 12 | Terminate | Application-defined |

### Sending Signals

```bash
# kill — send signal to process by PID
kill PID                     # Send SIGTERM (15) — graceful shutdown
kill -15 PID                 # Explicit SIGTERM
kill -9 PID                  # SIGKILL — immediate, forceful, no cleanup
kill -1 PID                  # SIGHUP — often used to reload config

# pkill — kill by name
pkill nginx                  # SIGTERM to all nginx processes
pkill -9 nginx               # SIGKILL all nginx processes
pkill -HUP nginx             # SIGHUP (reload) all nginx processes
pkill -u www-data            # Kill all processes of user www-data

# killall — similar to pkill
killall nginx                # SIGTERM all processes named 'nginx'
killall -9 python3           # SIGKILL all python3 processes

# The right order for stopping a process:
kill PID           # 1. Try SIGTERM first (graceful)
sleep 5
kill -0 PID 2>/dev/null && kill -9 PID  # 2. Check if still running, then SIGKILL

# Common SIGHUP use — nginx config reload:
nginx -t && kill -HUP $(cat /run/nginx.pid)   # Test config, then reload
systemctl reload nginx  # (cleaner way to do the same)
```

---

## 10. Background Jobs and Job Control

### Running Processes in Background

```bash
command &                    # Run command in background
sleep 100 &
# [1] 12345  ← job number and PID

jobs                         # List background jobs in this shell
jobs -l                      # Include PIDs

fg                           # Bring most recent background job to foreground
fg %1                        # Bring job 1 to foreground
bg %1                        # Send stopped job 1 to background

# Ctrl+Z                     # Suspend (pause) current foreground job
# Then:
bg %1                        # Resume it in background
fg %1                        # Resume it in foreground

# Kill a background job
kill %1                      # Kill job 1 by job number
kill %nginx                  # Kill job named nginx
```

### `nohup` and `disown` — Survive Shell Exit

```bash
# nohup — run command immune to hangup signal
nohup ./long-script.sh &                   # Run in background, survive logout
nohup ./long-script.sh > output.log 2>&1 & # Redirect both streams

# disown — detach a running job from shell
./long-script.sh &
disown %1                    # Detach from shell — won't be killed on logout
disown -a                    # Disown all jobs
```

### `screen` and `tmux` — Persistent Sessions

For production servers, use a terminal multiplexer to keep processes running:

```bash
# tmux (preferred)
tmux new -s deployment       # New named session
tmux attach -t deployment    # Reattach
tmux ls                      # List sessions
# Ctrl+B d                   # Detach (leave session running)

# screen
screen -S deployment         # New named session
screen -r deployment         # Reattach
screen -ls                   # List sessions
# Ctrl+A d                   # Detach
```

---

## 11. Resource Limits and Priority

### Process Priority with `nice` and `renice`

Priority range: -20 (highest) to 19 (lowest). Default is 0.

```bash
nice -n 10 ./backup.sh           # Start with low priority (won't compete with app)
nice -n -5 ./critical-task.sh    # Higher priority (requires sudo for negative)
sudo nice -n -20 ./urgent.sh     # Highest priority

renice 15 -p 12345               # Change priority of running process
renice 15 -u ajaz                # Change all processes of a user
renice -n -5 -p $(pgrep nginx)   # Increase nginx priority

# Check nice value in ps:
ps aux | awk '{print $1, $2, $18, $11}' | head -20
# USER  PID  NI  COMMAND
```

### System Resource Monitoring

```bash
free -h                      # Memory usage
free -h -s 2                 # Update every 2 seconds

vmstat 1 5                   # Virtual memory stats every 1s, 5 times
vmstat -s                    # Summary stats

iostat                       # I/O stats (apt/yum install sysstat)
iostat -x 1 5                # Extended, 1s interval, 5 times

sar -u 1 5                   # CPU utilization
sar -r 1 5                   # Memory utilization
sar -n DEV 1 5               # Network stats

uptime                       # System uptime and load average
cat /proc/loadavg            # Raw load average
cat /proc/meminfo            # Detailed memory info
cat /proc/cpuinfo            # CPU info
nproc                        # Number of processors
```

### `lsof` and Open Files

```bash
lsof                         # All open files (very verbose)
lsof -u ajaz                 # Files opened by user
lsof -p 12345                # Files opened by PID
lsof -i :80                  # Processes listening on port 80
lsof -i :443                 # Processes using port 443
lsof /var/log/nginx/access.log  # What has this file open

# Which process is using port 8080?
lsof -i :8080
# COMMAND  PID   USER   FD   TYPE   DEVICE SIZE/OFF NODE NAME
# java    4321  ajaz   123u  IPv6  12345      0t0  TCP *:8080 (LISTEN)
```

---

## 12. Summary & Cheatsheet

### User Management Cheatsheet

```bash
useradd -m -s /bin/bash user    # Create user with home dir
passwd user                     # Set password
usermod -aG docker user         # Add to group (append)
userdel -r user                 # Delete user + home dir
id user                         # Show UID/GID/groups
```

### Permissions Cheatsheet

```bash
chmod 755 file      # rwxr-xr-x (executable)
chmod 644 file      # rw-r--r-- (config file)
chmod 600 file      # rw------- (private)
chmod +x file       # Add execute for all
chmod -R 755 dir/   # Apply recursively
chown user:group f  # Change owner and group
```

### Process Cheatsheet

```bash
ps aux              # All processes
ps aux | grep app   # Find specific process
top / htop          # Real-time monitor
kill PID            # Graceful stop (SIGTERM)
kill -9 PID         # Force stop (SIGKILL)
pkill name          # Kill by name
pgrep name          # Find PID by name
```

### Background Jobs Cheatsheet

```bash
cmd &               # Run in background
jobs                # List background jobs
fg %1               # Bring job 1 foreground
Ctrl+Z              # Suspend foreground job
bg %1               # Resume in background
nohup cmd &         # Persist after logout
```

### Signal Cheatsheet

```bash
kill -15 PID    # SIGTERM — graceful
kill -9  PID    # SIGKILL — force
kill -1  PID    # SIGHUP  — reload config
kill -2  PID    # SIGINT  — like Ctrl+C
kill -19 PID    # SIGSTOP — pause
kill -18 PID    # SIGCONT — resume
```

---

**Previous:** [01 — Filesystem & Navigation](01-filesystem-and-navigation.md)
**Next:** [03 — Text Processing & Networking](03-text-processing-and-networking.md)
