---
title: How I Reset a RHEL Password Using GRUB
date: 2026-10-08 17:00:00 +0800
categories: [Linux, RHEL]
tags: [rhel, linux, grub, password-reset, system-administration]
description: A practical guide to resetting a forgotten RHEL root password using GRUB and rd.break.
---

## Introduction

I recently needed to reset the password on a RHEL system without knowing the existing root password.

If you have console access to the machine and can access the GRUB boot menu, you can use the `rd.break` boot parameter to interrupt the normal boot process and reset the password.

This method is useful for RHEL lab environments and systems where you have authorized physical or console access.

> **Note:** Only use this procedure on systems you own or are authorized to administer.

## How It Works

The basic process is:

```text
GRUB
  ↓
Add rd.break
  ↓
Boot into emergency shell
  ↓
Remount /sysroot as read-write
  ↓
chroot into /sysroot
  ↓
Reset the password
  ↓
Create SELinux autorelabel flag
  ↓
Reboot
```

## 1. Reboot the RHEL System

First, reboot the RHEL machine.

When the GRUB menu appears, select the RHEL kernel you want to boot.

Instead of pressing `Enter`, press:

```text
e
```

This will open the GRUB boot entry editor.

## 2. Find the Linux Kernel Line

Look for a line beginning with something similar to:

```text
linux /vmlinuz-5.x.x-xxx.el9.x86_64
```

The exact kernel version will depend on your RHEL installation.

For example:

```text
linux /vmlinuz-5.14.0-xxx.el9.x86_64 root=/dev/mapper/rhel-root ro ...
```

Move to the end of this line.

## 3. Add `rd.break`

Add the following parameter to the end of the kernel line:

```text
rd.break
```

For example:

```text
linux /vmlinuz-5.14.0-xxx.el9.x86_64 root=/dev/mapper/rhel-root ro ... rd.break
```

Do not remove the existing parameters.

After adding `rd.break`, press:

```text
Ctrl + X
```

This will boot the system with the modified kernel parameters.

## 4. Enter the Emergency Shell

The system should stop during the early boot process and give you an emergency shell.

You will usually see a prompt similar to:

```text
switch_root:/#
```

At this point, the installed RHEL system is mounted under:

```text
/sysroot
```

However, the filesystem is mounted read-only.

## 5. Remount `/sysroot` as Read-Write

Before changing the password, remount `/sysroot` with read-write permissions:

```bash
mount -o remount,rw /sysroot
```

You can verify the mount:

```bash
mount | grep sysroot
```

The filesystem should now be mounted with read-write permissions.

## 6. Change Root into the Installed System

Next, use `chroot` to change the root directory to the installed RHEL system:

```bash
chroot /sysroot
```

Your shell prompt should now change to something similar to:

```text
sh-5.1#
```

You are now operating inside the installed RHEL environment.

## 7. Reset the Root Password

Use the `passwd` command:

```bash
passwd root
```

Enter the new password twice when prompted.

You should see something similar to:

```text
Changing password for user root.
New password:
Retype new password:
passwd: all authentication tokens updated successfully.
```

The root password has now been changed.

## 8. Create the SELinux Relabel Flag

RHEL normally uses SELinux, so it is important to make sure the files modified during this process receive the correct SELinux labels.

Create the autorelabel file:

```bash
touch /.autorelabel
```

This tells SELinux to relabel the filesystem during the next boot.

> Skipping this step can cause SELinux-related problems after the password reset.

## 9. Exit the `chroot`

First, exit the `chroot` environment:

```bash
exit
```

You should return to the emergency shell:

```text
switch_root:/#
```

Then exit the emergency shell:

```bash
exit
```

The system should continue booting or restart depending on the RHEL version and environment.

If necessary, you can also reboot from the emergency shell:

```bash
reboot
```

## 10. Wait for SELinux Relabeling

During the next boot, RHEL may perform an SELinux filesystem relabel.

This can take some time depending on the size of the filesystem.

Do not interrupt the system while this process is running.

After the relabel is complete, RHEL should continue with the normal boot process.

## 11. Test the New Password

Once RHEL has finished booting, log in using the new root password.

For example:

```text
login: root
Password: ********
```

You can also verify that the password works by switching to root from another administrative account:

```bash
su -
```

Enter the new root password when prompted.

## Complete Command Sequence

For reference, the main commands used in the process are:

```bash
mount -o remount,rw /sysroot
chroot /sysroot
passwd root
touch /.autorelabel
exit
exit
```

The important part is remembering the order:

```text
1. remount /sysroot
2. chroot /sysroot
3. passwd root
4. touch /.autorelabel
5. exit
6. exit
```

## What Is `rd.break`?

The `rd.break` parameter tells the RHEL initial RAM filesystem (`initramfs`) to stop the boot process before the normal root filesystem is switched over.

This gives you access to an early boot shell.

The installed system is then available under:

```text
/sysroot
```

This is why we need to run:

```bash
chroot /sysroot
```

Without `chroot`, commands such as `passwd` would operate against the temporary environment rather than the installed RHEL system.

## Why Do We Need `touch /.autorelabel`?

RHEL uses SELinux to enforce security policies and labels files according to their expected contexts.

When working from the early boot environment, changes can result in incorrect or missing SELinux labels.

Running:

```bash
touch /.autorelabel
```

creates a marker that tells SELinux to perform a complete filesystem relabel during the next boot.

This is especially important on RHEL systems where SELinux is enabled.

## Troubleshooting

### `passwd` says the filesystem is read-only

If you receive an error indicating that the filesystem is read-only, make sure you ran:

```bash
mount -o remount,rw /sysroot
```

before running:

```bash
chroot /sysroot
```

### `chroot /sysroot` does not work

Check that `/sysroot` is mounted:

```bash
mount | grep sysroot
```

You can also inspect the directory:

```bash
ls /sysroot
```

You should see directories such as:

```text
etc
usr
var
home
root
```

### The system takes a long time to boot

If you created:

```bash
touch /.autorelabel
```

RHEL may spend some time relabeling the filesystem.

This is expected.

The amount of time depends on the number of files on the system.

## Important Limitations

This procedure assumes that you can access the GRUB boot menu.

It may not work directly if:

- GRUB is protected with a password.
- The system uses full-disk encryption such as LUKS and you cannot unlock the disk.
- You do not have console access.
- Additional security controls prevent modifying the boot parameters.

In production environments, password recovery procedures should follow your organization's security and change-management policies.

## Conclusion

Resetting a RHEL password through GRUB is relatively straightforward when you have authorized console access to the system.

The most important commands to remember are:

```bash
mount -o remount,rw /sysroot
chroot /sysroot
passwd root
touch /.autorelabel
```

The key idea is that `rd.break` interrupts the boot process before the normal system starts, allowing you to access the installed filesystem through `/sysroot`.

Once the password has been changed and SELinux has been scheduled for relabeling, the system can be rebooted normally.