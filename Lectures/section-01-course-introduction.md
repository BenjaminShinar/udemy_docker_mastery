<!--
ignore these words in spell check for this file
// cSpell:ignore vmware Moby cncf
-->

## Section 1: Course Introduction and Docker Intro

<details>
<summary>
What is this course about?
</summary>

### Why Docker? Why Now?

<details>
<summary>
What is Docker, why does it exist, what does it solve, and why we should use it?
</summary>

Docker was released in 2013 as an open source project by a company known .cloud, which later changed their name to "Docker inc.". docker is a major shift, which will probably effect all industry, just like previous waves of industry shifts:

- From Mainframe to PC in the 90's.
- From bare-metal to virtual (vmware,hyper-v) in the 00's to stack many OS on one machine.
- From dataCenter storage to cloud storage in the 10's

and the next wave will be moving from _host_ to _containers_ (going _serverless_). everything will be running inside containers. the hard part will be the migration process. but for this wave, Docker is focused on the migration experience (both for developers and system personal-sysAdmins), so maybe things will be easier for us.

the docker logo (a whale carrying containers) is called **Moby Dock**, the mascot is a turtle called [gordon the turtle](https://twitter.com/gordontheturtle).

#### What are the Benefits of Docker?

> Docker is all about speed:
>
> - Develop faster.
> - Build faster.
> - Test faster.
> - Deploy faster.
> - Update faster.
> - Recover faster.

the _"matrix of hell"_, we have a lot of projects/ components, such as: static website, web frontend, background workers, user database, analytics database, queue. each of those can run on different environments: desktop, test/QA cluster, production cluster, public cloud, mainframe, windows server, edge devices. we need to manage dependencies for each combination.

Containers help us reduce complexity by making everything consistent across the boards, we package, ship and run the software the same way regardless of the environment. 80% of the time is spent maintaining old code and software, with docker we an reduce the amount of time by streamlining the maintenance to one 'place' which works for all devices.

examples of docker adoption by PayPal and MetLife.

[The Docker landscape](landscape.cncf.io)\
CNCF - Cloud Native Computing Foundation

</details>

</details>
