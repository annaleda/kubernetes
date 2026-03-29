- [ Home ](../../readme.md)   | [ Teoria ](../../arguments.md)   | [ Info Exam ](../../doc/ckad_exam_strategy.md)    | [ Home Other Exercises ](../o_exercises.md)   
---
  ### Ingress (12 esercizi)

---

## ING-1 — Ingress Base

- Creare Deployment `ingress-web`
- Creare Service `ingress-svc`
- Creare Ingress `web-ingress`

- Specifiche
  - Host: `app.local`
  - Path: `/`
  - Backend: `ingress-svc`

- Validazione
  - `kubectl describe ingress web-ingress`

<details>
<summary>Soluzione</summary>

```sh
k create deploy ingress-web --image=nginx --replicas=2
k expose deploy ingress-web --name ingress-svc --port=80 --target-port=80
```

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-ingress
spec:
  rules:
  - host: app.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: ingress-svc
            port:
              number: 80
```

```sh
k apply -f ingress.yaml
k describe ingress web-ingress
```

</details>

---

## ING-2 — Path Based Routing

- Due Deployment:
  - `frontend`
  - `backend`

- Ingress
  - `/front` → `frontend`
  - `/back` → `backend`

- Validazione
  - Routing corretto

<details>
<summary>Soluzione</summary>

```sh
k create deploy frontend --image=nginx --replicas=2
k create deploy backend --image=nginx --replicas=2

k expose deploy frontend --name f-svc --port=80 --target-port=80
k expose deploy backend --name b-svc --port=80 --target-port=80
```

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: full-ingress
spec:
  rules:
  - http:
      paths:
      - path: /front
        pathType: Prefix
        backend:
          service:
            name: f-svc
            port:
              number: 80
      - path: /back
        pathType: Prefix
        backend:
          service:
            name: b-svc
            port:
              number: 80
```

```sh
k apply -f ingress2.yaml
kubectl describe ingress full-ingress
```

</details>

---

## ING-3 — TLS

- Creare Secret TLS
- Creare Ingress con TLS abilitato
- Host: `secure.local`

- Validazione
  - Sezione TLS presente

<details>
<summary>Soluzione</summary>

```sh
MSYS_NO_PATHCONV=1 openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout tls.key -out tls.crt -subj "/CN=secure.local"

kubectl create secret tls secure-tls --cert=tls.crt --key=tls.key

k create deploy s-deploy --image=nginx --replicas=2
k expose deploy s-deploy --name s-service --port=80 --target-port=80
```

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: secure-ingress
spec:
  tls:
  - hosts:
    - secure.local
    secretName: secure-tls
  rules:
  - host: secure.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: s-service
            port:
              number: 80
```

```sh
k apply -f secure-ingress.yaml
k describe ingress secure-ingress
```

</details>

---

## ING-4 — Multiple Hosts

- Ingress con host separati
  - `app1.local`
  - `app2.local`

- Backend distinti

- Validazione
  - Regole host separate

<details>
<summary>Soluzione</summary>

```sh
k create deploy dep-1 --image=nginx --replicas=2
k create deploy dep-2 --image=nginx --replicas=2

k expose deploy dep-1 --name app1-svc --port=80 --target-port=80
k expose deploy dep-2 --name app2-svc --port=80 --target-port=80
```

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: multi-host-ingress
spec:
  rules:
  - host: app1.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: app1-svc
            port:
              number: 80
  - host: app2.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: app2-svc
            port:
              number: 80
```

```sh
k apply -f multi-host-ingress.yaml
k describe ingress multi-host-ingress
```

</details>

---

## ING-5 — Default Backend

- Ingress con default backend

- Validazione
  - Richiesta a host sconosciuto va al default backend

<details>
<summary>Soluzione</summary>

```sh
k create deploy default-app --image=nginx
k expose deploy default-app --port=80 --name=default-svc

k create deploy dep-1 --image=nginx --replicas=2
k expose deploy dep-1 --name app1-svc --port=80 --target-port=80
```

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-default
spec:
  defaultBackend:
    service:
      name: default-svc
      port:
        number: 80
  rules:
  - host: app1.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: app1-svc
            port:
              number: 80
```

```sh
kubectl apply -f ingress-default.yaml
kubectl describe ingress ingress-default
```

</details>

---

## ING-6 — Rewrite Target

- Usare annotation `rewrite-target`
- Path `/app` → `/`

- Validazione
  - Routing corretto

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: rewrite-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - http:
      paths:
      - path: /app
        pathType: Prefix
        backend:
          service:
            name: app1-svc
            port:
              number: 80
```

```sh
kubectl apply -f rewrite-ingress.yaml
kubectl get ingress rewrite-ingress -o yaml
```

</details>

---

## ING-7 — Prefix vs Exact

- Creare due regole:
  - `/app` con `pathType: Prefix`
  - `/login` con `pathType: Exact`

- Backend:
  - `app-svc`
  - `login-svc`

- Validazione
  - I path usano i `pathType` richiesti

<details>
<summary>Soluzione</summary>

```sh
k create deploy app-deploy --image=nginx
k create deploy login-deploy --image=nginx

k expose deploy app-deploy --name app-svc --port=80
k expose deploy login-deploy --name login-svc --port=80
```

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: path-types-ingress
spec:
  rules:
  - host: paths.local
    http:
      paths:
      - path: /app
        pathType: Prefix
        backend:
          service:
            name: app-svc
            port:
              number: 80
      - path: /login
        pathType: Exact
        backend:
          service:
            name: login-svc
            port:
              number: 80
```

</details>

---

## ING-8 — Ingress con ingressClassName

- Creare Ingress `class-ingress`

- Specifiche
  - `ingressClassName: nginx`
  - Host: `class.local`
  - Path: `/`

- Validazione
  - `kubectl get ingress`
  - classe presente nel manifest

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: class-ingress
spec:
  ingressClassName: nginx
  rules:
  - host: class.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: ingress-svc
            port:
              number: 80
```

</details>

---

## ING-9 — Debug Service name errato

- Ingress `broken-ingress`
- Problema:
  - backend punta a `wrong-svc`

- Obiettivo
  - Identificare errore
  - Correggere il nome del Service

- Validazione
  - Ingress coerente con i Service esistenti

<details>
<summary>Soluzione</summary>

Manifest errato:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: broken-ingress
spec:
  rules:
  - host: broken.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: wrong-svc
            port:
              number: 80
```

Debug:

```sh
kubectl get svc
kubectl describe ingress broken-ingress
```

Fix:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: broken-ingress
spec:
  rules:
  - host: broken.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: ingress-svc
            port:
              number: 80
```

</details>

---

## ING-10 — Debug pathType mancante

- Manifest Ingress non valido
- Problema:
  - `pathType` mancante

- Obiettivo
  - Correggere il manifest

<details>
<summary>Soluzione</summary>

Errore tipico:

```yaml
- path: /
  backend:
    service:
      name: ingress-svc
      port:
        number: 80
```

Fix:

```yaml
- path: /
  pathType: Prefix
  backend:
    service:
      name: ingress-svc
      port:
        number: 80
```

</details>

---

## ING-11 — Host-based routing con 3 host

- Host:
  - `shop.local`
  - `blog.local`
  - `admin.local`

- Backend:
  - `shop-svc`
  - `blog-svc`
  - `admin-svc`

- Validazione
  - 3 regole host distinte

<details>
<summary>Soluzione</summary>

```sh
k create deploy shop --image=nginx
k create deploy blog --image=nginx
k create deploy admin --image=nginx

k expose deploy shop --name shop-svc --port=80
k expose deploy blog --name blog-svc --port=80
k expose deploy admin --name admin-svc --port=80
```

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: multi-host-3-ingress
spec:
  rules:
  - host: shop.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: shop-svc
            port:
              number: 80
  - host: blog.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: blog-svc
            port:
              number: 80
  - host: admin.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: admin-svc
            port:
              number: 80
```

</details>

---

## ING-12 — Ingress con TLS e host multipli

- Creare Secret TLS `multi-tls`
- Host:
  - `app1.local`
  - `app2.local`

- Obiettivo
  - Un solo secret TLS per più host

- Validazione
  - sezione TLS presente con entrambi gli host

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: multi-tls-ingress
spec:
  tls:
  - hosts:
    - app1.local
    - app2.local
    secretName: multi-tls
  rules:
  - host: app1.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: app1-svc
            port:
              number: 80
  - host: app2.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: app2-svc
            port:
              number: 80
```

</details>

---

## Cheatsheet Ingress

```sh
# vedere ingress
kubectl get ingress

# descrivere ingress
kubectl describe ingress <name>

# vedere yaml completo
kubectl get ingress <name> -o yaml

# vedere service backend
kubectl get svc

# controllare endpoints
kubectl get endpoints
```

---

## Mini schema mentale

```text
Ingress
  host + path
      ↓
   Service
      ↓
    Pod

Ingress moderno:
- apiVersion: networking.k8s.io/v1
- pathType obbligatorio
- backend.service.name
- backend.service.port.number
```
