# Exposing Kubernetes Ports

## Service Types
Definition of a Service: 


> A service can be defined as an abstraction layer on top of a set of pods which provides a single IP address and DNS name by which pods can be accessed. With a kubeservice, it is very easy to manage load balancing configurations. It helps pods to scale very easily.



##### Create a Service  with kubectl expose
```
kubectl expose
```

## Creating a ClusterIP Service
The aim of this demo is to show how a Cluster IP Service is created. For the demo we require an image with tools such as curl installed on a linux instance (alpine), and a tool within the image that will display the environment variables created for that particular pod once prompted. We will open a command shell and enter the following command  `kubectl get pods -w` which will enable us to  watch the result of the actions being taken in a second command shell. 
###### Create a deployment 
```
kubectl create deployment httpenv --image=bretfisher/httpenv
```

###### Scale the the number of pods 
```
kubectl scale deployment/httpenv --replicas=5
```

###### Create a Service 
```
kubectl expose deployment/httpenv --port 8888
```
###### Obtain the IP of the cluster
```
kubectl get service
```

###### Run a single pod 
```
kubectl run --generator run-pod/v1 tmp-shell --rm -it --image bretfisher/netshoot -- bash
```


##### Query the environment variables with httpenv from within the Pod inside the cluster
```
curl httpenv:8888
```
```
kubectl get service
```
___
## Creating a NodePort and LoadBalancer Service
We shall first create a NodePort which will enable us to expose our service to external requests and test it using cURL. Then we shall do the same with DockerDesktop
##### Recap on what is currently running in our Kubernetes cluster
```
kubectl get all
```

##### Create a Service that is exposed externally on the host IP through a Nodeport
```
kubectl expose deployment/httpenv --port 8888 --name httpenv-np --type NodePort
```

##### Show services created and how the respective ports are defined
```
kubectl get services
```
##### Access the service from outside the Cluster
```
curl localhost:32334 (your defined port)
```
##### Create a LoadBalancer Service
```
kubectl expose deployment/httpenv --port 8888 --name httpenv-lb --type LoadBalancer
```
##### Show current running Services in the cluster
```
kubectl get services
```
##### DockerDesktop will with the built in Loadbalancer will allow acces to the service 
```
curl localhost:8888
```

##### Clean up environment
```
kubectl delete service/httpenv service/httpenv-np
```

```
kubectl delete service/httpenv-lb deployment/httpenv
```
___
## Kubernetes Services DNS

```
curl <hostname>
```
```
kubectl get namespaces
```
```
curl <hostname>.<namespace>.svc.cluster.local
```

