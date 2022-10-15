<!--
ignore these words in spell check for this file
// cSpell:ignore Entrypoints gosu
-->

[Main](README.md)

## DevOps and Docker Clips

<!-- <details> -->
<summary>
//TODO: add Summary
</summary>

answering Questions in lecture form. also available on youtube and in podcast form.

### Alpine Base Images. Are They Really More Secure?

Container security scanning, scanning images for known vulnerabilities or security breaches.

there is an open database for known vulnerabilities, we can scan the image and the dependencies used to find problems and then fix the image.

there is an article ([part 1](https://kubedex.com/container-scanning/), [part 2](https://kubedex.com/follow-up-container-scanning-comparison/)), it argues that the common issue is the bast operating system which the image uses. and because the packages are in the OS, the scanning process needs to find those packages in the base image. so if there isn't a proper translation path, it won't be scanned properly.

Alpine is a great distribution with a small base image, it has risen in popularity over the past years. people argue that because it's minimal, then there less things which could go wrong. however, disk space isn't much of a priority in cloud native worlds, it's not very expensive, so optimizing for size isn't much of a gain. and if the app works great with some other OS, then maybe there's no need to switch over.

they argue that scanning alpine isn't great, and there are some issues with alpine and applications such as _nodeMon_. using non-alpine images allow us to use the default base images of `docker container run node`, which usually runs on debian. sometimes the alpine package manager doesn't have all the packages available to it, unlike `apt-get`, which usually has everything.

the other images (ubuntu, debian) are getting smaller, some stuff is getting removed, such as ping, and it might be required to install those packages as part of the image, so if they're removed in the future from the base image, the application won't crash.

### Dealing With Non-root Users In Containers and File Permissions

when we combine bind mounts and container permissions, we can have problems. we can create Users in the container image build. there is the option of `gosu`, which is similar to `sudo`.

the important thing is the id number of users and groups, not the user or group name.

### Apache Web Server Design. Many Sites In One Container, or Many Containers?

apache servers can host multiple website on one daemon (httpd instant). so what is the preferred option?

probably more containers, depending on the number of websites. for a small amount of websites, the gains from using one daemon aren't that relevant. also, if we need to update one website, it's easier not to take down every container.

### Docker Network IP Subnet Conflicts with Outside Networks

the default network drivers (bridge) is isolated from the other networks, it should have it's own subnet mappings, and they shouldn't conflict with them.

```sh
docker network ls
```

there can be problems with overlapping subnets. we should try pinging to see if the problem exists.

for cloud subnets and VPNs, there can be clashes.

### Raspberry Pi Development in Docker

tips on prototyping on raspberry PI, there are problems with architecture (ARM64 vs X86). we use "QEMU" as an emulation environment, so we can test our software through that. it's easier than trying to make the raspberry PI the development environment.

### Windows 10 Containers Get Process Isolation

an enhancement to how docker desktop runs on windows 10 pro. Window containers, a smaller server kernel.\
Process isolation allows running native binaries without spinning up an expensive hyper-v container.

### Should You Move Postgres to Containers

is there a good reason to move a database to a container (if it's currently running bare metal)? it might be easier to use a cloud serverless option such as mariaDB. the performance should be similar in both cases.\
There is white paper with a comparison of MySQL performance in VM, containers and bare metal.

### Using Supervisor To Run Multiple Apps In A Container

Supervisor-d as a root process of the container. running two applications on the same container. like a kubernetes pod. useful for when using docker swarm.

### Should You Use Docker Compose or Swarm For A Single Server?

probably better swarm - allows for rolling updates. docker compose wasn't meant for production.

### Docker Environment Configs, Variables, and Entrypoints

Twelve factor app - making a distributed app more successful. best practices for distributed applications.\
The **config** stage. we don't run to have different images for production, development, etc... we want the images to be the same, and have all of those configurations separated from the code itself.

we can use environment variables to hold of those values, they are platform agnostic, and are available for all languages. we have the value default in the dockerfile, and we can override them in all sorts of ways, such as docker compose, swarm, command line.\
Another option is to have the entrypoint write those values into a file inside the container. we can see an example of this in the MySQL dockerfile.

if possible, it's better to not have passwords and secrets as environment variables, there are better options such as vaults, docker secrets, and so on.

### Java and JBoss in Containers. One .war File Per Container?

the long term goal is probably to split them, so it's possible to have rolling updates, better isolation, etc...

### TLS in Dev and Prod with Docker

certifications, trusted certificates, passing them from the docker-compose.

### Multiple Docker Images From One Git Repo

we can have as many dockerfiles in the repo as we want, and we can pass the filename with `-f` to overwrite. just remember the context is where we run the `docker image build` command. whatever path we want to include, it must be part of the context.

### Docker + ARM, Using Raspberry Pi or AWS A1 Instances with Docker

### Docker and Swarm RBAC Options

### ENTRYPOINT vs. CMD, what's the difference in Dockerfiles

### How to Use External Storage in Docker

### Can I Turn a VM into a Container?

### Startup Order With Multi-Container Apps

</details>

[Main](README.md)
