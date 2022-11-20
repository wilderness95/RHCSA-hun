---
attachments: [variables.jpg]
tags: [Ansible, Linux, Udemy]
title: Ansible
created: '2022-11-18T22:22:15.735Z'
modified: '2022-11-19T23:07:02.245Z'
---

# Ansible

## Bevezetés
- Az Ansiblel egy vagy akár több rendszeren is dolgozhatunk egyszerre az infrasturktúránkban.
- Ahhoz, hogy több szerverren dolgozhasson, az Ansible-nek kapcsolatot kell létesítenie ezekkel a szerverekkel. Ez SSH segítségével történik linux és winrm segítségével Windows esetén. Ez teszi az Ansible-t Agentlessé.
- Az Agentless azt jelenti, hogy nem kell telepíteni semmilyen további szoftvert a célgépekre ahhoz, hogy az Ansible elérje őket. Egy egyszerű SSH kapcsolat is elegendő az Ansible-nek.
- Az egyik legnagyobb hátránya a legtöbb más konfigurációmanagment/automatizáló eszköznek, hogy szükség van valamilyen agent telepítésére/konfigurálására a célrendszereken mielőtt bármilyen automatizálást csinálhatnánk.
- A célrendszereket egy úgynevezett inventory file-ban tároljuk. Ha nem hozunk létre új új inventoryt, az Ansible az alapértelmezett inventoryt használja, amely a `etc/Ansible/hosts` alatt található.

## Inventory file

      # Sample Inventory File

      # Web Servers
      web_node1 ansible_host=web01.xyz.com ansible_connection=winrm ansible_user=administrator ansible_password=Win$Pass
      web_node2 ansible_host=web02.xyz.com ansible_connection=winrm ansible_user=administrator ansible_password=Win$Pass
      web_node3 ansible_host=web03.xyz.com ansible_connection=winrm ansible_user=administrator ansible_password=Win$Pass

      # DB Servers
      sql_db1 ansible_host=sql01.xyz.com ansible_connection=ssh ansible_user=root ansible_ssh_pass=Lin$Pass
      sql_db2 ansible_host=sql02.xyz.com ansible_connection=ssh ansible_user=root ansible_ssh_pass=Lin$Pass

      [db_nodes]
      sql_db1
      sql_db2

      [web_nodes]
      web_node1
      web_node2
      web_node3

- A fenti inventory file `INI` formátumu. Egy sorban van definiálva  az adott kapcsolat és annak tulajdonságai.
- A szerverekre az Ansible-ben egy aliassal szeretnék hivatkozni, mint pl web_done# vagy sql_db#. Ezután az `ansible_host`-al megadom az alaishoz tartozó `fqdn`-t vagy `ip`-t.
- Majd a szerverekre való csatlakozást módját állítom be. `SSH` linux esetén `winrm` windows esetén.
- A végén pedig az authentikációs adatokat állítom be. 
- Amennyiben nem egyeneként akarok a gépekre hivatkozni, úgy host groupokat is tudok definiálni: [db_nodes], [web_nodes]. Ha ezt adom be az Ansiblenek, úgy az összes gépre lefog futni, amik benne vannak az adott groupban

## Playbook

- A playbook egyetlen YAML fájl, amely playeket tartalmaz tartalmaz. Egy play meghatározza a futtatandó tevékenységeket egy vagy több hoston.

      name: Play 1
      hosts: localhost
      tasks:
      - name: Execute command ‘date’
          command: date
      - name: Execute script on server
          script: test_script.sh
      - name: Install httpd service
          yum:
            name: httpd
            state: present
      - name: Start web server
          service:
            name: httpd
            state: started

- A playbook gyakorlatilag egy dictionary, melynek vannak beálításai, eseteünkben, a `name` és a `hosts`. A properties-nél az egyes tagok sorrendje felcserélhető.
- A `hosts` alatt definiálhatjuk, hogy a playbook mely hostokra fusson le. Esetünkben a localhoston fut le, de itt meg adhatjuk expliciten, hogy mely host groupokra vagy akár mindre lefusson
- A taskok egy rendezett lista a feladatokról. Az itt lévő taskok a megadott sorrendben fognak végrehajtódni.
- A taskok-ban definiált, végrehajtandó műveleteket moduloknak nevezzük. Itt a `command`, `script`, `yum` és `service` modulokat használjuk. 
- Modulokból több száz áll rendelkezésre, ezeknek a dokumentációját itt találjuk meg: [ansible docs](https://docs.ansible.com/ansible/2.9/modules/list_of_all_modules.html)
- Futtatni így tudjuk: 

      ansible-playbook <playbookname>

  ## Modulok
  ## Változók
  - Mint bármely más programozási nyelvben, a változókkal különböző értékeket tudunk tárolni.
  - Változókat definiálhatunk az inventory file-ban:

              web_node1 ansible_host=web01.xyz.com ansible_connection=winrm ansible_user=administrator ansible_password=Win$Pass
  - Playbookon belül is tárolhatunk változókat a `vars` alatt
  - Lehetőségünk van teljesen külön file-ban tárolni változókat
  - A változók használatára egy jó példa:
  ![](C:\Users\horvathz\Documents\notes\variables.jpg)
  ## Ciklusok
  - Előfordulhat, hogy egy feladatot többször is meg kell ismételni. Például több felhasználót kell létre hozni, több service-t kell elindítani/leállítani, vagy több fájl jogosultságát megváltoztatni a menedzselt hostokon. Ezeket külön-külön taskokba is felvehetjük, vagy létrehozhatunk ciklusokat, amiket változok segítségével könnyedén felparaméterezhetünk
  - A lenti egy nagyon egyszerű példa egy ciklusra. Mint látható, a `loop` kulcsszót a feladat nevével azonos behúzási szinten használtuk. Ebben az esetben a yaml szintaxis segítségével megadtuk tömbként a létrehozandó userekről; majd magában a taskban az `item` változóval hivatkoztunk mindegyikre. Ez a változó minden egyes iterációnál az általunk megadott tömb egy elemére fog hivatkozni.

            - name: Create users
            hosts: all
            tasks:
            - user: name='{{ item.name }}' state=present uid='{{ item.uid }}'
              loop:
	            - name: joe
	              uid:1001
                pass: pass1
	            - name: jane
	              uid:1002
                pass: pass2
	            - name: mike
	              uid:1003
                pass: pass3



  ## Feltételek
  - A feltételek segítségével univerzálissá tehetünk playbookokat a `when` kulcsó segítségével. Az alábbi példában egy apache telepítést láthatunk, ami az OS verzió függvényében yum vagy apt modult fog használni. A feltételeknél lehetőségünk van logikai operátorok használatára is

        - name: Install NGINX
          hosts:
          tasks:
            - name: Install NGINX on Debian
              apt:
                name: nginx
                state: present
              when: ansible_os_family  == "Debian" and
        	          ansible_distribution_version == "22.04"

            - name: Install NGINX on Rhel
              apt:
                name: nginx
                state: present
              when: ansible_os_family  == "RedHat" or
        	          ansible_os_family == "SUSE”

  - Feltételeket megtudunk ciklusokban is fogalmazni. Ezzel a módszerrel a ciklús által létrehozott taskok lefutását tudjuk feltételhez kötni.

          - name: Install Softwares
            hosts: all
            vars:
              packages:
                 - name: nginx
                   required: False
                 - name: mysql
                   required : False
                 - name: httpd
                   required : True
            tasks:
            - name: Install "{{ item.name }}"
              yum:
                name: "{{ item.name }}"
                state: present
              when: item.required == True
              loop: "{{ packages }}"

  - Egy playbookban gyakran előfordul, hogy egy korábbi task eredménye alapján akarunk végrehajtani vagy kihagyni egy késöbbi taskot. Például egy szolgáltatás konfigurálására van szükség, miután egy korábbi feladat frissítette azt. Egy feltétel létrehozása register segítségével:
      - Mentsük el egy task eredményét egy registerbe
      - Csináljunk egy taskot ami a regeszter eredményétől függ

            - name: check registered variable for emptiness
              hosts: all
            
              tasks:
            
                  - name: List contents of directory
                    ansible.builtin.command: ls mydir
                    register: contents
            
                  - name: Check contents for emptiness
                    ansible.builtin.debug:
                      msg: "Directory is empty"
                    when: contents.stdout == ""
        
  
  ## Role-ok
- Gyakran előforduló probléma, hogy a playbook-ok taskjai ismétlődnek. Ennek a problémának az elkerülésére valók a role-ok alkalmazása. Segítségükkel az egyszer megírt kód újrahasználható:

           - name: MySQL role
           - tasks:
              - name: Install Pre-requisties
                yum:
                  name: pre-req-packages
                  state: present

              - name: Install MySQL
                yum:
                  name: mysql
                  state: present

              - name: Start MySQL service
                service:
                  name: mysql
                  state: started

              - name: Configure DB
                mysql_db:
                  name: db1
                  state: present

    ##########################################################################

               - name: install and configure MySQL
                 hosts: db_servers
                 roles:
                   - mysql   

- Segítségével az általunk megírt kód jól struktúrálható és karbantartható. Az ansible-galaxy ad nekünk egy skelt ami létrehozza a szükséges mappa szerkezetet az adott projekthez:

            [wilderness@Moby:~/AnsibleProjects/udemy]$ansible-galaxy init mysql
            - Role mysql was created successfully
            [wilderness@Moby:~/AnsibleProjects/udemy/mysql]$tree
            .
            |-- README.md
            |-- defaults
            |   `-- main.yml
            |-- files
            |-- handlers
            |   `-- main.yml
            |-- meta
            |   `-- main.yml
            |-- tasks
            |   `-- main.yml
            |-- templates
            |-- tests
            |   |-- inventory
            |   `-- test.yml
            `-- vars
                `-- main.yml

- A role-ok alapértelmezetten az `/etc/ansible/roles` alatt található. Ezt felültudjuk definiálni az `/etc/ansible/ansible.cfg` konfigban a `roles_path` attributumnál, viszont ajánlatos az alábbi szerkezetet létrehozni:

            |-- install-servers
            |   |-- playbook.yml
            |   `-- roles
            |       `-- mysql
            |           |-- README.md
            |           |-- defaults
            |           |   `-- main.yml
            |           |-- files
            |           |-- handlers
            |           |   `-- main.yml
            |           |-- meta
            |           |   `-- main.yml
            |           |-- tasks
            |           |   `-- main.yml
            |           |-- templates
            |           |-- tests
            |           |   |-- inventory
            |           |   `-- test.yml
            |           `-- vars
            |               `-- main.yml

