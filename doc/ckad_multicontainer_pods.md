# CKAD Multi-Container Pods

## Teoria

---

## Multi-Container Pod

Un Pod può contenere più container che:

- Condividono lo stesso IP
- Condividono lo stesso network namespace (localhost)
- Possono condividere volumi
- Vengono schedulati insieme sullo stesso nodo

I container all’interno di un Pod devono collaborare strettamente.

<img width="804" height="502" alt="Immagine 2026-02-24 142447" src="https://github.com/user-attachments/assets/cd2dd5e1-c588-4831-801d-93ae863eca3e" />

---
## Design Patterns


<img width="1002" height="586" alt="Immagine 2026-02-24 142632" src="https://github.com/user-attachments/assets/3602289e-5f11-4b11-8c53-569868f93d5b" />

---

## Co-located Containers

<img width="951" height="521" alt="Immagine 2026-02-24 142844" src="https://github.com/user-attachments/assets/f43f931f-7205-455a-abc2-ddca8d19669d" />


---

## Sidecar Containers

- Container secondario che estende il container principale.
- Esegue in parallelo al container principale.
- Condivide rete e volumi con esso.

### Casi d’uso comuni:
- Logging (es. raccolta log)
- Proxy (es. reverse proxy)
- Monitoraggio
- Sincronizzazione file

Esempio concettuale:
- Container principale → applicazione web
- Sidecar → scrive log su volume condiviso

<img width="943" height="513" alt="Immagine 2026-02-24 143209" src="https://github.com/user-attachments/assets/e167bdd4-feee-4562-a935-00da81562185" />

> Attenzione! restartPolicy è sotto spec allo stesso livello dei container!
---

## Init Containers

- Eseguiti **prima** dei container principali.
- Devono completare con successo prima che i container normali partano.
- Eseguiti in ordine sequenziale.
- Se falliscono → il Pod non parte.

### Casi d’uso:
- Preparare configurazioni
- Attendere che un servizio sia disponibile
- Creare file o directory
- Migrazione database

Struttura YAML base:

```yaml
initContainers:
  - name: init-myservice
    image: busybox
    command: ["sh", "-c", "echo Init completed"]
```
<img width="938" height="513" alt="Immagine 2026-02-24 143056" src="https://github.com/user-attachments/assets/196903e4-f46f-4388-8bd7-9856e2dee511" />

---

## Adapter Pattern

- Trasforma l’output del container principale.
- Rende compatibile l’output con sistemi esterni.
- Spesso usato per:
  - Modificare formato log
  - Tradurre metriche

Il container adapter legge da un volume condiviso e scrive output trasformato.

---

## Ambassador Pattern

- Container proxy che rappresenta un servizio esterno.
- Permette di isolare la logica di connessione.
- Il container principale comunica con `localhost`, l’ambassador inoltra il traffico.

---

## Comunicazione tra Container

Avviene tramite:

- `localhost:<porta>` (stesso network namespace)
- File condivisi tramite volume

Non serve Service per comunicazione interna al Pod.

---

## Differenza Init vs Sidecar

| Init Container | Sidecar Container |
|---------------|-------------------|
| Eseguito prima | Eseguito insieme al principale |
| Deve terminare | Rimane in esecuzione |
| Sequenziale | Parallelo |
| Preparazione ambiente | Estensione funzionalità |

---

## Esercizi

1. Creare un Pod con nginx + sidecar busybox che scrive log su volume condiviso.
2. Aggiungere un initContainer che crea un file prima dell’avvio.
4. Creare un Pod dove un container legge file generati dall’altro.
5. Implementare un Pod con initContainer che attende la disponibilità di un servizio.
6. Simulare un adapter che trasforma file di log.

---




