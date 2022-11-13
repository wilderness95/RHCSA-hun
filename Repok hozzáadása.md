---
title: Repok hozzáadása
created: '2022-11-13T08:14:20.988Z'
modified: '2022-11-13T08:24:05.198Z'
---

# Repok hozzáadása

- dnf config-manager segítségével tudunk új repot hozzáadni:

    dnf config-manager --add-repo repository_URL

- listázzuk ki a jelenlegi repokat:
    dnf repolist

- engedélyezzük a repokat:
    dnf config-manager --enable [repoid]

- teszteljük:
    dnf update
