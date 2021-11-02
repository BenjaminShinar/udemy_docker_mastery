<!--
ignore these words in spell check for this file
// cSpell:ignore Kubernetes kube kubelet kubectl Cloudfoundry Mazos bretfister httpenv katakoda Minikube microk8s nodeport kubeadm CRD netshoot Traefik Istio Jenkin Kustomize CNAB
-->

[Main](README.md)

## Diving Deeper into Kubernetes

<details>
<summary>
what does the future hold for Kubernetes.
</summary>

The Kubernetes eco-system is vast and large. 

### The Future of Kubernetes

<details>
<summary>
Upcoming features and changes in Kubernetes
</summary>

#### Storage in Kubernetes

<details>
<summary>
Storage and Databases.
</summary>

Kubernetes still use volumes for storage, but this is harder than in swarm.
we assume kubernetes containers are stateless, but we still need consistent data.

StatefulSets are a new resource type, which make Pods more sticky. they are designed around databases, and require the same name, ip, etc.
we should avoid them for now, as they make stuff more complicated and we want our basic deployment to be simple.
We should use cloud databases if we can, rather than running them on the cluster.

we can add a volume to any container, we simple add it to the spec in the yaml. the volume is tied to the lifecycle of the pod. the containers inside the pod can share the volumes

a newer feature is __PersistentVolumes__, which are created at a cluster level, and are able to outlive the pod. this is a separated storage from the pods, so multiple pods (and the containers inside them) can share them.

there are also __CSI plugins__ (container storage interface). in the past, support for cloud vendors was built into kubernetes, but that meant bloated code and coupled release cycles.
eventually, the CSI standard was reached and any storage vendor can offer plugins.

</details>

#### Ingress Controller

<details>
<summary>
Ingress is a proxy service that directs and routes connection
</summary>

layer 7 control of http, http name routing, dns routing. like a proxy.
when we have multiple containers that use the same ports.
it's not exactly a load balancer. we have many containers on the same endpoint.
this isn't installed by default, but it does have built-in support.

we cab use nginx, Traefik, HAProxy, F5, Envoy, Istio or something else.

</details>

#### CRDs and the Operator Pattern

<details>
<summary>
Custom, 3rd part, plug-ins for kubernetes
</summary>

CRD stands for __Custom Resource Definition__

kubernetes becomes a platform by itself, through using 3rd party resources and controller, this isn't just using the API, it's extending it.
we can use complex programs (databases, monitoring tools, backups, custom ingresses) together with kubernetes, and make kubernetes aware of them.
we use kubectl to control those plugins as if they were native to kubernetes.

</details>

#### Higher Deployment Abstractions

<details>
<summary>
tools that build on top of kubernetes.
</summary>

kubernetes is expanding beyond just the core.
kubernetes has a limited built-in template, versioning, tracking and management of the app.
there are many ways to add tools on top of kubernetes, JenkinX, helm, etc... they all have opinions.

helm to kubernetes is kubernetes to containers. it's a way to create templates yaml, and it does so by using _charts_.
Docker desktop comes with "Compose on kubernetes" to make kubernetes a default orchestrator rather than swarm. it takes the docker-compose file and "translates" it into kubernetes yaml format.
this way we can use the same yaml file for development, test and production, the downside is that it doesn't support 100% of kubernetes features. this is even an open source software, so we just use it as a support tool.
we can mix and match tools as we see fit.

many of those tools are about template-ing yaml files. but now kubernetes has customizing built in, called _Kustomize_ which comes natively and works with existing yaml files.
in docker, the `docker app \<COMMAND>` also does similar stuff, and allows us to ship yaml files as images, 
__CNAB__  standard.

</details>

#### Kubernetes Dashboard

<details>
<summary>
Visual Representation of the Nodes
</summary>

a gui interface to show us data,and to allow basic users to interact the kubernetes without learning all the command line syntax.
kubernetes has a standard dashboard, even if it's not entirely built-in. if we use a cloud service, we would probably use their dashboard, or we would need to deploy the dashboard by ourselves.

we need to be careful, the dashboard might not have authentication, and we should protect them somehow.

</details>

#### Namespaces and Context

<details>
<summary>
Limiting the scope.
</summary>

namespaces allow us to limit scopes, in terms of visibility and accesses. this isn't related to Docker namespaces, we set the context of what we see in each mini-cluster.

Kubernetes has by default some namespaces to protect and isolate the system and control clusters. 

```sh
kubectl get namespaces
kubectl get all --all-namespace
```

the Context controls the cluster, the authentication/User and the namespace. this is all defined in a file *~/.kube/config*. we can get them with a kubelet command or setting the contexts.

```sh
kubectl config get-contexts
kubectl config set
```


</details>

#### The Future of kubernetes

<details>
<summary>
How will Kubernetes look in the future
</summary>

kubernetes will eventually settle down and become standardized, with less changes and more stability, reliability and security.

some parts will be removed, such as built in support to cloud providers, which will be used with plug-ins instead.

we will probably see a move towards a more declarative style. and more window server support.
and of course, some edge cases will be packaged outside of the base program.

some more projects built on top of Kubernetes. like service mesh which will help with distributed work.

</details>

</details>

</details>


[Main](README.md)