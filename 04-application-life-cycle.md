# Application Life Cycle

## Rolling Update and Rollbacks

```
kubectl rollout status deployment/myapp-deployment
kubectl rollout history deployment/myapp-deployment
```

Deployment Strategy
```
Recreate > all down and all up
Rolling > one to one
```

When update the image, new replicaset is created.

```yaml
vim deployment-definition.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deployment
  labels:
    app: myapp
    type: front-end
spec:
  template:
    metadata:
      name: myapp-pod
      labels:
        app: myapp
        type: front-end
    spec:
      containers:
      - name: nginx-container
        image: nginx:1.7.1
    replicas: 3
    selector:
      matchLabels:
        type: front-end

kubectl apply -f deployment-definition.yaml
kubectl set image deployment/myapp-deployment 
kubectl rollout undo deployment/myapp-deployment
kubectl get deploy -o yaml | grep -i strategy -A2
kubectl set image deployment frontend simple-webapp=kodekloud/webapp-color:v2 --all
```

## Commands and arguments

Dockerfile
```
FROM ubuntu
CMD sleep 5


CMD command param
CMD ["command", "param"]


docker run sleeper
```

```
docker run sleeper sleep 10
> execute sleep 10

docker run sleeper
> execute sleep 5
```

if the dockerfile are write without arguments

```
FROM ubuntu
CMD sleep

docker run sleeper 10
> execute sleep 10

docker run sleeper
> ERROR don't execute anything
```

```
FROM ubuntu
ENTRYPOINT sleep
CMD 5

docker run sleeper
> execute sleep 5

docker run sleeper 10
> execute sleep 10

docker run sleeper --entrypoint ps aux
> execute ps aux
```

## Kubernetes commands and arguments

```yaml
vim ubuntu-sleeper-2.yaml
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper-2
spec:
  containers:
  - name: ubuntu
    image: ubuntu
    command:
    - sleep
    - "5000"

kubectl apply -f ubuntu-sleeper-2.yaml

> execute sleeper2.0 20 and override ENTRYPOINT and CMD in Dockerfile
```

```
controlplane $ cat /root/webapp-color
cat: /root/webapp-color: Is a directory
controlplane $ cat /root/webapp-color/Dockerfile
FROM python:3.6-alpine

RUN pip install flask

COPY . /opt/

EXPOSE 8080

WORKDIR /opt

ENTRYPOINT ["python", "app.py"]controlplane $ cat /root/webapp-color/Dockerfile2
FROM python:3.6-alpine

RUN pip install flask

COPY . /opt/

EXPOSE 8080

WORKDIR /opt

ENTRYPOINT ["python", "app.py"]

CMD ["--color", "red"]controlplane $ cat webapp-color-2/
Dockerfile2            webapp-color-pod.yaml
controlplane $ cat webapp-color-2/
Dockerfile2            webapp-color-pod.yaml
controlplane $ cat webapp-color-2/Dockerfile2
FROM python:3.6-alpine

RUN pip install flask

COPY . /opt/

EXPOSE 8080

WORKDIR /opt

ENTRYPOINT ["python", "app.py"]

CMD ["--color", "red"]
controlplane $ cat webapp-color-2/webapp-color-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp-green
  labels:
      name: webapp-green
spec:
  containers:
  - name: simple-webapp
    image: kodekloud/webapp-color
    command: ["--color","green"]

controlplane $
```

## Var ENV

```yaml
$ cat webapp-color-2/webapp-color-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
  labels:
      name: simple-webapp-color
spec:
  containers:
  - name: simple-webapp
    image: kodekloud/webapp-color
    env:
    - name: APP_COLOR
      value: pink

kubectl set env pods --all --list


```

## Config Maps

Imperative
```
kubectl create configmap <config-map-name> --from-literal=<key>=<value>
kubectl create configmap simple-app-color-config --from-literal=APP_COLOR=blue

kubectl create configmap <config-map-name> --from-file=<path-to-file>
kubectl create configmap simple-app-color-config --from-file=simple-app-config-file

cat simple-app-config-file
APP_COLOR=red
APP_MODE=prod
```

Declarative
```
cat app-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_COLOR=red
  APP_MODE=prod


kubectl apply -f app-config.yaml
```

POD mount configmap with ENV
```yaml
$ cat simple-webapp-color.yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
  labels:
      name: simple-webapp-color
spec:
  containers:
  - name: simple-webapp
    image: kodekloud/webapp-color
    envFrom:
      - configMapRef:
        name: app-config 
```

POD mount configmap with single ENV
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
  labels:
      name: simple-webapp-color
spec:
  containers:
  - name: simple-webapp
    image: kodekloud/webapp-color
    env:
      - name: APP_COLOR
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: APP_COLOR
```

POD mount configmap with volume
```yaml
volume:
  - name: app-config
    configMap:
      name: app-config


kubectl explain pod --recursive | grep envFrom -A3
kubectl create configmap webapp-config-map --from-literal=APP_COLOR=darkblue
kubectl set env pods --all --list
```

## Secrets

Python app

```python
import os
from flask import Flask

app = Flask(__name__)

@app.route("/")
def main():
  mysql.connector.connect(host="mysql", database="mysql",
                          user="root", password="password")

  return.render_template('hello.html', color=fetchcolor())

if __name__ == "__main__":
  app.run(host="0.0.0.0",port="8080")
```

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  DB_Host: mysql
  DB_User: root
  DB_Password: passwd
```

```
kubectl create secret generic <config-map-name> --from-literal=<key>=<value>
kubectl create secret generic app-secret --from-literal=DB_Host=mysql 
  --fron-literal=DB_User-root
  --from-literal=DB_Password=passwd

kubectl create secret generic <config-map-name> --from-file=<path-to-file>
kubectl create secret generic app-secret --from-file=simple-app-config-file


kubectl create secret generic db-secret --from-literal=DB_Host=sql01 --from-literal=DB_User=root --from-literal=DB_Password=password123

```

VAR ENV

```yaml
$ cat simple-webapp-color.yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
  labels:
      name: simple-webapp-color
spec:
  containers:
  - name: simple-webapp
    image: kodekloud/webapp-color
    envFrom:
      - secretRef:
        name: app-secret 
```

Single VAR ENV
```yaml
    env:
      - name: DB_Host
      valueFrom:
        configMapKeyRef:
          name: app-secret
          key: DB_Host
```

Volumen
```yaml
volume:
  - name: app-secret
    secret:
      name: app-config
```

Into the pods the volume has mounted in the mountpath and the files are the key and the content of the files are the values.
```
$ ls /secrets/DB_Host
DB_host

$ cat /secrets/DB_Host
mysql
```

alias k=kubectl

## Multi Containers

Elastic Stack

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app
  namespace: elastic-stack
  labels:
    name: app
spec:
  containers:
  - name: app
    image: kodekloud/event-simulator
    volumeMounts:
    - mountPath: /log
      name: log-volume
  - name: sidecar
    image: kodekloud/filebeat-configured
    volumeMounts:
    - mountPath: /var/log/event-simulator/
      name: log-volume

  volumes:
  - name: log-volume
    hostPath:
      path: /var/log/webapp
      type: DirectoryOrCreate
apiVersion: v1
kind: Pod
metadata:
  name: elastic-search
  namespace: elastic-stack
  labels:
    name: elastic-search
spec:
  containers:
  - name: elastic-search
    image: docker.elastic.co/elasticsearch/elasticsearch:6.4.2
    ports:
    - containerPort: 9200
    - containerPort: 9300
    env:
    - name: discovery.type
      value: single-nodekind: Service
apiVersion: v1
metadata:
  name: elasticsearch
  namespace: elastic-stack
spec:
  selector:
    name: elastic-search
  type: NodePort
  ports:
  - port: 9200
    targetPort: 9200
    nodePort: 30200
    name: port1

  - port: 9300
    targetPort: 9300
    nodePort: 30300
    name: port2
apiVersion: v1
kind: Pod
metadata:
  name: kibana
  namespace: elastic-stack
  labels:
    name: kibana
spec:
  containers:
  - name: kibana
    image: kibana:6.4.2
    ports:
    - containerPort: 5601
    env:
    - name: ELASTICSEARCH_URL
      value: http://elasticsearch:9200apiVersion: v1
kind: Service
metadata:
  name: kibana
  namespace: elastic-stack
spec:
  selector:
    name: kibana
  type: NodePort
  ports:
  - port: 5601
    targetPort: 5601
    nodePort: 30601
```

Example FluentD

```yaml
$ cat fluent-example/td-agent-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: td-agent-config
data:
  td-agent.conf: |
    <match td.*.*>
      @type tdlog
      apikey YOUR_API_KEY
      auto_create_table
      buffer_type file
      buffer_path /var/log/td-agent/buffer/td
      <secondary>
        @type file
        path /var/log/td-agent/failed_records
      </secondary>
    </match>

    <match debug.**>
      @type stdout
    </match>

    <source>
      @type forward
    </source>

    <source>
      @type http
      port 8888
    </source>

    <source>
      @type debug_agent
      bind 127.0.0.1
      port 24230
    </source>

    <source>
      type tail
      path /var/log/alternatives.log
      tag *
      format syslog
      time_format %b %d %H:%M:%S
      pos_file /tmp/fluentd--1540994275.pos
    </source>

    <source>
      type tail
      path /log/app.log
      tag *
      format /^\[(?<time>[^\]]*)\] (?<type>[^ ]*) (?<message>.*)$/
      pos_file /tmp/fluentd--1540995759.pos
    </source>

    <match log.*.*>
      @type forward
      flush_interval 5s
      log_level trace
      <server>
        name fluent-ui
        host fluent-ui-service
        port 24224
      </server>
      <secondary>
        @type file
        path /tmp/forwa1rd-failed
      </secondary>
    </match>
```

## Init Containers

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox:1.28
    command: ['sh', '-c', 'until nslookup myservice; do echo waiting for myservice; sleep 2; done;']
  - name: init-mydb
    image: busybox:1.28
    command: ['sh', '-c', 'until nslookup mydb; do echo waiting for mydb; sleep 2; done;']
```

## ConfigMaps in Pods

- ENV

```yaml
envFrom:
  - configMapRef:
    name: app-config
```

- SINGLE ENV

```yaml
env:
  - name: APP_COLOR
    valueFrom:
      configMapKeyRef:
        name: app-config
        key: APP_COLOR
```

- VOLUME

```yaml
volumes:
  - name: app-config-volume
    configMap:
      name: app-config
```