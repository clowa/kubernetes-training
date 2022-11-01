# StatefulSets

## What is Statefulset and how is it different from Deployment?

A Statefulset is a Kubernetes controller that is used to manage and maintain one or more Pods. However, so do other controllers like ReplicaSets and, the more robust, Deployments. So what does Kubernetes use StatefulSets for? To answer this question, we need to discuss stateless versus stateful applications.

A stateless application is one that does not care which network it is using, and it does not need permanent storage. Examples of stateless apps may include web servers (Apache, Nginx, or Tomcat).

On the other hand, we have stateful apps. Let’s say you have a Solr database cluster that is managed by several Zookeeper instances. For such an application to function correctly, each Solr instance must be aware of the Zookeeper instances that are controlling it. Similarly, the Zookeeper instances themselves establish connections between each other to elect a master node. Due to such a design, Solr clusters are an example of stateful applications. If you deploy Zookeeper on Kubernetes, you’ll need to ensure that pods can reach each other through a unique identity that does not change (hostnames, IPs...etc.). Other examples of stateful applications include MySQL clusters, Redis, Kafka, MongoDB, and others.

Given this difference, Deployment is more suited to work with stateless applications. As far as a Deployment is concerned, Pods are interchangeable. While a StatefulSet keeps a unique identity for each Pod it manages. It uses the same identity whenever it needs to reschedule those Pods.

## Storage Class

Storage classes are Kubernetes objects that let the users specify which type of storage they need from the cloud provider. Different storage classes represent various service quality, such as disk latency and throughput, and are selected depending on the scenario they are used for and the cloud provider’s support. Persistent Volumes and Persistent Volume Claims use Storage Classes.

## Persistent Volumes and Persistent Volume Claims

Persistent volumes act as an abstraction layer to save the user from going into the details of how storage is managed and provisioned by each cloud provider (in this example, we are using Google GCE). By definition, StatefulSets are the most frequent users of Persistent Volumes since they need permanent storage for their pods.

A Persistent Volume Claim is a request to use a Persistent Volume. If we are to use the Pods and Nodes analogy, then consider Persistent Volumes as the “nodes” and Persistent Volume Claims as the “pods” that use the node resources. The resources we are talking about here are storage properties, such as storage size, latency, throughput, etc.

Under this tutorial, we will see example of Azure Disks.

# Example

This example will deploy a mysql cluster running one primay node and two read nodes

The following resources will be created

- A ConfigMap with the MySQL cluster configuration
- A Service to read from all nodes
- A Headless Service used by MySQL to lookup the cluster members via DNS
- A StatefulSet running the actual MySQL nodes

First of all deploy the statefulSet. It will take some time till the MySQL cluster will be available, we will use this time to inspect what is going on.

```bash
kubectl apply -f mysql.yaml
```

Now let's check the pods

```bash
$ kubectl get pods
NAME      READY   STATUS     RESTARTS   AGE
mysql-0   0/2     Init:0/2   0          5s
```

Even though our statefulSet shuld have three pods (one primary and two read-only nodes) there is just one pod running. But that's okay, a statefulSet will create one pod after the other.

Because our application is stateful there should be some volumes storing the state - lets take a look

Applications running in kubernetes request stroage via a persistant volume claim, since we have one pod in the creating state we also have some persistant volume claims.

```bash
$ kubectl get pvc
NAME           STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
data-mysql-0   Bound    pvc-fcf74b42-1672-4de0-8e37-de70c708eabb   10Gi       RWO            default        3s
```

This claim is already bound to a persistant volume - lets check the persistant volumes

```bash
$ kubectl get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                        STORAGECLASS   REASON   AGE
pvc-fcf74b42-1672-4de0-8e37-de70c708eabb   10Gi       RWO            Delete           Bound    cedric-ahlers/data-mysql-0   default                 12s
```

Great! There is our volume. Let's check if a new pod is coming up to check his persistant volume and persistant volume claims. Repeat this section for each of the three pods. This maybe takes up to 10 minutes depending on the performance of the storage provider.

After all pods are up and running lets check the logs if our cluster is healthy.

On the primary node the logs should contain the following line.

```bash
$ kubectl logs mysql-0
[...]
2022-11-01T19:21:38.778444Z 0 [Note] mysqld: ready for connections.
[...]
```

So our primary node is ready for use. Now check if our read-only nodes/pods are aware of the MySQL cluster. On the second and third pod the log should contain the following lines.

Logs of the second MySQL node/pod

```bash
$ kubectl logs mysql-1
[...]
2022-11-01T19:25:56.004673Z 5 [Note] Slave SQL thread for channel '' initialized, starting replication in log 'mysql-0-bin.000003' at position 154, relay log './mysql-1-relay-bin.000001' position: 4
2022-11-01T19:25:56.014168Z 4 [Note] Slave I/O thread for channel '': connected to master 'root@mysql-0.mysql:3306',replication started in log 'mysql-0-bin.000003' at position 154
2022-11-01T19:25:56.014168Z 0 [Note] mysqld: ready for connections.
[...]
```

Logs of the third MySQL node/pod

```bash
$ kubectl logs mysql-2
[...]
2022-11-01T19:30:03.305078Z 5 [Note] Slave SQL thread for channel '' initialized, starting replication in log 'mysql-0-bin.000003' at position 154, relay log './mysql-2-relay-bin.000001' position: 4
2022-11-01T19:30:03.309760Z 4 [Note] Slave I/O thread for channel '': connected to master 'root@mysql-0.mysql:3306',replication started in log 'mysql-0-bin.000003' at position 154
2022-11-01T19:30:03.309760Z 0 [Note] mysqld: ready for connections.
[...]
```

Everything looks good. If you like you can play around with the MySQL cluster and establish a connection by running:

```bash
kubectl run mysql-client --image=mysql:5.7 -it --rm --restart=Never -- mysql -h mysql-0.mysql
```

## Cleanup

Now lets delete our MySQL Cluster. First of all delete all depleyed resources by running:

```bash
$ kubectl delete -f mysql.yaml
configmap "mysql" deleted
service "mysql" deleted
service "mysql-read" deleted
statefulset.apps "mysql" deleted
```

This will delete the configMap, both services and the statefulSet. But what's about the persistant volume claims? We didn't create them with our deployment manifest. Kubernetes has create them for us, so lets check this.

```bash
$ kubectl get pvc
NAME           STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
data-mysql-0   Bound    pvc-fcf74b42-1672-4de0-8e37-de70c708eabb   10Gi       RWO            default        19m
data-mysql-1   Bound    pvc-4ba179eb-e065-40ea-a8be-f5dfc1f1ec09   10Gi       RWO            default        18m
data-mysql-2   Bound    pvc-4e8dece4-b502-4dbf-9aa1-800a297b354f   10Gi       RWO            default        14m
```

They are still there and that's for a good reason. Remember statefulSets are for stateful workloads, so workloads that contains data. Therefore kubernetes preserve the persistant volumes and persistant volume claims to perserve our data. If we want to delete them we can simply delete the persistant volume claims and kubernetes will take care of the rest.

```bash
$ kubectl delete pvc data-mysql-0
persistentvolumeclaim "data-mysql-0" deleted
$ kubectl delete pvc data-mysql-1
persistentvolumeclaim "data-mysql-1" deleted
$ kubectl delete pvc data-mysql-2
persistentvolumeclaim "data-mysql-2" deleted
```

After deleting the persistant volume claims the storage provisioner will also go ahead and delte the persistant volumes since there were created with the reclaim policy `Delete`.
