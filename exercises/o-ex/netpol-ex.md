- [ Home ](../../readme.md)   | [ Teoria ](../../arguments.md)   | [ Info Exam ](../../doc/ckad_exam_strategy.md)    | [ Home Other Exercises ](../o_exercises.md)   
---
### Network Policies (12 esercizi)

---

## NP-0 — Allow All Ingress

- Namespace: `net-open`

- Creare NetworkPolicy
  - Consentire tutto il traffico in ingresso verso i Pod del namespace

- Specifiche
  - PolicyType: `Ingress`
  - Pod selector: tutti i Pod
  - Ingress: consentito da qualsiasi sorgente

- Validazione
  - I Pod possono ricevere traffico in ingresso

<details>
<summary>Soluzione</summary>

```sh
kubectl create ns net-open
```

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-all-ingress
  namespace: net-open
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  ingress:
  - {}
```

```sh
kubectl apply -f np-0.yaml
kubectl get netpol -n net-open
```

</details>

---

## NP-1 — Default Deny Ingress

- Namespace: `net-secure`

- Creare NetworkPolicy
  - Nessun traffico in ingresso consentito

- Validazione
  - I Pod non comunicano tra loro in ingresso

<details>
<summary>Soluzione</summary>

```sh
kubectl create ns net-secure
```

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny
  namespace: net-secure
spec:
  podSelector: {}
  policyTypes:
  - Ingress
```

```sh
kubectl apply -f np-1.yaml
kubectl get netpol -n net-secure
kubectl describe netpol default-deny -n net-secure
```

</details>

---

## NP-2 — Allow Ingress da label

- Namespace: `net-secure`

- Consentire traffico in ingresso solo da Pod con label:
  - `role=frontend`

- Validazione
  - Solo i Pod frontend possono connettersi

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend
  namespace: net-secure
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          role: frontend
```

```sh
kubectl apply -f np-2.yaml
kubectl run frontend --image=busybox --labels=role=frontend -n net-secure -- sleep 3600
kubectl run backend --image=busybox --labels=role=backend -n net-secure -- sleep 3600
```

</details>

---

## NP-3 — Allow Egress DNS

- Namespace: `net-secure`

- Consentire solo traffico egress verso DNS
  - porta UDP 53
  - porta TCP 53

- Validazione
  - DNS permesso
  - Altro egress bloccato

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-dns-egress
  namespace: net-secure
spec:
  podSelector: {}
  policyTypes:
  - Egress
  egress:
  - ports:
    - protocol: UDP
      port: 53
    - protocol: TCP
      port: 53
```

</details>

---

## NP-4 — Namespace Selector

- Namespace target: `net-secure`

- Consentire traffico in ingresso solo da namespace con label:
  - `access=trusted`

- Validazione
  - Namespace trusted consentito
  - altri namespace bloccati

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-from-trusted-ns
  namespace: net-secure
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          access: trusted
```

```sh
kubectl label ns trusted access=trusted
```

</details>

---

## NP-5 — Port Restriction

- Namespace: `net-secure`

- Consentire traffico in ingresso solo su:
  - TCP 80

- Validazione
  - Porta 80 consentita
  - altre porte bloccate

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-port-80
  namespace: net-secure
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  ingress:
  - ports:
    - protocol: TCP
      port: 80
```

</details>

---

## NP-6 — Combined Policy

- Namespace: `net-secure`

- Consentire:
  - Ingress da namespace con label `access=trusted`
  - Solo su porta 443

- Validazione
  - Traffico limitato correttamente

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-trusted-443
  namespace: net-secure
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          access: trusted
    ports:
    - protocol: TCP
      port: 443
```

</details>

---

## NP-7 — Default Deny Egress

- Namespace: `egress-lock`

- Creare NetworkPolicy
  - Nessun traffico in uscita consentito

- Validazione
  - I Pod non possono uscire verso altri servizi

<details>
<summary>Soluzione</summary>

```sh
kubectl create ns egress-lock
```

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-egress
  namespace: egress-lock
spec:
  podSelector: {}
  policyTypes:
  - Egress
```

```sh
kubectl apply -f np-7.yaml
kubectl describe netpol default-deny-egress -n egress-lock
```

</details>

---

## NP-8 — Allow ingress solo verso pod backend

- Namespace: `app-net`

- Consentire traffico in ingresso solo ai Pod con label:
  - `tier=backend`

- Solo da Pod con label:
  - `tier=frontend`

- Validazione
  - policy applicata solo ai backend

<details>
<summary>Soluzione</summary>

```sh
kubectl create ns app-net
```

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: backend-from-frontend
  namespace: app-net
spec:
  podSelector:
    matchLabels:
      tier: backend
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          tier: frontend
```

</details>

---

## NP-9 — Namespace + Pod selector combinati

- Namespace target: `app-net`

- Consentire traffico solo da:
  - namespace con label `team=blue`
  - Pod con label `role=client`

- Validazione
  - Solo quei Pod in quei namespace possono entrare

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-blue-clients
  namespace: app-net
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          team: blue
      podSelector:
        matchLabels:
          role: client
```

</details>

---

## NP-10 — Allow egress verso un CIDR specifico

- Namespace: `egress-lock`

- Consentire egress solo verso:
  - `10.0.0.0/24`

- Validazione
  - Solo quel range IP è consentito

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-egress-cidr
  namespace: egress-lock
spec:
  podSelector: {}
  policyTypes:
  - Egress
  egress:
  - to:
    - ipBlock:
        cidr: 10.0.0.0/24
```

</details>

---

## NP-11 — Deny except same namespace app=db

- Namespace: `db-net`

- Consentire traffico in ingresso ai Pod:
  - `app=db`

- Solo da Pod:
  - nello stesso namespace
  - con label `app=api`

- Validazione
  - accesso limitato ai Pod api interni al namespace

<details>
<summary>Soluzione</summary>

```sh
kubectl create ns db-net
```

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-only-from-api
  namespace: db-net
spec:
  podSelector:
    matchLabels:
      app: db
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: api
```

</details>

---

## NP-12 — Combined ingress + egress

- Namespace: `full-lock`

- Requisiti
  - Ingress consentito solo da Pod `role=frontend`
  - Egress consentito solo verso porta 53 UDP/TCP

- Validazione
  - policyTypes contiene entrambi
  - Ingress ed Egress limitati

<details>
<summary>Soluzione</summary>

```sh
kubectl create ns full-lock
```

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: ingress-egress-combo
  namespace: full-lock
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          role: frontend
  egress:
  - ports:
    - protocol: UDP
      port: 53
    - protocol: TCP
      port: 53
```

</details>

---

## NP-13 — Allow Ingress da IP specifico

* Namespace: `ip-allow`
* Consentire traffico solo da:

  * `192.168.1.0/24`

<details>
<summary>Soluzione</summary>

```sh
kubectl create ns ip-allow
```

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-ip-range
  namespace: ip-allow
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  ingress:
  - from:
    - ipBlock:
        cidr: 192.168.1.0/24
```

</details>

---

## NP-14 — Deny specific CIDR

* Namespace: `ip-block`
* Consentire tutto tranne:

  * `192.168.1.0/24`

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-specific-ip
  namespace: ip-block
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  ingress:
  - from:
    - ipBlock:
        cidr: 0.0.0.0/0
        except:
        - 192.168.1.0/24
```

</details>

---

## NP-15 — Allow Egress HTTP

* Namespace: `web-egress`
* Consentire solo:

  * TCP 80

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-http-egress
  namespace: web-egress
spec:
  podSelector: {}
  policyTypes:
  - Egress
  egress:
  - ports:
    - protocol: TCP
      port: 80
```

</details>

---

## NP-16 — Ingress su più porte

* Namespace: `multi-port`
* Consentire:

  * TCP 80
  * TCP 443

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-web-ports
  namespace: multi-port
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  ingress:
  - ports:
    - protocol: TCP
      port: 80
    - protocol: TCP
      port: 443
```

</details>

---

## NP-17 — Allow solo intra-namespace

* Namespace: `internal-only`
* Solo traffico interno

<details>
<summary>Soluzione</summary>

```sh
kubectl create ns internal-only
```

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-internal
  namespace: internal-only
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector: {}
```

</details>

---

## NP-18 — Allow Egress verso namespace specifico

* Namespace: `client-ns`
* Consentire solo verso:

  * `env=prod`

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-egress-prod
  namespace: client-ns
spec:
  podSelector: {}
  policyTypes:
  - Egress
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          env: prod
```

</details>

---

## NP-19 — PodSelector + Porta

* Namespace: `fine-grain`
* Consentire:

  * Pod `app=client`
  * Porta 8080

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: fine-grain-policy
  namespace: fine-grain
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: client
    ports:
    - port: 8080
      protocol: TCP
```

</details>

---

## NP-20 — Multiple rules OR

* Namespace: `multi-rule`
* Consentire:

  * frontend OR admin

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: multi-source
  namespace: multi-rule
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          role: frontend
  - from:
    - podSelector:
        matchLabels:
          role: admin
```

</details>

---

## NP-21 — AND logic namespace + pod

* Namespace: `and-logic`
* Consentire:

  * namespace `team=red`
  * Pod `app=client`

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: and-logic
  namespace: and-logic
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          team: red
      podSelector:
        matchLabels:
          app: client
```

</details>

---

## NP-22 — Egress verso Pod specifici

* Namespace: `egress-pod`
* Consentire solo verso:

  * `app=db`

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: egress-to-db
  namespace: egress-pod
spec:
  podSelector: {}
  policyTypes:
  - Egress
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: db
```

</details>

---

## NP-23 — DNS + HTTP only

* Namespace: `restricted-egress`
* Consentire:

  * TCP 80
  * TCP/UDP 53

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: restricted-egress
  namespace: restricted-egress
spec:
  podSelector: {}
  policyTypes:
  - Egress
  egress:
  - ports:
    - port: 80
      protocol: TCP
    - port: 53
      protocol: UDP
    - port: 53
      protocol: TCP
```

</details>

---

## NP-24 — Policy su subset Pod

* Namespace: `partial`
* Solo Pod `app=web`

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: web-only
  namespace: partial
spec:
  podSelector:
    matchLabels:
      app: web
  policyTypes:
  - Ingress
  ingress:
  - {}
```

</details>

---

## NP-25 — Debug selector mismatch

* Namespace: `debug-ns`

<details>
<summary>Soluzione</summary>

Errore:

```yaml
podSelector:
  matchLabels:
    app: wrong
```

Fix:

```yaml
podSelector:
  matchLabels:
    app: correct
```

```sh
kubectl get pods --show-labels
```

</details>

---

## NP-26 — Allow ingress + deny egress

* Namespace: `mixed`

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: mixed-policy
  namespace: mixed
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - {}
```

</details>

---

## NP-27 — Egress verso Service CIDR

* Namespace: `svc-egress`
* CIDR: `10.96.0.0/12`

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-cluster-svc
  namespace: svc-egress
spec:
  podSelector: {}
  policyTypes:
  - Egress
  egress:
  - to:
    - ipBlock:
        cidr: 10.96.0.0/12
```

</details>

---

## NP-28 — Solo Pod stessa label

* Namespace: `self-comm`
* Solo `app=api`

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: self-communication
  namespace: self-comm
spec:
  podSelector:
    matchLabels:
      app: api
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: api
```

</details>

---

## NP-29 — Multiple namespaceSelector

* Namespace: `multi-ns`
* Consentire:

  * dev
  * staging

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: multi-ns-access
  namespace: multi-ns
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          env: dev
  - from:
    - namespaceSelector:
        matchLabels:
          env: staging
```

</details>

---

## NP-30 — Zero Trust

* Namespace: `zero-trust`
* Consentire:

  * ingress: frontend
  * egress: db

<details>
<summary>Soluzione</summary>

```sh
kubectl create ns zero-trust
```

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: zero-trust
  namespace: zero-trust
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          role: frontend
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: db
```

</details>

---


## Cheatsheet NetworkPolicy

```sh
# vedere le networkpolicy
kubectl get netpol -A

# descrivere una policy
kubectl describe netpol <name> -n <namespace>

# vedere labels namespace
kubectl get ns --show-labels

# vedere labels pod
kubectl get pods --show-labels -n <namespace>
```

---

## Mini schema mentale

```text
podSelector        -> a quali Pod si applica la policy
from / to          -> chi può entrare / dove si può uscire
namespaceSelector  -> filtra namespace
podSelector        -> filtra Pod
ports              -> limita le porte
ipBlock            -> limita CIDR/IP

Se una policy seleziona un Pod:
- Ingress non esplicitamente consentito = negato
- Egress non esplicitamente consentito = negato
```
