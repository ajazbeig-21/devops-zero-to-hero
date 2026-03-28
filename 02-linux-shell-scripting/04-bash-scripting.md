# 04 — Bash Scripting

> "A bash script is just a sequence of commands you'd otherwise type by hand — made repeatable, reliable, and shareable. This is the definition of automation."

---

## Table of Contents

1. [Script Basics and Structure](#1-script-basics-and-structure)
2. [Variables and Data Types](#2-variables-and-data-types)
3. [User Input and Arguments](#3-user-input-and-arguments)
4. [Conditionals](#4-conditionals)
5. [Loops](#5-loops)
6. [Functions](#6-functions)
7. [Arrays and Associative Arrays](#7-arrays-and-associative-arrays)
8. [String Operations](#8-string-operations)
9. [Exit Codes and Error Handling](#9-exit-codes-and-error-handling)
10. [Working with Files in Scripts](#10-working-with-files-in-scripts)
11. [Real DevOps Scripts](#11-real-devops-scripts)
12. [Script Debugging and Best Practices](#12-script-debugging-and-best-practices)
13. [Summary & Cheatsheet](#13-summary--cheatsheet)

---

## 1. Script Basics and Structure

### The Shebang

The first line of every script tells the OS which interpreter to use:

```bash
#!/bin/bash          # Use bash (most common)
#!/usr/bin/env bash  # Use bash found in PATH (more portable)
#!/bin/sh            # POSIX sh (less features, more portable)
#!/usr/bin/env python3  # Not just for bash!
```

### A Minimal Working Script

```bash
#!/usr/bin/env bash
# Purpose: Example deployment check script
# Author: Ajaz
# Date: 2026-03-28

set -euo pipefail   # Safety options (explained in Section 9)

# Your script logic here
echo "Starting deployment check..."
echo "Script complete."
```

```bash
# Make it executable
chmod +x my-script.sh

# Run it
./my-script.sh            # Run from current directory
bash my-script.sh         # Run with explicit interpreter (no execute permission needed)
/path/to/my-script.sh    # Absolute path
```

### Script Execution Environment

```bash
# View environment variables available in your shell
env
printenv
printenv PATH

# Shell options
set                 # Show all shell variables and functions
set -x              # Enable debug mode (print each command before execution)
set +x              # Disable debug mode

# Check if running as root (common script requirement)
if [[ $EUID -ne 0 ]]; then
  echo "This script must run as root"
  exit 1
fi
```

---

## 2. Variables and Data Types

Bash is untyped — everything is a string by default.

### Declaring and Using Variables

```bash
# Variable declaration (no spaces around =)
NAME="ajaz"
PORT=8080
TIMEOUT=30

# Referencing variables
echo $NAME            # Simple reference
echo "${NAME}"        # Preferred — explicit, avoids ambiguity
echo "${NAME}_suffix" # Needed when followed by chars that look like var name

# Variable naming conventions
MY_VAR="value"          # UPPER_CASE for environment/global variables
local_var="value"       # lower_case for local/script variables
readonly CONFIG_FILE="/etc/myapp/config"   # Constant — can't be reassigned

# Unset a variable
unset NAME

# Check if variable is set
echo "${NAME:-default}"     # Use "default" if NAME is unset or empty
echo "${NAME:=default}"     # Set NAME to "default" if unset/empty
echo "${NAME:?error msg}"   # Exit with error if NAME is unset/empty
echo "${NAME:+override}"    # Use "override" if NAME is set (else empty)
```

### Command Substitution

```bash
# Capture command output into a variable
TODAY=$(date +%Y-%m-%d)
HOSTNAME=$(hostname)
IP=$(ip route get 1 | awk '{print $7; exit}')
FILES_COUNT=$(find /var/log -name "*.log" | wc -l)

echo "Today is $TODAY"
echo "Host: $HOSTNAME"
echo "There are $FILES_COUNT log files"

# Old syntax (avoid — nesting is harder)
TODAY=`date +%Y-%m-%d`

# Arithmetic
RESULT=$((5 + 3))
SIZE=$((1024 * 1024))
NEXT_PORT=$((PORT + 1))
echo "$((100 / 3))"         # Integer division only in bash
```

### Special Variables

```bash
$0       # Script name
$1 $2 $3 # Positional arguments (first, second, third)
$@       # All arguments as separate words (use this!)
$*       # All arguments as single string
$#       # Number of arguments
$$       # Current script's PID
$?       # Exit code of last command (0 = success)
$!       # PID of last background process
$_       # Last argument of previous command
```

### Environment Variables

```bash
# Set and export to child processes
export DB_HOST="prod-db.example.com"
export DB_PORT=5432

# Common environment variables
echo $HOME           # /home/ajaz
echo $USER           # ajaz
echo $PATH           # Search path for commands
echo $SHELL          # /bin/bash
echo $PWD            # Current directory
echo $OLDPWD         # Previous directory
echo $EDITOR         # Default text editor
echo $LANG           # System locale
```

---

## 3. User Input and Arguments

### Command-Line Arguments

```bash
#!/usr/bin/env bash
# Usage: ./deploy.sh environment version

ENVIRONMENT=$1
VERSION=$2

# Validate arguments
if [[ $# -lt 2 ]]; then
  echo "Usage: $0 <environment> <version>"
  echo "Example: $0 production 1.2.3"
  exit 1
fi

echo "Deploying version $VERSION to $ENVIRONMENT"
```

### Argument Parsing with getopts

```bash
#!/usr/bin/env bash
# Usage: ./backup.sh -d /data -r 7 -v

DAYS=30
VERBOSE=false

while getopts "d:r:v" opt; do
  case $opt in
    d) DIRECTORY=$OPTARG ;;
    r) DAYS=$OPTARG ;;
    v) VERBOSE=true ;;
    *) echo "Usage: $0 -d directory [-r days] [-v]"; exit 1 ;;
  esac
done

[[ -z "$DIRECTORY" ]] && { echo "Error: -d directory required"; exit 1; }

$VERBOSE && echo "Backing up $DIRECTORY, retaining $DAYS days"
```

### Reading User Input

```bash
# Read from terminal
read -p "Enter your name: " NAME
read -sp "Enter password: " PASSWORD    # -s = silent (no echo)
echo ""                                  # New line after silent input

# Read with timeout
read -t 10 -p "Continue? [y/N] " answer || answer="n"

# Read line from file / stdin
while IFS= read -r line; do
  echo "Processing: $line"
done < servers.txt

# Read into array
read -ra SERVERS <<< "server1 server2 server3"
```

---

## 4. Conditionals

### `if` Statements

```bash
# Basic syntax
if [[ condition ]]; then
  # commands if true
elif [[ condition2 ]]; then
  # commands if condition2 is true
else
  # commands if all false
fi
```

### Test Conditions

```bash
# String tests
[[ "$str" == "value" ]]     # Equal
[[ "$str" != "value" ]]     # Not equal
[[ "$str" == *"partial"* ]] # Contains (glob)
[[ "$str" =~ regex ]]       # Regex match
[[ -z "$str" ]]             # Is empty (zero length)
[[ -n "$str" ]]             # Is not empty (non-zero length)

# Numeric tests
[[ $n -eq 5 ]]              # Equal to 5
[[ $n -ne 5 ]]              # Not equal to 5
[[ $n -gt 5 ]]              # Greater than
[[ $n -lt 5 ]]              # Less than
[[ $n -ge 5 ]]              # Greater than or equal
[[ $n -le 5 ]]              # Less than or equal
(( n > 5 ))                 # Arithmetic test (cleaner syntax)

# File tests
[[ -f file ]]               # Is a regular file
[[ -d dir ]]                # Is a directory
[[ -L link ]]               # Is a symlink
[[ -e path ]]               # Exists (any type)
[[ -r file ]]               # Is readable
[[ -w file ]]               # Is writable
[[ -x file ]]               # Is executable
[[ -s file ]]               # Exists and is not empty
[[ file1 -nt file2 ]]       # file1 is newer than file2
[[ file1 -ot file2 ]]       # file1 is older than file2

# Logical operators
[[ cond1 && cond2 ]]        # AND
[[ cond1 || cond2 ]]        # OR
[[ ! condition ]]            # NOT
```

### Practical if Examples

```bash
#!/usr/bin/env bash

# Check if a service is running
if systemctl is-active --quiet nginx; then
  echo "nginx is running"
else
  echo "nginx is stopped — starting..."
  systemctl start nginx
fi

# Check disk space and alert
DISK_USE=$(df / | awk 'NR==2 {print $5}' | tr -d '%')
if (( DISK_USE > 90 )); then
  echo "CRITICAL: Disk usage is ${DISK_USE}%"
  # Send alert...
elif (( DISK_USE > 75 )); then
  echo "WARNING: Disk usage is ${DISK_USE}%"
fi

# Check if required tools are installed
for tool in docker kubectl terraform; do
  if ! command -v "$tool" &>/dev/null; then
    echo "Error: $tool is not installed"
    exit 1
  fi
done
```

### `case` Statements

```bash
ENVIRONMENT=$1

case $ENVIRONMENT in
  production|prod)
    REPLICAS=3
    DEBUG=false
    LOG_LEVEL="warn"
    ;;
  staging)
    REPLICAS=2
    DEBUG=false
    LOG_LEVEL="info"
    ;;
  development|dev)
    REPLICAS=1
    DEBUG=true
    LOG_LEVEL="debug"
    ;;
  *)
    echo "Unknown environment: $ENVIRONMENT"
    echo "Valid: production, staging, development"
    exit 1
    ;;
esac

echo "Deploying with $REPLICAS replicas, debug=$DEBUG, log=$LOG_LEVEL"
```

---

## 5. Loops

### `for` Loop

```bash
# Iterate over words
for server in web1 web2 web3 db1; do
  echo "Checking: $server"
  ssh "$server" "uptime"
done

# Iterate over files
for config in /etc/nginx/conf.d/*.conf; do
  echo "Testing: $config"
  nginx -t -c "$config"
done

# Iterate over a range
for i in {1..10}; do
  echo "Item $i"
done

for i in {1..100..5}; do   # 1 to 100, step 5
  echo "$i"
done

# C-style for loop
for (( i=0; i<5; i++ )); do
  echo "Count: $i"
done

# Iterate over command output
for user in $(getent passwd | awk -F: '$3>=1000 && $3<65534 {print $1}'); do
  echo "User: $user"
done

# Iterate over array
SERVERS=("web1" "web2" "web3")
for server in "${SERVERS[@]}"; do
  echo "$server"
done
```

### `while` Loop

```bash
# Basic while
COUNT=0
while (( COUNT < 10 )); do
  echo "Count: $COUNT"
  (( COUNT++ ))
done

# Read file line by line (most common use)
while IFS= read -r line; do
  echo "Processing: $line"
done < servers.txt

# Process command output line by line
docker ps --format '{{.Names}}' | while read -r container; do
  echo "Container: $container"
  docker logs "$container" --tail 10
done

# Retry loop with backoff
attempt=1
max_attempts=5
while (( attempt <= max_attempts )); do
  echo "Attempt $attempt of $max_attempts..."
  if curl -sf http://localhost:8080/health; then
    echo "Health check passed"
    break
  fi
  echo "Failed — waiting ${attempt}s before retry"
  sleep "$attempt"
  (( attempt++ ))
done

if (( attempt > max_attempts )); then
  echo "All retries exhausted — service unhealthy"
  exit 1
fi
```

### `until` Loop

```bash
# Run until condition is TRUE (opposite of while)
until curl -sf http://localhost:8080/health; do
  echo "Waiting for service to be healthy..."
  sleep 5
done
echo "Service is up!"
```

### Loop Control

```bash
for i in {1..10}; do
  [[ $i -eq 5 ]] && break      # Exit loop entirely
  [[ $i -eq 3 ]] && continue   # Skip to next iteration
  echo "$i"
done
```

---

## 6. Functions

### Defining and Calling Functions

```bash
#!/usr/bin/env bash

# Function definition (two syntaxes — use either)
function greet() {
  echo "Hello, $1!"
}

greet() {
  echo "Hello, $1!"
}

# Call the function
greet "Ajaz"          # Output: Hello, Ajaz!
greet "World"         # Output: Hello, World!
```

### Function Arguments and Return Values

```bash
#!/usr/bin/env bash

# Arguments: $1, $2, etc. (local to function)
deploy_app() {
  local app_name=$1       # local = scoped to function
  local environment=$2
  local version=${3:-latest}  # Default value if not passed

  echo "Deploying $app_name ($version) to $environment"
  # ... deployment logic
}

deploy_app "myapp" "production" "1.2.3"
deploy_app "myapp" "staging"             # version defaults to "latest"

# Return values:
# bash functions return exit codes (0-255), NOT strings
# Use echo/printf to return strings

get_ip() {
  local hostname=$1
  ip=$(dig +short "$hostname" | head -1)
  echo "$ip"              # "return" a string via echo
}

SERVER_IP=$(get_ip "myserver.example.com")
echo "Server IP: $SERVER_IP"

# Return exit codes
check_service() {
  local service=$1
  systemctl is-active --quiet "$service"
  return $?         # Return exit code from systemctl
}

if check_service nginx; then
  echo "nginx is running"
fi
```

### Functions with Error Handling

```bash
#!/usr/bin/env bash
set -euo pipefail

# Color output helpers
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m'  # No Color

log_info()    { echo -e "${GREEN}[INFO]${NC}  $*"; }
log_warn()    { echo -e "${YELLOW}[WARN]${NC}  $*"; }
log_error()   { echo -e "${RED}[ERROR]${NC} $*" >&2; }  # stderr

die() {
  log_error "$1"
  exit "${2:-1}"
}

require_command() {
  command -v "$1" &>/dev/null || die "$1 is required but not installed"
}

# Usage
require_command docker
require_command kubectl
require_command terraform
log_info "All required tools are present"
```

---

## 7. Arrays and Associative Arrays

### Indexed Arrays

```bash
# Create arrays
SERVERS=("web1" "web2" "web3")
PORTS=(8080 8081 8082)
FILES=()            # Empty array

# Add elements
SERVERS+=("web4")
SERVERS[4]="web5"   # Direct index assignment

# Access elements
echo "${SERVERS[0]}"        # First element: web1
echo "${SERVERS[-1]}"       # Last element: web5
echo "${SERVERS[@]}"        # All elements: web1 web2 web3 web4 web5
echo "${#SERVERS[@]}"       # Array length: 5
echo "${SERVERS[@]:1:3}"    # Slice: elements 1,2,3

# Iterate
for server in "${SERVERS[@]}"; do
  ping -c1 "$server" &>/dev/null && echo "$server: UP" || echo "$server: DOWN"
done

# Iterate with index
for i in "${!SERVERS[@]}"; do
  echo "[$i] ${SERVERS[$i]}"
done
```

### Associative Arrays (hash maps)

```bash
#!/usr/bin/env bash

# Declare associative array (required)
declare -A SERVICE_PORTS

# Populate
SERVICE_PORTS["nginx"]=80
SERVICE_PORTS["postgres"]=5432
SERVICE_PORTS["redis"]=6379
SERVICE_PORTS["prometheus"]=9090

# Access
echo "nginx port: ${SERVICE_PORTS[nginx]}"

# Iterate over keys
for service in "${!SERVICE_PORTS[@]}"; do
  port=${SERVICE_PORTS[$service]}
  echo "Checking $service on port $port..."
  nc -z -w3 localhost "$port" && echo "  ✓ Open" || echo "  ✗ Closed"
done

# All values
echo "${SERVICE_PORTS[@]}"

# All keys
echo "${!SERVICE_PORTS[@]}"
```

---

## 8. String Operations

```bash
STRING="Hello, World! This is Bash."

# Length
echo "${#STRING}"                    # 27

# Substring extraction
echo "${STRING:0:5}"                 # Hello  (start:length)
echo "${STRING:7}"                   # World! This is Bash. (from index 7)
echo "${STRING: -5}"                 # Bash.  (last 5 chars)

# Case conversion (bash 4+)
echo "${STRING,,}"                   # all lowercase
echo "${STRING^^}"                   # ALL UPPERCASE
echo "${STRING^}"                    # First char uppercase

# Remove prefix/suffix
FILE="backup-2026-03-28.tar.gz"
echo "${FILE#backup-}"              # Remove shortest prefix matching pattern
echo "${FILE##*.}"                  # Remove longest prefix — last extension: gz
echo "${FILE%.*}"                   # Remove shortest suffix — backup-2026-03-28.tar
echo "${FILE%%.*}"                  # Remove longest suffix: backup-2026-03-28

# Pattern substitution
echo "${FILE/tar.gz/zip}"           # Replace first match
echo "${FILE//2026/2025}"           # Replace all matches (// vs /)

# Practical: strip extension to get base name
FILENAME="script.sh"
BASE="${FILENAME%.sh}"
echo "$BASE"                        # script

# Practical: extract just the filename from a path
FULLPATH="/var/log/nginx/access.log"
echo "${FULLPATH##*/}"              # access.log    (same as basename)
echo "${FULLPATH%/*}"               # /var/log/nginx  (same as dirname)

# Check if string starts with a prefix
VERSION="v1.2.3"
if [[ $VERSION == v* ]]; then
  echo "Version has v prefix"
fi

# Trim whitespace (no built-in, but easy pattern)
STR="  hello  "
TRIMMED=$(echo "$STR" | sed 's/^[[:space:]]*//;s/[[:space:]]*$//')
```

---

## 9. Exit Codes and Error Handling

### Exit Codes

```bash
# Every command returns 0 (success) or non-zero (failure)
ls /tmp                  # Returns 0
ls /nonexistent          # Returns 2 (no such file)
echo $?                  # Print exit code of last command

# Always check $? or use set -e
cp source.txt dest.txt
if [[ $? -ne 0 ]]; then
  echo "Copy failed!"
  exit 1
fi

# Exit your script with a code
exit 0    # Success
exit 1    # General error
exit 2    # Misuse of shell builtin
exit 127  # Command not found
```

### `set` Options — Script Safety

```bash
#!/usr/bin/env bash
set -e          # Exit immediately if a command exits with non-zero status
set -u          # Treat unset variables as errors
set -o pipefail # Pipeline fails if ANY command in it fails
set -x          # Print commands as they execute (debug mode)

# In one line (recommended for all production scripts)
set -euo pipefail

# Why pipefail matters:
# Without pipefail:
grep "error" logfile | mail -s "errors" admin@example.com
# If grep finds nothing, it exits 1 — but without pipefail, the pipeline "succeeds"
# because mail exits 0

# With set -o pipefail, the pipeline correctly fails
```

### Trap — Cleanup on Exit

```bash
#!/usr/bin/env bash
set -euo pipefail

LOCKFILE="/tmp/myapp.lock"
TEMPDIR=$(mktemp -d)

# Cleanup function — runs on EXIT, INT, TERM
cleanup() {
  local exit_code=$?
  echo "Cleaning up..."
  rm -f "$LOCKFILE"
  rm -rf "$TEMPDIR"
  if [[ $exit_code -ne 0 ]]; then
    echo "Script failed with exit code $exit_code"
  fi
  exit $exit_code
}

# Register cleanup to run on these signals
trap cleanup EXIT INT TERM

# Create lock file
touch "$LOCKFILE"

# Your script logic here
echo "Working in $TEMPDIR"
# ... do stuff ... (TEMPDIR and LOCKFILE will be cleaned up even if script fails)
```

---

## 10. Working with Files in Scripts

```bash
#!/usr/bin/env bash

# Read file into variable
CONTENT=$(cat /etc/hostname)

# Check file exists before using
CONFIG_FILE="/etc/myapp/config.conf"
if [[ ! -f "$CONFIG_FILE" ]]; then
  echo "Config file not found: $CONFIG_FILE"
  exit 1
fi

# Read file line by line (correct way — handles spaces, special chars)
while IFS= read -r line; do
  echo "Line: $line"
done < "$CONFIG_FILE"

# Safely write to a file (atomic write — prevents partial writes)
TMP=$(mktemp)
cat > "$TMP" << EOF
APP_ENV=production
DB_HOST=db.example.com
DB_PORT=5432
EOF
mv "$TMP" /etc/myapp/config.conf    # Atomic rename

# Source a file (load its variables into current shell)
source /etc/myapp/config.conf      # or: . /etc/myapp/config.conf
echo "DB_HOST is: $DB_HOST"

# Parse a .env file safely
while IFS='=' read -r key value; do
  # Skip comments and empty lines
  [[ "$key" =~ ^[[:space:]]*# ]] && continue
  [[ -z "$key" ]] && continue
  export "$key"="$value"
done < .env
```

---

## 11. Real DevOps Scripts

### Script 1: Service Health Check

```bash
#!/usr/bin/env bash
# health-check.sh — Check if services are healthy, alert if not
set -euo pipefail

SERVICES=("nginx" "postgresql" "redis")
ALERT_EMAIL="ops@example.com"
FAILED=()

check_service() {
  local service=$1
  if systemctl is-active --quiet "$service"; then
    echo "[OK]   $service is running"
  else
    echo "[FAIL] $service is NOT running"
    FAILED+=("$service")
    systemctl status "$service" --no-pager -l 2>&1 | tail -20
  fi
}

echo "=== Service Health Check — $(date) ==="

for svc in "${SERVICES[@]}"; do
  check_service "$svc"
done

if (( ${#FAILED[@]} > 0 )); then
  echo ""
  echo "FAILED SERVICES: ${FAILED[*]}"
  # Uncomment to send email:
  # echo "Services down: ${FAILED[*]}" | mail -s "Health Check Alert" "$ALERT_EMAIL"
  exit 1
else
  echo ""
  echo "All services healthy."
fi
```

### Script 2: Deployment Script

```bash
#!/usr/bin/env bash
# deploy.sh — Deploy application version to an environment
set -euo pipefail

# ---- Configuration ----
APP_NAME="myapp"
DEPLOY_USER="deploy"
DEPLOY_DIR="/opt/myapp"
BACKUP_DIR="/opt/backups/myapp"
LOG_FILE="/var/log/deploy.log"

# ---- Functions ----
log() { echo "[$(date '+%Y-%m-%d %H:%M:%S')] $*" | tee -a "$LOG_FILE"; }
die() { log "ERROR: $1"; exit 1; }

# ---- Argument Handling ----
[[ $# -lt 2 ]] && die "Usage: $0 <version> <environment>"
VERSION=$1
ENVIRONMENT=$2

# ---- Validation ----
[[ "$ENVIRONMENT" =~ ^(production|staging|dev)$ ]] \
  || die "Invalid environment: $ENVIRONMENT. Must be production, staging, or dev"

command -v docker &>/dev/null || die "docker is required"

# ---- Pre-deployment Backup ----
log "Starting deployment of $APP_NAME v$VERSION to $ENVIRONMENT"
log "Backing up current version..."
mkdir -p "$BACKUP_DIR"
if [[ -d "$DEPLOY_DIR" ]]; then
  tar -czf "$BACKUP_DIR/backup-$(date +%Y%m%d-%H%M%S).tar.gz" "$DEPLOY_DIR"
fi

# ---- Deploy ----
log "Pulling image: $APP_NAME:$VERSION"
docker pull "$APP_NAME:$VERSION"

log "Stopping current container..."
docker stop "${APP_NAME}-${ENVIRONMENT}" 2>/dev/null || true
docker rm "${APP_NAME}-${ENVIRONMENT}" 2>/dev/null || true

log "Starting new container..."
docker run -d \
  --name "${APP_NAME}-${ENVIRONMENT}" \
  --restart=unless-stopped \
  -p 8080:8080 \
  -e "APP_ENV=$ENVIRONMENT" \
  "$APP_NAME:$VERSION"

# ---- Health Check ----
log "Waiting for health check..."
RETRIES=10
for (( i=1; i<=RETRIES; i++ )); do
  if curl -sf http://localhost:8080/health &>/dev/null; then
    log "Health check passed on attempt $i"
    break
  fi
  if (( i == RETRIES )); then
    die "Health check failed after $RETRIES attempts — rolling back"
  fi
  log "Attempt $i failed — retrying in 5s..."
  sleep 5
done

# ---- Cleanup ----
log "Cleaning old Docker images..."
docker image prune -f 2>/dev/null || true

log "Deployment complete: $APP_NAME v$VERSION on $ENVIRONMENT"
```

### Script 3: Log Analysis

```bash
#!/usr/bin/env bash
# analyze-logs.sh — Extract key metrics from nginx access logs
set -euo pipefail

LOGFILE="${1:-/var/log/nginx/access.log}"
[[ -f "$LOGFILE" ]] || { echo "Log file not found: $LOGFILE"; exit 1; }

echo "================================================"
echo " Nginx Access Log Analysis"
echo " File: $LOGFILE"
echo " Date: $(date)"
echo "================================================"

echo ""
echo "--- Request Count by Status Code ---"
awk '{print $9}' "$LOGFILE" \
  | sort | uniq -c | sort -rn | head -10

echo ""
echo "--- Top 10 IP Addresses ---"
awk '{print $1}' "$LOGFILE" \
  | sort | uniq -c | sort -rn | head -10

echo ""
echo "--- Top 10 Requested URLs ---"
awk '{print $7}' "$LOGFILE" \
  | cut -d? -f1 \
  | sort | uniq -c | sort -rn | head -10

echo ""
echo "--- 5xx Errors (Last 50) ---"
grep ' [5][0-9][0-9] ' "$LOGFILE" \
  | awk '{print $4, $7, $9}' \
  | tail -50

echo ""
echo "--- Total Requests ---"
wc -l < "$LOGFILE"
```

### Script 4: Automated Backup

```bash
#!/usr/bin/env bash
# backup.sh — Backup directories and rotate old backups
set -euo pipefail

BACKUP_DIRS=("/etc" "/home" "/var/www")
DEST="/mnt/backups"
RETAIN_DAYS=30
DATE=$(date +%Y%m%d-%H%M%S)
LOGFILE="/var/log/backup.log"

log() { echo "[$(date '+%Y-%m-%d %H:%M:%S')] $*" | tee -a "$LOGFILE"; }

mkdir -p "$DEST"

log "=== Backup started ==="

for dir in "${BACKUP_DIRS[@]}"; do
  if [[ ! -d "$dir" ]]; then
    log "WARNING: $dir not found — skipping"
    continue
  fi

  BASENAME=$(basename "$dir")
  ARCHIVE="$DEST/${BASENAME}-${DATE}.tar.gz"

  log "Backing up $dir → $ARCHIVE"
  tar -czf "$ARCHIVE" "$dir" 2>/dev/null

  SIZE=$(du -sh "$ARCHIVE" | cut -f1)
  log "  Archived: $SIZE"
done

# Rotate old backups
log "Rotating backups older than $RETAIN_DAYS days..."
find "$DEST" -name "*.tar.gz" -mtime "+$RETAIN_DAYS" -print -delete \
  | while read -r f; do log "  Deleted: $f"; done

log "=== Backup complete ==="
```

---

## 12. Script Debugging and Best Practices

### Debugging Techniques

```bash
# Run script with debug output (prints each command before executing)
bash -x my-script.sh

# Debug specific section inside script
set -x     # Turn on
   # ... suspect code ...
set +x     # Turn off

# Trace mode with timestamps (useful for profiling)
# Add to script top: PS4='+ $(date "+%H:%M:%S") ${BASH_SOURCE}:${LINENO}: '

# Dry run pattern — don't actually execute, just print
DRY_RUN=${DRY_RUN:-false}
run_cmd() {
  if $DRY_RUN; then
    echo "[DRY-RUN] $*"
  else
    "$@"
  fi
}

run_cmd systemctl restart nginx   # Prints command if DRY_RUN=true
# DRY_RUN=true ./deploy.sh prod v1.2.3
```

### Best Practices

```bash
#!/usr/bin/env bash
set -euo pipefail

# 1. Always quote variables (handles spaces in values)
# BAD:  rm -rf $DIRECTORY
# GOOD: rm -rf "$DIRECTORY"

# 2. Use [[ ]] not [ ] for conditionals
# BAD:  if [ $var = "value" ]; then
# GOOD: if [[ $var == "value" ]]; then

# 3. Use $(command) not backticks
# BAD:  RESULT=`command`
# GOOD: RESULT=$(command)

# 4. Declare local variables in functions
my_function() {
  local temp_var="value"   # Not visible outside function
}

# 5. Use meaningful variable names
# BAD:  d=$1; n=$2
# GOOD: deploy_dir=$1; app_name=$2

# 6. Handle errors explicitly
cp source.txt dest.txt || { echo "Copy failed"; exit 1; }

# 7. Use readonly for constants
readonly CONFIG_DIR="/etc/myapp"
readonly VERSION="1.0.0"

# 8. Validate all inputs before using them
validate_environment() {
  local env=$1
  case $env in
    production|staging|development) return 0 ;;
    *) echo "Invalid environment: $env"; return 1 ;;
  esac
}

# 9. Use mktemp for temporary files (not hardcoded /tmp/myfile)
TMPFILE=$(mktemp)
TMPDIR=$(mktemp -d)
trap 'rm -rf "$TMPFILE" "$TMPDIR"' EXIT

# 10. ShellCheck — static analysis for bash scripts
# Install: apt install shellcheck / brew install shellcheck
# Run: shellcheck my-script.sh
```

---

## 13. Summary & Cheatsheet

### Variables Cheatsheet

```bash
VAR="value"           # Assign
echo "$VAR"           # Use
echo "${VAR:-default}" # Default if unset
unset VAR             # Remove
export VAR            # Export to children
readonly VAR          # Make constant
local VAR             # Scope to function
```

### Conditionals Cheatsheet

```bash
[[ -f file ]]       # File exists
[[ -d dir ]]        # Directory exists
[[ -z "$str" ]]     # String is empty
[[ -n "$str" ]]     # String is not empty
[[ "$a" == "$b" ]]  # Strings equal
(( n > 5 ))         # Numeric compare
```

### Loops Cheatsheet

```bash
for x in a b c; do ... done
for i in {1..10}; do ... done
while [[ condition ]]; do ... done
while IFS= read -r line; do ... done < file
break             # Exit loop
continue          # Skip iteration
```

### Functions Cheatsheet

```bash
my_func() {
  local var=$1      # Local variable
  echo "result"     # Return string
  return 0          # Return exit code
}

result=$(my_func arg)  # Capture output
my_func arg            # Run for exit code
```

### Error Handling Cheatsheet

```bash
set -euo pipefail     # Safe script header
echo $?               # Check last exit code
cmd || exit 1         # Exit on failure
trap cleanup EXIT     # Cleanup on exit
[[ -z "$VAR" ]] && die "VAR required"
```

---

**Previous:** [03 — Text Processing & Networking](03-text-processing-and-networking.md)
**Next:** [05 — Cron & systemd](05-cron-and-systemd.md)
