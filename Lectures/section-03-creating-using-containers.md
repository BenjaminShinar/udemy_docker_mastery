<!--
ignore these words in spell check for this file
// cSpell:ignore distro Comms ddress centos nslookup hostnames
-->

## Creating and Using Containers Like a Boss

<details>
<summary>
Creating containers from the command line, stepping into them, removing them and doing network stuff.
</summary>

Containers are the fundamental building blocks of docker.

- we will get our container working
- create our first web server (container) with Nginx
- learn common container management commands
- learn docker networking basics.

lets make sure our docker is installed correctly

`docker container run` is the new way to run containers, with 'container' being the management command and 'run' being the sub command, but the old way `docker run' still works. existing commands will continue using both formats, but new commands will probably only be available in the new format.

### Starting a Nginx Web Server

<details>
<summary>
Start and stop a container from the command line, basic commands.
</summary>

- image vs container
- run/stop/remove containers
- check container logs and processes

> - An image is the application we want to run.
> - A container is an instance of that image running as a process.
> - You can have many containers running off the same image.
> - In this lecture, our image will be the Nginx web server.
> - Docker's default image "registry" is called [Docker Hub](https://hub.docker.com)

(registries are analogue to github for source code)

to start a new container we

```sh
docker container run --publish 80:80 nginx
```

and then we go to local host. we should see a Nginx welcoming screen.

the docker searched for an image called 'nginx' from docker hub, downloaded it, and started running it. we then route the traffic to the container via host port 80, to container port 80.

to run it in the background, we add the _--detach_ flag, it won't hold up the console. when we run it like this, we see the container unique id printed to the screen. we can then list all of our containers, and stop it, we can then see all the containers with the _-a_ flag to the ls command.

```sh
docker container run --publish 80:80 --detach nginx
docker container ls
docker container stop
docker container ls -a
```

containers also must have unique names, if we don't provide a unique name, one will be generated for us (from a list maintained in an open source, \<adjective>\_\<famous hacker or scientist>).

if we want to specify a name, we add _--name_ flag to the run command with the name afterward

```sh
docker container run --publish 80:80 --detach --name webHost nginx
docker container ls
```

if we want to see the logs of a container we can run the `logs` command, or the `top` command to see the processes running in it.

```sh
docker container logs webHost
docker container top webHost
```

to clear (remove) all dockers we use the `rm` command

```sh
docker container rm 63f 690 0de
```

we cannot remove an active (running) container this way. we can add the _-f_ flag to force it, or first stop it and then remove again.

docker is more than running containers. it looks for the image, first locally, and then at the remote image registry (repository),we can specify a version. the

> - Create a new container based on that image and prepares to start.
> - gives it an virtual IP on a private network inside docker engine.
> - Open a port on host and forward to a port in the container.
> - Starts container by using the CMD in the image docker file.

the container isn't actually a copy of the image, it's a new 'layer' on top of it (??). our CMD was Nginx.

#### Container VS VM: It's Just a Process

<details>
<summary>
Containers are just process on the host, not Virtual machines.
</summary>

containers are usually compared to virtual machines, but this comparison is't apt and doesn't serve us.

containers are just process running on our machine, and like all process, they are limited to what recourses they can access, and exit when the process stops.

if we run a container, and then the 'top' command shows us the process ids. these are the same process ids we will see if we look at what process the host machine is running (windows/mac not withstanding, as docker is running inside a VM itself).

</details>

#### Assignment #1: Manage Multiple Containers

<details>
<summary>
starting multiple containers and images at the same time.
</summary>

[docs.docker](<docs.docker.com](https://docs.docker.com)>) for help.

1. run nginx,mysql, httpd (apache) server
2. run them all as detached (--detach, -d) and names (--name)
3. ports should be 80:80, 8080:80, 3306:3306 (host port: container prt)
4. when running mysql, use the environment flag (--env, -e) to pass in "MYSQL_RANDOM_ROOT_PASSWORD=YES" as an environment variable.
5. use docker container logs to find the root password
6. stop and remove all of the containers
7. verify they are gone

```sh
docker container run --publish 80:80 --detach --name nginx_beny nginx
docker container run --publish 3306:3306 --detach --name mysql_beny --env "MYSQL_RANDOM_ROOT_PASSWORD=YES" mysql
# no grep, i don't know how to get this clean!
docker container logs mysql_beny |  Select-String -Pattern "GENERATED ROOT PASSWORD:"
docker container run --publish 8080:80 --detach --name apache_beny httpd
docker container ls
docker container stop nginx_beny mysql_beny apache_beny
docker container ls -a
docker container rm nginx_beny mysql_beny apache_beny
```

</details>
</details>

### What's Going On In Containers: CLI Process Monitoring

<details>
<summary>
running docker commands to understand what going on in the container
</summary>

we can see whats happening inside a running containers with the following commands,
_top_, _inspect_, _stats_.

lets start running then again. for mysql we can either specify the password in the command line,use no password or create a random password.

```sh
docker container run --detach --name nginx_beny nginx
docker container run --detach --name mysql_beny --env "MYSQL_RANDOM_ROOT_PASSWORD=YES" mysql
docker container top mysql_beny
docker container inspect mysql_beny
docker container stats
```

</details>

### Getting a Shell Inside Containers: No Need for SSH

<details>
<summary>
messing with a container from the inside without ssh.
</summary>

running command from inside the container process, _run -it_, _exec -it_.

differences between linux distro.

the -it is -i and --tty together

after the image we can add another command

```sh
docker container run -it --name proxy nginx bash
#exit from shell
exit
```

now we have a prompt from inside the container. if we exit, it actually exits the container, as if we would have stopped it the with `docker container stop proxy`

we can use the ubuntu image and install an image of a OS. the image is a minimal version, without any packages or stuff.

```sh
docker container run -it --name ubuntu ubuntu
apt-get install -y curl
exit
#start the stopped container
docker container start -ai ubuntu
```

to see the shell of a running container we use the exec command, exiting it wouldn't effect the running process.

```sh
docker container exec -it mysql bash
```

Alpine linux is a small, security-focused linux distro. it comes with it's own package manager. it doesn't even have bash in the image. it does have sh and we can install bash though apk.

```sh
docker image pull alpine
docker container run -it alpine sh
```

</details>

### Docker Networks

<details>
<summary>
How containers communicate with networks and with one another
</summary>

#### Concepts for Private and Public Comms in Containers

<details>
<summary>
Basic review of networks concepts regarding Docker.
</summary>

conceptual stuff, mostly

_-p,--publish_ exposes the port on the local machine to the container. most of the time, networks in dockers simply work, but there are stuff to learn.

##### Docker Networks Defaults

Each container is connected by default to a private virtual network "bridge", each virtual network routes through NAT firewall on the host IP. All the containers on the same virtual work can talk to each other without routing through the host (with _-p_). Best practices is to set different virtual networks for different apps

> network "my_web_app" for mysql and php/apache containers
> network "my_api" for mongo and node.js containers.

all of those setting are changeable, this ties in to the idiom of of Docker "Batteries Included, But Removable". things will work out of the box without us applying changes, but we can swap out parts to customize it to fit our needs.

we can make new virtual networks,attach one container to two or more virtual networks (or zero), or skip the virtual network and use the host IP directly
_--net=host_ flag. there is a plugin eco system of docker networks drivers that can give us new abilities.

```sh
docker container run -p 80:80 --name webHost -d nginx
docker container port webHost
docker container inspect --format "{{ .NetworkSettings.IPAddress }}" webHost
```

the ip address isn't the same as what we see in the _ipconfig_.

bridge/docker0 virtual network, is attached to the ethernet interface on one side, and containers on the other side.

only one container can listen on port on the host machine.

```sh
docker container run -p 80:80 --name webHost1 -d nginx #okay
docker container run -p 8080:80 --name webHost2 -d nginx #okay
docker container run -p 80:8080 --name webHost3 -d nginx #error!
```

</details>

#### CLI Management of Virtual Networks

<details>
<summary>
basic commands for networks.
</summary>

note: currently, use nginx:alpine image to get the `ping` command.

lets look at some network commands, _ls_, _inspect_, _create_, _connect_, _disconnect_.

```sh
docker network ls
```

the network bridge is the default network that bridges through the NAT firewall and the containers.

```sh
docker network inspect bridge
docker network inspect bridge --format "{{.Containers}}"
```

the default network IP for docker is 172.17.#.#. we can look at other networks and see the host network is directly connected outside.
the _--network none_ option removes the eth0 and leaves us only with local host interface on the container

```sh
docker network create my_app_net
docker network ls
docker network create --help
```

the default driver is 'bridge'. we look at the create command and see other options.

```sh
docker container run -d --name new_nginx --network my_app_net nginx
docker network inspect my_app_net
```

we can also attach networks to containers after they have been created. a container can be on more than one network.

```sh
docker network connect my_app_net webHost
docker network disconnect my_app_net webHost
```

> - Create your apps so frontend/backend sit on the same Docker network.
> - Their inter-communication never leaves host.
> - All externally exposed ports closed by default.
> - Must be manually exposed via _-p_,_--publish_, so default security is higher.

</details>

#### DNS

<details>
<summary>
Domain name service lookup in containers.
</summary>

DNS - Domain name service

how dns effect containers in custom and default networks,
we can't rely on IP address inside containers, because everything is so dynamic.
the _--link_ flag to enable DNS on default bridge networks.

containers change ips constantly, they stop and start again, the static IP just doesn't cut it anymore. the DNS is a built-in solution for this. in non default networks, the dns abilities are default enabled.

```sh
docker container run -d --name my_nginx --network my_app_net nginx:alpine
docker container exec -it my_nginx ping new_nginx
```

this is very important when running many containers, it doesn't matter the order of creation, as long as they belong to the same network, they can talk to one another.

the _--link_ flag allows us to link containers on the default bridge network, but we need to pass a list of other containers to work against.

</details>

#### Assignment #2 Using Containers for CLI Testing

<details>
<summary>
running two containers to get different images
</summary>

> - Use different linux distro containers to check _curl_ cli tool version.
> - Use two different terminal windows to start bash in both _centos:7_ and _ubuntu:14.04_, using _-it_
> - Learn about `docker container run --rm` option to save cleanup.
> - Ensure _curl_ is installed and on the latest version for that distro
>   - ubuntu: `apt-get update && apt-get install curl`
>   - centos: `yum update curl`
> - Check that `curl --version` displays properly.

```sh
docker image pull centos:7
docker container run -it --name cnt7 --rm centos:7
yum update curl
curl --version
docker image pull ubuntu:14.04
docker container run -it --name ubt --rm ubuntu:14.04 bash
apt-get update
apt-get install -y curl
curl -V
```

we can see that the versions of curl are different between the two images. nothing to clean up

</details>
 
#### Assignment #3 DNS Round Robin Test

<details>
<summary>
Using DNS aliases.
</summary>

[Round Robin DNS](https://en.wikipedia.org/wiki/Round-robin_DNS)

> Note: In the next assignment, you'll be using the tool nslookup inside the alpine:latest image, but in early 2020 there was a bug introduced to the latest Alpine image 3.11.3 that affects how nslookup works on hostnames, so for the next Assignment on DNS Round Robin, either change your command to work around it with nslookup search. (with a dot added) or use an older Alpine image like alpine:3.10.

two different hosts with dns aliases that respond to to same name, multiple ip address behind the same DNS name. we use this instead of the unique container name.elasticsearch gives out random names for itself.

> Create a new virtual network (default bridge driver) called 'search'
> Create two containers from image _elasticsearch:2_
> Research and use `docker container run --network-alias search` to give them an additional dns name to respond to.
> Run `alpine nslookup search` with _--net_ to see the two containers list for the same DNS name. from the alpine image
> Run `centos curl -s search:9200` with _--net_ multiple times until both "names" of the elasticsearch containers show.

```sh
docker image pull elasticsearch:2
docker network create round_robin
docker container run -d --rm --network round_robin --network-alias search --name els1 elasticsearch:2
docker container run -d --rm --network round_robin -network-alias search --name els2 elasticsearch:2
docker network inspect round_robin
docker container run -iy --rm --network round_robin --name alp alpine

docker container run -d --rm --network round_robin --name cnt centos
docker container exec -it cnt bash
curl -s search:9200
```

should have used

```
docker container run --rm --network round_robin alpine nslookup search.
docker container run --rm --network round_robin  centos curl -s search:9200

```

</details>

</details>

</details>
