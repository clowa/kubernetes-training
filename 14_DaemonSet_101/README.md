# Daemonsets

A daemonset will run exactly one pod on each node of the cluster. This is usefully to deploy a monitoring agent to existing nodes and every new node of the cluster.

## Example

This example will deploy the `node-exporter` exporter to every node of the cluster. The container will expose metrics of the node to pormetheus if it's also running in the cluster.

First of all take a look to the nodes of your cluster

```bash
kubectl get nodes
```

To realy follow along with this demo your cluster should run more than two nodes.

Now you can deploy the daemonset by running

```bash
kubectl apply -f daemonset.yml
```

To veryfy every pod is running on a different node you can run

```bash
$ kubectl get pods -o wide
NAME                         READY   STATUS        RESTARTS   AGE   IP             NODE                              NOMINATED NODE   READINESS GATES
prometheus-daemonset-8xh84   1/1     Running       0          76s   10.224.0.174   aks-general-70271185-vmss000000   <none>           <none>
prometheus-daemonset-9pss9   1/1     Running       0          76s   10.224.0.81    aks-default-15179752-vmss000006   <none>           <none>
prometheus-daemonset-d6866   1/1     Running       0          76s   10.224.0.18    aks-default-15179752-vmss000000   <none>           <none>
prometheus-daemonset-nh6qp   1/1     Terminating   0          76s   10.224.0.36    aks-default-15179752-vmss000005   <none>           <none>
```

Now take a look at the `NODE` column and you will see: each pod is running on a different node.
