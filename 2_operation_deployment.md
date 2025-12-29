# LFCS Practice: Operation of Running Systems

This guide provides hands-on exercises designed to prepare you for the Linux Foundation Certified System Administrator (LFCS) exam, focusing on operating and deploying running systems. All tasks are designed for **Ubuntu 22.04 LTS** and simulate real-world system administration scenarios.

> **Note**: This document follows the formatting standards defined in [FORMATTING_GUIDE.md](FORMATTING_GUIDE.md). Refer to that guide when creating new exercises.

---

## Task 1: Kernel Parameter Configuration and Tuning

### Learning Objectives
- Configure kernel parameters using `sysctl` for both runtime and persistent changes
- Understand the difference between temporary and permanent kernel parameter modifications
- Optimize system performance by tuning network, memory, and filesystem parameters

### Context
You're managing a high-traffic web application server that's experiencing performance issues during peak hours. The development team reports that the server is running out of file descriptors, network connections are being dropped, and there are occasional out-of-memory errors. Your job is to tune kernel parameters to improve system performance and stability. You need to understand which parameters to modify temporarily for testing and which ones to make permanent across reboots.

Kernel parameter tuning is a critical LFCS skill because every production Linux system requires optimization based on its workload. Understanding `sysctl` and `/etc/sysctl.conf` allows you to fine-tune system behavior for databases, web servers, network appliances, and other specialized workloads without recompiling the kernel.

### Task Instructions

**Prerequisites**: Set up a test environment
```bash
# Update system
sudo apt update

# Install utilities for monitoring
sudo apt install -y sysstat net-tools

# Create a working directory
mkdir -p ~/kernel_tuning
cd ~/kernel_tuning
```

**Your Tasks**:
1. Display all current kernel parameters and save them to `~/kernel_tuning/original_params.txt` for backup
2. View and document the current values of these specific parameters:
   - `fs.file-max` (maximum number of file handles)
   - `net.ipv4.ip_local_port_range` (range of ports for outbound connections)
   - `vm.swappiness` (how aggressively the kernel swaps memory to disk)
   - `net.core.somaxconn` (maximum number of queued connections)
3. Make the following **temporary** (non-persistent) changes and verify them:
   - Increase `fs.file-max` to 2097152
   - Set `vm.swappiness` to 10 (reduce swapping)
   - Increase `net.core.somaxconn` to 4096
4. Make the following **permanent** (persistent across reboots) changes:
   - Set `fs.file-max = 2097152`
   - Set `vm.swappiness = 10`
   - Set `net.ipv4.ip_local_port_range = 10000 65000`
   - Add a custom parameter: `net.core.netdev_max_backlog = 5000`
5. Create a custom configuration file `/etc/sysctl.d/99-custom-tuning.conf` with your optimizations
6. Test loading the configuration without rebooting
7. Create a script that compares current kernel parameters with saved baseline values

### Hints and Resources
1. `sysctl -a` lists all kernel parameters. Use `grep` to filter specific ones
2. `sysctl -w parameter=value` sets a parameter temporarily (lost on reboot)
3. `/etc/sysctl.conf` is the main persistent configuration file, but `/etc/sysctl.d/*.conf` files are preferred
4. `sysctl -p` or `sysctl --system` reloads configuration from files
5. References: [Ubuntu sysctl manual](https://manpages.ubuntu.com/manpages/jammy/man8/sysctl.8.html), [Linux kernel parameters](https://www.kernel.org/doc/Documentation/sysctl/)

### Estimated Time and Difficulty
**25-35 minutes, Intermediate**

### Verification
To verify your solution:
- Check temporary changes: `sysctl fs.file-max` should show new value, but it will revert after reboot
- Verify persistent changes are in `/etc/sysctl.conf` or `/etc/sysctl.d/99-custom-tuning.conf`
- Run `sysctl --system` and check for errors
- Use `sysctl -a | grep -E 'fs.file-max|vm.swappiness|net.core.somaxconn'` to verify active values
- Simulate a reboot test: save values, set different temporary values, reload config, verify original values return

<details>
<summary>Solution</summary>

```bash
# Task 1: Display and backup all kernel parameters
sysctl -a > ~/kernel_tuning/original_params.txt 2>/dev/null

# Alternative: Filter out some noisy parameters
sysctl -a 2>/dev/null | grep -v "^kernel.random" > ~/kernel_tuning/original_params.txt

# Task 2: View current values of specific parameters
echo "=== Current Kernel Parameters ==="
sysctl fs.file-max
sysctl net.ipv4.ip_local_port_range
sysctl vm.swappiness
sysctl net.core.somaxconn

# Save to file for comparison
cat > ~/kernel_tuning/baseline_params.txt << EOF
fs.file-max = $(sysctl -n fs.file-max)
net.ipv4.ip_local_port_range = $(sysctl -n net.ipv4.ip_local_port_range)
vm.swappiness = $(sysctl -n vm.swappiness)
net.core.somaxconn = $(sysctl -n net.core.somaxconn)
EOF

cat ~/kernel_tuning/baseline_params.txt

# Task 3: Make temporary (non-persistent) changes
# These will be lost after reboot
sudo sysctl -w fs.file-max=2097152
sudo sysctl -w vm.swappiness=10
sudo sysctl -w net.core.somaxconn=4096

# Verify temporary changes
echo "=== Temporary Changes Applied ==="
sysctl fs.file-max
sysctl vm.swappiness
sysctl net.core.somaxconn

# Task 4 & 5: Make permanent changes using custom config file
# Create custom sysctl configuration
sudo bash -c 'cat > /etc/sysctl.d/99-custom-tuning.conf << EOF
# Custom kernel parameter tuning
# Created: $(date)

# Filesystem parameters
fs.file-max = 2097152

# Memory management
vm.swappiness = 10

# Network parameters
net.ipv4.ip_local_port_range = 10000 65000
net.core.somaxconn = 4096
net.core.netdev_max_backlog = 5000
EOF'

# Alternative: Add to main sysctl.conf (less preferred)
# sudo bash -c 'cat >> /etc/sysctl.conf << EOF
# # Custom tuning parameters
# fs.file-max = 2097152
# vm.swappiness = 10
# EOF'

# Task 6: Load the configuration without rebooting
sudo sysctl --system

# Or load specific file:
sudo sysctl -p /etc/sysctl.d/99-custom-tuning.conf

# Verify persistent changes are loaded
echo "=== Persistent Changes Loaded ==="
sysctl fs.file-max
sysctl vm.swappiness
sysctl net.ipv4.ip_local_port_range
sysctl net.core.netdev_max_backlog

# Task 7: Create comparison script
cat > ~/kernel_tuning/compare_params.sh << 'EOF'
#!/bin/bash
# Kernel parameter comparison script

BASELINE="$HOME/kernel_tuning/baseline_params.txt"
CURRENT="/tmp/current_params.txt"

echo "=== Kernel Parameter Comparison ==="
echo "Baseline file: $BASELINE"
echo ""

# Get current values
cat > "$CURRENT" << PARAMS
fs.file-max = $(sysctl -n fs.file-max)
net.ipv4.ip_local_port_range = $(sysctl -n net.ipv4.ip_local_port_range)
vm.swappiness = $(sysctl -n vm.swappiness)
net.core.somaxconn = $(sysctl -n net.core.somaxconn)
PARAMS

if [ ! -f "$BASELINE" ]; then
    echo "ERROR: Baseline file not found!"
    exit 1
fi

echo "Parameter Comparison:"
echo "---------------------"

while IFS='=' read -r param baseline_value; do
    param=$(echo "$param" | xargs)  # Trim whitespace
    baseline_value=$(echo "$baseline_value" | xargs)
    
    # Get current value
    current_value=$(sysctl -n "$param" 2>/dev/null | xargs)
    
    if [ "$baseline_value" != "$current_value" ]; then
        echo "✗ $param"
        echo "  Baseline: $baseline_value"
        echo "  Current:  $current_value"
        echo ""
    else
        echo "✓ $param (unchanged: $current_value)"
    fi
done < "$BASELINE"

rm -f "$CURRENT"
EOF

chmod +x ~/kernel_tuning/compare_params.sh

# Run the comparison script
~/kernel_tuning/compare_params.sh
```

**Explanation**:
- **Task 1**: `sysctl -a` dumps all kernel parameters. Redirecting stderr (`2>/dev/null`) suppresses permission errors for some parameters
- **Task 2**: `sysctl -n` displays only the value without the parameter name, useful for scripting
- **Task 3**: `sysctl -w` writes changes immediately but they're non-persistent (lost on reboot)
- **Task 4-5**: `/etc/sysctl.d/*.conf` files are processed in lexical order. `99-` prefix ensures it loads late, overriding earlier configs
- **Task 6**: `sysctl --system` reloads all configuration files from `/etc/sysctl.conf` and `/etc/sysctl.d/`
- **Task 7**: The comparison script helps identify which parameters have changed from baseline

**Common Kernel Parameters**:

**Filesystem Parameters**:
```bash
fs.file-max = 2097152              # Maximum number of file handles
fs.inotify.max_user_watches = 524288  # For applications using inotify
```

**Network Parameters**:
```bash
net.ipv4.ip_local_port_range = 10000 65000  # Port range for connections
net.ipv4.tcp_fin_timeout = 30               # TCP timeout
net.core.somaxconn = 4096                   # Connection queue size
net.core.netdev_max_backlog = 5000          # Network device backlog
net.ipv4.tcp_max_syn_backlog = 4096         # SYN queue size
```

**Memory Parameters**:
```bash
vm.swappiness = 10                 # Lower = less aggressive swapping
vm.dirty_ratio = 15                # Percentage of RAM for dirty pages
vm.dirty_background_ratio = 5      # Background writeback threshold
```

**Security Parameters**:
```bash
kernel.dmesg_restrict = 1          # Restrict dmesg to root
kernel.kptr_restrict = 2           # Hide kernel pointers
```

**Troubleshooting**:
- If `sysctl -p` fails, check syntax in config files
- Use `sysctl -w` for testing before making permanent
- Check `dmesg` or `journalctl -xe` for kernel parameter errors
- Some parameters require kernel modules or specific hardware

</details>

### Extensions
1. **Advanced**: Research and implement a complete performance tuning configuration for a specific workload (database server, web server, or network router). Document why each parameter was chosen
2. **Challenge**: Write a script that automatically backs up current kernel parameters, applies optimizations from a profile file, and can rollback changes if system performance degrades (using performance metrics)

---

## Task 2: Process and Service Troubleshooting

### Learning Objectives
- Diagnose and troubleshoot problematic processes using tools like `ps`, `top`, `htop`, `lsof`, and `/proc`
- Manage process priorities with `nice` and `renice`
- Investigate and resolve zombie processes, runaway processes, and resource exhaustion

### Context
You've received urgent alerts that a production application server is running slowly and users are experiencing timeouts. The monitoring system shows high CPU usage and memory consumption, but the root cause is unclear. Several processes appear to be stuck, there are zombie processes accumulating, and one process is consuming excessive resources. You need to quickly identify the problematic processes, understand what they're doing, adjust their priorities, and resolve the issues before the situation escalates to a full outage.

Process troubleshooting is a daily task for system administrators and a core LFCS competency. Being able to quickly identify resource hogs, stuck processes, and system bottlenecks is essential for maintaining system health and uptime.

### Task Instructions

**Prerequisites**: Set up test processes to troubleshoot
```bash
# Install required tools
sudo apt update
sudo apt install -y htop iotop lsof strace

# Create directory for test processes
mkdir -p ~/process_lab
cd ~/process_lab

# Create a CPU-intensive process script
cat > cpu_hog.sh << 'EOF'
#!/bin/bash
# Simulates a CPU-intensive process
echo "Starting CPU hog (PID: $$)"
while true; do
    echo "scale=5000; a(1)*4" | bc -l > /dev/null
done
EOF

# Create a memory-consuming process
cat > memory_hog.sh << 'EOF'
#!/bin/bash
# Simulates a memory leak
echo "Starting memory hog (PID: $$)"
data=""
while true; do
    data="$data$(head -c 1M /dev/urandom | base64)"
    sleep 1
done
EOF

# Create a zombie process generator
cat > zombie_maker.sh << 'EOF'
#!/bin/bash
# Creates zombie processes
echo "Creating zombie process..."
(sleep 300 &)
sleep 1
EOF

# Create a script that opens many files
cat > file_hog.sh << 'EOF'
#!/bin/bash
# Opens many files
echo "Opening many files (PID: $$)"
for i in {1..100}; do
    exec 3< /etc/hosts
    sleep 0.1
done
sleep 3600
EOF

chmod +x *.sh

# Start some test processes in background
./cpu_hog.sh &
CPU_HOG_PID=$!
echo "Started CPU hog: $CPU_HOG_PID"

./file_hog.sh &
FILE_HOG_PID=$!
echo "Started file hog: $FILE_HOG_PID"
```

**Your Tasks**:
1. Use `ps` to display all running processes with detailed information (CPU, memory, state, command)
2. Identify the top 5 CPU-consuming processes and top 5 memory-consuming processes
3. Find the CPU hog process and:
   - Determine its parent process ID (PPID)
   - Check how long it's been running
   - View its command line arguments
   - Check which user owns it
4. Use `lsof` to list all files opened by the file hog process
5. Inspect the `/proc` filesystem to gather information about the CPU hog:
   - View its status (`/proc/PID/status`)
   - Check its environment variables
   - View its current working directory
   - Check its open file descriptors
6. Reduce the CPU hog's priority using `renice` to make it use less CPU time
7. Create a zombie process and identify it using `ps`, then clean it up
8. Use `strace` to monitor system calls made by a process
9. Find and kill all processes matching a pattern (e.g., all your test scripts)
10. Create a monitoring script that alerts when a process exceeds resource thresholds

### Hints and Resources
1. `ps aux` shows all processes. Use `--sort` for sorting by CPU or memory
2. `lsof -p PID` lists open files for a specific process. `lsof -u user` shows files opened by a user
3. `/proc/PID/` contains runtime information about process PID
4. `nice` sets priority when starting a process; `renice` changes priority of running process. Lower nice value = higher priority
5. References: [Ubuntu ps manual](https://manpages.ubuntu.com/manpages/jammy/man1/ps.1.html), [proc filesystem](https://man7.org/linux/man-pages/man5/proc.5.html)

### Estimated Time and Difficulty
**30-45 minutes, Intermediate to Advanced**

### Verification
To verify your solution:
- `ps` output should show processes sorted by resource usage
- `lsof -p PID` should list open files for the specified process
- Check `/proc/PID/status` exists and contains process information
- After `renice`, verify the process nice value changed with `ps -l`
- Zombie processes show as `<defunct>` or state `Z` in `ps`
- `pgrep` or `pidof` can find processes by name
- After killing processes, verify they're gone with `ps` or `pgrep`

<details>
<summary>Solution</summary>

```bash
# Task 1: Display all processes with detailed information
ps aux
# Alternative with custom format:
ps -eo pid,ppid,user,%cpu,%mem,stat,start,time,command

# Show processes in tree format:
ps auxf
# Or use pstree:
pstree -p

# Task 2: Identify top CPU and memory consumers
# Top 5 CPU processes:
ps aux --sort=-%cpu | head -6

# Top 5 memory processes:
ps aux --sort=-%mem | head -6

# Using awk for cleaner output:
echo "=== Top 5 CPU Processes ==="
ps aux --sort=-%cpu | awk 'NR==1; NR>1 {print $0}' | head -6

echo "=== Top 5 Memory Processes ==="
ps aux --sort=-%mem | awk 'NR==1; NR>1 {print $0}' | head -6

# Task 3: Investigate the CPU hog process
# Find the PID (replace with your actual CPU hog)
CPU_HOG_PID=$(pgrep -f cpu_hog.sh)
echo "CPU Hog PID: $CPU_HOG_PID"

# Get detailed information
ps -p $CPU_HOG_PID -o pid,ppid,user,%cpu,%mem,stat,start,etime,cmd

# Parent process ID:
ps -p $CPU_HOG_PID -o ppid=

# Running time (elapsed time):
ps -p $CPU_HOG_PID -o etime=

# Full command line:
ps -p $CPU_HOG_PID -o cmd=
# Or from /proc:
cat /proc/$CPU_HOG_PID/cmdline | tr '\0' ' '

# Process owner:
ps -p $CPU_HOG_PID -o user=

# Task 4: List files opened by file hog
FILE_HOG_PID=$(pgrep -f file_hog.sh)
echo "File Hog PID: $FILE_HOG_PID"

# List all open files:
sudo lsof -p $FILE_HOG_PID

# Count open files:
sudo lsof -p $FILE_HOG_PID | wc -l

# Show only regular files:
sudo lsof -p $FILE_HOG_PID -a -d 0-999

# Task 5: Inspect /proc filesystem
echo "=== Process Status ==="
cat /proc/$CPU_HOG_PID/status

# Environment variables:
echo "=== Environment Variables ==="
cat /proc/$CPU_HOG_PID/environ | tr '\0' '\n'

# Current working directory:
echo "=== Working Directory ==="
ls -l /proc/$CPU_HOG_PID/cwd

# Open file descriptors:
echo "=== Open File Descriptors ==="
ls -l /proc/$CPU_HOG_PID/fd/

# Command line:
echo "=== Command Line ==="
cat /proc/$CPU_HOG_PID/cmdline | tr '\0' ' ' && echo

# Memory maps:
echo "=== Memory Maps ==="
cat /proc/$CPU_HOG_PID/maps | head -20

# Task 6: Reduce CPU hog priority
# Check current nice value:
ps -p $CPU_HOG_PID -o pid,ni,cmd

# Increase nice value (lower priority):
sudo renice +10 -p $CPU_HOG_PID

# Verify change:
ps -p $CPU_HOG_PID -o pid,ni,cmd

# Alternative: Set to specific nice value
sudo renice -n 15 -p $CPU_HOG_PID

# Task 7: Create and identify zombie process
# Run zombie maker:
./zombie_maker.sh &
MAKER_PID=$!

# Wait a moment for zombie to appear
sleep 2

# Find zombie processes:
ps aux | grep -w Z
# Or:
ps aux | grep defunct

# More detailed zombie search:
ps -eo pid,ppid,stat,cmd | grep '^[0-9]* *[0-9]* Z'

# Clean up zombie by killing parent:
kill $MAKER_PID

# Verify zombie is gone:
ps aux | grep defunct

# Task 8: Monitor system calls with strace
# Attach to running process:
sudo strace -p $CPU_HOG_PID -c

# Monitor a new process from start:
strace -o trace.log ls -la

# Show only file operations:
sudo strace -p $CPU_HOG_PID -e trace=file

# Count system calls:
sudo strace -p $CPU_HOG_PID -c -f

# Task 9: Find and kill processes by pattern
# Find all test script processes:
pgrep -f "hog.sh"

# List with details:
pgrep -a -f "hog.sh"

# Kill all matching processes:
pkill -f "hog.sh"

# Alternative using ps and awk:
ps aux | grep 'hog.sh' | grep -v grep | awk '{print $2}' | xargs kill

# Verify they're killed:
pgrep -f "hog.sh"

# Force kill if needed:
pkill -9 -f "hog.sh"

# Task 10: Create resource monitoring script
cat > ~/process_lab/monitor_resources.sh << 'EOF'
#!/bin/bash
# Process resource monitoring script

# Thresholds
CPU_THRESHOLD=80
MEM_THRESHOLD=80
LOG_FILE="/var/log/process_monitor.log"

echo "=== Process Resource Monitor ===" | sudo tee -a $LOG_FILE
echo "Started: $(date)" | sudo tee -a $LOG_FILE
echo "" | sudo tee -a $LOG_FILE

# Monitor CPU usage
echo "Checking CPU usage..." | sudo tee -a $LOG_FILE
while read -r pid user cpu mem cmd; do
    cpu_int=${cpu%.*}  # Convert to integer
    if [ "$cpu_int" -gt "$CPU_THRESHOLD" ]; then
        echo "⚠️  ALERT: High CPU usage detected!" | sudo tee -a $LOG_FILE
        echo "  PID: $pid | User: $user | CPU: $cpu% | Command: $cmd" | sudo tee -a $LOG_FILE
    fi
done < <(ps aux --sort=-%cpu | awk 'NR>1 {print $2, $1, $3, $4, $11}' | head -10)

# Monitor Memory usage
echo "" | sudo tee -a $LOG_FILE
echo "Checking memory usage..." | sudo tee -a $LOG_FILE
while read -r pid user cpu mem cmd; do
    mem_int=${mem%.*}  # Convert to integer
    if [ "$mem_int" -gt "$MEM_THRESHOLD" ]; then
        echo "⚠️  ALERT: High memory usage detected!" | sudo tee -a $LOG_FILE
        echo "  PID: $pid | User: $user | Memory: $mem% | Command: $cmd" | sudo tee -a $LOG_FILE
    fi
done < <(ps aux --sort=-%mem | awk 'NR>1 {print $2, $1, $3, $4, $11}' | head -10)

# Check for zombie processes
echo "" | sudo tee -a $LOG_FILE
echo "Checking for zombie processes..." | sudo tee -a $LOG_FILE
zombies=$(ps aux | grep -w Z | grep -v grep)
if [ -n "$zombies" ]; then
    echo "⚠️  ALERT: Zombie processes found!" | sudo tee -a $LOG_FILE
    echo "$zombies" | sudo tee -a $LOG_FILE
else
    echo "✓ No zombie processes found" | sudo tee -a $LOG_FILE
fi

echo "" | sudo tee -a $LOG_FILE
echo "Completed: $(date)" | sudo tee -a $LOG_FILE
echo "===========================================" | sudo tee -a $LOG_FILE
EOF

chmod +x ~/process_lab/monitor_resources.sh

# Run the monitoring script:
~/process_lab/monitor_resources.sh

# View the log:
sudo tail -30 /var/log/process_monitor.log
```

**Explanation**:
- **Task 1**: `ps aux` shows all processes. Format codes: `a`=all users, `u`=user-oriented format, `x`=include processes without TTY
- **Task 2**: `--sort` can use `-` prefix for descending order. Sort keys: `%cpu`, `%mem`, `pid`, `cmd`
- **Task 3**: `pgrep` finds PIDs by name/pattern. `-f` matches full command line
- **Task 4**: `lsof` = "list open files". Shows files, sockets, pipes, etc.
- **Task 5**: `/proc/PID/` is a virtual filesystem providing process information in real-time
- **Task 6**: Nice values range from -20 (highest priority) to +19 (lowest priority). Default is 0
- **Task 7**: Zombie processes occur when child exits but parent hasn't called `wait()`. They only release resources when parent exits or calls `wait()`
- **Task 8**: `strace` intercepts and logs system calls. Use `-c` for summary, `-e` to filter by call type
- **Task 9**: `pkill` kills by name, `killall` kills by exact name
- **Task 10**: The monitoring script checks thresholds and logs alerts

**Additional Process Management Commands**:
```bash
# Process information
pidof <name>           # Find PID by process name
pgrep <pattern>        # Find PID by pattern
top                    # Interactive process viewer
htop                   # Enhanced interactive viewer
atop                   # Advanced system monitor

# Process control
kill <PID>             # Send TERM signal (default)
kill -9 <PID>          # Send KILL signal (force)
kill -STOP <PID>       # Pause process
kill -CONT <PID>       # Resume process
killall <name>         # Kill by process name
pkill <pattern>        # Kill by pattern

# Process priority
nice -n 10 command     # Start with nice value 10
renice -n 5 -p PID     # Change nice value
ionice -c2 -n0 -p PID  # Set I/O priority

# Process investigation
lsof -i :80            # Find process using port 80
fuser -v /path/file    # Find processes using file
pmap <PID>             # Memory map of process
pwdx <PID>             # Working directory of process
```

**Process States**:
- **R**: Running or runnable
- **S**: Interruptible sleep (waiting for event)
- **D**: Uninterruptible sleep (usually I/O)
- **Z**: Zombie (terminated but not reaped)
- **T**: Stopped (by signal or debugger)
- **I**: Idle kernel thread

</details>

### Extensions
1. **Advanced**: Create a comprehensive process diagnostic script that generates a report including: process tree, resource usage, open files, network connections, and system calls for a given PID
2. **Challenge**: Set up `systemd` resource limits (CPUQuota, MemoryLimit) for a custom service and verify they're enforced. Then create a monitoring dashboard using `systemd-cgtop`

---

## Task 3: Job Scheduling and Automation

### Learning Objectives
- Schedule one-time and recurring jobs using `at`, `cron`, and `systemd timers`
- Manage crontab files and understand cron syntax
- Troubleshoot scheduling issues and verify job execution

### Context
Your organization needs to automate various system maintenance tasks: daily database backups, weekly log rotation, monthly security scans, and one-time migration jobs. Different tasks have different schedules and requirements. Some need to run as specific users, others require precise timing, and some need complex scheduling logic. You need to implement a robust job scheduling strategy using the appropriate tools for each scenario.

Job scheduling is fundamental to system automation and a key LFCS skill. Understanding when to use cron vs systemd timers vs at commands allows you to build reliable automated maintenance workflows that run without manual intervention.

### Task Instructions

**Prerequisites**: Set up test environment for job scheduling
```bash
# Ensure cron is installed and running
sudo apt update
sudo apt install -y cron at

# Start and enable cron service
sudo systemctl start cron
sudo systemctl enable cron

# Start and enable atd (at daemon)
sudo systemctl start atd
sudo systemctl enable atd

# Create working directory
mkdir -p ~/scheduling_lab/{scripts,logs}
cd ~/scheduling_lab

# Create a sample backup script
cat > scripts/backup.sh << 'EOF'
#!/bin/bash
# Simple backup script
BACKUP_DIR="$HOME/scheduling_lab/backups"
LOG_FILE="$HOME/scheduling_lab/logs/backup.log"

mkdir -p "$BACKUP_DIR"

echo "$(date): Starting backup..." >> "$LOG_FILE"
tar -czf "$BACKUP_DIR/backup-$(date +%Y%m%d-%H%M%S).tar.gz" \
    "$HOME/scheduling_lab/scripts" >> "$LOG_FILE" 2>&1

echo "$(date): Backup completed" >> "$LOG_FILE"
EOF

chmod +x scripts/backup.sh

# Create a cleanup script
cat > scripts/cleanup.sh << 'EOF'
#!/bin/bash
# Cleanup old files
LOG_FILE="$HOME/scheduling_lab/logs/cleanup.log"

echo "$(date): Running cleanup..." >> "$LOG_FILE"
find "$HOME/scheduling_lab/backups" -name "*.tar.gz" -mtime +7 -delete
echo "$(date): Cleanup completed" >> "$LOG_FILE"
EOF

chmod +x scripts/cleanup.sh
```

**Your Tasks**:

**Part A: Using cron**
1. View the current user's crontab and system-wide cron jobs
2. Create a crontab entry that runs the backup script every day at 2:00 AM
3. Create a crontab entry that runs the cleanup script every Sunday at 3:00 AM
4. Schedule a job that runs every 5 minutes during business hours (9 AM - 5 PM, Monday-Friday)
5. Add a job that runs on the first day of every month at midnight
6. Configure environment variables in your crontab (PATH, MAILTO)
7. Redirect cron job output to a log file

**Part B: Using at for one-time jobs**
8. Schedule a one-time job to run in 2 minutes using `at`
9. Schedule a job to run at a specific time tomorrow
10. List all pending `at` jobs
11. Remove an `at` job before it executes

**Part C: Using systemd timers**
12. Create a systemd timer that runs a service every hour
13. Create a systemd service and timer for your backup script
14. Enable and start the timer
15. Check timer status and view when it will run next

**Part D: Troubleshooting and verification**
16. View cron logs to verify job execution
17. Test a cron job immediately without waiting for schedule
18. Create a monitoring script that checks if scheduled jobs are running correctly

### Hints and Resources
1. Cron syntax: `minute hour day month day_of_week command`. Use `*` for "every" and `*/5` for "every 5"
2. `crontab -e` edits current user's crontab, `crontab -l` lists it
3. System cron files are in `/etc/cron.d/`, `/etc/cron.daily/`, etc.
4. `at` uses natural language: `at 3pm tomorrow`, `at now + 2 hours`
5. References: [Ubuntu cron manual](https://manpages.ubuntu.com/manpages/jammy/man8/cron.8.html), [systemd timer documentation](https://www.freedesktop.org/software/systemd/man/systemd.timer.html)

### Estimated Time and Difficulty
**35-45 minutes, Intermediate**

### Verification
To verify your solution:
- `crontab -l` should show your scheduled jobs
- Check `/var/log/syslog` or `journalctl -u cron` for cron execution logs
- `atq` or `at -l` lists pending at jobs
- `systemctl list-timers` shows all systemd timers and their next run time
- Your log files should show entries after jobs run
- Test scripts manually to ensure they work before scheduling

<details>
<summary>Solution</summary>

```bash
# Part A: Using cron

# Task 1: View current crontab and system cron jobs
# View user crontab:
crontab -l

# View system-wide cron jobs:
ls -la /etc/cron.d/
ls -la /etc/cron.daily/
ls -la /etc/cron.hourly/
ls -la /etc/cron.weekly/
ls -la /etc/cron.monthly/

# View system crontab:
cat /etc/crontab

# Task 2-7: Create crontab entries
# Edit crontab:
crontab -e

# Add the following entries:
```

**Crontab entries to add**:
```cron
# Set environment variables
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
MAILTO=root
HOME=/home/yourusername

# Task 2: Daily backup at 2:00 AM
0 2 * * * /home/yourusername/scheduling_lab/scripts/backup.sh >> /home/yourusername/scheduling_lab/logs/cron.log 2>&1

# Task 3: Weekly cleanup on Sunday at 3:00 AM
0 3 * * 0 /home/yourusername/scheduling_lab/scripts/cleanup.sh >> /home/yourusername/scheduling_lab/logs/cron.log 2>&1

# Task 4: Every 5 minutes during business hours (9 AM - 5 PM, Mon-Fri)
*/5 9-17 * * 1-5 echo "Business hours check: $(date)" >> /home/yourusername/scheduling_lab/logs/business.log

# Task 5: First day of every month at midnight
0 0 1 * * /home/yourusername/scheduling_lab/scripts/monthly_task.sh >> /home/yourusername/scheduling_lab/logs/monthly.log 2>&1

# Alternative schedules (examples):
# Every 15 minutes:
# */15 * * * * command

# Every hour at minute 30:
# 30 * * * * command

# Every day at 6:30 AM:
# 30 6 * * * command

# Twice daily (10 AM and 10 PM):
# 0 10,22 * * * command

# Every weekday at 9 AM:
# 0 9 * * 1-5 command
```

**Continuing with solution**:
```bash
# After editing crontab, verify:
crontab -l

# Part B: Using at for one-time jobs

# Task 8: Schedule job in 2 minutes
echo "echo 'Job executed at $(date)' >> $HOME/scheduling_lab/logs/at.log" | at now + 2 minutes

# Alternative syntax:
at now + 2 minutes << EOF
echo "Task completed at \$(date)" >> $HOME/scheduling_lab/logs/at.log
EOF

# Task 9: Schedule for specific time tomorrow
at 10:30 tomorrow << EOF
$HOME/scheduling_lab/scripts/backup.sh
EOF

# Other time specifications:
at 3pm + 2 days << EOF
command
EOF

at 4:00 AM next Monday << EOF
command
EOF

at midnight << EOF
command
EOF

# Task 10: List pending at jobs
atq
# Or:
at -l

# Detailed view of a specific job:
at -c <job_number>

# Task 11: Remove an at job
# Get job number from atq, then:
atrm <job_number>
# Or:
at -r <job_number>

# Part C: Using systemd timers

# Task 12-15: Create systemd service and timer

# Create the service unit file
sudo bash -c 'cat > /etc/systemd/system/backup-timer.service << EOF
[Unit]
Description=Backup Service
Wants=backup-timer.timer

[Service]
Type=oneshot
ExecStart=/home/'$USER'/scheduling_lab/scripts/backup.sh

[Install]
WantedBy=multi-user.target
EOF'

# Create the timer unit file
sudo bash -c 'cat > /etc/systemd/system/backup-timer.timer << EOF
[Unit]
Description=Backup Timer
Requires=backup-timer.service

[Timer]
# Run every hour
OnCalendar=hourly
# Equivalent to:
# OnCalendar=*-*-* *:00:00

# Alternative schedules:
# OnCalendar=daily                    # Every day at midnight
# OnCalendar=weekly                   # Every Monday at midnight
# OnCalendar=monthly                  # First day of month at midnight
# OnCalendar=*-*-* 02:00:00          # Every day at 2 AM
# OnCalendar=Mon *-*-* 12:00:00      # Every Monday at noon
# OnCalendar=*-*-1 00:00:00          # First of month at midnight

# Run 5 minutes after boot if missed
OnBootSec=5min
# Run 10 minutes after last activation
OnUnitActiveSec=10min

Persistent=true

[Install]
WantedBy=timers.target
EOF'

# Reload systemd daemon
sudo systemctl daemon-reload

# Enable and start the timer
sudo systemctl enable backup-timer.timer
sudo systemctl start backup-timer.timer

# Check timer status
systemctl status backup-timer.timer

# List all timers
systemctl list-timers --all

# See when timer will run next
systemctl list-timers backup-timer.timer

# Manually trigger the service (for testing)
sudo systemctl start backup-timer.service

# View service logs
journalctl -u backup-timer.service

# Part D: Troubleshooting and verification

# Task 16: View cron logs
# On Ubuntu, cron logs to syslog:
sudo grep CRON /var/log/syslog | tail -20

# Using journalctl:
journalctl -u cron.service --since today

# View user-specific cron logs:
journalctl -t CRON --since "1 hour ago"

# Task 17: Test cron job immediately
# Extract command from crontab and run manually:
crontab -l | grep backup.sh | cut -d' ' -f6- | bash

# Or create a test crontab entry that runs every minute:
# */1 * * * * /path/to/script.sh

# Task 18: Create monitoring script
cat > ~/scheduling_lab/scripts/monitor_jobs.sh << 'EOF'
#!/bin/bash
# Job scheduling monitor

echo "=== Scheduled Jobs Monitor ==="
echo "Report generated: $(date)"
echo ""

# Check cron service status
echo "=== Cron Service Status ==="
systemctl is-active cron.service && echo "✓ Cron is running" || echo "✗ Cron is NOT running"
echo ""

# List user crontab
echo "=== User Crontab ==="
crontab -l 2>/dev/null || echo "No user crontab"
echo ""

# List pending at jobs
echo "=== Pending AT Jobs ==="
atq | awk '{print "Job ID: " $1 " | Time: " $2, $3, $4, $5}'
[ $(atq | wc -l) -eq 0 ] && echo "No pending at jobs"
echo ""

# List systemd timers
echo "=== Active Systemd Timers ==="
systemctl list-timers --no-pager | grep -v "^$"
echo ""

# Check recent cron executions
echo "=== Recent Cron Executions (last 10) ==="
sudo grep CRON /var/log/syslog | tail -10
echo ""

# Check backup script execution
echo "=== Backup Log Status ==="
BACKUP_LOG="$HOME/scheduling_lab/logs/backup.log"
if [ -f "$BACKUP_LOG" ]; then
    echo "Last backup:"
    tail -2 "$BACKUP_LOG"
    echo ""
    echo "Total backups: $(grep -c "Starting backup" "$BACKUP_LOG")"
else
    echo "No backup log found"
fi
echo ""

# Check for missed jobs (requires anacron)
if command -v anacron &> /dev/null; then
    echo "=== Anacron Status ==="
    sudo cat /var/spool/anacron/* 2>/dev/null || echo "No anacron spool data"
fi

echo ""
echo "=== Report Complete ==="
EOF

chmod +x ~/scheduling_lab/scripts/monitor_jobs.sh

# Run the monitoring script
~/scheduling_lab/scripts/monitor_jobs.sh
```

**Explanation**:
- **Cron syntax**: `minute(0-59) hour(0-23) day(1-31) month(1-12) weekday(0-7) command`
- **Special characters**: 
  - `*` = every value
  - `*/5` = every 5 units
  - `1,5,10` = specific values
  - `1-5` = range
  - `0-23/2` = every 2 hours
- **at command**: Uses natural language for time specifications
- **systemd timers**: More powerful than cron, integrated with systemd, better logging
- **OnCalendar**: systemd time specification. Can use keywords like `hourly`, `daily`, `weekly`, or exact times

**Cron Special Strings**:
```cron
@reboot       # Run once at startup
@yearly       # Run once a year:  0 0 1 1 *
@annually     # Same as @yearly
@monthly      # Run once a month: 0 0 1 * *
@weekly       # Run once a week:  0 0 * * 0
@daily        # Run once a day:   0 0 * * *
@midnight     # Same as @daily
@hourly       # Run once an hour: 0 * * * *
```

**systemd Timer Options**:
```ini
OnCalendar=      # Calendar-based activation
OnBootSec=       # Relative to system boot
OnStartupSec=    # Relative to systemd start
OnUnitActiveSec= # Relative to last activation
OnUnitInactiveSec= # Relative to last deactivation
Persistent=true  # Catch up if system was off
```

**Troubleshooting Tips**:
- Cron doesn't source your `.bashrc`, so use full paths
- Set `MAILTO` in crontab to receive error emails
- Test scripts manually before scheduling
- Check permissions on scripts (executable bit)
- Verify `crond` service is running: `systemctl status cron`
- For systemd timers, check `journalctl -u timer-name`
- Use `run-parts --test /etc/cron.daily` to test cron.daily scripts

</details>

### Extensions
1. **Advanced**: Create a complex systemd timer that runs a service only during low-usage hours (nights/weekends), with different frequencies for weekdays vs weekends, and includes failure retry logic
2. **Challenge**: Build an automated backup system using systemd timers with the following requirements: daily incremental backups, weekly full backups, automatic cleanup of old backups, health checks, and email notifications on failure

---

## Task 4: Software Package Management and Repository Configuration

### Learning Objectives
- Search for, install, update, and remove software packages using `apt` and `dpkg`
- Configure and manage software repositories
- Validate package integrity and troubleshoot package conflicts

### Context
As the system administrator for a development team, you need to install specific software versions, manage third-party repositories, and resolve package dependency issues. The team needs Docker, PostgreSQL 15 (not the default version), and various development tools. You also need to ensure all packages are validated for security and handle situations where packages have conflicts or broken dependencies. Additionally, you must maintain a local package cache for offline installations and create a strategy for keeping systems updated without breaking critical applications.

Package management is one of the most fundamental LFCS skills. Every Linux administrator needs to master package installation, repository management, and dependency resolution. Understanding the difference between high-level (`apt`) and low-level (`dpkg`) tools is essential for troubleshooting complex package issues.

### Task Instructions

**Prerequisites**: Set up package management environment
```bash
# Update package index
sudo apt update

# Create working directory
mkdir -p ~/package_lab/{downloads,repo_backups}
cd ~/package_lab

# Backup current repository configuration
sudo cp -r /etc/apt/sources.list /etc/apt/sources.list.backup
sudo cp -r /etc/apt/sources.list.d/ ~/package_lab/repo_backups/
```

**Your Tasks**:

**Part A: Basic Package Operations**
1. Search for packages related to "nginx" and "postgresql"
2. Display detailed information about the `nginx` package (version, dependencies, description)
3. Install `nginx` and verify it's installed correctly
4. List all installed packages and save to a file
5. Show which package provides the `/usr/bin/vim` file
6. Remove `nginx` but keep its configuration files, then purge it completely

**Part B: Repository Management**
7. List all configured repositories (both enabled and disabled)
8. Add the official PostgreSQL repository to install PostgreSQL 15
9. Add GPG keys for the new repository
10. Update package index and verify the new repository is working
11. Install PostgreSQL 15 from the new repository
12. Disable a repository without removing it

**Part C: Advanced Package Management**
13. Download a `.deb` package without installing it
14. Install a package from a local `.deb` file
15. Hold a package at its current version (prevent updates)
16. List all packages that can be upgraded
17. Simulate an upgrade without actually performing it

**Part D: Troubleshooting**
18. Fix broken package dependencies
19. Verify integrity of installed packages
20. Find which package a file belongs to
21. Create a script that audits system packages and reports security updates

### Hints and Resources
1. `apt search` searches for packages, `apt show` displays package details
2. `dpkg -l` lists installed packages, `dpkg -S` finds which package owns a file
3. Repository configuration is in `/etc/apt/sources.list` and `/etc/apt/sources.list.d/`
4. `apt-key` manages GPG keys (deprecated, use `/etc/apt/trusted.gpg.d/` instead)
5. References: [Ubuntu apt manual](https://manpages.ubuntu.com/manpages/jammy/man8/apt.8.html), [Debian Package Management](https://www.debian.org/doc/manuals/debian-reference/ch02.en.html)

### Estimated Time and Difficulty
**35-50 minutes, Intermediate to Advanced**

### Verification
To verify your solution:
- `dpkg -l | grep package-name` shows if a package is installed
- `apt policy package` shows available versions and installed version
- `apt-cache policy` shows repository priorities
- `/etc/apt/sources.list.d/` should contain your new repository files
- `apt update` should complete without errors after adding repositories
- `apt-mark showhold` lists packages on hold
- `apt list --upgradable` shows packages that can be updated

<details>
<summary>Solution</summary>

```bash
# Part A: Basic Package Operations

# Task 1: Search for packages
apt search nginx
apt search postgresql

# More targeted search:
apt search nginx | grep -i "^nginx"
apt search postgresql | head -20

# Task 2: Display detailed package information
apt show nginx
apt show postgresql

# Show specific information:
apt-cache policy nginx
apt-cache madison postgresql  # Shows all available versions

# Task 3: Install nginx
sudo apt update
sudo apt install -y nginx

# Verify installation:
dpkg -l | grep nginx
systemctl status nginx
nginx -v

# Task 4: List all installed packages
dpkg -l > ~/package_lab/installed_packages.txt

# Alternative: List with apt
apt list --installed > ~/package_lab/apt_installed.txt

# Count installed packages:
dpkg -l | grep "^ii" | wc -l

# Task 5: Find which package provides a file
dpkg -S /usr/bin/vim
# Or use apt-file (requires apt-file package):
# apt-file search /usr/bin/vim

# Find files provided by a package:
dpkg -L vim

# Task 6: Remove vs Purge
# Remove but keep config files:
sudo apt remove nginx

# Verify it's removed but config remains:
dpkg -l | grep nginx  # Shows 'rc' status
ls -la /etc/nginx/    # Config still exists

# Purge completely (removes config too):
sudo apt purge nginx

# Verify complete removal:
dpkg -l | grep nginx
ls -la /etc/nginx/    # Should not exist

# Alternative: Remove with autoremove for dependencies
sudo apt autoremove nginx

# Part B: Repository Management

# Task 7: List all configured repositories
cat /etc/apt/sources.list
ls -la /etc/apt/sources.list.d/

# Show only active repositories:
grep -v "^#" /etc/apt/sources.list | grep -v "^$"

# List with apt:
apt-cache policy

# Task 8-10: Add PostgreSQL repository
# Import repository GPG key (new method):
sudo apt install -y wget
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | \
    sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/postgresql.gpg

# Alternative old method (deprecated):
# wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -

# Add PostgreSQL repository
sudo bash -c 'cat > /etc/apt/sources.list.d/pgdg.list << EOF
deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main
EOF'

# Update package index:
sudo apt update

# Verify repository is working:
apt-cache policy postgresql-15
apt search postgresql-15

# Task 11: Install PostgreSQL 15
sudo apt install -y postgresql-15

# Verify installation:
dpkg -l | grep postgresql-15
systemctl status postgresql
psql --version

# Task 12: Disable repository without removing
# Comment out the repository:
sudo sed -i 's/^deb/# deb/' /etc/apt/sources.list.d/pgdg.list

# Verify it's disabled:
cat /etc/apt/sources.list.d/pgdg.list

# Re-enable:
sudo sed -i 's/^# deb/deb/' /etc/apt/sources.list.d/pgdg.list

# Part C: Advanced Package Management

# Task 13: Download package without installing
cd ~/package_lab/downloads
apt download htop

# Or download to specific directory:
apt download -o Dir::Cache::Archives="." htop

# Task 14: Install from local .deb file
sudo dpkg -i htop*.deb

# If there are dependency issues:
sudo apt install -f  # Fix dependencies

# Alternative using apt:
sudo apt install ./htop*.deb

# Task 15: Hold package at current version
sudo apt-mark hold postgresql-15

# Verify hold:
apt-mark showhold

# Unhold when needed:
sudo apt-mark unhold postgresql-15

# Task 16: List upgradable packages
apt list --upgradable

# Show only package names:
apt list --upgradable -qq

# Task 17: Simulate upgrade
apt list --upgradable
sudo apt upgrade --dry-run

# Or with full output:
sudo apt upgrade -s

# More verbose simulation:
sudo apt dist-upgrade --simulate

# Part D: Troubleshooting

# Task 18: Fix broken dependencies
sudo apt install -f
# Or:
sudo apt --fix-broken install

# Reconfigure broken packages:
sudo dpkg --configure -a

# Force reinstall if needed:
sudo apt install --reinstall package-name

# Task 19: Verify package integrity
# Verify specific package:
debsums nginx

# Install debsums if not available:
sudo apt install -y debsums

# Verify all packages:
sudo debsums -c

# Check for modified config files:
sudo debsums -e -c

# Task 20: Find which package a file belongs to
dpkg -S /usr/bin/python3
dpkg -S /etc/nginx/nginx.conf

# Search in non-installed packages (requires apt-file):
sudo apt install -y apt-file
sudo apt-file update
apt-file search filename

# Task 21: Create package audit script
cat > ~/package_lab/audit_packages.sh << 'EOF'
#!/bin/bash
# Package audit and security update check

REPORT_FILE="$HOME/package_lab/package_audit_$(date +%Y%m%d).txt"

echo "=== Package Audit Report ===" | tee "$REPORT_FILE"
echo "Generated: $(date)" | tee -a "$REPORT_FILE"
echo "" | tee -a "$REPORT_FILE"

# Total installed packages
echo "=== Package Statistics ===" | tee -a "$REPORT_FILE"
TOTAL_PKGS=$(dpkg -l | grep "^ii" | wc -l)
echo "Total installed packages: $TOTAL_PKGS" | tee -a "$REPORT_FILE"
echo "" | tee -a "$REPORT_FILE"

# Upgradable packages
echo "=== Upgradable Packages ===" | tee -a "$REPORT_FILE"
sudo apt update -qq
UPGRADABLE=$(apt list --upgradable 2>/dev/null | grep -v "^Listing" | wc -l)
echo "Packages with available updates: $UPGRADABLE" | tee -a "$REPORT_FILE"

if [ $UPGRADABLE -gt 0 ]; then
    echo "" | tee -a "$REPORT_FILE"
    apt list --upgradable 2>/dev/null | grep -v "^Listing" | tee -a "$REPORT_FILE"
fi
echo "" | tee -a "$REPORT_FILE"

# Security updates
echo "=== Security Updates ===" | tee -a "$REPORT_FILE"
SECURITY_UPDATES=$(apt list --upgradable 2>/dev/null | grep -i security | wc -l)
echo "Security updates available: $SECURITY_UPDATES" | tee -a "$REPORT_FILE"

if [ $SECURITY_UPDATES -gt 0 ]; then
    echo "" | tee -a "$REPORT_FILE"
    apt list --upgradable 2>/dev/null | grep -i security | tee -a "$REPORT_FILE"
fi
echo "" | tee -a "$REPORT_FILE"

# Packages on hold
echo "=== Packages on Hold ===" | tee -a "$REPORT_FILE"
HELD=$(apt-mark showhold | wc -l)
echo "Packages held at current version: $HELD" | tee -a "$REPORT_FILE"

if [ $HELD -gt 0 ]; then
    apt-mark showhold | tee -a "$REPORT_FILE"
fi
echo "" | tee -a "$REPORT_FILE"

# Check for broken packages
echo "=== Broken Packages Check ===" | tee -a "$REPORT_FILE"
if dpkg --audit 2>&1 | grep -q "broken"; then
    echo "⚠️  WARNING: Broken packages detected!" | tee -a "$REPORT_FILE"
    dpkg --audit | tee -a "$REPORT_FILE"
else
    echo "✓ No broken packages found" | tee -a "$REPORT_FILE"
fi
echo "" | tee -a "$REPORT_FILE"

# Repository status
echo "=== Repository Status ===" | tee -a "$REPORT_FILE"
REPO_COUNT=$(grep -h "^deb " /etc/apt/sources.list /etc/apt/sources.list.d/* 2>/dev/null | wc -l)
echo "Active repositories: $REPO_COUNT" | tee -a "$REPORT_FILE"
echo "" | tee -a "$REPORT_FILE"

# Disk space used by packages
echo "=== Package Cache Status ===" | tee -a "$REPORT_FILE"
CACHE_SIZE=$(du -sh /var/cache/apt/archives 2>/dev/null | cut -f1)
echo "Package cache size: $CACHE_SIZE" | tee -a "$REPORT_FILE"
echo "" | tee -a "$REPORT_FILE"

echo "=== Report Complete ===" | tee -a "$REPORT_FILE"
echo "Report saved to: $REPORT_FILE"
EOF

chmod +x ~/package_lab/audit_packages.sh

# Run the audit:
~/package_lab/audit_packages.sh
```

**Explanation**:
- **apt vs dpkg**: `apt` is high-level (handles dependencies), `dpkg` is low-level (doesn't resolve dependencies)
- **Task 6**: `remove` leaves config files, `purge` deletes everything including configs
- **Task 8-9**: Modern method uses `/etc/apt/trusted.gpg.d/` instead of deprecated `apt-key`
- **Task 15**: Holding packages prevents accidental upgrades during `apt upgrade`
- **Task 17**: `--dry-run` or `-s` simulates without making changes
- **Task 18**: `apt install -f` fixes broken dependencies automatically

**Common Package Management Commands**:
```bash
# Search and information
apt search <term>              # Search for packages
apt show <package>             # Show package details
apt-cache policy <package>     # Show available versions
apt-cache depends <package>    # Show dependencies
apt-cache rdepends <package>   # Show reverse dependencies

# Installation and removal
apt install <package>          # Install package
apt remove <package>           # Remove but keep config
apt purge <package>            # Remove including config
apt autoremove                 # Remove unused dependencies
apt install --reinstall <pkg>  # Reinstall package

# Updates and upgrades
apt update                     # Update package index
apt upgrade                    # Upgrade packages (safe)
apt full-upgrade               # Upgrade with removals if needed
apt dist-upgrade               # Same as full-upgrade

# Package information
dpkg -l                        # List installed packages
dpkg -L <package>              # List files in package
dpkg -S /path/to/file          # Find package owning file
dpkg --get-selections          # Export package selections

# Package holds
apt-mark hold <package>        # Hold package version
apt-mark unhold <package>      # Unhold package
apt-mark showhold              # Show held packages

# Cleanup
apt autoclean                  # Remove old cached packages
apt clean                      # Remove all cached packages
apt autoremove                 # Remove unneeded packages

# Low-level dpkg
dpkg -i package.deb            # Install .deb file
dpkg -r <package>              # Remove package
dpkg -P <package>              # Purge package
dpkg --configure -a            # Configure unpacked packages
```

**Repository Management**:
```bash
# Add repository (modern method)
sudo add-apt-repository ppa:user/repo
sudo add-apt-repository --remove ppa:user/repo  # Remove

# Manual repository addition
echo "deb [arch=amd64] https://example.com/repo stable main" | \
    sudo tee /etc/apt/sources.list.d/example.list

# Add GPG key (modern method)
curl -fsSL https://example.com/key.gpg | \
    sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/example.gpg
```

</details>

### Extensions
1. **Advanced**: Create a local package repository mirror for offline installations. Configure multiple systems to use this local mirror and implement a synchronization strategy
2. **Challenge**: Build an automated package management system that: tracks installed packages across multiple servers, detects configuration drift, applies security updates automatically during maintenance windows, and rolls back if health checks fail

---

## Task 5: System Recovery and Boot Troubleshooting

### Learning Objectives
- Diagnose and repair boot failures using GRUB rescue mode
- Recover from filesystem corruption and disk errors
- Use single-user mode and emergency mode for system recovery
- Repair broken initramfs and kernel issues

### Context
Your production server failed to boot after a power outage. The GRUB bootloader shows errors, and you need to recover the system quickly. Additionally, another server is experiencing filesystem corruption that prevents normal startup. You have access to a live USB/rescue environment and need to diagnose the issues, repair the filesystem, fix GRUB configuration, and restore normal operations. This is a critical scenario where your troubleshooting skills and knowledge of the Linux boot process will be tested under pressure.

System recovery is an essential LFCS competency. Administrators must be able to recover systems that won't boot, fix filesystem issues, repair bootloaders, and understand the entire boot sequence from BIOS/UEFI through systemd initialization. These skills can mean the difference between a quick recovery and extended downtime.

### Task Instructions

**Prerequisites**: Set up recovery lab environment
```bash
# This lab requires a test VM - DO NOT run on production systems
# Create working directory for recovery scenarios
mkdir -p ~/recovery_lab/{backups,logs,scripts}
cd ~/recovery_lab

# Back up critical boot files (in case of practice mistakes)
sudo cp /boot/grub/grub.cfg ~/recovery_lab/backups/
sudo cp /etc/fstab ~/recovery_lab/backups/
sudo cp /etc/default/grub ~/recovery_lab/backups/
```

**Your Tasks**:

**Part A: Understanding the Boot Process**
1. Document the Linux boot sequence from power-on to login prompt
2. Identify all systemd boot targets and their purposes
3. Change default boot target to multi-user (non-graphical)
4. Boot into emergency mode and explore available recovery tools
5. Understand the differences between rescue mode, emergency mode, and single-user mode

**Part B: GRUB Bootloader Recovery**
6. List all GRUB menu entries and examine GRUB configuration
7. Edit GRUB boot parameters temporarily (during boot)
8. Modify GRUB default settings permanently
9. Simulate and recover from GRUB corruption
10. Reinstall GRUB to MBR/ESP partition

**Part C: Filesystem Recovery**
11. Check filesystem integrity with `fsck`
12. Repair filesystem corruption (simulated on test partition)
13. Recover from `/etc/fstab` misconfiguration that prevents boot
14. Mount root filesystem from rescue environment
15. Access and repair system when root password is forgotten

**Part D: Initramfs and Kernel Issues**
16. Examine initramfs contents and understand its purpose
17. Rebuild initramfs (initrd) for current kernel
18. Boot with an older kernel version
19. Remove problematic kernel and clean up boot partition
20. Create a comprehensive system recovery script

### Hints and Resources
1. Boot targets: `systemctl list-units --type=target`
2. GRUB config is in `/boot/grub/grub.cfg` (auto-generated) and `/etc/default/grub` (manual editing)
3. To enter emergency mode: Add `systemd.unit=emergency.target` to kernel parameters
4. `fsck` must run on unmounted filesystems
5. References: [systemd boot process](https://www.freedesktop.org/software/systemd/man/bootup.html), [GRUB Manual](https://www.gnu.org/software/grub/manual/)

### Estimated Time and Difficulty
**45-60 minutes, Advanced**

### Verification
To verify your solution:
- `systemctl get-default` shows the default boot target
- `grub-install --version` confirms GRUB is installed
- `lsinitramfs /boot/initrd.img-$(uname -r)` lists initramfs contents
- `dpkg --list | grep linux-image` shows installed kernels
- System boots successfully to the expected target
- `journalctl -b` shows boot logs without critical errors
- `dmesg | grep -i error` shows no filesystem errors

<details>
<summary>Solution</summary>

```bash
# Part A: Understanding the Boot Process

# Task 1: Document boot sequence
cat > ~/recovery_lab/boot_sequence.md << 'EOF'
# Linux Boot Sequence (UEFI/systemd)

1. **BIOS/UEFI**: Hardware initialization, POST checks
2. **Bootloader (GRUB2)**: Loads kernel and initramfs
3. **Kernel**: Initializes hardware, mounts initramfs as root
4. **initramfs**: Contains drivers and tools for mounting real root filesystem
5. **systemd (PID 1)**: First user-space process, manages system initialization
6. **systemd targets**: Similar to runlevels
   - sysinit.target: Basic system initialization
   - basic.target: Basic system services
   - multi-user.target: Multi-user text mode
   - graphical.target: Graphical login
7. **Getty/Login**: Login prompts (text or graphical)
8. **User Session**: User environment starts

Key files:
- /boot/grub/grub.cfg: GRUB configuration
- /boot/vmlinuz-*: Kernel image
- /boot/initrd.img-*: Initial RAM filesystem
- /etc/fstab: Filesystem mount configuration
- /etc/systemd/system/default.target: Default boot target
EOF

cat ~/recovery_lab/boot_sequence.md

# Task 2: Identify systemd targets
systemctl list-units --type=target

# Show target dependencies:
systemctl list-dependencies graphical.target

# Show all available targets:
systemctl list-units --type=target --all

# Key targets:
# - poweroff.target
# - rescue.target (single-user, minimal services)
# - multi-user.target (text mode, multi-user)
# - graphical.target (GUI)
# - reboot.target
# - emergency.target (minimal shell, read-only root)

# Task 3: Change default boot target
# Check current default:
systemctl get-default

# Change to multi-user (text mode):
sudo systemctl set-default multi-user.target

# Verify:
systemctl get-default
ls -la /etc/systemd/system/default.target

# Change back to graphical if needed:
sudo systemctl set-default graphical.target

# Task 4: Boot into emergency mode
# Create script to document how to enter emergency mode:
cat > ~/recovery_lab/emergency_mode_guide.sh << 'EOF'
#!/bin/bash

echo "=== Emergency Mode Access Guide ==="
echo ""
echo "Method 1: From GRUB menu"
echo "  1. Reboot and press 'e' at GRUB menu"
echo "  2. Find line starting with 'linux'"
echo "  3. Add: systemd.unit=emergency.target"
echo "  4. Press Ctrl+X to boot"
echo ""
echo "Method 2: From running system"
echo "  sudo systemctl emergency"
echo ""
echo "In emergency mode, you'll have:"
echo "  - Minimal environment"
echo "  - Root filesystem mounted read-only"
echo "  - No network, minimal services"
echo "  - Root shell access"
echo ""
echo "Available recovery tools in emergency mode:"
which fsck e2fsck mount systemctl journalctl
echo ""
echo "To make root writable:"
echo "  mount -o remount,rw /"
echo ""
echo "To continue normal boot:"
echo "  systemctl default"
EOF

chmod +x ~/recovery_lab/emergency_mode_guide.sh
~/recovery_lab/emergency_mode_guide.sh

# Task 5: Differences between recovery modes
cat > ~/recovery_lab/recovery_modes.md << 'EOF'
# Recovery Mode Comparison

| Mode | Init Level | Root FS | Network | Services | Use Case |
|------|-----------|---------|---------|----------|----------|
| **Emergency** | Minimal | RO | No | Minimal | Critical repairs, fsck |
| **Rescue** | More | RW | Maybe | Basic | System recovery, password reset |
| **Single-User** | Single | RW | No | Minimal | Maintenance, recovery |
| **Multi-User** | Full | RW | Yes | Most | Normal text mode |

**Emergency Mode** (systemd.unit=emergency.target):
- Bare minimum environment
- Root filesystem mounted read-only
- Almost no services running
- Use for: filesystem repairs, critical fixes

**Rescue Mode** (systemd.unit=rescue.target):
- More services than emergency
- Root filesystem mounted read-write
- Basic system initialization
- Use for: password recovery, service repairs

**Single-User Mode** (single or 1):
- Legacy concept (pre-systemd)
- Now usually maps to rescue.target
- Root access without network

**Access Methods**:
- Add to kernel line: systemd.unit=emergency.target
- Or: systemd.unit=rescue.target
- Or: single or 1 (older method)
EOF

cat ~/recovery_lab/recovery_modes.md

# Part B: GRUB Bootloader Recovery

# Task 6: List GRUB entries and examine configuration
# View GRUB configuration:
sudo cat /boot/grub/grub.cfg | less

# Show just menu entries:
grep "menuentry" /boot/grub/grub.cfg

# View user-editable GRUB settings:
cat /etc/default/grub

# Task 7: Edit GRUB parameters temporarily (at boot time)
cat > ~/recovery_lab/grub_temporary_edit.md << 'EOF'
# Temporary GRUB Parameter Editing

At boot time (changes lost on reboot):

1. **Reboot and access GRUB menu**:
   - Hold Shift during boot (BIOS)
   - Or press Esc repeatedly (UEFI)

2. **Edit boot entry**:
   - Select entry and press 'e'

3. **Find linux line**:
   ```
   linux /boot/vmlinuz-5.15.0-56-generic root=UUID=xxx ro quiet splash
   ```

4. **Common modifications**:
   - Remove 'quiet splash' to see boot messages
   - Add 'systemd.unit=rescue.target' for rescue mode
   - Add 'systemd.unit=emergency.target' for emergency mode
   - Add 'single' for single-user mode
   - Add 'init=/bin/bash' for root shell (dangerous!)
   - Add '3' for text mode (legacy)

5. **Boot with changes**:
   - Press Ctrl+X or F10

**Example** (remove quiet mode):
```
linux /boot/vmlinuz-5.15.0-56-generic root=UUID=xxx ro
```
EOF

cat ~/recovery_lab/grub_temporary_edit.md

# Task 8: Modify GRUB settings permanently
# Edit GRUB defaults:
sudo cp /etc/default/grub /etc/default/grub.backup

# View current settings:
cat /etc/default/grub

# Example modifications (edit with your preferred editor):
cat > ~/recovery_lab/grub_modifications.sh << 'EOF'
#!/bin/bash

# Common GRUB customizations
echo "=== Common GRUB Modifications ==="

# 1. Remove quiet splash to see boot messages
# GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"
# Change to:
# GRUB_CMDLINE_LINUX_DEFAULT=""

# 2. Decrease timeout
# GRUB_TIMEOUT=10
# Change to:
# GRUB_TIMEOUT=3

# 3. Remember last booted entry
# Add:
# GRUB_DEFAULT=saved
# GRUB_SAVEDEFAULT=true

# 4. Add custom kernel parameters
# GRUB_CMDLINE_LINUX="net.ifnames=0 biosdevname=0"

# After editing, update GRUB:
# sudo update-grub
# Or:
# sudo grub-mkconfig -o /boot/grub/grub.cfg
EOF

chmod +x ~/recovery_lab/grub_modifications.sh
cat ~/recovery_lab/grub_modifications.sh

# Make a test change (remove quiet splash):
sudo sed -i.bak 's/GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"/GRUB_CMDLINE_LINUX_DEFAULT=""/' /etc/default/grub

# Update GRUB configuration:
sudo update-grub

# Verify change:
grep CMDLINE /etc/default/grub

# Restore original (for safety):
sudo cp /etc/default/grub.backup /etc/default/grub
sudo update-grub

# Task 9-10: GRUB recovery and reinstallation
cat > ~/recovery_lab/grub_recovery.sh << 'EOF'
#!/bin/bash

echo "=== GRUB Recovery Procedures ==="
echo ""

echo "** Scenario: GRUB is corrupted or missing **"
echo ""

echo "Method 1: Reinstall GRUB (from live system)"
echo "# Boot from live USB/CD"
echo "# Identify your root partition:"
echo "sudo fdisk -l"
echo "lsblk"
echo ""
echo "# Mount root partition:"
echo "sudo mount /dev/sdXY /mnt"
echo ""
echo "# Mount required filesystems:"
echo "sudo mount --bind /dev /mnt/dev"
echo "sudo mount --bind /dev/pts /mnt/dev/pts"
echo "sudo mount --bind /proc /mnt/proc"
echo "sudo mount --bind /sys /mnt/sys"
echo ""
echo "# If UEFI, mount ESP:"
echo "sudo mount /dev/sdXZ /mnt/boot/efi  # Usually /dev/sda1"
echo ""
echo "# Chroot into system:"
echo "sudo chroot /mnt"
echo ""
echo "# Reinstall GRUB:"
echo "# For BIOS systems:"
echo "grub-install /dev/sda  # Install to disk, not partition"
echo "update-grub"
echo ""
echo "# For UEFI systems:"
echo "grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=ubuntu"
echo "update-grub"
echo ""
echo "# Exit and reboot:"
echo "exit"
echo "sudo umount -R /mnt"
echo "sudo reboot"
echo ""

echo "Method 2: Repair GRUB from GRUB rescue prompt"
echo "If you see 'grub rescue>' prompt:"
echo ""
echo "grub rescue> ls"
echo "# Look for your root partition, test with:"
echo "grub rescue> ls (hd0,msdos1)/"
echo "# Find the one with /boot/grub"
echo ""
echo "grub rescue> set root=(hd0,msdos1)"
echo "grub rescue> set prefix=(hd0,msdos1)/boot/grub"
echo "grub rescue> insmod normal"
echo "grub rescue> normal"
echo ""
echo "# Once booted, reinstall GRUB:"
echo "sudo grub-install /dev/sda"
echo "sudo update-grub"
EOF

chmod +x ~/recovery_lab/grub_recovery.sh
cat ~/recovery_lab/grub_recovery.sh

# Part C: Filesystem Recovery

# Task 11-12: Check and repair filesystem
# NOTE: NEVER run fsck on mounted filesystem!

cat > ~/recovery_lab/filesystem_check.sh << 'EOF'
#!/bin/bash

echo "=== Filesystem Check and Repair ==="
echo ""
echo "⚠️  WARNING: Only run fsck on UNMOUNTED filesystems!"
echo ""

echo "Step 1: Identify filesystems"
lsblk -f
echo ""

echo "Step 2: Check if filesystem is mounted"
mount | grep sda

echo ""
echo "Step 3: Filesystem check commands"
echo ""

echo "For ext4 filesystems:"
echo "  sudo fsck.ext4 -f /dev/sdXY      # Force check"
echo "  sudo fsck.ext4 -p /dev/sdXY      # Auto repair"
echo "  sudo fsck.ext4 -n /dev/sdXY      # Dry run (no changes)"
echo "  sudo e2fsck -f -y -v /dev/sdXY   # Verbose auto-repair"
echo ""

echo "For other filesystems:"
echo "  sudo fsck.xfs /dev/sdXY          # XFS"
echo "  sudo fsck.vfat /dev/sdXY         # FAT32"
echo ""

echo "Generic fsck (auto-detects type):"
echo "  sudo fsck -A                     # Check all in /etc/fstab"
echo "  sudo fsck -AR                    # Skip root filesystem"
echo "  sudo fsck /dev/sdXY             # Check specific partition"
echo ""

echo "Step 4: Boot-time filesystem check"
echo "Force fsck on next boot:"
echo "  sudo touch /forcefsck"
echo ""
echo "Or set max mount count:"
echo "  sudo tune2fs -c 1 /dev/sdXY      # Check after 1 mount"
echo "  sudo tune2fs -c 30 /dev/sdXY     # Check after 30 mounts (default)"
echo ""

echo "Check filesystem health:"
tune2fs -l /dev/sda1 2>/dev/null | grep -i "mount count\|check\|state" || echo "Run on actual partition"
EOF

chmod +x ~/recovery_lab/filesystem_check.sh
~/recovery_lab/filesystem_check.sh

# Task 13: Recover from /etc/fstab misconfiguration
cat > ~/recovery_lab/fstab_recovery.sh << 'EOF'
#!/bin/bash

echo "=== /etc/fstab Recovery ==="
echo ""

echo "Symptom: System won't boot, drops to emergency mode"
echo "Error: 'Failed to mount /dev/sdXY' or 'UUID not found'"
echo ""

echo "Recovery Steps:"
echo ""
echo "1. Boot into emergency mode"
echo "   - Add to kernel line: systemd.unit=emergency.target"
echo ""
echo "2. Remount root as read-write:"
echo "   mount -o remount,rw /"
echo ""
echo "3. Check /etc/fstab for errors:"
echo "   cat /etc/fstab"
echo ""
echo "Common fstab errors:"
echo "   - Wrong UUID"
echo "   - Wrong device name"
echo "   - Missing filesystem type"
echo "   - Invalid mount options"
echo "   - Filesystem doesn't exist"
echo ""
echo "4. Find correct UUIDs:"
echo "   blkid"
echo "   lsblk -f"
echo ""
echo "5. Edit /etc/fstab:"
echo "   nano /etc/fstab"
echo "   # Comment out problematic line with #"
echo "   # Or fix UUID/device name"
echo ""
echo "6. Test the configuration:"
echo "   mount -a    # Try mounting all"
echo "   mount -fav  # Fake verbose (test without actually mounting)"
echo ""
echo "7. Reboot:"
echo "   systemctl reboot"

echo ""
echo "Current /etc/fstab:"
cat /etc/fstab

echo ""
echo "Current UUIDs:"
blkid
EOF

chmod +x ~/recovery_lab/fstab_recovery.sh
~/recovery_lab/fstab_recovery.sh

# Task 14: Mount root from rescue environment
cat > ~/recovery_lab/mount_from_rescue.sh << 'EOF'
#!/bin/bash

cat << 'RESCUE'
=== Mounting Root Filesystem from Rescue/Live Environment ===

Scenario: System won't boot, need to access files and repair

Step 1: Boot from Live USB/CD

Step 2: Identify root partition
$ sudo fdisk -l
$ lsblk
$ sudo blkid

Step 3: Create mount point and mount root
$ sudo mkdir /mnt/rescue
$ sudo mount /dev/sda2 /mnt/rescue  # Replace with your root partition

# If root is encrypted (LUKS):
$ sudo cryptsetup luksOpen /dev/sda2 cryptroot
$ sudo mount /dev/mapper/cryptroot /mnt/rescue

# If LVM:
$ sudo vgchange -ay
$ sudo mount /dev/vg0/root /mnt/rescue

Step 4: Mount other partitions
$ sudo mount /dev/sda1 /mnt/rescue/boot  # If separate /boot
$ sudo mount /dev/sda3 /mnt/rescue/home  # If separate /home

# For UEFI systems:
$ sudo mount /dev/sda1 /mnt/rescue/boot/efi

Step 5: Mount system filesystems for chroot
$ sudo mount --bind /dev /mnt/rescue/dev
$ sudo mount --bind /dev/pts /mnt/rescue/dev/pts
$ sudo mount --bind /proc /mnt/rescue/proc
$ sudo mount --bind /sys /mnt/rescue/sys
$ sudo mount --bind /run /mnt/rescue/run

Step 6: Chroot into system
$ sudo chroot /mnt/rescue

# Now you're "inside" your installed system
# Can run commands as if booted normally:
$ apt update
$ apt install --reinstall package
$ update-grub
$ systemctl enable service

Step 7: Exit and cleanup
$ exit
$ sudo umount -R /mnt/rescue  # Recursive unmount
$ sudo reboot

Common repairs from chroot:
- Fix broken packages: apt install -f
- Reinstall GRUB: grub-install /dev/sda && update-grub
- Reset passwords: passwd username
- Fix fstab: nano /etc/fstab
- Rebuild initramfs: update-initramfs -u
RESCUE
EOF

chmod +x ~/recovery_lab/mount_from_rescue.sh
~/recovery_lab/mount_from_rescue.sh

# Task 15: Reset root password
cat > ~/recovery_lab/reset_root_password.sh << 'EOF'
#!/bin/bash

cat << 'RESET'
=== Reset Root Password ===

Method 1: Using GRUB (quickest)

1. Reboot and press 'e' at GRUB menu
2. Find line starting with 'linux'
3. Change:
   ro quiet splash
   To:
   rw init=/bin/bash

4. Press Ctrl+X to boot
5. You'll get a root shell:
   # mount -o remount,rw /
   # passwd root
   # passwd username  # Reset user password
   # sync
   # exec /sbin/init  # Continue boot

Method 2: Using single-user mode

1. At GRUB, press 'e'
2. Add to linux line: single
3. Boot with Ctrl+X
4. You may be prompted for root password
5. If you get in:
   # passwd username
   # reboot

Method 3: From rescue/live environment

1. Boot from Live USB
2. Mount root partition:
   $ sudo mount /dev/sda2 /mnt
3. Chroot:
   $ sudo mount --bind /dev /mnt/dev
   $ sudo mount --bind /proc /mnt/proc
   $ sudo mount --bind /sys /mnt/sys
   $ sudo chroot /mnt
4. Reset password:
   # passwd root
   # passwd username
5. Exit and reboot:
   # exit
   $ sudo umount -R /mnt
   $ sudo reboot
RESET
EOF

chmod +x ~/recovery_lab/reset_root_password.sh
~/recovery_lab/reset_root_password.sh

# Part D: Initramfs and Kernel Issues

# Task 16: Examine initramfs
echo "=== Examining initramfs ==="

# List initramfs files:
ls -lh /boot/initrd.img-*

# Show contents of current initramfs:
lsinitramfs /boot/initrd.img-$(uname -r) | less

# Count files in initramfs:
lsinitramfs /boot/initrd.img-$(uname -r) | wc -l

# Find specific files:
lsinitramfs /boot/initrd.img-$(uname -r) | grep -i "fsck\|ext4"

# Extract initramfs (for detailed inspection):
mkdir -p ~/recovery_lab/initramfs_contents
cd ~/recovery_lab/initramfs_contents
unmkinitramfs /boot/initrd.img-$(uname -r) .

echo "Initramfs structure:"
tree -L 2 . 2>/dev/null || find . -maxdepth 2 -type d

# Task 17: Rebuild initramfs
cat > ~/recovery_lab/rebuild_initramfs.sh << 'EOF'
#!/bin/bash

echo "=== Rebuild initramfs (initrd) ==="
echo ""

echo "Current kernel:"
uname -r
echo ""

echo "Installed kernels:"
ls -1 /boot/vmlinuz-* | sed 's/\/boot\/vmlinuz-//'
echo ""

echo "Method 1: Rebuild for current kernel"
echo "  sudo update-initramfs -u"
echo ""

echo "Method 2: Rebuild for specific kernel"
echo "  sudo update-initramfs -u -k 5.15.0-56-generic"
echo ""

echo "Method 3: Rebuild all initramfs files"
echo "  sudo update-initramfs -u -k all"
echo ""

echo "Method 4: Create new initramfs"
echo "  sudo update-initramfs -c -k $(uname -r)"
echo ""

echo "Verbose rebuild:"
echo "  sudo update-initramfs -u -v"
echo ""

echo "After rebuilding, verify:"
echo "  ls -lh /boot/initrd.img-$(uname -r)"
echo "  lsinitramfs /boot/initrd.img-$(uname -r) | head"
EOF

chmod +x ~/recovery_lab/rebuild_initramfs.sh
~/recovery_lab/rebuild_initramfs.sh

# Task 18: Boot with older kernel
cat > ~/recovery_lab/boot_older_kernel.md << 'EOF'
# Boot with Older Kernel

Scenario: New kernel doesn't work, need to boot older version

Method 1: From GRUB menu
1. Reboot
2. Press Shift or Esc to show GRUB menu
3. Select "Advanced options for Ubuntu"
4. Choose older kernel version
5. Boot normally

Method 2: Set older kernel as default
1. List installed kernels:
   dpkg --list | grep linux-image

2. Check current GRUB entries:
   grep menuentry /boot/grub/grub.cfg

3. Edit GRUB default (Ubuntu):
   sudo nano /etc/default/grub
   
   Change:
   GRUB_DEFAULT=0
   
   To (example for 3rd submenu, 2nd entry):
   GRUB_DEFAULT="1>2"
   
   Or use saved:
   GRUB_DEFAULT=saved

4. Update GRUB:
   sudo update-grub

5. Set specific kernel to boot (with saved method):
   sudo grub-set-default "1>2"
   sudo grub-reboot "1>2"  # Just for next boot

6. Reboot and verify:
   uname -r
EOF

cat ~/recovery_lab/boot_older_kernel.md

# Task 19: Remove old kernels
cat > ~/recovery_lab/kernel_cleanup.sh << 'EOF'
#!/bin/bash

echo "=== Kernel Cleanup ==="
echo ""

echo "Current kernel:"
uname -r
echo ""

echo "Installed kernels:"
dpkg --list | grep linux-image | grep "^ii"
echo ""

echo "Disk usage in /boot:"
df -h /boot
echo ""

echo "Files in /boot:"
ls -lh /boot/ | grep -E "vmlinuz|initrd"
echo ""

echo "⚠️  DO NOT remove current kernel: $(uname -r)"
echo ""

echo "Method 1: Remove specific old kernel"
echo "  sudo apt remove linux-image-5.15.0-XX-generic"
echo "  sudo apt remove linux-headers-5.15.0-XX-generic"
echo ""

echo "Method 2: Auto-remove old kernels"
echo "  sudo apt autoremove"
echo "  sudo apt autoremove --purge"
echo ""

echo "Method 3: Keep only current and one previous"
echo "  sudo apt install byobu"  # Includes purge-old-kernels
echo "  sudo purge-old-kernels --keep 2 -qy"
echo ""

echo "After removal:"
echo "  sudo update-grub"
echo "  df -h /boot"
EOF

chmod +x ~/recovery_lab/kernel_cleanup.sh
~/recovery_lab/kernel_cleanup.sh

# Task 20: Comprehensive recovery script
cat > ~/recovery_lab/system_recovery_toolkit.sh << 'EOF'
#!/bin/bash

# System Recovery Toolkit
# Comprehensive diagnostics and recovery procedures

set -euo pipefail

REPORT_DIR="$HOME/recovery_lab/recovery_reports"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
REPORT_FILE="$REPORT_DIR/recovery_report_$TIMESTAMP.txt"

mkdir -p "$REPORT_DIR"

log() {
    echo "$@" | tee -a "$REPORT_FILE"
}

section() {
    log ""
    log "========================================"
    log "$@"
    log "========================================"
}

# Start recovery report
section "SYSTEM RECOVERY DIAGNOSTIC REPORT"
log "Generated: $(date)"
log "Hostname: $(hostname)"
log "Kernel: $(uname -r)"
log ""

# Boot status
section "BOOT STATUS"
log "Current boot target:"
systemctl get-default | tee -a "$REPORT_FILE"

log ""
log "Failed units:"
systemctl --failed | tee -a "$REPORT_FILE"

log ""
log "Boot time:"
systemd-analyze | tee -a "$REPORT_FILE"

# Filesystem status
section "FILESYSTEM STATUS"
log "Mounted filesystems:"
df -h | tee -a "$REPORT_FILE"

log ""
log "Filesystem errors in dmesg:"
dmesg | grep -i "error\|failed\|corrupt" | tail -20 | tee -a "$REPORT_FILE" || log "No errors found"

log ""
log "/etc/fstab configuration:"
cat /etc/fstab | tee -a "$REPORT_FILE"

log ""
log "Filesystem check status:"
for fs in $(lsblk -ln -o NAME,TYPE | grep part | awk '{print"/dev/"$1}'); do
    tune2fs -l "$fs" 2>/dev/null | grep -i "state\|check\|mount count" | head -5 || echo "$fs: not ext filesystem"
done | tee -a "$REPORT_FILE"

# GRUB status
section "GRUB BOOTLOADER STATUS"
log "GRUB version:"
grub-install --version | tee -a "$REPORT_FILE"

log ""
log "GRUB menu entries:"
grep "^menuentry" /boot/grub/grub.cfg | head -10 | tee -a "$REPORT_FILE"

log ""
log "GRUB configuration (/etc/default/grub):"
cat /etc/default/grub | grep -v "^#" | grep -v "^$" | tee -a "$REPORT_FILE"

# Kernel status
section "KERNEL STATUS"
log "Installed kernels:"
dpkg --list | grep linux-image | grep "^ii" | tee -a "$REPORT_FILE"

log ""
log "Boot partition usage:"
df -h /boot | tee -a "$REPORT_FILE"

log ""
log "Kernel modules loaded:"
lsmod | wc -l | xargs echo "Total modules:" | tee -a "$REPORT_FILE"

log ""
log "Kernel parameters:"
cat /proc/cmdline | tee -a "$REPORT_FILE"

# Initramfs status
section "INITRAMFS STATUS"
log "Initramfs files:"
ls -lh /boot/initrd.img-* | tee -a "$REPORT_FILE"

log ""
log "Current initramfs size:"
ls -lh /boot/initrd.img-$(uname -r) | tee -a "$REPORT_FILE"

# System logs
section "RECENT SYSTEM ERRORS"
log "Journal errors (last boot):"
journalctl -b -p err --no-pager | tail -20 | tee -a "$REPORT_FILE"

log ""
log "Critical dmesg messages:"
dmesg -l err,crit,alert,emerg | tail -20 | tee -a "$REPORT_FILE" || log "No critical messages"

# Service status
section "CRITICAL SERVICES STATUS"
for service in ssh systemd-logind NetworkManager systemd-resolved; do
    log ""
    log "Service: $service"
    systemctl status $service --no-pager | head -10 | tee -a "$REPORT_FILE" || log "  Not found"
done

# Recovery recommendations
section "RECOVERY RECOMMENDATIONS"

# Check for common issues
if systemctl --failed | grep -q "failed"; then
    log "⚠️  Found failed services. Run: systemctl --failed"
fi

if df -h /boot | tail -1 | awk '{print $5}' | sed 's/%//' | grep -qE '^[89][0-9]$|^100$'; then
    log "⚠️  /boot partition is >80% full. Clean old kernels with: sudo apt autoremove"
fi

if dmesg | grep -qi "filesystem.*error"; then
    log "⚠️  Filesystem errors detected. Boot into emergency mode and run: fsck"
fi

if ! systemctl is-enabled sshd >/dev/null 2>&1 && ! systemctl is-enabled ssh >/dev/null 2>&1; then
    log "ℹ️  SSH service not enabled. Enable with: sudo systemctl enable ssh"
fi

# Quick recovery commands
section "QUICK RECOVERY COMMANDS"
cat << 'RECOVERY' | tee -a "$REPORT_FILE"

Emergency Boot:
  - Press 'e' at GRUB menu
  - Add: systemd.unit=emergency.target
  - Press Ctrl+X

Reset Root Password:
  - Add to kernel line: rw init=/bin/bash
  - Boot and run: passwd root

Fix Broken Packages:
  sudo apt install -f
  sudo dpkg --configure -a

Rebuild initramfs:
  sudo update-initramfs -u

Reinstall GRUB:
  sudo grub-install /dev/sda
  sudo update-grub

Check Filesystem:
  sudo fsck -A -R  # All except root
  # Or boot to emergency mode:
  # mount -o remount,rw /
  # fsck.ext4 -f -y /dev/sdXY

Fix fstab:
  - Boot to emergency mode
  - mount -o remount,rw /
  - nano /etc/fstab
  - Comment out problematic lines
  - mount -a (test)
  - reboot

RECOVERY

section "REPORT COMPLETE"
log "Full report saved to: $REPORT_FILE"
log ""
log "Next steps:"
log "1. Review failed services and errors"
log "2. Address filesystem issues if any"
log "3. Clean up /boot if necessary"
log "4. Test recovery procedures in emergency mode if needed"
EOF

chmod +x ~/recovery_lab/system_recovery_toolkit.sh

# Run the recovery diagnostic:
echo ""
echo "Running system recovery diagnostic..."
~/recovery_lab/system_recovery_toolkit.sh
```

**Explanation**:
- **Emergency vs Rescue mode**: Emergency has minimal services and read-only root; rescue has more services and read-write root
- **Task 7**: Temporary GRUB edits (press 'e' at boot) are lost after reboot; useful for testing or one-time recovery
- **Task 13**: Common `fstab` errors include wrong UUIDs (devices changed), missing filesystems, or invalid mount options
- **Task 15**: `init=/bin/bash` gives root shell but bypasses systemd; use with caution
- **Task 16**: initramfs contains drivers and tools needed before the real root filesystem can be mounted
- **GRUB installation**: Install to disk (`/dev/sda`) not partition (`/dev/sda1`)

**Critical Recovery Procedures Summary**:

1. **Can't boot - GRUB error**:
   - Boot from live USB → chroot → grub-install → update-grub

2. **Filesystem corruption**:
   - Boot to emergency mode → remount root rw → fsck on problematic partition

3. **Forgot root password**:
   - Add `rw init=/bin/bash` to kernel line → passwd root

4. **Wrong fstab entry**:
   - Boot to emergency mode → remount rw → edit fstab → comment out bad line

5. **Kernel won't boot**:
   - Select older kernel from GRUB advanced options

**Boot Process Troubleshooting Flowchart**:
```
System won't boot
├─ No GRUB menu → Reinstall GRUB
├─ GRUB rescue prompt → Fix GRUB from rescue shell
├─ Kernel panic → Boot older kernel
├─ Emergency mode (filesystem errors) → Run fsck
├─ Drops to emergency (fstab issue) → Fix fstab
└─ Services fail to start → Check journalctl -b
```

</details>

### Extensions
1. **Advanced**: Set up automated boot monitoring that: detects failed boots, automatically boots into rescue mode after 3 failures, sends alerts, and maintains a boot history log
2. **Challenge**: Create a complete disaster recovery procedure including: full system backup strategy, documentation for bare-metal recovery from backups, automated testing of recovery procedures in VMs, and runbooks for common failure scenarios

---

## Task 6: Virtual Machine Management with KVM/QEMU

### Learning Objectives
- Install and configure KVM/QEMU virtualization on Ubuntu
- Create and manage virtual machines using command-line tools (`virsh`)
- Configure virtual networks and storage for VMs
- Manage VM snapshots, clones, and resource allocation

### Context
Your organization is migrating development and testing workloads to virtualized infrastructure. You need to set up a KVM hypervisor on Ubuntu 22.04, create several virtual machines for different purposes (web server, database, testing environment), configure virtual networking with isolated networks and bridging, manage storage efficiently, and implement snapshot strategies for quick recovery. The infrastructure must support both Linux and potentially Windows VMs, with proper resource allocation and monitoring. You'll need to automate VM deployment and ensure that VMs can communicate appropriately while maintaining security isolation.

Virtualization with KVM is a key LFCS competency. Understanding how to manage virtual machines from the command line using libvirt/virsh is essential for modern Linux administration. This includes VM lifecycle management, resource allocation, networking, and storage configuration.

### Task Instructions

**Prerequisites**: Set up KVM environment
```bash
# Check if your CPU supports virtualization
egrep -c '(vmx|svm)' /proc/cpuinfo
# Should return > 0

# Check if KVM is available
lsmod | grep kvm
# Should see kvm_intel or kvm_amd

# Create working directory
mkdir -p ~/vm_lab/{isos,images,scripts}
cd ~/vm_lab
```

**Your Tasks**:

**Part A: KVM Installation and Configuration**
1. Verify hardware virtualization support (VT-x/AMD-V)
2. Install KVM, QEMU, and libvirt packages
3. Configure and start libvirt service
4. Add your user to appropriate groups for VM management
5. Verify KVM installation with diagnostic tools

**Part B: Virtual Machine Creation**
6. Download an Ubuntu Server cloud image for quick VM deployment
7. Create a virtual machine using `virt-install`
8. Create a VM from an ISO image
9. List all VMs and show their states
10. Configure VM to auto-start on host boot

**Part C: VM Management Operations**
11. Start, stop, and restart VMs
12. Connect to VM console
13. Modify VM resources (CPU, memory) while powered off
14. Clone an existing VM
15. Take and restore VM snapshots

**Part D: Networking and Storage**
16. List available virtual networks
17. Create a custom isolated virtual network
18. Attach VM to multiple networks
19. Create and attach additional virtual disks to a VM
20. Implement a VM backup strategy using snapshots and exports

### Hints and Resources
1. Main commands: `virsh`, `virt-install`, `virt-clone`, `virt-viewer`
2. VM definitions stored in XML at `/etc/libvirt/qemu/`
3. Default storage pool location: `/var/lib/libvirt/images/`
4. Default network: `virbr0` (NAT network 192.168.122.0/24)
5. References: [KVM Documentation](https://www.linux-kvm.org/), [libvirt virsh command reference](https://libvirt.org/manpages/virsh.html)

### Estimated Time and Difficulty
**40-55 minutes, Intermediate to Advanced**

### Verification
To verify your solution:
- `virsh list --all` shows all VMs and their states
- `virsh net-list --all` shows all virtual networks
- `virsh pool-list --all` shows storage pools
- `systemctl status libvirtd` shows libvirt service is running
- `virsh dominfo VM-name` displays VM configuration
- `virsh net-dhcp-leases default` shows IP addresses assigned to VMs
- VM should be accessible via SSH or console

<details>
<summary>Solution</summary>

```bash
# Part A: KVM Installation and Configuration

# Task 1: Verify hardware virtualization support
echo "=== Checking Virtualization Support ==="

# Check CPU flags for virtualization:
egrep -c '(vmx|svm)' /proc/cpuinfo
# If returns 0, virtualization not supported or not enabled in BIOS

# More detailed check:
egrep --color '(vmx|svm)' /proc/cpuinfo

# Check if it's Intel (vmx) or AMD (svm):
if grep -q vmx /proc/cpuinfo; then
    echo "Intel VT-x detected"
elif grep -q svm /proc/cpuinfo; then
    echo "AMD-V detected"
else
    echo "No virtualization support found"
fi

# Verify using kvm-ok (if available):
sudo apt install -y cpu-checker
kvm-ok
# Expected: "KVM acceleration can be used"

# Task 2: Install KVM, QEMU, and libvirt
echo "=== Installing KVM and Tools ==="

# Update package index:
sudo apt update

# Install KVM and essential packages:
sudo apt install -y qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils virtinst virt-manager

# Additional useful tools:
sudo apt install -y libvirt-daemon virt-viewer libguestfs-tools

# Verify installation:
which virsh
which virt-install
qemu-system-x86_64 --version

# Task 3: Configure and start libvirt
echo "=== Configuring libvirt ==="

# Check libvirt service status:
systemctl status libvirtd

# Start and enable libvirt:
sudo systemctl enable --now libvirtd
sudo systemctl status libvirtd

# Verify libvirt is running:
sudo systemctl is-active libvirtd

# Task 4: Add user to groups
echo "=== Configuring User Permissions ==="

# Add current user to libvirt and kvm groups:
sudo usermod -aG libvirt $USER
sudo usermod -aG kvm $USER

# Verify group membership:
groups $USER

# Note: Log out and log back in for group changes to take effect
# Or use: newgrp libvirt

echo "⚠️  Log out and back in for group changes to take effect"

# Task 5: Verify KVM installation
cat > ~/vm_lab/verify_kvm.sh << 'EOF'
#!/bin/bash

echo "=== KVM Installation Verification ==="
echo ""

echo "1. CPU Virtualization Support:"
if egrep -q '(vmx|svm)' /proc/cpuinfo; then
    echo "   ✓ CPU supports virtualization"
    egrep '(vmx|svm)' /proc/cpuinfo | wc -l | xargs echo "   CPU cores:"
else
    echo "   ✗ No virtualization support"
fi
echo ""

echo "2. KVM Kernel Modules:"
if lsmod | grep -q kvm; then
    echo "   ✓ KVM modules loaded:"
    lsmod | grep kvm
else
    echo "   ✗ KVM modules not loaded"
fi
echo ""

echo "3. libvirt Service:"
if systemctl is-active --quiet libvirtd; then
    echo "   ✓ libvirtd is running"
else
    echo "   ✗ libvirtd is not running"
fi
echo ""

echo "4. Virtual Networks:"
virsh net-list --all
echo ""

echo "5. Storage Pools:"
virsh pool-list --all
echo ""

echo "6. Existing VMs:"
virsh list --all
echo ""

echo "7. User Groups:"
groups | grep -E '(libvirt|kvm)' && echo "   ✓ User in virtualization groups" || echo "   ⚠️  User not in groups (logout required)"
echo ""

echo "Installation verification complete!"
EOF

chmod +x ~/vm_lab/verify_kvm.sh
~/vm_lab/verify_kvm.sh

# Part B: Virtual Machine Creation

# Task 6: Download Ubuntu cloud image
echo "=== Downloading Cloud Image ==="

cd ~/vm_lab/images

# Download Ubuntu 22.04 cloud image (small and fast for testing):
wget https://cloud-images.ubuntu.com/jammy/current/jammy-server-cloudimg-amd64.img

# Verify download:
ls -lh jammy-server-cloudimg-amd64.img

# Check image info:
qemu-img info jammy-server-cloudimg-amd64.img

# Task 7: Create VM using cloud image
echo "=== Creating VM from Cloud Image ==="

# Create cloud-init configuration for automatic setup:
cat > ~/vm_lab/user-data << 'EOF'
#cloud-config
users:
  - name: ubuntu
    sudo: ALL=(ALL) NOPASSWD:ALL
    groups: sudo
    shell: /bin/bash
    ssh_authorized_keys:
      - ssh-rsa AAAAB3... # Add your SSH public key here
    lock_passwd: false
    passwd: $6$rounds=4096$salt$hashed_password # Or generate with: openssl passwd -6
# For testing, you can use simple password:
password: ubuntu
chpasswd:
  expire: false
ssh_pwauth: true
EOF

cat > ~/vm_lab/meta-data << 'EOF'
instance-id: vm-001
local-hostname: ubuntu-vm1
EOF

# Create cloud-init ISO:
sudo apt install -y cloud-image-utils
cloud-localds ~/vm_lab/images/cloud-init-vm1.iso ~/vm_lab/user-data ~/vm_lab/meta-data

# Create a copy of the cloud image for the VM:
cp ~/vm_lab/images/jammy-server-cloudimg-amd64.img ~/vm_lab/images/ubuntu-vm1.qcow2

# Resize the disk (cloud images are small by default):
qemu-img resize ~/vm_lab/images/ubuntu-vm1.qcow2 20G

# Create VM using virt-install:
sudo virt-install \
  --name ubuntu-vm1 \
  --memory 2048 \
  --vcpus 2 \
  --disk ~/vm_lab/images/ubuntu-vm1.qcow2,device=disk,bus=virtio \
  --disk ~/vm_lab/images/cloud-init-vm1.iso,device=cdrom \
  --os-variant ubuntu22.04 \
  --network network=default,model=virtio \
  --graphics none \
  --console pty,target_type=serial \
  --noautoconsole \
  --import

# Task 8: Create VM from ISO
echo "=== Creating VM from ISO ==="

# Download Ubuntu Server ISO (example - use smaller one for testing):
cd ~/vm_lab/isos
# wget https://releases.ubuntu.com/22.04/ubuntu-22.04.3-live-server-amd64.iso

# For this example, we'll simulate with the cloud image approach
# In production, use actual ISO:

# Create VM with ISO installation:
sudo virt-install \
  --name ubuntu-vm2 \
  --memory 2048 \
  --vcpus 2 \
  --disk size=20,path=/var/lib/libvirt/images/ubuntu-vm2.qcow2,bus=virtio \
  --os-variant ubuntu22.04 \
  --network network=default \
  --graphics none \
  --console pty,target_type=serial \
  --cdrom ~/vm_lab/isos/ubuntu-22.04.3-live-server-amd64.iso \
  --noautoconsole
# Note: This starts the installation process; you'd connect to console to complete

# Alternative: Create without starting:
sudo virt-install \
  --name ubuntu-vm3 \
  --memory 1024 \
  --vcpus 1 \
  --disk size=10,path=/var/lib/libvirt/images/ubuntu-vm3.qcow2 \
  --os-variant ubuntu22.04 \
  --network network=default \
  --graphics none \
  --noautoconsole \
  --cdrom ~/vm_lab/isos/ubuntu-22.04.3-live-server-amd64.iso \
  --noreboot

# Task 9: List all VMs
echo "=== Listing Virtual Machines ==="

# List running VMs:
virsh list

# List all VMs (including stopped):
virsh list --all

# List with more details:
virsh list --all --title

# Show VM info in detail:
virsh dominfo ubuntu-vm1

# Get VM state:
virsh domstate ubuntu-vm1

# Task 10: Configure VM auto-start
echo "=== Configuring VM Auto-start ==="

# Enable auto-start for VM:
virsh autostart ubuntu-vm1

# Verify auto-start is enabled:
virsh dominfo ubuntu-vm1 | grep "Autostart"

# Disable auto-start:
# virsh autostart ubuntu-vm1 --disable

# List all VMs with auto-start status:
for vm in $(virsh list --all --name); do
    echo -n "$vm: "
    virsh dominfo $vm | grep "Autostart:" | awk '{print $2}'
done

# Part C: VM Management Operations

# Task 11: Start, stop, restart VMs
echo "=== VM Lifecycle Management ==="

# Start a VM:
virsh start ubuntu-vm1

# Verify it's running:
virsh list

# Graceful shutdown (sends ACPI signal):
virsh shutdown ubuntu-vm1

# Force stop (like pulling power plug):
virsh destroy ubuntu-vm1

# Restart (graceful):
virsh reboot ubuntu-vm1

# Suspend (pause) VM:
virsh suspend ubuntu-vm1

# Resume suspended VM:
virsh resume ubuntu-vm1

# Save VM state to file (hibernation):
virsh save ubuntu-vm1 ~/vm_lab/ubuntu-vm1-state.save

# Restore from saved state:
virsh restore ~/vm_lab/ubuntu-vm1-state.save

# Task 12: Connect to VM console
echo "=== Connecting to VM Console ==="

echo "Method 1: virsh console (serial console)"
echo "  virsh console ubuntu-vm1"
echo "  To exit: Press Ctrl+]"
echo ""

echo "Method 2: VNC/SPICE (if GUI available)"
echo "  virt-viewer ubuntu-vm1"
echo ""

echo "Method 3: SSH (once VM is running and has network)"
echo "  Get VM IP:"
echo "  virsh domifaddr ubuntu-vm1"
echo "  Or:"
echo "  virsh net-dhcp-leases default"
echo "  Then SSH:"
echo "  ssh ubuntu@<vm-ip>"

# Get VM IP address:
virsh domifaddr ubuntu-vm1 || virsh net-dhcp-leases default

# Task 13: Modify VM resources
echo "=== Modifying VM Resources ==="

# View current configuration:
virsh dominfo ubuntu-vm1

# Shutdown VM first:
virsh shutdown ubuntu-vm1
# Wait for shutdown:
while virsh domstate ubuntu-vm1 | grep -q running; do sleep 1; done

# Change memory (requires VM to be off):
virsh setmem ubuntu-vm1 4G --config

# Change max memory:
virsh setmaxmem ubuntu-vm1 8G --config

# Change CPU count:
virsh setvcpus ubuntu-vm1 4 --config --maximum
virsh setvcpus ubuntu-vm1 4 --config

# Edit VM XML directly for advanced changes:
virsh edit ubuntu-vm1

# Apply changes by starting VM:
virsh start ubuntu-vm1

# Verify new settings:
virsh dominfo ubuntu-vm1

# For live changes (VM running) - if supported:
# virsh setmem ubuntu-vm1 3G --live
# virsh setvcpus ubuntu-vm1 2 --live

# Task 14: Clone a VM
echo "=== Cloning Virtual Machines ==="

# Shutdown source VM first:
virsh shutdown ubuntu-vm1
while virsh domstate ubuntu-vm1 | grep -q running; do sleep 1; done

# Clone VM:
sudo virt-clone \
  --original ubuntu-vm1 \
  --name ubuntu-vm1-clone \
  --file /var/lib/libvirt/images/ubuntu-vm1-clone.qcow2

# List VMs to see clone:
virsh list --all

# Clone with auto-generated name and disk:
sudo virt-clone --original ubuntu-vm1 --auto-clone

# Task 15: VM Snapshots
echo "=== Managing VM Snapshots ==="

# Create a snapshot (VM can be running or stopped):
virsh snapshot-create-as ubuntu-vm1 \
  snapshot1 \
  "Snapshot before updates" \
  --disk-only  # Or omit for internal snapshot

# Better: Create snapshot with memory (VM must be running):
virsh snapshot-create-as ubuntu-vm1 snapshot-with-memory \
  "Snapshot with memory state"

# List snapshots:
virsh snapshot-list ubuntu-vm1

# Show snapshot details:
virsh snapshot-info ubuntu-vm1 snapshot1

# Revert to snapshot:
virsh snapshot-revert ubuntu-vm1 snapshot1

# Delete a snapshot:
virsh snapshot-delete ubuntu-vm1 snapshot1

# Create external snapshot (better for large disks):
virsh snapshot-create-as ubuntu-vm1 external-snap1 \
  --diskspec vda,file=/var/lib/libvirt/images/ubuntu-vm1-snap1.qcow2 \
  --disk-only \
  --atomic

# Part D: Networking and Storage

# Task 16: List virtual networks
echo "=== Listing Virtual Networks ==="

# List active networks:
virsh net-list

# List all networks (including inactive):
virsh net-list --all

# Show network details:
virsh net-info default

# Show network configuration (XML):
virsh net-dumpxml default

# Show DHCP leases:
virsh net-dhcp-leases default

# Task 17: Create custom isolated network
echo "=== Creating Custom Virtual Network ==="

# Create network definition:
cat > ~/vm_lab/isolated-network.xml << 'EOF'
<network>
  <name>isolated-net</name>
  <bridge name='virbr1'/>
  <forward mode='none'/>
  <ip address='192.168.100.1' netmask='255.255.255.0'>
    <dhcp>
      <range start='192.168.100.100' end='192.168.100.200'/>
    </dhcp>
  </ip>
</network>
EOF

# Define the network:
virsh net-define ~/vm_lab/isolated-network.xml

# Start the network:
virsh net-start isolated-net

# Enable auto-start:
virsh net-autostart isolated-net

# Verify:
virsh net-list --all

# Create NAT network (with internet access):
cat > ~/vm_lab/nat-network.xml << 'EOF'
<network>
  <name>nat-network</name>
  <forward mode='nat'/>
  <bridge name='virbr2' stp='on' delay='0'/>
  <ip address='192.168.200.1' netmask='255.255.255.0'>
    <dhcp>
      <range start='192.168.200.100' end='192.168.200.200'/>
    </dhcp>
  </ip>
</network>
EOF

virsh net-define ~/vm_lab/nat-network.xml
virsh net-start nat-network
virsh net-autostart nat-network

# Task 18: Attach VM to multiple networks
echo "=== Attaching VM to Multiple Networks ==="

# View current network interfaces:
virsh domiflist ubuntu-vm1

# Attach VM to additional network (VM must be running):
virsh start ubuntu-vm1

# Attach new network interface:
virsh attach-interface ubuntu-vm1 network isolated-net \
  --model virtio \
  --config --live

# Verify:
virsh domiflist ubuntu-vm1

# Detach interface:
# Get the MAC address first:
MAC=$(virsh domiflist ubuntu-vm1 | grep isolated-net | awk '{print $5}')
# virsh detach-interface ubuntu-vm1 network --mac $MAC --config --live

# Edit VM XML to add network permanently:
virsh edit ubuntu-vm1
# Add in <devices> section:
# <interface type='network'>
#   <source network='isolated-net'/>
#   <model type='virtio'/>
# </interface>

# Task 19: Create and attach virtual disks
echo "=== Managing Virtual Disks ==="

# Create a new virtual disk:
sudo qemu-img create -f qcow2 /var/lib/libvirt/images/ubuntu-vm1-data.qcow2 50G

# Check disk info:
qemu-img info /var/lib/libvirt/images/ubuntu-vm1-data.qcow2

# Attach disk to running VM (temporary):
virsh attach-disk ubuntu-vm1 \
  /var/lib/libvirt/images/ubuntu-vm1-data.qcow2 \
  vdb \
  --driver qemu \
  --subdriver qcow2 \
  --targetbus virtio \
  --live

# Attach disk permanently:
virsh attach-disk ubuntu-vm1 \
  /var/lib/libvirt/images/ubuntu-vm1-data.qcow2 \
  vdb \
  --driver qemu \
  --subdriver qcow2 \
  --targetbus virtio \
  --config

# Verify:
virsh domblklist ubuntu-vm1

# Detach disk:
# virsh detach-disk ubuntu-vm1 vdb --config --live

# Resize existing disk (VM must be off):
virsh shutdown ubuntu-vm1
while virsh domstate ubuntu-vm1 | grep -q running; do sleep 1; done

qemu-img resize /var/lib/libvirt/images/ubuntu-vm1.qcow2 +10G

# Task 20: VM backup strategy
cat > ~/vm_lab/vm_backup.sh << 'EOF'
#!/bin/bash

# VM Backup Script
# Backs up VMs using snapshots and exports

set -euo pipefail

BACKUP_DIR="$HOME/vm_lab/backups"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"

backup_vm() {
    local VM_NAME=$1
    echo "=== Backing up VM: $VM_NAME ==="
    
    local BACKUP_PATH="$BACKUP_DIR/${VM_NAME}_${DATE}"
    mkdir -p "$BACKUP_PATH"
    
    # Check if VM is running
    if virsh domstate "$VM_NAME" | grep -q running; then
        echo "VM is running, creating snapshot..."
        
        # Create snapshot
        virsh snapshot-create-as "$VM_NAME" "backup-$DATE" \
            "Backup snapshot" --atomic
        
        RUNNING=true
    else
        RUNNING=false
    fi
    
    # Export VM XML configuration
    echo "Exporting VM configuration..."
    virsh dumpxml "$VM_NAME" > "$BACKUP_PATH/${VM_NAME}.xml"
    
    # Get disk paths
    echo "Backing up VM disks..."
    virsh domblklist "$VM_NAME" --details | grep disk | awk '{print $4}' | while read -r DISK; do
        if [ -f "$DISK" ]; then
            DISK_NAME=$(basename "$DISK")
            echo "  Copying: $DISK_NAME"
            
            # For running VMs, we'd use snapshots or blockcopy
            # For stopped VMs, simple copy works:
            if [ "$RUNNING" = false ]; then
                cp "$DISK" "$BACKUP_PATH/"
            else
                # Use qemu-img for live backup (external snapshot method)
                qemu-img convert -O qcow2 "$DISK" "$BACKUP_PATH/$DISK_NAME"
            fi
        fi
    done
    
    # Backup network configuration
    echo "Backing up network interfaces..."
    virsh domiflist "$VM_NAME" > "$BACKUP_PATH/network-interfaces.txt"
    
    # Remove snapshot if created
    if [ "$RUNNING" = true ]; then
        virsh snapshot-delete "$VM_NAME" "backup-$DATE"
    fi
    
    echo "Backup completed: $BACKUP_PATH"
    du -sh "$BACKUP_PATH"
}

restore_vm() {
    local BACKUP_PATH=$1
    local VM_XML="$BACKUP_PATH/"*.xml
    
    echo "=== Restoring VM from: $BACKUP_PATH ==="
    
    # Get VM name from XML:
    local VM_NAME=$(grep "<name>" "$VM_XML" | sed 's/.*<name>\(.*\)<\/name>/\1/')
    
    # Undefine VM if exists:
    virsh dominfo "$VM_NAME" &>/dev/null && virsh undefine "$VM_NAME"
    
    # Restore disk files:
    echo "Restoring disk files..."
    cp "$BACKUP_PATH"/*.qcow2 /var/lib/libvirt/images/ 2>/dev/null || true
    cp "$BACKUP_PATH"/*.img /var/lib/libvirt/images/ 2>/dev/null || true
    
    # Define VM from XML:
    echo "Defining VM..."
    virsh define "$VM_XML"
    
    echo "VM restored: $VM_NAME"
    virsh dominfo "$VM_NAME"
}

# Main
if [ $# -eq 0 ]; then
    echo "Usage:"
    echo "  $0 backup <vm-name>      - Backup a VM"
    echo "  $0 restore <backup-path> - Restore a VM from backup"
    echo "  $0 list                  - List available backups"
    exit 1
fi

case "$1" in
    backup)
        if [ -z "${2:-}" ]; then
            echo "Error: VM name required"
            exit 1
        fi
        backup_vm "$2"
        ;;
    restore)
        if [ -z "${2:-}" ]; then
            echo "Error: Backup path required"
            exit 1
        fi
        restore_vm "$2"
        ;;
    list)
        echo "=== Available Backups ==="
        ls -lh "$BACKUP_DIR/"
        ;;
    *)
        echo "Unknown command: $1"
        exit 1
        ;;
esac
EOF

chmod +x ~/vm_lab/vm_backup.sh

# Example usage:
echo "Backup script created: ~/vm_lab/vm_backup.sh"
echo ""
echo "Usage examples:"
echo "  ~/vm_lab/vm_backup.sh backup ubuntu-vm1"
echo "  ~/vm_lab/vm_backup.sh list"
echo "  ~/vm_lab/vm_backup.sh restore ~/vm_lab/backups/ubuntu-vm1_20240101_120000"

# Create comprehensive VM management script:
cat > ~/vm_lab/vm_manager.sh << 'EOF'
#!/bin/bash

# Comprehensive VM Management Script

show_menu() {
    cat << 'MENU'
=== VM Management Tool ===

1. List all VMs
2. Show VM details
3. Start VM
4. Stop VM
5. Restart VM
6. Create snapshot
7. List snapshots
8. Restore snapshot
9. Clone VM
10. Backup VM
11. Show network info
12. Show disk info
13. Monitor VM resources
14. Quit

MENU
}

list_vms() {
    echo "=== Virtual Machines ==="
    virsh list --all
}

vm_details() {
    read -p "Enter VM name: " vm
    virsh dominfo "$vm"
    echo ""
    echo "Network interfaces:"
    virsh domiflist "$vm"
    echo ""
    echo "Disks:"
    virsh domblklist "$vm"
}

monitor_resources() {
    read -p "Enter VM name: " vm
    echo "=== Resource Monitoring for $vm ==="
    
    echo "CPU Stats:"
    virsh cpu-stats "$vm"
    
    echo ""
    echo "Memory Stats:"
    virsh dommemstat "$vm"
    
    echo ""
    echo "Disk I/O:"
    virsh domblkstat "$vm" --device vda
    
    echo ""
    echo "Network I/O:"
    virsh domifstat "$vm" --interface vnet0
}

# Main loop
while true; do
    show_menu
    read -p "Select option: " choice
    
    case $choice in
        1) list_vms ;;
        2) vm_details ;;
        3) read -p "VM name: " vm; virsh start "$vm" ;;
        4) read -p "VM name: " vm; virsh shutdown "$vm" ;;
        5) read -p "VM name: " vm; virsh reboot "$vm" ;;
        6) read -p "VM name: " vm; read -p "Snapshot name: " snap
           virsh snapshot-create-as "$vm" "$snap" ;;
        7) read -p "VM name: " vm; virsh snapshot-list "$vm" ;;
        8) read -p "VM name: " vm; read -p "Snapshot name: " snap
           virsh snapshot-revert "$vm" "$snap" ;;
        9) read -p "Source VM: " src; read -p "New VM name: " dst
           sudo virt-clone --original "$src" --name "$dst" --auto-clone ;;
        10) read -p "VM name: " vm
            ~/vm_lab/vm_backup.sh backup "$vm" ;;
        11) virsh net-list --all
            echo ""
            virsh net-dhcp-leases default ;;
        12) read -p "VM name: " vm
            virsh domblklist "$vm" --details ;;
        13) monitor_resources ;;
        14) exit 0 ;;
        *) echo "Invalid option" ;;
    esac
    
    echo ""
    read -p "Press Enter to continue..."
done
EOF

chmod +x ~/vm_lab/vm_manager.sh

echo ""
echo "VM management script created: ~/vm_lab/vm_manager.sh"
echo "Run it to get an interactive VM management menu"
```

**Explanation**:
- **Task 2**: `qemu-kvm` is the QEMU binary with KVM support, `libvirt` provides management abstraction
- **Task 7**: Cloud images are pre-configured OS images that use cloud-init for customization; much faster than ISO installation
- **Task 11**: `shutdown` is graceful (ACPI), `destroy` is immediate (like power off)
- **Task 13**: `--config` changes persistent configuration, `--live` changes running state (if supported)
- **Task 15**: Internal snapshots store state in the same qcow2 file; external snapshots create new files (better for production)
- **Task 17**: `forward mode='none'` creates isolated network; `mode='nat'` provides internet access
- **Storage pools**: Manage VM disks centrally; default pool is `/var/lib/libvirt/images/`

**Essential virsh Commands Summary**:
```bash
# VM Lifecycle
virsh list --all                # List all VMs
virsh start <vm>                # Start VM
virsh shutdown <vm>             # Graceful shutdown
virsh destroy <vm>              # Force stop
virsh reboot <vm>               # Reboot
virsh autostart <vm>            # Enable auto-start
virsh console <vm>              # Connect to console (Ctrl+] to exit)

# VM Information
virsh dominfo <vm>              # VM details
virsh domstate <vm>             # VM state
virsh domifaddr <vm>            # Get VM IP address
virsh domiflist <vm>            # List network interfaces
virsh domblklist <vm>           # List disks

# VM Management
virsh edit <vm>                 # Edit VM XML
virsh dumpxml <vm>              # Export VM config
virsh define vm.xml             # Create VM from XML
virsh undefine <vm>             # Remove VM definition
virsh setmem <vm> 4G --config   # Set memory
virsh setvcpus <vm> 4 --config  # Set CPUs

# Snapshots
virsh snapshot-create-as <vm> <name>     # Create snapshot
virsh snapshot-list <vm>                 # List snapshots
virsh snapshot-revert <vm> <snapshot>    # Restore snapshot
virsh snapshot-delete <vm> <snapshot>    # Delete snapshot

# Networks
virsh net-list --all            # List networks
virsh net-start <network>       # Start network
virsh net-autostart <network>   # Auto-start network
virsh net-dhcp-leases <network> # Show DHCP leases

# Storage
virsh pool-list --all           # List storage pools
virsh vol-list <pool>           # List volumes in pool
virsh vol-create-as <pool> <name> 10G  # Create volume
```

**VM Networking Modes**:
- **NAT** (`forward mode='nat'`): VMs can access internet, not directly accessible from outside
- **Isolated** (`forward mode='none'`): VMs can talk to each other, no external access
- **Bridge** (`forward mode='bridge'`): VMs appear as physical network devices
- **Routed** (`forward mode='route'`): VMs are on routed network

</details>

### Extensions
1. **Advanced**: Set up nested virtualization to run VMs inside VMs. Configure a KVM host inside a VM and test performance implications. Implement live migration between two KVM hosts using shared storage
2. **Challenge**: Build a complete virtualization infrastructure with: automated VM provisioning using Ansible or Terraform, custom VM templates for different workloads, monitoring and resource allocation policies, automated backup and disaster recovery, and integration with cloud-init for custom VM configuration

---

## Task 7: Container Management with Docker and Podman

### Learning Objectives
- Install and configure Docker and Podman container engines
- Build, run, and manage containers using both Docker and Podman
- Create custom container images using Dockerfiles
- Manage container networking, volumes, and resource limits
- Understand rootless containers and security best practices

### Context
Your development team is transitioning from virtual machines to containerized applications. You need to set up container infrastructure on Ubuntu 22.04 servers, supporting both Docker (for compatibility with existing CI/CD pipelines) and Podman (for rootless operation and better security). The team needs to run web applications, databases, and microservices in containers. You must configure container networking for service communication, implement persistent storage for database containers, set resource limits to prevent resource exhaustion, and create custom container images for your organization's applications. Additionally, you need to establish security policies including rootless containers where possible and implement a container backup and migration strategy.

Container technology is a critical LFCS competency. Modern Linux administrators must understand how to manage containers, which have become the standard for application deployment. Understanding both Docker and Podman provides flexibility and demonstrates knowledge of different container runtime approaches.

### Task Instructions

**Prerequisites**: Set up container lab environment
```bash
# Create working directory
mkdir -p ~/container_lab/{dockerfiles,volumes,scripts}
cd ~/container_lab

# Check system requirements
uname -r  # Kernel 3.10+ required
```

**Your Tasks**:

**Part A: Docker Installation and Basic Operations**
1. Install Docker Engine on Ubuntu 22.04
2. Configure Docker daemon and add user to docker group
3. Verify Docker installation and run hello-world container
4. Run an interactive container (Ubuntu or Alpine)
5. List running and stopped containers

**Part B: Docker Container Management**
6. Run a web server container (nginx) with port mapping
7. Run a container in detached mode and attach to it
8. View container logs and inspect container details
9. Execute commands inside a running container
10. Stop, start, and remove containers
11. Set resource limits (CPU, memory) on containers

**Part C: Docker Images and Dockerfiles**
12. Search for and pull images from Docker Hub
13. List local images and remove unused images
14. Create a custom Dockerfile for a simple web application
15. Build a custom image and tag it properly
16. Push image to a registry (optional: local or Docker Hub)

**Part D: Docker Networking and Volumes**
17. Create and manage Docker networks
18. Connect containers to custom networks
19. Create and mount Docker volumes for persistent storage
20. Run a database container with persistent volume
21. Link multiple containers (web app + database)

**Part E: Podman Installation and Usage**
22. Install Podman on Ubuntu 22.04
23. Run containers with Podman (rootless)
24. Compare Docker and Podman commands
25. Generate systemd service files from Podman containers
26. Migrate a Docker container to Podman

### Hints and Resources
1. Docker commands: `docker run`, `docker ps`, `docker exec`, `docker build`, `docker logs`
2. Podman is daemonless and can run rootless containers by default
3. Docker images are stored in `/var/lib/docker/`, Podman in `~/.local/share/containers/`
4. `docker-compose` for multi-container applications (not covered in basic LFCS)
5. References: [Docker Documentation](https://docs.docker.com/), [Podman Documentation](https://docs.podman.io/)

### Estimated Time and Difficulty
**50-65 minutes, Intermediate to Advanced**

### Verification
To verify your solution:
- `docker --version` and `podman --version` show installed versions
- `docker ps -a` shows all containers
- `docker images` shows local images
- `curl http://localhost:port` accesses containerized web services
- `docker volume ls` shows created volumes
- `docker network ls` shows networks
- `podman generate systemd container-name` creates service file
- Containers persist data across restarts when using volumes

<details>
<summary>Solution</summary>

```bash
# Part A: Docker Installation and Basic Operations

# Task 1: Install Docker Engine
echo "=== Installing Docker ==="

# Update package index:
sudo apt update

# Install prerequisites:
sudo apt install -y ca-certificates curl gnupg lsb-release

# Add Docker's official GPG key:
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
    sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# Set up Docker repository:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker Engine:
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin

# Task 2: Configure Docker
echo "=== Configuring Docker ==="

# Start and enable Docker service:
sudo systemctl enable --now docker
sudo systemctl status docker

# Add current user to docker group (to run docker without sudo):
sudo usermod -aG docker $USER

echo "⚠️  Log out and back in for group changes to take effect"
echo "Or run: newgrp docker"

# Verify group membership:
groups $USER

# Configure Docker daemon (optional optimizations):
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json > /dev/null << 'EOF'
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  },
  "storage-driver": "overlay2"
}
EOF

# Restart Docker to apply configuration:
sudo systemctl restart docker

# Task 3: Verify Docker installation
echo "=== Verifying Docker Installation ==="

# Check Docker version:
docker --version
docker version

# Show system info:
docker info

# Run hello-world container:
docker run hello-world

# Verify it worked:
docker ps -a | grep hello-world

# Task 4: Run interactive container
echo "=== Running Interactive Containers ==="

# Run Ubuntu container interactively:
docker run -it ubuntu:22.04 /bin/bash
# Inside container:
# cat /etc/os-release
# apt update && apt install -y curl
# exit

# Run Alpine (minimal) container:
docker run -it alpine:latest /bin/sh
# Inside:
# apk add --no-cache curl
# exit

# Run with automatic cleanup (--rm removes container on exit):
docker run -it --rm alpine:latest /bin/sh

# Task 5: List containers
echo "=== Listing Containers ==="

# List running containers:
docker ps

# List all containers (including stopped):
docker ps -a

# List with custom format:
docker ps -a --format "table {{.ID}}\t{{.Names}}\t{{.Status}}\t{{.Image}}"

# List only container IDs:
docker ps -aq

# List container sizes:
docker ps -s

# Part B: Docker Container Management

# Task 6: Run nginx web server
echo "=== Running Web Server Container ==="

# Run nginx with port mapping:
docker run -d -p 8080:80 --name my-nginx nginx:latest

# Explanation:
# -d: detached mode (background)
# -p 8080:80: map host port 8080 to container port 80
# --name: give container a name
# nginx:latest: image to use

# Verify it's running:
docker ps

# Test the web server:
curl http://localhost:8080

# Or open in browser:
echo "Open http://localhost:8080 in your browser"

# Task 7: Detached mode and attach
echo "=== Detached Mode and Attaching ==="

# Run container in background:
docker run -d --name my-ubuntu ubuntu:22.04 sleep infinity

# Attach to running container:
docker attach my-ubuntu
# Note: Ctrl+P, Ctrl+Q to detach without stopping
# Ctrl+C will stop the container

# Better: Use exec for interactive shell:
docker exec -it my-ubuntu /bin/bash
# Now you can exit without stopping the container

# Task 8: View logs and inspect
echo "=== Container Logs and Inspection ==="

# View container logs:
docker logs my-nginx

# Follow logs in real-time:
docker logs -f my-nginx

# Show last 20 lines:
docker logs --tail 20 my-nginx

# Show logs with timestamps:
docker logs -t my-nginx

# Inspect container (detailed JSON):
docker inspect my-nginx

# Get specific information with --format:
docker inspect --format '{{.NetworkSettings.IPAddress}}' my-nginx
docker inspect --format '{{.State.Status}}' my-nginx

# Show container resource usage:
docker stats my-nginx --no-stream

# Task 9: Execute commands in container
echo "=== Executing Commands in Containers ==="

# Run single command:
docker exec my-nginx ls -la /usr/share/nginx/html

# Run interactive shell:
docker exec -it my-nginx /bin/bash

# Run as specific user:
docker exec -u root my-nginx whoami

# Run with environment variables:
docker exec -e VAR=value my-nginx env

# Multiple commands:
docker exec my-nginx sh -c "ls -la && pwd && whoami"

# Task 10: Stop, start, remove containers
echo "=== Container Lifecycle ==="

# Stop container (sends SIGTERM, waits, then SIGKILL):
docker stop my-nginx

# Stop with timeout:
docker stop -t 10 my-nginx  # Wait 10 seconds before SIGKILL

# Start stopped container:
docker start my-nginx

# Restart container:
docker restart my-nginx

# Pause container (freeze):
docker pause my-nginx

# Unpause:
docker unpause my-nginx

# Remove container (must be stopped):
docker stop my-nginx
docker rm my-nginx

# Force remove running container:
docker rm -f my-nginx

# Remove all stopped containers:
docker container prune

# Task 11: Resource limits
echo "=== Setting Resource Limits ==="

# Run with memory limit:
docker run -d --name nginx-limited \
  --memory="512m" \
  --memory-swap="512m" \
  nginx:latest

# Run with CPU limit:
docker run -d --name nginx-cpu \
  --cpus="1.5" \
  --cpu-shares=1024 \
  nginx:latest

# Combined limits:
docker run -d --name resource-limited \
  -p 8081:80 \
  --memory="256m" \
  --memory-swap="256m" \
  --cpus="0.5" \
  --pids-limit=100 \
  nginx:latest

# Verify limits:
docker stats resource-limited --no-stream

# Inspect resource settings:
docker inspect resource-limited | grep -A 10 "Memory"

# Part C: Docker Images and Dockerfiles

# Task 12: Search and pull images
echo "=== Managing Docker Images ==="

# Search for images on Docker Hub:
docker search nginx
docker search postgres --limit 5

# Pull image:
docker pull nginx:latest
docker pull nginx:alpine  # Specific tag

# Pull specific version:
docker pull ubuntu:22.04
docker pull postgres:15

# Task 13: List and remove images
echo "=== Listing and Cleaning Images ==="

# List all images:
docker images

# List with specific format:
docker images --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}"

# Show image IDs only:
docker images -q

# Remove image:
docker rmi nginx:alpine

# Remove image by ID:
docker rmi <image-id>

# Force remove (even if containers exist):
docker rmi -f <image-id>

# Remove all unused images:
docker image prune -a

# Remove dangling images (untagged):
docker image prune

# Task 14: Create Dockerfile
echo "=== Creating Custom Docker Image ==="

# Create a simple web application:
mkdir -p ~/container_lab/dockerfiles/webapp
cd ~/container_lab/dockerfiles/webapp

# Create application file:
cat > index.html << 'EOF'
<!DOCTYPE html>
<html>
<head><title>Custom Container</title></head>
<body>
    <h1>Hello from Custom Container!</h1>
    <p>This is a custom Docker image built for LFCS practice.</p>
    <p>Hostname: <span id="hostname"></span></p>
    <script>
        fetch('/api/hostname')
            .then(r => r.text())
            .then(h => document.getElementById('hostname').textContent = h);
    </script>
</body>
</html>
EOF

# Create a simple Python web server:
cat > app.py << 'EOF'
#!/usr/bin/env python3
from http.server import HTTPServer, SimpleHTTPRequestHandler
import socket

class CustomHandler(SimpleHTTPRequestHandler):
    def do_GET(self):
        if self.path == '/api/hostname':
            self.send_response(200)
            self.send_header('Content-type', 'text/plain')
            self.end_headers()
            self.wfile.write(socket.gethostname().encode())
        else:
            super().do_GET()

if __name__ == '__main__':
    server = HTTPServer(('0.0.0.0', 8000), CustomHandler)
    print("Server running on port 8000")
    server.serve_forever()
EOF

chmod +x app.py

# Create Dockerfile:
cat > Dockerfile << 'EOF'
# Use official Python image as base
FROM python:3.11-slim

# Set working directory
WORKDIR /app

# Copy application files
COPY index.html /app/
COPY app.py /app/

# Expose port
EXPOSE 8000

# Set environment variable
ENV ENVIRONMENT=production

# Add metadata
LABEL maintainer="admin@example.com"
LABEL version="1.0"
LABEL description="Simple web application"

# Run application
CMD ["python3", "app.py"]
EOF

# Create .dockerignore:
cat > .dockerignore << 'EOF'
.git
*.md
__pycache__
*.pyc
EOF

# Task 15: Build custom image
echo "=== Building Custom Image ==="

# Build image:
docker build -t my-webapp:1.0 .

# Build with build arguments:
docker build -t my-webapp:1.0 \
  --build-arg VERSION=1.0 \
  --no-cache \
  .

# Verify image was created:
docker images | grep my-webapp

# Inspect image:
docker inspect my-webapp:1.0

# View image history:
docker history my-webapp:1.0

# Run the custom image:
docker run -d -p 8000:8000 --name webapp my-webapp:1.0

# Test it:
curl http://localhost:8000
curl http://localhost:8000/api/hostname

# Task 16: Tag and push image (optional)
echo "=== Tagging and Pushing Images ==="

# Tag image with version:
docker tag my-webapp:1.0 my-webapp:latest
docker tag my-webapp:1.0 my-webapp:stable

# Tag for Docker Hub (replace username):
# docker tag my-webapp:1.0 username/my-webapp:1.0

# Login to Docker Hub:
# docker login

# Push to Docker Hub:
# docker push username/my-webapp:1.0

# For local registry:
# docker tag my-webapp:1.0 localhost:5000/my-webapp:1.0
# docker push localhost:5000/my-webapp:1.0

# Part D: Docker Networking and Volumes

# Task 17: Create and manage networks
echo "=== Docker Networking ==="

# List networks:
docker network ls

# Inspect default bridge network:
docker network inspect bridge

# Create custom bridge network:
docker network create my-network

# Create network with custom subnet:
docker network create --driver bridge \
  --subnet=172.18.0.0/16 \
  --gateway=172.18.0.1 \
  custom-network

# Create isolated network (no external access):
docker network create --internal isolated-net

# Remove network:
docker network rm my-network

# Task 18: Connect containers to networks
echo "=== Connecting Containers to Networks ==="

# Run container on custom network:
docker run -d --name web1 \
  --network custom-network \
  nginx:latest

# Connect existing container to network:
docker network connect custom-network webapp

# Disconnect from network:
docker network disconnect custom-network webapp

# Run container with network alias:
docker run -d --name web2 \
  --network custom-network \
  --network-alias webserver \
  nginx:latest

# Containers on same network can communicate by name:
docker run -it --network custom-network alpine ping web1

# Task 19: Create and mount volumes
echo "=== Docker Volumes ==="

# List volumes:
docker volume ls

# Create named volume:
docker volume create my-data

# Inspect volume:
docker volume inspect my-data

# Run container with volume:
docker run -d --name nginx-vol \
  -v my-data:/usr/share/nginx/html \
  -p 8082:80 \
  nginx:latest

# Write to volume:
docker exec nginx-vol sh -c 'echo "<h1>Persistent Data</h1>" > /usr/share/nginx/html/index.html'

# Verify:
curl http://localhost:8082

# Stop and remove container:
docker stop nginx-vol && docker rm nginx-vol

# Run new container with same volume (data persists):
docker run -d --name nginx-vol2 \
  -v my-data:/usr/share/nginx/html \
  -p 8082:80 \
  nginx:latest

curl http://localhost:8082  # Same content!

# Bind mount (mount host directory):
docker run -d --name nginx-bind \
  -v ~/container_lab/html:/usr/share/nginx/html:ro \
  -p 8083:80 \
  nginx:latest

# Remove volumes:
docker volume rm my-data

# Remove unused volumes:
docker volume prune

# Task 20: Database with persistent storage
echo "=== Running Database Container ==="

# Create volume for PostgreSQL:
docker volume create postgres-data

# Run PostgreSQL with persistent storage:
docker run -d \
  --name my-postgres \
  -e POSTGRES_PASSWORD=mysecretpassword \
  -e POSTGRES_USER=admin \
  -e POSTGRES_DB=myapp \
  -v postgres-data:/var/lib/postgresql/data \
  -p 5432:5432 \
  postgres:15

# Wait for database to start:
sleep 5

# Connect to database:
docker exec -it my-postgres psql -U admin -d myapp

# Or run SQL directly:
docker exec my-postgres psql -U admin -d myapp -c "CREATE TABLE users (id SERIAL PRIMARY KEY, name VARCHAR(100));"
docker exec my-postgres psql -U admin -d myapp -c "INSERT INTO users (name) VALUES ('Alice'), ('Bob');"
docker exec my-postgres psql -U admin -d myapp -c "SELECT * FROM users;"

# Task 21: Multi-container application
echo "=== Multi-Container Application ==="

# Create network for app:
docker network create app-network

# Run database:
docker run -d \
  --name app-db \
  --network app-network \
  -e POSTGRES_PASSWORD=secret \
  -e POSTGRES_DB=webapp \
  -v app-db-data:/var/lib/postgresql/data \
  postgres:15

# Create simple web app that connects to DB:
mkdir -p ~/container_lab/dockerfiles/dbapp
cd ~/container_lab/dockerfiles/dbapp

cat > app.py << 'EOF'
#!/usr/bin/env python3
from http.server import HTTPServer, BaseHTTPRequestHandler
import os

class Handler(BaseHTTPRequestHandler):
    def do_GET(self):
        self.send_response(200)
        self.send_header('Content-type', 'text/html')
        self.end_headers()
        
        html = f"""
        <html><body>
        <h1>Multi-Container App</h1>
        <p>Database Host: {os.getenv('DB_HOST', 'not set')}</p>
        <p>Database Name: {os.getenv('DB_NAME', 'not set')}</p>
        </body></html>
        """
        self.wfile.write(html.encode())

if __name__ == '__main__':
    HTTPServer(('0.0.0.0', 8000), Handler).serve_forever()
EOF

cat > Dockerfile << 'EOF'
FROM python:3.11-slim
WORKDIR /app
COPY app.py .
EXPOSE 8000
CMD ["python3", "app.py"]
EOF

# Build app image:
docker build -t dbapp:1.0 .

# Run web app connected to database:
docker run -d \
  --name app-web \
  --network app-network \
  -e DB_HOST=app-db \
  -e DB_NAME=webapp \
  -p 8090:8000 \
  dbapp:1.0

# Test:
curl http://localhost:8090

# Verify connectivity:
docker exec app-web ping -c 2 app-db

# Part E: Podman Installation and Usage

# Task 22: Install Podman
echo "=== Installing Podman ==="

# Update and install:
sudo apt update
sudo apt install -y podman

# Verify installation:
podman --version
podman info

# Task 23: Run containers with Podman (rootless)
echo "=== Using Podman (Rootless) ==="

# Run container with Podman (no sudo needed):
podman run -d --name podman-nginx -p 8888:80 nginx:latest

# List containers:
podman ps

# Check if running rootless:
podman info | grep -i root

# Run interactive container:
podman run -it --rm alpine:latest /bin/sh

# Task 24: Compare Docker and Podman
cat > ~/container_lab/docker_podman_comparison.md << 'EOF'
# Docker vs Podman Command Comparison

| Operation | Docker | Podman |
|-----------|--------|--------|
| Run container | `docker run` | `podman run` |
| List containers | `docker ps` | `podman ps` |
| Stop container | `docker stop` | `podman stop` |
| Remove container | `docker rm` | `podman rm` |
| List images | `docker images` | `podman images` |
| Pull image | `docker pull` | `podman pull` |
| Build image | `docker build` | `podman build` |
| Execute in container | `docker exec` | `podman exec` |
| View logs | `docker logs` | `podman logs` |

## Key Differences:

**Docker:**
- Requires daemon (dockerd)
- Runs as root by default
- Better ecosystem/tooling support
- More mature

**Podman:**
- Daemonless (fork-exec model)
- Rootless by default (better security)
- Compatible with Docker CLI
- Can generate systemd units
- Supports pods (Kubernetes-like)

**Alias for compatibility:**
```bash
alias docker=podman
```
EOF

cat ~/container_lab/docker_podman_comparison.md

# Create alias (optional):
echo "alias docker=podman" >> ~/.bashrc

# Task 25: Generate systemd service
echo "=== Generating Systemd Services from Podman ==="

# Run a container:
podman run -d --name web-service \
  -p 9090:80 \
  nginx:latest

# Generate systemd service file:
podman generate systemd --new --name web-service

# Save to systemd user directory:
mkdir -p ~/.config/systemd/user
podman generate systemd --new --name web-service \
  > ~/.config/systemd/user/web-service.service

# View service file:
cat ~/.config/systemd/user/web-service.service

# Enable and start with systemd:
systemctl --user daemon-reload
systemctl --user enable web-service.service
systemctl --user start web-service.service

# Check status:
systemctl --user status web-service.service

# For system-wide services (as root):
# sudo podman generate systemd --new --name web-service \
#   > /etc/systemd/system/web-service.service
# sudo systemctl daemon-reload
# sudo systemctl enable --now web-service.service

# Task 26: Migrate Docker to Podman
cat > ~/container_lab/migrate_docker_podman.sh << 'EOF'
#!/bin/bash

echo "=== Migrating Docker Container to Podman ==="

CONTAINER_NAME=${1:-"my-nginx"}

echo "Step 1: Export Docker container to tar"
docker export $CONTAINER_NAME > /tmp/${CONTAINER_NAME}.tar

echo "Step 2: Import into Podman"
cat /tmp/${CONTAINER_NAME}.tar | podman import - ${CONTAINER_NAME}:migrated

echo "Step 3: Get Docker container configuration"
docker inspect $CONTAINER_NAME > /tmp/${CONTAINER_NAME}-config.json

echo "Step 4: Run with Podman using similar config"
# Extract port mappings, volumes, etc. and recreate with podman

echo "Container migrated. Image available as: ${CONTAINER_NAME}:migrated"
podman images | grep $CONTAINER_NAME

echo ""
echo "Alternatively, save/load Docker image:"
echo "  docker save nginx:latest > nginx.tar"
echo "  podman load < nginx.tar"

# Cleanup
rm /tmp/${CONTAINER_NAME}.tar
EOF

chmod +x ~/container_lab/migrate_docker_podman.sh

# Demonstrate migration:
# Save Docker image:
docker save nginx:latest -o ~/container_lab/nginx.tar

# Load into Podman:
podman load -i ~/container_lab/nginx.tar

# Verify:
podman images | grep nginx

# Create comprehensive container management script:
cat > ~/container_lab/container_manager.sh << 'EOF'
#!/bin/bash

# Container Management Script (works with both Docker and Podman)

RUNTIME=${CONTAINER_RUNTIME:-docker}  # Use podman if preferred

cmd_wrapper() {
    $RUNTIME "$@"
}

show_menu() {
    cat << 'MENU'
=== Container Management Tool ===

Container Runtime: $RUNTIME

1.  List containers
2.  Run new container
3.  Stop container
4.  Start container
5.  Remove container
6.  View logs
7.  Execute command in container
8.  List images
9.  Pull image
10. Build image from Dockerfile
11. List networks
12. List volumes
13. Container stats
14. Clean up (prune)
15. Switch runtime (Docker/Podman)
16. Quit

MENU
}

while true; do
    show_menu
    read -p "Select option: " choice
    
    case $choice in
        1) cmd_wrapper ps -a ;;
        2) read -p "Image name: " img
           read -p "Container name: " name
           read -p "Port mapping (host:container or blank): " port
           if [ -n "$port" ]; then
               cmd_wrapper run -d -p $port --name $name $img
           else
               cmd_wrapper run -d --name $name $img
           fi ;;
        3) read -p "Container name: " name; cmd_wrapper stop $name ;;
        4) read -p "Container name: " name; cmd_wrapper start $name ;;
        5) read -p "Container name: " name; cmd_wrapper rm -f $name ;;
        6) read -p "Container name: " name; cmd_wrapper logs --tail 50 $name ;;
        7) read -p "Container name: " name
           read -p "Command: " command
           cmd_wrapper exec -it $name $command ;;
        8) cmd_wrapper images ;;
        9) read -p "Image name: " img; cmd_wrapper pull $img ;;
        10) read -p "Dockerfile path: " path
            read -p "Image tag: " tag
            cmd_wrapper build -t $tag $path ;;
        11) cmd_wrapper network ls ;;
        12) cmd_wrapper volume ls ;;
        13) cmd_wrapper stats --no-stream ;;
        14) cmd_wrapper system prune -a ;;
        15) if [ "$RUNTIME" = "docker" ]; then
                RUNTIME="podman"
            else
                RUNTIME="docker"
            fi
            echo "Switched to: $RUNTIME" ;;
        16) exit 0 ;;
        *) echo "Invalid option" ;;
    esac
    
    echo ""
    read -p "Press Enter to continue..."
done
EOF

chmod +x ~/container_lab/container_manager.sh

echo ""
echo "Container management script created: ~/container_lab/container_manager.sh"
echo "Run with: ~/container_lab/container_manager.sh"
```

**Explanation**:
- **Task 1**: Docker requires repository setup; Podman is in standard Ubuntu repos
- **Task 6**: `-d` runs detached, `-p` maps ports (host:container)
- **Task 11**: Resource limits prevent containers from consuming all system resources
- **Task 14**: Dockerfiles use layered filesystem; each instruction creates a new layer
- **Task 17**: Bridge networks allow container-to-container communication
- **Task 19**: Named volumes persist data; bind mounts link host directories
- **Task 23**: Podman runs rootless by default, improving security
- **Task 25**: Podman can generate systemd units for container lifecycle management

**Essential Container Commands**:
```bash
# Container Lifecycle
docker/podman run [options] image [command]     # Create and start container
  -d                        # Detached mode
  -it                       # Interactive with TTY
  --rm                      # Remove on exit
  --name <name>             # Container name
  -p <host>:<container>     # Port mapping
  -v <source>:<dest>        # Volume mount
  -e KEY=value              # Environment variable
  --network <network>       # Network
  --memory <limit>          # Memory limit
  --cpus <number>           # CPU limit

docker/podman ps [options]                      # List containers
  -a                        # All containers
  -q                        # IDs only

docker/podman start/stop/restart <container>    # Control containers
docker/podman rm <container>                    # Remove container
docker/podman exec -it <container> <command>    # Execute in container
docker/podman logs <container>                  # View logs

# Images
docker/podman images                            # List images
docker/podman pull <image>                      # Pull image
docker/podman build -t <tag> <path>            # Build image
docker/podman rmi <image>                       # Remove image
docker/podman tag <source> <target>            # Tag image

# Networks
docker/podman network ls                        # List networks
docker/podman network create <name>             # Create network
docker/podman network connect <net> <cont>      # Connect container

# Volumes
docker/podman volume ls                         # List volumes
docker/podman volume create <name>              # Create volume
docker/podman volume rm <name>                  # Remove volume

# Maintenance
docker/podman system prune                      # Remove unused data
docker/podman stats                             # Resource usage
```

**Dockerfile Best Practices**:
```dockerfile
# Use specific base image tags
FROM python:3.11-slim

# Set working directory
WORKDIR /app

# Copy only requirements first (layer caching)
COPY requirements.txt .
RUN pip install -r requirements.txt

# Copy application code
COPY . .

# Use non-root user
RUN useradd -m appuser
USER appuser

# Expose port (documentation)
EXPOSE 8000

# Use ENTRYPOINT for main command, CMD for arguments
ENTRYPOINT ["python3"]
CMD ["app.py"]
```

</details>

### Extensions
1. **Advanced**: Set up a complete CI/CD pipeline that: builds Docker images automatically from Git commits, runs automated tests in containers, pushes to a private registry, and deploys to production using container orchestration
2. **Challenge**: Implement a multi-tier application with: web tier (nginx/apache), application tier (Python/Node.js), database tier (PostgreSQL), caching tier (Redis), all in separate containers with proper networking, persistent storage, resource limits, health checks, and automated backup/restore procedures

---

## Task 8: Mandatory Access Control with AppArmor and SELinux

### Learning Objectives
- Understand Mandatory Access Control (MAC) vs Discretionary Access Control (DAC)
- Configure and manage AppArmor on Ubuntu 22.04
- Create custom AppArmor profiles for applications
- Troubleshoot AppArmor denials and audit logs
- Understand SELinux concepts and contexts (for RHEL/CentOS awareness)

### Context
Your organization has strict security requirements for production servers. You need to implement Mandatory Access Control to restrict what applications can do, even if they're compromised. Ubuntu 22.04 uses AppArmor by default, and you must configure it to protect critical services like web servers, databases, and custom applications. The security team requires that all public-facing services run with AppArmor profiles in enforce mode. You need to create a custom profile for a new application, troubleshoot AppArmor denials without breaking functionality, and document the security policies. Additionally, you should understand SELinux concepts since some servers in the infrastructure run RHEL/CentOS where SELinux is the standard MAC system.

Understanding Mandatory Access Control is an important LFCS competency. While Ubuntu uses AppArmor and RHEL/CentOS use SELinux, administrators should be familiar with MAC concepts and capable of managing at least one MAC system effectively.

### Task Instructions

**Prerequisites**: Set up MAC lab environment
```bash
# Create working directory
mkdir -p ~/mac_lab/{profiles,logs,scripts,testapp}
cd ~/mac_lab

# Check AppArmor status
sudo aa-status
```

**Your Tasks**:

**Part A: AppArmor Basics**
1. Check AppArmor status and loaded profiles
2. Understand profile modes (enforce, complain, disable)
3. List all loaded AppArmor profiles and their modes
4. Switch a profile between enforce and complain mode
5. Reload AppArmor profiles after changes

**Part B: Managing AppArmor Profiles**
6. View an existing AppArmor profile (e.g., `/etc/apparmor.d/usr.sbin.nginx`)
7. Disable an AppArmor profile
8. Enable a disabled AppArmor profile
9. Put a profile in complain mode for troubleshooting
10. Enforce a profile in complain mode

**Part C: Creating Custom AppArmor Profiles**
11. Create a simple test application
12. Generate an AppArmor profile for the application
13. Run the application and check for AppArmor denials
14. Update the profile to allow necessary operations
15. Test the profile in enforce mode

**Part D: Troubleshooting and Auditing**
16. View AppArmor denial messages in system logs
17. Use `aa-logprof` to update profiles based on denials
18. Use `aa-genprof` to generate a profile from application behavior
19. Analyze audit logs to understand access patterns
20. Create a script to monitor AppArmor denials

**Part E: SELinux Concepts (Theory and Basic Commands)**
21. Understand SELinux modes (enforcing, permissive, disabled)
22. Understand SELinux contexts (user, role, type, level)
23. Learn basic SELinux commands (for RHEL/CentOS compatibility)
24. Compare AppArmor and SELinux approaches
25. Document security policies for both systems

### Hints and Resources
1. AppArmor profiles are in `/etc/apparmor.d/`
2. Common commands: `aa-status`, `aa-enforce`, `aa-complain`, `aa-disable`
3. Denials logged to `/var/log/syslog` or `/var/log/audit/audit.log`
4. Profile syntax: paths, capabilities, network access, file permissions
5. References: [AppArmor Documentation](https://gitlab.com/apparmor/apparmor/-/wikis/home), [SELinux User's Guide](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/using_selinux/)

### Estimated Time and Difficulty
**40-55 minutes, Intermediate to Advanced**

### Verification
To verify your solution:
- `sudo aa-status` shows loaded profiles and their modes
- `sudo aa-enforce profile-name` puts profile in enforce mode
- `sudo aa-complain profile-name` puts profile in complain mode
- `grep apparmor /var/log/syslog` shows AppArmor events
- `sudo apparmor_parser -r /etc/apparmor.d/profile` reloads profile
- Custom profiles successfully restrict application behavior
- Applications work correctly with profiles in enforce mode

<details>
<summary>Solution</summary>

```bash
# Part A: AppArmor Basics

# Task 1: Check AppArmor status
echo "=== Checking AppArmor Status ==="

# Check if AppArmor is enabled:
sudo aa-status

# Alternative check:
sudo systemctl status apparmor

# Check AppArmor module:
sudo aa-enabled
echo $?  # Returns 0 if enabled

# Show summary:
sudo aa-status | head -20

# Count profiles:
sudo aa-status | grep "profiles are loaded"

# Task 2: Understand profile modes
cat > ~/mac_lab/apparmor_modes.md << 'EOF'
# AppArmor Profile Modes

## 1. Enforce Mode
- **Description**: Profiles actively prevent operations not explicitly allowed
- **Use case**: Production systems, maximum security
- **Effect**: Blocks and logs denied operations
- **Symbol**: ✓ Enforced

## 2. Complain Mode
- **Description**: Profiles log violations but don't block them
- **Use case**: Developing profiles, troubleshooting
- **Effect**: Allows operations but logs what would be denied
- **Symbol**: ⚠ Complain

## 3. Disabled
- **Description**: Profile exists but not loaded
- **Use case**: Temporarily disable security for testing
- **Effect**: No restrictions, no logging
- **Symbol**: ✗ Disabled

## Mode Transitions
```
Disabled → Complain → Enforce
         ← Complain ← Enforce
```

## Best Practice Workflow
1. Develop in **complain mode**
2. Test thoroughly
3. Switch to **enforce mode** for production
4. Monitor logs for issues
5. Return to **complain mode** if problems arise
EOF

cat ~/mac_lab/apparmor_modes.md

# Task 3: List loaded profiles
echo "=== Listing AppArmor Profiles ==="

# Show all profiles with modes:
sudo aa-status

# List profiles in enforce mode:
sudo aa-status | grep "enforce mode" -A 100 | grep "^   "

# List profiles in complain mode:
sudo aa-status | grep "complain mode" -A 100 | grep "^   "

# List all profile files:
ls -la /etc/apparmor.d/ | grep -v "^d"

# Find profiles:
find /etc/apparmor.d/ -type f -name "*.*.*"

# Task 4: Switch profile modes
echo "=== Switching Profile Modes ==="

# Put profile in complain mode:
sudo aa-complain /usr/sbin/tcpdump

# Verify:
sudo aa-status | grep tcpdump

# Put profile in enforce mode:
sudo aa-enforce /usr/sbin/tcpdump

# Verify:
sudo aa-status | grep tcpdump

# Switch using profile path:
sudo aa-complain /etc/apparmor.d/usr.sbin.tcpdump
sudo aa-enforce /etc/apparmor.d/usr.sbin.tcpdump

# Task 5: Reload AppArmor profiles
echo "=== Reloading Profiles ==="

# Reload all profiles:
sudo systemctl reload apparmor

# Reload specific profile:
sudo apparmor_parser -r /etc/apparmor.d/usr.sbin.tcpdump

# Reload all profiles in directory:
sudo apparmor_parser -r /etc/apparmor.d/*

# Restart AppArmor service:
sudo systemctl restart apparmor

# Part B: Managing AppArmor Profiles

# Task 6: View existing profile
echo "=== Viewing AppArmor Profiles ==="

# View tcpdump profile:
sudo cat /etc/apparmor.d/usr.sbin.tcpdump

# View nginx profile if available:
if [ -f /etc/apparmor.d/usr.sbin.nginx ]; then
    sudo cat /etc/apparmor.d/usr.sbin.nginx
fi

# View with syntax highlighting (if available):
sudo less /etc/apparmor.d/usr.sbin.tcpdump

# Examine profile structure:
cat > ~/mac_lab/profile_structure.md << 'EOF'
# AppArmor Profile Structure

```apparmor
#include <tunables/global>

# Profile starts with executable path
/usr/sbin/executable {
  #include <abstractions/base>
  #include <abstractions/nameservice>

  # Capabilities (special permissions)
  capability net_raw,
  capability setuid,

  # Network access
  network inet raw,
  network inet6 raw,

  # File access rules
  /path/to/file r,              # read
  /path/to/file w,              # write
  /path/to/file x,              # execute
  /path/to/file rw,             # read + write
  /path/to/dir/ r,              # read directory
  /path/to/dir/** r,            # read recursively

  # Owner-only access
  owner /home/*/.config/ rw,

  # Execute with profile transition
  /usr/bin/helper Px,           # Use helper's profile
  /usr/bin/tool Ux,             # Unconfined execution
  /usr/bin/cmd ix,              # Inherit this profile

  # Deny rules (explicit)
  deny /etc/shadow r,

  # Variables
  @{HOME}=/home/**/
  @{PROC}=/proc/
}
```

## Common Abstractions
- `abstractions/base`: Basic system files
- `abstractions/nameservice`: DNS, /etc/hosts
- `abstractions/ssl-certs`: SSL certificate access
- `abstractions/apache2-common`: Apache specifics
EOF

cat ~/mac_lab/profile_structure.md

# Task 7: Disable AppArmor profile
echo "=== Disabling Profiles ==="

# Disable profile:
sudo aa-disable /usr/sbin/tcpdump

# Verify:
sudo aa-status | grep tcpdump || echo "Profile disabled"

# Alternative method (create symlink to disable):
# sudo ln -s /etc/apparmor.d/usr.sbin.tcpdump /etc/apparmor.d/disable/

# Task 8: Enable disabled profile
echo "=== Enabling Profiles ==="

# Enable profile:
sudo aa-enforce /usr/sbin/tcpdump

# Or if using symlink method:
# sudo rm /etc/apparmor.d/disable/usr.sbin.tcpdump
# sudo apparmor_parser -r /etc/apparmor.d/usr.sbin.tcpdump

# Verify:
sudo aa-status | grep tcpdump

# Task 9-10: Complain and Enforce modes
echo "=== Managing Profile Modes ==="

# Set to complain mode for testing:
sudo aa-complain /usr/sbin/tcpdump

# Test application:
sudo tcpdump -i any -c 5

# Check for denials in complain mode:
sudo grep "apparmor.*tcpdump" /var/log/syslog | tail -5

# Switch to enforce mode:
sudo aa-enforce /usr/sbin/tcpdump

# Part C: Creating Custom AppArmor Profiles

# Task 11: Create test application
echo "=== Creating Test Application ==="

cd ~/mac_lab/testapp

# Create a simple app that accesses various resources:
cat > myapp.py << 'EOF'
#!/usr/bin/env python3
import os
import sys

print("=== Test Application for AppArmor ===")

# Try to read /etc/passwd (should be allowed)
try:
    with open('/etc/passwd', 'r') as f:
        print(f"✓ Read /etc/passwd: {len(f.readlines())} users")
except PermissionError as e:
    print(f"✗ Cannot read /etc/passwd: {e}")

# Try to read /etc/shadow (should be denied)
try:
    with open('/etc/shadow', 'r') as f:
        print(f"✓ Read /etc/shadow: {len(f.readlines())} entries")
except PermissionError as e:
    print(f"✗ Cannot read /etc/shadow: {e}")

# Try to write to /tmp
try:
    with open('/tmp/apparmor-test.txt', 'w') as f:
        f.write("AppArmor test\n")
    print("✓ Wrote to /tmp/apparmor-test.txt")
except PermissionError as e:
    print(f"✗ Cannot write to /tmp: {e}")

# Try to write to home directory
try:
    home_file = os.path.expanduser('~/apparmor-test.txt')
    with open(home_file, 'w') as f:
        f.write("Home directory test\n")
    print(f"✓ Wrote to {home_file}")
except PermissionError as e:
    print(f"✗ Cannot write to home: {e}")

print("\nApplication finished")
EOF

chmod +x myapp.py

# Copy to /usr/local/bin for system-wide access:
sudo cp myapp.py /usr/local/bin/myapp
sudo chmod 755 /usr/local/bin/myapp

# Test without AppArmor:
/usr/local/bin/myapp

# Task 12: Generate AppArmor profile
echo "=== Generating AppArmor Profile ==="

# Install AppArmor utilities:
sudo apt install -y apparmor-utils

# Generate profile interactively:
# sudo aa-genprof /usr/local/bin/myapp
# Then in another terminal: /usr/local/bin/myapp
# Follow prompts to allow/deny operations

# Or create profile manually:
sudo tee /etc/apparmor.d/usr.local.bin.myapp > /dev/null << 'EOF'
#include <tunables/global>

/usr/local/bin/myapp {
  #include <abstractions/base>
  #include <abstractions/python>

  # Python interpreter
  /usr/bin/python3.* r,

  # The application itself
  /usr/local/bin/myapp r,

  # Allow reading /etc/passwd (not sensitive)
  /etc/passwd r,

  # Explicitly deny /etc/shadow
  deny /etc/shadow r,

  # Allow /tmp writes
  /tmp/ r,
  /tmp/* rw,

  # Allow home directory writes (owner only)
  owner @{HOME}/ r,
  owner @{HOME}/** rw,

  # System libraries
  /usr/lib/python3*/** r,

  # Proc access
  @{PROC}/sys/kernel/random/uuid r,
}
EOF

# Load the profile in complain mode:
sudo aa-complain /usr/local/bin/myapp

# Verify:
sudo aa-status | grep myapp

# Task 13: Run and check denials
echo "=== Testing Profile ==="

# Run the application:
/usr/local/bin/myapp

# Check for AppArmor messages:
sudo grep "apparmor.*myapp" /var/log/syslog | tail -20

# Or use audit log:
sudo ausearch -m AVC -c myapp 2>/dev/null || echo "No audit entries (ausearch not available)"

# Task 14: Update profile
echo "=== Updating Profile Based on Denials ==="

# Use aa-logprof to update profile based on logs:
# sudo aa-logprof
# This interactively asks about each denial

# Or manually edit the profile:
sudo nano /etc/apparmor.d/usr.local.bin.myapp

# Reload profile:
sudo apparmor_parser -r /etc/apparmor.d/usr.local.bin.myapp

# Task 15: Enforce profile
echo "=== Enforcing Profile ==="

# Put in enforce mode:
sudo aa-enforce /usr/local/bin/myapp

# Verify mode:
sudo aa-status | grep myapp

# Test application:
/usr/local/bin/myapp

# Should see denials for unauthorized access:
sudo grep "apparmor.*DENIED.*myapp" /var/log/syslog | tail -10

# Part D: Troubleshooting and Auditing

# Task 16: View AppArmor denials
echo "=== Viewing AppArmor Denials ==="

# Recent AppArmor messages:
sudo grep apparmor /var/log/syslog | tail -50

# Only denials:
sudo grep "apparmor.*DENIED" /var/log/syslog | tail -20

# Denials for specific profile:
sudo grep "apparmor.*DENIED.*myapp" /var/log/syslog

# Format denial messages:
sudo grep "apparmor.*DENIED" /var/log/syslog | tail -10 | \
    awk '{print $NF}' | sort | uniq

# Using journalctl:
sudo journalctl | grep apparmor | tail -20

# Task 17: Use aa-logprof
echo "=== Using aa-logprof ==="

cat > ~/mac_lab/logprof_guide.md << 'EOF'
# Using aa-logprof

aa-logprof analyzes system logs and helps update profiles.

## Usage:
```bash
sudo aa-logprof
```

## Workflow:
1. Reads /var/log/syslog for AppArmor events
2. For each denial, offers options:
   - (A)llow: Add permission to profile
   - (D)eny: Add explicit deny rule
   - (I)gnore: Skip this event
   - (G)lob: Use wildcards in path
   - (S)ave: Save changes and continue

3. After reviewing all events, updates profiles

## Example session:
```
Reading log entries from /var/log/syslog.
Updating AppArmor profiles in /etc/apparmor.d.

Profile:  /usr/local/bin/myapp
Path:     /etc/ssl/openssl.cnf
Mode:     r
Severity: 6

 [1 - /etc/ssl/openssl.cnf]
  2 - /etc/ssl/*
  3 - /etc/**

(A)llow / [(D)eny] / (I)gnore / (G)lob / (S)ave Changes / (F)inish
```

## Tips:
- Run after testing application in complain mode
- Use glob patterns for similar paths
- Be conservative - only allow what's needed
- Review changes before saving
EOF

cat ~/mac_lab/logprof_guide.md

# Run aa-logprof (interactive):
# sudo aa-logprof

# Task 18: Use aa-genprof
cat > ~/mac_lab/genprof_guide.md << 'EOF'
# Using aa-genprof

aa-genprof generates new profiles by monitoring application behavior.

## Usage:
```bash
sudo aa-genprof /path/to/application
```

## Workflow:
1. Creates initial basic profile in complain mode
2. Prompts you to run the application
3. In another terminal, exercise all application features
4. Return to aa-genprof and press 'S' to scan logs
5. Review and approve/deny each access
6. Saves completed profile

## Example:
```bash
# Terminal 1:
sudo aa-genprof /usr/local/bin/myapp

# Terminal 2:
/usr/local/bin/myapp
# Test all features

# Terminal 1:
Press 'S' to scan logs
Review each access and choose (A)llow or (D)eny
Press 'F' to finish
```

## Best Practices:
- Thoroughly test application (all code paths)
- Include error conditions
- Test with different user permissions
- Review generated profile for over-permissiveness
EOF

cat ~/mac_lab/genprof_guide.md

# Task 19: Analyze audit logs
echo "=== Analyzing Audit Logs ==="

# Create log analysis script:
cat > ~/mac_lab/analyze_apparmor_logs.sh << 'EOF'
#!/bin/bash

echo "=== AppArmor Log Analysis ==="
echo ""

LOG_FILE="/var/log/syslog"
TIMEFRAME="24 hours"

echo "Analysis Period: Last $TIMEFRAME"
echo "Log File: $LOG_FILE"
echo ""

echo "1. Total AppArmor Events:"
sudo grep apparmor "$LOG_FILE" | wc -l

echo ""
echo "2. Denials by Profile:"
sudo grep "apparmor.*DENIED" "$LOG_FILE" | \
    sed 's/.*profile="\([^"]*\)".*/\1/' | \
    sort | uniq -c | sort -rn

echo ""
echo "3. Most Denied Operations:"
sudo grep "apparmor.*DENIED" "$LOG_FILE" | \
    grep -o 'operation="[^"]*"' | \
    sort | uniq -c | sort -rn | head -10

echo ""
echo "4. Most Denied Paths:"
sudo grep "apparmor.*DENIED" "$LOG_FILE" | \
    grep -o 'name="[^"]*"' | \
    sort | uniq -c | sort -rn | head -10

echo ""
echo "5. Recent Denials (last 10):"
sudo grep "apparmor.*DENIED" "$LOG_FILE" | tail -10

echo ""
echo "6. Profiles in Complain Mode:"
sudo aa-status | grep -A 100 "complain mode" | grep "^   " | wc -l

echo ""
echo "7. Profiles in Enforce Mode:"
sudo aa-status | grep -A 100 "enforce mode" | grep "^   " | wc -l

echo ""
echo "=== Analysis Complete ==="
EOF

chmod +x ~/mac_lab/analyze_apparmor_logs.sh
~/mac_lab/analyze_apparmor_logs.sh

# Task 20: Monitor AppArmor denials
cat > ~/mac_lab/monitor_denials.sh << 'EOF'
#!/bin/bash

# Real-time AppArmor denial monitor

echo "=== AppArmor Denial Monitor ==="
echo "Watching for denials... (Ctrl+C to stop)"
echo ""

# Monitor syslog for denials:
sudo tail -f /var/log/syslog | grep --line-buffered "apparmor.*DENIED" | \
while read -r line; do
    TIMESTAMP=$(echo "$line" | awk '{print $1, $2, $3}')
    PROFILE=$(echo "$line" | grep -o 'profile="[^"]*"' | cut -d'"' -f2)
    OPERATION=$(echo "$line" | grep -o 'operation="[^"]*"' | cut -d'"' -f2)
    NAME=$(echo "$line" | grep -o 'name="[^"]*"' | cut -d'"' -f2)
    
    echo "[$TIMESTAMP] $PROFILE"
    echo "  Operation: $OPERATION"
    echo "  Resource: $NAME"
    echo ""
done
EOF

chmod +x ~/mac_lab/monitor_denials.sh

# Run in background or separate terminal:
# ~/mac_lab/monitor_denials.sh

# Part E: SELinux Concepts

# Task 21-22: SELinux modes and contexts
cat > ~/mac_lab/selinux_overview.md << 'EOF'
# SELinux Overview (for RHEL/CentOS)

## SELinux Modes

### 1. Enforcing
- SELinux policy is enforced
- Access denials are blocked and logged
- **Production mode**

### 2. Permissive
- SELinux policy is not enforced
- Access denials are logged but allowed
- **Troubleshooting mode**

### 3. Disabled
- SELinux is completely disabled
- No logging or enforcement
- **Requires reboot to enable/disable**

## Checking Mode:
```bash
getenforce                    # Current mode
sestatus                      # Detailed status
cat /etc/selinux/config      # Persistent configuration
```

## Changing Mode:
```bash
setenforce 0                 # Temporary: Permissive
setenforce 1                 # Temporary: Enforcing
# Edit /etc/selinux/config for persistent changes
```

## SELinux Contexts

Format: `user:role:type:level`

### Example:
```
system_u:object_r:httpd_sys_content_t:s0
│         │        │                    │
│         │        │                    └─ Level (MLS/MCS)
│         │        └─ Type (most important for access control)
│         └─ Role
└─ User
```

### Context Types:
- **User**: SELinux user (not Linux user)
  - `unconfined_u`: Unconfined user
  - `system_u`: System processes
  - `user_u`: Regular user

- **Role**: Connects users to domains
  - `object_r`: For files
  - `system_r`: For processes

- **Type**: Main access control mechanism
  - Files: `httpd_sys_content_t`, `var_log_t`
  - Processes: `httpd_t`, `sshd_t`

- **Level**: Multi-Level/Category Security
  - `s0`: Lowest level
  - `s0-s0:c0.c1023`: Range

## Viewing Contexts:
```bash
ls -Z                        # File contexts
ps -eZ                       # Process contexts
id -Z                        # User context
```

## Setting Contexts:
```bash
chcon -t httpd_sys_content_t /var/www/html/index.html
restorecon -R /var/www/html  # Restore default contexts
semanage fcontext -a -t httpd_sys_content_t "/web(/.*)?"
```
EOF

cat ~/mac_lab/selinux_overview.md

# Task 23: SELinux basic commands
cat > ~/mac_lab/selinux_commands.md << 'EOF'
# SELinux Basic Commands

## Status and Mode
```bash
getenforce                   # Show current mode
sestatus                     # Show detailed status
setenforce 0                 # Set to permissive (temporary)
setenforce 1                 # Set to enforcing (temporary)
```

## Contexts
```bash
ls -Z file                   # Show file context
ps -eZ                       # Show process contexts
id -Z                        # Show user context
```

## Managing Contexts
```bash
# Temporary context change:
chcon -t type_t file

# Restore default contexts:
restorecon file
restorecon -R /directory

# Permanent context definition:
semanage fcontext -a -t httpd_sys_content_t "/web(/.*)?"
restorecon -R /web
```

## Booleans (Policy switches)
```bash
getsebool -a                 # List all booleans
getsebool httpd_can_network_connect
setsebool httpd_can_network_connect on          # Temporary
setsebool -P httpd_can_network_connect on       # Persistent
```

## Troubleshooting
```bash
# View denials:
ausearch -m AVC -ts recent
aureport -a

# Analyze and suggest fixes:
sealert -a /var/log/audit/audit.log

# Generate policy module from denials:
audit2allow -a
audit2allow -a -M mypolicy
semodule -i mypolicy.pp
```

## Policy Modules
```bash
semodule -l                  # List modules
semodule -i module.pp        # Install module
semodule -r module           # Remove module
```

## Common Scenarios

### Web Server (httpd)
```bash
# Allow httpd to connect to network:
setsebool -P httpd_can_network_connect on

# Set correct context for web files:
semanage fcontext -a -t httpd_sys_content_t "/web(/.*)?"
restorecon -R /web

# Allow httpd to send email:
setsebool -P httpd_can_sendmail on
```

### Database Files
```bash
# PostgreSQL data directory:
semanage fcontext -a -t postgresql_db_t "/pgdata(/.*)?"
restorecon -R /pgdata
```

### NFS
```bash
# Allow NFS home directories:
setsebool -P use_nfs_home_dirs on
```
EOF

cat ~/mac_lab/selinux_commands.md

# Task 24: Compare AppArmor and SELinux
cat > ~/mac_lab/apparmor_vs_selinux.md << 'EOF'
# AppArmor vs SELinux Comparison

| Feature | AppArmor | SELinux |
|---------|----------|---------|
| **Default on** | Ubuntu, SUSE | RHEL, CentOS, Fedora |
| **Approach** | Path-based | Label-based (contexts) |
| **Complexity** | Simpler, easier to learn | More complex, powerful |
| **Profile location** | `/etc/apparmor.d/` | Contexts in filesystem |
| **Configuration** | Text files (easy to read) | Binary policies |
| **Learning curve** | Gentle | Steep |
| **Granularity** | Per-application | System-wide with types |
| **Default stance** | Deny by default | Allow by default (with policy) |

## AppArmor Strengths
- ✓ Easier to understand and configure
- ✓ Path-based (intuitive)
- ✓ Human-readable profiles
- ✓ Faster to create custom profiles
- ✓ Less overhead
- ✓ Simpler troubleshooting

## SELinux Strengths
- ✓ More comprehensive security
- ✓ Label-based (follows files even if moved)
- ✓ Multi-Level Security (MLS)
- ✓ Role-Based Access Control (RBAC)
- ✓ More mature, longer history
- ✓ Better for complex environments

## When to Use Each

### Use AppArmor when:
- Running Ubuntu or SUSE
- Need quick, simple application confinement
- Have specific applications to protect
- Want easier management and troubleshooting
- Team has less SELinux experience

### Use SELinux when:
- Running RHEL, CentOS, or Fedora
- Need comprehensive system-wide security
- Require MLS/MCS
- Have dedicated security team
- Compliance requires SELinux (e.g., some government)

## Example Comparison

### AppArmor Profile:
```apparmor
/usr/bin/myapp {
  /etc/myapp.conf r,
  /var/log/myapp.log w,
  deny /etc/shadow r,
}
```

### SELinux Equivalent:
```bash
# File contexts:
/etc/myapp.conf → myapp_conf_t
/var/log/myapp.log → myapp_log_t

# Process runs as:
myapp_t

# Policy rules (compiled):
allow myapp_t myapp_conf_t:file read;
allow myapp_t myapp_log_t:file { write create };
```

## Migration

### AppArmor → SELinux
1. Document AppArmor profile rules
2. Map paths to SELinux types
3. Create/modify SELinux policy
4. Test in permissive mode
5. Deploy in enforcing mode

### SELinux → AppArmor
1. Analyze SELinux denials and allows
2. Map types to paths
3. Create AppArmor profile
4. Test in complain mode
5. Deploy in enforce mode

## Recommendation
- **Don't mix**: Use one MAC system per distribution
- **Stick with defaults**: Ubuntu → AppArmor, RHEL → SELinux
- **Learn both concepts**: Valuable for multi-distribution environments
EOF

cat ~/mac_lab/apparmor_vs_selinux.md

# Task 25: Document security policies
cat > ~/mac_lab/security_policy_template.md << 'EOF'
# Security Policy Documentation Template

## Application: [Application Name]
**Version**: [Version]  
**MAC System**: AppArmor / SELinux  
**Profile Mode**: Enforce / Complain  
**Last Updated**: [Date]

## Overview
Brief description of the application and its security requirements.

## Security Objectives
- Objective 1: Limit file system access
- Objective 2: Restrict network capabilities
- Objective 3: Prevent privilege escalation

## Allowed Operations

### File System Access
| Path | Permission | Justification |
|------|------------|---------------|
| `/etc/app.conf` | Read | Configuration file |
| `/var/log/app.log` | Write | Logging |
| `/var/lib/app/` | Read/Write | Application data |

### Network Access
| Type | Details | Justification |
|------|---------|---------------|
| TCP | Port 80, 443 | Web server |
| UDP | Port 53 | DNS queries |

### Capabilities
| Capability | Justification |
|------------|---------------|
| `net_bind_service` | Bind to ports < 1024 |
| `chown` | Change file ownership |

## Denied Operations
- `/etc/shadow` - Password file access
- `/root/` - Root directory access
- Raw network access
- Kernel module loading

## Profile Location
**AppArmor**: `/etc/apparmor.d/usr.bin.application`  
**SELinux**: Type `application_t`, Context rules in policy

## Testing Procedure
1. Enable profile in complain mode
2. Run application test suite
3. Review audit logs
4. Update profile based on legitimate denials
5. Re-test in enforce mode
6. Monitor for one week
7. Review and finalize

## Troubleshooting
Common issues and resolutions:
- Issue 1: [Description] → Solution: [Fix]
- Issue 2: [Description] → Solution: [Fix]

## Maintenance
- Review quarterly
- Update after application changes
- Monitor denial logs weekly

## References
- Profile source: [Git/URL]
- Related documentation: [Links]
- Compliance requirements: [Standards]
EOF

cat ~/mac_lab/security_policy_template.md

# Create comprehensive MAC management script:
cat > ~/mac_lab/mac_manager.sh << 'EOF'
#!/bin/bash

# Mandatory Access Control Management Script

check_mac_system() {
    if command -v aa-status &> /dev/null; then
        echo "AppArmor"
    elif command -v getenforce &> /dev/null; then
        echo "SELinux"
    else
        echo "None"
    fi
}

MAC_SYSTEM=$(check_mac_system)

echo "=== MAC Management Tool ==="
echo "Detected MAC System: $MAC_SYSTEM"
echo ""

if [ "$MAC_SYSTEM" = "AppArmor" ]; then
    cat << 'MENU'
1. Show AppArmor status
2. List profiles (enforce mode)
3. List profiles (complain mode)
4. Put profile in complain mode
5. Put profile in enforce mode
6. Disable profile
7. Reload profile
8. View recent denials
9. Analyze logs
10. Monitor denials (real-time)
11. Quit
MENU

    read -p "Select option: " choice
    
    case $choice in
        1) sudo aa-status ;;
        2) sudo aa-status | grep -A 100 "enforce mode" ;;
        3) sudo aa-status | grep -A 100 "complain mode" ;;
        4) read -p "Profile path: " profile
           sudo aa-complain "$profile" ;;
        5) read -p "Profile path: " profile
           sudo aa-enforce "$profile" ;;
        6) read -p "Profile path: " profile
           sudo aa-disable "$profile" ;;
        7) read -p "Profile path: " profile
           sudo apparmor_parser -r "$profile" ;;
        8) sudo grep "apparmor.*DENIED" /var/log/syslog | tail -20 ;;
        9) ~/mac_lab/analyze_apparmor_logs.sh ;;
        10) ~/mac_lab/monitor_denials.sh ;;
        11) exit 0 ;;
    esac

elif [ "$MAC_SYSTEM" = "SELinux" ]; then
    echo "SELinux commands (for reference):"
    echo "  getenforce - Show mode"
    echo "  sestatus - Show status"
    echo "  setenforce 0/1 - Change mode"
    echo "  ls -Z - Show contexts"
    echo "  ausearch -m AVC - Show denials"
else
    echo "No MAC system detected"
    echo "Install AppArmor or SELinux"
fi
EOF

chmod +x ~/mac_lab/mac_manager.sh

echo ""
echo "=== MAC Lab Setup Complete ==="
echo ""
echo "Scripts created:"
echo "  ~/mac_lab/analyze_apparmor_logs.sh - Analyze denial patterns"
echo "  ~/mac_lab/monitor_denials.sh - Real-time denial monitoring"
echo "  ~/mac_lab/mac_manager.sh - Interactive MAC management"
echo ""
echo "Documentation created:"
echo "  ~/mac_lab/apparmor_modes.md - Profile mode reference"
echo "  ~/mac_lab/selinux_overview.md - SELinux concepts"
echo "  ~/mac_lab/apparmor_vs_selinux.md - Comparison guide"
echo "  ~/mac_lab/security_policy_template.md - Policy documentation"
```

**Explanation**:
- **Task 1**: `aa-status` shows all loaded profiles and their modes (enforce/complain)
- **Task 4**: Complain mode logs violations without blocking; useful for profile development
- **Task 12**: AppArmor profiles use path-based rules; SELinux uses label-based contexts
- **Task 17**: `aa-logprof` helps update profiles by analyzing denial logs interactively
- **Task 21**: SELinux has three modes; changing to/from disabled requires reboot
- **Task 22**: SELinux contexts format: `user:role:type:level` where type is most important for access control
- **Task 24**: AppArmor is simpler and path-based; SELinux is more complex but more powerful with label-based security

**AppArmor Essential Commands**:
```bash
# Status
aa-status                              # Show all profiles and status
aa-enabled                             # Check if AppArmor is enabled

# Profile Management
aa-enforce /path/to/profile            # Set to enforce mode
aa-complain /path/to/profile           # Set to complain mode
aa-disable /path/to/profile            # Disable profile

# Profile Development
aa-genprof /path/to/binary             # Generate new profile
aa-logprof                             # Update profiles from logs

# Reloading
apparmor_parser -r /etc/apparmor.d/*   # Reload all profiles
systemctl reload apparmor              # Reload service

# Logs
grep apparmor /var/log/syslog          # View AppArmor events
grep "apparmor.*DENIED" /var/log/syslog # View denials only
```

**SELinux Essential Commands** (for reference):
```bash
# Status and Mode
getenforce                             # Get current mode
setenforce 0                           # Permissive (temporary)
setenforce 1                           # Enforcing (temporary)
sestatus                               # Detailed status

# Contexts
ls -Z                                  # File contexts
ps -eZ                                 # Process contexts
id -Z                                  # User context
chcon -t type_t file                   # Change context (temporary)
restorecon -R /path                    # Restore default contexts

# Booleans
getsebool -a                           # List all booleans
setsebool -P boolean on                # Set boolean permanently

# Troubleshooting
ausearch -m AVC -ts recent             # Recent denials
audit2allow -a                         # Generate policy from denials
sealert -a /var/log/audit/audit.log   # Analyze denials with suggestions
```

**Security Best Practices**:
1. **Start in complain/permissive mode** - Test thoroughly before enforcing
2. **Monitor logs regularly** - Watch for unexpected denials
3. **Principle of least privilege** - Only allow what's necessary
4. **Document everything** - Maintain profile documentation
5. **Test after updates** - Applications may need different permissions after updates
6. **Use standard abstractions** - Leverage existing profile templates
7. **Regular audits** - Review profiles quarterly
8. **Backup profiles** - Version control for security policies

</details>

### Extensions
1. **Advanced**: Create a complete AppArmor profile for a complex application stack (web server + database + caching layer). Include complain mode testing, log analysis, profile optimization, and documentation. Implement automated testing to verify profiles don't break functionality
2. **Challenge**: Build a security compliance framework that: automatically audits MAC configuration across multiple servers, detects profile violations, generates security reports, maintains profile version control, implements automated remediation for common issues, and provides a dashboard showing security posture across the infrastructure

---

**🎉 Operation of Running Systems - Complete! 🎉**

**Summary**: This comprehensive guide covers all major aspects of the "Operation of Running Systems" domain:

**Part 1** (Tasks 1-3):
- Kernel parameter configuration with sysctl
- Process troubleshooting and performance analysis
- Job scheduling with cron, at, and systemd timers

**Part 2** (Tasks 4-6):
- Software package management with apt/dpkg
- System recovery and boot troubleshooting
- Virtual machine management with KVM/QEMU

**Part 3** (Tasks 7-8):
- Container management with Docker and Podman
- Mandatory Access Control with AppArmor and SELinux

Each task includes realistic scenarios, comprehensive solutions, and practical exercises to prepare you for the LFCS exam. All content follows the FORMATTING_GUIDE.md standards for consistency.

**What's next?** Let me know if you'd like to continue with another LFCS domain!

---

## Task 9: SELinux Configuration and Troubleshooting (RHEL/CentOS Focus)

### Learning Objectives
- Install and configure SELinux on RHEL-based systems
- Manage SELinux modes (enforcing, permissive, disabled)
- Work with SELinux contexts, labels, and file contexts
- Configure SELinux booleans for service customization
- Troubleshoot SELinux denials using audit logs
- Create and manage custom SELinux policies

### Context
Your organization is deploying production services on RHEL 9/CentOS Stream servers where SELinux is enabled by default. The security team requires all systems to run with SELinux in enforcing mode for compliance. You need to deploy a web application that stores files in a non-standard location, configure an SSH server on a custom port, set up a database server with custom data directories, and troubleshoot applications that fail due to SELinux restrictions. You must understand how to read audit logs, modify contexts, use booleans appropriately, and create custom policy modules when necessary. The challenge is to maintain security while ensuring all services function correctly without simply disabling SELinux.

SELinux is a critical LFCS competency, especially for RHEL/CentOS environments. Understanding SELinux is essential for any Linux administrator working with Red Hat-based distributions, and knowing how to troubleshoot and configure it properly is a key differentiator between junior and senior administrators.

### Task Instructions

**Prerequisites**: SELinux lab environment setup
```bash
# Note: This task assumes RHEL 9, CentOS Stream 9, or similar
# For Ubuntu users: Consider using a RHEL VM for practice

# Create working directory
mkdir -p ~/selinux_lab/{policies,scripts,logs,webapp}
cd ~/selinux_lab

# Verify SELinux is available
getenforce || echo "SELinux not available - use RHEL/CentOS VM"
```

**Your Tasks**:

**Part A: SELinux Basics and Mode Management**
1. Check SELinux status and current mode
2. Understand the difference between targeted and MLS policies
3. Temporarily change SELinux mode (enforcing ↔ permissive)
4. Permanently change SELinux mode via configuration
5. Understand when to use each mode and reboot requirements

**Part B: SELinux Contexts and Labels**
6. View SELinux contexts for files, processes, and ports
7. Understand context format (user:role:type:level)
8. Temporarily change file contexts using `chcon`
9. Permanently set file contexts using `semanage fcontext`
10. Restore default contexts using `restorecon`

**Part C: SELinux Booleans**
11. List all SELinux booleans and their current states
12. Query specific boolean values
13. Temporarily enable/disable booleans
14. Permanently configure booleans
15. Identify which booleans to use for common services

**Part D: Port and Network Configuration**
16. List SELinux port types and assignments
17. Allow services to bind to non-standard ports
18. Configure SELinux for custom network services
19. Manage port contexts with `semanage port`

**Part E: Troubleshooting SELinux Denials**
20. Read and interpret AVC (Access Vector Cache) denial messages
21. Use `ausearch` to find SELinux denials
22. Use `aureport` to generate audit reports
23. Use `sealert` to analyze denials and get recommendations
24. Determine root cause of denials vs symptoms

**Part F: Custom SELinux Policies**
25. Use `audit2allow` to generate policy modules from denials
26. Create and compile custom policy modules
27. Install and manage policy modules with `semodule`
28. Test and validate custom policies
29. Create a complete workflow for handling new application deployment

### Hints and Resources
1. Main tools: `getenforce`, `setenforce`, `sestatus`, `semanage`, `restorecon`
2. Audit tools: `ausearch`, `aureport`, `sealert`, `audit2allow`, `audit2why`
3. Configuration file: `/etc/selinux/config`
4. Audit log: `/var/log/audit/audit.log`
5. References: [RHEL SELinux Guide](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9/html/using_selinux/), [SELinux Project Wiki](https://selinuxproject.org/)

### Estimated Time and Difficulty
**50-70 minutes, Advanced**

### Verification
To verify your solution:
- `getenforce` shows expected mode (Enforcing/Permissive)
- `sestatus` displays detailed SELinux status
- `ls -Z` shows correct contexts for files
- `ps -eZ` shows process contexts
- `semanage boolean -l` lists configured booleans
- `semanage port -l` shows port type assignments
- `ausearch -m AVC` finds denial messages
- Services run correctly with SELinux in enforcing mode
- `semodule -l` shows installed custom modules

<details>
<summary>Solution</summary>

```bash
# Part A: SELinux Basics and Mode Management

# Task 1: Check SELinux status
echo "=== SELinux Status Check ==="

# Quick mode check:
getenforce
# Returns: Enforcing, Permissive, or Disabled

# Detailed status:
sestatus

# Output explanation:
# SELinux status:                 enabled/disabled
# SELinuxfs mount:                /sys/fs/selinux
# SELinux root directory:         /etc/selinux
# Loaded policy name:             targeted
# Current mode:                   enforcing/permissive
# Mode from config file:          enforcing/permissive
# Policy MLS status:              enabled/disabled
# Policy deny_unknown status:     allowed
# Memory protection checking:     actual (secure)
# Max kernel policy version:      33

# Check if SELinux is enabled in kernel:
cat /proc/cmdline | grep selinux

# View configuration file:
cat /etc/selinux/config

# Task 2: Understand policy types
cat > ~/selinux_lab/policy_types.md << 'EOF'
# SELinux Policy Types

## 1. Targeted Policy (Default)
- **File**: `/etc/selinux/targeted/`
- **Description**: Targets specific system services
- **Default on**: RHEL, CentOS, Fedora
- **Coverage**: Network-facing services confined, most user processes unconfined
- **Examples**: httpd, sshd, named, mysql run confined
- **User processes**: Run as `unconfined_t` by default
- **Use case**: Standard servers and workstations

## 2. MLS (Multi-Level Security) Policy
- **File**: `/etc/selinux/mls/`
- **Description**: Multi-level security with classification levels
- **Coverage**: All processes and files have security levels
- **Levels**: Like Top Secret, Secret, Confidential, Unclassified
- **Complexity**: Much more complex to configure
- **Use case**: Government, military, high-security environments
- **Example**: s0 (unclassified) to s15 (top secret)

## 3. Minimum Policy
- **File**: `/etc/selinux/minimum/`
- **Description**: Minimal set of confined services
- **Coverage**: Only critical services confined
- **Use case**: Systems where performance is critical

## Policy Selection in /etc/selinux/config:
```
SELINUX=enforcing
SELINUXTYPE=targeted    # or mls, or minimum
```

## Checking Current Policy:
```bash
sestatus | grep "Loaded policy"
ls -la /etc/selinux/
```
EOF

cat ~/selinux_lab/policy_types.md

# View available policies:
ls -la /etc/selinux/

# Task 3: Temporarily change SELinux mode
echo "=== Changing SELinux Mode Temporarily ==="

# Check current mode:
getenforce

# Set to Permissive (allows but logs denials):
sudo setenforce 0

# Verify:
getenforce

# Set back to Enforcing:
sudo setenforce 1

# Verify:
getenforce

# Note: These changes are lost on reboot

# Task 4: Permanently change SELinux mode
echo "=== Permanently Changing SELinux Mode ==="

# View current configuration:
cat /etc/selinux/config

# Backup configuration:
sudo cp /etc/selinux/config /etc/selinux/config.backup

# Show configuration options:
cat > ~/selinux_lab/selinux_config_guide.md << 'EOF'
# /etc/selinux/config Options

## SELINUX= modes:

### enforcing
- SELinux policy is enforced
- Access denials are blocked and logged
- **Production recommended**

### permissive
- SELinux policy is NOT enforced
- Access denials are logged but allowed
- **Troubleshooting mode**

### disabled
- SELinux is completely disabled
- **Requires reboot to change to/from disabled**

## SELINUXTYPE= policies:
- targeted (default)
- mls (multi-level security)
- minimum (minimal confinement)

## Example Configuration:
```
# This file controls the state of SELinux on the system.
SELINUX=enforcing
SELINUXTYPE=targeted
```

## Changing Configuration:
```bash
sudo vi /etc/selinux/config
# Edit SELINUX= line
# Reboot if changing to/from disabled
sudo reboot
```
EOF

cat ~/selinux_lab/selinux_config_guide.md

# To change permanently (example - don't actually do this now):
# sudo sed -i 's/^SELINUX=.*/SELINUX=permissive/' /etc/selinux/config
# sudo reboot

# Task 5: Mode usage guidelines
cat > ~/selinux_lab/mode_usage.md << 'EOF'
# When to Use Each SELinux Mode

## Enforcing Mode
**Use when:**
- ✓ Production systems
- ✓ Security compliance required
- ✓ System is properly configured
- ✓ All applications working correctly

**Don't use when:**
- ✗ Initially setting up complex applications
- ✗ Troubleshooting application issues
- ✗ Learning/testing new configurations

## Permissive Mode
**Use when:**
- ✓ Troubleshooting denials
- ✓ Testing new application deployments
- ✓ Developing custom policies
- ✓ Verifying SELinux is the issue
- ✓ Learning SELinux

**Don't use when:**
- ✗ Running in production (unless temporarily troubleshooting)
- ✗ Security is critical

## Disabled Mode
**Use when:**
- ✓ Hardware/software incompatibility (rare)
- ✓ Specific vendor requirement (rare)

**Don't use when:**
- ✗ You can fix issues with permissive/booleans/policies
- ✗ Security is a concern
- ✗ You need compliance (PCI-DSS, HIPAA, etc.)

## ⚠️ Important Notes:
- Changing to/from disabled requires relabeling entire filesystem (slow)
- Prefer permissive over disabled for troubleshooting
- Always try to fix SELinux issues rather than disable
- Disabling SELinux often fails security audits

## Reboot Requirements:
- Enforcing ↔ Permissive: **No reboot** needed
- Any ↔ Disabled: **Reboot required**
- Filesystem relabeling: **Automatic on first boot** after enabling
EOF

cat ~/selinux_lab/mode_usage.md

# Part B: SELinux Contexts and Labels

# Task 6: View SELinux contexts
echo "=== Viewing SELinux Contexts ==="

# File contexts:
ls -Z /etc/passwd
ls -Z /var/www/html/
ls -lZ /home/

# Process contexts:
ps -eZ | head -20

# Specific process:
ps -eZ | grep httpd
ps -eZ | grep sshd

# Current user context:
id -Z

# Port contexts:
sudo semanage port -l | head -20

# Example outputs and meanings:
cat > ~/selinux_lab/context_examples.md << 'EOF'
# SELinux Context Examples

## Format: user:role:type:level

### File Context Examples:

```
system_u:object_r:etc_t:s0                    /etc/passwd
system_u:object_r:httpd_sys_content_t:s0      /var/www/html/index.html
unconfined_u:object_r:user_home_t:s0          /home/user/file.txt
system_u:object_r:var_log_t:s0                /var/log/messages
```

### Process Context Examples:

```
system_u:system_r:httpd_t:s0                  httpd process
system_u:system_r:sshd_t:s0-s0:c0.c1023      sshd process
unconfined_u:unconfined_r:unconfined_t:s0     user shell
```

## Context Components:

### User (first field):
- `unconfined_u`: Unconfined user (normal users)
- `system_u`: System processes
- `user_u`: Confined user
- `staff_u`: Staff user (more privileges than user_u)
- `root`: Root user (in some policies)

### Role (second field):
- `object_r`: Files and directories
- `system_r`: System processes
- `unconfined_r`: Unconfined processes
- `user_r`: User processes

### Type (third field) - MOST IMPORTANT:
- Files:
  - `httpd_sys_content_t`: Web server content
  - `httpd_log_t`: Web server logs
  - `user_home_t`: User home directory
  - `tmp_t`: Temporary files
  - `var_log_t`: System logs

- Processes:
  - `httpd_t`: Apache web server
  - `sshd_t`: SSH daemon
  - `mysqld_t`: MySQL database
  - `unconfined_t`: Unconfined process

### Level (fourth field) - MLS/MCS:
- `s0`: Lowest sensitivity level (default in targeted policy)
- `s0-s0:c0.c1023`: Range with categories
- In MLS: `s0` to `s15` (unclassified to top secret)

## Access Control:
SELinux allows process with type A to access file with type B
based on policy rules.

Example: `allow httpd_t httpd_sys_content_t:file { read open };`
EOF

cat ~/selinux_lab/context_examples.md

# Task 7: Understand context format
# Covered in previous output

# Task 8: Temporarily change file context
echo "=== Temporarily Changing File Contexts ==="

# Create test file:
touch ~/selinux_lab/testfile
ls -Z ~/selinux_lab/testfile

# Change type to httpd content:
sudo chcon -t httpd_sys_content_t ~/selinux_lab/testfile

# Verify:
ls -Z ~/selinux_lab/testfile

# Change entire context:
sudo chcon -u system_u -r object_r -t tmp_t ~/selinux_lab/testfile

# Verify:
ls -Z ~/selinux_lab/testfile

# Copy context from another file:
sudo chcon --reference=/etc/passwd ~/selinux_lab/testfile

# Verify:
ls -Z ~/selinux_lab/testfile

# ⚠️ WARNING: chcon changes are temporary!
# They're lost when running restorecon or relabeling

# Task 9: Permanently set file contexts
echo "=== Permanently Setting File Contexts ==="

# Create custom web directory:
sudo mkdir -p /web/mysite
echo "<h1>Custom Web Location</h1>" | sudo tee /web/mysite/index.html

# Check current context (wrong for web server):
ls -Zd /web/
ls -Z /web/mysite/

# Set permanent context mapping:
sudo semanage fcontext -a -t httpd_sys_content_t "/web(/.*)?"

# Verify the mapping was added:
sudo semanage fcontext -l | grep "^/web"

# Apply the context (restorecon reads the mapping):
sudo restorecon -Rv /web

# Verify:
ls -Zd /web/
ls -Z /web/mysite/

# Now even after relabeling, context persists!

# List all custom contexts:
sudo semanage fcontext -l -C

# Delete custom context (if needed):
# sudo semanage fcontext -d "/web(/.*)?"

# Task 10: Restore default contexts
echo "=== Restoring Default Contexts ==="

# Restore single file:
sudo restorecon -v ~/selinux_lab/testfile

# Restore directory recursively:
sudo restorecon -Rv /var/www/html/

# Preview without changing:
sudo restorecon -Rvn /var/www/html/

# Force restore (even if already correct):
sudo restorecon -RvF /var/www/html/

# Common use case - after copying files:
# cp /tmp/index.html /var/www/html/  # Wrong context!
# sudo restorecon -v /var/www/html/index.html  # Fix it

# Part C: SELinux Booleans

# Task 11: List all booleans
echo "=== SELinux Booleans ==="

# List all booleans:
sudo getsebool -a | head -20

# Count total booleans:
sudo getsebool -a | wc -l

# List with descriptions:
sudo semanage boolean -l | head -20

# Task 12: Query specific boolean
echo "=== Querying Booleans ==="

# Check httpd-related booleans:
sudo getsebool -a | grep httpd

# Check specific boolean:
sudo getsebool httpd_can_network_connect

# Get boolean description:
sudo semanage boolean -l | grep httpd_can_network_connect

# Task 13: Temporarily enable/disable booleans
echo "=== Temporarily Changing Booleans ==="

# Check current value:
sudo getsebool httpd_can_network_connect

# Enable temporarily (lost on reboot):
sudo setsebool httpd_can_network_connect on

# Verify:
sudo getsebool httpd_can_network_connect

# Disable temporarily:
sudo setsebool httpd_can_network_connect off

# Task 14: Permanently configure booleans
echo "=== Permanently Configuring Booleans ==="

# Enable permanently (survives reboot):
sudo setsebool -P httpd_can_network_connect on

# Verify:
sudo getsebool httpd_can_network_connect

# Check permanent vs current value:
sudo semanage boolean -l | grep httpd_can_network_connect

# Task 15: Common service booleans
cat > ~/selinux_lab/common_booleans.md << 'EOF'
# Common SELinux Booleans

## Web Server (httpd/nginx)

### Network Access
```bash
# Allow web server to make network connections (proxy, API calls)
setsebool -P httpd_can_network_connect on

# Allow web server to connect to databases
setsebool -P httpd_can_network_connect_db on

# Allow web server to send email
setsebool -P httpd_can_sendmail on
```

### File System
```bash
# Allow httpd to read user home directories
setsebool -P httpd_enable_homedirs on

# Allow httpd to use NFS
setsebool -P httpd_use_nfs on

# Allow httpd to use CIFS/SMB
setsebool -P httpd_use_cifs on

# Allow httpd scripts to connect to network
setsebool -P httpd_can_network_relay on
```

## SSH Server

```bash
# Allow SSH to forward ports
setsebool -P ssh_sysadm_login on
```

## Samba

```bash
# Allow Samba to share home directories
setsebool -P samba_enable_home_dirs on

# Allow Samba to share any file/directory
setsebool -P samba_export_all_rw on
```

## NFS

```bash
# Allow NFS to be used as home directories
setsebool -P use_nfs_home_dirs on

# Allow services to use NFS
setsebool -P nfs_export_all_rw on
```

## FTP

```bash
# Allow FTP to read/write files in user home directories
setsebool -P ftp_home_dir on

# Allow FTP full access
setsebool -P ftpd_full_access on
```

## MySQL/PostgreSQL

```bash
# Allow database to connect to network (replication)
setsebool -P mysql_connect_any on
```

## General

```bash
# Allow all daemons to write to /tmp
setsebool -P allow_daemons_use_tty on

# Allow users to execute files in /tmp and /home
setsebool -P allow_user_exec_content on
```

## Finding the Right Boolean:
```bash
# Search for booleans related to service:
getsebool -a | grep httpd
getsebool -a | grep samba
getsebool -a | grep ftp

# Get description:
semanage boolean -l | grep boolean_name
```
EOF

cat ~/selinux_lab/common_booleans.md

# Part D: Port and Network Configuration

# Task 16: List SELinux port types
echo "=== SELinux Port Management ==="

# List all port assignments:
sudo semanage port -l | head -30

# List specific service ports:
sudo semanage port -l | grep http
sudo semanage port -l | grep ssh
sudo semanage port -l | grep mysql

# Task 17: Allow services on non-standard ports
echo "=== Configuring Non-Standard Ports ==="

# Example: Run SSH on port 2222
# Check current SSH port assignments:
sudo semanage port -l | grep ssh

# Add port 2222 to ssh_port_t type:
sudo semanage port -a -t ssh_port_t -p tcp 2222

# Verify:
sudo semanage port -l | grep ssh

# Now sshd can bind to port 2222

# Example: Run HTTP on port 8080
sudo semanage port -a -t http_port_t -p tcp 8080

# Verify:
sudo semanage port -l | grep http_port_t

# Remove port assignment (if needed):
# sudo semanage port -d -t http_port_t -p tcp 8080

# Task 18-19: Custom network services
# Example covered above - use semanage port to assign types

# Part E: Troubleshooting SELinux Denials

# Task 20: Read AVC denial messages
echo "=== Understanding AVC Denials ==="

cat > ~/selinux_lab/avc_denial_guide.md << 'EOF'
# Understanding AVC (Access Vector Cache) Denial Messages

## Example Denial in /var/log/audit/audit.log:

```
type=AVC msg=audit(1703865600.123:456): avc: denied { read } for pid=1234 comm="httpd" name="file.txt" dev="sda1" ino=567890 scontext=system_u:system_r:httpd_t:s0 tcontext=unconfined_u:object_r:user_home_t:s0 tclass=file permissive=0
```

## Breaking Down the Message:

| Component | Value | Meaning |
|-----------|-------|---------|
| `type=AVC` | - | SELinux denial |
| `msg=audit(...)` | Timestamp:ID | When and unique ID |
| `denied { read }` | read | Denied action |
| `pid=1234` | 1234 | Process ID |
| `comm="httpd"` | httpd | Command name |
| `name="file.txt"` | file.txt | Target file |
| `scontext=...httpd_t` | httpd_t | Source (process) type |
| `tcontext=...user_home_t` | user_home_t | Target (file) type |
| `tclass=file` | file | Object class |
| `permissive=0` | 0 | Enforcing (1=permissive) |

## Translation:
"The httpd process (running as httpd_t) was denied read access to file.txt (labeled as user_home_t)"

## Common Denied Actions:
- `{ read }` - Read file
- `{ write }` - Write file
- `{ open }` - Open file
- `{ getattr }` - Get file attributes
- `{ execute }` - Execute file
- `{ bind }` - Bind to network port
- `{ connect }` - Connect to network
- `{ name_bind }` - Bind to named port

## Resolution Steps:
1. Identify source type (process)
2. Identify target type (resource)
3. Determine if denial is legitimate
4. Choose fix:
   - Fix file context (most common)
   - Enable boolean
   - Create custom policy (last resort)
EOF

cat ~/selinux_lab/avc_denial_guide.md

# Task 21: Use ausearch
echo "=== Using ausearch for SELinux Denials ==="

# Search for all AVC denials:
sudo ausearch -m AVC -ts today

# Recent denials (last 10 minutes):
sudo ausearch -m AVC -ts recent

# Denials for specific command:
sudo ausearch -m AVC -c httpd

# Denials since specific time:
sudo ausearch -m AVC -ts 10:00:00

# Denials with interpreted output:
sudo ausearch -m AVC -i -ts today

# Save to file:
sudo ausearch -m AVC -ts today > ~/selinux_lab/denials_today.txt

# Task 22: Use aureport
echo "=== Using aureport for Audit Reports ==="

# AVC summary report:
sudo aureport -a

# Detailed AVC report:
sudo aureport --avc

# Report for specific time:
sudo aureport -a -ts today

# Report with summary:
sudo aureport --summary -i

# Task 23: Use sealert
echo "=== Using sealert for Analysis ==="

# Install setroubleshoot if not available:
# sudo dnf install -y setroubleshoot-server

# Analyze audit log:
sudo sealert -a /var/log/audit/audit.log | head -100

# Get analysis for specific ID:
# sudo sealert -l <alert-id>

# Example sealert output provides:
# - What was denied
# - Why it was denied
# - Suggested fixes (booleans, context changes, custom policy)

# Create sealert guide:
cat > ~/selinux_lab/sealert_guide.md << 'EOF'
# Using sealert for SELinux Troubleshooting

## Installation:
```bash
sudo dnf install -y setroubleshoot-server
```

## Basic Usage:
```bash
# Analyze entire audit log:
sudo sealert -a /var/log/audit/audit.log

# Get specific alert details:
sudo sealert -l <alert-id>

# Browse alerts interactively (GUI):
sealert -b
```

## Example sealert Output:

```
SELinux is preventing /usr/sbin/httpd from read access on the file /home/user/public_html/index.html.

***** Plugin catchall_boolean (47.5 confidence) suggests   ***************

If you want to allow httpd to read user content
Then you must tell SELinux about this by enabling the 'httpd_enable_homedirs' boolean.

Do:
setsebool -P httpd_enable_homedirs 1

***** Plugin catchall (6.38 confidence) suggests   **************************

If you believe that httpd should be allowed read access on the index.html file by default.
Then you should report this as a bug.

You can generate a local policy module to allow this access.
Do:
# ausearch -c 'httpd' --raw | audit2allow -M my-httpd
# semodule -X 300 -i my-httpd.pp
```

## Interpreting sealert:
1. **Problem description**: What was denied
2. **Suggestions ranked by confidence**:
   - Boolean toggle (easiest)
   - Context fix
   - Custom policy (last resort)
3. **Commands to fix**: Ready to copy/paste

## Best Practice:
- Always review suggestions before applying
- Prefer booleans > context fixes > custom policies
- Understand why denial occurred
- Don't blindly run suggested commands
EOF

cat ~/selinux_lab/sealert_guide.md

# Task 24: Determine root cause
cat > ~/selinux_lab/troubleshooting_workflow.md << 'EOF'
# SELinux Troubleshooting Workflow

## Step 1: Verify SELinux is the Problem
```bash
# Check if in enforcing mode:
getenforce

# Temporarily set to permissive:
sudo setenforce 0

# Test if problem goes away
# If yes → SELinux is the cause
# If no → Look elsewhere

# Set back to enforcing:
sudo setenforce 1
```

## Step 2: Find the Denial
```bash
# Check recent denials:
sudo ausearch -m AVC -ts recent

# Or use sealert:
sudo sealert -a /var/log/audit/audit.log | tail -100
```

## Step 3: Analyze the Denial
Identify:
- **Source context** (what process)
- **Target context** (what resource)
- **Action** (what was attempted)
- **Why denied** (type mismatch)

## Step 4: Choose Fix Strategy

### Strategy 1: Fix File Context (Most Common)
**When**: File has wrong type label

```bash
# Check file context:
ls -Z /path/to/file

# Should be X, but is Y?

# Fix permanently:
sudo semanage fcontext -a -t correct_type_t "/path/to/file(/.*)?"
sudo restorecon -Rv /path/to/file
```

### Strategy 2: Enable Boolean
**When**: Policy allows it, but boolean is off

```bash
# Find relevant boolean:
getsebool -a | grep service_name

# Enable it:
sudo setsebool -P boolean_name on
```

### Strategy 3: Allow Custom Port
**When**: Service on non-standard port

```bash
# Add port:
sudo semanage port -a -t service_port_t -p tcp 1234
```

### Strategy 4: Custom Policy (Last Resort)
**When**: Legitimate access that policy doesn't allow

```bash
# Generate policy module:
sudo ausearch -c 'program' --raw | audit2allow -M mypolicy

# Review policy file:
cat mypolicy.te

# Install if looks correct:
sudo semodule -i mypolicy.pp
```

## Step 5: Test and Verify
```bash
# Test the application
# Monitor for new denials:
sudo ausearch -m AVC -ts recent

# Verify in enforcing mode:
getenforce
```

## ⚠️ DO NOT:
- ✗ Immediately disable SELinux
- ✗ Blindly run audit2allow suggestions
- ✗ Use audit2allow as first resort
- ✗ Give processes unconfined_t type

## ✓ DO:
- ✓ Understand the denial
- ✓ Fix the root cause
- ✓ Use least privilege principle
- ✓ Document custom policies
- ✓ Test thoroughly
EOF

cat ~/selinux_lab/troubleshooting_workflow.md

# Part F: Custom SELinux Policies

# Task 25: Use audit2allow
echo "=== Using audit2allow ==="

# View denials in human-readable policy format:
sudo ausearch -m AVC -ts recent | audit2allow

# Generate policy module:
sudo ausearch -m AVC -ts recent | audit2allow -M mycustom

# This creates:
# - mycustom.te (policy source)
# - mycustom.pp (compiled policy module)

# View the policy source:
cat mycustom.te 2>/dev/null || echo "No recent denials to create policy"

# Task 26: Create and compile policy module
echo "=== Creating Custom Policy Module ==="

# Example: Create policy for custom app
cat > ~/selinux_lab/example_policy.te << 'EOF'
module example_policy 1.0;

require {
    type httpd_t;
    type user_home_t;
    class file { read open getattr };
}

#============= httpd_t ==============
# Allow httpd to read files in user home directories
allow httpd_t user_home_t:file { read open getattr };
EOF

# Compile policy module:
cd ~/selinux_lab
checkmodule -M -m -o example_policy.mod example_policy.te
semodule_package -o example_policy.pp -m example_policy.mod

# Task 27: Install and manage modules
echo "=== Managing Policy Modules ==="

# List installed modules:
sudo semodule -l | head -20

# List custom modules:
sudo semodule -l | grep -v "^100"

# Install module:
# sudo semodule -i ~/selinux_lab/example_policy.pp

# Remove module:
# sudo semodule -r example_policy

# Disable module (without removing):
# sudo semodule -d example_policy

# Enable disabled module:
# sudo semodule -e example_policy

# Task 28-29: Complete deployment workflow
cat > ~/selinux_lab/deployment_workflow.sh << 'EOF'
#!/bin/bash

# Complete SELinux Workflow for New Application Deployment

set -euo pipefail

APP_NAME=${1:-"myapp"}
APP_PATH=${2:-"/opt/myapp"}
LOG_PATH=${3:-"/var/log/myapp"}

echo "=== SELinux Configuration for $APP_NAME ==="
echo ""

# Step 1: Set SELinux to permissive for testing
echo "Step 1: Setting SELinux to permissive mode for initial testing"
ORIGINAL_MODE=$(getenforce)
sudo setenforce 0
echo "  Mode: $(getenforce)"
echo ""

# Step 2: Create and deploy application
echo "Step 2: Application deployment (simulation)"
sudo mkdir -p "$APP_PATH"
sudo mkdir -p "$LOG_PATH"
echo "  Created: $APP_PATH"
echo "  Created: $LOG_PATH"
echo ""

# Step 3: Run application and collect denials
echo "Step 3: Run application and monitor for denials"
echo "  ⚠️  Run your application now and exercise all features"
echo "  Press Enter when done..."
read

# Step 4: Analyze denials
echo ""
echo "Step 4: Analyzing SELinux denials"
DENIALS=$(sudo ausearch -m AVC -ts boot 2>/dev/null | grep "$APP_NAME" | wc -l || echo "0")
echo "  Found $DENIALS denial(s)"

if [ "$DENIALS" -gt 0 ]; then
    echo ""
    echo "  Recent denials:"
    sudo ausearch -m AVC -ts boot 2>/dev/null | grep "$APP_NAME" | tail -10 || true
fi

echo ""

# Step 5: Get recommendations
echo "Step 5: Getting sealert recommendations"
if command -v sealert &> /dev/null; then
    sudo sealert -a /var/log/audit/audit.log | grep -A 20 "$APP_NAME" || echo "  No specific recommendations"
else
    echo "  sealert not available (install setroubleshoot-server)"
fi

echo ""

# Step 6: Fix contexts
echo "Step 6: Setting file contexts"
read -p "  Set context for $APP_PATH? (y/n) " -n 1 -r
echo
if [[ $REPLY =~ ^[Yy]$ ]]; then
    read -p "  Enter SELinux type (e.g., bin_t, usr_t): " CONTEXT_TYPE
    sudo semanage fcontext -a -t "$CONTEXT_TYPE" "$APP_PATH(/.*)?"
    sudo restorecon -Rv "$APP_PATH"
fi

read -p "  Set context for $LOG_PATH? (y/n) " -n 1 -r
echo
if [[ $REPLY =~ ^[Yy]$ ]]; then
    sudo semanage fcontext -a -t var_log_t "$LOG_PATH(/.*)?"
    sudo restorecon -Rv "$LOG_PATH"
fi

echo ""

# Step 7: Configure booleans
echo "Step 7: Configure SELinux booleans"
echo "  Common booleans for applications:"
echo "    - httpd_can_network_connect"
echo "    - httpd_can_network_connect_db"
echo "    - allow_user_exec_content"
echo ""
read -p "  Enable any boolean? (name or 'n' to skip): " BOOLEAN
if [[ ! $BOOLEAN =~ ^[Nn]$ ]] && [ -n "$BOOLEAN" ]; then
    sudo setsebool -P "$BOOLEAN" on
    echo "  Enabled: $BOOLEAN"
fi

echo ""

# Step 8: Generate custom policy if needed
echo "Step 8: Generate custom policy module (if needed)"
read -p "  Generate custom policy from denials? (y/n) " -n 1 -r
echo
if [[ $REPLY =~ ^[Yy]$ ]]; then
    MODULE_NAME="${APP_NAME}_policy"
    sudo ausearch -m AVC -ts boot | grep "$APP_NAME" | audit2allow -M "$MODULE_NAME"
    
    echo "  Generated policy module: ${MODULE_NAME}.pp"
    echo ""
    echo "  Policy source (${MODULE_NAME}.te):"
    cat "${MODULE_NAME}.te"
    echo ""
    
    read -p "  Install this policy module? (y/n) " -n 1 -r
    echo
    if [[ $REPLY =~ ^[Yy]$ ]]; then
        sudo semodule -i "${MODULE_NAME}.pp"
        echo "  Installed: $MODULE_NAME"
    fi
fi

echo ""

# Step 9: Return to enforcing mode
echo "Step 9: Returning to enforcing mode"
sudo setenforce 1
echo "  Mode: $(getenforce)"

echo ""

# Step 10: Final testing
echo "Step 10: Final testing in enforcing mode"
echo "  ✓ Run application and verify all functions work"
echo "  ✓ Monitor for new denials: ausearch -m AVC -ts recent"
echo "  ✓ Check logs: tail -f /var/log/audit/audit.log"

echo ""
echo "=== Workflow Complete ==="
echo ""
echo "Summary of changes:"
sudo semanage fcontext -l -C
echo ""
sudo semanage boolean -l -C
echo ""
sudo semodule -l | grep -v "^100" | tail -10

echo ""
echo "Documentation saved to: ~/selinux_lab/${APP_NAME}_config.txt"

cat > ~/selinux_lab/${APP_NAME}_config.txt << DOCEOF
# SELinux Configuration for $APP_NAME
# Generated: $(date)

## File Contexts:
$(sudo semanage fcontext -l | grep "$APP_PATH\|$LOG_PATH")

## Custom Policies:
$(sudo semodule -l | grep "$APP_NAME" || echo "None")

## Booleans:
$(sudo getsebool -a | grep -i "$APP_NAME" || echo "None")

## Testing:
1. Verify enforcing mode: getenforce
2. Test application functionality
3. Monitor denials: ausearch -m AVC -ts recent
4. Check application logs

## Rollback:
# Remove custom contexts:
sudo semanage fcontext -d "$APP_PATH(/.*)?"
sudo restorecon -Rv "$APP_PATH"

# Remove custom module:
sudo semodule -r ${APP_NAME}_policy

# Disable boolean:
sudo setsebool -P boolean_name off
DOCEOF

EOF

chmod +x ~/selinux_lab/deployment_workflow.sh

echo ""
echo "Deployment workflow script created: ~/selinux_lab/deployment_workflow.sh"
echo "Usage: ~/selinux_lab/deployment_workflow.sh [app_name] [app_path] [log_path]"

# Create comprehensive SELinux troubleshooting script
cat > ~/selinux_lab/selinux_troubleshoot.sh << 'EOF'
#!/bin/bash

# SELinux Comprehensive Troubleshooting Script

echo "=== SELinux Troubleshooting Report ==="
echo "Generated: $(date)"
echo ""

# Check SELinux status
echo "1. SELinux Status:"
echo "  Mode: $(getenforce)"
sestatus | grep -E "SELinux status|Current mode|Mode from config"
echo ""

# Recent denials
echo "2. Recent AVC Denials (last 10):"
if sudo ausearch -m AVC -ts today &>/dev/null; then
    sudo ausearch -m AVC -ts today 2>/dev/null | tail -20 | grep "type=AVC" | tail -10
else
    echo "  No denials today or ausearch not available"
fi
echo ""

# Denial statistics
echo "3. Denial Statistics:"
echo "  Today: $(sudo ausearch -m AVC -ts today 2>/dev/null | grep -c "type=AVC" || echo "0")"
echo "  This boot: $(sudo ausearch -m AVC -ts boot 2>/dev/null | grep -c "type=AVC" || echo "0")"
echo ""

# Top denied processes
echo "4. Most Denied Processes:"
sudo ausearch -m AVC -ts today 2>/dev/null | grep -o 'comm="[^"]*"' | sort | uniq -c | sort -rn | head -5 || echo "  None"
echo ""

# Boolean status
echo "5. Non-Default Booleans:"
CUSTOM_BOOLS=$(sudo semanage boolean -l -C 2>/dev/null | wc -l)
if [ "$CUSTOM_BOOLS" -gt 0 ]; then
    sudo semanage boolean -l -C
else
    echo "  All booleans at default values"
fi
echo ""

# Custom file contexts
echo "6. Custom File Contexts:"
CUSTOM_CONTEXTS=$(sudo semanage fcontext -l -C 2>/dev/null | wc -l)
if [ "$CUSTOM_CONTEXTS" -gt 0 ]; then
    sudo semanage fcontext -l -C
else
    echo "  No custom file contexts"
fi
echo ""

# Custom modules
echo "7. Custom Policy Modules:"
sudo semodule -l | grep -v "^100 " | head -10 || echo "  None"
echo ""

# Port customizations
echo "8. Custom Port Assignments:"
CUSTOM_PORTS=$(sudo semanage port -l -C 2>/dev/null | wc -l)
if [ "$CUSTOM_PORTS" -gt 0 ]; then
    sudo semanage port -l -C
else
    echo "  No custom port assignments"
fi
echo ""

# Recommendations
echo "9. Recommendations:"
DENIALS_COUNT=$(sudo ausearch -m AVC -ts today 2>/dev/null | grep -c "type=AVC" || echo "0")
if [ "$DENIALS_COUNT" -gt 0 ]; then
    echo "  ⚠️  Found $DENIALS_COUNT denials today - investigation recommended"
    echo "  Run: sudo sealert -a /var/log/audit/audit.log"
    echo "  Or: sudo ausearch -m AVC -ts today | audit2allow"
else
    echo "  ✓ No denials detected today"
fi

if [ "$(getenforce)" != "Enforcing" ]; then
    echo "  ⚠️  SELinux not in enforcing mode - security reduced"
    echo "  Run: sudo setenforce 1"
fi

echo ""
echo "=== Report Complete ==="
EOF

chmod +x ~/selinux_lab/selinux_troubleshoot.sh

echo ""
echo "=== SELinux Lab Setup Complete ==="
echo ""
echo "Created scripts:"
echo "  ~/selinux_lab/deployment_workflow.sh - Complete app deployment workflow"
echo "  ~/selinux_lab/selinux_troubleshoot.sh - Comprehensive troubleshooting"
echo ""
echo "Created documentation:"
echo "  ~/selinux_lab/policy_types.md - Policy type reference"
echo "  ~/selinux_lab/context_examples.md - Context format guide"
echo "  ~/selinux_lab/common_booleans.md - Boolean reference"
echo "  ~/selinux_lab/avc_denial_guide.md - Understanding denials"
echo "  ~/selinux_lab/sealert_guide.md - Using sealert"
echo "  ~/selinux_lab/troubleshooting_workflow.md - Step-by-step troubleshooting"
echo ""
echo "Quick reference commands:"
echo "  getenforce                          # Check mode"
echo "  sudo ausearch -m AVC -ts recent     # Recent denials"
echo "  sudo sealert -a /var/log/audit/audit.log  # Analyze denials"
echo "  ls -Z /path                         # Check file contexts"
echo "  sudo restorecon -Rv /path           # Fix contexts"
echo "  sudo getsebool -a | grep service    # Find booleans"
echo "  sudo setsebool -P boolean on        # Enable boolean"
```

**Explanation**:
- **Task 3**: `setenforce` changes mode temporarily; useful for testing if SELinux is causing issues
- **Task 9**: `semanage fcontext` sets permanent context mappings; `restorecon` applies them
- **Task 14**: `-P` flag makes boolean changes persistent across reboots
- **Task 17**: Services can only bind to ports with matching SELinux types
- **Task 23**: `sealert` provides human-readable analysis with specific fix recommendations
- **Task 25**: `audit2allow` generates policy from denials but should be used carefully
- **Contexts**: The type field (third component) is the most important for access control decisions
- **Troubleshooting**: Always try context fixes and booleans before creating custom policies

**SELinux Essential Commands Summary**:
```bash
# Status and Mode
getenforce                              # Show current mode
setenforce 0|1                          # Temporary mode change
sestatus                                # Detailed status
vi /etc/selinux/config                  # Permanent mode change

# Contexts
ls -Z                                   # File contexts
ps -eZ                                  # Process contexts
chcon -t type_t file                    # Temporary context change
semanage fcontext -a -t type_t "path(/.*)?"  # Permanent context mapping
restorecon -Rv /path                    # Apply/restore contexts

# Booleans
getsebool -a                            # List all booleans
getsebool boolean_name                  # Query specific boolean
setsebool boolean_name on               # Temporary enable
setsebool -P boolean_name on            # Permanent enable

# Ports
semanage port -l                        # List port assignments
semanage port -a -t type_t -p tcp 1234  # Add port assignment

# Troubleshooting
ausearch -m AVC -ts recent              # Find recent denials
aureport -a                             # Denial summary
sealert -a /var/log/audit/audit.log    # Detailed analysis
audit2allow -a                          # Generate policy from denials

# Policy Modules
audit2allow -a -M mypolicy              # Create policy module
semodule -i mypolicy.pp                 # Install module
semodule -l                             # List modules
semodule -r mypolicy                    # Remove module
```

**Common SELinux Scenarios**:

1. **Web server with custom document root**:
```bash
sudo semanage fcontext -a -t httpd_sys_content_t "/web(/.*)?"
sudo restorecon -Rv /web
```

2. **SSH on non-standard port**:
```bash
sudo semanage port -a -t ssh_port_t -p tcp 2222
```

3. **Web server needs database access**:
```bash
sudo setsebool -P httpd_can_network_connect_db on
```

4. **Fix mislabeled files**:
```bash
sudo restorecon -Rv /var/www/html
```

5. **Debug denials**:
```bash
sudo ausearch -m AVC -ts recent
sudo sealert -a /var/log/audit/audit.log
```

</details>

### Extensions
1. **Advanced**: Set up a complete LAMP stack (Linux, Apache, MariaDB, PHP) with all services confined by SELinux. Store web files in `/web`, database in `/data/mysql`, and configure custom ports. Create comprehensive documentation of all SELinux configurations, booleans enabled, and custom contexts created. Implement monitoring for SELinux denials
2. **Challenge**: Build an SELinux policy compliance framework that: automatically audits SELinux configuration across multiple RHEL servers, verifies all services run in enforcing mode, detects and alerts on denial patterns, maintains inventory of custom policies and contexts, implements automated remediation for common misconfigurations, and generates compliance reports showing security posture with SELinux metrics

---

## Summary and Next Steps

You've completed comprehensive exercises covering Operation of Running Systems for the LFCS exam:

- **Task 1**: Kernel parameters and system tuning with sysctl
- **Task 2**: Process management and system performance monitoring
- **Task 3**: Job scheduling with cron and systemd timers
- **Task 4**: Package management and repository configuration
- **Task 5**: System recovery and boot troubleshooting
- **Task 6**: Virtual machine management with KVM/QEMU and virsh
- **Task 7**: Container management with Docker and Podman
- **Task 8**: Mandatory Access Control with AppArmor and SELinux overview
- **Task 9**: SELinux configuration and troubleshooting (RHEL/CentOS)

**Key Takeaways**:
- Kernel tuning requires careful testing and validation before production deployment
- Process management and scheduling are essential for maintaining system stability
- Package management differs between distributions (apt vs yum/dnf)
- Understanding boot process and recovery is critical for troubleshooting
- Container technologies are replacing traditional VMs for many workloads
- Mandatory Access Control (AppArmor/SELinux) provides defense-in-depth security
- SELinux is required knowledge for RHEL-based system administration

**Recommended Practice**:
1. Set up a test environment with both Ubuntu and RHEL/CentOS VMs
2. Practice boot recovery scenarios regularly (simulated failures)
3. Build and deploy containerized applications using both Docker and Podman
4. Create custom AppArmor profiles for your own applications
5. Work through SELinux troubleshooting scenarios using audit logs
6. Automate system maintenance tasks with systemd timers

**Additional Resources**:
- [LFCS Exam Domains](https://training.linuxfoundation.org/certification/linux-foundation-certified-sysadmin-lfcs/)
- [Linux Kernel Documentation](https://www.kernel.org/doc/html/latest/)
- [systemd Documentation](https://www.freedesktop.org/software/systemd/man/)
- [Docker Documentation](https://docs.docker.com/)
- [Podman Documentation](https://docs.podman.io/)
- [AppArmor Wiki](https://gitlab.com/apparmor/apparmor/-/wikis/home)
- [SELinux Project](https://github.com/SELinuxProject)
- [Red Hat SELinux Guide](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9/html/using_selinux/)

Continue practicing these operational skills in real Ubuntu and RHEL environments to build confidence for the LFCS exam!
