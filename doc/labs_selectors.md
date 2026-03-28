- [ Home ](../readme.md)   | [ Teoria ](../arguments.md)   | [ Info Exam ](./ckad_exam_strategy.md)   | [ Teoria Application Deployment](../arg/second_arg.md)   |
---

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


| Selector                  | Livello                       | Dove si usa                          | Scopo                                      | Complessità | Note                                      |
| ------------------------- | ----------------------------- | ------------------------------------ | ------------------------------------------ | ----------- | ----------------------------------------- |
| Label Selector            | Resource metadata             | Deployment, Service, ReplicaSet, Job | Filtrare risorse tramite label             | Bassa       | Base fondamentale di molti altri selector |
| nodeSelector              | Pod spec                      | Pod scheduling                       | Vincolare pod su nodi con label specifiche | Molto bassa | Match esatto solo key/value               |
| nodeAffinity              | Pod spec                      | Scheduling avanzato                  | Regole di placement sui nodi               | Media       | Supporta preferenze e vincoli obbligatori |
| podAffinity               | Pod spec                      | Scheduling topology-aware            | Collocare pod vicino ad altri pod          | Alta        | Usa topologyKey                           |
| podAntiAffinity           | Pod spec                      | Scheduling resilienza                | Evitare co-locazione di pod simili         | Alta        | Molto usato per HA                        |
| topologySpreadConstraints | Pod spec                      | High availability scheduling         | Distribuzione uniforme dei pod             | Alta        | Migliora fault tolerance                  |
| podSelector               | NetworkPolicy                 | Networking security                  | Selezionare pod target per policy          | Media       | Tipico in security isolation              |
| namespaceSelector         | NetworkPolicy / Policy engine | Isolation tra namespace              | Filtrare namespace di appartenenza         | Media       | Usato in security context                 |
| service selector          | Service spec                  | Traffic routing                      | Associare Service ai Pod target            | Bassa       | Fondamentale per networking interno       |
| fieldSelector             | API query layer               | kubectl, API filtering               | Filtrare risorse per campi dello schema    | Media       | Non è uno scheduler selector              |
| matchLabels               | Selector expression           | Deployment, Affinity, Policy         | Match esatto su label                      | Bassa       | Parte di strutture selector più complesse |
| matchExpressions          | Selector expression           | Affinity, Policy                     | Match logico avanzato (In, NotIn, Exists…) | Media       | Consigliato per flessibilità              |

---
