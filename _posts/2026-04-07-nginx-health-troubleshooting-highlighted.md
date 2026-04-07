---
title: "Troubleshooting Nginx Health Check & systemd Timer Issues"
date: 2026-04-07
author: "Admin"
description: "Step-by-step debugging guide for Nginx localhost connectivity issues, firewall (iptables) blocks, and systemd timer problems."
categories: ["linux", "nginx", "troubleshooting"]
tags: ["nginx", "systemd", "curl", "iptables", "debugging"]
toc: true
---

<!--
Syntax Highlighting Support:

For Prism.js (recommended):
Include in your site layout:
<link href="https://cdn.jsdelivr.net/npm/prismjs/themes/prism-tomorrow.css" rel="stylesheet" />
<script src="https://cdn.jsdelivr.net/npm/prismjs/prism.js"></script>

For Highlight.js:
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/highlight.js/11.9.0/styles/github-dark.min.css">
<script src="https://cdnjs.cloudflare.com/ajax/libs/highlight.js/11.9.0/highlight.min.js"></script>
<script>hljs.highlightAll();</script>
-->

## 🧩 Scenario

You have a health check script:

```bash
#!/bin/bash
if curl -s --max-time 2 http://localhost | grep -q "Welcome to nginx"; then
  echo "$(date): STATUS: OK" >> /var/log/health.log
  exit 0
else
  echo "$(date): STATUS: FAILED" >> /var/log/health.log
  exit 1
fi
```

And a checker script:

```bash
/opt/scripts/health.sh &> /dev/null
SCRIPT_WORKS=$?

if systemctl is-enabled --quiet health.timer; then
  TIMER_ENABLED=0
else
  TIMER_ENABLED=1
fi

if systemctl is-active --quiet health.timer; then
  TIMER_ACTIVE=0
else
  TIMER_ACTIVE=1
fi
```

---

## 🚨 Problem Symptoms

- `curl http://localhost` hangs / times out  
- Nginx is running  
- Health script fails or behaves inconsistently  
- `health.timer` is inactive  
- Output: **NO**

---

## 🔍 Step-by-Step Troubleshooting

### 1. Check if Nginx is running

```bash
systemctl status nginx
```

**Expected:**
```text
active (running)
```

---

### 2. Verify Nginx is listening

```bash
ss -ltnp | grep ':80'
```

**Expected:**
```text
0.0.0.0:80
[::]:80
```

---

### 3. Test connectivity

```bash
curl -v --max-time 5 http://127.0.0.1/
curl -v --max-time 5 http://[::1]/
```

---

### 4. Check iptables (CRITICAL STEP)

```bash
sudo iptables -L -n --line-numbers
```

Look for:

```text
DROP tcp -- 0.0.0.0/0 127.0.0.1 tcp dpt:80
```

---

### 5. Remove blocking rule

```bash
sudo iptables -L OUTPUT -n --line-numbers
sudo iptables -D OUTPUT <rule-number>
```

---

### 6. Verify Nginx content

```bash
curl http://localhost
```

---

### 7. Fix permissions (log file issue)

```bash
sudo touch /var/log/health.log
sudo chown www-data:www-data /var/log/health.log
sudo chmod 664 /var/log/health.log
```

---

### 8. Check health script manually

```bash
/opt/scripts/health.sh
echo $?
```

---

### 9. Fix systemd timer

```bash
systemctl is-enabled health.timer
systemctl is-active health.timer
```

---

### 10. Enable and start timer

```bash
sudo systemctl enable --now health.timer
```

---

## ⚡ Quick Fix Summary

```bash
sudo iptables -L OUTPUT -n --line-numbers
sudo iptables -D OUTPUT <rule-number>

sudo systemctl enable --now health.timer

curl http://127.0.0.1/
systemctl is-active health.timer
/home/admin/agent/check.sh
```
