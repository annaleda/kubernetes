### Services (12 esercizi)

---

## SVC-1 — ClusterIP

- Creare Deployment `web-svc`
  - Image: nginx
  - Replicas: 3
- Creare Service `web-clusterip`
  - Type: ClusterIP
  - Port: 80
- Validazione
  - Service ha endpoint
  - `kubectl get endpoints`

<details>
<summary>Soluzione</summary>

```sh
k create deploy web-svc --image=nginx --replicas=3
k expose deploy web-svc --name web-clusterip --type=ClusterIP --port=80
k get endpoints web-clusterip
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
  - `kubectl get svc` mostra NodePort

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-nodeport
spec:
  type: NodePort
  selector:
    app: web-svc
  ports:
  - port: 81
    targetPort: 81
    nodePort: 30067
```

```sh
k apply -f svc2.yaml
k get svc web-nodeport
```

</details>

---

## SVC-3 — LoadBalancer

- Creare Service `web-lb`
  - Type: LoadBalancer
- Validazione
  - External IP assegnato se supportato dal cluster

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-lb
spec:
  type: LoadBalancer
  selector:
    app: web-svc
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
```

```sh
k apply -f web-lb.yaml
k get svc web-lb
```

</details>

---

## SVC-4 — Headless Service

- Creare Service `headless-svc`
- Specifiche
  - ClusterIP: None
- Collegare a StatefulSet
- Validazione
  - DNS restituisce IP dei singoli Pod

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: v1
kind: Service
metadata:
  name: headless-svc
spec:
  clusterIP: None
  selector:
    app: headless-stateful
  ports:
  - port: 80
    targetPort: 80
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: headless-stateful
spec:
  serviceName: headless-svc
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

```sh
k apply -f headless.yaml
k get svc headless-svc
k get pods
```

</details>

---

## SVC-5 — Service Selector Mismatch

- Creare Service con selector errato
- Obiettivo
  - Debug e correzione
- Validazione
  - Endpoints presenti dopo fix

<details>
<summary>Soluzione</summary>

Manifest errato:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-svc
spec:
  selector:
    app: wrong-label
  ports:
  - port: 80
    targetPort: 80
```

Debug:

```sh
kubectl get endpoints web-svc
kubectl get pods --show-labels
```

Fix:

```yaml
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
```

```sh
kubectl get endpoints web-svc
```

</details>

---

## SVC-6 — Port Mapping

- Deployment `multi-port-app`
  - Container espone 8080
- Service `svc-sei`
  - Port: 80
  - TargetPort: 8080
- Validazione
  - Mapping corretto

<details>
<summary>Soluzione</summary>

```sh
k create deploy multi-port-app --replicas=1 --image=nginx --port=8080
k expose deploy multi-port-app --name svc-sei --port=80 --target-port=8080
k describe svc svc-sei
```

</details>

---

## SVC-7 — Service con selector multipli

- Creare Deployment `api-svc`
  - Image: nginx
  - Labels:
    - `app=api`
    - `tier=backend`

- Creare Service `api-backend-svc`
  - Type: ClusterIP
  - Port: 80
  - Selector:
    - `app=api`
    - `tier=backend`

- Validazione
  - Il Service seleziona solo i Pod corretti

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-svc
spec:
  replicas: 2
  selector:
    matchLabels:
      app: api
      tier: backend
  template:
    metadata:
      labels:
        app: api
        tier: backend
    spec:
      containers:
      - name: nginx
        image: nginx
---
apiVersion: v1
kind: Service
metadata:
  name: api-backend-svc
spec:
  type: ClusterIP
  selector:
    app: api
    tier: backend
  ports:
  - port: 80
    targetPort: 80
```

```sh
k apply -f svc7.yaml
k get endpoints api-backend-svc
```

</details>

---

## SVC-8 — Service named targetPort

- Pod / Deployment espone una porta con nome `http`
- Creare Service `named-port-svc`
  - Port: 80
  - TargetPort: `http`

- Validazione
  - Il Service usa il nome porta corretto

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: named-port-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: named-port-app
  template:
    metadata:
      labels:
        app: named-port-app
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - name: http
          containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: named-port-svc
spec:
  selector:
    app: named-port-app
  ports:
  - port: 80
    targetPort: http
```

```sh
k apply -f svc8.yaml
k describe svc named-port-svc
```

</details>

---

## SVC-9 — Service senza selector

- Creare un Service `external-svc`
- Tipo: ClusterIP
- Nessun selector
- Porta: 80

- Obiettivo
  - Simulare un Service che punta a endpoint gestiti manualmente

- Validazione
  - Il Service esiste senza selector

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: v1
kind: Service
metadata:
  name: external-svc
spec:
  ports:
  - port: 80
    targetPort: 80
```

```sh
k apply -f external-svc.yaml
k describe svc external-svc
```

</details>

---

## SVC-10 — Debug targetPort errato

- Deployment `wrong-target-app`
  - Container espone porta 80
- Service `wrong-target-svc`
  - Port: 80
  - TargetPort: 8080

- Obiettivo
  - Identificare perché il traffico non arriva
  - Correggere il mapping

<details>
<summary>Soluzione</summary>

Manifest errato:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: wrong-target-svc
spec:
  selector:
    app: wrong-target-app
  ports:
  - port: 80
    targetPort: 8080
```

Problema:
- il container ascolta su 80, non su 8080

Fix:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: wrong-target-svc
spec:
  selector:
    app: wrong-target-app
  ports:
  - port: 80
    targetPort: 80
```

```sh
kubectl describe svc wrong-target-svc
kubectl get endpoints wrong-target-svc
```

</details>

---

## SVC-11 — Esporre Pod singolo con Service

- Pod: `single-nginx`
- Image: nginx
- Creare Service `single-nginx-svc`
  - Type: ClusterIP
  - Port: 80

- Validazione
  - Il Service espone il Pod correttamente

<details>
<summary>Soluzione</summary>

```sh
k run single-nginx --image=nginx --labels=app=single-nginx
```

```yaml
apiVersion: v1
kind: Service
metadata:
  name: single-nginx-svc
spec:
  selector:
    app: single-nginx
  ports:
  - port: 80
    targetPort: 80
```

```sh
k apply -f single-nginx-svc.yaml
k get endpoints single-nginx-svc
```

</details>

---

## SVC-12 — NodePort con targetPort diverso

- Deployment `custom-nodeport-app`
  - Image: nginx
  - ContainerPort: 8080

- Service `custom-nodeport-svc`
  - Type: NodePort
  - Port: 80
  - TargetPort: 8080
  - NodePort: 30080

- Validazione
  - `kubectl get svc` mostra NodePort corretto

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: custom-nodeport-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: custom-nodeport-app
  template:
    metadata:
      labels:
        app: custom-nodeport-app
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: custom-nodeport-svc
spec:
  type: NodePort
  selector:
    app: custom-nodeport-app
  ports:
  - port: 80
    targetPort: 8080
    nodePort: 30080
```

```sh
k apply -f svc12.yaml
k get svc custom-nodeport-svc
```

</details>

---

## Cheatsheet Service

```sh
# vedere i service
kubectl get svc

# vedere gli endpoint
kubectl get endpoints
kubectl get endpoints <service-name>

# descrivere un service
kubectl describe svc <service-name>

# creare service da deployment
kubectl expose deploy <deploy-name> --name <svc-name> --port=80

# controllare labels dei pod
kubectl get pods --show-labels
```

---

## Mini schema mentale

```text
ClusterIP    -> accesso interno al cluster
NodePort     -> accesso da IP nodo + porta alta
LoadBalancer -> accesso esterno tramite LB
Headless     -> niente VIP, DNS verso singoli Pod

selector     -> trova i Pod
port         -> porta del Service
targetPort   -> porta del container
nodePort     -> porta esposta sul nodo
```
