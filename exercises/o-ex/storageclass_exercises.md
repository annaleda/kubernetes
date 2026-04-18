- [ Home ](../../readme.md)   | [ Teoria ](../../arguments.md)   | [ Info Exam ](../../doc/ckad_exam_strategy.md)    | [ Home Other Exercises ](../o_exercises.md)   
--- 

### StorageClass (10 esercizi)
---

## SC-1 — Creare una StorageClass base

- StorageClass
  - Nome: `fast-sc`
  - Provisioner: `kubernetes.io/no-provisioner`
  - volumeBindingMode: `WaitForFirstConsumer`

- Obiettivo
  - Creare correttamente la StorageClass

- Validazione
  - La StorageClass esiste
  - `volumeBindingMode` è corretto

---
<details>
<summary>Soluzione</summary>

```sh
vi fast-sc.yaml
```

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-sc
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
```

```sh
k apply -f fast-sc.yaml
k get storageclass
k describe storageclass fast-sc
```

</details>

---

## SC-2 — PVC con StorageClass specifica

- PersistentVolumeClaim
  - Nome: `sc-pvc`
  - Storage: `200Mi`
  - AccessMode: `ReadWriteOnce`
  - StorageClass: `standard`

- Obiettivo
  - Creare un PVC che richieda provisioning dinamico

- Validazione
  - Il PVC usa la StorageClass corretta
  - Il PVC va in `Bound` se il cluster supporta il provisioner

---
<details>
<summary>Soluzione</summary>

```sh
vi sc-pvc.yaml
```

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: sc-pvc
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: standard
  resources:
    requests:
      storage: 200Mi
```

```sh
k apply -f sc-pvc.yaml
k get pvc sc-pvc
k describe pvc sc-pvc
```

</details>

---

## SC-3 — Pod che usa PVC dinamico

- PVC
  - Nome: `dynamic-pvc`
  - Storage: `150Mi`
  - StorageClass: `standard`

- Pod
  - Nome: `dynamic-pod`
  - Image: `busybox`
  - Monta il PVC in `/data`
  - Comando: scrivere `hello storageclass` in `/data/test.txt`

- Validazione
  - PVC `Bound`
  - Pod `Running`
  - Il file esiste nel volume

---
<details>
<summary>Soluzione</summary>

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: dynamic-pvc
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: standard
  resources:
    requests:
      storage: 150Mi
---
apiVersion: v1
kind: Pod
metadata:
  name: dynamic-pod
spec:
  containers:
  - name: app
    image: busybox
    command:
    - sh
    - -c
    - echo "hello storageclass" > /data/test.txt && sleep 3600
    volumeMounts:
    - name: dyn-storage
      mountPath: /data
  volumes:
  - name: dyn-storage
    persistentVolumeClaim:
      claimName: dynamic-pvc
```

```sh
k apply -f dynamic-pod.yaml
k get pvc
k get po
k exec -it dynamic-pod -- cat /data/test.txt
```

</details>

---

## SC-4 — StorageClass con reclaimPolicy Delete

- StorageClass
  - Nome: `delete-sc`
  - Provisioner: `kubernetes.io/no-provisioner`
  - reclaimPolicy: `Delete`

- Obiettivo
  - Creare una StorageClass che cancelli il backend storage quando il PVC viene eliminato

- Validazione
  - La StorageClass mostra `Delete` come reclaim policy

---
<details>
<summary>Soluzione</summary>

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: delete-sc
provisioner: kubernetes.io/no-provisioner
reclaimPolicy: Delete
volumeBindingMode: WaitForFirstConsumer
```

```sh
k apply -f delete-sc.yaml
k describe storageclass delete-sc
```

</details>

---

## SC-5 — StorageClass con allowVolumeExpansion

- StorageClass
  - Nome: `expandable-sc`
  - Provisioner: `kubernetes.io/no-provisioner`
  - allowVolumeExpansion: `true`

- PVC
  - Nome: `expand-pvc`
  - Storage iniziale: `1Gi`
  - StorageClass: `expandable-sc`

- Obiettivo
  - Preparare una StorageClass che supporti espansione del PVC

- Validazione
  - La StorageClass ha `allowVolumeExpansion: true`
  - Il PVC è creato correttamente

---
<details>
<summary>Soluzione</summary>

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: expandable-sc
provisioner: kubernetes.io/no-provisioner
allowVolumeExpansion: true
volumeBindingMode: WaitForFirstConsumer
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: expand-pvc
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: expandable-sc
  resources:
    requests:
      storage: 1Gi
```

```sh
k apply -f expandable-sc.yaml
k get sc expandable-sc
k get pvc expand-pvc
```

</details>

---

## SC-6 — Ridimensionare un PVC con StorageClass compatibile

- Scenario
  - Esiste un PVC `expand-pvc`
  - Storage attuale: `1Gi`
  - StorageClass con `allowVolumeExpansion: true`

- Obiettivo
  - Modificare il PVC portandolo a `2Gi`

- Validazione
  - Il PVC mostra la nuova dimensione richiesta
  - Il resize funziona solo se supportato dal provisioner

---
<details>
<summary>Soluzione</summary>

```sh
k edit pvc expand-pvc
```

Modificare:

```yaml
spec:
  resources:
    requests:
      storage: 2Gi
```

Oppure:

```sh
k patch pvc expand-pvc -p '{"spec":{"resources":{"requests":{"storage":"2Gi"}}}}'
```

```sh
k get pvc expand-pvc
k describe pvc expand-pvc
```

</details>

---

## SC-7 — Default StorageClass del cluster

- Obiettivo
  - Identificare quale StorageClass è impostata come default

- Validazione
  - Individuare la StorageClass con annotation default
  - Verificare il comportamento di un PVC senza `storageClassName`

---
<details>
<summary>Soluzione</summary>

```sh
k get storageclass
k get storageclass -o yaml
```

Cercare l'annotation:

```yaml
storageclass.kubernetes.io/is-default-class: "true"
```

Esempio PVC senza `storageClassName`:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: default-sc-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 100Mi
```

```sh
k apply -f default-sc-pvc.yaml
k describe pvc default-sc-pvc
```

</details>

---

## SC-8 — Deployment con PVC dinamico

- PersistentVolumeClaim
  - Nome: `web-dynamic-pvc`
  - Storage: `300Mi`
  - AccessMode: `ReadWriteOnce`
  - StorageClass: `standard`

- Deployment
  - Nome: `web-dynamic-deploy`
  - Replicas: `1`
  - Image: `nginx`
  - Monta il PVC in `/usr/share/nginx/html`

- Validazione
  - PVC `Bound`
  - Pod del Deployment `Running`
  - Volume montato correttamente

---
<details>
<summary>Soluzione</summary>

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: web-dynamic-pvc
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: standard
  resources:
    requests:
      storage: 300Mi
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-dynamic-deploy
spec:
  replicas: 1
  selector:
    matchLabels:
      app: web-dynamic
  template:
    metadata:
      labels:
        app: web-dynamic
    spec:
      containers:
      - name: nginx
        image: nginx
        volumeMounts:
        - name: web-content
          mountPath: /usr/share/nginx/html
      volumes:
      - name: web-content
        persistentVolumeClaim:
          claimName: web-dynamic-pvc
```

```sh
k apply -f web-dynamic-deploy.yaml
k get pvc
k get po
k describe pod <pod-name>
```

</details>

---

## SC-9 — StatefulSet con volumeClaimTemplates e StorageClass

- StatefulSet
  - Nome: `web`
  - ServiceName: `web`
  - Replicas: `2`
  - Image: `nginx`

- volumeClaimTemplates
  - Nome volume: `data`
  - Storage: `1Gi`
  - AccessMode: `ReadWriteOnce`
  - StorageClass: `standard`

- Obiettivo
  - Creare storage persistente dedicato per ogni replica

- Validazione
  - Vengono creati PVC separati per ogni Pod
  - Esempio: `data-web-0`, `data-web-1`

---
<details>
<summary>Soluzione</summary>

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web
spec:
  clusterIP: None
  selector:
    app: web
  ports:
  - port: 80
    targetPort: 80
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: web
  replicas: 2
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: nginx
        image: nginx
        volumeMounts:
        - name: data
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes:
      - ReadWriteOnce
      storageClassName: standard
      resources:
        requests:
          storage: 1Gi
```

```sh
k apply -f web-statefulset.yaml
k get pvc
k get po
```

</details>

---

## SC-10 — Debug PVC Pending con StorageClass

- Scenario
  - Il PVC `broken-sc-pvc` resta in stato `Pending`

- Obiettivo
  - Identificare le cause possibili legate alla StorageClass

- Validazione
  - Usare i comandi di debug corretti
  - Elencare almeno 4 cause possibili

---
<details>
<summary>Soluzione</summary>

Comandi utili:

```sh
k get pvc
k describe pvc broken-sc-pvc
k get storageclass
k describe storageclass <storageclass-name>
k get pv
k get events --sort-by=.metadata.creationTimestamp
```

Cause possibili:
- `storageClassName` errata o inesistente
- nessuna default StorageClass disponibile
- provisioner assente o non supportato nel cluster
- il cluster non supporta provisioning dinamico
- accessMode non compatibile con il backend
- richiesta storage non soddisfabile
- driver CSI non installato o non funzionante
- `volumeBindingMode: WaitForFirstConsumer` e nessun Pod usa ancora il PVC

</details>

---
