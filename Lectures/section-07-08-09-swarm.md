<!--
ignore these words in spell check for this file
// cSpell:ignore psql examplevotingapp_vote  examplevotingapp bretfisher
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

<!-- <details> -->
<summary>
//TODO: add Summary
</summary>

How to use the swarm features in our work flow.

new features

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

<!-- <details> -->
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
docker network create --driver overlay backend
docker network create --driver overlay frontend

docker service create --name vote --network frontend --replicas 3 --publish 5000:80 bretfisher/examplevotingapp_vote

docker service create --name redis --network frontend --replicas 1 redis:3.2

docker service create --name worker --network frontend --network backend --replicas 1 bretfisher/examplevotingapp_vote

docker service create --name db --network backend --replicas 1 --mount type=volume,source=db-data,target=/var/lib/postgresql/data -e POSTGRES_HOST_AUTH_METHOD=trust postgres:9.4

docker service create --name result --network backend -p 5001:80 --replicas 1 bretfisher/examplevotingapp_result

```

</details>

##

</details>

##

</details>

### Swarm Stack and Secrets

<details>
<summary>
//TODO: add Summary
</summary>

</details>

#

</details>
