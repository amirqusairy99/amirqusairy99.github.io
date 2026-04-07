---
layout: post
title: "Linux Server Review: A Guided Walkthrough of a Debian 11 Host"
date: 2026-04-07
categories: [linux, server, devops, debugging]
tags: [debian, docker, postgresql, gotty, systemd, troubleshooting]
excerpt: "A guided review of a Debian 11 server to identify its purpose, running services, listening ports, and notable security observations."
---

# Linux Server Review: A Guided Walkthrough of a Debian 11 Host

I recently worked through a guided Linux server review scenario with a simple goal: inspect a server well enough to answer a few practical questions.

- What is the purpose of the server?
- What is running on it?
- Are there any obvious problems?
- What should I pay attention to from an operations or security perspective?

This post documents the investigation and the conclusions.

## Scenario

**Level:** Easy  
**OS:** Debian 11  
**Access:** sudo available  
**Challenge type:** Guided learning, no fixed solution

## Initial Context

The machine presented itself like this:

```bash
admin@ip-10-1-12-102:~$ id -u
1000
admin@ip-10-1-12-102:~$ whoami
admin
```

This tells me I was logged in as the `admin` user, not as `root`.

- `id -u` returns `0` only for root
- `1000` is a regular user ID on a typical Linux system

So the shell session started as a non-root account, but the scenario stated that sudo access was available.

## First Question: What services are running?

I started with:

```bash
systemctl list-units --type=service --state=running
```

Output:

```text
UNIT                        LOAD   ACTIVE SUB     DESCRIPTION
chrony.service              loaded active running chrony, an NTP client/server
containerd.service          loaded active running containerd container runtime
cron.service                loaded active running Regular background program processing daemon
dbus.service                loaded active running D-Bus System Message Bus
docker.service              loaded active running Docker Application Container Engine
getty@tty1.service          loaded active running Getty on tty1
gotty.service               loaded active running Gotty
postgresql@13-main.service  loaded active running PostgreSQL Cluster 13-main
rsyslog.service             loaded active running System Logging Service
serial-getty@ttyS0.service  loaded active running Serial Getty on ttyS0
ssh.service                 loaded active running OpenBSD Secure Shell server
systemd-journald.service    loaded active running Journal Service
systemd-logind.service      loaded active running User Login Management
systemd-udevd.service       loaded active running Rule-based Manager for Device Events and Files
unattended-upgrades.service loaded active running Unattended Upgrades Shutdown
webapp.service              loaded active running webapp
wrk.service                 loaded active running wrk web stress
```

### What this already suggests

A few services stand out immediately:

- `docker.service` and `containerd.service` suggest the host is used for containerized workloads
- `postgresql@13-main.service` suggests a local PostgreSQL database
- `webapp.service` strongly suggests a custom application
- `wrk.service` suggests HTTP load or stress testing
- `gotty.service` suggests a browser-accessible terminal
- `ssh.service` is standard for remote management

At this point, the host looked like a small app server used for testing or lab work.

## Second Question: Which ports are open?

Next I checked listening sockets:

```bash
ss -tulpn
```

Output:

```text
Netid    State     Recv-Q    Send-Q                         Local Address:Port       Peer Address:Port   Process
udp      UNCONN    0         0                                  127.0.0.1:323             0.0.0.0:*
udp      UNCONN    0         0                                    0.0.0.0:68              0.0.0.0:*
udp      UNCONN    0         0            [fe80::46e:b4ff:fe80:70cf]%ens5:546                [::]:*
udp      UNCONN    0         0                                      [::1]:323                [::]:*
tcp      LISTEN    0         256                                  0.0.0.0:8000            0.0.0.0:*
tcp      LISTEN    0         128                                127.0.0.1:9000            0.0.0.0:*       users:(("webapp.py",pid=602,fd=3))
tcp      LISTEN    0         128                                  0.0.0.0:22              0.0.0.0:*
tcp      LISTEN    0         244                                127.0.0.1:5432            0.0.0.0:*
tcp      LISTEN    0         4096                                       *:8080                  *:*       users:(("gotty",pid=589,fd=6))
tcp      LISTEN    0         128                                     [::]:22                 [::]:*
```

### Port-by-port interpretation

#### Port 22
SSH is exposed publicly, which is expected for remote administration.

#### Port 5432
PostgreSQL is listening only on `127.0.0.1`, which is good. The database is not directly exposed to the network.

#### Port 9000
A Python process named `webapp.py` is listening only on localhost. That strongly suggests an internal backend service.

#### Port 8000
Something is exposed on all interfaces on port `8000`. This may be the public-facing side of the application or a simple service used by the exercise.

#### Port 8080
`gotty` is exposed broadly on port `8080`. That immediately deserved a closer look.

## Third Question: What do these services actually return?

I tested a few local HTTP endpoints.

### Port 80

```bash
curl http://localhost
```

Output:

```text
curl: (7) Failed to connect to localhost port 80: Connection refused
```

So there is no web server listening on the default HTTP port.

### Port 8000

```bash
curl http://localhost:8000
```

Output:

```text
33111
```

### Port 9000

```bash
curl http://localhost:9000
```

Output:

```text
34356
```

Those responses look like simple numeric outputs, possibly generated by a toy web app or a benchmarking exercise.

### Port 8080

```bash
curl http://localhost:8080
```

Output:

```html
<!doctype html>
<html>

<head>
  <title>bash@ip-10-1-12-102</title>
  <link rel="manifest" href="manifest.json" crossorigin="use-credentials">
  <link rel="icon" href="favicon.ico">
  <link rel="icon" href="icon.svg" type="image/svg+xml">
  <link rel="stylesheet" href="./css/index.css" />
  <link rel="stylesheet" href="./css/xterm.css" />
  <link rel="stylesheet" href="./css/xterm_customize.css" />
  <meta name="viewport" content="width=device-width, initial-scale=1">
</head>

<body>
  <div id="terminal"></div>
  <script src="./auth_token.js"></script>
  <script src="./config.js"></script>
  <script src="./js/gotty.js"></script>
</body>
```

This confirmed that port `8080` is serving a browser-based terminal interface.

## Investigating GoTTY

I checked the process directly:

```bash
ps aux | grep gotty
```

Output:

```text
admin        589  0.0  1.9 1525912 8952 ?        S<sl 01:31   0:00 /usr/local/gotty --permit-write --reconnect --max-connection 5 bash -l
admin     232606  0.0  0.1   5264   648 pts/1    S<+  01:46   0:00 grep gotty
```

Then I reviewed the systemd unit:

```bash
systemctl cat gotty
```

Output:

```ini
# /etc/systemd/system/gotty.service
[Unit]
Description=Gotty

[Service]
User=admin
Group=admin
ExecStart=/usr/local/gotty --permit-write --reconnect --max-connection 5 bash -l
WorkingDirectory=/home/admin
Restart=on-failure
Nice=-20

[Install]
WantedBy=multi-user.target
```

### What this means

This is the most interesting part of the server.

`gotty` is launching:

```bash
bash -l
```

through a web interface, with these important flags:

- `--permit-write` means the terminal is interactive, not read-only
- `--reconnect` allows clients to reconnect
- `--max-connection 5` allows up to 5 simultaneous clients

It is running as the `admin` user, not as root, but that is still significant. If `admin` has sudo access, then a user who reaches this terminal may be only one step away from full control of the host.

## Firewall note

I tried to check firewall status:

```bash
sudo ufw status
```

Output:

```text
sudo: ufw: command not found
```

That tells us `ufw` is not installed. It does **not** prove the host is unfiltered, but it does mean uncomplicated firewall management is not available through `ufw` on this machine.

Other firewall layers may still exist, such as:

- cloud security groups
- `iptables`
- `nftables`

## So what is this server for?

Based on the services and ports, the most likely purpose of the server is:

## A small Debian-based lab or training server used to run and test a web application

The evidence points to:

- a Python app (`webapp.py`)
- a PostgreSQL backend
- Docker and containerd
- a web stress-testing tool (`wrk`)
- a browser-based terminal (`gotty`)

This combination makes sense for a demo environment, learning lab, or lightweight development/test host.

## What is running and what is going on?

Here is the practical summary:

- `webapp.py` is listening on `127.0.0.1:9000`
- another service is exposed on `0.0.0.0:8000`
- PostgreSQL is running locally on `127.0.0.1:5432`
- SSH is exposed on port `22`
- GoTTY is exposing a live shell on port `8080`
- `wrk` is installed and running, likely for HTTP load testing
- Docker and containerd are available for container workloads

In plain language: this host looks like an app box used for development, testing, or guided operations practice.

## Are there any problems?

### 1. The biggest security concern: GoTTY on port 8080

This is the standout issue.

A browser-accessible terminal with `--permit-write` is powerful and risky. Even though it runs as `admin` rather than `root`, it still provides shell access. If the account has sudo privileges, that risk increases further.

At minimum, I would want to verify:

- whether port `8080` is restricted by cloud firewall rules
- whether GoTTY is protected by authentication
- whether the service is intended for this environment

### 2. No obvious sign of a database exposure problem

PostgreSQL is bound to localhost only, which is a good sign.

### 3. No service on port 80

That is not inherently a problem. It just tells us this host is not using a default web server configuration.

### 4. Hardware utilization is still unknown

At this stage, the service and network review is solid, but hardware utilization has not yet been measured. To fully answer CPU, memory, disk, and network questions, I would still run:

```bash
top
free -h
df -h
iostat -xz 1 3
vmstat 1 5
sar -n DEV 1 3
```

If `iostat` or `sar` are not installed, I would use:

```bash
cat /proc/meminfo
cat /proc/loadavg
ip -s link
```

## Commands I would run next

If I were continuing the review, these would be my next checks:

### CPU and load

```bash
uptime
top
ps aux --sort=-%cpu | head
```

### Memory

```bash
free -h
ps aux --sort=-%mem | head
```

### Disk

```bash
df -h
du -sh /var/* 2>/dev/null | sort -h
```

### Network

```bash
ss -s
ip -s link
```

### Docker

```bash
docker ps
docker stats --no-stream
```

### Application and service definitions

```bash
systemctl cat webapp
systemctl cat wrk
journalctl -u webapp --no-pager -n 100
journalctl -u gotty --no-pager -n 100
```

## Final Assessment

My working conclusion is:

> This Debian 11 server is a lightweight application/testing host running a Python web application, PostgreSQL, Docker tooling, a stress-testing service, and a browser-based shell through GoTTY.

The environment looks intentional and educational rather than production-hardened.

The main caution area is the exposed GoTTY service on port `8080`, especially because it is configured with interactive write access.

## Closing Thoughts

This was a good reminder that a fast server review does not need to start with complicated tooling. A handful of commands can reveal a lot:

- `systemctl`
- `ss`
- `curl`
- `ps`
- `systemctl cat`

That small set was enough to build a fairly accurate picture of the server's purpose and risk profile.

If I continue this review later, the next step will be to measure resource utilization and inspect the application service definitions in more detail.
