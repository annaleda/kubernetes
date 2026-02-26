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

