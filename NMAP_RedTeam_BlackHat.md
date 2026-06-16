# NMAP — Red Team / Black Hat Ops

**Professional Advanced Reconnaissance & Stealth Scanning Guide**

---

## 1. Full Stealth SYN Scan

**Objective:** Bilkul chup chaap scan karo jaha IDS/IPS ko kuch pata na chale.

```bash
nmap -sS -Pn -T2 -f --mtu 24 --data-length 50 --spoof-mac 0 -D RND:8 <target_ip>
```

| Flag | Kaam | Matlab |
|------|------|--------|
| `-sS` | Stealth SYN Scan | TCP SYN bina complete kiye, koi log nahi |
| `-Pn` | Ping ko chhod do | ICMP filtering ko bypass kar do |
| `-T2` | Dhimi gati | Bahut dhire scan karo, detection kam ho |
| `-f` | Packet tod do | IP packets ko tukdon mein bhejo |
| `--mtu 24` | MTU kam karo | Extreme fragmentation karo |
| `--data-length 50` | Kachda daalo | Random bytes se DPI ko confuse karo |
| `--spoof-mac 0` | Random MAC | Har packet ka alag MAC ho |
| `-D RND:8` | 8 Decoys lagao | 8 fake IPs se scan dikhے |

**Use Case:** Puri tarah secret reconnaissance. Target ko 8 alag-alag IPs dikhenge, tera IP kabhi nahi.

**Detection Difficulty:** ⭐⭐⭐⭐⭐ (Sirf expert IDS pakad sakta hai)

---

## 2. Firewall Kill — Idle/Zombie Scan

**Objective:** Kissi teesre computer (zombie) ko use karke scan karna taaki tera IP chupa rahe.

```bash
nmap -sI <zombie_ip> <target_ip>
```

| Flag | Kaam | Matlab |
|------|------|--------|
| `-sI` | Idle Scan | Zombie machine se scanning karo |
| `<zombie_ip>` | Zombie Host | Woh PC jiska IP predictable ho (printer/router) |

**Use Case:** Tera IP bilkul chup jata hai. Target ko zombie ka IP dikhta hai, tera nahi.

**Zombie Devices:**
- Purane printer (HP, Canon)
- Purane router jo update nahi hue
- IoT devices jinke paas static IP count ho

**Detection Difficulty:** ⭐⭐⭐⭐⭐ (Forensics chahiye pakadne ko)

---

## 3. DNS Tunneling Detection

**Objective:** DNS ke through hidden command & control channels ko detect karna.

```bash
nmap --script dns-random-srcport -p 53 <target_ip>
```

| Component | Function |
|-----------|----------|
| `--script dns-random-srcport` | DNS ka randomization detect karo |
| `-p 53` | DNS port ko target karo |

**Kya detect hoga:**
- DNS tunneling (iodine, dns2tcp)
- Command & Control beacons
- Data exfiltration (chhori ho raha data)
- DNS rebinding attacks

**Detection Difficulty:** ⭐⭐⭐ (DNS logs chahiye detect karne ko)

---

## 4. Tor Ke Through Anonymous Scanning

**Objective:** Pure Tor network se scan karna taaki real IP kabhi na mile.

```bash
proxychains nmap -sS -Pn -p 22,80,443 <target_ip>
```

| Flag | Function | Kya hoga |
|------|----------|----------|
| `proxychains` | Socks proxy se route karo | Tor ke through hoga scan |
| `-sS` | Stealth SYN | Tor ke through bhi stealth rahe |
| `-Pn` | Ping mat karo | Jaldi detect na ho jaye |
| `-p 22,80,443` | Sirf ye ports | SSH, HTTP, HTTPS hi scan karo |

**Setup Kaise Karo:**
```bash
apt-get install proxychains tor
systemctl start tor
# /etc/proxychains.conf edit karo → socks5 127.0.0.1 9050 ko uncomment karo
```

**Use Case:** Tera real IP bilkul hide ho jata hai. Target ko Tor exit node ka IP dikhega.

**Detection Difficulty:** ⭐⭐⭐⭐ (Deep packet inspection chahiye)

---

## 5. Firewall Rules Reverse Engineering

**Objective:** Pata karo ki firewall kaun se ports ko allow karta hai.

```bash
nmap --script firewall-bypass -p 22,80,443 <target_ip>
nmap --script http-methods --script-args http-methods.url-path=/test <target_ip>
```

| Technique | Script | Kya reveal hoga |
|-----------|--------|-----------------|
| Firewall Bypass | `firewall-bypass` | Kaun se protocols aur ports allowed hain |
| HTTP Methods | `http-methods` | Kaun se HTTP methods enable hain |
| CORS Headers | `http-cors` | Cross-origin policies kya hain |

**Bypass Ke Tarike:**
- Alternate ports (8080, 8888, 9999)
- HTTP tunneling
- Protocol ko downgrade karna
- Source port ko spoof karna (port 53, 25)

**Detection Difficulty:** ⭐⭐⭐ (IDS ko bypass karna padega)

---

## 6. Backdoor Aur Reverse Shell Dhundna

**Objective:** Common backdoor ports ko scan karke dekhna ki shell pehle se open hai ya nahi.

```bash
nmap -p 4444,5555,6666,7777,8888,9999,31337 --script backdoor <target_ip>
```

| Port | Backdoor Ka Naam | Kya Hota Hai |
|------|-----------------|------------|
| 4444 | Metasploit Default | msfvenom reverse_tcp shell |
| 5555 | ADB (Android) | Android shell access |
| 6666 | IRCd Backdoor | IRC bot command |
| 7777 | QAZ Trojan | Remote shell |
| 8888 | HTTP Proxy | CONNECT tunnel |
| 9999 | Anixter | Remote access |
| 31337 | NetSpy/Subseven | Backdoor port |

**Aur Ports Scan Karne Ke Liye:**
```bash
nmap -p 1024-65535 --script-args backdoor.check-all <target_ip>
```

**Detection Difficulty:** ⭐⭐⭐⭐ (Behavioral analysis se pata chale)

---

## 7. Network Sniffer Detect Karna (ARP Poison / MITM)

**Objective:** Pata karo ki network mein sniffing attack chal raha hai ya nahi.

```bash
nmap --script sniffer-detect -p 80 <target_ip>
```

| Attack Type | Detection Sign |
|------------|----------------|
| ARP Spoofing | Gratuitous ARP packets |
| MAC Flooding | CAM table bhur jata hai |
| Network Tap | Bridged interface detect ho |
| DNS Spoofing | Response timing gadbad ho |

**Detect Hoga:**
- Wireshark/tcpdump jo promiscuous mode mein chal raha ho
- Ettercap/Cain & Abel sessions
- arpspoof jo chal raha ho
- MITM proxy devices

**Detection Difficulty:** ⭐⭐⭐ (Timing-based detection)

---

## 8. Exploit Direct Karna (Turant Attack)

**Objective:** Known vulnerabilities ko check karna aur sidhe exploit karna.

```bash
nmap --script exploit -p 445,139,3389 <target_ip>
```

| Port | Service | Exploit Ka Naam | CVE |
|------|---------|-----------------|-----|
| **445** | SMB | EternalBlue | CVE-2017-0144 |
| **139** | Samba | SambaCry | CVE-2017-7494 |
| **3389** | RDP | BlueKeep | CVE-2019-0708 |
| **3306** | MySQL | UDF Injection | Multiple |
| **5984** | CouchDB | No Auth RCE | CVE-2017-12635 |

**Advanced Exploit Scanning:**
```bash
nmap --script vuln -p- <target_ip> | grep "VULNERABLE"
```

**Detection Difficulty:** ⭐⭐ (Exploit attempts sab jagah log hote hain)

---

## 9. Reverse DNS Aur Hidden Subdomains Khojhna

**Objective:** Sab subdomains, DNS records, aur internal IPs ko nikalna.

```bash
nmap --script dns-brute -p 53 <target_domain>
```

| Script | Kya Milega |
|--------|-----------|
| `dns-brute` | Sab registered subdomains |
| `dns-reverse-query` | Reverse DNS zones |
| `dns-zone-transfer` | Pure zone file (agar misconfigured ho) |
| `dns-ip6-arpa-scan` | IPv6 reverse records |

**Zone Transfer Karna (Agar Authentication Na Ho):**
```bash
nmap --script dns-zone-transfer.nse --script-args dns-zone-transfer.domain=<target> <nameserver_ip>
```

**Common Internal Leaks:**
- `internal.domain.com` → 10.x.x.x
- `admin.domain.com` → VPN gateway
- `vpn.domain.com` → Fortinet/Cisco firewall
- `mail.domain.com` → Exchange server

**Detection Difficulty:** ⭐ (DNS queries ka koi log nahi hota)

---

## 10. Black Hat Full Combo — Pure Anonymous Aur Zero Trace

**Objective:** Maximum stealth + maximum information. Sab defenses ko fool karna.

```bash
proxychains nmap -sS -sV -O -p- -T2 -f --mtu 16 --data-length 100 --spoof-mac Cisco \
--ttl 64 --badsum -D RND:15 -Pn --randomize-hosts --source-port 53 -g 53 \
--scan-delay 5s --max-retries 0 -oA ghost_scan <target_ip>
```

### Advanced Parameters Ki Jankari:

| Parameter | Function | Kya Faida Hai |
|-----------|----------|--------------|
| **proxychains** | Tor routing | IP bilkul hidden hai |
| **-sS** | Stealth SYN | Connection ka koi log nahi |
| **-sV** | Service detect karo | OS/application fingerprint nikalo |
| **-O** | OS pata karo | Operating system identify karo |
| **-p-** | Sab ports | 65535 ports sab scan karo |
| **-T2** | Paranoid timing | 5+ minute ka scan, slow aur steady |
| **-f** | Packets tod do | Packet filters ko bypass karo |
| **--mtu 16** | Minimal payload | Maximum fragmentation |
| **--data-length 100** | Kachda daalo | DPI engines ko confuse karo |
| **--spoof-mac Cisco** | MAC spoofing | Cisco device ban jao |
| **--ttl 64** | Time-to-live | Windows system jaisa lag jao |
| **--badsum** | Galat checksum | IDS ko confuse karo |
| **-D RND:15** | 15 Decoys | 16 alag IP sources dikhen |
| **-Pn** | Ping mat karo | Jaldi detect na ho jaye |
| **--randomize-hosts** | Random order | Log correlation ko mess karo |
| **--source-port 53** | DNS port se | DNS query jaisa lag jaye |
| **-g 53** | Gateway port 53 | Sab kuch DNS port se bhejo |
| **--scan-delay 5s** | 5 sec ka gap | Time-based detection bypass karo |
| **--max-retries 0** | No retransmission | Network noise kam karo |
| **-oA ghost_scan** | Output formats | Binary + XML + text format mein save karo |

### Complete Stealth Profile:

```
┌──────────────────────────────────────────────┐
│ SOURCE IP:  15 Decoys ke piche chupa hua     │
│ REAL IP:    Tor exit node ke through         │
│ MAC:        Cisco (fake - spoofed)           │
│ TTL:        64 (Windows jaisa)               │
│ SOURCE PORT: 53 (DNS - trusted port)         │
│ PACKETS:    Fragmented + kachda data bhara   │
│ CHECKSUMS:  Intentionally galat              │
│ TIMING:     5 sec delays se slow             │
│ RESULT:     100% untrackable - koi trace nahi│
└──────────────────────────────────────────────┘
```

**Use Case - Kab Istimal Karo:**
- Government / Military targets
- High-security banking systems
- Fortune 500 companies
- Targets jinke paas advanced SOC/SIEM ho
- Jab arrest ka risk bahut zyada ho

**Detection Difficulty:** ⭐⭐⭐⭐⭐⭐ (Impossible - Iske liye chahiye:)
- Tor node ko hack karna
- ISP-level interception
- Physical surveillance karna

---

## Advanced Evasion Techniques

### 1. Packet Fragmentation Ko Double Karna
```bash
nmap -f -f --mtu 8 -p 22,80,443 <target_ip>
# Double fragmentation 8-byte MTU ke sath (bahut extreme)
```

### 2. Decoy + Spoofing Ka Combo
```bash
nmap -D 192.168.1.1,RND:10,ME -S <spoofed_ip> <target_ip>
# Real IPs, random IPs, tera IP, sab mix karo spoofed source ke sath
```

### 3. UDP Fragmentation (IPID Ke Through)
```bash
nmap -sU -f --mtu 24 -p 53,123,161 <target_ip>
# UDP scan aggressive fragmentation ke sath
```

### 4. ACK Scanning (Firewall Ko Map Karo)
```bash
nmap -sA -p- <target_ip>
# Firewall state ko map karo alerts ke bina
```

### 5. Window Size Ko Analyze Karo
```bash
nmap -sW -p 22,80,443 <target_ip>
# OS ko fingerprint karo TCP window size ke through
```

---

## Detection Aur Forensics Se Bachna

### Logs Ko Kaise Avoid Karo:

❌ **Pakde Mat Jao:**
- Apna real IP use mat karo
- Hamesha Tor + proxychains use karo
- Decoys lagao (kam se kam RND:10)
- Host order ko randomize karo
- Slow timing use karo (-T1 ya -T2)
- Source port ko change karo (1024 range nahi)
- MAC address ko spoof karo
- Sab kuch ko fragment karo
- Scan delays lagao (10-30 seconds)

### Detection Ke Signs - Ye Dikh Jaenge:

⚠️ **Ye Cheezy Logs Mein Dikh Jati Hain:**
- Same IP bar-bar scan karta rahe
- Sab ports sequentially scan hon
- Fast timing (T4, T5)
- Default nmap user-agent scripts mein
- Non-fragmented packets
- Standard port order
- Koi decoys nahi

---

## Kanuni Chetavni

```
⚖️ Savdhani! ⚖️

Ye techniques sirf iske liye authorized hain:
✓ Authorized penetration testing
✓ Apne khud ke infrastructure par
✓ Defensive security research
✓ Licensed red team operations

Ye techniques ILLEGAL hain yaha:
✗ Unauthorized access
✗ Dusron ke systems ko hack karna
✗ Permission ke bina security ko bypass karna
✗ Criminal activity

Violators ko saza milti hai:
- 10+ saal federal prison (18 USC § 1030)
- $250,000+ ka jarmana
- Civil liability
- Asset seizure

Zimmedari se istmal karo. Tu apne actions ke liye responsible hai.
```

---

## Quick Reference Commands Sheet

```bash
# Stealth scan karo
nmap -sS -Pn -T2 -f --mtu 24 --spoof-mac 0 -D RND:8 <target>

# Tor ke through anonymous
proxychains nmap -sS -Pn -p 22,80,443 <target>

# Zombie se scan karo
nmap -sI <zombie> <target>

# Full combo (advanced - maximum anonymity)
proxychains nmap -sS -sV -O -p- -T2 -f --mtu 16 --data-length 100 \
--spoof-mac Cisco --ttl 64 --badsum -D RND:15 --randomize-hosts \
--source-port 53 -g 53 --scan-delay 5s -oA output <target>

# Zone transfer karo
nmap --script dns-zone-transfer --script-args dns-zone-transfer.domain=<domain> <ns>

# Backdoor ko dhundho
nmap -p 4444,5555,6666,7777,8888,9999,31337 --script backdoor <target>

# Firewall ko map karo
nmap -sA -p- <target>
```

---

**Banaya Gaya:** Red Team Operations Database  
**Classification:** Advanced Reconnaissance  
**Skill Level:** Expert  
**Risk Level:** Critical  
**Language:** Hinglish (Hindi + English Mix)

---

**Last Updated:** 2026-06-16  
**Creator:** Red Team Ops

---
