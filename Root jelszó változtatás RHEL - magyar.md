---
title: Root jelszó változtatás RHEL - magyar
created: '2022-11-13T07:43:56.361Z'
modified: '2022-11-13T19:35:17.606Z'
---

# Root jelszó változtatás RHEL - magyar

- Ha nem tudunk root userként bejelentkezni, akkor boot alatt van lehetőségünk megváltoztatni egy speciális `chroot jail` környezetben.

- Indítsuk újra a gépet és GRUB alatt nyomjunk `e` betüt, hogy megszakítsuk a bootot. Ekkor megjelennek a kernel boot paraméterei.
- A linuxxal kezdődő sor végére menjünk `ctrl + e` és írjuk oda, hogy `rd.break`. `ctrl + x` -el a változtatott beálításokkal indítjuk a rendszert.
- Ekkor `switch_root` prompt jelenik meg.
- Remountoljuk a file rendszert írhatóként. Alapértelmezetten csak olvashatóként van a /sysroot könyvtár felcsatolva. Ezzel elérhetővé tegyük, hogy megváltoztassuk a jelszavunkat.
```mount -o remount,rw /sysroot```
- Lépjünk be a `chroot` környezetbe:
```chroot /sysroot```
- `passwd` segítségével változtassuk meg a jelszót
- Engedélyezzük a SELinux átcimkézzési folyamat a következő bootkor: ```touch /.autorelabel```
- Lépjünk ki - `exit`. Ez után a rendszer elkezdi az átcimkézést. Ez akár sokáig is eltarthat, nagy disk esetén. Ha lefutott, a rendszer automatikusan bootol.  
