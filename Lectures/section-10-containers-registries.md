<!--
ignore these words in spell check for this file
// cSpell:ignore 
-->

## Container Registries: Image Storage and Distribution

<details>
<summary>
Registeries are image storage repositories
</summary>

Container images registeries, we need a place in the server to store the images.
can be public or private. docker hub auto build. doker store and docker cloud.
new swarm features in docker cloud. intalling docker Registery as private docker storage or other 3rd party software.


### Docker Hub: Digging Deeper

<details>
<summary>
Docker Hub repository options
</summary>

Advanced topics of DockerHub.

hub is a registry that has extra stuff, like image building.
we can link gitghuv to docker hub and auto build images on commit, and we can chain image building together.

let's look at other features of DockerHub.

repositories have premsissions, and we can have web-hooks to other services (jenkins, travisCI,etc...). we can also have organizations, which contain several users together.
if we are using github, we shouldn't press 'Create Repository', we should *Create* -> *Create Automated Build* instead to link our image to source code. this is like a **reverse web-hook**, we are notified of a github change and do something, as opposed to notifing an upstream service.

in automated build, we can see more information, such as build settings and history. we can also link ourselves to other repositories on docker hub if we are depending on other images (like ruby, nginx) so we will always be uptodate.
build triggers is another way to automate image build.


</details>

### Docker Registery

<details>
<summary>
Understanding Doker Registery and running a private Registery
</summary>

> - A private image registry.
> - Part of the docker/distribution GitHub repo.
> - The de facto (standard?) in priavte container registries
> - Not as full featured as Hub or others, No Web UI, basic auth only.
> - At it's core: a web API and storage system, written in Go.
> - Storage supports local, S3/Azure/Alibaba/Google Cloud, and OpenStack Swift

the current registry version was wrriten in go, and is called **distribution** on github, and **registry** on dockerhub. there are less features than other services, no web UI, only basic authentication. still not very scalble.

the local is the default version.

we should first look at the TLS to make things secure. we also need storage cleanup with automated garbage collection.

we can also enable hub caching via `--registry-mirror` flag, this is good when we have limited bandwidth and if we have many images.

the default port for the registery is port 5000.



the registery prefers proper TLS, it won't talk to unsecure registeries, except for the localhost(127.0.0.0/8), if we want to use remote self-signed TLS, we should enable "insecure-registery" in engine

we will re-tag an existing image to push it to a new registery, then remove it from the local cahce and pull it from the new registery. and afterwards we will re-create the registery with a bindmount to see how it stores data.


we will be working in the folder *register-sample-1*

```sh
docker container run --name registery -d -p 5000:5000  registery
docker container ls
docker image pull hello-world
docker container run --rm hello-world
docker image tag hello-world 127.0.0.1:5000/hello-world
docker image ls
docker image push 127.0.0.1:5000/hello-world
docker image remove hello-world
docker image remove 127.0.0.1:5000/hello-world
docker image ls
docker image pull 127.0.0.1:5000/hello-world:latest
docker container kill registery
docker container rm registery
```

now the same, but with bound mounts and volumes

```sh
docker container run --name registery -d -p --rm 5000:5000 -v${pwd}/registery-data:/var/lib/registery registery
ls registery-data #show files, OS command
docker image ls 
docker push 127.0.0.1:5000/hello-world
ls registery-data #show files, OS command
tree registery-data #Linux command
```

the steps that we did
> 1. run the registery image.
> 2. re tag an existing image an push it to the new registery.
> 3. remove the image from the locak cache and pull it from the new registery.
> 4. re-create registery using a bind mount and see how it stores data. 

#### Assignment: Secure Docker Registry With TLS and Authentication

<details>
<summary>
Pulling an image from registery, creating self TLS authentication
</summary>

> The default registry install is rather bare bones, and is open by default, meaning anyone can push and pull images.  You'll likely want to at least add TLS to it so you can work with it easily via HTTPS, and then also add some basic authentication.  
>
> These aren't actually that hard to setup, but do require some commands.  You can learn the basics by creating a self-signed certificate for HTTPS, and then enabling *htpasswd* auth, which you'll add users too with basic cli commands.
>
> For this assignment you'll use Play With Docker, a great resource for web-based docker testing and also has a library of labs built by Docker Captains and others, and supported by Docker Inc. 
>
> I'd like you to do the Part 2 and 3 of "Docker Registry for Linux" for this assignment. You can use their text to do this assignment on your own machine, or jump back to their Part 1 and run the container on their infrastructure  using their web-based interface to a real docker engine and learn how "PWD" works!
>
> For more extra credit labs, look through their growing list: http://training.play-with-docker.com/

generating a self-signed certificate in linux

part2
```sh
mkdir -p certs #create if doesn't exist
openssl req -newkey rsa:4096 -nodes -sha256 -keyout certs/domain.key -x509 -days 365 -out certs/domain.crt
# fill form

mkdir /etc/docker/certs.d
mkdir /etc/docker/certs.d/127.0.0.1:5000 
cp $(pwd)/certs/domain.crt /etc/docker/certs.d/127.0.0.1:5000/ca.crt #copy with differnt name

diff certs/domain.crt /etc/docker/certs/127.0.0.1:5000/ca.crt #check that files are the same
ps dockerd
pkill dockerd
ps dockerd # again, vertif less process
dockerd > /dev/null 2>&1 & # run dockerd, The /dev/null part is to avoid the output logs from docker daemon.

mkdir registry-data
docker continaer run -d -p 5000:5000 --name registry \
  --restart unless-stopped \
  -v $(pwd)/registry-data:/var/lib/registry -v $(pwd)/certs:/certs \
  -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/domain.crt \
  -e REGISTRY_HTTP_TLS_KEY=/certs/domain.key \
  registry
  
docker container ls

docker image pull hello-world
docker image tag hello-world 127.0.0.1:5000/hello-world
docker image push 127.0.0.1:5000/hello-world
docker image pull 127.0.0.1:5000/hello-world
```

part 3:

[code doesn't work properly?](https://stackoverflow.com/questions/62531462/docker-local-registry-exec-htpasswd-executable-file-not-found-in-path)

```sh
mkdir auth
docker container run --entrypoint htpasswd --rm registry:2.6 -Bbn moby gordon > auth/htpasswd #Create the password file with an entry for user “moby” with password “gordon”;
cat auth/htpasswd

```

> The options are:
>
> - _–-entrypoint_ Overwrite the default ENTRYPOINT of the image
> - _-B_ Use bcrypt encryption (required)
> - _-b_ run in batch mode
> - _-n_ display results

Running an Authenticated Secure Registry
```sh
docker kill registry
docker rm registry
docker container run -d -p 5000:5000 --name registry \
  --restart unless-stopped \
  -v $(pwd)/registry-data:/var/lib/registry \
  -v $(pwd)/certs:/certs \
  -v $(pwd)/auth:/auth \
  -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/domain.crt \
  -e REGISTRY_HTTP_TLS_KEY=/certs/domain.key \
  -e REGISTRY_AUTH=htpasswd \
  -e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm" \
  -e "REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd" \
  registry:2.6  
  
docker container ls
docker imgae pull 127.0.0.1:5000/hello-world #this should fail
docker login 127.0.0.1:5000 # user moby, password gordon
docker imgae pull 127.0.0.1:5000/hello-world #now this works
```

> The options for this container are:

> - _-v $(pwd)/auth:/auth_ - mount the local auth folder into the container, so the registry server can access htpasswd file;
> - _-e REGISTRY_AUTH=htpasswd_ - use the registry’s htpasswd authentication method;
> - _-e REGISTRY_AUTH_HTPASSWD_REALM='Registry Realm'_ - specify the authentication realm;
> - _-e REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd_ - specify the location of the htpasswd file.

</details>
</details>

### Using Docker Registry With Swarm

<details>
<summary>
running a registery on a swarm.
</summary>

docker registeries in a swarm, it works the same as localhost, just with some differnt commands, and because of the routing mesh, we don't need to make it insecure, all nodes can see the 127.0.0.1:5000 port.

we do need to decide how to store the images over time (volume driver)

for this example, we use play-with-docker again.

we will start with the template of 5 managers and no workers
```sh
docker node ls
docker service create --name registery --publish 5000:5000 registry
docker service ps registery

```

we can look at the catalog by changing the url and adding "/2/_catalog" and see the json.

```sh
docker image pull hello-world
docker image tag hello-world 127.0.0.1:5000/hello-world
docker image push 127.0.0.1:5000/hello-world # now we will see this repo in the catalog
docker image pull nginx
docker image tag nginx 127.0.0.1:5000/nginx
docker image push 127.0.0.1:5000/nginx # now we will see this repo in the catalog

doocker service create --name nginx -p 80:80 --replicas 5 --detach=false 127.0.0.1:5000/nginx
docker service ps nginx
```

when we are in a swarm, we must be able to reach the image, we can't build it locally on a node and get it from another node, we need something they can all access.

in this example, we didn't persistanlty store the images.
</details>

some 3rd pary registries are [Quay.io](Quay.io), gitLab Container Registry, or those located on each cloud storage provider
see also this [list](https://github.com/veggiemonk/awesome-docker#hosting-images-registries).

 
</details>
