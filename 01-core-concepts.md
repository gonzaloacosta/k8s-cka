# Core Concepts

# Imperative

```
kubectl run --image=nginx nginx
kubectl create deployment --image=nginx nginx
kubectl expose deployment nginx --port 80
kubectl edit deployment nginx
kubectl scale deployment nginx --replicas=5
kubectl set image deployment nginx nginx=nginx:1.18
```

# Imperative form

```
kubectl create -f nginx.yaml
```

# For imperative form, the pods will be recreate with the argument --force

```
kubectl replace -f nginx.yaml
```

# force

```
kubectl replace --force -f nginx.yaml
```

# Without force the command fail

```
kubectl replace -f nginx.yaml
kubectl create -f nginx.yaml

kubectl delete -f nginx.yaml
kubectl create deployment --image=nginx nginx --dry-run > nginx-deployment.yaml
```

# Declarative form
# Create

```
kubectl apply -f nginx.yaml
```

# Update

```
kubectl apply -f nginx.yaml
```

# Exam Tips!!!! as much as possible resolve the exercise with imperative form and declarative for when you need update objects


# Exercise imperative and declarative form

```
kubectl run nginx-pod --image=nginx:alpine
kubectl run redis --image=redis:alpine --labels=tier=db
kubectl expose pod redis --port=6379 --type=ClusterIP --selector=tier=db --name=redis-service
kubectl create deployment webapp --image=kodekloud/webapp-color --replicas=3
kubectl run custom-nginx --image=nginx --port 8080
kubectl create ns dev-ns
kubectl create deployment redis-deploy --image=redis --replicas=2 -n dev-ns
kubectl run httpd --image=httpd:alpine --port 80 --expose
#kubectl expose pod httpd --name=httpd --type=ClusterIP --target-port=80
```

## POD

```
kubectl run nginx --image=nginx
kubectl run nginx --image=nginx  --dry-run=client -o yaml
```

## Deployment

```
kubectl create deployment --image=nginx nginx
kubectl create deployment nginx --image=nginx --replicas=4
kubectl scale deployment nginx --replicas=4
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml > nginx-deployment.yaml
```

## Services

```
kubectl expose pod redis --port=6379 --name redis-service --dry-run=client -o yaml
kubectl create service clusterip redis --tcp=6379:6379 --dry-run=client -o yaml 
```
