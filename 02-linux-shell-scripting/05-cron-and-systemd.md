# 05 — Cron & systemd

> "Automation without scheduling is only half done. cron and systemd are how Linux turns 'run this once' into 'run this forever, reliably'."

---

## Table of Contents

1. [Cron Overview](#1-cron-overview)
2. [Crontab Syntax](#2-crontab-syntax)
3. [Managing Crontabs](#3-managing-crontabs)
4. [System-Level Cron](#4-system-level-cron)
5. [Cron Best Practices](#5-cron-best-practices)
6. [systemd Overview](#6-systemd-overview)
7. [systemctl — Managing Services](#7-systemctl--managing-services)
8. [Writing systemd Service Units](#8-writing-systemd-service-units)
9. [systemd Timers — Modern Cron](#9-systemd-timers--modern-cron)
10. [journald — Centralized Logging](#10-journald--centralized-logging)
11. [journalctl — Reading Logs](#11-journalctl--reading-logs)
12. [Real-World Patterns](#12-real-world-patterns)
13. [Summary & Cheatsheet](#13-summary--cheatsheet)

---

## 1. Cron Overview

`cron` is the classic Unix job scheduler daemon. It reads job definitions (crontabs) and executes them at specified times.

### How cron works

```
┌─────────────────────────────────────────────┐
│                cron daemon                   │
│  Reads crontab files every minute           │
│  For each job: if time matches → run it     │
│                                             │
│  Sources:                                   │
│  ├── /var/spool/cron/crontabs/  (users)    │
│  ├── /etc/crontab               (system)   │
│  └── /etc/cron.d/               (packages) │
│                                             │
│  Also executes scripts in:                  │
│  ├── /etc/cron.hourly/                      │
│  ├── /etc/cron.daily/                       │
│  ├── /etc/cron.weekly/                      │
│  └── /etc/cron.monthly/                     │
└─────────────────────────────────────────────┘
```

---

## 2. Crontab Syntax

### Field Layout

```
┌───────── Minute        (0-59)
│ ┌─────── Hour          (0-23)
│ │ ┌───── Day of month  (1-31)
│ │ │ ┌─── Month         (1-12 or Jan-Dec)
│ │ │ │ ┌─ Day of week   (0-7, 0 and 7 = Sunday, or Mon-Sun)
│ │ │ │ │
* * * * *   command to execute
```

### Special Characters

| Character | Meaning | Example |
|-----------|---------|---------|
| `*` | Every value | `* * * * *` = every minute |
| `,` | List of values | `0,30 * * * *` = at :00 and :30 |
| `-` | Range | `9-17 * * * *` = every minute 9am–5pm |
| `/` | Step/interval | `*/5 * * * *` = every 5 minutes |

### Common Cron Expressions

```bash
# Every minute
* * * * *

# Every 5 minutes
*/5 * * * *

# Every hour on the hour
0 * * * *

# Every day at 2:30 AM
30 2 * * *

# Every day at midnight
0 0 * * *

# Every Monday at 8 AM
0 8 * * 1

# Every weekday (Mon-Fri) at 9 AM
0 9 * * 1-5

# 1st of every month at midnight
0 0 1 * *

# Every 15 minutes during business hours (9am-5pm, weekdays)
*/15 9-17 * * 1-5

# Every 6 hours
0 */6 * * *

# Twice daily — 6am and 6pm
0 6,18 * * *

# January 1st at midnight
0 0 1 1 *
```

### Special Shortcuts (non-POSIX, supported by most modern cron)

```bash
@reboot     # Run once at startup
@hourly     # 0 * * * *
@daily      # 0 0 * * *
@midnight   # 0 0 * * *
@weekly     # 0 0 * * 0
@monthly    # 0 0 1 * *
@yearly     # 0 0 1 1 *
@annually   # 0 0 1 1 *
```

---

## 3. Managing Crontabs

### User Crontabs

```bash
crontab -e          # Edit your crontab (opens in $EDITOR)
crontab -l          # List your crontab
crontab -r          # REMOVE your entire crontab (careful!)
crontab -l > backup.txt   # Backup before editing

# Edit another user's crontab (root only)
crontab -u ajaz -e
crontab -u ajaz -l
```

### Crontab File Format (user crontabs)

```bash
# This is a comment
# Syntax: minute hour dom month dow command

# Set PATH for cron jobs (cron has minimal environment)
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

# Daily backup at 2 AM
0 2 * * * /usr/local/bin/backup.sh >> /var/log/backup.log 2>&1

# Health check every 5 minutes
*/5 * * * * /usr/local/bin/health-check.sh >> /var/log/health.log 2>&1

# Cleanup temp files at 3 AM every Sunday
0 3 * * 0 find /tmp -type f -mtime +7 -delete

# Rotate logs daily at midnight
@daily /usr/sbin/logrotate /etc/logrotate.conf

# Run at system startup
@reboot /opt/myapp/bin/start.sh
```

---

## 4. System-Level Cron

### `/etc/crontab` — System Crontab

The system crontab adds a `USER` field before the command:

```bash
cat /etc/crontab
# SHELL=/bin/sh
# PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
#
# m h dom mon dow user  command
17  *  * * *   root    cd / && run-parts --report /etc/cron.hourly
25  6  * * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47  6  * * 7   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52  6  1 * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
```

### `/etc/cron.d/` — Application Cron Jobs

Packages and applications drop cron jobs here. Format is same as `/etc/crontab` (with USER field):

```bash
# /etc/cron.d/myapp
SHELL=/bin/bash
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# Daily cleanup for myapp
0 1 * * * myappuser /opt/myapp/bin/cleanup.sh >> /var/log/myapp/cleanup.log 2>&1
```

### Drop-in Directories

```bash
# Place executable scripts here — cron.daily, cron.weekly, etc.
ls /etc/cron.daily/
ls /etc/cron.hourly/

# Requirements for scripts in these dirs:
# - Must be executable (chmod +x)
# - No dot in filename (no .sh extension — it breaks run-parts)
# - Should be idempotent (safe to run multiple times)
```

---

## 5. Cron Best Practices

### Environment Limitations

Cron has a minimal environment:

```bash
# Problem: cron won't find your command
0 * * * * docker ps   # Fails if docker isn't in cron's PATH

# Solution 1: Full path
0 * * * * /usr/bin/docker ps

# Solution 2: Set PATH in crontab
PATH=/usr/local/sbin:/usr/local/bin:/usr/bin:/sbin:/bin
0 * * * * docker ps

# Solution 3: Source profile in script
0 * * * * bash -l -c '/usr/local/bin/myscript.sh'
# -l = login shell, loads /etc/profile and ~/.bash_profile
```

### Logging Cron Output

```bash
# Discard output (silent — only works if you don't need to debug)
*/5 * * * * /opt/check.sh > /dev/null 2>&1

# Log to a file (append)
*/5 * * * * /opt/check.sh >> /var/log/myapp/check.log 2>&1

# Log with timestamp
*/5 * * * * echo "$(date): Starting check" >> /var/log/check.log && /opt/check.sh >> /var/log/check.log 2>&1

# Send stdout to mail (default cron behavior without redirect)
# Cron emails output to the user unless redirected
MAILTO=""              # Suppress cron email
MAILTO="admin@example.com"  # Send to email
```

### Avoiding Cron Race Conditions

```bash
# Problem: job takes longer than its interval, two instances overlap
# Solution: Use a lock file

#!/usr/bin/env bash
LOCKFILE="/var/lock/myjob.lock"
exec 200>"$LOCKFILE"

# Try to get exclusive lock (non-blocking)
flock -n 200 || {
  echo "Another instance is running — exiting"
  exit 1
}

# Your job runs here with the lock held
echo "Running job at $(date)"

# Lock is automatically released when script exits
```

---

## 6. systemd Overview

systemd is the modern init system and service manager. It replaced SysV init and has become the standard on almost all major Linux distributions.

### systemd Architecture

```
┌──────────────────────────────────────────────────────┐
│                    systemd (PID 1)                    │
│                                                       │
│  Manages:                                             │
│  ├── Services (.service units)                       │
│  ├── Timers (.timer units) ← cron replacement        │
│  ├── Mount points (.mount units)                     │
│  ├── Sockets (.socket units)                         │
│  ├── Targets (.target units) ← runlevels             │
│  └── ...and more                                     │
│                                                       │
│  Features:                                            │
│  ├── Parallel startup (faster boot)                  │
│  ├── Dependency management (start A before B)        │
│  ├── Automatic restart on failure                    │
│  ├── Resource limits (cgroups)                       │
│  └── Centralized logging (journald)                  │
└──────────────────────────────────────────────────────┘
```

### Unit File Locations

| Location | Purpose |
|----------|---------|
| `/lib/systemd/system/` | Units installed by packages (do not edit) |
| `/etc/systemd/system/` | Your custom units or overrides (edit here) |
| `/run/systemd/system/` | Runtime units (temporary) |
| `~/.config/systemd/user/` | User-level units (no root) |

**Priority:** `/etc/` overrides `/lib/`

---

## 7. systemctl — Managing Services

### Essential systemctl Commands

```bash
# Service state control
systemctl start nginx           # Start service
systemctl stop nginx            # Stop service
systemctl restart nginx         # Stop + Start
systemctl reload nginx          # Reload config without restart (if supported)
systemctl reload-or-restart nginx  # Reload if supported, else restart

# Enable/disable at boot
systemctl enable nginx          # Start automatically at boot
systemctl disable nginx         # Don't start at boot
systemctl enable --now nginx    # Enable AND start immediately
systemctl disable --now nginx   # Disable AND stop immediately

# Status and info
systemctl status nginx          # Detailed status (state, logs, PID)
systemctl is-active nginx       # Returns 0 if active, non-zero if not
systemctl is-enabled nginx      # Returns 0 if enabled for boot
systemctl is-failed nginx       # Returns 0 if in failed state
systemctl show nginx            # All unit properties (detailed)

# List units
systemctl list-units            # All active units
systemctl list-units --type=service         # Only services
systemctl list-units --state=failed         # Failed units
systemctl list-units --state=running        # Running services

# After editing unit files
systemctl daemon-reload         # Reload systemd — MUST run after editing .service files
```

### Runlevels (Targets in systemd)

```bash
# systemd targets replace SysV runlevels
# Runlevel 0 = poweroff.target
# Runlevel 1 = rescue.target
# Runlevel 3 = multi-user.target
# Runlevel 5 = graphical.target
# Runlevel 6 = reboot.target

systemctl get-default           # Show current default target
systemctl set-default multi-user.target   # Server — no GUI
systemctl isolate rescue.target  # Switch to rescue mode (immediate)

# System power
systemctl poweroff
systemctl reboot
systemctl suspend
```

---

## 8. Writing systemd Service Units

### Basic Service Unit Structure

```ini
[Unit]
Description=Short description shown in systemctl status
Documentation=https://docs.example.com
After=network.target postgresql.service    # Start AFTER these
Requires=postgresql.service               # MUST have these (hard dependency)
Wants=redis.service                       # Nice to have (soft dependency)

[Service]
Type=simple
User=myappuser
Group=myappuser
WorkingDirectory=/opt/myapp
ExecStart=/opt/myapp/bin/myapp server
ExecStop=/opt/myapp/bin/myapp stop
ExecReload=/bin/kill -HUP $MAINPID

Restart=on-failure
RestartSec=5s
StartLimitIntervalSec=60s
StartLimitBurst=3

# Environment
Environment=APP_ENV=production
Environment=PORT=8080
EnvironmentFile=/etc/myapp/config       # Load vars from file

# Logging
StandardOutput=journal
StandardError=journal

# Security Hardening
NoNewPrivileges=yes
ProtectSystem=full
ProtectHome=yes

[Install]
WantedBy=multi-user.target   # Enable for normal multi-user boot
```

### Service Types

| Type | Behavior | Use When |
|------|----------|---------|
| `simple` | Process in ExecStart IS the service | Default — most apps |
| `exec` | Like simple but blocks until process ready | systemd 250+ |
| `forking` | Process forks to background | Traditional daemons (mysql, apache) |
| `oneshot` | Runs once, then exits | Scripts, setup tasks |
| `notify` | Process sends sd_notify() when ready | Apps that explicitly support it (postgres, nginx) |
| `idle` | Delayed until other jobs finish | Background cleanup tasks |

### Restart Policies

```ini
Restart=no              # Never restart (default)
Restart=on-success      # Restart only if exits 0  
Restart=on-failure      # Restart on non-zero exit, signal, timeout (MOST USEFUL)
Restart=on-abnormal     # Restart on signal, watchdog timeout
Restart=always          # Always restart regardless of exit code
```

### Real Example 1: Node.js Application

```ini
# /etc/systemd/system/myapp.service
[Unit]
Description=MyApp Node.js Application
After=network.target
StartLimitIntervalSec=0

[Service]
Type=simple
User=deploy
WorkingDirectory=/opt/myapp
ExecStart=/usr/bin/node /opt/myapp/server.js
Restart=on-failure
RestartSec=10
Environment=NODE_ENV=production
Environment=PORT=3000
EnvironmentFile=-/etc/myapp/config    # - prefix: don't fail if file missing
StandardOutput=journal
StandardError=journal
SyslogIdentifier=myapp

[Install]
WantedBy=multi-user.target
```

```bash
# Install and start
sudo cp myapp.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable --now myapp
sudo systemctl status myapp
```

### Real Example 2: Python Worker

```ini
# /etc/systemd/system/myapp-worker@.service
# The @ makes it a template — instantiate with: systemctl start myapp-worker@1
[Unit]
Description=MyApp Worker Instance %i
After=network.target redis.service
Wants=redis.service

[Service]
Type=simple
User=appuser
WorkingDirectory=/opt/myapp
ExecStart=/opt/myapp/venv/bin/python -m myapp.worker --instance %i
Restart=on-failure
RestartSec=5
KillSignal=SIGTERM
TimeoutStopSec=30

[Install]
WantedBy=multi-user.target
```

```bash
# Start 3 worker instances
systemctl enable --now myapp-worker@1
systemctl enable --now myapp-worker@2
systemctl enable --now myapp-worker@3
```

### Overriding Package Unit Files (Drop-in Overrides)

```bash
# NEVER edit /lib/systemd/system/ — it gets overwritten on package update
# Instead, use override files:

systemctl edit nginx          # Creates /etc/systemd/system/nginx.service.d/override.conf

# Or manually:
mkdir -p /etc/systemd/system/nginx.service.d/
cat > /etc/systemd/system/nginx.service.d/limits.conf << 'EOF'
[Service]
LimitNOFILE=65536
LimitNPROC=65536
EOF

systemctl daemon-reload
systemctl restart nginx
```

---

## 9. systemd Timers — Modern Cron

systemd timers are the modern replacement for cron. Each timer activates a corresponding `.service` unit.

### Advantages Over Cron

- Logs are captured by journald (searchable, structured)
- Can run based on elapsed time, not just clock
- Dependencies — run only when network is available
- Missed jobs are handled (randomized delays prevent thundering herd)
- Easier debugging with `systemctl status`

### Timer Unit File

```ini
# /etc/systemd/system/daily-backup.timer
[Unit]
Description=Daily Backup Timer

[Timer]
OnCalendar=*-*-* 02:00:00       # Every day at 2 AM
Persistent=true                  # Run missed jobs after downtime
RandomizedDelaySec=300           # Spread by up to 5 minutes (prevents load spikes)
Unit=daily-backup.service        # Which service to activate (optional if names match)

[Install]
WantedBy=timers.target
```

```ini
# /etc/systemd/system/daily-backup.service
[Unit]
Description=Daily Backup Service
Wants=network-online.target
After=network-online.target

[Service]
Type=oneshot
User=backup
ExecStart=/usr/local/bin/backup.sh
StandardOutput=journal
StandardError=journal
```

```bash
# Install and enable
systemctl daemon-reload
systemctl enable --now daily-backup.timer

# Check timer status
systemctl status daily-backup.timer
systemctl list-timers                    # All active timers
systemctl list-timers --all             # Including inactive

# Manually trigger the service (test it)
systemctl start daily-backup.service
```

### Timer Expressions

```ini
# OnCalendar syntax: DOW YYYY-MM-DD HH:MM:SS

OnCalendar=*-*-* *:*:00         # Every minute (= cron: * * * * *)
OnCalendar=*-*-* *:00:00        # Every hour   (= cron: 0 * * * *)
OnCalendar=*-*-* 00:00:00       # Every day at midnight
OnCalendar=Mon *-*-* 00:00:00   # Every Monday midnight
OnCalendar=Mon..Fri *-*-* 09:00:00  # Weekdays at 9 AM
OnCalendar=*-*-01 00:00:00      # 1st of each month

# Shortcuts
OnCalendar=hourly
OnCalendar=daily
OnCalendar=weekly
OnCalendar=monthly

# Elapsed time timers
OnBootSec=15min          # 15 minutes after boot
OnUnitActiveSec=1h       # 1 hour after the unit was last activated

# Verify the expression
systemd-analyze calendar "Mon..Fri *-*-* 09:00:00"
```

---

## 10. journald — Centralized Logging

journald is the logging daemon that collects logs from all systemd services, the kernel, and other sources. It stores them in a structured binary format.

### How journald Works

```
┌──────────────────────────────────────────────────────┐
│                                                       │
│  Log Sources:                                         │
│  ├── systemd services (stdout/stderr → journal)      │
│  ├── kernel (dmesg → journal)                        │
│  ├── Syslog API (legacy apps)                        │
│  └── native journal API (sd_journal_send)             │
│               ↓                                       │
│          journald                                     │
│               ↓                                       │
│  Storage: /var/log/journal/ (persistent)             │
│        or /run/log/journal/ (volatile, clears reboot)│
│               ↓                                       │
│          journalctl                                   │
│  (query tool — filter by unit, time, priority, etc.) │
└──────────────────────────────────────────────────────┘
```

### Making Logs Persistent

By default, logs may only survive until reboot on some distros:

```bash
# Check current storage
journalctl --disk-usage

# Make persistent (survives reboot)
mkdir -p /var/log/journal
systemd-tmpfiles --create --prefix /var/log/journal
systemctl restart systemd-journald

# Configure in /etc/systemd/journald.conf:
# Storage=persistent   # Keep logs between reboots
# SystemMaxUse=500M    # Max disk space for journal
# MaxRetentionSec=1month  # Keep logs for 1 month
```

---

## 11. journalctl — Reading Logs

```bash
# View all logs
journalctl                              # All logs (oldest first)
journalctl -r                           # Reverse — newest first
journalctl -n 50                        # Last 50 lines
journalctl -f                           # Follow — stream new entries (like tail -f)

# Filter by service/unit
journalctl -u nginx                     # All nginx logs
journalctl -u nginx -f                  # Follow nginx logs
journalctl -u nginx -n 100             # Last 100 nginx log lines
journalctl -u nginx -u postgresql       # Multiple units

# Filter by time
journalctl --since "2026-03-28"
journalctl --since "2026-03-28 10:00:00"
journalctl --until "2026-03-28 12:00:00"
journalctl --since "2026-03-28 10:00" --until "2026-03-28 11:00"
journalctl --since "1 hour ago"
journalctl --since "yesterday"
journalctl --since today

# Filter by priority (severity)
journalctl -p err                       # Error and above
journalctl -p warning                   # Warning and above
journalctl -p debug                     # Everything (very verbose)
# Priorities: emerg, alert, crit, err, warning, notice, info, debug

# Filter by PID or user
journalctl _PID=1234
journalctl _UID=1000                    # Logs from user with UID 1000

# Kernel messages (dmesg)
journalctl -k                           # Kernel messages
journalctl -k --since "1 hour ago"

# Boot logs
journalctl -b                           # Current boot
journalctl -b -1                        # Previous boot
journalctl --list-boots                 # List all boots

# Output formats
journalctl -u nginx -o json            # JSON output (for log shipping)
journalctl -u nginx -o json-pretty     # Pretty JSON
journalctl -u nginx -o short-precise   # Microsecond timestamps
journalctl -u nginx --no-pager         # Don't use pager (pipe-friendly)

# Search (grep equivalent)
journalctl -u myapp | grep "error"
journalctl -u myapp -g "connection refused"   # Built-in grep (-g)

# Disk usage
journalctl --disk-usage                 # How much space journals use
journalctl --vacuum-size=500M           # Shrink to 500 MB
journalctl --vacuum-time=30days         # Delete logs older than 30 days
```

---

## 12. Real-World Patterns

### Pattern 1: Web App with systemd

Complete setup for deploying a web application:

```bash
# Create system user for the app
useradd -r -s /sbin/nologin -c "MyApp Service" myapp

# Create directories
mkdir -p /opt/myapp/{bin,conf,logs}
chown -R myapp:myapp /opt/myapp

# Install your application binary
cp myapp-binary /opt/myapp/bin/
chmod 755 /opt/myapp/bin/myapp-binary

# Create environment file (keep secrets out of unit file)
cat > /etc/myapp/environment << 'EOF'
APP_ENV=production
DB_URL=postgresql://myapp:secret@localhost/myappdb
SECRET_KEY=your-secret-key-here
PORT=8080
EOF
chmod 600 /etc/myapp/environment
chown myapp:myapp /etc/myapp/environment

# Create the service unit
cat > /etc/systemd/system/myapp.service << 'EOF'
[Unit]
Description=My Web Application
After=network.target postgresql.service
Wants=postgresql.service
StartLimitIntervalSec=0

[Service]
Type=simple
User=myapp
Group=myapp
WorkingDirectory=/opt/myapp
ExecStart=/opt/myapp/bin/myapp-binary
EnvironmentFile=/etc/myapp/environment
Restart=on-failure
RestartSec=5
StandardOutput=journal
StandardError=journal
SyslogIdentifier=myapp
NoNewPrivileges=yes

[Install]
WantedBy=multi-user.target
EOF

# Enable and start
systemctl daemon-reload
systemctl enable --now myapp
systemctl status myapp
journalctl -u myapp -f
```

### Pattern 2: Scheduled Data Pipeline

```bash
# Service that processes data
cat > /etc/systemd/system/data-pipeline.service << 'EOF'
[Unit]
Description=Data Processing Pipeline

[Service]
Type=oneshot
User=datauser
ExecStart=/opt/pipeline/run.sh
StandardOutput=journal
StandardError=journal
SyslogIdentifier=data-pipeline
EOF

# Timer that runs it every 15 minutes
cat > /etc/systemd/system/data-pipeline.timer << 'EOF'
[Unit]
Description=Run data pipeline every 15 minutes

[Timer]
OnCalendar=*:0/15
Persistent=true
RandomizedDelaySec=60

[Install]
WantedBy=timers.target
EOF

systemctl daemon-reload
systemctl enable --now data-pipeline.timer
systemctl list-timers data-pipeline.timer
```

### Pattern 3: Convert a Cron Job to a systemd Timer

```bash
# OLD cron job:
# 0 3 * * * /usr/local/bin/backup.sh >> /var/log/backup.log 2>&1

# NEW systemd service + timer:

cat > /etc/systemd/system/nightly-backup.service << 'EOF'
[Unit]
Description=Nightly Backup

[Service]
Type=oneshot
ExecStart=/usr/local/bin/backup.sh
StandardOutput=journal
StandardError=journal
EOF

cat > /etc/systemd/system/nightly-backup.timer << 'EOF'
[Unit]
Description=Nightly Backup at 3 AM

[Timer]
OnCalendar=*-*-* 03:00:00
Persistent=true

[Install]
WantedBy=timers.target
EOF

systemctl daemon-reload
systemctl enable --now nightly-backup.timer

# Verify
systemctl list-timers nightly-backup.timer

# View logs of the last run
journalctl -u nightly-backup.service -n 50
```

---

## 13. Summary & Cheatsheet

### Cron Cheatsheet

```bash
crontab -e          # Edit user crontab
crontab -l          # List user crontab
crontab -r          # Remove user crontab (careful!)

# Syntax: minutes hour dom month dow command
*/5 * * * * /path/to/script.sh   # Every 5 minutes
0 2 * * * /path/to/script.sh     # Daily at 2 AM
0 9 * * 1-5 cmd                  # Weekdays at 9 AM
@reboot /path/to/script.sh       # On boot
@daily cmd >> /var/log/cmd.log 2>&1  # With logging
```

### systemctl Cheatsheet

```bash
systemctl start/stop/restart/reload nginx
systemctl enable/disable nginx
systemctl enable --now nginx        # Enable + start
systemctl status nginx
systemctl is-active nginx
systemctl list-units --type=service
systemctl list-units --state=failed
systemctl daemon-reload              # After editing unit files
```

### journalctl Cheatsheet

```bash
journalctl -u nginx              # Service logs
journalctl -u nginx -f           # Follow live
journalctl -u nginx -n 50        # Last 50 lines
journalctl --since "1 hour ago"  # Recent logs
journalctl -p err                # Errors only
journalctl -b                    # This boot
journalctl --disk-usage          # Space used
```

### systemd Unit File Skeleton

```ini
[Unit]
Description=My Service
After=network.target

[Service]
Type=simple
User=myuser
ExecStart=/path/to/binary
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

### systemd Timer Skeleton

```ini
[Unit]
Description=My Timer

[Timer]
OnCalendar=*-*-* 02:00:00
Persistent=true

[Install]
WantedBy=timers.target
```

---

**Previous:** [04 — Bash Scripting](04-bash-scripting.md)
**Next:** [Interview Questions](interview-questions.md)
