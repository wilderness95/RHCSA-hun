---
title: Docker
created: '2022-11-15T18:47:24.826Z'
modified: '2022-11-15T18:47:53.240Z'
---

# Docker


MANAGE REGISTRY
---------------

Log in to the Docker hub registry
$ docker login <someregistry.example.com>

Log out from the registry previously logged in and check if it was really removed:
$ docker logout; ls -la ~/.docker

Setup private registry:
$ docker run -p 5000:5000 registry
$ ID='docker run -id fedora /bin/bash'
$ docker commit $ID fedora-20
$ docker tag fedora-20 registry-host:5000/lmaly/f20
$ docker -d --insecure-registry registry-host:5000
$ docker push registry-host:5000/lmaly/f20
$ docker pull registry-host:5000/lmaly/f20


GENERAL USAGE
-------------

Find out installed Docker version:
$ docker --version
$ docker version

Check if the Docker daemon is running and Docker server info:
$ docker info

Make sure Docker server starts automatically on boot (RHEL7):
$ sudo systemctl enable docker; sudo systemctl start docker

Use docker from a nonroot user, add the user to the docker group:
$ sudo gpasswd -a $USER docker

Create a new container with a specific name (otherwise name is randomized), but don't start it:
$ docker create --name="Lucian's awesome service" ubuntu:latest

Start/restart a stopped container:
$ docker start mydb
$ docker start c7337

Run a container in the background (docker run = docker create + docker start):
$ docker run -d jenkins
$ docker run -d <image>:<tag>

Run an interactive container:
$ docker run -it ubuntu bash
$ docker run -it <image>:<tag> <command>

Run a container automatically removed on stop:
$ docker run --rm ubuntu bash

Run a named container:
$ docker run --name mydb redis

Inspect a docker container:
$ docker inspect mydb

Pausing/unpausing the container:
$ docker pause c7337
$ docker unpause c7337

Stop a container:
$ docker stop mydb

SIGKILL after 25s if has not stopped by SIGTERM:
$ docker stop -t 25 mydb

SIGKILL a container:
$ docker kill mydb
$ docker kill --signal=USR1 c7337

Add metadata/labels to a container:
$ docker run -d label=traffic.backend=jenkins jenkins
$ docker run -d --name labels -l deployer=Lucian -l tester=Lucian ubuntu:latest sleep 1000

Add environment variable to the container run:
$ docker run -d -p 8080:8080 -e ENV="Hello" mydb

Run another process in the running container, get inside the container:
$ docker exec -it c7337 bash
$ docker exec nginx "yum -y update nginx"

Show live logs of the running daemon container as well as timestamps:
$ docker logs -ft c7337
$ docker logs <container_name>


MANAGE CONTAINERS
-----------------

List running containers:
$ docker ps

List all containers (running & stopped):
$ docker ps -a

Searching Docker images:
$ docker search ubuntu

Download image from the registry:
$ docker pull ubuntu:latest
$ docker pull debian@sha256:asd654ferf564rwer21fvwrd654rwefv213s

Push the image to the registry:
$ docker push <dockerhub_username>/<image_name>

Inspect containers metadata:
$ docker inspect c7337

List local available images:
$ docker images

Rename container:
$ docker rename <current_name> <new_name>

Delete an image and all associated filesystem layers:
$ docker rmi c7337

Remove unwanted container, or reuse a name you previously assigned to a container:
$ docker rm <container_name>

Force removal of the image, if the image is referenced in one or more repositories:
$ docker rmi -f ubuntu:trusty

Delete all stopped containers:
$ docker rm $(docker ps --filter status=exited -q)

Remove stopped containers and all it's volumes:
$ docker rm -v $(docker ps -a -q)
$ sudo ls /var/lib/docker/volumes/65sd4f35s4df35sd4f354sdf

Delete all containers that exited with a nonzero state:
$ docker rm $(docker ps -a -q --filter 'exited!=0')

Delete all of the containers and images on the Docker host:
$ docker rm $(docker ps -a -q); docker rmi $(docker images -q -)

List all containers with a specific label:
$ docker ps --filter label=traffic.backend

Query a specific metadata of a running container:
$ docker inspect -f '{{.NetworkSettings.IPAddress}}' c7337
$ docker inspect -f '{{.Config.Hostname}}' b7e2fe3b2e9b


BUILD IMAGES
------------

Build an image from Dockerfile in the current directory:
$ docker build --tag myimage .
$ docker build -t <dockerhub_username>/<dockerhub_repository> .
$ docker build -f <path_to_Dockerfile> -t <repository>:<tag>

Force rebuild of Docker image:
$ docker build --no-cache .

Convert a container to an image:
$ docker commit c7337 myimage
-When you exit the container, it stops running but IT IS still available to you until you remove it entirely with 'docker rm'
Before removing it, you need to commit:
$ docker commit -a <AUTHOR> -m <COMMENT> <container_ID> <repository>:<tag>
Now you can safely remove the stopped container

Preserve any CMD or ENTRYPOINT instructions that are inherited from the base image:
$ docker commit --change 'CMD ["/bin/bash"]' <container_ID> <image_name>

Remove all untagged images:
$ docker rmi $(docker images -q -f "dangling=true")

See the timeline/history of your images:
$ docker history myimage

Get tree-like view of images:
$ docker images -t

See dependencies graphically:
$ docker images --viz | dot -Tpng -o /tmp/docker_dependencies.png
$ display /tmp/docker_dependencies.png

Building images of existing machines using tar:
$ sudo apt-get install -y debootstrap
$ sudo debootstrap <distribution_codename> <directory_name> > /dev/null
$ sudo tar -C <directory_name> -c . | sudo docker import - <image_name>
$ docker images

$ docker diff <container_ID>
A=added
C=changed
D=deleted

Rename images (some Docker versions only allow lowercase tags!):
$ docker tag ubuntu:14.04 foobar

Tag image (some Docker versions only allow lowercase tags!):
$ docker tag ubuntu:14.04 foo:bar

Change repository:
$ docker tag flask how2dock/flask sebimac:flask
$ docker tag <REGISTRYHOST>/<USERNAME>/<REPOSITORY> <IMAGE>:<TAG>

Export/import/load/save container to/from tar:
 (1) Stop container
 (2) $ docker export <container_ID> > container.tar
 (3) $ docker import - <image_name> < container.tar
Example of importing image, making changes and exporting image:
$ docker import APOL_EL6_VM_Base_1-0-0.tar
$ docker tag 54358f7e7c88 luckylittle/apol_el6_vm_base:version1.0.0
$ docker run -it -h oel6.dedc2.oraclecloud.com --name oel6.dedc2.oraclecloud.com -v /fsnadmin:/fsnadmin luckylittle/apol_el6_vm_base:version1.0.0 /bin/bash
<- MAKE CHANGES HERE AND THEN EXIT THE CONTAINER ->
$ docker commit -a lucian.maly@oracle.com -m "Added PDIT Puppet" 931322f3b88a luckylittle/apol_el6_vm_base:version1.0.1
$ docker save luckylittle/apol_el6_vm_base:version1.0.1 > ~/apol_el6_vm_base_v1.0.1.tar
-this will remove history!!!

Already commited image:
$ docker save -o <new_image>.tar <old_image>
$ docker rmi <old_image>
$ docker load < <new_image>.tar
$ docker images
-this will retain history!!!


VOLUMES
-------

Create a local volume:
$ docker volume create --name myvol

Mounting a volume on container start:
$ docker run -v myvol:/data redis

Destroy a volume:
$ docker volume rm myvol

List volumes:
$ docker volume ls

Mount /mnt/session_data to /data within the container read-only:
$ docker run --rm -ti --read-only=true -v /mnt/session_data:/data ubuntu:latest /bin/bash

Have multiple data volumes within a container:
$ docker run -t -d -P -v /data -v logs --name f20 fedora /bin/bash

Data volume container scenario:
$ docker run -d -v /data --name data fedora echo "data volume container"
$ docker run -d -it --volumes-from data --name client1 fedora /bin/bash
$ docker run -d -it --volumes-from data --name client2 fedora /bin/bash

You can give access of a host devie to the container:
$ docker run --device=/dev/sdc:/dev/xvdc -it fedora /bin/bash

Copying data from container to host:
$ docker cp <container_name>:/<dir>/<file.ext> .

Copying data from host to container:
$ docker cp <file.ext> <container_name>:/<dir>/<file.ext>

Copying data between two containers:
$ docker cp c1:/root/file.txt .; docker file.txt c2:/root/file.txt

Inspect mount mapping:
$ docker inspect -f {{.Mounts}} c773


NETWORKING
----------

-Most of the time, docker0 bridge on the host will have address 172.17.42.1
-All containers will get address in 172.17.42.0/24 network and docker0 will be their gw
-on the host itself, there will be virtual NIC (e.g. veth450b81a)
--net=none                       - only loopback
--net=host                       - network of the container is the same as the one of the host
--net=bridge                     - default
--net=container:<container_name> - container has sane ip and hostname as the first container
--bip=172.17.0.1/17              - docker0 bridge ip address
--fixed-cidr=172.17.0.0/17       - give containers ip addresses in the specific network

Export port 80 from a container to port 80 on the host:
$ docker run -p 80:80 -d nginx
$ docker run -dp <host_port>:<container_port> <image>:<tag>

Expose/Publish all exposed ports to random ports on the host:
$ docker run -d -P flask

To bind to a specific interface
$ docker run -id -p 192.168.1.10:5000:22 --name f20 fedora /bin/bash
-Requests coming to port 5000 on the interface that has the IP 192.168.1.10 on the Docker host will be forwarded to port 22 of the f20 container

Map port 22 of the container to the dynamic port of the host:
$ docker run -id -p 192.168.1.10::22 --name f20 fedora /bin/bash

Bind multiple ports on containers to ports on hosts:
$ docker run -id -p 5000:22 -p 8080:80 --name f20 fedora /bin/bash

Create a local network:
$ docker network create mynet

Attach a container to a network on start:
$ docker run -d --net mynet redis

Connect a running container from a network:
$ docker network connect mynet c7337

Disconnect container from a network:
$ docker network disconnect mynet c7337

Configure a hostname in the container:
$ docker run --rm -ti --hostname="mycontainer.example.com" ubuntu:latest

Configure domain name service (DNS) in the container:
$ docker run --rm -ti --dns=8.8.8.8 --dns=8.8.4.4 --dns-search=example1.com --dns-search=example2.com ubuntu:latest /bin/bash

Specify MAC address for the container:
$ docker run --rm -ti --max-address="68:F7:28:8A:C4:7F" ubuntu:latest

Show exposed ports of a container:
$ docker port c7337

Linking two containers together:
Docker will automatically set up the networking so that the ports exposed by the MySQL container are reachable inside the Wordpress container:
--link <container_name>:<alias>
$ docker run --name mysqlwordpress -e MYSQL_ROOT_PASSWORD=password -d mysql
$ docker run --name wordpress --link mysqlwordpress:mysql -p 80:80 -d wordpress

Linking three containers together:
$ docker run -d --name database -e MYSQL_ROOT_PASSWORD=password mysql
$ docker run -d --name web --link database:db runsweb/hostname
$ docker run -d --name lb --link web:application nginx
Result of linking:
 - the application container now contains environment variable that point to the database, load balancer contains environment variables that point to the application
 - /etc/hosts file is automatically updated to contain information for name resolution
 - <alias> will be found in the /etc/host entry as a prefix
 
What is linked together?
$ docker inspect -f "{{.HostConfig.Links}}" application
$ docker inspect -f "{{.HostConfig.Links}}" lb

Create new overlay network:
$ docker network create -d overlay foobar
$ docker network ls
Start container on this network:
$ docker run -it --rm --publish-service=bar.foobar.overlay ubuntu:14.04 bash
$ docker service ls

IPv6
$ vim /etc/sysconfig/docker
  OPTIONS='--selinux-enabled --ipv6'
$ docker -d --ipv6


PERFORMANCE & DEBUG
-------------------

Starting Docker in debug mode:
$ docker -d -D

Container stats:
$ docker stats c7337

See the event log of the containers:
$ docker events
$ docker events --since '2015-01-01' --until '2016-01-01'
$ docker events --filter 'event=start'
$ docker events --filter 'image=centos:centos7'
$ docker events --filter 'container=c7337dkj324kjnbr23hbv34'

Debugging the container's resources:
$ docker top c7337
$ docker top <container_name>

Create load average of around 5 by creating two CPU-bound processes, one I/O bound process and two memory allocation processes:
$ docker run --rm -ti progrium/stress --cpu 2 --io 1 --vm 2 --vm-bytes 128M --timeout 120s

...same, but with only half the amount of available CPU time:
$ docker run --rm -ti -c 512 progrium/stress --cpu 2 --io 1 --vm 2 --vm-bytes 128M --timeout 120s

Pin a container to one or more CPUs, indexed from 0:
$ docker run --rm -ti -c 512 --cpuset=0 progrium/stress --cpu 2 --io 1 --vm 2 --vm-bytes 128M --timeout 120s

Passing the memory constraints:
$ docker run --rm -ti -m 512m progrium/stress --cpu 2 --io 1 --vm 2 --vm-bytes 128M --timeout 120s

Define total amount of memory and swap available to the container:
$ docker run --rm -ti -m 512m --memory-swap=769m progrium/stress --cpu 2 --io 1 --vm 2 --vm-bytes 128M --timeout 120s

Start all containers with a hard limit of 150 open files and 20 processes:
$ sudo docker -d --default-ulimit nofile=50:150 --default-ulimit nproc=10:20

...override specific container values by passing:
$ docker run -d --ulimit nproc=100:200 nginx

Change the containerization driver to LXC:
$ docker -d -e lxc

Change the storage driver from AUFS to devicemapper:
$ docker -d --storage-driver=devicemapper

CPU controller:
$ sudo ls /sys/fs/cgroup/cpu/docker/<c7337...>
$ sudo cat /sys/fs/cgroup/cpu/docker/<c7337...>/cpu.shares

Example of running container in privileged mode, memory swapfile creation:
$ docker run -ti --rm --privileged=true ubuntu /bin/bash
  $ dd if=/dev/zero of=/swapfile1 bs=1024 count=100
  $ mkswap /swapfile1
  $ swapon /swapfile1
  $ swapoff /swapfile1
  $ exit
$ sudo swapoff /var/lib/docker/overlay/<c7337...>/upper/swapfile1

Add only certain linux capabilities to the run:
$ docker run -ti --rm --cap-add=NET_ADMIN ubuntu /bin/bash

Explore dependencies:
$ docker run --publish=8085:8080 --detach=true --name=static-helloworld adejonge/helloworld:latest
$ docker info | grep 'Root Dir:'
$ ls -R /mnt/sda1/var/lib/docker/aufs/mnt/<c7337...>


SECURITY
--------

Defang setuid/setgid binaries:
$ docker run debian \
  find \ -perm +6000 -type f -exec ls -ld {} \; 2> /dev/null
$ vim Dockerfile
  FROM debian:wheezy
  RUN find / -perm +6000 -type f -exec chmod a-s {} \; || true
  
Be aware of CPU shares and memory:
$ docker run -d -c 512 myimage
$ docker run -m 512m myimage

Set container filesystem to read-only:
$ docker run --read-only debian

Set volumes to read-only:
$ docker run -v $(pwd)/secrets:/secrets:ro debian

Turn off inter-container communication:
$ docker -d --icc=false --iptables

Do not install unnecessary packages in the container:
$ docker exec $INSTANCE_ID rpm -qa
$ docker exec $INSTANCE_ID dpkg -l

Docker will not be able to manage iptable rules and ipv4 forwarding is also disabled if you use this as default:
$ echo DOCKER_OPTS=\"--iptables=false --ip-forward=false\" >> /etc/sysconfig/docker
# sudo systemctl restart docker

Allowing writes to volume with SELinux enabled:
$ docker run -it -v /tmp/:/tmp/host:z fedora bash
$ docker run -it -v /tmp/:/tmp/host:Z fedora bash


APPENDIX#1 - USAGE
------------------

[lmaly@oracle-linux ~]$ docker --help
[lmaly@oracle-linux ~]$ docker COMMAND --help

docker [OPTIONS] COMMAND [arg...]
docker daemon [ --help | ... ]
docker [ --help | -v | --version ]

OPTIONS:
--config=~/.docker              Location of client config files
-D, --debug                     Enable debug mode
-H, --host=[]                   Daemon socket(s) to connect to
-h, --help                      Print usage
-l, --log-level=info            Set the logging level
--tls                           Use TLS; implied by --tlsverify
--tlscacert=~/.docker/ca.pem    Trust certs signed only by this CA
--tlscert=~/.docker/cert.pem    Path to TLS certificate file
--tlskey=~/.docker/key.pem      Path to TLS key file
--tlsverify                     Use TLS and verify the remote
-v, --version                   Print version information and quit

COMMANDS:
attach    Attach to a running container
build     Build an image from a Dockerfile
commit    Create a new image from a container's changes
cp        Copy files/folders between a container and the local filesystem
create    Create a new container
diff      Inspect changes on a container's filesystem
events    Get real time events from the server
exec      Run a command in a running container
export    Export a container's filesystem as a tar archive
history   Show the history of an image
images    List images
import    Import the contents from a tarball to create a filesystem image
info      Display system-wide information
inspect   Return low-level information on a container or image
kill      Kill a running container
load      Load an image from a tar archive or STDIN
login     Register or log in to a Docker registry
logout    Log out from a Docker registry
logs      Fetch the logs of a container
network   Manage Docker networks
pause     Pause all processes within a container
port      List port mappings or a specific mapping for the CONTAINER
ps        List containers
pull      Pull an image or a repository from a registry
push      Push an image or a repository to a registry
rename    Rename a container
restart   Restart a container
rm        Remove one or more containers
rmi       Remove one or more images
run       Run a command in a new container
save      Save an image(s) to a tar archive
search    Search the Docker Hub for images
start     Start one or more stopped containers
stats     Display a live stream of container(s) resource usage statistics
stop      Stop a running container
tag       Tag an image into a repository
top       Display the running processes of a container
unpause   Unpause all processes within a container
update    Update resources of one or more containers
version   Show the Docker version information
volume    Manage Docker volumes
wait      Block until a container stops, then print its exit code


APPENDIX#2 - SWARM
------------------
https://docs.docker.com/swarm/provision-with-machine

#########################################
## Docker Swarm v1 (before Docker v1.12)
#########################################

Do we have some running machines/nodes already?
$ docker-machine ls

Create machine/node 'default' using the virtualbox:
$ docker-machine create --driver virtualbox <default>

Configure your shell (export env variables) to remotely gain access:
$ docker-machine env <default>

Currently active machine?
$ docker-machine active

SSH to the node and verify running processes:
$ docker-machine ssh <default> ps aux | grep docker

Provision other three hosts:
$ docker-machine create --driver virtualbox node1
$ docker-machine create --driver virtualbox node2
$ docker-machine create --driver virtualbox node3
$ docker-machine ls

Port numbers:
+--------------+---------------------------------------------------+
| Insecure     | Secure                                            |
+--------------+---------------------------------------------------+
| Engine: 2375 | Engine: 2376                                      |
| Swarm: 3375  | Swarm: 3376                                       |
|              | Swarm v2 uses 2377 for node discovery among nodes |
+--------------+---------------------------------------------------+

Change the listening port on the 'default' node and disable TLS:
$ docker-machine ssh <default>
  $ sudo su -
  # cp /var/lib/boot2docker/profile /var/lib/boot2docker/profile-bak
  # vi /var/lib/boot2docker/profile
    EXTRA_ARGS='
    --label provider=virtualbox
    '
    DOCKER_HOST='-H tcp://0.0.0.0:2375'
    DOCKER_STORAGE=aufs
    DOCKER_TLS=no
  # exit
  $ exit
$ docker-machine restart <default>
$ unset DOCKER_TLS_VERIFY
$ export DOCKER_HOST="tcp://192.168.XX.XXX:2375"

To get started with swarm, pull the image from registry on each of the nodes:
$ docker pull swarm

See the Swarm help:
$ docker run swarm --help
  !Create - is used to create clusters with a discovery service, for example token://
  !List   - shows the list of the cluster nodes
  !Manage - allows you to operate a Swarm cluster
  !Join   - in combination with a discovery service, is used for joining new nodes to an existing cluster
$ docker run swarm manage --help

To connect nodes, in each machine/node terminal, execute this:
$ docker run \
-d \
-p 3376:2375 \
swarm manage \
nodes://192.168.XX.XXX:2375,192.168.XX.XXX:2375,
192.168.XX.XXX:2375,192.168.XX.XXX:2375
 !Note: nodes:// can be simplified (same as Ansible) to nodes://192.168.XX.[XX1:XX3]:2375

When we want to talk to the whole cluster, we need to use port 3376, to do that:
$ docker-machine env <default>
$ eval $(docker-machine env default)
$ export | grep DOCKER_
$ export DOCKER_HOST="tcp://192.168.XX.XXX:3376"
$ unset DOCKER_TLS_VERIFY
$ unset DOCKER_CERT_PATH
$ docker info

Test your Swarm cluster - run container and process status will show node number in the NAME section:
$ docker run -d -p 80:80 nginx
$ docker ps
 !When you run the same container three more times, it will spread them across the four nodes
 !It will not allow 5th container, because on all hosts, port tcp/80 are already occupied


#########################################
## Docker Swarm v2 (after Docker v1.12)
#########################################

(1) Download the swarm container onto the Docker host:
$ docker pull swarm

(2) Create a Docker cluster by running the Swarm container:
$ docker run --rm swarm create

(3a) Register a Docker host with our cluster, provide IP+port of our Docker host and the first received token:
$ docker run -d swarm join --addr=172.17.42.10:2375 token://ekjn3kjbn34kjb34kbekj3bdopfjl43jkkjefl

OR with different dicovery service 'token':

 (3b) Invoke a Docker Machine command to create a new Docker container and join it to the existing Swarm cluster:
 $ docker-machine create \
   -d virtualbox \
   --swarm \
   --swarm-discovery token://ekjn3kjbn34kjb34kbekj3bdopfjl43jkkjefl \
   swarm-node1
 $ eval "$(docker-machine env local)"

(4) Check if it started:
$ docker ps

(5) Deploy swarm manager to any one of the Docker hosts in the cluster:
$ docker run -d -p 9999:2375 swarm manage token://ekjn3kjbn34kjb34kbekj3bdopfjl43jkkjefl

(6) List all of the nodes in our cluster:
$ docker run --rm swarm list token://ekjn3kjbn34kjb34kbekj3bdopfjl43jkkjefl 172.17.42.10:2375

(7) Make sure Docker does not try to use TLS when connecting to the Swarm port:
$ echo $DOCKER_HOST; unset DOCKER_HOST
$ echo $DOCKER_TLS_VERIFY; unset DOCKER_TLS_VERIFY
$ echo $DOCKER_TLS; unset DOCKER_TLS
$ echo $DOCKER_CERT_PATH; unset DOCKER_CERT_PATH

(8) Set DOCKER_HOST environment variable to the IP+port that our manager is running on:
$ export DOCKER_HOST="tcp://172.17.42.10:9999"((("docker", "info")))

(9) Run this command to configure your shell:
$ eval $(docker-machine env --swarm <HOST_NODE_NAME>)
$ eval $(docker-machine env --swarm swarm-manager) 

(10) And you will be able to run normal Docker commands against the entire swarm cluster:
$ docker info

(11) Run an nginx container in our new cluster:
$ docker run -d nginx
$ docker ps
$ docker ps -a

 +New in v1.12:
  # You can now use TLS
  $ docker-machine env --swarm <HOST_NODE_NAME>
  $ export DOCKER_MACHINE_NAME="swarm-manager"
  $ export DOCKER_HOST="tcp://000.000.00.000:0000"
  $ export DOCKER_TLS_VERIFY="1"
  $ export DOCKER_CERT_PATH="/Users/lmaly/.docker/machine/machines/swarm-manager"
 
  # You can limit the set of nodes where a task can be scheduled by defining constraint expressions:
  $ docker service create --replicas 3 --name frontend --network mynet --publish 80:80/tcp --constraint engine.labels.com.example.storage==ssd frontend_image:latest
  $ docker service scale frontend=10

 +Things to consider:
  Load balancing & service discovery - Progrium's registrator, Consul
  Metrics & monitoring               - Datadog and friends
  Networked volumes                  - Flocker
  Network segregation & security     - AWS VPC
  Infrastructure automation          - Terraform


APPENDIX#3 - AWS
----------------

(1) Setup Amazon EC2 container service:
$ sudo yum -y install python
$ pip install awscli
$ aws --version
$ aws configure
$ aws iam list-users

(2) Start the cluster:
$ aws ecs create-cluster --cluster-name testing

(3) Add the existing Docker hosts to that cluster:
$ sudo docker run --name ecs-agent -d -v /var/run/docker.sock:/var/run/docker.sock -v /var/log/ecs/:/log -p 127.0.0.1:51678:51678 -e ECS_LOGFILE=/log/ecs-agent.log -e ECS_LOGLEVEL=info -e ECS_CLUSTER=testing amazon/amazon-ecs-agent:latest

(4) Check if the instance was added to the cluster:
$ aws ecs list-container-instances --cluster testing
$ aws ecs describe-container-instances --cluster testing --container-instances zse12345-12b3-45gf-6789-12ab34cd56ef78

(5) Upload task definition to Amazon:
$ aws ecs register-task-definition --family starwars-telnet --container-definitions file://$HOME/starwars-task.json

(6) List all task definitions:
$ aws ecs list-task-definitions

(7) Run the task in the cluster:
$ aws ecs run-task --cluster testing --task-definition starwars-telnet:1 --count 1

(8) Ensure the task transitioned to the running state by passing ARN task hash from the previous step:
$ aws ecs describe-tasks --cluster testing --task b64b1d23-bad2-872e-b007-88fd6

(9) Stop the task:
$ aws ecs stop-tasks --cluster testing --task b64b1d23-bad2-872e-b007-88fd6


APPENDIX#4 - DOCKER MACHINE
---------------------------

Docker-machine is client-side tool that you run on your local host and that allows you to start a server in a remote public cloud and use it as a Docker host if it were local.

(1) Generate an API access token for using Docker Machine in the Digital Ocean UI.
(2) Set environment variable DIGITALOCEAN_ACCESS_TOKEN in your local computer shell that defines the token you created and then create the driver:
$ export DIGITALOCEAN_ACCESS_TOKEN=sdf354sefg86w4erf351f534efw
$ docker-machine create -d digitalocean foobar
(4) To see how to connect Docker to the Docker machine:
$ docker-machine env foobar
(5) List the machine created previously, obtain it's IP & ssh to it:
$ docker-machine ls
$ docker-machine ip foobar
$ docker-machine ssh foobar
(6) Remove the machine you created:
$ docker-machine rm foobar

[lmaly@lmaly-au-linux ~]$ docker-machine --help
Usage: docker-machine [OPTIONS] COMMAND [arg...]
Run 'docker-machine COMMAND --help' for more information on a command.

Create and manage machines running Docker.

Version: 0.7.0, build a650a40

Author:
Docker Machine Contributors - <https://github.com/docker/machine>

Options:
--debug, -D                                       Enable debug mode
-s, --storage-path "/home/lmaly/.docker/machine"  Configures storage path [$MACHINE_STORAGE_PATH]
--tls-ca-cert                                     CA to verify remotes against [$MACHINE_TLS_CA_CERT]
--tls-ca-key                                      Private key to generate certificates [$MACHINE_TLS_CA_KEY]
--tls-client-cert                                 Client cert to use for TLS [$MACHINE_TLS_CLIENT_CERT]
--tls-client-key                                  Private key used in client TLS auth [$MACHINE_TLS_CLIENT_KEY]
--github-api-token                                Token to use for requests to the Github API [$MACHINE_GITHUB_API_TOKEN]
--native-ssh                                      Use the native (Go-based) SSH implementation. [$MACHINE_NATIVE_SSH]
--bugsnag-api-token                               BugSnag API token for crash reporting [$MACHINE_BUGSNAG_API_TOKEN]
--help, -h                                        show help
--version, -v                                     print the version

Commands:
active            Print which machine is active
config            Print the connection config for machine
create            Create a machine
env               Display the commands to set up the environment for the Docker client
inspect           Inspect information about a machine
ip                Get the IP address of a machine
kill              Kill a machine
ls                List machines
provision         Re-provision existing machines
regenerate-certs  Regenerate TLS Certificates for a machine
restart           Restart a machine
rm                Remove a machine
ssh               Log into or run a command on a machine with SSH.
scp               Copy files between machines
start             Start a machine
status            Get the status of a machine
stop              Stop a machine
upgrade           Upgrade a machine to the latest version of Docker
url               Get the URL of a machine
version           Show the Docker Machine version or a machine docker version
help              Shows a list of commands or help for one command

$ docker-machine [OPTIONS] COMMAND [arg...]
$ docker-machine create
$ docker-machine create -d virtualbox node1
$ docker-machine ls
$ docker-machine restart
$ docker-machine restart chefclient
$ docker-machine stop node1


APPENDIX#5 - DOCKER COMPOSE
---------------------------

[lmaly@lmaly-au-linux ~]$ docker-compose --help
Define and run multi-container applications with Docker.

Usage:
docker-compose [-f=<arg>...] [options] [COMMAND] [ARGS...]
docker-compose -h|--help

Options:
-f, --file FILE           Specify an alternate compose file (default: docker-compose.yml)
-p, --project-name NAME   Specify an alternate project name (default: directory name)
--verbose                 Show more output
-v, --version             Print version and exit

Commands:
build              Build or rebuild services
config             Validate and view the compose file
create             Create services
down               Stop and remove containers, networks, images, and volumes
events             Receive real time events from containers
help               Get help on a command
kill               Kill containers
logs               View output from containers
pause              Pause services
port               Print the public port for a port binding
ps                 List containers
pull               Pulls service images
restart            Restart services
rm                 Remove stopped containers
run                Run a one-off command
scale              Set number of containers for a service
start              Start services
stop               Stop services
unpause            Unpause services
up                 Create and start containers
version            Show the Docker-Compose version information

This will merge lists/YMLs together (latter one on the command line always overrides the earlier one if there are same variables), it also automatically does that with 'docker-compose.override.yml' (when you NOT use -f):
$ docker-compose -f <docker-compose1.yml> -f <docker-compose2.yml> config

Default project name is the directory name by default, you can change that:
$ docker-compose --project-name foo up -d

$ docker-compose config
$ docker-compose ps
$ docker-compose pause|unpause
$ docker-compose restart
$ docker-compose restart <service>
$ docker-compose restart node1
$ docker-compose up
$ docker-compose up -d
$ docker-compose down

$ docker-compose scale <SERVICE>=3
$ docker-compose run <SERVICE> ls /

$ docker-compose port <SERVICE> 80
$ docker-compose port --index=2 <SERVICE> 80    <-second worker's exposed port number on the host
$ docker-compose logs
$ docker-compose events

Environment variables in 'docker-compose':
==========================================
 1, TAG=latest <CMD>  - when ${TAG} is in the YML
 2, touch .env        - automatically read by compose
 3, touch staging.env - env_file: - "staging.env" in the YML
 4, environment:      - in the YML file
 
Extends example in 'docker-compose':
====================================
$ cat docker-compose.yml
version: '2'
services:
  hello:
    extends:
	  file: base.yml
	  service: base
	ports:
	  - 80
	environment:
	  SERVICE: hello

$ cat base.yml
version: '2'
services:
  base:
    image: tutum/hello-world
	environment:
	  SERVICE: base
	
$ docker-compose config    <-this will show the two merged

APPENDIX #6 - DOCKERFILE
------------------------
a/ FROM           - what base image will be used to create a new image
b/ MAINTAINER     - who is the owner of this new image
c/ WORKDIR        - sets the working directory for the same set of instructions that the USER can use (ADD, RUN, CMD, ENTRYPOINT)
    *ONBUILD      - stash set of commands that will be used when the image is used again as a base image in another Dockerfile
                  - if you want to give an image to i.e. developers and they all have different code that they want to test
                  - you can lay the groundwork ahead of needing the actual code, developers simply add their code in the directory that you tell them
                  - will be executed immediately after FROM
                  - can be used in conjunction with ADD, RUN
d/ RUN            - fetch and install packages or execute any other commands
                  - every time you utilize RUN, it creates a new layer, it is better to chain commands (; and \)
e/ LABEL          - version/description, it will be shown by docker inspect <image>
f/ COPY or ADD    - add files or folders into the image, ADD can even download and unpack from the URL
                  - COPY is the same without URL and unpacking capability
g/ EXPOSE         - expose ports from the image to the outside world
h/ ENTRYPOINT     - use in conjunction with CMD, set additional switches that may change over time based on the CMD command
    *CMD          - executes the given commands and keeps the container running
i/ USER           - what username to use when a command is run (influences RUN, CMD, ENTRYPOINT)

ENTRYPOINT vs. CMD
------------------
ENTRYPOINT defines the executable invoked when the container is started:
e.g. ENTRYPOINT ["node", "app.js"] => This runs the 'node' process directly and not inside the shell!
CMD specifies the arguments that get passed to the ENTRYPOINT.
Footer
© 2022 GitHub, Inc.
Footer navigation
Terms
Privacy
Security
Status
Docs
Contact GitHub
Pricing
API
Training
Blog
About
docker-cheatsheet/docker-cheatsheet.txt at master · luckylittle/docker-cheatsheet
