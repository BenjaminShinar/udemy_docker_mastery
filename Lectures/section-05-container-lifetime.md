<!--
ignore these words in spell check for this file
// cSpell:ignore tini mkdir psql Jeykll bindmount bretfisher
-->

## Container Lifetime & Persistent Data: Volumes, Volumes, Volumes

<details>
<summary>
Persistent data, immutable, ephemeral containers. data volumes, bind mounts.
</summary>

Shell Commands:

the shell commands are sometimes different depending on the shell (bash, powershell, etc) and the OS

| command                 | powershell | cmd.exe | bash   |
| ----------------------- | ---------- | ------- | ------ |
| print working directory | ${pwd}     | %cd%    | $(pwd) |

if we have spaces, we need quotes.

Database Passwords in Containers:

until 2020, postgres could run without a password, but that changed, we now must either give it a password as part of the environment variable.\
So we should either use `POSTGRES_PASSWORD=my_password` to provide a password or
`POSTGRES_HOST_AUTH_METHOD=trust` to not require at all.

they are passed with the _--env_ flag to the container run command.

### Container Lifetime & Persistent Data

<details>
<summary>
The problem of consistency
</summary>

containers are usually immutable and ephemeral - unchanging,and interchangeable.

**Immutable infrastructure** - only re-deploy containers, never change them. but what happens with the database? data that the app creates or stores? ideally, it shouldn't be stored in the containers, this is _separation of concerns_.

we want persistent data, this is a problem that was born in the world of docker.

there are two solutions for this problem, _Data Volumes_ and _Bind Mounts_:

- Data volumes are a a configuration option that create a special location outside of the UFS (union file system, the read/ copy of write layer) this lives cross container removal.
- Bind Mounts are a way to link containers path to a host path

</details>

### Persistent Data: Data volumes

<details>
<summary>
Data volume, persistent data across containers restarts.
</summary>

we can tell a container that it needs to worry about a volume thorough a `VOLUME` stanza in the dockerfile.

we can look at this in the mysql official dockerfile. volume data can only be manually deleted (or pruned). we can the data volume as part of the _image inspect_ command. when we inspect a container, we can see the volume data on the container with that destination, which points to somewhere in the host files.

we can also look at the volumes on the local host.

```sh
docker volume ls
```

docker for linux will have the data on the disk, but docker for windows is using a small vm.

the volumes outlive the executables. but if we wish to have the volumes names, we can specify a name for the volume (_-v with a name_), either a new or connect to an existing name.

this is a named volumed

```sh
docker container run -d --name mysql -e MYSQL_ALLOW_EMPTY_PASSWORD=True --volume mysql-db:/var/lib/mysql mysql
docker volume inspect mysql-db
docker container inspect mysql
```

we check the _Mounts_ data of the containers to see the actual mounting point.

we can use the `docker volume create` command if we want some special flags and drivers.

#### Assignment #1: Named Volumes

<details>
<summary>
using the same mounted volume for containers with different versions!
</summary>

a situation with a database upgrade. normally we would just do an update, but how do we replace a container to get a new db version?

> - create a _postgres_ container with the named volume psql-data using version 9.6.1.
> - Use docker hub to understand what the volume path should be (on the right side)
> - Check logs, stop container.
> - Create a new _postgres_ container with the same named volume psql-data using version 9.6.2 instead.
> - checks logs and validate (less logs)
> - (this will usually just work with small patches, not with big updates)

[Docker hub postgres](https://hub.docker.com/_/postgres), look at the tags to find older versions, and then open up the dockerfile and search for the `VOLUME` stanza

```sh
#docker image pull postgres:9.6.1-alpine
#docker image pull postgres:9.6.2-alpine
docker container run -d --rm --name pg_1 --volume psql-data:/var/lib/postgresql/data postgres:9.6.1-alpine
docker container logs pg_1
docker container stop pg_1
docker container run -d --rm --name pg_2 --volume psql-data:/var/lib/postgresql/data postgres:9.6.2-alpine
docker container logs pg_2
```

</details>

</details>

### Persistent Data: Bind Mounting

<details>
<summary>
Binding data to files on the host machine.
</summary>

bind mounting to get persistent data.

Mapping a host file (file or directory) to file (or folder) in the container. the two will point to the same location on disk skipping the UFS, the host wins over the container.

we can't specify them in the docker file, only in the `docker container run` command at runtime. the format is still the _--volume, -v_, on the left side of the ':' we have a full path to the host file, starting with a forward slash, and on the right side we have a path on the container side.

```sh
docker container run --volume //c/users/a/b/c:path/container -d --name my_nginx nginx
```

lets take a look at dockerfile-sample-2 again. \
there is no volume here, the bind map doesn't require it.

```sh
docker container run -d --rm --name my_nginx1 -p 80:80 --volume ${pwd}:/usr/share/nginx/html nginx
docker container inspect --format "{{ json  .Mounts }}" my_nginx2

docker container run -d --rm --name my_nginx2 -p 8080:80 nginx
```

in the first one we used the regular nginx image, but directed it to the html file on the host (not copying it), in the second container, we run it as usual and saw the regular html page.

we can open a connection to the mapped container and see the directory, with the html and the docker file. we could even create files locally and see them appear there, and vice versa.

```sh
docker container exec -it my_nginx1 bash
cd usr/share/nginx/html
touch text.txt
echo "hello"  > text.txt
```

edit files on the host machine that interact with the container.

#### Assignment #2:Edit Code Running In Containers With Bind Mounts

<details>
<summary>
using bind mounts to track changes on host, using Jeykll for static html page
</summary>

> - Use a Jeykll "static site generator" to start a local web server. this is what is used to create github pages and other static stuff.
> - We will see things being changed when we edit our local file.
> - source code is in 'bindmount-sample-1' folder
> - start container with `docker container run -p 80:4000 -v ${pwd}:/site bretfisher/jeykll-serve `
> - change files in the _\_posts_ directory and refresh page to see them in effect

```
docker container run -d --rm -p 80:4000 -v ${pwd}:/site bretfisher/jekyll-serve
```

</details>

</details>

</details>
