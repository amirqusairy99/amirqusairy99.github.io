---
title: "Linux Cheatsheet"
categories: ["Linux", "System"]
tags: ["administrator", "linux", "ubuntu"]
---

# Linux Command Cheat Sheet

A comprehensive reference guide for essential Linux CLI commands.

---

## 📂 File & Directory Navigation
* `pwd` - Print working directory (shows where you are).
* `ls` - List files and directories in the current folder.
* `ls -la` - List all files, including hidden ones (`.filename`), with detailed attributes.
* `cd /path/to/dir` - Change directory to a specific path.
* `cd ..` - Move up one directory level.
* `cd ~` - Go directly to your home directory.

## 🛠️ File Management
* `mkdir folder_name` - Create a new directory.
* `touch file.txt` - Create an empty file or update timestamps.
* `cp source.txt dest.txt` - Copy a file.
* `cp -r source_dir dest_dir` - Copy a directory and its contents recursively.
* `mv file.txt /new/path/` - Move or rename a file.
* `rm file.txt` - Remove/delete a file.
* `rm -rf folder_name` - Force remove a directory and everything inside it (**use with caution**).

## 👁️ Viewing & Text Processing
* `cat file.txt` - Display the entire contents of a file.
* `less file.txt` - View file content with page-by-page scrolling.
* `head -n 10 file.txt` - View the first 10 lines of a file.
* `tail -n 10 file.txt` - View the last 10 lines of a file.
* `tail -f file.log` - Follow a log file in real-time as it updates.
* `grep "pattern" file.txt` - Search for a specific string inside a file.

## 🔒 Permissions & Ownership
* `chmod 755 file.sh` - Change file permissions (755 allows execution).
* `chmod +x script.sh` - Make a script file executable.
* `chown user:group file.txt` - Change the owner and group of a file.
* `sudo command` - Run a command with administrative/root privileges.

## 🖥️ System Information & Processes
* `uname -a` - Display complete system architecture and kernel info.
* `df -h` - Show available disk space on mounted filesystems in human-readable format.
* `free -m` - Check free and used RAM in Megabytes.
* `top` or `htop` - Monitor live system resources and running processes.
* `ps aux` - Snapshot of all currently running processes.
* `kill PID` - Terminate a process using its Process ID (PID).

## 🌐 Networking
* `ping host` - Send ICMP echo requests to test network connectivity.
* `curl -I https://example.com` - Fetch HTTP web headers.
* `wget url` - Download a file directly from the internet.
* `ip a` - Display all network interfaces and local IP addresses.
* `ssh user@host` - Connect to a remote machine securely over SSH.

## 📦 Archiving & Compression
* `tar -czvf archive.tar.gz /dir` - Compress a directory into a `.tar.gz` file.
* `tar -xzvf archive.tar.gz` - Extract a compressed `.tar.gz` file.
* `zip -r folder.zip folder` - Create a `.zip` file from a directory.
* `unzip folder.zip` - Extract a `.zip` archive.
