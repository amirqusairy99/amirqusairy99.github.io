---
layout: post
title: "Solving Docker Traefik Load Balancer Bad Gateway Issue"
date: 2026-05-25 02:29:00 +0000
categories: [devops, docker, traefik]
tags: [docker, traefik, load-balancer, debugging, docker-compose]
---

While troubleshooting a Traefik load balancer setup, `curl -s app.sadserver | head -n1` was intermittently returning a `Bad Gateway` instead of rotating through backend hostnames. Here's how I diagnosed and fixed it.

## Symptoms

Running the curl command three times produced inconsistent results:

```bash
$ curl -s app.sadserver | head -n1
Hostname: 2990d7e439ce   # ✅ 1st request — OK

$ curl -s app.sadserver | head -n1
Hostname: 049eb38a4bf4   # ✅ 2nd request — OK

$ curl -s app.sadserver | head -n1
Bad Gateway              # ❌ 3rd request — FAIL
```

---

## Verifying Running Containers

```bash
sudo docker ps
```

```
CONTAINER ID   IMAGE            COMMAND       STATUS          NAMES
a2f3f16b0928   traefik:v3.6.1   "..."         Up ~1 minute    app-traefik-1
2990d7e439ce   traefik/whoami   "/whoami"     Up ~1 minute    app-app03-1
049eb38a4bf4   traefik/whoami   "/whoami"     Up ~1 minute    app-app01-1
5b3c8c3c22bd   traefik/whoami   "/whoami"     Up ~1 minute    app-app04-1
2fc39d0d7412   traefik/whoami   "/whoami"     Up ~1 minute    app-app02-1
```

All four backend containers and Traefik appeared healthy.

---

## Inspecting the docker-compose.yml

```bash
cat /home/admin/app/docker-compose.yml
```

```yaml
services:
  traefik:
    image: traefik:v3.6.1
    restart: unless-stopped
    command:
      - "--api.dashboard=true"
      - "--providers.docker=true"
      - "--providers.docker.exposedbydefault=false"
      - "--entrypoints.web.address=:80"
      - "--entrypoints.websecure.address=:443"
      - "--log.level=DEBUG"
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    networks:
      - web

  app01:
    image: traefik/whoami
    restart: unless-stopped
    labels:
      traefik.enable: "true"
      traefik.http.routers.app.rule: Host(`app.sadserver`)
      traefik.http.services.app.loadbalancer.server.port: "80"
    networks:
      - web

  app02:
    image: traefik/whoami
    restart: unless-stopped
    labels:
      traefik.enable: "true"
      traefik.http.routers.app.rule: Host(`app.sadserver`)
      traefik.http.services.app.loadbalancer.server.port: "81"  # ⚠️ Wrong port!
    networks:
      - web

  app03:
    image: traefik/whoami
    restart: unless-stopped
    labels:
      traefik.enable: "true"
      traefik.http.routers.app.rule: Host(`app.sadserver`)
      traefik.http.services.app.loadbalancer.server.port: "80"
    networks:
      - web

  app04:
    image: traefik/whoami
    restart: unless-stopped
    labels:
      traefik.enable: "true"
      traefik.http.routers.app.rule: Host(`app.sadserver`)
      traefik.http.services.app.loadbalancer.server.port: "80"
    networks:
      - internal  # ⚠️ Wrong network!

networks:
  web:
  internal:
```

Two issues were immediately visible (more on `app04` below).

---

## Reading the Traefik Logs

```bash
docker compose logs -f
```

```
traefik-1 | DBG > Service selected by WRR: http://172.19.0.4:80
traefik-1 | DBG > Service selected by WRR: http://172.19.0.2:81
traefik-1 | DBG > 502 Bad Gateway error="dial tcp 172.19.0.2:81: connect: connection refused"
```

The WRR (Weighted Round Robin) load balancer selected `app02` and attempted to forward traffic to port `81` — but `traefik/whoami` only listens on port `80`.

---

## Root Cause

### Issue 1 — Wrong port on `app02`

The `traefik/whoami` image listens exclusively on port `80`. The `app02` service had its Traefik label misconfigured to route to port `81`:

```yaml
traefik.http.services.app.loadbalancer.server.port: "81"  # ❌
```

### Confirming with a manual test

From inside the Traefik container:

```bash
docker exec -it app-traefik-1 sh
/ # wget -qO- http://app-app02-1:80
Hostname: 2fc39d0d7412
IP: 172.19.0.2
...
```

Port `80` responds correctly — confirming the label was the only problem.

### Issue 2 — `app04` on the wrong network

`app04` is attached to the `internal` network instead of `web`, meaning Traefik (which is only on the `web` network) cannot reach it at all.

---

## The Fix

Update `docker-compose.yml` with two corrections:

```yaml
  app02:
    labels:
      traefik.http.services.app.loadbalancer.server.port: "80"  # ✅ was 81

  app04:
    networks:
      - web  # ✅ was internal
```

Then redeploy:

```bash
docker compose up -d
```

---

## Takeaways

- Always double-check `loadbalancer.server.port` in Traefik labels — it must match the port the container is **actually** listening on.
- Containers on a different Docker network than Traefik are silently unreachable; Traefik may still register them but fail to proxy requests.
- `--log.level=DEBUG` on Traefik is invaluable — the WRR selection logs and connection errors pinpoint the failing backend immediately.
