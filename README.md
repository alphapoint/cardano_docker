# Cardano Node Docker 

This repository contains docker files used to build docker image of the Cardano-sl node.
- *cardano-sl* folder : contains docker file to build image of cardano-sl full node with wallet feature

### Requirements

  Only Docker is required to build and run the Cardano full node.
  To install Docker follow those instructions :  [Install Docker].
  
### How to build Docker image

First you need to build docker image from a docker file, docker file contains all instructions to download projects sources, dependencies and then builds the project binaries. Build of docker image takes some time, be patient ...
The docker image used to host cardano node is Ubuntu 16.04.

```sh
$ cd ./cardano-ls
$ docker build -f DOCKERFILE
````
After build is finished you can see the list of available images on your Docker local repository. In this example our build name (or image Id) is d899e7637a94. 
```sh
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
<none>              <none>              d899e7637a94        20 hours ago        3.3GB
ubuntu              16.04               2a4cca5ac898        8 days ago          111MB
````

### How to run Docker Cardano node 
To start docker container and run Cardano full node run the following command. Successful run should return you the id of the container.
```sh
$  docker run -d --name=cardano -p 127.0.0.1:8090:8090 d899e7637a94
04ca92a3d8d2b7d181de181ed480c45034a12f45e4851522ac29d5a26ecf39a5
````
Command arguments :
- *-d* : run container in background
- *--name* : define the name of the Docker container
- *-p* : port binding, bind the 8090 of the host to the port 8090 inside the container. Be careful that desired port are available on the host environment.
- *d899e7637a94* : docker image id in this example

The next command will show you the containers hosted on Docker and there status.    

```sh
$  docker ps -a 
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                      NAMES
04ca92a3d8d2        d899e7637a94        "/bin/sh -c ./connec…"   10 minutes ago      Up 10 minutes       127.0.0.1:8090->8090/tcp   cardano
````
### Verify node is running 

Cardano node running inside the container has the wallet feature available. You can connect node using wallet API on port 8090 (if you run container with this port binded). To check if the node is running properly you can run the following command :
```sh
$  curl -k https://localhost:8090/api/settings/sync/progress
{"Right":{"_spLocalCD":{"getChainDifficulty":{"getBlockCount":182509}},"_spNetworkCD":{"getChainDifficulty":{"getBlockCount":527963}},"_spPeers":0}}c
````
This command should return the synchronizarion state of you node : total blockchain lenght and the blocks you already fetched.
For more information about Cardano wallet api : [Cardano Waller API]


### Display container logs 
```sh
$  docker logs -f cardano
...
[node.worker.block:DEBUG:ThreadId 2553] [2018-01-24 03:05:58 UTC] Checkpoints available, headers request assembled
[node.worker.block:DEBUG:ThreadId 2553] [2018-01-24 03:05:58 UTC] Updating recovery header...
````

### Connect inside your container 
```sh
$  docker exec -it cardano /bin/bash
cardano@04ca92a3d8d2:~/cardano-sl$ ...
````

### Start/stop container

The followings command allow you to start and stop the Cardano container.
Note that restarting a stopped Cardano node is not sure to success, it depends on the node state and it can happen that some lock files are not released properly inside the Cardano container, this makes impossible to restart the container. 
```sh
$  docker stop cardano 
$  docker start cardano 
````

### Comming soon :
- Bind external volume to save database/blockchain
- Write Dockerbuild for explorer 



**Emurgo Vietnam - 2018**

   [Install Docker]: <https://docs.docker.com/engine/installation/>
   [Cardano Waller API]: <https://cardanodocs.com/technical/wallet/api/#>

