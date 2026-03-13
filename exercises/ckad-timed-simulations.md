
# CKAD Timed Practice – 10 Simulazioni da Esame

Queste simulazioni sono pensate per allenarti **come nel vero esame CKAD**.

Regole:
- Tempo massimo suggerito: **15 minuti per esercizio**
- Namespace consigliato: `exam-sim`
- Usa sempre:
  - `kubectl get`
  - `kubectl describe`
  - `kubectl logs`

Setup:

```bash
kubectl create ns exam-sim
kubectl config set-context --current --namespace=exam-sim
```

---

# Simulazione 1 – Probes

## Task

Crea un Deployment chiamato **probe-app** con:

- immagine `nginx:1.25`
- 2 repliche
- readiness probe HTTP `/`
- liveness probe HTTP `/`
- readiness `initialDelaySeconds: 5`
- liveness `initialDelaySeconds: 10`

Tempo: **15 min**

---

## Soluzione

```bash
kubectl create deployment probe-app --image=nginx:1.25 --replicas=2 --dry-run=client -o yaml > probe-app.yaml
```

Modifica:

```yaml
readinessProbe:
  httpGet:
    path: /
    port: 80
  initialDelaySeconds: 5

livenessProbe:
  httpGet:
    path: /
    port: 80
  initialDelaySeconds: 10
```

```bash
kubectl apply -f probe-app.yaml
```

---

# Simulazione 2 – Logs

## Task

Crea un Pod `log-demo` con container:

```
busybox
sh -c "while true; do echo CKAD-LOG; sleep 4; done"
```

Poi:

1. mostra i log
2. segui i log live

---

## Soluzione

```bash
kubectl run log-demo --image=busybox --restart=Never -- sh -c "while true; do echo CKAD-LOG; sleep 4; done"
```

```bash
kubectl logs log-demo
kubectl logs -f log-demo
```

---

# Simulazione 3 – Job

## Task

Crea un Job:

nome: `calc-job`

- completions: 3
- parallelism: 2
- comando:

```
echo run; sleep 5
```

---

## Soluzione

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: calc-job
spec:
  completions: 3
  parallelism: 2
  template:
    spec:
      restartPolicy: Never
      containers:
      - name: worker
        image: busybox
        command: ["sh","-c","echo run; sleep 5"]
```

---

# Simulazione 4 – CronJob

## Task

Crea un CronJob:

nome: `date-job`

schedule:

```
*/5 * * * *
```

comando:

```
date
```

---

## Soluzione

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: date-job
spec:
  schedule: "*/5 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          restartPolicy: Never
          containers:
          - name: date
            image: busybox
            command: ["sh","-c","date"]
```

---

# Simulazione 5 – ConfigMap

## Task

Crea ConfigMap:

```
app-mode=dev
app-color=green
```

Usala in un Pod `config-demo` come variabili ambiente.

---

## Soluzione

```bash
kubectl create configmap app-config --from-literal=app-mode=dev --from-literal=app-color=green
```

Pod:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: config-demo
spec:
  containers:
  - name: app
    image: busybox
    command: ["sh","-c","env; sleep 3600"]
    envFrom:
    - configMapRef:
        name: app-config
```

---

# Simulazione 6 – Secret

## Task

Crea Secret:

```
username=admin
password=topsecret
```

Usalo come env.

---

## Soluzione

```bash
kubectl create secret generic db-secret --from-literal=username=admin --from-literal=password=topsecret
```

---

# Simulazione 7 – Service

## Task

Crea Deployment `web-test`:

- nginx
- 2 repliche

Esporlo come **ClusterIP** sulla porta 80.

---

## Soluzione

```bash
kubectl create deployment web-test --image=nginx --replicas=2
kubectl expose deployment web-test --port=80 --type=ClusterIP
```

---

# Simulazione 8 – NetworkPolicy

## Task

Permetti traffico verso Pod con label:

```
app=backend
```

solo da Pod con label:

```
app=frontend
```

porta 80.

---

## Soluzione

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend
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
    - port: 80
      protocol: TCP
```

---

# Simulazione 9 – Rolling Update

## Task

Deployment `update-app`:

- nginx:1.25
- 3 repliche

Aggiorna a:

```
nginx:1.26
```

---

## Soluzione

```bash
kubectl create deployment update-app --image=nginx:1.25 --replicas=3
kubectl set image deployment/update-app nginx=nginx:1.26
kubectl rollout status deployment update-app
```

Rollback:

```bash
kubectl rollout undo deployment update-app
```

---

# Simulazione 10 – Troubleshooting

## Task

Un Pod è in stato:

```
ImagePullBackOff
```

Trova il problema.

---

## Soluzione

```bash
kubectl describe pod <pod>
kubectl get events
```

Tipico problema:

```
wrong image name
```

---

# Strategie velocissime per CKAD

### YAML veloce

```bash
kubectl create deployment app --image=nginx --dry-run=client -o yaml
kubectl run pod1 --image=busybox --dry-run=client -o yaml
kubectl create job job1 --image=busybox --dry-run=client -o yaml
```

### Debug

```bash
kubectl get all
kubectl describe pod
kubectl logs
kubectl exec -it
kubectl get events
```

### Networking

```bash
kubectl get svc
kubectl get endpoints
kubectl get netpol
```

---

# Consiglio reale per passare CKAD

Fai sempre:

1️⃣ Task facili prima  
2️⃣ Usa dry-run YAML  
3️⃣ Controlla label e selector  
4️⃣ Testa con `kubectl get` subito  

---

Pulizia:

```bash
kubectl delete ns exam-sim
```
