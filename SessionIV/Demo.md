Demo04.md

Instanciate a Wordpress application using Kubernetes with Yaml Files 
---
Deploying WordPress and MySQL with Persistent Volumes



## Objectives

* Create PersistentVolumeClaims and PersistentVolumes
* Create a `kustomization.yaml` with
  * a Secret generator
  * MySQL resource configs
  * WordPress resource configs
* Apply the kustomization directory by `kubectl apply -k ./`
* Clean up



## Prerequisites


Download the following configuration files:

> 1. [mysql-deployment.yaml] 
>  `curl -LO https://github.com/ioksmio/dip/raw/master/SessionIV/mysql-deployment.yaml`
>
> 2. [wordpress-deployment.yaml] 
> `curl -LO https://github.com/ioksmio/dip/raw/master/SessionIV/wordpress-deployment.yaml`
> 3. [kustomization.yaml]
> `curl -LO https://github.com/ioksmio/dip/blob/master/SessionIV/kustomization.yaml`



## Apply and Verify
The `kustomization.yaml` contains all the resources for deploying a WordPress site and a 
MySQL database. You can apply the directory by
```shell
kubectl apply -k ./
```

Now you can verify that all objects exist.

1. Verify that the Secret exists by running the following command:

      ```shell
      kubectl get secrets
      ```

      The response should be like this:

      ```shell
      NAME                    TYPE                                  DATA   AGE
      mysql-pass-c57bb4t7mf   Opaque                                1      9s
      ```

2. Verify that a PersistentVolume got dynamically provisioned.
 
      ```shell
      kubectl get pvc
      ```
      
     Note:
      It can take up to a few minutes for the PVs to be provisioned and bound.
    

      The response should be like this:

      ```shell
      NAME             STATUS    VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS       AGE
      mysql-pv-claim   Bound     pvc-8cbd7b2e-4044-11e9-b2bb-42010a800002   20Gi       RWO            standard           77s
      wp-pv-claim      Bound     pvc-8cd0df54-4044-11e9-b2bb-42010a800002   20Gi       RWO            standard           77s
      ```

3. Verify that the Pod is running by running the following command:

      ```shell
      kubectl get pods
      ```

    Note:
      It can take up to a few minutes for the Pod's Status to be `RUNNING`.


      The response should be like this:

      ```
      NAME                               READY     STATUS    RESTARTS   AGE
      wordpress-mysql-1894417608-x5dzt   1/1       Running   0          40s
      ```

4. Verify that the Service is running by running the following command:

      ```shell
      kubectl get services wordpress
      ```

      The response should be like this:

      ```
      NAME        TYPE            CLUSTER-IP   EXTERNAL-IP   PORT(S)        AGE
      wordpress   LoadBalancer    10.0.0.89    localhost     80:32406/TCP   4m
      ```

 
      


5. Copy the IP address, and load the page in your browser to view your site.

   You should see the WordPress set up page similar to the following screenshot.

   ![wordpress-init](https://raw.githubusercontent.com/kubernetes/examples/master/mysql-wordpress-pd/WordPress.png)



## Cleanup


1. Run the following command to delete your Secret, Deployments, Services and PersistentVolumeClaims:

      ```shell
      kubectl delete -k ./
      ```
