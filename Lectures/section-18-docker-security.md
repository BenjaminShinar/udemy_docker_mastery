<!--
ignore these words in spell check for this file
// cSpell:ignore kube kubelet kubectl Cloudfoundry Mazos httpenv katakoda Minikube kubeadm Sysdig Falco Seccomp Distroless Snyk Trivy
-->

[Main](README.md)

## Docker Security

<details>
<summary>
Making Docker Safe and secure
</summary>

how can we make our code secure?

### Good Defaults

<details>
<summary>
Good defaults and some stuff we can do to increase docker security.
</summary>

first thing to do is to understand the docker security engine

##### Docker Control Groups and Namespaces

<details>
<summary>
Limiting visibility and resource consumptions.
</summary>

Docker is possible because of linux kernel namespaces, limiting the scope of the program of what it can do and what it can access. it provides isolation to the containers, the file space.

Control groups (CGroups) limit containers in terms of resources, so we can decide how much memory and cpu any container can receive. 


</details>

#### Docker Engine's Out-Of-The-Box Security Features

<details>
<summary>
What can we do with the basic Docker to be more secure.
</summary>

Only use images you trust.
Docker uses some security features by default, which makes docker a bit more secure than directly running software.

</details>

#### Docker Bench, The Host Configuration Scanner

<details>
<summary>
A tool to scan the host for issues in the installation of docker configurations.
</summary>

this will tell us if we have problems in our docker configurations, such as missing permissions, things that aren't enabled, and so on.

[Docker Bench Security](https://github.com/docker/docker-bench-security)

</details>

#### Using USER in Dockerfiles to Avoid Running as Root

<details>
<summary>
Running containers as a none-admin user.
</summary>

we can get some extra security if we run the container as a user which isn't the root/admin user. if we look at how daemon services work, we can see that they spawn of most of the work into process that run as normal users, and not as the admin. this protects the container from vulnerabilities.
```sh
docker container run --rm -d --name nginx nginx
docker container top nginx
```
we should also do something similar, if we run a programming language file in a container (like python, node), then it will usually run as root. we should add the `USER` stanza when we build an image to change the user that is running.

```sh
docker container run --rm -it node
whoami #shows root!
```
this also requires us to use *--chown* when we `RUN` or `COPY`.

note: php is notorious with this.

this also prevents the rare cases of linux or docker vulnerabilities, such as the [dirty cow exploit](https://en.wikipedia.org/wiki/Dirty_COW).
</details>

#### Docker User Namespaces for Extra Host Security

<details>
<summary>
Prevent containers from escaping into the root namespace in the host machine
</summary>

The next level of security is to protect the host machine from containers trying to mess with it. we make docker run the containers as non-root user. The user namespace feature will make the containers spawn as weak users, and not as root.

unfortunately, not everything works with this feature. this can be problematic with volumes and bind mounts.

</details>

#### Code Repo and Image Scanning for CVE's
<details>
<summary>
Scan the dependencies and images for vulnerabilities.
</summary>
tools such as *Snyk* that scan dependencies for vulnerabilities, github also does this by default. start doing this as early as possible, part of the *"Shift left"* mindset, we can also scan our images (such as the OS) for known issues.

Tools that scan for CVE vulnerabilities, CVE stands for common variability exploit, we find issues that are known and that we should avoid. there is *MicroScanner* or *Trivy* as open source options.

There is also the issue of when to scan: at image build time? at runtime? before pushing to the registry? etc...
</details>

#### Sysdig Falco, Content Trust, and Custom Seccomp and AppArmor Profiles

<details>
<summary>
Bad behavior during runtime
</summary>
Sysdig Falco is a tool that monitors containers for suspicious behavior. it audits and logs those behaviors. this comes with a default set of 'bad behavior' and we can add more rules of our own.

Content Trust is a set of tools to verify the all parts of the pipeline (building images, publishing, etc...) are signed.

customizing linux capabilities to increase security per container. having a well defined security policy depending on the image and the container.
</details>

#### Docker Rootless Mode

<details>
<summary>
Running the docker engine without root
</summary>

doesn't always work, doesn't allow custom networking. but maybe we can run docker as a regular user, it works well for Development, and maybe with some effort, it can also be used in Deployment.
</details>

#### The Security Top 10 Differences for Windows Containers

<details>
<summary>
Unique security stuff for windows containers
</summary>
windows containers don't have many of the linux specific features.
</details>

#### What are Distroless Images?

<details>
<summary>
Images that are bare naked from more applications.
</summary>
Distroless images are images without any additional packages, such as *"apt"* or *"yum"*, some languages (c, golang) can build without those, but most don't. the problem is that those containers are hard to debug and understand, they might not even have a shell command.\
the gains might be more theoretical than anything else.
</details>

#### Are Swarm and Kubernetes Secrets Really Secure?

<details>
<summary>
Protecting data from inside the container
</summary>
docker secrets are stored in container memory, the container needs those, so they are exposed if someone gains access to it. we need to prevent those secrets from ever getting permanently to disk. the perfect solution (one time passwords) isn't feasible for most cases.
</details>

</details>

</details>

[Main](README.md)