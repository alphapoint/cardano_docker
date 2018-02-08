 # Cardano Node Docker 

This repository contains docker files used to build docker image of the Cardano-sl node.
- *cardano-sl-wallet* folder : contains docker file to build image of cardano-sl full node with wallet feature
- *cardano-sl-explorer* folder : contains docker file to build image of cardano-sl full node with explorer feature

### Requirements

  Only Docker is required to build and run the Cardano full node.
  To install Docker follow those instructions :  [Install Docker].
  
### How to build Docker image

First you need to build docker image from a docker file, docker file contains all instructions to download projects sources, dependencies and then builds the project binaries. Build of docker image takes some time, be patient ...
The docker image used to host cardano node is Ubuntu 16.04.

Commands below have to executed in the same folder as the Dockerfile file.

For wallet :
```sh
$ cd ./cardano-ls-wallet
$ docker build .
````
For explorer : 
```sh
$ cd ./cardano-ls-explorer
$ docker build .
````



After build is finished you can see the list of available images on your Docker local repository. In this example our build name (or image Id) is d899e7637a94. 
```sh
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
<none>              <none>              d899e7637a94        20 hours ago        3.3GB
ubuntu              16.04               2a4cca5ac898        8 days ago          111MB
````

### How to run Docker Cardano node 
#### A. Wallet node
Before running Docker container hosting Cardano node it's important to define a volume on the host machine where to bind data : this means blockchain database and other data will be persisted on host environment and not inside the container itself. In case you need to stop/update or run again you container you won't loose you data.
You need to create a folder somewhere on your environment and give access right to 'root' user.
```sh
$  sudo mkdir /data/cardano_docker_db
$  sudo chmod 777  /data/cardano_docker_db
````

To start docker container and run Cardano full node run the following command. Successful run should return you the id of the container.
```sh
$  docker run -d --name=cardano -v /data/cardano_docker_db:/home/cardano/cardano-sl/state-wallet-mainnet:Z -p 127.0.0.1:8090:443 d899e7637a94
04ca92a3d8d2b7d181de181ed480c45034a12f45e4851522ac29d5a26ecf39a5
````
Command arguments :
- *-d* : run container in background
- *--name* : define the name of the Docker container
- *-p* : port binding, bind the 8090 of the host to the port 443 inside the container. Port 443 is the default port that Nginx (installed and running in the container) opens for SSL connexion, please be sure to bind on this port. Inside the container Nginx will forward the incoming request to port 8090 where Cardano wallet API can be accessed. Be careful that desired port are available on the host environment
- *-v* : bind volume, this option allows to bind volume/folder from inside the container to a volume on the host enivornment. This way you can keep all data (blockchain db , wallet db) out of the container, you can delete and restart container without loosing your data.
- *d899e7637a94* : docker image id in this example

The next command will show you the containers hosted on Docker and there status.    

```sh
$  docker ps -a 
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                      NAMES
04ca92a3d8d2        d899e7637a94        "/bin/sh -c ./connec…"   10 minutes ago      Up 10 minutes       127.0.0.1:8090->8090/tcp   cardano
````
#### B. Explorer node

Create folder to bind (to presist D) outsidbe container.
```sh
$  sudo mkdir /data/cardano_docker_explorer_db
$  sudo chmod 777 /data/cardano_docker_explorer_db
````

To start docker container and run Cardano full node run the following command. Successful run should return you the id of the container.
```sh
$  docker run -d --name=cardano-explorer -v /data/cardano_docker_explorer_db:/home/cardano/cardano-sl/state-explorer-mainnet/:Z -p 127.0.0.1:8100:8100 d899e7637a94
04ca92a3d8d2b7d181de181ed480c45034a12f45e4851522ac29d5a26ecf39a5
````
Command arguments :
- *-d* : run container in background
- *--name* : define the name of the Docker container
- *-p* : port binding, bind the 8100 of the host to the port 8100 inside the container. Be careful that desired port are available on the host environment
- *-v* : bind volume, this option allows to bind volume/folder from inside the container to a volume on the host enivornment. This way you can keep all data (blockchain db , wallet db) out of the container, you can delete and restart container without loosing your data.
- *d899e7637a94* : docker image id in this example

The next command will show you the containers hosted on Docker and there status.    

```sh
$  docker ps -a 
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                      NAMES
04ca92a3d8d2        d899e7637a94        "/bin/sh -c ./connec…"   10 minutes ago      Up 10 minutes       127.0.0.1:8100->8100/tcp   cardano
````


### Verify node is running 
#### A. Wallet node
Cardano node running inside the container has the wallet feature available. You can connect node using wallet API on port 8090 (if you run container with this port binded). 

To check if the node is running properly you can run the following command (Not : this CURL command ignore TSL certificate) :
```sh
$  curl -k https://localhost:8090/api/settings/sync/progress
{"Right":{"_spLocalCD":{"getChainDifficulty":{"getBlockCount":182509}},"_spNetworkCD":{"getChainDifficulty":{"getBlockCount":527963}},"_spPeers":0}}c
````
This command should return the synchronizarion state of you node : total blockchain lenght and the blocks you already fetched.
For more information about Cardano wallet api : [Cardano Wallet API]

If you want to connect to server using certificate you can tun the following command :
```sh
$  curl --cacert /data/cardano_docker_explorer_db/tls/server.cert https://localhost:8090/api/settings/sync/progress
{"Right":{"_spLocalCD":{"getChainDifficulty":{"getBlockCount":182509}},"_spNetworkCD":{"getChainDifficulty":{"getBlockCount":527963}},"_spPeers":0}}c
````

#### B. Explorer node

Cardano node running inside the container has the explorer feature available. You can connect node using explorer API on port 8100 (if you run container with this port binded). To check if the node is running properly you can run the following command :
```sh
$ curl -k http://localhost:8100/api/txs/last
{"Right":[{"cteId":"b2772547040ac92515b3becbc5e3cf8df8866c26f56f8efd65c55f7724a5f797","cteTimeIssued":1507930691,"cteAmount":{"getCoin":"23749618279429"}},{"cteId":"f7dc6cd52df36fbb6b0049011b5367147485b31cb9ba586285fcb99327b3a0f6","cteTimeIssued":1507930571,"cteAmount":{"getCoin":"23750389693241"}},{"cteId":"80a7cb26946c989d4109dac602989ba4110b9b3d3113243b8b26d686706a7a...
````
The above command should return you the list of last transaction on the blockchain.
For more information about Cardano wallet api : [Cardano Explorer API]

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




**Emurgo Vietnam - 2018**

   [Install Docker]: <https://docs.docker.com/engine/installation/>
   [Cardano Wallet API]: <https://cardanodocs.com/technical/wallet/api/#>
   [Cardano Explorer API]: <https://cardanodocs.com/technical/explorer/api/>
 # Cardano Node Docker 

This repository contains docker files used to build docker image of the Cardano-sl node.
- *cardano-sl-wallet* folder : contains docker file to build image of cardano-sl full node with wallet feature
- *cardano-sl-explorer* folder : contains docker file to build image of cardano-sl full node with explorer feature

### Requirements

  Only Docker is required to build and run the Cardano full node.
  To install Docker follow those instructions :  [Install Docker].
  
### How to build Docker image

First you need to build docker image from a docker file, docker file contains all instructions to download projects sources, dependencies and then builds the project binaries. Build of docker image takes some time, be patient ...
The docker image used to host cardano node is Ubuntu 16.04.

For wallet :
```sh
$ cd ./cardano-ls-wallet
$ docker build - < DOCKERFILE
````
For explorer : 
```sh
$ cd ./cardano-ls-explorer
$ docker build - < DOCKERFILE
````



After build is finished you can see the list of available images on your Docker local repository. In this example our build name (or image Id) is d899e7637a94. 
```sh
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
<none>              <none>              d899e7637a94        20 hours ago        3.3GB
ubuntu              16.04               2a4cca5ac898        8 days ago          111MB
````

### How to run Docker Cardano node 
#### A. Wallet node
Before running Docker container hosting Cardano node it's important to define a volume on the host machine where to bind data : this means blockchain database and other data will be persisted on host environment and not inside the container itself. In case you need to stop/update or run again you container you won't loose you data.
You need to create a folder somewhere on your environment and give access right to 'root' user.
```sh
$  sudo mkdir /data/cardano_docker_db
$  sudo chmod 777  /data/cardano_docker_db
````

To start docker container and run Cardano full node run the following command. Successful run should return you the id of the container.
```sh
$  docker run -d --name=cardano -v /data/cardano_docker_db:/home/cardano/cardano-sl/state-wallet-mainnet:Z -p 127.0.0.1:8090:443 d899e7637a94
04ca92a3d8d2b7d181de181ed480c45034a12f45e4851522ac29d5a26ecf39a5
````
Command arguments :
- *-d* : run container in background
- *--name* : define the name of the Docker container
- *-p* : port binding, bind the 8090 of the host to the port 443 inside the container. Port 443 is the default port that Nginx (installed and running in the container) opens for SSL connexion, please be sure to bind on this port. Inside the container Nginx will forward the incoming request to port 8090 where Cardano wallet API can be accessed. Be careful that desired port are available on the host environment
- *-v* : bind volume, this option allows to bind volume/folder from inside the container to a volume on the host enivornment. This way you can keep all data (blockchain db , wallet db) out of the container, you can delete and restart container without loosing your data.
- *d899e7637a94* : docker image id in this example

The next command will show you the containers hosted on Docker and there status.    

```sh
$  docker ps -a 
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                      NAMES
04ca92a3d8d2        d899e7637a94        "/bin/sh -c ./connec…"   10 minutes ago      Up 10 minutes       127.0.0.1:8090->8090/tcp   cardano
````
#### B. Explorer node

Create folder to bind (to presist D) outsidbe container.
```sh
$  sudo mkdir /data/cardano_docker_explorer_db
$  sudo chmod 777 /data/cardano_docker_explorer_db
````

To start docker container and run Cardano full node run the following command. Successful run should return you the id of the container.
```sh
$  docker run -d --name=cardano-explorer -v /data/cardano_docker_explorer_db:/home/cardano/cardano-sl/state-explorer-mainnet/:Z -p 127.0.0.1:8100:8100 d899e7637a94
04ca92a3d8d2b7d181de181ed480c45034a12f45e4851522ac29d5a26ecf39a5
````
Command arguments :
- *-d* : run container in background
- *--name* : define the name of the Docker container
- *-p* : port binding, bind the 8100 of the host to the port 8100 inside the container. Be careful that desired port are available on the host environment
- *-v* : bind volume, this option allows to bind volume/folder from inside the container to a volume on the host enivornment. This way you can keep all data (blockchain db , wallet db) out of the container, you can delete and restart container without loosing your data.
- *d899e7637a94* : docker image id in this example

The next command will show you the containers hosted on Docker and there status.    

```sh
$  docker ps -a 
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                      NAMES
04ca92a3d8d2        d899e7637a94        "/bin/sh -c ./connec…"   10 minutes ago      Up 10 minutes       127.0.0.1:8100->8100/tcp   cardano
````


### Verify node is running 
#### A. Wallet node
Cardano node running inside the container has the wallet feature available. You can connect node using wallet API on port 8090 (if you run container with this port binded). 

To check if the node is running properly you can run the following command (Not : this CURL command ignore TSL certificate) :
```sh
$  curl -k https://localhost:8090/api/settings/sync/progress
{"Right":{"_spLocalCD":{"getChainDifficulty":{"getBlockCount":182509}},"_spNetworkCD":{"getChainDifficulty":{"getBlockCount":527963}},"_spPeers":0}}c
````
This command should return the synchronizarion state of you node : total blockchain lenght and the blocks you already fetched.
For more information about Cardano wallet api : [Cardano Wallet API]

If you want to connect to server using certificate you can tun the following command :
```sh
$  curl --cacert /data/cardano_docker_explorer_db/tls/server.cert https://localhost:8090/api/settings/sync/progress
{"Right":{"_spLocalCD":{"getChainDifficulty":{"getBlockCount":182509}},"_spNetworkCD":{"getChainDifficulty":{"getBlockCount":527963}},"_spPeers":0}}c
````

#### B. Explorer node

Cardano node running inside the container has the explorer feature available. You can connect node using explorer API on port 8100 (if you run container with this port binded). To check if the node is running properly you can run the following command :
```sh
$ curl -k http://localhost:8100/api/txs/last
{"Right":[{"cteId":"b2772547040ac92515b3becbc5e3cf8df8866c26f56f8efd65c55f7724a5f797","cteTimeIssued":1507930691,"cteAmount":{"getCoin":"23749618279429"}},{"cteId":"f7dc6cd52df36fbb6b0049011b5367147485b31cb9ba586285fcb99327b3a0f6","cteTimeIssued":1507930571,"cteAmount":{"getCoin":"23750389693241"}},{"cteId":"80a7cb26946c989d4109dac602989ba4110b9b3d3113243b8b26d686706a7a...
````
The above command should return you the list of last transaction on the blockchain.
For more information about Cardano wallet api : [Cardano Explorer API]

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




**Emurgo Vietnam - 2018**

   [Install Docker]: <https://docs.docker.com/engine/installation/>
   [Cardano Wallet API]: <https://cardanodocs.com/technical/wallet/api/#>
   [Cardano Explorer API]: <https://cardanodocs.com/technical/explorer/api/>

