
# CKAD Practice Labs – Esercizi con Soluzioni

> Raccolta pratica in stile esame CKAD, focalizzata su:
> probes, log analysis, Jobs/CronJobs, NetworkPolicy, ConfigMap, Secret, volumes, multi-container pods, rolling updates, resource limits, security context, Service, ingress base, OCI basics e troubleshooting.
>
> Convenzioni usate:
> - namespace: `ckad-lab`
> - quando richiesto, crea i file YAML nella directory corrente
> - le soluzioni mostrano un approccio rapido da esame

---

## Indice

1. Probes: readiness + liveness
2. Probes: startupProbe
3. Analisi log di pod multi-container
4. `kubectl describe` + eventi
5. Job base
6. Job con `completions`, `parallelism`, `backoffLimit`, `activeDeadlineSeconds`, `ttlSecondsAfterFinished`
7. CronJob
8. ConfigMap come env e volume
9. Secret come env e volume
10. emptyDir tra due container
11. Sidecar logger
12. Resource requests/limits
13. SecurityContext
14. Rolling update e rollback
15. Service ClusterIP
16. NetworkPolicy ingress
17. NetworkPolicy ingress + egress
18. OCI: image, runtime, distribution
19. Troubleshooting rapido CKAD
20. Mini mock exam finale

---

## Setup iniziale

```bash
kubectl create ns ckad-lab
kubectl config set-context --current --namespace=ckad-lab
```

---

# 1) Probes: readiness + liveness

## Esercizio
Crea un Deployment `web-probe` con immagine `nginx:1.25`.
Imposta:
- 3 repliche
- readiness probe HTTP su `/` porta 80
- liveness probe HTTP su `/` porta 80
- readiness con `initialDelaySeconds: 3`, `periodSeconds: 5`
- liveness con `initialDelaySeconds: 10`, `periodSeconds: 10`

## Soluzione

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-probe
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web-probe
  template:
    metadata:
      labels:
        app: web-probe
    spec:
      containers:
      - name: nginx
        image: nginx:1.25
        ports:
        - containerPort: 80
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 3
          periodSeconds: 5
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 10
          periodSeconds: 10
```

```bash
kubectl apply -f 01-web-probe.yaml
kubectl get pods
kubectl describe pod <pod-name>
```

### Variante rapida da esame
```bash
kubectl create deployment web-probe --image=nginx:1.25 --replicas=3 --dry-run=client -o yaml > 01-web-probe.yaml
```
Poi editi il file e aggiungi le probes.

---

# 2) Probes: startupProbe

## Esercizio
Crea un Pod `slow-app` con immagine `busybox:1.36` che esegue:
```sh
sh -c 'sleep 20; touch /tmp/ready; sleep 3600'
```
Imposta:
- startupProbe `exec` che controlla `cat /tmp/ready`
- `failureThreshold: 10`
- `periodSeconds: 3`

## Soluzione

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: slow-app
spec:
  containers:
  - name: slow-app
    image: busybox:1.36
    command:
    - sh
    - -c
    - "sleep 20; touch /tmp/ready; sleep 3600"
    startupProbe:
      exec:
        command:
        - cat
        - /tmp/ready
      failureThreshold: 10
      periodSeconds: 3
```

```bash
kubectl apply -f 02-slow-app.yaml
kubectl describe pod slow-app
```

### Nota
`startupProbe` è utile per applicazioni lente ad avviarsi: finché non passa, liveness e readiness non entrano in gioco.

---

# 3) Analisi log di pod multi-container

## Esercizio
Crea un Pod `multi-log` con due container:
- `main`: `busybox:1.36`, comando `sh -c 'while true; do echo MAIN-OK; sleep 5; done'`
- `sidecar`: `busybox:1.36`, comando `sh -c 'while true; do echo SIDECAR-OK; sleep 7; done'`

Poi:
1. visualizza i log del container `main`
2. visualizza i log del container `sidecar`
3. segui in streaming i log di `main`

## Soluzione

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-log
spec:
  containers:
  - name: main
    image: busybox:1.36
    command: ["sh", "-c", "while true; do echo MAIN-OK; sleep 5; done"]
  - name: sidecar
    image: busybox:1.36
    command: ["sh", "-c", "while true; do echo SIDECAR-OK; sleep 7; done"]
```

```bash
kubectl apply -f 03-multi-log.yaml
kubectl logs multi-log -c main
kubectl logs multi-log -c sidecar
kubectl logs -f multi-log -c main
```

### Comandi chiave
```bash
kubectl logs <pod>
kubectl logs <pod> -c <container>
kubectl logs -f <pod> -c <container>
kubectl logs --previous <pod> -c <container>
```

---

# 4) `kubectl describe` + eventi

## Esercizio
Crea un Pod `bad-image` con immagine inesistente `nginx:notexist`.
Individua il problema usando:
- `kubectl get pod`
- `kubectl describe pod`
- eventi ordinati per data

## Soluzione

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: bad-image
spec:
  containers:
  - name: nginx
    image: nginx:notexist
```

```bash
kubectl apply -f 04-bad-image.yaml
kubectl get pod bad-image
kubectl describe pod bad-image
kubectl get events --sort-by=.metadata.creationTimestamp
```

### Cosa devi vedere
- stato: `ErrImagePull` / `ImagePullBackOff`
- negli eventi: errore di pull immagine

### Mindset da esame
Se qualcosa non parte:
1. `kubectl get`
2. `kubectl describe`
3. `kubectl logs`
4. `kubectl get events`

---

# 5) Job base

## Esercizio
Crea un Job `job-hello` che esegua:
```sh
echo hello-ckad; sleep 5
```
con immagine `busybox:1.36`.

## Soluzione

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: job-hello
spec:
  template:
    spec:
      restartPolicy: Never
      containers:
      - name: job-hello
        image: busybox:1.36
        command: ["sh", "-c", "echo hello-ckad; sleep 5"]
```

```bash
kubectl apply -f 05-job-hello.yaml
kubectl get jobs
kubectl get pods
kubectl logs job/job-hello
```

---

# 6) Job con tutti i parametri importanti

## Esercizio
Crea un Job `job-advanced` con:
- immagine `busybox:1.36`
- comando `sh -c 'echo start; sleep 8; echo done'`
- `completions: 4`
- `parallelism: 2`
- `backoffLimit: 1`
- `activeDeadlineSeconds: 30`
- `ttlSecondsAfterFinished: 20`

## Soluzione

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: job-advanced
spec:
  completions: 4
  parallelism: 2
  backoffLimit: 1
  activeDeadlineSeconds: 30
  ttlSecondsAfterFinished: 20
  template:
    spec:
      restartPolicy: Never
      containers:
      - name: worker
        image: busybox:1.36
        command: ["sh", "-c", "echo start; sleep 8; echo done"]
```

```bash
kubectl apply -f 06-job-advanced.yaml
kubectl get jobs
kubectl describe job job-advanced
kubectl get pods
```

## Spiegazione rapida parametri
- `completions`: quante esecuzioni completate servono
- `parallelism`: quanti pod del Job possono correre insieme
- `backoffLimit`: quanti retry dopo failure
- `activeDeadlineSeconds`: timeout totale del Job
- `ttlSecondsAfterFinished`: cleanup automatico dopo fine

### Domanda tipica da esame
> “Voglio 6 esecuzioni totali ma massimo 2 in parallelo”
- `completions: 6`
- `parallelism: 2`

---

# 7) CronJob

## Esercizio
Crea un CronJob `cj-report` che ogni 2 minuti esegua:
```sh
date; echo report-ok
```
con immagine `busybox:1.36`.
Imposta:
- `concurrencyPolicy: Forbid`
- `successfulJobsHistoryLimit: 2`
- `failedJobsHistoryLimit: 1`

## Soluzione

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: cj-report
spec:
  schedule: "*/2 * * * *"
  concurrencyPolicy: Forbid
  successfulJobsHistoryLimit: 2
  failedJobsHistoryLimit: 1
  jobTemplate:
    spec:
      template:
        spec:
          restartPolicy: Never
          containers:
          - name: reporter
            image: busybox:1.36
            command: ["sh", "-c", "date; echo report-ok"]
```

```bash
kubectl apply -f 07-cj-report.yaml
kubectl get cronjobs
kubectl get jobs
kubectl get pods
```

---

# 8) ConfigMap come env e volume

## Esercizio
Crea una ConfigMap `app-config` con:
- `APP_MODE=prod`
- `APP_COLOR=blue`

Poi crea un Pod `cm-consumer` che:
- usi la ConfigMap come variabili ambiente
- monti la ConfigMap in `/etc/config`

## Soluzione

```bash
kubectl create configmap app-config \
  --from-literal=APP_MODE=prod \
  --from-literal=APP_COLOR=blue
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: cm-consumer
spec:
  containers:
  - name: app
    image: busybox:1.36
    command: ["sh", "-c", "env; ls /etc/config; sleep 3600"]
    envFrom:
    - configMapRef:
        name: app-config
    volumeMounts:
    - name: cfg
      mountPath: /etc/config
  volumes:
  - name: cfg
    configMap:
      name: app-config
```

```bash
kubectl apply -f 08-cm-consumer.yaml
kubectl logs cm-consumer
kubectl exec -it cm-consumer -- sh
```

---

# 9) Secret come env e volume

## Esercizio
Crea un Secret `db-secret` con:
- `username=admin`
- `password=s3cr3t`

Poi crea un Pod `secret-consumer` che:
- esponga `username` e `password` come env
- monti il Secret in `/etc/secret`

## Soluzione

```bash
kubectl create secret generic db-secret \
  --from-literal=username=admin \
  --from-literal=password=s3cr3t
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-consumer
spec:
  containers:
  - name: app
    image: busybox:1.36
    command: ["sh", "-c", "env; ls /etc/secret; sleep 3600"]
    env:
    - name: DB_USER
      valueFrom:
        secretKeyRef:
          name: db-secret
          key: username
    - name: DB_PASS
      valueFrom:
        secretKeyRef:
          name: db-secret
          key: password
    volumeMounts:
    - name: s
      mountPath: /etc/secret
      readOnly: true
  volumes:
  - name: s
    secret:
      secretName: db-secret
```

```bash
kubectl apply -f 09-secret-consumer.yaml
kubectl exec -it secret-consumer -- sh
```

---

# 10) emptyDir tra due container

## Esercizio
Crea un Pod `shared-data` con due container:
- `writer` scrive ogni 5 secondi in `/data/out.txt`
- `reader` legge lo stesso file ogni 5 secondi
Usa un volume `emptyDir`.

## Soluzione

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: shared-data
spec:
  containers:
  - name: writer
    image: busybox:1.36
    command: ["sh", "-c", "while true; do date >> /data/out.txt; sleep 5; done"]
    volumeMounts:
    - name: shared
      mountPath: /data
  - name: reader
    image: busybox:1.36
    command: ["sh", "-c", "while true; do cat /data/out.txt; sleep 5; done"]
    volumeMounts:
    - name: shared
      mountPath: /data
  volumes:
  - name: shared
    emptyDir: {}
```

```bash
kubectl apply -f 10-shared-data.yaml
kubectl logs shared-data -c reader
```

---

# 11) Sidecar logger

## Esercizio
Crea un Pod `sidecar-log` con:
- container `app` che scrive righe di log in `/var/log/app/app.log`
- container `sidecar` che esegue `tail -f` di quel file
- volume condiviso `emptyDir`

## Soluzione

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sidecar-log
spec:
  containers:
  - name: app
    image: busybox:1.36
    command:
    - sh
    - -c
    - "mkdir -p /var/log/app; while true; do echo APP $(date) >> /var/log/app/app.log; sleep 5; done"
    volumeMounts:
    - name: logs
      mountPath: /var/log/app
  - name: sidecar
    image: busybox:1.36
    command:
    - sh
    - -c
    - "touch /var/log/app/app.log; tail -f /var/log/app/app.log"
    volumeMounts:
    - name: logs
      mountPath: /var/log/app
  volumes:
  - name: logs
    emptyDir: {}
```

```bash
kubectl apply -f 11-sidecar-log.yaml
kubectl logs sidecar-log -c sidecar
```

---

# 12) Resource requests/limits

## Esercizio
Crea un Pod `res-pod` con immagine `nginx:1.25` e:
- request CPU `100m`
- request memory `128Mi`
- limit CPU `200m`
- limit memory `256Mi`

## Soluzione

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: res-pod
spec:
  containers:
  - name: nginx
    image: nginx:1.25
    resources:
      requests:
        cpu: "100m"
        memory: "128Mi"
      limits:
        cpu: "200m"
        memory: "256Mi"
```

```bash
kubectl apply -f 12-res-pod.yaml
kubectl describe pod res-pod
```

### Da ricordare
- `requests`: scheduling minimo richiesto
- `limits`: massimo consentito
- se superi memory limit → possibile `OOMKilled`

---

# 13) SecurityContext

## Esercizio
Crea un Pod `secure-pod` con:
- immagine `busybox:1.36`
- comando `sleep 3600`
- eseguito come user `1000`
- `allowPrivilegeEscalation: false`
- filesystem root in sola lettura

## Soluzione

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secure-pod
spec:
  containers:
  - name: app
    image: busybox:1.36
    command: ["sh", "-c", "sleep 3600"]
    securityContext:
      runAsUser: 1000
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
```

```bash
kubectl apply -f 13-secure-pod.yaml
kubectl exec -it secure-pod -- id
```

---

# 14) Rolling update e rollback

## Esercizio
Crea un Deployment `rollapp` con `nginx:1.25`, 3 repliche.
Poi aggiorna l'immagine a `nginx:1.26` e, se serve, fai rollback.

## Soluzione

```bash
kubectl create deployment rollapp --image=nginx:1.25 --replicas=3
kubectl rollout status deployment rollapp
kubectl set image deployment/rollapp nginx=nginx:1.26
kubectl rollout status deployment rollapp
kubectl rollout history deployment rollapp
kubectl rollout undo deployment rollapp
```

### Comandi chiave
```bash
kubectl set image deployment/<name> <container>=<image>
kubectl rollout status deployment <name>
kubectl rollout history deployment <name>
kubectl rollout undo deployment <name>
```

---

# 15) Service ClusterIP

## Esercizio
Crea un Deployment `web-svc` con immagine `nginx:1.25` e 2 repliche.
Esporlo con un Service `ClusterIP` sulla porta 80.

## Soluzione

```bash
kubectl create deployment web-svc --image=nginx:1.25 --replicas=2
kubectl expose deployment web-svc --port=80 --target-port=80 --type=ClusterIP
kubectl get svc
```

### Verifica veloce
```bash
kubectl run tester --image=busybox:1.36 -it --rm --restart=Never -- sh
wget -qO- http://web-svc
```

---

# 16) NetworkPolicy ingress

## Esercizio
Crea:
- Pod `backend` con label `app=backend`
- Pod `frontend` con label `app=frontend`
- Pod `other` con label `app=other`

Applica una NetworkPolicy che permetta traffico in ingresso verso `backend` solo dai pod con label `app=frontend` sulla porta TCP 80.

## Soluzione

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: backend
  labels:
    app: backend
spec:
  containers:
  - name: nginx
    image: nginx:1.25
---
apiVersion: v1
kind: Pod
metadata:
  name: frontend
  labels:
    app: frontend
spec:
  containers:
  - name: client
    image: busybox:1.36
    command: ["sh", "-c", "sleep 3600"]
---
apiVersion: v1
kind: Pod
metadata:
  name: other
  labels:
    app: other
spec:
  containers:
  - name: client
    image: busybox:1.36
    command: ["sh", "-c", "sleep 3600"]
```

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-to-backend
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    ports:
    - protocol: TCP
      port: 80
```

```bash
kubectl apply -f 16-pods.yaml
kubectl apply -f 16-netpol.yaml
```

### Test
```bash
kubectl exec -it frontend -- sh
wget -qO- --timeout=2 http://backend

kubectl exec -it other -- sh
wget -qO- --timeout=2 http://backend
```

---

# 17) NetworkPolicy ingress + egress

## Esercizio
Crea una policy per i pod con label `role=api` che:
- accetti ingress solo dai pod `role=web` sulla porta 8080
- permetta egress solo verso pod `role=db` sulla porta 5432

## Soluzione

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: api-policy
spec:
  podSelector:
    matchLabels:
      role: api
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          role: web
    ports:
    - protocol: TCP
      port: 8080
  egress:
  - to:
    - podSelector:
        matchLabels:
          role: db
    ports:
    - protocol: TCP
      port: 5432
```

### Da sapere
Una volta che una NetworkPolicy seleziona un pod:
- per quel tipo di traffico (`Ingress` e/o `Egress`) il default diventa deny, salvo regole esplicite.

---

# 18) OCI: image, runtime, distribution

## Esercizio teorico-pratico
Rispondi alle domande:

1. Che cos'è OCI?
2. Differenza tra OCI Image Spec, Runtime Spec, Distribution Spec
3. Containerd / CRI-O / runc dove si collocano?
4. Cos'è un registry OCI-compatible?
5. Perché su CKAD può uscire una domanda OCI?

## Soluzione

### 1. Che cos'è OCI?
OCI = Open Container Initiative. Definisce standard aperti per formato immagine container, runtime e distribuzione tramite registry.

### 2. Le 3 specifiche principali
- **Image Spec**: definisce come è fatta un'immagine container
- **Runtime Spec**: definisce come eseguire un container da un filesystem bundle
- **Distribution Spec**: definisce come client e registry si scambiano immagini e artifact

### 3. Dove si collocano i componenti
- `runc`: implementazione del runtime OCI
- `containerd` / `CRI-O`: runtime/container engines usati nel nodo Kubernetes
- registry come Harbor o Docker Hub: lato distribution

### 4. Registry OCI-compatible
Un registry che supporta il modello OCI per push/pull di immagini e artifact.

### 5. Perché in CKAD?
Perché CKAD tocca:
- immagini container
- registries
- push/pull
- artifact e standard usati da Kubernetes

### Mini flashcard
- immagine = **OCI Image**
- esecuzione = **OCI Runtime**
- push/pull = **OCI Distribution**

---

# 19) Troubleshooting rapido CKAD

## Checklist operativa

### Pod non parte
```bash
kubectl get pod
kubectl describe pod <pod>
kubectl logs <pod>
kubectl logs <pod> -c <container>
kubectl get events --sort-by=.metadata.creationTimestamp
```

### Deployment non pronto
```bash
kubectl get deploy
kubectl describe deploy <name>
kubectl get rs
kubectl get pod -l app=<label>
```

### Service non raggiungibile
```bash
kubectl get svc
kubectl get endpoints
kubectl describe svc <name>
kubectl exec -it <client-pod> -- sh
```

### NetworkPolicy dubbi
```bash
kubectl get netpol
kubectl describe netpol <name>
kubectl get pod --show-labels
```

### ConfigMap / Secret non montati
```bash
kubectl describe pod <pod>
kubectl exec -it <pod> -- sh
env
ls <mount-path>
```

---

# 20) Mini mock exam finale

## Task 1
Crea un Deployment `mock-web` con:
- immagine `nginx:1.25`
- 2 repliche
- readiness probe su `/`
- Service ClusterIP chiamato `mock-web-svc`

### Soluzione
```bash
kubectl create deployment mock-web --image=nginx:1.25 --replicas=2 --dry-run=client -o yaml > mock-web.yaml
```

Poi modifica così:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mock-web
spec:
  replicas: 2
  selector:
    matchLabels:
      app: mock-web
  template:
    metadata:
      labels:
        app: mock-web
    spec:
      containers:
      - name: nginx
        image: nginx:1.25
        ports:
        - containerPort: 80
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 3
          periodSeconds: 5
```

```bash
kubectl apply -f mock-web.yaml
kubectl expose deployment mock-web --port=80 --target-port=80 --type=ClusterIP --name=mock-web-svc
```

---

## Task 2
Crea un Job `mock-job` con:
- `completions: 3`
- `parallelism: 2`
- comando `echo run; sleep 4`

### Soluzione
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: mock-job
spec:
  completions: 3
  parallelism: 2
  template:
    spec:
      restartPolicy: Never
      containers:
      - name: worker
        image: busybox:1.36
        command: ["sh", "-c", "echo run; sleep 4"]
```

---

## Task 3
Crea una ConfigMap `mock-cm` con chiave `MODE=test` e usala come env in un Pod `mock-pod`.

### Soluzione
```bash
kubectl create configmap mock-cm --from-literal=MODE=test
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mock-pod
spec:
  containers:
  - name: app
    image: busybox:1.36
    command: ["sh", "-c", "env; sleep 3600"]
    env:
    - name: MODE
      valueFrom:
        configMapKeyRef:
          name: mock-cm
          key: MODE
```

---

## Task 4
Consenti traffico a `mock-web` solo dai pod con label `role=client`.

### Soluzione
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: mock-web-netpol
spec:
  podSelector:
    matchLabels:
      app: mock-web
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          role: client
    ports:
    - protocol: TCP
      port: 80
```

---

# Speed sheet finale da memorizzare

## Generazione YAML rapida
```bash
kubectl create deployment app --image=nginx --dry-run=client -o yaml
kubectl run pod1 --image=busybox:1.36 --dry-run=client -o yaml -- sh
kubectl create job job1 --image=busybox:1.36 --dry-run=client -o yaml -- sh -c 'echo hi'
kubectl create cronjob cj1 --image=busybox:1.36 --schedule="*/5 * * * *" --dry-run=client -o yaml -- sh -c 'date'
```

## Debug
```bash
kubectl get all
kubectl describe pod <pod>
kubectl logs <pod>
kubectl logs <pod> -c <container>
kubectl exec -it <pod> -- sh
kubectl get events --sort-by=.metadata.creationTimestamp
```

## Rollout
```bash
kubectl set image deployment/app nginx=nginx:1.26
kubectl rollout status deployment app
kubectl rollout history deployment app
kubectl rollout undo deployment app
```

## Exposure
```bash
kubectl expose deployment app --port=80 --target-port=80 --type=ClusterIP
```

## Config / Secret
```bash
kubectl create configmap cm1 --from-literal=key=value
kubectl create secret generic sec1 --from-literal=user=admin
```

## Etichette
```bash
kubectl get pod --show-labels
kubectl label pod p1 env=dev
kubectl get pods -l env=dev
```

---

# Strategia pratica per passare CKAD

1. Parti dai task che riconosci subito.
2. Usa `--dry-run=client -o yaml` quando possibile.
3. Non scrivere YAML da zero se puoi generare uno scheletro.
4. Verifica sempre con `kubectl get`, `describe`, `logs`.
5. Se un task parla di networking, controlla label e selector prima di tutto.
6. Per Jobs/CronJobs ricordati i parametri chiave.
7. Per probes, distingui bene:
   - `readiness` = pronto a ricevere traffico
   - `liveness` = processo sano
   - `startup` = startup lenta

---

# Pulizia finale

```bash
kubectl delete ns ckad-lab
```
