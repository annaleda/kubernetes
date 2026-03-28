
###  Configuration (12 esercizi)

## CONF-1 — ConfigMap

- Creare ConfigMap: `app-config`
  - key: APP_MODE
  - value: production
- Montare in Pod config-pod
- Validazione
  - Variabile disponibile nel container
---
<details>
<summary>Soluzione</summary>
  
```
k create ns configuration
k create cm app-config --from-literal=APP_MODE=production -n configuration

k run config-pod --image=nginx -n configuration --dry-run=client -o yaml > config-pod.yaml

vi config-pod.yaml

apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: config-pod
  name: config-pod
  namespace: configuration
spec:
  containers:
  - image: nginx
    name: config-pod
    volumeMounts:
    - name: app-config-volume
      mountPath: /etc/config
  volumes:
    - name: app-config-volume
      configMap:
        name: app-config

 k apply -f config-pod.yaml
 k describe po config-pod -n configuration
 k exec -n configuration config-pod -- ls /etc/config
```
</details>

---

## CONF-2 — Secret

- Creare Secret: `db-secret`
  - DB_USER=admin
  - DB_PASS=pass123
  - Montare come volume nel pod `secret-pod`
- Validazione
  - File presente nel container

---
<details>
<summary>Soluzione</summary>
  
```
k create secret generic db-secret -n configuration --from-literal=DB_USER=admin --from-literal=DB_PASS=pass123

k run secret-pod --image=nginx -n configuration --dry-run=client -o yaml > secret-pod.yaml

apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: secret-pod
  name: secret-pod
  namespace: configuration
spec:
  containers:
  - image: nginx
    name: secred-pod
    volumeMounts:
    - name: db-secret-volume
      mountPath: /etc/secret
  volumes:
    - name: db-secret-volume
      secret:
        name: db-secret

 k exec -n configuration secret-pod -- ls /etc/secret
```
</details>

---

## CONF-3 — ResourceQuota

- Namespace: `quota-ns`
- Impostare ResourceQuota
  - Pods: 2
  - Requests CPU totali: 500m
- Validazione
  - Creazione terzo pod fallisce
---
<details>
<summary>Soluzione</summary>
  
```
 k create ns quota-ns 

apiVersion: v1
kind: ResourceQuota
metadata:
  name: quota
  namespace: quota-ns
spec:
  hard:
    pods: "2"
    requests.cpu: "500m"
 
```
</details>

---

### CONF-4 — LimitRange

- Namespace: `limit-ns`
- Creare LimitRange
  - Default CPU limit: 200m
  - Default memory limit: 128Mi
- Validazione
  - Pod senza limiti riceve default

---
<details>
<summary>Soluzione</summary>
  
```
 k create ns limit-ns

  apiVersion: v1
  kind: LimitRange
  metadata:
    name: limit-range
    namespace: limit-ns
  spec:
    limits:
    - type: Container
    default:
      cpu: 200m
      memory: 128Mi

```
</details>

---

### CONF-5 — Requests vs Limits

- Deployment: `resource-test`
- Container
  - Requests: 100m
  - Limits: 200m
- Validazione
  - Differenza visibile nel describe
---
<details>
<summary>Soluzione</summary>
  
```
kubectl create deploy resource-test --image=nginx --dry-run=client -o yaml > resource-test.yaml

  apiVersion: apps/v1
kind: Deployment
metadata:
  name: resource-test
spec:
  replicas: 1
  selector:
    matchLabels:
      app: resource-test
  template:
    metadata:
      labels:
        app: resource-test
    spec:
      containers:
      - name: nginx
        image: nginx
        resources:
          requests:
            cpu: 100m
          limits:
            cpu: 200m
```
</details>

---

### CONF-6 — ConfigMap + Secret insieme

- Pod: `env-app`
- Usare
  - ConfigMap per config non sensibili
  - Secret per credenziali
- Validazione
  - Entrambi montati correttamente
---
<details>
<summary>Soluzione</summary>
  
```
k create secret generic app-secret --from-literal=DB_USER=admin --from-literal=DB_PASS=pass123
k create cm app-config --from-literal=APP_MODE=production
k run env-app --image=nginx --dry-run=client -o yaml > env-app.yaml

apiVersion: v1
kind: Pod
metadata:
  name: env-app
spec:
  containers:
  - name: app
    image: nginx
    envFrom:
    - configMapRef:
        name: app-config
    - secretRef:
        name: app-secret

    volumeMounts:
    - name: config-volume
      mountPath: /etc/config
    - name: secret-volume
      mountPath: /etc/secret
      readOnly: true
      
  volumes:
  - name: config-volume
    configMap:
      name: app-config

  - name: secret-volume
    secret:
      secretName: app-secret
```
</details>

---

## CONF-7 — ConfigMap come env (envFrom)

- ConfigMap: `env-config`
  - APP_ENV=dev
  - LOG_LEVEL=debug

- Pod: `envfrom-pod`

- Obiettivo
  - Usare envFrom per caricare tutte le variabili

- Validazione
  - Variabili visibili nel container

---

<details>
<summary>Soluzione</summary>

```sh
k create cm env-config \
  --from-literal=APP_ENV=dev \
  --from-literal=LOG_LEVEL=debug
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: envfrom-pod
spec:
  containers:
  - name: app
    image: busybox
    command: ["sh","-c","env && sleep 3600"]
    envFrom:
    - configMapRef:
        name: env-config
```

```sh
k exec -it envfrom-pod -- env
```

</details>

---

## CONF-8 — Secret come variabili d’ambiente

- Secret: `env-secret`
  - API_KEY=12345

- Pod: `secret-env-pod`

- Obiettivo
  - Usare Secret come variabile d’ambiente

---

<details>
<summary>Soluzione</summary>

```sh
k create secret generic env-secret \
  --from-literal=API_KEY=12345
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-env-pod
spec:
  containers:
  - name: app
    image: busybox
    command: ["sh","-c","env && sleep 3600"]
    env:
    - name: API_KEY
      valueFrom:
        secretKeyRef:
          name: env-secret
          key: API_KEY
```

</details>

---

## CONF-9 — ConfigMap singola chiave (env)

- ConfigMap: `single-config`
  - key: MODE
  - value: test

- Pod: `single-env-pod`

- Obiettivo
  - Usare solo una chiave specifica

---

<details>
<summary>Soluzione</summary>

```sh
k create cm single-config --from-literal=MODE=test
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: single-env-pod
spec:
  containers:
  - name: app
    image: busybox
    command: ["sh","-c","env && sleep 3600"]
    env:
    - name: MODE
      valueFrom:
        configMapKeyRef:
          name: single-config
          key: MODE
```

</details>

---

## CONF-10 — ConfigMap con subPath

- ConfigMap: `file-config`
  - file: app.conf

- Pod: `subpath-config-pod`

- Obiettivo
  - Montare SOLO un file specifico

---

<details>
<summary>Soluzione</summary>

```sh
k create cm file-config --from-literal=app.conf="hello=config"
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: subpath-config-pod
spec:
  containers:
  - name: app
    image: busybox
    command: ["sh","-c","cat /etc/app.conf && sleep 3600"]
    volumeMounts:
    - name: config
      mountPath: /etc/app.conf
      subPath: app.conf
  volumes:
  - name: config
    configMap:
      name: file-config
```

</details>

---

## CONF-11 — ResourceQuota (debug superamento)

- Namespace: `quota-test`

- Configurazione
  - max pods: 1

- Obiettivo
  - Creare 2 Pod e capire perché fallisce

---

<details>
<summary>Soluzione</summary>

```sh
k create ns quota-test
```

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: pod-limit
  namespace: quota-test
spec:
  hard:
    pods: "1"
```

```sh
k apply -f quota.yaml
k run pod1 --image=nginx -n quota-test
k run pod2 --image=nginx -n quota-test
```

👉 il secondo fallisce

```sh
k describe quota -n quota-test
```

</details>

---

## CONF-12 — LimitRange (requests automatici)

- Namespace: `limit-test`

- Configurazione
  - default request CPU: 100m
  - default limit CPU: 300m

- Obiettivo
  - Pod senza risorse → valori automatici

---

<details>
<summary>Soluzione</summary>

```sh
k create ns limit-test
```

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-limit
  namespace: limit-test
spec:
  limits:
  - type: Container
    defaultRequest:
      cpu: 100m
    default:
      cpu: 300m
```

```sh
k apply -f limit.yaml
k run test --image=nginx -n limit-test
k describe pod test -n limit-test
```

</details>

---
