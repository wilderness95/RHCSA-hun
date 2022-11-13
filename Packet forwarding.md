---
title: Packet forwarding
created: '2022-11-13T19:38:22.355Z'
modified: '2022-11-13T19:51:31.077Z'
---

# Packet forwarding

- Ellenőrizzük, hogy engedélyezve van-e a packet forwarding:

      [root@server1 ~]# sysctl net.ipv4.ip_forward
      net.ipv4.ip_forward = 0

- Amennyiben kivan kapcsolva engedélyezzük:

      [root@server1 ~]# vi /etc/sysctl.conf 
      [root@server1 ~]# sysctl -p
      net.ipv6.conf.all.disable_ipv6 = 0
      net.ipv6.conf.lo.disable_ipv6 = 0
      net.ipv4.ip_forward = 1
      [root@server1 ~]# sysctl net.ipv4.ip_forward
      net.ipv4.ip_forward = 1
      [root@server1 ~]# 


