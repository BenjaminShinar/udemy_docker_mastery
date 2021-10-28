<!--
ignore these words in spell check for this file
// cSpell:ignore 
-->

## Docker in Production

<details>
<summary>
Talks and youtube videos about Docker
</summary>

### Docker and Swarm in Production

<details>
<summary>
A talk from docker con 2017.
</summary>


[resources](https://www.bretfisher.com/dockercon17eu)

getting docker into production.

analogies of video games and docker stuff.

#### limit you simulatous innovation - avoid Project/feature/scope creep.

> - many initial containers projects are too big in scope.
> - some stuff that we don't need at day one:
> 	- Fully automaitc CI/CD.
> 	- Dynamic performance scaling.
> 	- Containrerizing all or nothing.
> 	- Startomg with persistent data

> Legacy apps work in containers too.
> - Microservice conversion isn't required.
> - __12 Factor__ is a horizion we're always chasing.
> - Don't let these ideals delay containerization.

#### Docker file power ups: 

First thing to focus on: docker files.

> - docker files are more important than fancy orchestration
> - it's the new build and enviornment documentation
> - study Dockerfiles  and `ENTRYPOINT` of hub officials
> - `FROM` official distors that are most familiar

> The docker maturaity model (steps)
> 1. Make it start.
> 2. Make it log all the things to stdout/stdrr.
> 3. Make it documented in file.
> 4. Make it work for others.
> 5. Make it lean.
> 6. Make it scale.

we don't use log files, that's an anti pattern. the infrastuture does the logging.
if stuff is changed in the app to work with containers, document it.
the image size isn't the issue, it's stored once, we should focus on getting stuff right, and then verify that it's scalble.

#### Avoid Dockerfile Anti-Patterns

Trapping data - not being able to get the unique data out of the container.
> Problem: Storing unique data in container.
> Solution: Define `VOLUME` for each location.

```dockerfile
VOLUME /var/lib/mysql
ENTRYPOINT ["docker-entrypoint.sh"]
CMD ["mysqld"]
```

Using Latest - image building can break when new versions are released.
> Problem: Image builds pull `FROM` latest
> Solution: use specific `FROM` tags.
> Problem: Image builds install latest packages.
> Solution: specify versions for critical apt/yum/apk packages.

here, we define a base image to build on, and define which versions we use as part of the enviornment variables.
```dockerfile
FROM php:7.0.24-fpm

ENV NGINX_VERSION 1.12.1-1~jessie \ NJS_VERSION 1.12.1.0.1.10-1~jessie \ COMPOSER-VERSION=1.5.2 \ NODE_VERSION 6.11.4
```

here we define exactly which packge versions we want to install.

```dockerfile
FROM ubuntu:xenial-20170915

RUN apt-get update && apt-get install ca-certificates \ g++ \ldap-utils=2.4.40+dfsg-1+deb8u3 \ libedit-dev=3.14-20140620-2 \
```

we don't deploy random versions of our code, so why should we deploy random versions of dependencies?

Leaving default configs
> Problem: Not changing app defaults, or blindly copying VM configurations. eg php.in, mysql.conf.d, java memmory configurations.
> Solution: update default configts via `ENV`, `RUN`, and `ENTRYPOINT`.


Enviornment specific
> Problem: Copy in enviornment config at image build
> Solution: Single dockerfile with default `ENV`, and overwrite per enviornment with `ENTRYPOINT` script.

this is a bad way to do this.

```dockerfile
FROM  node:6.10

COPY test-enviornment.json test-enviornment.json
#COPY dev-enviornment.json dev-enviornment.json
#COPY prod-enviornment.json prod-enviornment.json
```

### 3 Big decisons about infrastuture

Containers-on-VM or Containers-on-Bare-Metal

either? both?
stick with what you know first, and do some basic performance testing. white paper on MYSQL benchmarks in vm, containers-on-vm and containers on-bare-metal.

OS linux Distribution / Kernel - we should use something current (4.x in the time of the talk - 2017)
Docker is kernel and stroage driver depenedent
Innovations are still happening.
The minimum version isn't the 'best' version.

we can use ubuntu lts or **InfraKit** and **linuxKit**. we should get the correct docker for the version we use, what we get from apt-get isn't necessarly what we need

Which Container base distribution

> - Which `FROM` image should we use?
> - don't make a decision based on image size (it's a single instance stroage)
> - At first, match your existing deployment process
> - later, even much later, consider switching to alpine.

we should have a team standard image that everything builds on.


#### Build Your Swarm

good default: swarm arhitecturs

> Simple sizing guidelines based off:
> - Docker internal testing.
> - Docker reference arhitecturs.
> - Real world deployments.
> - Swarm3k lessons learned.


baby swarm - 1 node swarm, like a demo, it's  ok to use swarm for 1 node. it's just one line to initilaze a swarm, and it gives us more featues than docker container run.

HA swarm- 3-node (highly avaialbe)
> - Minimum for HA.
> - All managers.
> - One node can fail.
> - Use when very small budget.
> - Pet projects or Test/CI.

Biz Swarm - 5 nodes
> - Better high-avaialblity.
> - All managers.
> - Two nodes can fail
> - minimum for uptime that affects revenue.

Flexy Swarm - 10+ nodes
> - 5 dedicated managers
> - Workers in DMZ(???)
> - Anything beyond 5 nodes, stick with 5 managers and the rest are workers.
> - Control container placement with labels + constraints.

Swole Swarm - 100+ nodes
> - 5 dedicated managers, resize managers as you grow
> - multiple workers subents on private/DMZ
> - Control container placement with labels + constraints.


name | number of nodes | HA | usage?
------|------------|------------|----------|-----------------
baby swarm | 1 | no | //todo
HA swarm | 3 (all managers) | yes | test, not critical stuff
Biz Swarm | 5 (all managers) | yes | real stuff
Flexy Swarm, |10+ (5 managers top) |yes|
Swole Swarm, |100+ (5 managers and resizing as needed) |yes|

> "Dont turn cattle into pets"
> - Assume nodes will be replaced.
> - Assume containers will be recreated.
> - Docker for (AWS/Azure) does this.
> - LinuxKit and InfraKit expect it

don't do special stuff on the host.don't store special stuff on the nodes, 

Reasons for multiple swarms:

Bad reasons:
> - differnt hards configurations.
> - differnt subnets or secuirty groups.
> - Different availabilty zones.
> - Secuirty boundaries for compliance.

Good reasons:
> - Learning: Run stuff on test swarm.
> - Geographical bounderaies.
> - Management boundaries using Docker API (or docker EE RBAC, or other auth plugins)

RBAC: Roll based Access Control.

Windows server 2016 Swarm. hybrid swarm of differnt OS, many stuff is still linux only, and the innovations are happening on linux stuff.

#### Bring in Reinforcements

Outsorece well defined plumbing.
> - Beware the "not implemented here" syndrome.
> -  if challenge to implent and maintain
> - + Saas / commercial market is mature
> - = Oppertunites for outsourcing

outsourcing candidates
> - Image registry.
> - Logs.
> - Monitoring and Alerting.
> - Tools/Projects [ecosystem](https://github.com/cncf/landscape)

Tech Stacks: Designs for a full featured cluster.
pure open source self hosted tech stack. (not kubernetes ready in time of the talk).

starting from lower layers.

1. HW/OS
2. Runtime
3. Orchestrarito
4. Networking
5. Storage
6. CI/CD
7. Registry
8. Layer 7 Proxy
9. Central Logging
10. Central montioring
11. Swarm GUI

OpenFass for *functions as a service*.

#### Must we have an Orchestraritor?

accelrating the docker migration, maybe we can use VM, one container per VM works, this isn't perfect, but it's a step that still user dockerfiles and teaches us a lot.
Windows does it with Hyper-V,  linux does it with Interl Clear Conatienrs, Windows has "LCOW" - linux container on windows.

#### summary

> - Trim the optioanl requirement at first.
> - focus on dockerfile/docker-compose.yml
> - watch out for dockerfile anti-patterns
> - Stick with familiar OS and `FROM` images.
> - Grow Swarm as you grow
> - Find ways to outsource plumbing
> - Realize parts of your tech stack may change, stay flexible

</details>

### The Future of Swarm
<details>
<summary>
  //todo
</summary>

> "In 2020, What's Up With Swarm's Latest Features? With all the media excitement about the never-ending new Kubernetes projects, Swarm news can get drowned out.  I've written two articles on Swarm in 2018, and updated in 2019, and now there's more news in 2020, so I made a YouTube Live about everything going on.
Basically, Mirantis is pledging public support in 2020 and beyond by growing the Swarm/SwarmKit team and telling us about planned new features."

</details>

### Swarm Raft Quorum and Recovery

<details>
<summary>
</summary>
//todo
</details>

</details>
