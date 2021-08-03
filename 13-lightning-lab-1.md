# 1 - Upgrade Cluster

```
Upgrade the current version of kubernetes from 1.18 to 1.19.0 exactly using the kubeadm utility. Make sure that the upgrade is carried out one node at a time starting with the master node. To minimize downtime, the deployment gold-nginx should be rescheduled on an alternate node before upgrading each node.


Upgrade master/controlplane node first. Drain node01 before upgrading it. Pods for gold-nginx should run on the master/controlplane node subsequently.
```

https://v1-19.docs.kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/


## Control Plane

### Upgrade kubeadm

```bash
apt update
apt-cache madison kubeadm

apt-get update && apt-get install -y --allow-change-held-packages kubeadm=1.19.0-00

kubeadm version

sudo kubeadm upgrade plan

sudo kubeadm upgrade apply v1.19.0

sudo kubeadm upgrade node

kubectl drain controlplane --ignore-daemonsets
```

### Kubelet Master Node

```bash
apt-get update && apt-get install -y --allow-change-held-packages kubelet=1.19.0-00 kubectl=1.19.0-00

sudo systemctl daemon-reload
sudo systemctl restart kubelet

kubectl uncordon controlplane
```

## Worker node

### Upgrade kubeadm

```bash
apt-get update && apt-get install -y --allow-change-held-packages kubeadm=1.19.0-00

sudo kubeadm upgrade node
```

From master node

```bash
kubectl drain node01 --ignore-daemonsets
```

From node01

```bash
apt-get update && apt-get install -y --allow-change-held-packages kubelet=1.19.0-00 kubectl=1.19.0-00

sudo systemctl daemon-reload
sudo systemctl restart kubelet
```

From master node

```bash
kubectl uncordon node01
```

## Verify

```bash
kubectl get nodes
```

# 2

```
Print the names of all deployments in the admin2406 namespace in the following format:
DEPLOYMENT CONTAINER_IMAGE READY_REPLICAS NAMESPACE
<deployment name> <container image used> <ready replica count> <Namespace>
. The data should be sorted by the increasing order of the deployment name.


Example:
DEPLOYMENT CONTAINER_IMAGE READY_REPLICAS NAMESPACE
deploy0 nginx:alpine 1 admin2406
Write the result to the file /opt/admin2406_data.

Hint: Make use of -o custom-columns and --sort-by to print the data in the required format.
```

```bash
kubectl -n admin2406 get deploy -o custom-columns='DEPLOYMENT:.metadata.name,CONTAINER_IMAGE:.spec.template.spec.containers[0].image,READY_REPLICAS:.status.readyReplicas,NAMESPACE:.metadata.namespace' --sort-by=.metadata.name > /opt/admin2406_data
```

# 3

```
A kubeconfig file called admin.kubeconfig has been created in /root/CKA. There is something wrong with the configuration. Troubleshoot and fix it.
```

```
diff .kube/config /root/CKA/admin.kubeconfig
vi /root/CKA/admin.kubeconfig
```

# 4

```
Create a new deployment called nginx-deploy, with image nginx:1.16 and 1 replica. Next upgrade the deployment to version 1.17 using rolling update. Make sure that the version upgrade is recorded in the resource annotation.
```

```
kubectl create deployment nginx-deploy --image=nginx:1.16 --dry-run=client --yaml > nginx-deploy.yaml
kubectl create -f nginx-deploy.yaml --record=true
kubectl rollout history deployment nginx-deploy
kubectl set image deployment/nginx-deploy nginx-deploy=nginx:1.17 --record=true
kubectl rollout history deployment nginx-deploy
kubectl get deploy nginx-deploy -o yaml | grep -i image
```

# 5

```
A new deployment called alpha-mysql has been deployed in the alpha namespace. However, the pods are not running. Troubleshoot and fix the issue. The deployment should make use of the persistent volume alpha-pv to be mounted at /var/lib/mysql and should use the environment variable MYSQL_ALLOW_EMPTY_PASSWORD=1 to make use of an empty root password.


Important: Do not alter the persistent volume.
```

```
kubectl -n alpha get pv,pvc,deploy,pod
kubectl -n alpha get pvc alpha-claim -o yaml > mysql-alpha-pvc.yaml
cat << mysql-alpha-pvc.yaml >> EOF
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-alpha-pvc
  namespace: alpha
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: slow
  volumeMode: Filesystem
EOF
kubectl -n alpha get pv,pvc,deploy,pod
```

# 6

```
Take the backup of ETCD at the location /opt/etcd-backup.db on the master node
```

```
export ETCDCTL_API=3
etcdctl -h | grep -A 1 API
    API VERSION:
            3.3

head -n 35 /etc/kubernetes/manifests/etcd.yaml  | grep -A 20 containers

etcdctl \
    --endpoints=https://127.0.0.1:2379 \
    --cacert=/etc/kubernetes/pki/etcd/ca.crt \
    --cert=/etc/kubernetes/pki/etcd/server.crt \
    --key=/etc/kubernetes/pki/etcd/server.key \
    snapshot save /opt/etcd-backup.db

etcdctl \
    --endpoints=https://127.0.0.1:2379 \
    --cacert=/etc/kubernetes/pki/etcd/ca.crt \
    --cert=/etc/kubernetes/pki/etcd/server.crt \
    --key=/etc/kubernetes/pki/etcd/server.key \
    snapshot status /opt/etcd-backup.db -w table

```
# 7

```
Create a pod called secret-1401 in the admin1401 namespace using the busybox image. The container within the pod should be called secret-admin and should sleep for 4800 seconds.

The container should mount a read-only secret volume called secret-volume at the path /etc/secret-volume. The secret being mounted has already been created for you and is called dotfile-secret.
```

```
kubectl run secret-1401 --image=busybox --dry-run=client -o yaml --command -- sleep 4800 > secret-1401.yaml

cat << secret-1401.yaml >> EOF
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: secret-1401
  name: secret-1401
spec:
  containers:
  - command:
    - sleep
    - "4800"
    image: busybox
    name: secret-1401
    volumeMounts:
    - name: secret-volume
      mountPath: "/etc/secret-volume"
      readOnly: true
  volumes:
  - name: secret-volume
    secret:
      secretName: dotfile-secret
EOF

kubectl -n admin1401 apply -f secret-1401.yaml
kubectl -n admin1401 get pods
```
