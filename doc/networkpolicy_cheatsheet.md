- [ Home ](../readme.md)   | [ Teoria ](../arguments.md)   | [ Info Exam ](../doc/ckad_exam_strategy.md)

---

## NetworkPolicy — Esempi progressivi (Ingress + Egress )

---

##  1. Default Deny (blocca tutto)

Blocca **tutto il traffico in ingresso e uscita** per tutti i Pod nel namespace.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
  namespace: backend-ns
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
```

 Nessun Pod può:
- ricevere traffico ❌
- inviare traffico ❌

---

##  2. Allow All Ingress (egress bloccato)

Permette tutto il traffico in ingresso, ma blocca l’uscita.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-all-ingress
  namespace: backend-ns
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - {}
```

 Risultato:
- ingresso: tutto permesso ✅
- uscita: tutto bloccato ❌

 Considerazioni:
- Le richieste arrivano ma le risposte possono fallire

---

##  3. Allow All Egress (ingress bloccato)

Permette uscita verso tutti, ma blocca ingresso.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-all-egress
  namespace: backend-ns
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
  egress:
  - {}
```

 Risultato:
- ingresso: bloccato ❌
- uscita: tutto permesso ✅

 Considerazioni:
- Utile per job/batch

---

##  4. Allow specific Ingress (da frontend)

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend
  namespace: backend-ns
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          role: frontend
```

Risultato:

- frontend → backend ✅
- altri → backend ❌

---

##  5. Allow specific Egress (verso database)

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: backend-egress
  namespace: backend-ns
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
  - Egress
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: database
```
Risultato:

 - backend → database ✅
 - backend → altri ❌
---

##  6. Full control

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: backend-full
  namespace: backend-ns
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: database
```

Risultato:

- frontend → backend ✅
- altri → backend ❌
- backend → database ✅
- backend → altri ❌

---

##  7. Ingress + Egress da più label

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: backend-advanced
  namespace: backend-ns
spec:
  podSelector:
    matchLabels:
      app: backend

  policyTypes:
  - Ingress
  - Egress

  ingress:
  - from:
    - podSelector:
        matchLabels:
          role: frontend
    - podSelector:
        matchLabels:
          role: monitoring

  egress:
  - to:
    - podSelector:
        matchLabels:
          app: database
    - podSelector:
        matchLabels:
          app: cache
```

Risultato:

- frontend → backend ✅
- monitoring → backend ✅
- altri → backend ❌
- backend → database ✅
- backend → cache ✅
- backend → altri ❌

##  8. Allow all

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-all
  namespace: backend-ns
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - {}
  egress:
  - {}
```

Risultato:

- ingresso: tutto ✅
- uscita: tutto ✅

---

| Scenario       | Ingress | Egress |
| -------------- | ------- | ------ |
| Default deny   | ❌       | ❌      |
| Allow ingress  | ✅       | ❌      |
| Allow egress   | ❌       | ✅      |
| Allow specific | 🔒      | 🔒     |
| Allow all      | ✅       | ✅      |

---

## Esempi di NetworkPolicy che usano selector diversi da `podSelector`:

- namespaceSelector
- ipBlock
- combinazione namespaceSelector + podSelector
- matchExpressions

---

##  1. namespaceSelector (ingress da namespace specifico)

Permette traffico ai Pod `backend` solo da namespace con label:

- team=frontend

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-from-frontend-ns
  namespace: backend-ns
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          team: frontend
```

 Risultato:
- Pod in namespace `team=frontend` → backend ✅
- altri namespace → backend ❌

---

##  2. namespaceSelector + podSelector (AND logico)

Permette traffico solo da Pod con:

- role=frontend
- nel namespace team=web

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-from-web
  namespace: backend-ns
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          team: web
      podSelector:
        matchLabels:
          role: frontend
```

 Risultato:
- frontend nel ns web → backend ✅
- frontend in altri ns → ❌
- altri pod nel ns web → ❌

---

 Le **NetworkPolicy sono namespaced**

- Vivono dentro un namespace
- Si applicano solo ai Pod di quel namespace

---

>  Le NetworkPolicy non sono globali

- Non controllano tutto il cluster
- Controllano solo i Pod selezionati nel loro namespace

Esempio base:

```yaml
metadata:
  namespace: backend-ns
```
 Questa policy:
- protegge solo i Pod in `backend-ns`
- non ha effetto su altri namespace

---

Quando si usa:

```yaml
namespaceSelector:
```

> Serve solo a selezionare **da dove arriva il traffico**

---

```yaml
podSelector:
  matchLabels:
    app: backend
```

Stai proteggendo:
- Pod `backend` nel namespace della policy

> La policy resta sempre nel namespace del **destinatario** non nel namespace della sorgente

---

### Schema mentale

```
[Namespace frontend]   --->   [Namespace backend]
        (source)                (destination)
                                  ↑
                           NetworkPolicy QUI
```

---

 Le NetworkPolicy si definiscono sempre:

> **dove stanno i Pod da proteggere**

---

##  3. ipBlock (egress verso rete esterna)

Permette uscita solo verso subnet:

- 10.0.0.0/24
- escludendo 10.0.0.5

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-egress-ip
  namespace: backend-ns
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
  - Egress
  egress:
  - to:
    - ipBlock:
        cidr: 10.0.0.0/24
        except:
        - 10.0.0.5/32
```

 Risultato:
- backend → subnet ✅
- backend → 10.0.0.5 ❌
- backend → altri IP ❌

---

##  4. matchExpressions (selector avanzato)

Permette ingress da Pod con:

- role IN (frontend, monitoring)

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-role-expression
  namespace: backend-ns
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchExpressions:
        - key: role
          operator: In
          values:
          - frontend
          - monitoring
```

 Risultato:
- frontend → backend ✅
- monitoring → backend ✅
- altri → ❌

---

##  Tabella riassuntiva

| Selector | Uso | Note |
|---------|-----|-----|
| podSelector | Pod stesso namespace | default |
| namespaceSelector | Namespace | tutti i Pod del ns |
| ipBlock | IP/CIDR | traffico esterno |
| matchExpressions | Logica avanzata | In, NotIn, Exists |

---

##  Regole importanti

- namespaceSelector → seleziona namespace
- podSelector → seleziona Pod nello stesso namespace
- insieme nello stesso blocco → AND
- liste → OR
- matchLabels multipli → AND

---
---

## Considerazioni generali

- Le policy sono additive
- Si applicano ai Pod selezionati
- Ingress ed Egress sono indipendenti
- Default: tutto permesso finché non definisci policy
