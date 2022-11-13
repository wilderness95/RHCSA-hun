---
title: Másodlagos IP hozzáadása
created: '2022-11-13T17:11:51.233Z'
modified: '2022-11-13T17:18:33.972Z'
---

# Másodlagos IP hozzáadása


- A jelenlegi háló kártyák ki listázása: `nmcli connection show`

      [root@server1 ~]# nmcli connection show
      NAME         UUID                                  TYPE      DEVICE 
      System eth0  5fb06bd0-0bb0-7ffb-45f1-d6edd65f3e03  ethernet  eth0   
      System eth1  9c92fad9-6ecb-3e6c-eb4d-8a47c6f50c04  ethernet  eth1  

- Másodlagos IP hozzáadása - IPv4:

      nmcli connection modify System\ eth1 +ipv4.address 10.0.0.5/24

- Másodlagos IP hozzáadása - IPv6:

      nmcli connection modify System\ eth1 +ipv6.method manual ipv6.addresses fd01::100/64

- Majd: 

      nmcli connection reload 

