# Linux Commands for DevOps Engineers

## Table of Contents
1. [What is Linux?](#what-is-linux)
2. [Why Linux is Used in DevOps](#why-linux-is-used-in-devops)
3. [Architecture](#architecture)
4. [Key Components](#key-components)
5. [Installation and Setup](#installation-and-setup)
6. [Common Linux Commands](#common-linux-commands)
7. [Real-World Use Cases](#real-world-use-cases)
8. [Troubleshooting](#troubleshooting)
9. [Best Practices](#best-practices)
10. [Security](#security)
11. [Interview Questions](#interview-questions)

---

## What is Linux?

Linux is a free, open-source operating system kernel created by Linus Torvalds in 1991. It serves as the core component of various Linux distributions (Ubuntu, CentOS, Red Hat, Debian, etc.) and is widely used in enterprise environments, cloud infrastructure, and DevOps ecosystems.

**Key Characteristics:**
- **Open Source:** Source code is publicly available for modification and distribution
- **Multi-user & Multitasking:** Supports multiple users and processes simultaneously
- **Portable:** Runs on diverse hardware platforms
- **Secure:** Built-in user privilege separation and authentication mechanisms
- **Scalable:** Efficiently handles systems from embedded devices to supercomputers

---

## Why Linux is Used in DevOps

Linux is the backbone of modern DevOps practices for several reasons:

### 1. **Cost Efficiency**
- Free and open-source reduces licensing costs
- Minimal resource requirements for servers

### 2. **Stability and Reliability**
- High uptime and stability (often 99.9%+)
- Proven in enterprise environments for decades

### 3. **Scalability**
- Handles massive workloads efficiently
- Ideal for containerization (Docker) and orchestration (Kubernetes)

### 4. **Security**
- Granular user permission system
- Strong authentication and authorization mechanisms
- Regular security updates and patches

### 5. **Automation-Friendly**
- Command-line interface (CLI) enables easy scripting
- Perfect for Infrastructure as Code (IaC)
- Integrates seamlessly with CI/CD pipelines

### 6. **Industry Standard**
- De facto operating system for servers and cloud platforms
- Extensive community support and documentation
- Widely supported by major cloud providers (AWS, Azure, GCP)

---

## Architecture

Linux follows a **Monolithic Kernel Architecture** with layered components:

```
┌─────────────────────────────────────────┐
│         User Applications Layer          │
│  (Shells, Tools, Services, Daemons)     │
├─────────────────────────────────────────┤
│         System Call Interface            │
├─────────────────────────────────────────┤
│            Kernel Space                  │
│  ┌─────────────────────────────────────┐ │
│  │  - Process Management               │ │
│  │  - Memory Management                │ │
│  │  - File System Management           │ │
│  │  - Device Drivers                   │ │
│  │  - Networking Stack                 │ │
│  │  - Interrupt Handling               │ │
│  └─────────────────────────────────────┘ │
├─────────────────────────────────────────┤
│         Hardware Abstraction Layer       │
├─────────────────────────────────────────┤
│              Hardware Layer              │
│  (CPU, Memory, Disk, Network Devices)   │
└─────────────────────────────────────────┘
```

### Key Layers:

1. **User Space:** Applications and user shells
2. **System Call Interface:** Bridge between user and kernel space
3. **Kernel:** Core OS managing resources
4. **Hardware Abstraction Layer:** Device drivers and hardware management

---

## Key Components

### 1. **Bootloader (GRUB/LILO)**
- Loads the kernel into memory during system startup
- Allows boot parameter configuration

### 2. **Kernel**
- Core component managing:
  - Process scheduling
  - Memory management
  - Interrupt handling
  - File system operations

### 3. **Shell**
- Command interpreter (bash, sh, zsh, etc.)
- Processes user commands and scripts

### 4. **File System**
- Hierarchical directory structure (ext4, XFS, Btrfs, etc.)
- Manages file storage and retrieval

### 5. **System Utilities and Libraries**
- **glibc:** GNU C Library for system functions
- **coreutils:** Basic command-line utilities

### 6. **Daemons (Services)**
- Background processes (sshd, httpd, systemd, etc.)
- Run independently of user interaction

### 7. **System Calls**
- Interface for applications to request kernel services
- Examples: `open()`, `read()`, `write()`, `fork()`

---

## Installation and Setup

### Linux Distribution Selection for DevOps

**Popular Choices:**
- **Ubuntu (20.04 LTS, 22.04 LTS):** User-friendly, excellent community support
- **CentOS/Red Hat Enterprise Linux:** Enterprise-grade, long support lifecycle
- **Debian:** Stable, minimal installation footprint
- **Amazon Linux:** Optimized for AWS environments

### Basic Setup Steps

#### 1. **System Update**
```bash
# For Ubuntu/Debian
sudo apt update
sudo apt upgrade -y

# For CentOS/RHEL
sudo yum update -y
sudo yum upgrade -y
```

#### 2. **Install Essential Tools**
```bash
# For Ubuntu/Debian
sudo apt install -y curl wget git vim nano htop net-tools

# For CentOS/RHEL
sudo yum install -y curl wget git vim nano htop net-tools
```

#### 3. **Configure SSH Access**
```bash
sudo systemctl start ssh
sudo systemctl enable ssh
```

#### 4. **Set Up Sudo Access**
```bash
sudo visudo  # Edit sudoers file safely
```

#### 5. **Firewall Configuration**
```bash
# For UFW (Ubuntu)
sudo ufw enable
sudo ufw allow 22/tcp

# For firewalld (CentOS/RHEL)
sudo systemctl start firewalld
sudo firewall-cmd --permanent --add-service=ssh
```

---

## Common Linux Commands

### File and Directory Operations

```bash
# List files with details
ls -la
ls -lh                    # Human-readable sizes
ls -lt                    # Sort by modification time

# Create directories
mkdir directory_name
mkdir -p parent/child/sub # Create parent directories

# Change directory
cd /path/to/directory
cd ~                      # Home directory
cd -                      # Previous directory

# Print working directory
pwd

# Create files
touch filename.txt
echo "content" > file.txt

# Copy files/directories
cp source.txt destination.txt
cp -r source_dir/ destination_dir/

# Move/rename files
mv old_name.txt new_name.txt
mv /path/from/ /path/to/

# Remove files/directories
rm filename.txt
rm -r directory_name/     # Recursive deletion
rm -f filename.txt        # Force deletion without confirmation

# Display file contents
cat filename.txt
less filename.txt         # Paged viewing
more filename.txt
head -n 20 filename.txt   # First 20 lines
tail -n 20 filename.txt   # Last 20 lines

# Find files
find / -name "filename.txt"
find /home -type f -name "*.log"
find /var -mtime -7       # Modified in last 7 days

# Search within files
grep "search_term" filename.txt
grep -r "search_term" /path/
grep -i "case_insensitive" file.txt
grep -n "pattern" file.txt # Show line numbers
```

### User and Permission Management

```bash
# Switch user
su username
sudo command              # Execute as root

# User information
whoami                    # Current user
id                        # User ID and group information
groups                    # User's groups

# Add/remove users
sudo useradd username
sudo userdel username
sudo usermod -aG groupname username  # Add to group

# Change file permissions
chmod 755 filename        # Owner: rwx, Group: r-x, Others: r-x
chmod u+x script.sh       # Add execute for user
chmod g-w filename        # Remove write for group
chmod -R 755 directory/   # Recursive

# Change file ownership
chown user:group filename
chown -R user:group directory/

# View current user permissions
stat filename
getfacl filename          # ACL details
```

### Process Management

```bash
# List running processes
ps aux                    # All processes
ps aux | grep process_name
ps -ef                    # Full format

# Real-time process monitoring
top
htop                      # Enhanced top

# Get process information
pgrep process_name        # PID by name
pidof process_name
ps -p PID -o pid,ppid,cmd

# Send signals to processes
kill PID                  # Terminate process (SIGTERM)
kill -9 PID               # Force kill (SIGKILL)
killall process_name      # Kill by name
pkill -f "pattern"        # Kill matching pattern

# Run processes in background
command &                 # Run in background
nohup command &           # Run immune to hangups
disown                    # Remove from job control

# Job control
jobs                      # List background jobs
fg                        # Bring to foreground
bg                        # Resume in background
```

### System Information and Monitoring

```bash
# System information
uname -a                  # All system information
uname -r                  # Kernel version
hostnamectl              # Hostname and OS info
cat /etc/os-release      # OS details

# Disk usage
df -h                     # Disk space usage
df -i                     # Inode usage
du -sh /path              # Directory size
du -sh /path/*            # Size of each item

# Memory usage
free -h                   # Memory and swap
cat /proc/meminfo         # Detailed memory info
vmstat 1 5                # Virtual memory stats

# CPU information
lscpu                     # CPU details
cat /proc/cpuinfo         # Detailed CPU info
top -n 1 -b | head -n 12  # Quick CPU stats

# System load
uptime                    # Uptime and load average
cat /proc/loadavg         # Load average details

# Disk I/O statistics
iostat -x 1 5             # Extended I/O stats
iotop                     # I/O usage by process
```

### Network Commands

```bash
# Network configuration
ip addr show              # IP addresses
ip route show             # Routing table
ifconfig                  # Network interfaces (deprecated)

# Network connectivity
ping -c 4 example.com     # Test connectivity
traceroute example.com    # Trace route to host
mtr example.com           # Network diagnostic tool

# DNS resolution
nslookup example.com
dig example.com
host example.com
getent hosts example.com

# Network statistics
netstat -an               # All connections
netstat -tuln             # Listen ports
ss -tunap                 # Socket statistics
ss -tnap | grep LISTEN    # Listening services

# Port testing
nc -zv hostname port      # Check port connectivity
lsof -i :port_number      # List process using port
telnet hostname port      # Telnet to port

# Network troubleshooting
curl -I https://example.com  # HTTP headers
wget https://example.com/file # Download file
arp -a                    # ARP table
ip link show              # Link layer info
```

### File Searching and Text Processing

```bash
# Search and filter
grep "pattern" file.txt
grep -E "regex" file.txt  # Extended regex
awk '{print $1}' file.txt # Text processing
sed 's/old/new/g' file.txt # Stream editor
cut -d':' -f1 /etc/passwd # Field extraction

# Sorting and counting
sort file.txt
uniq -c file.txt          # Unique lines with count
wc -l file.txt            # Line count
wc -w file.txt            # Word count
wc -c file.txt            # Byte count

# Text comparison
diff file1.txt file2.txt
comm file1.txt file2.txt
cmp file1.txt file2.txt

# String operations
echo "text" | tr 'a-z' 'A-Z'  # Case conversion
echo "text" | wc -c           # Character count
echo "text" | cut -c1-5       # Extract characters
```

### Archiving and Compression

```bash
# Tar operations
tar -cvf archive.tar files/           # Create tar
tar -xvf archive.tar                  # Extract tar
tar -tvf archive.tar                  # List contents

# Compression
tar -czvf archive.tar.gz files/       # Create gzip
tar -xzvf archive.tar.gz              # Extract gzip
tar -cjvf archive.tar.bz2 files/      # Create bzip2

# Zip operations
zip -r archive.zip files/             # Create zip
unzip archive.zip                     # Extract zip
unzip -l archive.zip                  # List contents

# Compression tools
gzip filename
gunzip filename.gz
bzip2 filename
bunzip2 filename.bz2
```

### System Services and Package Management

```bash
# Service management
systemctl start service_name
systemctl stop service_name
systemctl restart service_name
systemctl enable service_name       # Auto-start on boot
systemctl disable service_name
systemctl status service_name
systemctl list-units --type=service # List all services

# Package management (Ubuntu/Debian)
apt search package_name
apt install package_name
apt remove package_name
apt purge package_name              # Remove with config
apt update                          # Update package list
apt upgrade                         # Upgrade packages
apt autoremove                      # Remove unused packages

# Package management (CentOS/RHEL)
yum search package_name
yum install package_name
yum remove package_name
yum update                          # Update packages
yum list installed                  # List installed packages
yum info package_name               # Package info

# Snap packages
snap install package_name
snap remove package_name
snap update
```

### Logging and System Monitoring

```bash
# View logs
tail -f /var/log/syslog              # Follow syslog
journalctl -u service_name           # Service logs
journalctl -xe                       # Recent errors
journalctl --since "2025-12-28"      # Logs since date
cat /var/log/auth.log                # Authentication logs

# System audit
auditctl -l                          # List audit rules
ausearch -k audit_key                # Search audit logs
audit2why                            # Explain audit logs

# Monitor system activity
dmesg                                # Kernel messages
dmesg | tail -20                     # Recent kernel messages
```

---

## Real-World Use Cases

### 1. **Server Deployment and Management**
```bash
# Deploy a Node.js application
cd /opt/apps
git clone https://github.com/user/app.git
cd app
npm install
npm start &

# Monitor deployment
ps aux | grep node
tail -f logs/app.log
```

### 2. **Log Analysis**
```bash
# Find errors in logs
grep "ERROR" /var/log/app.log | wc -l

# Extract timestamps
grep "ERROR" /var/log/app.log | awk '{print $1, $2}' | sort | uniq -c

# Search for specific patterns
journalctl -u nginx | grep "failed"
```

### 3. **Backup and Disaster Recovery**
```bash
# Create system backup
tar -czvf backup-$(date +%Y%m%d).tar.gz /home /etc /var

# Restore from backup
tar -xzvf backup-20251229.tar.gz -C /

# Incremental backup
rsync -av --delete /source/ /destination/
```

### 4. **Performance Troubleshooting**
```bash
# Identify high CPU processes
top -b -n1 | head -20

# Find memory hogs
ps aux --sort=-%mem | head -10

# Check disk I/O
iostat -x 1 5
iotop -b -n1 -o
```

### 5. **Container Orchestration (Docker/Kubernetes)**
```bash
# Docker operations
docker ps -a                        # List containers
docker logs container_id            # View container logs
docker exec -it container_id bash   # Execute in container

# Kubernetes operations
kubectl get pods
kubectl logs pod_name
kubectl describe node node_name
```

### 6. **CI/CD Pipeline Automation**
```bash
# Build and test automation
#!/bin/bash
set -e
git checkout main
git pull origin main
./build.sh
./test.sh
./deploy.sh
```

### 7. **Security Auditing**
```bash
# Check open ports
netstat -tuln | grep LISTEN

# Verify file permissions
find /home -type f -perm 0777 -ls

# Audit user accounts
awk -F: '$3 >= 1000' /etc/passwd

# Check failed login attempts
grep "Failed password" /var/log/auth.log | wc -l
```

---

## Troubleshooting

### Common Issues and Solutions

#### 1. **High CPU Usage**
```bash
# Identify process
top -p $(pgrep -o process_name)

# Check process details
ps aux | grep process_name

# Kill and restart
systemctl restart service_name
```

#### 2. **Disk Space Issues**
```bash
# Find large files
find / -type f -size +1G -ls

# Check disk usage
du -sh /* | sort -rh | head -10

# Clean up
rm -rf /var/log/*.old
apt autoremove
docker system prune
```

#### 3. **Memory Leaks**
```bash
# Monitor memory growth
watch -n 1 'free -h && echo "---" && ps aux --sort=-%mem | head -5'

# Get memory map
pmap -x $(pgrep process_name)

# Force garbage collection (app-specific)
kill -USR1 $(pgrep process_name)
```

#### 4. **Network Connectivity Issues**
```bash
# Test connectivity
ping -c 4 8.8.8.8
traceroute -m 15 example.com

# Check DNS
nslookup example.com
dig example.com +short

# Verify listening ports
ss -tunap | grep :port

# Check firewall rules
sudo ufw status
sudo firewall-cmd --list-all
```

#### 5. **Service Not Starting**
```bash
# Check service status
systemctl status service_name

# View error logs
journalctl -u service_name -n 50 -p err

# Verify configuration
systemctl list-unit-files | grep service_name
cat /etc/systemd/system/service_name.service
```

#### 6. **SSH Connection Issues**
```bash
# Test SSH connectivity
ssh -vvv user@hostname

# Check SSH daemon
systemctl status ssh
sudo sshd -t                    # Test configuration

# Check SSH logs
tail -f /var/log/auth.log | grep sshd
```

#### 7. **Permission Denied Errors**
```bash
# Check file ownership
ls -la filename
stat filename

# Fix permissions
sudo chown user:group filename
sudo chmod 644 filename         # For files
sudo chmod 755 directory        # For directories

# Check SELinux (if enabled)
getenforce
ls -Z filename
```

---

## Best Practices

### 1. **System Hardening**
```bash
# Disable root login
sudo sed -i 's/PermitRootLogin yes/PermitRootLogin no/' /etc/ssh/sshd_config
sudo systemctl restart ssh

# Set strong SSH options
# /etc/ssh/sshd_config
# PasswordAuthentication no
# PermitEmptyPasswords no
# X11Forwarding no
# Port 2222

# Enable firewall
sudo ufw enable
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow ssh

# Regular updates
sudo apt update && sudo apt upgrade -y
sudo unattended-upgrade
```

### 2. **Log Management**
```bash
# Centralize logs
# Configure rsyslog to forward logs
echo "*.* @@logserver.example.com:514" >> /etc/rsyslog.d/30-forward.conf

# Monitor disk usage
du -sh /var/log

# Rotate logs
sudo nano /etc/logrotate.d/custom

# Example logrotate config:
# /var/log/myapp.log {
#     daily
#     rotate 7
#     compress
#     delaycompress
#     postrotate
#         systemctl reload myapp > /dev/null 2>&1 || true
#     endscript
# }
```

### 3. **Monitoring and Alerting**
```bash
# Install monitoring tools
sudo apt install -y prometheus grafana-server

# Create custom monitoring scripts
#!/bin/bash
THRESHOLD=80
USAGE=$(df / | awk 'NR==2 {print $5}' | cut -d'%' -f1)

if [ $USAGE -gt $THRESHOLD ]; then
    echo "Disk usage alert: $USAGE%"
    # Send alert
fi

# Schedule monitoring
crontab -e
# */5 * * * * /usr/local/bin/monitor.sh
```

### 4. **Backup Strategy**
```bash
# Automated daily backups
#!/bin/bash
BACKUP_DIR="/mnt/backups"
RETENTION_DAYS=30

# Create backup
tar -czvf "$BACKUP_DIR/backup-$(date +%Y%m%d-%H%M%S).tar.gz" /home /etc

# Remove old backups
find "$BACKUP_DIR" -type f -name "backup-*.tar.gz" -mtime +$RETENTION_DAYS -delete

# Verify backup
tar -tzf "$BACKUP_DIR/latest-backup.tar.gz" > /dev/null && echo "Backup OK"
```

### 5. **Configuration Management**
```bash
# Version control system configurations
git init /etc
cd /etc
git config user.email "admin@example.com"
git config user.name "System Admin"
git add .
git commit -m "Initial commit"

# Track configuration changes
git log --oneline
git diff HEAD~1 HEAD
```

### 6. **Documentation**
```bash
# Create runbooks for operations
cat > /docs/runbook-deploy.md << 'EOF'
# Deployment Runbook

## Pre-deployment checks
- [ ] Verify database backups
- [ ] Check disk space
- [ ] Review changelog

## Deployment steps
1. Pull latest code
2. Run tests
3. Deploy to staging
4. Deploy to production

## Post-deployment verification
- [ ] Health checks
- [ ] Log monitoring
- [ ] User testing
EOF
```

### 7. **Performance Tuning**
```bash
# Kernel tuning parameters
# /etc/sysctl.conf
net.core.somaxconn = 65536
net.ipv4.tcp_max_syn_backlog = 65536
net.ipv4.ip_local_port_range = 1024 65535

# Apply changes
sudo sysctl -p

# Check current values
sysctl net.core.somaxconn
cat /proc/sys/net/core/somaxconn
```

---

## Security

### Security Best Practices

#### 1. **User Access Control**
```bash
# Principle of Least Privilege
sudo usermod -G sudo username      # Add to sudo group
sudo visudo                        # Edit sudoers safely

# Example sudoers config:
# username ALL=(ALL) NOPASSWD: /usr/bin/systemctl

# Audit user accounts
awk -F: '$3 >= 1000' /etc/passwd   # Check regular users
lastlog                             # Last login
who                                 # Currently logged in

# Lock unused accounts
sudo usermod -L username            # Lock account
sudo usermod -U username            # Unlock account
```

#### 2. **File and Directory Permissions**
```bash
# Apply restrictive defaults
umask 0077                         # Files: 600, Dirs: 700

# Check for world-writable files
find / -perm -002 -type f -ls

# Secure sensitive files
chmod 600 ~/.ssh/id_rsa
chmod 600 /etc/shadow
chmod 644 /etc/passwd

# Use ACLs for granular permissions
setfacl -m u:username:rx /path
getfacl /path
```

#### 3. **SSH Security**
```bash
# Generate strong SSH keys
ssh-keygen -t ed25519 -C "user@hostname"
ssh-keygen -t rsa -b 4096 -C "user@hostname"

# Configure SSH daemon securely
# /etc/ssh/sshd_config
Port 2222
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
X11Forwarding no
MaxAuthTries 3
MaxSessions 10
ClientAliveInterval 300
ClientAliveCountMax 0

# Restart SSH
sudo systemctl restart ssh

# Test configuration
sudo sshd -t
```

#### 4. **Firewall Configuration**
```bash
# UFW (Ubuntu)
sudo ufw enable
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 22/tcp
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw status verbose

# firewalld (CentOS/RHEL)
sudo firewall-cmd --permanent --add-service=ssh
sudo firewall-cmd --permanent --add-port=80/tcp
sudo firewall-cmd --permanent --add-port=443/tcp
sudo firewall-cmd --reload
sudo firewall-cmd --list-all

# iptables (Advanced)
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT
sudo iptables -A INPUT -j DROP
```

#### 5. **SELinux and AppArmor**
```bash
# Check SELinux status (RHEL/CentOS)
getenforce
sestatus

# Set SELinux mode
sudo setenforce Enforcing
sudo setenforce Permissive

# Check AppArmor status (Ubuntu/Debian)
sudo apparmor_status

# Load AppArmor profile
sudo apparmor_parser -r /etc/apparmor.d/usr.bin.app
```

#### 6. **Audit Logging**
```bash
# Enable auditd
sudo apt install auditd
sudo systemctl start auditd
sudo systemctl enable auditd

# Add audit rules
sudo auditctl -a always,exit -F arch=b64 -S execve -k audit_exec
sudo auditctl -a always,exit -F arch=b64 -S open,unlink -F auid>=1000 -k audit_files

# Search audit logs
sudo ausearch -k audit_exec
sudo ausearch -i | grep "type=EXECVE"
```

#### 7. **Regular Security Updates**
```bash
# Check available updates
apt list --upgradable

# Automatic updates
sudo apt install unattended-upgrades
sudo dpkg-reconfigure -plow unattended-upgrades

# Review update logs
tail -f /var/log/unattended-upgrades/unattended-upgrades.log
```

#### 8. **Intrusion Detection**
```bash
# Install AIDE
sudo apt install aide aide-common
sudo aideinit

# Check file integrity
sudo aide --check

# Install fail2ban
sudo apt install fail2ban
sudo systemctl enable fail2ban
sudo fail2ban-client status

# Monitor fail2ban logs
tail -f /var/log/fail2ban.log
```

### Security Commands Reference

```bash
# Check open ports and services
sudo netstat -tlnp
sudo ss -tlnp

# Monitor network connections
sudo tcpdump -i eth0 -n
sudo tcpdump -i eth0 -n 'tcp port 22'

# Check installed packages for vulnerabilities
sudo apt list --upgradable

# Scan system for rootkits
sudo apt install rkhunter
sudo rkhunter --check --skip-keypress --report-warnings-only

# Generate SSH host keys
sudo ssh-keygen -A

# Verify package signatures
apt-key adv --keyserver keyserver.ubuntu.com --recv-keys KEY_ID
```

---

## Interview Questions

### Beginner Level

**1. What is Linux?**
- Answer: Linux is a free, open-source operating system kernel that serves as the core of various Linux distributions. It's widely used in servers, cloud infrastructure, and DevOps environments.

**2. What is the difference between Ubuntu and CentOS?**
- Answer:
  - Ubuntu: Debian-based, rolling releases, user-friendly, used apt
  - CentOS: RHEL-based, long support cycle, enterprise-focused, uses yum

**3. What is a Linux shell?**
- Answer: A shell is a command interpreter that translates user commands into kernel operations. Common shells include bash, sh, zsh, and ksh.

**4. Explain the Linux file system hierarchy.**
- Answer: Linux follows FHS (Filesystem Hierarchy Standard):
  - `/`: Root directory
  - `/bin`: Essential executables
  - `/home`: User home directories
  - `/etc`: Configuration files
  - `/var`: Variable data (logs, temp files)
  - `/usr`: User programs and libraries
  - `/tmp`: Temporary files
  - `/root`: Root user's home

**5. What are file permissions in Linux?**
- Answer: Three types: Read (r=4), Write (w=2), Execute (x=1) for Owner, Group, and Others. Example: 755 = rwxr-xr-x

**6. How do you change file permissions?**
- Answer: Using `chmod` command
  - `chmod 755 file.txt`
  - `chmod u+x script.sh`
  - `chmod -R 755 directory/`

**7. What is sudo and why is it important?**
- Answer: sudo (superuser do) allows authorized users to execute commands as root. It's important for security as it logs all sudo activities and restricts root access.

**8. What is a process in Linux?**
- Answer: A process is a running instance of a program with its own memory space, process ID (PID), parent process ID (PPID), and resources.

### Intermediate Level

**9. What is the difference between hard links and soft (symbolic) links?**
- Answer:
  - Hard link: Direct reference to inode, shares the same inode number
  - Soft link: Symbolic reference, creates a separate file pointing to original
  - Hard links: `ln file hardlink`
  - Soft links: `ln -s file softlink`

**10. Explain the boot process of Linux.**
- Answer:
  1. BIOS/UEFI initialization
  2. Bootloader (GRUB) loads kernel
  3. Kernel initialization
  4. Init process (systemd/init) starts
  5. Mount filesystems
  6. Start services and daemons
  7. Login prompt

**11. What is systemd and its advantages?**
- Answer: systemd is a system and service manager that replaced traditional init systems. Advantages:
  - Parallel service startup
  - Dependency management
  - Socket activation
  - Better logging with journalctl
  - Unit files for easier management

**12. How do you manage processes in Linux?**
- Answer:
  - `ps aux`: List processes
  - `top`/`htop`: Real-time monitoring
  - `kill PID`: Terminate process
  - `bg`/`fg`: Background/foreground jobs
  - `nice`/`renice`: Adjust priority

**13. What is the difference between 'apt' and 'yum'?**
- Answer:
  - apt: Advanced Packaging Tool (Debian/Ubuntu)
  - yum: Yellowdog Updater Modified (RHEL/CentOS)
  - Both manage package installation, but apt is newer and simpler

**14. Explain 'grep' and its common options.**
- Answer: grep searches for patterns in files
  - `grep "pattern" file`
  - `-i`: Case insensitive
  - `-r`: Recursive
  - `-n`: Show line numbers
  - `-c`: Count matches
  - `-E`: Extended regex

**15. What is the difference between 'chmod' and 'chown'?**
- Answer:
  - chmod: Change file permissions
  - chown: Change file ownership (owner and group)
  - Example: `chown user:group file.txt`

**16. How do you check disk usage in Linux?**
- Answer:
  - `df -h`: Filesystem disk space
  - `du -sh`: Directory size
  - `du -sh /*`: Size of top-level directories
  - `lsof +D /path`: Open files in directory

**17. What is iptables and how does it work?**
- Answer: iptables is a command-line utility for configuring Linux kernel firewall. It uses rules organized in chains (INPUT, OUTPUT, FORWARD) and tables (filter, nat, mangle).

**18. Explain the 'find' command and its usage.**
- Answer: find searches for files based on criteria
  - `find / -name "filename"`
  - `find /path -type f -size +1G`
  - `find /path -mtime -7` (modified in last 7 days)
  - `-exec`: Execute command on results

### Advanced Level

**19. What is a file descriptor?**
- Answer: A file descriptor is an integer representing an open file or I/O resource. Standard file descriptors:
  - 0: stdin
  - 1: stdout
  - 2: stderr
  - Can redirect: `command > output.txt 2>&1`

**20. Explain the difference between 'kill -9' and 'kill -15'.**
- Answer:
  - kill -15 (SIGTERM): Graceful termination, allows cleanup
  - kill -9 (SIGKILL): Force kill, no cleanup, cannot be caught
  - Always try SIGTERM first, use SIGKILL only if necessary

**21. What is the /proc filesystem?**
- Answer: Virtual filesystem providing information about kernel and processes:
  - `/proc/cpuinfo`: CPU information
  - `/proc/meminfo`: Memory information
  - `/proc/[pid]/`: Process-specific information
  - `/proc/net/`: Network information

**22. How do you perform system performance troubleshooting?**
- Answer:
  - Check load: `uptime`, `cat /proc/loadavg`
  - CPU: `top`, `mpstat`, `vmstat`
  - Memory: `free`, `/proc/meminfo`, `ps aux --sort=-%mem`
  - Disk: `df`, `du`, `iostat`, `iotop`
  - Network: `netstat`, `ss`, `iftop`
  - Logs: `journalctl`, `tail -f /var/log/*`

**23. What is a swap space and how do you configure it?**
- Answer: Swap is disk space used as virtual memory when RAM is full. Configuration:
  - Create swap file: `fallocate -l 2G /swapfile`
  - Format: `mkswap /swapfile`
  - Enable: `swapon /swapfile`
  - Check: `swapon --show`
  - Add to /etc/fstab for persistence

**24. Explain SSH tunneling and its use cases.**
- Answer: SSH tunneling creates secure tunnels through SSH connections:
  - Local forwarding: `ssh -L 8080:localhost:80 user@host`
  - Remote forwarding: `ssh -R 8080:localhost:80 user@host`
  - Dynamic forwarding (SOCKS): `ssh -D 1080 user@host`
  - Use cases: Secure remote access, bypass firewalls, secure database connections

**25. What is the 'ld.so.cache' and 'ldconfig'?**
- Answer:
  - ld.so.cache: Cache of shared library locations
  - ldconfig: Updates the cache and optimizes library loading
  - Used to manage dynamic library dependencies
  - `ldconfig -p`: Print current cache

**26. How do you implement automatic system updates?**
- Answer:
  - Ubuntu: `unattended-upgrades` package
  - Configure: `/etc/apt/apt.conf.d/50unattended-upgrades`
  - Enable: `dpkg-reconfigure -plow unattended-upgrades`
  - Monitor: `/var/log/unattended-upgrades/`

**27. Explain the difference between RAID levels.**
- Answer:
  - RAID 0: Striping (performance, no redundancy)
  - RAID 1: Mirroring (redundancy)
  - RAID 5: Striping with parity (balance)
  - RAID 6: Dual parity
  - RAID 10: Combination of 1 and 0

**28. What is LVM (Logical Volume Manager)?**
- Answer: LVM provides logical abstraction of storage:
  - Physical Volumes (PV): Physical disks
  - Volume Groups (VG): Combines PVs
  - Logical Volumes (LV): Virtual partitions
  - Benefits: Flexible resizing, snapshots, migration

**29. How do you troubleshoot SSH connection issues?**
- Answer:
  - Test connectivity: `ssh -vvv user@host`
  - Check SSH status: `systemctl status ssh`
  - Verify configuration: `sshd -t`
  - Check logs: `tail -f /var/log/auth.log`
  - Check permissions: SSH key should be 600, .ssh dir 700
  - Check firewall: `ufw status`, `firewall-cmd --list-all`

**30. Explain containers and their advantages in DevOps.**
- Answer: Containers (Docker) package applications with dependencies:
  - Lightweight and fast to start
  - Consistent across environments
  - Easy to scale
  - Isolation from host system
  - Supports orchestration (Kubernetes)
  - Works with CI/CD pipelines

---

## Conclusion

Linux commands and system administration are fundamental skills for DevOps engineers. Mastery of these commands enables efficient system management, troubleshooting, automation, and security implementation. Continuous practice and staying updated with new tools and techniques are essential for success in DevOps roles.

### Key Takeaways:
- **Master fundamentals:** File operations, user management, processes
- **Understand the system:** Know how Linux architecture and components work
- **Practice regularly:** Hands-on experience is crucial
- **Stay secure:** Always follow security best practices
- **Keep learning:** Technology evolves, so continuous learning is necessary
- **Document everything:** Good documentation helps team collaboration

---

**Last Updated:** 2025-12-29
**Version:** 1.0
