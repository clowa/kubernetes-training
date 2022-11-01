# Steps

```bash
git clone https://github.com/clowa/kubernetes-training
cd kubernetes-training/10_Pods_101
kubectl apply -f pods01.yaml
```

## Viewing Your Pods

```bash
kubectl get pods
```

## First visit of the website

```bash
kubectl proxy
```

Open [http://localhost:8001/api/v1/namespaces/default/pods/http:webserver:80/proxy/](http://localhost:8001/api/v1/namespaces/default/pods/http:webserver:80/proxy/) in your Browser

## Get logs of Pod

```bash
$ kubectl logs webserver

/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Configuration complete; ready for start up

```

You shoud see the website request you made before. Something like this:

```txt
10.224.0.12 - - [01/Nov/2022:14:49:52 +0000] "GET / HTTP/1.1" 200 615 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/107.0.0.0 Safari/537.36 Edg/107.0.1418.24" "127.0.0.1, 93.233.184.3"
```

## Which Node Is This Pod Running On?

```bash
kubectl get pods -o wide
```

```yaml
$ kubectl describe po webserver
Name:               webserver
Namespace:          default
Priority:           0
PriorityClassName:  <none>
Node:               gke-standard-cluster-1-default-pool-78257330-5hs8/10.128.0.3
Start Time:         Thu, 28 Nov 2019 13:02:19 +0530
Labels:             <none>
Annotations:        kubectl.kubernetes.io/last-applied-configuration:
                      {"apiVersion":"v1","kind":"Pod","metadata":{"annotations":{},"name":"webserver","namespace":"default"},"spec":{"containers":[{"image":"ngi...
                    kubernetes.io/limit-ranger: LimitRanger plugin set: cpu request for container webserver
Status:             Running
IP:                 10.8.0.3
Containers:
  webserver:
    Container ID:   docker://ff06c3e6877724ec706485374936ac6163aff10822246a40093eb82b9113189c
    Image:          nginx:latest
    Image ID:       docker-pullable://nginx@sha256:189cce606b29fb2a33ebc2fcecfa8e33b0b99740da4737133cdbcee92f3aba0a
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Thu, 28 Nov 2019 13:02:25 +0530
    Ready:          True
    Restart Count:  0
    Requests:
      cpu:        100m
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-mpxxg (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  default-token-mpxxg:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-mpxxg
    Optional:    false
QoS Class:       Burstable
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason     Age    From                                                        Message
  ----    ------     ----   ----                                                        -------
  Normal  Scheduled  2m54s  default-scheduler                                           Successfully assigned default/webserver to gke-standard-cluster-1-default-pool-78257330-5hs8
  Normal  Pulling    2m53s  kubelet, gke-standard-cluster-1-default-pool-78257330-5hs8  pulling image "nginx:latest"
  Normal  Pulled     2m50s  kubelet, gke-standard-cluster-1-default-pool-78257330-5hs8  Successfully pulled image "nginx:latest"
  Normal  Created    2m48s  kubelet, gke-standard-cluster-1-default-pool-78257330-5hs8  Created container
  Normal  Started    2m48s  kubelet, gke-standard-cluster-1-default-pool-78257330-5hs8  Started container
```

## Output in JSON

```json
$ kubectl get pods -o json
{
    "apiVersion": "v1",
    "items": [
        {
            "apiVersion": "v1",
            "kind": "Pod",
            "metadata": {
                "annotations": {
                    "kubectl.kubernetes.io/last-applied-configuration": "{\"apiVersion\":\"v1\",\"kind\":\"Pod\",\"metadata\":{\"annotations\":{},\"name\":\"webserver\",\"namespace\":\"default\"},\"spec\":{\"con
tainers\":[{\"image\":\"nginx:latest\",\"name\":\"webserver\",\"ports\":[{\"containerPort\":80}]}]}}\n",
                    "kubernetes.io/limit-ranger": "LimitRanger plugin set: cpu request for container webserver"
                },
                "creationTimestamp": "2019-11-28T08:48:28Z",
                "name": "webserver",
                "namespace": "default",
                "resourceVersion": "20080",
                "selfLink": "/api/v1/namespaces/default/pods/webserver",
                "uid": "d8e0b56b-11bb-11ea-a1bf-42010a800006"
            },
            "spec": {
                "containers": [
                    {
                        "image": "nginx:latest",
                        "imagePullPolicy": "Always",
                        "name": "webserver",
                        "ports": [
                            {
                                "containerPort": 80,
                                "protocol": "TCP"
                            }
                        ],
                        "resources": {
                            "requests": {
                                "cpu": "100m"
                            }
                        },
                        "terminationMessagePath": "/dev/termination-log",
                        "terminationMessagePolicy": "File",

```

## Executing Commands within Pods

```bash
$ kubectl exec -it webserver -- /bin/bash
root@webserver:/#
```

```bash
root@webserver:/# cat /etc/os-release
PRETTY_NAME="Debian GNU/Linux 11 (bullseye)"
NAME="Debian GNU/Linux"
VERSION_ID="11"
VERSION="11 (bullseye)"
VERSION_CODENAME=bullseye
ID=debian
HOME_URL="https://www.debian.org/"
SUPPORT_URL="https://www.debian.org/support"
BUG_REPORT_URL="https://bugs.debian.org/"
```

Please exit from the shell (`/bin/bash`) session.

```bash
root@webserver:/# exit
```

## Deleting the Pod

```bash
$ kubectl delete -f pods01.yaml
pod "webserver" deleted

$ kubectl get po -o wide
No resources found.
```

# Ading a 2nd container to a Pod

In the microservices architecture, each module should live in its own space and communicate with other modules following a set of rules. But, sometimes we need to deviate a little from this principle. Suppose you have an Nginx web server running and we need to analyze its web logs in real-time. The logs we need to parse are obtained from GET requests to the web server. The developers created a log watcher application that will do this job and they built a container for it. In typical conditions, you’d have a pod for Nginx and another for the log watcher. However, we need to eliminate any network latency so that the watcher can analyze logs the moment they are available. A solution for this is to place both containers on the same pod.

Having both containers on the same pod allows them to communicate through the loopback interface (`ifconfig lo`) as if they were two processes running on the same host. They also share the same storage volume.

Let us see how a pod can host more than one container. Let’s take a look to the [`pods02.yaml`](pods02.yaml) file. It contains the following lines:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webserver
spec:
  containers:
    - name: webserver
      image: nginx:latest
      ports:
        - containerPort: 80
    - name: webwatcher
      image: afakharany/watcher:latest
```

Run the following command:

```bash
kubectl apply -f pods02.yaml
```

```bash
$ kubectl get po
NAME            READY   STATUS    RESTARTS   AGE
pod/webserver   2/2     Running   0          3m6s
```

```bash
$ kubectl get po -o wide
NAME        READY   STATUS    RESTARTS   AGE     IP         NODE                                                NOMINATED NODE   READINESS GATES
webserver   2/2     Running   0          3m37s   10.8.0.5   gke-standard-cluster-1-default-pool-78257330-5hs8   <none>           <none>
```

## How to verify 2 containers are running inside a Pod?

```bash
kubectl describe po
```

```yaml
Containers:
  webserver:
    Container ID:   docker://0564fcb88f7c329610e7da24cba9de6555c0183814cf517e55d2816c6539b829
    Image:          nginx:latest
    Image ID:       docker-pullable://nginx@sha256:36b77d8bb27ffca25c7f6f53cadd059aca2747d46fb6ef34064e31727325784e
    Port:           80/TCP
    State:          Running
      Started:      Wed, 08 Jan 2020 13:21:57 +0530
    Ready:          True
    Restart Count:  0
    Requests:
      cpu:        100m
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-xhgmm (ro)
  webwatcher:
    Container ID:   docker://4cebbb220f7f9695f4d6492509e58152ba661f3ab8f4b5d0a7adec6c61bdde26
    Image:          afakharany/watcher:latest
    Image ID:       docker-pullable://afakharany/watcher@sha256:43d1b12bb4ce6e549e85447678a28a8e7b9d4fc398938a6f3e57d2908a9b7d80
    Port:           <none>
    State:          Running
      Started:      Wed, 08 Jan 2020 13:22:26 +0530
    Ready:          True
    Restart Count:  0
    Requests:
```

Since we have two containers in a pod, we will need to use the `-c` option with `kubectl` when we need to address a specific container. For example:

```bash
$ kubectl exec -it webserver -c webwatcher -- /bin/bash

root@webserver:/# cat /etc/hosts
# Kubernetes-managed hosts file.
127.0.0.1       localhost
::1     localhost ip6-localhost ip6-loopback
fe00::0 ip6-localnet
fe00::0 ip6-mcastprefix
fe00::1 ip6-allnodes
fe00::2 ip6-allrouters
10.8.0.5        webserver
```

Please exit from the shell (`/bin/bash`) session.

```bash
root@webserver:/# exit
```

## Cleaning up

```bash
kubectl delete -f pods02.yaml
```

# Example of Multi-Container Pod

Let's talk about communication between containers in a Pod. Having multiple containers in a single Pod makes it relatively straightforward for them to communicate with each other. They can do this using several different methods.

## Use Cases for Multi-Container Pods

The primary purpose of a multi-container Pod is to support co-located, co-managed helper processes for a primary application. There are some general patterns for using helper processes in Pods:

_Sidecar containers_ help the main container. Some examples include log or data change watchers, monitoring adapters, and so on. A log watcher, for example, can be built once by a different team and reused across different applications. Another example of a sidecar container is a file or data loader that generates data for the main container.

_Proxies, bridges, and adapters_ connect the main container with the external world. For example, Apache HTTP server or nginx can serve static files. It can also act as a reverse proxy to a web application in the main container to log and limit HTTP requests.
Another example is a helper container that re-routes requests from the main container to the external world. This makes it possible for the main container to connect to the localhost to access, for example, an external database, but without any service discovery.

## Shared volumes in a Kubernetes Pod

In Kubernetes, you can use a shared Kubernetes Volume as a simple and efficient way to share data between containers in a Pod. For most cases, it is sufficient to use a directory on the host that is shared with all containers within a Pod.

Kubernetes Volumes enables data to survive container restarts, but these volumes have the same lifetime as the Pod. That means that the volume (and the data it holds) exists exactly as long as that Pod exists. If that Pod is deleted for any reason, even if an identical replacement is created, the shared Volume is also destroyed and created anew.

A standard use case for a multi-container Pod with a shared Volume is when one container writes logs or other files to the shared directory, and the other container reads from the shared directory. For example, we can create a Pod like so ([pods03.yaml](pods03.yaml)):

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mc1
spec:
  volumes:
    - name: html
      emptyDir: {}
  containers:
    - name: 1st
      image: nginx
      volumeMounts:
        - name: html
          mountPath: /usr/share/nginx/html
    - name: 2nd
      image: debian
      volumeMounts:
        - name: html
          mountPath: /html
      command: ["/bin/sh", "-c"]
      args:
        - |
          while true; do
          printf "%s </br>" "$(date)" >> /html/index.html;
          sleep 1;
          done
```

In this file (`pods03.yaml`) a volume named `html` has been defined. Its type is `emptyDir`, which means that the volume is first created when a Pod is assigned to a node, and exists as long as that Pod is running on that node. As the name says, it is initially empty. The `1st` container runs nginx server and has the shared volume mounted to the directory `/usr/share/nginx/html`. The `2nd` container uses the Debian image and has the shared volume mounted to the directory `/html`. Every second, the `2nd` container adds the current date and time into the `index.html` file, which is located in the shared volume. When the user makes an HTTP request to the Pod, the Nginx server reads this file and transfers it back to the user in response to the request.

![Image](https://raw.githubusercontent.com/collabnix/kubelabs/master/pods101/multicontainerpod.png)

```bash
kubectl apply -f pods03.yaml
```

```bash
$ kubectl get po
NAME      READY     STATUS    RESTARTS   AGE
po/mc1    2/2       Running   0          11s

$ kubectl describe po mc1
Name:         mc1
Namespace:    default
Node:         gke-k8s-lab1-default-pool-fd9ef5ad-pc18/10.140.0.16
Start Time:   Wed, 08 Jan 2020 14:29:08 +0530
Labels:       <none>
Annotations:  kubectl.kubernetes.io/last-applied-configuration={"apiVersion":"v1","kind":"Pod","metadata":{"annotations":{},"name":"mc1","namespace":"default"},"spec":{"containers":[{"image":"nginx","name":"1st","v...
              kubernetes.io/limit-ranger=LimitRanger plugin set: cpu request for container 1st; cpu request for container 2nd
Status:       Running
IP:           10.12.2.6
Containers:
  1st:
    Container ID:   docker://b08eb646f90f981cd36c605bf8fead3ca62178c7863598fd4558cb026ed067dd
    Image:          nginx
    Image ID:       docker-pullable://nginx@sha256:36b77d8bb27ffca25c7f6f53cadd059aca2747d46fb6ef34064e31727325784e
    Port:           <none>
    State:          Running
      Started:      Wed, 08 Jan 2020 14:29:09 +0530
    Ready:          True
    Restart Count:  0
    Requests:
      cpu:        100m
    Environment:  <none>
    Mounts:
      /usr/share/nginx/html from html (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-xhgmm (ro)
  2nd:
    Container ID:  docker://63180b4128d477810d6062342f4b8e499de684ffd69ad245c29118e1661eafcb
    Image:         debian
    Image ID:      docker-pullable://debian@sha256:c99ed5d068d4f7ff36c7a6f31810defebecca3a92267fefbe0e0cf2d9639115a
    Port:          <none>
    Command:
      /bin/sh
      -c
    Args:
      while true; do date >> /html/index.html; sleep 1; done
    State:          Running
      Started:      Wed, 08 Jan 2020 14:29:14 +0530
    Ready:          True
    Restart Count:  0
    Requests:
      cpu:        100m
    Environment:  <none>
    Mounts:
      /html from html (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-xhgmm (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  html:
    Type:    EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:
  default-token-xhgmm:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-xhgmm
    Optional:    false
QoS Class:       Burstable
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason     Age   From                                              Message
  ----    ------     ----  ----                                              -------
  Normal  Scheduled  18s   default-scheduler                                 Successfully assigned default/mc1 to gke-k8s-lab1-default-pool-fd9ef5ad-pc18
  Normal  Pulling    17s   kubelet, gke-k8s-lab1-default-pool-fd9ef5ad-pc18  pulling image "nginx"
  Normal  Pulled     17s   kubelet, gke-k8s-lab1-default-pool-fd9ef5ad-pc18  Successfully pulled image "nginx"
  Normal  Created    17s   kubelet, gke-k8s-lab1-default-pool-fd9ef5ad-pc18  Created container
  Normal  Started    17s   kubelet, gke-k8s-lab1-default-pool-fd9ef5ad-pc18  Started container
  Normal  Pulling    17s   kubelet, gke-k8s-lab1-default-pool-fd9ef5ad-pc18  pulling image "debian"
  Normal  Pulled     13s   kubelet, gke-k8s-lab1-default-pool-fd9ef5ad-pc18  Successfully pulled image "debian"
  Normal  Created    12s   kubelet, gke-k8s-lab1-default-pool-fd9ef5ad-pc18  Created container
  Normal  Started    12s   kubelet, gke-k8s-lab1-default-pool-fd9ef5ad-pc18  Started container
```

## Visit the website

```bash
kubectl proxy
```

Open [http://localhost:8001/api/v1/namespaces/default/pods/http:mc1:80/proxy/](http://localhost:8001/api/v1/namespaces/default/pods/http:mc1:80/proxy/) in your Browser

## Cleaning Up

```bash
kubectl delete -f pods03.yaml
```

# Contributor

[Ajeet S Raina](https://twitter.com/ajeetsraina)
[Cedric Ahlers](https://github.com/clowa)

# Reviewers

[vinay agarwal](https://twitter.com/vnyagarwal)
