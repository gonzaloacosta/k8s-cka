# Mock Exam 4

```bash
cat << EOF >> ~/.vimrc
set ts=2
set sts=2
set sw=2
set expandtab
syntax on
filetype indent plugin on
set ruler
```

```bash
cat << EOF >> .bashrc
# Kubernetes
alias k=kubectl
alias kd='kubectl delete'
alias kr='kubectl run'
export do="-o yaml --dry-run=client"
export ETCDCTL_API=3
EOF

kubectl completion bash >/etc/bash_completion.d/kubectl
echo 'alias k=kubectl' >>~/.bashrc
echo 'complete -F __start_kubectl k' >>~/.bashrc
source /usr/share/bash-completion/bash_completion

source ~/.bashrc
```

# 1. 

Use context: kubectl config use-context k8s-c2-AC

The cluster admin asked you to find out the following information about etcd running on cluster2-master1:

Server private key location
Server certificate expiration date
Is client certificate authentication enabled
Write these information into /opt/course/p1/etcd-info.txt

Finally you're asked to save an etcd snapshot at /etc/etcd-snapshot.db on cluster2-master1 and display its status.

```bash
# Cert
k get nodes
ssh master
sudo su -
k get pods -A
ls -ltr /etc/kubernetes/manifests/etcd.yaml

head -40 /etc/kubernetes/manifests/etcd.yaml
...
spec:
  containers:
  - command:
    - etcd
    - --advertise-client-urls=https://10.0.0.10:2379    >> endpoints
    - --cert-file=/etc/kubernetes/pki/etcd/server.crt   >> validate date cert 
    - --client-cert-auth=true                           >> validate user auth
...

openssl x509 -noout -dates -in /etc/kubernetes/pki/etcd/server.crt 
notBefore=Nov  4 13:23:24 2020 GMT
notAfter=Nov  4 13:23:24 2021 GMT

cat << EOF >> /root/CKA/etcd-info.txt
Server private key location: /etc/kubernetes/pki/etcd/server.key
Server certificate expiration date: Nov  4 13:23:24 2021 GMT
Is client certificate authentication enabled: yes
EOF

# Backup
export ETCDCTL_API=3
etcdctl --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key member list -w table
etcdctl --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key snapshot save /root/CKA/etcd-snapshot.db
etcdctl --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key snapshot status /root/CKA/etcd-snapshot.db -w table
```


## 2

Use context: kubectl config use-context k8s-c1-H

 

You're asked to confirm that kube-proxy is running correctly on all nodes. For this perform the following in Namespace project-hamster:

Create a new Pod named p2-pod with two containers, one of image nginx:1.17-alpine and one of image busybox:1.31. Make sure the busybox container keeps running for some time.

Create a new Service named p2-service which exposes that Pod internally in the cluster on port 3000->80.

Confirm that kube-proxy is running on all nodes cluster1-master1, cluster1-worker1 and cluster1-worker2 and that it's using iptables.

Write the iptables rules of all nodes belonging the created Service p2-service into file /opt/course/p2/iptables.txt.

Finally delete the Service and confirm that the iptables rules are gone from all nodes.

```bash
# Create pod
k run p2-pod --image=nginx:1.17-alpine $do > p2.yaml

vim p2.yaml
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: p2-pod
  name: p2-pod
  namespace: project-hamster #ADD
spec:
  containers:
  - image: nginx:1.17-alpine
    name: c1
  - image: busybox:1.31
    name: c2                 #ADD
    command:                 #ADD or command: ["sh","-c","sleep 4800"]
    - sleep
    - "4800"

k apply -f p2.yaml

k -n project-hamster exec p2-pod -c c2 -- ps aux

# Expose service
k -n project-hamster expose pod p2-pod --name=p2-service --port=3000 --target-port=80

# Check iptables rules
for i in $(echo master node1 node2 node3) ; do echo ">>>> $i <<<<<< " ; ssh $i -C 'sudo iptables-save' | grep p2-service ; done 
```

# 3

Preview Question 3
Use context: kubectl config use-context k8s-c1-H

 

There should be two schedulers on cluster1-master1, but only one is is reported to be running. Write all scheduler Pod names and their status into /opt/course/p3/schedulers.txt.

There is an existing Pod named special in Namespace default which should be scheduled by the second scheduler, but it's in a pending state.

Fix the second scheduler. Confirm it's working by checking that Pod special is scheduled on a node and running.

```bash
# Check the pod labels
k -n kube-system get pod --show-labels
k -n kube-system get pods -l component=kube-scheduler > ~/CKA/schedulers.txt

k get pods -o wide

k get pods -o jsonpath="{.items[*].spec.schedulerName}{'\n'}"

# Create a new scheduller
sudo cp /etc/kubernetes/manifest/kube-scheduler.yaml my-scheduler.yaml

# Change 

# liveness.httpGet.scheme: HTTP
# readiness.httpGet.scheme: HTTP
# liveness.httpGet.port: 10282
# readiness.httpGet.port: 10282

# ADD 

#- --port=10282
#- --secure-port=0 (not https)
#- --scheduler-name=my-scheduler
#- --leader-elect=false

vim my-scheduler.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    component: kube-scheduler
    tier: control-plane
  name: my-scheduler
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-scheduler
    - --authentication-kubeconfig=/etc/kubernetes/scheduler.conf
    - --authorization-kubeconfig=/etc/kubernetes/scheduler.conf
    - --bind-address=127.0.0.1
    - --kubeconfig=/etc/kubernetes/scheduler.conf
    - --leader-elect=false
    #- --leader-elect=true
    - --scheduler-name=my-scheduler
    #- --lock-object-name=kube-scheduler-custom
    - --port=10282
    - --secure-port=0
    image: k8s.gcr.io/kube-scheduler:v1.19.3
    imagePullPolicy: IfNotPresent
    livenessProbe:
      failureThreshold: 8
      httpGet:
        host: 127.0.0.1
        path: /healthz
        port: 10282
        scheme: HTTP
      initialDelaySeconds: 10
      periodSeconds: 10
      timeoutSeconds: 15
    name: kube-scheduler
    resources:
      requests:
        cpu: 100m
    startupProbe:
      failureThreshold: 24
      httpGet:
        host: 127.0.0.1
        path: /healthz
        port: 10282
        scheme: HTTP
      initialDelaySeconds: 10
      periodSeconds: 10
      timeoutSeconds: 15
    volumeMounts:
    - mountPath: /etc/kubernetes/scheduler.conf
      name: kubeconfig
      readOnly: true
  hostNetwork: true
  priorityClassName: system-node-critical
  volumes:
  - hostPath:
      path: /etc/kubernetes/scheduler.conf
      type: FileOrCreate
    name: kubeconfig
status: {}


sudo mv my-scheduler.yaml /etc/kubernetes/manifests/
k -n kube-system get pods


# Create pod
k run p2 --image=busybox:1.28 --dry-run=client -o yaml --command -- sleep 4800 > p2.yaml
k explain pod.spec.schedulerName 
vim p2.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: p2
  name: p2
spec:
  containers:
  - command:
    - sleep
    - "4800"
    image: busybox:1.28
    name: p2
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
  schedulerName: my-scheduler
status: {}
```