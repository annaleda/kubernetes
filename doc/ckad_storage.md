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

## Condivisione Storage

I container possono condividere volumi definiti nel Pod.

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

Montaggio nei container:

```yaml
volumeMounts:
  - name: shared-data
    mountPath: /data
```

---
<img width="722" height="534" alt="Immagine 2026-02-25 130738" src="https://github.com/user-attachments/assets/5bfbf9b4-c795-44b8-8d6b-bf1e5dca7c7d" />


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

Il PersistentVolume rappresenta lo storage fisico disponibile nel cluster.

È una risorsa cluster-wide, indipendente dal Pod o dal Namespace.

Può essere:

Provisionato manualmente dall’amministratore

Creato dinamicamente tramite StorageClass 

Supportato da storage backend esterni (cloud, NFS, ecc.)

> Il PV vive indipendentemente dal ciclo di vita del Pod.

| Campo                           | Descrizione                                 |
| ------------------------------- | ------------------------------------------- |
| `capacity`                      | Dimensione dello storage disponibile        |
| `accessModes`                   | Modalità di accesso al volume               |
| `persistentVolumeReclaimPolicy` | Comportamento quando il PVC viene eliminato |
| `storageClassName`              | Associa PV a una StorageClass               |
| `hostPath / nfs / csi`          | Backend storage utilizzato                  |

---
`accessModes` 

| Mode                  | Significato                                 |
| --------------------- | ------------------------------------------- |
| `ReadWriteOnce (RWO)` | Montabile in **read/write da un solo nodo** |
| `ReadOnlyMany (ROX)`  | Montabile in sola lettura da più nodi       |
| `ReadWriteMany (RWX)` | Montabile in read/write da più nodi         |


`persistentVolumeReclaimPolicy`

| Policy     | Comportamento                               |
| ---------- | ------------------------------------------- |
| `Retain`   | Mantiene i dati (richiede cleanup manuale)  |
| `Delete`   | Cancella automaticamente lo storage backend |

**Binding tra PV e PVC**

Il PV viene assegnato a un PVC solo se:

- Capacity compatibile
- AccessMode compatibile
- StorageClass compatibile (se specificata)

Se non esiste PV compatibile → il PVC rimane in Pending.

<img width="993" height="612" alt="Immagine 2026-02-24 230354" src="https://github.com/user-attachments/assets/02a863db-ad09-4675-b0b0-62ff8a98969b" />

---

## PersistentVolumeClaim (PVC)
Il PersistentVolumeClaim rappresenta una richiesta di storage effettuata da un Pod.

> È il livello di astrazione tra Pod e storage fisico.

Il PVC permette al Pod di non conoscere i dettagli dell’infrastruttura storage.

Un PVC specifica:
  - Dimensione dello storage richiesto
  - Modalità di accesso al volume
  - StorageClass (opzionale)
  - Binding verso un PersistentVolume compatibile

**Binding PVC → PV**

Kubernetes prova automaticamente a trovare un PV che soddisfi la richiesta.

Il binding avviene quando:
  - Capacity è sufficiente
  - AccessMode è compatibile
  - StorageClass coincide (se specificata)

> Se non trova un PV compatibile → il PVC resta in Pending state

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
<img width="1090" height="500" alt="Immagine 2026-02-24 230559" src="https://github.com/user-attachments/assets/5718d328-02aa-4a59-87ef-23ca3a19efd4" />

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
Esempio:

<img width="390" height="454" alt="Immagine 2026-02-26 033143" src="https://github.com/user-attachments/assets/f4a84d4e-25bc-4d84-9775-ecf14195bf84" />

<img width="1223" height="610" alt="Immagine 2026-02-24 231025" src="https://github.com/user-attachments/assets/20163d66-dbf4-4445-b638-b0b284a6ee51" />

---

## StorageClass

| Parametro              | Obbligatorio | Valori possibili                                                                                                             | Descrizione                                                 |
| ---------------------- | ------------ | ---------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------- |
  | `provisioner`          | ✅            | `kubernetes.io/aws-ebs`<br>`kubernetes.io/no-provisioner`<br>`kubernetes.io/gce-pd`<br>`kubernetes.io/cinder`<br>         | Definisce il backend che crea lo storage                    |
| `volumeBindingMode`    | ❌            | `Immediate`<br>`WaitForFirstConsumer`                                                                                        | Determina quando il PVC viene bound al PV                   |
| `reclaimPolicy`        | ❌            | `Retain`<br>`Delete`                                                                                                         | Decide cosa succede al volume quando il PVC viene eliminato |
| `allowVolumeExpansion` | ❌            | `true` / `false`                                                                                                             | Permette o meno l’espansione dinamica del volume            |
| `parameters`           | ❌            | Dipende dal provisioner                                                                                                      | Configurazioni specifiche del backend                       |

- Permette provisioning dinamico dei volumi
- Automatizza la creazione di PV
- Definisce:
  - Tipo di storage (es. SSD, HDD)
  - Parametri del provisioner

Se un PVC specifica una StorageClass, il PV viene creato automaticamente.

<img width="1229" height="638" alt="Immagine 2026-02-24 231300" src="https://github.com/user-attachments/assets/e68d5ca8-59a7-48b3-9cc5-a88ba6f404cb" />

---

## Static vs Dynamic Provisioning

| Static | Dynamic |
|--------|----------|
| PV creato manualmente | PV creato automaticamente |
| Amministratore definisce storage | StorageClass gestisce provisioning |
| Meno flessibile | Più usato in produzione |

---

## Volume Lifecycle

- Container restart → dati persistono
- Pod restart → dati persistono se PVC
- Pod deletion:
  - emptyDir → perso
  - PVC → dati persistono

In Kubernetes, i PersistentVolume (PV) rappresentano storage esterno al Pod e al ciclo di vita del container.

I PV sono risorse di storage indipendenti, spesso supportate da sistemi esterni come NFS, cloud block storage o altri backend, e permettono ai dati di sopravvivere alla distruzione dei Pod.

<img width="1147" height="453" alt="Immagine 2026-02-25 131210" src="https://github.com/user-attachments/assets/ffb1edc1-f986-49d1-aa26-1d3faa8bdb15" />


---

## Stateful Applications

Le applicazioni stateful sono quelle che devono conservare lo stato tra restart, scaling o ricreazione dei Pod.

Esempi tipici:

- Database
- Cache distribuite
- Messaging systems
- Storage persistente per servizi specifici

Quando usare StatefulSet

Si usa StatefulSet quando servono:

- Identità stabile dei Pod
- Storage persistente dedicato
- Ordinal indexing dei Pod
- Network identity stabile
  
Per applicazioni con stato si usa spesso:

- StatefulSet
- PVC dedicato per ogni replica
- Headless Service

| Feature      | Deployment         | StatefulSet               |
| ------------ | ------------------ | ------------------------- |
| Pod Identity | Random             | Stable (es. web-0, web-1) |
| Storage      | Shared o stateless | PVC dedicato per replica  |
| Scaling      | Parallel           | Ordinal order             |
| Network      | Service normale    | Headless Service          |
| Use case     | Stateless apps     | Stateful apps             |

---
Uno `StatefulSet` richiede spesso un `Headless Service`.

Differenza:
  - Service normale → load balancing
  - Headless Service → DNS diretto verso ogni Pod

Permette:
 - Discovery diretto dei Pod
 - Comunicazione stabile tra repliche

`spec.podManagementPolicy`

| Policy                 | Descrizione                                          |
|------------------------|------------------------------------------------------|
| OrderedReady (default) | I Pod vengono creati e pronti in ordine sequenziale  |
| Parallel               | I Pod possono essere creati o terminati in parallelo |

<img width="1207" height="703" alt="Immagine 2026-02-24 231557" src="https://github.com/user-attachments/assets/e813e89a-3f31-43f3-81d3-63d6b0926e29" />
<img width="1093" height="713" alt="Immagine 2026-02-24 231756" src="https://github.com/user-attachments/assets/5acbad8b-dffe-40be-87c3-a87b9c770358" />

---

## volumeClaimTemplates (StatefulSet)

Nel caso degli **StatefulSet**, i volumi devono essere:

- Persistenti
- Unici per ogni replica
- Associati stabilmente al Pod

Per questo si usa `volumeClaimTemplates`.

---

### Cos’è volumeClaimTemplates?

È un template di **PersistentVolumeClaim** definito dentro lo StatefulSet.

Per ogni replica viene creato automaticamente:

- Un PVC dedicato
- Un PV associato (dinamicamente se c'è una StorageClass)

Ogni Pod ottiene quindi il **proprio volume persistente**.

---

### Perché è importante?

A differenza dei Deployment:

- Le repliche di uno StatefulSet **non condividono lo stesso volume**
- Ogni Pod mantiene il proprio storage anche se viene riavviato

Esempio:
- `web-0` → PVC `data-web-0`
- `web-1` → PVC `data-web-1`
- `web-2` → PVC `data-web-2`

Se `web-1` viene ricreato, riutilizza il **suo stesso volume**.

---

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: "web"
  replicas: 3
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
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 1Gi
      storageClassName: standard
```

Lo StatefulSet crea il Pod web-0

Viene creato automaticamente il PVC data-web-0

La StorageClass crea il PV

Il volume viene montato nel container

Questo processo si ripete per ogni replica.

<img width="1262" height="702" alt="Immagine 2026-02-24 232040" src="https://github.com/user-attachments/assets/f00959ba-d64d-4a39-b3b5-b8170a8cf04e" />

---

## Esercizi

1. Creare un Pod con volume emptyDir e verificare condivisione dati.
2. Creare un PersistentVolumeClaim da 1Gi.
3. Montare il PVC in un Pod.
4. Scrivere dati nel volume e riavviare il Pod per verificare la persistenza.
5. Eliminare il Pod e verificare che i dati persistano.
6. Creare una StorageClass (se supportata dal cluster).
7. Verificare accessModes supportati dal cluster.
8. Condividere un volume emptyDir tra container.


---



