<!--
ignore these words in spell check for this file
// cSpell:ignore tini mkdir
-->

## Container Images, Where To Find Them and How To Build Them

<details>
<summary>
What are images, where to find them and manage them, how to build them.
</summary>

### Images

<details>
<summary>
Images, Tags and versions, layers, DockerHub registry.
</summary>

An Image is:

> - App binaries and dependencies.
> - Metadata about the image data and how to run the image.
> - "An Image is an ordered collection of root filesystem changes and the corresponding execution parameters for use within a a container runtime."
> - The image isn't a complete OS, no kernel, no kernel modules (e.g. drivers)
> - Can be as small as one file (like golang static library) or as big as several GB, like ubuntu or mysql packages.

#### The Mighty Hub: Using Docker Hub Registry Images

<details>
<summary>
Introducing DockerHub, the default image repository.
</summary>

basics of dockerHub, where to find images, what types of them there are.

lets look at docker hun and search for _"nginx"_, we should always start with the 'official' version of the image. there can be only one 'official' image, images have versions, the mainline, the stable. images are tagged (not named)

downloading images. we can specify the tag we want to download, in development, we can use the latest or the stable version, but for production, we should specify the
exact version, so our users get what we want them to get, and not use an updated version which might break our app.

```sh
docker image ls
docker image pull nginx
docker image pull nginx:1.11.9 # a specific version
docker image pull nginx:latest # a specific version, tag latest
docker image pull nginx:stable-alpine # a specific version, tagged
docker image ls # same image ID, not actual copies

```

alpine is a distribution of linux that's very very small, so the alpine versions come from that and tend to be much smaller. just like github, we can create our own images based on those of others. we can see the number of pulls and stars. we can also inspect the source code itself.

pressing 'explore' will show us the most common images

</details>

#### Images and Their Layers: Discover the Image Cache

<details>
<summary>
How are images layered. layers are changeset.
</summary>

Image layers, how docker works, the union file system, changes from the file system. the _history_, _inspect_ commands. copy on write concept.

```sh
docker image ls
docker image history nginx
```

every image stars with a blank history 'scratch', every change is a new layer. changes can be copying files, adding meta data, etc...
the first layer is the OS, the second layer can be getting an package, and the third can be setting a environment variable, it can also be creating a file.

if we have more than one image using the same basic layer, then we only need to store the changes from that layer, we don't need to store the entire chain from zero. this is the concept of **Image cache**. each layer has a unique SHA. each unique layer is stored only once. we don't need to get the same basic layers for each image.

with containers, we create a new read/write layer on top of the image. the image is read only when facing the container. they cannot change it. if they try to change it, the docker performs _copy on write_, it duplicates the file and stores the duplication on the container.

```sh
docker image inspect nginx
```

</details>

#### Image Tagging and Pushing to Docker Hub

<details>
<summary>
Image identification via tags.
</summary>

images don't have names actually. we usually refer to them by "\<user>/\<repo>:\<tag>". official images don't have the 'user' data.

```sh
docker image ls
```

the tag isn't a build version, it's like a github tag, it's like a marker of a node, it can be anything, they are labels for image id.

```sh
docker image tag nginx usr/nginx #create a new image based on that image
docker login #login to hub
cat ./docker/config.json #see login
docker image push usr/nginx # push to registry
```

if the images already exist, they won't be pushed or pulled. this is how we save space.

</details>

</details>

### Building Images

<details>
<summary>
Actually Building images.
</summary>

#### The Docker File Basics

<details>
<summary>
Docker file basic structure
</summary>

looking at the dockerfile-sample-1 folder at seeing how it's made.
it's not a shell script, the default name is 'Dockerfile', but when we use the command line, we can specify a file name with _-f_ flag.\
`docker build -f some-dockerfile-arbitrary-name`

each part is called **stanza**, a term originally referring to block of lines in a poem.

the first stanza\part is the `FROM` commands, it normally is from a minimal distribution, such as debian or alpine.

the next stanza\part is `ENV`, setting environment variables. they can be used anywhere.

each stanza is actually a layer, the order matters. a lot of times we have multiple commands chained together. Chaining the commands together ensures they all go in the same stanza.

docker handles all the logging for us, our app should direct the logging to the stdout and stderr. we shouldn't use logging files.

`EXPOSE` opens up ports from the container towards the virtual network. we still need to run the container with the _--publish_ flag to forward ot these port on the host.

The `CMD` stanza is required. it's the command that will run when the container is launched. only one is allowed

```dockerfile
#comment
FROM debian:jessie
ENV NGINX_VERSION 1.13.6-1~stretch

RUN apt-get update \
	&& apt-get install --no-install-recommends --no-install-suggests -y gnupg1 \
    ## more installations

#LOGGING
RUN ln -sf /dev/stdout /var/log/nginx/access.log \
	&& ln -sf /dev/stderr /var/log/nginx/error.log
# forward request and error logs to docker log collector

# PORTS
EXPOSE 80 443

# Command that runs
CMD ["nginx", "-g", "daemon off;"]
```

</details>

#### Running Docker Builds

<details>
<summary>
running image builds, caching and order of layers.
</summary>
now that we have the file, running the build command will run the commands inside the docker  runtime and cache the results into the image.

```sh
docker image build -t custom-nginx-beny .
```

if the dockerfile doesn't change, then the build is cached and it won't take long to build.

we can look at our images to see that our new image exists as latest. we can add an additional port and try building again this time it's really quick.

if we get things out of order, the building time changes. we want to keep the top of the file with the parts that don't change, and the bottom of the file with things that do.

</details>

#### Extending Official Images

<details>
<summary>
Adding layers to official images.
</summary>

in the dockerfile-sample-2 folder we see two files, one dockerfile and one html index file.

in this file, we use an image as the source for the `FROM` stanza.

we have a `WORKDIR` stanza, which is easier to describe than `run cd some/path`, even if they do the same.

`COPY` copies source code from the build source into container.

```dockerfile
COPY index.html index.html
```

we don't need to add `CMD` because it's already inside the `from` stanza.

lets first look at the default behavior of nginx. we run it and then look at the browser in localhost and see the default html page.\
next we build and run the custom nginx. we look at the local host and the html is different!

```sh
docker container run -d -p 80:80 --rm nginx

docker image build -t nginx-custom-index .
docker container run -d -p 80:80 --rm  nginx-custom-index
```

</details>

#### Assignment: Build Your Own Dockerfile and Run Containers From It

<details>
<summary>
building a docker file for a node.js app.
</summary>

file and details in docker-file-assignment-1/dockerfile.

> Instructions from the app developer
>
> - you should use the 'node' official image, with the alpine 6.x branch ( node:6-alpine)
>   - yes this is a 2-year old image of node, but all official images are always
>     available on Docker Hub forever, to ensure even old apps still work. It is common to still need to deploy old app versions, even years later.
> - this app listens on port 3000, but the container should launch on port 80 so it will respond to http://localhost:80 on your computer
> - then it should use alpine package manager to install tini: 'apk add --update tini'
> - then it should create directory /usr/src/app for app files with 'mkdir -p /usr/src/app'
> - Node uses a "package manager", so it needs to copy in package.json file
> - then it needs to run 'npm install' to install dependencies from that file
> - to keep it clean and small, run 'npm cache clean --force' after above
> - then it needs to copy in all files from current directory
> - then it needs to start container with command 'tini -- node ./bin/www'
> - in the end you should be using `FROM`, `RUN`, `WORKDIR`, `COPY`, `EXPOSE`, and `CMD` commands

[Node.js docker hub](https://hub.docker.com/_/node)

- [x] building minimal - able to build image and run container
- [x] create directory
- [x] install packages
- [x] clear cache
- [x] copy all from current directory
- [x] expose ports
- [x] install dependencies
- [x] start with node command

commands

```sh
docker image build -t app-one .
docker image ls
docker container run -d -p 80:3000 --rm app-one
```

the rest is pushing to dockerHub.

</details>

#### Using Prune to Keep Your Docker System Clean (YouTube)

<details>
<summary>
Cleanup
</summary>

[youtube](https://youtu.be/_4QzP7uwtvI)

builtin features, to see space usage and clean them

```sh
docker image prune
docker system prune
docker system df
```

</details>
</details>
</details>
