# Linux & Shell Scripting — Interview Questions

> These questions cover everything from junior to senior level. Master these and you will handle any Linux interview with confidence.

---

## Table of Contents

1. [Filesystem & Navigation](#1-filesystem--navigation)
2. [Users & Permissions](#2-users--permissions)
3. [Process Management](#3-process-management)
4. [Text Processing & Networking](#4-text-processing--networking)
5. [Bash Scripting](#5-bash-scripting)
6. [cron & systemd](#6-cron--systemd)
7. [Scenario-Based Questions](#7-scenario-based-questions)
8. [Rapid Fire Round — 25 Quick Q&A](#8-rapid-fire-round--25-quick-qa)
9. [Common Interview Mistakes](#9-common-interview-mistakes)

---

## 1. Filesystem & Navigation

**Q1: What is the Linux Filesystem Hierarchy Standard (FHS)? What lives in `/etc`, `/var`, and `/tmp`?**

> **A:** FHS defines a standard directory structure for Unix-like systems. Everything is mounted under `/` (root).
> - `/etc/` — System-wide configuration files (nginx.conf, sshd_config, crontab). Never executable programs.
> - `/var/` — Variable data that changes at runtime: logs (`/var/log/`), databases, caches, mail spools.
> - `/tmp/` — Temporary files cleared on reboot. Any user can write here; only owner can delete their files (sticky bit).

---

**Q2: What is the difference between a hard link and a symbolic link?**

> **A:**
> | | Hard Link | Symbolic Link |
> |---|---|---|
> | Inode | Same as original | Different inode, stores path |
> | Cross filesystems | No | Yes |
> | Target deleted | Data survives | Link breaks |
> | Works on dirs | No (usually) | Yes |
> | `ls -l` indicator | Shows as regular file | Shows `->` target |
>
> ```bash
> ln original.txt hardlink      # Hard link
> ln -s original.txt symlink    # Symbolic link
> ```

---

**Q3: How do you find all files modified in the last 24 hours under `/var/log/`?**

> **A:**
> ```bash
> find /var/log -type f -mtime -1
> # -mtime -1 means modified less than 1 day ago
> ```
> To find files modified in the last 60 minutes: `find /var/log -mtime -0 -newer /var/log`  
> Or more precisely: `find /var/log -type f -mmin -60`

---

**Q4: What is the `/proc` filesystem? How would you use it to troubleshoot a process?**

> **A:** `/proc` is a virtual filesystem that exposes kernel and process information in real-time. Nothing is stored on disk.
>
> ```bash
> cat /proc/cpuinfo         # CPU details
> cat /proc/meminfo         # Memory details
> cat /proc/loadavg         # Load average
> ls /proc/$(pgrep nginx)/  # Everything about the nginx process
> cat /proc/$(pgrep nginx)/cmdline  # Full command line
> cat /proc/$(pgrep nginx)/environ  # Environment variables
> cat /proc/$(pgrep nginx)/fd/ | ls -la  # Open file descriptors
> ```

---

**Q5: Explain the difference between `>` and `>>` and `2>&1`.**

> **A:**
> - `>` — Redirect stdout to a file, **overwriting** its contents
> - `>>` — Redirect stdout to a file, **appending** to it
> - `2>&1` — Redirect stderr (file descriptor 2) to wherever stdout (fd 1) currently points
>
> ```bash
> cmd > output.txt        # stdout to file (overwrite)
> cmd >> output.txt       # stdout to file (append)
> cmd 2>&1               # stderr goes to terminal with stdout
> cmd > output.txt 2>&1  # BOTH stdout and stderr to file
> cmd > /dev/null 2>&1   # Discard all output
> ```

---

## 2. Users & Permissions

**Q6: A file shows `-rw-r--r--`. Explain what this means.**

> **A:** This is decomposed as:
> - `-` — Regular file (not a directory or symlink)
> - `rw-` — Owner can read and write, not execute (6)
> - `r--` — Group members can read only (4)
> - `r--` — Others (everyone else) can read only (4)
>
> Octal notation: `644`. This is the standard permission for a config or data file.

---

**Q7: What is the difference between `chmod 755 file` and `chmod +x file`?**

> **A:**
> - `chmod 755 file` — **Sets** exact permissions: owner `rwx` (7), group `r-x` (5), others `r-x` (5). It replaces whatever was there before.
> - `chmod +x file` — **Adds** execute bit for all three (owner, group, others) without changing other bits.
>
> So if file was `rw-r--r--` (644):
> - `chmod +x` → `rwxr-xr-x` (755)
> - `chmod 755` → `rwxr-xr-x` (755) — same result in this case, but different for other starting permissions

---

**Q8: What is the sticky bit and where is it used?**

> **A:** The sticky bit on a directory means users can only delete files they own, even if they have write permission on the directory.
>
> `/tmp` is the classic example: `drwxrwxrwt` — anyone can create files, but you can't delete another user's files.
>
> ```bash
> chmod +t /shared/uploads    # Set sticky bit
> ls -ld /tmp
> # drwxrwxrwt 12 root root 4096 /tmp
> ```

---

**Q9: What is SUID? Why is `/usr/bin/passwd` SUID?**

> **A:** SUID (Set User ID) causes the binary to run as its **owner** rather than the user executing it. Indicated by `s` in the owner execute position.
>
> `/usr/bin/passwd` is owned by root and has SUID set. When you run it as a regular user, it executes with root privileges so it can write to `/etc/shadow`. Without SUID, regular users couldn't change their own passwords.
>
> Security note: Extra SUID files are a common attack vector. Audit them regularly:
> ```bash
> find / -perm -4000 -type f 2>/dev/null
> ```

---

**Q10: How do you give a user sudo access to restart only nginx, without a password?**

> **A:**
> ```bash
> sudo visudo -f /etc/sudoers.d/deploy
> # Add this line:
> deploy ALL=(root) NOPASSWD: /bin/systemctl restart nginx, /bin/systemctl reload nginx
> ```
> Always use `visudo` (never `vi /etc/sudoers` directly) to validate syntax before saving. Use absolute paths for commands.

---

**Q11: How do you check which groups a user belongs to? How do you add them to a group?**

> **A:**
> ```bash
> id username              # Show UID, GID, and all group memberships
> groups username          # List groups
>
> usermod -aG docker username   # Add to group (-a = append, don't replace)
> # User must log out and back in for group changes to take effect
> # Or: newgrp docker   (apply in current session)
> ```

---

## 3. Process Management

**Q12: What is the difference between `kill -9` and `kill -15`?**

> **A:**
> - `kill -15` (SIGTERM) — The **polite** way to stop a process. Asks the process to shut down gracefully. The process can catch this signal, save state, close connections, and exit cleanly. This is the default when you run `kill PID`.
> - `kill -9` (SIGKILL) — The **forceful** way. The kernel terminates the process immediately — it cannot be caught, blocked, or ignored. Use only when SIGTERM doesn't work. Risk: open files may be left in inconsistent state.
>
> Best practice: Always try SIGTERM first, wait a few seconds, then SIGKILL only if necessary.

---

**Q13: What is a zombie process? How do you deal with it?**

> **A:** A zombie process has finished executing but its entry still exists in the process table because its parent hasn't collected its exit status via `wait()`. It appears as `Z` in `ps aux`.
>
> Zombies use almost no resources (just a PID), but if there are thousands it can exhaust PIDs.
>
> ```bash
> ps aux | grep 'Z'          # Find zombies
> # You cannot kill a zombie with kill -9 — it's already dead
> # Fix: send SIGCHLD to the parent (may trigger it to reap the child)
> kill -CHLD <PPID>
> # Or: restart/kill the parent process
> # Or: if PID 1, the kernel automatically reaps orphaned processes
> ```

---

**Q14: How do you monitor system load? What does a load average of 4.0 mean on a 2-core system?**

> **A:**
> ```bash
> uptime                    # Shows 1, 5, 15-minute load averages
> top                       # Real-time process + load info
> cat /proc/loadavg         # Raw numbers
> ```
>
> Load average represents the average number of processes waiting to run or currently running. On a 2-core system:
> - `1.0` = 50% utilized — comfortable
> - `2.0` = 100% utilized — exactly at capacity
> - `4.0` = 200% utilized — overloaded, processes waiting
>
> Rule of thumb: For `N` cores, load average of `N` = fully loaded.

---

**Q15: What is the difference between VSZ and RSS in `ps aux` output?**

> **A:**
> - **VSZ** (Virtual Set Size) — Total virtual memory a process could use, including memory not yet actually used (includes memory-mapped files, shared libs, etc.)
> - **RSS** (Resident Set Size) — Actual physical RAM currently in use by the process. This is the more meaningful metric for memory consumption.
>
> VSZ >> RSS is normal. If RSS is growing continuously over time, it may indicate a memory leak.

---

## 4. Text Processing & Networking

**Q16: How would you find the top 10 IP addresses making requests in an nginx access log?**

> **A:**
> ```bash
> awk '{print $1}' /var/log/nginx/access.log \
>   | sort \
>   | uniq -c \
>   | sort -rn \
>   | head -10
> ```
> Steps:
> 1. `awk '{print $1}'` — Extract first field (IP address)
> 2. `sort` — Sort IPs (required for uniq to work correctly)
> 3. `uniq -c` — Count occurrences of each IP
> 4. `sort -rn` — Sort numerically in reverse (highest count first)
> 5. `head -10` — Take the top 10

---

**Q17: What is the difference between `grep` and `awk`?**

> **A:**
> - **grep** — Searches for pattern matches. Returns matching lines. Excellent for filtering.
> - **awk** — Full text-processing language. Can split by field, perform calculations, conditional logic, and produce formatted output.
>
> ```bash
> # grep: Is there an error?
> grep "ERROR" app.log
>
> # awk: How many errors per hour?
> awk '/ERROR/ {hour=substr($2,1,2); count[hour]++} END {for(h in count) print h, count[h]}' app.log
> ```
>
> Think of grep as a filter, awk as a reporter.

---

**Q18: Explain `sed -i 's/old/new/g' file`. What does the `g` flag do?**

> **A:** This performs in-place substitution in a file:
> - `s/old/new/` — Substitutes "old" with "new"
> - `g` — Global flag — replaces ALL occurrences on each line (without `g`, only the first occurrence per line is replaced)
> - `-i` — In-place edit (modifies file directly)
>
> ```bash
> # Replace first occurrence per line:
> sed 's/foo/bar/' file.txt
>
> # Replace ALL occurrences:
> sed 's/foo/bar/g' file.txt
>
> # In-place with backup (safest):
> sed -i.bak 's/foo/bar/g' file.txt
> ```

---

**Q19: How do you check what process is listening on port 80?**

> **A:**
> ```bash
> ss -tlnp | grep ':80'
> # or
> lsof -i :80
> # or
> netstat -tlnp | grep ':80'   # (older)
>
> # Example output:
> # LISTEN 0  128  0.0.0.0:80  0.0.0.0:*  users:(("nginx",pid=1234,fd=6))
> ```

---

**Q20: How do you test if a remote port is open without Telnet?**

> **A:**
> ```bash
> nc -zv hostname 5432          # netcat — verbose port test
> nc -z -w3 hostname 5432       # timeout 3 seconds
>
> # Using bash built-in (no extra tools needed):
> timeout 3 bash -c 'cat < /dev/null > /dev/tcp/hostname/5432' \
>   && echo "Open" || echo "Closed"
>
> # Using curl:
> curl -v telnet://hostname:5432 2>&1 | head -5
> ```

---

**Q21: Explain the purpose of `ssh-keygen`, `ssh-copy-id`, and `authorized_keys`.**

> **A:**
> - `ssh-keygen` — Generates a public/private key pair. Private key stays on your machine (`~/.ssh/id_ed25519`). Public key is shared.
> - `ssh-copy-id user@server` — Copies your public key to `~/.ssh/authorized_keys` on the remote server. Appends it safely.
> - `authorized_keys` — A file on the server listing all public keys allowed to authenticate. When you connect, the server checks if your private key matches any entry.
>
> After setup, you authenticate with your private key — no password needed.

---

## 5. Bash Scripting

**Q22: What does `set -euo pipefail` do? Why is it important?**

> **A:**
> - `set -e` — Exit immediately if any command returns a non-zero exit code (fail fast)
> - `set -u` — Treat unset variable references as errors (`$UNDEFINED` would cause an exit instead of silently expanding to empty)
> - `set -o pipefail` — Pipeline fails if ANY command in the pipe fails (without this, `grep error log | mail -s alert ops@` would "succeed" even if grep found nothing)
>
> Together, they make scripts fail fast and predictably — critical in CI/CD and automation where silent failures cause real damage.

---

**Q23: What is the difference between `$@` and `$*`?**

> **A:** Both hold all script/function arguments, but they differ when quoted:
> - `"$@"` — Each argument as a **separate** word. Preserves argument boundaries. Always use this.
> - `"$*"` — All arguments as a **single** word joined by the first character of IFS (usually space).
>
> ```bash
> my_func "hello world" "foo"
> # "$@" → "hello world" "foo"  (2 arguments — correct)
> # "$*" → "hello world foo"    (1 argument — wrong when args have spaces)
> ```

---

**Q24: How do you safely handle errors in a bash script?**

> **A:**
> ```bash
> #!/usr/bin/env bash
> set -euo pipefail
>
> # Trap for cleanup
> cleanup() {
>   echo "Cleaning up..."
>   rm -f "$LOCKFILE"
> }
> trap cleanup EXIT INT TERM
>
> # Safe command execution with error message
> cp source.txt dest.txt || { echo "Copy failed"; exit 1; }
>
> # Validate required variables
> : "${CONFIG_FILE:?CONFIG_FILE must be set}"
>
> # Check file exists before use
> [[ -f "$CONFIG_FILE" ]] || { echo "Config not found: $CONFIG_FILE"; exit 1; }
> ```

---

**Q25: Write a bash one-liner to check if a service is healthy and restart it if not.**

> **A:**
> ```bash
> systemctl is-active --quiet nginx || systemctl restart nginx
>
> # Or with logging:
> systemctl is-active --quiet nginx \
>   || { echo "$(date): nginx down, restarting"; systemctl restart nginx; }
>
> # Using curl for HTTP-level check:
> curl -sf http://localhost/health > /dev/null 2>&1 \
>   || systemctl restart myapp
> ```

---

**Q26: What is a heredoc? Give a DevOps use case.**

> **A:** A heredoc lets you pass multi-line input to a command inline:
>
> ```bash
> cat << EOF > /etc/nginx/conf.d/myapp.conf
> server {
>     listen 80;
>     server_name myapp.example.com;
>     location / {
>         proxy_pass http://localhost:3000;
>     }
> }
> EOF
> ```
> DevOps use cases: writing config files, embedding Kubernetes manifests in scripts, passing SQL to databases, creating cloud-init scripts.

---

## 6. cron & systemd

**Q27: Explain the crontab field order. How would you schedule a job every 15 minutes between 9 AM and 5 PM on weekdays?**

> **A:** The field order is: `minute hour day-of-month month day-of-week`
>
> ```bash
> # Every 15 minutes, 9 AM to 5 PM, Mon–Fri
> */15 9-17 * * 1-5 /usr/local/bin/my-script.sh
> ```

---

**Q28: What is the difference between cron and a systemd timer? When would you use each?**

> **A:**
> | Feature | cron | systemd timer |
> |---------|------|--------------|
> | Logging | Email or file manually | Automatic via journald |
> | Missed job recovery | No (job missed if system down) | Yes (Persistent=true) |
> | Dependencies | No | Yes (network, services) |
> | Easy syntax | Yes (5-field) | Slightly verbose |
> | On single system | Best | Best |
>
> **Use cron when:** Simple, widely portable, quick one-liners. Works everywhere.  
> **Use systemd timer when:** Need structured logging, missed-job recovery, dependencies on other services, or you're already on a systemd-based system.

---

**Q29: A cron job is not running. How do you debug it?**

> **A:**
> 1. **Check cron daemon is running:** `systemctl status cron` (or `crond`)
> 2. **Check cron logs:** `grep cron /var/log/syslog` or `journalctl -u cron`
> 3. **Test the command manually:** Run the exact command as the cron user
> 4. **Check PATH:** Cron has minimal PATH — use full paths in crontab (`/usr/bin/docker`, not `docker`)
> 5. **Check environment:** Cron doesn't load `~/.bashrc` — add `SHELL=/bin/bash` and `PATH=...` to crontab
> 6. **Check redirect:** Always redirect output: add `>> /var/log/job.log 2>&1` to see what's happening
> 7. **Check permissions:** Is the script executable? `chmod +x script.sh`

---

**Q30: What does `Type=oneshot` mean in a systemd unit file? When would you use it?**

> **A:** `Type=oneshot` means the service process runs, finishes, and exits — systemd considers the service "active" only while it's running, then transitions to "inactive" (not failed). Used for tasks that are meant to complete:
>
> - Backup scripts
> - Database migrations
> - Cleanup jobs
> - Any script that runs and exits
>
> It pairs well with systemd timers (the timer activates the service, which runs once, then stops — and the timer triggers it again at the next scheduled time).

---

## 7. Scenario-Based Questions

**Q31: A production server shows 100% disk usage and services are failing. Walk me through your troubleshooting steps.**

> **A:**
> ```bash
> # 1. Confirm the problem
> df -h                          # See which filesystem is full
>
> # 2. Find what's taking space
> du -sh /var/* 2>/dev/null | sort -rh | head -10
> du -sh /home/* 2>/dev/null | sort -rh | head -5
>
> # 3. Common culprits:
> du -sh /var/log/               # Logs
> docker system df               # Docker
> du -sh /tmp/                   # Temp files
>
> # 4. Quick wins:
> journalctl --vacuum-size=200M  # Shrink system logs
> docker system prune -f         # Clean Docker
> find /var/log -name "*.gz" -mtime +30 -delete  # Old rotated logs
> find /tmp -mtime +7 -delete    # Old temp files
>
> # 5. Force log rotation:
> logrotate -f /etc/logrotate.conf
>
> # 6. Check for big files:
> find / -type f -size +500M 2>/dev/null
>
> # 7. After freeing space, verify:
> df -h
> systemctl restart affected-service
> ```

---

**Q32: You deployed a new version and the service is returning 500 errors. How do you diagnose?**

> **A:**
> ```bash
> # 1. Check service health
> systemctl status myapp
>
> # 2. Check application logs (most important)
> journalctl -u myapp -n 100 --no-pager | grep -i "error\|exception\|fatal"
> tail -100 /var/log/myapp/app.log | grep -i error
>
> # 3. Check nginx/load balancer logs for upstream errors
> tail -100 /var/log/nginx/error.log
>
> # 4. Test the endpoint directly
> curl -v http://localhost:8080/health
> curl -v http://localhost:8080/api/v1/test
>
> # 5. Check resource constraints
> free -h          # Memory full?
> df -h            # Disk full?
> ps aux | grep myapp  # Is it running?
>
> # 6. Check config differences
> git diff HEAD~1 -- config/
>
> # 7. Roll back if needed
> git checkout HEAD~1
> ./deploy.sh production
> ```

---

**Q33: You need to run a script on 50 servers. How do you do it?**

> **A:** Multiple approaches by preference:
>
> **1. Ansible (production standard):**
> ```bash
> ansible all -m script -a "/path/to/script.sh" -i inventory.ini
> ```
>
> **2. Parallel SSH (pdsh or parallel-ssh):**
> ```bash
> pdsh -w server[1-50] 'sudo systemctl restart myapp'
> ```
>
> **3. xargs with ssh:**
> ```bash
> cat servers.txt | xargs -P10 -I{} ssh {} 'sudo /opt/script.sh'
> # -P10 = 10 parallel connections
> ```
>
> **4. for loop (simple but sequential):**
> ```bash
> while IFS= read -r server; do
>   ssh "$server" 'sudo /opt/script.sh'
> done < servers.txt
> ```

---

**Q34: How would you monitor a log file in real time and send an alert if a specific error appears?**

> **A:**
> ```bash
> #!/usr/bin/env bash
> # alert-on-error.sh
>
> LOGFILE="/var/log/myapp/app.log"
> PATTERN="CRITICAL|OutOfMemory|FATAL"
> ALERT_SCRIPT="/usr/local/bin/send-alert.sh"
>
> tail -F "$LOGFILE" | while IFS= read -r line; do
>   if echo "$line" | grep -qE "$PATTERN"; then
>     echo "$(date): Detected critical error: $line" | \
>       "$ALERT_SCRIPT" "Critical Error in myapp"
>   fi
> done
>
> # Or simpler with grep --line-buffered:
> tail -F /var/log/myapp/app.log | \
>   grep --line-buffered -E "CRITICAL|OutOfMemory" | \
>   while read -r line; do
>     curl -X POST https://hooks.slack.com/... \
>       -d "{\"text\": \"Alert: $line\"}"
>   done
> ```

---

## 8. Rapid Fire Round — 25 Quick Q&A

| # | Question | Answer |
|---|----------|--------|
| 1 | What does `chmod 777` do? | Gives everyone read, write, execute — usually a security mistake |
| 2 | How do you check your current directory? | `pwd` |
| 3 | What is `/dev/null`? | A discard sink — anything written disappears |
| 4 | How do you find files larger than 1 GB? | `find / -type f -size +1G` |
| 5 | What command shows the last 100 lines of a file? | `tail -n 100 file` |
| 6 | What does `!!` do in bash? | Repeat the last command |
| 7 | How do you suppress all output from a command? | `command > /dev/null 2>&1` |
| 8 | What is `nohup` for? | Run a command that survives after you log out |
| 9 | What does `df -h` show? | Disk free space per filesystem, human-readable |
| 10 | What does `du -sh *` show? | Disk usage of each item in current directory |
| 11 | How do you make a script executable? | `chmod +x script.sh` |
| 12 | What is a shebang? | First line `#!/usr/bin/env bash` specifying the interpreter |
| 13 | What does `$?` mean? | Exit code of the last command (0 = success) |
| 14 | How do you count lines in a file? | `wc -l file.txt` |
| 15 | What is the difference between `su` and `sudo`? | `su` switches user identity; `sudo` runs one command with elevated privileges |
| 16 | What signal does Ctrl+C send? | SIGINT (signal 2) |
| 17 | What signal does Ctrl+Z send? | SIGTSTP — suspends the process |
| 18 | How do you view all running services? | `systemctl list-units --type=service --state=running` |
| 19 | What is `journalctl -f` equivalent to? | `tail -f` but for the system journal |
| 20 | How do you reload nginx config without downtime? | `systemctl reload nginx` or `nginx -s reload` |
| 21 | What does `set -e` do in bash? | Exit immediately on any command that returns non-zero |
| 22 | How do you check open ports? | `ss -tlnp` |
| 23 | What is a symlink? | A file pointing to another file path (`ln -s target link`) |
| 24 | What does `uniq -c` do? | Counts consecutive duplicate lines |
| 25 | How do you run the previous command as root? | `sudo !!` |

---

## 9. Common Interview Mistakes

**Mistake 1: Saying `kill -9` is the normal way to stop a process**
> Always prefer `kill -15` (SIGTERM) first. Save `kill -9` for processes that won't terminate gracefully.

**Mistake 2: Confusing `chmod +x` with `chmod 755`**
> `+x` adds execute permission. `chmod 755` sets exact permissions. They both result in execute permission but `chmod 755` also sets the non-execute bits.

**Mistake 3: Editing `/lib/systemd/system/*.service` files directly**
> Package updates overwrite those files. Always create `/etc/systemd/system/` overrides via `systemctl edit servicename`.

**Mistake 4: Writing `rm -rf $DIRECTORY` without quoting**
> If `$DIRECTORY` is unset or empty, this becomes `rm -rf /` — catastrophic. Always quote: `rm -rf "$DIRECTORY"` and use `set -u` to error on unset variables.

**Mistake 5: Forgetting `2>&1` in crontab**
> Without it, stderr is emailed to the cron user (and often silently discarded). Always: `cmd >> /var/log/job.log 2>&1`

**Mistake 6: Using `>` when you mean `>>`**
> `>` overwrites the file every time the script runs. In crontabs and monitoring scripts, you almost always want `>>` (append).

**Mistake 7: Forgetting `daemon-reload` after editing unit files**
> After editing any `.service` file, you must run `systemctl daemon-reload` before the changes take effect. This is one of the most common Linux interview gotcha questions.

**Mistake 8: Using relative paths in cron jobs**
> Cron's working directory is not predictable. Always use absolute paths: `/usr/bin/docker`, `/opt/myapp/script.sh`.

---

**Back to:** [README](README.md) | [Module 01 — DevOps Fundamentals](../01-devops-fundamentals/README.md)
