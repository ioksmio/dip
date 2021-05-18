# Exposing Kubernetes Ports

## Service Types
Definition of a Service: 


> A service can be defined as an abstraction layer on top of a set of pods which provides a single IP address and DNS name by which pods can be accessed. With a kubeservice, it is very easy to manage load balancing configurations. It helps pods to scale very easily.



##### Create a Service  with kubectl expose
```
kubectl expose
```
* Creates a service with an IP Address
* Allows for communication between pods and external services
* Will resolve service requests from outside the cluster
* The created service will act as a load balancer
* The types of Services that can be created are:
    * Cluster IP
    * Load Balancer
    * Node Port
    * External Name
* Cluster IP (Default)
    * Internal virtual IP - The pods are reachable within the cluster
    * Default defined Port of the cluster will be used
* NodePort
    * Exposes a static port of the Service on each node's IP
    * High port allocated on each node
    * Always available whether on local install or in the cloud
* Load Balancer
    * The service LoadBalancer controls traffic inbound into the Kubernetes cluster and works with external load balancers in the cloud such as AWS Elastic Load Balancer  
    *  when instigated creates a Node Port and Cluster IP and then communicates with external LBs (AWS...)
* External Name 
    * Can be used for migrations (external services to internal services)
    * The service is maped to the contents of the external Name Field (foobar.com)
___
##### Enable the DNS Server Add-On provided by microk8s 

microk8s.enable dns
___
## Creating a ClusterIP Service
The aim of this demo is to show how a Cluster IP Service is created. For the demo we require an image with tools such as curl installed on a linux instance (alpine), and a tool within the image that will display the environment variables created for that particular pod once prompted. We will open a command shell and enter the following command  `kubectl get pods -w` which will enable us to  watch the result of the actions being taken in a second command shell. 
###### Create a deployment 
```
kubectl create deployment httpenv --image=bretfisher/httpenv
```
* The image with the name bretfisher/httpenv will be deployed
* 
###### Scale the the number of pods 
```
kubectl scale deployment/httpenv --replicas=5
```
* will deploy up five pods in total

###### Create a Service 
```
kubectl expose deployment/httpenv --port 8888
```
* Creates an abstraction layer above the pods
* The service will listen on port 8888
* Cluster IP will be created
###### Obtain the IP of the cluster
```
kubectl get service
```
* Shows all services running
* By default the Kubernetes Service is depicted
* The cluster obtains an IP that is only accessible from within the cluster ie this IP cannot be accessed from outside the cluster

###### Run a single pod 
```
kubectl run --generator run-pod/v1 tmp-shell --rm -it --image bretfisher/netshoot -- bash
```
* Pod template is used because of just running one pod without deployment or replicaset
* use a template to generate the pod
* The name of the pod is tmp-shell
* [--rm] will remove the pod once done
* [-it] will open a shell into the pod
* [--image] will define the image to use (we will use an image that has by default a few networking applications one of whihch is curl - when run will dump all environment variables into the shell)
* [" -- "] That tells the run command to stop looking at options and that anything after the double dashes is the command to be run
* [bash] starts the bash shell
* The assigned Service name also becomes the DNS name


##### Query the environment variables with httpenv from within the Pod inside the cluster
```
curl httpenv:8888
```
* Displays the environment variables
* Pod name displayed is equal to hostname
* Will display other Kubernetes defaults out of the box
##### From a linux host the IP Addresses are resolvable 
```
curl [ip of service]:8888 (on a linux machine)
```
* On a linux host nodes and containers can talk to each other out of the box
* Thus Kubernetes on a local linux machine has access to all private IP addresses
* This will not be possible from DockerDesktop
```
kubectl get service
```
* The IP Address for the cluster IP is not  resolvable from the linux host.
* DNS Name is not known to local linux machine. However, address is known and accessible
* cURLing the IP address :8888 is possible, just as from any another machine the local linux network.
*  From Docker Desktop this is not possible because the commands run are outside the Kubernetes cluster. DockerDesktop on a Mac or Windows machine will run in separate Linux VM
___
## Creating a NodePort and LoadBalancer Service
We shall first create a NodePort which will enable us to expose our service to external requests and test it using cURL. Then we shall do the same with DockerDesktop
##### Recap on what is currently running in our Kubernetes cluster
```
kubectl get all
```
will show us:
> * One Deployment
> * A ReplicaSet of five 
> * Pods with different names running
> * A default Kubernetes service (always present)
> * The httpenv Cluster IP Service 
> * An IP Address accessible from outside of the cluster and also adressable via its service name (httpenv)

##### Create a Service that is exposed externally on the host IP through a Nodeport
```
kubectl expose deployment/httpenv --port 8888 --name httpenv-np --type NodePort
```
* [--name httpenv-np] Name of Service (no two same names are allowed to exist at a time)
* [--type Nodeport] denotes the service type - Cluster IP is default

##### Show services created and how the respective ports are defined
```
kubectl get services
```
* Depicts all services defined
* 8888 depicts the  the container-listening port in the cluster
* 32334 depicts the host-port exposed to the outside world which is chosen from a default range of high ports
* Services are additive ie when a Nodeport is created automatically a ClusterIP will be created with it. 
* Port ranges are customizable using YAML
##### Access the service from outside the Cluster
```
curl localhost:32334 (your defined port)
```
* Localhost:32334 will redirect the traffic to the listening port of the container because of the created Nodeport similar to a direct cURL to the ClusterIP 
* On DockerDesktop there is a VPN-kit that allows for the Windows/Mac host to connect to the cluster
* Jumping into the Pods is not neccessary
* The Nodeport is randomly unless defined via YAML
##### Create a LoadBalancer Service
```
kubectl expose deployment/httpenv --port 8888 --name httpenv-lb --type LoadBalancer
```
* Requires a Plugin to be installed so that Kubernetes can talk to respective Load Balancers in the cloud
* In case of DockerDesktop, the command lets the the cluster know that port 8888 is part of th Deployment
> * The loadbalancer service will publish it on port 8888
* LoadBalancers can be seen as separate APIs thus coming with further capabilities
##### Show current running Services in the cluster
```
kubectl get services
```
* Three services will be depicted for the current deployment (2 on linux/microk8s/ minikube)
* DockerDesktop has an inbuilt Loadbalancer Service thus it will depcit 3 services
  
##### DockerDesktop will with the built in Loadbalancer will allow acces to the service 
```
curl localhost:8888
```
* The service should be accessible 
* Built In Nodeport has been created with the Loadbalancer Service
* Traffic flow is External Traffic-> LoadBalancer -> NodePort -> ClusterIP

##### Clean up environment
```
kubectl delete service/httpenv service/httpenv-np
```

```
kubectl delete service/httpenv-lb deployment/httpenv
```
* Multiple objects can be cleaned up at a go by using one command
* Objects need not be related

___
## Kubernetes Services DNS
* DNS is an optional service
* Ping or cURL a service name gives a positive response
* From Version 1.11 CoreDNS is the default standard
* Creating a Service will create a FQDN with hostname as part of the service name
* Namespaces allow the same application to be deployed in different contexts thus not conflicting even if they have the same name




```
curl <hostname>
```
* Will only obtain names in the same Namespace
```
kubectl get namespaces
```
* On DockerDesktop the docker is there to help do the "magic" (background services)
* Any created service will be placed into the default namespace
* Other depicted services come with Kubernetes cluster
> * kube-system the namespace created for objects created by the Kubernetes system
> * kube-node-lease improves the performance of the node as the cluster scales
> * kube-public mostly reserved for cluster usage where public and visible ressources are visible to any entity
```
curl <hostname>.<namespace>.svc.cluster.local
```
Fully Qualified Domain Name
> * hostname = name of host
> * namespace = organisational parameter for segmentation (used in scaling scenarios)
> * svc = service
> * .local = default DNS name given to cluster when created 
> * DNS name is only known within a cluster and can be accessed via the API