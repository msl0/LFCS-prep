# LFCS Practice: Storage Management

This document provides hands-on practice tasks for mastering storage management on Linux systems, a critical component of the LFCS exam. All exercises are designed for Ubuntu 22.04 LTS and simulate real-world system administration scenarios.

---

## Task 1: Logical Volume Management (LVM) Setup and Expansion

### Learning Objectives
- Master the creation and configuration of LVM physical volumes, volume groups, and logical volumes
- Implement dynamic storage expansion without system downtime
- Understand LVM architecture and its advantages over traditional partitioning

### Context
Your company's database server is running low on disk space. The application stores critical customer data in `/var/lib/mysql`, and the current partition is 80% full. Management has allocated additional disk space, but the application cannot tolerate downtime during business hours. As the system administrator, you need to implement a solution that allows for flexible storage management and seamless expansion in the future.

LVM (Logical Volume Manager) is essential for LFCS candidates because it provides flexibility that traditional partitioning cannot match. With LVM, you can resize storage on-the-fly, create snapshots for backups, and manage storage pools across multiple physical disks. This scenario tests your ability to implement enterprise-grade storage solutions that are both scalable and maintainable.

### Task Instructions

**Prerequisites**: Set up a test environment with simulated block devices
```bash
# Create loop devices to simulate physical disks
sudo dd if=/dev/zero of=/tmp/disk1.img bs=1M count=1024
sudo dd if=/dev/zero of=/tmp/disk2.img bs=1M count=1024
sudo losetup -f /tmp/disk1.img
sudo losetup -f /tmp/disk2.img

# Verify loop devices are created
losetup -a
```

**Your Tasks**:
1. Initialize the two loop devices as LVM physical volumes
2. Create a volume group named `vg_data` using both physical volumes
3. Create a logical volume named `lv_database` with 500MB initial size from the volume group
4. Format the logical volume with ext4 filesystem and mount it at `/mnt/database`
5. Simulate running out of space: create a 400MB file in the mounted filesystem
6. Extend the logical volume to 800MB without unmounting
7. Resize the filesystem to use the new space
8. Verify that the additional space is available

### Hints and Resources
1. Use `pvcreate` to initialize physical volumes, `vgcreate` for volume groups, and `lvcreate` for logical volumes
2. The `-L` flag specifies size in `lvcreate` (e.g., `-L 500M`)
3. To extend a logical volume, use `lvextend` with the `-L` flag for new total size or `-l +100%FREE` for all available space
4. After extending the logical volume, use `resize2fs` for ext4 or `xfs_growfs` for XFS filesystems
5. Reference: [Ubuntu LVM Guide](https://ubuntu.com/server/docs/lvm) and [LVM man pages](https://manpages.ubuntu.com/manpages/jammy/man8/lvm.8.html)

### Estimated Time and Difficulty
**35-45 minutes, Intermediate**

### Verification
To verify your solution:
- Run `pvdisplay` and confirm two physical volumes are active
- Run `vgdisplay vg_data` and verify the volume group shows correct size and free space
- Run `lvdisplay /dev/vg_data/lv_database` and confirm the logical volume is 800MB
- Check `df -h /mnt/database` to ensure the filesystem shows approximately 800MB total
- Verify the test file exists and the filesystem has additional free space

<details>
<summary>Solution</summary>

```bash
# Task 1: Initialize physical volumes
sudo pvcreate /dev/loop0 /dev/loop1

# Verify physical volumes
sudo pvdisplay

# Task 2: Create volume group
sudo vgcreate vg_data /dev/loop0 /dev/loop1

# Verify volume group
sudo vgdisplay vg_data

# Task 3: Create logical volume
sudo lvcreate -L 500M -n lv_database vg_data

# Verify logical volume
sudo lvdisplay /dev/vg_data/lv_database

# Task 4: Format and mount
sudo mkfs.ext4 /dev/vg_data/lv_database
sudo mkdir -p /mnt/database
sudo mount /dev/vg_data/lv_database /mnt/database

# Task 5: Create test file to simulate usage
sudo dd if=/dev/zero of=/mnt/database/testfile bs=1M count=400

# Check current space
df -h /mnt/database

# Task 6: Extend logical volume to 800MB
sudo lvextend -L 800M /dev/vg_data/lv_database

# Task 7: Resize filesystem
sudo resize2fs /dev/vg_data/lv_database

# Task 8: Verify expansion
df -h /mnt/database
sudo lvdisplay /dev/vg_data/lv_database
```

**Explanation**:
- **Tasks 1-2**: `pvcreate` initializes block devices for LVM use, while `vgcreate` pools them into a volume group that acts as a storage container
- **Task 3**: `lvcreate` carves out a logical volume from the volume group, which behaves like a traditional partition but with flexibility
- **Task 4**: The logical volume is formatted with ext4 and mounted like any other block device
- **Tasks 6-7**: `lvextend` increases the logical volume size, but the filesystem doesn't automatically know about the extra space—`resize2fs` expands the ext4 filesystem to fill the enlarged logical volume
- **Note**: For XFS filesystems, use `xfs_growfs /mnt/database` instead of `resize2fs`

**Additional LVM Commands Reference**:
```bash
# Remove logical volume (unmount first)
sudo umount /mnt/database
sudo lvremove /dev/vg_data/lv_database

# Remove volume group
sudo vgremove vg_data

# Remove physical volumes
sudo pvremove /dev/loop0 /dev/loop1

# Cleanup loop devices
sudo losetup -d /dev/loop0
sudo losetup -d /dev/loop1
rm /tmp/disk1.img /tmp/disk2.img
```

</details>

### Extensions
1. **Advanced**: Create an LVM snapshot of `lv_database`, make changes to files, then restore from the snapshot
2. **Challenge**: Migrate the logical volume from one physical volume to another using `pvmove` without unmounting

---

## Task 2: Filesystem Creation, Management, and Troubleshooting

### Learning Objectives
- Create and configure different filesystem types (ext4, XFS) with appropriate options
- Implement filesystem integrity checks and repair procedures
- Configure persistent mount options and understand `/etc/fstab` syntax

### Context
You're managing a multi-purpose file server that hosts different types of workloads. The development team needs a partition for large media files that requires high-performance sequential writes, while the accounting department needs a reliable partition for database files with journaling. Additionally, one of the existing filesystems has been experiencing issues after an unexpected power outage, and you need to diagnose and repair it.

Understanding filesystem types and their use cases is crucial for LFCS exam success. Different filesystems excel at different tasks: ext4 provides excellent general-purpose performance with strong stability, while XFS excels at handling large files and parallel I/O. Knowing when to use each filesystem type, how to configure them properly, and how to troubleshoot filesystem corruption are essential sysadmin skills.

### Task Instructions

**Prerequisites**: Create test block devices
```bash
# Create three loop devices for different filesystem tests
sudo dd if=/dev/zero of=/tmp/fs_ext4.img bs=1M count=512
sudo dd if=/dev/zero of=/tmp/fs_xfs.img bs=1M count=512
sudo dd if=/dev/zero of=/tmp/fs_corrupt.img bs=1M count=256

sudo losetup -f /tmp/fs_ext4.img
sudo losetup -f /tmp/fs_xfs.img
sudo losetup -f /tmp/fs_corrupt.img

# Verify loop devices
losetup -a | grep fs_
```

**Your Tasks**:

**Part A: Filesystem Creation**
1. Create an ext4 filesystem on the first loop device with a volume label "DATA_EXT4" and 4096-byte block size
2. Create an XFS filesystem on the second loop device with a volume label "DATA_XFS"
3. Create mount points at `/mnt/ext4_data` and `/mnt/xfs_data`
4. Mount both filesystems and verify they are accessible

**Part B: Persistent Mounts and Options**
5. Add entries to `/etc/fstab` to mount both filesystems automatically at boot
6. Configure the ext4 mount to use `noatime` option (for performance) and `errors=remount-ro` for error handling
7. Test the fstab configuration using `mount -a` without rebooting

**Part C: Filesystem Troubleshooting**
8. Create an ext4 filesystem on the third loop device and mount it
9. Write some test files to it, then forcefully unmount and simulate corruption
10. Use filesystem check tools to detect and repair the corruption
11. Remount and verify data integrity

### Hints and Resources
1. Use `mkfs.ext4` with `-L` for label and `-b` for block size; use `mkfs.xfs` with `-L` for label
2. The `/etc/fstab` format is: `device mountpoint fstype options dump pass`
3. For UUID-based mounting, use `blkid` to find the filesystem UUID
4. To force filesystem check on ext4, use `fsck.ext4 -f` or `e2fsck -f` (must be unmounted)
5. Reference: [Ubuntu Fstab Guide](https://help.ubuntu.com/community/Fstab) and [ext4 documentation](https://ext4.wiki.kernel.org/)

### Estimated Time and Difficulty
**40-50 minutes, Intermediate**

### Verification
To verify your solution:
- Run `lsblk -f` to see all filesystems with their labels and types
- Check `df -Th` to confirm both filesystems are mounted with correct types
- Run `cat /etc/fstab` and verify the entries are syntactically correct
- Run `mount | grep -E 'ext4_data|xfs_data'` to verify mount options are applied
- For the corrupted filesystem, run `fsck` again and confirm no errors are found
- Create and read files on all three filesystems to ensure they are functional

<details>
<summary>Solution</summary>

```bash
# Task 1: Create ext4 filesystem with label and custom block size
sudo mkfs.ext4 -L DATA_EXT4 -b 4096 /dev/loop0

# Task 2: Create XFS filesystem with label
sudo mkfs.xfs -L DATA_XFS /dev/loop1

# Task 3: Create mount points
sudo mkdir -p /mnt/ext4_data /mnt/xfs_data

# Task 4: Mount filesystems
sudo mount /dev/loop0 /mnt/ext4_data
sudo mount /dev/loop1 /mnt/xfs_data

# Verify mounts
df -Th | grep -E 'ext4_data|xfs_data'
lsblk -f | grep -E 'loop0|loop1'

# Task 5-6: Get UUIDs for fstab entries
sudo blkid /dev/loop0
sudo blkid /dev/loop1

# Add to /etc/fstab (replace UUIDs with actual values from blkid)
sudo bash -c 'cat >> /etc/fstab << EOF
# Ext4 data partition
UUID=$(blkid -s UUID -o value /dev/loop0)  /mnt/ext4_data  ext4  noatime,errors=remount-ro  0  2
# XFS data partition
UUID=$(blkid -s UUID -o value /dev/loop1)  /mnt/xfs_data   xfs   defaults                   0  2
EOF'

# Alternative: Using device path instead of UUID
# /dev/loop0  /mnt/ext4_data  ext4  noatime,errors=remount-ro  0  2
# /dev/loop1  /mnt/xfs_data   xfs   defaults                   0  2

# Task 7: Test fstab without rebooting
sudo umount /mnt/ext4_data /mnt/xfs_data
sudo mount -a

# Verify mount options
mount | grep ext4_data
mount | grep xfs_data

# Task 8: Create filesystem on third device
sudo mkfs.ext4 -L TEST_CORRUPT /dev/loop2
sudo mkdir -p /mnt/corrupt_test
sudo mount /dev/loop2 /mnt/corrupt_test

# Task 9: Create test data and simulate corruption
sudo bash -c 'echo "Important data" > /mnt/corrupt_test/file1.txt'
sudo bash -c 'echo "More data" > /mnt/corrupt_test/file2.txt'

# Unmount before corruption simulation
sudo umount /mnt/corrupt_test

# Simulate corruption by writing random data to the device
sudo dd if=/dev/urandom of=/dev/loop2 bs=1M count=1 seek=10 conv=notrunc

# Task 10: Check and repair filesystem
sudo fsck.ext4 -f -y /dev/loop2

# Alternative: Interactive mode (without -y)
# sudo e2fsck -f /dev/loop2

# Task 11: Remount and verify
sudo mount /dev/loop2 /mnt/corrupt_test
ls -la /mnt/corrupt_test
cat /mnt/corrupt_test/file1.txt 2>/dev/null || echo "File may be corrupted"
```

**Explanation**:
- **Task 1**: The `-b 4096` flag sets the block size, which affects performance for different workload types. Ext4's journaling provides reliability.
- **Task 2**: XFS is optimized for parallel I/O and large files, making it ideal for media workloads
- **Tasks 5-6**: Using UUID instead of device names in `/etc/fstab` ensures correct mounting even if device names change. The `noatime` option improves performance by not updating access times. `errors=remount-ro` protects data by remounting read-only on errors
- **Task 10**: `fsck.ext4 -f` forces a check even if the filesystem appears clean, while `-y` automatically answers "yes" to repair prompts

**Filesystem Comparison Reference**:
```bash
# Check filesystem type
df -T /mnt/ext4_data

# View detailed filesystem info
sudo tune2fs -l /dev/loop0  # For ext4
sudo xfs_info /mnt/xfs_data  # For XFS

# Change filesystem label
sudo e2label /dev/loop0 NEW_LABEL  # For ext4
sudo xfs_admin -L NEW_LABEL /dev/loop1  # For XFS

# Check filesystem usage statistics
sudo dumpe2fs /dev/loop0 | grep -i "block"  # For ext4
```

</details>

### Extensions
1. **Advanced**: Create a RAID 1 array using `mdadm`, format it with XFS, and configure it in `/etc/fstab` with appropriate options
2. **Challenge**: Set up filesystem quotas on the ext4 partition to limit a specific user to 100MB of storage

---

## Task 3: Swap Space Configuration and Management

### Learning Objectives
- Create and configure swap partitions and swap files for memory management
- Understand swap priority and how to optimize swap usage
- Monitor and troubleshoot swap performance issues

### Context
Your organization is running several Ubuntu servers with 4GB of RAM each. The development team frequently runs memory-intensive applications that occasionally cause out-of-memory (OOM) situations, leading to process crashes. While additional RAM is on order, you need an immediate solution to prevent application failures. Additionally, some servers were deployed without adequate swap space, and you need to add swap without reinstalling the system.

Swap space management is a fundamental LFCS competency because it directly impacts system stability and performance. Understanding when and how to configure swap, the differences between swap partitions and files, and how to tune swap behavior can prevent critical system failures. This knowledge is essential for managing production systems where memory constraints are common.

### Task Instructions

**Prerequisites**: Prepare the test environment
```bash
# Check current swap configuration
swapon --show
free -h

# Create a loop device for swap partition simulation
sudo dd if=/dev/zero of=/tmp/swap_partition.img bs=1M count=512
sudo losetup -f /tmp/swap_partition.img

# Verify loop device
losetup -a | grep swap_partition
```

**Your Tasks**:
1. Create a 512MB swap partition on the loop device
2. Enable the swap partition with priority 10
3. Verify the swap is active and check total available swap
4. Create a 256MB swap file at `/swapfile`
5. Set correct permissions (600) on the swap file
6. Enable the swap file with priority 5 (lower than the partition)
7. Configure both swap spaces to be activated automatically at boot by editing `/etc/fstab`
8. Test the configuration by disabling and re-enabling all swap
9. Monitor swap usage and simulate memory pressure to observe swap behavior

### Hints and Resources
1. Use `mkswap` to format a partition or file as swap space
2. Use `swapon` to activate swap and `swapoff` to deactivate it
3. The `-p` flag with `swapon` sets the priority (higher numbers = higher priority)
4. For swap files, use `dd` or `fallocate` to create the file, then `chmod 600` for security
5. Reference: [Ubuntu Swap FAQ](https://help.ubuntu.com/community/SwapFaq) and [swapon man page](https://manpages.ubuntu.com/manpages/jammy/man8/swapon.8.html)

### Estimated Time and Difficulty
**25-35 minutes, Beginner to Intermediate**

### Verification
To verify your solution:
- Run `swapon --show` and confirm both swap spaces appear with correct sizes and priorities
- Run `free -h` and verify total swap matches the sum of both swap spaces (768MB)
- Check `cat /etc/fstab | grep swap` to confirm entries are present
- Run `sudo swapoff -a && sudo swapon -a` to test that swap can be disabled and re-enabled
- Use `watch -n 1 free -h` while running a memory-intensive program to see swap being used

<details>
<summary>Solution</summary>

```bash
# Task 1: Create swap partition
sudo mkswap /dev/loop0

# Alternative: Add a label
# sudo mkswap -L SWAP_PARTITION /dev/loop0

# Task 2: Enable swap partition with priority 10
sudo swapon -p 10 /dev/loop0

# Task 3: Verify swap is active
swapon --show
free -h

# Task 4: Create swap file (256MB)
sudo dd if=/dev/zero of=/swapfile bs=1M count=256

# Alternative: Using fallocate (faster)
# sudo fallocate -l 256M /swapfile

# Task 5: Set correct permissions
sudo chmod 600 /swapfile

# Verify permissions
ls -lh /swapfile

# Format as swap
sudo mkswap /swapfile

# Task 6: Enable swap file with priority 5
sudo swapon -p 5 /swapfile

# Verify both swaps are active
swapon --show

# Task 7: Add to /etc/fstab for persistence
sudo bash -c 'cat >> /etc/fstab << EOF
# Swap partition (higher priority)
/dev/loop0  none  swap  sw,pri=10  0  0
# Swap file (lower priority)
/swapfile   none  swap  sw,pri=5   0  0
EOF'

# Alternative: Using UUID for swap partition
# UUID=$(blkid -s UUID -o value /dev/loop0)  none  swap  sw,pri=10  0  0

# Task 8: Test configuration
sudo swapoff -a
swapon --show  # Should show nothing

sudo swapon -a
swapon --show  # Should show both swaps

# Task 9: Monitor swap usage
# Watch swap in real-time
watch -n 1 free -h

# In another terminal, simulate memory pressure
# stress --vm 2 --vm-bytes 512M --timeout 30s

# Or use a simple memory consumer
# sudo apt install -y stress
```

**Explanation**:
- **Task 1**: `mkswap` formats the device with a swap signature, preparing it for use as virtual memory
- **Task 2**: The priority system determines which swap space the kernel uses first. Higher priority (10) means it will be used before lower priority (5)
- **Task 4-5**: Swap files must have 600 permissions to prevent other users from potentially reading sensitive memory contents
- **Task 7**: The `/etc/fstab` entry uses "none" as the mount point since swap isn't mounted in the traditional sense. The `sw` option indicates swap, and `pri=` sets the priority

**Additional Swap Management Commands**:
```bash
# View swap usage by process
for dir in /proc/*/; do
    pid=$(basename "$dir")
    if [[ $pid =~ ^[0-9]+$ ]]; then
        swap=$(grep VmSwap "/proc/$pid/status" 2>/dev/null | awk '{print $2}')
        if [[ $swap -gt 0 ]]; then
            cmd=$(cat "/proc/$pid/cmdline" 2>/dev/null | tr '\0' ' ')
            echo "PID: $pid, Swap: ${swap}kB, Command: $cmd"
        fi
    fi
done | sort -k4 -rn | head

# Adjust swappiness (how aggressively kernel uses swap)
cat /proc/sys/vm/swappiness
sudo sysctl vm.swappiness=10  # Lower = less aggressive (default is 60)

# Make swappiness permanent
echo "vm.swappiness=10" | sudo tee -a /etc/sysctl.conf

# Remove swap file (disable first)
sudo swapoff /swapfile
sudo rm /swapfile

# Remove swap partition
sudo swapoff /dev/loop0
```

</details>

### Extensions
1. **Advanced**: Configure `vm.swappiness` to different values and observe the impact on swap usage under memory pressure
2. **Challenge**: Create a systemd service that monitors swap usage and sends an alert when swap usage exceeds 80%

---

## Task 4: Network File System (NFS) Configuration

### Learning Objectives
- Configure an NFS server to export filesystems to network clients
- Mount remote NFS shares with appropriate security and performance options
- Troubleshoot common NFS connectivity and permission issues

### Context
Your organization has a centralized file server that hosts project documentation and shared resources. Multiple development workstations need access to these files, and maintaining local copies leads to synchronization issues and wasted storage. The infrastructure team has decided to implement NFS to provide seamless file sharing across the network. As the system administrator, you need to set up both the NFS server and configure client workstations to mount the shared directories.

NFS knowledge is critical for the LFCS exam because network storage is ubiquitous in enterprise environments. Understanding how to export filesystems securely, configure client mounts with proper options, and troubleshoot network filesystem issues are essential skills. This task simulates real-world scenarios where centralized storage improves collaboration and reduces administrative overhead.

### Task Instructions

**Prerequisites**: Set up NFS server environment (simulating server and client on same system)
```bash
# Install NFS server and client packages
sudo apt update
sudo apt install -y nfs-kernel-server nfs-common

# Create directories for NFS exports
sudo mkdir -p /srv/nfs/projects
sudo mkdir -p /srv/nfs/shared

# Create test data
sudo bash -c 'echo "Project Documentation" > /srv/nfs/projects/README.txt'
sudo bash -c 'echo "Shared Resources" > /srv/nfs/shared/info.txt'

# Create client mount points
sudo mkdir -p /mnt/nfs_projects
sudo mkdir -p /mnt/nfs_shared
```

**Your Tasks**:

**Part A: NFS Server Configuration**
1. Configure `/srv/nfs/projects` to be exported to localhost with read-write access
2. Configure `/srv/nfs/shared` to be exported to localhost with read-only access
3. Apply the NFS export configuration and verify exports are active
4. Ensure the NFS server starts automatically at boot

**Part B: NFS Client Configuration**
5. Mount the `/srv/nfs/projects` export to `/mnt/nfs_projects` using NFSv4
6. Mount the `/srv/nfs/shared` export to `/mnt/nfs_shared` with read-only option
7. Test write access on the read-write mount and verify read-only enforcement on the other
8. Add both mounts to `/etc/fstab` for automatic mounting at boot
9. Use the `_netdev` option to ensure mounts wait for network availability

**Part C: Troubleshooting**
10. Verify file permissions work correctly across NFS
11. Check NFS server status and active connections
12. Test the mounts survive a reboot simulation (unmount and remount)

### Hints and Resources
1. NFS exports are configured in `/etc/exports` with format: `directory client(options)`
2. Use `exportfs -arv` to apply changes to `/etc/exports` without restarting
3. Common export options include: `rw` (read-write), `ro` (read-only), `sync`, `no_subtree_check`
4. NFSv4 uses a single port (2049) and has better security than NFSv3
5. Reference: [Ubuntu NFS Guide](https://ubuntu.com/server/docs/network-file-system-nfs) and [exports man page](https://manpages.ubuntu.com/manpages/jammy/man5/exports.5.html)

### Estimated Time and Difficulty
**30-40 minutes, Intermediate**

### Verification
To verify your solution:
- Run `sudo exportfs -v` on the server to see active exports with their options
- Run `showmount -e localhost` to list available exports
- Check `df -hT | grep nfs` to confirm both NFS mounts are active
- Try creating a file in `/mnt/nfs_projects` (should succeed)
- Try creating a file in `/mnt/nfs_shared` (should fail with read-only error)
- Run `sudo systemctl status nfs-server` to verify the service is running
- Check `/etc/fstab` contains correct NFS mount entries

<details>
<summary>Solution</summary>

```bash
# Task 1-2: Configure NFS exports
sudo bash -c 'cat > /etc/exports << EOF
# NFS exports configuration
/srv/nfs/projects  127.0.0.1(rw,sync,no_subtree_check,no_root_squash)
/srv/nfs/shared    127.0.0.1(ro,sync,no_subtree_check)
EOF'

# Alternative: Export to entire subnet
# /srv/nfs/projects  192.168.1.0/24(rw,sync,no_subtree_check)

# Task 3: Apply export configuration
sudo exportfs -arv

# Verify exports
sudo exportfs -v
showmount -e localhost

# Task 4: Enable NFS server at boot
sudo systemctl enable nfs-server
sudo systemctl start nfs-server
sudo systemctl status nfs-server

# Task 5: Mount projects directory (read-write)
sudo mount -t nfs -o vers=4 localhost:/srv/nfs/projects /mnt/nfs_projects

# Task 6: Mount shared directory (read-only)
sudo mount -t nfs -o vers=4,ro localhost:/srv/nfs/shared /mnt/nfs_shared

# Verify mounts
df -hT | grep nfs
mount | grep nfs

# Task 7: Test access
# Test write on read-write mount
sudo bash -c 'echo "Test write" > /mnt/nfs_projects/test.txt'
cat /mnt/nfs_projects/test.txt

# Test write on read-only mount (should fail)
sudo bash -c 'echo "Should fail" > /mnt/nfs_shared/test.txt' 2>&1 | head -1

# Task 8-9: Add to /etc/fstab
sudo bash -c 'cat >> /etc/fstab << EOF
# NFS mounts
localhost:/srv/nfs/projects  /mnt/nfs_projects  nfs  defaults,_netdev,vers=4  0  0
localhost:/srv/nfs/shared    /mnt/nfs_shared    nfs  ro,_netdev,vers=4        0  0
EOF'

# Alternative: Using IP address
# 127.0.0.1:/srv/nfs/projects  /mnt/nfs_projects  nfs  defaults,_netdev  0  0

# Task 10: Verify file permissions
ls -la /mnt/nfs_projects/
ls -la /mnt/nfs_shared/

# Task 11: Check NFS server status
sudo systemctl status nfs-server

# View active NFS connections
sudo nfsstat -c  # Client statistics
sudo nfsstat -s  # Server statistics

# View mounted NFS filesystems
nfsstat -m

# Task 12: Test fstab configuration
sudo umount /mnt/nfs_projects /mnt/nfs_shared
sudo mount -a

# Verify remount successful
df -hT | grep nfs
```

**Explanation**:
- **Tasks 1-2**: The `/etc/exports` file defines what directories are shared and with what permissions. `rw` allows read-write, `ro` allows read-only. `sync` ensures data is written to disk before confirming to clients. `no_subtree_check` improves reliability. `no_root_squash` allows root on client to have root access (use carefully!)
- **Task 3**: `exportfs -arv` exports all (`-a`) directories, re-exporting (`-r`) and showing verbose output (`-v`)
- **Tasks 5-6**: The `-o vers=4` option forces NFSv4, which has better security and performance than older versions
- **Task 9**: The `_netdev` option tells the system to wait for network before mounting, preventing boot delays

**Additional NFS Commands Reference**:
```bash
# Unmount NFS shares
sudo umount /mnt/nfs_projects
sudo umount /mnt/nfs_shared

# Force unmount if busy
sudo umount -f /mnt/nfs_projects

# Lazy unmount (unmount when no longer busy)
sudo umount -l /mnt/nfs_projects

# Unexport all NFS shares
sudo exportfs -ua

# Re-export all shares
sudo exportfs -ra

# Check NFS server logs
sudo journalctl -u nfs-server -f

# Restart NFS server
sudo systemctl restart nfs-server
```

</details>

### Extensions
1. **Advanced**: Configure NFS with Kerberos authentication (NFSv4 with `sec=krb5`) for enhanced security
2. **Challenge**: Set up an NFS server to export home directories with user-specific permissions and test with multiple user accounts

---

## Task 5: Automount Configuration and Storage Performance Monitoring

### Learning Objectives
- Configure systemd automount units for on-demand filesystem mounting
- Implement autofs for automatic mounting of network and local filesystems
- Monitor storage performance using various tools and interpret metrics

### Context
Your company's file server hosts dozens of network shares that different teams access occasionally throughout the day. Keeping all shares mounted continuously consumes system resources and creates unnecessary network traffic. Additionally, portable USB drives are frequently connected for backups, but users forget to manually mount them. You need to implement an automount solution that mounts filesystems only when accessed and unmounts them after a period of inactivity. Furthermore, management has requested detailed reports on storage I/O performance to justify hardware upgrades.

Automounting and performance monitoring are advanced LFCS topics that demonstrate a deeper understanding of Linux systems. Automounting improves resource efficiency and user experience by mounting filesystems on-demand. Performance monitoring skills enable you to identify bottlenecks, plan capacity, and troubleshoot slow storage. These skills are essential for managing large-scale production environments where efficiency and proactive monitoring are critical.

### Task Instructions

**Prerequisites**: Set up test environment
```bash
# Create a directory to simulate a network share
sudo mkdir -p /srv/automount_source/documents
sudo bash -c 'echo "Automounted content" > /srv/automount_source/documents/readme.txt'

# Install automount and monitoring tools
sudo apt update
sudo apt install -y autofs sysstat iotop

# Create loop device for performance testing
sudo dd if=/dev/zero of=/tmp/perf_test.img bs=1M count=1024
sudo losetup -f /tmp/perf_test.img
sudo mkfs.ext4 /dev/loop0
```

**Your Tasks**:

**Part A: Systemd Automount**
1. Create a systemd mount unit for `/srv/automount_source/documents` to be mounted at `/mnt/auto_docs`
2. Create a corresponding automount unit that triggers the mount on access
3. Enable the automount unit and test that the directory is mounted only when accessed
4. Verify the mount is automatically unmounted after a timeout period

**Part B: Autofs Configuration**
5. Configure autofs to automatically mount the loop device at `/mnt/autofs_test` when accessed
6. Set the autofs timeout to 60 seconds
7. Test the autofs configuration by accessing the mount point
8. Monitor autofs activity and verify automatic unmounting

**Part C: Storage Performance Monitoring**
9. Use `iostat` to monitor I/O statistics for all block devices
10. Use `iotop` to identify which processes are generating I/O load
11. Generate disk I/O using `dd` and observe the metrics
12. Create a performance report showing read/write speeds and IOPS

### Hints and Resources
1. Systemd mount units go in `/etc/systemd/system/` and use `.mount` extension; automount units use `.automount`
2. The mount unit filename must match the mount path with `-` replacing `/` (e.g., `mnt-auto_docs.mount`)
3. Autofs is configured in `/etc/auto.master` (master map) and individual map files
4. Use `iostat -x 1` for extended statistics every second; `iotop` requires root privileges
5. Reference: [systemd.automount](https://manpages.ubuntu.com/manpages/jammy/man5/systemd.automount.5.html) and [autofs guide](https://help.ubuntu.com/community/Autofs)

### Estimated Time and Difficulty
**45-55 minutes, Advanced**

### Verification
To verify your solution:
- Run `systemctl list-units --type=automount` to see active automount units
- Check `mount | grep auto_docs` before and after accessing `/mnt/auto_docs`
- Run `systemctl status mnt-auto_docs.automount` to verify the automount unit is active
- For autofs, run `sudo automount -m` to show active automounts
- Check `iostat -x 2 3` to see I/O statistics over 6 seconds
- Run `iotop -o` (as root) to see only processes doing I/O
- Verify your performance report includes throughput (MB/s) and await time metrics

<details>
<summary>Solution</summary>

```bash
# Task 1: Create systemd mount unit
sudo bash -c 'cat > /etc/systemd/system/mnt-auto_docs.mount << EOF
[Unit]
Description=Automount test for documents
After=network.target

[Mount]
What=/srv/automount_source/documents
Where=/mnt/auto_docs
Type=none
Options=bind

[Install]
WantedBy=multi-user.target
EOF'

# Task 2: Create automount unit
sudo bash -c 'cat > /etc/systemd/system/mnt-auto_docs.automount << EOF
[Unit]
Description=Automount for documents directory
After=network.target

[Automount]
Where=/mnt/auto_docs
TimeoutIdleSec=30

[Install]
WantedBy=multi-user.target
EOF'

# Task 3: Enable and start automount
sudo systemctl daemon-reload
sudo systemctl enable mnt-auto_docs.automount
sudo systemctl start mnt-auto_docs.automount

# Verify automount is active but mount is not
systemctl status mnt-auto_docs.automount
mount | grep auto_docs  # Should show nothing

# Trigger mount by accessing
ls /mnt/auto_docs
mount | grep auto_docs  # Should now show mounted

# Task 4: Wait for timeout and verify unmount
# Wait 30+ seconds, then check
sleep 35
mount | grep auto_docs  # Should be unmounted

# Task 5-6: Configure autofs
sudo bash -c 'cat > /etc/auto.master.d/test.autofs << EOF
/mnt/autofs /etc/auto.test --timeout=60
EOF'

# Create the map file
sudo bash -c 'cat > /etc/auto.test << EOF
test -fstype=ext4 :/dev/loop0
EOF'

# Restart autofs
sudo systemctl restart autofs
sudo systemctl status autofs

# Task 7: Test autofs
# The mount point is created automatically
ls /mnt/autofs/test
mount | grep autofs

# Task 8: Monitor autofs
# View autofs debug output
sudo automount -f -v &

# Or check status
systemctl status autofs

# Access and verify
cat /mnt/autofs/test/lost+found
df -h | grep autofs

# Wait for timeout and verify unmount
sleep 65
mount | grep autofs  # Should be gone

# Task 9: Monitor I/O statistics
iostat -x 1 5  # Extended stats, 1 second interval, 5 iterations

# Focus on specific device
iostat -x /dev/loop0 2 3

# Task 10: Monitor process I/O (requires root)
sudo iotop -o -b -n 3  # Only show active processes, batch mode, 3 iterations

# Task 11: Generate I/O load for testing
# Mount the test device
sudo mkdir -p /mnt/perf_test
sudo mount /dev/loop0 /mnt/perf_test

# Write test (in background while monitoring)
dd if=/dev/zero of=/mnt/perf_test/testfile bs=1M count=512 oflag=direct

# In another terminal, run:
# iostat -x 1

# Read test
dd if=/mnt/perf_test/testfile of=/dev/null bs=1M iflag=direct

# Task 12: Create performance report
echo "=== Storage Performance Report ===" > ~/storage_report.txt
echo "Date: $(date)" >> ~/storage_report.txt
echo "" >> ~/storage_report.txt

echo "=== Write Performance Test ===" >> ~/storage_report.txt
dd if=/dev/zero of=/mnt/perf_test/write_test bs=1M count=256 oflag=direct 2>&1 | \
    grep -E 'copied|MB/s' >> ~/storage_report.txt

echo "" >> ~/storage_report.txt
echo "=== Read Performance Test ===" >> ~/storage_report.txt
dd if=/mnt/perf_test/write_test of=/dev/null bs=1M iflag=direct 2>&1 | \
    grep -E 'copied|MB/s' >> ~/storage_report.txt

echo "" >> ~/storage_report.txt
echo "=== I/O Statistics ===" >> ~/storage_report.txt
iostat -x /dev/loop0 1 3 >> ~/storage_report.txt

echo "" >> ~/storage_report.txt
echo "=== Disk Usage ===" >> ~/storage_report.txt
df -h /mnt/perf_test >> ~/storage_report.txt

# View the report
cat ~/storage_report.txt
```

**Explanation**:
- **Tasks 1-2**: Systemd mount units define what to mount (`What=`), where to mount it (`Where=`), and the filesystem type. The automount unit references the same path and includes `TimeoutIdleSec` for automatic unmounting after inactivity
- **Tasks 5-6**: Autofs uses a master map (`/etc/auto.master.d/`) that points to specific map files. The map file defines mount points relative to the base path and their options
- **Task 9**: `iostat -x` provides extended statistics including: `r/s` (reads/sec), `w/s` (writes/sec), `rkB/s` (read KB/sec), `wkB/s` (write KB/sec), `await` (average wait time), `%util` (device utilization)
- **Task 10**: `iotop` shows real-time I/O usage per process, helping identify which processes are causing I/O load
- **Task 11**: The `oflag=direct` and `iflag=direct` flags bypass cache for more accurate performance testing

**Additional Performance Monitoring Commands**:
```bash
# View detailed block device statistics
cat /proc/diskstats

# Monitor I/O wait time
vmstat 1 5  # Shows %iowait

# Check for I/O scheduling
cat /sys/block/loop0/queue/scheduler

# View disk latency
ioping /mnt/perf_test

# Stress test with multiple threads
sudo apt install -y fio
fio --name=randwrite --ioengine=libaio --iodepth=16 --rw=randwrite \
    --bs=4k --direct=1 --size=128M --numjobs=4 --runtime=60 \
    --group_reporting --filename=/mnt/perf_test/fio_test

# Monitor specific process I/O
pidstat -d 1  # Disk I/O stats per process

# Clean up
sudo umount /mnt/perf_test
sudo systemctl stop mnt-auto_docs.automount
sudo systemctl disable mnt-auto_docs.automount
```

</details>

### Extensions
1. **Advanced**: Configure autofs to automatically mount user home directories from an NFS server when users log in
2. **Challenge**: Create a monitoring script that logs storage performance metrics to a database and generates alerts when I/O wait exceeds 20% for more than 5 minutes

---

## Summary and Next Steps

Congratulations on completing the LFCS Storage Management practice exercises! You've worked through:

- **LVM configuration** for flexible, scalable storage management
- **Filesystem creation and troubleshooting** with ext4 and XFS
- **Swap space management** for system memory optimization
- **NFS configuration** for network file sharing
- **Automounting and performance monitoring** for advanced storage operations

### Key Takeaways
- LVM provides flexibility that traditional partitioning cannot match
- Different filesystems excel at different workloads (ext4 for general use, XFS for large files)
- Proper swap configuration prevents out-of-memory failures
- NFS enables centralized storage and collaboration
- Automounting and monitoring optimize resource usage and help identify performance issues

### Recommended Next Steps
1. Practice these tasks multiple times until you can complete them without referring to the solutions
2. Experiment with different filesystem types (Btrfs, F2FS) and compare their characteristics
3. Set up more complex storage scenarios (LVM snapshots, RAID arrays, iSCSI)
4. Review the official LFCS domains and ensure you're comfortable with all storage-related competencies
5. Move on to the next practice domain: [User and Group Management](5_users_and_groups.md)

### Additional Resources
- [Linux Foundation LFCS Exam Details](https://training.linuxfoundation.org/certification/linux-foundation-certified-sysadmin-lfcs/)
- [Ubuntu Server Guide - Storage](https://ubuntu.com/server/docs/storage)
- [Red Hat Storage Administration Guide](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/managing_storage_devices/)
- [Linux I/O Tuning Guide](https://wiki.archlinux.org/title/Improving_performance#Storage_devices)

Good luck with your LFCS preparation!
