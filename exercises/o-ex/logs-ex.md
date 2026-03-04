### Logs (sei esercizi)

---

## LOG-1 — Visualizzare Logs di un Pod

---
- Pod: `nginx-app`
- Container: nginx

- Obiettivo:
  - Visualizzare i log in tempo reale

- Validazione:
  - Comando mostra output del container

<details> <summary>Soluzione</summary>
  
```
  kubectl logs nginx-app -c nginx -f
```
</details>

---

## LOG-2 — Logs di Pod Multi-Container

- Pod: `multi-app`
- Containers: frontend, backend

- Obiettivo:
  - Visualizzare i log solo del container backend

- Validazione:
  - Output mostra solo log del backend

<details> <summary>Soluzione</summary>
  
```
kubectl logs multi-app -c backend
  
```
</details>

---

## LOG-3 — Logs Storici (Pod Restartato)

- Pod: `restart-app`
- Container: app
- Obiettivo:
  - Visualizzare i log del container prima del riavvio

- Validazione:
  - Comando mostra log precedenti al crash/restart

<details> <summary>Soluzione</summary>
  
```
kubectl logs restart-app -c app --previous
  
```
</details>

---

## OBS-1 — Container Exec per Debug

- Pod: `debug-app`
- Container: nginx
- Obiettivo:
  - Entrare nel container per diagnosticare
- Validazione:
  - Shell funzionante dentro il container

<details> <summary>Soluzione</summary>
  
```
kubectl exec -it debug-app -c nginx -- /bin/sh
  
```
</details>

---

## OBS-2 — Eventi di Namespace

- Namespace: `observability`
- Obiettivo:
  - Visualizzare eventi recenti dei Pod
- Validazione:
  - Mostra crash, scheduling e warning dei pod

<details> <summary>Soluzione</summary>
  
```
kubectl get events -n observability --sort-by='.metadata.creationTimestamp'
  
```
</details>

---

## OBS-3 — Container Resource Usage

- Pod: `metrics-app`
- Container: app
- Obiettivo:
  - Visualizzare uso CPU e memoria in tempo reale
- Validazione:
  - Mostra consumo risorse del container

<details> <summary>Soluzione</summary>
  
```
kubectl top pod metrics-app -c app
```
</details>

---
