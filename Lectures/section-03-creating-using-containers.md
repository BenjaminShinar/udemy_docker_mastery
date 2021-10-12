<!--
ignore these words in spell check for this file
// cSpell:ignore distro
-->

## Creating and Using Containers Like a Boss

<!-- <details> -->
<summary>
//TODO: add Summary
</summary>

Containers are the fundamental building blocks of docker.

- we will get our container working
- create our first web server (container) with Nginx
- learn common container management commands
- learn docker networking basics.

lets make sure our docker is installed correctly

the management commands are a new way to sub section commands.
commands (power shell):

- _docker version_ - return versions of the clients and server dockers.
- _docker info_ - more information
- _docker_ - get list of commands, including management commands.
- _docker \<command> \<sub-command> (options)_ - new way, with management commands.
- _docker \<command> (options)_ - old way.
- _docker \<command> --help_ - information on command.
- _docker container run_ - start a container, with an image
  - _--detach_, _-d_ - run in background
  - _--publish_, _-p_ - port forwarding (host:container),
  - _--name_ - specify name of the container
  - _--env_ - pass environments variables
  - _-it_ - start new container interactively.
- _docker container ls_ - list all the containers (new format). by default only shows running containers,
  - _-a_ - all, show also stopped containers.
- _docker ps_ - list all containers (old format)
- _docker container stop <#id>_ - stop a container.
- _docker stop <#id>_ - stop a container.
- _docker container logs \<name>_ - see logs for container.
- _docker logs \<name>_ - see logs for container (old format).
- _docker container top \<name>_ - see the processes running inside the container.
- _docker container rm <#id1> <#id2>_ - remove containers, we can specify more than one container.
  - _-f_ - flag to remove running containers.
- _docker start \<name>_ - run a stopped container. (old format)
- _docker image ls_ - see list of images.
- _docker container inspect <#id>_- details of one container config meta data (JSON).
- _docker container stats_ - details of one or all of our container config.
- _docker container exec -it_ - run an additional command in existing container
- _docker image pull \<image name>_ - download image from hub.

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

## Getting a Shell Inside Containers: No Need for SSH

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

## aa

</details>
