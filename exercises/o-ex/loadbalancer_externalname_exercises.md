- [ Home ](../../readme.md)   | [ Teoria ](../../arguments.md)   | [ Info Exam ](../../doc/ckad_exam_strategy.md)    | [ Home Other Exercises ](../o_exercises.md)   
--- 

### LoadBalancer e ExternalName (10 esercizi)
---

## LB-1 — Service LoadBalancer base

- Deployment: `web-lb`
  - Image: `nginx`

- Service: `web-lb-svc`

- Configurazione
  - Esporre il Deployment con un Service di tipo `LoadBalancer`
  - Porta Service: `80`
  - targetPort: `80`

- Validazione
  - Il Service esiste
  - TYPE = `LoadBalancer`

---

<details>
<summary>Soluzione</summary>

```sh
k create deploy web-lb --image=nginx
```

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-lb-svc
spec:
  type: LoadBalancer
  selector:
    app: web-lb
  ports:
  - port: 80
    targetPort: 80
```

```sh
k apply -f web-lb-svc.yaml
k get svc web-lb-svc
```

</details>

---

## LB-2 — LoadBalancer con porta personalizzata

- Deployment: `api-lb`
  - Image: `nginx`

- Service: `api-lb-svc`

- Configurazione
  - Tipo: `LoadBalancer`
  - Porta esposta: `8080`
  - targetPort: `80`

- Validazione
  - Il Service espone la porta `8080`

---

<details>
<summary>Soluzione</summary>

```sh
k create deploy api-lb --image=nginx
```

```yaml
apiVersion: v1
kind: Service
metadata:
  name: api-lb-svc
spec:
  type: LoadBalancer
  selector:
    app: api-lb
  ports:
  - port: 8080
    targetPort: 80
```

```sh
k apply -f api-lb-svc.yaml
k get svc api-lb-svc
```

</details>

---

## LB-3 — LoadBalancer in namespace dedicato

- Namespace: `lb-test`
- Deployment: `frontend`
  - Image: `nginx`

- Service: `frontend-lb`

- Configurazione
  - Creare tutto nel namespace `lb-test`
  - Tipo Service: `LoadBalancer`
  - Porta: `80`

- Validazione
  - Il Service è nel namespace corretto
  - TYPE = `LoadBalancer`

---

<details>
<summary>Soluzione</summary>

```sh
k create ns lb-test
k -n lb-test create deploy frontend --image=nginx
```

```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend-lb
  namespace: lb-test
spec:
  type: LoadBalancer
  selector:
    app: frontend
  ports:
  - port: 80
    targetPort: 80
```

```sh
k apply -f frontend-lb.yaml
k -n lb-test get svc frontend-lb
```

</details>

---

## LB-4 — LoadBalancer con due porte

- Deployment: `multi-port-app`
  - Image: `nginx`

- Service: `multi-port-lb`

- Configurazione
  - Tipo: `LoadBalancer`
  - Esporre:
    - porta `80` -> targetPort `80`
    - porta `443` -> targetPort `80`

- Validazione
  - Il Service ha due porte configurate

---

<details>
<summary>Soluzione</summary>

```sh
k create deploy multi-port-app --image=nginx
```

```yaml
apiVersion: v1
kind: Service
metadata:
  name: multi-port-lb
spec:
  type: LoadBalancer
  selector:
    app: multi-port-app
  ports:
  - name: http
    port: 80
    targetPort: 80
  - name: https
    port: 443
    targetPort: 80
```

```sh
k apply -f multi-port-lb.yaml
k describe svc multi-port-lb
```

</details>

---

## LB-5 — Convertire ClusterIP in LoadBalancer

- Service esistente: `web-internal`

- Configurazione
  - Modificare il Service esistente
  - Cambiare il tipo da `ClusterIP` a `LoadBalancer`

- Validazione
  - TYPE = `LoadBalancer`

---

<details>
<summary>Soluzione</summary>

```sh
k edit svc web-internal
```

Campo da modificare:

```yaml
spec:
  type: LoadBalancer
```

Oppure:

```sh
kubectl patch svc web-internal -p '{"spec":{"type":"LoadBalancer"}}'
```

```sh
k get svc web-internal
```

</details>

---

## EN-6 — ExternalName base

- Service: `external-db`

- Configurazione
  - Tipo: `ExternalName`
  - externalName: `mydb.example.com`

- Validazione
  - Il Service esiste
  - TYPE = `ExternalName`

---

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: v1
kind: Service
metadata:
  name: external-db
spec:
  type: ExternalName
  externalName: mydb.example.com
```

```sh
k apply -f external-db.yaml
k get svc external-db
```

</details>

---

## EN-7 — ExternalName per API esterna

- Namespace: `ext-services`
- Service: `weather-api`

- Configurazione
  - Creare namespace `ext-services`
  - Tipo: `ExternalName`
  - externalName: `api.weather.example.com`

- Validazione
  - Service creato nel namespace corretto

---

<details>
<summary>Soluzione</summary>

```sh
k create ns ext-services
```

```yaml
apiVersion: v1
kind: Service
metadata:
  name: weather-api
  namespace: ext-services
spec:
  type: ExternalName
  externalName: api.weather.example.com
```

```sh
k apply -f weather-api.yaml
k -n ext-services get svc weather-api
```

</details>

---

## EN-8 — Verificare risoluzione DNS di ExternalName

- Service: `external-db`
- Pod temporaneo: `dns-test`
  - Image: `busybox`

- Configurazione
  - Usare un Pod per verificare la risoluzione DNS del Service `external-db`

- Validazione
  - Il nome viene risolto tramite DNS

---

<details>
<summary>Soluzione</summary>

```sh
k run dns-test --image=busybox --restart=Never -it --rm -- nslookup external-db
```

Oppure:

```sh
k run dns-test --image=busybox --restart=Never -- sleep 3600
k exec -it dns-test -- nslookup external-db
```

</details>

---

## EN-9 — Convertire Service esistente in ExternalName

- Service esistente: `legacy-db`

- Configurazione
  - Cambiare tipo in `ExternalName`
  - externalName: `legacy.database.example.com`

- Validazione
  - TYPE = `ExternalName`
  - Nessun selector necessario

---

<details>
<summary>Soluzione</summary>

```sh
k edit svc legacy-db
```

Esempio finale:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: legacy-db
spec:
  type: ExternalName
  externalName: legacy.database.example.com
```

```sh
k get svc legacy-db -o yaml
```

</details>

---

## EN-10 — ExternalName senza selector

- Service: `external-cache`

- Configurazione
  - Creare un Service di tipo `ExternalName`
  - externalName: `redis.example.com`
  - Non usare `selector`

- Validazione
  - Il manifest non contiene `selector`
  - TYPE = `ExternalName`

---

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: v1
kind: Service
metadata:
  name: external-cache
spec:
  type: ExternalName
  externalName: redis.example.com
```

```sh
k apply -f external-cache.yaml
k get svc external-cache -o yaml
```

</details>

---
