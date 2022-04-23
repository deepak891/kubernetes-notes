# kubernetes-notes

In Kubernetes as operator we declare a desired state to the kubernetes cluster by creating objects using the kubernetes API.
Kubernetes continuously compares the desired state with the current state. If it detects differences, it takes action to ensure that the current state is the
same as the desired state.

One of the main purposes of a Kubernetes cluster is to deploy and run containers,
but also to support zero-downtime rolling upgrades using techniques such as green/blue and canary deployments.

To be able to monitor the health of running containers, Kubernetes assumes that containers implement a liveness probe. If a liveness probe reports an unhealthy container, Kubernetes will restart the container
Containers can be scaled in the cluster manually or automatically using a horizontal autoscaler

To optimize the use of the available hardware resources in a cluster, for example, memory and CPU, containers can be configured with quotas that specify the amount of resources a container needs

On the other hand, limits regarding how much a container is allowed to consume can be specified on the Pod or for a group of Pods on the namespace level

Kubernetes Service objects can be defined for service discovery
Another main purpose of Kubernetes is to provide service discovery of the running Pods and their containers, Service objects can be exposed to the outside of a Kubernetes cluster
Ingress object is, in many cases, better suited to handling externally incoming traffic to a group of services
To help Kubernetes find out whether a container is ready to accept incoming requests, a container can implement a readiness probe

Internally, a Kubernetes cluster provides one big flat IP network where each Pod gets its own IP address and can reach all the other Pods, independent of which node they run on

To allow multiple teams to work on the same Kubernetes cluster in a safe way, Role-Based Access Control
For example, administrators can be authorized to access resources on a cluster level, while the access of team members can be locked down to resources that are created in a namespace owned by the teams.



Kubernetes API objects
======================
Node: A node represents a server, virtual or physical, in the cluster.

Pod: A Pod represents the smallest possible deployable component in Kubernetes, consisting of one or more co-located containers. The containers share the same IP address and port range.

Deployment: A Deployment is used to deploy and upgrade Pods. The Deployment objects hand over the responsibility of creating and monitoring the Pods to a ReplicaSet. When performing a rolling upgrade of a Deployment, the role of the Deployment object is more involved.

ReplicaSet: A ReplicaSet is used to ensure that a specified number of Pods are running at all times. If a Pod is deleted, it will be replaced with a new Pod by the ReplicaSet.

Service: A Service is a stable network endpoint that you can use to connect to one or multiple Pods.
A Service is assigned an IP address and a DNS name in the internal network of the Kubernetes cluster.
The IP address of the Service will stay the same for the lifetime of the Service.
Requests that are sent to a Service will be forwarded to one of the available Pods using round robin-based load balancing.
By default, a Service is only exposed inside the cluster using a cluster IP address.
It is also possible to expose a Service outside the cluster, either on a dedicated port on each node in the cluster or – even better – through an external load balancer that is aware of Kubernetes; that is, it can automatically provision a public IP address and/or DNS name for the Service.
Cloud providers that offer Kubernetes as a service, in general, support this type of load balancer.

Ingress: Ingress can manage external access to Services in a Kubernetes cluster, typically using HTTP or HTTPS. For example, it can route traffic to the underlying Services based on URL paths or HTTP headers such as the hostname.

Namespace: A namespace is used to group and, on some levels, isolate resources in a Kubernetes cluster. The names of resources must be unique in their namespaces, but not between namespaces.

ConfigMap: A ConfigMap is used to store configuration that's used by containers. ConfigMaps can be mapped into a running container as environment variables or files.

Secret: This is used to store sensitive data used by containers, such as credentials. Secrets can be made available to containers in the same way as ConfigMaps.

DaemonSet: This ensures that one Pod is running on each node in a set of nodes in the cluster

Kubernetes runtime components
=============================
A Kubernetes cluster contains two types of nodes:
master nodes and worker nodes.
Master nodes manage the cluster, while the main purpose of worker nodes is to run the actual workload, for example, the containers we deploy in the cluster.

Components that run on master nodes, constituting the control plane:
1. api-server, the entry point to the control plane. This exposes a RESTful API, which, for example, the Kubernetes CLI tool known as kubectl uses.
2. etcd, a highly available and distributed key/value store, used as a database for all cluster data.
3. controller manager, which contains a number of controllers that continuously evaluate the desired state versus the current state for the objects defined in the etcd database.
4. scheduler, which is responsible for assigning newly created Pods to a node with available capacity, for example, in terms of memory and CPU.
5. Affinity rules can be used to control how Pods are assigned to nodes. For example, Pods that perform a lot of disk I/O operations can be assigned to a group of worker nodes that have fast SSD disks
Components that run on all the nodes, constituting the data plane
1. kubelet, a node agent that executes as a process directly in the nodes' operating system and not as a container
2. kube-proxy, a network proxy that enables the Service concept in Kubernetes and is capable of forwarding requests to the appropriate Pods
3. Container runtime, which is the software that runs the containers on a node. Historically, Kubernetes used Docker, but today any implementation of the Kubernetes Container Runtime Interface (CRI)
4. Kubernetes DNS, which is a DNS server that's used in the cluster's internal network. Services and Pods are assigned a DNS name, and Pods are configured to use this DNS server to resolve the internal DNS names. The DNS server is deployed as a Deployment object and a Service object.

Sequence of Events
1. An operator uses kubectl to send in a new desired state to Kubernetes, containing manifests declaring a new Deployment, Service, and Ingress object.
   The Ingress defines a route to the Service object and the Service object is defined to select Pods that are configured by the Deployment object.
2. kubectl talks to the api-server and it stores the new desired state as objects in the etcd database.
3. Various controllers will react on the creation of the new objects and take the following actions:
   a) For the Deployment object
   1. New ReplicaSet and Pod objects will be registered in the api-server.
   2. The scheduler will see the new Pod(s) and schedule them to appropriate worker nodes.
   3. On each worker node, the kubelet agent will launch containers as described by the Pods. The kubelet will use the container runtime on the worker node to manage the containers.
   b) For the Service object
   1. A DNS name will be registered in the internal DNS server for the Service object and the kube-proxies will be able to route requests that use the DNS name to one of the available Pods.
   c) For the Ingress object
   An Ingress controller will set up routes according to the Ingress object and be ready to accept requests from outside of the Kubernetes cluster. External requests that match the routes defined by the Ingress object will be forwarded by the Ingress controller to the Service object

Minikube profiles
Run multiple Kubernetes clusters locally
minikube profile my-profile
minikube config get profile

Kubernetes CLI (kubectl)
To manage cluster we will need only kubectl
To manage API objects we need kubectl apply command (it is declarative command)
kubectl delete (imperative command)
kubectl create namespace

Command to retrieving information about a Kubernetes cluster
kubectl get
kubectl described
kubectl logs

To get help
kubectl help (ex kubectl <command> --help
kubectl explain (ex kubectl explain deployment.spec.template.spec.containers)

Kubectl context
It helps to work with more than one kubernetes cluster.
So the context is combination of
- A Kubernetes cluster
- Authentication information for a user
- A default namespace

By default context are saved in ~/.kube/config files
But the file can be changed using unset KUBECONFIG

To check list of available context
kubectl config get-contexts

To switch context
kubectl config use-context my-cluster

To update context
kubectl config set-context (e.g kubectl config set-context $(kubectl config current-context) --namespace my-namespace)

Creating a kubernetes cluster
Run the following commands
1. unset KUBECONFIG
   macOs
2. minikube start \
   --profile=handson-spring-boot-cloud \
   --memory=10240 \
   --cpus=4 \
   --disk-size=30g \
   --kubernetes-version=v1.20.5 \
   --driver=hyperkit
   WSL 2
2. minikube start \
   --profile=handson-spring-boot-cloud \
   --memory=10240 \
   --cpus=4 \
   --disk-size=30g \
   --kubernetes-version=v1.20.5 \
   --driver=docker \
   --ports=8080:80 --ports=8443:443 \
   --ports=30080:30080 --ports=30443:30443

3. minikube profile handson-spring-boot-cloud

4. minikube addons enable ingress
5. minikube addons enable metrics-server

Unset the KUBECONFIG environment variable to ensure that the kubectl context is created in the default config file, ~/.kube/config.
Create the cluster using the minikube start command, where we can also specify what version of Kubernetes to use and the amount of hardware resources we want to allocate to the cluster

To communicate with cluster try
kubectl get nodes

To monitor its progress
kubectl get pods --namespace=kube-system

Note :  Pods created by Job objects, used to execute a container a fixed number of times like a batch job (status Completed)


Trying out a sample deployment
==============================
1. Deploy a simple web server based on NGINX in our Kubernetes cluster
2. Apply some changes to the deployment:
   -Change the current state by deleting the Pod and verify that the ReplicaSet creates a new one
   -Change the desired state by scaling the web server to three Pods and verify that the ReplicaSet fills the gap by starting up two new Pods
3. Route external traffic to the web server using a Service with a node port
   ============
   a) First create namespace & update kubectl context to use this
   kubectl create namespace first-attempts
   kubectl config set-context $(kubectl config current-context) --namespace=first-attempts

b) create deployment of NGINX in the namespace kubernetes/first-attempts/nginx-deployment.yaml
--
apiVersion: apps/v1
kind: Deployment  		#we are declaring a Deployment object
metadata:				#describe the Deployment object
name: nginx-deploy
spec:					#defines our desired state
replicas: 1			#we want to have one Pod up and running.
selector:				#how the Deployment will find the Pods it manages.
matchLabels:
app: nginx-app
template:				#specify how Pods will be created
metadata:			#which is used to identify the Pods
labels:
app: nginx-app
spec:				#specifies details for the creation of the single container in the Pod
containers:
- name: nginx-container
image: nginx:latest
ports:
- containerPort: 80
--
create deployment with following command
kubectl apply -f kubernetes/first-attempts/nginx-deployment.yaml


kubectl get all

Now, we will change the current state by deleting the Pod.
kubectl get pod --watch
kubectl delete pod --selector app=nginx-app

Change the desired state by setting the number of desired Pods to three replicas in the kubernetes/first-attempts/nginx-deployment.yaml deployment file

kubectl get all

To enable external communication with the web servers, create a Service using the kubernetes/first-attempts/nginx-service.yaml

---
apiVersion: v1				#pecify that we are declaring a Service object.
kind: Service				#describe the Service objec
metadata:
name: nginx-service		#which defines the desired state of the Service object
spec:
type: NodePort			#external caller can reach the Pods behind this Service using this port on any of the nodes in the cluster, independent of which nodes the Pods actually run on.
selector:
app: nginx-app			#used by the Service to find available Pods
ports:
- targetPort: 80		#port in the Pod where the requests will be forwarded to
port: 80				#Service will be accessible on, that is, internally in the cluster
nodePort: 30080		#port the Service will be externally accessible on using any of the nodes in the cluster
---
Run commnad
kubectl apply -f kubernetes/first-attempts/nginx-service.yaml

c) Verifty internal cluster IP address and port
kubectl run -i --rm --restart=Never curl-client --image=curlimages/curl --command -- curl -s 'http://nginx-service:80'



Wrap this up by removing the namespace containing the nginx deployment:
kubectl delete namespace first-attempts

Managing a local kubernetes cluster
stop command can be used to hibernate a kubernetes cluster. The start command to resume and delete to permanently remove

minikube stop
minikube start  	# resume and updated to use this cluster with the currently used namespace set to default
If you are working with another namespace
kubectl config set-context $(kubectl config current-context) --namespace=hands-on

minikube delete --profile handson-spring-boot-cloud  

