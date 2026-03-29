- [ Home ](../../readme.md)   | [ Teoria ](../../arguments.md)   | [ Info Exam ](../../doc/ckad_exam_strategy.md)    | [ Home Other Exercises ](../o_exercises.md)   
--- 

### Storage (23 esercizi)
---

## ST-1 — Static PV + PVC

- PersistentVolume
  - Nome: `pv-static`
  - Storage: `100Mi`
  - AccessMode: ReadWriteOnce
  - hostPath: `/mnt/static`

- PersistentVolumeClaim
  - Nome: `pvc-static`
  - Richiede: `100Mi`

- Pod
  - Nome: `static-consumer`
  - Image: nginx
  - Monta PVC in /data

- Validazione
  - PVC Bound
  - Pod Running

---
<details>
<summary>Soluzione</summary>
  
```
vi pv.yaml


apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-static
spec:
  capacity:
    storage: 100Mi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: /mnt/static

vi pvc.yaml


apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-static
spec:
  storageClassName: ""
  accessModes:
    - ReadWriteOnce
  volumeMode: Filesystem
  resources:
    requests:
      storage: 100Mi

k run static-consumer --image=nginx --dry-run=client -o yaml > static-consumer.yaml


vi static-consumer.yaml


apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: static-consumer
  name: static-consumer
spec:
  containers:
  - image: nginx
    name: static-consumer
    resources: {}
    volumeMounts:
    - mountPath: /data
      name: pvc-static
  dnsPolicy: ClusterFirst
  restartPolicy: Always
  volumes:
    - name: pvc-static
      persistentVolumeClaim:
        claimName: pvc-static
status: {}
PS C:\Users\annaleda\ckad\o-ex\1\3>
```
</details>

---

## ST-2 — Multiple PVC in Pod

- PVC
  - pvc-config
  - pvc-data

- Pod: `multi-volume-app`
  - Monta pvc-config in /config
  - Monta pvc-data in /data

- Validazione
  - Entrambi i volumi montati correttamente

---
<details>
<summary>Soluzione</summary>
  
```
 k get storageclass

vi pvc-config.yaml

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-config
spec:
  storageClassName: ""
  accessModes:
    - ReadWriteOnce
  volumeMode: Filesystem
  resources:
    requests:
      storage: 5Gi


vi pvc-data.yaml


apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-data
spec:
  storageClassName: ""
  accessModes:
    - ReadWriteOnce
  volumeMode: Filesystem
  resources:
    requests:
      storage: 8Gi

k run multi-volume-app --image=nginx --dry-run=client -o yaml >  multi-volume-app.yaml

apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: multi-volume-app
  name: multi-volume-app
spec:
  containers:
  - image: nginx
    name: multi-volume-app
    resources: {}
    volumeMounts:
    - name: pvc-config
      mountPath: /config
    - name: pvc-data
      mountPath: /data
  dnsPolicy: ClusterFirst
  restartPolicy: Always
  volumes:
    - name: pvc-config
      persistentVolumeClaim:
        claimName: pvc-config
    - name: pvc-data
      persistentVolumeClaim:
        claimName: pvc-data
status: {}


```
</details>

---

## ST-3 — ReadWriteMany Scenario

- PersistentVolume
  - Storage: `200Mi`
  - AccessMode: ReadWriteMany
  - hostPath: `/mnt/shared`

- PVC collegato al PV
  - Pod 1: shared-1
  - Pod 2: shared-2

Entrambi montano stesso PVC

- Validazione
  - Entrambi Running
  - File scritto da uno visibile nell’altro

---
<details>
<summary>Soluzione</summary>
  
```
vi pv-many.yaml


apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-many
spec:
  capacity:
    storage: 200Mi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteMany
  hostPath:
    path: /mnt/shared

k apply -f pv-many.yaml


vi pvc-many.yaml


apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-many
spec:
  storageClassName: ""
  accessModes:
    - ReadWriteMany
  volumeMode: Filesystem
  resources:
    requests:
      storage: 200Mi


k apply -f pvc-many.yaml


k run shared-2 --image=nginx --dry-run=client -o yaml > shared-2.yaml

k run shared-1 --image=nginx --dry-run=client -o yaml > shared-1.yaml

vi shared-1.yaml


apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: shared-1
  name: shared-1
spec:
  containers:
  - image: nginx
    name: shared-1
    resources: {}
    volumeMounts:
    - name: pvc-many
      mountPath: /many
  dnsPolicy: ClusterFirst
  restartPolicy: Always
  volumes:
  - name: pvc-many
    persistentVolumeClaim:
      claimName: pvc-many
status: {}

k apply -f shared-1.yaml

 vi shared-2.yaml


apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: shared-2
  name: shared-2
spec:
  containers:
  - image: nginx
    name: shared-2
    resources: {}
    volumeMounts:
    - name: pvc-many
      mountPath: /many
  dnsPolicy: ClusterFirst
  restartPolicy: Always
  volumes:
  - name: pvc-many
    persistentVolumeClaim:
      claimName: pvc-many
status: {}

k apply -f shared-2.yaml

kubectl exec -it shared-2 -- cat /many/test.txt
```
Altra soluzione:

```
vi pv-many.yaml


apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-shared
spec:
  capacity:
    storage: 200Mi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteMany
  hostPath:
    path: /mnt/shared

k apply -f pv-many.yaml


vi pvc-many.yaml


apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-shared
spec:
  storageClassName: ""
  accessModes:
    - ReadWriteMany
  volumeMode: Filesystem
  resources:
    requests:
      storage: 200Mi


k apply -f pvc-many.yaml


k run shared-2 --image=nginx --dry-run=client -o yaml > shared-2.yaml

k run shared-1 --image=nginx --dry-run=client -o yaml > shared-1.yaml

vi shared-1.yaml


apiVersion: v1
kind: Pod
metadata:
  labels:
    run: shared-2
  name: shared-2
spec:
  volumes:
  - name: pvc-shared
    persistentVolumeClaim:
      claimName: pvc-shared
  containers:
  - image: busybox
    name: shared-2
    command: ["/bin/sh","-c"]
    args:
      - while true; do echo "fuck" >> /pvc/app.log; sleep 5; done
    volumeMounts:
    - name: pvc-shared
      mountPath: /pvc
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}

k apply -f shared-1.yaml

 vi shared-2.yaml


apiVersion: v1
kind: Pod
metadata:
  labels:
    run: shared-1
  name: shared-1
spec:
  volumes:
  - name: pvc-shared
    persistentVolumeClaim:
      claimName: pvc-shared
  containers:
  - image: busybox
    name: shared-1
    command: ["/bin/sh","-c"]
    args:
    - touch /pvc/app.log; while true;do cat /pvc/app.log; sleep 5; done
    volumeMounts:
    - name: pvc-shared
      mountPath: /pvc
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}


k apply -f shared-2.yaml

 k logs shared-1

```
</details>

---

## ST-4 — SubPath Mount

- Pod: `subpath-app`

- PVC: `pvc-static`

- Configurazione
  - Montare solo subPath logs in /var/log/app

- Validazione
  - Solo la directory logs è montata

---
<details>
<summary>Soluzione</summary>
  
```
k run subpath-app --image=nginx --dry-run=client -o yaml > subpath-app.yaml

spec:
  containers:
  - name: subpath-app
    ....
    volumeMounts:
    - mountPath: /var/log/app
      name: storage
      subPath: logs

  volumes:
  - name: storage
    persistentVolumeClaim:
      claimName: pvc-static

kubectl exec -it subpath-app -- ls /var/log/app
```
</details>

---

## ST-5 — StorageClass Usage

- PVC: `dynamic-storage`
- Storage: `150Mi`
- StorageClass: standard
- Pod: dynamic-consumer
  - Image: busybox
  - Monta volume in `/data`
  - Scrive file `test.txt`

-Validazione
- PVC Bound automaticamente

---
<details>
<summary>Soluzione</summary>
  
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: dynamic-storage                  # claim name
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: standard
  resources:
    requests:
      storage: 150Mi


apiVersion: v1
kind: Pod
metadata:
  name: dynamic-consumer
spec:
  containers:
  - name: app
    image: busybox
    command: ["/bin/sh", "-c", "echo hello > /data/test.txt && sleep 3600"]
    volumeMounts:
    - name: storage
      mountPath: /data
  volumes:
  - name: storage
    persistentVolumeClaim:
      claimName: dynamic-storage          # claim name


kubectl get pvc
```
</details>

---

## ST-6 — VolumeMount ReadOnly

- Pod: `readonly-app`

- PVC: `pvc-static`

- Configurazione
  - volumeMount readOnly: true

- Validazione
  - Tentativo di scrittura fallisce

---
<details>
<summary>Soluzione</summary>
  
```

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pv-static
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 150Mi



apiVersion: v1
kind: Pod
metadata:
  name: readonly-app
spec:
  containers:
  - name: app
    image: busybox
    command: ["/bin/sh", "-c", "sleep 3600"]
    volumeMounts:
    - name: storage
      mountPath: /data
      readOnly: true
  volumes:
  - name: storage
    persistentVolumeClaim:
      claimName: pvc-static
```
</details>

---
## ST-7 — PVC in Deployment

- PVC
  - Nome: `web-pvc`
  - Storage: `200Mi`
  - AccessMode: ReadWriteOnce
  - StorageClass: standard

- Deployment
  - Nome: `web-storage-deploy`
  - Replicas: 1
  - Image: nginx
  - Monta il PVC in `/usr/share/nginx/html`

- Validazione
  - PVC Bound
  - Pod Running
  - Il Pod del Deployment monta correttamente il volume

---

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: web-pvc
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: standard
  resources:
    requests:
      storage: 200Mi
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-storage-deploy
spec:
  replicas: 1
  selector:
    matchLabels:
      app: web-storage
  template:
    metadata:
      labels:
        app: web-storage
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
          claimName: web-pvc
```

```sh
k apply -f web-storage-deploy.yaml
k get pvc
k get po
k describe pod <pod-name>
```

</details>

---

## ST-8 — emptyDir tra due container

- Pod: `shared-emptydir-pod`

- Container 1
  - Name: `writer`
  - Image: busybox
  - Scrive ogni 5 secondi in `/shared/data.txt`

- Container 2
  - Name: `reader`
  - Image: busybox
  - Legge continuamente `/shared/data.txt`

- Configurazione
  - Usare `emptyDir`

- Validazione
  - I due container condividono i dati
  - Il reader vede quello che scrive il writer

---

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: shared-emptydir-pod
spec:
  volumes:
  - name: shared-vol
    emptyDir: {}

  containers:
  - name: writer
    image: busybox
    command:
    - sh
    - -c
    - while true; do date >> /shared/data.txt; sleep 5; done
    volumeMounts:
    - name: shared-vol
      mountPath: /shared

  - name: reader
    image: busybox
    command:
    - sh
    - -c
    - touch /shared/data.txt; tail -f /shared/data.txt
    volumeMounts:
    - name: shared-vol
      mountPath: /shared
```

```sh
k apply -f shared-emptydir-pod.yaml
k logs shared-emptydir-pod -c reader
```

</details>

---

## ST-9 — ConfigMap come volume

- ConfigMap
  - Nome: `app-config`
  - Chiave: `settings.conf`
  - Valore: `mode=prod`

- Pod
  - Nome: `configmap-volume-pod`
  - Image: busybox
  - Monta il ConfigMap in `/etc/config`

- Validazione
  - Il file `/etc/config/settings.conf` esiste nel container

---

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  settings.conf: |
    mode=prod
---
apiVersion: v1
kind: Pod
metadata:
  name: configmap-volume-pod
spec:
  containers:
  - name: app
    image: busybox
    command:
    - sh
    - -c
    - sleep 3600
    volumeMounts:
    - name: config-vol
      mountPath: /etc/config
  volumes:
  - name: config-vol
    configMap:
      name: app-config
```

```sh
k apply -f configmap-volume-pod.yaml
k exec -it configmap-volume-pod -- ls /etc/config
k exec -it configmap-volume-pod -- cat /etc/config/settings.conf
```

</details>

---

## ST-10 — Secret come volume

- Secret
  - Nome: `db-secret`
  - Chiavi:
    - `username=admin`
    - `password=changeme`

- Pod
  - Nome: `secret-volume-pod`
  - Image: busybox
  - Monta il Secret in `/etc/secret`

- Validazione
  - I file del secret esistono
  - Il Pod è Running

---

<details>
<summary>Soluzione</summary>

```sh
k create secret generic db-secret \
  --from-literal=username=admin \
  --from-literal=password=changeme \
  --dry-run=client -o yaml > db-secret.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-volume-pod
spec:
  containers:
  - name: app
    image: busybox
    command:
    - sh
    - -c
    - sleep 3600
    volumeMounts:
    - name: secret-vol
      mountPath: /etc/secret
      readOnly: true
  volumes:
  - name: secret-vol
    secret:
      secretName: db-secret
```

```sh
k apply -f db-secret.yaml
k apply -f secret-volume-pod.yaml
k exec -it secret-volume-pod -- ls /etc/secret
k exec -it secret-volume-pod -- cat /etc/secret/username
```

</details>

---

## ST-11 — hostPath Volume

- Pod
  - Nome: `hostpath-pod`
  - Image: busybox

- Volume
  - Tipo: `hostPath`
  - Path host: `/tmp/hostdata`
  - Mount nel container: `/data/host`

- Command
  - Scrivere `hello from hostPath` nel file `/data/host/test.txt`
  - Poi andare in sleep

- Validazione
  - Pod Running
  - Il file viene creato nel volume montato

---

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hostpath-pod
spec:
  containers:
  - name: app
    image: busybox
    command:
    - sh
    - -c
    - echo "hello from hostPath" > /data/host/test.txt && sleep 3600
    volumeMounts:
    - name: host-vol
      mountPath: /data/host
  volumes:
  - name: host-vol
    hostPath:
      path: /tmp/hostdata
      type: DirectoryOrCreate
```

```sh
k apply -f hostpath-pod.yaml
k exec -it hostpath-pod -- cat /data/host/test.txt
```

</details>

---

## ST-12 — emptyDir con limite di memoria

- Pod: `memory-emptydir-pod`

- Volume
  - Tipo: `emptyDir`
  - medium: `Memory`

- Container
  - Image: busybox
  - Scrive file in `/cache`

- Validazione
  - Il volume è montato correttamente
  - `emptyDir` usa memoria

---

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: memory-emptydir-pod
spec:
  containers:
  - name: app
    image: busybox
    command:
    - sh
    - -c
    - echo "cached data" > /cache/data.txt && sleep 3600
    volumeMounts:
    - name: cache-vol
      mountPath: /cache
  volumes:
  - name: cache-vol
    emptyDir:
      medium: Memory
```

```sh
k apply -f memory-emptydir-pod.yaml
k exec -it memory-emptydir-pod -- ls /cache
```

</details>

---

## ST-13 — PersistentVolume con reclaim policy Delete

- PersistentVolume
  - Nome: `pv-delete`
  - Storage: `100Mi`
  - AccessMode: ReadWriteOnce
  - hostPath: `/mnt/delete-test`
  - reclaimPolicy: `Delete`

- Validazione
  - PV creato con reclaim policy corretta

---

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-delete
spec:
  capacity:
    storage: 100Mi
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Delete
  hostPath:
    path: /mnt/delete-test
```

```sh
k apply -f pv-delete.yaml
k get pv pv-delete
```

</details>

---

## ST-14 — PVC con accessMode ReadOnlyMany

- PersistentVolumeClaim
  - Nome: `readonly-claim`
  - Storage: `50Mi`
  - AccessMode: ReadOnlyMany

- Obiettivo
  - Creare il PVC con modalità corretta

- Validazione
  - `kubectl get pvc readonly-claim`

---

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: readonly-claim
spec:
  accessModes:
  - ReadOnlyMany
  resources:
    requests:
      storage: 50Mi
```

```sh
k apply -f readonly-claim.yaml
k get pvc readonly-claim
```

</details>

---

## ST-15 — Pod con due emptyDir distinti

- Pod: `dual-emptydir-pod`

- Volume 1
  - Nome: `logs-vol`
  - Mount: `/logs`

- Volume 2
  - Nome: `cache-vol`
  - Mount: `/cache`

- Container
  - Image: busybox

- Validazione
  - Entrambi i volumi sono montati

---

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: dual-emptydir-pod
spec:
  containers:
  - name: app
    image: busybox
    command:
    - sh
    - -c
    - mkdir -p /logs /cache && touch /logs/app.log /cache/data.txt && sleep 3600
    volumeMounts:
    - name: logs-vol
      mountPath: /logs
    - name: cache-vol
      mountPath: /cache
  volumes:
  - name: logs-vol
    emptyDir: {}
  - name: cache-vol
    emptyDir: {}
```

```sh
k apply -f dual-emptydir-pod.yaml
k exec -it dual-emptydir-pod -- ls /logs /cache
```

</details>

---

## ST-16 — ConfigMap con singolo file via subPath

- ConfigMap
  - Nome: `single-file-config`
  - Chiave: `app.properties`

- Pod
  - Nome: `subpath-configmap-pod`
  - Image: busybox

- Configurazione
  - Montare solo il file `app.properties` in `/etc/app.properties`

- Validazione
  - Il file è montato nel path richiesto

---

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: single-file-config
data:
  app.properties: |
    color=blue
---
apiVersion: v1
kind: Pod
metadata:
  name: subpath-configmap-pod
spec:
  containers:
  - name: app
    image: busybox
    command:
    - sh
    - -c
    - cat /etc/app.properties && sleep 3600
    volumeMounts:
    - name: config-vol
      mountPath: /etc/app.properties
      subPath: app.properties
  volumes:
  - name: config-vol
    configMap:
      name: single-file-config
```

```sh
k apply -f subpath-configmap-pod.yaml
k exec -it subpath-configmap-pod -- cat /etc/app.properties
```

</details>

---

## ST-17 — Secret con chiave singola via items

- Secret
  - Nome: `api-secret`
  - Chiavi:
    - token
    - user

- Pod
  - Nome: `secret-items-pod`
  - Image: busybox

- Configurazione
  - Montare solo la chiave `token` come file `api-token`

- Validazione
  - Solo il file richiesto è presente

---

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: api-secret
type: Opaque
stringData:
  token: abc123
  user: admin
---
apiVersion: v1
kind: Pod
metadata:
  name: secret-items-pod
spec:
  containers:
  - name: app
    image: busybox
    command:
    - sh
    - -c
    - ls /etc/secret && cat /etc/secret/api-token && sleep 3600
    volumeMounts:
    - name: secret-vol
      mountPath: /etc/secret
      readOnly: true
  volumes:
  - name: secret-vol
    secret:
      secretName: api-secret
      items:
      - key: token
        path: api-token
```

```sh
k apply -f secret-items-pod.yaml
k exec -it secret-items-pod -- ls /etc/secret
```

</details>

---

## ST-18 — PVC con selector

- PersistentVolume
  - Nome: `labeled-pv`
  - Label: `type=fast`
  - Storage: `100Mi`

- PersistentVolumeClaim
  - Nome: `selected-pvc`
  - Selector: `type=fast`

- Validazione
  - Il PVC si lega al PV con label corretta

---

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: labeled-pv
  labels:
    type: fast
spec:
  capacity:
    storage: 100Mi
  accessModes:
  - ReadWriteOnce
  hostPath:
    path: /mnt/labeled
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: selected-pvc
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 100Mi
  selector:
    matchLabels:
      type: fast
```

```sh
k apply -f selected-pvc.yaml
k get pvc selected-pvc
k get pv labeled-pv --show-labels
```

</details>

---

## ST-19 — Pod con projected volume

- Pod: `projected-pod`

- Configurazione
  - Unire in un solo volume:
    - ConfigMap `proj-config`
    - Secret `proj-secret`

- Mount: `/projected`

- Validazione
  - Entrambi i contenuti sono presenti

---

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: proj-config
data:
  app.conf: "mode=test"
---
apiVersion: v1
kind: Secret
metadata:
  name: proj-secret
type: Opaque
stringData:
  password: changeme
---
apiVersion: v1
kind: Pod
metadata:
  name: projected-pod
spec:
  containers:
  - name: app
    image: busybox
    command:
    - sh
    - -c
    - ls /projected && sleep 3600
    volumeMounts:
    - name: projected-vol
      mountPath: /projected
  volumes:
  - name: projected-vol
    projected:
      sources:
      - configMap:
          name: proj-config
      - secret:
          name: proj-secret
```

```sh
k apply -f projected-pod.yaml
k exec -it projected-pod -- ls /projected
```

</details>

---

## ST-20 — emptyDir condiviso con 3 container

- Pod: `three-share-pod`

- Container 1
  - scrive `/shared/a.txt`

- Container 2
  - scrive `/shared/b.txt`

- Container 3
  - legge entrambi

- Validazione
  - Il terzo container vede i file creati dagli altri due

---

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: three-share-pod
spec:
  containers:
  - name: writer-a
    image: busybox
    command:
    - sh
    - -c
    - echo "file a" > /shared/a.txt && sleep 3600
    volumeMounts:
    - name: shared-vol
      mountPath: /shared

  - name: writer-b
    image: busybox
    command:
    - sh
    - -c
    - echo "file b" > /shared/b.txt && sleep 3600
    volumeMounts:
    - name: shared-vol
      mountPath: /shared

  - name: reader
    image: busybox
    command:
    - sh
    - -c
    - sleep 20
    volumeMounts:
    - name: shared-vol
      mountPath: /shared

  volumes:
  - name: shared-vol
    emptyDir: {}
```

```sh
k apply -f three-share-pod.yaml
k exec -it three-share-pod -c reader -- ls /shared
```

</details>

---

## ST-21 — Pod con ConfigMap readOnly

- ConfigMap
  - Nome: `readonly-config`

- Pod
  - Nome: `readonly-config-pod`
  - Image: busybox

- Configurazione
  - Montare ConfigMap in `/etc/config`
  - Tentativo di scrittura deve fallire

- Validazione
  - Scrittura non consentita

---

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: readonly-config
data:
  settings.conf: "readonly=true"
---
apiVersion: v1
kind: Pod
metadata:
  name: readonly-config-pod
spec:
  containers:
  - name: app
    image: busybox
    command:
    - sh
    - -c
    - sleep 3600
    volumeMounts:
    - name: config-vol
      mountPath: /etc/config
      readOnly: true
  volumes:
  - name: config-vol
    configMap:
      name: readonly-config
```

```sh
k apply -f readonly-config-pod.yaml
k exec -it readonly-config-pod -- sh
```

</details>

---

## ST-22 — PVC montato con subPath

- PVC: `pvc-static`

- Pod
  - Nome: `subpath-pvc-pod`
  - Image: busybox

- Configurazione
  - Montare solo subPath `data` in `/app/data`

- Validazione
  - Il mount usa subPath del PVC

---

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: subpath-pvc-pod
spec:
  containers:
  - name: app
    image: busybox
    command:
    - sh
    - -c
    - mkdir -p /app/data && sleep 3600
    volumeMounts:
    - name: storage
      mountPath: /app/data
      subPath: data
  volumes:
  - name: storage
    persistentVolumeClaim:
      claimName: pvc-static
```

```sh
k apply -f subpath-pvc-pod.yaml
```

</details>

---

## ST-23 — Debug PVC Pending

- Scenario
  - PVC `broken-pvc` resta in stato Pending

- Obiettivo
  - Identificare le possibili cause

- Validazione
  - Usare comandi di debug corretti

---

<details>
<summary>Soluzione</summary>

Comandi utili:

```sh
k get pvc
k describe pvc broken-pvc
k get pv
k get storageclass
```

Cause possibili:
- nessun PV disponibile
- storageClassName errata
- accessMode non compatibile
- richiesta storage troppo alta
- provisioner dinamico assente
- selector PVC che non matcha nessun PV

</details>

---
