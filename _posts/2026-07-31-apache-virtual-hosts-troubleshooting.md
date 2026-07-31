---
title: "Troubleshooting Apache Virtual Hosts: A SadServers Journey"
date: 2026-07-31
tags: ["apache", "linux", "troubleshooting", "devops"]
categories: ["System Administration"]
author: "Your Name"
description: "A step-by-step guide to resolving Apache virtual host configuration issues and 404 errors"
---

# Troubleshooting Apache Virtual Hosts: A SadServers Journey

## The Problem

I recently encountered an interesting challenge while working on a SadServers scenario. The server had Apache configured with multiple virtual hosts, and the `/reports` endpoint was returning a 404 error when it should have displayed "SadServers - Status OK". Additionally, the root path `/` was returning a 403 Forbidden error.

## Initial Investigation

The first step was to explore the file structure:

```bash
cd /var/www
ls -la
# Found: html  portal

cd portal/status
cat index.html
# Output: SadServers - Status OK
```

I discovered that the status/index.html file existed and contained the correct content, but it wasn't being served properly.

## Checking Apache Logs

The error logs provided the first clue:

```bash
tail -n 20 /var/log/apache2/portal-error.log
```

The logs showed:

```text
AH01276: Cannot serve directory /var/www/portal/: No matching DirectoryIndex found
```

This indicated that Apache was trying to serve the /var/www/portal/ directory but couldn't find an index file.

## Analyzing Apache Configuration

Checking the virtual host configurations revealed the issue:

```bash
ls /etc/apache2/sites-available/
# Output: 000-default.conf  default-ssl.conf  portal.conf

cat portal.conf
```

The portal.conf file showed:

```apache
<VirtualHost *:80>
    ServerName localhost
    DocumentRoot /var/www/portal
    # ... other configuration
</VirtualHost>
```

## The Solution

The problem had two parts:

### 1. Missing Index File for Root Path

The root URL / was returning 403 because there was no index.html in /var/www/portal/. I created one:

```bash
echo "SadServers - Portal Ready" | sudo tee /var/www/portal/index.html
```

### 2. Missing Alias for /reports Endpoint

The /reports endpoint wasn't mapped to the status directory. I added an Alias directive to portal.conf:

```apache
<VirtualHost *:80>
    ServerName localhost
    DocumentRoot /var/www/portal

    # Alias for /reports
    Alias /reports /var/www/portal/status

    <Directory /var/www/portal>
        Options Indexes FollowSymLinks
        AllowOverride None
        Require all granted
    </Directory>

    <Directory /var/www/portal/status>
        Options Indexes FollowSymLinks
        AllowOverride None
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/portal-error.log
    CustomLog ${APACHE_LOG_DIR}/portal-access.log combined
</VirtualHost>
```

### 3. Ensuring Proper Site Configuration

I verified that the correct virtual host was enabled:

```bash
sudo a2dissite 000-default.conf  # Disable default site
sudo a2ensite portal.conf        # Enable portal site
sudo systemctl reload apache2    # Reload configuration
```

## Testing the Solution

After applying the changes, I tested both endpoints:

```bash
curl http://localhost/
# Output: SadServers - Portal Ready

curl http://localhost/reports
# Output: SadServers - Status OK
```

## Verification Script

The SadServers environment included a verification script that confirmed the fix:

```bash
cat ~/agent/check.sh
# The script checks both endpoints and outputs "OK" when successful
./check.sh
# Output: OK
```

## Key Takeaways

1. **Always check Apache logs first** - They provide specific error messages that guide troubleshooting
2. **DocumentRoot requires an index file** - Apache needs a DirectoryIndex file (like index.html) to serve directory listings
3. **Alias is useful for mapping URLs** - Use Alias to map specific URL paths to directories outside DocumentRoot
4. **Virtual host ordering matters** - Disable conflicting virtual hosts to ensure the correct one handles requests
5. **Directory permissions are crucial** - Each directory needs proper <Directory> directives for access control

## Additional Resources

- Apache Virtual Host Documentation
- Apache Alias Directive
- Apache Directory Directive

## Conclusion

This SadServers challenge was a great reminder that web server configuration requires attention to detail. The combination of proper virtual host setup, correct directory directives, and appropriate aliases is essential for serving content correctly.

The key lesson: when a URL path doesn't map directly to the filesystem structure, you need explicit configuration (like Alias) to tell Apache how to handle it.
