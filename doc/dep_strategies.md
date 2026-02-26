## Deployment Strategies

<img width="899" height="346" alt="Immagine 2026-02-26 125603" src="https://github.com/user-attachments/assets/fee71a2b-40b5-430e-be51-f20c8b264d5d" />

- Recreate, Rollig Update and Rollback → []()
- Scaling and Descaling                → []()
- Blue/Green
- Canary
---
### Blue/Green

<img width="1343" height="484" alt="Immagine 2026-02-26 130401" src="https://github.com/user-attachments/assets/94c21a5c-3dec-4359-a724-a345dd6fbdbb" />

Hai due ambienti identici:

- Blue → versione attuale (1.0)
- Green → nuova versione (1.1)

Il traffico va solo a uno dei due.

Quando vuoi fare switch:
> cambi il Service.

```bash
Deployment v1.0
metadata:
  name: nginx-deployment-1.0
labels:
  app: nginx-1.0
```
```bash
Deployment v1.1
metadata:
  name: nginx-deployment-1.1
labels:
  app: nginx-1.1
```
```bash
Service
selector:
  app: nginx-1.0
```

> Attualmente il Service manda traffico SOLO alla 1.0.

Per fare lo switch Blue → Green modifichi il selector del Service:

```bash
selector:
  app: nginx-1.1
```

Applichi:

```bash
kubectl apply -f service.yaml
```
---
### Canary

<img width="648" height="198" alt="Immagine 2026-02-26 125012" src="https://github.com/user-attachments/assets/bd0865a9-370e-4dd6-ad59-c6b79372f513" />
