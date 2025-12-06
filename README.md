# RHCSA Practice Questions & Solutions

---

## 🎯 Q1: Configure network and set the static parameters
Consider machine configured as DHCP, need to config it with static parameters.
- IP-ADDRESS= 172.25.250.10
- NETMASK= 255.255.255.0
- GATEWAY= 172.25.250.254
- Nameserver= 172.24.254.254
- Hostname= servera.lab.example.com

<details>
<summary><b>🔍 SOLUTION - Click to Reveal</b></summary>

```bash
nmcli connection show
nmcli connection modify LAN ipv4.addresses 172.25.250.10/24 ipv4.gateway 172.25.250.254 ipv4.dns 172.25.254.254
nmcli connection modify LAN ipv4.method manual
nmcli connection up LAN
nmcli connection show
```

Set up hostname:
```bash
hostnamectl set-hostname servera.lab.example.com
hostnamectl status
```

</details>

---

## 🎯 Q2: Configure your system to use this location as a default repository (public/local repo)
1. http://content.example.com/rhel9.0/x86_64/rhcsa-practice/rht
2. http://content.example.com/rhel9.0/x86_64/rhcsa-practice/errata

<details>
<summary><b>🔍 SOLUTION - Click to Reveal</b></summary>

```bash
vim /etc/yum.repos.d/exam.repo
```

Add the following content:
```ini
[demo1]
name=demo1 repo
baseurl=http://content.example.com/rhel9.0/x86_64/rhcsa-practice/rht
enabled=1
gpgcheck=0

[demo2]
name=AppStream repo
baseurl=http://content.example.com/rhel9.0/x86_64/rhcsa-practice/errata
enabled=1
gpgcheck=0
```

Verify the repository:
```bash
dnf repolist
```

If other repos are listed, disable them to set default repo:
```bash
sudo dnf config-manager --disable rhel-9-for-x86_64-appstream-rpms
sudo dnf config-manager --disable rhel-9-for-x86_64-baseos-rpms
```

</details>

---

## 🎯 Q3: Managing Local Users and Groups
Create the following users, groups and group memberships:
- A group named sharegrp
- A user harry who belongs to sharegrp as a secondary group
- A user natasha who also belongs to sharegrp as a secondary group
- A user copper who does not have access to an interactive shell on the system and who is not a member of sharegrp
- harry, natasha and copper should have the password redhat

<details>
<summary><b>🔍 SOLUTION - Click to Reveal</b></summary>

```bash
groupadd sharegrp
useradd -G sharegrp harry
useradd -G sharegrp natasha
useradd -s /sbin/nologin copper
passwd harry
passwd natasha
passwd copper
```

For verification:
```bash
tail -5 /etc/passwd
tail -5 /etc/group
tail -5 /etc/shadow
```

</details>

---

## 🎯 Q4: Controlling Access to Files
Create collaborative directory /var/shares with the following characteristics:
- Group ownership of /var/shares should be sharegrp
- The directory should be readable, writable and accessible to member of sharegrp but not to any other user
- Files created in /var/shares automatically have group ownership set to the sharegrp group

<details>
<summary><b>🔍 SOLUTION - Click to Reveal</b></summary>

```bash
mkdir -p /var/shares
ls -ld /var/shares
chgrp sharegrp /var/shares/
```

Or alternatively:
```bash
chown :sharegrp /var/shares/
```

Set permissions:
```bash
ls -ld /var/shares
chmod 770 /var/shares
chmod 2770 /var/shares
```

</details>

---

## 🎯 Q5: Find all lines containing a string
Find all lines in the file /usr/share/mime/packages/freedesktop.org.xml that contain the string "ich". Put a copy of these lines in the original order in the file /root/lines. /root/lines should contain no empty lines and all lines must be exact copies of the original lines.

<details>
<summary><b>🔍 SOLUTION - Click to Reveal</b></summary>

```bash
grep ich /usr/share/mime/packages/freedesktop.org.xml > /root/lines
cat /root/lines
```

</details>

---

## 🎯 Q6: Find files by owner and size
- Find all the files owned by user natasha and redirect the output to /tmp/output
- Find all files that are larger than 5MiB in the /etc directory and copy them to /find/largedir and redirect the output to /find/largefiles

<details>
<summary><b>🔍 SOLUTION - Click to Reveal</b></summary>

```bash
find / -user natasha -type f > /tmp/output
cat /tmp/output
```

For large files:
```bash
mkdir -p /find/largedir
find /etc -size +5M -exec cp {} /find/largedir \;
```

And redirect the list:
```bash
find /etc -size +5M > /find/largefiles
```

</details>

---

## 🎯 Q7: Create a user with specific UID
Create a user fred with a user ID 3945. Give the password as iamredhatman

<details>
<summary><b>🔍 SOLUTION - Click to Reveal</b></summary>

```bash
useradd -u 3945 fred
passwd fred
```

For verification:
```bash
tail -1 /etc/passwd
tail -1 /etc/shadow
```

</details>

---

## 🎯 Q8: Search string in a file
Search the string "nologin" in the /etc/passwd file and save the output in /root/strings

<details>
<summary><b>🔍 SOLUTION - Click to Reveal</b></summary>

```bash
grep nologin /etc/passwd > /root/strings
cat /root/strings
```

</details>

---

## 🎯 Q9: Configuring NTP/Time Synchronization
Configure your system so that it is an NTP client of classroom.example.com

<details>
<summary><b>🔍 SOLUTION - Click to Reveal</b></summary>

```bash
dnf install chrony
vim /etc/chrony.conf
```

Add this line (comment out other server lines):
```
server classroom.example.com iburst
```

Enable and start the service:
```bash
systemctl enable chronyd --now
systemctl restart chronyd
systemctl status chronyd
chronyc sources -v
```

</details>

---

## 🎯 Q10: Scheduling Future Tasks with Cron
The user natasha must configure a cron job that runs daily at 14:23 local time or also the same cron job will run after every 2 minutes and executes: /bin/echo hello

<details>
<summary><b>🔍 SOLUTION - Click to Reveal</b></summary>

```bash
crontab -u natasha -e
```

Add the following lines:
```
23 14 * * * /bin/echo hello
*/2 * * * * /bin/echo hello
```

Verify:
```bash
crontab -l -u natasha
systemctl status crond.service
```

</details>

---

## 🎯 Q11: Archiving and Transferring Files & SELinux
Create a backup file named /root/backup.tar.bz2 or /root/backup.tar.gz2. The backup file should contain the content of /usr/local and should be zipped with bzip2 or gzip2 compression format. Furthermore, ensure SELinux is in enforcing mode.

<details>
<summary><b>🔍 SOLUTION - Click to Reveal</b></summary>

```bash
tar cjvf /root/backup.tar.bz2 /usr/local
ls
```

</details>

---

## 🎯 Q12: Create a Bash Script
Create a script file name find.sh. When you run this script, it will find all files from 30K to 60k file size from the /etc directory & copies those files to /root/data directory.

<details>
<summary><b>🔍 SOLUTION - Click to Reveal</b></summary>

```bash
cat > find.sh << 'EOF'
#!/bin/bash

# Directory where files will be copied
DEST_DIR="/root/data"

# Check if destination directory exists, if not create it
if [ ! -d "$DEST_DIR" ]; then
    mkdir -p "$DEST_DIR"
fi

# Find files in /etc with size between 30 KB and 60 KB and copy them to /root/data
find /etc -type f -size +30k -size -60k -exec cp {} "$DEST_DIR" \;

echo "Files copied successfully to $DEST_DIR"
EOF

chmod a+x find.sh
```

</details>

---

## 🎯 Q13: Managing SELinux Security for Web Server
Your webcontent has been configured in port 82 at the /var/www/html directory. Make the content accessible.

<details>
<summary><b>🔍 SOLUTION - Click to Reveal</b></summary>

Check the current status:
```bash
curl http://servera.lab.example.com:82
systemctl status httpd
systemctl start httpd
journalctl -xe
```

Add SELinux port:
```bash
semanage port -l | grep http
semanage port -a -t http_port_t -p tcp 82
semanage port -l | grep http
```

Configure firewall:
```bash
firewall-cmd --permanent --add-port=82/tcp
firewall-cmd --reload
firewall-cmd --list-all
```

Restart and verify:
```bash
systemctl restart httpd
curl http://servera.lab.example.com:82
```

</details>

---

## 🎯 Q14: Set the Password Expire Date
The password for all new users in servera.lab.example.com should expire after 30 days.

<details>
<summary><b>🔍 SOLUTION - Click to Reveal</b></summary>

```bash
vim /etc/login.defs
```

Find and modify the line:
```
PASS_MAX_DAYS 30
```

</details>

---

## 🎯 Q15: Autofs Configuration
Configure autofs to automount the home directories of user remoteuser15:
- utility.lab.example.com (172.24.10.10) NFS-exports /netdir to your system
- remoteuser15's home directory is utility.lab.example.com:/netdir/remoteuser15
- remoteuser15's home directory should be auto mounted locally beneath /netdir as /netdir/remoteuser15
- Home directories must be writable by their users

<details>
<summary><b>🔍 SOLUTION - Click to Reveal</b></summary>

```bash
dnf install autofs
systemctl enable --now autofs
vim /etc/auto.master.d/demo.autofs
```

Add:
```
/netdir /etc/auto.demo
```

Create auto.demo file:
```bash
vim /etc/auto.demo
```

Add:
```
remoteuser15 -rw,sync utility.lab.example.com:/netdir/remoteuser15
```

Restart and verify:
```bash
systemctl restart autofs
df -h
su - remoteuser15
ls
touch file1
```

</details>

---

## 🎯 Q16: Add a Swap Partition
Add an additional swap partition of 512 MiB to your system. The swap partition should automatically mount when your system boots. Do not remove or otherwise alter any existing swap partition.

<details>
<summary><b>🔍 SOLUTION - Click to Reveal</b></summary>

```bash
parted /dev/vdb print
parted /dev/vdb mkpart myswap linux-swap 1001MB 1501MB
udevadm settle
mkswap /dev/vdb2
swapon /dev/vdb2
vim /etc/fstab
```

Append:
```
/dev/vdb2 swap swap defaults 0 0
```

Verify:
```bash
swapon -a
free -m
```

</details>

---

## 🎯 Q17: Create a Logical Volume
Create a new logical volume according to the following requirements:
- The logical volume is named database and belongs to the datastore volume group and has a size of 50 extents
- Logical volume in the datastore volume group should have an extent size of 16 MiB
- Format the new logical volume with vfat filesystem
- The logical volume should be automatically mounted under /mnt/database at system boot time

<details>
<summary><b>🔍 SOLUTION - Click to Reveal</b></summary>

```bash
parted /dev/vdb print
parted /dev/vdb mkpart primary 1001MB 2001MB
parted /dev/vdb set 3 lvm on
udevadm settle
pvcreate /dev/vdb3
vgcreate -s 16M datastore /dev/vdb3
vgdisplay
lvcreate -n database -L 800M datastore
lvdisplay
mkfs.vfat /dev/datastore/database
mkdir /mnt/database
vim /etc/fstab
```

Append:
```
/dev/datastore/database /mnt/database vfat defaults 0 0
```

Mount:
```bash
mount -a
df -h
```

</details>

---

## 🎯 Q18: LVM Partition Resize
Resize LVM partition to 850MB. Where LV name is database. Partition size must be within approximately 830MB to 865MB and usable.

<details>
<summary><b>🔍 SOLUTION - Click to Reveal</b></summary>

```bash
df -h
lvdisplay
lvextend -r -L 850M /dev/datastore/database
lvdisplay
df -h
```

</details>

---

## 🎯 Q19: Tuning System Performance
Change the current tuning profile for serverb to default profile

<details>
<summary><b>🔍 SOLUTION - Click to Reveal</b></summary>

```bash
dnf install tuned -y
systemctl enable tuned
systemctl start tuned
tuned-adm recommend
tuned-adm profile virtual-guest
tuned-adm active
tuned-adm profile_info
```

</details>

---

## 🎯 Q20: UMASK Configure
Set permission for user daffy. User will get the permission below for file & directory when he creates new files or directory:
- Files: -rw-------
- Directories: drwx-------

<details>
<summary><b>🔍 SOLUTION - Click to Reveal</b></summary>

```bash
su - daffy
vim .bashrc
```

Append:
```
umask 0077
```

Save and exit:
```bash
exit
```

</details>

---

## 🎯 Q21: SUDO Configuration
Configure sudo permissions for users who are members of the admin group, allowing them to use sudo without a password.

<details>
<summary><b>🔍 SOLUTION - Click to Reveal</b></summary>

```bash
vim /etc/sudoers.d/exam
```

Add:
```
%admin ALL=(ALL) NOPASSWD: ALL
```

</details>

---

## 🎯 Q22: Container - Create a Container Image
Create a container image named monitor from a Containerfile from http://fromwebserver/Containerfile. All this task done using student user.

<details>
<summary><b>🔍 SOLUTION - Click to Reveal</b></summary>

```bash
podman login <exam registry url>
# Enter username and password

mkdir demo
cd demo
wget http://fromwebserver/Containerfile
podman build -t monitor .
podman images
```

</details>

---

## 🎯 Q23: Container - Create Rootless Container
Create rootless container according to the following requirements:
- Create a container named as 'ascii2pdf' using the previously created container image monitor
- Map the '/opt/files' to container '/opt/incoming'
- Map the '/opt/processed' to container '/opt/outgoing'
- Create systemd service as container-ascii2pdf.service
- Make service active after all server reboots

<details>
<summary><b>🔍 SOLUTION - Click to Reveal</b></summary>

As root:
```bash
mkdir /opt/files /opt/processed
chown student:student /opt/files /opt/processed
```

As student user:
```bash
ssh student@localhost
podman run -d --name ascii2pdf -v /opt/files:/opt/incoming:Z -v /opt/processed:/opt/outgoing:Z localhost/monitor:latest

mkdir -p ~/.config/systemd/user
cd ~/.config/systemd/user
podman generate systemd --name ascii2pdf --new --files
podman stop ascii2pdf
podman rm ascii2pdf
systemctl --user daemon-reload
loginctl enable-linger
systemctl --user enable container-ascii2pdf.service --now
podman ps
```

</details>

---
