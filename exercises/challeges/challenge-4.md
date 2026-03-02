
## Challenge 4 – Redis Cluster Stateful Architecture Solution

- Namespace

Namespace non specificato → crea redis namespace.
```
kubectl create ns redis
```
---

- Headless Service – redis-cluster-service

  - StatefulSet cluster networking.

Specification

| Field     | Value                 |
| --------- | --------------------- |
| Name      | redis-cluster-service |
| Namespace | redis                 |
| Type      | ClusterIP (default)   |
| Selector  | app=redis-cluster     |
| Ports     | client=6379           |
| Ports     | gossip=16379          |


Create Service YAML
```
kubectl create service clusterip redis-cluster-service \
--tcp=6379:6379 \
-n redis \
--dry-run=client -o yaml > redis-service.yaml
```
Edit file:

```
apiVersion: v1
kind: Service
metadata:
  name: redis-cluster-service
  namespace: redis
spec:
  clusterIP: None
  selector:
    app: redis-cluster
  ports:
  - name: client
    port: 6379
    targetPort: 6379

  - name: gossip
    port: 16379
    targetPort: 16379
```
Apply:
```
kubectl apply -f redis-service.yaml
```

---
- StatefulSet – redis-cluster 

Specification


| Field          | Value                                              |
| -------------- | -------------------------------------------------- |
| Name           | redis-cluster                                      |
| Namespace      | redis                                              |
| Replicas       | 6                                                  |
| Image          | redis:5.0.1-alpine                                 |
| Label          | app=redis-cluster                                  |
| Container Name | redis                                              |
| Command        | /conf/update-node.sh redis-server /conf/redis.conf |


Create StatefulSet YAML
```
kubectl create statefulset redis-cluster \
--image=redis:5.0.1-alpine \
--replicas=6 \
-n redis \
--dry-run=client -o yaml > redis-statefulset.yaml
```
Edit YAML:
```
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: redis-cluster
  namespace: redis

spec:
  serviceName: redis-cluster-service
  replicas: 6

  selector:
    matchLabels:
      app: redis-cluster

  template:
    metadata:
      labels:
        app: redis-cluster

    spec:
      containers:
      - name: redis
        image: redis:5.0.1-alpine

        command:
        - /conf/update-node.sh
        - redis-server
        - /conf/redis.conf

        env:
        - name: POD_IP
          valueFrom:
            fieldRef:
              fieldPath: status.podIP

        ports:
        - containerPort: 6379
          name: client

        - containerPort: 16379
          name: gossip

        volumeMounts:
        - name: conf
          mountPath: /conf
          readOnly: false

        - name: data
          mountPath: /data

      volumes:
      - name: conf
        configMap:
          name: redis-cluster-configmap
          defaultMode: 0755

  volumeClaimTemplates:
  - metadata:
      name: data

    spec:
      accessModes:
      - ReadWriteOnce

      resources:
        requests:
          storage: 1Gi
```
Apply:
```
kubectl apply -f redis-statefulset.yaml
```
---

- PersistentVolumes (6 PV)

Worker node directory creation:
```
mkdir -p /redis01 /redis02 /redis03 /redis04 /redis05 /redis06
```
   - PV Template (replicate 6 times)
Example PV redis01
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: redis01

spec:
  capacity:
    storage: 1Gi

  accessModes:
  - ReadWriteOnce

  hostPath:
    path: /redis01
```
---

Repeat changing:
```
name
path
```
for:
```
redis02 → /redis02
redis03 → /redis03
redis04 → /redis04
redis05 → /redis05
redis06 → /redis06
```
Apply PVs:
```
kubectl apply -f pv.yaml
```

---

- Cluster Initialization
  
Dopo che i pod sono Running:

```
kubectl exec -it redis-cluster-0 -- redis-cli \
--cluster create \
--cluster-replicas 1 \
$(kubectl get pods -l app=redis-cluster -o jsonpath='{range.items[*]}{.status.podIP}:6379 {end}')
```
---

- Verification Checklist
Pods
```
kubectl get pods -n redis
```
Expected:
 - redis-cluster-0 → redis-cluster-5 Running
StatefulSet
```
kubectl get sts -n redis
```
PV
```
kubectl get pv
```
PVC
```
kubectl get pvc -n redis
```
---
