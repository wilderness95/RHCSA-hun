---
deleted: true
tags: [RHCSA 8 Practice Exam - I]
title: Idő beállítása - Original link
created: '2022-11-13T08:26:33.207Z'
modified: '2022-11-15T19:21:32.689Z'
---

# Idő beállítása - [Original link](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/system_administrators_guide/chap-configuring_the_date_and_time)

- Az aktuális dátum és idő, valamint a rendszer és a hardver órájának konfigurációjával kapcsolatos részletes információk megjelenítéséhez a `timedatectl` parancsot futassuk:

      [root@server1 ~]# timedatectl
                   Local time: Sun 2022-11-13 17:28:16 CET
               Universal time: Sun 2022-11-13 16:28:16 UTC
                     RTC time: Sun 2022-11-13 12:37:59
                    Time zone: Europe/Budapest (CET, +0100)
      System clock synchronized: yes
                  NTP service: active
              RTC in local TZ: yes

      Warning: The system is configured to read the RTC time in the local time zone.
             This mode cannot be fully supported. It will create various problems
             with time zone changes and daylight saving time adjustments. The RTC
             time is never updated, it relies on external facilities to maintain it.
             If at all possible, use RTC in UTC by calling
             'timedatectl set-local-rtc 0'.

## Aktuális időzóna módosítása

- Összes elérhető időzóna listázását a `timedatectl list-timezones` parancssal tudjuk megteni

- A jelenleg használt időzóna módosítását a `timedatectl set-timezone [time_zone]` parancssal tudjuk megteni

## NTP beálítása
- A rendszer automatikus szinkronizációjának engedélyezése a távoli szerverrel: `timedatectl set-ntp yes`
- Amennyiben a parancs sikertelenül lesz, ha a crony/ntp nincs telepítve
    



