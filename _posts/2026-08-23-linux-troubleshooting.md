---
title: "Linux DNS Troubleshooting Guide: From Local Config to Upstream Resolution"
date: 2026-08-20 14:10:00 +0800
categories: [Sysadmin, Networking]
tags: [dns, linux, networking, troubleshooting, systemd-resolved, dig, nsswitch]
description: "A comprehensive guide and cheat sheet for diagnosing Linux DNS resolution failures, NSS configuration bugs, systemd-resolved quirks, and performance issues."
toc: true
comments: true
math: false
---

## Overview

DNS issues in Linux environments can stem from a variety of sources: local configuration files (`/etc/hosts`, `/etc/nsswitch.conf`), system resolvers (`systemd-resolved`), firewall blocks on UDP/TCP port 53, or upstream domain/DNSSEC failures. 

This guide breaks down common DNS symptoms, explains why they happen, and provides direct troubleshooting workflows formatted specifically for Jekyll Chirpy sites.

---

## Quick Debugging Workflow

When faced with a DNS issue, follow this systematic 3-step investigation process:

### 1. Compare NSS Path vs. Raw DNS
Applications use Name Service Switch (NSS) via `getaddrinfo()`, whereas `dig` directly queries configured DNS servers. Compare the results:

```bash
# Check NSS resolution path (used by applications)
getent hosts api.example.com

# Query DNS server directly
dig +short api.example.com

# Check for hardcoded local overrides
grep api /etc/hosts