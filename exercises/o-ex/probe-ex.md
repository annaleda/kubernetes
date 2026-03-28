- [ Home ](../../readme.md)   | [ Teoria ](../../arguments.md)   | [ Info Exam ](../../doc/ckad_exam_strategy.md)    | [ Home Other Exercises ](../o_exercises.md)   
--- 
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

## PR-13 — Readiness Probe con timeoutSeconds

- Deployment: `ready-timeout-app`

- Specifiche
  - Image: nginx
  - Replicas: 2

- Readiness Probe
  - HTTP GET `/`
  - Port: 80
  - initialDelaySeconds: 5
  - timeoutSeconds: 2

- Validazione
  - `kubectl describe pod` mostra timeout configurato

---

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ready-timeout-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: ready-timeout-app
  template:
    metadata:
      labels:
        app: ready-timeout-app
    spec:
      containers:
      - name: nginx
        image: nginx
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          timeoutSeconds: 2
```

</details>

---

## PR-14 — Liveness Probe con failureThreshold alto

- Pod: `liveness-threshold`

- Specifiche
  - Image: nginx

- Liveness Probe
  - HTTP GET `/`
  - Port: 80
  - failureThreshold: 5
  - periodSeconds: 5

- Obiettivo
  - Ritardare il restart fino a 5 fallimenti consecutivi

---

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: liveness-threshold
spec:
  containers:
  - name: nginx
    image: nginx
    livenessProbe:
      httpGet:
        path: /
        port: 80
      failureThreshold: 5
      periodSeconds: 5
```

</details>

---

## PR-15 — Startup Probe con initialDelaySeconds

- Deployment: `startup-delay-app`

- Specifiche
  - Image: nginx

- Startup Probe
  - HTTP GET `/`
  - Port: 80
  - initialDelaySeconds: 10
  - periodSeconds: 5

- Validazione
  - Startup probe configurata con delay iniziale

---

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: startup-delay-app
spec:
  selector:
    matchLabels:
      app: startup-delay-app
  template:
    metadata:
      labels:
        app: startup-delay-app
    spec:
      containers:
      - name: nginx
        image: nginx
        startupProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 10
          periodSeconds: 5
```

</details>

---

## PR-16 — Readiness Probe su path sbagliato (debug)

- Deployment: `wrong-readiness`
- Image: nginx

- Configurazione
  - readinessProbe HTTP su `/notfound`

- Obiettivo
  - Identificare perché i Pod non diventano Ready
  - Correggere il path

- Validazione
  - Pod Ready dopo il fix

---

<details>
<summary>Soluzione</summary>

Problema:
- nginx risponde su `/`
- readiness usa `/notfound`

Fix:

```yaml
readinessProbe:
  httpGet:
    path: /
    port: 80
```

Debug:

```sh
k describe pod <pod-name>
k get pods
```

</details>

---

## PR-17 — Exec Probe con file creato da command

- Pod: `exec-file-probe`

- Container
  - Image: busybox
  - Command:
    - crea `/tmp/alive`
    - poi `sleep 3600`

- Liveness Probe
  - exec:
    - `cat /tmp/alive`

- Validazione
  - Il Pod resta Running

---

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: exec-file-probe
spec:
  containers:
  - name: app
    image: busybox
    command:
    - sh
    - -c
    - touch /tmp/alive && sleep 3600
    livenessProbe:
      exec:
        command:
        - sh
        - -c
        - cat /tmp/alive
      initialDelaySeconds: 5
```

</details>

---

## PR-18 — TCP Readiness Probe

- Deployment: `tcp-ready-app`

- Specifiche
  - Image: nginx
  - Replicas: 2

- Readiness Probe
  - TCP socket
  - Port: 80

- Validazione
  - Readiness configurata correttamente

---

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: tcp-ready-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: tcp-ready-app
  template:
    metadata:
      labels:
        app: tcp-ready-app
    spec:
      containers:
      - name: nginx
        image: nginx
        readinessProbe:
          tcpSocket:
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
```

</details>

---

## PR-19 — Liveness + timeout + initial delay

- Pod: `combo-liveness`

- Specifiche
  - Image: nginx

- Liveness Probe
  - HTTP GET `/`
  - Port: 80
  - initialDelaySeconds: 10
  - timeoutSeconds: 2
  - periodSeconds: 5

- Validazione
  - Tutti i parametri compaiono nel describe

---

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: combo-liveness
spec:
  containers:
  - name: nginx
    image: nginx
    livenessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 10
      timeoutSeconds: 2
      periodSeconds: 5
```

</details>

---

## PR-20 — Probe su named port

- Pod: `named-port-probe`

- Container
  - Image: nginx
  - Espone porta con nome: `http`

- Readiness Probe
  - HTTP GET `/`
  - Port: `http`

- Validazione
  - Probe usa il nome porta invece del numero

---

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: named-port-probe
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - name: http
      containerPort: 80
    readinessProbe:
      httpGet:
        path: /
        port: http
```

</details>

---

## PR-21 — Multi-container con startup probe solo su uno

- Pod: `startup-multi`

- Container 1
  - nginx
  - startupProbe HTTP

- Container 2
  - busybox
  - senza probe

- Obiettivo
  - Configurare probe solo sul container web

---

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: startup-multi
spec:
  containers:
  - name: web
    image: nginx
    startupProbe:
      httpGet:
        path: /
        port: 80
      failureThreshold: 10
      periodSeconds: 5

  - name: helper
    image: busybox
    command:
    - sh
    - -c
    - sleep 3600
```

</details>

---

## PR-22 — Readiness Probe con failureThreshold

- Deployment: `ready-threshold-app`

- Specifiche
  - Image: nginx
  - Replicas: 1

- Readiness Probe
  - HTTP GET `/`
  - failureThreshold: 4
  - periodSeconds: 5

- Obiettivo
  - Controllare la soglia prima di marcare il pod come non pronto

---

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ready-threshold-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: ready-threshold-app
  template:
    metadata:
      labels:
        app: ready-threshold-app
    spec:
      containers:
      - name: nginx
        image: nginx
        readinessProbe:
          httpGet:
            path: /
            port: 80
          failureThreshold: 4
          periodSeconds: 5
```

</details>

---

## PR-23 — Exec Readiness su comando custom

- Pod: `custom-readiness`

- Container
  - Image: busybox
  - Command: sleep 3600

- Readiness Probe
  - exec:
    - `echo ready`

- Obiettivo
  - Usare una probe exec semplice che ritorna successo

- Validazione
  - Pod Ready

---

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: custom-readiness
spec:
  containers:
  - name: app
    image: busybox
    command:
    - sh
    - -c
    - sleep 3600
    readinessProbe:
      exec:
        command:
        - sh
        - -c
        - echo ready
      initialDelaySeconds: 5
```

</details>

---

## PR-24 — Debug completo: liveness e readiness entrambe errate

- Deployment: `double-broken-probe`
- Image: nginx

- Configurazione errata
  - livenessProbe su `/wrong`
  - readinessProbe su porta `8080`

- Obiettivo
  - Identificare entrambi i problemi
  - Correggere manifest

- Validazione
  - Deployment stabile
  - Pod diventano Ready

---

<details>
<summary>Soluzione</summary>

Manifest errato:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: double-broken-probe
spec:
  replicas: 1
  selector:
    matchLabels:
      app: double-broken-probe
  template:
    metadata:
      labels:
        app: double-broken-probe
    spec:
      containers:
      - name: nginx
        image: nginx
        livenessProbe:
          httpGet:
            path: /wrong
            port: 80
        readinessProbe:
          httpGet:
            path: /
            port: 8080
```

Fix:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: double-broken-probe
spec:
  replicas: 1
  selector:
    matchLabels:
      app: double-broken-probe
  template:
    metadata:
      labels:
        app: double-broken-probe
    spec:
      containers:
      - name: nginx
        image: nginx
        livenessProbe:
          httpGet:
            path: /
            port: 80
        readinessProbe:
          httpGet:
            path: /
            port: 80
```

Debug:

```sh
k describe pod <pod-name>
k get pods
```

</details>

---
