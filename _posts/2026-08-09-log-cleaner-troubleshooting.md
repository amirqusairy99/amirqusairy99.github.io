---
title: "Troubleshooting a Linux Log Cleaner systemd Service"
date: 2026-08-09 11:00:00 +0800
categories: [Linux, System Administration]
tags: [linux, systemd, bash, troubleshooting, logs]
---

# Troubleshooting a Linux Log Cleaner systemd Service

This guide documents the troubleshooting process for a `log-cleaner.service` systemd service used to remove old application log files from `/var/log/app`.

## 1. Check the systemd service

First, inspect the service configuration:

```bash
sudo systemctl cat log-cleaner.service
```

The service configuration was:

```ini
[Unit]
Description=Daily Log Cleaner

[Service]
Type=oneshot
ExecStart=/bin/bash /opt/scripts/log-cleaner.sh
WorkingDirectory=/root
```

The important part is:

```ini
ExecStart=/bin/bash /opt/scripts/log-cleaner.sh
```

This tells us that systemd executes:

```text
/opt/scripts/log-cleaner.sh
```

## 2. Inspect the cleanup script

The script can be viewed with:

```bash
cat /opt/scripts/log-cleaner.sh
```

The original script was:

```bash
#!/bin/bash

LOG_DIR="/var/log/app"
DAYS=7

echo "Starting Cleanup..."

find . -maxdepth 1 -name "*.log" -type f -mtime -7 -print -delete

echo "Cleanup finished."
```

There were two issues with this script.

## 3. Issue: Wrong directory

Although the script defined:

```bash
LOG_DIR="/var/log/app"
```

the variable was never used.

Instead, the `find` command used:

```bash
find .
```

The `.` represents the current working directory.

Therefore, if the script was executed from `/home/admin`, it searched:

```text
/home/admin
```

instead of:

```text
/var/log/app
```

### Fix

Change:

```bash
find .
```

to:

```bash
find "$LOG_DIR"
```

## 4. Issue: Incorrect `mtime` condition

The next issue was the use of:

```bash
-mtime -7
```

The `-mtime` option controls file age.

### `-mtime -7`

This matches files modified less than 7 days ago.

Therefore, a recent log file such as:

```text
recent_data.log
```

could be deleted.

### `-mtime +7`

This matches files modified more than 7 days ago.

Therefore, an old log file such as:

```text
old_data.log
```

would be selected for deletion.

For a log cleaner that removes old logs, the correct condition is:

```bash
-mtime +7
```

## 5. Final cleanup script

The corrected script is:

```bash
#!/bin/bash

LOG_DIR="/var/log/app"
DAYS=7

echo "Starting Cleanup in $LOG_DIR..."

find "$LOG_DIR" -maxdepth 1 -name "*.log" -type f -mtime +$DAYS -print -delete

echo "Cleanup finished."
```

The script now:

1. Searches `/var/log/app`.
2. Only checks files directly inside the directory.
3. Only matches `.log` files.
4. Only matches regular files.
5. Deletes files older than 7 days.
6. Prints each file before deleting it.

## 6. Fix the Bash shebang

During testing, the following error appeared:

```text
/opt/scripts/log-cleaner.sh: 1: !/bin/bash: not found
```

The first line was incorrectly written as:

```bash
!/bin/bash
```

The correct Bash shebang is:

```bash
#!/bin/bash
```

The `#` is required.

## 7. Make the script executable

Set the executable permission:

```bash
sudo chmod +x /opt/scripts/log-cleaner.sh
```

Verify it:

```bash
ls -l /opt/scripts/log-cleaner.sh
```

The permissions should include `x`, for example:

```text
-rwxr-xr-x
```

## 8. Test before deleting

Before running the deletion command, test which files match:

```bash
find /var/log/app -maxdepth 1 -name "*.log" -type f -mtime +7 -print
```

This command only prints matching files.

It does not delete anything.

For example:

```text
/var/log/app/old_data.log
```

If `recent_data.log` is newer than 7 days, it should not appear.

## 9. Run the script manually

Run:

```bash
sudo /opt/scripts/log-cleaner.sh
```

Expected output:

```text
Starting Cleanup in /var/log/app...
/var/log/app/old_data.log
Cleanup finished.
```

Then verify the directory:

```bash
ls -l /var/log/app
```

The expected result is:

```text
recent_data.log
```

The old log file should have been removed.

## 10. Run the service through systemd

The service can be started manually with:

```bash
sudo systemctl start log-cleaner.service
```

Check its status:

```bash
sudo systemctl status log-cleaner.service
```

Because the service uses:

```ini
Type=oneshot
```

it is expected to complete and become inactive after the script finishes successfully.

## 11. Check systemd logs

Use `journalctl` to inspect the service output:

```bash
sudo journalctl -u log-cleaner.service -n 20 --no-pager
```

This is useful for determining whether the script ran successfully and which files were removed.

## 12. Check for a systemd timer

If the service is supposed to run automatically, check whether a timer exists:

```bash
systemctl list-timers | grep log-cleaner
```

If a timer exists, inspect it:

```bash
sudo systemctl cat log-cleaner.timer
```

Check its status:

```bash
sudo systemctl status log-cleaner.timer
```

## 13. Troubleshooting summary

| Problem | Cause | Solution |
| --- | --- | --- |
| No logs were deleted | `find .` searched the wrong directory | Use `find "$LOG_DIR"` |
| Recent logs were deleted | Used `-mtime -7` | Use `-mtime +7` |
| Old logs were not deleted | File age condition was reversed | Use `-mtime +7` |
| `!/bin/bash: not found` | Invalid shebang | Use `#!/bin/bash` |
| Script cannot execute | Missing execute permission | Use `chmod +x` |
| Unknown script location | `ExecStart` was not checked | Use `systemctl cat` |
| Need to see service output | Service logs were not checked | Use `journalctl` |

## 14. Useful commands

### View the service

```bash
sudo systemctl cat log-cleaner.service
```

### View the script

```bash
cat /opt/scripts/log-cleaner.sh
```

### Check permissions

```bash
ls -l /opt/scripts/log-cleaner.sh
```

### Test matching files

```bash
find /var/log/app -maxdepth 1 -name "*.log" -type f -mtime +7 -print
```

### Run the script

```bash
sudo /opt/scripts/log-cleaner.sh
```

### Start the service

```bash
sudo systemctl start log-cleaner.service
```

### Check service status

```bash
sudo systemctl status log-cleaner.service
```

### View service logs

```bash
sudo journalctl -u log-cleaner.service -n 20 --no-pager
```

### Check timers

```bash
systemctl list-timers | grep log-cleaner
```

## Conclusion

The main problems were the incorrect `find` directory and reversed `mtime` condition.

The final cleanup command is:

```bash
find "$LOG_DIR" -maxdepth 1 -name "*.log" -type f -mtime +$DAYS -print -delete
```

This removes `.log` files older than 7 days while keeping recent log files.