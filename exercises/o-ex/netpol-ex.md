- [ Home ](../../readme.md)   | [ Teoria ](../../arguments.md)   | [ Info Exam ](../../doc/ckad_exam_strategy.md)    | [ Home Other Exercises ](../o_exercises.md)   
---
### Network Policies (31 esercizi)

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

# Pod test
kubectl run server -n net-open --image=busybox --restart=Never --labels=app=server -- sh -c "nc -lk -p 8080"
kubectl expose pod server -n net-open --port=8080 --name=server-svc

kubectl run client -n net-open --image=busybox --restart=Never -- sleep 3600

# Test
kubectl exec client -n net-open -- nc -vz server-svc 8080
```

</details>

---

## NP-1 — Default Deny Ingress

- Namespace: `net-secure1`

- Creare NetworkPolicy
  - Nessun traffico in ingresso consentito

- Validazione
  - I Pod non comunicano tra loro in ingresso

<details>
<summary>Soluzione</summary>

```sh
kubectl create ns net-secure1
```

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny
  namespace: net-secure1
spec:
  podSelector: {}
  policyTypes:
  - Ingress
```

```sh
kubectl apply -f np-1.yaml
kubectl get netpol -n net-secure1
kubectl describe netpol default-deny -n net-secure1

kubectl run server -n net-secure1 --image=busybox --restart=Never --labels=app=server -- sh -c "nc -lk -p 8080"
kubectl expose pod server -n net-secure1 --port=8080 --name=server-svc

kubectl run client -n net-secure1 --image=busybox --restart=Never -- sleep 3600

# Test
kubectl exec client -n net-secure1 -- nc -vz server-svc 8080
```

</details>

---

## NP-2 — Allow Ingress da label

- Namespace: `net-secure2`

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
  namespace: net-secure2
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
kubectl run frontend --image=busybox --labels=role=frontend -n net-secure2 -- sleep 3600
kubectl run backend --image=busybox --labels=role=backend -n net-secure2 -- sleep 3600

# Verifica
kubectl run server -n net-secure2 --image=busybox --labels=app=server -- sh -c "nc -lk -p 8080"
kubectl expose pod server -n net-secure2 --port=8080 --name=server-svc

kubectl exec frontend -n net-secure2 -- nc -vz server-svc 8080
kubectl exec backend -n net-secure2 -- nc -vz server-svc 8080
server-svc (10.96.15.44:8080) open
command terminated with exit code 1

```

</details>

---

## NP-3 — Allow Egress DNS

- Namespace: `net-secure3`

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
  namespace: net-secure3
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

kubectl create ns net-secure3
kubectl apply -f np-3.yaml
kubectl run test -n net-secure3 --image=busybox --restart=Never -- sleep 3600

# DNS deve funzionare
kubectl exec test -n net-secure3 -- nslookup kubernetes.default

# HTTP deve fallire
kubectl exec test -n net-secure3 -- wget -qO- http://google.com
```

</details>

---

## NP-4 — Namespace Selector

- Namespace target: `net-secure4`

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
  namespace: net-secure4
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
kubectl create ns net-secure4
kubectl create ns trusted
kubectl create ns untrusted

kubectl label ns trusted access=trusted

kubectl apply -f np-4.yaml

# server
kubectl run server -n net-secure4 --image=busybox --restart=Never -- sh -c "nc -lk -p 8080"
kubectl expose pod server -n net-secure4 --port=8080 --name=svc

# client
kubectl run client-ok -n trusted --image=busybox --restart=Never -- sleep 3600
kubectl run client-bad -n untrusted --image=busybox --restart=Never -- sleep 3600

kubectl exec client-ok -n trusted -- nc -vz svc.net-secure4 8080
kubectl exec client-bad -n untrusted -- nc -vz svc.net-secure4 8080
```

</details>

---

## NP-5 — Port Restriction

- Namespace: `net-secure5`

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
  namespace: net-secure5
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  ingress:
  - ports:
    - protocol: TCP
      port: 80

kubectl create ns net-secure5
kubectl apply -f np-5.yaml

kubectl run server -n net-secure5 --image=busybox --restart=Never -- sh -c "nc -lk -p 80 & nc -lk -p 8080"
kubectl expose pod server -n net-secure5 --port=80 --name=svc80
kubectl expose pod server -n net-secure5 --port=8080 --name=svc8080

kubectl run client -n net-secure5 --image=busybox --restart=Never -- sleep 3600

kubectl exec client -n net-secure5 -- nc -vz svc80 80
kubectl exec client -n net-secure5 -- nc -vz svc8080 8080
```

</details>

---

## NP-6 — Combined Policy

- Namespace: `net-secure6`

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
  namespace: net-secure6
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

kubectl create ns net-secure6
kubectl create ns trusted
kubectl label ns trusted access=trusted

kubectl apply -f np-6.yaml

kubectl run server -n net-secure6 --image=busybox --restart=Never -- sh -c "nc -lk -p 443 & nc -lk -p 80"
kubectl expose pod server -n net-secure6 --port=443 --name=svc443
kubectl expose pod server -n net-secure6 --port=80 --name=svc80

kubectl run client -n trusted --image=busybox --restart=Never -- sleep 3600

kubectl exec client -n trusted -- nc -vz svc443.net-secure6 443
kubectl exec client -n trusted -- nc -vz svc80.net-secure6 80
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

kubectl create ns egress-lock
kubectl apply -f np-7.yaml
kubectl describe netpol default-deny-egress -n egress-lock

kubectl run test -n egress-lock --image=busybox --restart=Never -- sleep 3600

kubectl exec test -n egress-lock -- nslookup kubernetes.default
kubectl exec test -n egress-lock -- wget -qO- http://google.com
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

kubectl create ns app-net
kubectl apply -f np-8.yaml

kubectl run backend -n app-net --image=busybox --labels=tier=backend --restart=Never -- sh -c "nc -lk -p 8080"
kubectl expose pod backend -n app-net --port=8080 --name=svc

kubectl run frontend -n app-net --image=busybox --labels=tier=frontend --restart=Never -- sleep 3600
kubectl run other -n app-net --image=busybox --restart=Never -- sleep 3600

kubectl exec frontend -n app-net -- nc -vz svc 8080
kubectl exec other -n app-net -- nc -vz svc 8080
```

</details>

---

## NP-9 — Namespace + Pod selector combinati

- Namespace target: `app-net2`

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
  namespace: app-net2
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

kubectl create ns app-net2
kubectl create ns blue
kubectl create ns red

kubectl label ns blue team=blue

kubectl apply -f np-9.yaml

kubectl run server -n app-net2 --image=busybox --restart=Never -- sh -c "nc -lk -p 8080"
kubectl expose pod server -n app-net2 --port=8080 --name=svc

kubectl run ok -n blue --image=busybox --labels=role=client --restart=Never -- sleep 3600
kubectl run bad -n red --image=busybox --labels=role=client --restart=Never -- sleep 3600

kubectl exec ok -n blue -- nc -vz svc.app-net2 8080
kubectl exec bad -n red -- nc -vz svc.app-net2 8080
```

</details>

---

## NP-10 — Allow egress verso un CIDR specifico

- Namespace: `egress-lock2`

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
  namespace: egress-lock2
spec:
  podSelector: {}
  policyTypes:
  - Egress
  egress:
  - to:
    - ipBlock:
        cidr: 10.0.0.0/24

kubectl create ns egress-lock2
kubectl apply -f np-10.yaml

kubectl run test -n egress-lock2 --image=busybox --restart=Never -- sleep 3600

kubectl exec test -n egress-lock2 -- wget -qO- http://10.0.0.5
kubectl exec test -n egress-lock2 -- wget -qO- http://8.8.8.8
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

kubectl create ns db-net
kubectl apply -f np-11.yaml

kubectl run db -n db-net --image=busybox --labels=app=db --restart=Never -- sh -c "nc -lk -p 8080"
kubectl expose pod db -n db-net --port=8080 --name=svc

kubectl run api -n db-net --image=busybox --labels=app=api --restart=Never -- sleep 3600
kubectl run other -n db-net --image=busybox --restart=Never -- sleep 3600

kubectl exec api -n db-net -- nc -vz svc 8080
kubectl exec other -n db-net -- nc -vz svc 8080
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

kubectl create ns full-lock
kubectl apply -f np-12.yaml

kubectl run server -n full-lock --image=busybox --restart=Never -- sh -c "nc -lk -p 8080"
kubectl expose pod server -n full-lock --port=8080 --name=svc

kubectl run frontend -n full-lock --image=busybox --labels=role=frontend --restart=Never -- sleep 3600
kubectl run other -n full-lock --image=busybox --restart=Never -- sleep 3600

# ingress test
kubectl exec frontend -n full-lock -- nc -vz svc 8080
kubectl exec other -n full-lock -- nc -vz svc 8080

# egress test
kubectl exec frontend -n full-lock -- nslookup kubernetes.default
kubectl exec other -n full-lock -- wget -qO- http://google.com
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

kubectl create ns ip-allow
kubectl apply -f np-13.yaml

kubectl run server -n ip-allow --image=busybox --restart=Never -- sh -c "nc -lk -p 8080"
kubectl expose pod server -n ip-allow --port=8080 --name=svc

kubectl run client -n ip-allow --image=busybox --restart=Never -- sleep 3600

kubectl exec client -n ip-allow -- nc -vz svc 8080
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

kubectl create ns ip-block
kubectl apply -f np-14.yaml

kubectl run server -n ip-block --image=busybox --restart=Never -- sh -c "nc -lk -p 8080"
kubectl expose pod server -n ip-block --port=8080 --name=svc

kubectl run client -n ip-block --image=busybox --restart=Never -- sleep 3600

kubectl exec client -n ip-block -- nc -vz svc 8080
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

kubectl create ns web-egress
kubectl apply -f np-15.yaml

kubectl run test -n web-egress --image=busybox --restart=Never -- sleep 3600

kubectl exec test -n web-egress -- wget -qO- http://google.com
kubectl exec test -n web-egress -- nslookup kubernetes.default
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

kubectl create ns multi-port
kubectl apply -f np-16.yaml

kubectl run server -n multi-port --image=busybox --restart=Never -- sh -c "nc -lk -p 80 & nc -lk -p 443 & nc -lk -p 8080"

kubectl expose pod server -n multi-port --port=80 --name=svc80
kubectl expose pod server -n multi-port --port=443 --name=svc443
kubectl expose pod server -n multi-port --port=8080 --name=svc8080

kubectl run client -n multi-port --image=busybox --restart=Never -- sleep 3600

kubectl exec client -n multi-port -- nc -vz svc80 80
kubectl exec client -n multi-port -- nc -vz svc443 443
kubectl exec client -n multi-port -- nc -vz svc8080 8080
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

kubectl create ns internal-only
kubectl apply -f np-17.yaml

kubectl run server -n internal-only --image=busybox --restart=Never -- sh -c "nc -lk -p 8080"
kubectl expose pod server -n internal-only --port=8080 --name=svc

kubectl run client -n internal-only --image=busybox --restart=Never -- sleep 3600
kubectl run external -n default --image=busybox --restart=Never -- sleep 3600

kubectl exec client -n internal-only -- nc -vz svc 8080
kubectl exec external -n default -- nc -vz svc.internal-only 8080
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

kubectl create ns client-ns
kubectl create ns prod
kubectl label ns prod env=prod

kubectl apply -f np-18.yaml

kubectl run server -n prod --image=busybox --restart=Never -- sh -c "nc -lk -p 8080"
kubectl expose pod server -n prod --port=8080 --name=svc

kubectl run test -n client-ns --image=busybox --restart=Never -- sleep 3600

kubectl exec test -n client-ns -- nc -vz svc.prod 8080
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

kubectl create ns fine-grain
kubectl apply -f np-19.yaml

kubectl run server -n fine-grain --image=busybox --labels=app=server --restart=Never -- sh -c "nc -lk -p 8080"
kubectl expose pod server -n fine-grain --port=8080 --name=svc

kubectl run client -n fine-grain --image=busybox --labels=app=client --restart=Never -- sleep 3600
kubectl run other -n fine-grain --image=busybox --restart=Never -- sleep 3600

kubectl exec client -n fine-grain -- nc -vz svc 8080
kubectl exec other -n fine-grain -- nc -vz svc 8080
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

kubectl create ns multi-rule
kubectl apply -f np-20.yaml

kubectl run server -n multi-rule --image=busybox --restart=Never -- sh -c "nc -lk -p 8080"
kubectl expose pod server -n multi-rule --port=8080 --name=svc

kubectl run f -n multi-rule --image=busybox --labels=role=frontend --restart=Never -- sleep 3600
kubectl run a -n multi-rule --image=busybox --labels=role=admin --restart=Never -- sleep 3600
kubectl run o -n multi-rule --image=busybox --restart=Never -- sleep 3600

kubectl exec f -n multi-rule -- nc -vz svc 8080
kubectl exec a -n multi-rule -- nc -vz svc 8080
kubectl exec o -n multi-rule -- nc -vz svc 8080
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

kubectl create ns and-logic
kubectl create ns red
kubectl label ns red team=red

kubectl apply -f np-21.yaml

kubectl run server -n and-logic --image=busybox --restart=Never -- sh -c "nc -lk -p 8080"
kubectl expose pod server -n and-logic --port=8080 --name=svc

kubectl run ok -n red --image=busybox --labels=app=client --restart=Never -- sleep 3600
kubectl run bad -n red --image=busybox --restart=Never -- sleep 3600

kubectl exec ok -n red -- nc -vz svc.and-logic 8080
kubectl exec bad -n red -- nc -vz svc.and-logic 8080
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

kubectl create ns egress-pod
kubectl apply -f np-22.yaml

kubectl run db -n egress-pod --image=busybox --labels=app=db --restart=Never -- sh -c "nc -lk -p 8080"
kubectl expose pod db -n egress-pod --port=8080 --name=svc

kubectl run test -n egress-pod --image=busybox --restart=Never -- sleep 3600

kubectl exec test -n egress-pod -- nc -vz svc 8080
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

kubectl create ns restricted-egress
kubectl apply -f np-23.yaml

kubectl run test -n restricted-egress --image=busybox --restart=Never -- sleep 3600

kubectl exec test -n restricted-egress -- nslookup kubernetes.default
kubectl exec test -n restricted-egress -- wget -qO- http://google.com
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

kubectl create ns partial
kubectl apply -f np-24.yaml

kubectl run web -n partial --image=busybox --labels=app=web --restart=Never -- sh -c "nc -lk -p 8080"
kubectl expose pod web -n partial --port=8080 --name=svc

kubectl run other -n partial --image=busybox --restart=Never -- sleep 3600

kubectl exec other -n partial -- nc -vz svc 8080
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
kubectl create ns debug-ns

# controlla le label reali
kubectl get pods -n debug-ns --show-labels

# controlla la policy
kubectl describe netpol -n debug-ns

# se la policy non matcha nessun pod, correggi il selector e riapplica
kubectl apply -f np-25.yaml
kubectl get netpol -n debug-ns
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

kubectl create ns mixed
kubectl apply -f np-26.yaml

kubectl run test -n mixed --image=busybox --restart=Never -- sleep 3600

kubectl exec test -n mixed -- nc -vz kubernetes.default 443
kubectl exec test -n mixed -- wget -qO- http://google.com
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

kubectl create ns svc-egress
kubectl apply -f np-27.yaml

kubectl run test -n svc-egress --image=busybox --restart=Never -- sleep 3600

# verso Service ClusterIP deve funzionare
kubectl exec test -n svc-egress -- nslookup kubernetes.default
kubectl exec test -n svc-egress -- nc -vz kubernetes.default 443

# verso IP esterno deve fallire
kubectl exec test -n svc-egress -- wget -qO- http://8.8.8.8
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

kubectl create ns self-comm
kubectl apply -f np-28.yaml

kubectl run api1 -n self-comm --image=busybox --labels=app=api --restart=Never -- sh -c "nc -lk -p 8080"
kubectl expose pod api1 -n self-comm --port=8080 --name=svc

kubectl run api2 -n self-comm --image=busybox --labels=app=api --restart=Never -- sleep 3600
kubectl run other -n self-comm --image=busybox --restart=Never -- sleep 3600

kubectl exec api2 -n self-comm -- nc -vz svc 8080
kubectl exec other -n self-comm -- nc -vz svc 8080
```

</details>

---

## NP-29 — Multiple namespaceSelector

* Namespace: `multi-ns2`
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
  namespace: multi-ns2
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

kubectl create ns multi-ns2
kubectl create ns dev
kubectl create ns staging
kubectl create ns other

kubectl label ns dev env=dev
kubectl label ns staging env=staging

kubectl apply -f np-29.yaml

kubectl run server -n multi-ns2 --image=busybox --restart=Never -- sh -c "nc -lk -p 8080"
kubectl expose pod server -n multi-ns2 --port=8080 --name=svc

kubectl run client-dev -n dev --image=busybox --restart=Never -- sleep 3600
kubectl run client-staging -n staging --image=busybox --restart=Never -- sleep 3600
kubectl run client-other -n other --image=busybox --restart=Never -- sleep 3600

kubectl exec client-dev -n dev -- nc -vz svc.multi-ns2 8080
kubectl exec client-staging -n staging -- nc -vz svc.multi-ns2 8080
kubectl exec client-other -n other -- nc -vz svc.multi-ns2 8080
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


kubectl create ns zero-trust
kubectl apply -f np-30.yaml

kubectl run db -n zero-trust --image=busybox --labels=app=db --restart=Never -- sh -c "nc -lk -p 8080"
kubectl expose pod db -n zero-trust --port=8080 --name=db-svc

kubectl run target -n zero-trust --image=busybox --restart=Never -- sh -c "nc -lk -p 8081"
kubectl expose pod target -n zero-trust --port=8081 --name=target-svc

kubectl run frontend -n zero-trust --image=busybox --labels=role=frontend --restart=Never -- sleep 3600
kubectl run other -n zero-trust --image=busybox --restart=Never -- sleep 3600

# ingress verso pod protetti
kubectl exec frontend -n zero-trust -- nc -vz target-svc 8081
kubectl exec other -n zero-trust -- nc -vz target-svc 8081

# egress dai pod protetti
kubectl exec frontend -n zero-trust -- nc -vz db-svc 8080
kubectl exec frontend -n zero-trust -- nc -vz target-svc 8081
```

</details>

---

## NP-31 — Frontend ↔ Backend ↔ Database Control

- Namespace: `project-net`

- Creare una o più NetworkPolicy
  - Consentire ai Pod `backend` di ricevere traffico **solo** dai Pod `frontend`
  - Consentire ai Pod `database` di ricevere traffico **solo** dai Pod `backend`
  - Consentire ai Pod `backend` di uscire **solo** verso i Pod `database`
  - Consentire ai Pod `frontend` di uscire **solo** verso i Pod `backend` e `external`
  - Impedire ai Pod `database` qualsiasi traffico in uscita

- Specifiche
  - Usare solo label selector
  - Label coinvolte:
    - `app=frontend`
    - `app=backend`
    - `app=database`
    - `app=external`
  - Usare `Ingress` ed `Egress`
  - È possibile creare più di una NetworkPolicy
  - Non usare indirizzi IP

- Validazione
  - `frontend` → `backend` consentito
  - `frontend` → `database` negato
  - `backend` → `database` consentito
  - `backend` → altri Pod negato
  - `database` → qualsiasi destinazione negato
  - Pod diversi da `frontend` → `backend` negato
  - Pod diversi da `backend` → `database` negato

<details>
<summary>Soluzione</summary>

```sh
k create ns project-net

apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: backend-policy
  namespace: project-net
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
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: database-policy
  namespace: project-net
spec:
  podSelector:
    matchLabels:
      app: database
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: backend
  egress: []
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: frontend-policy
  namespace: project-net
spec:
  podSelector:
    matchLabels:
      app: frontend
  policyTypes:
  - Egress
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: backend
  - to:
    - podSelector:
        matchLabels:
          app: external
```

```sh
kubectl create ns project-net
kubectl apply -f np-31.yaml

kubectl run backend -n project-net --image=busybox --labels=app=backend --restart=Never -- sh -c "nc -lk -p 8080"
kubectl expose pod backend -n project-net --port=8080 --name=backend-svc

kubectl run database -n project-net --image=busybox --labels=app=database --restart=Never -- sh -c "nc -lk -p 5432"
kubectl expose pod database -n project-net --port=5432 --name=database-svc

kubectl run external -n project-net --image=busybox --labels=app=external --restart=Never -- sh -c "nc -lk -p 9090"
kubectl expose pod external -n project-net --port=9090 --name=external-svc

kubectl run frontend -n project-net --image=busybox --labels=app=frontend --restart=Never -- sleep 3600
kubectl run other -n project-net --image=busybox --restart=Never -- sleep 3600

# frontend -> backend OK
kubectl exec frontend -n project-net -- nc -vz backend-svc 8080

# frontend -> database FAIL
kubectl exec frontend -n project-net -- nc -vz database-svc 5432

# other -> backend FAIL
kubectl exec other -n project-net -- nc -vz backend-svc 8080

# other -> database FAIL
kubectl exec other -n project-net -- nc -vz database-svc 5432

# backend -> database OK
kubectl exec backend -n project-net -- nc -vz database-svc 5432

# backend -> external FAIL
kubectl exec backend -n project-net -- nc -vz external-svc 9090

# frontend -> external OK
kubectl exec frontend -n project-net -- nc -vz external-svc 9090

# database -> backend FAIL
kubectl exec database -n project-net -- nc -vz backend-svc 8080
```
</details>



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

server pod:
  nc -lk -p 8080   → ascolta

client pod:
  nc -vz server 8080 → testa connessione

nc -vz host port
nc → netcat (un tool che apre connessioni TCP/UDP)
-v → verbose
-z → scan (non manda dati, solo test connessione)

nc -lk -p 8080  → apri porta 8080 e resta in ascolto per sempre

-l → listen [il pod si mette in ascolto (diventa un “server”)]
-k → keep alive (resta in ascolto anche dopo una connessione senza questo → si chiude dopo 1 connessione)
-p 8080 → porta su cui ascoltare


```
