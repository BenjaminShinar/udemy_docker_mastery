<!--
ignore these words in spell check for this file
// cSpell:ignore Entrypoints
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

### Apache Web Server Design. Many Sites In One Container, or Many Containers?

### Docker Network IP Subnet Conflicts with Outside Networks

### Raspberry Pi Development in Docker

### Windows 10 Containers Get Process Isolation

### Should You Move Postgres to Containers

### Using Supervisor To Run Multiple Apps In A Container

### Should You Use Docker Compose or Swarm For A Single Server?

### Docker Environment Configs, Variables, and Entrypoints

### Java and JBoss in Containers. One .war File Per Container?

### TLS in Dev and Prod with Docker

### Multiple Docker Images From One Git Repo

### Docker + ARM, Using Raspberry Pi or AWS A1 Instances with Docker

### Docker and Swarm RBAC Options

### ENTRYPOINT vs. CMD, what's the difference in Dockerfiles

### How to Use External Storage in Docker

### Can I Turn a VM into a Container?

### Startup Order With Multi-Container Apps

</details>

[Main](README.md)
