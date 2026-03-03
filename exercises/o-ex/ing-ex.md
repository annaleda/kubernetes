
###  Ingress (6 esercizi)

## ING-1 — Ingress Base

- Creare Deployment `ingress-web`
- Creare Service `ingress-svc`
- Creare Ingress `web-ingress`
- Specifiche
  - Host: app.local
  - Path: `/`
  - Backend: ingress-svc
- Validazione
  - kubectl describe ingress
---
<details>
<summary>Soluzione</summary>
  
```
 k create deploy ingress-web --image=nginx --replicas=2

 k expose deploy ingress-web --name ingress-svc --port=82

 vi ingress.yaml



apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-ingress
spec:
  ingressClassName: nginx
  rules:
  - http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: ingress-svc
            port:
              number: 82

k apply -f ingress.yaml

```
</details>

---

## ING-2 — Path Based Routing

- Due Deployment:
  - frontend
  - backend
- Ingress
  - `/front` → frontend
  - `/back` → backend
- Validazione
  - Routing corretto

---
<details>
<summary>Soluzione</summary>
  
```
 k create deploy frontend --image=nginx --replicas=2
 k create deploy backend --image=nginx --replicas=2

 k expose deploy frontend --name f-svc --port=83 --target-port=80
 k expose deploy backend --name b-svc --port=84 --target-port=80

vi ingress2.yaml



apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: full-ingress
spec:
  ingressClassName: nginx
  rules:
  - http:
      paths:
      - path: /front
        pathType: Prefix
        backend:
          service:
            name: f-svc
            port:
              number: 83
      - path: /back
        pathType: Prefix
        backend:
          service:
            name: b-svc
            port:
              number: 84

k apply -f ingress2.yaml

kubectl get ingress
kubectl describe ingress full-ingress

curl http://<IP>/front
curl http://<IP>/back
```
</details>

---

## ING-3 — TLS

- Creare Secret TLS
  - ( MSYS_NO_PATHCONV=1 openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout tls.key -out tls.crt -subj "/CN=secure.local" # da git bash)
- Ingress
  - TLS abilitato
  - Host: secure.local
- Validazione
  - Sezione TLS presente
---
<details>
<summary>Soluzione</summary>
  
```
 MSYS_NO_PATHCONV=1 openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout tls.key -out tls.crt -subj "/CN=secure.local" # da git bash

kubectl create secret tls secure-tls --cert=tls.crt --key=tls.key

kubectl get secret secure-tls

kubectl describe secret secure-tls

k create deploy s-deploy --image=nginx --replicas=2
k expose deploy s-deploy --name s-service --port=80 --target-port=80

vi secure-ingress.yaml

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: secure-ingress
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - secure.local           # se arriva request da https://secure.local usa
    secretName: secure-tls   # secret creato in precedenza
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
</details>

---

### ING-4 — Multiple Hosts

- Ingress
  - app1.local
  - app2.local

- Backend distinti
- Validazione
  - Regole host separate

---
<details>
<summary>Soluzione</summary>
  
```  
```
</details>

---

### ING-5 — Default Backend

- Ingress con default backend
- Validazione
  - Richiesta a host sconosciuto va al default
---
<details>
<summary>Soluzione</summary>
  
```  
```
</details>

---

### ING-6 — Rewrite Target

- Usare annotation rewrite-target
- Path `/app` → `/`
- Validazione
  - Routing corretto
---
<details>
<summary>Soluzione</summary>
  
```  
```
</details>

---
