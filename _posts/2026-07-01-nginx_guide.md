---
title: "Nginx Guide"
categories: ["Linux","Web Servers"]
tags: ["nginx","web servers"]
---

# Nginx Guide

## What Nginx Does in Production

Nginx handles HTTP and HTTPS connections using an **event-driven architecture**, allowing it to efficiently manage thousands of concurrent connections with low resource usage.

Common production use cases include:

- Serving static files directly from disk
- Reverse proxying requests to application servers such as:
  - Gunicorn
  - uWSGI
  - Node.js
- Load balancing across multiple backend servers
- TLS/SSL termination
- Gzip compression
- Response caching
- Rate limiting

Most of these features are configured using plain-text configuration files without requiring additional modules.

---

# How a Request Is Handled

When a client sends a request, Nginx processes it through several stages.

```text
Client
   │
   ▼
TCP Accept
   │
   ▼
TLS Handshake (HTTPS only)
   │
   ▼
Server Block Selection
   │
   ▼
Location Matching
   │
   ▼
Request Handler
   │
   ├── Static File
   ├── Reverse Proxy
   └── Redirect
   │
   ▼
Response Sent
   │
   ├── access_log
   └── error_log
```

### 1. TCP Accept

A worker process accepts the incoming connection on a configured `listen` socket.

### 2. TLS Handshake

If the request uses HTTPS, Nginx selects the appropriate SSL certificate using **SNI (Server Name Indication)**.

### 3. Server Block Matching

Nginx selects the appropriate `server {}` block based on:

- `listen`
- `server_name`
- `default_server`

If no hostname matches, the **default server** for that port is used (typically the first server block defined).

### 4. Location Matching

Within the selected server block, Nginx determines which `location` should process the request.

Possible location types include:

- Exact match (`=`)
- Prefix match
- Regular expression (`~`, `~*`)

### 5. Request Handler

The matched location performs one of several actions:

- Serve a static file
- Reverse proxy the request
- Redirect the client
- Return an error

### 6. Response

The completed response is returned to the client.

- Successful requests are recorded in `access_log`
- Errors are written to `error_log`

---

# Configuration Layout

The configuration directory depends on the operating system.

## Debian / Ubuntu

```text
/etc/nginx/
├── nginx.conf
├── sites-available/
├── sites-enabled/
└── snippets/
```

`nginx.conf` usually includes all files inside `sites-enabled/`.

To enable a site:

```bash
ln -s /etc/nginx/sites-available/example.conf \
      /etc/nginx/sites-enabled/
```

---

## RHEL / CentOS / Rocky Linux

```text
/etc/nginx/
├── nginx.conf
└── conf.d/
    ├── app.conf
    └── ssl.conf
```

The main configuration typically includes:

```nginx
include /etc/nginx/conf.d/*.conf;
```

---

## Snippets

Shared configuration fragments are commonly stored in:

```text
/etc/nginx/snippets/
```

Examples include:

- SSL settings
- Proxy headers
- Security headers

---

# Testing Configuration

Always validate the configuration before reloading Nginx.

Test syntax:

```bash
sudo nginx -t
```

Reload without disconnecting clients:

```bash
sudo systemctl reload nginx
```

Restart completely:

```bash
sudo systemctl restart nginx
```

---

# Key Directives

## server_name

Defines which hostnames a server block responds to.

```nginx
server_name example.com www.example.com;
```

---

## root

Defines the document root for static files.

```nginx
root /var/www/html;
```

---

## alias

Maps a location directly to another directory.

```nginx
location /images/ {
    alias /data/images/;
}
```

Unlike `root`, `alias` replaces the matched location path.

---

## proxy_pass

Forwards requests to another server.

```nginx
location / {
    proxy_pass http://127.0.0.1:3000;
}
```

---

## upstream

Creates a named backend pool for load balancing.

```nginx
upstream backend {
    server 10.0.0.10;
    server 10.0.0.11;
}
```

Used together with:

```nginx
proxy_pass http://backend;
```

---

## try_files

Attempts multiple paths before returning an error.

Typical SPA configuration:

```nginx
location / {
    try_files $uri $uri/ /index.html;
}
```

---

## client_max_body_size

Sets the maximum upload size.

```nginx
client_max_body_size 100M;
```

If exceeded, Nginx returns:

```text
413 Request Entity Too Large
```

---

## include

Imports another configuration file.

```nginx
include snippets/proxy.conf;
```

---

# Location Matching Priority

Nginx evaluates locations in a specific order.

| Priority | Type | Example |
|----------|------|---------|
| 1 | Exact Match | `location = /login` |
| 2 | Longest Prefix | `location /api/` |
| 3 | Regex (definition order) | `location ~ \.php$` |
| 4 | General Prefix | `location /` |

A common mistake is allowing a broad `location /` block to intercept requests that should be handled by a more specific location such as:

```nginx
location /api/ {
    proxy_pass http://backend;
}
```

Always make specific locations more precise than general ones.

---

# Useful Commands

View configuration:

```bash
nginx -T
```

Check syntax:

```bash
nginx -t
```

Reload configuration:

```bash
systemctl reload nginx
```

Restart Nginx:

```bash
systemctl restart nginx
```

View status:

```bash
systemctl status nginx
```

Follow access log:

```bash
tail -f /var/log/nginx/access.log
```

Follow error log:

```bash
tail -f /var/log/nginx/error.log
```

---

# Learning Resources

- Official Nginx Documentation  
  https://nginx.org/en/docs/

- Server Name Selection  
  https://nginx.org/en/docs/http/server_names.html

- Reverse Proxy Module  
  https://nginx.org/en/docs/http/ngx_http_proxy_module.html

- Nginx Beginner's Guide  
  https://nginx.org/en/docs/beginners_guide.html