- [ Home ](../../readme.md)   | [ Teoria ](../../arguments.md)   | [ Info Exam ](../../doc/ckad_exam_strategy.md)    | [ Home Other Exercises ](../o_exercises.md)   
--- 
###  Probe (24 esercizi)

## PR-1 — Liveness Probe HTTP

### Preparazione

```bash
kubectl delete namespace probe-test --ignore-not-found
kubectl create namespace probe-test
```

### Task

Crea il Deployment `web-liveness` nel namespace `probe-test` con:

- immagine `nginx`;
- 2 repliche;
- liveness probe HTTP GET su `/`, porta `80`;
- `initialDelaySeconds: 10`;
- `periodSeconds: 5`.

### Validazione

```bash
kubectl get deployment,pod -n probe-test
kubectl describe pod -n probe-test -l app=web-liveness | grep -A4 Liveness
```

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-liveness
  namespace: probe-test
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web-liveness
  template:
    metadata:
      labels:
        app: web-liveness
    spec:
      containers:
      - name: nginx
        image: nginx
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 10
          periodSeconds: 5
```

</details>

### Cleanup

```bash
kubectl delete namespace probe-test
```

---

## PR-2 — Readiness Probe HTTP

### Preparazione

```bash
kubectl delete namespace probe-test --ignore-not-found
kubectl create namespace probe-test
```

### Task

Crea il Deployment `api-readiness` con:

- immagine `nginx`;
- 3 repliche;
- readiness HTTP GET su `/`, porta `80`;
- `initialDelaySeconds: 5`.

Esponilo con un Service ClusterIP chiamato `api-readiness`.

### Validazione

```bash
kubectl get pod -n probe-test -l app=api-readiness
kubectl get endpoints api-readiness -n probe-test
kubectl describe pod -n probe-test -l app=api-readiness | grep -A4 Readiness
```

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-readiness
  namespace: probe-test
spec:
  replicas: 3
  selector:
    matchLabels:
      app: api-readiness
  template:
    metadata:
      labels:
        app: api-readiness
    spec:
      containers:
      - name: nginx
        image: nginx
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
---
apiVersion: v1
kind: Service
metadata:
  name: api-readiness
  namespace: probe-test
spec:
  selector:
    app: api-readiness
  ports:
  - port: 80
    targetPort: 80
```

</details>

### Cleanup

```bash
kubectl delete namespace probe-test
```

---

## PR-3 — Liveness exec

### Preparazione

```bash
kubectl delete namespace probe-test --ignore-not-found
kubectl create namespace probe-test
```

### Task

Crea il Pod `exec-liveness` con:

- immagine `busybox`;
- comando `sleep 3600`;
- liveness exec `cat /tmp/healthy`;
- `initialDelaySeconds: 5`;
- `periodSeconds: 5`.

Osserva i riavvii, poi crea `/tmp/healthy` nel container.

### Validazione

```bash
kubectl get pod exec-liveness -n probe-test -w
kubectl describe pod exec-liveness -n probe-test
kubectl exec exec-liveness -n probe-test -- touch /tmp/healthy
```

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: exec-liveness
  namespace: probe-test
spec:
  containers:
  - name: app
    image: busybox
    command: ["sh", "-c", "sleep 3600"]
    livenessProbe:
      exec:
        command: ["sh", "-c", "cat /tmp/healthy"]
      initialDelaySeconds: 5
      periodSeconds: 5
```

</details>

### Cleanup

```bash
kubectl delete namespace probe-test
```

---

## PR-4 — Readiness e Liveness combinate

### Preparazione

```bash
kubectl delete namespace probe-test --ignore-not-found
kubectl create namespace probe-test
```

### Task

Crea il Deployment `combined-probes` con immagine `nginx`.

Configura:

- liveness HTTP su `/`, porta `80`;
- readiness HTTP su `/`, porta `80`;
- readiness con `initialDelaySeconds: 15`.

### Validazione

```bash
kubectl get pods -n probe-test -l app=combined-probes -w
kubectl describe pod -n probe-test -l app=combined-probes | grep -A5 -E 'Liveness|Readiness'
```

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: combined-probes
  namespace: probe-test
spec:
  selector:
    matchLabels:
      app: combined-probes
  template:
    metadata:
      labels:
        app: combined-probes
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
          initialDelaySeconds: 15
```

</details>

### Cleanup

```bash
kubectl delete namespace probe-test
```

---

## PR-5 — Startup Probe

### Preparazione

```bash
kubectl delete namespace probe-test --ignore-not-found
kubectl create namespace probe-test
```

### Task

Crea il Deployment `startup-app` con immagine `nginx` e startup probe:

- HTTP GET `/`;
- porta `80`;
- `failureThreshold: 30`;
- `periodSeconds: 5`.

### Validazione

```bash
kubectl describe pod -n probe-test -l app=startup-app | grep -A5 Startup
kubectl get pod -n probe-test -l app=startup-app
```

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: startup-app
  namespace: probe-test
spec:
  selector:
    matchLabels:
      app: startup-app
  template:
    metadata:
      labels:
        app: startup-app
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
```

</details>

### Cleanup

```bash
kubectl delete namespace probe-test
```

---

## PR-6 — Debug di una liveness errata

### Preparazione

```bash
kubectl delete namespace probe-test --ignore-not-found
kubectl create namespace probe-test
cat > broken-probe.yaml <<'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: broken-probe
  namespace: probe-test
spec:
  selector:
    matchLabels:
      app: broken-probe
  template:
    metadata:
      labels:
        app: broken-probe
    spec:
      containers:
      - name: nginx
        image: nginx
        livenessProbe:
          httpGet:
            path: /wrongpath
            port: 80
          initialDelaySeconds: 3
          periodSeconds: 3
EOF
kubectl apply -f broken-probe.yaml
```

### Task

Il Deployment `broken-probe` riavvia continuamente il container.

Individua la causa e correggi la probe senza ricreare il namespace.

### Validazione

```bash
kubectl get pods -n probe-test
kubectl describe pod -n probe-test -l app=broken-probe
kubectl get events -n probe-test --sort-by=.metadata.creationTimestamp
```

<details>
<summary>Soluzione</summary>

```bash
kubectl patch deployment broken-probe -n probe-test --type='json'   -p='[{"op":"replace","path":"/spec/template/spec/containers/0/livenessProbe/httpGet/path","value":"/"}]'
```

</details>

### Cleanup

```bash
kubectl delete namespace probe-test
rm -f broken-probe.yaml
```

---

## PR-7 — Readiness exec con file

### Preparazione

```bash
kubectl delete namespace probe-test --ignore-not-found
kubectl create namespace probe-test
```

### Task

Crea il Pod `exec-readiness` con:

- immagine `busybox`;
- comando `sleep 3600`;
- readiness exec `test -f /tmp/ready`;
- `initialDelaySeconds: 5`;
- `periodSeconds: 3`.

Il Pod deve diventare Ready solo dopo la creazione del file.

### Validazione

```bash
kubectl get pod exec-readiness -n probe-test -w
kubectl exec exec-readiness -n probe-test -- touch /tmp/ready
```

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: exec-readiness
  namespace: probe-test
spec:
  containers:
  - name: app
    image: busybox
    command: ["sh", "-c", "sleep 3600"]
    readinessProbe:
      exec:
        command: ["sh", "-c", "test -f /tmp/ready"]
      initialDelaySeconds: 5
      periodSeconds: 3
```

</details>

### Cleanup

```bash
kubectl delete namespace probe-test
```

---

## PR-8 — Liveness TCP

### Preparazione

```bash
kubectl delete namespace probe-test --ignore-not-found
kubectl create namespace probe-test
```

### Task

Crea il Deployment `tcp-probe-app` con:

- immagine `nginx`;
- liveness TCP sulla porta `80`;
- `initialDelaySeconds: 5`;
- `periodSeconds: 5`.

### Validazione

```bash
kubectl describe pod -n probe-test -l app=tcp-probe-app | grep -A4 Liveness
kubectl get pod -n probe-test -l app=tcp-probe-app
```

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: tcp-probe-app
  namespace: probe-test
spec:
  selector:
    matchLabels:
      app: tcp-probe-app
  template:
    metadata:
      labels:
        app: tcp-probe-app
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

### Cleanup

```bash
kubectl delete namespace probe-test
```

---

## PR-9 — Startup e Liveness

### Preparazione

```bash
kubectl delete namespace probe-test --ignore-not-found
kubectl create namespace probe-test
```

### Task

Crea il Deployment `slow-app` con immagine `nginx`.

Configura:

- startup HTTP su `/`, porta `80`, `failureThreshold: 30`, `periodSeconds: 5`;
- liveness HTTP su `/`, porta `80`, `periodSeconds: 5`.

Non impostare un lungo initial delay sulla liveness: la startup probe deve proteggerla.

### Validazione

```bash
kubectl describe pod -n probe-test -l app=slow-app | grep -A5 -E 'Startup|Liveness'
kubectl get pod -n probe-test -l app=slow-app
```

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: slow-app
  namespace: probe-test
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
          periodSeconds: 5
```

</details>

### Cleanup

```bash
kubectl delete namespace probe-test
```

---

## PR-10 — Timeout e FailureThreshold

### Preparazione

```bash
kubectl delete namespace probe-test --ignore-not-found
kubectl create namespace probe-test
```

### Task

Crea il Pod `probe-timeout` con immagine `nginx` e liveness HTTP:

- path `/`;
- porta `80`;
- `timeoutSeconds: 2`;
- `failureThreshold: 3`;
- `periodSeconds: 5`.

### Validazione

```bash
kubectl describe pod probe-timeout -n probe-test | grep -A5 Liveness
```

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: probe-timeout
  namespace: probe-test
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

### Cleanup

```bash
kubectl delete namespace probe-test
```

---

## PR-11 — Debug della porta sbagliata

### Preparazione

```bash
kubectl delete namespace probe-test --ignore-not-found
kubectl create namespace probe-test
cat > wrong-port-probe.yaml <<'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: wrong-port-probe
  namespace: probe-test
spec:
  containers:
  - name: nginx
    image: nginx
    livenessProbe:
      httpGet:
        path: /
        port: 8080
      initialDelaySeconds: 3
      periodSeconds: 3
EOF
kubectl apply -f wrong-port-probe.yaml
```

### Task

Il Pod `wrong-port-probe` viene riavviato.

Individua il problema e correggi la porta della liveness probe.

### Validazione

```bash
kubectl describe pod wrong-port-probe -n probe-test
kubectl get pod wrong-port-probe -n probe-test -w
```

<details>
<summary>Soluzione</summary>

```bash
kubectl get pod wrong-port-probe -n probe-test -o yaml > fixed.yaml
# Modifica port: 8080 in port: 80, elimina i campi runtime e ricrea il Pod.
kubectl replace --force -f fixed.yaml
```

</details>

### Cleanup

```bash
kubectl delete namespace probe-test
rm -f wrong-port-probe.yaml fixed.yaml
```

---

## PR-12 — Pod multi-container

### Preparazione

```bash
kubectl delete namespace probe-test --ignore-not-found
kubectl create namespace probe-test
```

### Task

Crea il Pod `multi-probe` con:

- container `web`, immagine `nginx`, readiness HTTP su `/`:80;
- container `worker`, immagine `busybox`, comando `sleep 3600`;
- liveness exec sul worker con comando `echo ok`.

### Validazione

```bash
kubectl get pod multi-probe -n probe-test
kubectl describe pod multi-probe -n probe-test | grep -A4 -E 'Liveness|Readiness'
```

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-probe
  namespace: probe-test
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
    command: ["sh", "-c", "sleep 3600"]
    livenessProbe:
      exec:
        command: ["sh", "-c", "echo ok"]
```

</details>

### Cleanup

```bash
kubectl delete namespace probe-test
```

---

## PR-13 — Readiness con timeoutSeconds

### Preparazione

```bash
kubectl delete namespace probe-test --ignore-not-found
kubectl create namespace probe-test
```

### Task

Crea il Deployment `ready-timeout-app` con:

- immagine `nginx`;
- 2 repliche;
- readiness HTTP su `/`:80;
- `initialDelaySeconds: 5`;
- `timeoutSeconds: 2`.

### Validazione

```bash
kubectl describe pod -n probe-test -l app=ready-timeout-app | grep -A5 Readiness
```

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ready-timeout-app
  namespace: probe-test
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

### Cleanup

```bash
kubectl delete namespace probe-test
```

---

## PR-14 — Liveness con failureThreshold alto

### Preparazione

```bash
kubectl delete namespace probe-test --ignore-not-found
kubectl create namespace probe-test
```

### Task

Crea il Pod `liveness-threshold` con immagine `nginx` e liveness HTTP:

- path `/`;
- porta `80`;
- `failureThreshold: 5`;
- `periodSeconds: 5`.

### Validazione

```bash
kubectl describe pod liveness-threshold -n probe-test | grep -A5 Liveness
```

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: liveness-threshold
  namespace: probe-test
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

### Cleanup

```bash
kubectl delete namespace probe-test
```

---

## PR-15 — Startup con initialDelaySeconds

### Preparazione

```bash
kubectl delete namespace probe-test --ignore-not-found
kubectl create namespace probe-test
```

### Task

Crea il Deployment `startup-delay-app` con immagine `nginx` e startup probe:

- HTTP GET `/`;
- porta `80`;
- `initialDelaySeconds: 10`;
- `periodSeconds: 5`.

### Validazione

```bash
kubectl describe pod -n probe-test -l app=startup-delay-app | grep -A5 Startup
```

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: startup-delay-app
  namespace: probe-test
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

### Cleanup

```bash
kubectl delete namespace probe-test
```

---

## PR-16 — Debug della readiness errata

### Preparazione

```bash
kubectl delete namespace probe-test --ignore-not-found
kubectl create namespace probe-test
cat > wrong-readiness.yaml <<'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wrong-readiness
  namespace: probe-test
spec:
  selector:
    matchLabels:
      app: wrong-readiness
  template:
    metadata:
      labels:
        app: wrong-readiness
    spec:
      containers:
      - name: nginx
        image: nginx
        readinessProbe:
          httpGet:
            path: /notfound
            port: 80
          periodSeconds: 3
EOF
kubectl apply -f wrong-readiness.yaml
```

### Task

I Pod del Deployment `wrong-readiness` restano `Running` ma non `Ready`.

Individua la causa e correggi il path.

### Validazione

```bash
kubectl get pods -n probe-test -l app=wrong-readiness
kubectl describe pod -n probe-test -l app=wrong-readiness
kubectl get endpoints -n probe-test
```

<details>
<summary>Soluzione</summary>

```bash
kubectl patch deployment wrong-readiness -n probe-test --type='json'   -p='[{"op":"replace","path":"/spec/template/spec/containers/0/readinessProbe/httpGet/path","value":"/"}]'
```

</details>

### Cleanup

```bash
kubectl delete namespace probe-test
rm -f wrong-readiness.yaml
```

---

## PR-17 — Exec probe con file creato all'avvio

### Preparazione

```bash
kubectl delete namespace probe-test --ignore-not-found
kubectl create namespace probe-test
```

### Task

Crea il Pod `exec-file-probe` con:

- immagine `busybox`;
- comando che crea `/tmp/alive` e poi esegue `sleep 3600`;
- liveness exec `cat /tmp/alive`;
- `initialDelaySeconds: 5`.

### Validazione

```bash
kubectl get pod exec-file-probe -n probe-test
kubectl describe pod exec-file-probe -n probe-test | grep -A4 Liveness
```

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: exec-file-probe
  namespace: probe-test
spec:
  containers:
  - name: app
    image: busybox
    command: ["sh", "-c", "touch /tmp/alive && sleep 3600"]
    livenessProbe:
      exec:
        command: ["sh", "-c", "cat /tmp/alive"]
      initialDelaySeconds: 5
```

</details>

### Cleanup

```bash
kubectl delete namespace probe-test
```

---

## PR-18 — Readiness TCP

### Preparazione

```bash
kubectl delete namespace probe-test --ignore-not-found
kubectl create namespace probe-test
```

### Task

Crea il Deployment `tcp-ready-app` con:

- immagine `nginx`;
- 2 repliche;
- readiness TCP sulla porta `80`;
- `initialDelaySeconds: 5`;
- `periodSeconds: 5`.

### Validazione

```bash
kubectl get pods -n probe-test -l app=tcp-ready-app
kubectl describe pod -n probe-test -l app=tcp-ready-app | grep -A4 Readiness
```

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: tcp-ready-app
  namespace: probe-test
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

### Cleanup

```bash
kubectl delete namespace probe-test
```

---

## PR-19 — Liveness con più parametri

### Preparazione

```bash
kubectl delete namespace probe-test --ignore-not-found
kubectl create namespace probe-test
```

### Task

Crea il Pod `combo-liveness` con immagine `nginx` e liveness HTTP:

- `/`;
- porta `80`;
- `initialDelaySeconds: 10`;
- `timeoutSeconds: 2`;
- `periodSeconds: 5`.

### Validazione

```bash
kubectl describe pod combo-liveness -n probe-test | grep -A6 Liveness
```

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: combo-liveness
  namespace: probe-test
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

### Cleanup

```bash
kubectl delete namespace probe-test
```

---

## PR-20 — Probe su named port

### Preparazione

```bash
kubectl delete namespace probe-test --ignore-not-found
kubectl create namespace probe-test
```

### Task

Crea il Pod `named-port-probe` con:

- immagine `nginx`;
- container port `80` chiamata `http`;
- readiness HTTP GET su `/`;
- usa `port: http`, non il numero.

### Validazione

```bash
kubectl get pod named-port-probe -n probe-test
kubectl get pod named-port-probe -n probe-test   -o jsonpath='{.spec.containers[0].readinessProbe.httpGet.port}{"
"}'
```

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: named-port-probe
  namespace: probe-test
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

### Cleanup

```bash
kubectl delete namespace probe-test
```

---

## PR-21 — Startup probe in un Pod multi-container

### Preparazione

```bash
kubectl delete namespace probe-test --ignore-not-found
kubectl create namespace probe-test
```

### Task

Crea il Pod `startup-multi` con:

- container `web`, immagine `nginx`, startup HTTP `/`:80;
- `failureThreshold: 10`;
- `periodSeconds: 5`;
- container `helper`, immagine `busybox`, comando `sleep 3600`;
- nessuna probe sul container helper.

### Validazione

```bash
kubectl get pod startup-multi -n probe-test
kubectl describe pod startup-multi -n probe-test | grep -A5 Startup
```

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: startup-multi
  namespace: probe-test
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
    command: ["sh", "-c", "sleep 3600"]
```

</details>

### Cleanup

```bash
kubectl delete namespace probe-test
```

---

## PR-22 — Readiness con failureThreshold

### Preparazione

```bash
kubectl delete namespace probe-test --ignore-not-found
kubectl create namespace probe-test
```

### Task

Crea il Deployment `ready-threshold-app` con:

- immagine `nginx`;
- una replica;
- readiness HTTP su `/`:80;
- `failureThreshold: 4`;
- `periodSeconds: 5`.

### Validazione

```bash
kubectl describe pod -n probe-test -l app=ready-threshold-app | grep -A5 Readiness
```

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ready-threshold-app
  namespace: probe-test
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

### Cleanup

```bash
kubectl delete namespace probe-test
```

---

## PR-23 — Exec readiness con comando personalizzato

### Preparazione

```bash
kubectl delete namespace probe-test --ignore-not-found
kubectl create namespace probe-test
```

### Task

Crea il Pod `custom-readiness` con:

- immagine `busybox`;
- comando `sleep 3600`;
- readiness exec `echo ready`;
- `initialDelaySeconds: 5`.

La probe deve restituire sempre successo.

### Validazione

```bash
kubectl get pod custom-readiness -n probe-test
kubectl describe pod custom-readiness -n probe-test | grep -A4 Readiness
```

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: custom-readiness
  namespace: probe-test
spec:
  containers:
  - name: app
    image: busybox
    command: ["sh", "-c", "sleep 3600"]
    readinessProbe:
      exec:
        command: ["sh", "-c", "echo ready"]
      initialDelaySeconds: 5
```

</details>

### Cleanup

```bash
kubectl delete namespace probe-test
```

---

## PR-24 — Debug completo di Liveness e Readiness

### Preparazione

```bash
kubectl delete namespace probe-test --ignore-not-found
kubectl create namespace probe-test
cat > double-broken-probe.yaml <<'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: double-broken-probe
  namespace: probe-test
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
          initialDelaySeconds: 3
          periodSeconds: 3
        readinessProbe:
          httpGet:
            path: /
            port: 8080
          periodSeconds: 3
EOF
kubectl apply -f double-broken-probe.yaml
```

### Task

Il Deployment presenta due problemi distinti:

- il container viene riavviato;
- il Pod non diventa Ready.

Individua e correggi entrambe le probe.

### Validazione

```bash
kubectl get pod -n probe-test -l app=double-broken-probe
kubectl describe pod -n probe-test -l app=double-broken-probe
kubectl get events -n probe-test --sort-by=.metadata.creationTimestamp
```

<details>
<summary>Soluzione</summary>

```bash
kubectl patch deployment double-broken-probe -n probe-test --type='json'   -p='[
    {"op":"replace","path":"/spec/template/spec/containers/0/livenessProbe/httpGet/path","value":"/"},
    {"op":"replace","path":"/spec/template/spec/containers/0/readinessProbe/httpGet/port","value":80}
  ]'
```

</details>

### Cleanup

```bash
kubectl delete namespace probe-test
rm -f double-broken-probe.yaml
```

---

# Comandi rapidi per il debug

```bash
kubectl get pods -n probe-test -w
kubectl describe pod <pod> -n probe-test
kubectl get events -n probe-test --sort-by=.metadata.creationTimestamp
kubectl logs <pod> -n probe-test
kubectl logs <pod> -n probe-test --previous
kubectl get pod <pod> -n probe-test -o yaml
kubectl get endpoints -n probe-test
```

## Effetti delle probe

| Probe | Quando fallisce |
|---|---|
| `livenessProbe` | Kubernetes riavvia il container dopo il numero configurato di fallimenti |
| `readinessProbe` | Il Pod resta in esecuzione, ma viene marcato non Ready e rimosso dagli endpoint |
| `startupProbe` | Liveness e readiness non vengono eseguite finché la startup non ha successo |
