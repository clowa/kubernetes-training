# Overview

Deploy the prometheus operator via helm

## Task

Add the helm repository to your local machine

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

Create a new namespace for your first helm deployment

```bash
kubectl create ns monitoring
```

Install

```bash
$ helm install prometheus prometheus-community/kube-prometheus-stack -n monitoring --values ./values.yaml
NAME: prometheus
LAST DEPLOYED: Thu Nov  3 20:11:00 2022
NAMESPACE: monitoring
STATUS: deployed
REVISION: 1
NOTES:
kube-prometheus-stack has been installed. Check its status by running:
  kubectl --namespace monitoring get pods -l "release=prometheus"

Visit https://github.com/prometheus-operator/kube-prometheus for instructions on how to create & configure Alertmanager and Prometheus instances using the Operator.
```

Take a look at your deployed resources

```bash
kubectl get all -n monitoring
```

Take a special look at the api extensions also called `CRD` you have deployed

```bash
$ kubectl get crds
NAME                                                         CREATED AT
alertmanagerconfigs.monitoring.coreos.com                    2022-11-03T19:18:55Z
alertmanagers.monitoring.coreos.com                          2022-11-03T19:18:56Z
podmonitors.monitoring.coreos.com                            2022-11-03T19:18:56Z
probes.monitoring.coreos.com                                 2022-11-03T19:18:56Z
prometheuses.monitoring.coreos.com                           2022-11-03T19:18:57Z
prometheusrules.monitoring.coreos.com                        2022-11-03T19:18:57Z
servicemonitors.monitoring.coreos.com                        2022-11-03T19:18:57Z
thanosrulers.monitoring.coreos.com                           2022-11-03T19:18:58Z
[...]
```

We take a look at the `prometheus` object. That's what instructs the operator to deploy a prometheus cluster.

```bash
$ kubectl get prometheus -n monitoring
NAME                                    VERSION   DESIRED   READY   RECONCILED   AVAILABLE   AGE
prometheus-kube-promethe-prometheus   v2.39.1   1         1       True         True        4m55s
```

Let's edit the replica count and the verion of the prometheus cluster and see what will happen. Therefore open a second terminal session and watch the running pods

```bash
kubectl get pods -w -n monitoring
```

Now let's edit our prometheus cluster. Let' say we will downgrade our prometheus version by setting `version: v2.38.1` and also increase the replicas to 3.

```bash
kubectl edit prometheus prometheus-kube-promethe-prometheus -n monitoring
```

Safe and close the file. You should immediately see some pod activity while the prometheus-operator do the needed changes to our existing environment.

Now let's access the prometheus UI by establishing a tunnel by running

```bash
kubectl port-forward prometheus-prometheus-prometheus-oper-prometheus-0 9090:9090
```

## Cleanup

Uninstall the helm deployment

```bash
helm uninstall prometheus -n monitoring
```

You also have to delete the `CustomResourceDefinition` the helm chart has created.

```bash
kubectl delete crd alertmanagerconfigs.monitoring.coreos.com
kubectl delete crd alertmanagers.monitoring.coreos.com
kubectl delete crd podmonitors.monitoring.coreos.com
kubectl delete crd probes.monitoring.coreos.com
kubectl delete crd prometheuses.monitoring.coreos.com
kubectl delete crd prometheusrules.monitoring.coreos.com
kubectl delete crd servicemonitors.monitoring.coreos.com
kubectl delete crd thanosrulers.monitoring.coreos.com
```
