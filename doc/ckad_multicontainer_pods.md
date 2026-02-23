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

---

## Sidecar Pattern

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

## Condivisione Storage

I container possono condividere volumi definiti nel Pod.

### emptyDir

- Creato quando il Pod viene avviato.
- Eliminato quando il Pod viene distrutto.
- Usato per scambio dati tra container.

Esempio:

```yaml
volumes:
  - name: shared-data
    emptyDir: {}
```

Montaggio nei container:

```yaml
volumeMounts:
  - name: shared-data
    mountPath: /data
```

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
3. Condividere un volume emptyDir tra container.
4. Creare un Pod dove un container legge file generati dall’altro.
5. Implementare un Pod con initContainer che attende la disponibilità di un servizio.
6. Simulare un adapter che trasforma file di log.

---




