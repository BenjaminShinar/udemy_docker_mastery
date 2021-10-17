<!--
ignore these words in spell check for this file
// cSpell:ignore
-->

## Command line

the management commands are a new way to sub section commands.
commands (power shell):

- _docker version_ - return versions of the clients and server dockers.
- _docker info_ - more information
- _docker_ - get list of commands, including management commands.
- _docker \<command> \<sub-command> (options)_ - new way, with management commands.
- _docker \<command> (options)_ - old way.
- _docker \<command> --help_ - information on command.
- _docker login_ - login to hub
- _docker logout_ - logout from hub

### Docker Container

- _docker container run_ - start new a container, with an image
  - _--detach_, _-d_ - run in background
  - _--publish_, _-p_ - port forwarding (host:container),
  - _--name_ - specify name of the container
  - _--env_,-e - pass environments variables
  - _-it_ - start new container interactively.
  - _--network, --net_ \<name|NONE> - at a specific network
  - _--network-alias,--net-alias_ \<alias> - Add network-scoped alias for the container.
  - _--link \<list>_ - enable dns on default bridge network.
  - _--rm_ - Automatically remove the container when it exits.
  - _--volume, -v_ - connect to a named volume.
- _docker container ls_ - list all the containers (new format). by default only shows running containers.
  - _-a_ - all, show also stopped containers.
  - _docker ps_ - list all containers (old format)
- _docker container stop <#id>_ - stop a container.
  - _docker stop <#id>_ - stop a container (old format).
- _docker container logs \<name>_ - see logs for container.
  - _docker logs \<name>_ - see logs for container (old format).
  - _--follow,-f_ - follow, keep getting logs.
- _docker container top \<name>_ - see the processes running inside the container.
- _docker container rm <#id1> <#id2>_ - remove containers, we can specify more than one container.
  - _-f_ - flag to remove running containers.
- _docker start \<name>_ - run a stopped container. (old format)
- _docker container inspect <#id>_- details of one container config meta data (JSON).
  - _--format {{...}}_ - to get a single node object from inside
    - _--format {{ **json** ...}}_ - to format searched objects as json
- _docker container stats_ - details of one or all of our container config.
- _docker container exec -it_ - run an additional command in existing container
- _docker container port \<name>_ - which ports are forwarding traffic.

### Docker Network

- _docker network ls_ - list networks
- _docker network inspect_ - inspect network
- _docker network create \<network name>_ - create a network
  - _--driver,-d_ - which driver it should use, default bridge.
- _docker network connect \<network> \<container>_- attach network to container
- _docker network disconnect \<network> \<container>_ - detach network to container

### Docker Image

- _docker image ls_ - see list of images.
- _docker image inspect_ - inspect image for metadata.
- _docker image pull \<image name>_ - download image from hub.
- _docker image history \<image name>_ - show history union (the changes).
- _docker image tag \<source image> \<tag>_ - create a tag image based on an existing image. we can add tags. if we don't change it, the id is still the same.
- _docker image push_ - upload a changed image layers to registry.
- _docker image build \<destination>_ - build an image from file
  - _--tag, -t_ - specify tag.
  - _--file, -f_ - specify docker file to build from.
- _docker image prune_ - clean dangling images.
  - _--all, -a_ - remove all unused images, not just dangling.
  - _--force, -f_ - skip confirmation.
  - _--filter_ - provide filter.

### Docker System

- _docker system prune_ - remove unused data
  - _--all, -a_ - remove all unused , not just dangling.
  - _--force, -f_ - skip confirmation.
  - _--filter_ - provide filter.
  - _--volumes_ - prune volumes.
- _docker system df_ - see space usage

### Docker Machine

- _docker-machine rm default_
- _docker-machine create_

### Docker Volume

- _docker volume ls_ - list volumes
  - _--filter, -f_ - filter.
  - _--format_ - print format.
  - _--quiet, -q_ - show only names.
- _docker volume inspect_ - inspect
- _docker volume create_ - create volume before running to use custom drivers and labels

## Docker File

Stanzas:

- FROM
- ENV
- RUN
- EXPOSE
- CMD
- WORKDIR
- COPY
- VOLUME
- ENTRYPOINT

## Docker Compose YML

set up the version of the yml

- version:

which containers to run

- services:
  - image
  - environment
  - ports
  - networks
  - depends_on
  - build
    - context
    - dockerfile

data volumes

- volumes:

networks

- networks:

## Docker Compose Command Line

- _docker-compose up_ - setup volumes/networks and start all containers.
  - _-d_ - run detached, in the background.
- _docker-compose down_ - stop and remove all containers, also remove networks and volumes.
  - _--volumes, -v_ - clean up volumes as well
  - _--rmi \<type>_ - remove images
    - _all_ - all images used by the services
    - _local_ - only images that don't have a custom tag set (locally built)
- _docker-compose logs_ - see logs
- _docker-compose build_ - build images
