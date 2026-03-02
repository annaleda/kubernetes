
### Multi-Container (6 esercizi)
## MC-1 — Sidecar Logging Pattern

  - Pod: `order-app`
  - Container 1 – app
    - Image: nginx
    - Porta: 80
    - Scrive ogni 5s in `/var/log/app.log`

  - Container 2 – logger
    - Image: busybox
    - Legge continuamente `/var/log/app.log`

- Vincoli
  - Volume: `emptyDir`
  - Nome volume: shared-logs

- Validazione
  - Entrambi Running
  - Logs del container logger mostrano output


```  
``` 
---

## MC-2 — Ambassador Pattern

- Pod: `external-proxy`
- Container 1
  - Image: httpd
  - Porta: 80

- Container 2
  - Image: alpine
  - Esegue curl continuo verso `http://localhost:80`

- Vincoli
  - Nessun Service
  - Comunicazione via localhost

- Validazione
  - Logs mostrano risposta HTTP

``` 
```
---

## MC-3 — Adapter Pattern

- Pod: `metrics-adapter`

- Container 1
  Image: busybox
  Genera file `/data/raw.txt`

- Container 2
  Image: busybox
  Trasforma contenuto in uppercase e salva in `/data/processed.txt`

- Vincoli
  Volume shared `emptyDir`
  Nome volume: data-volume

- Validazione
File `processed.txt` esiste

```  
```
---

### MC-4 — Init + App

- Pod: `init-config-app`

- Init Container
  - Image: busybox
  - Scrive file `/config/settings.conf`

- Main Container
  - Image: nginx
  - Monta stesso volume
  - Deve partire solo dopo init

- Vincoli
  - Volume: `emptyDir`
  - Nome: config-data

```  
```
---

### MC-5 — Multi-Container con variabili condivise

- Pod: `env-sharing-app`

- Container 1
  - Image: busybox
  - Env: `MODE=production`

- Container 2
  - Image: busybox
  - Deve stampare valore MODE letto da file condiviso

- Vincoli
  - Shared volume
  - File generato da primo container

```  
```
---

### MC-6 — Multi-Container con resource limits diversi

- Pod: `resource-multi`

- Container 1
  - Image: nginx
  - CPU limit: `200m`

- Container 2
  - Image: busybox
  - Memory limit: `128Mi`

- Validazione
  - kubectl describe pod mostra limiti distinti

```  
```
---
