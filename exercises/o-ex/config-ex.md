- [ Home ](../../readme.md)   | [ Teoria ](../../arguments.md)   | [ Info Exam ](../../doc/ckad_exam_strategy.md)    | [ Home Other Exercises ](../o_exercises.md)   
--- 
###  Configuration (24 esercizi)

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
  labels:
    run: config-pod
  name: config-pod
  namespace: configuration
spec:
  volumes:
  - name: config
    configMap:
      name: app-config
  containers:
  - image: nginx
    name: config-pod
    command:
    - sh
    - -c
    - echo $APP_MODE; sleep 3600
    envFrom:
    - configMapRef:
        name: app-config
    volumeMounts:
    - name: config
      mountPath: /config

  dnsPolicy: ClusterFirst
  restartPolicy: Always

 k apply -f config-pod.yaml
 
 k logs -n configuration config-pod
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
         secretName: db-secret

 k exec -it -n configuration secret-pod -- sh
 #ls /etc/secret
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

k create quota quota -n quota-ns --hard=pods=2,requests.cpu=500m -oyaml --dry-run=client > quota1.yaml


apiVersion: v1
kind: ResourceQuota
metadata:
  name: quota
  namespace: quota-ns
spec:
  hard:
    pods: "2"
    requests.cpu: 500m
 
```
</details>

---

## CONF-4 — LimitRange

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

k run test-pod --image=nginx -n limit-ns
#  son stati messi i limits,requests nel resources del container del pod test-pod
k get po test-pod -n limit-ns -oyaml

```
</details>

---

## CONF-5 — Requests vs Limits

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

## CONF-6 — ConfigMap + Secret insieme

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
  labels:
    run: env-app
  name: env-app
spec:
  volumes:
  - name: config
    configMap:
      name: app-config
  - name: secret
    secret:
      secretName: app-secret
  containers:
  - image: nginx
    name: env-app
    command:
    - sh
    - -c
    - echo $APP_MODE; echo $DB_USER; echo $DB_PASS; sleep 3600
    volumeMounts:
    - name: config
      mountPath: /config
    - name: secret
      mountPath: /etc/secret
    envFrom:
    - configMapRef:
        name: app-config
    - secretRef:
        name: app-secret
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}


$ k logs env-app
production
admin
pass123

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

 il secondo fallisce

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

## CONF-13 — ConfigMap da file

- Creare un file locale `app.properties`
  - contenuto: `color=blue`

- Creare ConfigMap: `file-based-config`
  - usando il file locale

- Pod: `file-config-pod`
  - Image: busybox
  - Montare il ConfigMap in `/etc/config`

- Validazione
  - Il file `/etc/config/app.properties` esiste nel container

---

<details>
<summary>Soluzione</summary>

```sh
echo "color=blue" > app.properties
k create cm file-based-config --from-file=app.properties
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: file-config-pod
spec:
  containers:
  - name: app
    image: busybox
    command: ["sh","-c","ls /etc/config && cat /etc/config/app.properties && sleep 3600"]
    volumeMounts:
    - name: config-vol
      mountPath: /etc/config
  volumes:
  - name: config-vol
    configMap:
      name: file-based-config
```

```sh
k apply -f file-config-pod.yaml
k exec -it file-config-pod -- cat /etc/config/app.properties
```

</details>

---

## CONF-14 — Secret da file

- Creare un file locale `password.txt`
  - contenuto: `mypassword`

- Creare Secret: `file-secret`
  - usando il file locale

- Pod: `file-secret-pod`
  - Image: busybox
  - Montare il Secret in `/etc/secret`

- Validazione
  - Il file montato è visibile nel container

---

<details>
<summary>Soluzione</summary>

```sh
echo "mypassword" > password.txt
k create secret generic file-secret --from-file=password.txt
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: file-secret-pod
spec:
  containers:
  - name: app
    image: busybox
    command: ["sh","-c","ls /etc/secret && cat /etc/secret/password.txt && sleep 3600"]
    volumeMounts:
    - name: secret-vol
      mountPath: /etc/secret
      readOnly: true
  volumes:
  - name: secret-vol
    secret:
      secretName: file-secret
```

```sh
k apply -f file-secret-pod.yaml
k exec -it file-secret-pod -- ls /etc/secret
```

</details>

---

## CONF-15 — ConfigMap con env e volume insieme

- ConfigMap: `dual-config`
  - APP_MODE=prod
  - app.conf=mode=prod

- Pod: `dual-config-pod`

- Obiettivo
  - Usare lo stesso ConfigMap:
    - come variabile d’ambiente
    - come volume

- Validazione
  - Variabile disponibile
  - File montato presente

---

<details>
<summary>Soluzione</summary>

```sh
k create cm dual-config \
  --from-literal=APP_MODE=prod \
  --from-literal=app.conf=mode=prod
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: dual-config-pod
spec:
  containers:
  - name: app
    image: busybox
    command: ["sh","-c","echo $APP_MODE && cat /etc/config/app.conf && sleep 3600"]
    env:
    - name: APP_MODE
      valueFrom:
        configMapKeyRef:
          name: dual-config
          key: APP_MODE
    volumeMounts:
    - name: config-vol
      mountPath: /etc/config
  volumes:
  - name: config-vol
    configMap:
      name: dual-config
```

</details>

---

## CONF-16 — Secret con envFrom

- Secret: `db-env-secret`
  - DB_USER=admin
  - DB_PASS=topsecret

- Pod: `secret-envfrom-pod`

- Obiettivo
  - Caricare tutte le chiavi del Secret come variabili d’ambiente

- Validazione
  - `env` nel container mostra le variabili

---

<details>
<summary>Soluzione</summary>

```sh
k create secret generic db-env-secret \
  --from-literal=DB_USER=admin \
  --from-literal=DB_PASS=topsecret
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-envfrom-pod
spec:
  containers:
  - name: app
    image: busybox
    command: ["sh","-c","env && sleep 3600"]
    envFrom:
    - secretRef:
        name: db-env-secret
```

```sh
k apply -f secret-envfrom-pod.yaml
k exec -it secret-envfrom-pod -- env
```

</details>

---

## CONF-17 — LimitRange con default request e default memory

- Namespace: `memory-limit-ns`

- Creare LimitRange
  - defaultRequest memory: 64Mi
  - default memory limit: 256Mi

- Validazione
  - Pod senza memory esplicita riceve i valori di default

---

<details>
<summary>Soluzione</summary>

```sh
k create ns memory-limit-ns
```

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: mem-limit
  namespace: memory-limit-ns
spec:
  limits:
  - type: Container
    defaultRequest:
      memory: 64Mi
    default:
      memory: 256Mi
```

```sh
k apply -f mem-limit.yaml
k run mem-pod --image=nginx -n memory-limit-ns
k describe pod mem-pod -n memory-limit-ns
```

</details>

---

## CONF-18 — ResourceQuota su memoria

- Namespace: `mem-quota`

- Creare ResourceQuota
  - requests.memory: 256Mi
  - limits.memory: 512Mi

- Obiettivo
  - Limitare la memoria totale del namespace

- Validazione
  - `kubectl describe quota -n mem-quota`

---

<details>
<summary>Soluzione</summary>

```sh
k create ns mem-quota
```

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: memory-quota
  namespace: mem-quota
spec:
  hard:
    requests.memory: 256Mi
    limits.memory: 512Mi
```

```sh
k apply -f memory-quota.yaml
k describe quota memory-quota -n mem-quota
```

</details>

---

## CONF-19 — Pod con requests e limits CPU+memory

- Pod: `full-resources-pod`

- Container
  - Image: nginx
  - Requests:
    - cpu: 100m
    - memory: 128Mi
  - Limits:
    - cpu: 300m
    - memory: 256Mi

- Validazione
  - `kubectl describe pod full-resources-pod`

---

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: full-resources-pod
spec:
  containers:
  - name: nginx
    image: nginx
    resources:
      requests:
        cpu: 100m
        memory: 128Mi
      limits:
        cpu: 300m
        memory: 256Mi
```

```sh
k apply -f full-resources-pod.yaml
k describe pod full-resources-pod
```

</details>

---

## CONF-20 — ConfigMap con items

- ConfigMap: `multi-file-config`
  - app.properties=mode=prod
  - db.properties=user=admin

- Pod: `items-config-pod`

- Obiettivo
  - Montare solo `db.properties`

- Validazione
  - Nel container esiste solo il file selezionato

---

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: multi-file-config
data:
  app.properties: |
    mode=prod
  db.properties: |
    user=admin
---
apiVersion: v1
kind: Pod
metadata:
  name: items-config-pod
spec:
  containers:
  - name: app
    image: busybox
    command: ["sh","-c","ls /etc/config && cat /etc/config/db.properties && sleep 3600"]
    volumeMounts:
    - name: config-vol
      mountPath: /etc/config
  volumes:
  - name: config-vol
    configMap:
      name: multi-file-config
      items:
      - key: db.properties
        path: db.properties
```

</details>

---

## CONF-21 — Secret con chiave singola in env

- Secret: `single-secret`
  - PASSWORD=abc123
  - USER=admin

- Pod: `single-secret-env-pod`

- Obiettivo
  - Usare solo la chiave `PASSWORD` come env

- Validazione
  - La variabile PASSWORD è disponibile nel container

---

<details>
<summary>Soluzione</summary>

```sh
k create secret generic single-secret \
  --from-literal=PASSWORD=abc123 \
  --from-literal=USER=admin
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: single-secret-env-pod
spec:
  containers:
  - name: app
    image: busybox
    command: ["sh","-c","echo $PASSWORD && sleep 3600"]
    env:
    - name: PASSWORD
      valueFrom:
        secretKeyRef:
          name: single-secret
          key: PASSWORD
```

</details>

---

## CONF-22 — ResourceQuota con max servizi

- Namespace: `svc-quota`

- Creare ResourceQuota
  - services: 2

- Obiettivo
  - Impedire la creazione di un terzo Service

- Validazione
  - Il terzo Service fallisce

---

<details>
<summary>Soluzione</summary>

```sh
k create ns svc-quota
```

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: service-quota
  namespace: svc-quota
spec:
  hard:
    services: "2"
```

```sh
k apply -f service-quota.yaml
k describe quota service-quota -n svc-quota
```

</details>

---

## CONF-23 — LimitRange con min e max CPU

- Namespace: `cpu-range`

- Creare LimitRange
  - min cpu: 50m
  - max cpu: 500m

- Obiettivo
  - Forzare i container a stare nel range CPU definito

- Validazione
  - `kubectl describe limitrange -n cpu-range`

---

<details>
<summary>Soluzione</summary>

```sh
k create ns cpu-range
```

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-range-limit
  namespace: cpu-range
spec:
  limits:
  - type: Container
    min:
      cpu: 50m
    max:
      cpu: 500m
```

```sh
k apply -f cpu-range-limit.yaml
k describe limitrange cpu-range-limit -n cpu-range
```

</details>

---

## CONF-24 — Debug ConfigMap / Secret mancanti

- Pod: `broken-config-pod`

- Problema
  - Il Pod referenzia un ConfigMap o Secret inesistente

- Obiettivo
  - Identificare il problema con i comandi giusti

- Validazione
  - Capire perché il Pod non parte correttamente

---

<details>
<summary>Soluzione</summary>

Comandi utili:

```sh
k get pod broken-config-pod
k describe pod broken-config-pod
k get cm
k get secret
```

Cause tipiche:
- ConfigMap inesistente
- Secret inesistente
- chiave sbagliata in `configMapKeyRef` o `secretKeyRef`
- nome errato nel volume mount

</details>

---
