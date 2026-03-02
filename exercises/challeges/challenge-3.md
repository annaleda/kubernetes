
## Challenge 3 – Voting Architecture Solution
- Namespace
Name: vote
Create namespace
```
kubectl create ns vote
```
---
- Deployment – vote
 Specification

| Field     | Value                               |
| --------- | ----------------------------------- |
| Name      | vote                                |
| Namespace | vote                                |
| Image     | dockersamples/examplevotingapp_vote |
| Status    | Running                             |
 
```
kubectl create deployment vote \
--image=dockersamples/examplevotingapp_vote \
--dry-run=client -o yaml > vote-deploy.yaml
```
```
vi vote-deploy.yaml
```
YAML
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: vote
  namespace: vote
spec:
  replicas: 1
  selector:
    matchLabels:
      app: vote
  template:
    metadata:
      labels:
        app: vote
    spec:
      containers:
      - name: vote
        image: dockersamples/examplevotingapp_vote
        ports:
        - containerPort: 80
```

```
kapply vote-deploy.yaml
```
---
- Service – vote

Specification

| Field      | Value           |
| ---------- | --------------- |
| Name       | vote            |
| Port       | 8080            |
| TargetPort | 80              |
| NodePort   | 31000           |
| Type       | NodePort        |
| Endpoint   | deployment vote |

```
kubectl expose deployment vote \
--name=vote \
--port=8080 \
--target-port=80 \
--type=NodePort \
--node-port=31000 \
-n vote \
--dry-run=client -o yaml > vote-service.yaml
```
```
vi vote-service.yaml
```

YAML
```
apiVersion: v1
kind: Service
metadata:
  name: vote
  namespace: vote
spec:
  type: NodePort
  selector:
    app: vote
  ports:
  - port: 8080
    targetPort: 80
    nodePort: 31000
```

```
kapply vote-service.yaml
```
---
- Redis Deployment + EmptyDir Volume
Specification

| Field       | Value        |
| ----------- | ------------ |
| Name        | redis        |
| Image       | redis:alpine |
| Volume      | EmptyDir     |
| Volume Name | redis-data   |
| MountPath   | /data        |
| Status      | Running      |


```
kubectl create deployment redis \
--image=redis:alpine \
--dry-run=client -o yaml > redis-deploy.yaml
```

```
vi redis-deploy.yaml
```
YAML
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis
  namespace: vote
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
      - name: redis
        image: redis:alpine
        volumeMounts:
        - name: redis-data
          mountPath: /data
      volumes:
      - name: redis-data
        emptyDir: {}
```

```
kapply redis-deploy.yaml
```
---
- Worker Deployment
Specification

| Field  | Value                                 |
| ------ | ------------------------------------- |
| Name   | worker                                |
| Image  | dockersamples/examplevotingapp_worker |
| Status | Running                               |

```
kubectl create deployment worker \
--image=dockersamples/examplevotingapp_worker \
--dry-run=client -o yaml > worker-deploy.yaml
```
```
vi worker-deploy.yaml
```
YAML
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: worker
  namespace: vote
spec:
  replicas: 1
  selector:
    matchLabels:
      app: worker
  template:
    metadata:
      labels:
        app: worker
    spec:
      containers:
      - name: worker
        image: dockersamples/examplevotingapp_worker
```
```
kapply worker-deploy.yaml
```
---
- DB Deployment – PostgreSQL
Specification

| Field       | Value                           |
| ----------- | ------------------------------- |
| Name        | db                              |
| Image       | postgres:15-alpine              |
| Env         | POSTGRES_HOST_AUTH_METHOD=trust |
| Volume      | EmptyDir                        |
| Volume Name | db-data                         |
| MountPath   | /var/lib/postgresql/data        |
| Status      | Running                         |

```
kubectl create deployment db \
--image=postgres:15-alpine \
--dry-run=client -o yaml > db-deploy.yaml
```
```
vi db-deploy.yaml
```
YAML
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: db
  namespace: vote
spec:
  replicas: 1
  selector:
    matchLabels:
      app: db
  template:
    metadata:
      labels:
        app: db
    spec:
      containers:
      - name: db
        image: postgres:15-alpine
        env:
        - name: POSTGRES_HOST_AUTH_METHOD
          value: trust
        volumeMounts:
        - name: db-data
          mountPath: /var/lib/postgresql/data
      volumes:
      - name: db-data
        emptyDir: {}
```

```
kapply db-deploy.yaml
```
---
- Service – db
Specification

| Field      | Value     |
| ---------- | --------- |
| Name       | db        |
| Port       | 5432      |
| TargetPort | 5432      |
| Type       | ClusterIP |

```
kubectl expose deployment db \
--name=db \
--port=5432 \
--target-port=5432 \
--type=ClusterIP \
-n vote \
--dry-run=client -o yaml > db-service.yaml
```

```
vi db-service.yaml
```
YAML
```
apiVersion: v1
kind: Service
metadata:
  name: db
  namespace: vote
spec:
  selector:
    app: db
  ports:
  - port: 5432
    targetPort: 5432
```

```
kapply db-service.yaml
```
---
- Result Deployment

Specification

| Field  | Value                                 |
| ------ | ------------------------------------- |
| Name   | result                                |
| Image  | dockersamples/examplevotingapp_result |
| Status | Running                               |

```
kubectl create deployment result \
--image=dockersamples/examplevotingapp_result \
--dry-run=client -o yaml > result-deploy.yaml
```

```
vi result-deploy.yaml
```
YAML
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: result
  namespace: vote
spec:
  replicas: 1
  selector:
    matchLabels:
      app: result
  template:
    metadata:
      labels:
        app: result
    spec:
      containers:
      - name: result
        image: dockersamples/examplevotingapp_result
        ports:
        - containerPort: 80
```
```
kapply result-deploy.yaml
```
---
- Service – result
Specification

| Field      | Value    |
| ---------- | -------- |
| Name       | result   |
| Port       | 8081     |
| TargetPort | 80       |
| NodePort   | 31001    |
| Type       | NodePort |

```

kubectl expose deployment result \
--name=result \
--port=8081 \
--target-port=80 \
--type=NodePort \
--node-port=31001 \
-n vote \
--dry-run=client -o yaml > result-service.yaml
```

```
vi result-service.yaml
```
YAML
```
apiVersion: v1
kind: Service
metadata:
  name: result
  namespace: vote
spec:
  type: NodePort
  selector:
    app: result
  ports:
  - port: 8081
    targetPort: 80
    nodePort: 31001
```
```
kapply result-service.yaml
```
---
- Checklist finale
```
kubectl get all -n vote
```

---
