# Security

## Kubernetes Security Primitives

## Authentication

```
# User File Contents
password123,user1,u0001
password123,user2,u0002
password123,user3,u0003
password123,user4,u0004
password123,user5,u0005
```

```
vi /etc/kubernetes/manifests/kube-apiserver.yaml
apiVersion: v1
kind: Pod
metadata:
  name: kube-apiserver
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-apiserver
      <content-hidden>
    image: k8s.gcr.io/kube-apiserver-amd64:v1.11.3
    name: kube-apiserver
    volumeMounts:
    - mountPath: /tmp/users
      name: usr-details
      readOnly: true
  volumes:
  - hostPath:
      path: /tmp/users
      type: DirectoryOrCreate
    name: usr-details
```


```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  name: kube-apiserver
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-apiserver
    - --authorization-mode=Node,RBAC
      <content-hidden>
    - --basic-auth-file=/tmp/users/user-details.csv
```

```
---
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""] # "" indicates the core API group
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
 
---
# This role binding allows "jane" to read pods in the "default" namespace.
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: read-pods
  namespace: default
subjects:
- kind: User
  name: user1 # Name is case sensitive
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role #this must be Role or ClusterRole
  name: pod-reader # this must match the name of the Role or ClusterRole you wish to bind to
  apiGroup: rbac.authorization.k8s.io
```

```
curl -v -k https://localhost:6443/api/v1/pods -u "user1:password123"
```

## TLS Introduction

View the spreadsheet [here](docs/kubernetes-certs-checker.xlsx)

```
ls -ltr /etc/kubernetes/pki/
ls -ltr /etc/kubernetes/pki/etcd/

openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text -noout
```

Generate CSR

```
$ ls akshay.*
akshay.csr  akshay.key
$ USER_CSR=$(base64 -w0 akshay.csr)

cat << EOF >> akshay-csr.yaml
apiVersion: certificates.k8s.io/v1beta1
kind: CertificateSigningRequest
metadata:
  name: akshay
spec:
  groups:
  - system:authenticated
  request: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ1ZqQ0NBVDRDQVFBd0VURVBNQTBHQTFVRUF3d0dZV3R6YUdGNU1JSUJJakFOQmdrcWhraUc5dzBCQVFFRgpBQU9DQVE4QU1JSUJDZ0tDQVFFQTMyOSticklXVXU3WGJ0SFdwMllJTWE2OU1hL0xUaG9aOWNNRW5NM0Jab0FJCjNsZTA4WmRzVFZxcVZuZGhwNktPdnBZdzVmNyt2a3UxZGNpZU0rTFdLd2Jkbmt1M21qYjJQcFk0bGNHOXZGR3AKYW9Sc2FoeW8zYWQ4UWZMVTZKTGZoZ1NCWXNCQTBWWUJ0cVRJcU1kTk0rNTJIS1ZMcjBCSzZtZmQ4TUlIdGk0NwpHdlA0U3piSjZ6QXRlSWJ2MWlITVVQZWgwOGxwTDZMZmdVcnF1MzRJSVJtaUZRN3VwaTNnT3dZOFNmYjFscSttClpsa21JMjIyc0xrWkdKcE43anA4b1VPRUtxNktyMFg3a2t5b2p3SWxCazNzSW4rai85c3VRZXAwK2pTdkFISHMKTmVWdUFFRnNtLzkwUmpLTU9JS01CMTFuUTdaTCtkSVhqN3NWbEJOeWt3SURBUUFCb0FBd0RRWUpLb1pJaHZjTgpBUUVMQlFBRGdnRUJBQW4yeGcyeWQ3WDFXdEtjcWMrN29IZ2Fyd01sMnYxaTc4ajNRWHZvcVUvM0doQkdyRjZsCmRKYXdDb01JV1o0My9pcDRrQUV0UkRHKzNJcTV4RWRiSllLNHpSc3kyNThicElWbi9uZ3FPbklaUVY0VFNFMWwKMDQwRXlKK1ZNMHIrNU9hUDZsb1hVWE4wYmFhbWkvRllYRXdXSTZkQW5XNW53UVg5SFhvUElMdk5KbjJKYVp5eAp2UldHWkd0VVgyK2dqenlUTTI1UHIxY2V4RHgwY3hQQk52YXhTYWRWQ090akxWcjVQZnlNemtUY2pwK2E2MS81CmwwNWNnWThqZWduM2FGMFZpYk1hNjFxc1RmcDlSd0tNZzd2VUxoeElpbXpZT0lNbk11cjc3Mk16WkZCTDFDVEoKcFYycm1YRVlqYUlhdDVuaXVjcUFpWUduZmVQUElZWjVCNnM9Ci0tLS0tRU5EIENFUlRJRklDQVRFIFJFUVVFU1QtLS0tLQo=
  usages:
  - digital signature
  - key encipherment
  - server auth
EOF

kubectl apply -f akshay-csr.yaml

kubectl get csr

kubectl certificate approve akshay

kubectl certificate deny agent-smith
```


Context file example
```
controlplane $ cat my-kube-config
apiVersion: v1
kind: Config

clusters:
- name: production
  cluster:
    certificate-authority: /etc/kubernetes/pki/ca.crt
    server: KUBE_ADDRESS

- name: development
  cluster:
    certificate-authority: /etc/kubernetes/pki/ca.crt
    server: KUBE_ADDRESS

- name: kubernetes-on-aws
  cluster:
    certificate-authority: /etc/kubernetes/pki/ca.crt
    server: KUBE_ADDRESS

- name: test-cluster-1
  cluster:
    certificate-authority: /etc/kubernetes/pki/ca.crt
    server: KUBE_ADDRESS

contexts:
- name: test-user@development
  context:
    cluster: development
    user: test-user

- name: aws-user@kubernetes-on-aws
  context:
    cluster: kubernetes-on-aws
    user: aws-user

- name: test-user@production
  context:
    cluster: production
    user: test-user

- name: research
  context:
    cluster: test-cluster-1
    user: dev-user

users:
- name: test-user
  user:
    client-certificate: /etc/kubernetes/pki/users/test-user/test-user.crt
    client-key: /etc/kubernetes/pki/users/test-user/test-user.key
- name: dev-user
  user:
    client-certificate: /etc/kubernetes/pki/users/dev-user/developer-user.crt
    client-key: /etc/kubernetes/pki/users/dev-user/dev-user.key
- name: aws-user
  user:
    client-certificate: /etc/kubernetes/pki/users/aws-user/aws-user.crt
    client-key: /etc/kubernetes/pki/users/aws-user/aws-user.key

current-context: test-user@development
```

Set current context

```
kubectl config --kubeconfig=/root/my-kube-config use-context research

kubectl config get-context
```


## Role Based Access Control

Test authorization mode

```
grep authorization-mode /etc/kubernetes/manifiests/kube-apiserver.yaml
    - --authorization-mode=Node,RBAC
```

How many role exist in all namespaces

```
kubectl get roles -A --no-headers | wc -l

kubectl auth can-i get pods --as dev-user
no

```

Create role for developer user list and create pods.

```yaml
$ cat /var/answers/developer-role.yaml
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: default
  name: developer
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["list", "create"]


---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: dev-user-binding
subjects:
- kind: User
  name: dev-user
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: developer
  apiGroup: rbac.authorization.k8s.io
```


The dev-user is trying to get details about the dark-blue-app pod in the blue namespace. Investigate and fix the issue.

```
vi developer-role.yaml 
# change namespace and add "get" verb.

kubectl describe pods dark-blue-app -n blue --as dev-user
```



Permite to dev-user create deployment into default namespaces

```yaml
cat /var/answers/dev-user-deploy.yaml
---
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: blue
  name: deploy-role
rules:
- apiGroups: ["apps", "extensions"]
  resources: ["deployments"]
  verbs: ["create"]

---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: dev-user-deploy-binding
  namespace: blue
subjects:
- kind: User
  name: dev-user
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: deploy-role
  apiGroup: rbac.authorization.k8s.io
```

## ClusterRole and ClusterRoleBinding



Grant node permission

```yaml
$ cat /var/answers/michelle-node-admin.yaml
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: node-admin
rules:
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["get", "watch", "list", "create", "delete"]

---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: michelle-binding
subjects:
- kind: User
  name: michelle
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: node-admin
  apiGroup: rbac.authorization.k8s.io
```

Grant storage permission



```yaml
kubectl api-resources

$ cat /var/answers/michelle-storage-admin.yaml
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: storage-admin
rules:
- apiGroups: [""]
  resources: ["persistentvolumes"]
  verbs: ["get", "watch", "list", "create", "delete"]
- apiGroups: ["storage.k8s.io"]
  resources: ["storageclasses"]
  verbs: ["get", "watch", "list", "create", "delete"]

---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: michelle-storage-admin
subjects:
- kind: User
  name: michelle
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: storage-admin
  apiGroup: rbac.authorization.k8s.io
```

## Images

Create docker registry password

```
kubectl create secret docker-registry myregistrykey --docker-server=DUMMY_SERVER \
        --docker-username=DUMMY_USERNAME --docker-password=DUMMY_DOCKER_PASSWORD \
        --docker-email=DUMMY_DOCKER_EMAIL

kubectl get secrets myregistrykey

kubectl patch serviceaccount default -p '{"imagePullSecrets": [{"name": "myregistrykey"}]}'

kubectl run nginx --image=nginx --restart=Never
kubectl get pod nginx -o=jsonpath='{.spec.imagePullSecrets[0].name}{"\n"}'

https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/#add-imagepullsecrets-to-a-service-account
```

## Security Context

```
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper
  namespace: default
spec:
  securityContext:
    runAsUser: 1010
  containers:
  - command:
    - sleep
    - "4800"
    image: ubuntu
    name: ubuntu-sleeper
```

```
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper
  namespace: default
spec:
  containers:
  - command:
    - sleep
    - "4800"
    image: ubuntu
    name: ubuntu-sleeper
    securityContext:
      capabilities:
        add: ["SYS_TIME"]
```

```
controlplane $ kubectl exec -it ubuntu-sleeper -- date -s '19 APR 2012 11:14:00'
Thu Apr 19 11:14:00 UTC 2012
controlplane $ date
Sat Dec 19 23:02:22 UTC 2020
controlplane $
```

## Network Policies

```
internal pod ---> payroll pod ---> db pod
```


* Ingress Policy

```
 cat payroll-policy.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: payroll-policy
  namespace: default
spec:
  ingress:
  - from:
    - podSelector:
        matchLabels:
          name: internal
    ports:
    - port: 8080
      protocol: TCP
  podSelector:
    matchLabels:
      name: payroll
  policyTypes:
  - Ingress
```

* Egress Policy

```
cat /var/answers/answer-internal-policy.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: internal-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      name: internal
  policyTypes:
  - Egress
  - Ingress
  ingress:
    - {}
  egress:
  - to:
    - podSelector:
        matchLabels:
          name: mysql
    ports:
    - protocol: TCP
      port: 3306

  - to:
    - podSelector:
        matchLabels:
          name: payroll
    ports:
    - protocol: TCP
      port: 8080

  - ports:
    - port: 53
      protocol: UDP
    - port: 53
      protocol: TCP
```



### Examples


# Generate CSR


## cfssl

```
cat << EOF | cfssl genkey - | cfssljson -bare gonzalo
{
  "CN": "gonzalo",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "AR",
      "L": "Capital Federal",
      "O": "system:authenticated",
      "OU": "CKA practice exercises",
      "ST": "Buenos Aires"
    }
  ]
}
EOF


cat << EOF > gonzalo-csr.yml
apiVersion: certificates.k8s.io/v1beta1
kind: CertificateSigningRequest
metadata:
  name: gonzalo-csr
spec:
  request: $(cat gonzalo.csr | base64 | tr -d '\n')
  username: gonzalo
  usages:
  - digital signature
  - key encipherment
  - client auth
EOF
```

## OpenSSL


```bash
# Generate key for kubernetes ca
openssl genrsa -out ca.key 2048

# Generate csr for kubernetes ca
openssl req -new -key ca.key -subj "/CN=KUBERNETES-CA" -out ca.csr

# Sign certificate
openssl x509 -req -in ca.csr -signkey ca.key -out ca.crt
```

```bash
# Generate key for admin user
openssl genrsa -out admin.key 2048

# Generate csr for admin user
openssl req -new -key admin.key -subj "/CN=admin" -out admin.csr
```


```bash
# Sing certificate with the CA
openssl x509 -req -in admin.csr -CA ca.crt -CAkey ca.key -out admin.crt
```

## Openssl generate certificate for any user

```bash
openssl genrsa -out clara.key 2048
openssl req -new -key clara.key -subj "/CN=clara" -out clara.csr
cat << EOF > clara-csr.yml
apiVersion: certificates.k8s.io/v1beta1
kind: CertificateSigningRequest
metadata:
  name: clara-csr
spec:
  request: $(cat clara.csr | base64 | tr -d '\n')
  username: clara
  groups:
  - system:authenticated
  usages:
  - digital signature
  - key encipherment
  - client auth
EOF
k get csr clara-csr -o yaml
```

## Consume API with cURL

```
curl https://my-kube-api:6443/api/v1/pods --key admin.key --cert admin.crt --cacert ca.crt

kubect get pods --server my-kube-api:6443 --client-key admin.key --client-certificate admin.crt --certificate-authority ca.crt
```

```
cat << EOF > akshay-csr.yml
apiVersion: certificates.k8s.io/v1beta1
kind: CertificateSigningRequest
metadata:
  name: akshay-csr
spec:
  request: $(cat akshay.csr | base64 | tr -d '\n')
  username: akshay
  groups:
  - system:authenticated
  usages:
  - digital signature
  - key encipherment
  - client auth
EOF
```