
### Storage 
---

## ST-1 — Static PV + PVC

- PersistentVolume
  - Nome: `pv-static`
  - Storage: `100Mi`
  - AccessMode: ReadWriteOnce
  - hostPath: `/mnt/static`

- PersistentVolumeClaim
  - Nome: `pvc-static`
  - Richiede: `100Mi`

- Pod
  - Nome: `static-consumer`
  - Image: nginx
  - Monta PVC in /data

- Validazione
  - PVC Bound
  - Pod Running

```

```
---

## ST-2 — Multiple PVC in Pod

- PVC
  - pvc-config
  - pvc-data

- Pod: `multi-volume-app`
  - Monta pvc-config in /config
  - Monta pvc-data in /data

- Validazione
  - Entrambi i volumi montati correttamente

```

```
---

## ST-3 — ReadWriteMany Scenario

- PersistentVolume
  - Storage: `200Mi`
  - AccessMode: ReadWriteMany
  - hostPath: `/mnt/shared`

- PVC collegato al PV
  - Pod 1: shared-1
  - Pod 2: shared-2

Entrambi montano stesso PVC

- Validazione
  - Entrambi Running
  - File scritto da uno visibile nell’altro

```

```
---

## ST-4 — SubPath Mount

- Pod: `subpath-app`

- PVC: `pvc-static`

- Configurazione
  - Montare solo subPath logs in /var/log/app

- Validazione
  - Solo la directory logs è montata

```

```
---

## ST-5 — StorageClass Usage

- PVC: `dynamic-storage`
- Storage: `150Mi`
- StorageClass: standard
- Pod: dynamic-consumer
  - Image: busybox
  - Monta volume in `/data`
  - Scrive file `test.txt`

-Validazione
- PVC Bound automaticamente

```

```
---

## ST-6 — VolumeMount ReadOnly

- Pod: `readonly-app`

- PVC: `pvc-static`

- Configurazione
  - volumeMount readOnly: true

- Validazione
  - Tentativo di scrittura fallisce

```

```
---
