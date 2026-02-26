## Application Design and Build
---
### 1.1 [Core Concepts](../doc/ckad_core_concepts.md)
   - (Pod,ReplicaSet,Deployment,Namespace,Labels e Selectors)
     
### 1.2 [Multi-Container Pods](../doc/ckad_multicontainer_pods.md)
   - (Patterns, Init Container,Sidecar Container)

### 1.3 [Pod Design](../doc/ckad_pod_design.md)
   - (Workload,Job,CronJob,Rollback,Scaling)

### 1.4 [Storage (State Persistence)](../doc/ckad_storage.md)
   - (Volumes,Type of volumes,StorageClass, StatefulSet)

---

> Nota: Un `DaemonSet` è una risorsa Kubernetes che garantisce che un Pod venga eseguito su ogni nodo del cluster.

```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: nginx-daemon
spec:
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
```


     
     

     

     



     
