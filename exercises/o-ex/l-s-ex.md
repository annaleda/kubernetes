
###  Labels and Selectors (6 esercizi)

## LS-1 — Deployment con Label Specifiche

Creare un Deployment chiamato `frontend-app`

- Specifiche
  - Image: nginx
  - Replicas: 3

Labels:
  - app=frontend
  - tier=web

- Obiettivo
  - Verificare che i Pod abbiano le stesse label

- Validazione
  - kubectl get pods --show-labels
  - Label coerenti tra Deployment e Pod
---
<details>
<summary>Soluzione</summary>
  
```
k create deploy frontend-app --image=nginx --replicas=3 --dry-run=client -o yaml > deploy1.yaml

vi deploy1.yaml


apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: frontend-app
  name: frontend-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: frontend
      tier: web
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: frontend
        tier: web
    spec:
      containers:
      - image: nginx
        name: nginx
        resources: {}
status: {}


k apply -f deploy1.yaml

kubectl get pods --show-labels

```
</details>

---

## LS-2 — Service con Selector Multipli

Creare Deployment `backend-app`

- Labels richieste
  - app=backend
  - tier=api

Creare Service `backend-service`

- Specifiche Service
  - Type: ClusterIP
  - Porta: 80

- Selector:
  - app=backend
  - tier=api

- Validazione
  - Service seleziona solo i Pod corretti
  - kubectl describe svc backend-service mostra Endpoints

---
<details>
<summary>Soluzione</summary>
  
```
k create deploy backend-app --image=nginx --replicas=3 --dry-run=client -o yaml > deploy2.yaml

vi deploy2.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: backend-app
  name: backend-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: backend
      tier: api
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: backend
        tier: api
    spec:
      containers:
      - image: nginx
        name: nginx
        resources: {}
status: {}


k apply -f deploy2.yaml

kubectl get pods --show-labels

k expose deploy backend-app --name backend-service --port=80 --dry-run=client -o yaml > svc1.yaml

k apply -f svc1.yaml

k get svc

k get pod -l app=backend
```
</details>

---

## LS-3 — Modifica Label Live

Creare Pod `temp-app`
  - Image: nginx

- Task
  - Aggiungere label: environment=dev
  - Poi cambiarla in environment=prod

- Validazione
  - kubectl get pod temp-app --show-labels
---
<details>
<summary>Soluzione</summary>
  
```
 k run temp-app --image=nginx -l environment=dev -o yaml > pod-label.yaml

 vi pod-label.yaml

apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2026-03-04T10:28:14Z"
  generation: 1
  labels:
    environment: prod
  name: temp-app
  namespace: default
  resourceVersion: "379101"
  uid: 9d6f384e-317e-4986-afe7-5f3691bc8d2c
spec:
  containers:
  - image: nginx
    imagePullPolicy: Always
    name: temp-app
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-rd47r
      readOnly: true
  dnsPolicy: ClusterFirst
  enableServiceLinks: true
  preemptionPolicy: PreemptLowerPriority
  priority: 0
  restartPolicy: Always
  schedulerName: default-scheduler
  securityContext: {}
  serviceAccount: default
  serviceAccountName: default
  terminationGracePeriodSeconds: 30
  tolerations:
  - effect: NoExecute
    key: node.kubernetes.io/not-ready
    operator: Exists
    tolerationSeconds: 300
  - effect: NoExecute
    key: node.kubernetes.io/unreachable
    operator: Exists
    tolerationSeconds: 300
  volumes:
  - name: kube-api-access-rd47r
    projected:
      defaultMode: 420
      sources:
      - serviceAccountToken:
          expirationSeconds: 3607
          path: token
      - configMap:
          items:
          - key: ca.crt
            path: ca.crt
          name: kube-root-ca.crt
      - downwardAPI:
          items:
          - fieldRef:
              apiVersion: v1
              fieldPath: metadata.namespace
            path: namespace
status:
  phase: Pending
  qosClass: BestEffort

k replace -f pod-label.yaml --force

k get pod temp-app --show-labels

NAME       READY   STATUS              RESTARTS   AGE   LABELS
temp-app   1/1     Running             0          83s   environment=prod

```
</details>

---

### LS-4 — Selector Non Matchante (Debug)

Creare Deployment `api-app`

- Label: app=api
- Creare Service `api-service`
- Selector: app=backend

- Obiettivo
  - Identificare perché il Service non ha endpoint
  - Correggere il selector

- Validazione
  - Service ha endpoint dopo la correzione

---
<details>
<summary>Soluzione</summary>
  
```
k create deploy api --image=nginx --dry-run=client -o yaml > api-app.yaml

vi api-app.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: api
  name: api-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: api
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: api
    spec:
      containers:
      - image: nginx
        name: nginx

k apply -f api-app.yaml

k expose deploy api-app --name api-service --port=80 --dry-run=client -o yaml > api-service.yaml

vi api-service.yaml

apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    app: api
  name: api-service
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: backend

k get svc api-service
NAME          TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
api-service   ClusterIP   10.96.207.250   <none>        80/TCP    63s

k describe svc api-service
```
</details>

---

### LS-5 — Node Selector

Label su un nodo: `disktype=ssd`

Creare Pod `ssd-app`

- Configurazione
  - nodeSelector:
    - disktype=ssd

- Validazione
  - Pod schedulato solo sul nodo corretto
---
<details>
<summary>Soluzione</summary>
  
```
kubectl label node ckad-exam-control-plane disktype=ssd

 k get nodes --show-labels
NAME                      STATUS   ROLES           AGE   VERSION    LABELS
ckad-exam-control-plane   Ready    control-plane   13d   v1.35.0    beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,
                                                                    disktype=ssd,kubernetes.io/arch=amd64,
                                                                    kubernetes.io/hostname=ckad-exam-control-plane,
                                                                    kubernetes.io/os=linux,node-role.kubernetes.io/control-plane=

k run ssd-app --image=nginx --dry-run=client -o yaml > ssd-app.yaml

vi ssd-app.yaml

apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: ssd-app
  name: ssd-app
spec:
  containers:
  - image: nginx
    name: ssd-app
    resources: {}
  nodeSelector:
    disktype: ssd
  dnsPolicy: ClusterFirst
  restartPolicy: Always

k get po ssd-app -o wide
NAME      READY   STATUS    RESTARTS   AGE   IP       NODE                   
ssd-app   0/1     Pending   0          24s   <none>   ckad-exam-control-plane

```
</details>

---

### LS-6 — Set-Based Selectors

Creare Deployment `multi-env-app`

- Labels possibili:
  - environment=dev
  - environment=staging
  - environment=prod

- Obiettivo
  - Recuperare solo Pod con:
    - environment in (dev, staging)

- Validazione
  - Usare selector set-based via CLI
---
<details>
<summary>Soluzione</summary>
  
```
k create deploy multi-env-app --replicas=2 --image=nginx --dry-run=client -o yaml > multi-env-app.yaml
# modifica labels

kubectl get pods -l 'environment in (dev,staging)'
```
</details>

---
