- [ Home ](../../readme.md)   | [ Teoria ](../../arguments.md)   | [ Info Exam ](../../doc/ckad_exam_strategy.md)    | [ Home Other Exercises ](../o_exercises.md)   
--- 

### Service senza Selector e EndpointSlice (10 esercizi)
---

## ES-1 — Service senza selector base

- Service: `external-api`

- Configurazione
  - Creare un Service senza `selector`
  - Porta Service: `8080`
  - targetPort: `8080`

- Validazione
  - Il Service esiste
  - Il manifest non contiene `selector`

---

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: v1
kind: Service
metadata:
  name: external-api
spec:
  ports:
  - port: 8080
    targetPort: 8080
    protocol: TCP
```

```sh
k apply -f external-api-svc.yaml
k get svc external-api
k get svc external-api -o yaml
```

</details>

---

## ES-2 — EndpointSlice per Service senza selector

- Service esistente: `external-api`

- EndpointSlice: `external-api-1`

- Configurazione
  - Collegare il Service all'IP `10.0.0.50`
  - Porta: `8080`
  - addressType: `IPv4`

- Validazione
  - La EndpointSlice esiste
  - La label collega la EndpointSlice al Service

---

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: discovery.k8s.io/v1
kind: EndpointSlice
metadata:
  name: external-api-1
  labels:
    kubernetes.io/service-name: external-api
addressType: IPv4
ports:
- name: tcp
  protocol: TCP
  port: 8080
endpoints:
- addresses:
  - 10.0.0.50
```

```sh
k apply -f external-api-eps.yaml
k get endpointslice external-api-1
k get endpointslice -l kubernetes.io/service-name=external-api
```

</details>

---

## ES-3 — Service database esterno

- Service: `external-db`

- EndpointSlice: `external-db-1`

- Configurazione
  - Creare un Service senza selector
  - Porta Service: `5432`
  - Endpoint esterno: `192.168.1.100`
  - Porta endpoint: `5432`

- Validazione
  - Service creato
  - EndpointSlice associata correttamente

---

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: v1
kind: Service
metadata:
  name: external-db
spec:
  ports:
  - name: postgres
    port: 5432
    targetPort: 5432
    protocol: TCP
---
apiVersion: discovery.k8s.io/v1
kind: EndpointSlice
metadata:
  name: external-db-1
  labels:
    kubernetes.io/service-name: external-db
addressType: IPv4
ports:
- name: postgres
  protocol: TCP
  port: 5432
endpoints:
- addresses:
  - 192.168.1.100
```

```sh
k apply -f external-db.yaml
k get svc external-db
k get endpointslice -l kubernetes.io/service-name=external-db
```

</details>

---

## ES-4 — EndpointSlice con due backend

- Service: `web-external`

- EndpointSlice: `web-external-1`

- Configurazione
  - Service senza selector
  - Porta Service: `80`
  - Backend:
    - `192.168.1.10:80`
    - `192.168.1.11:80`

- Validazione
  - La EndpointSlice contiene due indirizzi

---

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-external
spec:
  ports:
  - name: http
    port: 80
    targetPort: 80
---
apiVersion: discovery.k8s.io/v1
kind: EndpointSlice
metadata:
  name: web-external-1
  labels:
    kubernetes.io/service-name: web-external
addressType: IPv4
ports:
- name: http
  protocol: TCP
  port: 80
endpoints:
- addresses:
  - 192.168.1.10
- addresses:
  - 192.168.1.11
```

```sh
k apply -f web-external.yaml
k describe endpointslice web-external-1
```

</details>
