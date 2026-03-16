# Kubernetes Practice Workbook --- Solutions

------------------------------------------------------------------------

## Exercise 1

``` yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-basic
spec:
  containers:
  - name: nginx
    image: nginx
  - name: busy
    image: busybox
    command: ["sh","-c","while true; do echo hello; sleep 5; done"]
```

## Exercise 2

``` yaml
apiVersion: v1
kind: Pod
metadata:
  name: sidecar-logs
spec:
  volumes:
  - name: logs
    emptyDir: {}
  containers:
  - name: app
    image: busybox
    command: ["sh","-c","while true; do date >> /var/log/app.log; sleep 5; done"]
    volumeMounts:
    - name: logs
      mountPath: /var/log
  - name: sidecar
    image: busybox
    command: ["sh","-c","tail -f /var/log/app.log"]
    volumeMounts:
    - name: logs
      mountPath: /var/log
```

## Exercise 11 Default Deny

``` yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny
spec:
  podSelector: {}
  policyTypes:
  - Ingress
```

## Exercise 16 DNS Allow

``` yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-dns
spec:
  podSelector: {}
  policyTypes:
  - Egress
  egress:
  - ports:
    - protocol: UDP
      port: 53
    - protocol: TCP
      port: 53
```

## Exercise 23 PV

``` yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-demo
spec:
  capacity:
    storage: 1Gi
  accessModes:
  - ReadWriteOnce
  hostPath:
    path: /data/pv
```

## Exercise 24 PVC

``` yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-demo
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

## Exercise 29 ServiceAccount

``` yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: app-sa
```

## Exercise 30 Role

``` yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get","list"]
```

## Exercise 34 SecurityContext

``` yaml
apiVersion: v1
kind: Pod
metadata:
  name: secure-pod
spec:
  securityContext:
    runAsUser: 1000
  containers:
  - name: nginx
    image: nginx
```

## Exercise 40 Deployment

``` yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
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
      - name: nginx
        image: nginx
```
