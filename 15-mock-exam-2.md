# Mock Exam 2

## 1. Take a backup of the etcd cluster and save it to /opt/etcd-backup.db

```
etcdctl -v | grep VERSION -A1

export ETCDCTL_API=3

etcdctl --endpoints=https://127.0.0.1:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key snapshot save /opt/etcd-backup.db

etcdctl --endpoints=https://127.0.0.1:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key snapshot status /opt/etcd-backup.db -w table
```

## 2. Create a Pod called redis-storage with image: redis:alpine with a Volume of type emptyDir that lasts for the life of the Pod. Specs on the right.

```
cat << EOF >> redis-storage.yaml
apiVersion: v1
kind: Pod
metadata:
  name: redis-storage
spec:
  containers:
  - image: redis:alpine
    name: redis-storage
    volumeMounts:
    - mountPath: /data/redis
      name: redis-storage
  volumes:
  - name: redis-storage
    emptyDir: {}
EOF

kubectl create -f redis-storage.yaml
```

## 3. Create a new pod called super-user-pod with image busybox:1.28. Allow the pod to be able to set system_time The container should sleep for 4800 seconds

```
Pod: super-user-pod
Container Image: busybox:1.28
SYS_TIME capabilities for the conatiner?
```

```
cat << EOF >> super-user-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: super-user-pod
  name: super-user-pod
spec:
  containers:
  - command:
    - sleep
    - "4800"
    image: busybox:1.28
    securityContext:
      capabilities:
        add: ["SYS_TIME"]
    name: super-user-pod
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
EOF

kubectl create -f super-user-pod.yaml
```

## 4. A pod definition file is created at /root/CKA/use-pv.yaml. Make use of this manifest file and mount the persistent volume called pv-1. Ensure the pod is running and the PV is bound.


mountPath: /data persistentVolumeClaim Name: my-pvc


Weight: 12

persistentVolume Claim configured correctly
pod using the correct mountPath
pod using the persistent volume claim?


```
cat << EOF >> use-pv.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: use-pv
  name: use-pv
spec:
  containers:
  - image: nginx
    name: use-pv
    volumeMounts:
      - mountPath: "/data"
        name: my-data
  volumes:
    - name: my-data
      persistentVolumeClaim:
        claimName: my-pvc
EOF

cat << EOF >> my-pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  volumeName: pv-1
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Mi
EOF

kubectl apply -f my-pvc.yaml
kubectl apply -f use-pv.yaml
kubectl get pods,pvc
```


## 4. Create a new deployment called nginx-deploy, with image nginx:1.16 and 1 replica. Record the version. Next upgrade the deployment to version 1.17 using rolling update. Make sure that the version upgrade is recorded in the resource annotation.

```
kubectl create deployment nginx-deploy --image=nginx:1.16 --replicas=1 --dry-run=client -o yaml > nginx-deploy.yaml
kubectl create -f nginx-deploy.yaml --record=true
kubectl rollout history deployment nginx-deploy
kubectl set image deployment nginx-deploy nginx=nginx:1.17 --record=true
kubectl rollout history deployment nginx-deploy
```


## 5. Create a new user called john. Grant him access to the cluster. John should have permission to create, list, get, update and delete pods in the development namespace . The private key exists in the location: /root/CKA/john.key and csr at /root/CKA/john.csr


Important Note: As of kubernetes 1.19, the CertificateSigningRequest object expects a signerName.

Please refer documentation below to see the example usage:
https://kubernetes.io/docs/reference/access-authn-authz/certificate-signing-requests/#create-certificate-request-kubernetes-object


Weight: 15

CSR: john-developer Status:Approved
Role Name: developer, namespace: development, Resource: Pods
Access: User 'john' has appropriate permissions

```
controlplane $ cat /root/CKA/john.csr | base64 -w0
LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ1ZEQ0NBVHdDQVFBd0R6RU5NQXNHQTFVRUF3d0VhbTlvYmpDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRApnZ0VQQURDQ0FRb0NnZ0VCQUxWbjhnOS9XR2tKTFNsa3hUd1B6cTUwRzlNTjl3bXRkdEJHK2VoeG5kdktLY2lKCkdGTWFzcFVaZ1BVY0VkL1NRSlQ0T09TeUZ4SU5kMm55UnhPa25PQzRxbElCS3h5WXQ1bXJLSUZ3d1RqRnVTMnYKM2xRQWtCampzMlNoUHFDUjdZTFBJT0pBZk1GU29scUFjRHNiaUtYeDJ1WUhVS003TU00ZHViM1Q0Q3FBRlBWSQp1ZU95bTIySERjY253TXJBVm5sazE5U3NwRU1NMG5Xd2YrSTYzbnV6dDRlSTFxT0FWd1JuOTNPT2h6eVFLby8wCjRrbmd5RENEZ01PVDJLVFlrb3VpZG9waHZkcUN2U2hCMWRsQTdwYnhSVy85RTRUbjZlRWE5dEJUQUZNN0x5a3IKZVdxdkgwMzJrbXdxOGFsdTc3dHVnWmZkQzN0SVJYancxdU5qb3FFQ0F3RUFBYUFBTUEwR0NTcUdTSWIzRFFFQgpDd1VBQTRJQkFRQUdvYVRXdFB1NXdxK1V6NHVTRW96bHRzdkpKY0t1Y2YzYmZnY1pSN1lqQ3ZjLzNIS2kybVNXCkpWemozN2t0WWJFQVFCRkRKSER4ZWRFSzR0Z2FxSjVTUmUwRnpvdC8rOS8wTFIxSWxSaWljMjVqTlU4bjBGQ28KY3hRTjl6bzlPOFlnc21FRGVQTTNLZExQclZvMm03d0FMQ3JWYVVEczUwd3FvVWdmaWpkbnBGQVg4bEg3TGtXUwphR1hBZzhGYk9QMVlzZjJGUFFDbjVBWit4ek9xUm5NRlFzajZ2N2NtQXpaYUhiTm9weEJiN0JUS1dZQUZlQjJtCllTS1hmS1hNeTJiVVRPVHFpdmtJS1AwNUVxMC90MVppSy9qOU9ISkVpRWVyNGVUdGoyd0VVQkZlRWRjSlFZbnEKVG5PYVJreHZpSzhVOWxFQUxMMjBSNTc5c3FTOHg0WTkKLS0tLS1FTkQgQ0VSVElGSUNBVEUgUkVRVUVTVC0tLS0tCg==

cat << EOF > john-csr.yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: john
spec:
  groups:
  - system:authenticated
  request: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ1ZEQ0NBVHdDQVFBd0R6RU5NQXNHQTFVRUF3d0VhbTlvYmpDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRApnZ0VQQURDQ0FRb0NnZ0VCQUxWbjhnOS9XR2tKTFNsa3hUd1B6cTUwRzlNTjl3bXRkdEJHK2VoeG5kdktLY2lKCkdGTWFzcFVaZ1BVY0VkL1NRSlQ0T09TeUZ4SU5kMm55UnhPa25PQzRxbElCS3h5WXQ1bXJLSUZ3d1RqRnVTMnYKM2xRQWtCampzMlNoUHFDUjdZTFBJT0pBZk1GU29scUFjRHNiaUtYeDJ1WUhVS003TU00ZHViM1Q0Q3FBRlBWSQp1ZU95bTIySERjY253TXJBVm5sazE5U3NwRU1NMG5Xd2YrSTYzbnV6dDRlSTFxT0FWd1JuOTNPT2h6eVFLby8wCjRrbmd5RENEZ01PVDJLVFlrb3VpZG9waHZkcUN2U2hCMWRsQTdwYnhSVy85RTRUbjZlRWE5dEJUQUZNN0x5a3IKZVdxdkgwMzJrbXdxOGFsdTc3dHVnWmZkQzN0SVJYancxdU5qb3FFQ0F3RUFBYUFBTUEwR0NTcUdTSWIzRFFFQgpDd1VBQTRJQkFRQUdvYVRXdFB1NXdxK1V6NHVTRW96bHRzdkpKY0t1Y2YzYmZnY1pSN1lqQ3ZjLzNIS2kybVNXCkpWemozN2t0WWJFQVFCRkRKSER4ZWRFSzR0Z2FxSjVTUmUwRnpvdC8rOS8wTFIxSWxSaWljMjVqTlU4bjBGQ28KY3hRTjl6bzlPOFlnc21FRGVQTTNLZExQclZvMm03d0FMQ3JWYVVEczUwd3FvVWdmaWpkbnBGQVg4bEg3TGtXUwphR1hBZzhGYk9QMVlzZjJGUFFDbjVBWit4ek9xUm5NRlFzajZ2N2NtQXpaYUhiTm9weEJiN0JUS1dZQUZlQjJtCllTS1hmS1hNeTJiVVRPVHFpdmtJS1AwNUVxMC90MVppSy9qOU9ISkVpRWVyNGVUdGoyd0VVQkZlRWRjSlFZbnEKVG5PYVJreHZpSzhVOWxFQUxMMjBSNTc5c3FTOHg0WTkKLS0tLS1FTkQgQ0VSVElGSUNBVEUgUkVRVUVTVC0tLS0tCg==
  signerName: kubernetes.io/kube-apiserver-client
  usages:
  - client auth
  username: john
EOF
```

```
kubectl apply -f john-csr.yaml
kubectl get csr
kubectl certificate approve john-developer
kubectl get csr
kubectl get csr/john-developer -o yaml
kubectl create role developer --verb=create,get,list,update,delete --resource=pods -n development
kubectl create rolebinding developer-binding-john --role=developer --user=john -n development
kubectl auth can-i get pods -n development
kubectl auth can-i get pods -n development --as=john
kubectl auth can-i create pods -n development --as=john
kubectl auth can-i list pods -n development --as=john
kubectl auth can-i update pods -n development --as=john
kubectl auth can-i delete pods -n development --as=john
kubectl auth can-i get nodes -n development --as=john
```

## 6. Create an nginx pod called nginx-resolver using image nginx, expose it internally with a service called nginx-resolver-service. Test that you are able to look up the service and pod names from within the cluster. Use the image: busybox:1.28 for dns lookup. Record results in /root/CKA/nginx.svc and /root/CKA/nginx.pod


Weight: 15

Pod: nginx-resolver created
Service DNS Resolution recorded correctly
Pod DNS resolution recorded correctly

```
kubectl run nginx-resolver --image=nginx
kubectl expose pod nginx-resolver --port=80 --target-port=80 --name=nginx-resolver-service
kubectl run test-dns --image=busybox:1.28 --command -- sleep 48000
kubectl exec test-dns -- nslookup nginx-resolver-service > /root/CKA/nginx.svc
kubectl exec test-dns -- nslookup 10-244-1-8.default.pod > /root/CKA/nginx.pod
```

## 7. Create a static pod on node01 called nginx-critical with image nginx. Create this pod on node01 and make sure that it is recreated/restarted automatically in case of a failure. Use /etc/kubernetes/manifests as the Static Pod path for example.


Weight: 15

Kubelet Configured for Static Pods
Pod nginx-critical-node01 is Up and running

```
kubectl run nginx-critical --image=nginx --dry-run=client -o yaml > nginx-critical.yaml
scp nginx-critical.yaml node01:/etc/kubernetes/manifests/
grep static /var/lib/kubelet/config.yaml
ssh node01
cat "staticPodPath: /etc/kubernetes/manifests" >> /var/lib/kubelet/config.yaml
systemctl restart kubelet
exit
kubectl get pods 
kubectl delete pod nginx-critical-node01
kubectl get pods 
```
