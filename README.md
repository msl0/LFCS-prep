# LFCS Exam Preparation Guide

A comprehensive collection of hands-on exercises and practice tasks for the Linux Foundation Certified System Administrator (LFCS) exam. All exercises are designed for **Ubuntu 22.04 LTS** and simulate real-world system administration scenarios.

---

## Contents

The exercises are organized by LFCS exam domains:

1. **[Essential Commands](1_essential_commands.md)**
2. **[Operation of Running Systems](2_operation_deployment.md)**
3. **[User and Group Management](5_users_and_groups.md)**
4. **[Networking](3_networking.md)**
5. **[Storage Management](4_storage.md)**

---

## About This Repository

This repository provides task-based learning exercises that mirror real-world scenarios you'll encounter as a Linux system administrator. Each task includes:

- **Learning Objectives** - Key skills you'll practice
- **Context** - Real-world scenario explaining why this matters
- **Step-by-Step Instructions** - Detailed commands and explanations
- **Verification Steps** - How to confirm you've completed the task correctly
- **Solution** - Complete walkthrough with explanations

---

## Getting Started

### Prerequisites
- A Linux environment (Ubuntu 22.04 LTS recommended)
- Root or sudo access
- Basic familiarity with the command line

### Setup Options
**Option 1: Local VM**
- Use VirtualBox, VMware, or Hyper-V
- Install Ubuntu 22.04 LTS

**Option 2: Cloud Instance**
- AWS EC2, Azure VM, or Google Cloud Compute
- Use Ubuntu 22.04 LTS image

**Option 3: WSL2** (Windows users)
```bash
wsl --install -d Ubuntu-22.04
```

---

## How to Use This Guide

1. **Work sequentially** through each domain, or jump to topics you want to strengthen
2. **Read the context** to understand the real-world application
3. **Attempt the task** before looking at the solution
4. **Verify your work** using the provided validation steps
5. **Review the solution** to understand best practices

---

## Exam Tips and Recommendations

### Master man pages and quick references
- Learn to search man pages efficiently using `man -k` or `man -K` for keyword searches
- Consider installing and using `tldr` at the exam start for quick syntax examples
- Be prepared to fix any installation issues manually if needed

### Focus on key topics
Prioritize these high-value areas:
- **Git basics** - Common operations and workflows
- **Docker** - Often includes straightforward questions
- **LVM/disks/mounting** - Partition management and filesystem operations
- **Networking** - IP persistence, SSL, basic troubleshooting
- **Users/groups/ACLs** - Permission management and access control
- **Cron jobs** - Task scheduling
- **NTP/time synchronization** - System time management
- **Services** - systemd and service management
- **iptables/firewalld** - Firewall configuration

> Note: The exam is vendor-agnostic, so avoid relying on distribution-specific tools.

### Read questions carefully
- Misinterpreting requirements is a common failure point
- Take time to fully understand what's being asked
- Flag difficult questions and revisit them later

### Manage time effectively
- Expect 17-40 tasks in 2 hours
- Handle easier tasks first to build momentum
- Use shell features (redirection, piping) efficiently
- Don't over-rely on man pages to avoid running out of time

### Track mistakes and review
- Note errors from practice exams on a whiteboard or notebook
- Redo weak areas repeatedly until confident
- Don't practice new mocks on exam day—just review your notes

### Prepare your exam setup
- Use a large monitor (27+ inches, high resolution) for clear text
- Ensure a quiet space with no interruptions
- Have a clear ID ready for the proctor
- Get plenty of sleep the night before

### General mindset
- The exam is doable with proper hands-on practice
- Focus on understanding mechanisms (permissions, ACLs, etc.) rather than memorizing every command switch
- Learn by doing—experiment and troubleshoot actively

### Post-exam expectations
- Results typically arrive within 24 hours
- Practice platforms like killer.sh are often harder than the actual exam, which builds confidence

### Additional Resources

**Similar LFCS Preparation Repositories:**
- [tomwechsler/lfcs](https://github.com/tomwechsler/lfcs) - Additional practice materials and exercises
- [giulianopz/lfcs](https://github.com/giulianopz/lfcs) - Alternative study guide and resources

---

## Exam Information

**Official Resources:**
- [LFCS Certification Details](https://training.linuxfoundation.org/certification/linux-foundation-certified-sysadmin-lfcs/)

---

## Contributing

If you find any errors or inaccuracies in the exercises, please open a pull request with a clear explanation of what needs to be corrected and why.

---

## Acknowledgments

Created for LFCS exam preparation. Best of luck on your certification journey!
