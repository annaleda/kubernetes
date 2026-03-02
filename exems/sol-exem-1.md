
### Mock Exam-1
---
Deploy a pod named nginx-448839 using the nginx:alpine image.
Once done, click on the Next Question button in the top right corner of this panel. You may navigate back and forth freely between all questions. 
Once done with all questions, click on End Exam. Your work will be validated at the end and score shown.
  - Name: nginx-448839
  - Image: nginx:alpine

```
k run nginx-448839 --image=nginx:alpine

```
---
Create a namespace named apx-z993845
  - Namespace: apx-z993845

    ```
    k create ns apx-z993845
    ```
---
Create a new Deployment named httpd-frontend with 3 replicas using image httpd:2.4-alpine
  - Name: httpd-frontend
  - Replicas: 3
  - Image: httpd:2.4-alpine
    ```
    k create deploy --image=httpd:2.4-alpine --replicas=3
    ```
---
Deploy a messaging pod using the redis:alpine image with the labels set to tier=msg.
  - Pod Name: messaging
  - Image: redis:alpine
  - Labels: tier=msg

    ```
    k run messaging --image=redis:alpine --dry-run=client -o yaml > pad1.yaml
    vi pod1.yaml

    labels:
      tier: msg

    k apply -f pod1.yaml
    ```
---
A replicaset rs-d33393 is created. However the pods are not coming up. Identify and fix the issue.
Once fixed, ensure the ReplicaSet has 4 Ready replicas.
  - Replicas: 4
    ```
    k scale rs rs-d33393 --replicas=4
    ```
---
Create a service messaging-service to expose the redis deployment in the marketing namespace within the cluster on port 6379.
Use imperative commands
  - Service: messaging-service
  - Port: 6379
  - Use the right type of Service
  - Use the right labels

    ```
    k expose deploy redis --name=messaging-service --port=6379 --type=ClusterIP  -n marketing --dry-run=client -o yaml > svc1.yaml
    vi svc1.yaml

    labels:
      name: redis-pod

    k apply -f svc1.yaml
    ```
---
Update the environment variable on the pod webapp-color to use a green background.
  - Pod Name: webapp-color
  - Label Name: webapp-color
  - Env: APP_COLOR=green

    ```
    k edit pod webbapp-color

    pink -> green
    ```
---
Create a new ConfigMap named cm-3392845. Use the spec given on the below.
  - ConfigName Name: cm-3392845
  - Data: DB_NAME=SQL3322
  - Data: DB_HOST=sql322.mycompany.com
  - Data: DB_PORT=3306
    ```
    k create cm cm-3392845 --from-literal=DB_NAME=SQL3322 --from-literal=DB_HOST=sql322.mycompany.com --from-literal=DB_PORT=3306
    ```
---
Create a new Secret named db-secret-xxdf with the data given (on the below).
  - Secret Name: db-secret-xxdf
  - Secret 1: DB_Host=sql01
  - Secret 2: DB_User=root
  - Secret 3: DB_Password=password123

    ```
    k create secret generic db-secret-xxdf --from-literal=DB_HOST=sql01 --from-literal=DB_USER=root --from-literal=DB_PASSWORD=passeord123 
    ```
---
Update pod app-sec-kff3345 to run as Root user and with the SYS_TIME capability.
  - Pod Name: app-sec-kff3345
  - Image Name: ubuntu
  - SecurityContext: Capability SYS_TIME

  nel container
  
  ```
   securityContext:
      runAsUser: 0
      capabilities:
        add:
        - SYS_TIME
  ```
---
Export the logs of the e-com-1123 pod to the file /opt/outputs/e-com-1123.logs
It is in a different namespace. Identify the namespace first.
  - Task Completed

    ```
    controlplane ~ ➜  kubectl get pods -A | grep e-com-1123
    e-commerce    e-com-1123                                 1/1     Running            0          45m
    
    controlplane ~ ➜  kubectl logs e-com-1123 -n e-commerce > /opt/outputs/e-com-1123.logs
    ```
---
Create a Persistent Volume with the given specification.
  - Volume Name: pv-analytics
  - Storage: 100Mi
  - Access modes: ReadWriteMany
  - Host Path: /pv/data-analytics
```
vi pv.yaml

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
    path: /pv/data-analytics

k apply pv.yaml

```
---
Create a redis deployment using the image redis:alpine with 1 replica and label app=redis. Expose it via a ClusterIP service called redis on port 6379. Create a new Ingress Type NetworkPolicy called redis-access which allows only the pods with label access=redis to access the deployment.
  - Image: redis:alpine
  - Deployment created correctly?
  - Service created correctly?
  - Network Policy allows the correct pods?
  - Network Policy applied on the correct pods?
```
kubectl create deployment redis \
--image=redis:alpine \
--replicas=1 \
--dry-run=client -o yaml > redis-deploy.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis
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

kubectl apply -f redis-deploy.yaml

kubectl expose deployment redis \
--name=redis \
--port=6379 \
--target-port=6379 \
--type=ClusterIP \
--dry-run=client -o yaml > redis-service.yaml



kubectl apply -f redis-service.yaml



vi redis-networkpolicy.yaml


apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: redis-access
spec:
  podSelector:
    matchLabels:
      app: redis

  policyTypes:
  - Ingress

  ingress:
  - from:
    - podSelector:
        matchLabels:
          access: redis
    ports:
    - protocol: TCP
      port: 6379

kubectl apply -f redis-networkpolicy.yaml
```
---
Create a Pod called sega with two containers:
- Container 1: Name tails with image busybox and command: sleep 3600.
- Container 2: Name sonic with image nginx and Environment variable: NGINX_PORT with the value 8080.

  - Container Sonic has the correct ENV name
  - Container Sonic has the correct ENV value
  - Container tails created correctly?
---
```
vi sega-pod.yaml


apiVersion: v1
kind: Pod
metadata:
  name: sega
spec:
  containers:

  - name: tails
    image: busybox
    command: ["sleep", "3600"]

  - name: sonic
    image: nginx
    env:
    - name: NGINX_PORT
      value: "8080"


kubectl apply -f sega-pod.yaml
```
