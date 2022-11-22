---
tags: [Kubernetes, Udemy]
title: Kubernetes
created: '2022-11-19T20:45:08.746Z'
modified: '2022-11-22T13:44:54.637Z'
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

 ## Pod

 - A kubernetes segítségével a végső célunk az, hogy deployoljuk a alkalmazásunkat konténerként gépekre, amik worker node-nak vannak konfigurálva a clusterben
 - A kubernetes azonban nem telepít közvetlenül konténereket a worker node-okra. 
 - A konténerek egy Kubernetes objektumba vannak zárva, amit `POD`-nak nevezünk. Egy POD az alkalmazás egyetlen példánya. A POD a legkisebb
objektum, amit a kubernetesben létrehozhatunk.
 - Nézzünk egy egyszerű példát. Az alkalmazás egyetlen példánya egy 1 node-os kubernetes clusterben, egy POD-ba zárt docker konténerben fut. 
 Abban az esetben ha skálznod kell az alkalmazást, akkor új POD-ot hozunk létre amiben ugyannak az alkalmazásnak egy új példánynával. Így jelenleg az 1 worker node-unkon 2 külön POD-ban futnak az alkalmazás példányaink. Abban az esetben, ha a node kifogy a hardveres erőforrásból, úgy a clusterhez egy új worker node-ot adunk. Az új node-on is indítunk egy vagy több POD-ot az alkalmazásunk új példányaival.

### Több konténeres POD-ok
- Általában a POD-ok egy az egyben kapcsolatban állnak a konténerekkel, viszont egyetlen POD-ban akár több konténer is lehet, csakhogy ezek általában nem azonos típusu konténerek.
- Előfordulhat, hogy az alkalmazásunknak szüksége van egy segédkonténerre, például a webalkalmazásunk mellett fut egy feldolgozó, ami például feldolgozza a felhasználó által bevitt adatokat, feltöltött fájlt stb. és azt szeretnénk, ha ezek a segédkonténerek az alkalmazáskonténerünk mellett élnének.
- Ebben az esetben mindkét konténer része lehet ugyanannak a POD-nak, így amikor egy új alkalmazáskonténer jön létre, a segédkonténer is létrejön, és amikor az meghal, a segédkonténer is meghal, mivel mindkettő ugyanannak a POD-nak a része.
- A két konténer közvetlenül is tud egymással kommunikálni, úgy, hogy 'localhost'-ként hivatkoznak egymásra, mivel ugyanazt a hálózati névteret használják. Ráadásul könnyedén megoszthatják ugyanazt a tárhelyet is.
- Azonban a multi konténeres POD-ok elég ritkák.

## YAML a kubernetesben

      apiVersion: v1
      kind: Pod
      metadata:
        name: myapp-pod
        labels:
          app: myapp
      spec:
        containers:
         - name: nginx-container
           image:nginx

- A Kubernetes YAML fájlokat használja bemenetként olyan objektumok létrehozásához, mint a POD-ok, replicas, deployments, services stb. 
- Ezek mindegyike hasonló struktúrát követ. Egy kubernetes definíciós fájl mindig 4 felső szintű fieldet tartalmaz.
- Az apiVersion, kind, metadata és spec. Ezek a legfelső szintű vagy gyökérszintű tulajdonságok. Ezek mind kötelező mezők, tehát a konfigurációs fájlban benne kell, hogy legyenek. 

### apiVersion
- Ez a kubernetes API verziója, amit az objektum létrehozásához használunk.  Attól függően, hogy mit akarunk létrehozni, a JÓ apiVersiont kell használnunk.
- Néhány lehetséges API version:
    - v1
    - apps/v1
    - batch/v1
    - extensions/v1beta1
### kind
- A kind-al definiáljuk, hogy milyen típusú objektumot próbálunk létrehozni. Esetünkben ez `POD`.
- Néhány más lehetséges érték:

  | Kind        | Version  | 
  | ------      | ---      | 
  | Pod         | v1       | 
  | Service     | v1       | 
  | ReplicaSet  | apps/v1  | 
  | Deployment  | apps/v1  | 
 
### metaadata
- A `metadata` az objektummal kapcsolatos adatok, mint például a `name`, `labels`.
- A `metadata`, mint látható itt az értéket dictionary formályában adjuk meg. Minden, ami alatt van, az a gyermekei. A `name` és `labels` mivel testvérek, tehát egyenragúak.
-  A name egy string érték, eseetünkben a POD-ot myapp-pod-nak neveztük el.
-  A labels pedig egy dictionary, a metadata dictionaryn belül és tetszőleges kulcs-érték párokat tartalmazhat. jelenleg egy label-t adtunk hozzá, az app-ot myapp értékkel. Természetesen más labeleket is hozzárendelhetünk, ezek később segíthetnek azonosítani az objektumokat.

### spec
- A konfigurációs fájl utolsó szakasza a specifikáció. Attól függően, hogy milyen objektumot fogunk létrehozni, itt adunk meg további információkat a kubernetes számára az adott objektummal kapcsolatban.
-  A Spec egy dictionary, ennek megfelelően várja a tulajdonságokatm lista formájában. Esetünkben ez a containers tulajdonság.

## Controllers

- A kontrollerek a Kubernetes mögött álló agy. Ezek olyan folyamatok, amelyek figyelik a kubernetes objektumokat és ennek megfelelően reagálnak.
- Abban az esetben, ha az alkalmazásunk egyetlen POD-on fut, és az valamilyen oknál fogva összeomlik, akkor az alkalmazás elérhetetlen lesz. A replicationController segít ennek a megoldásában. Akár egyetlen POD-nál is segíthet azzal, hogy automatikusan létrehoz egy új POD-ot az app egy őj példányával, ha a meglévő meghibásodik. A replicationController biztosítja, hogy mindig a megadott számú POD fusson, még akkor is ha ez csak 1 vagy akár 100.
- A másik ok, amiért replicationControllerre szükségünk van, az az, hogy a POD-ok közt megteremetsük a terhelés elsoztását. Például ebben az egyszerű esetben egyetlen POD szolgálja ki a felhasználók egy csoportját. Amikor a felhasználók száma megnő, további POD-ot hozunk létre, hogy a terhelést a két pod között kiegyenlítsük. Ha az igény tovább nő, és ha az első node-on kifogynának az erőforrások, további POD-okat indíthatunk a cluster többi nodejára. Mint látható, a replicationController a cluster több node-jára is kiterjed. Segít nekünk a loadbalance-ban a podok közt, akár különböző node-kon, valamint az alkalmazásunk skálázásában az igény függvényében.
- Fontos, hogy két hasonló `kind` létezik, a  Replication Controller  és a Replica Set. Mindkettőnek ugyanaz a célja, de nem ugyanaz. A Replication Controller a régebbi technológia, amelyet a Replica Set vált fel, ez az új, ajánlott módja a eplikák beálításának. 

### Replica Set

          apiVersion: apps/v1
          kind: ReplicaSet
          metadata: 
            name: myapp-replicaset
            labels:
              app: myapp
          spec:
            selector:
              matchLabels:
                env: myapp
            replicas: 3
            template:
              metadata:
                name: nginx-2
                labels:
                  env: myapp
              spec:
                containers:
                 - name: nginx
                   image: nginx



- Mint minden kubernetes definíciós fájlban, itt is 4 szakaszunk lesz. Az `apiVersion`, a `kind`, a `metadata` és a `spec`. 
- A replicaSet-et `apps/v1` apiVersion támogatja 
- A kind replicaSet
- A metadata alatt definiáljuk a name-t és myapp-replicaset-nek fogjuk hívni. Hozzáadunk még egy labelt, app és értéket rendelünk hozzá
- A következő szakasz a definíciós fájl legfontosabb része, ez pedig a specifikáció. Minden kubernetes definíciós fájlban a spec szakasz határozza meg, hogy mi van az általunk létrehozott objektum belsejében
- A replicaSet egy POD több példányát hozza létre. Hozzunk létre egy `template` szakaszt, hogy megadjunk egy POD-template-t, amelyet a replicaSet használ a replikációk létrehozásához. Az előző példában létrehozott pod-definition fájl teljes tartalmát áthelyezzük a replikációs vezérlő template szakaszába, kivéve az első két sort - amelyek az apiVersionés a kind.
- Itt definiálhatjuk `replicas` szakaszban, hogy hány replikát szeretnénk
- A replikakészlethez szükség van egy `selector` definícióra. A `selector` szakasz segít a replicaSet-nek azonosítani, hogy milyen podok tartoznak alá. A replikakészlet olyan podokat is kezelhet, amelyek nem a replicaset létrehozása során jöttek létre. Tegyük fel például, hogy a ReplicaSet létrehozása előtt létrehoztunk podokat, amelyek megfelelnek a szelektorban megadott labelnek, a replicaSet ezeket a podokat is figyelembe veszi a replikák létrehozásakor. Ezt pedig az itt látható matchLabels formájában kell definiálni. A `matchLabels` egyszerűen az alatta megadott lebel-eket egyezteti a POD-okon lévő label-ekkel.
- Ha a fájl elkészült, futtassa a következő parancsot:  `kubectl create -f <filename>` parancsot.
- Ezzel a replikációs vezérlő létrejön. A replikációs vezérlő létrehozásakor először létrehozza a POD-okat a pod-definíció sablon segítségével, annyit, amennyi szükséges, ami ebben az esetben 3 darab.

          student@c-plane1:~/kubernetes/pods$ kubectl create -f nginx.yaml 
          pod/nginx-2 created
          student@c-plane1:~/kubernetes/pods$ kubectl get pods
          NAME                     READY   STATUS    RESTARTS   AGE
          myapp-replicaset-5cgxq   1/1     Running   0          15m
          myapp-replicaset-s8pkj   1/1     Running   0          16m
          myapp-replicaset-sb2jw   1/1     Running   0          16m


## Deployment

- Tegyük fel, hogy van egy webszerverem, amelyet egy produktív környezetben akarok kitenni. Nem EGY, hanem sok példányra van szükségem a webkiszolgáló futtatásához. Másodszor, amikor az alkalmazásnak újabb verziója elérhetővé válik a docker registryben, szeretném zökkenőmentesen UPGRADE-elni. Verzió váltáskor nem szeretné egyszerre frissíteni az összeset, ezért érdemes egymás után frissíteni őket. Ezt a fajta frissítést pedig `rolling update`-nek nevezzük.
-  Tegyük fel, hogy az egyik frissítés váratlan hibát eredményezett, és szeretnénk vissza állni egy korábbi példányra, ezt `rollBACK` nevezzük
- Továbbá a kubernetesben a `Deployment` segítségével  képesek vagyunk  több módosítást(verzióváltás, skálázás, erőforrás limitek beállítása) végezni egyetlen rollout-ban. 

          apiVersion: apps/v1
          kind: Deployment
          metadata: 
            name: myapp-deployment
            labels:
              tier: frontend
              app: nginx
          spec:
            selector:
              matchLabels:
                env: myapp
            replicas: 3
            template:
              metadata:
                name: nginx-2
                labels:
                  env: myapp
              spec:
                containers:
                 - name: nginx
                   image: nginx


- A deployment-definition fájl tartalma pontosan megegyezik a replicaset definíciós fájléval, kivéve a `kind`-ot, ami most Deployment lesz.
- Ha végigmegyünk a fájl tartalmán, van egy `apiVersion`, ami apps/v1, `metadata`, ami `name` és `labels`  tartalmaz, és egy `spec`, ami `template`, `replicas` és `selector`. A `template`-ben van egy POD definíció
- A file-on megfuttatjuk a `kubectl create` parancsot. Ezután a telepítés automatikusan létrehoz egy replika készletet. 

### Rollout

- Amikor új deploymentet hozunk létre, vagy frissítjük a meglévő imageit, az egy Rolloutot indít el. A rollout az alkalmazáskonténerek fokozatos telepítésének vagy frissítésének folyamata. 
- Amikor először hozunk létre egy új deployt, az rolloutot indít el. Egy új rollout egy új Deployment revíziót hoz létre, ezt nevezzük `revision1`-nek 
- A jövőben, amikor az alkalmazás frissítésre kerül - vagyis amikor a konténer verziója új verzióra frissül -, egy új rolloutot indítunk el, és egy új telepítési revíziót hozunk létre, amelynek a neve `revision2` lesz. Ez segít nekünk nyomon követni a deploymentünkön végrehajtott változásokat, és lehetővé teszi, hogy szükség esetén visszaállítsuk a telepítés korábbi verzióját.

### Deployment stratégiák

- Kétféle Deployment stratégia létezik. Tegyük fel, hogy 5 replikát deployolunk a webalkalmazásunkból. Ezek frissítésének egyik módja az, hogy a kubernetes mindegyiket megsemmisíti, majd létrehozza az alkalmazás-példányok újabb verzióit. A probléma ezzel az, hogy a régebbi verziók leállása utáni időszakban, és az újabb verziók élesítése előtt az alkalmazás leáll, és a felhasználók számára elérhetetlen lesz. Ezt a stratégiát `Recreate` stratégiának nevezik
-  A második stratégia az, ahol nem pusztítjuk el az összeset egyszerre. Ehelyett a régebbi verziót leszedjük, és egyesével hozunk fel egy újabb verziót. Így nem lesz kiesés, és a frissítés zökkenőmentes lesz
- Amennyiben nem definiálunk explicit stratégiát, úgy a rendszer `RollingUpdate`-t fog feltételezni. A RollingUpdate az alapértelmezett telepítési stratégia.

 
