<!--
ignore these words in spell check for this file
// cSpell:ignore Kubernetes kube kubelet kubectl Cloudfoundry Mazos bretfister httpenv katakoda Minikube kubeadm
-->

[Main](README.md)

## Docker Security

<!-- <details> -->
<summary>
//todo: add
</summary>

how can we make our code secure?

### Good Defaults

<!-- <details> -->
<summary>
//todo: add
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

</summary>
</details>

three
</details>

two
</details>

[Main](README.md)