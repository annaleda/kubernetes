### Logs (12 esercizi)

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

## LOG-4 — Logs di tutti i container

- Pod: `multi-app`
- Containers: frontend, backend

- Obiettivo:
  - Visualizzare i log di TUTTI i container

- Validazione:
  - Output mostra log di entrambi

<details> <summary>Soluzione</summary>

```sh
kubectl logs multi-app --all-containers=true
```

</details>

---

## LOG-5 — Logs con filtro (grep)

- Pod: `error-app`

- Obiettivo:
  - Mostrare solo log contenenti "ERROR"

- Validazione:
  - Output filtrato

<details> <summary>Soluzione</summary>

```sh
kubectl logs error-app | grep ERROR
```

</details>

---

## LOG-6 — Logs di più Pod (Deployment)

- Deployment: `web-deploy`

- Obiettivo:
  - Visualizzare logs di tutti i Pod del Deployment

- Validazione:
  - Output aggregato

<details> <summary>Soluzione</summary>

```sh
kubectl logs -l app=web-deploy
```

oppure:

```sh
kubectl get pods -l app=web-deploy
kubectl logs <pod-name>
```

</details>

---

## LOG-7 — Logs con limite righe

- Pod: `nginx-app`

- Obiettivo:
  - Mostrare solo le ultime 20 righe

- Validazione:
  - Output limitato

<details> <summary>Soluzione</summary>

```sh
kubectl logs nginx-app --tail=20
```

</details>

---

## LOG-8 — Logs con timestamp

- Pod: `time-app`

- Obiettivo:
  - Visualizzare logs con timestamp

- Validazione:
  - Output include timestamp

<details> <summary>Soluzione</summary>

```sh
kubectl logs time-app --timestamps
```

</details>

---

## LOG-9 — Logs da un intervallo temporale

- Pod: `recent-app`

- Obiettivo:
  - Visualizzare log ultimi 5 minuti

- Validazione:
  - Output limitato nel tempo

<details> <summary>Soluzione</summary>

```sh
kubectl logs recent-app --since=5m
```

</details>

---

## OBS-4 — Descrivere Pod (debug completo)

- Pod: `broken-app`

- Obiettivo:
  - Identificare errori (CrashLoop, ImagePull, Probe)

- Validazione:
  - Output mostra eventi e stato container

<details> <summary>Soluzione</summary>

```sh
kubectl describe pod broken-app
```

</details>

---

## OBS-5 — Logs + Exec combinati

- Pod: `investigate-app`

- Obiettivo:
  - Leggere logs
  - Entrare nel container
  - Controllare file interni

- Validazione:
  - Debug completo

<details> <summary>Soluzione</summary>

```sh
kubectl logs investigate-app
kubectl exec -it investigate-app -- sh
```

</details>

---

## OBS-6 — Logs di initContainer

- Pod: `init-app`

- Obiettivo:
  - Visualizzare logs initContainer

- Validazione:
  - Output init container

<details> <summary>Soluzione</summary>

```sh
kubectl logs init-app -c <init-container-name>
```

</details>

---

---
