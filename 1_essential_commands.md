# LFCS Practice: Essential Commands

This guide provides hands-on exercises designed to prepare you for the Linux Foundation Certified System Administrator (LFCS) exam. All tasks are designed for **Ubuntu 22.04 LTS** and simulate real-world system administration scenarios.

---

## Task 1: Log Analysis and Text Processing Pipeline

### Learning Objectives
- Master text processing tools: `grep`, `awk`, `sed`, `cut`, `sort`, and `uniq`
- Implement advanced command pipelines with redirection operators (`|`, `>`, `>>`, `tee`)
- Extract actionable insights from system logs to troubleshoot issues

### Context
You're a system administrator at a web hosting company. The senior engineer has reported that one of the production web servers has been experiencing intermittent connection failures. Users are complaining about slow response times during peak hours. Your task is to analyze the Apache/Nginx access logs to identify patterns: which IP addresses are making the most requests, what are the most frequently accessed URLs, and are there any suspicious patterns that might indicate a DDoS attack or misconfigured client applications?

This skill is essential for LFCS because log analysis is a daily task for system administrators. Being able to quickly extract meaningful data from logs helps you identify security threats, optimize performance, and troubleshoot issues before they escalate.

### Task Instructions

**Prerequisites**: Set up a sample web server log file for analysis
```bash
sudo apt update
sudo apt install -y apache2
# Generate some sample log entries
sudo bash -c 'cat > /var/log/apache2/sample_access.log << EOF
192.168.1.10 - - [29/Dec/2025:10:15:23 +0000] "GET /index.html HTTP/1.1" 200 1234
192.168.1.10 - - [29/Dec/2025:10:15:24 +0000] "GET /about.html HTTP/1.1" 200 2345
10.0.0.5 - - [29/Dec/2025:10:16:01 +0000] "GET /api/users HTTP/1.1" 404 512
192.168.1.10 - - [29/Dec/2025:10:16:15 +0000] "GET /index.html HTTP/1.1" 200 1234
203.0.113.50 - - [29/Dec/2025:10:17:30 +0000] "POST /login HTTP/1.1" 500 128
10.0.0.5 - - [29/Dec/2025:10:17:45 +0000] "GET /api/users HTTP/1.1" 404 512
192.168.1.10 - - [29/Dec/2025:10:18:00 +0000] "GET /contact.html HTTP/1.1" 200 987
203.0.113.50 - - [29/Dec/2025:10:18:20 +0000] "POST /login HTTP/1.1" 200 2048
10.0.0.5 - - [29/Dec/2025:10:19:00 +0000] "GET /api/users HTTP/1.1" 404 512
192.168.1.20 - - [29/Dec/2025:10:19:30 +0000] "GET /index.html HTTP/1.1" 200 1234
203.0.113.50 - - [29/Dec/2025:10:20:00 +0000] "GET /dashboard HTTP/1.1" 200 4096
192.168.1.10 - - [29/Dec/2025:10:20:15 +0000] "GET /index.html HTTP/1.1" 304 0
172.16.50.100 - - [29/Dec/2025:10:20:45 +0000] "GET /api/products HTTP/1.1" 200 5120
192.168.1.10 - - [29/Dec/2025:10:21:00 +0000] "GET /index.html HTTP/1.1" 200 1234
198.51.100.25 - - [29/Dec/2025:10:21:30 +0000] "GET /search?q=test HTTP/1.1" 200 3072
10.0.0.5 - - [29/Dec/2025:10:22:00 +0000] "GET /api/users HTTP/1.1" 404 512
192.168.1.50 - - [29/Dec/2025:10:22:15 +0000] "POST /api/submit HTTP/1.1" 201 256
172.16.50.100 - - [29/Dec/2025:10:22:45 +0000] "GET /api/products HTTP/1.1" 200 5120
192.168.1.10 - - [29/Dec/2025:10:23:00 +0000] "GET /about.html HTTP/1.1" 200 2345
203.0.113.50 - - [29/Dec/2025:10:23:30 +0000] "GET /dashboard HTTP/1.1" 200 4096
198.51.100.25 - - [29/Dec/2025:10:24:00 +0000] "GET /index.html HTTP/1.1" 200 1234
192.168.1.75 - - [29/Dec/2025:10:24:30 +0000] "GET /contact.html HTTP/1.1" 200 987
10.0.0.12 - - [29/Dec/2025:10:25:00 +0000] "POST /api/data HTTP/1.1" 403 128
192.168.1.10 - - [29/Dec/2025:10:25:15 +0000] "GET /index.html HTTP/1.1" 200 1234
172.16.50.100 - - [29/Dec/2025:10:25:45 +0000] "GET /api/products HTTP/1.1" 200 5120
EOF'
```

**Your Tasks**:
1. Extract and count the top 5 IP addresses making the most requests. Save the results to `~/top_ips.txt`
2. Find all requests that resulted in errors (HTTP status codes 4xx or 5xx) and save them to `~/error_requests.txt`
3. Create a report showing the most frequently accessed URLs (the path part of the request). Save the top 5 to `~/top_urls.txt`
4. Use `sed` to replace all private IP addresses (192.168.x.x and 10.x.x.x) with the text "INTERNAL_IP" and save the modified log to `~/anonymized.log`
5. Create a single pipeline that: filters for successful requests (status 200), extracts the requested URLs, sorts them, and uses `tee` to both display the results on screen AND save them to `~/successful_urls.txt`

### Hints and Resources
1. The `awk` command can extract specific fields from text. In Apache logs, the IP is typically the first field. See `man awk` or [GNU Awk User's Guide](https://www.gnu.org/software/gawk/manual/gawk.html)
2. Use `grep -E` for extended regular expressions to match patterns like status codes
3. The `cut` command with `-d` (delimiter) can help extract specific parts of strings
4. `sort | uniq -c` is a powerful combination for counting occurrences. Add `sort -rn` to get the highest counts first
5. Reference: [Ubuntu grep manual](https://manpages.ubuntu.com/manpages/jammy/man1/grep.1.html) and [AWK tutorial](https://www.linuxfoundation.org/)

### Estimated Time and Difficulty
**30-45 minutes, Intermediate**

### Verification
To verify your solution:
- Check that `~/top_ips.txt` contains IP addresses with counts, sorted from most to least frequent
- Verify `~/error_requests.txt` contains only log lines with 4xx or 5xx status codes
- Confirm `~/top_urls.txt` shows URL paths (like `/index.html`, `/api/users`) with their frequencies
- Open `~/anonymized.log` and ensure private IPs are replaced but public IPs (like 203.0.113.50) remain unchanged
- Check that `~/successful_urls.txt` exists and contains the same URLs you saw on screen

<details>
<summary>Solution</summary>

```bash
# Task 1: Extract top 5 IP addresses
awk '{print $1}' /var/log/apache2/sample_access.log | sort | uniq -c | sort -rn | head -5 > ~/top_ips.txt

# Alternative using cut:
cut -d' ' -f1 /var/log/apache2/sample_access.log | sort | uniq -c | sort -rn | head -5 > ~/top_ips.txt

# Task 2: Find error requests (4xx and 5xx status codes)
grep -E '" [45][0-9]{2} ' /var/log/apache2/sample_access.log > ~/error_requests.txt

# Alternative using awk:
awk '$9 ~ /^[45]/ {print}' /var/log/apache2/sample_access.log > ~/error_requests.txt

# Task 3: Most frequently accessed URLs
awk '{print $7}' /var/log/apache2/sample_access.log | sort | uniq -c | sort -rn | head -5 > ~/top_urls.txt

# Alternative approach:
grep -oP '"\w+ \K[^ ]+' /var/log/apache2/sample_access.log | sort | uniq -c | sort -rn | head -5 > ~/top_urls.txt

# Task 4: Anonymize private IPs using sed
sed -E 's/192\.168\.[0-9]+\.[0-9]+/INTERNAL_IP/g; s/10\.[0-9]+\.[0-9]+\.[0-9]+/INTERNAL_IP/g' /var/log/apache2/sample_access.log > ~/anonymized.log

# Task 5: Pipeline with tee for successful requests
awk '$9 == 200 {print $7}' /var/log/apache2/sample_access.log | sort | tee ~/successful_urls.txt
```

**Explanation**:
- **Task 1**: `awk '{print $1}'` extracts the first field (IP address), `sort | uniq -c` counts unique occurrences, `sort -rn` sorts numerically in reverse order, and `head -5` shows top 5
- **Task 2**: `grep -E '" [45][0-9]{2} '` uses regex to match 4xx or 5xx status codes in the log format
- **Task 3**: `awk '{print $7}'` extracts the URL field (7th field in standard Apache log format)
- **Task 4**: `sed -E` uses extended regex with two substitution patterns to replace private IP ranges
- **Task 5**: The pipeline filters for status 200, extracts URLs, sorts them, and `tee` writes to file while also displaying on stdout

</details>

### Extensions
1. **Advanced**: Parse the timestamp field and create a report showing request distribution by hour
2. **Challenge**: Write a bash script that monitors the log file in real-time (using `tail -f`) and sends an alert (prints a message) if more than 10 error requests occur within 1 minute

---

## Task 2: Disk Space Management and File Operations

### Learning Objectives
- Diagnose and troubleshoot disk space issues using `df`, `du`, and `find`
- Master file archival and compression tools: `tar`, `gzip`, `bzip2`, `xz`, and `zip`
- Implement efficient file search strategies with complex `find` predicates

### Context
You've just received an urgent alert: the production database server's root partition is at 95% capacity, and the database is about to stop accepting writes. The development team has been storing old database backups and log files in various directories, and nobody has cleaned them up in months. Your mission is to quickly identify what's consuming the most space, archive old files to free up disk space, and implement a cleanup strategy.

Understanding disk space management is critical for the LFCS exam and real-world scenarios. Servers running out of disk space can cause application failures, data loss, and service outages. Being able to quickly identify, archive, and clean up files is an essential skill.

### Task Instructions

**Prerequisites**: Set up a test environment with files
```bash
# Create test directory structure
mkdir -p ~/disk_test/{logs,backups,data,temp}

# Generate sample files
dd if=/dev/zero of=~/disk_test/logs/app.log bs=1M count=50
dd if=/dev/zero of=~/disk_test/logs/old_app.log.1 bs=1M count=30
dd if=/dev/zero of=~/disk_test/backups/db_backup_2024_01.sql bs=1M count=100
dd if=/dev/zero of=~/disk_test/backups/db_backup_2024_06.sql bs=1M count=100
dd if=/dev/zero of=~/disk_test/backups/db_backup_2025_12.sql bs=1M count=100
dd if=/dev/zero of=~/disk_test/data/important.dat bs=1M count=20
dd if=/dev/zero of=~/disk_test/temp/cache1.tmp bs=1M count=15
dd if=/dev/zero of=~/disk_test/temp/cache2.tmp bs=1M count=15

# Create some old files (modified more than 180 days ago)
touch -d "200 days ago" ~/disk_test/logs/old_app.log.1
touch -d "250 days ago" ~/disk_test/backups/db_backup_2024_01.sql
touch -d "150 days ago" ~/disk_test/backups/db_backup_2024_06.sql
```

**Your Tasks**:
1. Use `du` to identify which subdirectory under `~/disk_test/` is consuming the most space. Save the output showing all subdirectories with human-readable sizes to `~/disk_usage.txt`
2. Find all files larger than 40MB in the `~/disk_test/` directory tree and list them with their sizes in a human-readable format. Save to `~/large_files.txt`
3. Find all files in `~/disk_test/` that were modified more than 180 days ago
4. Create a compressed archive named `~/old_backups.tar.gz` containing all backup files older than 180 days from `~/disk_test/backups/`. After creating the archive, verify its contents without extracting
5. Create three different compressed versions of `~/disk_test/logs/app.log`: one with `gzip` (`.gz`), one with `bzip2` (`.bz2`), and one with `xz` (`.xz`). Compare their sizes and compression times
6. Find and delete all `.tmp` files in the `~/disk_test/temp/` directory, then verify they're gone

### Hints and Resources
1. `du -h` provides human-readable output. Use `--max-depth` or `-d` to limit directory recursion depth
2. The `find` command has powerful predicates like `-size`, `-mtime`, and `-type`. See `man find`
3. For `tar`, the `czf` flags create a gzipped archive. Use `tzf` to list contents without extracting
4. Use `time` command to measure compression times: `time gzip file.log`
5. References: [Ubuntu find manual](https://manpages.ubuntu.com/manpages/jammy/man1/find.1.html), [GNU tar guide](https://www.gnu.org/software/tar/manual/)

### Estimated Time and Difficulty
**25-35 minutes, Intermediate**

### Verification
- `~/disk_usage.txt` should show all subdirectories with their sizes
- `~/large_files.txt` should list files bigger than 40MB
- Your `find` command for old files should return at least 2 files
- `~/old_backups.tar.gz` should exist and contain the old backup files when you list it
- You should have three compressed versions of `app.log` with different extensions and sizes
- The `~/disk_test/temp/` directory should be empty after deletion

<details>
<summary>Solution</summary>

```bash
# Task 1: Identify disk usage by subdirectory
du -h --max-depth=1 ~/disk_test/ | sort -hr > ~/disk_usage.txt
# Alternative:
du -h -d 1 ~/disk_test/ | sort -hr > ~/disk_usage.txt

# Task 2: Find files larger than 40MB
find ~/disk_test/ -type f -size +40M -exec ls -lh {} \; | awk '{print $5, $9}' > ~/large_files.txt
# Alternative simpler version:
find ~/disk_test/ -type f -size +40M -ls | awk '{print $7, $11}' > ~/large_files.txt

# Task 3: Find files modified more than 180 days ago
find ~/disk_test/ -type f -mtime +180

# Task 4: Archive old backups
find ~/disk_test/backups/ -type f -mtime +180 -print0 | tar czf ~/old_backups.tar.gz --null -T -
# Alternative traditional approach:
tar czf ~/old_backups.tar.gz $(find ~/disk_test/backups/ -type f -mtime +180)

# Verify archive contents:
tar tzf ~/old_backups.tar.gz

# Task 5: Create different compressions and compare
# First, make copies to compress (since compression removes the original)
cp ~/disk_test/logs/app.log ~/app.log.copy1
cp ~/disk_test/logs/app.log ~/app.log.copy2
cp ~/disk_test/logs/app.log ~/app.log.copy3

# Compress with different algorithms
time gzip ~/app.log.copy1      # Creates app.log.copy1.gz
time bzip2 ~/app.log.copy2     # Creates app.log.copy2.bz2
time xz ~/app.log.copy3        # Creates app.log.copy3.xz

# Compare sizes
ls -lh ~/app.log.copy*

# Task 6: Delete .tmp files
find ~/disk_test/temp/ -type f -name "*.tmp" -delete

# Verify deletion:
ls ~/disk_test/temp/
# Alternative deletion method:
find ~/disk_test/temp/ -type f -name "*.tmp" -exec rm {} \;
```

**Explanation**:
- **Task 1**: `du -h --max-depth=1` shows disk usage for immediate subdirectories only, `sort -hr` sorts by size (human-readable, reverse order)
- **Task 2**: `find` with `-size +40M` finds files larger than 40 megabytes. `-exec ls -lh {} \;` shows details
- **Task 3**: `-mtime +180` means modified more than 180 days ago
- **Task 4**: Using `find` with `-print0` and `tar --null -T -` handles filenames with spaces safely
- **Task 5**: Each compression tool has different tradeoffs: gzip is fast, xz has best compression, bzip2 is in between
- **Task 6**: `-delete` is the safest way to remove files found by `find`

**Comparison of compression tools** (on the zero-filled file):
- **gzip**: Fast compression, moderate ratio (typically smallest for zero-filled data)
- **bzip2**: Better compression than gzip on real data, slower
- **xz**: Best compression ratio, slowest but most efficient for archival

</details>

### Extensions
1. **Advanced**: Write a bash script that finds all log files older than 30 days, archives them with today's date in the filename, and deletes the originals
2. **Challenge**: Use `find` with `-exec` to calculate the total size of all `.log` files in the directory tree using only one command line

---

## Task 3: File and Process Permissions Management

### Learning Objectives
- Master permission management with `chmod`, `chown`, and `chgrp`
- Understand and configure `umask` to set default permissions
- Apply special permissions: SUID, SGID, and sticky bit

### Context
Your company is deploying a new shared project management application. The development, QA, and operations teams all need access to different parts of the application directory, but with different permission levels. The developers need to read and write code files, QA needs to read and execute test scripts but not modify them, and operations needs full control over deployment scripts. Additionally, you need to set up a shared directory where team members can create files, but only the owner can delete them (similar to `/tmp`).

Permission management is a fundamental LFCS skill and a daily sysadmin task. Incorrect permissions can lead to security vulnerabilities or prevent applications from functioning correctly. Understanding ownership, permissions, and special bits is essential.

### Task Instructions

**Prerequisites**: Set up the test environment
```bash
# Create users and groups
sudo groupadd developers
sudo groupadd qa_team
sudo groupadd ops_team

sudo useradd -m -G developers alice
sudo useradd -m -G qa_team bob
sudo useradd -m -G ops_team charlie
sudo useradd -m -G developers,qa_team david

# Create project structure
sudo mkdir -p /opt/project/{src,tests,deploy,shared}
sudo mkdir -p /opt/project/src/{backend,frontend}

# Create sample files
sudo touch /opt/project/src/backend/app.py
sudo touch /opt/project/src/frontend/index.html
sudo touch /opt/project/tests/test_suite.sh
sudo touch /opt/project/deploy/deploy.sh

echo '#!/bin/bash' | sudo tee /opt/project/tests/test_suite.sh > /dev/null
echo 'echo "Running tests..."' | sudo tee -a /opt/project/tests/test_suite.sh > /dev/null

echo '#!/bin/bash' | sudo tee /opt/project/deploy/deploy.sh > /dev/null
echo 'echo "Deploying application..."' | sudo tee -a /opt/project/deploy/deploy.sh > /dev/null
```

**Your Tasks**:
1. Set ownership of `/opt/project/src/` and all its contents to user `alice` and group `developers`. Developers should be able to read, write, and execute (for directories), while others can only read
2. Configure `/opt/project/tests/` so that it's owned by group `qa_team`. The test scripts should be readable and executable by the group, but not writable. Only the owner (root) should be able to modify them
3. Set up `/opt/project/deploy/` for the `ops_team` group with full read, write, and execute permissions. Set the SGID bit so that new files created in this directory automatically inherit the `ops_team` group
4. Configure the `/opt/project/shared/` directory with the sticky bit so that:
   - Everyone can create files in it (rwx for all)
   - Only the file owner can delete their own files
   - The directory is owned by root with group `developers`
5. Set a umask value that creates new files with permissions `rw-r-----` (640) and new directories with `rwxr-x---` (750) by default. Test this by creating a file and directory
6. Create a file `/opt/project/backup_tool.sh` and set the SUID bit so it always runs with owner privileges (demonstrate understanding, but note security implications)

### Hints and Resources
1. Use `chmod` with symbolic notation (u+x, g+w) or octal notation (755, 644). The `-R` flag applies recursively
2. Remember that `chown user:group` can set both user and group ownership at once
3. SGID on directories is set with `chmod g+s` or `chmod 2755` (octal)
4. Sticky bit is set with `chmod +t` or `chmod 1777` (octal)
5. References: [Ubuntu chmod manual](https://manpages.ubuntu.com/manpages/jammy/man1/chmod.1.html), [File Permissions Guide](https://wiki.archlinux.org/title/File_permissions_and_attributes)

### Estimated Time and Difficulty
**30-40 minutes, Intermediate**

### Verification
- Run `ls -ld /opt/project/src/` and verify alice owns it with group developers and appropriate permissions
- Check `/opt/project/tests/test_suite.sh` is executable by qa_team but not writable
- Verify `/opt/project/deploy/` has the SGID bit set (`ls -ld` should show `s` in group execute position)
- Test the sticky bit by creating files as different users in `/opt/project/shared/` and attempting to delete files owned by others
- Create a test file and directory, then check their permissions match your umask settings
- Verify the SUID bit is set on the backup tool (`ls -l` should show `s` in user execute position)

<details>
<summary>Solution</summary>

```bash
# Task 1: Set ownership and permissions for src directory
sudo chown -R alice:developers /opt/project/src/
sudo chmod -R 754 /opt/project/src/
# Explanation: 754 = rwxr-xr-- (owner: rwx, group: r-x, others: r--)

# Alternative: using symbolic notation
sudo chmod -R u=rwx,g=rx,o=r /opt/project/src/

# Task 2: Configure tests directory for qa_team
sudo chgrp -R qa_team /opt/project/tests/
sudo chmod -R 755 /opt/project/tests/
sudo chmod 755 /opt/project/tests/test_suite.sh
# Make scripts executable but not writable by group:
sudo chmod 554 /opt/project/tests/test_suite.sh
# Explanation: 554 = r-xr-xr-- (owner: r-x, group: r-x, others: r--)

# Task 3: Set up deploy directory with SGID for ops_team
sudo chown -R root:ops_team /opt/project/deploy/
sudo chmod -R 2770 /opt/project/deploy/
# Explanation: 2770 = first 2 sets SGID, 770 = rwxrwx---

# Alternative symbolic notation:
sudo chmod -R g+s,u=rwx,g=rwx,o= /opt/project/deploy/

# Verify SGID:
ls -ld /opt/project/deploy/
# Should show: drwxrws--- ... ops_team

# Task 4: Configure shared directory with sticky bit
sudo chown root:developers /opt/project/shared/
sudo chmod 1777 /opt/project/shared/
# Explanation: 1777 = first 1 sets sticky bit, 777 = rwxrwxrwx

# Alternative symbolic notation:
sudo chmod +t /opt/project/shared/
sudo chmod 777 /opt/project/shared/

# Verify sticky bit:
ls -ld /opt/project/shared/
# Should show: drwxrwxrwt

# Task 5: Set umask for 640 files and 750 directories
# Calculate umask: 666 - 640 = 026 for files (or 777 - 750 = 027 for dirs)
# Use 027 as umask (affects both files and directories)
umask 027

# Test it:
touch ~/test_file.txt
mkdir ~/test_directory

# Verify permissions:
ls -l ~/test_file.txt        # Should show: rw-r----- (640)
ls -ld ~/test_directory      # Should show: rwxr-x--- (750)

# To make it permanent, add to ~/.bashrc:
echo "umask 027" >> ~/.bashrc

# Task 6: Create SUID file
sudo bash -c 'cat > /opt/project/backup_tool.sh << EOF
#!/bin/bash
echo "Running as user: \$(whoami)"
echo "This tool has elevated privileges"
EOF'

sudo chmod 4755 /opt/project/backup_tool.sh
# Explanation: 4755 = first 4 sets SUID, 755 = rwxr-xr-x

# Verify SUID:
ls -l /opt/project/backup_tool.sh
# Should show: rwsr-xr-x (note the 's' in user execute position)
```

**Explanation of Special Permissions**:
- **SUID (Set User ID)**: When set on executable files, the file runs with the permissions of the file owner, not the user executing it. Octal: 4000, Symbolic: u+s. **Security risk** if not carefully managed.
- **SGID (Set Group ID)**: On executables, runs with group permissions. On directories, new files inherit the directory's group. Octal: 2000, Symbolic: g+s.
- **Sticky Bit**: On directories, only file owners can delete their own files, even if others have write permission to the directory. Octal: 1000, Symbolic: +t.

**Permission Calculation**:
- Read (r) = 4, Write (w) = 2, Execute (x) = 1
- 640 = rw-r----- (owner: 4+2=6, group: 4, others: 0)
- 750 = rwxr-x--- (owner: 4+2+1=7, group: 4+1=5, others: 0)
- Umask 027 means: subtract 0 from user, 2 from group, 7 from others
  - Files: 666 - 027 = 640
  - Directories: 777 - 027 = 750

</details>

### Extensions
1. **Advanced**: Create a script that audits all files in `/opt/project/` and reports any files with permissions more permissive than `rw-r--r--` (644)
2. **Challenge**: Set up ACLs (Access Control Lists) using `setfacl` to give user `david` special read-only access to `/opt/project/deploy/` without changing the standard Unix permissions

---

## Task 4: Service Management and Performance Monitoring

### Learning Objectives
- Manage systemd services: create, configure, start, stop, enable, and troubleshoot
- Monitor system performance using `top`, `htop`, `ps`, `systemctl`, and `journalctl`
- Diagnose service-specific constraints and resource issues

### Context
You're managing a fleet of web application servers. The development team has created a new Python web application that needs to run as a background service, automatically starting on boot and restarting if it crashes. However, the application has been consuming excessive CPU and memory during peak hours, causing other services to slow down. Your job is to set up the application as a proper systemd service, monitor its resource usage, identify performance bottlenecks, and implement resource constraints to prevent it from overwhelming the server.

Service management and performance monitoring are core LFCS competencies. In production environments, you need to ensure services run reliably, diagnose issues quickly, and maintain system stability under varying loads.

### Task Instructions

**Prerequisites**: Set up the test application
```bash
# Install required packages
sudo apt update
sudo apt install -y python3 python3-pip htop

# Create a simple Python web application
sudo mkdir -p /opt/webapp
sudo bash -c 'cat > /opt/webapp/app.py << EOF
#!/usr/bin/env python3
import time
import http.server
import socketserver
from datetime import datetime

PORT = 8080

class MyHandler(http.server.SimpleHTTPRequestHandler):
    def do_GET(self):
        # Simulate some CPU work
        total = 0
        for i in range(1000000):
            total += i
        
        self.send_response(200)
        self.send_header("Content-type", "text/html")
        self.end_headers()
        message = f"Hello! Server time: {datetime.now()}, Calculation: {total}"
        self.wfile.write(bytes(message, "utf8"))
        print(f"Request served at {datetime.now()}")

with socketserver.TCPServer(("", PORT), MyHandler) as httpd:
    print(f"Server running on port {PORT}")
    httpd.serve_forever()
EOF'

sudo chmod +x /opt/webapp/app.py

# Create a CPU stress script for testing
sudo bash -c 'cat > /opt/webapp/cpu_stress.py << EOF
#!/usr/bin/env python3
import time
while True:
    total = sum(range(10000000))
    time.sleep(0.1)
EOF'

sudo chmod +x /opt/webapp/cpu_stress.py
```

**Your Tasks**:
1. Create a systemd service unit file at `/etc/systemd/system/webapp.service` that:
   - Runs `/opt/webapp/app.py` as user `www-data`
   - Starts after network is available
   - Automatically restarts on failure
   - Includes a description: "Custom Web Application Service"
2. Enable the service to start on boot and start it immediately
3. Verify the service is running and check its status. View the service logs from the last 20 lines
4. Use `systemctl` to limit the service to use no more than 20% of CPU (CPUQuota) and maximum 100MB of memory (MemoryMax). Reload and restart the service to apply constraints
5. Monitor the service's resource usage using `systemctl status` and `journalctl -f -u webapp.service`
6. Use `ps`, `top`, or `htop` to identify the PID and resource consumption of the webapp process
7. Simulate a problem: stop the service, then check the journal logs for the stop event. Start it again and verify recovery

### Hints and Resources
1. Systemd unit files go in `/etc/systemd/system/`. Use sections like `[Unit]`, `[Service]`, and `[Install]`
2. After creating/modifying service files, run `sudo systemctl daemon-reload`
3. Key service directives: `Type=simple`, `Restart=always`, `User=`, `ExecStart=`
4. Resource limits can be set in the `[Service]` section: `CPUQuota=`, `MemoryMax=`
5. References: [Ubuntu systemd manual](https://manpages.ubuntu.com/manpages/jammy/man1/systemd.1.html), [systemd service units](https://www.freedesktop.org/software/systemd/man/systemd.service.html)

### Estimated Time and Difficulty
**35-45 minutes, Intermediate to Advanced**

### Verification
- `systemctl status webapp.service` should show "active (running)" with green indicator
- `systemctl is-enabled webapp.service` should return "enabled"
- `journalctl -u webapp.service -n 20` should show recent log entries from the application
- After setting resource limits, use `systemctl show webapp.service | grep -E 'CPUQuota|MemoryMax'` to verify they're applied
- `curl http://localhost:8080` should return a response from the web application
- The service should appear in `ps aux | grep app.py` output

<details>
<summary>Solution</summary>

```bash
# Task 1: Create systemd service unit file
sudo bash -c 'cat > /etc/systemd/system/webapp.service << EOF
[Unit]
Description=Custom Web Application Service
After=network.target

[Service]
Type=simple
User=www-data
WorkingDirectory=/opt/webapp
ExecStart=/usr/bin/python3 /opt/webapp/app.py
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
EOF'

# Task 2: Reload systemd, enable and start the service
sudo systemctl daemon-reload
sudo systemctl enable webapp.service
sudo systemctl start webapp.service

# Verify it's enabled:
systemctl is-enabled webapp.service

# Task 3: Check service status and logs
sudo systemctl status webapp.service

# View last 20 lines of logs:
sudo journalctl -u webapp.service -n 20

# Follow logs in real-time:
sudo journalctl -f -u webapp.service

# Task 4: Add resource constraints
# Edit the service file to add resource limits
sudo bash -c 'cat > /etc/systemd/system/webapp.service << EOF
[Unit]
Description=Custom Web Application Service
After=network.target

[Service]
Type=simple
User=www-data
WorkingDirectory=/opt/webapp
ExecStart=/usr/bin/python3 /opt/webapp/app.py
Restart=always
RestartSec=10

# Resource Constraints
CPUQuota=20%
MemoryMax=100M
MemoryHigh=80M

[Install]
WantedBy=multi-user.target
EOF'

# Reload and restart to apply changes:
sudo systemctl daemon-reload
sudo systemctl restart webapp.service

# Verify resource limits are applied:
systemctl show webapp.service | grep -E 'CPUQuota|MemoryMax'

# Task 5: Monitor resource usage
# Check current status with resource info:
systemctl status webapp.service

# Monitor logs in real-time:
sudo journalctl -f -u webapp.service

# Task 6: Find PID and monitor with ps/top
# Get the PID:
systemctl show webapp.service | grep MainPID
# Or:
ps aux | grep app.py | grep -v grep

# Monitor with top (press 'q' to quit):
top -p $(systemctl show webapp.service --property MainPID --value)

# Or use htop (more user-friendly):
sudo htop -p $(systemctl show webapp.service --property MainPID --value)

# Monitor specific process:
ps aux | grep app.py

# Task 7: Simulate problem and check logs
# Stop the service:
sudo systemctl stop webapp.service

# Check status (should show "inactive (dead)"):
systemctl status webapp.service

# Check journal for stop event:
sudo journalctl -u webapp.service -n 30

# Start the service again:
sudo systemctl start webapp.service

# Verify it's running:
systemctl status webapp.service

# Test the web application:
curl http://localhost:8080
```

**Additional Monitoring Commands**:
```bash
# View all failed services:
systemctl --failed

# Check service start time and uptime:
systemctl show webapp.service --property=ActiveEnterTimestamp

# View resource consumption in real-time:
systemd-cgtop

# Check all properties of the service:
systemctl show webapp.service

# View dependency tree:
systemctl list-dependencies webapp.service

# Check if service will start on boot:
systemctl is-enabled webapp.service
```

**Explanation**:
- **[Unit] section**: Defines metadata and dependencies. `After=network.target` ensures network is ready before starting
- **[Service] section**: 
  - `Type=simple`: Service is considered started immediately after `ExecStart` is executed
  - `User=www-data`: Runs as www-data for security (not root)
  - `Restart=always`: Automatically restarts if it crashes
  - `CPUQuota=20%`: Limits to 20% of one CPU core
  - `MemoryMax=100M`: Hard limit at 100MB (service killed if exceeded)
  - `MemoryHigh=80M`: Soft limit at 80MB (throttled if exceeded)
- **[Install] section**: `WantedBy=multi-user.target` enables the service at boot for multi-user runlevel

**Performance Monitoring Tips**:
- `journalctl -u <service>`: View service logs
- `systemctl status <service>`: Quick overview with recent logs
- `systemd-cgtop`: Live view of resource usage by cgroup (systemd units)
- `htop`: Interactive process viewer (better than top)
- `ps aux --sort=-%cpu | head`: Top CPU-consuming processes

</details>

### Extensions
1. **Advanced**: Configure the service to send email notifications (or log to a specific file) when it restarts due to failure
2. **Challenge**: Create a second service that depends on the webapp service and only starts after webapp is fully running. Set up proper dependency ordering using `Wants=` and `After=`

---

## Task 5: Version Control with Git and Shell Scripting Basics

### Learning Objectives
- Perform essential Git operations: clone, commit, push, branch, and merge
- Create and manage shell environment variables, aliases, and functions
- Write practical bash scripts with control structures and error handling

### Context
Your team is transitioning to a DevOps workflow, and you need to set up version control for system configuration files and deployment scripts. You'll create a Git repository to track changes to critical configuration files, set up your shell environment for efficiency, and write automation scripts that the team can use. This includes a script that performs automated backups with git versioning, and another that sets up a standardized development environment for new team members.

Git and shell scripting are fundamental skills for modern system administrators. The LFCS exam tests your ability to use version control for configuration management and write scripts to automate repetitive tasks.

### Task Instructions

**Prerequisites**: Set up Git and the working environment
```bash
# Install git if not already installed
sudo apt update
sudo apt install -y git

# Configure git (use your own name and email)
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
git config --global init.defaultBranch main

# Create a working directory
mkdir -p ~/sysadmin_scripts
cd ~/sysadmin_scripts
```

**Your Tasks**:

**Part A: Git Operations**

1. Initialize a new Git repository in `~/sysadmin_scripts/`
2. Create a README.md file with a project description. Add and commit it with message "Initial commit: Add README"
3. Create a new branch called `development` and switch to it
4. Create a script file `backup.sh` (content provided below) on the development branch. Add and commit it
5. Switch back to the `main` branch and merge the `development` branch into `main`
6. View the commit history with a graphical representation showing branches
7. Create a `.gitignore` file that ignores all files ending in `.log`, `.tmp`, and the directory `temp/`

**Part B: Environment Variables and Aliases**

8. Create a persistent environment variable `BACKUP_DIR=/opt/backups` that survives reboots (add to `~/.bashrc`)
9. Create an alias `ll` for `ls -lah --color=auto` and make it permanent
10. Create a bash function called `mkcd` that creates a directory and immediately changes into it. Make it permanent

**Part C: Shell Scripting**

11. Write a bash script `~/sysadmin_scripts/system_info.sh` that:
    - Displays current date and time
    - Shows system hostname
    - Displays disk usage for root partition
    - Shows memory usage
    - Lists the top 5 CPU-consuming processes
    - Accepts an optional command-line argument `-v` for verbose output (shows additional info)
    - Has proper error handling and exit codes

12. Write a script `~/sysadmin_scripts/user_report.sh` that:
    - Reads user names from a file `users.txt` (one per line)
    - For each user, checks if they exist on the system
    - Outputs a report showing which users exist and which don't
    - Uses a loop and conditional statements

**Script Template for backup.sh** (create and customize):
```bash
#!/bin/bash
# Simple backup script with git versioning
echo "Running backup at $(date)"
# Your implementation here
```

### Hints and Resources
1. Git basics: `git init`, `git add`, `git commit -m "message"`, `git branch`, `git checkout`, `git merge`
2. View history: `git log --oneline --graph --all`
3. Environment variables in `~/.bashrc` are loaded for each new shell session
4. Use `source ~/.bashrc` to reload configuration without restarting the shell
5. References: [Git documentation](https://git-scm.com/doc), [Bash scripting guide](https://www.gnu.org/software/bash/manual/), [Advanced Bash-Scripting Guide](https://tldp.org/LDP/abs/html/)

### Estimated Time and Difficulty
**40-50 minutes, Intermediate to Advanced**

### Verification
- `git status` should show a clean working tree after commits
- `git branch` should show both `main` and `development` branches
- `git log --oneline --graph --all` should show your commit history with branch merge
- After reloading your shell, `echo $BACKUP_DIR` should output `/opt/backups`
- The `ll` alias should work and show detailed file listings
- `mkcd test_directory` should create and enter the directory
- Running your scripts should produce the expected output without errors
- Scripts should be executable (`chmod +x`)

<details>
<summary>Solution</summary>

```bash
# Part A: Git Operations

# Task 1: Initialize Git repository
cd ~/sysadmin_scripts
git init

# Task 2: Create and commit README
cat > README.md << 'EOF'
# System Administration Scripts

This repository contains automation scripts and configuration management tools for Linux system administration.

## Contents
- backup.sh: Automated backup script with git versioning
- system_info.sh: System information gathering tool
- user_report.sh: User account validation script

## Usage
All scripts are documented with inline comments. Run with `./script_name.sh`
EOF

git add README.md
git commit -m "Initial commit: Add README"

# Task 3: Create and switch to development branch
git branch development
git checkout development
# Alternative: git checkout -b development (creates and switches in one command)

# Task 4: Create backup.sh script
cat > backup.sh << 'EOF'
#!/bin/bash
# Simple backup script with git versioning

set -e  # Exit on error

BACKUP_DIR="/tmp/config_backups"
CONFIG_FILES="/etc/hostname /etc/hosts /etc/fstab"

# Create backup directory if it doesn't exist
mkdir -p "$BACKUP_DIR"
cd "$BACKUP_DIR"

# Initialize git repo if needed
if [ ! -d ".git" ]; then
    git init
    echo "Initialized git repository in $BACKUP_DIR"
fi

# Copy configuration files
for file in $CONFIG_FILES; do
    if [ -f "$file" ]; then
        cp "$file" "$BACKUP_DIR/"
        echo "Backed up: $file"
    fi
done

# Commit changes
git add .
git commit -m "Backup at $(date '+%Y-%m-%d %H:%M:%S')" || echo "No changes to commit"

echo "Backup completed at $(date)"
EOF

chmod +x backup.sh
git add backup.sh
git commit -m "Add backup script with git versioning"

# Task 5: Merge development into main
git checkout main
git merge development -m "Merge development branch"

# Task 6: View commit history
git log --oneline --graph --all --decorate

# Task 7: Create .gitignore
cat > .gitignore << 'EOF'
# Ignore log files
*.log

# Ignore temporary files
*.tmp

# Ignore temp directory
temp/
EOF

git add .gitignore
git commit -m "Add .gitignore for logs and temp files"

# Part B: Environment Variables and Aliases

# Task 8: Create persistent environment variable
echo 'export BACKUP_DIR=/opt/backups' >> ~/.bashrc

# Task 9: Create ll alias
echo "alias ll='ls -lah --color=auto'" >> ~/.bashrc

# Task 10: Create mkcd function
cat >> ~/.bashrc << 'EOF'

# Function to create directory and cd into it
mkcd() {
    if [ -z "$1" ]; then
        echo "Usage: mkcd <directory_name>"
        return 1
    fi
    mkdir -p "$1" && cd "$1"
}
EOF

# Reload bashrc to apply changes:
source ~/.bashrc

# Verify:
echo $BACKUP_DIR  # Should output: /opt/backups
ll                # Should show detailed listing
mkcd test_dir     # Should create and enter directory

# Part C: Shell Scripting

# Task 11: Create system_info.sh script
cat > ~/sysadmin_scripts/system_info.sh << 'EOF'
#!/bin/bash
# System Information Report Script

# Color codes for output
GREEN='\033[0;32m'
BLUE='\033[0;34m'
NC='\033[0m' # No Color

# Parse command line arguments
VERBOSE=false
while getopts "v" opt; do
    case $opt in
        v)
            VERBOSE=true
            ;;
        \?)
            echo "Invalid option: -$OPTARG" >&2
            echo "Usage: $0 [-v]"
            exit 1
            ;;
    esac
done

# Function to print section header
print_header() {
    echo -e "${BLUE}=== $1 ===${NC}"
}

# Main script
echo -e "${GREEN}System Information Report${NC}"
echo "Generated: $(date '+%Y-%m-%d %H:%M:%S')"
echo ""

# Date and Time
print_header "Current Date and Time"
date

# Hostname
print_header "System Hostname"
hostname

# Disk Usage
print_header "Disk Usage (Root Partition)"
df -h / | tail -n 1

if [ "$VERBOSE" = true ]; then
    echo ""
    print_header "All Partitions"
    df -h
fi

# Memory Usage
print_header "Memory Usage"
free -h | grep -E 'Mem|Swap'

if [ "$VERBOSE" = true ]; then
    echo ""
    print_header "Detailed Memory Info"
    free -h
fi

# Top CPU Processes
print_header "Top 5 CPU-Consuming Processes"
ps aux --sort=-%cpu | head -n 6

if [ "$VERBOSE" = true ]; then
    echo ""
    print_header "System Uptime"
    uptime
    
    echo ""
    print_header "Load Average"
    cat /proc/loadavg
    
    echo ""
    print_header "CPU Information"
    lscpu | grep -E 'Model name|CPU\(s\)|Thread|Core'
fi

echo ""
echo -e "${GREEN}Report complete!${NC}"
exit 0
EOF

chmod +x ~/sysadmin_scripts/system_info.sh

# Task 12: Create user_report.sh script
cat > ~/sysadmin_scripts/user_report.sh << 'EOF'
#!/bin/bash
# User Account Validation Report

# Check if users.txt exists
USERFILE="${1:-users.txt}"

if [ ! -f "$USERFILE" ]; then
    echo "Error: User file '$USERFILE' not found!"
    echo "Usage: $0 [userfile]"
    echo "Creating sample users.txt file..."
    cat > users.txt << SAMPLE
root
www-data
nobody
nonexistent_user
test_user123
daemon
SAMPLE
    echo "Sample file created. Run the script again."
    exit 1
fi

echo "User Account Validation Report"
echo "==============================="
echo "Generated: $(date)"
echo ""

# Counters
existing_count=0
missing_count=0

# Arrays to store results
existing_users=()
missing_users=()

# Read users from file and check existence
while IFS= read -r username; do
    # Skip empty lines and comments
    [[ -z "$username" || "$username" =~ ^[[:space:]]*# ]] && continue
    
    # Check if user exists
    if id "$username" &>/dev/null; then
        existing_users+=("$username")
        ((existing_count++))
    else
        missing_users+=("$username")
        ((missing_count++))
    fi
done < "$USERFILE"

# Print results
echo "EXISTING USERS ($existing_count):"
echo "----------------------------"
if [ ${#existing_users[@]} -gt 0 ]; then
    for user in "${existing_users[@]}"; do
        uid=$(id -u "$user" 2>/dev/null)
        echo "  ✓ $user (UID: $uid)"
    done
else
    echo "  None found"
fi

echo ""
echo "MISSING USERS ($missing_count):"
echo "----------------------------"
if [ ${#missing_users[@]} -gt 0 ]; then
    for user in "${missing_users[@]}"; do
        echo "  ✗ $user"
    done
else
    echo "  None"
fi

echo ""
echo "SUMMARY:"
echo "--------"
echo "Total users checked: $((existing_count + missing_count))"
echo "Existing: $existing_count"
echo "Missing: $missing_count"

# Exit with appropriate code
if [ $missing_count -gt 0 ]; then
    exit 1
else
    exit 0
fi
EOF

chmod +x ~/sysadmin_scripts/user_report.sh

# Create sample users.txt for testing
cat > ~/sysadmin_scripts/users.txt << 'EOF'
root
www-data
nobody
nonexistent_user
daemon
bin
sys
EOF

# Test the scripts
echo "Testing system_info.sh:"
~/sysadmin_scripts/system_info.sh

echo -e "\n\nTesting system_info.sh with verbose flag:"
~/sysadmin_scripts/system_info.sh -v

echo -e "\n\nTesting user_report.sh:"
cd ~/sysadmin_scripts
./user_report.sh users.txt

# Commit all new scripts
git add system_info.sh user_report.sh users.txt
git commit -m "Add system info and user report scripts"

# Final git status
git status
git log --oneline --graph --all
```

**Additional Git Commands Reference**:
```bash
# View changes before committing:
git diff

# View staged changes:
git diff --staged

# Undo last commit (keep changes):
git reset --soft HEAD~1

# Undo last commit (discard changes):
git reset --hard HEAD~1

# View file changes history:
git log -p filename

# Create and push to remote (if you have a remote repo):
git remote add origin <url>
git push -u origin main

# Pull latest changes:
git pull origin main

# View all branches:
git branch -a

# Delete a branch:
git branch -d development
```

**Shell Scripting Best Practices**:
1. Always use `#!/bin/bash` shebang
2. Use `set -e` to exit on error (or handle errors explicitly)
3. Quote variables: `"$variable"` to prevent word splitting
4. Use meaningful variable names in UPPERCASE for globals
5. Add comments explaining complex logic
6. Validate input and provide usage information
7. Use exit codes: 0 for success, non-zero for errors
8. Make scripts executable: `chmod +x script.sh`

</details>

### Extensions
1. **Advanced**: Set up a remote Git repository (GitHub/GitLab) and push your local repository to it. Configure SSH keys for authentication
2. **Challenge**: Write a script that automatically commits changes to configuration files in `/etc/` if they've been modified, creating a git-based configuration versioning system. Include proper error handling and logging

---

## Task 6: Working with SSL/TLS Certificates

### Learning Objectives
- Generate and manage SSL/TLS certificates using OpenSSL
- Understand certificate formats (PEM, DER, CRT, KEY) and convert between them
- Configure web servers (Apache/Nginx) with SSL certificates
- Troubleshoot certificate expiration, validation, and chain issues

### Context
Your company is expanding its web infrastructure and needs to secure multiple web applications with HTTPS. You're responsible for generating SSL certificates for development and staging environments, configuring web servers to use them, and troubleshooting certificate-related issues. The security team has reported that one of the production servers has an expired certificate causing client connection failures, and you need to quickly diagnose and resolve the issue.

SSL/TLS certificate management is a critical skill for the LFCS exam and modern system administration. Improperly configured certificates can cause security vulnerabilities, service outages, and compliance issues. Understanding how to generate, inspect, and troubleshoot certificates is essential.

### Task Instructions

**Prerequisites**: Set up the test environment
```bash
# Install required packages
sudo apt update
sudo apt install -y openssl apache2 curl

# Create working directory
mkdir -p ~/ssl_lab/{certs,private,csr}
cd ~/ssl_lab
```

**Your Tasks**:

**Part A: Certificate Generation**

1. Generate a self-signed SSL certificate for domain `test.local`:
   - Create a 2048-bit RSA private key
   - Generate a Certificate Signing Request (CSR) with the following details:
     - Country: US
     - State: California
     - City: San Francisco
     - Organization: Test Company
     - Common Name: test.local
   - Create a self-signed certificate valid for 365 days
   - Store the private key in `~/ssl_lab/private/test.local.key` and certificate in `~/ssl_lab/certs/test.local.crt`

2. Create a certificate with Subject Alternative Names (SANs) for multiple domains: `webapp.local`, `api.webapp.local`, `www.webapp.local`

**Part B: Certificate Inspection and Conversion**

3. Display the certificate details for `test.local.crt` including:
   - Issuer and subject information
   - Validity dates (not before, not after)
   - Serial number
   - Public key algorithm and size

4. Verify the certificate matches its private key by comparing their modulus

5. Convert the certificate from PEM format to DER format and back

**Part C: Web Server Configuration**

6. Configure Apache to use your SSL certificate:
   - Enable SSL module
   - Create a virtual host on port 443 for `test.local`
   - Configure it to use your certificate and private key
   - Set up HTTP to HTTPS redirect

7. Test the SSL configuration and verify the certificate is being served correctly

**Part D: Certificate Troubleshooting**

8. Check certificate expiration dates and create a script that warns if a certificate expires within 30 days

9. Download and inspect a real website's certificate chain (e.g., from `www.google.com`)

10. Simulate and troubleshoot a certificate mismatch error

### Hints and Resources
1. `openssl genrsa` generates RSA private keys. Use `-aes256` for password protection
2. `openssl req` creates certificate signing requests. Use `-new` for new CSR
3. `openssl x509` works with certificates. Use `-text` to view details, `-noout` to suppress encoded output
4. For Apache, configuration files are in `/etc/apache2/sites-available/`
5. References: [OpenSSL Cookbook](https://www.feistyduck.com/library/openssl-cookbook/), [Ubuntu OpenSSL manual](https://manpages.ubuntu.com/manpages/jammy/man1/openssl.1ssl.html)

### Estimated Time and Difficulty
**35-50 minutes, Intermediate to Advanced**

### Verification
- Private key files should have 600 permissions for security
- `openssl x509 -in cert.crt -text -noout` should display certificate details
- Apache should restart without errors after SSL configuration
- `curl -k https://test.local` should return a response (with self-signed warning)
- Your expiration check script should correctly identify certificates expiring soon
- Certificate and key modulus should match when compared

<details>
<summary>Solution</summary>

```bash
# Part A: Certificate Generation

# Task 1: Generate self-signed certificate for test.local

# Generate private key (2048-bit RSA)
openssl genrsa -out ~/ssl_lab/private/test.local.key 2048

# Set secure permissions on private key
chmod 600 ~/ssl_lab/private/test.local.key

# Generate Certificate Signing Request (CSR)
openssl req -new -key ~/ssl_lab/private/test.local.key \
    -out ~/ssl_lab/csr/test.local.csr \
    -subj "/C=US/ST=California/L=San Francisco/O=Test Company/CN=test.local"

# Generate self-signed certificate (valid 365 days)
openssl x509 -req -days 365 \
    -in ~/ssl_lab/csr/test.local.csr \
    -signkey ~/ssl_lab/private/test.local.key \
    -out ~/ssl_lab/certs/test.local.crt

# Alternative: Generate private key and self-signed cert in one command
openssl req -x509 -newkey rsa:2048 -nodes \
    -keyout ~/ssl_lab/private/test.local.key \
    -out ~/ssl_lab/certs/test.local.crt \
    -days 365 \
    -subj "/C=US/ST=California/L=San Francisco/O=Test Company/CN=test.local"

# Task 2: Create certificate with Subject Alternative Names (SANs)

# Create OpenSSL configuration file for SAN
cat > ~/ssl_lab/san.cnf << 'EOF'
[req]
default_bits = 2048
prompt = no
default_md = sha256
distinguished_name = dn
req_extensions = v3_req

[dn]
C = US
ST = California
L = San Francisco
O = Test Company
CN = webapp.local

[v3_req]
subjectAltName = @alt_names

[alt_names]
DNS.1 = webapp.local
DNS.2 = api.webapp.local
DNS.3 = www.webapp.local
EOF

# Generate key and CSR with SAN
openssl req -new -newkey rsa:2048 -nodes \
    -keyout ~/ssl_lab/private/webapp.local.key \
    -out ~/ssl_lab/csr/webapp.local.csr \
    -config ~/ssl_lab/san.cnf

# Self-sign the certificate with SAN extensions
openssl x509 -req -days 365 \
    -in ~/ssl_lab/csr/webapp.local.csr \
    -signkey ~/ssl_lab/private/webapp.local.key \
    -out ~/ssl_lab/certs/webapp.local.crt \
    -extensions v3_req \
    -extfile ~/ssl_lab/san.cnf

chmod 600 ~/ssl_lab/private/webapp.local.key

# Part B: Certificate Inspection and Conversion

# Task 3: Display certificate details
openssl x509 -in ~/ssl_lab/certs/test.local.crt -text -noout

# Display specific information:
echo "=== Subject Information ==="
openssl x509 -in ~/ssl_lab/certs/test.local.crt -noout -subject

echo "=== Issuer Information ==="
openssl x509 -in ~/ssl_lab/certs/test.local.crt -noout -issuer

echo "=== Validity Dates ==="
openssl x509 -in ~/ssl_lab/certs/test.local.crt -noout -dates

echo "=== Serial Number ==="
openssl x509 -in ~/ssl_lab/certs/test.local.crt -noout -serial

echo "=== Fingerprint ==="
openssl x509 -in ~/ssl_lab/certs/test.local.crt -noout -fingerprint

# Task 4: Verify certificate matches private key
# Compare modulus of certificate and private key
cert_modulus=$(openssl x509 -noout -modulus -in ~/ssl_lab/certs/test.local.crt | openssl md5)
key_modulus=$(openssl rsa -noout -modulus -in ~/ssl_lab/private/test.local.key | openssl md5)

echo "Certificate modulus: $cert_modulus"
echo "Private key modulus: $key_modulus"

if [ "$cert_modulus" = "$key_modulus" ]; then
    echo "✓ Certificate and private key match!"
else
    echo "✗ Certificate and private key DO NOT match!"
fi

# Task 5: Convert certificate formats
# PEM to DER
openssl x509 -in ~/ssl_lab/certs/test.local.crt \
    -outform DER \
    -out ~/ssl_lab/certs/test.local.der

# DER to PEM
openssl x509 -in ~/ssl_lab/certs/test.local.der \
    -inform DER \
    -out ~/ssl_lab/certs/test.local.pem \
    -outform PEM

# Verify conversion
ls -lh ~/ssl_lab/certs/

# Part C: Web Server Configuration

# Task 6: Configure Apache with SSL

# Enable SSL module and site
sudo a2enmod ssl
sudo a2enmod rewrite

# Create SSL virtual host configuration
sudo bash -c 'cat > /etc/apache2/sites-available/test-ssl.conf << EOF
<VirtualHost *:80>
    ServerName test.local
    
    # Redirect all HTTP to HTTPS
    RewriteEngine On
    RewriteCond %{HTTPS} off
    RewriteRule ^(.*)$ https://%{HTTP_HOST}$1 [R=301,L]
</VirtualHost>

<VirtualHost *:443>
    ServerName test.local
    DocumentRoot /var/www/test.local
    
    # SSL Configuration
    SSLEngine on
    SSLCertificateFile /home/'$USER'/ssl_lab/certs/test.local.crt
    SSLCertificateKeyFile /home/'$USER'/ssl_lab/private/test.local.key
    
    # Modern SSL configuration
    SSLProtocol all -SSLv3 -TLSv1 -TLSv1.1
    SSLCipherSuite HIGH:!aNULL:!MD5
    
    <Directory /var/www/test.local>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
    
    ErrorLog \${APACHE_LOG_DIR}/test-ssl-error.log
    CustomLog \${APACHE_LOG_DIR}/test-ssl-access.log combined
</VirtualHost>
EOF'

# Create document root and test page
sudo mkdir -p /var/www/test.local
sudo bash -c 'cat > /var/www/test.local/index.html << EOF
<!DOCTYPE html>
<html>
<head><title>SSL Test Page</title></head>
<body>
    <h1>SSL is working!</h1>
    <p>This page is served over HTTPS</p>
</body>
</html>
EOF'

# Enable the site
sudo a2ensite test-ssl.conf

# Test Apache configuration
sudo apache2ctl configtest

# Reload Apache
sudo systemctl reload apache2

# Add test.local to /etc/hosts
echo "127.0.0.1 test.local" | sudo tee -a /etc/hosts

# Task 7: Test SSL configuration
# Test with curl (accept self-signed cert)
curl -k https://test.local

# Check certificate being served
echo | openssl s_client -connect test.local:443 -servername test.local 2>/dev/null | openssl x509 -noout -dates

# Verify SSL configuration
openssl s_client -connect test.local:443 -servername test.local < /dev/null

# Part D: Certificate Troubleshooting

# Task 8: Certificate expiration check script
cat > ~/ssl_lab/check_cert_expiry.sh << 'EOF'
#!/bin/bash
# Certificate expiration checker

CERT_FILE="${1}"
WARN_DAYS=30

if [ -z "$CERT_FILE" ]; then
    echo "Usage: $0 <certificate_file>"
    exit 1
fi

if [ ! -f "$CERT_FILE" ]; then
    echo "Error: Certificate file not found: $CERT_FILE"
    exit 1
fi

# Get expiration date
expiry_date=$(openssl x509 -in "$CERT_FILE" -noout -enddate | cut -d= -f2)
expiry_epoch=$(date -d "$expiry_date" +%s)
current_epoch=$(date +%s)
days_until_expiry=$(( ($expiry_epoch - $current_epoch) / 86400 ))

echo "Certificate: $CERT_FILE"
echo "Expires on: $expiry_date"
echo "Days until expiry: $days_until_expiry"

if [ $days_until_expiry -lt 0 ]; then
    echo "⚠️  CRITICAL: Certificate has EXPIRED!"
    exit 2
elif [ $days_until_expiry -lt $WARN_DAYS ]; then
    echo "⚠️  WARNING: Certificate expires in less than $WARN_DAYS days!"
    exit 1
else
    echo "✓ Certificate is valid"
    exit 0
fi
EOF

chmod +x ~/ssl_lab/check_cert_expiry.sh

# Test the script
~/ssl_lab/check_cert_expiry.sh ~/ssl_lab/certs/test.local.crt

# Task 9: Download and inspect remote certificate
# Download Google's certificate chain
echo | openssl s_client -connect www.google.com:443 -servername www.google.com 2>/dev/null | \
    openssl x509 -out ~/ssl_lab/certs/google.crt

# View the certificate
openssl x509 -in ~/ssl_lab/certs/google.crt -text -noout | head -30

# Check certificate chain
openssl s_client -connect www.google.com:443 -servername www.google.com -showcerts < /dev/null 2>/dev/null | \
    grep -E 'subject=|issuer='

# Task 10: Simulate certificate mismatch
# Create a certificate for wrong domain
openssl req -x509 -newkey rsa:2048 -nodes \
    -keyout ~/ssl_lab/private/wrong.key \
    -out ~/ssl_lab/certs/wrong.crt \
    -days 365 \
    -subj "/C=US/ST=California/L=San Francisco/O=Test Company/CN=wrongdomain.local"

# Try to use it with test.local (will cause mismatch)
sudo cp ~/ssl_lab/certs/wrong.crt ~/ssl_lab/certs/test.local.crt.bak
sudo cp ~/ssl_lab/certs/test.local.crt ~/ssl_lab/certs/test.local.crt.orig

# This would cause a hostname mismatch error when accessed
echo "Certificate mismatch can be tested by replacing the cert with wrong.crt"
echo "The error would be: 'certificate subject name (wrongdomain.local) does not match target host name (test.local)'"
```

**Additional Certificate Management Commands**:
```bash
# Check if certificate is valid for a specific domain
openssl verify -CAfile ca-bundle.crt certificate.crt

# Extract all certificates from a bundle
openssl crl2pkcs7 -nocrl -certfile bundle.crt | \
    openssl pkcs7 -print_certs -out certificates.txt

# Create a certificate bundle (concatenate chain)
cat domain.crt intermediate.crt root.crt > bundle.crt

# Generate a strong Diffie-Hellman parameters file (for better SSL security)
openssl dhparam -out /etc/ssl/certs/dhparam.pem 2048

# Test SSL/TLS connection with specific protocol
openssl s_client -connect example.com:443 -tls1_2

# Check certificate transparency logs
# (Note: This requires external tools like ct-submit)

# Benchmark SSL performance
openssl speed rsa2048
```

**Common Certificate Issues and Solutions**:

1. **"Certificate has expired"**
   - Check: `openssl x509 -in cert.crt -noout -dates`
   - Solution: Renew the certificate

2. **"Certificate name mismatch"**
   - Check: `openssl x509 -in cert.crt -noout -text | grep -A1 "Subject:"`
   - Solution: Ensure CN or SAN matches the server hostname

3. **"Unable to verify certificate chain"**
   - Check: `openssl verify -CAfile ca.crt cert.crt`
   - Solution: Install intermediate certificates

4. **"Private key does not match certificate"**
   - Check: Compare modulus as shown in Task 4
   - Solution: Use the correct private key that was used to generate the CSR

**Certificate File Formats**:
- **PEM**: Base64 encoded, ASCII text, most common (begins with `-----BEGIN CERTIFICATE-----`)
- **DER**: Binary format
- **CRT/CER**: Can be either PEM or DER (usually PEM on Linux)
- **KEY**: Private key file (PEM or DER)
- **CSR**: Certificate Signing Request
- **PFX/P12**: PKCS#12 format, contains certificate and private key (Windows)

</details>

### Extensions
1. **Advanced**: Set up a local Certificate Authority (CA), generate intermediate certificates, and sign server certificates with your CA chain
2. **Challenge**: Configure mutual TLS (mTLS) authentication where both server and client present certificates. Create client certificates and configure Apache to require them

---

## Task 7: Advanced Text Editing with vim/vi

### Learning Objectives
- Master vim navigation, editing modes, and essential commands
- Perform advanced text manipulation: search/replace, macros, visual mode
- Edit multiple files efficiently and customize vim configuration
- Use vim for system administration tasks (editing config files, scripts)

### Context
As a system administrator, you frequently need to edit configuration files on remote servers where graphical editors aren't available. Vim/vi is universally available on Linux systems and mastering it significantly improves your productivity. You need to quickly edit systemd service files, modify configuration files with precise search-and-replace operations, and make bulk changes across multiple files. Understanding vim's powerful features can turn hours of tedious editing into minutes of efficient work.

Vim proficiency is tested in the LFCS exam and is an essential skill for any Linux administrator. Being able to navigate, edit, and save files quickly in vim can make the difference between meeting and missing critical deadlines.

### Task Instructions

**Prerequisites**: Set up practice files
```bash
# Create practice directory
mkdir -p ~/vim_practice/{configs,scripts,logs}
cd ~/vim_practice

# Create sample configuration file
cat > configs/app.conf << 'EOF'
# Application Configuration
server_name = production_server
port = 8080
max_connections = 100
timeout = 30
enable_ssl = false
log_level = info
database_host = localhost
database_port = 5432
cache_enabled = true
cache_size = 512
EOF

# Create sample script with errors
cat > scripts/backup.sh << 'EOF'
#!/bin/bash
# Backup script with intentional errors

BACKUP_DIR=/var/backups
SOURCE_DIR=/home/user/data

# Create backup
mkdir -p $BACKUP_DIR
cp -r $SOURCE_DIR $BACKUP_DIR/backup_$(date +%Y%m%d)

# Cleanup old backups (older than 7 days)
find $BACKUP_DIR -name "backup_*" -mtime +7 -delete

echo "Backup completed at $(date)"
EOF

# Create sample log file
cat > logs/application.log << 'EOF'
2025-12-29 10:00:01 INFO Starting application
2025-12-29 10:00:02 INFO Loading configuration
2025-12-29 10:00:05 ERROR Failed to connect to database
2025-12-29 10:00:06 INFO Retrying connection
2025-12-29 10:00:10 INFO Connected successfully
2025-12-29 10:01:00 WARNING High memory usage detected
2025-12-29 10:02:15 ERROR Connection timeout
2025-12-29 10:02:20 INFO Reconnected
2025-12-29 10:05:00 INFO Processing request from 192.168.1.10
2025-12-29 10:05:01 INFO Processing request from 192.168.1.20
EOF

# Create a file with repeated patterns
cat > configs/hosts.txt << 'EOF'
192.168.1.10 server1
192.168.1.20 server2
192.168.1.30 server3
10.0.0.5 database1
10.0.0.6 database2
172.16.0.1 router1
EOF
```

**Your Tasks**:

**Part A: Basic Navigation and Editing**

1. Open `configs/app.conf` in vim and:
   - Navigate to line 5 using line number command
   - Change `enable_ssl = false` to `enable_ssl = true`
   - Jump to the end of the file
   - Add a new line: `backup_enabled = true`
   - Save and quit

**Part B: Search and Replace**

2. In `configs/app.conf`:
   - Search for the word "localhost" 
   - Replace ALL occurrences of "localhost" with "db.production.local"
   - Change port 8080 to 8443 (only this occurrence)
   - Verify changes and save

**Part C: Advanced Editing**

3. In `logs/application.log`:
   - Delete all lines containing "INFO"
   - Copy all lines containing "ERROR" to a new section at the end of the file with header "=== Error Summary ==="
   - Number all lines in the file
   - Save as `logs/errors_filtered.log`

**Part D: Visual Mode and Blocks**

4. In `configs/hosts.txt`:
   - Use visual block mode to comment out all lines (add # at the beginning)
   - Select the IP address column and copy it
   - Paste it at the end of each line
   - Undo the changes, then redo them

**Part E: Multiple Files and Buffers**

5. Open all three configuration files (`app.conf`, `backup.sh`, `hosts.txt`) in vim simultaneously:
   - Navigate between buffers
   - In each file, add a comment header with the filename and current date
   - Save all files and quit

**Part F: Macros and Automation**

6. In `configs/hosts.txt`:
   - Record a macro that converts each line to an `/etc/hosts` entry format
   - Apply the macro to all lines
   - Save the result

**Part G: Vim Configuration**

7. Create a `~/.vimrc` file with useful settings:
   - Enable line numbers
   - Set tab size to 4 spaces
   - Enable syntax highlighting
   - Show matching brackets
   - Enable search highlighting

### Hints and Resources
1. **Modes**: Normal mode (ESC), Insert mode (i), Visual mode (v), Command mode (:)
2. **Navigation**: h/j/k/l (left/down/up/right), gg (start), G (end), :n (line n)
3. **Editing**: x (delete char), dd (delete line), yy (copy line), p (paste)
4. **Search**: /pattern (search forward), ?pattern (search backward), n (next match)
5. References: [Vim documentation](https://www.vim.org/docs.php), [Interactive Vim tutorial](https://www.openvim.com/)

### Estimated Time and Difficulty
**30-40 minutes, Intermediate**

### Verification
- Files should be modified as specified without corruption
- Search/replace operations should affect only intended text
- Macros should execute correctly on multiple lines
- `.vimrc` should be loaded when vim starts (check with `:set number?`)
- You should be able to navigate between buffers smoothly
- All files should save without errors

<details>
<summary>Solution</summary>

```bash
# Part A: Basic Navigation and Editing

# Task 1: Open and edit app.conf
vim configs/app.conf

# In vim (commands to type):
# :5              - Go to line 5
# /enable_ssl     - Search for enable_ssl
# cw              - Change word (while cursor on "false")
# true<ESC>       - Type "true" and exit insert mode
# G               - Jump to end of file
# o               - Open new line below
# backup_enabled = true<ESC>  - Type the new setting
# :wq             - Save and quit

# Alternative using sed (for verification):
sed -i 's/enable_ssl = false/enable_ssl = true/' configs/app.conf
echo "backup_enabled = true" >> configs/app.conf

# Part B: Search and Replace

# Task 2: Search and replace in app.conf
vim configs/app.conf

# In vim:
# /localhost      - Search for localhost (press n to find next)
# :%s/localhost/db.production.local/g  - Replace all localhost
# /8080           - Find 8080
# cw8443<ESC>     - Change to 8443
# :wq             - Save and quit

# Alternative command mode approach (all at once):
# vim -c '%s/localhost/db.production.local/g' -c '%s/port = 8080/port = 8443/' -c 'wq' configs/app.conf

# Or using ex mode:
ex -sc '%s/localhost/db.production.local/g|%s/port = 8080/port = 8443/|x' configs/app.conf

# Part C: Advanced Editing

# Task 3: Filter and manipulate logs
vim logs/application.log

# In vim:
# :g/INFO/d       - Delete all lines containing INFO
# /ERROR          - Find first ERROR
# :.,/ERROR/y     - Yank (copy) from current to next ERROR
# G               - Go to end
# o<ESC>          - New line
# i=== Error Summary ===<ESC>  - Add header
# :g/ERROR/t$     - Copy all ERROR lines to end of file
# :%s/^/\=line('.') . '. '/  - Number all lines (or use :set number)
# :w logs/errors_filtered.log  - Save as new file
# :q              - Quit

# Alternative approach for line numbers:
# :%s/^/\=printf('%3d. ', line('.'))

# Part D: Visual Mode and Blocks

# Task 4: Block editing in hosts.txt
vim configs/hosts.txt

# In vim:
# gg              - Go to start
# Ctrl+v          - Enter visual block mode
# 5j              - Select 6 lines (down 5)
# I               - Insert at beginning of block
# # <ESC>         - Type # and space, then ESC (applies to all lines)
# 
# To copy IP column:
# gg              - Back to start
# Ctrl+v          - Visual block mode
# 5j              - Down 5 lines
# w               - Move to end of IP address
# y               - Yank (copy)
# $               - Move to end of line
# p               - Paste
#
# u               - Undo
# Ctrl+r          - Redo
# :wq             - Save and quit

# Part E: Multiple Files and Buffers

# Task 5: Edit multiple files
vim configs/app.conf scripts/backup.sh configs/hosts.txt

# In vim:
# Add header to first file (app.conf):
# gg              - Go to top
# O               - Open line above
# # File: app.conf<CR># Date: 2025-12-29<ESC>
#
# :bn             - Next buffer (backup.sh)
# gg
# O
# # File: backup.sh<CR># Date: 2025-12-29<ESC>
#
# :bn             - Next buffer (hosts.txt)
# gg
# O
# # File: hosts.txt<CR># Date: 2025-12-29<ESC>
#
# :wa             - Write all
# :qa             - Quit all

# List buffers:
# :ls             - Show all buffers
# :b2             - Switch to buffer 2
# :bd             - Delete current buffer

# Part F: Macros and Automation

# Task 6: Record and use macro
vim configs/hosts.txt

# In vim:
# qq              - Start recording macro in register 'q'
# 0               - Go to start of line
# i127.0.0.1 <ESC> - Insert localhost IP
# $               - End of line
# a # local<ESC>  - Append comment
# j               - Move down
# q               - Stop recording
#
# @q              - Run macro once
# 5@q             - Run macro 5 more times (for remaining lines)
# OR
# :%normal @q     - Apply macro to all lines
# :wq

# Alternative: Format for /etc/hosts entry
# qq              - Start recording
# I127.0.0.1 <ESC>  - Add IP
# A # Added<ESC>    - Add comment
# j               - Next line
# q               - Stop
# 5@q             - Repeat 5 times

# Part G: Vim Configuration

# Task 7: Create .vimrc
cat > ~/.vimrc << 'EOF'
" Enable line numbers
set number
set relativenumber  " Relative line numbers (optional)

" Tab and indentation settings
set tabstop=4       " Tab width
set shiftwidth=4    " Indent width
set expandtab       " Use spaces instead of tabs
set autoindent      " Auto-indent new lines
set smartindent     " Smart indentation

" Search settings
set hlsearch        " Highlight search results
set incsearch       " Incremental search
set ignorecase      " Case-insensitive search
set smartcase       " Case-sensitive if uppercase used

" Display settings
syntax on           " Enable syntax highlighting
set showmatch       " Show matching brackets
set ruler           " Show cursor position
set showcmd         " Show command in status line
set wildmenu        " Enhanced command completion
set cursorline      " Highlight current line

" Behavior settings
set backspace=indent,eol,start  " Better backspace behavior
set mouse=a         " Enable mouse support
set clipboard=unnamedplus  " Use system clipboard

" Performance
set lazyredraw      " Don't redraw during macros

" File handling
set noswapfile      " Disable swap files (optional)
set nobackup        " Disable backups (optional)
set autoread        " Auto-reload changed files

" Status line
set laststatus=2    " Always show status line
set statusline=%F\ %m%r%h%w\ [%l/%L,%c]\ [%p%%]

" Color scheme (if available)
" colorscheme desert

" Key mappings
" Map jj to ESC in insert mode
inoremap jj <ESC>

" Map leader key to space
let mapleader = " "

" Quick save with leader+w
nnoremap <leader>w :w<CR>

" Quick quit with leader+q
nnoremap <leader>q :q<CR>
EOF

# Test the configuration
vim ~/.vimrc
# Check if settings are applied:
# :set number?
# :set tabstop?
```

**Essential Vim Commands Reference**:

**Navigation:**
```
h, j, k, l       - Left, down, up, right
w, b             - Next/previous word
0, $             - Start/end of line
gg, G            - Start/end of file
Ctrl+f, Ctrl+b   - Page down/up
:n or nG         - Go to line n
%                - Jump to matching bracket
```

**Editing:**
```
i, a             - Insert before/after cursor
I, A             - Insert at start/end of line
o, O             - Open line below/above
x, X             - Delete char forward/backward
dd               - Delete line
D                - Delete to end of line
yy               - Yank (copy) line
p, P             - Paste after/before
u, Ctrl+r        - Undo/redo
.                - Repeat last command
~                - Toggle case
```

**Search and Replace:**
```
/pattern         - Search forward
?pattern         - Search backward
n, N             - Next/previous match
*                - Search word under cursor
:s/old/new/      - Replace in current line
:%s/old/new/g    - Replace in all file
:%s/old/new/gc   - Replace with confirmation
:g/pattern/d     - Delete lines matching pattern
:v/pattern/d     - Delete lines NOT matching
```

**Visual Mode:**
```
v                - Visual mode (character)
V                - Visual line mode
Ctrl+v           - Visual block mode
gv               - Reselect last selection
```

**File Operations:**
```
:w               - Save
:w filename      - Save as
:q               - Quit
:wq or :x        - Save and quit
:q!              - Quit without saving
:e filename      - Edit file
:bn, :bp         - Next/previous buffer
:ls              - List buffers
:bd              - Delete buffer
```

**Advanced:**
```
qa...q           - Record macro in register a
@a               - Run macro a
@@               - Repeat last macro
:set option      - Set option
:set option?     - Check option value
:syntax on       - Enable syntax
:! command       - Run shell command
:r !command      - Insert command output
```

</details>

### Extensions
1. **Advanced**: Create a vim plugin using vimscript that automatically adds file headers with metadata (author, date, description) when creating new files
2. **Challenge**: Set up vim as an IDE for Python development with plugins (NERDTree, airline, syntastic). Configure code folding, auto-completion, and linting

---

## Summary and Next Steps

You've completed comprehensive exercises covering essential Linux commands for the LFCS exam:

- **Task 1**: Log analysis and text processing with grep, awk, sed, and pipelines
- **Task 2**: Disk space management and file operations
- **Task 3**: File and process permissions management
- **Task 4**: Service management and performance monitoring with systemd
- **Task 5**: Version control with Git and shell scripting basics
- **Task 6**: Working with SSL/TLS certificates and OpenSSL
- **Task 7**: Advanced text editing with vim/vi

**Key Takeaways**:
- Mastering text processing tools (grep, awk, sed) is crucial for daily system administration
- Understanding file permissions and ownership prevents security vulnerabilities
- Systemd is the standard init system for modern Linux distributions
- Shell scripting automates repetitive tasks and improves efficiency
- SSL/TLS certificates are essential for securing modern applications
- Proficiency in vim/vi is required for efficient command-line editing

**Recommended Practice**:
1. Practice text processing daily on real system logs
2. Create automation scripts for common administrative tasks
3. Set up a personal Git repository for configuration management
4. Build a complete systemd service from scratch
5. Practice vim keybindings until they become muscle memory

**Additional Resources**:
- [LFCS Exam Domains](https://training.linuxfoundation.org/certification/linux-foundation-certified-sysadmin-lfcs/)
- [GNU Coreutils Manual](https://www.gnu.org/software/coreutils/manual/)
- [Systemd Documentation](https://www.freedesktop.org/software/systemd/man/)
- [Advanced Bash-Scripting Guide](https://tldp.org/LDP/abs/html/)
- [Vim User Manual](https://vimhelp.org/)

Continue practicing these essential commands in real Ubuntu environments to build confidence for the LFCS exam!
