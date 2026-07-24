
# Kubernetes Probes - CKAD Notes

> Obiettivo CKAD: capire **quando usare una probe, come interpretarne il comportamento e come risolvere problemi di Pod non Ready o in CrashLoopBackOff.**

---

# 1. Perché esistono le Probe?

Kubernetes non sa se un'applicazione funziona davvero.

Vede solo che il container è in esecuzione.

Le probe permettono di verificare lo stato reale dell'applicazione.

---

# 2. Le tre Probe

```text
startupProbe

↓

livenessProbe

↓

readinessProbe
```

Hanno scopi completamente diversi.

---

# 3. Startup Probe

Serve per applicazioni lente ad avviarsi.

Finché la Startup Probe non ha successo:

* la Liveness Probe è disabilitata
* la Readiness Probe è disabilitata

Flusso:

```text
Container

↓

Startup Probe

↓

Success

↓

Liveness + Readiness iniziano
```

---

## Quando usarla

Applicazioni che impiegano molto tempo ad avviarsi.

Ad esempio:

* Spring Boot
* JVM
* Database
* Elasticsearch

---

## Esempio

```yaml
startupProbe:

  httpGet:
    path: /health
    port: 8080

  periodSeconds: 5

  failureThreshold: 30
```

L'app ha:

```text
30 × 5 = 150 secondi
```

per partire.

---

# 4. Liveness Probe

Domanda che Kubernetes si pone:

```text
"L'applicazione è bloccata?"
```

Se fallisce:

↓

Il container viene riavviato.

---

Flusso

```text
Container

↓

Liveness

↓

Failed

↓

Restart
```

---

Usarla quando:

* deadlock
* processi bloccati
* applicazione non risponde più

---

Mai usarla per verificare se l'app è pronta a ricevere traffico.

---

# 5. Readiness Probe

Domanda:

```text
"L'app può ricevere traffico?"
```

Se fallisce

↓

Il Pod NON viene riavviato.

↓

Viene semplicemente rimosso dagli Endpoint del Service.

---

Flusso

```text
Readiness Failed

↓

Pod Running

↓

Not Ready

↓

Service non invia traffico
```

---

Questa è la differenza fondamentale.

---

# 6. Differenza

## Startup

```text
Serve per partire.
```

---

## Liveness

```text
Serve a capire se il container deve essere riavviato.
```

---

## Readiness

```text
Serve a decidere se il Pod riceve traffico.
```

---

# 7. Tipi di Probe

Tutte e tre possono usare:

---

## HTTP

```yaml
httpGet:

  path: /health

  port: 8080
```

---

## TCP

```yaml
tcpSocket:

  port: 8080
```

Verifica solo che la porta sia aperta.

---

## Exec

```yaml
exec:

  command:

  - cat

  - /tmp/healthy
```

Molto frequente nel CKAD.

---

# 8. Parametri importanti

## initialDelaySeconds

Quanto aspettare prima del primo controllo.

---

## periodSeconds

Ogni quanto controllare.

---

## timeoutSeconds

Tempo massimo di risposta.

---

## failureThreshold

Numero di errori consecutivi prima del fallimento.

---

## successThreshold

Quanti successi servono.

Usato quasi sempre solo con Readiness.

---

# 9. Esercizi CKAD

## Caso 1

Il Pod è

```text
Running

0/1 Ready
```

Domanda mentale:

```text
Readiness?
```

Controllare

```bash
kubectl describe pod
```

Poi Events.

---

## Caso 2

Il Pod

```text
CrashLoopBackOff
```

Domanda

```text
Liveness?
```

Controllare

```bash
kubectl describe pod
```

Eventi

```text
Liveness probe failed
```

↓

Il container continua a riavviarsi.

---

## Caso 3

L'app impiega 90 secondi per partire.

Hai:

```yaml
livenessProbe:

  periodSeconds: 5

  failureThreshold: 3
```

↓

Dopo

```text
15 secondi
```

viene riavviata.

Non partirà mai.

Soluzione

↓

Aggiungere una

```text
startupProbe
```

oppure aumentare i tempi.

---

# 10. Troubleshooting

Sempre nello stesso ordine.

```text
Pod

↓

Describe

↓

Events

↓

Probe

↓

Logs
```

---

Comandi

```bash
kubectl describe pod

kubectl logs

kubectl get events
```

---

# 11. Errori tipici

## HTTP path errato

Applicazione

```text
/health
```

Probe

```text
/healthz
```

↓

404

↓

Probe Failed

---

## Porta sbagliata

Applicazione

```text
8080
```

Probe

```text
80
```

↓

Connection refused

---

## Exec errata

```yaml
command:

- cat

- /tmp/healthy
```

Il file non esiste.

↓

Probe fallisce.

---

## Timeout troppo basso

L'app risponde dopo

```text
3 secondi
```

timeout

```yaml
timeoutSeconds: 1
```

↓

Failure.

---

# 12. Come ragionare durante il CKAD

Se il Pod è

```text
Running

Ready
```

↓

Le probe funzionano.

---

Se è

```text
Running

Not Ready
```

↓

Pensare immediatamente:

```text
Readiness Probe
```

---

Se è

```text
CrashLoopBackOff
```

↓

Pensare subito:

```text
Liveness Probe
```

---

Se non riesce mai a partire

↓

Applicazione lenta.

Pensare:

```text
Startup Probe
```

---

# 13. Checklist CKAD

Quando leggi:

> Application is not receiving traffic.

Controlla:

```text
Readiness
```

---

Quando leggi:

> Pod restarts continuously.

Controlla:

```text
Liveness
```

---

Quando leggi:

> Application needs several minutes to start.

Controlla:

```text
Startup
```

---

# 14. Flow mentale da esame

```text
Pod

↓

STATUS

↓

Running?

↓

Ready?

↓

Describe

↓

Events

↓

Logs

↓

Probe

↓

Correggere path / porta / comando / timing
```

---

# Regole

✅ Startup blocca temporaneamente Liveness e Readiness.

✅ Liveness fallita → il container viene riavviato.

✅ Readiness fallita → il Pod resta in esecuzione ma non riceve traffico.

✅ Startup serve solo all'avvio.

✅ Readiness non riavvia mai il Pod.

✅ Liveness non decide se il Service invia traffico.

✅ Il primo comando nel troubleshooting è quasi sempre:

```bash
kubectl describe pod
```

e poi:

```bash
kubectl logs
```
## Relazione tra Probe, EndpointSlice e Service

Le probe non hanno tutte lo stesso effetto sul traffico instradato dai Service.

### Readiness Probe

Quando una **Readiness Probe** fallisce:

```text
Readiness Failed
        │
        ▼
Pod = NotReady
        │
        ▼
EndpointSlice aggiornato
        │
        ▼
L'IP del Pod viene rimosso dagli Endpoint
        │
        ▼
Il Service non invia più traffico al Pod
```

Il container **continua a essere in esecuzione**, ma il Pod viene escluso dal bilanciamento del Service finché non torna nello stato `Ready`. Kubernetes aggiorna automaticamente gli `EndpointSlice` rimuovendo il Pod dai Service che lo selezionano. :contentReference[oaicite:0]{index=0}

### Liveness Probe

Quando una **Liveness Probe** fallisce:

```text
Liveness Failed
        │
        ▼
Kubelet riavvia il container
        │
        ▼
Il Pod diventa temporaneamente NotReady
        │
        ▼
EndpointSlice aggiornato
        │
        ▼
L'IP viene temporaneamente rimosso dagli Endpoint
        │
        ▼
Quando il Pod torna Ready,
l'IP viene reinserito
```

La **Liveness Probe non rimuove direttamente l'IP dagli Endpoint**. Il suo compito è riavviare il container. Durante il riavvio il Pod non è `Ready` e Kubernetes aggiorna gli `EndpointSlice`, escludendo temporaneamente il Pod dal traffico del Service. :contentReference[oaicite:1]{index=1}

### Regole da ricordare

- **Readiness Probe** → determina se un Pod può ricevere traffico da un Service.
- **Liveness Probe** → determina se il container deve essere riavviato.
- Un Pod `NotReady` viene escluso dagli `EndpointSlice` e quindi non riceve traffico dal Service.
- Quando il Pod torna `Ready`, Kubernetes lo reinserisce automaticamente negli `EndpointSlice`. :contentReference[oaicite:2]{index=2}
