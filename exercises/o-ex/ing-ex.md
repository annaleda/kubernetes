
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
