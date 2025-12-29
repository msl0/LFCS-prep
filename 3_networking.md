# LFCS Practice: Networking

These exercises cover essential networking skills for the LFCS exam, including IPv4/IPv6 configuration, time synchronization, SSH configuration, firewall rules, routing, network bonding, and advanced topics like reverse proxies. All tasks are designed for Ubuntu 22.04 LTS.

---

## Task 1: Configure Static and Dynamic Network Interfaces with IPv4 and IPv6

### Learning Objectives
- Configure static IPv4 and IPv6 addresses using Netplan
- Understand dual-stack networking and hostname resolution
- Manage network interfaces and verify connectivity

### Context

Your organization is deploying a new web server that needs to be accessible via both IPv4 and IPv6. The server will have two network interfaces: one with a static IP configuration for the production network (enp0s8) and one using DHCP for the management network (enp0s3). Additionally, you need to configure custom DNS servers and ensure the hostname is properly set and resolvable.

This task simulates a common real-world scenario where system administrators must configure network interfaces for servers in mixed environments. Understanding both static and dynamic IP configuration, along with IPv6, is critical for modern infrastructure management and is a core competency tested in the LFCS exam.

### Task Instructions

**Prerequisites**: Ensure you have a fresh Ubuntu 22.04 LTS VM with at least one network interface.
```bash
# Check available network interfaces
ip link show

# Check current network configuration
ip addr show
```

**Your Tasks**:
1. Set the system hostname to "webserver01.example.com" and ensure it persists after reboot
2. Configure the first interface (typically enp0s3 or eth0) to use DHCP for IPv4
3. Configure a second interface (or the same if only one available) with a static IPv4 address: 192.168.100.50/24, gateway 192.168.100.1
4. Add a static IPv6 address to the interface: 2001:db8:1::50/64
5. Configure custom DNS servers: 8.8.8.8 and 2001:4860:4860::8888
6. Add a static hostname entry in /etc/hosts mapping both IPv4 and IPv6 addresses to your hostname
7. Apply the configuration without rebooting and verify connectivity

### Hints and Resources
1. Ubuntu uses **Netplan** for network configuration. Configuration files are stored in `/etc/netplan/` with a `.yaml` extension
2. Use `netplan try` to test configurations safely (it auto-reverts after timeout)
3. For hostname configuration, use the `hostnamectl` command and update `/etc/hosts`
4. The basic Netplan structure uses `network: version: 2, renderer: networkd, ethernets:`
5. Reference: [Netplan Documentation](https://netplan.io/reference) and [Ubuntu Networking Guide](https://ubuntu.com/server/docs/network-configuration)

### Estimated Time and Difficulty
**30-45 minutes, Intermediate**

### Verification
To verify your solution:
- Run `hostnamectl` and confirm the hostname is set correctly
- Run `ip addr show` and verify both IPv4 and IPv6 addresses are assigned
- Run `ip route show` and check the default gateway is configured
- Test IPv4 connectivity: `ping -c 3 8.8.8.8`
- Test IPv6 connectivity: `ping6 -c 3 2001:4860:4860::8888`
- Check DNS resolution: `resolvectl status` or `cat /etc/resolv.conf`
- Verify hostname resolution: `getent hosts webserver01.example.com`

<details>
<summary>Solution</summary>

```bash
# Task 1: Set hostname
sudo hostnamectl set-hostname webserver01.example.com
hostnamectl  # Verify

# Task 2-5: Configure network interfaces
# Edit or create Netplan configuration
sudo nano /etc/netplan/01-netcfg.yaml

# Add the following configuration (adjust interface names as needed):
```

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s3:
      dhcp4: true
      dhcp6: false
    enp0s8:
      addresses:
        - 192.168.100.50/24
        - 2001:db8:1::50/64
      routes:
        - to: default
          via: 192.168.100.1
      nameservers:
        addresses:
          - 8.8.8.8
          - 2001:4860:4860::8888
```

```bash
# Task 6: Add hostname entries to /etc/hosts
sudo nano /etc/hosts

# Add these lines:
192.168.100.50    webserver01.example.com webserver01
2001:db8:1::50    webserver01.example.com webserver01

# Task 7: Apply configuration
sudo netplan try  # Test configuration (auto-reverts after 120s)
# Press Enter to accept if everything works

# Or apply directly:
sudo netplan apply

# Verify configuration
ip addr show
ip route show
ip -6 route show
resolvectl status
```

**Explanation**:
- **hostnamectl**: Sets both static and transient hostnames, updates `/etc/hostname` automatically
- **Netplan YAML**: Uses structured YAML format; `dhcp4: true` enables DHCP, `addresses` sets static IPs
- **IPv6 notation**: Uses CIDR notation (e.g., `/64`); IPv6 addresses are written with colons
- **routes**: Defines the default gateway; for IPv6, you could add `- to: default, via: fe80::1`
- **nameservers**: Sets DNS servers that will be written to `/etc/resolv.conf` via systemd-resolved
- **/etc/hosts**: Local hostname resolution; entries here are checked before DNS

**Additional Network Testing Commands**:
```bash
# Test network connectivity
ping -c 3 192.168.100.1  # Gateway
ping6 -c 3 2001:db8:1::1  # IPv6 gateway

# Check routing tables
ip route show
ip -6 route show

# Monitor network interfaces
ip link show
networkctl status

# Check DNS resolution
nslookup google.com
dig google.com
resolvectl query google.com
```

</details>

### Extensions
1. **Advanced**: Configure a secondary IP address (IP alias) on the same interface and set up policy-based routing to route traffic from this IP through a different gateway
2. **Challenge**: Set up IPv6 SLAAC (Stateless Address Autoconfiguration) alongside your static configuration, and configure IPv6 privacy extensions

---

## Task 2: Synchronize System Time with NTP Servers

### Learning Objectives
- Configure and manage system time using timedatectl and systemd-timesyncd
- Understand NTP synchronization and troubleshoot time-related issues
- Set timezone and verify time synchronization status

### Context

You're managing a distributed application cluster where accurate time synchronization is critical. Log correlation, authentication tokens, and distributed transactions all depend on synchronized clocks across servers. A time drift of even a few seconds can cause authentication failures, certificate validation errors, and make troubleshooting nearly impossible.

Your task is to ensure the system is synchronized with reliable NTP servers, configure the correct timezone, and verify that time synchronization is working correctly. This is a fundamental skill for any Linux system administrator and a common requirement in production environments.

### Task Instructions

**Prerequisites**: Start with a fresh Ubuntu 22.04 LTS system.
```bash
# Check current time status
timedatectl status

# Check if timesyncd service is active
systemctl status systemd-timesyncd
```

**Your Tasks**:
1. Set the system timezone to "America/New_York" (or your preferred timezone)
2. Enable NTP synchronization
3. Configure systemd-timesyncd to use custom NTP servers: time.google.com (primary) and pool.ntp.org (fallback)
4. Restart the time synchronization service and verify it's syncing
5. Force an immediate time synchronization
6. Configure the hardware clock (RTC) to use UTC
7. Verify the time is synchronized and check the time offset

### Hints and Resources
1. The `timedatectl` command is your primary tool for time management on systemd-based systems
2. Configuration for systemd-timesyncd is in `/etc/systemd/timesyncd.conf`
3. Use `timedatectl timesync-status` to see detailed synchronization information
4. The `NTP=` directive specifies primary servers, `FallbackNTP=` specifies fallback servers
5. Reference: [systemd-timesyncd man page](https://manpages.ubuntu.com/manpages/jammy/man8/systemd-timesyncd.8.html) and [timedatectl man page](https://manpages.ubuntu.com/manpages/jammy/man1/timedatectl.1.html)

### Estimated Time and Difficulty
**15-25 minutes, Beginner**

### Verification
To verify your solution:
- Run `timedatectl status` and confirm "System clock synchronized: yes" and NTP service is active
- Run `timedatectl timesync-status` and verify it shows the configured NTP server and a small offset
- Check the timezone is correctly set
- Run `systemctl status systemd-timesyncd` and ensure it's active and running
- Verify `/etc/systemd/timesyncd.conf` contains your custom NTP servers
- Check journalctl for synchronization messages: `journalctl -u systemd-timesyncd -n 20`

<details>
<summary>Solution</summary>

```bash
# Task 1: Set timezone
sudo timedatectl set-timezone America/New_York
timedatectl status  # Verify

# List all available timezones:
timedatectl list-timezones

# Task 2: Enable NTP synchronization
sudo timedatectl set-ntp true

# Task 3: Configure custom NTP servers
sudo nano /etc/systemd/timesyncd.conf

# Add/modify these lines under [Time] section:
```

```ini
[Time]
NTP=time.google.com
FallbackNTP=pool.ntp.org 0.ubuntu.pool.ntp.org 1.ubuntu.pool.ntp.org
```

```bash
# Task 4: Restart timesyncd service
sudo systemctl restart systemd-timesyncd

# Check service status
systemctl status systemd-timesyncd

# Task 5: Force immediate synchronization
# First, stop the service
sudo systemctl stop systemd-timesyncd

# Manually sync (if ntpdate is available, or use systemd-timesyncd)
sudo systemctl start systemd-timesyncd

# Or use chronyd if available:
# sudo chronyc -a makestep

# Task 6: Set hardware clock to UTC
sudo timedatectl set-local-rtc 0

# Task 7: Verify synchronization
timedatectl status
timedatectl timesync-status

# Check detailed sync information
systemctl status systemd-timesyncd
journalctl -u systemd-timesyncd --since "10 minutes ago"
```

**Explanation**:
- **set-timezone**: Changes system timezone; stored in `/etc/timezone` and `/etc/localtime` symlink
- **set-ntp true**: Enables NTP synchronization via systemd-timesyncd
- **timesyncd.conf**: Configuration file for systemd's lightweight NTP client
- **NTP vs FallbackNTP**: Primary servers are tried first; fallback servers used if primary fails
- **set-local-rtc 0**: Sets hardware clock to UTC (0=UTC, 1=local time); UTC is recommended for servers
- **timesync-status**: Shows current NTP server, polling interval, and time offset

**Additional Time Management Commands**:
```bash
# View current time in different formats
date
date +"%Y-%m-%d %H:%M:%S"
date -u  # UTC time

# Set time manually (disables NTP)
sudo timedatectl set-time "2025-12-29 15:30:00"

# Check hardware clock
sudo hwclock --show
sudo hwclock --systohc  # Sync hardware clock to system time

# Monitor time synchronization
watch -n 5 timedatectl timesync-status

# Alternative: Install and use chrony (more advanced NTP daemon)
sudo apt install chrony
sudo systemctl enable chrony
chronyc sources  # Show time sources
chronyc tracking  # Show synchronization status
```

</details>

### Extensions
1. **Advanced**: Install and configure chrony as an alternative to systemd-timesyncd, configure it to serve time to other systems on your network, and set up authentication between NTP peers
2. **Challenge**: Create a monitoring script that alerts when time drift exceeds 1 second, and configure it to run every 5 minutes via cron

---

## Task 3: Configure OpenSSH Server with Security Hardening

### Learning Objectives
- Install and configure OpenSSH server with security best practices
- Implement key-based authentication and disable password authentication
- Configure SSH client for convenient and secure connections

### Context

You've been tasked with securing SSH access to a production server. The company's security policy requires that SSH must use key-based authentication only, run on a non-standard port, and prevent root login. Additionally, you need to set up SSH multiplexing on your client machine to improve connection speed for automated scripts that frequently connect to servers.

SSH is the primary method for remote administration of Linux systems, making it a critical target for attacks. Proper SSH configuration is essential for maintaining security while enabling efficient remote management. This task covers both server-side hardening and client-side optimization, both important for LFCS certification and real-world administration.

### Task Instructions

**Prerequisites**: Two systems (or one system accessing itself via localhost) running Ubuntu 22.04 LTS.
```bash
# Install OpenSSH server if not already present
sudo apt update
sudo apt install openssh-server

# Check SSH service status
sudo systemctl status ssh
```

**Your Tasks**:

**Part A: Server Configuration**
1. Change the SSH port from 22 to 2222
2. Disable root login via SSH
3. Disable password authentication (only after setting up key auth)
4. Allow only specific users to SSH (create a user "admin" and allow only this user)
5. Set a login banner message from `/etc/issue.net`
6. Configure SSH to automatically disconnect idle sessions after 10 minutes
7. Enable verbose logging for SSH connections

**Part B: Client Configuration**
8. Generate an ED25519 SSH key pair with a passphrase
9. Copy the public key to the server for the "admin" user
10. Configure SSH client to use connection multiplexing and compression
11. Create an SSH config entry for easy connection with alias "prodserver"

### Hints and Resources
1. SSH server configuration is in `/etc/ssh/sshd_config`; client configuration is in `~/.ssh/config`
2. Always test SSH changes in a separate terminal before closing your current connection
3. Use `ssh-keygen` for key generation and `ssh-copy-id` for key deployment
4. The `ClientAliveInterval` and `ClientAliveCountMax` directives control idle timeout
5. Reference: [OpenSSH Server Configuration](https://manpages.ubuntu.com/manpages/jammy/man5/sshd_config.5.html) and [SSH Client Config](https://manpages.ubuntu.com/manpages/jammy/man5/ssh_config.5.html)

### Estimated Time and Difficulty
**35-50 minutes, Intermediate**

### Verification
To verify your solution:
- Attempt to SSH as root and confirm it's denied
- Attempt to SSH with password and confirm it's denied (after key setup)
- Successfully connect using SSH key authentication on port 2222
- Verify the banner displays when connecting
- Check SSH logs: `sudo journalctl -u ssh -n 20`
- Verify connection multiplexing: make multiple SSH connections and check only one TCP connection exists (`ss -tn | grep 2222`)
- Test the SSH alias: `ssh prodserver`
- Leave an SSH session idle and verify it disconnects after the timeout period

<details>
<summary>Solution</summary>

```bash
# Server Configuration

# Task 1-7: Configure SSH server
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.backup  # Backup first!
sudo nano /etc/ssh/sshd_config

# Modify/add these directives:
```

```bash
Port 2222
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
AllowUsers admin
Banner /etc/issue.net
ClientAliveInterval 300
ClientAliveCountMax 2
LogLevel VERBOSE
```

```bash
# Create the admin user (if not exists)
sudo useradd -m -s /bin/bash admin
sudo passwd admin  # Set a temporary password

# Create banner
sudo nano /etc/issue.net
# Add content like:
# "WARNING: Unauthorized access prohibited. All connections are monitored and recorded."

# Test configuration syntax
sudo sshd -t

# Restart SSH service
sudo systemctl restart ssh

# Verify SSH is listening on new port
sudo ss -tlnp | grep 2222

# Client Configuration (run on client machine or same machine)

# Task 8: Generate SSH key pair
ssh-keygen -t ed25519 -C "admin@prodserver" -f ~/.ssh/id_ed25519_prodserver
# Enter a strong passphrase when prompted

# Task 9: Copy public key to server
# Method 1: Using ssh-copy-id (on standard port first, before changing)
ssh-copy-id -i ~/.ssh/id_ed25519_prodserver.pub admin@localhost

# If port already changed:
ssh-copy-id -i ~/.ssh/id_ed25519_prodserver.pub -p 2222 admin@localhost

# Method 2: Manual copy
# On client:
cat ~/.ssh/id_ed25519_prodserver.pub
# On server (as admin user):
mkdir -p ~/.ssh
chmod 700 ~/.ssh
nano ~/.ssh/authorized_keys  # Paste the public key
chmod 600 ~/.ssh/authorized_keys

# Task 10-11: Configure SSH client
nano ~/.ssh/config

# Add configuration:
```

```bash
# Global settings
Host *
    ServerAliveInterval 60
    ServerAliveCountMax 3
    Compression yes
    ControlMaster auto
    ControlPath ~/.ssh/sockets/%r@%h:%p
    ControlPersist 10m

# Specific host configuration
Host prodserver
    HostName localhost
    Port 2222
    User admin
    IdentityFile ~/.ssh/id_ed25519_prodserver
```

```bash
# Create sockets directory for multiplexing
mkdir -p ~/.ssh/sockets
chmod 700 ~/.ssh/sockets

# Set proper permissions on config
chmod 600 ~/.ssh/config

# Test connection
ssh prodserver
# Or: ssh -p 2222 admin@localhost
```

**Explanation**:
- **Port 2222**: Security through obscurity; reduces automated attacks on default port
- **PermitRootLogin no**: Prevents direct root access; users must login as regular user then sudo
- **PasswordAuthentication no**: Enforces key-based auth; much more secure than passwords
- **AllowUsers**: Whitelist approach; only specified users can login
- **ClientAliveInterval/CountMax**: Server sends keepalive every 300s; disconnects after 2 missed (600s idle)
- **ED25519**: Modern, fast, secure key algorithm; preferred over RSA
- **ControlMaster**: Multiplexes multiple SSH sessions over single TCP connection
- **ControlPersist 10m**: Keeps connection open for 10 minutes for reuse

**Additional SSH Security Configurations**:
```bash
# Two-factor authentication with Google Authenticator
sudo apt install libpam-google-authenticator
google-authenticator  # Set up for user

# In /etc/ssh/sshd_config:
ChallengeResponseAuthentication yes
AuthenticationMethods publickey,keyboard-interactive

# Restrict SSH to specific IP addresses
# In /etc/ssh/sshd_config:
AllowUsers admin@192.168.1.0/24

# Use fail2ban to block brute force attempts
sudo apt install fail2ban
sudo systemctl enable fail2ban
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
# Edit jail.local to configure SSH protection

# Monitor SSH attempts
sudo journalctl -u ssh -f  # Follow mode
sudo tail -f /var/log/auth.log
```

</details>

### Extensions
1. **Advanced**: Set up SSH certificate-based authentication using an SSH CA (Certificate Authority), and configure the server to trust certificates signed by your CA instead of individual public keys
2. **Challenge**: Configure SSH to use two-factor authentication combining SSH keys with Google Authenticator (PAM), and set up port knocking to hide the SSH port until a specific sequence of packets is received

---

## Task 4: Configure Firewall Rules and Port Forwarding with UFW and iptables

### Learning Objectives
- Configure firewall rules using UFW (Uncomplicated Firewall) and iptables
- Implement port forwarding and NAT (Network Address Translation)
- Understand packet filtering concepts and troubleshoot firewall issues

### Context

Your company is setting up a new application server that needs specific firewall rules. The server runs a web application on port 8080 (internal) that should be accessible via port 80 from the internet. Additionally, you need to allow SSH access only from the office network (192.168.1.0/24), block all other incoming connections, and configure NAT to allow internal network clients (10.0.0.0/24) to access the internet through this server.

Firewall configuration is a critical security skill for system administrators. Improperly configured firewalls can either leave systems vulnerable to attacks or block legitimate traffic, causing service disruptions. This task covers both UFW (the Ubuntu-friendly interface) and iptables (the underlying technology), essential for LFCS exam success.

### Task Instructions

**Prerequisites**: Ubuntu 22.04 LTS with root/sudo access.
```bash
# Check if UFW is installed
sudo ufw status

# Install if needed
sudo apt update
sudo apt install ufw

# Check current iptables rules
sudo iptables -L -v -n
```

**Your Tasks**:

**Part A: Using UFW**
1. Enable UFW and set default policies: deny incoming, allow outgoing
2. Allow SSH (port 2222 from previous task) only from 192.168.1.0/24
3. Allow HTTP (port 80) and HTTPS (port 443) from anywhere
4. Allow DNS queries (port 53 UDP and TCP)
5. Enable logging for denied connections
6. Configure rate limiting on SSH to prevent brute force attacks

**Part B: Using iptables (NAT and Port Forwarding)**
7. Enable IP forwarding in the kernel
8. Configure SNAT/Masquerading for the internal network 10.0.0.0/24 to access internet
9. Set up port forwarding (DNAT) to redirect incoming traffic on port 80 to internal port 8080
10. Add a rule to drop all incoming traffic from a specific malicious IP: 203.0.113.42
11. Make iptables rules persistent across reboots

### Hints and Resources
1. UFW is a frontend to iptables; you can use both, but UFW may overwrite some iptables rules
2. IP forwarding is controlled by `/proc/sys/net/ipv4/ip_forward` and `/etc/sysctl.conf`
3. NAT rules go in the `nat` table, use `-t nat` flag with iptables
4. Use `iptables-persistent` package to save rules across reboots
5. Reference: [UFW Documentation](https://help.ubuntu.com/community/UFW) and [iptables Tutorial](https://netfilter.org/documentation/)

### Estimated Time and Difficulty
**40-60 minutes, Advanced**

### Verification
To verify your solution:
- Run `sudo ufw status verbose` and verify all UFW rules are present
- Check default policies are set correctly
- Test SSH connection from allowed and denied networks (use `ssh -p 2222`)
- Verify HTTP/HTTPS ports are open: `sudo nmap -p 80,443 localhost`
- Check iptables rules: `sudo iptables -L -v -n` and `sudo iptables -t nat -L -v -n`
- Verify IP forwarding: `cat /proc/sys/net/ipv4/ip_forward` should show "1"
- Test port forwarding: `curl localhost:80` should reach service on port 8080
- Check logs for denied connections: `sudo tail /var/log/ufw.log`
- Reboot and verify rules persist

<details>
<summary>Solution</summary>

```bash
# UFW Configuration

# Task 1: Enable UFW with default policies
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Task 2: Allow SSH from specific network
sudo ufw allow from 192.168.1.0/24 to any port 2222 proto tcp
# Or if using standard port 22:
# sudo ufw allow from 192.168.1.0/24 to any port 22

# Task 3: Allow HTTP and HTTPS
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
# Or: sudo ufw allow 'Apache Full'

# Task 4: Allow DNS
sudo ufw allow 53/tcp
sudo ufw allow 53/udp
# Or: sudo ufw allow 'Bind9'

# Task 5: Enable logging
sudo ufw logging on
# For more verbose logging:
# sudo ufw logging high

# Task 6: Configure rate limiting on SSH
sudo ufw limit 2222/tcp
# This allows max 6 connections per 30 seconds from an IP

# Enable UFW
sudo ufw enable

# Check status
sudo ufw status numbered
sudo ufw status verbose

# iptables Configuration

# Task 7: Enable IP forwarding
# Temporary (until reboot):
sudo sysctl -w net.ipv4.ip_forward=1

# Permanent:
sudo nano /etc/sysctl.conf
# Uncomment or add:
net.ipv4.ip_forward=1

# Apply changes
sudo sysctl -p

# Verify
cat /proc/sys/net/ipv4/ip_forward  # Should show 1

# Task 8: Configure NAT/Masquerading
# Assuming eth0 or enp0s3 is the external interface
# Replace with your actual interface name
EXTERNAL_IF="enp0s3"
INTERNAL_NET="10.0.0.0/24"

sudo iptables -t nat -A POSTROUTING -s $INTERNAL_NET -o $EXTERNAL_IF -j MASQUERADE

# Alternative using SNAT (if you have static external IP):
# EXTERNAL_IP="203.0.113.10"
# sudo iptables -t nat -A POSTROUTING -s $INTERNAL_NET -o $EXTERNAL_IF -j SNAT --to-source $EXTERNAL_IP

# Task 9: Port forwarding (DNAT)
# Forward external port 80 to internal port 8080 on same host
sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -j REDIRECT --to-port 8080

# Or forward to different host:
# sudo iptables -t nat -A PREROUTING -i $EXTERNAL_IF -p tcp --dport 80 -j DNAT --to-destination 10.0.0.100:8080
# sudo iptables -A FORWARD -p tcp -d 10.0.0.100 --dport 8080 -j ACCEPT

# Task 10: Block specific IP
sudo iptables -A INPUT -s 203.0.113.42 -j DROP

# Task 11: Make rules persistent
# Install iptables-persistent
sudo apt install iptables-persistent

# During installation, choose Yes to save current rules

# To manually save rules:
sudo netfilter-persistent save
# Or:
# sudo iptables-save | sudo tee /etc/iptables/rules.v4
# sudo ip6tables-save | sudo tee /etc/iptables/rules.v6

# Verify saved rules
cat /etc/iptables/rules.v4

# To restore rules manually:
# sudo iptables-restore < /etc/iptables/rules.v4
```

**Explanation**:
- **UFW default policies**: Sets baseline security; deny all incoming unless explicitly allowed
- **ufw limit**: Implements rate limiting using recent module; blocks IPs exceeding connection threshold
- **IP forwarding**: Kernel parameter allowing packet routing between interfaces; required for NAT/routing
- **MASQUERADE**: Dynamic SNAT for interfaces with changing IPs (e.g., DHCP)
- **PREROUTING chain**: In nat table, processes packets before routing decision
- **REDIRECT**: Special DNAT target for forwarding to local ports
- **iptables-persistent**: Service that loads rules from /etc/iptables/ at boot

**Additional Firewall Management**:
```bash
# View UFW rules with numbers for easy deletion
sudo ufw status numbered

# Delete a rule by number
sudo ufw delete 3

# Delete by specification
sudo ufw delete allow 80/tcp

# Reset UFW (remove all rules)
sudo ufw reset

# View detailed iptables rules
sudo iptables -L -v -n --line-numbers
sudo iptables -t nat -L -v -n --line-numbers

# Delete iptables rule by line number
sudo iptables -D INPUT 5
sudo iptables -t nat -D PREROUTING 1

# Flush all iptables rules (WARNING: may lose connection)
sudo iptables -F
sudo iptables -t nat -F
sudo iptables -t mangle -F
sudo iptables -X

# Insert rule at specific position
sudo iptables -I INPUT 1 -p tcp --dport 22 -j ACCEPT

# Connection tracking
sudo iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT

# Logging
sudo iptables -A INPUT -j LOG --log-prefix "iptables-dropped: " --log-level 4

# Check logs
sudo tail -f /var/log/kern.log | grep iptables
```

**Testing Port Forwarding**:
```bash
# Start a test service on port 8080
python3 -m http.server 8080 &

# Test from localhost
curl localhost:80
curl localhost:8080

# Check DNAT rule is being used
sudo iptables -t nat -L -v -n

# Monitor connections
sudo conntrack -L | grep 8080
sudo ss -tlnp | grep -E '(80|8080)'
```

</details>

### Extensions
1. **Advanced**: Implement application-level firewall rules using nftables (the modern replacement for iptables), create custom chains for different service types, and configure connection rate limiting per source IP
2. **Challenge**: Set up a transparent proxy using TPROXY iptables target to redirect all HTTP traffic (port 80) to a Squid proxy running on port 3128, and configure geoblocking to deny traffic from specific countries using ipset and GeoIP database

---

## Task 5: Configure Static Routing and Network Bonding

### Learning Objectives
- Configure static routes for multi-network routing scenarios
- Implement network bonding (link aggregation) for redundancy and performance
- Understand routing tables, network interfaces, and failover mechanisms

### Context

Your data center has multiple network segments: a public network (192.168.1.0/24), a private application network (10.10.0.0/24), and a storage network (172.16.0.0/24). The default gateway only provides access to the public network, so you need to configure static routes to reach the other networks through specific gateway routers. Additionally, to ensure high availability and increased bandwidth for critical database servers, you need to configure network bonding that combines two network interfaces into a single logical interface with automatic failover.

This scenario is common in enterprise environments where network segmentation is used for security and performance. Understanding static routing and bonding is crucial for building resilient infrastructure and is a key competency for LFCS certification.

### Task Instructions

**Prerequisites**: Ubuntu 22.04 LTS VM with at least two network interfaces (for bonding).
```bash
# Check available network interfaces
ip link show

# View current routing table
ip route show
route -n
```

**Your Tasks**:

**Part A: Static Routing**
1. Add a static route for the 10.10.0.0/24 network via gateway 192.168.1.254
2. Add a static route for the 172.16.0.0/24 network via gateway 192.168.1.253
3. Configure these routes to persist after reboot using Netplan
4. Add a route for a specific host 203.0.113.100 through gateway 192.168.1.1
5. Verify routing table and test connectivity

**Part B: Network Bonding**
6. Install bonding module and dependencies
7. Create a bonding interface (bond0) using two Ethernet interfaces in active-backup mode
8. Assign IP address 192.168.100.100/24 to the bond interface
9. Configure the bond to use miimon link monitoring
10. Test failover by disabling one interface and verifying connectivity remains

### Hints and Resources
1. Netplan uses the `routes:` directive to configure static routes; format is `- to: <network>, via: <gateway>`
2. Bonding requires the `bonding` kernel module and configuration in Netplan or `/etc/network/interfaces`
3. Common bonding modes: 0 (balance-rr), 1 (active-backup), 2 (balance-xor), 4 (802.3ad), 6 (balance-alb)
4. Use `ip route add` for temporary routes; use Netplan for persistent routes
5. Reference: [Ubuntu Netplan Bonding](https://netplan.io/examples#bonding) and [Linux Bonding Documentation](https://www.kernel.org/doc/Documentation/networking/bonding.txt)

### Estimated Time and Difficulty
**35-50 minutes, Advanced**

### Verification
To verify your solution:
- Run `ip route show` and verify all static routes are present
- Test routes: `ip route get 10.10.0.50` and `ip route get 172.16.0.50`
- Check route persistence after reboot: `sudo reboot`, then check routes again
- Verify bonding interface: `ip link show bond0`
- Check bond status: `cat /proc/net/bonding/bond0`
- Test failover: disable one slave interface with `sudo ip link set <interface> down`, verify bond0 stays up
- Ping through bond interface: `ping -I bond0 192.168.100.1`
- Check which interface is active in bond: `cat /proc/net/bonding/bond0 | grep "Currently Active Slave"`

<details>
<summary>Solution</summary>

```bash
# Static Routing

# Task 1-4: Add temporary static routes (not persistent)
sudo ip route add 10.10.0.0/24 via 192.168.1.254
sudo ip route add 172.16.0.0/24 via 192.168.1.253
sudo ip route add 203.0.113.100 via 192.168.1.1

# View routing table
ip route show
route -n

# Task 3: Make routes persistent via Netplan
sudo nano /etc/netplan/01-netcfg.yaml

# Example configuration with static routes:
```

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s3:
      addresses:
        - 192.168.1.50/24
      routes:
        - to: default
          via: 192.168.1.1
        - to: 10.10.0.0/24
          via: 192.168.1.254
        - to: 172.16.0.0/24
          via: 192.168.1.253
        - to: 203.0.113.100/32
          via: 192.168.1.1
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]
```

```bash
# Apply configuration
sudo netplan apply

# Verify routes
ip route show

# Task 5: Test routing
ip route get 10.10.0.50
ip route get 172.16.0.50
traceroute 10.10.0.1  # If traceroute is installed

# Network Bonding

# Task 6: Load bonding module
sudo modprobe bonding

# Make it load at boot
echo "bonding" | sudo tee -a /etc/modules

# Verify module is loaded
lsmod | grep bonding

# Task 7-9: Configure bonding via Netplan
# Backup existing config
sudo cp /etc/netplan/01-netcfg.yaml /etc/netplan/01-netcfg.yaml.backup

sudo nano /etc/netplan/01-netcfg.yaml

# Add bonding configuration:
```

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s8:
      dhcp4: no
    enp0s9:
      dhcp4: no
  bonds:
    bond0:
      interfaces:
        - enp0s8
        - enp0s9
      addresses:
        - 192.168.100.100/24
      parameters:
        mode: active-backup
        mii-monitor-interval: 100
        primary: enp0s8
      routes:
        - to: default
          via: 192.168.100.1
```

```bash
# Apply bonding configuration
sudo netplan try  # Test first
sudo netplan apply

# Alternative: Complete configuration with both routing and bonding
```

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s3:
      addresses:
        - 192.168.1.50/24
      routes:
        - to: default
          via: 192.168.1.1
        - to: 10.10.0.0/24
          via: 192.168.1.254
        - to: 172.16.0.0/24
          via: 192.168.1.253
      nameservers:
        addresses: [8.8.8.8]
    enp0s8:
      dhcp4: no
    enp0s9:
      dhcp4: no
  bonds:
    bond0:
      interfaces: [enp0s8, enp0s9]
      addresses:
        - 192.168.100.100/24
      parameters:
        mode: active-backup
        mii-monitor-interval: 100
        primary: enp0s8
        fail-over-mac-policy: active
```

```bash
# Apply bonding configuration
sudo netplan try  # Test first
sudo netplan apply

# Task 10: Verify bonding
ip link show bond0
cat /proc/net/bonding/bond0

# Expected output shows:
# - Bonding Mode
# - Currently Active Slave
# - MII Status
# - Slave Interface details

# Test connectivity
ping -c 3 -I bond0 192.168.100.1

# Test failover
# In one terminal, run continuous ping:
ping -I bond0 192.168.100.1

# In another terminal, disable primary interface:
sudo ip link set enp0s8 down

# Check bond status (should switch to enp0s9)
cat /proc/net/bonding/bond0 | grep "Currently Active Slave"

# Ping should continue without interruption

# Re-enable interface
sudo ip link set enp0s8 up

# Monitor interface status
ip -s link show bond0
```

**Explanation**:
- **Static routes**: `via` specifies next-hop gateway; `/32` is host route (single IP)
- **ip route add**: Adds temporary route to kernel routing table; lost on reboot
- **Netplan routes**: Persistent routes; applied during network initialization
- **Bonding mode active-backup (1)**: Only one slave active at a time; provides failover but not load balancing
- **mii-monitor-interval**: Checks link status every 100ms; detects failures quickly
- **primary**: Preferred slave when both are available
- **fail-over-mac-policy: active**: Bond MAC address changes to active slave's MAC

**Bonding Modes Explained**:
```bash
# Mode 0 - balance-rr (Round-robin)
# Load balancing, packets distributed sequentially
mode: balance-rr

# Mode 1 - active-backup
# Fault tolerance, only one interface active
mode: active-backup

# Mode 2 - balance-xor
# XOR of source and destination MAC
mode: balance-xor

# Mode 4 - 802.3ad (LACP)
# Requires switch support for dynamic link aggregation
mode: 802.3ad
lacp-rate: fast

# Mode 5 - balance-tlb
# Adaptive transmit load balancing
mode: balance-tlb

# Mode 6 - balance-alb
# Adaptive load balancing (requires driver support)
mode: balance-alb
```

**Additional Routing Commands**:
```bash
# View detailed routing table
ip route show table all
netstat -rn

# View routing cache
ip route show cache

# Delete a route
sudo ip route del 10.10.0.0/24

# Add route with metric (priority)
sudo ip route add 10.10.0.0/24 via 192.168.1.254 metric 100

# Add route to specific routing table
sudo ip route add 10.10.0.0/24 via 192.168.1.254 table 100

# View specific routing table
ip route show table 100

# Policy-based routing
sudo ip rule add from 192.168.1.0/24 table 100
ip rule show

# Monitor routing changes
ip monitor route
```

**Bonding Troubleshooting**:
```bash
# Check bonding driver info
ethtool bond0

# Monitor bond0 status in real-time
watch -n 1 cat /proc/net/bonding/bond0

# Check slave interface status
ip -d link show enp0s8
ip -d link show enp0s9

# View bonding parameters
cat /sys/class/net/bond0/bonding/mode
cat /sys/class/net/bond0/bonding/miimon
cat /sys/class/net/bond0/bonding/slaves

# Manually add/remove slaves (temporary)
echo "+enp0s8" | sudo tee /sys/class/net/bond0/bonding/slaves
echo "-enp0s8" | sudo tee /sys/class/net/bond0/bonding/slaves

# Check network statistics
ip -s link show bond0
ifconfig bond0
```

</details>

### Extensions
1. **Advanced**: Configure policy-based routing using multiple routing tables to route traffic from different source networks through different gateways, and set up ECMP (Equal-Cost Multi-Path) routing to load balance traffic across multiple gateways
2. **Challenge**: Create a bridge interface (br0) that bridges together a bond interface and a VLAN interface, configure it for use in a virtualization environment (e.g., for KVM virtual machines), and implement proper VLAN tagging (802.1Q) on the bonded interfaces

---

## Task 6: Configure Nginx as a Reverse Proxy and Load Balancer

### Learning Objectives
- Install and configure Nginx as a reverse proxy server
- Implement load balancing across multiple backend servers
- Configure SSL/TLS termination and understand proxy-related concepts

### Context

Your company is experiencing increased traffic to its web application and needs to scale horizontally by adding more application servers. You've been tasked with setting up Nginx as a reverse proxy and load balancer to distribute traffic across three backend application servers (running on ports 8081, 8082, and 8083). Additionally, the proxy should terminate SSL/TLS connections, add security headers, implement health checks, and provide session persistence for applications that require it.

Reverse proxies and load balancers are critical components in modern web architectures, providing scalability, security, and high availability. Understanding how to configure and manage these systems is essential for any system administrator working with web applications and is relevant to LFCS certification.

### Task Instructions

**Prerequisites**: Ubuntu 22.04 LTS with sudo access.
```bash
# Update package list
sudo apt update

# Install Nginx
sudo apt install nginx

# Check Nginx status
sudo systemctl status nginx
```

**Your Tasks**:
1. Install Nginx and verify it's running
2. Create three simple backend services (using Python HTTP server or netcat) on ports 8081, 8082, 8083
3. Configure Nginx as a reverse proxy to forward requests to these backends using round-robin load balancing
4. Configure health checks for backend servers
5. Implement session persistence (sticky sessions) using IP hash
6. Add custom headers to identify which backend served the request
7. Generate a self-signed SSL certificate and configure HTTPS with SSL termination on Nginx
8. Configure Nginx to redirect all HTTP traffic to HTTPS
9. Add security headers (X-Frame-Options, X-Content-Type-Options, etc.)
10. Set up access logging with custom format showing backend server information

### Hints and Resources
1. Nginx configuration is typically in `/etc/nginx/nginx.conf` and `/etc/nginx/sites-available/`
2. Use the `upstream` directive to define backend server pools
3. The `proxy_pass` directive forwards requests to upstream servers
4. Use `openssl` to generate self-signed certificates
5. Reference: [Nginx Documentation](https://nginx.org/en/docs/) and [Nginx Reverse Proxy Guide](https://docs.nginx.com/nginx/admin-guide/web-server/reverse-proxy/)

### Estimated Time and Difficulty
**45-60 minutes, Advanced**

### Verification
To verify your solution:
- Access `http://localhost` and verify you get a response from backend servers
- Make multiple requests and confirm load balancing is working (responses come from different backends)
- Check backend server logs to see requests coming from Nginx
- Verify HTTPS works: `curl -k https://localhost` (the -k flag allows self-signed certs)
- Confirm HTTP redirects to HTTPS: `curl -I http://localhost`
- Check health check functionality: stop one backend, verify traffic only goes to healthy backends
- Verify custom headers: `curl -I https://localhost -k | grep X-Backend-Server`
- Review access logs: `sudo tail -f /var/log/nginx/access.log`
- Test session persistence: make requests from same IP, verify they go to same backend

<details>
<summary>Solution</summary>

```bash
# Task 1: Install and verify Nginx
sudo apt update
sudo apt install nginx
sudo systemctl status nginx
sudo systemctl enable nginx

# Task 2: Create simple backend services
# Create directory for backend scripts
mkdir -p ~/backends

# Backend 1 (port 8081)
cat > ~/backends/backend1.py << 'EOF'
#!/usr/bin/env python3
from http.server import HTTPServer, BaseHTTPRequestHandler

class Handler(BaseHTTPRequestHandler):
    def do_GET(self):
        self.send_response(200)
        self.send_header('Content-type', 'text/html')
        self.end_headers()
        self.wfile.write(b'<h1>Backend Server 1 (Port 8081)</h1>')
    def log_message(self, format, *args):
        print(f"Backend 1: {format % args}")

HTTPServer(('127.0.0.1', 8081), Handler).serve_forever()
EOF

chmod +x ~/backends/backend1.py

# Backend 2 (port 8082)
cat > ~/backends/backend2.py << 'EOF'
#!/usr/bin/env python3
from http.server import HTTPServer, BaseHTTPRequestHandler

class Handler(BaseHTTPRequestHandler):
    def do_GET(self):
        self.send_response(200)
        self.send_header('Content-type', 'text/html')
        self.end_headers()
        self.wfile.write(b'<h1>Backend Server 2 (Port 8082)</h1>')
    def log_message(self, format, *args):
        print(f"Backend 2: {format % args}")

HTTPServer(('127.0.0.1', 8082), Handler).serve_forever()
EOF

chmod +x ~/backends/backend2.py

# Backend 3 (port 8083)
cat > ~/backends/backend3.py << 'EOF'
#!/usr/bin/env python3
from http.server import HTTPServer, BaseHTTPRequestHandler

class Handler(BaseHTTPRequestHandler):
    def do_GET(self):
        self.send_response(200)
        self.send_header('Content-type', 'text/html')
        self.end_headers()
        self.wfile.write(b'<h1>Backend Server 3 (Port 8083)</h1>')
    def log_message(self, format, *args):
        print(f"Backend 3: {format % args}")

HTTPServer(('127.0.0.1', 8083), Handler).serve_forever()
EOF

chmod +x ~/backends/backend3.py

# Start backends (in background or separate terminals)
python3 ~/backends/backend1.py &
python3 ~/backends/backend2.py &
python3 ~/backends/backend3.py &

# Verify backends are running
curl http://localhost:8081
curl http://localhost:8082
curl http://localhost:8083

# Task 7: Generate self-signed SSL certificate
sudo mkdir -p /etc/nginx/ssl
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout /etc/nginx/ssl/nginx-selfsigned.key \
  -out /etc/nginx/ssl/nginx-selfsigned.crt \
  -subj "/C=US/ST=State/L=City/O=Organization/CN=localhost"

# Generate Diffie-Hellman parameters for enhanced security
sudo openssl dhparam -out /etc/nginx/ssl/dhparam.pem 2048

# Task 3-10: Configure Nginx
sudo nano /etc/nginx/sites-available/loadbalancer

# Add the following configuration:
```

```nginx
# Upstream backend servers
upstream backend_servers {
    # Load balancing method: round-robin (default)
    # Alternatives: least_conn, ip_hash, hash $request_uri consistent
    
    # For round-robin:
    # server 127.0.0.1:8081;
    # server 127.0.0.1:8082;
    # server 127.0.0.1:8083;
    
    # For IP hash (session persistence):
    ip_hash;
    
    server 127.0.0.1:8081 max_fails=3 fail_timeout=30s;
    server 127.0.0.1:8082 max_fails=3 fail_timeout=30s;
    server 127.0.0.1:8083 max_fails=3 fail_timeout=30s;
}

# Custom log format
log_format detailed '$remote_addr - $remote_user [$time_local] '
                    '"$request" $status $body_bytes_sent '
                    '"$http_referer" "$http_user_agent" '
                    'backend: $upstream_addr response_time: $upstream_response_time';

# Redirect HTTP to HTTPS
server {
    listen 80;
    listen [::]:80;
    server_name localhost;
    
    return 301 https://$server_name$request_uri;
}

# HTTPS server
server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name localhost;
    
    # SSL configuration
    ssl_certificate /etc/nginx/ssl/nginx-selfsigned.crt;
    ssl_certificate_key /etc/nginx/ssl/nginx-selfsigned.key;
    ssl_dhparam /etc/nginx/ssl/dhparam.pem;
    
    # SSL security settings
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 10m;
    
    # Security headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
    
    # Access logging
    access_log /var/log/nginx/loadbalancer-access.log detailed;
    error_log /var/log/nginx/loadbalancer-error.log;
    
    location / {
        # Proxy to backend servers
        proxy_pass http://backend_servers;
        
        # Proxy headers
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # Custom header to identify backend
        add_header X-Backend-Server $upstream_addr always;
        
        # Timeouts
        proxy_connect_timeout 30s;
        proxy_send_timeout 30s;
        proxy_read_timeout 30s;
        
        # Health check (passive)
        # Active health checks require nginx-plus or third-party modules
    }
    
    # Status page (optional)
    location /nginx_status {
        stub_status on;
        access_log off;
        allow 127.0.0.1;
        deny all;
    }
}
```

```bash
# Enable the site
sudo ln -s /etc/nginx/sites-available/loadbalancer /etc/nginx/sites-enabled/

# Remove default site
sudo rm /etc/nginx/sites-enabled/default

# Test configuration
sudo nginx -t

# Reload Nginx
sudo systemctl reload nginx

# Verify Nginx is running
sudo systemctl status nginx

# Testing

# Test HTTP redirect to HTTPS
curl -I http://localhost
# Should return 301 redirect to https://localhost

# Test HTTPS (allow self-signed cert)
curl -k https://localhost

# Test multiple times to see load balancing
for i in {1..10}; do
    curl -k -s https://localhost | grep -o "Backend Server [0-9]"
done

# Test with IP hash (same IP should go to same backend)
for i in {1..5}; do
    curl -k -s https://localhost
done

# Check custom headers
curl -k -I https://localhost

# Monitor logs
sudo tail -f /var/log/nginx/loadbalancer-access.log

# Test health check by stopping one backend
# Kill backend 2
pkill -f backend2.py

# Make requests - should only go to backends 1 and 3
for i in {1..6}; do
    curl -k -s https://localhost | grep -o "Backend Server [0-9]"
    sleep 1
done

# Restart backend 2
python3 ~/backends/backend2.py &

# Check Nginx status page
curl http://localhost/nginx_status
```

**Explanation**:
- **upstream**: Defines a group of backend servers for load balancing
- **ip_hash**: Ensures requests from same client IP go to same backend (session persistence)
- **max_fails/fail_timeout**: Health check parameters; marks server down after 3 fails for 30s
- **proxy_pass**: Forwards requests to upstream backend servers
- **proxy_set_header**: Passes original client information to backends
- **SSL certificates**: self-signed certs for development; use Let's Encrypt for production
- **return 301**: HTTP to HTTPS redirect for all traffic
- **add_header**: Adds security headers to responses
- **stub_status**: Provides basic Nginx statistics

**Alternative Load Balancing Methods**:
```nginx
# Least connections
upstream backend_servers {
    least_conn;
    server 127.0.0.1:8081;
    server 127.0.0.1:8082;
    server 127.0.0.1:8083;
}

# Hash-based on URI
upstream backend_servers {
    hash $request_uri consistent;
    server 127.0.0.1:8081;
    server 127.0.0.1:8082;
    server 127.0.0.1:8083;
}

# Weighted round-robin
upstream backend_servers {
    server 127.0.0.1:8081 weight=3;
    server 127.0.0.1:8082 weight=2;
    server 127.0.0.1:8083 weight=1;
}

# With backup server
upstream backend_servers {
    server 127.0.0.1:8081;
    server 127.0.0.1:8082;
    server 127.0.0.1:8083 backup;
}
```

**Advanced Nginx Configuration**:
```nginx
# Rate limiting
http {
    limit_req_zone $binary_remote_addr zone=one:10m rate=10r/s;
    
    server {
        location / {
            limit_req zone=one burst=20 nodelay;
            proxy_pass http://backend_servers;
        }
    }
}

# Caching
http {
    proxy_cache_path /var/cache/nginx levels=1:2 keys_zone=my_cache:10m max_size=1g inactive=60m;
    
    server {
        location / {
            proxy_cache my_cache;
            proxy_cache_valid 200 60m;
            proxy_cache_use_stale error timeout http_500 http_502 http_503;
            add_header X-Cache-Status $upstream_cache_status;
            proxy_pass http://backend_servers;
        }
    }
}

# WebSocket support
location /ws/ {
    proxy_pass http://backend_servers;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
}
```

</details>

### Extensions
1. **Advanced**: Configure Nginx with dynamic upstream configuration using DNS or Consul for service discovery, implement active health checks using a third-party module (nginx-upstream-check-module), and set up request/connection rate limiting per client IP
2. **Challenge**: Deploy HAProxy as an alternative to Nginx, configure it for Layer 4 (TCP) load balancing for a MySQL database cluster, implement ACLs for routing based on request patterns, and set up monitoring with HAProxy stats page and integration with Prometheus/Grafana

---

## Summary and Next Steps

You've completed comprehensive exercises covering all aspects of networking for the LFCS exam:

- **Task 1**: Static and dynamic network configuration with IPv4/IPv6 using Netplan
- **Task 2**: Time synchronization with NTP for distributed systems
- **Task 3**: SSH server hardening and client optimization
- **Task 4**: Firewall configuration with UFW and iptables, including NAT and port forwarding
- **Task 5**: Static routing and network bonding for high availability
- **Task 6**: Reverse proxy and load balancing with Nginx

**Key Takeaways**:
- Networking is fundamental to all Linux system administration tasks
- Understanding the difference between configuration methods (Netplan, NetworkManager, ifupdown) is crucial for different distributions
- Security hardening (SSH, firewalls) must be balanced with accessibility
- High availability requires redundancy at multiple layers (bonding, load balancing)
- Modern infrastructure relies on reverse proxies for scalability and security

**Recommended Practice**:
1. Build a multi-server lab environment with different network segments
2. Practice network troubleshooting using tools like tcpdump, netstat, ss, and traceroute
3. Implement a complete web application stack with load balancing and SSL/TLS
4. Create automation scripts for network configuration management
5. Explore advanced topics like VLAN tagging, VPN configuration, and network namespaces

**Additional Resources**:
- [LFCS Exam Domains](https://training.linuxfoundation.org/certification/linux-foundation-certified-sysadmin-lfcs/)
- [Ubuntu Networking Guide](https://ubuntu.com/server/docs/network-configuration)
- [Netplan Documentation](https://netplan.io/)
- [Nginx Official Documentation](https://nginx.org/en/docs/)
- [Linux Network Administrators Guide](https://www.tldp.org/LDP/nag2/index.html)

Continue practicing these skills in real Ubuntu environments to build confidence for the LFCS exam!

---