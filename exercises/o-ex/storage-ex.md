

### Storage (6 esercizi)
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
```
</details>

---
