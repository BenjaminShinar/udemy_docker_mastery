<!--
ignore these words in spell check for this file
// cSpell:ignore Jeykll  bretfisher
-->

## Making it Easier With Docker Compose: The Multi-Container Tool

<details>
<summary>
getting stuff done faster with docker-compose
</summary>

Docker Compose is a combination of a command line tool and a configuration file.

### Docker Compose and The docker-compose.yml File

<details>
<summary>
the docker-compose yml file
</summary>

a way to configure relationships between containers. save the run settings in an 'easy-to-read' file, which will allow us to start the developer environment with one line.

there are two parts:

1. the yaml file which configures containers, networks and volume (images, environment variable).
2. The Cli tool **docker-compose** that is used to develop and test the yaml files

docker-compose.yml started as .fig, and now has different versions. it can also work directly in the docker command line and swarm.

the default file name is `docker-compose.yml', but we can use any file name and pass it with the _-f_ flag.

the yml has 'version' at the top, it defaults to 1.0, but we should at least work with 2.0 or above, depending on what we want to use.

there are also "services", "volumes" and "networks" that we can configure. the yml is hierarchical in the format (indentations). each container is a 'service', we give it a name, the image, the command (override what it's in the image), the environment variable, the volumes, any anything that goes into the docker container run command.

this is an example.
it starts with the version number (2), then declares which services to run (containers), in this case, one container name _jekyll_, the image, the ports and the volume for the bind mount.

```yml
version: "2"

# same as
# docker run -p 80:4000 -v $(pwd):/site bretfisher/jekyll-serve

services:
  jekyll:
    image: bretfisher/jekyll-serve
    volumes:
      - .:/site
    ports:
      - "80:4000"
```

we can note that if there can be a list of items, it will be a list format (and usually plural named), if only one value is expected, then a key-value pair is used with a singular name.

for environment variables, we can write key-value pairs without a list form (or with).

the _depends_on_ is a way to define relationships between services.

</details>

### Basic Docker Compose Commands

<details>
<summary>
The docker-compose cli-tool.
</summary>

the cli tool comes bundled with the windows docker cli.
this isn't a production tool, instead, it's used to run developments and test.

the most common commands are 'up' to start and 'down' for cleanup.

- _docker-compose up_ - setup volumes/networks and start all containers.
- _docker-compose down_ - stop and remove all containers, also remove networks and volumes.

if, theoretically, all of our projects had DockerFiles and docker-compose.yml, then onboarding a new developer would be as easy as cloning a repo from git and running `docker-compose up`.

lets look at compose-sample-2 folder, it has yml file with two containers, nginx called proxy using a nginx config file. (which is a read only on the container side), it also has an httpd (apache) image.

we run it with

```sh
docker-compose up
```

we can see the localhost:80 in the browser, which is from the apache (gt), we see different colors for each image.

we can use most of the commands from the docker command line, and they usually work.

#### Assignment #1: Build a Compose File For a Multi-Container Service

<details>
<summary>
Creating a compose file from scratch.
</summary>

> - Build a basic compose file for a Drupal content management system website.
> - use the **drupal** and **postgres** images
> - use the ports in the compose-file to expose 8080 on local host
> - postgres will require a password.
> - **drupal** assumes DB is local host, but it's a service name, so we need to change this.
> - extra credit: use volumes to store drupal unique data

[docker hub Drupal](https://hub.docker.com/_/drupal)

default postgres db is 'postgres', so is default user 'postgres', we need to change the host in 'advanced settings'.

</details>

</details>

### Adding Image Building to Compose Files

<details>
<summary>
image building as part of the compose file
</summary>

docker compose can also build images, when we have images we want to use.

using 'compose-sample-3' folder.

we have the 'build' section with 'context' and 'dockerfile' sub sections, we can build the image directly if it doesn't exist in cache.

in the example we have another nginx with reverse proxy.

once we run it we can start changing local data because it's mounted. the image won't be cleaned by default. we can add the _--rmi \<all | local>_ flag to remove images.

the directory name is added to the names of containers/volumes/etc to avoid name pollution.

#### Assignment #2: Compose For Run-Time Image Building and Multi-Container Development

<details>
<summary>
both compose and dockerfile working together.
</summary>

> - Building custom **drupal** image for local testing.
> - not just for developers. also for users who want to try something
> - continuing with our example from before (compose-assignment-2), we will have our own Dockerfile to create images
> - see README.md for details

</details>

</details>

</details>
