# 03 — Text Processing & Networking

> "Text processing is how you turn raw logs and data into answers. Networking is how you validate that your services are actually talking to each other."

---

## Table of Contents

1. [Standard Streams Deep Dive](#1-standard-streams-deep-dive)
2. [grep — Search Text](#2-grep--search-text)
3. [sed — Stream Editor](#3-sed--stream-editor)
4. [awk — Pattern Scanning and Processing](#4-awk--pattern-scanning-and-processing)
5. [cut, sort, uniq, wc, tr](#5-cut-sort-uniq-wc-tr)
6. [xargs — Build Command Lines from Input](#6-xargs--build-command-lines-from-input)
7. [Networking Fundamentals](#7-networking-fundamentals)
8. [Network Troubleshooting Tools](#8-network-troubleshooting-tools)
9. [SSH — Secure Shell](#9-ssh--secure-shell)
10. [File Transfer — scp, rsync, wget, curl](#10-file-transfer--scp-rsync-wget-curl)
11. [DNS Tools](#11-dns-tools)
12. [DevOps Networking Patterns](#12-devops-networking-patterns)
13. [Summary & Cheatsheet](#13-summary--cheatsheet)

---

## 1. Standard Streams Deep Dive

Every Linux process has three default file descriptors:

```
0 = stdin  (standard input)    ← keyboard / pipe / redirect
1 = stdout (standard output)   → terminal / file / pipe
2 = stderr (standard error)    → terminal / file
```

```bash
# Redirect examples
cmd > out.txt           # stdout to file (overwrite)
cmd >> out.txt          # stdout append
cmd 2> err.txt          # stderr to file
cmd 2>&1                # stderr to wherever stdout goes
cmd > out.txt 2>&1      # both to file
cmd &> out.txt          # shorthand: both to file (bash 4+)
cmd > /dev/null 2>&1    # discard ALL output

# Practical: suppress only errors
find / -name "hosts" 2>/dev/null        # Find file, hide permission errors

# Practical: save output AND see it live
make build 2>&1 | tee build.log         # tee = output to file AND terminal

# Separate stdout and stderr
cmd > stdout.log 2> stderr.log

# Here-string (single line stdin)
grep "pattern" <<< "Some text to search in"
base64 <<< "encode me"
```

---

## 2. grep — Search Text

`grep` (Global Regular Expression Print) searches for patterns in text.

### Basic grep

```bash
grep "pattern" file.txt             # Search for pattern in file
grep "pattern" *.log                # Search in multiple files
grep "pattern" file1 file2          # Multiple explicit files
grep -r "pattern" /var/log/         # Recursive search in directory

# Common flags:
grep -i "error" app.log             # Case-Insensitive
grep -n "error" app.log             # Show line Numbers
grep -v "info" app.log              # inVert — lines NOT matching
grep -c "error" app.log             # Count matching lines
grep -l "error" /var/log/*.log      # List filenames with matches
grep -w "port" nginx.conf           # Whole Word match only
grep -A 3 "error" app.log          # 3 lines After match
grep -B 3 "error" app.log          # 3 lines Before match
grep -C 5 "error" app.log          # 5 lines Context (before+after)
grep -m 10 "error" app.log         # Show Max 10 matches then stop
grep -o "pattern" app.log          # Only show matching part (not whole line)

# Combine flags
grep -in "error" *.log              # Case insensitive + line numbers
grep -v "^#" nginx.conf | grep -v "^$"  # Remove comments and blank lines
```

### Extended grep and Regex

```bash
grep -E "pattern1|pattern2" file    # Extended regex — use | for OR
grep -E "ERROR|WARN|CRIT" app.log   # Multiple patterns
grep -E "^[0-9]{4}-[0-9]{2}" log   # Lines starting with date YYYY-MM
grep -P "(?<=:)\d+" file            # Perl regex (lookahead/behind)

# Useful DevOps grep patterns:
grep -E "^[^#]" /etc/nginx/nginx.conf        # Non-comment lines
grep -rn "password\|secret\|token" /etc/     # Find credentials in configs (audit)
grep -E "[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}" access.log  # Find IPs
grep -E "HTTP/[0-9]\.[0-9]\" [45]" access.log  # Find 4xx and 5xx errors
```

### Real-World grep Chaining

```bash
# Busiest IP addresses in nginx log
grep -oE "[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+" /var/log/nginx/access.log \
  | sort | uniq -c | sort -rn | head -20

# Count HTTP 500 errors per hour
grep " 500 " /var/log/nginx/access.log \
  | awk '{print $4}' \
  | cut -d: -f1-2 \
  | sort | uniq -c

# Find all running cron jobs for a specific user
crontab -l | grep -v "^#" | grep -v "^$"

# Find files containing a string, show filename and line number
grep -rn "JAVA_HOME" /etc/profile.d/ /etc/environment
```

---

## 3. sed — Stream Editor

`sed` processes text line by line, applying editing commands. Most commonly used for search-and-replace.

### Basic sed

```bash
# Syntax: sed OPTIONS 'COMMAND' file
# Most common command: s/pattern/replacement/flags  (substitute)

sed 's/foo/bar/' file.txt           # Replace first occurrence per line
sed 's/foo/bar/g' file.txt          # Replace all occurrences (global)
sed 's/foo/bar/gi' file.txt         # Case-insensitive replace, global
sed 's/foo/bar/2' file.txt          # Replace second occurrence only

# In-place editing (MODIFY THE FILE)
sed -i 's/foo/bar/g' file.txt           # Edit in place
sed -i.bak 's/foo/bar/g' file.txt       # Edit in place, keep .bak backup

# Deleting lines
sed '/pattern/d' file.txt           # Delete lines matching pattern
sed '/^#/d' file.txt                # Delete comment lines
sed '/^$/d' file.txt                # Delete blank lines
sed '/^#/d; /^$/d' file.txt         # Delete both (chain commands with ;)

# Printing specific lines
sed -n '5p' file.txt                # Print line 5 only (-n suppresses default output)
sed -n '5,10p' file.txt             # Print lines 5–10
sed -n '/start/,/end/p' file.txt    # Print lines between patterns
```

### Practical sed for DevOps

```bash
# Update a version number in a config file
sed -i 's/version: 1.0.0/version: 1.0.1/g' docker-compose.yml
sed -i "s/APP_VERSION=.*/APP_VERSION=${NEW_VERSION}/" .env

# Comment out a line
sed -i '/ServerName/s/^/#/' /etc/apache2/sites-enabled/mysite.conf

# Uncomment a line
sed -i '/^#ServerName/s/^#//' /etc/apache2/sites-enabled/mysite.conf

# Append a line after a pattern
sed -i '/^\[Service\]/a Environment=NODE_ENV=production' /etc/systemd/system/myapp.service

# Insert a line before a pattern
sed -i '/^ExecStart/i ExecStartPre=/usr/bin/sleep 2' /etc/systemd/system/myapp.service

# Remove trailing whitespace from all lines
sed -i 's/[[:space:]]*$//' file.txt

# Extract a block between markers (useful for extracting certs, config blocks)
sed -n '/-----BEGIN CERTIFICATE-----/,/-----END CERTIFICATE-----/p' bundle.crt

# Replace the 3rd occurrence of a pattern:
sed 's/old/new/3' file.txt

# Use a different delimiter (useful when pattern contains /)
sed 's|/old/path|/new/path|g' script.sh
```

---

## 4. awk — Pattern Scanning and Processing

`awk` processes text column by column. It's a mini programming language — ideal for structured data like log files, CSVs, and command output.

### awk Fundamentals

```bash
# Syntax: awk 'PATTERN { ACTION }' file
# Default separator: whitespace

# Built-in variables:
# $0 = entire line
# $1 = first field, $2 = second, $NF = last field
# NR = current line number
# NF = number of fields in current line
# FS = field separator (default: whitespace)
# OFS = output field separator

# Basic examples
awk '{print $1}' file.txt           # Print first column
awk '{print $1, $3}' file.txt       # Print columns 1 and 3
awk '{print NR, $0}' file.txt       # Print with line numbers
awk '{print NF}' file.txt           # Print field count per line
awk '{print $NF}' file.txt          # Print last field

# Custom field separator
awk -F: '{print $1, $3}' /etc/passwd       # : as separator → user and UID
awk -F, '{print $2}' data.csv              # CSV — second column
awk -F'\t' '{print $3}' data.tsv           # Tab-separated

# Pattern matching
awk '/ERROR/ {print}' app.log              # Print lines containing ERROR
awk '!/^#/ {print}' nginx.conf             # Print non-comment lines
awk '$3 > 100 {print $1, $3}' data.txt    # Print if column 3 > 100
awk 'NR==5' file.txt                       # Print line 5
awk 'NR>=5 && NR<=10' file.txt            # Print lines 5–10
```

### awk in DevOps — Real Examples

```bash
# Analyze nginx access.log
# Format: IP - - [date] "METHOD path HTTP" status bytes "referer" "agent"

# Top 10 IPs by request count
awk '{print $1}' /var/log/nginx/access.log \
  | sort | uniq -c | sort -rn | head -10

# Count HTTP status codes
awk '{print $9}' /var/log/nginx/access.log \
  | sort | uniq -c | sort -rn

# Average response time (if log has response time as last field)
awk '{sum += $NF; count++} END {print "Avg:", sum/count, "ms"}' timing.log

# Find all 5xx errors with timestamp and URL
awk '$9 ~ /^5/ {print $4, $7, $9}' /var/log/nginx/access.log

# Get memory usage of specific process
ps aux | grep nginx | awk '{sum+=$4} END {print "Total %MEM:", sum}'

# Parse docker stats
docker stats --no-stream | awk 'NR>1 {print $2, $3}'  # Container name + CPU

# List users with UID >= 1000 (regular users)
awk -F: '$3 >= 1000 && $3 < 65534 {print $1, $3}' /etc/passwd

# Print lines between two patterns (inclusive)
awk '/^server {/,/^}/' /etc/nginx/nginx.conf

# Calculate total disk usage from du output
du -sh /var/log/* | awk '{sum+=$1} END {print sum, "total"}'

# Extract unique endpoints from access log
awk '{print $7}' /var/log/nginx/access.log | cut -d? -f1 | sort | uniq -c | sort -rn

# Sum a column (e.g., bytes transferred)
awk '{sum += $10} END {printf "Total: %.2f GB\n", sum/1024/1024/1024}' access.log
```

---

## 5. cut, sort, uniq, wc, tr

These simple tools become powerful when combined with pipes.

### `cut` — Extract Columns

```bash
cut -d: -f1 /etc/passwd             # Delimiter : field 1 (usernames)
cut -d: -f1,3 /etc/passwd           # Fields 1 and 3
cut -d, -f2 data.csv                # CSV second field
cut -c1-10 file.txt                 # Characters 1–10 of each line
cut -c5- file.txt                   # From character 5 to end
```

### `sort` — Sort Lines

```bash
sort file.txt                       # Alphabetical sort
sort -r file.txt                    # Reverse sort
sort -n file.txt                    # Numeric sort
sort -rn file.txt                   # Reverse numeric
sort -k2 file.txt                   # Sort by field 2
sort -k2,2n file.txt                # Sort numerically by field 2
sort -t: -k3 -n /etc/passwd         # Sort passwd by UID (numeric, : separator)
sort -u file.txt                    # Sort and remove duplicates
sort -h file.txt                    # Human-readable numbers (1K, 2M)

# Sort disk usage output by size (largest first)
du -sh /var/* | sort -rh | head -10
```

### `uniq` — Filter/Count Duplicates

```bash
uniq file.txt                       # Remove ADJACENT duplicates (must sort first!)
sort file.txt | uniq                # Remove all duplicates
sort file.txt | uniq -c             # Count occurrences of each line
sort file.txt | uniq -d             # Show only duplicate lines
sort file.txt | uniq -u             # Show only unique lines (appear once)
sort file.txt | uniq -c | sort -rn  # Frequency table (most common first)
```

### `wc` — Word Count

```bash
wc file.txt                         # Lines, words, bytes
wc -l file.txt                      # Lines only
wc -w file.txt                      # Words only
wc -c file.txt                      # Bytes
wc -m file.txt                      # Characters (handles multibyte)

# Count log entries per minute
grep "ERROR" app.log | wc -l        # How many errors today?
ls /var/log/*.log | wc -l           # How many log files?
```

### `tr` — Translate Characters

```bash
tr 'a-z' 'A-Z' < file.txt           # Lowercase to uppercase
tr -d '\r' < windows.txt > unix.txt  # Remove Windows carriage returns
tr -d ' ' < file.txt                 # Remove all spaces
tr -s ' ' < file.txt                 # Squeeze multiple spaces into one
tr ':' '\n' <<< "$PATH"              # Show PATH entries one per line
echo "hello world" | tr ' ' '_'     # Replace spaces with underscores
```

---

## 6. xargs — Build Command Lines from Input

`xargs` reads from stdin and constructs command arguments — bridges the gap between pipes and commands that don't read stdin.

```bash
# Basic usage
echo "file1 file2 file3" | xargs rm          # Delete those files
find /tmp -name "*.tmp" | xargs rm -f         # Delete all .tmp files

# Control how arguments are passed
cat urls.txt | xargs -I{} curl -O {}         # {} is placeholder for each line
ls *.conf | xargs -I{} cp {} /backup/{}.bak  # Copy each with .bak suffix

# Parallel execution
find . -name "*.jpg" | xargs -P4 -I{} convert {} {}.png   # Convert 4 at a time

# Handle filenames with spaces
find . -name "*.log" -print0 | xargs -0 rm   # NUL delimiter for safety

# Limit arguments per command invocation
cat files.txt | xargs -n1 chmod 644          # Run chmod once per file
cat files.txt | xargs -n10 ls -la            # Pass 10 args at a time

# Practical DevOps examples
# Restart all services matching pattern
systemctl list-units --state=failed --no-legend | awk '{print $1}' \
  | xargs systemctl restart

# Delete old Docker images
docker images -q --filter "dangling=true" | xargs docker rmi

# Run tests on multiple files
find . -name "*_test.go" | xargs go test
```

---

## 7. Networking Fundamentals

### OSI Model (Quick Reference)

```
Layer 7  Application   HTTP, DNS, SSH, FTP, SMTP
Layer 6  Presentation  TLS/SSL, encoding
Layer 5  Session       Session management
Layer 4  Transport     TCP (reliable), UDP (fast)
Layer 3  Network       IP addressing, routing
Layer 2  Data Link     MAC addresses, Ethernet, switches
Layer 1  Physical      Cables, signals
```

**DevOps focus:** Layers 3–7. You'll diagnose issues at Layer 4 (TCP ports), Layer 3 (routing), and Layer 7 (HTTP).

### Key Networking Concepts

```bash
# IP address types
192.168.0.0/24    # Private (RFC 1918) — home/office network
10.0.0.0/8        # Private — often used in cloud VPCs
172.16.0.0/12     # Private
127.0.0.1         # Localhost — loopback
0.0.0.0           # All interfaces (bind to any)

# CIDR notation
/24 = 256 addresses (255.255.255.0)
/16 = 65536 addresses
/8  = 16.7M addresses
/32 = exactly 1 IP

# Well-known ports
22   SSH
25   SMTP
53   DNS
80   HTTP
443  HTTPS
3306 MySQL
5432 PostgreSQL
6379 Redis
8080 Alternative HTTP / app servers
9090 Prometheus
27017 MongoDB
```

---

## 8. Network Troubleshooting Tools

### `ip` — Modern Network Configuration

```bash
ip addr                              # Show all interfaces and IP addresses
ip addr show eth0                    # Show specific interface
ip link                              # Show link layer info
ip link show eth0                    # Specific interface status
ip route                             # Show routing table
ip route show default                # Show default gateway

# Add/remove IP address (temporary — lost on reboot)
ip addr add 192.168.1.100/24 dev eth0
ip addr del 192.168.1.100/24 dev eth0

# Bring interface up/down
ip link set eth0 up
ip link set eth0 down

# ip replaces the older: ifconfig, route, netstat commands
# ifconfig eth0    → ip addr show eth0
# route -n         → ip route
```

### `ss` and `netstat` — Socket Statistics

```bash
ss -tlnp                    # TCP Listening sockets with process info
# -t = TCP
# -l = listening only
# -n = numeric (don't resolve hostnames)
# -p = show process

ss -tlnp | grep :80         # What's on port 80?
ss -tlnp | grep :443        # What's on 443?
ss -tulnp                   # TCP + UDP listening
ss -an                      # All connections, numeric
ss -tn state established    # Active TCP connections

# netstat (older, may not be installed)
netstat -tlnp               # Same as ss -tlnp
netstat -an | grep LISTEN   # Can also filter for listening
netstat -s                  # Network statistics summary

# Common use case: find what's using port 80
ss -tlnp | grep ':80'
# LISTEN  0  128  0.0.0.0:80  0.0.0.0:*  users:(("nginx",pid=1234,fd=6))
```

### `ping` and `traceroute`

```bash
ping 8.8.8.8                # Ping Google DNS — basic connectivity test
ping -c 4 8.8.8.8           # Ping 4 times then stop
ping -i 0.5 8.8.8.8         # Ping every 0.5 seconds
ping -s 1400 hostname       # Ping with larger packet (MTU testing)

traceroute 8.8.8.8          # Trace route to destination (may need install)
traceroute -n 8.8.8.8       # Numeric only (no DNS lookup, faster)
mtr 8.8.8.8                 # Combined ping + traceroute in real-time

# Check connectivity inside containers or restricted networks
curl -s --connect-timeout 3 http://service-name:8080/health
```

### `curl` — Transfer Data with URLs

`curl` is one of the most important tools for testing APIs and services:

```bash
# Basic GET request
curl http://localhost:8080
curl -s http://localhost:8080          # Silent (no progress)
curl -i http://localhost:8080          # Include response headers
curl -I http://localhost:8080          # HEAD request only
curl -v http://localhost:8080          # Verbose (full request/response)

# POST request (REST API testing)
curl -X POST http://api/endpoint \
  -H "Content-Type: application/json" \
  -d '{"key": "value"}'

curl -X POST http://api/endpoint \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d @payload.json                      # Read body from file

# Check HTTP status code only
curl -o /dev/null -s -w "%{http_code}" http://localhost:8080/health
# Output: 200

# Follow redirects
curl -L http://example.com

# Download a file
curl -O https://example.com/file.tar.gz           # Keep original filename
curl -o myfile.tar.gz https://example.com/file    # Custom filename

# Set timeout
curl --max-time 10 http://service:8080/health     # Fail after 10s

# Use with a proxy
curl -x http://proxy:3128 http://target.com

# TLS/Certificate testing
curl --insecure https://self-signed.example.com   # Skip cert validation
curl --cacert /etc/ssl/certs/ca.crt https://internal.api  # Custom CA

# Health check script pattern:
check_health() {
  local status
  status=$(curl -o /dev/null -s -w "%{http_code}" \
    --max-time 5 "http://localhost:${PORT}/health")
  if [[ "$status" == "200" ]]; then
    echo "Healthy"
  else
    echo "Unhealthy: HTTP $status"
    return 1
  fi
}
```

---

## 9. SSH — Secure Shell

SSH is the most critical tool for remote server administration in DevOps.

### Basic SSH Usage

```bash
ssh user@hostname              # Connect to remote server
ssh user@192.168.1.100         # Connect by IP
ssh -p 2222 user@hostname      # Non-standard port
ssh -i ~/.ssh/mykey user@host  # Specify private key file
ssh -v user@host               # Verbose — debug connection issues
ssh -vvv user@host             # Very verbose

# Run a command remotely without interactive session
ssh user@server "systemctl status nginx"
ssh user@server "df -h && free -h"
ssh user@server "tail -100 /var/log/app.log"

# Use bastion/jump host
ssh -J bastion-user@bastion.example.com target-user@internal-server

# SSH tunneling (port forwarding):
# Local forward — access remote DB at localhost:5432
ssh -L 5432:db-server:5432 user@jump-server

# Remote forward — expose local service through remote server
ssh -R 8080:localhost:3000 user@public-server

# Dynamic forward — SOCKS proxy
ssh -D 9090 user@server
```

### SSH Key Management

```bash
# Generate SSH key pair
ssh-keygen -t ed25519 -C "ajaz@work"          # Modern: Ed25519
ssh-keygen -t rsa -b 4096 -C "ajaz@work"      # RSA 4096-bit

# Keys are stored at:
~/.ssh/id_ed25519      # Private key (KEEP SECRET — chmod 600)
~/.ssh/id_ed25519.pub  # Public key (can be shared)

# Copy public key to remote server (sets up passwordless login)
ssh-copy-id user@server                        # Appends to authorized_keys
ssh-copy-id -i ~/.ssh/mykey.pub user@server    # Specific key

# Manually add key (when ssh-copy-id not available):
cat ~/.ssh/id_ed25519.pub >> ~/.ssh/authorized_keys
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys

# SSH config file — avoid typing long commands
cat ~/.ssh/config
# Host prod-web
#     HostName 10.0.0.100
#     User ubuntu
#     IdentityFile ~/.ssh/prod-key
#     Port 22
#
# Host bastion
#     HostName bastion.example.com
#     User ec2-user
#     IdentityFile ~/.ssh/aws-key.pem

# Use: ssh prod-web     (instead of: ssh -i ~/.ssh/prod-key ubuntu@10.0.0.100)

# Start ssh-agent (manage keys in memory)
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519                      # Add key to agent
ssh-add -l                                     # List loaded keys
```

### SSH Server Configuration

```bash
# Key configuration file: /etc/ssh/sshd_config
# Common security hardening:

Port 22                          # Change to non-standard port (e.g., 2222)
PermitRootLogin no               # Never allow direct root login
PasswordAuthentication no        # Keys only — no passwords
PubkeyAuthentication yes
AllowUsers ubuntu deploy         # Only these users can SSH
MaxAuthTries 3
ClientAliveInterval 300          # Close idle sessions after 5 min
ClientAliveCountMax 2

# After changing sshd_config:
sshd -t && systemctl reload sshd   # Test config, then reload

# Check who is currently logged in via SSH
who
w
ss -tnp | grep :22
```

---

## 10. File Transfer — scp, rsync, wget, curl

### `scp` — Secure Copy

```bash
# Copy local file to remote
scp file.txt user@server:/remote/path/

# Copy remote file to local
scp user@server:/remote/file.txt ./

# Copy entire directory
scp -r ./mydir user@server:/remote/

# Use specific key
scp -i ~/.ssh/mykey file.txt user@server:/path/

# Copy between two remote servers (through local machine)
scp user1@server1:/file user2@server2:/destination
```

### `rsync` — Efficient File Sync

`rsync` only transfers changed parts — far more efficient than `scp` for repeated transfers.

```bash
# Sync local dir to remote
rsync -avz ./mydir/ user@server:/remote/mydir/
# -a = archive mode (preserve permissions, timestamps, symlinks)
# -v = verbose
# -z = compress during transfer

# Sync remote to local
rsync -avz user@server:/remote/logs/ ./local-logs/

# Dry run — see what would change without doing it
rsync -avzn ./mydir/ user@server:/remote/

# Delete files at destination that no longer exist at source
rsync -avz --delete ./mydir/ user@server:/remote/

# Exclude files
rsync -avz --exclude "*.log" --exclude ".git/" ./myapp/ user@server:/opt/myapp/

# Deployment pattern with rsync:
rsync -avz --delete \
  --exclude ".git/" \
  --exclude "node_modules/" \
  --exclude "*.env" \
  ./build/ deploy@webserver:/var/www/myapp/
```

### `wget` — Download Files

```bash
wget https://example.com/file.tar.gz            # Download file
wget -O newname.tar.gz https://example.com/file  # Custom filename
wget -q https://example.com/file                 # Quiet — minimal output
wget --no-check-certificate https://...          # Skip SSL check
wget -r -np https://example.com/docs/           # Recursive download
wget -c https://example.com/large.file          # Resume partial download
wget -P /tmp/ https://example.com/file          # Download to /tmp/

# Download in background
wget -b -o wget.log https://example.com/large.file
```

---

## 11. DNS Tools

### `dig` — DNS Lookup

```bash
dig example.com                    # Default A record lookup
dig example.com A                  # Explicitly ask for A records
dig example.com AAAA               # IPv6 address
dig example.com MX                 # Mail exchange records
dig example.com NS                 # Name servers
dig example.com TXT                # TXT records (SPF, DKIM, verification)
dig example.com CNAME              # Canonical name (alias)

dig @8.8.8.8 example.com           # Query specific DNS server (Google)
dig @1.1.1.1 example.com           # Query Cloudflare DNS

dig +short example.com             # Short output — just the answer
dig +trace example.com             # Trace full resolution path

# Reverse DNS (IP to hostname)
dig -x 8.8.8.8
# 8.8.8.8.in-addr.arpa   →  dns.google.

# Check if DNS change has propagated
dig +short example.com @8.8.8.8     # Google's view
dig +short example.com @1.1.1.1     # Cloudflare's view
dig +short example.com @your-dns   # Your server's view
```

### `nslookup` and `host`

```bash
nslookup example.com               # Older but widely available
nslookup -type=MX example.com      # Query specific type
nslookup example.com 8.8.8.8       # Query specific server

host example.com                   # Simple forward lookup
host -t MX example.com            # MX records
host 8.8.8.8                       # Reverse lookup
```

---

## 12. DevOps Networking Patterns

### Health Check Patterns

```bash
#!/bin/bash
# HTTP health check
check_http() {
  local url=$1
  local expected_status=${2:-200}
  local status
  status=$(curl -o /dev/null -s -w "%{http_code}" --max-time 5 "$url")
  if [[ "$status" == "$expected_status" ]]; then
    echo "OK: $url returned $status"
    return 0
  else
    echo "FAIL: $url returned $status (expected $expected_status)"
    return 1
  fi
}

check_http "http://localhost:8080/health"
check_http "https://api.example.com/v1/status" 200
```

### Port Connectivity Testing

```bash
# Check if a port is reachable
nc -zv hostname 5432               # Check if PostgreSQL port is open
nc -zv hostname 6379               # Check Redis
nc -z -w3 hostname port             # -w3 = timeout 3 seconds

# Using bash built-in (no nc required in scripts)
timeout 3 bash -c 'cat < /dev/null > /dev/tcp/hostname/5432' \
  && echo "Port open" || echo "Port closed"

# Using curl for TCP check
curl -v telnet://hostname:5432

# nmap — comprehensive port scanning
nmap -p 80,443,8080 hostname       # Scan specific ports
nmap -sV hostname                   # Detect service versions
nmap -p 1-1000 hostname            # Scan first 1000 ports
```

### Firewall (iptables / ufw / firewalld)

```bash
# ufw — Uncomplicated Firewall (Debian/Ubuntu)
ufw status                         # Current rules
ufw allow 22/tcp                   # Allow SSH
ufw allow 80/tcp                   # Allow HTTP
ufw allow 443/tcp                  # Allow HTTPS
ufw deny 8080                      # Deny port 8080
ufw enable                         # Activate firewall
ufw delete allow 80               # Remove a rule

# firewalld (RHEL/CentOS)
firewall-cmd --list-all
firewall-cmd --add-service=http --permanent
firewall-cmd --add-port=8080/tcp --permanent
firewall-cmd --reload

# iptables (low-level — for advanced use)
iptables -L                        # List rules
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
iptables -A INPUT -p tcp --dport 22 -j ACCEPT
iptables -A INPUT -j DROP          # Drop everything else
```

### Network Interface Info — AWS/Cloud Context

```bash
# On AWS EC2 — get instance metadata
curl -s http://169.254.169.254/latest/meta-data/public-ipv4    # Public IP
curl -s http://169.254.169.254/latest/meta-data/local-ipv4     # Private IP
curl -s http://169.254.169.254/latest/meta-data/instance-id    # Instance ID

# On GCP
curl -s -H "Metadata-Flavor: Google" \
  "http://metadata.google.internal/computeMetadata/v1/instance/network-interfaces/0/access-configs/0/external-ip"
```

---

## 13. Summary & Cheatsheet

### grep Cheatsheet

```bash
grep -r "text" dir/          # Recursive search
grep -in "text" file         # Case-insensitive + line numbers
grep -v "text" file          # Invert match
grep -A 3 "text" file        # 3 lines after
grep -E "pat1|pat2" file     # Extended regex (OR)
grep -c "text" file          # Count matches
```

### sed Cheatsheet

```bash
sed 's/old/new/g' file       # Replace all
sed -i 's/old/new/g' file    # Edit in place
sed '/pattern/d' file        # Delete matching lines
sed -n '5,10p' file          # Print lines 5-10
sed -i.bak 's/a/b/g' file   # Edit with backup
```

### awk Cheatsheet

```bash
awk '{print $1}' file         # First column
awk -F: '{print $1}' file     # Custom delimiter
awk '/pattern/ {print}' file  # Print matching lines
awk '{sum+=$1} END {print sum}' file  # Sum first column
awk 'NR>1' file               # Skip first line (header)
```

### Networking Cheatsheet

```bash
ip addr                   # Network interfaces
ss -tlnp                  # Listening ports
ping -c 4 host            # Test connectivity
curl -I http://host       # HTTP check
ssh user@host             # Remote login
scp file user@host:/path  # Secure copy
rsync -avz src/ user@host:/dst/  # Sync files
dig +short domain.com     # DNS lookup
nc -zv host port          # TCP port test
```

---

**Previous:** [02 — Users, Permissions & Processes](02-users-permissions-processes.md)
**Next:** [04 — Bash Scripting](04-bash-scripting.md)
