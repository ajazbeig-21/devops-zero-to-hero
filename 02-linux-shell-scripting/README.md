# Module 02 — Linux & Shell Scripting

> **YouTube Playlist:** DevOps Zero to Hero — Module 2
> **Goal:** Master Linux fundamentals and Bash scripting to the level required for day-to-day DevOps work — SSH into servers, read logs, manage processes, automate tasks, and write production-grade scripts.

---

## Why Linux for DevOps?

**90%+ of production servers run Linux.** Every tool in the DevOps ecosystem — Docker, Kubernetes, Terraform, Ansible, Prometheus — was built on and for Linux. CI/CD runners are Linux containers. Cloud VMs default to Linux. If you're not comfortable on a Linux command line, you cannot function as a DevOps engineer.

This module makes you dangerous at the terminal.

---

## What You Will Learn

By the end of this module you will be able to:

- Navigate the Linux filesystem with confidence
- Manage files, directories, users, groups, and permissions
- Understand and manage Linux processes
- Write Bash scripts that automate real DevOps tasks
- Schedule tasks with cron and manage services with systemd
- Analyze logs effectively
- Use text processing tools (grep, awk, sed, cut, sort) like a pro
- Troubleshoot common Linux problems
- Answer every Linux interview question thrown at you

---

## Module Files

| File | Topic | Approx. Read Time |
|------|--------|------------------|
| [01-filesystem-and-navigation.md](01-filesystem-and-navigation.md) | Linux filesystem hierarchy, navigation, file operations | 25 min |
| [02-users-permissions-processes.md](02-users-permissions-processes.md) | Users, groups, permissions, processes, signals | 30 min |
| [03-text-processing-and-networking.md](03-text-processing-and-networking.md) | grep, awk, sed, pipes, redirects, networking commands | 30 min |
| [04-bash-scripting.md](04-bash-scripting.md) | Variables, conditionals, loops, functions, real scripts | 40 min |
| [05-cron-and-systemd.md](05-cron-and-systemd.md) | Task scheduling, service management, journald | 25 min |
| [interview-questions.md](interview-questions.md) | 50+ Linux & Bash interview Q&A | 40 min |

---

## Prerequisites

- Module 01 (DevOps Fundamentals) completed
- A terminal to practice on:
  - **macOS:** Terminal.app or iTerm2 (uses zsh but bash-compatible)
  - **Windows:** WSL2 (Windows Subsystem for Linux) with Ubuntu
  - **Linux:** You're already there
  - **Browser-based:** [killercoda.com](https://killercoda.com) — free Linux playground, no install needed

---

## Practice Lab Setup

The fastest way to get a Linux environment for practice:

```bash
# Option 1 — Docker (works on any OS)
docker run -it --rm ubuntu:22.04 bash

# Option 2 — Multipass (lightweight Ubuntu VM)
brew install multipass          # macOS
multipass launch --name devlab
multipass shell devlab

# Option 3 — WSL2 (Windows)
wsl --install -d Ubuntu-22.04
```

---

## Key Terms Cheatsheet

| Term | Definition |
|------|-----------|
| Shell | The program that interprets your commands (bash, zsh, sh) |
| Terminal | The application that runs the shell |
| Kernel | The core of Linux OS — manages hardware |
| POSIX | Standard that defines Unix-compatible behavior |
| Shebang (`#!/bin/bash`) | First line of a script — tells the OS which interpreter to use |
| stdin / stdout / stderr | Standard input (0), output (1), and error (2) streams |
| Pipe (`\|`) | Connect stdout of one command to stdin of the next |
| Redirect (`>`, `>>`) | Send stdout to a file (overwrite or append) |
| Environment variable | Named value available to the shell and its child processes |
| Daemon | A background service process (usually ends in `d`: sshd, nginx, systemd) |
| PID | Process ID — unique number assigned to every running process |
| inode | Filesystem data structure storing file metadata (not the filename) |
| Symbolic link | A pointer file that references another file (like a shortcut) |

---

## Quick Reference Card

```bash
# Navigation
pwd          # Where am I?
ls -la       # List all files with details
cd /etc      # Change to /etc
cd -         # Go back to previous directory
cd ~         # Go to home directory

# Files
cp src dst   # Copy
mv src dst   # Move/rename
rm -rf dir   # Delete recursively (careful!)
mkdir -p a/b/c  # Create nested directories
touch file   # Create empty file / update timestamp
ln -s target link  # Create symlink

# Permissions
chmod 755 file    # rwxr-xr-x
chown user:group file
chmod +x script.sh  # Make executable

# Processes
ps aux           # All processes
top / htop       # Live process monitor
kill -9 PID      # Force kill
pkill nginx      # Kill by name

# Text
grep -r "error" /var/log   # Recursive search
tail -f /var/log/syslog    # Follow log file
cat file | wc -l           # Count lines
awk '{print $1}' file      # Print first column
sed 's/old/new/g' file     # Replace text
```

---

## Next Module

After completing this module, proceed to **[Module 03 — Networking Fundamentals](../03-networking/README.md)**.
