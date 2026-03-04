
###  Configuration (6 esercizi)

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
```
</details>

---
