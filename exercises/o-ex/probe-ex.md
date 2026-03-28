
###  Probe (24 esercizi)

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
 k create deploy startup-app --image=nginx --dry-run=client -o yaml > deploy5.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: startup-app
  name: startup-app
spec:
  selector:
    matchLabels:
      app: startup-app
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: startup-app
    spec:
      containers:
      - image: nginx
        name: nginx

        startupProbe:
          httpGet:
            path: /
            port: 80
          failureThreshold: 30
          periodSeconds: 5

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

 k create deploy broken-probe --image=nginx --dry-run=client -o yaml > deploy6.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: broken-probe
  name: broken-probe
spec:
  selector:
    matchLabels:
      app: broken-probe
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: broken-probe
    spec:
      containers:
      - image: nginx
        name: nginx

        livenessProbe:
          httpGet:
            path: /wrongpath
            port: 80


soluzione:
        livenessProbe:
          httpGet:
            path: /
            port: 80

```
</details>

---
## PR-7 — Readiness con comando (exec)

- Pod: `exec-readiness`
- Container
  - Image: busybox
  - Command: sleep 3600
- Readiness Probe
  - exec:
    
        test -f /tmp/ready

  - initialDelaySeconds: 5

- Obiettivo
  - Il Pod NON deve essere Ready finché il file non esiste

- Validazione
  - `kubectl get pod` mostra NOT READY
  - Dopo creazione file → READY

---

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: exec-readiness
spec:
  containers:
  - name: app
    image: busybox
    command: ["sh","-c","sleep 3600"]
    readinessProbe:
      exec:
        command:
        - sh
        - -c
        - test -f /tmp/ready
      initialDelaySeconds: 5
```

```sh
k apply -f exec-readiness.yaml
k get pod

# fix manuale
k exec -it exec-readiness -- touch /tmp/ready
```

</details>

---

## PR-8 — TCP Probe

- Deployment: `tcp-probe-app`
- Specifiche
  - Image: nginx
- Liveness Probe
  - TCP check
  - Port: 80

- Validazione
  - Probe configurata correttamente

---

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: tcp-probe-app
spec:
  selector:
    matchLabels:
      app: tcp-probe
  template:
    metadata:
      labels:
        app: tcp-probe
    spec:
      containers:
      - name: nginx
        image: nginx
        livenessProbe:
          tcpSocket:
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
```

</details>

---

## PR-9 — Startup + Liveness interplay

- Deployment: `slow-app`
- Specifiche
  - Image: nginx

- Configurazione
  - startupProbe (HTTP `/`)
  - livenessProbe (HTTP `/`)

- Obiettivo
  - Evitare restart prematuri durante startup

- Validazione
  - Nessun restart durante avvio

---

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: slow-app
spec:
  selector:
    matchLabels:
      app: slow-app
  template:
    metadata:
      labels:
        app: slow-app
    spec:
      containers:
      - name: nginx
        image: nginx

        startupProbe:
          httpGet:
            path: /
            port: 80
          failureThreshold: 30
          periodSeconds: 5

        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
```

</details>

---

## PR-10 — Timeout + FailureThreshold

- Pod: `probe-timeout`
- Image: nginx

- Configurazione
  - livenessProbe HTTP
  - timeoutSeconds: 2
  - failureThreshold: 3

- Obiettivo
  - Controllare quando il container viene riavviato

---

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: probe-timeout
spec:
  containers:
  - name: nginx
    image: nginx
    livenessProbe:
      httpGet:
        path: /
        port: 80
      timeoutSeconds: 2
      failureThreshold: 3
      periodSeconds: 5
```

</details>

---

## PR-11 — Probe su porta sbagliata (debug)

- Pod: `wrong-port-probe`
- Image: nginx

- Configurazione
  - livenessProbe su porta 8080

- Obiettivo
  - Debuggare perché il Pod restarta

---

<details>
<summary>Soluzione</summary>

Problema:
- nginx ascolta su 80
- probe usa 8080 → FAIL

Fix:

```yaml
livenessProbe:
  httpGet:
    path: /
    port: 80
```

Debug:

```sh
k describe pod wrong-port-probe
```

</details>

---

## PR-12 — Multi-container con probe

- Pod: `multi-probe`

- Container 1
  - nginx
  - readinessProbe HTTP

- Container 2
  - busybox
  - livenessProbe exec (sleep loop)

- Obiettivo
  - Usare probe diverse nello stesso Pod

---

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-probe
spec:
  containers:
  - name: web
    image: nginx
    readinessProbe:
      httpGet:
        path: /
        port: 80

  - name: worker
    image: busybox
    command: ["sh","-c","sleep 3600"]
    livenessProbe:
      exec:
        command:
        - sh
        - -c
        - echo ok
```

</details>

---
