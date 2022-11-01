# Deployment 101

We looked at ReplicaSets earlier. However, ReplicaSet have one major drawback:
once you select the pods that are managed by a ReplicaSet, you cannot change their pod templates.

For example, if you are using a ReplicaSet to deploy four pods with NodeJS running and you want to change the NodeJS image to a newer version, you need to delete the ReplicaSet and recreate it. Restarting the pods causes downtime till the images are available and the pods are running again.

A Deployment resource uses a ReplicaSet to manage the pods. However, it handles updating them in a controlled way.
Let’s dig deeper into Deployment Controllers and patterns.

## Creating Your First Deployment

The following Deployment definition deploys four pods with nginx as their hosted application:

```bash
git clone https://github.com/clowa/kubernetes-training
cd kubernetes-training/12_Deployment_101
kubectl create -f nginx-dep.yaml
```

## Checking the list of application deployments

To list your deployments use the get deployments command:

```bash
$ kubectl get deployments
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3/3     3            3           6m26s
```

```bash
$ kubectl describe deploy
Name:                   nginx-deployment
Namespace:              cedric-ahlers
CreationTimestamp:      Tue, 01 Nov 2022 17:29:14 +0100
Labels:                 <none>
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=nginx
Replicas:               3 desired | 3 updated | 3 total | 3 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=nginx
  Containers:
   nginx:
    Image:        nginx:1.7.9
    Port:         80/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   nginx-deployment-84df99548d (3/3 replicas created)
Events:
  Type    Reason             Age    From                   Message
  ----    ------             ----   ----                   -------
  Normal  ScalingReplicaSet  6m53s  deployment-controller  Scaled up replica set nginx-deployment-84df99548d to 3
```

```bash
$ kubectl get deployments
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3/3     3            3           6m26s
```

```bash
$ kubectl get pods
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-84df99548d-87rkn   1/1     Running   0          8m36s
nginx-deployment-84df99548d-8rx9q   1/1     Running   0          8m36s
nginx-deployment-84df99548d-gc5d5   1/1     Running   0          8m36s
```

## Scale up/down application deployment

Now let’s scale the Deployment to 4 replicas. We are going to use the kubectl scale command,
followed by the deployment type, name and desired number of instances:

```bash
$ kubectl scale deployments/nginx-deployment --replicas=4
deployment.extensions/nginx-deployment scaled
```

The change was applied, and we have 4 instances of the application available. Next,
let’s check if the number of Pods changed:

Now There should be 4 pods running in the cluster

```bash
$ kubectl get deployments
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   4/4     4            4           4m

```

There are 4 Pods now, with different IP addresses. The change was registered in the Deployment events log. To check that, use the describe command:

```bash
$ kubectl describe deployments/nginx-deployment
Name:                   nginx-deployment
Namespace:              cedric-ahlers
CreationTimestamp:      Tue, 01 Nov 2022 17:29:14 +0100
Labels:                 <none>
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=nginx
Replicas:               4 desired | 4 updated | 4 total | 4 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=nginx
  Containers:
   nginx:
    Image:        nginx:1.7.9
    Port:         80/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   nginx-deployment-84df99548d (4/4 replicas created)
Events:
  Type    Reason             Age    From                   Message
  ----    ------             ----   ----                   -------
  Normal  ScalingReplicaSet  9m52s  deployment-controller  Scaled up replica set nginx-deployment-84df99548d to 3
  Normal  ScalingReplicaSet  33s    deployment-controller  Scaled up replica set nginx-deployment-84df99548d to 4
```

```bash
$ kubectl get pods -o wide
NAME                                READY   STATUS    RESTARTS   AGE   IP             NODE                              NOMINATED NODE   READINESS GATES
nginx-deployment-84df99548d-87rkn   1/1     Running   0          10m   10.224.0.28    aks-default-15179752-vmss000000   <none>           <none>
nginx-deployment-84df99548d-8rx9q   1/1     Running   0          10m   10.224.0.162   aks-general-70271185-vmss000000   <none>           <none>
nginx-deployment-84df99548d-gc5d5   1/1     Running   0          10m   10.224.0.173   aks-general-70271185-vmss000000   <none>           <none>
nginx-deployment-84df99548d-hk5j7   1/1     Running   0          59s   10.224.0.167   aks-general-70271185-vmss000000   <none>           <none>
```

You can also view in the output of this command that there are 4 replicas now.

# Scaling down the deployment to 2 Replicas

To scale down the deployment to 2 replicas, run again the scale command:

```bash
$ kubectl scale deployments/nginx-deployment --replicas=2
deployment.extensions/nginx-deployment scaled
$ kubectl get deployments
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   2/2     2            2           11m
```

## Perform a rolling updates of the deployment

So far, everything our Deployment did is no different than a typical ReplicaSet. The real power of a Deployment lies in its ability to update the pod templates without causing application outage.

Let’s say you want to update the nginx version used. The current pods are using an older nginx version. The following will show you how to update the pod template of the deployment to use the new image:

Frist of all let's take a last look at our current deployment and pods . We see the current image is `nginx:1.7.9`

```bash
$ kubectl get deployments
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   2/2     2            2           7m23s
$ kubectl describe pods
Name:               nginx-deployment-6dd86d77d-b4v7k
Namespace:          default
Priority:           0
PriorityClassName:  <none>
Node:               docker-desktop/192.168.65.3
Start Time:         Sat, 30 Nov 2019 20:04:34 +0530
Labels:             app=nginx
                    pod-template-hash=6dd86d77d
Annotations:        <none>
Status:             Running
IP:                 10.1.0.237
Controlled By:      ReplicaSet/nginx-deployment-6dd86d77d
Containers:
  nginx:
    Container ID:   docker://2c739cf9fe4dac53a4cc5c6097207da0c5edc2183f1f36f9f3e5c7057f85da43
    Image:          nginx:1.7.9
    Image ID:       docker-pullable://nginx@sha256:e3456c851a152494c3e4ff5fcc26f240206abac0c9d794affb40e0714846c451
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Sat, 30 Nov 2019 20:05:28 +0530
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-ds5tg (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  default-token-ds5tg:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-ds5tg
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason     Age    From                     Message
  ----    ------     ----   ----                     -------
  Normal  Scheduled  10m    default-scheduler        Successfully assigned default/nginx-deployment-6dd86d77d-b4v7k to docker-desktop
  Normal  Pulling    10m    kubelet, docker-desktop  Pulling image "nginx:1.7.9"
  Normal  Pulled     9m17s  kubelet, docker-desktop  Successfully pulled image "nginx:1.7.9"
  Normal  Created    9m17s  kubelet, docker-desktop  Created container nginx
  Normal  Started    9m17s  kubelet, docker-desktop  Started container nginx

  [...]
```

To update the image of the application to new version, use the set image command,
followed by the deployment name and the new image version:

```bash
$ kubectl set image  deployments/nginx-deployment nginx=nginx:1.9.1
deployment.extensions/nginx-deployment image updated
```

The command notified the Deployment to use a different image for your app and initiated a rolling update. Check the status of the new Pods, and view the old one terminating with the get pods command:

```bash
$ kubectl get pods -w
NAME                                READY   STATUS              RESTARTS   AGE
nginx-deployment-84df99548d-87rkn   1/1     Running             0          17m
nginx-deployment-84df99548d-8rx9q   1/1     Running             0          17m
nginx-deployment-8bf4959b6-gthdz    0/1     ContainerCreating   0          4s
nginx-deployment-8bf4959b6-gthdz    1/1     Running             0          13s
nginx-deployment-84df99548d-8rx9q   1/1     Terminating         0          17m
nginx-deployment-8bf4959b6-6bhbv    0/1     Pending             0          0s
nginx-deployment-8bf4959b6-6bhbv    0/1     ContainerCreating   0          0s
nginx-deployment-8bf4959b6-6bhbv    1/1     Running             0          4s
nginx-deployment-84df99548d-87rkn   1/1     Terminating         0          17m
```

Press `Ctrl` + `C` to cancel the command.

# Checking description of pod again

```bash
$ kubectl describe pods
Name:               nginx-deployment-6dd86d77d-b4v7k
Namespace:          default
Priority:           0
PriorityClassName:  <none>
Node:               docker-desktop/192.168.65.3
Start Time:         Sat, 30 Nov 2019 20:04:34 +0530
Labels:             app=nginx
                    pod-template-hash=6dd86d77d
Annotations:        <none>
Status:             Running
IP:                 10.1.0.237
Controlled By:      ReplicaSet/nginx-deployment-6dd86d77d
Containers:
  nginx:
    Container ID:   docker://2c739cf9fe4dac53a4cc5c6097207da0c5edc2183f1f36f9f3e5c7057f85da43
    Image:          nginx:1.7.9
    Image ID:       docker-pullable://nginx@sha256:e3456c851a152494c3e4ff5fcc26f240206abac0c9d794affb40e0714846c451
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Sat, 30 Nov 2019 20:05:28 +0530
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-ds5tg (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  default-token-ds5tg:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-ds5tg
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason     Age   From                     Message
  ----    ------     ----  ----                     -------
  Normal  Scheduled  16m   default-scheduler        Successfully assigned default/nginx-deployment-6dd86d77d-b4v7k to docker-desktop
  Normal  Pulling    16m   kubelet, docker-desktop  Pulling image "nginx:1.7.9"
  Normal  Pulled     15m   kubelet, docker-desktop  Successfully pulled image "nginx:1.7.9"
  Normal  Created    15m   kubelet, docker-desktop  Created container nginx
  Normal  Started    15m   kubelet, docker-desktop  Started container nginx

  [...]
```

## Rollback updates to application deployment

Because the deployment stores a revision history of the replicaSets we can simply role back to the pervious used replicaset. The rollout command reverted the deployment to the previous known state. Updates are versioned and you can revert to any previously know state of a Deployment. List again the Pods:

```bash
$ kubectl rollout undo deployments/nginx-deployment
deployment.extensions/nginx-deployment rolled back

$ kubectl rollout status deployments/nginx-deployment
deployment "nginx-deployment" successfully rolled out
```

After the rollout succeeds, you may want to get the Deployment.

The output shows the update progress until all the pods use the new container image.

The algorithm that Kubernetes Deployments use when deciding how to roll updates is to keep at least 25% of the pods running. Accordingly, it doesn’t kill old pods unless a sufficient number of new ones are up. In the same sense, it does not create new pods until enough pods are no longer running. Through this algorithm, the application is always available during updates.

You can use the following command to determine the update strategy that the Deployment is using:

```bash
Biradars-MacBook-Air-4:~ sangam$ kubectl describe deployments | grep Strategy
StrategyType:           RollingUpdate
RollingUpdateStrategy:  25% max unavailable, 25% max surge
```

## Cleanup

Finally you can clean up the resources you created in your cluster:

```bash
kubectl delete deployment nginx-deployment
```

# Contributors

[Sangam Biradar](https://twitter.com/BiradarSangam)
[Cedric Ahlers](https://github.com/clowa)

# Reviewers

[Ajeet Singh Raina](https://twitter.com/ajeetsraina)

[Next >>](https://collabnix.github.io/kubelabs/Scheduler101/index.html)
