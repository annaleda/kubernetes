# CKAD Observability

## Teoria

---

## Observability in Kubernetes

Permette di monitorare:

- Stato dei Pod
- Salute dei container
- Log applicativi
- Utilizzo risorse
- Eventi del cluster

Strumenti principali in CKAD:
- Probes
- Logs
- Events
- Metrics (kubectl top)

---

## Probes (Health Checks)

Le probe permettono a Kubernetes di determinare lo stato di un container.

Tipi di probe supportati:
- HTTP
- TCP
- Exec command

---

## Liveness Probe

- Verifica se il container è vivo.
- Se fallisce → Kubernetes riavvia il container.
- Usata per rilevare deadlock o crash interni.

<img width="1193" height="386" alt="image" src="https://github.com/user-attachments/assets/339927db-3e24-4cc3-872e-9f0d7ddd5aec" />

Esempio:

```yaml
livenessProbe:
  httpGet:
    path: /
    port: 80
  initialDelaySeconds: 80
  periodSeconds: 5
  failureThreshold: 3
```

---

## Readiness Probe

- Verifica se il container è pronto a ricevere traffico.
- Se fallisce → il Pod viene rimosso dal Service.
- Non riavvia il container.

<img width="963" height="575" alt="Immagine 2026-02-24 143752" src="https://github.com/user-attachments/assets/78cde84c-d52b-488b-990c-8f6a095b6cf1" />


Utile quando:
- L’app è in fase di inizializzazione
- Dipende da un database esterno

Esempio:

```yaml
readinessProbe:
  httpGet:
    path: /
    port: 80
  initialDelaySeconds: 5
  periodSeconds: 5
```

---

## Startup Probe

- Usata per applicazioni lente all’avvio.
- Disabilita liveness e readiness finché non completa.
- Se fallisce → container riavviato.

Esempio:

```yaml
startupProbe:
  httpGet:
    path: /
    port: 80
  failureThreshold: 30
  periodSeconds: 10
```

---

## Differenza tra le Probe

| Tipo Probe | Riavvia Container | Rimuove dal Service | Uso Tipico             |
| ---------- | ----------------- | ------------------- | ---------------------- |
| Liveness   | ✅                 | ❌                   | Crash, deadlock        |
| Readiness  | ❌                 | ✅                   | Non pronto al traffico |
| Startup    | ✅                 | ❌                   | Avvio lento            |

---

| Metodo    | Descrizione                  | Quando usarlo        |
| --------- | ---------------------------- | -------------------- |
| httpGet   | Effettua richiesta HTTP      | Web app / API        |
| tcpSocket | Verifica apertura porta TCP  | DB / servizi TCP     |
| exec      | Esegue comando nel container | Check personalizzati |

---

| Parametro   | Obbligatorio | Descrizione          | Esempio        |
| ----------- | ------------ | -------------------- | -------------- |
| path        | ✅            | Endpoint da chiamare | `/health`      |
| port        | ✅            | Porta del container  | `8080`         |
| host        | ❌            | Host da contattare   | `localhost`    |
| scheme      | ❌            | HTTP o HTTPS         | `HTTPS`        |
| httpHeaders | ❌            | Header custom        | `name: X-Test` |

---

| Parametro           | Default | Descrizione                         | Effetto Pratico               |
| ------------------- | ------- | ----------------------------------- | ----------------------------- |
| initialDelaySeconds | 0       | Attesa prima del primo check        | Evita crash loop su app lente |
| periodSeconds       | 10      | Ogni quanto esegue il check         | Frequenza controllo           |
| timeoutSeconds      | 1       | Tempo massimo attesa risposta       | Se supera → failure           |
| successThreshold    | 1       | Successi consecutivi richiesti      | Usato spesso per readiness    |
| failureThreshold    | 3       | Fallimenti consecutivi prima azione | Dopo questo interviene K8s    |

Esempio exec:

```yaml
livenessProbe:
  exec:
    command:
      - cat
      - /tmp/healthy
```

Esempio tcpSocket:

```yaml
readinessProbe:
  tcpSocket:
    port: 3306
  initialDelaySeconds: 5
  periodSeconds: 10
```

| Caso                          | Cosa succede                                         |
| ----------------------------- | ---------------------------------------------------- |
| Liveness fallisce             | Container riavviato                                  |
| Readiness fallisce            | Pod = NotReady                                       |
| Startup fallisce              | Container riavviato                                  |
| Readiness + Liveness presenti | Liveness può riavviare, readiness controlla traffico |


---

- Caso reale
  - App con delayed start:

```yaml
startupProbe:
  httpGet:
    path: /
    port: 8080
  failureThreshold: 30
  periodSeconds: 10
```

> Significa:
> 
> 30 × 10 = 300 secondi massimo di startup consentiti.

---
---
---

## Logging

Per visualizzare log:

```
kubectl logs <pod>
```

Entra nel container
```
kubectl exec -it <pod> -- /bin/sh
```

<img width="1107" height="617" alt="Immagine 2026-02-24 144309" src="https://github.com/user-attachments/assets/a1f85930-3208-4c51-9e6a-faf4143ca8eb" />


Container specifico:

```
kubectl logs <pod> -c <container>
```
<img width="1125" height="502" alt="Immagine 2026-02-24 144513" src="https://github.com/user-attachments/assets/7a3234c5-f734-417b-a32e-305ba532362c" />

Seguire log in tempo reale:

```
kubectl logs -f <pod>
```

---

## Events

Permettono di diagnosticare problemi nel cluster.

```
kubectl get events
```

Per dettagli:

```
kubectl describe pod <pod>
```

---

## Metrics (Resource Usage)

Richiede Metrics Server installato.

Visualizzare uso CPU/Memoria:

```
kubectl top pod
kubectl top node
```

---

## Restart Policy

Definisce comportamento al fallimento:

- `Always` (default per Deployment)
- `OnFailure`
- `Never`

---

## CrashLoopBackOff

Stato comune quando:
- Container continua a crashare
- Liveness probe fallisce ripetutamente

Diagnosi:
- Controllare logs
- Controllare events
- Verificare configurazioni

---

## Esercizi

1. Aggiungere una liveness probe HTTP a un Deployment nginx.
2. Simulare un crash del container e osservare il restart.
3. Aggiungere una readiness probe HTTP.
4. Aggiungere una startup probe per simulare avvio lento.
5. Visualizzare logs di un container specifico.
6. Usare `kubectl describe` per diagnosticare un errore.

---



