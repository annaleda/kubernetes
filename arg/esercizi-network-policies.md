# Esercizi Kubernetes NetworkPolicy con soluzioni

> Materiale originale, in italiano, **ispirato ai casi d'uso** del repository `ahmetb/kubernetes-network-policy-recipes`.
> 
> Repository di riferimento: https://github.com/ahmetb/kubernetes-network-policy-recipes

## Come usare questo file

Ogni esercizio contiene:
- **Obiettivo**
- **Scenario**
- **Richiesta**
- **Soluzione YAML**
- **Spiegazione rapida**

Le soluzioni assumono Kubernetes `networking.k8s.io/v1` e un CNI che supporti le NetworkPolicy.

---

## Esercizio 1 — Deny all ingress per un'applicazione

### Obiettivo
Bloccare tutto il traffico **in ingresso** verso un gruppo di Pod.

### Scenario
Nel namespace `app`, tutti i Pod con label `app=api` devono diventare non raggiungibili da altri Pod.

### Richiesta
Crea una NetworkPolicy chiamata `deny-all-ingress-api`.

### Soluzione
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all-ingress-api
  namespace: app
spec:
  podSelector:
    matchLabels:
      app: api
  policyTypes:
  - Ingress
  ingress: []
```

### Spiegazione
- `podSelector` seleziona i Pod `app=api`
- `policyTypes: [Ingress]` applica restrizioni solo al traffico entrante
- `ingress: []` significa **nega tutto l'ingresso**

---

## Esercizio 2 — Deny all egress per un'applicazione

### Obiettivo
Bloccare tutto il traffico **in uscita** da un gruppo di Pod.

### Scenario
Nel namespace `app`, i Pod con label `role=batch` non devono poter contattare nulla.

### Richiesta
Crea `deny-all-egress-batch`.

### Soluzione
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all-egress-batch
  namespace: app
spec:
  podSelector:
    matchLabels:
      role: batch
  policyTypes:
  - Egress
  egress: []
```

### Spiegazione
- Blocca ogni connessione in uscita dai Pod selezionati
- Il traffico in ingresso resta invariato

---

## Esercizio 3 — Deny all ingress e egress (isolamento completo)

### Obiettivo
Isolare completamente alcuni Pod.

### Scenario
Nel namespace `secure`, i Pod `app=vault` devono essere completamente isolati.

### Richiesta
Crea `isolate-vault`.

### Soluzione
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: isolate-vault
  namespace: secure
spec:
  podSelector:
    matchLabels:
      app: vault
  policyTypes:
  - Ingress
  - Egress
  ingress: []
  egress: []
```

### Spiegazione
- Nessun Pod può entrare
- I Pod selezionati non possono uscire verso nessuno

---

## Esercizio 4 — Allow ingress solo da Pod specifici nello stesso namespace

### Obiettivo
Permettere accesso a un'app solo da un frontend interno.

### Scenario
Nel namespace `shop`, i Pod `app=backend` devono accettare traffico solo dai Pod `app=frontend`.

### Richiesta
Crea `backend-from-frontend`.

### Soluzione
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: backend-from-frontend
  namespace: shop
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
          app: frontend
```

### Spiegazione
- Solo i Pod `frontend` dello **stesso namespace** possono entrare nei `backend`
- Tutte le altre sorgenti vengono negate

---

## Esercizio 5 — Allow ingress solo su una porta specifica

### Obiettivo
Permettere accesso a un'app solo su TCP/80.

### Scenario
Nel namespace `web`, i Pod `app=nginx` devono accettare traffico solo sulla porta 80.

### Richiesta
Crea `allow-http-nginx`.

### Soluzione
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-http-nginx
  namespace: web
spec:
  podSelector:
    matchLabels:
      app: nginx
  policyTypes:
  - Ingress
  ingress:
  - ports:
    - protocol: TCP
      port: 80
```

### Spiegazione
- Qualsiasi sorgente può raggiungere i Pod selezionati
- Ma solo sulla porta TCP 80

---

## Esercizio 6 — Allow ingress da un namespace specifico

### Obiettivo
Consentire traffico solo da un altro namespace.

### Scenario
I Pod `app=api` nel namespace `prod` devono accettare richieste solo da Pod nel namespace `monitoring`.

### Richiesta
Crea `api-from-monitoring`.

### Soluzione
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: api-from-monitoring
  namespace: prod
spec:
  podSelector:
    matchLabels:
      app: api
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: monitoring
```

### Spiegazione
- Viene usato un `namespaceSelector`
- Tutti i Pod del namespace `monitoring` sono ammessi

> Nota: se il tuo cluster non supporta `kubernetes.io/metadata.name`, etichetta il namespace con una label custom e usa quella.

---

## Esercizio 7 — Allow ingress da namespace e Pod specifici

### Obiettivo
Permettere accesso solo a Pod precisi di un namespace preciso.

### Scenario
Nel namespace `payments`, i Pod `app=db` devono ricevere traffico solo dai Pod `app=api` del namespace `payments`.

### Richiesta
Crea `db-from-api-only`.

### Soluzione
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-from-api-only
  namespace: payments
spec:
  podSelector:
    matchLabels:
      app: db
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: payments
      podSelector:
        matchLabels:
          app: api
```

### Spiegazione
Con `namespaceSelector + podSelector` nella **stessa entry**, consenti solo ai Pod `app=api` del namespace `payments`.

---

## Esercizio 8 — Allow all ingress a un'applicazione

### Obiettivo
Consentire tutto il traffico in ingresso, ma solo rendendo esplicita la policy.

### Scenario
Nel namespace `public`, i Pod `app=landing` devono poter ricevere traffico da chiunque.

### Richiesta
Crea `allow-all-ingress-landing`.

### Soluzione
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-all-ingress-landing
  namespace: public
spec:
  podSelector:
    matchLabels:
      app: landing
  policyTypes:
  - Ingress
  ingress:
  - {}
```

### Spiegazione
- La regola vuota `- {}` consente tutto il traffico in ingresso
- È utile per documentare intenti o per override in ambienti con più policy

---

## Esercizio 9 — Allow egress solo verso un namespace specifico

### Obiettivo
Limitare l'uscita a un namespace autorizzato.

### Scenario
Tutti i Pod del namespace `space1` devono poter uscire solo verso Pod nel namespace `space2`.

### Richiesta
Crea `np` in `space1`.

### Soluzione
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: np
  namespace: space1
spec:
  podSelector: {}
  policyTypes:
  - Egress
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: space2
```

### Spiegazione
- `podSelector: {}` = tutti i Pod di `space1`
- È permessa solo l'uscita verso namespace `space2`
- L'ingresso non viene toccato

---

## Esercizio 10 — Allow egress verso namespace specifico + DNS

### Obiettivo
Limitare il traffico in uscita ma senza rompere la risoluzione DNS.

### Scenario
Tutti i Pod del namespace `space1` devono uscire solo verso `space2`, ma devono continuare a fare DNS su 53/TCP e 53/UDP.

### Richiesta
Crea `np-dns`.

### Soluzione
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: np-dns
  namespace: space1
spec:
  podSelector: {}
  policyTypes:
  - Egress
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: space2
  - ports:
    - protocol: UDP
      port: 53
    - protocol: TCP
      port: 53
```

### Spiegazione
- Prima regola: traffico verso `space2`
- Seconda regola: traffico DNS verso qualsiasi destinazione sulla porta 53 TCP/UDP

> Variante più restrittiva: limita il DNS solo ai Pod `kube-dns` nel namespace `kube-system`.

---

## Esercizio 11 — Allow egress verso IP esterno specifico

### Obiettivo
Consentire uscita solo verso un endpoint esterno noto.

### Scenario
I Pod `app=reporting` nel namespace `corp` devono poter chiamare solo `10.20.30.40/32` sulla porta 443.

### Richiesta
Crea `reporting-external-https`.

### Soluzione
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: reporting-external-https
  namespace: corp
spec:
  podSelector:
    matchLabels:
      app: reporting
  policyTypes:
  - Egress
  egress:
  - to:
    - ipBlock:
        cidr: 10.20.30.40/32
    ports:
    - protocol: TCP
      port: 443
```

### Spiegazione
- `ipBlock` serve per IP/CIDR esterni al cluster o noti staticamente
- La policy ammette solo HTTPS verso quell'IP

---

## Esercizio 12 — Deny ingress da altri namespace, ma allow dallo stesso namespace

### Obiettivo
Consentire traffico solo intra-namespace.

### Scenario
Nel namespace `team-a`, i Pod `app=internal` devono accettare traffico solo da Pod dello stesso namespace.

### Richiesta
Crea `internal-same-namespace-only`.

### Soluzione
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: internal-same-namespace-only
  namespace: team-a
spec:
  podSelector:
    matchLabels:
      app: internal
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector: {}
```

### Spiegazione
- Un `podSelector` senza `namespaceSelector` seleziona Pod dello **stesso namespace**
- I Pod da namespace diversi vengono esclusi

---

## Esercizio 13 — Default deny ingress per tutto il namespace

### Obiettivo
Impostare un baseline di sicurezza a livello namespace.

### Scenario
Nel namespace `dev`, vuoi negare tutto l'ingresso per tutti i Pod, e poi aggiungere eccezioni con altre policy.

### Richiesta
Crea `default-deny-ingress`.

### Soluzione
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-ingress
  namespace: dev
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  ingress: []
```

### Spiegazione
Questa è una delle policy più usate come base: da qui costruisci whitelist più mirate.

---

## Esercizio 14 — Default deny egress per tutto il namespace

### Obiettivo
Bloccare per default tutte le uscite.

### Scenario
Nel namespace `finance` vuoi adottare un modello zero-trust sulle connessioni in uscita.

### Richiesta
Crea `default-deny-egress`.

### Soluzione
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-egress
  namespace: finance
spec:
  podSelector: {}
  policyTypes:
  - Egress
  egress: []
```

### Spiegazione
Tutti i Pod del namespace vengono bloccati in uscita, salvo eccezioni introdotte da altre policy.

---

## Esercizio 15 — Consentire ingress da un CIDR esterno

### Obiettivo
Consentire accesso da una rete esterna nota.

### Scenario
Nel namespace `dmz`, i Pod `app=ingress-demo` devono essere raggiungibili solo dalla rete `192.168.100.0/24` sulla porta 443.

### Richiesta
Crea `allow-cidr-https`.

### Soluzione
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-cidr-https
  namespace: dmz
spec:
  podSelector:
    matchLabels:
      app: ingress-demo
  policyTypes:
  - Ingress
  ingress:
  - from:
    - ipBlock:
        cidr: 192.168.100.0/24
    ports:
    - protocol: TCP
      port: 443
```

### Spiegazione
È il caso tipico di accesso da VPN, proxy aziendale o subnet on-prem.

---

## Esercizio 16 — Pod applicativi che possono parlare solo col database su 5432

### Obiettivo
Consentire un flusso east-west molto specifico.

### Scenario
Nel namespace `erp`, i Pod `app=web` devono poter raggiungere solo i Pod `app=postgres` sulla porta 5432.

### Richiesta
Crea `web-to-postgres-only` applicata ai Pod `web`.

### Soluzione
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: web-to-postgres-only
  namespace: erp
spec:
  podSelector:
    matchLabels:
      app: web
  policyTypes:
  - Egress
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: postgres
    ports:
    - protocol: TCP
      port: 5432
```

### Spiegazione
- I Pod `web` possono uscire solo verso i Pod `postgres`
- Solo sulla porta TCP 5432

---

## Esercizio 17 — Consentire solo monitoring verso exporter

### Obiettivo
Separare traffico applicativo e traffico di observability.

### Scenario
Nel namespace `apps`, i Pod `app=node-exporter` devono essere interrogabili solo dai Pod `app=prometheus` nel namespace `monitoring` sulla porta 9100.

### Richiesta
Crea `exporter-from-prometheus`.

### Soluzione
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: exporter-from-prometheus
  namespace: apps
spec:
  podSelector:
    matchLabels:
      app: node-exporter
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: monitoring
      podSelector:
        matchLabels:
          app: prometheus
    ports:
    - protocol: TCP
      port: 9100
```

### Spiegazione
Policy utile per metrics scraping controllato e riduzione della superficie di attacco.

---

## Esercizio 18 — Default deny namespace + eccezione per frontend→backend

### Obiettivo
Vedere il pattern reale più comune: baseline + allow mirata.

### Scenario
Nel namespace `demo` vuoi:
1. negare tutto l'ingresso a tutti i Pod
2. permettere ai Pod `app=frontend` di raggiungere i Pod `app=backend` su 8080

### Richiesta
Scrivi entrambe le policy.

### Soluzione
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-ingress
  namespace: demo
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  ingress: []
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: backend-allow-from-frontend
  namespace: demo
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
          app: frontend
    ports:
    - protocol: TCP
      port: 8080
```

### Spiegazione
Le NetworkPolicy sono additive: il default deny crea il blocco, la seconda policy apre solo il traffico desiderato.

---

# Mini laboratorio di verifica

Puoi provare i casi con Pod temporanei:

```bash
kubectl run test-a --image=busybox:1.36 -n demo -- sleep 3600
kubectl run test-b --image=busybox:1.36 -n demo -- sleep 3600
kubectl exec -it -n demo test-a -- sh
```

Dalla shell del Pod:

```bash
wget -qO- http://service-name:8080
nc -vz pod-ip 5432
nslookup kubernetes.default.svc.cluster.local
```

---

# Errori comuni da evitare

## 1. Pensare che una policy neghi tutto il namespace automaticamente
No: la policy si applica solo ai Pod selezionati da `podSelector`.

## 2. Dimenticare il DNS in egress
Se blocchi l'egress e non permetti DNS, molte app sembreranno “rotte”.

## 3. Confondere `podSelector` con `namespaceSelector`
- solo `podSelector` = Pod dello stesso namespace
- `namespaceSelector` = namespace filtrati per label
- entrambi nella stessa voce = Pod specifici di namespace specifici

## 4. Usare NetworkPolicy senza supporto del CNI
Serve un plugin che implementi davvero le NetworkPolicy.

## 5. Aspettarsi ordine di valutazione
Non c'è un “first match wins”: le policy sono additive.

---

# Cheat sheet veloce

## Deny all ingress namespace
```yaml
spec:
  podSelector: {}
  policyTypes: [Ingress]
  ingress: []
```

## Deny all egress namespace
```yaml
spec:
  podSelector: {}
  policyTypes: [Egress]
  egress: []
```

## Allow same namespace
```yaml
ingress:
- from:
  - podSelector: {}
```

## Allow from specific namespace
```yaml
ingress:
- from:
  - namespaceSelector:
      matchLabels:
        kubernetes.io/metadata.name: monitoring
```

## Allow DNS egress
```yaml
egress:
- ports:
  - protocol: UDP
    port: 53
  - protocol: TCP
    port: 53
```

---

# Fonti e ispirazione

Questi esercizi sono stati riorganizzati e riscritti in italiano prendendo spunto dai pattern e dai casi d'uso pubblicati nel repository:
- `ahmetb/kubernetes-network-policy-recipes`

Utile in particolare per pattern come:
- deny all ingress
- deny all egress
- allow da namespace specifici
- allow da pod specifici
- combinazioni ingress/egress
- gestione del DNS in policy restrittive
