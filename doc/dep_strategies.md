## Deployment Strategies

<img width="899" height="346" alt="Immagine 2026-02-26 125603" src="https://github.com/user-attachments/assets/fee71a2b-40b5-430e-be51-f20c8b264d5d" />

- [Deployment - Core Concepts](../doc/ckad_core_concepts.md#deployment)
- Recreate, Rollig Update and Rollback 
- Scaling and Descaling                
- Blue/Green
- Canary
  
---
### Recreate, Rollig Update and Rollback 

`Recreate Strategy`

Termina tutti i Pod della vecchia versione prima di avviare quelli nuovi.

```yaml
strategy:
  type: Recreate
```
---
`Rolling Update Strategy (Default)`

Aggiorna i Pod gradualmente senza downtime.

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 1
    maxSurge: 1
```

- `maxUnavailable`: quanti Pod possono essere non disponibili

- `maxSurge`: quanti Pod extra possono essere creati temporaneamente

---
`Rollback`

Vedere la cronologia:

```yaml
kubectl rollout history deployment <nome>
```

Fare rollback:

```yaml
kubectl rollout undo deployment <nome>
```

Rollback a revisione specifica:

```yaml
kubectl rollout undo deployment <nome> --to-revision=2
```
---
### Scaling and Descaling     

`Scalare manualmente`

```yaml
kubectl scale deployment <nome> --replicas=5
```

Oppure modificando il file YAML:

```yaml
spec:
  replicas: 5
```

`Descaling`

```yaml
kubectl scale deployment <nome> --replicas=2
```
---
### Blue/Green

<img width="1343" height="484" alt="Immagine 2026-02-26 130401" src="https://github.com/user-attachments/assets/94c21a5c-3dec-4359-a724-a345dd6fbdbb" />

Hai due ambienti identici:

- Blue → versione attuale (1.0)
- Green → nuova versione (1.1)

Il traffico va solo a uno dei due.

Quando vuoi fare switch:
> cambi il Service.

```bash
Deployment v1.0
metadata:
  name: nginx-deployment-1.0
labels:
  app: nginx-1.0
```
```bash
Deployment v1.1
metadata:
  name: nginx-deployment-1.1
labels:
  app: nginx-1.1
```
```bash
Service
selector:
  app: nginx-1.0
```

> Attualmente il Service manda traffico SOLO alla 1.0.

Per fare lo switch Blue → Green modifichi il selector del Service:

```bash
selector:
  app: nginx-1.1
```

Applichi:

```bash
kubectl apply -f service.yaml
```
---
### Canary

La nuova versione viene rilasciata gradualmente. Il Service punta alla stessa label per entrambe le versioni. La distribuzione del traffico dipende dal numero di Pod.

- Deployment stabile (v1.0)

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-v1
spec:
  replicas: 5
  selector:
    matchLabels:
      app: nginx
      version: v1
  template:
    metadata:
      labels:
        app: nginx
        version: v1
```
- Deployment canary (v1.1)

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-v2
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
      version: v2
  template:
    metadata:
      labels:
        app: nginx
        version: v2
```

- Service(label comune)
  
```bash
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
```
- Distribuzione traffico
    - Totale Pod: 6
    - Nuova versione: 1

  16% del traffico va alla nuova versione.

- Progressione Canary

    - 5 vecchi / 1 nuovo
    - 4 vecchi / 2 nuovi
    - 3 vecchi / 3 nuovi
    - 0 vecchi / 6 nuovi

✔ Rilascio graduale

✔ Riduce rischio

❌ Rollback meno immediato rispetto a Blue/Green

| Strategia     | Downtime | Graduale | Cambio Service | Uso Risorse |
| ------------- | -------- | -------- | -------------- | ----------- |
| Recreate      | ✅ Sì     | ❌ No     | ❌ No           | Basso       |
| RollingUpdate | ❌ No     | ✅ Sì     | ❌ No           | Medio       |
| Blue/Green    | ❌ No     | ❌ No     | ✅ Sì           | Alto        |
| Canary        | ❌ No     | ✅ Sì     | ❌ No           | Medio       |

---
