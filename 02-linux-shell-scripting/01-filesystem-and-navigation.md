# 01 — Linux Filesystem & Navigation

> "The filesystem is the map of Linux. You cannot work effectively until you know where everything lives and why."

---

## Table of Contents

1. [Linux Filesystem Hierarchy Standard (FHS)](#1-linux-filesystem-hierarchy-standard-fhs)
2. [Navigating the Filesystem](#2-navigating-the-filesystem)
3. [Working with Files and Directories](#3-working-with-files-and-directories)
4. [File Types in Linux](#4-file-types-in-linux)
5. [Viewing File Contents](#5-viewing-file-contents)
6. [Finding Files](#6-finding-files)
7. [Links — Hard and Symbolic](#7-links--hard-and-symbolic)
8. [Disk Usage and Filesystem Info](#8-disk-usage-and-filesystem-info)
9. [Archives and Compression](#9-archives-and-compression)
10. [Standard Streams and Redirection](#10-standard-streams-and-redirection)
11. [Pipes](#11-pipes)
12. [Summary & Cheatsheet](#12-summary--cheatsheet)

---

## 1. Linux Filesystem Hierarchy Standard (FHS)

Linux organizes everything under a single root `/`. Every file, device, and process lives somewhere under this tree. This is fundamentally different from Windows (C:\, D:\).

```
/                       ← Root — the top of everything
├── bin/                ← Essential user binaries (ls, cp, mv, grep)
├── sbin/               ← Essential system binaries (only root needs: fdisk, iptables)
├── usr/                ← User programs and data
│   ├── bin/            ← Most user commands (git, docker, python3)
│   ├── sbin/           ← Non-essential system binaries
│   ├── lib/            ← Libraries for /usr/bin and /usr/sbin
│   └── local/          ← Locally compiled software (not from package manager)
├── etc/                ← System-wide configuration files (nginx.conf, sshd_config)
├── var/                ← Variable data (logs, databases, mail, caches)
│   ├── log/            ← Log files (/var/log/syslog, /var/log/nginx/)
│   ├── lib/            ← Application state data
│   └── www/            ← Web server root (on some distros)
├── home/               ← User home directories (/home/ajaz, /home/ubuntu)
├── root/               ← Home directory for the root user
├── tmp/                ← Temporary files (cleared on reboot)
├── dev/                ← Device files (disks, terminals, /dev/null, /dev/zero)
├── proc/               ← Virtual filesystem — kernel and process info
│   ├── cpuinfo         ← CPU information
│   ├── meminfo         ← Memory information
│   └── [PID]/         ← Directory per running process
├── sys/                ← Virtual filesystem — hardware and kernel info
├── mnt/                ← Temporary mount points
├── media/              ← Removable media mount points (USB, CD)
├── opt/                ← Optional/third-party software (some apps install here)
├── lib/                ← Essential shared libraries (like .dll on Windows)
├── lib64/              ← 64-bit libraries
└── boot/               ← Boot files (kernel image, GRUB)
```

### Critical Directories for DevOps

| Directory | Why you'll be here |
|-----------|-------------------|
| `/etc/` | Edit config files for nginx, sshd, cron, systemd units |
| `/var/log/` | Read application and system logs |
| `/home/` or `~` | Your files, SSH keys, bash config |
| `/tmp/` | Download and extract things temporarily |
| `/usr/local/bin/` | Install custom scripts/tools for all users |
| `/proc/` | Debug running processes, check system resources |
| `/dev/null` | Discard output (`command > /dev/null 2>&1`) |
| `/etc/systemd/system/` | Your custom service unit files |
| `/etc/cron.d/` | Application cron jobs |

### Special Files Under `/dev/`

| File | Description |
|------|-------------|
| `/dev/null` | Discard sink — anything written here disappears |
| `/dev/zero` | Source of infinite zero bytes (useful for creating blank files) |
| `/dev/random` | Source of random bytes (used in cryptography) |
| `/dev/sda`, `/dev/sdb` | Physical disk devices |
| `/dev/sda1` | First partition of first disk |
| `/dev/tty` | Current terminal |

---

## 2. Navigating the Filesystem

### Essential Navigation Commands

```bash
pwd                    # Print working directory — "where am I?"
# Output: /home/ajaz

ls                     # List files in current directory
ls -l                  # Long format (permissions, owner, size, date)
ls -a                  # Include hidden files (starting with .)
ls -la                 # Long format + hidden files
ls -lh                 # Long format with human-readable sizes (KB, MB)
ls -lt                 # Sort by modification time (newest first)
ls -R                  # Recursive — list all subdirectories
ls /etc                # List a specific directory

cd /var/log            # Change to absolute path
cd ..                  # Go up one level
cd ../..               # Go up two levels
cd ~                   # Go to your home directory (same as cd $HOME)
cd -                   # Go to previous directory (toggle between two dirs)
cd /                   # Go to root

# Useful shortcuts
!!                     # Repeat last command
!$                     # Last argument of previous command
Alt + .                # Insert last argument of previous command
Ctrl + L               # Clear screen (same as clear)
Ctrl + R               # Reverse search command history
History                # Show command history
```

### Understanding Absolute vs Relative Paths

```bash
# Absolute path — always starts with /
cd /home/ajaz/projects/myapp

# Relative path — relative to current directory
# Assuming you're in /home/ajaz
cd projects/myapp         # same as above
cd ./projects/myapp       # ./ explicitly means "current directory"

# Special path symbols
.                         # Current directory
..                        # Parent directory
~                         # Home directory (/home/ajaz or /root)
-                         # Previous directory (cd -)
```

### Tab Completion — Use It Always

Pressing `Tab` auto-completes file paths and commands. This:
- Saves typing time
- Prevents typos in paths
- Shows you what options are available (press Tab twice if nothing completes)

```bash
cd /var/lo<TAB>         # Completes to /var/log/
cat /etc/ngi<TAB>       # Completes to /etc/nginx/
```

---

## 3. Working with Files and Directories

### Creating Files and Directories

```bash
touch filename.txt           # Create empty file (or update timestamp if exists)
touch file1 file2 file3      # Create multiple files at once

mkdir mydir                  # Create directory
mkdir -p parent/child/grand  # Create nested directories (-p = parents)
mkdir -m 755 mydir           # Create with specific permissions

# Create a file with content immediately
echo "Hello World" > hello.txt
cat > config.txt << EOF
server_name=myapp
port=8080
debug=false
EOF
```

### Copying Files and Directories

```bash
cp source.txt dest.txt          # Copy file
cp -r sourcedir/ destdir/       # Copy directory recursively
cp -p file backup               # Preserve timestamps and permissions
cp -u source dest               # Copy only if source is newer
cp *.log /tmp/                  # Copy all .log files to /tmp/
cp -v file dest                 # Verbose — show what's being copied

# Practical DevOps example — backup config before editing
cp /etc/nginx/nginx.conf /etc/nginx/nginx.conf.bak
```

### Moving and Renaming

```bash
mv oldname.txt newname.txt      # Rename a file
mv file.txt /tmp/               # Move to different directory
mv *.log /var/log/archive/      # Move all .log files
mv -v file dest                 # Verbose
mv -i file dest                 # Interactive — ask before overwrite
mv -n file dest                 # No overwrite — skip if dest exists
```

### Deleting Files and Directories

```bash
rm file.txt                     # Delete a file
rm -f file.txt                  # Force delete (no error if not found)
rm -r directory/                # Delete directory and all contents
rm -rf directory/               # Force recursive delete (NO CONFIRMATION)
rm *.tmp                        # Delete all .tmp files

# DANGER ZONE — these are irreversible:
rm -rf /                        # DELETE EVERYTHING (never run this)
rm -rf ./                       # Delete everything in current directory
rm -rf *                        # Delete everything in current directory

# Safer alternative — move to trash first (on desktop Linux)
# On servers: rm is permanent, no trash

# Safe practice — always double-check before rm -rf
ls -la thing-to-delete/        # Verify what you're about to delete
rm -rf thing-to-delete/        # Then delete
```

### Practical DevOps File Operations

```bash
# Deploy a config file
cp /tmp/new-nginx.conf /etc/nginx/nginx.conf
nginx -t && systemctl reload nginx  # Test and reload only if valid

# Clean up old log files
find /var/log -name "*.log" -mtime +30 -delete  # Delete logs older than 30 days

# Archive and compress
tar -czf backup-$(date +%Y%m%d).tar.gz /etc/nginx/  # Backup with date in filename

# Create directory structure for a new project
mkdir -p /opt/myapp/{bin,conf,logs,data}
ls /opt/myapp/
# bin  conf  data  logs
```

---

## 4. File Types in Linux

In Linux, **everything is a file** — including devices, sockets, and directories.

The first character of `ls -l` output tells you the file type:

```bash
ls -la /dev/ | head -20
# drwxr-xr-x  → d = directory
# -rw-r--r--  → - = regular file
# lrwxrwxrwx  → l = symbolic link
# brw-rw----  → b = block device (disk)
# crw-rw-rw-  → c = character device (terminal)
# prw-r--r--  → p = named pipe (FIFO)
# srwxrwxrwx  → s = socket
```

| Symbol | Type | Example |
|--------|------|---------|
| `-` | Regular file | `/etc/nginx/nginx.conf` |
| `d` | Directory | `/var/log/` |
| `l` | Symbolic link | `/usr/bin/python → python3` |
| `b` | Block device | `/dev/sda` (disk) |
| `c` | Character device | `/dev/tty` (terminal) |
| `p` | Named pipe | Inter-process communication |
| `s` | Socket | `/var/run/docker.sock` |

### The Docker Socket — Critical DevOps Knowledge

```bash
ls -la /var/run/docker.sock
# srw-rw---- 1 root docker 0 Mar 28 10:00 /var/run/docker.sock

# This is a Unix socket file
# Docker CLI communicates with the Docker daemon through this socket
# If your user is not in the 'docker' group, you need sudo for docker commands
sudo usermod -aG docker $USER  # Add yourself to docker group
```

---

## 5. Viewing File Contents

### Basic Viewing

```bash
cat file.txt              # Print entire file to terminal
cat -n file.txt           # Print with line numbers
cat file1 file2           # Concatenate and print multiple files

tac file.txt              # Print in reverse order (line by line)

head file.txt             # First 10 lines
head -n 20 file.txt       # First 20 lines
head -c 100 file.txt      # First 100 bytes

tail file.txt             # Last 10 lines
tail -n 50 file.txt       # Last 50 lines
tail -f /var/log/syslog   # FOLLOW — stream new lines as they're written
tail -F /var/log/nginx/access.log  # Follow, retry if file rotates
```

### `tail -f` — Your Most Used DevOps Command

```bash
# Watch nginx access log in real time
tail -f /var/log/nginx/access.log

# Watch multiple log files at once
tail -f /var/log/nginx/access.log /var/log/nginx/error.log

# Watch logs AND filter for errors only
tail -f /var/log/app.log | grep --line-buffered "ERROR"

# Watch k8s pod logs (equivalent in Kubernetes world)
kubectl logs -f deployment/myapp --tail=100
```

### Pagers — For Large Files

```bash
less file.txt             # Page through file (preferred over more)
# Inside less:
#   Space / Page Down  → Next page
#   b / Page Up        → Previous page
#   /searchterm        → Search forward
#   ?searchterm        → Search backward
#   n                  → Next match
#   N                  → Previous match
#   g                  → Go to beginning
#   G                  → Go to end
#   q                  → Quit

more file.txt             # Older pager (less is better)
```

### Checking File Type and Info

```bash
file /bin/bash            # Determine file type
# /bin/bash: ELF 64-bit LSB pie executable, x86-64

file /etc/hosts
# /etc/hosts: ASCII text

stat file.txt             # Detailed file metadata
# File: file.txt
# Size: 1234      Blocks: 8          IO Block: 4096  regular file
# Device: fd00h   Inode: 123456      Links: 1
# Access: (0644/-rw-r--r--)  Uid: (1000/ajaz)   Gid: (1000/ajaz)
# Access: 2026-03-28 10:00:00
# Modify: 2026-03-27 15:30:00
# Change: 2026-03-27 15:30:00

wc file.txt               # Word count — lines, words, bytes
wc -l file.txt            # Just line count
wc -l *.log               # Count lines in all log files
```

---

## 6. Finding Files

### `find` — The Most Powerful File Search

```bash
# Basic syntax: find [path] [criteria] [action]

find / -name "nginx.conf"           # Find by name (case-sensitive)
find / -iname "nginx.conf"          # Case-insensitive
find /etc -name "*.conf"            # Find all .conf files under /etc
find /var/log -name "*.log" -type f # Only regular files
find /home -type d                  # Only directories

# Find by size
find /var/log -size +100M           # Files > 100 MB
find /tmp -size -1k                 # Files < 1 KB

# Find by time
find /var/log -mtime -7             # Modified in last 7 days
find /tmp -mtime +30                # Not modified for 30+ days
find /etc -newer /etc/nginx/nginx.conf  # Newer than a reference file

# Find by permissions
find / -perm 777                    # World-writable files (security risk)
find / -perm -4000                  # SUID files (security audit)

# Find by owner
find /home -user ajaz               # Files owned by ajaz
find / -nouser                      # Files with no owner (orphaned)

# Execute a command on each found file
find /tmp -name "*.tmp" -delete                    # Delete all .tmp
find /var/log -name "*.log" -exec wc -l {} \;      # Count lines in each log
find /etc -name "*.conf" -exec grep -l "port" {} \; # Find configs containing "port"

# The most useful find commands in DevOps:
find / -name "docker-compose.yml" 2>/dev/null     # Find all compose files
find /home -name ".ssh" -type d                    # Find all SSH dirs
find /var/log -name "*.log" -mtime +7 -delete     # Clean old logs
```

### `locate` — Fast File Search (Uses Database)

```bash
updatedb                    # Update the locate database (run as root)
locate nginx.conf           # Very fast — searches a database, not disk
locate -i nginx.conf        # Case-insensitive
```

**`find` vs `locate`:** `find` searches in real-time (always current, slower). `locate` uses a pre-built database (faster but may be outdated).

### `which` and `whereis` — Finding Commands

```bash
which python3               # Full path of command in $PATH
# /usr/bin/python3

which docker git terraform  # Find multiple commands

whereis bash                # Find binary, source, and man page
# bash: /usr/bin/bash /usr/share/man/man1/bash.1.gz

type ls                     # Is it a command, alias, or shell builtin?
# ls is aliased to 'ls --color=auto'

command -v git              # POSIX-portable version of which
```

---

## 7. Links — Hard and Symbolic

### Symbolic Links (Soft Links)

A symbolic link is a pointer file that contains the path to another file. Like a Windows shortcut.

```bash
ln -s /usr/local/bin/python3.11 /usr/local/bin/python  # Create symlink

ls -la /usr/local/bin/python
# lrwxrwxrwx 1 root root 7 Mar 28 python -> python3.11

# Broken symlink (target doesn't exist)
ln -s /nonexistent/path mylink
ls -la mylink
# lrwxrwxrwx ... mylink -> /nonexistent/path (shown in red — broken)

# Common DevOps use cases:
ln -s /opt/myapp-1.2.3 /opt/myapp   # Version-independent path
# Update app: just change the symlink to new version
# ln -sf /opt/myapp-1.3.0 /opt/myapp   (-f: force overwrite)
```

### Hard Links

A hard link creates another name for the same inode (same file content on disk). Deleting one hard link doesn't delete the data — only when ALL hard links are removed.

```bash
ln original.txt hardlink.txt       # Create hard link (no -s flag)

ls -li original.txt hardlink.txt
# 123456 -rw-r--r-- 2 ajaz ajaz 100 Mar 28 original.txt
# 123456 -rw-r--r-- 2 ajaz ajaz 100 Mar 28 hardlink.txt
# Same inode number (123456), link count is 2
```

**Key difference:**

| Feature | Symbolic Link | Hard Link |
|---------|-------------|-----------|
| Crossing filesystems | Yes | No |
| Works for directories | Yes | No (usually) |
| If original deleted | Link breaks | Data preserved |
| Is a separate inode | Yes | No (same inode) |
| `ls -l` shows | `->` target | Normal file |

---

## 8. Disk Usage and Filesystem Info

```bash
# Disk space
df -h                           # Disk free — all mounted filesystems, human-readable
df -h /                         # Just the root filesystem
df -h /var                      # Specific directory's filesystem

# Output:
# Filesystem      Size  Used Avail Use% Mounted on
# /dev/sda1        50G   23G   25G  48% /
# tmpfs           2.0G  0     2.0G   0% /dev/shm

# Directory/file size
du -sh /var/log                 # Total size of /var/log
du -sh *                        # Size of everything in current directory
du -sh /* 2>/dev/null           # Size of all top-level directories
du -h --max-depth=1 /var        # One level deep, human-readable

# Find the biggest directories (DevOps disk troubleshooting):
du -sh /var/* | sort -rh | head -20    # Top 20 biggest in /var
du -sh /home/* | sort -rh              # Biggest home directories

# Inode usage (can fill up even if bytes are free)
df -i                           # Show inode usage
# Common cause: millions of tiny files (log dirs, package caches)

# Mount points
mount                           # Show all mounted filesystems
mount | grep sda                # Show mounts for a specific disk
lsblk                           # List block devices in tree format
lsblk -f                        # Include filesystem types and UUIDs
```

### DevOps Disk Troubleshooting Workflow

```bash
# Step 1: Check if disk is full
df -h | awk '$5 > "80%"'        # Show filesystems >80% full

# Step 2: Find what's taking space
du -sh /var/* 2>/dev/null | sort -rh | head -10

# Step 3: Usually it's logs or Docker
du -sh /var/log/                # Check logs
docker system df                # Check Docker disk usage
docker system prune -f          # Clean unused Docker resources

# Step 4: Rotating logs
logrotate -f /etc/logrotate.conf  # Force log rotation
```

---

## 9. Archives and Compression

Tar is the most important archiving tool for DevOps — used for backups, deployments, and software distribution.

### `tar` — The Tape Archiver

```bash
# Create archives
tar -czf archive.tar.gz directory/     # Create gzip-compressed archive
tar -cjf archive.tar.bz2 directory/   # Create bzip2-compressed archive
tar -cJf archive.tar.xz directory/    # Create xz-compressed (best ratio)
tar -cf archive.tar directory/         # Create uncompressed archive

# Flags:
# c = create
# z = gzip compression
# j = bzip2 compression
# J = xz compression
# f = specify filename (must come last)
# v = verbose (show files being added)

# Extract archives
tar -xzf archive.tar.gz                # Extract gzip archive here
tar -xzf archive.tar.gz -C /opt/       # Extract to specific directory
tar -xzf archive.tar.gz --strip=1     # Strip leading directory component

# Inspect without extracting
tar -tzf archive.tar.gz                # List contents of gzip archive
tar -tzf archive.tar.gz | head -20    # First 20 entries

# Practical DevOps examples:
# Backup nginx config with timestamp
tar -czf nginx-backup-$(date +%Y%m%d-%H%M%S).tar.gz /etc/nginx/

# Extract a software release
tar -xzf terraform_1.7.0_linux_amd64.zip -C /usr/local/bin/

# Quick backup of a directory
tar -czf /tmp/app-backup.tar.gz /opt/myapp/
```

### Compression Tools

```bash
# gzip — most common, good speed/ratio balance
gzip file.txt                   # Compress: creates file.txt.gz, removes original
gzip -d file.txt.gz             # Decompress (same as gunzip)
gunzip file.txt.gz              # Decompress
gzip -k file.txt                # Keep original (-k)
gzip -l file.txt.gz             # Show compression ratio

# zip/unzip — cross-platform (Windows compatible)
zip archive.zip file1 file2     # Create zip
zip -r archive.zip directory/   # Zip entire directory
unzip archive.zip               # Extract
unzip -l archive.zip            # List contents

# Quick size comparison on a log file:
ls -lh app.log                  # 200M
gzip -k app.log
ls -lh app.log.gz               # 8M (96% compression on logs)
```

---

## 10. Standard Streams and Redirection

Every process in Linux has three standard streams:

```
stdin  (0)  ← Keyboard input (or input from a file/pipe)
stdout (1)  → Normal output (terminal screen)
stderr (2)  → Error output (terminal screen, in red on some terminals)
```

### Redirection Operators

```bash
# Redirect stdout to a file
ls -la > filelist.txt           # Overwrite (create or replace)
ls -la >> filelist.txt          # Append (add to existing)

# Redirect stderr
find / -name "foo" 2> errors.txt        # Errors to file, output to terminal
find / -name "foo" 2>> errors.log       # Append errors to log

# Redirect both stdout and stderr
find / -name "foo" > output.txt 2>&1    # Both to same file
find / -name "foo" &> output.txt        # Shorthand (bash 4+)
find / -name "foo" > output.txt 2>/dev/null  # stdout to file, discard errors

# Discard output entirely
command > /dev/null 2>&1        # Silent — suppress all output
cron job script > /dev/null 2>&1  # Common in crontab

# Input redirection
mysql -u root -p database < schema.sql  # Feed SQL file to mysql
```

### Here Documents (heredoc)

Used to pass multi-line input to a command — very common in scripts and CI/CD:

```bash
cat << EOF > /etc/nginx/conf.d/myapp.conf
server {
    listen 80;
    server_name myapp.example.com;
    
    location / {
        proxy_pass http://localhost:3000;
        proxy_set_header Host \$host;
    }
}
EOF

# Used heavily in Kubernetes manifests and cloud-init scripts:
kubectl apply -f - << EOF
apiVersion: v1
kind: ConfigMap
metadata:
  name: myapp-config
data:
  APP_ENV: production
  LOG_LEVEL: info
EOF
```

---

## 11. Pipes

The pipe `|` is one of the most powerful features of Linux. It connects the stdout of one command to the stdin of the next — letting you chain simple tools to perform complex operations.

```
command1 | command2 | command3 | command4
```

### The Philosophy
> "Write programs that do one thing and do it well. Write programs to work together." — Unix Philosophy

Each command in a pipeline does one thing. Combining them creates power.

### Pipe Examples

```bash
# Count the number of running processes
ps aux | wc -l

# Find all unique IP addresses in nginx access log
cat /var/log/nginx/access.log | awk '{print $1}' | sort | uniq -c | sort -rn | head -20

# Find the 10 largest files in a directory
find /var -type f | xargs du -h | sort -rh | head -10

# See which ports are listening
ss -tlnp | grep LISTEN

# Find all failed SSH login attempts
grep "Failed password" /var/log/auth.log | awk '{print $11}' | sort | uniq -c | sort -rn

# Monitor log errors in real time
tail -f /var/log/app.log | grep --line-buffered "ERROR\|CRITICAL"

# Count HTTP status codes in nginx log
awk '{print $9}' /var/log/nginx/access.log | sort | uniq -c | sort -rn
# Output:
#  52431 200
#   1423 304
#    347 404
#     23 500

# Get just the names of docker containers that are running
docker ps | awk 'NR>1 {print $NF}'

# Find and kill a process by name
ps aux | grep nginx | grep -v grep | awk '{print $2}' | xargs kill

# Watch CPU usage of k8s nodes
kubectl top nodes | sort -k3 -rn
```

---

## 12. Summary & Cheatsheet

### Navigation Cheatsheet

```bash
pwd           # Current directory
ls -la        # List all files with details
cd /path      # Change directory
cd ..         # Go up
cd ~          # Home directory
cd -          # Previous directory
```

### File Operations Cheatsheet

```bash
touch file              # Create empty file
mkdir -p a/b/c          # Create nested directories
cp -r src dst           # Copy directory
mv old new              # Move/rename
rm -rf dir              # Delete directory (permanent!)
ln -s target link       # Create symlink
```

### Search Cheatsheet

```bash
find / -name "file"                   # Find by name
find /etc -name "*.conf" -type f      # Find files only
find /var -mtime +30 -delete         # Find and delete old files
which command                         # Where is this command?
grep -r "text" /etc/                 # Search text in files
```

### Disk Cheatsheet

```bash
df -h           # Disk free (filesystem level)
du -sh *        # Disk usage (directory level)
lsblk           # List block devices
mount           # Show mount points
```

### Archive Cheatsheet

```bash
tar -czf archive.tar.gz dir/    # Create compressed archive
tar -xzf archive.tar.gz         # Extract
tar -tzf archive.tar.gz         # List contents
gzip file                       # Compress single file
```

### Redirection Cheatsheet

```bash
cmd > file           # Redirect stdout (overwrite)
cmd >> file          # Redirect stdout (append)
cmd 2> errors        # Redirect stderr
cmd > out 2>&1       # Redirect both stdout and stderr
cmd > /dev/null 2>&1 # Discard all output
cmd1 | cmd2          # Pipe stdout of cmd1 to stdin of cmd2
```

---

**Previous:** [README](README.md)
**Next:** [02 — Users, Permissions & Processes](02-users-permissions-processes.md)
