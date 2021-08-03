# Mock Exam 1

## 1. Deploy a pod named nginx-pod using the nginx:alpine image.

Once done, click on the Next Question button in the top right corner of this panel. You may navigate back and forth freely between all questions. Once done with all questions, click on End Exam. Your work will be validated at the end and score shown. Good Luck!

## 2. Deploy a messaging pod using the redis:alpine image with the labels set to tier=msg.

```
kubectl run messaging --image=redis:alpine --labels=tier=msg
```

## 3. Create a namespace named apx-x9984574

```
kubectl create ns apx-x9984574
```

## 4. Get the list of nodes in JSON format and store it in a file at /opt/outputs/nodes-z3444kd9.json

```
kubectl get nodes -o json > /opt/outputs/nodes-z3444kd9.json
```

## 5. Create a service messaging-service to expose the messaging application within the cluster on port 6379.

```
kubectl expose pod messaging --name=messaging-service --port=6379 --type=ClusterIP --selector=tier=msg
```

## 6. Create a deployment named hr-web-app using the image kodekloud/webapp-color with 2 replicas

```
kubectl create deploy hr-web-app --image=kodekloud/webapp-color --replicas=2
```

## 7. Create a static pod named static-busybox on the master node that uses the busybox image and the command sleep 1000

```
kubectl run static-busybox --image=busybox --dry-run=client -o yaml --command -- sleep 1000 > /etc/kubernetes/manifests/static-busybox.yaml
```

## 8. Create a POD in the finance namespace named temp-bus with the image redis:alpine.

```
kubectl -n finance run temp-bus --image=redis:alpine
```

## 9. A new application orange is deployed. There is something wrong with it. Identify and fix the issue.

```
kubectl get pods orange -o yaml > orange.yaml
```

## 10. Expose the hr-web-app as service hr-web-app-service application on port 30082 on the nodes on the cluster. The web application listens on port 8080

```
kubectl expose deployment hr-web-app --name=hr-web-app-service --port=30082 --target-port=8080 --type=NodePort
```

## 11. Use JSON PATH query to retrieve the osImages of all the nodes and store it in a file /opt/outputs/nodes_os_x43kj56.txt. The osImages are under the nodeInfo section under status of each node.

```
kubectl get nodes -o=jsonpath='{.items[*].status.nodeInfo.osImage}' > /opt/outputs/nodes_os_x43kj56.txt
```

## 12. Create a Persistent Volume with the given specification.

```
cat << pv-analytics.yaml >> EOF
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-analytics
spec:
  capacity:
    storage: 100Mi
  accessModes:
    - ReadWriteMany
  hostPath:
    path: "/pv/data-analytics"
EOF

kubectl create -f pv-analytics.yaml
```