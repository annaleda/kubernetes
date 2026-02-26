## Labels and Selectors
---

## Labels

Le **Labels** sono coppie chiave/valore associate a una risorsa Kubernetes.

Servono per:

- Organizzare le risorse
- Raggruppare oggetti
- Permettere ai Controller di selezionare Pod
- Collegare Service ai Pod

---

### Sintassi

```yaml
metadata:
  labels:
    app: nginx
    env: prod
    tier: frontend
```
Formato:

```
key: value
```
Esempio:

```
app: nginx
env: prod
version: v1
```

Le labels possono essere applicate a:
  - Pod
  - Deployment
  - Service
  - ReplicaSet
  - Node
  - Namespace

### Visualizzare Labels
```
kubectl get pods --show-labels
```
---

## Selectors

I Selectors permettono di selezionare oggetti in base alle labels.

Sono usati da:
  - Service
  - Deployment
  - ReplicaSet
  - NetworkPolicy

### MatchLabels (Equality-based)

Selezione diretta chiave = valore.
```
selector:
  matchLabels:
    app: nginx
    env: prod
```
Significa:

> Seleziona tutti i Pod con app=nginx E env=prod

### MatchExpressions (Set-based)

Permette espressioni più avanzate.

```
selector:
  matchExpressions:
    - key: env
      operator: In
      values:
        - prod
        - staging
```
Operatori disponibili:
  - In
  - NotIn
  - Exists
  - DoesNotExist

### Labels e Service

Un Service usa un selector per sapere quali Pod riceveranno traffico.

Pod
```
metadata:
  labels:
    app: nginx
```

Service
```
spec:
  selector:
    app: nginx
```
> Il Service invia traffico solo ai Pod con app=nginx.

---

Il Deployment usa due parti fondamentali:

```
spec:
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
```

> Il selector.matchLabels deve combaciare con template.metadata.labels.

Se non combaciano → errore.

---

Selezionare Pod specifici:

```
kubectl get pods -l app=nginx
```

Più condizioni:

```
kubectl get pods -l app=nginx,env=prod
```

Set-based:
```
kubectl get pods -l 'env in (prod,staging)'
```

---
