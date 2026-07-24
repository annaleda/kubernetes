# Kubernetes Storage 

> Obiettivo CKAD: capire **come Kubernetes collega un Pod allo storage**, come eseguire troubleshooting e riconoscere le situazioni in cui è necessario ricreare le risorse.

---

# 1. Architettura dello Storage

Quando un Pod utilizza uno storage persistente il flusso è sempre:

```text
Pod
 │
 ▼
PersistentVolumeClaim (PVC)
 │
 ▼
PersistentVolume (PV)
 │
 ▼
Physical Storage
(hostPath, NFS, EBS, Azure Disk, Ceph...)
```

Il Pod **non monta mai direttamente un PV**.

Monta sempre un **PVC**.

Il PVC rappresenta la richiesta di storage.

Il PV rappresenta lo storage disponibile.

---

# 2. PersistentVolume (PV)

Un PV è una risorsa del cluster che rappresenta un volume.

Può essere:

* creato manualmente
* creato automaticamente tramite una StorageClass

È una risorsa **cluster-scoped**.

```bash
kubectl get pv
kubectl describe pv
```

---

## Esempio

```yaml
apiVersion: v1
kind: PersistentVolume

metadata:
  name: pv-data

spec:

  capacity:
    storage: 10Gi

  accessModes:
    - ReadWriteOnce

  storageClassName: standard

  hostPath:
    path: /data
```

---

# Campi importanti

## capacity

Dimensione del volume.

```yaml
capacity:
  storage: 10Gi
```

---

## accessModes

Specifica come il volume può essere montato.

I più comuni:

```text
ReadWriteOnce (RWO)

ReadWriteMany (RWX)

ReadOnlyMany (ROX)

ReadWriteOncePod (RWOP)
```

---

## storageClassName

Indica a quale StorageClass appartiene.

```yaml
storageClassName: standard
```

---

## persistentVolumeReclaimPolicy

Determina cosa succede quando viene eliminato il PVC.

```text
Delete

Retain
```

Molto importante negli esercizi.

---

# 3. PersistentVolumeClaim (PVC)

Il PVC rappresenta una richiesta di storage.

È una risorsa **namespaced**.

```bash
kubectl get pvc
kubectl describe pvc
```

---

## Esempio

```yaml
apiVersion: v1
kind: PersistentVolumeClaim

metadata:
  name: app-pvc

spec:

  accessModes:
    - ReadWriteOnce

  resources:
    requests:
      storage: 5Gi

  storageClassName: standard
```

---

# Campi importanti

## request

Quanto spazio richiede.

```yaml
resources:
  requests:
    storage: 5Gi
```

---

## accessModes

Devono essere compatibili con il PV.

---

## storageClassName

Deve corrispondere.

---

# 4. StorageClass

Una StorageClass descrive **come creare automaticamente** un PV.

```text
PVC

↓

StorageClass

↓

Provisioner

↓

PV
```

---

Esempio

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass

metadata:
  name: standard

provisioner: kubernetes.io/no-provisioner
```

---

Visualizzazione

```bash
kubectl get sc
kubectl describe sc
```

---

# 5. Binding

Il binding è il collegamento tra PVC e PV.

Kubernetes confronta:

```text
storageClassName

↓

accessModes

↓

capacity
```

Se tutto coincide:

```text
PVC

↓

Bound

↓

PV
```

---

Verifica

```bash
kubectl get pv,pvc
```

---

# 6. Come Kubernetes sceglie un PV

Esempio

PV

```yaml
capacity:
  storage: 10Gi
```

PVC

```yaml
requests:
  storage: 5Gi
```

Il binding è possibile.

---

Viceversa

PV

```text
5Gi
```

PVC

```text
10Gi
```

↓

Mai Bound.

---

Devono coincidere anche:

```text
accessModes

storageClassName
```

---

# 7. Access Modes

## ReadWriteOnce (RWO)

Può essere montato in lettura/scrittura da **un solo nodo**.

È il più comune.

---

## ReadWriteMany (RWX)

Più nodi possono leggere e scrivere.

Serve un backend che lo supporti.

---

## ReadOnlyMany (ROX)

Più nodi possono leggere.

---

## ReadWriteOncePod (RWOP)

Può essere montato da un solo Pod.

Disponibile solo con driver CSI compatibili.

---

# 8. Provisioning Statico

Il PV viene creato manualmente.

```text
Administrator

↓

PV

↓

PVC

↓

Pod
```

Esempio

```yaml
hostPath:

nfs:

local:
```

Molto frequente negli esercizi CKAD.

---

# 9. Provisioning Dinamico

Il PV viene creato automaticamente.

```text
PVC

↓

StorageClass

↓

Provisioner

↓

PV

↓

Bound
```

Non serve creare manualmente il PV.

---

# 10. Come un Pod usa un PVC

Il Pod monta il claim.

```yaml
volumes:

- name: storage

  persistentVolumeClaim:

    claimName: app-pvc
```

Container

```yaml
volumeMounts:

- mountPath: /data

  name: storage
```

Flusso

```text
Pod

↓

PVC

↓

PV

↓

Storage
```

---

# 11. hostPath

Molto comune negli esercizi.

```yaml
hostPath:
  path: /data
```

Va bene nei laboratori.

Non è consigliato in produzione.

---

# 12. emptyDir vs PVC

emptyDir

```text
vive finché esiste il Pod
```

PVC

```text
vive indipendentemente dal Pod
```

---

# 13. Checklist mentale

Quando leggi un esercizio chiediti subito:

```text
Il Pod usa un PVC?

↓

Il PVC è Bound?

↓

Esiste il PV?

↓

StorageClass corretta?

↓

Capacity sufficiente?

↓

AccessMode corretto?
```

---

# Comandi da sapere a memoria

```bash
kubectl get pv

kubectl get pvc

kubectl get sc

kubectl describe pv

kubectl describe pvc

kubectl describe sc

kubectl explain pv.spec

kubectl explain pvc.spec

kubectl explain storageclass
```
