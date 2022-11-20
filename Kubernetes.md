---
tags: [Kubernetes, Udemy]
title: Kubernetes
created: '2022-11-19T20:45:08.746Z'
modified: '2022-11-19T23:07:14.917Z'
---

# Kubernetes

## bevezetés

- Az alkalmazásunk már konténerbe csomagolva futtatjuk. Abban az esetben, ha az alkalmazást éles környezetbe kiakarjuk tenni, vagy ha a konténerünk más konténertől függ, vagy a megnövekedett használat miatt fel kell skálázni, terhelés csökkenésekor leskálázni, szükségünk van egy olyan mögöttes paltforma, ami ad nekünk ilyen funkciókat. Ennek a platformnak kell megoldani az összeköttetést a konténerek közt, és terhelés függvénynében skálázza az alkalmazást. Ezt a folyamatot hívjuk orcestrálásnak

### orchestration rendszerek
- A Kubernetes tehát egy konténer-orchestrációs technológia. Több ilyen technológia áll ma rendelkezésre.
- A Dockernek saját eszköze van, a `Docker Swarm`. A `Kubernetest` a Google, a `Mesost` az Apache fejlesztette. A `Docker Swarm`-nak egyszerű a telepítése és az indítása, viszont hiányzik belőle néhány fejlett automatikus skálázási funkció, ami szükséges a komplex alkalmazásokhoz.
- A `Mesost` elég nehéz beállítani és beüzemelni, de számos fejlett funkciót támogat. 
- A `Kubernetes` vitathatatlanul a leginkább népszerűbb mind közül - kissé nehezen telepíthető és indítható, de számos lehetőséget kínál a telepítések testreszabásához, és támogatja az összetett architektúrák telepítését.
-  A `Kubernetes` ma már minden nyilvános felhőszolgáltatónál támogatott, mint például a GCP, az Azure és az AWS. A `kubernetes` projekt pedig a Github egyik legjobban rangsorolt projektje.

### orchestration előnyei

- Az alkalmazás mostantól nagy rendelkezésre állásu, mivel a hardverhibák nem befolyásolják le az alkalmazást, mert az alkalmazás több példánya fut különböző nodeokon. - A load balancing a különböző konténerek között történik. Amikor az igény megnő, az alkalmazás több példányát zökkenőmentesen és másodpercek alatt indíthatjuk és ezt szolgáltatási szinten is megtehetjük. 
- Ha kifogyunk a hardveres erőforrásokból, a node-ok számát felfelé/lefelé skálázhatjuk anélkül, hogy le kellene állítani az alkalmazást. 

## Kubernetes architektúra

  ### node
  - A node egy olyan - fizikai vagy virtuális - gép, amelyre a kubernetes telepítve van. A node egy worker, és itt fognak indulni a konténerek a kubernetes által.
  - Amennyiben csak egy node fut és meghibásodik, úgy az alkalmazásunk leáll. Következés képpen egynél több node-ra van szükségünk
  ### cluster
  - A cluster a node-ok csoportosított halmaza. Így még ha az egyik node meghibásodik is, akkor is elérhető marad az alkalmazás a többi node-ról Ráadásul a több node    segít a terhelés megosztásában is
  -  A master egy másik node, amelyre a Kubernetes telepítve van, és amelyet masterré konfiguráltak. A master felügyeli a csomópontokat a clusterben, és felelős a   konténerek tényleges orchestrálásáért a worker node-kon. 
  - A masteren tárolódnak a cluster tagjaira vonatkozó információk. Itt folyik a worker node-ok monitorozása, skálázása és a loadbalance.
  ## komponensek
  - Amikor Kubernetes-t telepítünk, valójában a következő komponenseket telepítünk:
      - API-server
      - ETCD service
      - kubelet service
      - Container runtime
      - Controllers
      - Schedulers 
  ### Api-server
  - Az API szerver a kubernetes front-endjeként működik. A felhasználók, a menedzsment eszközök, a  CLI mind az API-kiszolgálóval kommunikálnak, hogy interakcióba   lépjenek a kubernetes clusterrel.
  ### ETCD service
  - Az ETCD egy elosztott, megbízható kulcs-érték tároló, amelyet a kubernetes a cluster kezeléséhez használt összes adat tárolására alkalmaz. Amennyiben több node és   több master van a clusterben, az etcd elosztott módon tárolja az összes információt a cluster összes node-ján. Az ETCD felelős a zárak megvalósításáért a clusteren  belül, hogy a masterek között ne legyen konfliktus. 
  ### Scheduler
  - Az ütemező felelős a munka vagy a konténerek elosztásáért a nodeok között. Megkeresi az újonnan létrehozott konténereket, és hozzárendeli őket a node-okhoz.
  ### Controllers
   - Az controllerek az orkesztráció mögött álló logika. Ezek afelelősek azért, hogy észleljék és reagáljanak, ha a node-ok, konténerek vagy végpontok leállnak. Ilyen   esetekben a controllerek döntéseket hoznak az új konténerek indításáról.
   ### Container-runtime
   -  A Container-runtime a konténerek futtatásához használt mögöttes szoftver.Pl: docker, containerd, CRI-O, stb
   ### Kubelet service
   -  A kubelet az az szolgáltatás, amely a cluster minden egyes node-ján fut. Ez az agent felelős azért, hogy a konténerek az elvárásoknak megfelelően fussanak a   node-kon.

 ## Master vs Worker

  | Master               | Worker              | 
  | ------               | ---                 | 
  |  kube-apiserver      |  kubelet            | 
  | etcd                 | •                   | 
  | Controller           | •                   | 
  | Scheduler            | •                   | 
  | •                    | Container runtime   | 

 - A worker node (vagy más néven minion) az, ahol a konténerek vannak hostolva. Ahhoz, hogy ezek a konténereket futtassuk, szükségünk van egy telepített Container runtime-ra. 
 - A master node rendelkezik a kube-apiserverrel, és ez teszi masterré. Hasonlóképpen a worker nodeokk rendelkeznek a kubelet-ügynökkel, amely a masterrel való interakcióért felelős, hogy a worker nodeok állapotáról információt szolgáltasson, és a master által kért műveleteket végre hajtsa a worker node-kon
 - Az összes összegyűjtött információt a master egy kulcsérték-tárolóban tárolja. A kulcs-értéktároló a népszerű etcd keretrendszeren alapul. A masterben található a Controller és a Scheduler is.





