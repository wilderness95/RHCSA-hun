---
tags: [EX200, Linux, RHEL]
title: Change root password RHEL - original link
created: '2022-11-13T08:02:33.609Z'
modified: '2022-11-13T08:14:09.878Z'
---

# Change root password RHEL - [original link ](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/configuring_basic_system_settings/changing-and-resetting-the-root-password-from-the-command-line_configuring-basic-system-settings)

- If you are unable to log in as a non-root user or do not belong to the administrative wheel group, you can reset the root password on boot by switching into a specialized chroot jail environment.

## Procedure

Reboot the system and, on the GRUB 2 boot screen, press the e key to interrupt the boot process.

### The kernel boot parameters appear.

    load_video
    set gfx_payload=keep
    insmod gzio
    linux ($root)/vmlinuz-4.18.0-80.e18.x86_64 root=/dev/mapper/rhel-root ro crash\
    kernel=auto resume=/dev/mapper/rhel-swap rd.lvm.lv/swap rhgb quiet
    initrd ($root)/initramfs-4.18.0-80.e18.x86_64.img $tuned_initrd```

### Go to the end of the line that starts with linux.

    linux ($root)/vmlinuz-4.18.0-80.e18.x86_64 root=/dev/mapper/rhel-root ro crash\
    kernel=auto resume=/dev/mapper/rhel-swap rd.lvm.lv/swap rhgb quiet

Press Ctrl+e to jump to the end of the line.

### Add rd.break to the end of the line that starts with linux.

    linux ($root)/vmlinuz-4.18.0-80.e18.x86_64 root=/dev/mapper/rhel-root ro crash\
    kernel=auto resume=/dev/mapper/rhel-swap rd.lvm.lv/swap rhgb quiet rd.break

### Press Ctrl+x to start the system with the changed parameters.

The `switch_root` prompt appears.

### Remount the file system as writable:

    mount -o remount,rw /sysroot
The file system is mounted as read-only in the /sysroot directory. Remounting the file system as writable allows you to change the password.

### Enter the chroot environment:

    chroot /sysroot
The `sh-4.4#` prompt appears.

### Reset the root password:

    passwd
Follow the instructions displayed by the command line to finalize the change of the `root` password.

### Enable the SELinux relabeling process on the next system boot:

    touch /.autorelabel
### Exit the chroot environment:

    exit
### Exit the `switch_root` prompt:

    exit
Wait until the SELinux relabeling process is finished. Note that relabeling a large disk might take a long time. The system reboots automatically when the process is complete.
