# NMAP — Red Team / Black Hat Ops

**Professional Advanced Reconnaissance & Stealth Scanning Guide**

---

## 1. Full Stealth SYN Scan

**Objective:** Conduct undetectable reconnaissance with minimal IDS/IPS evasion.

```bash
nmap -sS -Pn -T2 -f --mtu 24 --data-length 50 --spoof-mac 0 -D RND:8 <target_ip>
```

| Flag | Function | Purpose |
|------|----------|---------|
| `-sS` | Stealth SYN Scan | TCP SYN without completion, no logs |
| `-Pn` | Skip Ping Discovery | Bypass ICMP filtering |
| `-T2` | Paranoid Timing | Extremely slow scan, minimal detection |
| `-f` | Fragment Packets | Split IP packets to evade filters |
| `--mtu 24` | Reduce MTU | Force extreme fragmentation |
| `--data-length 50` | Inject Garbage Data | Add random bytes to confuse DPI |
| `--spoof-mac 0` | Random MAC Address | Every packet gets random MAC |
| `-D RND:8` | Use 8 Decoys | Mix traffic with 8 fake sources |

**Use Case:** Complete stealth reconnaissance. Target will see 8 different IPs scanning, making attribution nearly impossible.

**Detection Difficulty:** ⭐⭐⭐⭐⭐ (Expert IDS only)

---

## 2. Firewall Kill — Idle/Zombie Scan

**Objective:** Scan target using a third-party "zombie" machine to hide source IP.

```bash
nmap -sI <zombie_ip> <target_ip>
```

| Flag | Function | Purpose |
|------|----------|---------|
| `-sI` | Idle Scan | Use zombie machine for scanning |
| `<zombie_ip>` | Zombie Host | Predictable IP count machine (printer/router) |

**Use Case:** When you need complete IP spoofing. Target logs show zombie IP, not your IP.

**Zombie Selection:** 
- Old printers (HP, Canon)
- Unmaintained routers
- IoT devices with static IP count

**Detection Difficulty:** ⭐⭐⭐⭐⭐ (Requires forensics)

---

## 3. DNS Tunneling Detection & Leak

**Objective:** Identify DNS-based C2 channels and data exfiltration methods.

```bash
nmap --script dns-random-srcport -p 53 <target_ip>
```

| Component | Function |
|-----------|----------|
| `--script dns-random-srcport` | Detect DNS response randomization |
| `-p 53` | Target DNS service |

**Detects:**
- DNS tunneling (iodine, dns2tcp)
- Command & Control (C2) beacons
- Data exfiltration channels
- DNS rebinding attacks

**Detection Difficulty:** ⭐⭐⭐ (Moderate — DNS logs needed)

---

## 4. Anonymous Scanning via Tor

**Objective:** Route all traffic through Tor network for complete anonymity.

```bash
proxychains nmap -sS -Pn -p 22,80,443 <target_ip>
```

| Flag | Function | Purpose |
|------|----------|---------|
| `proxychains` | Socks Proxy Chain | Routes nmap through Tor |
| `-sS` | Stealth SYN | Maintains stealth through Tor |
| `-Pn` | Skip Ping | Avoid early detection |
| `-p 22,80,443` | Specific Ports | SSH, HTTP, HTTPS only |

**Setup:**
```bash
apt-get install proxychains tor
systemctl start tor
# Edit /etc/proxychains.conf → uncomment socks5 127.0.0.1 9050
```

**Use Case:** Your real IP remains completely hidden. Target sees Tor exit node IP.

**Detection Difficulty:** ⭐⭐⭐⭐ (Requires deep packet inspection)

---

## 5. Firewall Rules Reverse Engineering (Firewalking)

**Objective:** Map firewall policy and discover filtered/open rules.

```bash
nmap --script firewall-bypass -p 22,80,443 <target_ip>
nmap --script http-methods --script-args http-methods.url-path=/test <target_ip>
```

| Technique | Script | Reveals |
|-----------|--------|---------|
| Firewall Bypass | `firewall-bypass` | Allowed protocols & ports |
| HTTP Methods | `http-methods` | Enabled verbs (PUT, DELETE, TRACE) |
| CORS Headers | `http-cors` | Cross-origin access policies |

**Bypass Vectors:**
- Alternate ports (8080, 8888, 9999)
- HTTP tunneling
- Protocol downgrade attacks
- Source port spoofing (port 53, 25)

**Detection Difficulty:** ⭐⭐⭐ (IDS signature evasion required)

---

## 6. Backdoor & Reverse Shell Discovery

**Objective:** Identify common backdoor ports and implanted shells.

```bash
nmap -p 4444,5555,6666,7777,8888,9999,31337 --script backdoor <target_ip>
```

| Port | Backdoor Type | Payload |
|------|---------------|---------|
| 4444 | Metasploit Default | msfvenom reverse_tcp |
| 5555 | ADB (Android) | adb shell access |
| 6666 | IRCd Backdoor | IRC command bot |
| 7777 | QAZ Trojan | Remote shell |
| 8888 | HTTP Proxy | CONNECT tunnel |
| 9999 | Anixter | Remote access |
| 31337 | NetSpy/Subseven | Backdoor port |

**Extended Detection:**
```bash
nmap -p 1024-65535 --script-args backdoor.check-all <target_ip>
```

**Detection Difficulty:** ⭐⭐⭐⭐ (Requires behavioral analysis)

---

## 7. Network Sniffer Detection (ARP Poison / MITM)

**Objective:** Detect active network sniffing and ARP spoofing on target network.

```bash
nmap --script sniffer-detect -p 80 <target_ip>
```

| Attack Method | Detection Flag |
|---------------|----------------|
| ARP Spoofing | Gratuitous ARP packets |
| MAC Flooding | CAM table overflow |
| Network Tap | Bridged interface detection |
| DNS Spoofing | Response timing anomalies |

**Detects:**
- Wireshark/tcpdump in promiscuous mode
- Ettercap/Cain & Abel sessions
- arpspoof running
- MITM proxy devices

**Detection Difficulty:** ⭐⭐⭐ (Timing-based detection)

---

## 8. Exploit Validation (Direct Exploitation)

**Objective:** Verify known CVEs and attempt direct exploitation.

```bash
nmap --script exploit -p 445,139,3389 <target_ip>
```

| Port | Service | Exploit | CVE |
|------|---------|---------|-----|
| **445** | SMB | EternalBlue | CVE-2017-0144 |
| **139** | Samba | SambaCry | CVE-2017-7494 |
| **3389** | RDP | BlueKeep | CVE-2019-0708 |
| **3306** | MySQL | UDF Injection | Multiple |
| **5984** | CouchDB | No Auth RCE | CVE-2017-12635 |

**Advanced Exploit Scanning:**
```bash
nmap --script vuln -p- <target_ip> | grep "VULNERABLE"
```

**Detection Difficulty:** ⭐⭐ (Exploit attempts logged everywhere)

---

## 9. Reverse DNS & Hidden Subdomain Enumeration

**Objective:** Extract all subdomains, DNS records, and internal IP leaks.

```bash
nmap --script dns-brute -p 53 <target_domain>
```

| Script | Discovers |
|--------|-----------|
| `dns-brute` | All registered subdomains |
| `dns-reverse-query` | Reverse DNS zones |
| `dns-zone-transfer` | Full zone file (if misconfigured) |
| `dns-ip6-arpa-scan` | IPv6 reverse records |

**Zone Transfer Attempt (No Authentication):**
```bash
nmap --script dns-zone-transfer.nse --script-args dns-zone-transfer.domain=<target> <nameserver_ip>
```

**Common Internal Leaks:**
- `internal.domain.com` → 10.x.x.x
- `admin.domain.com` → VPN gateway
- `vpn.domain.com` → Fortinet/Cisco
- `mail.domain.com` → Exchange server

**Detection Difficulty:** ⭐ (No logging on DNS queries)

---

## 10. Black Hat Full Combo — Complete Anonymity & Zero Trace

**Objective:** Maximum stealth + maximum information gathering. Undetectable by all defenses.

```bash
proxychains nmap -sS -sV -O -p- -T2 -f --mtu 16 --data-length 100 --spoof-mac Cisco \
--ttl 64 --badsum -D RND:15 -Pn --randomize-hosts --source-port 53 -g 53 \
--scan-delay 5s --max-retries 0 -oA ghost_scan <target_ip>
```

### Advanced Parameter Breakdown:

| Parameter | Function | Evasion Effect |
|-----------|----------|-----------------|
| **proxychains** | Tor routing | IP completely hidden |
| **-sS** | Stealth SYN | No connection logs |
| **-sV** | Service detection | OS/application fingerprinting |
| **-O** | OS detection | Identify operating system |
| **-p-** | All ports | 65535 ports scanned |
| **-T2** | Paranoid timing | 5 minute+ scan duration |
| **-f** | Fragment packets | Bypass packet filters |
| **--mtu 16** | Minimal payload | Maximum fragmentation |
| **--data-length 100** | Junk data | Confuse DPI engines |
| **--spoof-mac Cisco** | Fake MAC | Impersonate Cisco device |
| **--ttl 64** | Time-to-live | Appear as Windows system |
| **--badsum** | Invalid checksum | IDS correlation bypass |
| **-D RND:15** | 15 decoys | 16 IP sources seen by target |
| **-Pn** | Skip ping | Avoid early detection |
| **--randomize-hosts** | Random order | Confuse log correlation |
| **--source-port 53** | DNS source port | Appear as DNS query |
| **-g 53** | Gateway port | DNS port for everything |
| **--scan-delay 5s** | 5 second gap | Evade time-based detection |
| **--max-retries 0** | No retransmission | Reduce network noise |
| **-oA ghost_scan** | Output format | Binary + XML + greppable |

### Complete Stealth Profile:

```
┌─────────────────────────────────────────┐
│ SOURCE IP:  Hidden behind 15 decoys     │
│ REAL IP:    Tor exit node               │
│ MAC:        Cisco (spoofed)             │
│ TTL:        64 (Windows-like)           │
│ SOURCE PORT: 53 (DNS - trusted)         │
│ PACKETS:    Fragmented, with junk data  │
│ CHECKSUMS:  Intentionally bad           │
│ TIMING:     5 second delays             │
│ RESULT:     Completely untrackable      │
└─────────────────────────────────────────┘
```

**Use Case:** 
- Government / Military targets
- High-security banking systems
- Fortune 500 companies
- Targets with advanced SOC/SIEM
- When arrest risk is real

**Detection Difficulty:** ⭐⭐⭐⭐⭐⭐ (Impossible — requires:)
- Tor node compromise
- ISP-level interception
- Physical surveillance

---

## Advanced Evasion Techniques

### 1. Packet Fragmentation Stacking
```bash
nmap -f -f --mtu 8 -p 22,80,443 <target_ip>
# Double fragmentation with 8-byte MTU (extreme)
```

### 2. Decoy + Spoofing Combo
```bash
nmap -D 192.168.1.1,RND:10,ME -S <spoofed_ip> <target_ip>
# Mix real IPs, random IPs, your IP, with spoofed source
```

### 3. UDP Fragmentation (IPID)
```bash
nmap -sU -f --mtu 24 -p 53,123,161 <target_ip>
# UDP scan with aggressive fragmentation
```

### 4. ACK Scanning (Firewall Mapping)
```bash
nmap -sA -p- <target_ip>
# Map firewall state without triggering alerts
```

### 5. Window Size Analysis
```bash
nmap -sW -p 22,80,443 <target_ip>
# Fingerprint OS via TCP window size
```

---

## Detection & Forensics Prevention

### Logs to Avoid:

❌ **Don't Get Caught:**
- Avoid scanning with YOUR real IP
- Use Tor + proxychains always
- Use decoys (RND:10 minimum)
- Randomize host order
- Use slow timing (-T1 or -T2)
- Change source port (not 1024 range)
- Spoof MAC address
- Fragment everything
- Use scan delays (10-30 seconds)

### Detection Signs You're Leaving:

⚠️ **These create logs:**
- Same source IP repeatedly
- All ports scanned sequentially
- Fast timing (T4, T5)
- Default nmap user-agent in scripts
- Non-fragmented packets
- Standard port order
- No decoys

---

## Legal Disclaimer

```
⚖️ WARNING ⚖️

These techniques are for:
✓ Authorized penetration testing
✓ Your own infrastructure
✓ Defensive security research
✓ Licensed red team operations

These are ILLEGAL for:
✗ Unauthorized access
✗ Hacking other people's systems
✗ Bypassing security without permission
✗ Criminal activity

Violators face:
- 10+ years federal prison (18 USC § 1030)
- $250,000+ in fines
- Civil liability
- Asset seizure

Use responsibly. You are responsible for your actions.
```

---

## Reference Commands Quick Sheet

```bash
# Stealth scan
nmap -sS -Pn -T2 -f --mtu 24 --spoof-mac 0 -D RND:8 <target>

# Tor anonymous
proxychains nmap -sS -Pn -p 22,80,443 <target>

# Zombie scan
nmap -sI <zombie> <target>

# Full combo (advanced)
proxychains nmap -sS -sV -O -p- -T2 -f --mtu 16 --data-length 100 \
--spoof-mac Cisco --ttl 64 --badsum -D RND:15 --randomize-hosts \
--source-port 53 -g 53 --scan-delay 5s -oA output <target>

# Zone transfer
nmap --script dns-zone-transfer --script-args dns-zone-transfer.domain=<domain> <ns>

# Backdoor hunt
nmap -p 4444,5555,6666,7777,8888,9999,31337 --script backdoor <target>

# Firewall map
nmap -sA -p- <target>
```

---

**Created:** Red Team Operations Database  
**Classification:** Advanced Reconnaissance  
**Skill Level:** Expert  
**Risk Level:** Critical

---
