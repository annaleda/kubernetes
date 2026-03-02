# 📘 CKAD – Esercizi e Soluzioni con Alias / Shortcut

*(contiene esercizi CKAD principali e comandi shortcut.)*

---

## 🔹 1. Core Concepts

### 1.1 Pod Singolo
**Esercizio:** Crea un Pod `nginx-pod` con immagine nginx.

<details>
<summary>Soluzione</summary>

```bash
# Normale
kubectl run nginx-pod --image=nginx

# Shortcut
k run nginx-pod --image=nginx
```
</details>

### 1.2 Deployment e Scaling

**Esercizio:** Deployment web-deploy con 3 repliche, scala a 5.

<details> <summary>Soluzione</summary>
  
```bash
# Deployment YAML
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-deploy
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: web
        image: nginx
```
```bash
#Scalare Deployment
kubectl scale deployment web-deploy --replicas=5

#Shortcut
kd scale web-deploy --replicas=5
```
</details>

### 1.3 Namespace

**Esercizio:** Creare namespace dev e deploy pod dentro.

<details> <summary>Soluzione</summary>

```bash
# Normale
kubectl create ns dev
kubectl run nginx-dev --image=nginx -n dev

# Shortcut
k create ns dev
k run nginx-dev --image=nginx -n dev
```

</details>

## 🔹 2. Configuration

### 2.1 ConfigMap

**Esercizio:** Creare ConfigMap app-config con variabile ENV=prod.

<details> <summary>Soluzione</summary>

```bash  
# Normale
kubectl create configmap app-config --from-literal=ENV=prod

# Shortcut
kdry "create configmap app-config --from-literal=ENV=prod"
```

</details>

### 2.2 Secret

**Esercizio:** Creare Secret db-secret con password=xyz.

<details> <summary>Soluzione</summary>

```bash
# Normale
kubectl create secret generic db-secret --from-literal=password=xyz

# Shortcut
kdry "create secret generic db-secret --from-literal=password=xyz"
```

</details>

### 2.3 Montaggio ConfigMap / Secret

**Esercizio:** Montare ConfigMap e Secret creati nei precenti esercizi (app-config, db-secret) come volume in un Pod.
  - volumeMounts:
      - name: config-volume
      - mountPath: /etc/config
      - name: secret-volume
      - mountPath: /etc/secret
  - volumes:
     - name: config-volume 
     - name: secret-volume
    
<details> <summary>Soluzione</summary>
  
```bash  
k run mount-pod --image=nginx --dry-run=client -o yaml > pod2.yaml

vi pod2.yaml

apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: mount-pod
  name: mount-pod
spec:
  containers:
  - image: nginx
    name: mount-pod
    resources: {}
    volumeMounts:
      - name: config-volume
        mountPath: /etc/config
      - name: secret-volume
        mountPath: /etc/secret
  dnsPolicy: ClusterFirst
  restartPolicy: Always
  volumes:
  - name: config-volume
    configMap:
      name: app-config
  - name: secret-volume
    secret:
      secretName: db-secret
status: {}


k apply -f pod2.yaml

```

</details>

### 2.4 Resource Requests & Limits

**Esercizio:** Definire requests e limits su un Pod.
  - requests:
        - memory: 64Mi
        - cpu: 250m
  - limits:
        - memory: 128Mi
        - cpu: 500m

<details> <summary>Soluzione</summary>
  
```bash  
 k run resource-pod --image=nginx --dry-run=client -o yaml > pod3.yaml

 vi pod3.yaml

apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: resource-pod
  name: resource-pod
spec:
  containers:
  - image: nginx
    name: resource-pod
    resources:
      requests:
        memory: 64Mi
        cpu: 250m
      limits:
        memory: 128Mi
        cpu: 500m
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}

k apply -f pod3.yaml
```
</details>


## 3. Multi-Container Pods

### 3.1 Sidecar + InitContainer

**Esercizio:** Pod con nginx + busybox sidecar, initContainer crea file.

  - initContainer:
    - name: init-myfile
    - image: busybox
    - command: sh' -c echo hello > /data/message' 
    - mounts:
      - name: shared-data 
      - path: /data
  - container1:
    - name: nginx
    - image: nginx
    - mounts:
      - name: shared-data 
      - path: /usr/share/nginx/html  
  - container2:  
    - name: busybox
    - image: busybox
    - command: sh -c tail -f /dev/null
    - mounts:
      - name: shared-data 
      - path: /data  
  volume condiviso emptyDir:
  - name: shared-data

<details> <summary>Soluzione</summary>

```bash

k run sidecar-example --image=busybox --dry-run=client -o yaml > pod4.yaml

vi pod4.yaml
 
apiVersion: v1
kind: Pod
metadata:
  name: sidecar-example
spec:
  initContainers:
  - name: init-myfile
    image: busybox
    command: ['sh', '-c', 'echo hello > /data/message']
    volumeMounts:
    - name: shared-data
      mountPath: /data
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - name: shared-data
      mountPath: /usr/share/nginx/html
  - name: busybox
    image: busybox
    command: ['sh', '-c', 'tail -f /dev/null']
    volumeMounts:
    - name: shared-data
      mountPath: /data
  volumes:
  - name: shared-data
    emptyDir: {}

k apply -f pod4.yam
```
  
</details>

## 🔹 4. Services & Networking

### 4.1 ClusterIP & NodePort

**Esercizio:** Esporre deployment web-deploy1 come ClusterIP (port:80) e web-deploy2 come NodePort (port:80, nodePort:30097).

<details> <summary>Soluzione</summary>

```bash
k create deploy web-deploy --replicas=5 --image=nginx
 
# ClusterIP
kubectl expose deployment web-deploy --type=ClusterIP --port=80 --dry-run=client -o yaml > svc1.yaml
ks expose deployment web-deploy --type=ClusterIP --port=80 --dry-run=client  -o yaml > svc1.yaml

k apply -f svc1.yaml

# NodePort
kubectl expose deployment web-deploy --type=NodePort --port=80 --target-port=30097 --dry-run=client -o yaml > svc2.yaml
ks expose deployment web-deploy --type=NodePort --port=80  --target-port=30097 --dry-run=client  -o yaml > svc2.yaml

vi svc2.yaml

apiVersion: v1
kind: Service
metadata:
  creationTimestamp: "2026-03-02T21:53:26Z"
  labels:
    app: web-deploy
  name: web-deploy
  namespace: default
  resourceVersion: "292484"
  uid: 028e34b1-c365-4798-9d6d-95c7f418d212
spec:
  clusterIP: 10.96.209.244
  clusterIPs:
  - 10.96.209.244
  externalTrafficPolicy: Cluster
  internalTrafficPolicy: Cluster
  ipFamilies:
  - IPv4
  ipFamilyPolicy: SingleStack
  ports:
  - nodePort: 30097
    port: 80
    protocol: TCP
  selector:
    app: web-deploy
  sessionAffinity: None
  type: NodePort
status:
  loadBalancer: {}

k apply -f svc2.yaml



```

</details>

### 4.2 Ingress

**Esercizio:** Creare regole HTTP.

<details> <summary>Soluzione</summary>

```bash
k expose deployment web-deploy --name=test --dry-run=client --port=80 -o yaml > svc3.yaml
k apply -f svc3.yaml

vi ingress.yaml


apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: minimal-ingress
spec:
  ingressClassName: nginx-example
  rules:
    - http:
        paths:
        - path: /test
          pathType: Prefix
          backend:
            service:
              name: test
              port:
                number: 80


kubectl apply -f ingress.yaml
kapply ingress.yaml
```

</details>

## 🔹 5. Storage

### 5.1 emptyDir

**Esercizio:** Pod con volume temporaneo.

<details> <summary>Soluzione</summary>

```bash   
volumes:
- name: temp-storage
  emptyDir: {}
  
```
  
</details>

### 5.2 PVC + Mount

**Esercizio:** PVC 1Gi montato in un Pod.

<details> <summary>Soluzione</summary>
  
```bash   
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi

volumes:
- name: my-storage
  persistentVolumeClaim:
    claimName: my-pvc
volumeMounts:
- name: my-storage
  mountPath: /data

# Shortcut
kdry "create pvc my-pvc --storage=1Gi --access-modes=ReadWriteOnce" > pvc.yaml
kapply pvc.yaml

```
</details>

## 🔹 6. Observability

### 6.1 Liveness / Readiness / Startup

**Esercizio:** Aggiungi probe su nginx.

<details> <summary>Soluzione</summary>

```bash   
livenessProbe:
  httpGet:
    path: /
    port: 80
  initialDelaySeconds: 5
  periodSeconds: 10

readinessProbe:
  httpGet:
    path: /
    port: 80
  initialDelaySeconds: 5
  periodSeconds: 5

startupProbe:
  httpGet:
    path: /
    port: 80
  failureThreshold: 30
  periodSeconds: 10
```

</details>

### 6.2 Logs / Events

**Esercizio:** Vedere logs e eventi.

<details> <summary>Soluzione</summary>

```bash   
kubectl logs nginx-pod
kubectl get events

# Shortcut
kp logs nginx-pod
k get events
```

</details>

## 🔹 7. Pod Design

### 7.1 Job

**Esercizio:** Job stampa "Hello CKAD".

<details> <summary>Soluzione</summary>

```bash   
apiVersion: batch/v1
kind: Job
metadata:
  name: hello-job
spec:
  backoffLimit: 3
  template:
    spec:
      restartPolicy: Never
      containers:
      - name: hello
        image: busybox
        command: ["echo", "Hello CKAD"]

# Shortcut
kdry "create job hello-job --image=busybox -- date"
```

</details>

### 7.2 CronJob

**Esercizio:** CronJob ogni minuto.

<details> <summary>Soluzione</summary>

```bash   
apiVersion: batch/v1
kind: CronJob
metadata:
  name: hello-cron
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          restartPolicy: OnFailure
          containers:
          - name: hello
            image: busybox
            command: ["echo", "Hello CKAD"]

# Shortcut
kdry "create cronjob hello-cron --image=busybox --schedule='*/1 * * * *'"
```
</details>

## 🔹 8. Security
### 8.1 ServiceAccount

**Esercizio:** Creare SA app-sa.

<details> <summary>Soluzione</summary>

```bash   
kubectl create serviceaccount app-sa
kdry "create sa app-sa"
```
</details>

### 8.2 Role / RoleBinding

**Esercizio:** Role pod-reader e binding a app-sa.

<details> <summary>Soluzione</summary>

```bash   
kdry "create role pod-reader --verb=get,list,watch --resource=pods"
kdry "create rolebinding read-pods --role=pod-reader --serviceaccount=default:app-sa"
```

</details>

### 8.3 SecurityContext

**Esercizio:** Pod non-root.

<details> <summary>Soluzione</summary>

```bash  
securityContext:
  runAsNonRoot: true
  runAsUser: 1000
  allowPrivilegeEscalation: false
  readOnlyRootFilesystem: true
```
</details>
