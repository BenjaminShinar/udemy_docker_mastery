<!--
ignore these words in spell check for this file
// cSpell:ignore psql  voteapp
-->

## Swarms

<!-- <details> -->
<summary>
//TODO: add Summary about swarms
</summary>

Swarm mode - a built-in clustering solution inside Docker. not related for 'swarm' add on from versions before pre-1.12 versions.

note:
there were some changes in the docker service commands of create / update.
mainly the _--detach_ flag, we should set it to true when running.

### Swarm Intro and Creating a 3-Node Swarm Cluster

<details>
<summary>
Intro to swarms
</summary>
we want to deploy our applications as if we were a cloud provider, no matter the environment, we don't want any differences that come from the platform

- automating container lifecycle?
- what about scaling up/down and in/out?
- re-creating failed containers.
- replacing containers without downtime for updates/upgrades (blue/green deploy).
- tracking and controlling containers.
- cross node virtual networks.
- security: do containers run only in trusted servers?

- security: storing secrets(keys, passwords) and making them available for only the right container.

#### Built-In Orchestration

<details>
<summary>
The basic terms for Swarms
</summary>

swarm kit was added in 2016 as part of the docker library, was then enhanced in 2017 with 'stacks' and 'secrets'.

we can't use swarm commands by default, we will get errors.

> - docker swarm
> - docker node
> - docker service
> - docker stack
> - docker secret

basic concepts:

> - Manager nodes - have built-in database (RAFT), the authority of the swarm. manage the swarm.
> - TLS
> - Certificate Authority
> - Worker nodes -
> - The Control Plain -
> - Gossip network
> - replica / task - a wrap over a container.

we can promote and demote managers and workers. containers are now managed by the swarm manager, via the docker service commands, which add extra features on top of the docker container commands. the manager nodes places/creates nodes with task (replicas).

> we start with the `docker service create` command line, which creates the managerial level nodes.
>
> - Api - accept commands from the client and create service objects
> - Orchestrator - reconciliation loop for service objects and creating tasks
> - Allocator - allocates IP address to tasks.
> - Scheduler - Assigns nodes to tasks
> - Dispatcher - Checks on worker nodes.
>
> in the worker node level:
>
> - Worker - connect to the dispatcher to check on assigned tasks
> - Executor - executes the tasks assigned to the worker node

</details>

#### Create Your First Service and Scale It Locally

<details>
<summary>
playing with our first swarm.
</summary>

creating a single node swarm.

we check if a swarm is active by type `docker info` and looking at the swarm attribute. we can initialize a swarm with one node in a simple command line, we see the newly created node as a manager

```sh
docker info --format "{{ .Swarm }}"
docker swarm init
docker info --format "{{json .Swarm }}"
docker node ls
```

our swarm has a Root signing certificate, certification in the first manager node, join tokens.

RAFT database ensures consistency across our swarm (with configurations), will wait for other nodes, logs are replicated amongst managers.

when we list our nodes we see the manager status, there can only be one leader at any given time.

a service in a swarm replaces docker container run. multiple containers (a cluster) instead of individual containers.

we can create a worker service: the id of the service is not the same as that of the container. we see the replicas columns, the ratio of running tasks vs requested

```sh
docker service create alpine ping 8.8.8.8
docker service ls
docker container ls
docker service ps #name
```

when we run the `docker service ps` command, we see the containers attached to the node. we can match them by names.

to scale up, we run the update command and specify the number of replicas, we will then see three tasks.

```sh
docker service update \<service> --replicas 3
docker service ls
docker service ps #name
docker container ls
```

for a local machine, we can use containers as we want. for a production environment, we always want our services to be running at some capacity (the blue green rolling update pattern).

The docker container also has an _update_ command, for changes (without removing and starting again), they mostly relate to resources. for the docker service update command, there are much more options.

if we try to remove a container manually with `docker container rm -f`, the swarm will identify that and create a new one to replace it. we will see the failed one in the `docker service ps` command with the error of "task: non-zero exit (137)". this is part of the orchestration, we don't talk to containers directly, we specify the state of the of system.

if we remove a service, the containers will also go down in a short while

```sh
docker service rm #name
```

</details>

#### Creating a 3-Node Swarm Cluster

<details>
<summary>
Getting nodes on the cloud and playing with them
</summary>

we will now set a 3-node swarm. but we can't do this on our local machine.

- we can use the website [play-with-docker](https://www.docker.com/play-with-docker), it has a time limit of 4 hours per session.
- we can also use docker-machine with virtual box.
- we can also use Digital ocean, which is a cloud service that we pay for nodes.
- at most, we can use a docker machine with other cloud providers.

Play with docker

- press add new instances to create more nodes.
- run `docker info`
- `ping` other nodes by ip.
- we care about the ports

to init, we need an ip and open ports. we init from any node the swarm by specifying the ip address. then we go to the other swarms and join as worker with the command that we got

```sh
docker swarm init --advertise-addr #ip address

#from the other node
docker swarm join --token #token #ip

#from manager node
docker node ls
```

to update a node to be a manager we can use docker node update, now the manager node is reachable,

```sh
docker node update --role manager node2
docker node ls

docker swarm join-token manager
docker service create --replicas 3 alpine ping 8.8.8.8
```

</details>

</details>

### Swarm Basic Features

<details>
<summary>
Basic behavior of Swarm
</summary>

How to use the swarm features in our work flow.

overlay - when creating a network, we add the _--driver overlay_ flag.\
routing mesh

#### Scaling Up with Overlay

<details>
<summary>
Using overlay to control bridge networks.
</summary>

overlay is used for swarm-wide bridge network, for container-to-container traffic inside a single swarm (not so much incoming connections). we can enables encryption with Optional IPSec. a service can be connected to more than one overlay network.

lets try the example of drupal.

```sh
#in node1- the leader
docker network create --driver overlay my_drupal_nw
docker network ls
docker service create --name psql --network my_drupal_nw -e POSTGRES_PASSWORD=myPass postgres
docker service ls
docker service ps psql
docker container logs psql

docker service create --name drupal --network my_drupal_nw -p 80:80 drupal
```

service are are created by the orchestrator, so we don't see the downloading parts. we can then do the parts from before with setting up the drupal database.

we can go in the browser to all ips and that will look like all of them refer to the same database, but it's actually running on just one host.

</details>

#### Scaling Out with Routing Mesh

<details>
<summary>
Routing incoming packets between different containers in the same network
</summary>

the routing mesh (_ingress_, incoming) routes packets for a service to a proper task, in our case, the database.
in spans all nodes in the swarm, and uses IPVS from the linux kernel, it performs load balancing.

this works in two modes:

- container to container in a overlay network (using vip: virtual ip).
- external traffic incoming to published ports, all nodes listen and then re-route to the proper container.

if it's on the same node, packets are routed to the correct port, if it's not the correct node, it'll be routed to the correct node via the overlay network.

similar to dns.

lets do another one this time with elastic search.

```sh
docker service create --name search --replicas 3 -p 9200:9200 elasticsearch:2
docker service ps search
curl localhost:9200
```

> - the routing mesh is currently a stateless load balancing.
> - it's an OSI layer 3 load balancer (TCP), not a layer 4 (DNS). won't work for multiple websites on the same host and port.
>
> both limitations can be solved
>
> - nginx or HAProxy Load balancing proxy
> - use docker enterprise edition that has a layer 4 web proxy

</details>

#### Assignment #1: Create A Multi-Service Multi-Node Web App

<details>
<summary>
creating the assets we need for running a multi-tier app.
</summary>

> - Using Docker Distributed Voting App.
> - the folder is swarm-app-1 folder
> - 1 volume, 2 networks, 5 services needed.
> - create the commands needed, spin up the services, and test app.
> - everything uses docker hub images, no data need on the swarm.

[example voting app](https://hub.docker.com/r/docker/example-voting-app-vote), [redis](https://hub.docker.com/_/redis)

```sh

docker swarm init --advertise-addr 192.168.0.13 #ip address
docker swarm join-token manager

#copy and run the command in node2, node3

docker network create --driver overlay backend
docker network create --driver overlay frontend

docker service create --name vote --network frontend --replicas 3 --publish 80:80 bretfisher/examplevotingapp_vote

docker service create --name redis --network frontend --replicas 1 redis:3.2

docker service create --name worker --network frontend --network backend --replicas 1 bretfisher/examplevotingapp_worker

docker service create --name db --network backend --replicas 1 --mount type=volume,source=db-data,target=/var/lib/postgresql/data -e POSTGRES_HOST_AUTH_METHOD=trust postgres:9.4

docker service create --name result --network backend -p 5001:80 --replicas 1 bretfisher/examplevotingapp_result

#open ip by pressing the open ports button and typing 80 and 5001
docker service ps worker
docker service logs worker

```

the default is one replica, so if we don't want more, we can ignore this.
there is an issue with the _--volume_ flag on swarms, so we use the _--mount_ flag with key-value map, we must have type, source,target.\
the routing mesh doesn't play well with web socket, so that's another issue.

</details>

</details>

### Swarm Stack and Secrets

<details>
<summary>
Stacks are ways to configure the swarm in a file. secrets are stored securly and can only be ace
</summary>

stacks of service, another layer of abstraction, stack accept compose files as declarative definition for services, networks, volumes.

#### Swarm Stacks and Production Grade Compose

<details>
<summary>
A basic configuration,
</summary>

`docker stack deploy` replaces `docker service create` as the command to start running.

the stack manages all the objects for us, including overlay networks per stack. it adds the stack name to the start of their name.
the compose file now has **deploy** key, and it removes the **build**, we shouldn't build images in the swarm. but the docker compose knows to ignore _deploy_, and the swarm compose knows to ignore _build_. it's fine.

we don't need the _docker-compose cli_ on the swarm server, the docker stack command knows how to read the yml files. a stack has the services (each with tasks/replicas), the volumes and the overlay networks (also secrets)

a stack works with one swarm only.

the yml must be version 3 or higher to use stacks. we can control the deploy, decide where it's deployed (which node role), set delay time for warm up, etc...

the stack doesn't run them immediately, it creates the objects and passes them to the scheduler.

```sh
docker stack deploy -c example-voting-app-stack.yml voteapp
docker stack services
```

running deploy again updates the stack if there were changes to the yml file

</details>

#### Secrets Storage for Swarm: Protecting Your Environment Variables

<details>
<summary>
creating and using secrets.
</summary>

Secret storage, built into the swarm.

> "The easiest "secure" solution for storing secrets in Swarm"

encrypted on disk, on transit, built in into the infrastructure

> Secrets: any data you would prefer to keep to yourself
>
> - Usernames and passwords
> - TLS certificates and keys
> - SSH keys
> - OAuth API Keys
> - and more...

supports generic strings or binary content (up to 500k, half a megabyte), doesn't require the app to talk to somewhere else.

the Swarm Raft DB is encrypted on disk by default. only stored on the manager nodes. the keys are passed to the workers "control plane" via TLS + mutual auth. we store them in Swarm, and the assign the secrets to the services.for workers stored in memory only, not on disk. local docker-compose can use file-based secrets, but not securely.

secret belong to swarm, docker-compose can 'read' secrets from 'files', but it won't be secure.

to create a secret, we can pass a file or pass the secret directly (the **-** option). we can list the secrets or inspect them, but we will never see the content itself.

```sh
docker secret create psql_user psql_user.txt
echo "myDBpassWORD" | docker secret create psql_pass -
docker secret ls
docker secret inspect
```

only the services can view the content of the secrets, they are exposed to them as if they were files on the disk in _/run/secrets/secret_name_ path. if we go into the nodes with `exec` we can view the content.

```
docker service create --name psql --secret psql_user --secret psql_pass -e POSTGRES_PASSWORD_FILE=/run/secrets/psql_pass -e POSTGRESS_USER_FILE=/run_secrets/psql_user postgres
```

we can remove the secret or add other, this would redeploy the service. so it's not great

```sh
docker service update --secret-rm
```

</details>

#### Using Secret with Swarm Stacks

<details>
<summary>
using secrets in stacks
</summary>

we can also have secrets with stacks, looking at "secrets-sample-2" folder, the minimal version is 3.1 for using secrets with stack.

we can either use files of pre-create them, and then use externally by referring to them. there is a longer form to protect secrets with permissions and stuff

```sh
docker stack deploy -c docker-compose.yml my_db
docker secret ls
docker stack rm my_db
```

we should always clean up and remove secrets.

</details>

#### Assignment #2: Create A Stack with Secrets and Deploy

<details>
<summary>
Adding secrets to stack in the docker-compose.yml file.
</summary>

> - using the drupal compose file from last assignment _compose-assignment-2_
> - rename the image back to official drupal:8.2
> - remove the _build:_ key
> - add secret via external
> - use environment variable _POSTGRES_PASSWORD_FILE_
> - add secret via cli with `echo "<pw>" | docker secret create psql-pw -`
> - copy compose into a new tml file on swarm node1

- [x] change version to 3.1
- [x] change image.
- [x] add secrets section to the docker compose file
  - [x] add psql-pw secret as external
- [x] add secrets to service
- [x] set environment _POSTGRES_PASSWORD_FILE_ to _run/secrets/psql-pw_
- [x] add secret in the swarm manager cli
- [x] open file on vim and copy the docker-compose content.yml / or touch to create file and then open the _editor_

vim:

- `:x` to save and exit
- `:q!` to quit

</details>

#

##

swarmstacks and sectert

</details>

##

swarms

</details>
