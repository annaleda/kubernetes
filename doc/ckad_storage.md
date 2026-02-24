# CKAD Storage

## Teoria

---

## Volumes

I volumi permettono ai container di:

- Condividere dati tra container nello stesso Pod
- Persistire dati oltre il ciclo di vita del container
- Montare configurazioni o segreti

Un volume è definito nel Pod e montato nei container.


<img width="1017" height="600" alt="Immagine 2026-02-24 225608" src="https://github.com/user-attachments/assets/6eca54d4-9e0f-4cda-af3e-6ab86a9defac" />

---

## Tipi di Volume Comuni (CKAD Focus)

### emptyDir

- Creato quando il Pod viene avviato
- Eliminato quando il Pod viene distrutto
- Usato per:
  - Condivisione dati tra container
  - Storage temporaneo
- Non è persistente

Esempio:

```yaml
volumes:
  - name: temp-storage
    emptyDir: {}
```

---

### hostPath

Il volume **hostPath** monta un file o una directory **direttamente dal filesystem del nodo** dentro il Pod.

È uno storage a **livello di nodo**, NON a livello di cluster.

Questo significa che:
- I dati esistono solo su quel nodo
- Se il Pod viene schedulato su un altro nodo → i dati non saranno presenti
- Non è una soluzione portabile o cloud-native

---

### Come funziona il mount

1. Il volume viene definito nella sezione `volumes`
2. Viene montato nel container tramite `volumeMounts`
3. Il path del nodo viene "collegato" a un path interno al container

---

### Esempio hostPath

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hostpath-example
spec:
  containers:
  - name: app
    image: nginx
    volumeMounts:
    - name: node-storage
      mountPath: /data
  volumes:
  - name: node-storage
    hostPath:
      path: /var/data
      type: Directory
```
---

### configMap

- Monta una ConfigMap come file
- Ogni chiave diventa un file

---

### secret

- Monta un Secret come file
- I dati vengono decodificati automaticamente

---

## Persistenza dei Dati

Per persistenza oltre il ciclo di vita del Pod si usano:

- PersistentVolume (PV)
- PersistentVolumeClaim (PVC)

---

## PersistentVolume (PV)

- Rappresenta lo storage fisico nel cluster
- Può essere:
  - Provisionato manualmente
  - Creato dinamicamente tramite StorageClass
- È una risorsa cluster-wide

Caratteristiche:
- capacity
- accessModes
- persistentVolumeReclaimPolicy

Access Modes comuni:
- `ReadWriteOnce` (RWO)
- `ReadOnlyMany` (ROX)
- `ReadWriteMany` (RWX)

---

## PersistentVolumeClaim (PVC)

- Richiesta di storage da parte di un Pod
- Specifica:
  - Dimensione richiesta
  - Access mode
  - StorageClass (opzionale)

Esempio:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

---

## Montare un PVC in un Pod

```yaml
volumes:
  - name: my-storage
    persistentVolumeClaim:
      claimName: my-pvc
```

Nel container:

```yaml
volumeMounts:
  - mountPath: /data
    name: my-storage
```

---

## StorageClass

- Permette provisioning dinamico dei volumi
- Automatizza la creazione di PV
- Definisce:
  - Tipo di storage (es. SSD, HDD)
  - Parametri del provisioner

Se un PVC specifica una StorageClass, il PV viene creato automaticamente.

---

## Static vs Dynamic Provisioning

| Static | Dynamic |
|--------|----------|
| PV creato manualmente | PV creato automaticamente |
| Amministratore definisce storage | StorageClass gestisce provisioning |
| Meno flessibile | Più usato in produzione |

---

## Reclaim Policy

Definisce cosa succede al PV quando il PVC viene eliminato:

- `Retain` → mantiene i dati
- `Delete` → elimina volume
- `Recycle` (deprecato)

---

## Volume Lifecycle

- Container restart → dati persistono
- Pod restart → dati persistono se PVC
- Pod deletion:
  - emptyDir → perso
  - PVC → dati persistono

---

## Stateful Applications

Per applicazioni con stato si usa spesso:

- StatefulSet
- PVC dedicato per ogni replica
- Headless Service

---

## Esercizi

1. Creare un Pod con volume emptyDir e verificare condivisione dati.
2. Creare un PersistentVolumeClaim da 1Gi.
3. Montare il PVC in un Pod.
4. Scrivere dati nel volume e riavviare il Pod per verificare la persistenza.
5. Eliminare il Pod e verificare che i dati persistano.
6. Creare una StorageClass (se supportata dal cluster).
7. Verificare accessModes supportati dal cluster.

---



