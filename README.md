# 
---

## 🎯 Q 1: Configure network and set the static parameters. Consider machine configured as DHCP, need to config it with static parameters.
IP-ADDRESS= 172.25.250.10
NETMASK= 255.255.255.0
GATEWAY= 172.25.250.254
Nameserver= 172.24.254.254
Hostname= servera.lab.example.com

<details>
<summary><b>🔍 SOLUTION - Click to Reveal</b></summary>



```bash
nmcli connection show
nmcli connection modify LAN ipv4.addresses 172.25.250.10/24 ipv4.gateway 172.25.250.254 ipv4.dns
172.25.254.254
nmcli connection modify LAN ipv4.method manual
nmcli connection up LAN
nmcli connection show
```
Set up hostname
```bash
hostnamectl set-hostname servera.lab.example.com
hostnamectl status
```
</details>
---

## 🎯 Q 2: • Configure your system to use this location as a default repository (public/local repo):
1. http://content.example.com/rhel9.0/x86_64/rhcsa-practice/rht
2. http://content.example.com/rhel9.0/x86_64/rhcsa-practice/errata

<details>
<summary><b>🔍 SOLUTION - Click to Reveal</b></summary>



```bash
vim /etc/yum.repos.d/exam.repo
```
[demo1]  
name=demo1 repo  
baseurl= http://content.example.com/rhel9.0/x86_64/rhcsa-practice/rht  
enabled=1  
gpgcheck=0  
[demo2]  
name=AppStram repo  
baseurl= http://content.example.com/rhel9.0/x86_64/rhcsa-practice/errata  
enabled=1  
gpgcheck=0  

```bash
dnf repolist
```
output:  
repo id repo name  
demo1 demo1 repo  
demo2 demo2 repo  

If other repo listed here, then default repository is not set.For set default repo, do this: 
Suppose other repo name is rhel-9-for-x86_64-appstream-rpms and rhel-9-for-x86_64-baseos-rpms. 
sudo dnf config-manager --disable rhel-9-for-x86_64-appstream-rpms
sudo dnf config-manager --disable rhel-9-for-x86_64-baseos-rpms
