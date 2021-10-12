<!--
ignore these words in spell check for this file
// cSpell:ignore vmware Moby cncf cmder
-->

## Section 2: The Best Way to Setup Docker for Your OS

<details>
<summary>
Getting Docker to run on the local machine.
</summary>

### Docker Editions: Which to Use?

<details>
<summary>
Different types of Docker.
</summary>
Docker has different versions for different operation systems, there's docker community edition (CE) and enterprize edition (EE), stable and edge releases.

[Docker Hub](https://hub.docker.com/)

docker is much bigger than what it was in the past, it's no longer just a 'container runtime'. the versions are updated fairly regularly.

there are three types of installs:

- Direct - just install on an OS, that kernel, that OS. started on linux, and eventually other machines.
- Mac/Win - a suite of tools for ease of use, creates a small VM to run the container with the docker.
- Cloud - docker for cloud machines, like AWS/Azure/Google, comes with features dedicated to the cloud provider.

we will focus on the Mac/Win distribution.\
the Enterprise version has additional features and support, as opposed to the community edition of Docker.

in the docker world, EDGE Means 'beta', it's released once a month, and the support is also one month. stable version are released quarterly (4 times a year), and have a 4 month support cycle (3 months + 1 month to migrate). the EE have longer support for stable releases (one year).

</details>

### Docker for Windows

<details>
<summary>
Installing Docker on Windows
</summary>

we can have either 'linux' or 'windows' containers, usually we use the term 'container' for a linux container. they are mostly the same, just with different binaries. the "Docker for Windows" installation works only on win10 Pro/Enterprise edition, so home edition should use the _Docker Toolbox_.

_NOTE: it seems like currently the Windows installation also works for windows 10 home edition._

also something called server 2016.

check if docker is running (shell, bash, powerShell)

```ps
docker version
```

it's also running hyper v.

change settings: shared device (not working for me), changing memory for the docker, etc...

getting the repo from git. using vscode. getting a [cmder.net gui emulator](https://cmder.net/) gui (unblock zip). running as powerShell.

</details>

###

</details>
