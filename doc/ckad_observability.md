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

Esempio:

```yaml
livenessProbe:
  httpGet:
    path: /
    port: 80
  initialDelaySeconds: 5
  periodSeconds: 10
```

---

## Readiness Probe

- Verifica se il container è pronto a ricevere traffico.
- Se fallisce → il Pod viene rimosso dal Service.
- Non riavvia il container.

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

| Probe | Riavvia Container | Rimuove dal Service | Quando usarla |
|--------|-------------------|---------------------|---------------|
| Liveness | ✅ | ❌ | Crash / deadlock |
| Readiness | ❌ | ✅ | Non pronto al traffico |
| Startup | ✅ | ❌ | Avvio lento |

---

## Logging

Per visualizzare log:

```
kubectl logs <pod>
```

Container specifico:

```
kubectl logs <pod> -c <container>
```

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
7. Verificare consumo CPU/memoria con `kubectl top`.

---



