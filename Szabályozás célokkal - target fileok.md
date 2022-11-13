---
title: Szabályozás célokkal - target fileok
created: '2022-11-13T20:04:01.302Z'
modified: '2022-11-13T20:15:03.949Z'
---

# Szabályozás célokkal - target fileok

- A futási szintek a systemd esetén nem léteznek, de ezeket helyettesíti a systemd esetén a cél (target). A célok speciális egységfájlok, amelyek a rendszer állapotát és a szinkronizációs pontokat írják le. A célfájlok .target kiterjesztést kapnak, hasonlóan a normál egységfájlokhoz. A célok valójában nem tesznek semmit, helyette más egységeket csoportosítanak.

- Ez a rendszer arra használható, hogy bizonyos állapotokat vegyen fel egy rendszer, más előkészítő (init) rendszerek futási szintjeihez hasonlóan. Ezeket arra használjuk, hogy bizonyos funkciók elérhetők legyenek, ahelyett, hogy az egyes egységeket külön-külön állítanám be.

- Például van egy `network.target`. Ha ez be van töltve, azt jelzi, hogy a hálózat készen áll a működésre. Ez a cél függőségként használható más célok beállításaiban. Megadható milyen viszonyban van az egyik cél a másikkal. Olyan beállításokra gondolunk mint:

    - WantedBy=
    - RequiredBy=

- A fenti direktívák után több cél is megadható szóközzel tagolva.

- Ha egy beállításban a WantedBy= beállítás után írjuk például a `network.target` célt, akkor a konfigurált cél a `network.target` céllal fog együtt indulni. Ha `network.target` cél még sem indul el, ez nem érinti az éppen konfigurált célt működését.

      WantedBy=network.target

- Ha a RequiredBy= után írjuk a `network.target` célt, a és a network nem indul el, akkor a konfigurált cél is kikapcsolásra kerül.

      RequiredBy=network.target

## Az alapértelmezett cél

- A systemd rendelkezik egy alapértelmezett céllal, amelyet akkor használ, amikor a rendszer indul (boot).

- Az alapértelmezett cél lekérdezése:

      [root@server1 ~]# systemctl get-default 
      multi-user.target

- Az új alapértelmezett cél beállítása set-default paranccsal lehetséges:

      systemctl set-default multi-user.target


## Az elérhető célok

      systemctl list-unit-files --type=target

- A futási szintekkel ellentétben több cél is lehet egyszerre aktív.

- Az összes aktív cél megjelenítése:

      systemctl list-units --type=target


## Célok elkülönítése

- Lehetőség van a egy egység (unit) minden függőségének elindítására, és minden más leállítására. Erre használható a isolate parancs. Ez hasonlít a futási szintváltáshoz.

- Ha például a `graphical.target` aktív, de közben szeretnénk minden grafikus rendszert leállítani, a `multi-user.target` elkülöníthető. A `graphical.target` függ a `multi-user.target`-től, de fordítva nem igaz.

- Az elkülönítés előtt nézzük meg, milyen függőségei vannak a `multi-user.target`-nek:

      root@server1 ~]# systemctl list-dependencies multi-user.target 
      multi-user.target
      ● ├─auditd.service
      ● ├─chronyd.service
      ● ├─crond.service
      ● ├─dbus.service
      ● ├─dnf-makecache.timer
      ...

- Az elkülönítés:

      systemctl isolate multi-user.target
