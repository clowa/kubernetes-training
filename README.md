# Overview

This repository contains little hands on labs originally from [collabnix/kubelabs](https://collabnix.github.io/kubelabs/) with some customizations to fit my training. Big thanks to the guys of [collabnix](https://github.com/collabnix).

## Getting started

Create your own namespace if your share your cluster with others to avoid name conficts and unattended resource deletion

```bash
kubectl create ns <NAME>-<SURNAME>
```

Set your current context to your playground namespace created before

```bash
kubectl config set-context --current --namespace=<NAME>-<SURNAME>
```

You can now start with the hands on labs - just take a look at each folder with the suffix `_101`.
