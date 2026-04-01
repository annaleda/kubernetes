## NetworkPolicy — Esempi progressivi (Ingress + Egress )

---

##  1. Default Deny (blocca tutto)

Blocca **tutto il traffico in ingresso e uscita** per tutti i Pod nel namespace.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
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

 frontend → backend ✅

---

##  5. Allow specific Egress (verso database)

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: backend-egress
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

 backend → database ✅

---

##  6. Full control

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: backend-full
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

---

##  7. Allow tutto

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-all
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

---

#  Considerazioni generali

- Le policy sono additive
- Si applicano ai Pod selezionati
- Ingress ed Egress sono indipendenti
- Default: tutto permesso finché non definisci policy
