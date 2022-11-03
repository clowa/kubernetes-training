# Overview

This example simulates the behaviour of the kubernets `HorizontalPodAutoscaler`.

Clone the reposioty to your local machine if you haven't did so before

```bash
git clone https://github.com/clowa/kubernetes-training
cd kubernetes-training/21_hpa_101
kubectl apply -f .
```

## Example

First of all start two other terminals to watch what goes on within kubernetes.

In the first terminal session take a look at the `HorizontalPodAutoscaler` by running

```bash
kubectl get hpa -w
```

in the second terminal session watch the pods

```bash
kubectl get pods -w
```

Now we can deploy the sample application by running

```bash
kubectl apply -f .
```

You should see some pods stating in one of you terminal sessions and the other one should show you the stats of the hpa. It should look somthing like this

```bash
$ kubectl get pods -w
NAME                           READY   STATUS    RESTARTS   AGE
hpa-example-5bccc48b8b-vzr69   0/1     Pending   0          0s
hpa-example-5bccc48b8b-vzr69   0/1     Pending   0          0s
hpa-example-5bccc48b8b-vzr69   0/1     ContainerCreating   0          0s
load-simulator-d9b54bb66-9sshg   0/1     Pending             0          0s
load-simulator-d9b54bb66-9sshg   0/1     Pending             0          0s
load-simulator-d9b54bb66-75lw9   0/1     Pending             0          0s
load-simulator-d9b54bb66-9sshg   0/1     ContainerCreating   0          0s
load-simulator-d9b54bb66-75lw9   0/1     Pending             0          0s
load-simulator-d9b54bb66-75lw9   0/1     ContainerCreating   0          0s
hpa-example-5bccc48b8b-vzr69     1/1     Running             0          2s
load-simulator-d9b54bb66-9sshg   1/1     Running             0          2s
load-simulator-d9b54bb66-75lw9   1/1     Running             0          3s
hpa-example-5bccc48b8b-j9hfq     0/1     Pending             0          0s
hpa-example-5bccc48b8b-j9hfq     0/1     Pending             0          0s
hpa-example-5bccc48b8b-dxn7p     0/1     Pending             0          0s
```

Here we see the pods comming up. We will also see the hpa spawning new pods to scale the web application.

```bash
$ kubectl get hpa -w
NAME          REFERENCE                TARGETS         MINPODS   MAXPODS   REPLICAS   AGE
hpa-example   Deployment/hpa-example   <unknown>/80%   1         20        0          0s
hpa-example   Deployment/hpa-example   <unknown>/80%   1         20        1          15s
hpa-example   Deployment/hpa-example   <unknown>/80%   1         20        1          60s
hpa-example   Deployment/hpa-example   1804%/80%       1         20        1          2m1s
```

The other terminal will show us the stats of the hpa - take a special look at the `TARGETS` and `REPLICAS` column. Be patient since it can take some minutes till metircs are available.

If you like to reduce or increase the load on your application you can simply do so by scaling our benchmark containers. For example you can stop the benchmark by running

```bash
kubectl scale --replicas=0 -f 20-load-simulator.yaml
```

After some minutes you can watch the hpa deleting some pods to scale in.

## Cleanup

Delete the lab resources with the following command

```bash
kubectl delete -f .
```
