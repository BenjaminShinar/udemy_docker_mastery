<!--
ignore these words in spell check for this file
// cSpell:ignore 
-->

## Kubernetes

<details>
<summary>

</summary>

### The What And Why of Kubernetes

<details>
<summary>
The origins of Kubernetes and its benefits.
</summary>

a container orchestrator makes many servers act like one. Kubernetes was released in 2012 by google, it's a set of APIs that run on top of docker (or other container runtime), that do management.
It provides api/cli to manage containers, similar to swarm, which is called "kube control", most cloud vendor provide Kubernetes as a service, and others cat make thier own "distributions" of Kubernetes with additional tools.

as we recall from the [swarm lecture](//todo: make link), there are benefits for orchestration, and many projects are moving towards it. it's not mandatory, but very useful.
a rough estimate for the benefit of orchestration is the number of servers/apps and the change rate. orchestration makes managing change easy, especially when things scale.

some platforms for orchestrations are swarm, kubernetes, and ECS (only for AWS), cloudfoundry, mazos and marathon. kubernetes is hybrid, which means it can run on every cloud service and inside data centers and locally.

the next step is to decide on the distribution for kubernetes, there are many vendors.
One optionn is to use the raw githubs upstream kubernetes solution. but vendors can bundle many additional tools, which are required to use kubernetes in the real scenario.

#### Kubernetes Or Swarm

<details>
<summary>
Comparing two orchestration options.
</summary>

Kubernetes and swarm are both container orchestrations tools that solve similar problems.
swarm is eaiser to start with deployment and management, while kubernetes has more options for control and flexability, and can solve stuff that swarm can't.

Advantages to Swarm:
- Comes with docker, a single vendor solution. runs whereever docker runs, linux, windows, arm devices, 32-bit, 64-bit, windows server.
- Lightweight, doesn't consume many resources.
- Easiest orchestrator to deploy and run 
- Has the basic, most common and most used features to handle most use cases and issues.
- Out of the box solution with docker.
- Secure by default.
- Easier to troubleshoot.

advantages for Kubernetes:
- Wide support from cloud vendors and mand kubernetes distributions. "Kubernetes first support" model.
- Flexability for many use cases, more edge cases than swarm.
- Trendy, popular, nearly industry standard.
	- "No one ever got fired for buying IBM."

</details>
</details>

### Starting with Kubernetes

<details>
<summary>
Installing kubernetes and running our first pod.
</summary>

#### Kubernetes Architecture Terminology

<details>
<summary>
Commonly used terms and system topology.
</summary>

[Kubernetes Components Overview](https://kubernetes.io/docs/concepts/overview/components/)

- Kubernetes: the whole orchestration system abbrivated as __K8s__ ("k-eights" - the 8 is the eight letters skipped), also __Kube__.
- Kubectl(kube control): the command line tool, configure kubernetes, manage applications on it. 
- Node: single server in the kubernetes cluster
- Kubelet: the kubernetes agent running on the nodes. talks back to the kubernetes master. there is no similar thing in swarm.
- Control Plane:(Master) the set of containers that manage the cluster, similar to swarm managers. includes the API sever, scheduler, controller manager, etcd database.

Function | Swarm | Kubernetes
------|-------|---------
Command line tool (CLI) | docker cli | Kubectl
Node agent | none | kubelt
cluster management | manager nodes | control plane containers ("master")



Raft protocl to maintain consistency.

a master has 
- etcd database,built on top of Raft. 
- API
- Scheduler
- Contoller manager to detect differnces between the current state and the required state
- Core DNS
- other stuff (networking)

a worker nod has
- Kubelet agent
- Kube-proxy 
- other stuff

Kubernetes Containers Abstractions:

- Pod: basic unit of deployment, pods have containers (one or more running together) on a single node.
- Contollers - control the pods, this is what we use to deploy pods. (like services in swarm). types of contollres
	- Deployment
	- Replica Set
	- Stateful Set
	- Daemon Set
	- Job
	- CronJob
- Service (not service in swarm) - the network endpoint to connect to a pod. a persistent inpoint in the cluster
- Namespace - filtered group of object in the cluster, not a security feature, not the same as docker namespaces.

</details>

#### Kubernetes Local Install

<details>
<summary>
setting up kubernetes on a local machine.
</summary>

Kubernetes is a series of containers, CLI's and configurations. there are different installations packages, but we will focus on the easiest one to install for learnning purposes.

Docker desktop is probably the easiest think to use, otherwise, we can use Docker toolboox or MicroK8s. we can also use online options such as play-with-k8s and katakoda.

docker desktop is easy to install and manages installations of the correct version of kubernetes, and can help with resources management.
Minikube is another tool to run kubernetes, which has similar syntax to docker, but doesn't have kubectl out of the box.
for linux we can use microK8s. it uses snap for installtion (rather than apt or yum). to access kubectl we need to prefix it, but we can alias this in the bash profile file.
```sh
#alias kubectl
alias kubectl=microk8s.kubectl
```

</details>

#### Kubernetes command line - Kubectl

<details>
<summary>
creating deployments, scaling and inspecting objects.
</summary>

in kubernetes, we have more layets of abstractions: pod inside a replicaSet inside a Deployment.

kubectl is still evolving, there are many ways to do the same thing, and stuff is changing all the time. the instructions in the video are pre 2021 era. 

> kubectl run - like running a container
> kubectl create - like docker swarm service create
> kubectl apply - like stack deploy / update, via yaml files.

since then, `kubectl run` was changed to behave more like `docker container run`, so it creaes a single pod (not repllicaSet). the deployment functionaliy was moved to `kubectl create deployment`.


we first check that we have kubernetes and that's its running, we have a client and a server versions. if we don't see a server version that means we have a problem.\
we can deploy pods either via the cli command or with yml file. we can use the *get* command to list our pods or our objects. we then do cleanup by deleting the object by name which would remove all of it's sub objects.


```sh
kubectl version
kubectl run my-nginx-name --image nginx
kubectl get pods
kubectl get all
kubectl delete deployment my-nginx-name
```

the updated form is
```sh
kubectl create deployment some-name --image nginx
```
like in swarm, we have the option to scale the number of pods in the replica set. there are shortend forms to the scale command.

```sh
#still old form, pre 1.18 version
kubectl run my-apache --image httpd
kubectl get all
kubectl scale deploy/my-apache --replicas 2
kubectl scale deployment my-apache --replicas 2
```

1. deployment updated to 2 replicas
2. replicaset contollre sets pod count to 2
3. contol plane assigns node to pod
4. Kublet sees pod is needed, starts container

now we want to inspect our objects

for the logs, we can add flags like  _--follow_ or _--tail 1_, if we want logs from multiple pods,we can use a selector with a label (_-l_ flag). in the real world we should use logging tool
the describe commands also shows events, not just configurations.
```sh
kubectl get pods
kubectl logs deployment/my-apache
kubectl logs -l run=my-apache
#we need pods name
kubectl describe pod/my-apache-XXXXX-XXX 
#all pods
kubectl describe pods
```

if we remove a pod, it will be recreated by the replica set. we start by creating a second terminal, so we can monitor what's happening

the *-w* flag means we "watch" the changes as they happen, we see how the pods are terminated and re-created
```sh
kubectl get pods -w 

#terminal 2
kubectl delete pod/my-apache-XXXXX-XXX 
kubectl get pods
kubectl deployment my-apache
```

kubernetes allows us flexability, but it's dangerous and we might end up shooting ourslves in the foot.
</details>
</details>

### Expose Kubernetes Ports

<details>
<summary>
Making pods visible to the outside world via services
</summary>

Services and types, what is expose internally and externally

#### Service types

<details>
<summary>
Four service types.
</summary>

exposing containers to the outside world, allow them to recieve connections, we use the `kubectl expose` command to create a serice for __existing pods__.
a service is a consistent end point for a pod, the common way is by using CoreDNS to resolve services by name.

there are four diffrent types of serivs:
- ClusterIP (default)
- NodePort
- LoadBalancer
- ExternalName

ClusterIP is only availble in the cluster, has it's own dns address, a single, internal virtual IP. pods can reach serivce on apps port number. like in swarm, when we have an internal ip.
NodePort is for stuff outside the cluster, high port allocated on each node. port is open of every node IP,
the LoadBalancer is more advanced, mostly used on the cloud, an external load balancer, comes from the infrastrucutre.
ExternalName is less common, its about resolving external names in the DNS. might be used in migration

there is also a kubernetes _ingress_ for http requests.

Creating a ClusterIP Service:

again, we create two terminals, one that watches the pods and one tha does the actual thing. the container itself knows to return the enviornment variables.
we create the service with the `kubectl expose` command and the port number. this doesn't change the pods actually. it creates a ClusterIP.
this ip is only accessible from within the cluster.
```sh
kubectl get pods -w
#other shell
kubectl create deployment httpenv --image=bretfister/httpenv
kubectl scale deployment/httpenv --replicas =5
kubectl expose deployment/httpenv --port 8888
kubectl get service
```

if we are using a hosting OS, such as docker desktop, we need to create a pod to curl the other service.
(uses the old syntax, we explictly specify that we want a pod, not a deployment) the generator is the template, this is supposed to look similar to the docker container run command.
the double dash flag -- means that there are no more flags, and that the following is command to run on the pod.
```sh
kubectl run --generator run-pod/v1 tmp-shell --rm -it --image bretfisher/netshoot -- bash
#this will be a new interactive pod container
curl httpenv:8888
#when we exit the shell, the container will be removed
```
if we are on linux and we are runnig kubernetes locally, we can simply curl it from the shell with the ip address and the port (but not with the service name)
```sh
curl ##.###.##.##:8888
```

Creating a node Port and LoadBalancer service:

contiuing with the same set up.
if we want to expose a node port externally, we need to set the type with the _--type_ flag.
now we can see the new service, with the internal and external port. it's reversed from the docker syntax, now it's internal on the left and external on the right.
the services are additive, each level creates the base level, so nodeport creates a clusterIP, and loadbalancer created a nodePort
docker 
```sh
kubectl get all
kubectl expose deployment/httpenv --port 8888 --name httpenv-np --type NodePort
kubectl get services
#get the external ip, it's in the range of 30000-33... ports.
curl localhost:3#####
```

the docker desktop comes with a load balancer, which allows us. minikub, microk8s and kubeadm don't have a built-in load balancer (in the time of the video)
now we can use the lowport address
```sh
kubectl expose deployment/httpenv ---port 8888 --name httpend-lb --type LoadBalancer
kubectl get services
curl localhost:8888
```

for cleanup, we can delete multiple objects of differnt types in the same line
```sh
kubectl delete service/httpenv serivce/httpenv-np
kubectl delete serivce/httpenv-lb deployment/httpenv
```


</details>

#### Kubernetes Services DNS

<details>
<summary>
FQDN: fully qualified domain name
</summary>

Even though DNS usually works out of the box (when we are inside a container), it's  built-in option into kubernetes.
this is called DNS-Based service discovery, just like what we had in docker or swarms. in kubernetes we can get fully qualified domain names, this works together with namespaces

```sh
kubectl get namespaces
curl <hostname>.<namespace>.svc.cluster.local
```

</details>
</details>


### Kubernetes Managment Techniques

<details>
<summary>
</summary>

#### Generators

<details>
<summary>
seeing the deafult templates for common commands.
</summary>
Run, create and expose Generators.
generators are helper templates that activate when we use the `rul`,`creat` and `expose` commands of kubectl.
every resource in kubernetes has a specification, this specification is created by the generator for the defaults.

generators are "opinionated defaults", they work for most use cases.


when we run a command, the rest of the options are filled in by the generator template, we can specify which generator version we want to use.
the easiert way to see the specfications is to use the _--dry-run_ flag  together with the output flag _-o yaml_ to show the template.

in the video, we see the api version V1, the kind (deployment), metadata such as labels, replicas, a selector, a template for the replicaSet and the containers.
```sh
kubectl create deployment sample --image nginx --dry-run -o yaml
```

when we create a job, we see different stuff, such as restartPolicy which is set to Never


```sh
kubectl create job test --image nginx --dry-run -o yaml
```
we can also see the template of the `expose` command, it still needs the deployment to exist before, so we need to create it.  we can see the spec is a bit different this time.
```sh
#fails
kubectl expose deployment/test --port 80 --dry-run -o yaml
kubectl create deployment test --image nginx
kubectl expose deployment/test --port 80 --dry-run -o yaml
kubectl delete deployment test
```

however, in the future (post 2017 probably), we won't be using templates that much.

</details>

#### The Future of Kubectl Run

<details>
<summary>
//todo
</summary>

the kubectl run command is scheduled to change and be reduced, this should happen in version 1.12-1.15, the idea is to reduce the scope of the run command and make it able to only create pods.
the rest of the functionality would go in `kubectl create` command. this is done to make the `run` command similar to `docker container run`, so it'll be easy to create one-off tasks.
this won't be used in production enviornments, but has it's uses.

we will see this in the future, it will be a long time until everybody updates this.

lets see what's really created based on the template

base | extra flags | created 
-----|------------|-------------
`kubectl run test --image nginx --dry-run` |  none | deployment created
`kubectl run test --image nginx --port 80 --expose --dry-run` | ports, expose | deployment and service created
`kubectl run test --image nginx --restart OnFailure --dry-run` | restart | batch job created
`kubectl run test --image nginx --restart Never --dry-run` | restart | pod created (future deafult)
`kubectl run test --image nginx --schedule "*/1 * * * *" --dry-run` | schedule | cron job created

</details>

#### Imperative vs. Declarative

<details>
<summary>
//todo
</summary>

#### Three Management Approaches

<details>
<summary>
//todo
</summary>

# four
</details>

# three
</details>

# two file closing
</details>