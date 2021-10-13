<!--
ignore these words in spell check for this file
// cSpell:ignore
-->

## Container Images, Where To Find Them and How To Build Them

<!-- <details> -->
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

<!-- <details> -->
<summary>
//TODO: add Summary
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

<!-- <details> -->
<summary>
//TODO: add Summary
</summary>

</details>

##

</details>
