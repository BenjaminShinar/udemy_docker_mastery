<!--
ignore these words in spell check for this file
// cSpell:ignore tini mkdir
-->

## Container Lifetime & Persistent Data: Volumes, Volumes, Volumes

<!-- <details> -->
<summary>
Persistent data, immutable, ephemeral containers. data volumes, bind mounts.
</summary>

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

we can also look at the volumes

```sh
docker volume ls
```

the volumes outlive the executables. but if we wish to have the volumes names, we can specify a name for the volume, either a new or connect to an existing name

```sh
docker container run -d --name mysql -e MYSQL_ALLOW_EMPTY_PASSWORD=True --volume mysql-db:/var/lib/mysql mysql
```

we can use the `docker volume create` command

</details>

##

</details>
