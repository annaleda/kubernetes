
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
- Ingress
  - TLS abilitato
  - Host: secure.local
- Validazione
  - Sezione TLS presente
---
<details>
<summary>Soluzione</summary>
  
```  
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
