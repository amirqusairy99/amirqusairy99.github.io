---
title: "Bucharest — Connecting to PostgreSQL (Debian 11)"
categories: ["PostgreSQL", "Database Administration"]
tags: ["postgres", "troubleshooting", "linux", "debian"]
---

## Scenario

A web application could not connect to the PostgreSQL 13 database on the server.

### Application Credentials

- **Database:** `app1`
- **Username:** `app1user`
- **Password:** `app1user`

## Goal

The following command must succeed without errors:

```bash
PGPASSWORD=app1user psql -h 127.0.0.1 -d app1 -U app1user -c '\q'
```

## Initial Error

Running the command returned the following error message:

```
psql: error: FATAL:  pg_hba.conf rejects connection for host "127.0.0.1", user "app1user", database "app1", SSL on
FATAL:  pg_hba.conf rejects connection for host "127.0.0.1", user "app1user", database "app1", SSL off
```

### What This Indicated

- PostgreSQL service was running
- Network connectivity was working
- Authentication requests were reaching the PostgreSQL engine
- **The issue:** Access control configuration in `pg_hba.conf` was explicitly denying the connection

## Troubleshooting Process

### Step 1: Verify Firewall Status

Ensure that the OS-level firewall is not blocking traffic on the loopback interface.

Check UFW status:

```bash
sudo ufw status
```

**Result:** UFW was disabled.

Check iptables rules:

```bash
sudo iptables -L -n -v
```

**Result:**

```
Chain INPUT (policy ACCEPT)
Chain FORWARD (policy ACCEPT)
Chain OUTPUT (policy ACCEPT)
```

No firewall restrictions were present.

### Step 2: Locate PostgreSQL HBA Configuration

Identify the active `pg_hba.conf` file used by the running instance:

```bash
sudo -u postgres psql -c "SHOW hba_file;"
```

**Result:** `/etc/postgresql/13/main/pg_hba.conf`

Open the file for editing:

```bash
sudo nano /etc/postgresql/13/main/pg_hba.conf
```

### Step 3: Identify Problematic Rules

Upon inspection, the following entries were found near the top of the file:

```
host    all             all             all                     reject
host    all             all             all                     reject
```

**Problem:** PostgreSQL processes `pg_hba.conf` from top to bottom. The connection from `127.0.0.1` matched these generic reject rules before it could reach the valid localhost allow rule:

```
host    all             all             127.0.0.1/32            md5
```

### Step 4: Fix Configuration

Comment out the broad reject rules to allow the specific host rules to be evaluated:

```
#host    all             all             all                     reject
#host    all             all             all                     reject
```

### Step 5: Reload PostgreSQL

Apply the configuration changes without restarting the entire service:

```bash
sudo systemctl reload postgresql
```

### Step 6: Verify the Fix

Retest the database connectivity:

```bash
PGPASSWORD=app1user psql -h 127.0.0.1 -d app1 -U app1user -c '\q'
```

**Result:** The command completed successfully without errors.

## Root Cause

The issue was caused by overly broad reject rules placed at the beginning of the `pg_hba.conf` file. Because PostgreSQL uses a "first-match" logic, these rules denied all host-based connections before the specific localhost permission could be granted.

## Key Learning Points

- **Order Matters:** `pg_hba.conf` uses first-match rule processing. Always place specific "allow" rules above general "reject" rules.
- **Configuration vs. Network:** Errors stating "pg_hba.conf rejects connection" indicate a configuration problem within PostgreSQL, not a network or firewall blockage.
- **Live Reloads:** Configuration changes in HBA files can be applied with a reload, which is safer and faster than a full service restart.

## Useful Commands Summary

| Task | Command |
|------|---------|
| Check Logs | `sudo journalctl -u postgresql -f` |
| Find HBA File | `sudo -u postgres psql -c "SHOW hba_file;"` |
| Reload Config | `sudo systemctl reload postgresql` |
| Test Connection | `PGPASSWORD=app1user psql -h 127.0.0.1 -d app1 -U app1user -c '\q'` |