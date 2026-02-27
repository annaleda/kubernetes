## CDR: Custom Resource Definition, Operators
---

### CDR: Custom Resource Definition

Una **Custom Resource Definition (CRD)** permette di estendere l’API di Kubernetes.

Con una CRD puoi creare nuovi tipi di risorse come se fossero native:

- Pod
- Service
- Deployment
- **MyDatabase** (custom)
- **MyApp**
- **Backup**
- ecc.

In pratica:
> Kubernetes diventa estendibile.

---

Kubernetes funziona tramite:

- API Server
- etcd
- Controller

Una CRD:

1. Registra un nuovo tipo di oggetto nell'API Server
2. Permette di usare `kubectl` su quella nuova risorsa

---

<img width="851" height="397" alt="Immagine 2026-02-26 174417" src="https://github.com/user-attachments/assets/bdf2e8f9-3ea7-474f-9b43-6a0e790ad6db" />

### Esempio di CRD

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: backups.stable.example.com
spec:
  group: stable.example.com
  names:
    kind: Backup
    plural: backups
    singular: backup
  scope: Namespaced
  versions:
    - name: v1
      served: true
      storage: true
      schema:
        openAPIV3Schema:
          type: object
```

> Dopo aver creato la CRD puoi creare una nuova risorsa custom:

```yml
apiVersion: stable.example.com/v1
kind: Backup
metadata:
  name: my-backup
spec:
  schedule: "0 0 * * *"
```
E usarla come qualsiasi altra risorsa:

```yaml
kubectl get backups
kubectl describe backup my-backup
```

---
- `etcd`: Storage del Cluster State

In Kubernetes, lo stato del cluster viene memorizzato in un database distribuito chiamato: etcd.
  - È il data store principale del cluster.
  - Funziona come un key-value database distribuito.
  - Contiene lo stato desiderato e corrente delle risorse.

All’interno di etcd vengono memorizzati:
  - Metadata delle risorse
  - Spec delle risorse
  - Configurazioni cluster
  - Secrets

Quando crei una CRD:
  - Definisci un nuovo schema API
  - API Server registra il nuovo resource type
  - Le istanze della custom resource vengono salvate in etcd
  - Controller o operator gestiscono il reconciliation logic

-Sicurezza di etcd 

Poiché contiene dati sensibili del cluster:
  - Deve essere accessibile solo dal control plane
  - Usa comunicazione TLS
  - Supporta encryption at rest
  - Deve essere hardenizzato in produzione
---
### Operators

Un Operator è un controller che gestisce una CRD.

CRD = definisce il nuovo oggetto
Operator = implementa la logica di gestione

|Componente| Funzione                 |
|----------|--------------------------|
|CRD	     | Estende l'API            |
|Operator  | Automatizza la gestione  |

Esempio CRD:

kind: MyDatabase

Quando crei:

kind: MyDatabase
spec:
  version: 15
  storage: 10Gi

L’Operator:

- Crea Deployment
- Crea Service
- Crea PVC
- Gestisce backup
- Gestisce failover
- Aggiorna versioni
 -Ripristina in caso di crash

Esempi Operator:

- Prometheus Operator
- PostgreSQL Operator
- MongoDB Operator
- ArgoCD
- Cert-Manager

internamente funziona:

- L’utente crea una Custom Resource
- L’API Server la salva in etcd
- L’Operator la osserva (watch)
- L’Operator esegue azioni per raggiungere lo stato desiderato


| Native     | Custom     |
| ---------- | ---------- |
| Pod        | MyApp      |
| Deployment | MyDatabase |
| Service    | Backup     |


---

