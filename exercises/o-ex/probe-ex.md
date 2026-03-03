
###  Probe (6 esercizi)

## PR-1 — Liveness Probe HTTP

- Deployment: `web-liveness`
- Specifiche
  - Image: nginx
  - Replicas: 2
- Aggiungere Liveness Probe
  - HTTP GET
  - Path: `/`
  - Port: 80
  - initialDelaySeconds: 10
  - periodSeconds: 5
- Validazione
  - `kubectl describe pod` mostra Liveness configurata
---
<details>
<summary>Soluzione</summary>
  
```
 k create deploy web-liveness --image=nginx --replicas=2 --dry-run=client -o yaml > deploy2.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: web-liveness
  name: web-liveness
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web-liveness
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: web-liveness
    spec:
      containers:
      - image: nginx
        name: nginx
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 10
          periodSeconds: 5
```
</details>

---

## PR-2 — Readiness Probe HTTP

- Deployment: `api-readiness`
- Specifiche
  - Image: nginx
  - Replicas: 3
- Aggiungere Readiness Probe
  - HTTP GET
  - Path: `/`
  - Port: 80
  - initialDelaySeconds: 5
- Validazione
  - Pod diventano Ready
  - Service riceve endpoint solo se Ready

---
<details>
<summary>Soluzione</summary>
  
```

k create deploy api-readiness --image=nginx --replicas=3 --dry-run=client -o yaml > deploy3.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: api-readiness
  name: api-readiness
spec:
  replicas: 3
  selector:
    matchLabels:
      app: api-readiness
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: api-readiness
    spec:
      containers:
      - image: nginx
        name: nginx
        resources: {}
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
```
</details>

---

## PR-3 — Liveness con comando (exec)

- Pod: `exec-liveness`
- Container
  - Image: busybox
  - Command: sleep 3600
- Liveness Probe
  - exec:
    
        cat /tmp/healthy

  - initialDelaySeconds: 5
- Obiettivo
  - Creare manualmente il file per far passare la probe
- Validazione
  - Senza file il Pod viene riavviato
---
<details>
<summary>Soluzione</summary>
  
```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: exec-liveness
  name: exec-liveness
spec:
  containers:
  - image: busybox
    name: exec-liveness
    command:
      - sh
      - -c
      - sleep 3600
    livenessProbe:
      exec:
        command:
          - sh
          - -c
          - cat /tmp/healthy
      initialDelaySeconds: 5
  dnsPolicy: ClusterFirst
  restartPolicy: Always

```
</details>

---

### PR-4 — Readiness + Liveness combinate

- Deployment: `combined-probes`
- Specifiche
  - Image: nginx
- Liveness
  - HTTP GET `/`
- Readiness
  - HTTP GET `/`
  - initialDelaySeconds: 15
- Validazione
  - Pod Running ma non Ready nei primi 15 secondi

---
<details>
<summary>Soluzione</summary>
  
```
 k create deploy combined-probes --image=nginx --dry-run=client -o yaml > deploy4.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: combined-probes
  name: combined-probes
spec:
  selector:
    matchLabels:
      app: combined-probes
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: combined-probes
    spec:
      containers:
      - image: nginx
        name: nginx

        livenessProbe:
          httpGet:
            path: /
            port: 80

        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 15
         
```
</details>

---

### PR-5 — Startup Probe

- Deployment: `startup-app`
- Specifiche
  - Image: nginx
- Aggiungere Startup Probe
  - HTTP GET `/`
  - failureThreshold: 30
  - periodSeconds: 5
- Validazione
  - Startup protegge da restart prematuri
---
<details>
<summary>Soluzione</summary>
  
```  
```
</details>

---

### PR-6 — Debug Probe Failure

- Deployment: `broken-probe`
- Specifiche
  - Image: nginx
- Liveness
  - HTTP GET `/wrongpath`
- Obiettivo
  - Identificare motivo dei restart
  - Correggere il path
- Validazione
  - Pod stabile dopo correzione
---
<details>
<summary>Soluzione</summary>
  
```  
```
</details>

---
