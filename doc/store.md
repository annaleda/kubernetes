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

> Questa sezione è orientata al troubleshooting. Ogni volta che un esercizio parla di PVC, PV o StorageClass, è qui che devi ragionare.

---

# 14. Campi modificabili e immutabili

Una delle domande più frequenti è:

> Posso modificare questo PVC oppure devo ricrearlo?

## Campi modificabili

Se supportati dal backend:

```yaml
resources:
  requests:
    storage:
```

 Solo **aumentare** la dimensione.

Mai diminuirla.

---

## Campi immutabili

Non possono essere modificati dopo la creazione.

```text
storageClassName

accessModes

volumeMode

selector

volumeName
```

Se un esercizio ti chiede di cambiarli:

↓

Devi ricreare il PVC.

---

# 15. Resize del PVC

Questa è la trappola più comune.

Domanda da farsi immediatamente:

```text
Il backend supporta il resize?
```

---

## Caso A

StorageClass

```yaml
allowVolumeExpansion: true
```

e driver compatibile.

↓

Basta:

```bash
kubectl edit pvc app-pvc
```

e modificare

```yaml
resources:
  requests:
    storage: 10Gi
```

Fine.

---

## Caso B

StorageClass senza expansion

oppure

driver che non supporta resize

↓

Il resize fallisce.

Bisogna ricreare le risorse.

---

## Caso C (tipico CKAD)

PV creato manualmente.

Esempio

```text
PV
capacity: 20Gi

↓

Bound

↓

PVC

request: 5Gi
```

L'esercizio dice:

> Increase request to 10Gi.

Molti pensano:

> "Il PV è già da 20Gi."

↓

"Mi basta modificare il PVC."

Non è detto.

Il semplice fatto che il PV abbia spazio sufficiente **non implica** che il claim possa essere espanso.

L'espansione richiede il supporto del backend di storage.

Negli esercizi CKAD con storage statico (`hostPath`, `local`, ecc.) è frequente che il resize non sia supportato.

Il PVC può quindi rimanere:

```text
Pending
```

La soluzione pratica dell'esercizio è spesso:

```text
Delete Pod

↓

Delete PVC

↓

Ricreare (o aggiornare) il PV se richiesto

↓

Create nuovo PVC

↓

Bound

↓

Ricreare il Pod
```

 Non è il fatto che il PV abbia spazio disponibile a determinare il successo del resize.

Conta il supporto del backend.

---

# 16. Reclaim Policy

Campo del PV.

```yaml
persistentVolumeReclaimPolicy:
```

---

## Delete

```text
Delete
```

Quando elimini il PVC

↓

viene eliminato anche il volume.

Molto comune nel cloud.

---

## Retain

```text
Retain
```

Quando elimini il PVC

↓

il PV resta.

Anche i dati rimangono.

Molto usato negli esercizi.

---

## Controllo

```bash
kubectl describe pv
```

---

# 17. Volume Binding Mode

Campo della StorageClass.

```yaml
volumeBindingMode:
```

---

## Immediate

Il PV viene creato subito.

---

## WaitForFirstConsumer

Il PV viene creato solo quando esiste un Pod.

Flusso

```text
PVC

↓

Pending

↓

Pod

↓

Scheduler

↓

PV

↓

Bound
```

Non confondere questo comportamento con un errore.

---

# 18. allowVolumeExpansion

StorageClass

```yaml
allowVolumeExpansion: true
```

Significa:

Il backend **può** espandere il volume.

Non significa automaticamente che tutti i driver lo facciano.

---

# 19. Pending

Il PVC rimane Pending quando:

```text
Non esiste un PV compatibile

oppure

StorageClass errata

oppure

Provisioner assente

oppure

Capacity insufficiente

oppure

AccessMode incompatibile
```

---

# 20. Pod Pending

Molti pensano subito al scheduler.

Errore.

Prima controllare sempre:

```text
Pod

↓

PVC

↓

PV
```

Se il PVC è Pending

↓

anche il Pod rimane Pending.

---

# 21. Troubleshooting

Ordine corretto.

```text
Pod

↓

PVC

↓

PV

↓

StorageClass

↓

Events
```

Mai partire dal PV.

---

Comandi

```bash
kubectl get pod

kubectl describe pod

kubectl get pvc

kubectl describe pvc

kubectl get pv

kubectl describe pv

kubectl get sc

kubectl get events
```

---

# 22. Errori tipici CKAD

## PVC Pending

Controllare:

```text
StorageClass

Capacity

AccessModes

Events
```

---

## Pod Pending

Controllare:

```text
PVC
```

prima di tutto.

---

## Bound sbagliato

Verificare

```yaml
storageClassName

accessModes
```

---

## Resize

Domanda mentale

```text
Il backend supporta expansion?
```

Se no

↓

probabilmente bisogna ricreare le risorse.

---

# 23. Checklist da esame

Quando leggi

> Application cannot start

fai immediatamente:

```text
Pod

↓

PVC

↓

PV

↓

StorageClass

↓

Events
```

---

Quando leggi

> Increase PVC size

Domandati:

```text
StorageClass?

↓

allowVolumeExpansion?

↓

Driver supporta resize?

↓

Campo modificabile?

↓

Serve ricreare il PVC?
```

---


✅ Il Pod usa sempre un PVC.

✅ Il PVC si lega a un solo PV.

✅ Un PV può essere creato manualmente o dinamicamente.

✅ Non puoi diminuire la dimensione di un PVC.

✅ I campi come `storageClassName` e `accessModes` sono immutabili.

✅ Se il backend non supporta l'espansione, modificare `requests.storage` non basta.

✅ Prima di eliminare un PVC controlla sempre la `persistentVolumeReclaimPolicy`.

✅ Quando un Pod è `Pending`, controlla sempre il PVC prima di cercare altri problemi.

