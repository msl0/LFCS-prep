# LFCS Practice: User and Group Management

This module covers essential user and group management skills for the LFCS exam, including local account management, environment configuration, resource limits, ACLs, and LDAP integration on Ubuntu 22.04 LTS.

---

## Task 1: Multi-Tenant User and Group Management

### Learning Objectives
- Master local user and group account creation, modification, and deletion
- Configure group memberships and understand primary vs. supplementary groups
- Implement password policies and account expiration controls

### Context
Your company is onboarding three development teams (Frontend, Backend, DevOps) to a shared Ubuntu server. Each team needs their own group with specific users, and some users belong to multiple teams. The security policy requires password expiration after 90 days, and certain administrative users need password-less sudo access. Additionally, a contractor account needs to expire automatically after 60 days.

This scenario is critical for LFCS certification as user and group management forms the foundation of Linux system administration. Understanding how to properly create users, manage group memberships, and enforce security policies through password aging and account expiration is essential for maintaining secure multi-user environments. The LFCS exam tests your ability to handle real-world scenarios where multiple users with different access requirements need to be managed efficiently.

### Task Instructions

**Prerequisites**: Start with a clean Ubuntu 22.04 system. You'll need sudo privileges.
```bash
# Verify your Ubuntu version
lsb_release -a

# Ensure you have sudo access
sudo whoami
```

**Your Tasks**:

**Part A: Create Groups and Users**

1. Create three groups: `frontend`, `backend`, and `devops`
2. Create the following users with their primary groups:
   - User `alice` (primary group: `frontend`, secondary groups: `devops`)
   - User `bob` (primary group: `backend`)
   - User `charlie` (primary group: `devops`, secondary groups: `backend`)
   - User `dana` (primary group: `frontend`)
3. Create a contractor user `contractor1` whose account should expire exactly 60 days from today
4. Set initial passwords for all users (use `Password123!` for testing purposes)

**Part B: Configure Password Policies**

5. Configure all regular users to change their passwords every 90 days
6. Set bob's password to expire immediately, forcing a password change on next login
7. Lock the contractor1 account (without deleting it) to temporarily disable access

**Part C: Administrative Access**

8. Create a group called `sysadmins` and add `alice` to it
9. Configure the `sysadmins` group to have sudo access without requiring a password

**Part D: Verification and Management**

10. Display detailed information about `charlie` including all group memberships
11. List all members of the `devops` group
12. Remove `charlie` from the `backend` secondary group

### Hints and Resources
1. Use `useradd` with `-m` flag to create home directories, or consider `adduser` for interactive creation
2. The `-G` option specifies supplementary groups, while `-g` sets the primary group
3. User password aging is controlled with the `chage` command; check `chage -l username` to view current settings
4. Account expiration dates can be set with `usermod -e` or `chage -E` using YYYY-MM-DD format
5. The `/etc/sudoers` file controls sudo access, but use `visudo` to edit it safely; group permissions use `%groupname`
6. The `id` command shows user and group information; `getent group groupname` lists group members
7. Reference: [Ubuntu User Management](https://help.ubuntu.com/community/AddUsersHowto) and [chage manual](https://manpages.ubuntu.com/manpages/jammy/en/man1/chage.1.html)

### Estimated Time and Difficulty
**40-50 minutes, Intermediate**

### Verification
To verify your solution:
- Run `getent passwd alice bob charlie dana contractor1` and confirm all users exist with correct UIDs
- Execute `id alice` and verify she belongs to `frontend` (primary), `devops`, and `sysadmins` (secondary)
- Run `id charlie` and confirm he's only in `devops` (primary) after removal from `backend`
- Check `chage -l alice` to verify password must be changed every 90 days
- Verify `sudo -l -U alice` shows NOPASSWD permissions
- Run `chage -l contractor1` to confirm the account expires in 60 days
- Check `passwd -S contractor1` to verify the account is locked (should show 'L')
- Attempt `sudo -l -U bob` as bob and confirm password change is required

<details>
<summary>Solution</summary>

```bash
# Task 1: Create groups
sudo groupadd frontend
sudo groupadd backend
sudo groupadd devops

# Task 2: Create users with primary and secondary groups
sudo useradd -m -g frontend -G devops alice
sudo useradd -m -g backend bob
sudo useradd -m -g devops -G backend charlie
sudo useradd -m -g frontend dana

# Task 3: Create contractor with expiration date (60 days from today)
# Calculate date 60 days from now
EXPIRE_DATE=$(date -d "+60 days" +%Y-%m-%d)
sudo useradd -m -e $EXPIRE_DATE contractor1

# Alternative: Set expiration after user creation
# sudo useradd -m contractor1
# sudo chage -E $(date -d "+60 days" +%Y-%m-%d) contractor1

# Task 4: Set passwords for all users
echo "alice:Password123!" | sudo chpasswd
echo "bob:Password123!" | sudo chpasswd
echo "charlie:Password123!" | sudo chpasswd
echo "dana:Password123!" | sudo chpasswd
echo "contractor1:Password123!" | sudo chpasswd

# Task 5: Configure 90-day password expiration for all regular users
sudo chage -M 90 alice
sudo chage -M 90 bob
sudo chage -M 90 charlie
sudo chage -M 90 dana

# Task 6: Force bob to change password on next login
sudo chage -d 0 bob
# Alternative using passwd:
# sudo passwd -e bob

# Task 7: Lock contractor1 account
sudo usermod -L contractor1
# Alternative:
# sudo passwd -l contractor1

# Task 8: Create sysadmins group and add alice
sudo groupadd sysadmins
sudo usermod -aG sysadmins alice

# Task 9: Configure passwordless sudo for sysadmins group
echo '%sysadmins ALL=(ALL) NOPASSWD: ALL' | sudo EDITOR='tee -a' visudo -f /etc/sudoers.d/sysadmins
# Alternative direct approach (less safe):
# sudo bash -c 'echo "%sysadmins ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers.d/sysadmins'

# Ensure proper permissions on sudoers file
sudo chmod 440 /etc/sudoers.d/sysadmins

# Task 10: Display detailed information about charlie
id charlie
groups charlie
getent passwd charlie

# Task 11: List all members of devops group
getent group devops
# Alternative:
# grep devops /etc/group

# Task 12: Remove charlie from backend secondary group
sudo gpasswd -d charlie backend
# Alternative:
# sudo deluser charlie backend
```

**Explanation**:
- **Tasks 1-2**: `groupadd` creates groups. `useradd -m` creates users with home directories. The `-g` flag sets the primary group, and `-G` adds supplementary groups. Without `-g`, the system creates a private group matching the username.
- **Task 3**: The `-e` flag with `useradd` sets account expiration. We use `date -d "+60 days" +%Y-%m-%d` to calculate the exact date. Alternatively, `chage -E` can set expiration after user creation.
- **Task 4**: `chpasswd` allows batch password setting from stdin in `username:password` format. In production, use stronger passwords and force changes on first login.
- **Task 5**: `chage -M 90` sets maximum password age to 90 days. The `-M` flag controls password expiration policy.
- **Task 6**: `chage -d 0` sets the last password change date to epoch (Jan 1, 1970), forcing a password change on next login. `passwd -e` accomplishes the same thing.
- **Task 7**: `usermod -L` locks the account by prefixing the password hash with `!`. The account exists but cannot authenticate. Use `-U` to unlock.
- **Task 8**: `usermod -aG` adds a user to supplementary groups without removing them from existing groups. The `-a` (append) flag is critical.
- **Task 9**: Sudoers configuration uses `/etc/sudoers.d/` directory for modular configuration. The `%` prefix indicates a group. `NOPASSWD` removes password requirement. Always use `visudo` to validate syntax.
- **Task 10**: `id` shows UID, GID, and all group memberships. `getent passwd` queries the user database, showing the user's entry from `/etc/passwd`.
- **Task 11**: `getent group` queries the group database, displaying members listed in `/etc/group`. Note that users whose primary group is the specified group may not appear in this list.
- **Task 12**: `gpasswd -d` removes a user from a group. This only affects supplementary groups, not primary groups.

**Additional User Management Commands**:
```bash
# View all users on the system
getent passwd

# View all groups
getent group

# Check password status
sudo passwd -S username

# View detailed password aging information
sudo chage -l username

# Change user's shell
sudo usermod -s /bin/bash username

# Change user's home directory
sudo usermod -d /new/home/path -m username

# Delete user and home directory
sudo userdel -r username

# Delete group
sudo groupdel groupname
```

</details>

### Extensions
1. **Advanced**: Create a script that automatically creates users from a CSV file containing username, primary group, and supplementary groups. Include error handling for duplicate users.
2. **Challenge**: Configure PAM (Pluggable Authentication Modules) to enforce password complexity requirements (minimum length, special characters, etc.) and test with the users you created.

---

## Task 2: Environment Profile Management

### Learning Objectives
- Configure system-wide and user-specific environment variables
- Understand the difference between login and non-login shell configuration files
- Implement custom shell prompts and aliases for productivity

### Context
You're setting up a development server where multiple teams will work. Management has requested that all users see a custom welcome message upon login, have access to a set of company-standard aliases, and have certain environment variables set for development tools. Additionally, the DevOps team needs custom PATH settings to access their specialized tools, while maintaining the standard environment for other users.

Environment configuration is a crucial LFCS competency because it affects how users interact with the system, which tools they can access, and how applications behave. Understanding the difference between `/etc/profile`, `/etc/bash.bashrc`, `~/.bashrc`, and `~/.profile` is essential for troubleshooting user environment issues and implementing organization-wide configurations. The exam tests your ability to configure both global and user-specific environments appropriately.

### Task Instructions

**Prerequisites**: Use the users created in Task 1, or create test users if needed.
```bash
# Create a test user if you haven't already
sudo useradd -m testuser
sudo passwd testuser

# You'll need to switch between users
# Use: sudo su - username
```

**Your Tasks**:

**Part A: System-Wide Configuration**

1. Create a system-wide welcome message that displays when any user logs in, showing: "Welcome to DevServer - $(date)"
2. Configure a system-wide alias for all users: `ll` should execute `ls -lah --color=auto`
3. Set a system-wide environment variable `COMPANY_ENV=production` that is available to all users in all shells

**Part B: User-Specific Configuration**

4. For user `alice`, configure her personal environment to:
   - Add `/opt/devtools/bin` to her PATH (at the beginning)
   - Create an alias `gst` that executes `git status`
   - Set her PS1 prompt to display as: `[alice@hostname] current_directory $`
   - Add a custom environment variable `DEV_MODE=true`
5. Ensure these settings work for both login and non-login shells

**Part C: Profile Script Management**

6. Create a custom script in `/etc/profile.d/` that sets up development environment variables for all users:
   - `JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64`
   - `MAVEN_HOME=/opt/maven`
   - Adds both to the PATH
7. Ensure the script only executes for interactive shells, not for scripts or non-interactive sessions

### Hints and Resources
1. Login shells read `/etc/profile` and `~/.profile` (or `~/.bash_profile`); non-login shells read `/etc/bash.bashrc` and `~/.bashrc`
2. The `/etc/profile.d/` directory allows modular configuration; scripts there are sourced by `/etc/profile`
3. The `PS1` variable controls the bash prompt; use `\u` for username, `\h` for hostname, `\w` for working directory
4. To make environment variables persistent, export them in the appropriate configuration file
5. Test by opening a new login shell with `su - username` versus a non-login shell with `su username`
6. The `$-` variable contains shell options; if it contains 'i', the shell is interactive
7. Reference: [Bash Startup Files](https://www.gnu.org/software/bash/manual/html_node/Bash-Startup-Files.html) and [Ubuntu Environment Variables](https://help.ubuntu.com/community/EnvironmentVariables)

### Estimated Time and Difficulty
**30-40 minutes, Intermediate**

### Verification
To verify your solution:
- Log in as any user with `sudo su - testuser` and confirm the welcome message appears
- Run `ll` as any user and verify it shows detailed, colored directory listing
- Check `echo $COMPANY_ENV` from any user account and verify it shows "production"
- Switch to alice with `sudo su - alice` and verify:
  - `echo $PATH` shows `/opt/devtools/bin` at the beginning
  - `alias gst` shows the git status alias
  - The prompt displays as `[alice@hostname] ~ $`
  - `echo $DEV_MODE` returns "true"
- Open a non-login shell as alice (`sudo su alice` without dash) and confirm aliases and environment still work
- Verify `echo $JAVA_HOME` and `echo $MAVEN_HOME` work for all users
- Run `echo $PATH | grep maven` to confirm Maven is in the PATH

<details>
<summary>Solution</summary>

```bash
# Task 1: System-wide welcome message on login
sudo bash -c 'cat > /etc/profile.d/welcome.sh << '\''EOF'\''
#!/bin/bash
# Display welcome message for login shells
if [ -n "$PS1" ]; then
    echo "Welcome to DevServer - $(date)"
fi
EOF'

sudo chmod +x /etc/profile.d/welcome.sh

# Task 2: System-wide alias for ll
sudo bash -c 'cat >> /etc/bash.bashrc << '\''EOF'\''

# Custom system-wide aliases
alias ll="ls -lah --color=auto"
EOF'

# Task 3: System-wide environment variable
# Add to /etc/environment for universal access
echo 'COMPANY_ENV=production' | sudo tee -a /etc/environment

# Alternative: Add to /etc/profile.d/ for shell-specific
sudo bash -c 'cat > /etc/profile.d/company_env.sh << '\''EOF'\''
#!/bin/bash
export COMPANY_ENV=production
EOF'

# Task 4: Configure alice's personal environment
# First, ensure the directory exists (it should from Task 1)
sudo mkdir -p /opt/devtools/bin

# Configure .bashrc for non-login shells (aliases and prompt)
sudo su - alice -c 'cat >> ~/.bashrc << '\''EOF'\''

# Personal aliases
alias gst="git status"

# Custom prompt
export PS1="[alice@\h] \w $ "

# Custom environment variables
export DEV_MODE=true
EOF'

# Configure .profile for login shells (PATH modification)
sudo su - alice -c 'cat >> ~/.profile << '\''EOF'\''

# Add custom tools to PATH
export PATH="/opt/devtools/bin:$PATH"
EOF'

# Alternative: Add PATH to .bashrc as well for non-login shells
sudo su - alice -c 'cat >> ~/.bashrc << '\''EOF'\''

# Ensure PATH includes devtools for non-login shells too
export PATH="/opt/devtools/bin:$PATH"
EOF'

# Task 6: Create development environment script
sudo bash -c 'cat > /etc/profile.d/dev_environment.sh << '\''EOF'\''
#!/bin/bash
# Development environment configuration
# Only execute for interactive shells

# Check if this is an interactive shell
if [[ $- == *i* ]]; then
    export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
    export MAVEN_HOME=/opt/maven
    export PATH="$JAVA_HOME/bin:$MAVEN_HOME/bin:$PATH"
fi
EOF'

sudo chmod +x /etc/profile.d/dev_environment.sh

# Create placeholder directories for the tools (so PATH doesn't break)
sudo mkdir -p /usr/lib/jvm/java-11-openjdk-amd64
sudo mkdir -p /opt/maven/bin
```

**Explanation**:
- **Task 1**: Scripts in `/etc/profile.d/` are automatically sourced by `/etc/profile` during login. The `[ -n "$PS1" ]` check ensures the message only appears in interactive shells. The `.sh` extension and executable permission are required.
- **Task 2**: `/etc/bash.bashrc` is read by all interactive bash shells (both login and non-login), making it ideal for system-wide aliases. Aliases defined here are available to all users.
- **Task 3**: `/etc/environment` is read by PAM and makes variables available to all processes, not just shells. This is the most universal method. Variables here should NOT use `export` keyword or variable expansion. For shell-specific variables, use `/etc/profile.d/`.
- **Task 4**: User-specific configuration goes in the home directory. `.bashrc` handles aliases, prompt, and non-login shell setup. `.profile` (or `.bash_profile`) handles login shell setup, particularly PATH modifications. To ensure consistency, we add PATH to both files or source `.bashrc` from `.profile`.
- **Task 6**: The `$-` variable contains current shell options. If it includes 'i', the shell is interactive. The pattern `[[ $- == *i* ]]` checks for this. This prevents environment setup from interfering with scripts that might expect a clean environment.

**Additional Environment Configuration Reference**:
```bash
# Shell startup file order for LOGIN shells:
# 1. /etc/profile
# 2. /etc/profile.d/*.sh (sourced by /etc/profile)
# 3. ~/.bash_profile (or ~/.bash_login or ~/.profile - first found)

# Shell startup file order for NON-LOGIN shells:
# 1. /etc/bash.bashrc
# 2. ~/.bashrc

# View current environment variables
env
printenv

# View specific variable
echo $VARIABLE_NAME

# Set temporary environment variable (current shell only)
export TEMP_VAR=value

# Reload shell configuration without logging out
source ~/.bashrc
# or
. ~/.profile

# View all aliases
alias

# Remove an alias
unalias alias_name

# Common PS1 prompt variables:
# \u - username
# \h - hostname
# \H - full hostname
# \w - full working directory
# \W - basename of working directory
# \$ - # for root, $ for regular users
# \d - date
# \t - time (24-hour)
```

</details>

### Extensions
1. **Advanced**: Create a script that automatically backs up user environment files (.bashrc, .profile) before making changes, storing them in `/var/backups/user-configs/` with timestamps.
2. **Challenge**: Configure different PS1 prompts for different user groups (e.g., red prompt for sudo users, green for regular users) using conditional logic in `/etc/profile.d/`.

---

## Task 3: User Resource Limits Management

### Learning Objectives
- Configure system resource limits using `/etc/security/limits.conf` and `ulimit`
- Understand soft vs. hard limits and their enforcement
- Prevent resource exhaustion attacks and manage system resources effectively

### Context
Your server is hosting multiple development teams, and you've noticed that some users are accidentally (or intentionally) consuming excessive resources - opening thousands of files, spawning too many processes, or running memory-intensive operations that impact other users. The security team has also requested that resource limits be enforced to prevent fork bomb attacks and other resource exhaustion scenarios.

Resource limits are a critical LFCS topic because they protect system stability and ensure fair resource allocation in multi-user environments. Understanding how to configure limits through `/etc/security/limits.conf`, how PAM enforces these limits, and the difference between hard and soft limits is essential for preventing system crashes and managing resource contention. The exam tests your ability to diagnose resource-related issues and implement appropriate controls.

### Task Instructions

**Prerequisites**: You'll need sudo access and test users.
```bash
# Verify PAM modules are installed
dpkg -l | grep libpam-modules

# Create a test user for resource limit testing
sudo useradd -m limituser
sudo passwd limituser
```

**Your Tasks**:

**Part A: Configure System-Wide Limits**

1. Set a system-wide hard limit of maximum 50 processes per user for all regular users (UID >= 1000)
2. Configure the `frontend` group to have a maximum of 1024 open files (both soft and hard limits)
3. Set a soft limit of 512 MB (524288 KB) maximum memory size for the `backend` group, with a hard limit of 1 GB (1048576 KB)
4. Configure a maximum CPU time limit of 60 minutes for user `limituser`

**Part B: Priority and Nice Limits**

5. Allow members of the `devops` group to set process priority up to nice value -10 (higher priority than normal)
6. Configure maximum file size of 100 MB (102400 KB) for user `contractor1`

**Part C: Testing and Verification**

7. Test the limits by logging in as `limituser` and attempting to exceed the configured resource limits
8. Use `ulimit` to verify current limits for different users

### Hints and Resources
1. The `/etc/security/limits.conf` file controls resource limits; changes require re-login to take effect
2. The format is: `<domain> <type> <item> <value>` where domain can be user, @group, or *
3. Type can be `soft`, `hard`, or `-` (both); items include `nproc`, `nofile`, `as`, `cpu`, `nice`, etc.
4. Soft limits can be increased by users up to the hard limit; hard limits require root to change
5. The `ulimit` command shows and sets shell resource limits for the current session
6. UIDs are typically >= 1000 for regular users on Ubuntu; use wildcards with caution
7. Reference: [limits.conf manual](https://manpages.ubuntu.com/manpages/jammy/en/man5/limits.conf.5.html) and [ulimit documentation](https://ss64.com/bash/ulimit.html)

### Estimated Time and Difficulty
**25-35 minutes, Intermediate**

### Verification
To verify your solution:
- Check that `/etc/security/limits.conf` contains your configurations with correct syntax
- Switch to a regular user and run `ulimit -u` to verify process limit is 50
- Log in as a user in the `frontend` group and run `ulimit -n` to see the file descriptor limit (should be 1024)
- As a `backend` group member, run `ulimit -v` to check memory limit (should show 524288 or 1048576)
- Log in as `limituser` and run `ulimit -t` to verify CPU time limit (should be 3600 seconds)
- As a `devops` member, run `ulimit -e` to check nice priority limit (should allow -10)
- Try to create a large file as `contractor1` and verify it's limited to 100 MB
- Test by running commands like `ulimit -n 2000` as `frontend` group member (should fail to exceed 1024)

<details>
<summary>Solution</summary>

```bash
# Edit /etc/security/limits.conf
# It's better to create a separate file in /etc/security/limits.d/

# Task 1: Maximum 50 processes for regular users (UID >= 1000)
sudo bash -c 'cat >> /etc/security/limits.conf << EOF

# Custom resource limits
# Maximum processes for regular users
*               hard    nproc           50
EOF'

# Alternative: Use UID range (if supported)
# Note: Standard limits.conf uses user/group names, not UID ranges
# For UID-based limits, use @groupname for groups

# Task 2: Open files limit for frontend group
sudo bash -c 'cat >> /etc/security/limits.conf << EOF

# File descriptor limits for frontend group
@frontend       soft    nofile          1024
@frontend       hard    nofile          1024
EOF'

# Task 3: Memory limits for backend group
sudo bash -c 'cat >> /etc/security/limits.conf << EOF

# Memory limits for backend group (in KB)
@backend        soft    as              524288
@backend        hard    as              1048576
EOF'

# Task 4: CPU time limit for limituser (60 minutes = 3600 seconds)
sudo bash -c 'cat >> /etc/security/limits.conf << EOF

# CPU time limit for limituser
limituser       hard    cpu             60
EOF'

# Task 5: Nice priority for devops group
sudo bash -c 'cat >> /etc/security/limits.conf << EOF

# Allow devops group to set higher priority (negative nice values)
@devops         soft    nice            -10
@devops         hard    nice            -10
EOF'

# Task 6: Maximum file size for contractor1 (100 MB = 102400 KB)
sudo bash -c 'cat >> /etc/security/limits.conf << EOF

# Maximum file size for contractor1
contractor1     hard    fsize           102400
EOF'

# Alternative: Create a modular configuration file
sudo bash -c 'cat > /etc/security/limits.d/custom-limits.conf << EOF
# Custom Resource Limits Configuration

# Maximum processes for all users
*               hard    nproc           50

# Frontend group - open files
@frontend       soft    nofile          1024
@frontend       hard    nofile          1024

# Backend group - memory limits (KB)
@backend        soft    as              524288
@backend        hard    as              1048576

# Limituser - CPU time (minutes)
limituser       hard    cpu             60

# DevOps group - process priority
@devops         soft    nice            -10
@devops         hard    nice            -10

# Contractor1 - file size (KB)
contractor1     hard    fsize           102400
EOF'

# Verify the syntax
sudo cat /etc/security/limits.conf | grep -v "^#" | grep -v "^$"

# Test limits (must login as user for limits to take effect)
# You cannot test without actually logging in, but here are verification commands:

# For testing, you would need to do:
# sudo su - limituser
# Then run:
ulimit -a  # View all limits

# Test process limit
ulimit -u  # Should show 50

# Test as frontend group member
# sudo usermod -aG frontend testuser
# sudo su - testuser
# ulimit -n  # Should show 1024

# Try to exceed the limit (should fail)
# ulimit -n 2000  # Should give error: "cannot modify limit: Operation not permitted"
```

**Explanation**:
- **Format**: The limits.conf format is `<domain> <type> <item> <value>`. Domain can be a username, `@groupname`, or `*` (wildcard). Type is `soft`, `hard`, or `-` (both).
- **Task 1**: The `nproc` item limits the number of processes. The `*` wildcard applies to all users. Hard limits cannot be exceeded or raised without root privileges.
- **Task 2**: The `nofile` item controls open file descriptors. Setting both soft and hard to the same value prevents users from raising the limit. The `@` prefix indicates a group.
- **Task 3**: The `as` item limits address space (virtual memory) in KB. Soft limits can be increased by the user up to the hard limit. This prevents memory exhaustion.
- **Task 4**: The `cpu` item limits CPU time in minutes per process. After exceeding this time, the process receives SIGKILL.
- **Task 5**: The `nice` item controls process priority. Negative values (higher priority) are normally restricted. The range is -20 (highest) to 19 (lowest).
- **Task 6**: The `fsize` item limits the maximum file size a user can create, in KB. Useful for preventing disk space exhaustion.

**Important Notes**:
- Limits only apply to NEW login sessions; existing sessions retain old limits
- PAM must be configured to enforce limits (it is by default on Ubuntu)
- Check `/etc/pam.d/common-session` for `pam_limits.so` module
- Use `/etc/security/limits.d/` directory for modular configuration (recommended)
- Soft limits can be raised by users up to hard limit using `ulimit`
- Hard limits can only be lowered by users, not raised (except by root)

**Resource Limit Items Reference**:
```bash
# Common limit items:
# core      - Core file size (KB)
# data      - Max data segment size (KB)
# fsize     - Maximum file size (KB)
# memlock   - Max locked-in-memory address space (KB)
# nofile    - Max number of open files
# rss       - Max resident set size (KB)
# stack     - Max stack size (KB)
# cpu       - Max CPU time (minutes)
# nproc     - Max number of processes
# as        - Address space limit (KB) - virtual memory
# maxlogins - Max number of logins for this user
# maxsyslogins - Max logins on system
# priority  - Priority to run user processes with
# nice      - Max nice priority allowed to raise to
# rtprio    - Max real-time priority

# View current limits
ulimit -a

# View specific limits
ulimit -u  # Processes
ulimit -n  # Open files
ulimit -t  # CPU time (seconds)
ulimit -v  # Virtual memory (KB)
ulimit -f  # File size (blocks)

# Set soft limit for current shell (temporary)
ulimit -u 100  # Set max processes to 100

# Try to set hard limit (requires root)
ulimit -Hu 200  # Set hard limit for processes
```

</details>

### Extensions
1. **Advanced**: Create a monitoring script that checks current resource usage against configured limits and alerts when users approach their limits (e.g., using 90% of allowed processes).
2. **Challenge**: Implement core dump size limits for debugging purposes, configure the core dump location to `/var/coredumps/`, and test by triggering a segmentation fault in a C program.

---

## Task 4: Access Control Lists (ACLs)

### Learning Objectives
- Implement file and directory ACLs to provide granular permissions beyond traditional Unix permissions
- Configure default ACLs for automatic inheritance on new files
- Troubleshoot and manage complex permission scenarios using ACL tools

### Context
Your development team has a shared project directory where traditional Unix permissions (owner, group, other) are insufficient. You need to grant specific users read/write access to certain subdirectories while maintaining different permissions for others, without adding everyone to the same group. Additionally, you need to ensure that new files created in these directories automatically inherit the correct permissions.

ACLs are an essential LFCS competency because they extend the traditional Unix permission model to handle complex, real-world access control scenarios. Understanding how to use `setfacl` and `getfacl`, configure default ACLs for directories, and manage the interaction between ACLs and traditional permissions is critical for implementing security policies in enterprise environments. The exam tests your ability to implement fine-grained access control without compromising security or usability.

### Task Instructions

**Prerequisites**: Ensure your filesystem supports ACLs (ext4 and XFS support it by default).
```bash
# Check if ACL packages are installed
dpkg -l | grep acl

# Install if needed
sudo apt update && sudo apt install -y acl

# Verify filesystem ACL support
mount | grep "$(df . | tail -1 | awk '{print $1}')"
```

**Your Tasks**:

**Part A: Project Directory Setup**

1. Create a project directory structure:
   - `/projects/webapp/` (owned by root:root)
   - `/projects/webapp/frontend/` 
   - `/projects/webapp/backend/`
   - `/projects/webapp/shared/`

**Part B: Configure ACLs for Specific Access**

2. Grant user `alice` read, write, and execute permissions on `/projects/webapp/frontend/` directory (in addition to standard permissions)
3. Grant user `bob` read and execute (but NOT write) permissions on `/projects/webapp/frontend/`
4. Give the `devops` group full access (rwx) to `/projects/webapp/backend/`
5. Grant user `charlie` read-only access to `/projects/webapp/shared/`

**Part C: Default ACLs for Inheritance**

6. Configure default ACLs on `/projects/webapp/frontend/` so that:
   - New files and directories automatically grant `alice` read and write permissions
   - New files and directories automatically grant `bob` read-only permissions
7. Configure `/projects/webapp/shared/` with default ACLs that give the `frontend` group read and write access to all new files

**Part D: Testing and Management**

8. Create test files in each directory and verify that ACLs are correctly applied
9. Remove `bob`'s ACL entry from `/projects/webapp/frontend/` while keeping other ACLs intact
10. Backup ACLs for `/projects/webapp/` to a file and demonstrate how to restore them

### Hints and Resources
1. Use `setfacl -m` to modify ACLs and `setfacl -x` to remove them
2. Default ACLs for directories are set with `setfacl -d -m`, affecting newly created files
3. The `getfacl` command displays current ACLs; `getfacl -R` recurses through directories
4. ACL masks limit the maximum permissions that can be granted; use `setfacl -m m::rwx` to modify
5. When listing files with `ls -l`, a `+` after permissions indicates ACLs are present
6. Use `getfacl file1 | setfacl --set-file=- file2` to copy ACLs between files
7. Reference: [setfacl manual](https://manpages.ubuntu.com/manpages/jammy/en/man1/setfacl.1.html) and [Ubuntu ACL Guide](https://help.ubuntu.com/community/FilePermissionsACLs)

### Estimated Time and Difficulty
**35-45 minutes, Intermediate**

### Verification
To verify your solution:
- Run `ls -l /projects/webapp/` and look for `+` symbols indicating ACLs
- Execute `getfacl /projects/webapp/frontend/` and confirm `alice` has `rwx` and `bob` has `r-x`
- Check `getfacl /projects/webapp/backend/` shows the `devops` group with `rwx`
- Verify `getfacl /projects/webapp/shared/` shows `charlie` with read-only (`r-x` or `r--`)
- Create a test file as root in `/projects/webapp/frontend/` and run `getfacl` on it to confirm default ACLs applied
- Confirm `bob` can read but cannot write to files in `/projects/webapp/frontend/`
- Check that the ACL backup file contains valid ACL data
- After removing `bob`'s ACL, verify `getfacl /projects/webapp/frontend/` no longer shows `bob`

<details>
<summary>Solution</summary>

```bash
# Task 1: Create project directory structure
sudo mkdir -p /projects/webapp/{frontend,backend,shared}

# Set base ownership
sudo chown -R root:root /projects/webapp

# Set permissive base permissions (ACLs will restrict)
sudo chmod -R 755 /projects/webapp

# Task 2: Grant alice rwx on frontend directory
sudo setfacl -m u:alice:rwx /projects/webapp/frontend/

# Task 3: Grant bob r-x on frontend directory
sudo setfacl -m u:bob:r-x /projects/webapp/frontend/

# Task 4: Grant devops group rwx on backend directory
sudo setfacl -m g:devops:rwx /projects/webapp/backend/

# Task 5: Grant charlie read-only on shared directory
sudo setfacl -m u:charlie:r-x /projects/webapp/shared/

# Task 6: Configure default ACLs on frontend directory
# Default ACLs apply to newly created files and subdirectories
sudo setfacl -d -m u:alice:rw /projects/webapp/frontend/
sudo setfacl -d -m u:bob:r /projects/webapp/frontend/

# Also set defaults for directories (they need execute permission)
sudo setfacl -d -m u:alice:rwx /projects/webapp/frontend/
# For files, the execute bit will be masked off automatically

# Task 7: Configure default ACLs on shared directory for frontend group
sudo setfacl -d -m g:frontend:rw /projects/webapp/shared/

# Task 8: Create test files to verify ACL inheritance
sudo touch /projects/webapp/frontend/test_file.txt
sudo mkdir /projects/webapp/frontend/test_dir

# Check ACLs on test file
getfacl /projects/webapp/frontend/test_file.txt

# Create test file in shared directory
sudo touch /projects/webapp/shared/test_shared.txt
getfacl /projects/webapp/shared/test_shared.txt

# Task 9: Remove bob's ACL from frontend directory
sudo setfacl -x u:bob /projects/webapp/frontend/

# Verify removal
getfacl /projects/webapp/frontend/

# Task 10: Backup and restore ACLs
# Backup ACLs for entire webapp directory tree
getfacl -R /projects/webapp > /tmp/webapp_acls_backup.txt

# Alternative: Backup with preserved structure
sudo getfacl -R /projects/webapp/ > /root/webapp_acls_backup.txt

# To restore ACLs from backup:
# setfacl --restore=/tmp/webapp_acls_backup.txt

# Verify backup file content
cat /tmp/webapp_acls_backup.txt

# Additional: Set recursive ACLs (apply to existing files)
# If you need to apply ACLs recursively to existing content:
sudo setfacl -R -m u:alice:rwx /projects/webapp/frontend/
```

**Explanation**:
- **Task 2-5**: The `setfacl -m` command modifies ACLs. Format is `u:username:permissions` for users or `g:groupname:permissions` for groups. Permissions are `r` (read), `w` (write), `x` (execute).
- **Task 6**: Default ACLs (`-d` flag) define permissions for newly created files and directories. They don't affect existing files. For files, execute permission is typically masked off unless the file is created as executable.
- **Task 7**: Default ACLs work for groups too, using `g:groupname:permissions`. New files in the directory will automatically have these ACL entries.
- **Task 8**: The `getfacl` command displays ACL entries. When ACLs exist, `ls -l` shows a `+` after the permission string (e.g., `drwxr-xr-x+`).
- **Task 9**: The `setfacl -x` removes specific ACL entries without affecting others. Use `setfacl -b` to remove all ACLs.
- **Task 10**: `getfacl -R` recursively captures ACLs for an entire directory tree. The output can be piped to a file and later restored with `setfacl --restore=filename`.

**Important ACL Concepts**:
- **ACL Mask**: The mask limits maximum effective permissions. It's automatically calculated but can be set with `setfacl -m m::rwx`.
- **Effective Permissions**: Displayed in `getfacl` output with `#effective:` comments when mask restricts permissions.
- **Inheritance**: Only default ACLs are inherited; regular ACLs must be applied recursively to existing files.
- **Interaction with chmod**: Running `chmod` can modify the ACL mask, potentially restricting ACL permissions.

**Additional ACL Commands**:
```bash
# View ACLs on a file or directory
getfacl /path/to/file

# View ACLs recursively
getfacl -R /path/to/directory

# Set ACL for a user
setfacl -m u:username:rwx /path/to/file

# Set ACL for a group
setfacl -m g:groupname:rwx /path/to/file

# Set ACL recursively
setfacl -R -m u:username:rwx /path/to/directory

# Set default ACL (for directories)
setfacl -d -m u:username:rwx /path/to/directory

# Remove specific ACL entry
setfacl -x u:username /path/to/file

# Remove all ACLs
setfacl -b /path/to/file

# Set mask (maximum effective permissions)
setfacl -m m::rwx /path/to/file

# Copy ACLs from one file to another
getfacl file1 | setfacl --set-file=- file2

# Backup ACLs
getfacl -R /directory > backup.acl

# Restore ACLs
setfacl --restore=backup.acl

# Set ACL and make it default in one command
setfacl -m d:u:username:rwx /directory

# View only default ACLs
getfacl --access /path  # Only access ACLs
getfacl --default /path # Only default ACLs
```

</details>

### Extensions
1. **Advanced**: Create a script that audits all ACL-protected files in `/projects/` and generates a report showing which users have access to which directories, formatted as a CSV.
2. **Challenge**: Implement a "drop box" directory where users can create and write files but cannot read or modify files created by other users (using ACLs and special permissions like sticky bit).

---

## Task 5: LDAP User Authentication Integration

### Learning Objectives
- Configure Ubuntu to authenticate users against an LDAP directory server
- Implement SSSD (System Security Services Daemon) for centralized authentication
- Troubleshoot LDAP connectivity and authentication issues

### Context
Your organization is expanding, and managing local user accounts on dozens of servers has become impractical. The IT department has set up a central LDAP directory server that stores all user credentials and information. Your task is to configure this Ubuntu server to authenticate users against the LDAP server instead of using local `/etc/passwd` accounts, enabling centralized user management and single sign-on capabilities across the infrastructure.

LDAP integration is a critical LFCS competency because enterprise environments rarely manage users locally on each server. Understanding how to configure SSSD, PAM, and NSS to integrate with LDAP directories (like OpenLDAP or Active Directory) is essential for working in professional Linux environments. The exam tests your ability to configure authentication services, troubleshoot connectivity, and ensure users can authenticate seamlessly against centralized directory services.

### Task Instructions

**Prerequisites**: You'll need a test LDAP server. For this exercise, we'll set up a simple OpenLDAP server on the same machine.
```bash
# Update package list
sudo apt update

# Install LDAP server packages
sudo apt install -y slapd ldap-utils

# During installation, set LDAP admin password to: AdminPass123

# Reconfigure LDAP with proper settings
sudo dpkg-reconfigure slapd
# Choose:
# - Omit OpenLDAP server configuration? No
# - DNS domain name: example.com
# - Organization name: Example Org
# - Administrator password: AdminPass123
# - Database backend: MDB
# - Remove database when purged? No
# - Move old database? Yes
```

**Your Tasks**:

**Part A: LDAP Server Setup (Prerequisite)**

1. Add a test organizational unit (OU) and user to LDAP:
   - Create OU: `users` under `dc=example,dc=com`
   - Create user: `ldapuser1` with UID 5001 and password `LdapPass123`

**Part B: Client-Side LDAP Configuration**

2. Install required packages for LDAP authentication: `sssd`, `libpam-sss`, `libnss-sss`, `ldap-utils`
3. Configure SSSD to connect to the LDAP server at `ldap://localhost`
4. Configure the LDAP search base as `dc=example,dc=com`
5. Set up SSSD to cache credentials for offline authentication

**Part C: System Integration**

6. Configure PAM to use SSSD for authentication
7. Configure NSS (Name Service Switch) to query SSSD for user information
8. Enable and start the SSSD service
9. Configure automatic home directory creation for LDAP users on first login

**Part D: Testing and Verification**

10. Verify LDAP connectivity using `ldapsearch`
11. Test that LDAP user `ldapuser1` can be queried with `getent passwd ldapuser1`
12. Attempt to switch to `ldapuser1` and verify authentication works
13. Check SSSD logs for any errors or warnings

### Hints and Resources
1. SSSD configuration is done in `/etc/sssd/sssd.conf` with strict permissions (600)
2. NSS configuration is in `/etc/nsswitch.conf`; add `sss` to passwd, group, and shadow lines
3. PAM configuration is typically automatic when installing `libpam-sss`, but check `/etc/pam.d/common-*` files
4. The `pam_mkhomedir.so` module creates home directories; add it to `/etc/pam.d/common-session`
5. Use `sssctl user-checks username` to debug SSSD authentication issues
6. LDIF (LDAP Data Interchange Format) files are used to add entries; use `ldapadd` command
7. Reference: [SSSD Documentation](https://sssd.io/docs/quick-start.html), [Ubuntu LDAP Client](https://ubuntu.com/server/docs/service-ldap), and [OpenLDAP Admin Guide](https://www.openldap.org/doc/admin24/)

### Estimated Time and Difficulty
**50-60 minutes, Advanced**

### Verification
To verify your solution:
- Run `systemctl status sssd` and confirm SSSD is active and running
- Execute `getent passwd ldapuser1` and verify user information is returned from LDAP
- Run `id ldapuser1` and confirm UID is 5001
- Test `ldapsearch -x -b "dc=example,dc=com" -H ldap://localhost` and verify LDAP server responds
- Attempt `su - ldapuser1` and confirm you can authenticate with `LdapPass123`
- Check that home directory `/home/ldapuser1` was created automatically
- Review `/var/log/sssd/*.log` for successful authentication entries
- Run `sssctl user-checks ldapuser1` to validate SSSD can authenticate the user

<details>
<summary>Solution</summary>

```bash
# Part A: LDAP Server Setup with Test User

# Create LDIF file for organizational unit
cat > /tmp/base.ldif << 'EOF'
dn: ou=users,dc=example,dc=com
objectClass: organizationalUnit
ou: users
EOF

# Add the organizational unit
ldapadd -x -D "cn=admin,dc=example,dc=com" -w AdminPass123 -f /tmp/base.ldif

# Create LDIF file for test user
cat > /tmp/ldapuser1.ldif << 'EOF'
dn: uid=ldapuser1,ou=users,dc=example,dc=com
objectClass: inetOrgPerson
objectClass: posixAccount
objectClass: shadowAccount
uid: ldapuser1
sn: User
givenName: LDAP
cn: LDAP User
displayName: LDAP User One
uidNumber: 5001
gidNumber: 5001
userPassword: LdapPass123
gecos: LDAP User
loginShell: /bin/bash
homeDirectory: /home/ldapuser1
EOF

# Add the user to LDAP
ldapadd -x -D "cn=admin,dc=example,dc=com" -w AdminPass123 -f /tmp/ldapuser1.ldif

# Verify user was added
ldapsearch -x -b "dc=example,dc=com" "(uid=ldapuser1)"

# Part B: Install LDAP client packages
sudo apt install -y sssd libpam-sss libnss-sss ldap-utils sssd-tools

# Part C: Configure SSSD
sudo bash -c 'cat > /etc/sssd/sssd.conf << EOF
[sssd]
config_file_version = 2
services = nss, pam
domains = example.com

[domain/example.com]
id_provider = ldap
auth_provider = ldap
ldap_uri = ldap://localhost
ldap_search_base = dc=example,dc=com
ldap_id_use_start_tls = false
cache_credentials = true
ldap_tls_reqcert = never

# Enable offline authentication
ldap_offline_timeout = 3

# Automatically create home directories
ldap_default_bind_dn = cn=admin,dc=example,dc=com
ldap_default_authtok = AdminPass123

[nss]
filter_groups = root
filter_users = root

[pam]
offline_credentials_expiration = 2
EOF'

# Set strict permissions on sssd.conf (required)
sudo chmod 600 /etc/sssd/sssd.conf
sudo chown root:root /etc/sssd/sssd.conf

# Part D: Configure NSS to use SSSD
# Backup original nsswitch.conf
sudo cp /etc/nsswitch.conf /etc/nsswitch.conf.backup

# Update NSS configuration to include sss
sudo sed -i 's/^passwd:.*/passwd:         files systemd sss/' /etc/nsswitch.conf
sudo sed -i 's/^group:.*/group:          files systemd sss/' /etc/nsswitch.conf
sudo sed -i 's/^shadow:.*/shadow:         files sss/' /etc/nsswitch.conf

# Alternative manual approach:
# Edit /etc/nsswitch.conf and change these lines:
# passwd:         files systemd sss
# group:          files systemd sss
# shadow:         files sss

# Part E: Configure PAM for home directory creation
# Add pam_mkhomedir to common-session
sudo bash -c 'echo "session required pam_mkhomedir.so skel=/etc/skel/ umask=0077" >> /etc/pam.d/common-session'

# Verify PAM configuration includes sssd (should be automatic from package)
grep pam_sss /etc/pam.d/common-auth
grep pam_sss /etc/pam.d/common-account
grep pam_sss /etc/pam.d/common-password
grep pam_sss /etc/pam.d/common-session

# Part F: Enable and start SSSD
sudo systemctl enable sssd
sudo systemctl restart sssd

# Check SSSD status
sudo systemctl status sssd

# Part G: Testing and Verification

# Test LDAP connectivity
ldapsearch -x -b "dc=example,dc=com" -H ldap://localhost

# Query LDAP user through NSS
getent passwd ldapuser1

# Should return something like:
# ldapuser1:*:5001:5001:LDAP User:/home/ldapuser1:/bin/bash

# Check user ID
id ldapuser1

# Test authentication (switch to user)
su - ldapuser1
# Enter password: LdapPass123

# After successful login, verify home directory was created
ls -la /home/ldapuser1

# Check SSSD logs
sudo tail -f /var/log/sssd/sssd_example.com.log

# Debug SSSD user checks
sudo sssctl user-checks ldapuser1

# Clear SSSD cache if needed (for troubleshooting)
# sudo sss_cache -E

# Restart SSSD after configuration changes
# sudo systemctl restart sssd
```

**Explanation**:
- **Part A**: LDIF files define LDAP entries in a standardized format. The `ldapadd` command adds entries to the directory. We create an organizational unit (OU) for users and then add a test user with POSIX attributes (UID, GID, home directory, shell).
- **Part B**: SSSD is the modern approach for LDAP integration on Ubuntu. `libpam-sss` and `libnss-sss` provide PAM and NSS modules respectively. `sssd-tools` includes debugging utilities.
- **Part C**: `/etc/sssd/sssd.conf` must have 600 permissions for security. The configuration specifies the LDAP URI, search base, and authentication method. `cache_credentials = true` enables offline authentication. We disable TLS for this local test setup (never do this in production).
- **Part D**: NSS (Name Service Switch) determines where the system looks for user information. Adding `sss` to the passwd, group, and shadow lines tells the system to query SSSD after checking local files.
- **Part E**: PAM (Pluggable Authentication Modules) controls authentication. `pam_mkhomedir.so` automatically creates home directories on first login. The `libpam-sss` package automatically configures PAM to use SSSD for authentication.
- **Part F**: SSSD must be enabled and started. The service reads the configuration and establishes connection to the LDAP server.
- **Part G**: Verification ensures all components work together. `getent` queries NSS, `ldapsearch` tests LDAP connectivity, and `sssctl` debugs SSSD-specific issues.

**Important LDAP/SSSD Concepts**:
- **DN (Distinguished Name)**: Unique identifier for LDAP entries (e.g., `uid=ldapuser1,ou=users,dc=example,dc=com`)
- **Base DN**: Starting point for LDAP searches (e.g., `dc=example,dc=com`)
- **Bind DN**: Account used to connect to LDAP (e.g., `cn=admin,dc=example,dc=com`)
- **SSSD Domains**: Logical groupings of identity providers; can have multiple domains
- **Offline Authentication**: SSSD caches credentials, allowing authentication when LDAP server is unavailable

**Troubleshooting Commands**:
```bash
# Check SSSD service status
sudo systemctl status sssd

# View SSSD logs (verbose)
sudo tail -100 /var/log/sssd/sssd_example.com.log

# Test LDAP search
ldapsearch -x -H ldap://localhost -b "dc=example,dc=com"

# Query user through NSS
getent passwd ldapuser1

# Clear SSSD cache
sudo sss_cache -E

# Restart SSSD with debug logging
sudo sssd -i -d 9

# Check PAM configuration
sudo pam-auth-update --package

# Validate SSSD config syntax
sudo sssctl config-check

# List SSSD domains
sudo sssctl domain-list

# Show SSSD user information
sudo sssctl user-show ldapuser1
```

**Security Considerations**:
- Never use plain LDAP (ldap://) in production; always use LDAPS (ldap**s**://) or StartTLS
- Store bind credentials securely; avoid storing passwords in `sssd.conf`
- Use Kerberos authentication when possible
- Implement certificate validation for TLS connections
- Set appropriate file permissions (600) on `sssd.conf`

</details>

### Extensions
1. **Advanced**: Configure SSSD to use TLS encryption for LDAP communication, generate self-signed certificates, and verify certificate validation works correctly.
2. **Challenge**: Set up LDAP group-based sudo access where members of an LDAP group called `ldap-admins` automatically receive sudo privileges without modifying local `/etc/sudoers` files. Test with a new LDAP user.

---

## Summary and Next Steps

You've completed comprehensive exercises covering all aspects of user and group management for the LFCS exam:

- **Task 1**: Local user and group management, password policies, and account controls
- **Task 2**: Environment profile configuration at system and user levels
- **Task 3**: Resource limit implementation to prevent system exhaustion
- **Task 4**: Access Control Lists for granular permission management
- **Task 5**: LDAP integration for centralized authentication

**Key Takeaways**:
- User management is fundamental to Linux system administration
- Understanding the difference between files like `/etc/passwd`, `/etc/shadow`, `/etc/group`, and LDAP directories is crucial
- Resource limits prevent abuse and ensure fair resource allocation
- ACLs extend traditional Unix permissions for complex scenarios
- Centralized authentication with LDAP/SSSD is standard in enterprise environments

**Recommended Practice**:
1. Build a multi-server environment and configure LDAP replication
2. Implement Kerberos authentication with SSSD
3. Create automation scripts for bulk user management
4. Practice recovering from permission and authentication failures
5. Explore integration with Active Directory for hybrid environments

**Additional Resources**:
- [LFCS Exam Domains](https://training.linuxfoundation.org/certification/linux-foundation-certified-sysadmin-lfcs/)
- [Ubuntu Server Guide - User Management](https://ubuntu.com/server/docs/security-users)
- [Red Hat System Administration Guide](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9/html/configuring_authentication_and_authorization_in_rhel/)
- [SSSD Documentation](https://sssd.io/docs/)

Continue practicing these skills in real Ubuntu environments to build confidence for the LFCS exam!
