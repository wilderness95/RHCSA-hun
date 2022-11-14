---
tags: [kubernetes, openshift]
title: Kubernetes és az openshift áttekintése
created: '2022-11-14T10:20:22.408Z'
modified: '2022-11-14T11:21:20.646Z'
---

# Kubernetes és az openshift áttekintése

- A konténerek egyszereű megoldást nyújtanak az alkalmazások csomagolására és futtatására. A konténerek számának növekedése exponenciálisan növeli a velük járó munkát is (kézi indítás, skálázás, külső igényekre való reakció)

- A konténerekkel szembeni igények produktív környezetben:
    - könnyű kommunikáció a szolgáltatások közt
    - alkalmazások erőforrás korlátai konténer számtól függetlenü
    - gyors skálázás
    - reakció a szolgáltatás romlására
    - új release fokozatos bevezetése

- A kubernetes egy konténer orkesztrációs szolgáltatás, mely leegyszerűsíti a konténeres alkalmazások telepítését, kezelését és skálázását.

- A kubernetesben kezelehető legkisebb egység a pod. A pod egy vagy több konténerből áll, amelynek a storage, valamint IP-e egyetlen alkalmazást képvisel.
