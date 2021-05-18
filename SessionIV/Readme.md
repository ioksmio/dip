# Instanciate a  2 Tiered application with Kubernetes using YAML files

## In this excersize we shall instanciate a wordpress application with a backend mysql server. We shall also

Download the mysql-deployment YAML-File
```
curl -LO https://github.com/ioksmio/dip/raw/master/SessionIV/mysql-deployment.yaml
```

Download 
```
curl -LO https://github.com/ioksmio/dip/raw/master/SessionIV/wordpress-deployment.yaml
```
Download the wordpress-deployment YAML-File

There are different generators and these are specific to any particular resource (service, deployment, cron job etc). They evolve with the kubernetes as a whole ie they also have versions. They may start as a ß-version and mature to a full version V1.
___
##### Create a Deployment with minimum options required without deploying onto a kubernetes cluster, output to yaml and observe ressources created by respective template 
```
kubectl create deployment sample --image nginx --dry-run -o yaml
```
* [--dry-run] will not deploy to the kubernetes cluster instead it outputs the type of ressource it would create (similar to `kubectl get command`)
* [-o yaml] will output in yaml format
* the template in the background completed missing options with default values
* These will be used at Dev and Test levels
* This will be sent to the API-Server when we hit enter
```
apiVersion: apps/v1         <-- Version of this type of ressource
kind: Deployment            <-- kind is deployment because we created a deployment
metadata:
  creationTimestamp: null   <-- timestamp is part of the created automatically
  labels:                    
    app: sample             <-- sample - assigning an app as it was defined as deployment 
  name: sample
spec:                       <-- spec for the deployment   
  replicas: 1               <-- one replica
  selector:                 <-- how this app will be selected inside the deployment
    matchLabels:
      app: sample
  strategy: {}
  template:                 <-- is for the ReplicaSet
    metadata:
      creationTimestamp: null
      labels:
        app: sample
    spec:                   <-- the containers it will create will be found under spec
      containers:
      - image: nginx
        name: nginx
        resources: {}
status: {}
```
___
##### If required the following commands can also be used for further experimentation
```
kubectl create deployment sample --image nginx --dry-run -o yaml
kubectl create deployment test --image nginx --dry-run
kubectl create deployment test --image nginx --dry-run -o yaml
kubectl create job test --image nginx --dry-run -o yaml
```
@@ text in purple (and bold)@@
___
##### Output the template of a service in Yaml format
```
kubectl create deployment test --image nginx                 <--is required for this demo
kubectl expose deployment/test --port 80 --dry-run -o yaml
```
 *  This requires a deployment to be up and running otherwise you will get an error
 *  kind is Service
 *  The spec will define protocols and ports
 *  Serivce is actually nothing more than a combination of network routes and subnetting using either TCP or UDP
 *  Listening Port is 80
 *  Target Port is 80
  
```
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    app: test
  name: test
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: test
status:
  loadBalancer: {}
```
___
##### Create a Deployment with minimum options without deploying onto a kubernetes cluster, output to yaml and observe resources created by respective template 
```
kubectl create deployment sample --image nginx --dry-run -o yaml
```
* [--dry-run] will not deploy to the kubernetes cluster instead it outputs the type of ressource it would create (similar to `kubectl get command`)
* [-o yaml] will output in yaml format
* the template in the background completed missing options with default values
* These will be used at Dev and Test levels
* This will be sent to the API-Server when we hit enter
```
apiVersion: apps/v1         <-- Version of this type of ressource
kind: Deployment            <-- kind is deployment because we created a deployment
metadata:
  creationTimestamp: null   <-- timestamp is part of the created automatically
  labels:                    
    app: sample             <-- sample - assigning an app as it was defined as deployment 
  name: sample
spec:                       <-- spec for the deployment   
  replicas: 1               <-- one replica
  selector:                 <-- how this app will be selected inside the deployment
    matchLabels:
      app: sample
  strategy: {}
  template:                 <-- is for the ReplicaSet
    metadata:
      creationTimestamp: null
      labels:
        app: sample
    spec:                   <-- the containers it will create will be found under spec
      containers:
      - image: nginx
        name: nginx
        resources: {}
status: {}
```
___
##### Clean the deployment and prepare for the next lecture
```
kubectl delete deployment/test
```
 *  Deletes the created deployment




## The future of kubectl run

The run command will be depreciated in funcitonality. This has been an ongoing effort since version 1.12. Generally the use of the run command has changed over the years and the aim is to reduce the functionality. 
___
##### Show message that the certain generators will be depreciated in future
```
kubectl run job km --image nginx
```
 *  is supposed to run one specific pod with a container running the image nginx
 *  we get the message below that the generator has been depreciated and removed
 *  we are rather asked to use `kubectl run --generator=run-pod/v1` or `kubectl create` instead
 *  aim is to reduce complexity and have the run command only start a pod rather than start a deployment which is available in the create command
 *  used case is when one is on a machine that is using CRI-O or containerd as a runtime
 *  or as seen earlier in the demo, we need a runtime in a network for testing/troubleshooting purposes (one off tasks)
 *  Not recommended for production environments
 *  reason for depreciating was its complex behaviour due to activating different controllers with respective options
  

```diff
-kubectl run --generator=deployment/apps.v1 is DEPRECATED and will be removed in a future version. Use kubectl run --generator=run-pod/v1 or kubectl create instead.
```
___
##### Show different behaviours of the run command
##### Create a deployment via run
```
kubectl run test --image nginx --dry-run
```
* will create a deployment today, however in future it will create a pod and not a deployment
* [--generator] would specify a generator, however not used here
  
##### Create a service and a deployment with one command
```
kubectl run test --image nginx --port 80 --expose --dry-run
```

```diff
+service/test created (dry run)             <-- service created
+deploymeent.apps/test created (dry run)    <-- deployment created with one command
```
* This will be depreciated at one point in time
* deployment.apps/v1 is the generator used here
##### Create a job
```
kubectl run test --image nginx --restart OnFailure --dry-run
```

```diff
+job.batch/test created (dry run)             <-- job created
```
* This will be depreciated at one point in time
* Schema update on your database (used case)
* Generator used here was job/v1

##### Create a pod
```
kubectl run test --image nginx --schedule "* /1 * * * *" --dry-run
```

```diff
+kubectl run --generator=cronjob/v1beta1 is DEPRECATED and will be removed in a future version. Use kubectl run --generator=run-pod/v1or kubectl create instead.
cronjob.batch/test created (dry run)
```
* This will be depreciated at one point in time
* Generator cronjob/v1beta1 was used