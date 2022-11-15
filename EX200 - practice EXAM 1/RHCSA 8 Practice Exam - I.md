---
tags: [RHCSA 8 Practice Exam - I]
title: RHCSA 8 Practice Exam - I
created: '2022-11-13T20:33:17.960Z'
modified: '2022-11-13T20:48:29.885Z'
---

# RHCSA 8 Practice Exam - I

## Exam
Ensure all the tasks are implemented with firewalld and SELinux enabled. Your server should be able to survive a reboot. Good luck!

1. Interrupt the boot process and reset the root password. Change it to “wander” to gain access to the system.

2. Repos are available from the repo server at http://repo.eight.example.com/BaseOS and and http://repo.eight.example.com/AppStream for you to use during the exam.

3. The system time should be set to your (or nearest to you) timezone and ensure NTP sync is configured.

4. Add the following secondary IP addresses statically to your current running interface. Do this in a way that doesn’t compromise your existing settings:
    1. IPV4 - 10.0.0.5/24
    2. IPV6 - fd01::100/64

5. Enable packet forwarding on system1. This should persist after reboot.

6. System1 should boot into the multiuser target by default and boot messages should be present (not silenced).

7. Create a new 2GB volume group named "vgprac"

8. Create a 500MB logical volume named "lvprac" inside the "vgprac" volume group.

9. The "lvprac" logical volume should be formatted with the xfs filesystem and mount persistently on the /mnt/lvprac directory

10. Extend the XFS filesystem on "lvprac" by 500MB

11. Use the appropriate utility to create a 5TiB thin provisioned volume.

12. Configure a basic web server that displays "Welcome to the web server" once connected to it. Ensure the firewall allows the http/https services.

13. Find all files that are larger then 5MB in the /etc directory and copy them to /find/largefiles

14. Write a script named awesome.sh in the root directory on system1.
    1. If "me" is given as an argument, then the script should output "Yes, I'm awesome"
    2. If "them" is given as an argument, then the script should output "Okay, they are awesome"
    3. If the argument is empty or anything else is given, the script should output “Usage ./awesome.sh me|them”

15. Create users phil, laura, stewart, and kevin.
    1. All new users should have a file named “Welcome” in their home folder after account creation.
    2. All user passwords should expire after 60 days and be atleast 8 characters in length.
    3. phil and laura should be part of the “accounting” group. If the group doesn’t already exist, create it.
    4. stewart and kevin should be part of the “marketing” group. If the group doesn’t already exist, create it.

16. Only members of the accounting group should have access to the “/accounting” directory. Make laura the owner of this directory. Make the accounting group the group owner of the “/accounting” directory.

17. Only members of the marketing group should have access to the “/marketing” directory. Make stewart the owner of this directory. Make the marketing group the group owner of the “/marketing” directory.

18. New files should be owned by the group owner and only the file creator should have the permissions to delete their own files.

19. Create a cron job that writes “This practice exam was easy and I’m ready to ace my RHCSA” to /var/log/messages at 12pm only on weekdays.

