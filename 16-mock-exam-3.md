# Mock Exam 3

## 1 Create a new service account with the name pvviewer. Grant this Service account access to list all PersistentVolumes in the cluster by creating an appropriate cluster role called pvviewer-role and ClusterRoleBinding called pvviewer-role-binding. Next, create a pod called pvviewer with the image: redis and serviceAccount: pvviewer in the default namespace


Weight: 12

ServiceAccount: pvviewer
ClusterRole: pvviewer-role
ClusterRoleBinding: pvviewer-role-binding
Pod: pvviewer
Pod configured to use ServiceAccount pvviewer ?


```
   70  kubectl create sa pvviewer
   71  kubectl create role -h
   72  kubectl create clusterrole pvviewer-role --verb=list --resource=PersistentVolumes
   73  kubectl create clusterrolebinding -h
   74  kubectl create clusterrolebinding pvviewer-role-binding --clusterrole=pvviewer-role --serviceaccount=default:pvviewer
   75  kubectl run pvviewer --image=redis --serviceaccount=default:pvviewer
   76  kubectl run pvviewer --image=redis --serviceaccount=pvviewer
   77  kubectl get pods
   78  kubectl describe pod pvviewer
   79  kubectl get pod pvviewer -o yaml | grep -i serviceaccount
```

## 2. List the InternalIP of all nodes of the cluster. Save the result to a file /root/CKA/node_ips


Answer should be in the format: InternalIP of master<space>InternalIP of node1<space>InternalIP of node2<space>InternalIP of node3 (in a single line)


Weight: 12

Task Completed

```
kubectl get nodes -o jsonpath='{.items[*].status.addresses[?(@.type=="InternalIP")].address}'
172.17.0.53 172.17.0.54 172.17.0.57 172.17.0.58controlplane $ kubectl get nodes -o jsonpath='{.items[*].status.addresses[?(@.type=="InternalIP")].address}' > /root/CKA/node_ips
```

## 3. Create a pod called multi-pod with two containers.
Container 1, name: alpha, image: nginx
Container 2: beta, image: busybox, command sleep 4800.

Environment Variables:
container 1:
name: alpha

Container 2:
name: beta


Weight: 12

Pod Name: multi-pod
Container 1: alpha
Container 2: beta
Container beta commands set correctly?
Container 1 Environment Value Set
Container 2 Environment Value Set


```
cat << EOF >> multi-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: multi-pod
  name: multi-pod
spec:
  containers:
  - name: alpha
    image: nginx
    env:
    - name: name
      value: alpha
  - command:
    - sleep
    - "4800"
    env:
    - name: name
      value: beta
    image: busybox
    name: beta
EOF

kubectl apply -f multi-pod.yaml
```


## 4. Create a Pod called non-root-pod , image: redis:alpine
runAsUser: 1000
fsGroup: 2000


Weight: 8

Pod `non-root-pod` fsGroup configured
Pod `non-root-pod` runAsUser configured

```
cat << EOF >> non-root-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: non-root-pod
  name: non-root-pod
spec:
  securityContext:
    runAsUser: 1000
    runAsGroup: 2000
    fsGroup: 2000
  containers:
  - image: redis
    name: non-root-pod
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
EOF

kubectl apply -f non-root-pod.yaml
```

## 5. We have deployed a new pod called np-test-1 and a service called np-test-service. Incoming connections to this service are not working. Troubleshoot and fix it.
Create NetworkPolicy, by the name ingress-to-nptest that allows incoming connections to the service over port 80


Important: Don't delete any current objects deployed.


Weight: 14

Important: Don't Alter Existing Objects!
NetworkPolicy: Applied to All sources (Incoming traffic from all pods)?
NetWorkPolicy: Correct Port?
NetWorkPolicy: Applied to correct Pod?


## 6. Fix the staticPod kube-controller-manager


