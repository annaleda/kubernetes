
###  Services (6 esercizi)

## SVC-1 — ClusterIP

- Creare Deployment `web-svc`  
  - Image: nginx
  - Replicas: 3
- Creare Service `web-clusterip`
  - Type: ClusterIP
  - Port: 80
- Validazione
  - Service ha endpoint
  - kubectl get endpoints
---
<details>
<summary>Soluzione</summary>
  
```
k create deploy web-svc --image=nginx --replicas=3
k expose deploy web-svc --name web-clusterip --type=ClusterIP --port=80
```
</details>

---

## SVC-2 — NodePort

- Creare Service `web-nodeport`
- Specifiche
  - Type: NodePort
  - Porta interna: 81
  - NodePort manuale: 30067
- Validazione
  - kubectl get svc mostra NodePort

---
<details>
<summary>Soluzione</summary>
  
```
k expose deploy web-svc --name web-nodeport --type=NodePort --port=81 --dry-run=client -o yaml > svc2.yaml

vi svc2.yaml

apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    app: web-svc
  name: web-nodeport
spec:
  ports:
  - port: 81
    nodePort: 30067
    targetPort: 81
  selector:
    app: web-svc
  type: NodePort
status:
  loadBalancer: {}
```
</details>

---

## SVC-3 — LoadBalancer

- Creare Service `web-lb`
  - Type: LoadBalancer
- Validazione
  - External IP assegnato (se supportato)
---
<details>
<summary>Soluzione</summary>
  
```
apiVersion: v1
kind: Service
metadata:
  name: web-lb
spec:
  type: LoadBalancer
  selector:
    env: prod
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80



```
</details>

---

### SVC-4 — Headless Service

- Creare Service `headless-svc`
- Specifiche
  - ClusterIP: None
- Collegare a StatefulSet
- Validazione
  - DNS restituisce IP dei singoli pod

---
<details>
<summary>Soluzione</summary>
  
```
k create deploy web-svc --image=nginx --replicas=3
k expose deploy web-svc --name headless-svc  --port=81 --dry-run=client -o yaml > svc4.yaml
k create stateful headless-stateful --image=nginx --replicas=2 --dry-run=client -o yaml > stateful.yaml

vi svc4.yaml

apiVersion: v1
kind: Service
metadata:
  name: headless-svc
spec:
  clusterIP: None                     # modificato
  selector:
    app: headless-stateful            # diverso
  ports:
    - port: 80
      targetPort: 80



vi stateful.yaml



apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: headless-stateful
spec:
  serviceName: headless-svc            # aggiunto
  replicas: 2
  selector:
    matchLabels:
      app: headless-stateful
  template:
    metadata:
      labels:
        app: headless-stateful
    spec:
      containers:
        - name: nginx
          image: nginx
          ports:
            - containerPort: 80


```
</details>

---

### SVC-5 — Service Selector Mismatch

- Creare Service con selector errato
- Obiettivo
  - Debug e correzione
- Validazione
  - Endpoints presenti dopo fix
---
<details>
<summary>Soluzione</summary>
  
```
apiVersion: v1
kind: Service
metadata:
  name: web-svc
spec:
  selector:
    app: wrong-label   # ERRORE
  ports:
  - port: 80
    targetPort: 80

kubectl get endpoints web-svc
kubectl get pods --show-labels

apiVersion: v1
kind: Service
metadata:
  name: web-svc
spec:
  selector:
    app: web
  ports:
  - port: 80
    targetPort: 80

kubectl get endpoints web-svc
```
</details>

---

### SVC-6 — Port Mapping

- Deployment `multi-port-app`
  - Container espone 8080
- Service  `svc-sei`
  - Port: 80
  - TargetPort: 8080
- Validazione
  - Mapping corretto
---
<details>
<summary>Soluzione</summary>
  
```
k create deploy multi-port-app --replicas=1 --image=nginx --port=8080

k expose deploy multi-port-app --name svc-sei --port=80 --target-port=8080
```
</details>

---
