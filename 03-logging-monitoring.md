# Logging & Monitoring

## Monitor Cluster Components

```
kubectl get pods
git clone http://github.com/kodekloudhub/kubernetes-metrics-server.git
cd kubernetes-metrics-server
kubectl apply -f .
kubectl top pods
kubectl top nodes
```

## Logging

```
kubectl get pods
kubectl logs webapp-1 | grep USER5
kubectl logs webapp-2 -c simple-app
```

