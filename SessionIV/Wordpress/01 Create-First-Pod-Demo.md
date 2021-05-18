
# Create the first Pod



```
kubectl version
```

```
kubectl run my-nginx --image nginx
```

### List all created Pods 

```
kubectl get pods
```

### List most important information about ressources
```
kubectl get all
```

### Cleaning up by deleteing ressources
```
kubectl delete deployment my-nginx
```
### Verify what has been deleted
```
kubectl get all
```
___
# Scaling Replica Sets

#### Start a new Apache Webserver

```
kubectl run my-apache --image httpd
```

#### Show the objects that have been deployed
```
kubectl get all
```

#### Scale the Replica Sets

```
kubectl scale deploy/my-apache --replicas 2

```

 `kubectl scale deployment my-apache --replicas` 2 will do the same job


```
kubectl get all
```
___
# Inspecting Kubernetes Objects

#### Which pods are running
```
kubectl get pods
```
#### Show logs of a particular deployment
```
kubectl logs deployment/my-apache
```

#### Show the a fixed number of the last defined number of lines in a log continously
```
kubectl logs deployment/my-apache --follow --tail 1
```

#### Combine the logs from multiple pods by using a selector
```
kubectl logs -l run=my-apache
```

#### Get the name of the Pod whose details are needed
```
kubectl get pods
```
#### Obtain required details
```
kubectl describe pod/my-apache-<pod id>
```
# Deleted Pods are automatically replaced by new ones through instruction on higher levels

#### Monitor through the watch command
```
kubectl get pods -w
```
#### Delete a pod
```
kubectl delete pod/my-apache-<pod id>
```
#### Verify whether the pod has been deleted
```
kubectl get pods
```


#### Clean up
```
kubectl delete deployment my-apache
```
