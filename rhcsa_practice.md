# Red Hat Certified System Administrator (RHCSA) EX200 Practice Questions

## Red Hat Enterprise Linux 9.0

> **Note**: These practice questions are designed to help you prepare for the RHCSA exam. Always test in a lab environment before implementing on production systems.

---

## Table of Contents

1. [System Configuration](#1-system-configuration)
2. [User and Group Management](#2-user-and-group-management)
3. [File Permissions and ACLs](#3-file-permissions-and-acls)
4. [Storage Management](#4-storage-management)
5. [Logical Volume Management (LVM)](#5-logical-volume-management-lvm)
6. [Networking](#6-networking)
7. [Service Management](#7-service-management)
8. [SELinux](#8-selinux)
9. [Task Scheduling](#9-task-scheduling)
10. [Container Management](#10-container-management)
11. [Advanced Topics](#11-advanced-topics)

---

## 1. System Configuration

### Question 1.1: Configure System Hostname and Timezone

Configure your system with the following parameters:
- Hostname: `workstation.lab.example.com`
- Timezone: `Asia/Dhaka`
- Verify that the hostname persists after reboot

<details>
<summary><b>Click to reveal solution</b></summary>

```bash
# Set hostname
[root@workstation ~]# hostnamectl set-hostname workstation.lab.example.com

# Verify hostname
[root@workstation ~]# hostnamectl status

# Set timezone
[root@workstation ~]# timedatectl set-timezone Asia/Dhaka

# Verify timezone
[root@workstation ~]# timedatectl status

# Verify persistence (optional)
[root@workstation ~]# cat /etc/hostname
workstation.lab.example.com
```

**Verification:**
```bash
[root@workstation ~]# hostnamectl
   Static hostname: workstation.lab.example.com
         Icon name: computer-vm
           Chassis: vm
        Machine ID: xxxxxxxxxxxxxxxxxxxxxxxxxxxxx
           Boot ID: xxxxxxxxxxxxxxxxxxxxxxxxxxxxx
    Virtualization: kvm
  Operating System: Red Hat Enterprise Linux 9.0
       CPE OS Name: cpe:/o:redhat:enterprise_linux:9.0:GA
            Kernel: Linux 5.14.0-70.el9.x86_64
      Architecture: x86-64
```

</details>

---

### Question 1.2: Configure YUM/DNF Repositories

Configure your system to use the following repositories:
- Repository ID: `BaseOS`
  - Name: `BaseOS Repository`
  - BaseURL: `http://repo.example.com/rhel9/BaseOS`
- Repository ID: `AppStream`
  - Name: `AppStream Repository`
  - BaseURL: `http://repo.example.com/rhel9/AppStream`

Both repositories should be enabled with GPG check disabled.

<details>
<summary><b>Click to reveal solution</b></summary>

```bash
# Create repository configuration file
[root@workstation ~]# vim /etc/yum.repos.d/local.repo

[BaseOS]
name=BaseOS Repository
baseurl=http://repo.example.com/rhel9/BaseOS
enabled=1
gpgcheck=0

[AppStream]
name=AppStream Repository
baseurl=http://repo.example.com/rhel9/AppStream
enabled=1
gpgcheck=0

# Clean and verify repositories
[root@workstation ~]# dnf clean all
[root@workstation ~]# dnf repolist

# Expected output:
repo id                    repo name
AppStream                  AppStream Repository
BaseOS                     BaseOS Repository
```

</details>

---

## 2. User and Group Management

### Question 2.1: Create Users with Specific Requirements

Create the following users with the specified requirements:

1. Create a group named `sysadmins` with GID `2000`
2. Create user `john` with UID `2001`, primary group `sysadmins`, password `redhat123`
3. Create user `jane` with UID `2002`, secondary group `sysadmins`, password `redhat123`
4. Create user `systemuser` with no home directory and no interactive shell
5. Set password aging for user `john`: minimum 7 days, maximum 90 days, warning 14 days before expiry

<details>
<summary><b>Click to reveal solution</b></summary>

```bash
# Create group with specific GID
[root@workstation ~]# groupadd -g 2000 sysadmins

# Create user john with primary group
[root@workstation ~]# useradd -u 2001 -g sysadmins john
[root@workstation ~]# echo "redhat123" | passwd --stdin john

# Create user jane with secondary group
[root@workstation ~]# useradd -u 2002 -G sysadmins jane
[root@workstation ~]# echo "redhat123" | passwd --stdin jane

# Create system user without home and shell
[root@workstation ~]# useradd -M -s /sbin/nologin systemuser

# Set password aging for john
[root@workstation ~]# chage -m 7 -M 90 -W 14 john

# Verify password aging
[root@workstation ~]# chage -l john
```

**Verification:**
```bash
[root@workstation ~]# id john
uid=2001(john) gid=2000(sysadmins) groups=2000(sysadmins)

[root@workstation ~]# id jane
uid=2002(jane) gid=2002(jane) groups=2002(jane),2000(sysadmins)

[root@workstation ~]# grep systemuser /etc/passwd
systemuser:x:1001:1001::/home/systemuser:/sbin/nologin
```

</details>

---

### Question 2.2: Manage User Account Expiration

Set the account expiration date for user `jane` to December 31, 2025. After this date, the account should be locked.

<details>
<summary><b>Click to reveal solution</b></summary>

```bash
# Set account expiration date
[root@workstation ~]# chage -E 2025-12-31 jane

# Verify expiration
[root@workstation ~]# chage -l jane

# Alternative method using usermod
[root@workstation ~]# usermod -e 2025-12-31 jane

# Check in /etc/shadow
[root@workstation ~]# grep jane /etc/shadow
```

</details>

---

## 3. File Permissions and ACLs

### Question 3.1: Create Collaborative Directory with Special Permissions

Create a shared directory `/shared/projects` with the following requirements:
- Group ownership: `sysadmins`
- Permissions: Members of `sysadmins` can read, write, and access the directory
- Other users have no permissions
- Files created in this directory automatically inherit the group `sysadmins`
- Prevent users from deleting files they don't own

<details>
<summary><b>Click to reveal solution</b></summary>

```bash
# Create directory
[root@workstation ~]# mkdir -p /shared/projects

# Set group ownership
[root@workstation ~]# chgrp sysadmins /shared/projects

# Set permissions with SGID and sticky bit
[root@workstation ~]# chmod 2770 /shared/projects

# Add sticky bit to prevent deletion
[root@workstation ~]# chmod 3770 /shared/projects

# Verify permissions
[root@workstation ~]# ls -ld /shared/projects
drwxrws--T. 2 root sysadmins 6 Dec 11 10:00 /shared/projects
```

**Explanation:**
- `3770`: 3 = sticky bit + SGID, 770 = rwx for owner and group
- SGID (2): Files created inherit the directory's group
- Sticky bit (1): Only file owners can delete their own files

**Testing:**
```bash
[root@workstation ~]# su - john
[john@workstation ~]$ cd /shared/projects
[john@workstation projects]$ touch testfile
[john@workstation projects]$ ls -l testfile
-rw-r--r--. 1 john sysadmins 0 Dec 11 10:00 testfile
```

</details>

---

### Question 3.2: Configure ACLs for Fine-Grained Permissions

Configure Access Control Lists (ACLs) on directory `/data/reports`:
- User `john` should have read and execute permissions
- User `jane` should have read, write, and execute permissions
- Group `sysadmins` should have read and execute permissions
- Default ACLs: New files should grant `jane` read and write permissions automatically

<details>
<summary><b>Click to reveal solution</b></summary>

```bash
# Create directory
[root@workstation ~]# mkdir -p /data/reports

# Set ACLs for users
[root@workstation ~]# setfacl -m u:john:rx /data/reports
[root@workstation ~]# setfacl -m u:jane:rwx /data/reports
[root@workstation ~]# setfacl -m g:sysadmins:rx /data/reports

# Set default ACLs for new files
[root@workstation ~]# setfacl -d -m u:jane:rw /data/reports

# Verify ACLs
[root@workstation ~]# getfacl /data/reports

# Output:
# file: data/reports
# owner: root
# group: root
user::rwx
user:john:r-x
user:jane:rwx
group::r-x
group:sysadmins:r-x
mask::rwx
other::---
default:user::rwx
default:user:jane:rw-
default:group::r-x
default:mask::rwx
default:other::---
```

**Testing:**
```bash
[root@workstation ~]# su - jane
[jane@workstation ~]$ cd /data/reports
[jane@workstation reports]$ touch newfile
[jane@workstation reports]$ getfacl newfile
```

</details>

---

## 4. Storage Management

### Question 4.1: Create and Mount Partitions

Using the disk `/dev/vdb`, create the following partitions:
- Partition 1: 500 MiB, formatted as ext4, mounted at `/mnt/data1`
- Partition 2: 1 GiB, formatted as xfs, mounted at `/mnt/data2`
- Both partitions should mount automatically at boot with default options

<details>
<summary><b>Click to reveal solution</b></summary>

```bash
# View current disk layout
[root@workstation ~]# lsblk
[root@workstation ~]# fdisk -l /dev/vdb

# Create partitions using parted
[root@workstation ~]# parted /dev/vdb mklabel gpt
[root@workstation ~]# parted /dev/vdb mkpart primary ext4 1MiB 501MiB
[root@workstation ~]# parted /dev/vdb mkpart primary xfs 501MiB 1525MiB

# Inform kernel of partition changes
[root@workstation ~]# udevadm settle

# Format partitions
[root@workstation ~]# mkfs.ext4 /dev/vdb1
[root@workstation ~]# mkfs.xfs /dev/vdb2

# Create mount points
[root@workstation ~]# mkdir -p /mnt/data1 /mnt/data2

# Get UUIDs
[root@workstation ~]# blkid /dev/vdb1
[root@workstation ~]# blkid /dev/vdb2

# Add to /etc/fstab
[root@workstation ~]# vim /etc/fstab

UUID=xxxx-xxxx-xxxx-xxxx /mnt/data1 ext4 defaults 0 0
UUID=xxxx-xxxx-xxxx-xxxx /mnt/data2 xfs defaults 0 0

# Mount all filesystems
[root@workstation ~]# mount -a

# Verify
[root@workstation ~]# df -h | grep /mnt
[root@workstation ~]# lsblk
```

**Alternative using device names:**
```bash
# Add to /etc/fstab
/dev/vdb1 /mnt/data1 ext4 defaults 0 0
/dev/vdb2 /mnt/data2 xfs defaults 0 0
```

</details>

---

### Question 4.2: Add Swap Space

Add a swap partition of 1 GiB to your system using `/dev/vdb3`. The swap should be activated automatically at boot. Do not remove any existing swap.

<details>
<summary><b>Click to reveal solution</b></summary>

```bash
# Check current swap
[root@workstation ~]# free -h
[root@workstation ~]# swapon --show

# Create partition
[root@workstation ~]# parted /dev/vdb mkpart primary linux-swap 1525MiB 2549MiB
[root@workstation ~]# udevadm settle

# Create swap filesystem
[root@workstation ~]# mkswap /dev/vdb3

# Get UUID
[root@workstation ~]# blkid /dev/vdb3

# Add to /etc/fstab
[root@workstation ~]# vim /etc/fstab
UUID=xxxx-xxxx-xxxx-xxxx swap swap defaults 0 0

# Activate swap
[root@workstation ~]# swapon -a

# Verify
[root@workstation ~]# swapon --show
[root@workstation ~]# free -h
```

</details>

---

## 5. Logical Volume Management (LVM)

### Question 5.1: Create LVM with Specific Requirements

Create a logical volume with the following specifications:
- Physical volume: `/dev/vdc1` (1 GiB partition)
- Volume group name: `vgdata` with PE size of 8 MiB
- Logical volume name: `lvapp` with size of 50 extents
- Format with ext4 filesystem
- Mount at `/mnt/app` persistently

<details>
<summary><b>Click to reveal solution</b></summary>

```bash
# Create partition
[root@workstation ~]# parted /dev/vdc mklabel gpt
[root@workstation ~]# parted /dev/vdc mkpart primary 1MiB 1025MiB
[root@workstation ~]# parted /dev/vdc set 1 lvm on
[root@workstation ~]# udevadm settle

# Create physical volume
[root@workstation ~]# pvcreate /dev/vdc1
[root@workstation ~]# pvdisplay

# Create volume group with 8 MiB PE size
[root@workstation ~]# vgcreate -s 8M vgdata /dev/vdc1
[root@workstation ~]# vgdisplay vgdata

# Calculate size: 50 extents × 8 MiB = 400 MiB
[root@workstation ~]# lvcreate -l 50 -n lvapp vgdata
[root@workstation ~]# lvdisplay /dev/vgdata/lvapp

# Format with ext4
[root@workstation ~]# mkfs.ext4 /dev/vgdata/lvapp

# Create mount point and mount
[root@workstation ~]# mkdir /mnt/app
[root@workstation ~]# blkid /dev/vgdata/lvapp

# Add to /etc/fstab
[root@workstation ~]# vim /etc/fstab
/dev/vgdata/lvapp /mnt/app ext4 defaults 0 0

# Mount and verify
[root@workstation ~]# mount -a
[root@workstation ~]# df -h /mnt/app
```

</details>

---

### Question 5.2: Extend Logical Volume

Extend the logical volume `/dev/vgdata/lvapp` to 600 MiB. The filesystem must be resized as well, and all existing data must remain intact.

<details>
<summary><b>Click to reveal solution</b></summary>

```bash
# Check current size
[root@workstation ~]# lvdisplay /dev/vgdata/lvapp
[root@workstation ~]# df -h /mnt/app

# Check available space in VG
[root@workstation ~]# vgdisplay vgdata

# Extend LV and resize filesystem in one command
[root@workstation ~]# lvextend -r -L 600M /dev/vgdata/lvapp

# Alternatively, extend then resize separately
[root@workstation ~]# lvextend -L 600M /dev/vgdata/lvapp
[root@workstation ~]# resize2fs /dev/vgdata/lvapp

# Verify
[root@workstation ~]# lvdisplay /dev/vgdata/lvapp
[root@workstation ~]# df -h /mnt/app

# Check filesystem integrity
[root@workstation ~]# ls -la /mnt/app
```

</details>

---

### Question 5.3: Create VDO Volume

Create a VDO (Virtual Data Optimizer) volume with the following specifications:
- Name: `vdo1`
- Device: `/dev/vdd`
- Logical size: 50 GiB
- Mount point: `/mnt/vdo`
- Filesystem: xfs
- Enable compression and deduplication

<details>
<summary><b>Click to reveal solution</b></summary>

```bash
# Install VDO
[root@workstation ~]# dnf install vdo kmod-kvdo -y

# Create VDO volume
[root@workstation ~]# vdo create --name=vdo1 --device=/dev/vdd --vdoLogicalSize=50G

# Check VDO status
[root@workstation ~]# vdo status --name=vdo1

# Format with XFS
[root@workstation ~]# mkfs.xfs -K /dev/mapper/vdo1

# Create mount point
[root@workstation ~]# mkdir /mnt/vdo

# Get device path for fstab
[root@workstation ~]# blkid /dev/mapper/vdo1

# Add to /etc/fstab
[root@workstation ~]# vim /etc/fstab
/dev/mapper/vdo1 /mnt/vdo xfs defaults,x-systemd.requires=vdo.service 0 0

# Mount and verify
[root@workstation ~]# mount -a
[root@workstation ~]# df -h /mnt/vdo
[root@workstation ~]# vdostats --human-readable
```

</details>

---

## 6. Networking

### Question 6.1: Configure Static Network Connection

Configure the network interface with static IP settings:
- Connection name: `static-conn`
- Interface: `enp1s0`
- IP address: `192.168.100.50/24`
- Gateway: `192.168.100.1`
- DNS servers: `8.8.8.8`, `8.8.4.4`
- Search domain: `example.com`

<details>
<summary><b>Click to reveal solution</b></summary>

```bash
# View current connections
[root@workstation ~]# nmcli connection show

# Create new connection or modify existing
[root@workstation ~]# nmcli connection add \
  con-name static-conn \
  type ethernet \
  ifname enp1s0 \
  ipv4.addresses 192.168.100.50/24 \
  ipv4.gateway 192.168.100.1 \
  ipv4.dns "8.8.8.8 8.8.4.4" \
  ipv4.dns-search example.com \
  ipv4.method manual

# Activate connection
[root@workstation ~]# nmcli connection up static-conn

# Verify configuration
[root@workstation ~]# nmcli connection show static-conn
[root@workstation ~]# ip addr show enp1s0
[root@workstation ~]# ip route
[root@workstation ~]# cat /etc/resolv.conf

# Test connectivity
[root@workstation ~]# ping -c 3 192.168.100.1
[root@workstation ~]# ping -c 3 google.com
```

**Alternative method to modify existing connection:**
```bash
[root@workstation ~]# nmcli connection modify enp1s0 \
  ipv4.addresses 192.168.100.50/24 \
  ipv4.gateway 192.168.100.1 \
  ipv4.dns "8.8.8.8 8.8.4.4" \
  ipv4.method manual

[root@workstation ~]# nmcli connection down enp1s0
[root@workstation ~]# nmcli connection up enp1s0
```

</details>

---

### Question 6.2: Configure Firewall Rules

Configure the firewall with the following requirements:
- Allow HTTP (port 80) and HTTPS (port 443) traffic
- Allow SSH on port 2222 instead of default port 22
- Block all incoming traffic from IP `192.168.50.100`
- Add custom port 8080/tcp for web application
- Make all changes permanent

<details>
<summary><b>Click to reveal solution</b></summary>

```bash
# Check firewall status
[root@workstation ~]# firewall-cmd --state
[root@workstation ~]# firewall-cmd --list-all

# Allow HTTP and HTTPS
[root@workstation ~]# firewall-cmd --permanent --add-service=http
[root@workstation ~]# firewall-cmd --permanent --add-service=https

# Remove default SSH and add custom SSH port
[root@workstation ~]# firewall-cmd --permanent --remove-service=ssh
[root@workstation ~]# firewall-cmd --permanent --add-port=2222/tcp

# Add custom port for web application
[root@workstation ~]# firewall-cmd --permanent --add-port=8080/tcp

# Block specific IP address
[root@workstation ~]# firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="192.168.50.100" reject'

# Reload firewall to apply changes
[root@workstation ~]# firewall-cmd --reload

# Verify all rules
[root@workstation ~]# firewall-cmd --list-all

# Test port accessibility (from another system)
# telnet workstation.lab.example.com 8080
```

</details>

---

## 7. Service Management

### Question 7.1: Configure and Manage Services

Perform the following service management tasks:
- Install and configure Apache web server (httpd)
- Enable httpd to start automatically at boot
- Configure httpd to listen on port 8080
- Create a simple index page at `/var/www/html/index.html` with content "Welcome to RHCSA Lab"
- Ensure the service is running and accessible

<details>
<summary><b>Click to reveal solution</b></summary>

```bash
# Install Apache
[root@workstation ~]# dnf install httpd -y

# Configure Apache to listen on port 8080
[root@workstation ~]# vim /etc/httpd/conf/httpd.conf
# Change the line: Listen 80
# To: Listen 8080

# Create index page
[root@workstation ~]# echo "Welcome to RHCSA Lab" > /var/www/html/index.html

# Enable and start httpd
[root@workstation ~]# systemctl enable httpd --now

# Verify service status
[root@workstation ~]# systemctl status httpd
[root@workstation ~]# systemctl is-enabled httpd

# Configure firewall
[root@workstation ~]# firewall-cmd --permanent --add-port=8080/tcp
[root@workstation ~]# firewall-cmd --reload

# Test locally
[root@workstation ~]# curl http://localhost:8080

# Check if service starts at boot
[root@workstation ~]# systemctl list-unit-files | grep httpd
```

</details>

---

### Question 7.2: Configure Time Synchronization

Configure your system as an NTP client:
- Use `time.example.com` as the NTP server
- Ensure chronyd service is enabled and running
- Verify time synchronization is working

<details>
<summary><b>Click to reveal solution</b></summary>

```bash
# Install chrony (if not installed)
[root@workstation ~]# dnf install chrony -y

# Edit chrony configuration
[root@workstation ~]# vim /etc/chrony.conf

# Comment out existing server lines and add:
# pool 2.rhel.pool.ntp.org iburst
server time.example.com iburst

# Enable and restart chronyd
[root@workstation ~]# systemctl enable chronyd --now
[root@workstation ~]# systemctl restart chronyd

# Verify service status
[root@workstation ~]# systemctl status chronyd

# Check NTP sources
[root@workstation ~]# chronyc sources -v

# Check time synchronization status
[root@workstation ~]# chronyc tracking

# Verify system time
[root@workstation ~]# timedatectl status
```

**Expected output from `chronyc sources -v`:**
```
^* time.example.com    2   6    377    12   +0.123ms[+0.145ms] +/-   15ms
```
The `^*` indicates the currently selected source.

</details>

---

## 8. SELinux

### Question 8.1: Troubleshoot SELinux for Web Server

You have a web server running on port 8888, but it fails to start. The configuration file is `/etc/httpd/conf/httpd.conf` and document root is `/web/content`. Fix all SELinux issues to make the web server accessible.

<details>
<summary><b>Click to reveal solution</b></summary>

```bash
# Check SELinux status
[root@workstation ~]# getenforce
[root@workstation ~]# sestatus

# Ensure SELinux is in enforcing mode
[root@workstation ~]# setenforce 1
[root@workstation ~]# vim /etc/selinux/config
SELINUX=enforcing

# Try to start httpd and check for errors
[root@workstation ~]# systemctl start httpd
[root@workstation ~]# systemctl status httpd

# Check SELinux denials
[root@workstation ~]# ausearch -m avc -ts recent
[root@workstation ~]# journalctl -xe

# Issue 1: Allow httpd to bind to port 8888
[root@workstation ~]# semanage port -l | grep http_port_t
[root@workstation ~]# semanage port -a -t http_port_t -p tcp 8888

# Issue 2: Set correct SELinux context for document root
[root@workstation ~]# mkdir -p /web/content
[root@workstation ~]# ls -Zd /web/content

# Set correct context
[root@workstation ~]# semanage fcontext -a -t httpd_sys_content_t "/web/content(/.*)?"
[root@workstation ~]# restorecon -Rv /web/content

# Verify context
[root@workstation ~]# ls -Zd /web/content

# Configure firewall
[root@workstation ~]# firewall-cmd --permanent --add-port=8888/tcp
[root@workstation ~]# firewall-cmd --reload

# Restart httpd
[root@workstation ~]# systemctl restart httpd
[root@workstation ~]# systemctl status httpd

# Test access
[root@workstation ~]# curl http://localhost:8888
```

**Useful SELinux commands:**
```bash
# View SELinux booleans for httpd
[root@workstation ~]# getsebool -a | grep httpd

# Enable boolean if needed (example)
[root@workstation ~]# setsebool -P httpd_can_network_connect on

# Check file contexts
[root@workstation ~]# matchpathcon /web/content
```

</details>

---

### Question 8.2: Configure SELinux File Contexts

Configure SELinux contexts so that user `jane` can host web content from her home directory at `/home/jane/public_html`:
- Create the directory structure
- Set appropriate SELinux contexts
- Enable necessary SELinux booleans
- Test by placing a test HTML file

<details>
<summary><b>Click to reveal solution</b></summary>

```bash
# Create directory structure as jane
[root@workstation ~]# su - jane
[jane@workstation ~]$ mkdir -p ~/public_html
[jane@workstation ~]$ echo "Jane's Web Page" > ~/public_html/index.html
[jane@workstation ~]$ exit

# Set file permissions
[root@workstation ~]# chmod 755 /home/jane
[root@workstation ~]# chmod 755 /home/jane/public_html
[root@workstation ~]# chmod 644 /home/jane/public_html/index.html

# Check current SELinux context
[root@workstation ~]# ls -Zd /home/jane/public_html

# Set SELinux context for user web content
[root@workstation ~]# semanage fcontext -a -t httpd_user_content_t "/home/jane/public_html(/.*)?"
[root@workstation ~]# restorecon -Rv /home/jane/public_html

# Verify context
[root@workstation ~]# ls -Z /home/jane/public_html/index.html

# Enable SELinux boolean for user directories
[root@workstation ~]# setsebool -P httpd_enable_homedirs on

# Verify boolean
[root@workstation ~]# getsebool httpd_enable_homedirs

# Configure Apache for user directories (if needed)
[root@workstation ~]# vim /etc/httpd/conf.d/userdir.conf
# Uncomment: UserDir public_html

# Restart httpd
[root@workstation ~]# systemctl restart httpd

# Test access
[root@workstation ~]# curl http://localhost/~jane/
```

</details>

---

## 9. Task Scheduling

### Question 9.1: Configure Cron Jobs

Create the following scheduled tasks:
- User `john` should run a backup script `/usr/local/bin/backup.sh` every day at 3:30 AM
- User `jane` should run a cleanup script `/home/jane/cleanup.sh` every Monday at 6:00 PM
- System-wide task: Clear `/tmp` directory every hour
- Create a job that runs every 5 minutes during business hours (9 AM - 5 PM, Monday-Friday)

<details>
<summary><b>Click to reveal solution</b></summary>

```bash
# Task 1: Create backup script and cron job for john
[root@workstation ~]# vim /usr/local/bin/backup.sh
#!/bin/bash
tar -czf /backups/backup-$(date +\%Y\%m\%d).tar.gz /home/john
# Make executable
[root@workstation ~]# chmod +x /usr/local/bin/backup.sh

# Create cron job for john
[root@workstation ~]# crontab -u john -e
30 3 * * * /usr/local/bin/backup.sh

# Verify john's crontab
[root@workstation ~]# crontab -u john -l

# Task 2: Create cron job for jane
[root@workstation ~]# crontab -u jane -e
0 18 * * 1 /home/jane/cleanup.sh

# Task 3: System-wide task to clear /tmp
[root@workstation ~]# vim /etc/cron.d/clear-tmp
0 * * * * root find /tmp -type f -mtime +1 -delete

# Task 4: Every 5 minutes during business hours
[root@workstation ~]# crontab -e
*/5 9-17 * * 1-5 /usr/local/bin/business-task.sh

# Verify all cron jobs
[root@workstation ~]# crontab -l
[root@workstation ~]# crontab -u john -l
[root@workstation ~]# crontab -u jane -l
[root@workstation ~]# cat /etc/cron.d/clear-tmp
```

**Cron format reminder:**
```
* * * * * command
│ │ │ │ │
│ │ │ │ └─── Day of week (0-7, Sunday=0 or 7)
│ │ │ └──────── Month (1-12)
│ │ └───────────── Day of month (1-31)
│ └────────────────── Hour (0-23)
└─────────────────────── Minute (0-59)
```

</details>

---

### Question 9.2: Configure systemd Timer

Create a systemd timer that runs a disk usage report every Sunday at 11:00 PM:
- Service name: `disk-report.service`
- Timer name: `disk-report.timer`
- Script: `/usr/local/bin/disk-report.sh` (reports disk usage to `/var/log/disk-report.log`)

<details>
<summary><b>Click to reveal solution</b></summary>

```bash
# Create the script
[root@workstation ~]# vim /usr/local/bin/disk-report.sh
#!/bin/bash
echo "=== Disk Usage Report - $(date) ===" >> /var/log/disk-report.log
df -h >> /var/log/disk-report.log
echo "" >> /var/log/disk-report.log

[root@workstation ~]# chmod +x /usr/local/bin/disk-report.sh

# Create the service unit
[root@workstation ~]# vim /etc/systemd/system/disk-report.service
[Unit]
Description=Disk Usage Report Service

[Service]
Type=oneshot
ExecStart=/usr/local/bin/disk-report.sh

# Create the timer unit
[root@workstation ~]# vim /etc/systemd/system/disk-report.timer
[Unit]
Description=Run Disk Usage Report every Sunday at 11 PM

[Timer]
OnCalendar=Sun *-*-* 23:00:00
Persistent=true

[Install]
WantedBy=timers.target

# Reload systemd daemon
[root@workstation ~]# systemctl daemon-reload

# Enable and start the timer
[root@workstation ~]# systemctl enable disk-report.timer
[root@workstation ~]# systemctl start disk-report.timer

# Verify timer status
[root@workstation ~]# systemctl status disk-report.timer
[root@workstation ~]# systemctl list-timers disk-report.timer

# Test the service manually
[root@workstation ~]# systemctl start disk-report.service
[root@workstation ~]# cat /var/log/disk-report.log
```

</details>

---

## 10. Container Management

### Question 10.1: Run Rootless Container with Persistent Storage

As user `student`, create a rootless container with the following requirements:
- Image: `registry.access.redhat.com/ubi9/ubi:latest`
- Container name: `webapp`
- Port mapping: Host port 8080 to container port 80
- Volume mount: `/home/student/webdata` to `/var/www/html` in container
- Container should start automatically at boot
- Create systemd service for the container

<details>
<summary><b>Click to reveal solution</b></summary>

```bash
# Switch to student user
[root@workstation ~]# su - student

# Login to registry (if required)
[student@workstation ~]$ podman login registry.access.redhat.com

# Create directory for persistent data
[student@workstation ~]$ mkdir -p ~/webdata
[student@workstation ~]$ echo "Container Web Page" > ~/webdata/index.html

# Run the container
[student@workstation ~]$ podman run -d \
  --name webapp \
  -p 8080:80 \
  -v ~/webdata:/var/www/html:Z \
  registry.access.redhat.com/ubi9/ubi:latest \
  sleep infinity

# Verify container is running
[student@workstation ~]$ podman ps

# Create systemd service directory
[student@workstation ~]$ mkdir -p ~/.config/systemd/user

# Generate systemd service
[student@workstation ~]$ cd ~/.config/systemd/user
[student@workstation user]$ podman generate systemd --name webapp --files --new

# Stop and remove the original container
[student@workstation user]$ podman stop webapp
[student@workstation user]$ podman rm webapp

# Reload systemd user daemon
[student@workstation user]$ systemctl --user daemon-reload

# Enable and start the service
[student@workstation user]$ systemctl --user enable container-webapp.service
[student@workstation user]$ systemctl --user start container-webapp.service

# Enable lingering for user (start services at boot without login)
[student@workstation user]$ loginctl enable-linger

# Verify service status
[student@workstation user]$ systemctl --user status container-webapp.service
[student@workstation user]$ podman ps

# Test the container
[student@workstation user]$ curl http://localhost:8080
```

**Configure firewall (as root):**
```bash
[root@workstation ~]# firewall-cmd --permanent --add-port=8080/tcp
[root@workstation ~]# firewall-cmd --reload
```

</details>

---

### Question 10.2: Build Custom Container Image

Build a custom container image with the following specifications:
- Base image: `registry.access.redhat.com/ubi9/ubi:latest`
- Install `httpd` package
- Copy custom `index.html` to `/var/www/html/`
- Expose port 80
- Image name: `custom-web:v1`
- Tag and push to local registry at `localhost:5000`

<details>
<summary><b>Click to reveal solution</b></summary>

```bash
# Create working directory
[student@workstation ~]$ mkdir ~/containerfiles
[student@workstation ~]$ cd ~/containerfiles

# Create custom index.html
[student@workstation containerfiles]$ cat > index.html << EOF
<!DOCTYPE html>
<html>
<head><title>Custom Web Server</title></head>
<body>
<h1>Welcome to Custom Container</h1>
<p>RHCSA Exam Practice</p>
</body>
</html>
EOF

# Create Containerfile (Dockerfile)
[student@workstation containerfiles]$ vim Containerfile
FROM registry.access.redhat.com/ubi9/ubi:latest

# Install httpd
RUN dnf install -y httpd && \
    dnf clean all

# Copy custom index page
COPY index.html /var/www/html/

# Expose port
EXPOSE 80

# Start httpd in foreground
CMD ["/usr/sbin/httpd", "-D", "FOREGROUND"]

# Build the image
[student@workstation containerfiles]$ podman build -t custom-web:v1 .

# Verify image
[student@workstation containerfiles]$ podman images

# Test the image
[student@workstation containerfiles]$ podman run -d --name test-web -p 8081:80 custom-web:v1
[student@workstation containerfiles]$ curl http://localhost:8081
[student@workstation containerfiles]$ podman stop test-web
[student@workstation containerfiles]$ podman rm test-web

# Tag for local registry
[student@workstation containerfiles]$ podman tag custom-web:v1 localhost:5000/custom-web:v1

# Push to local registry (assuming registry is running)
[student@workstation containerfiles]$ podman push localhost:5000/custom-web:v1

# Verify pushed image
[student@workstation containerfiles]$ podman search localhost:5000/custom-web
```

</details>

---

## 11. Advanced Topics

### Question 11.1: Configure AutoFS for NFS Mounts

Configure AutoFS to automatically mount NFS shares:
- NFS server: `nfs.example.com`
- Export: `/exports/data`
- Local mount point: `/nfsdata`
- User `john` should be able to access the mount automatically
- The mount should unmount after 60 seconds of inactivity

<details>
<summary><b>Click to reveal solution</b></summary>

```bash
# Install required packages
[root@workstation ~]# dnf install autofs nfs-utils -y

# Create master map file
[root@workstation ~]# vim /etc/auto.master.d/nfs.autofs
/nfsdata /etc/auto.nfs --timeout=60

# Create map file
[root@workstation ~]# vim /etc/auto.nfs
data -rw,sync nfs.example.com:/exports/data

# Enable and start autofs
[root@workstation ~]# systemctl enable autofs --now
[root@workstation ~]# systemctl status autofs

# Verify configuration
[root@workstation ~]# cat /etc/auto.master.d/nfs.autofs
[root@workstation ~]# cat /etc/auto.nfs

# Test the automount
[root@workstation ~]# su - john
[john@workstation ~]$ cd /nfsdata/data
[john@workstation data]$ ls -la
[john@workstation data]$ df -h | grep nfsdata

# Wait 60 seconds and check if unmounted
[john@workstation ~]$ sleep 70
[john@workstation ~]$ df -h | grep nfsdata
# Should not be mounted

# Access again to trigger remount
[john@workstation ~]$ ls /nfsdata/data
[john@workstation ~]$ df -h | grep nfsdata
# Should be mounted again
```

</details>

---

### Question 11.2: Configure Stratis Storage

Create a Stratis pool and filesystem with the following specifications:
- Pool name: `pool1`
- Block device: `/dev/vde`
- Filesystem name: `fs1`
- Mount point: `/stratis/fs1`
- Filesystem should mount automatically at boot

<details>
<summary><b>Click to reveal solution</b></summary>

```bash
# Install Stratis packages
[root@workstation ~]# dnf install stratis-cli stratisd -y

# Enable and start stratisd
[root@workstation ~]# systemctl enable stratisd --now
[root@workstation ~]# systemctl status stratisd

# Create Stratis pool
[root@workstation ~]# stratis pool create pool1 /dev/vde

# Verify pool creation
[root@workstation ~]# stratis pool list

# Create filesystem
[root@workstation ~]# stratis filesystem create pool1 fs1

# Verify filesystem
[root@workstation ~]# stratis filesystem list

# Get filesystem UUID for fstab
[root@workstation ~]# blkid | grep stratis

# Create mount point
[root@workstation ~]# mkdir -p /stratis/fs1

# Add to /etc/fstab
[root@workstation ~]# vim /etc/fstab
UUID=xxxx-xxxx-xxxx-xxxx /stratis/fs1 xfs defaults,x-systemd.requires=stratisd.service 0 0

# Or use device path:
/dev/stratis/pool1/fs1 /stratis/fs1 xfs defaults,x-systemd.requires=stratisd.service 0 0

# Mount filesystem
[root@workstation ~]# mount -a

# Verify mount
[root@workstation ~]# df -h /stratis/fs1
[root@workstation ~]# stratis filesystem list

# Test by creating files
[root@workstation ~]# dd if=/dev/zero of=/stratis/fs1/testfile bs=1M count=100
[root@workstation ~]# ls -lh /stratis/fs1/
```

</details>

---

### Question 11.3: Create and Configure a Bash Script with Arguments

Create a script `/usr/local/bin/userinfo.sh` that:
- Accepts username as an argument
- Displays user's UID, GID, home directory, and shell
- Shows all groups the user belongs to
- If no argument provided, displays usage message
- Exits with appropriate error codes

<details>
<summary><b>Click to reveal solution</b></summary>

```bash
# Create the script
[root@workstation ~]# vim /usr/local/bin/userinfo.sh
#!/bin/bash

# Check if argument is provided
if [ $# -eq 0 ]; then
    echo "Usage: $0 <username>"
    echo "Example: $0 john"
    exit 1
fi

USERNAME=$1

# Check if user exists
if ! id "$USERNAME" &>/dev/null; then
    echo "Error: User '$USERNAME' does not exist"
    exit 2
fi

# Display user information
echo "=== User Information for: $USERNAME ==="
echo ""

# Get UID
UID=$(id -u "$USERNAME")
echo "UID: $UID"

# Get GID
GID=$(id -g "$USERNAME")
echo "GID: $GID"

# Get home directory
HOME_DIR=$(getent passwd "$USERNAME" | cut -d: -f6)
echo "Home Directory: $HOME_DIR"

# Get shell
SHELL=$(getent passwd "$USERNAME" | cut -d: -f7)
echo "Shell: $SHELL"

# Get all groups
echo ""
echo "Groups:"
id -Gn "$USERNAME" | tr ' ' '\n' | sed 's/^/  - /'

echo ""
echo "=== End of Information ==="

exit 0

# Make script executable
[root@workstation ~]# chmod +x /usr/local/bin/userinfo.sh

# Test the script
[root@workstation ~]# userinfo.sh
# Should show usage message

[root@workstation ~]# userinfo.sh john
# Should display john's information

[root@workstation ~]# userinfo.sh nonexistentuser
# Should show error message

# Test exit codes
[root@workstation ~]# userinfo.sh
[root@workstation ~]# echo $?
1

[root@workstation ~]# userinfo.sh john
[root@workstation ~]# echo $?
0

[root@workstation ~]# userinfo.sh baduser
[root@workstation ~]# echo $?
2
```

</details>

---

### Question 11.4: Configure Password Policy

Implement the following password policy for all new users:
- Minimum password length: 12 characters
- Password must contain at least 1 digit
- Password must contain at least 1 uppercase letter
- Password must contain at least 1 special character
- Minimum 2 character classes
- Password expires after 60 days
- Warning 7 days before expiration

<details>
<summary><b>Click to reveal solution</b></summary>

```bash
# Configure password quality requirements
[root@workstation ~]# vim /etc/security/pwquality.conf

# Uncomment and set the following:
minlen = 12
dcredit = -1
ucredit = -1
ocredit = -1
minclass = 2

# Configure password aging defaults
[root@workstation ~]# vim /etc/login.defs

# Set the following values:
PASS_MAX_DAYS   60
PASS_MIN_DAYS   0
PASS_MIN_LEN    12
PASS_WARN_AGE   7

# Verify configuration
[root@workstation ~]# grep -E "^PASS" /etc/login.defs

# Test with a new user
[root@workstation ~]# useradd testuser

# Try weak password (should fail)
[root@workstation ~]# passwd testuser
# Try: simple123
# Should fail with quality check error

# Try strong password (should succeed)
[root@workstation ~]# passwd testuser
# Try: ComplexP@ss123
# Should succeed

# Verify password aging for the new user
[root@workstation ~]# chage -l testuser

# Apply password policy to existing users (optional)
[root@workstation ~]# chage -M 60 -W 7 john
[root@workstation ~]# chage -l john

# Force password change at next login
[root@workstation ~]# chage -d 0 testuser
```

**Test password requirements:**
```bash
# Install password quality testing tool
[root@workstation ~]# dnf install libpwquality-tools -y

# Test password strength
[root@workstation ~]# pwscore
simple
# Enter password and press Ctrl+D
# Should show low score or error

[root@workstation ~]# pwscore
ComplexP@ss123
# Should show high score (>60)
```

</details>

---

## Quick Reference Commands

### Essential Commands Cheat Sheet

<details>
<summary><b>Click to see command reference</b></summary>

**System Information:**
```bash
hostnamectl                    # View/set hostname
timedatectl                    # View/set date and time
uname -a                       # System information
cat /etc/os-release            # OS version
```

**User Management:**
```bash
useradd -u UID -G group user   # Create user
usermod -aG group user         # Add user to group
passwd user                    # Set password
chage -l user                  # Password aging info
id user                        # User ID information
```

**File Permissions:**
```bash
chmod 755 file                 # Change permissions
chown user:group file          # Change ownership
setfacl -m u:user:rwx file     # Set ACL
getfacl file                   # View ACL
```

**Storage:**
```bash
lsblk                          # List block devices
parted /dev/sda print          # View partitions
pvcreate /dev/sdb1             # Create physical volume
vgcreate vg_name /dev/sdb1     # Create volume group
lvcreate -L 1G -n lv_name vg   # Create logical volume
mkfs.xfs /dev/vg/lv            # Format filesystem
```

**Networking:**
```bash
nmcli con show                 # Show connections
nmcli dev status               # Show devices
ip addr show                   # Show IP addresses
firewall-cmd --list-all        # List firewall rules
```

**Services:**
```bash
systemctl start service        # Start service
systemctl enable service       # Enable at boot
systemctl status service       # Check status
journalctl -xe                 # View logs
```

**SELinux:**
```bash
getenforce                     # Check SELinux mode
semanage port -l               # List port contexts
restorecon -Rv /path           # Restore context
setsebool -P bool on           # Set boolean
ausearch -m avc -ts recent     # Check denials
```

**Containers:**
```bash
podman run -d image            # Run container
podman ps                      # List containers
podman images                  # List images
podman generate systemd        # Generate service
```

</details>

---

## Exam Tips

1. **Time Management**: You have 3 hours. Don't spend too much time on one question.
2. **Read Carefully**: Read each question twice before starting.
3. **Verify Your Work**: Always test your solutions before moving on.
4. **Use Man Pages**: `man` and `--help` are your friends during the exam.
5. **Persistence**: Ensure changes survive reboot (fstab, systemd, firewall --permanent).
6. **SELinux**: Never disable SELinux; fix the contexts instead.
7. **Document**: Keep notes of what you've done in case you need to troubleshoot.

---

## Additional Practice Resources

- [Red Hat Enterprise Linux Documentation](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9)
- [RHEL 9 System Administration I & II](https://www.redhat.com/en/services/training)
- Practice with virtual machines using VirtualBox or KVM
- Use `ansible-playbook` to reset lab environments

---

## Contributing

Found an issue or want to add more questions? Feel free to submit a pull request or open an issue!

---

## License

This practice material is provided for educational purposes. Red Hat, RHCSA, and RHEL are trademarks of Red Hat, Inc.

---

**Last Updated**: December 2025  
**RHEL Version**: 9.0  
**Exam Code**: EX200
