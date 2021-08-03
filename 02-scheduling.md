# Scheduling

## Manual selector

```
kubectl run nginx --dry-run=client --image=nginx > nginx.yaml

kubectl get pods

kubectl get pods -n kube-system

vim nginx.yaml
..
   nodeName: node01
...
kubectl apply -f nginx.yaml

vim nginx.yaml
..
   nodeName: controlplane
...
kubectl apply -f nginx.yaml
```

## Labels and Selectors

```
kubectl get pods --show-labels

kubectl get pods -l env=dev --no-headers | wc -l

kubectl get pods -l bu=finance --no-headers | wc -l

kubectl get all -l env=prod --no-headers | wc -l

$ cat replicaset-definition-1.yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: replicaset-1
spec:
  replicas: 2
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: nginx >> changed for frontend
    spec:
      containers:
      - name: nginx
        image: nginx
kubectl apply -f replicaset-definition-1.yaml
```

## Taints and Toleration

Taint

```
kubectl get nodes
kubectl describe node node01 | grep -i taint
kubectl taint node node01
kubectl taint node node01 spray=mortein:NoSchedule

kubectl taint nodes node-name key=value:taint-effect
   # NoSchedule | PreferNoSchedule | NoExecute
kubectl taint nodes node1 app=myapp:NoSchedule

kubectl describe node node01 | grep -i tain

kubectl run mosquito --image=nginx

kubectl get pods

kubectl run bee --image=nginx --restart=Never --dry-run=client -o yaml > bee.yaml

$ kubectl explain pod --recursive | grep -A5 tolerations
      tolerations       <[]Object>
         effect <string>
         key    <string>
         operator       <string>
         tolerationSeconds      <integer>
         value  <string>

vim bee.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: bee
  name: bee
spec:
  containers:
  - image: nginx
    name: bee
  tolerations:
  - effect: NoSchedule
    key: spray
    operator: Equal
    value: mortein

kubectl apply -f bee.yaml
kubectl get pods -o wide

kubectl describe node controleplane | grep -i taint

kubectl taint node controlplane node-role.kubernetes.io/master:NoSchedule-
```

Tolerations

```
kubectl get nodes --show-labels
kubectl label node node01 color=blue
kubectl create deployment blue --replicas=6 --image=nginx

kubectl create deployment red --replicas=3 --image=nginx
$ vim red.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: red
  name: red
spec:
  replicas: 3
  selector:
    matchLabels:
      app: red
  template:
    metadata:
      labels:
        app: red
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: node-role.kubernetes.io/master
                operator: Exists
      containers:
      - image: nginx
        name: nginx

kubectl apply -f red.yaml
kubectl get pods -o wide
```

## Resource Requirements and Limits

``` 
1000m = 1 CPU in Kubernetes

1 CPU = 1 vCPU
1 CPU = 1 vCPU AWS / 1 GCP Core / 1 Azure Core / Hyperthread
```

```
1 G (Gigabyte) = 1,000,000,000 bytes
1 M (Megabyte) = 1,000,000 bytes
1 K (Kilobyte) = 1,000 bytes

1 Gi (Gigibyte) = 1,073,741,824 bytes
1 Mi (Megibyte) = 1,048,576 bytes
1 Ki (Kibibyte) = 1,024 bytes
```

```
CPU over Limits >> THROTTLE
MEM over Limits >> TERMINATE
```

### LimitRange

```
apiVersion: v1
kind: LimitRange
metadata:
  name: mem-limit-range
spec:
  limits:
  - default:
      memory: 512Mi
    defaultRequest:
      memory: 256Mi
    type: Container
```

```
apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-limit-range
spec:
  limits:
  - default:
      cpu: 1
    defaultRequest:
      cpu: 0.5
    type: Container
```

[LimitRange](https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/cpu-default-namespace/)

```
apiVersion: apps/v1
kind: Pod
metadata:
  name: elephant
  namespace: default
spec:
  containers:
  - args:
    - --vm
    - "1"
    - --vm-bytes
    - 15M
    - --vm-hang
    - "1"
    command:
    - stress
    image: polinux/stress
    name: mem-stress
    resources:
      limits:
        memory: 10Mi
      requests:
        memory: 5Mi
  nodeName: node01
  tolerations:
  - effect: NoExecute
    key: node.kubernetes.io/not-ready
    operator: Exists
    tolerationSeconds: 300
  - effect: NoExecute
    key: node.kubernetes.io/unreachable
    operator: Exists
    tolerationSeconds: 300
```

## DaemonSets

DaemonSets it's equal to ReplicaSets insted of changed kind options.

```
kubectl get daemonsets
kubectl get daemonset -A
kubectl get daemonset -A --no-headers | wc -l
kubectl get nodes -n kube-system -l kubernetes.io/os=linux --no-headers | wc -l
kubectl describe ds -n kube-system kube-flannel-ds-amd64 | grep -i image

$ cat daemonset.yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: elasticsearch
  namespace: kube-system
  labels:
    app: elasticsearch
spec:
  selector:
    matchLabels:
      name: elasticsearch
  template:
    metadata:
      labels:
        name: elasticsearch
    spec:
      tolerations:
      # this toleration is to have the daemonset runnable on master nodes
      # remove it if your masters can't run pods
      - key: node-role.kubernetes.io/master
        effect: NoSchedule
      containers:
      - name: elasticsearch
        image: k8s.gcr.io/fluentd-elasticsearch:1.20

$ kubectl apply -f daemonset.yaml
```

## Static PODs

Static PODs vs DaemonSets

Static PODs created by kubelet // DaemonSets create by Kube-API Server (DaemonSets Controller)
Deploy ControlPlane Componentes as Static Pods // Deploy Monitoring Agents, Logging and Agent on nodes.

```
kubect get pods --all-namespaces | grep controlplane
ls /etc/kubernets/manifests/ | wc -l
grep -i image /etc/kubernetes/manifests/kube-apiserver.yaml

$ grep -i static /var/lib/kubelet/config.yaml
staticPodPath: /etc/kubernetes/manifests

cd /etc/kubebernetes/manifest

kubectl run static-busybox --image=busybox --command sleep 1000 --restart=Never --dry-ru
n=client -o yaml > static-busybox.yaml

$ vi static-busybox.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: static-busybox
  name: static-busybox
spec:
  containers:
  - command:
    - sleep
    - "1000"
    image: busybox:1.28.4
    name: static-busybox
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}

# Delete static pods in worker node

kubect get nodes -o wide
ssh <INTERNAL IP>
cd $(grep -i static /var/lib/kubelet/config.yaml | awk '{print $2}')
rm greenbox.yaml
exit

kubectl get pods -o wide
```

## Managing Multiple Schedulers

Create a second scheduler

https://kubernetes.io/docs/tasks/extend-kubernetes/configure-multiple-schedulers/

```
cp /etc/kubernetes/manifests/kube-scheduler.yaml my-scheduler.yaml
...
    spec:
      serviceAccountName: my-scheduler
      containers:
      - command:
        - /usr/local/bin/kube-scheduler
        - --address=0.0.0.0
        - --leader-elect=false
        - --scheduler-name=my-scheduler
...
:g/kube-scheduler/s//my-scheduler/g

$ vi nginx-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: annotation-second-scheduler
  labels:
    name: nginx-pod
spec:
  schedulerName: my-scheduler
  containers:
  - name: nginx
    image: nginx

kubectl apply -f nginx-pod.yaml
```

