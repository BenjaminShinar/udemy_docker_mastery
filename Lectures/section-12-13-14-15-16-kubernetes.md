<!--
ignore these words in spell check for this file
// cSpell:ignore Kubernetes kube kubelet kubectl Cloudfoundry Mazos bretfister httpenv katakoda Minikube microk8s nodeport kubeadm CRDS netshoot
-->

[Main](README.md)

## Kubernetes

<details>
<summary>
Kubernetes is service agnostic container orchestrator.
</summary>

### The What And Why of Kubernetes

<details>
<summary>
The origins of Kubernetes and its benefits.
</summary>

a container orchestrator makes many servers act like one. Kubernetes was released in 2012 by google, it's a set of APIs that run on top of docker (or other container runtime), that do management.
It provides api/cli to manage containers, similar to swarm, which is called "kube control", most cloud vendor provide Kubernetes as a service, and others cat make their own "distributions" of Kubernetes with additional tools.

as we recall from the [swarm lecture](//todo: make link), there are benefits for orchestration, and many projects are moving towards it. it's not mandatory, but very useful.
a rough estimate for the benefit of orchestration is the number of servers/apps and the change rate. orchestration makes managing change easy, especially when things scale.

some platforms for orchestrations are swarm, kubernetes, and ECS (only for AWS), Cloudfoundry, Mazos and marathon. kubernetes is hybrid, which means it can run on every cloud service and inside data centers and locally.

the next step is to decide on the distribution for kubernetes, there are many vendors.
One options is to use the raw github upstream kubernetes solution. but vendors can bundle many additional tools, which are required to use kubernetes in the real scenario.

#### Kubernetes Or Swarm

<details>
<summary>
Comparing two orchestration options.
</summary>

Kubernetes and swarm are both container orchestrations tools that solve similar problems.
swarm is easier to start with deployment and management, while kubernetes has more options for control and flexibility, and can solve stuff that swarm can't.

Advantages to Swarm:
- Comes with docker, a single vendor solution. runs wherever docker runs, linux, windows, arm devices, 32-bit, 64-bit, windows server.
- Lightweight, doesn't consume many resources.
- Easiest orchestrator to deploy and run 
- Has the basic, most common and most used features to handle most use cases and issues.
- Out of the box solution with docker.
- Secure by default.
- Easier to troubleshoot.

advantages for Kubernetes:
- Wide support from cloud vendors and and kubernetes distributions. "Kubernetes first support" model.
- Flexibility for many use cases, more edge cases than swarm.
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

- Kubernetes: the whole orchestration system abbreviated as __K8s__ ("k-eights" - the 8 is the eight letters skipped), also __Kube__.
- Kubectl(kube control): the command line tool, configure kubernetes, manage applications on it. 
- Node: single server in the kubernetes cluster
- Kubelet: the kubernetes agent running on the nodes. talks back to the kubernetes master. there is no similar thing in swarm.
- Control Plane:(Master) the set of containers that manage the cluster, similar to swarm managers. includes the API sever, scheduler, controller manager, etcd database.

Function | Swarm | Kubernetes
------|-------|---------
Command line tool (CLI) | docker cli | Kubectl
Node agent | none | kubelet
cluster management | manager nodes | control plane containers ("master")



Raft protocol to maintain consistency.

a master has 
- etcd database,built on top of Raft. 
- API
- Scheduler
- Controller manager to detect differences between the current state and the required state
- Core DNS
- other stuff (networking)

a worker nod has
- Kubelet agent
- Kube-proxy 
- other stuff

Kubernetes Containers Abstractions:

- Pod: basic unit of deployment, pods have containers (one or more running together) on a single node.
- Controllers - control the pods, this is what we use to deploy pods. (like services in swarm). types of controllers
	- Deployment
	- Replica Set
	- Stateful Set
	- Daemon Set
	- Job
	- CronJob
- Service (not service in swarm) - the network endpoint to connect to a pod. a persistent In-point in the cluster
- Namespace - filtered group of object in the cluster, not a security feature, not the same as docker namespaces.

</details>

#### Kubernetes Local Install

<details>
<summary>
setting up kubernetes on a local machine.
</summary>

Kubernetes is a series of containers, CLI's and configurations. there are different installations packages, but we will focus on the easiest one to install for learning purposes.

Docker desktop is probably the easiest think to use, otherwise, we can use Docker toolbox or MicroK8s. we can also use online options such as play-with-k8s and katakoda.

docker desktop is easy to install and manages installations of the correct version of kubernetes, and can help with resources management.
Minikube is another tool to run kubernetes, which has similar syntax to docker, but doesn't have kubectl out of the box.
for linux we can use microK8s. it uses snap for installation (rather than apt or yum). to access kubectl we need to prefix it, but we can alias this in the bash profile file.
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

in kubernetes, we have more layers of abstractions: pod inside a replicaSet inside a Deployment.

kubectl is still evolving, there are many ways to do the same thing, and stuff is changing all the time. the instructions in the video are pre 2021 era. 

> kubectl run - like running a container
> kubectl create - like docker swarm service create
> kubectl apply - like stack deploy / update, via yaml files.

since then, `kubectl run` was changed to behave more like `docker container run`, so it creates a single pod (not replicaSet). the deployment functionality was moved to `kubectl create deployment`.


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
like in swarm, we have the option to scale the number of pods in the replica set. there are shorthand forms to the scale command.

```sh
#still old form, pre 1.18 version
kubectl run my-apache --image httpd
kubectl get all
kubectl scale deploy/my-apache --replicas 2
kubectl scale deployment my-apache --replicas 2
```

1. deployment updated to 2 replicas
2. ReplicaSet controller sets pod count to 2
3. Control plane assigns node to pod
4. Kubelet sees pod is needed, starts container

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

kubernetes allows us flexibility, but it's dangerous and we might end up shooting ourselves in the foot.
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

exposing containers to the outside world, allow them to receive connections, we use the `kubectl expose` command to create a service for __existing pods__.
a service is a consistent end point for a pod, the common way is by using CoreDNS to resolve services by name.

there are four different types of services:
- ClusterIP (default)
- NodePort
- LoadBalancer
- ExternalName

ClusterIP is only available in the cluster, has it's own dns address, a single, internal virtual IP. pods can reach service on apps port number. like in swarm, when we have an internal ip.
NodePort is for stuff outside the cluster, high port allocated on each node. port is open of every node IP,
the LoadBalancer is more advanced, mostly used on the cloud, an external load balancer, comes from the infrastructure.
ExternalName is less common, its about resolving external names in the DNS. might be used in migration

there is also a kubernetes _ingress_ for http requests.

Creating a ClusterIP Service:

again, we create two terminals, one that watches the pods and one tha does the actual thing. the container itself knows to return the environment variables.
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
(uses the old syntax, we explicitly specify that we want a pod, not a deployment) the generator is the template, this is supposed to look similar to the docker container run command.
the double dash flag -- means that there are no more flags, and that the following is command to run on the pod.
```sh
kubectl run --generator run-pod/v1 tmp-shell --rm -it --image bretfisher/netshoot -- bash
#this will be a new interactive pod container
curl httpenv:8888
#when we exit the shell, the container will be removed
```
if we are on linux and we are running kubernetes locally, we can simply curl it from the shell with the ip address and the port (but not with the service name)
```sh
curl ##.###.##.##:8888
```

Creating a node Port and LoadBalancer service:

continuing with the same set up.
if we want to expose a node port externally, we need to set the type with the _--type_ flag.
now we can see the new service, with the internal and external port. it's reversed from the docker syntax, now it's internal on the left and external on the right.
the services are additive, each level creates the base level, so nodeport creates a clusterIP, and load balancer created a nodePort
docker 
```sh
kubectl get all
kubectl expose deployment/httpenv --port 8888 --name httpenv-np --type NodePort
kubectl get services
#get the external ip, it's in the range of 30000-33... ports.
curl localhost:3#####
```

the docker desktop comes with a load balancer, which allows us. minikube, microk8s and kubeadm don't have a built-in load balancer (in the time of the video)
now we can use the low port address
```sh
kubectl expose deployment/httpenv ---port 8888 --name httpenv-lb --type LoadBalancer
kubectl get services
curl localhost:8888
```

for cleanup, we can delete multiple objects of different types in the same line
```sh
kubectl delete service/httpenv service/httpenv-np
kubectl delete service/httpenv-lb deployment/httpenv
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

### Kubernetes Management Techniques

<details>
<summary>
Hands on, Practical usage of Kubernetes
</summary>

#### Generators

<details>
<summary>
seeing the default templates for common commands.
</summary>
	
Run, create and expose Generators.
generators are helper templates that activate when we use the `rul`,`create` and `expose` commands of kubectl.
every resource in kubernetes has a specification, this specification is created by the generator for the defaults.

generators are "opinionated defaults", they work for most use cases.

when we run a command, the rest of the options are filled in by the generator template, we can specify which generator version we want to use.
the easiest way to see the specifications is to use the _--dry-run_ flag  together with the output flag _-o yaml_ to show the template.

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
Changes to the run command.
</summary>

the kubectl run command is scheduled to change and be reduced, this should happen in version 1.12-1.15, the idea is to reduce the scope of the run command and make it able to only create pods.
the rest of the functionality would go in `kubectl create` command. this is done to make the `run` command similar to `docker container run`, so it'll be easy to create one-off tasks.
this won't be used in production environments, but has it's uses.

we will see this in the future, it will be a long time until everybody updates this.

lets see what's really created based on the template

base | extra flags | created 
-----|------------|-------------
`kubectl run test --image nginx --dry-run` |  none | deployment created
`kubectl run test --image nginx --port 80 --expose --dry-run` | ports, expose | deployment and service created
`kubectl run test --image nginx --restart OnFailure --dry-run` | restart | batch job created
`kubectl run test --image nginx --restart Never --dry-run` | restart | pod created (future default)
`kubectl run test --image nginx --schedule "*/1 * * * *" --dry-run` | schedule | cron job created

</details>

#### Imperative vs. Declarative

<details>
<summary>
declarative and imperative kubernetes deployment.
</summary>

how to use kubernetes with best practices, basic terms

> Imperative: Focus on _how_ a program operates. /
> Declarative: focus on _what_ a program should accomplish.

example of imperative and declarative styles as they relate to getting a cup of coffee.

kubernetes imperative - the way to start learning any tool. explicit instruction, such as:
`kubectl run`,`kubectl create deployment`,`kubectl update`.
We start in a known state, different commands are required to change the object, and different objects require different commands.
imperative style is easy for humans to understand, but unfortunately, it's hard to automate.

declarative style works with an unknown state but a known end state. the same command does most of the work.
`kubectl apply -f my-resources.yaml`
(small exception for _delete_) 
a yaml can be a single file or split across many files.

the declarative approach works for large and complex situations.
something called _GetOps_ model.

</details>

#### Three Management Approaches

<details>
<summary>
Imperative, Imperative Objects, Declarative Objects
</summary>

so far we have been doing imperative commands: `run`,`expose`, `scale`,`edit`,`create deployment`.
this works well for learning, for a single developer, for playing around.

there is a middle ground between imperative and declarative, this is the __imperative objects__:
`create -f file.yml`, `replace -f file.yml`, `delete...`
this works for small environments, single file per command, all the changes are stored in the files, so we don't lose them, but we still use the human words to describe what we do (create, replace, delete)

the declarative approach uses __declarative objects__
`apply -f file.yml`, `apply -f dir\`,`diff`.
best for production, easier to automate, but harder to understand as humans and hard to predict changes.
this is most similar to swarm stack, where the same command does everything.

the best advice is to avoid mixing the approaches, and to use declarative objects as soon as possible, we combine this with source control so we can always recover our changes.

> Bret's Recommendations:
> - Learn the imperative CLI for easy control of local and test setups
> - Move to `apply -f file.yml` and `apply -f directory\` for production
> - Store yaml in _git_ ,`git commit` each change before you apply.
> - This trains you for later doing _GitOps_ - where git commits are automatically applied to clusters

</details>
	

### Moving To Declarative Kubernetes

<details>
<summary>
Using yaml files to run kubernetes.
</summary>

adopting the `kubectl apply` method, where all the changes are done to the yaml file, like how swarm uses stack deploy.

#### kubectl apply

<details>
<summary>
One command for files, directories and online resources.
</summary>

Kubernetes is un-opinionated. it has many ways to do the same things, like the three approaches from above.

Infrastructures as code, "__GitOps__", fully declarative kubernetes.
The command that we almost exclusively use is `kubectl apply -f <filename>.yml`
We will skip the middleman commands of `kubectl create`,`kubectl replace` and `kubectl edit`, which belong to the imperative objects approach.

quick examples
create/update resources in a file
`kubectl apply -f my_file.yaml`
create/update a whole directory of yaml files
`kubectl apply -f my-yaml/`
create/update from a URL 
`kubectl apply -f https://bret.run/pod/yml`
but be careful, it's really dangerous if we don't know what it is, so we should check if first
unix: `curl -L https://bret.run/pod`
windows power shell: `start https://bret.run/pod.yml`

</details>

#### Kubernetes Configuration YAML

<details>
<summary>
How the configuration file looks
</summary>

the file is more complex than a "docker-compose" format, it has more flexibility, and that comes with a learning curve.
we can also write this in json, but ths industry standard is yaml (which is then converted into json for the machine).

we can a file with many resources or one file per resource. the description of a single resource is called a _manifest_, this can be for a deployment, a job, a secret or something else.

each manifest needs four parts - RootKeys - key-value pairs at the root level.
> - apiVersion
> - kind
> - metadata
> - spec

lets look at the "k8s-yaml" folder, we start by looking at the "pod.yml" file.

this is the simplest thing to create, we can see all the four required root keys.
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.17.3
    ports:
    - containerPort: 80
```
same with the "deployment.yaml" file. the bulk of the data is in the 'spec', the apiVersion is related to the kind.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.17.3
        ports:
        - containerPort: 80
```

in the final file, "app.yaml", the two resources are combined, so we have two manifests in the same file.
each has the four mandatory root keys. the manifests are separated by three dashes `---`
```yaml
apiVersion: v1
kind: Service
metadata:
  name: app-nginx-service
spec:
  type: NodePort
  ports:
  - port: 80
  selector:
    app: app-nginx
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: app-nginx
  template:
    metadata:
      labels:
        app: app-nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.17.3
        ports:
        - containerPort: 80
```
there are dozens of resource types, and each has different spec, we can extend resource types with something called __CRDS__. but that's for later.
	
</details>

#### Building Your YAML Files

<details>
<summary>
How to find what possible keys there are for each resource.
</summary>

let's create a yaml from scratch. because kubernetes is always changing, many online resource are becoming outdated very quickly.

if we want a list of the __kind__, we can get it from a cluster. 3rd party tools can add more of those.

we get a list with many resources, but we care about the column __kind__, which is the name we use in the yaml file.
the APIGroup is related to the APIVersions, some resource belong to more than one api group. this usually means old and new api versions.
we can see all the api-versions with a different command
```sh
kubectl api-resources
kubectl api-versions
```
the metadata rootKey must have a name, this is the only required part.

all the real action is in the spec rootKey.

we can see all the keys that each kind support by using the `explain` command, we see the name and the type.
we can drill down with the dot notation to get a reference manual for some stuff. we can drill down even to get a more readable text.
this way we can see what's actually required, without relying on external documentation, which might be outdated.
there is no limit to how much we can drill down.
```sh
kubectl explain pods
kubectl explain services --recursive
kubectl explain services.spec 
kubectl explain services.spec.type
kubectl explain deployment.spec.template.spec.volumes.nfs.server
```
(Note: the version we get from the `explain` command might be incorrect. we should use the `api-versions` command to be sure)

[Kubernetes Documentations](https://kubernetes.io/docs/reference/#api-reference) can also help.

</details>

#### Dry Run and Diffs

<details>
<summary>
seeing if stuff would change, and what would change.
</summary>

in the past, dry-run was client only, but now there is a server option, this will compare what we have and what we send it, and tell us what would happen.

selectors are how an object finds the resources it talks to.

```sh
#traditional, client only, dry run check
kubectl apply -f app.yml --dry-run
```

let's see this in action, this time we will what would have happened if we had run the command for real
we only see if something would have changed  or not, if we wish to know what would have changed, we use the `diff` command.

```sh
kubectl apply -f app.yml #actually create

#modern way, talks to server
kubectl apply -f app.yml --server-dry-run
# check differences
kubectl diff -f app.yml

```
we can change the yaml file and run the `diff` command again and see the changes in diff format.
in the time of the video, these features are beta, but they are likely to change.

</details>

#### Labels and Label Selectors

<details>
<summary>
Label,Label Selectors and Annotations.
</summary>

label go under the metadata section in the yaml.
They are a list of key-value pair for identifying the resource later by selecting, grouping of filtering for it.
it can be anything that describes the objects, we can describe the environment, the tier, the name, the customer, the region.

we can use multiple labels, logical conditions, etc...
```sh
kubectl get pods -l app=nginx #filter
kubectl apply -f my_file.yaml -l app=nginx #only apply on resources with label
```

label selectors.
how do services know which pods to send traffic to? label selectors are "linked"  to labels.
in some versions we can get this validated for matchLabels and labels in the yaml file.

swarm has something similar, but it's done in the background and we can't really customize it directly.

selectors also use "taints and toleration" to control node placements, which are like an inverse

annotations are for more complex data, like configurations for stuff that talks back to kubernetes.

final cleanup
```sh
kubectl get all
kubectl delete <resource type>/<resource name>
```

</details>

</details>

</details>

[Main](README.md)