# Overview

This example demonstrate the benefits of kubernetes operators and also covers helm deployments.

## Taks

First of all let's deploy the prometheus operator the classic way without helm.  
Take a look at the `./manifests` folder and see how overwhelming complex kubernetes applications can be.

The files itself contains the whole setup of prometheus monitoring stack and the prometheus-operator itself. Take a look at the files ending with `-crd`. This are the new kubernetes objects the operators uses to determine the target state of the components.

You can take a look at the existing `CRD` of your cluster by running the following

```bash
kubectl get crd
```

Create a new namespace for this operator since it contains a lot of resources

```bash
kubectl create ns monitoring
```

Deploy all of the manifest by running

```bash
kubectl create -f .
```

This will create a bunch of resources! Now let's take a look at the new created kubernetes objects by the `CRD`s.

Each of them has a new name you can find within the yaml manifest. For example `014-prometheus-crd.yaml` has the follwoing section ...

```yaml
[...]
spec:
  group: monitoring.coreos.com
  [...]
    kind: Prometheus # This is a name
    listKind: PrometheusList
    plural: prometheuses # This is a alias
    shortNames:
      - prom # This is a alias
[...]
```

... you use each of the names to get instances of the new resource type. For example:

```bash
kubectl get prom
kubectl get Prometheus
kubectlget prometheus
```

```bash
$ kubectl get prom -n monitoring
NAME                                    VERSION   DESIRED   READY   RECONCILED   AVAILABLE   AGE
k8s                                     2.39.1    1         0       True         True        3h9m
```

Now let's edit our prometheus cluster. Let' say we will downgrade our prometheus version by setting `version: v2.38.1` and also increase the replicas to 3.

```bash
kubectl edit prometheus k8s -n monitoring
```

Safe and close the file. You should immediately see some pod activity while the prometheus-operator do the needed changes to our existing environment.

Now let's access the prometheus UI by establishing a tunnel by running

```bash
kubectl port-forward prometheus-k8s 9090:9090
```

Are there any additional new resources we have created? Go and find out! There are new ones!

## Cleanup

Remove the namespace to delete all of its resources

```bash
kubectl delete ns monitoring
```

You also have to delete the `CustomResourceDefinition` you have created.

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
