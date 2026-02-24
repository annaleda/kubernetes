# CKAD Configuration

## Teoria

---

## ConfigMap

- Contiene configurazioni **non sensibili**
- Permette di separare configurazione dal codice
- Può essere creata:
  - Da literal (`--from-literal`)
  - Da file (`--from-file`)
  - Da env file (`--from-env-file`)
- Può essere usata:
  - Come variabile d’ambiente
  - Come file montato in un volume
- Se aggiornata, i Pod **non si aggiornano automaticamente** (serve restart o rollout)

| | |
|---|---|
|<img width="100%" alt="Immagine 2026-02-24 124207" src="https://github.com/user-attachments/assets/44d5176e-2c0d-435b-bad8-932ef174ae81" /> |<img width="100%" alt="Immagine 2026-02-24 124305" src="https://github.com/user-attachments/assets/2520bfae-95a9-492c-a333-228c7f2e673c" />|

<img width="1040" height="587" alt="Immagine 2026-02-24 124744" src="https://github.com/user-attachments/assets/4d5ee76f-7407-42b4-aaec-46a197043e8b" />
<img width="1040" height="446" alt="Immagine 2026-02-24 124919" src="https://github.com/user-attachments/assets/bcf22e1d-5580-4b95-837c-5bf020134bb9" />

---

## Secret

- Contiene dati sensibili (password, token, certificati)
- I dati sono codificati in **Base64** (non cifrati realmente)
- Tipi comuni:
  - `Opaque` (default)
  - `kubernetes.io/dockerconfigjson`
  - `kubernetes.io/tls`
- Può essere montato:
  - Come environment variable
  - Come volume
- Buona pratica: evitare segreti hardcoded nei manifest

| | |
|---|---|
|<img width="100%"  alt="Immagine 2026-02-24 130303" src="https://github.com/user-attachments/assets/3572c504-bc45-42cc-9788-2758a162289f" /> |<img width="100%" alt="Immagine 2026-02-24 130303" src="https://github.com/user-attachments/assets/74986ffe-5b11-481b-b828-b560e5ee8bdd" /> |

<img width="1040" height="587" alt="Immagine 2026-02-24 130603" src="https://github.com/user-attachments/assets/3045e84b-ea98-42fb-a37d-993a070d9c83" />
<img width="1040" height="446" alt="Immagine 2026-02-24 130634" src="https://github.com/user-attachments/assets/f8fd589f-9e1c-46f1-9a93-7d54a9b229ca" />


---

## Environment Variables

### Definizione diretta

```yaml
env:
  - name: APP_ENV
    value: "production"
```

### Da ConfigMap

```yaml
envFrom:
  - configMapRef:
      name: my-config
```

### Da Secret

```yaml
env:
  - name: DB_PASSWORD
    valueFrom:
      secretKeyRef:
        name: my-secret
        key: password
```

---

## ConfigMap / Secret come Volume

### Definizione volume

```yaml
volumes:
  - name: config-volume
    configMap:
      name: my-config
```

### Mount nel container

```yaml
volumeMounts:
  - name: config-volume
    mountPath: /etc/config
```

Ogni chiave diventa un file nella directory montata.

---
## Resource Requests & Limits

Permettono di controllare l’uso di **CPU** e **Memoria** nei container.

Sono fondamentali per:

- Scheduling corretto
- Stabilità del cluster
- Controllo delle risorse
- Classificazione QoS

---

### Requests

Le **requests** sono le risorse minime garantite al container.

- Usate dallo scheduler per decidere su quale nodo posizionare il Pod
- Il nodo deve avere almeno quelle risorse disponibili

Esempio:

```yaml
requests:
  cpu: "100m"
  memory: "128Mi"
```

### Limits
Massimo utilizzabile dal container
Se il container supera il limite:
  - CPU → viene applicato throttling
  - Memory → il container viene terminato (OOMKilled)

```yaml
resources:
  requests:
    cpu: "100m"
    memory: "128Mi"
  limits:
    cpu: "500m"
    memory: "256Mi"
```

---

### QoS Classes (Quality of Service)

Kubernetes assegna automaticamente una classe QoS al Pod in base a requests e limits.

Esistono 3 classi:
- Guaranteed (massima priorità)

  - Condizioni:
    
    - Requests = Limits
    - Per CPU e memoria
    - Per tutti i container del Pod
    - Più protetto in caso di pressione memoria
    - Ultimo ad essere terminato
      
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: guaranteed-pod
    spec:
      containers:
      - name: my-container
        image: nginx
        resources:
          requests:
            memory: "256Mi"
            cpu: "100m"
          limits:
            memory: "256Mi"
            cpu: "100m"
    ```

- Burstable

  - Condizioni:

    - Ha almeno una request
    - Requests ≠ Limits (oppure limits mancanti)
    - Può usare più risorse
    - Può essere terminato prima dei Guaranteed

    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: burstable-pod
    spec:
      containers:
      - name: my-container
        image: nginx
        resources:
          requests:
            memory: "64Mi"
            cpu: "50m"
      ```


- BestEffort (priorità più bassa)

  - Condizione:
    
    - Nessuna request
    - Nessun limit
    - Primo ad essere terminato in caso di pressione memoria
      
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: besteffort-pod
    spec:
      containers:
      - name: my-container
        image: nginx
    ```

## LimitRange

- Definisce limiti min/max in un namespace
- Può impostare default requests/limits per i container

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: container-limits
  namespace: demo
spec:
  limits:
  - type: Container
    default:
      cpu: "500m"
      memory: "256Mi"
    defaultRequest:
      cpu: "100m"
      memory: "128Mi"
    min:
      cpu: "50m"
      memory: "64Mi"
    max:
      cpu: "1"
      memory: "512Mi"
```
---

## ResourceQuota

- Limita il consumo totale di risorse in un namespace
- Può limitare:
  - Numero massimo di Pod
  - CPU totale
  - Memoria totale
  - Numero di ConfigMap o Secret
> “Questo namespace può usare massimo 4 CPU e 8Gi di RAM”

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: namespace-quota
  namespace: demo
spec:
  hard:
    requests.cpu: "4"
    requests.memory: 8Gi
    limits.cpu: "6"
    limits.memory: 12Gi
    pods: "10"
    configmaps: "5"
    persistentvolumeclaims: "4"
```

---

| Concetto       | Serve a                          |
|---------------|-----------------------------------|
| Requests      | Scheduling                        |
| Limits        | Controllo massimo utilizzo        |
| QoS           | Priorità in caso di pressione     |
| LimitRange    | Regole per singolo container      |
| ResourceQuota | Limite globale del namespace      |

## Unità di misura (CPU e Memoria)

Kubernetes usa unità specifiche per CPU e memoria.

---

### CPU

La CPU si misura in **core** o **millicore (m)**.

| Valore | Significato |
|--------|------------|
| `1`    | 1 core CPU |
| `500m` | 0.5 core (mezzo core) |
| `100m` | 0.1 core |
| `250m` | 0.25 core |

`1000m = 1 core`

La CPU è **compressibile** → se supera il limit viene applicato throttling.

---

### Memoria

La memoria si misura in byte.

| Valore | Significato |
|--------|------------|
| `128Mi` | 128 Mebibyte |
| `1Gi`   | 1 Gibibyte |
| `512Mi` | 0.5 Gi |
| `100M`  | 100 Megabyte (decimale) |

---

### Differenza tra Mi e M

| Unità | Tipo | Base |
|-------|------|------|
| `Mi`, `Gi` | Binaria | 1024 |
| `M`, `G`   | Decimale | 1000 |

Esempio:

- `1Gi = 1024Mi`
- `1G = 1000M`

⚠ In Kubernetes si usa quasi sempre `Mi` e `Gi`.

---

## Da ricordare per CKAD

- CPU → `m` = millicore
- 1000m = 1 core
- Memoria → meglio usare `Mi` e `Gi`
- CPU supera limit → throttling
- Memoria supera limit → OOMKilled

---

## Image Configuration

### imagePullPolicy
- `Always`
- `IfNotPresent`
- `Never`

### imagePullSecrets

Usato per autenticazione a registry privati:

```yaml
imagePullSecrets:
  - name: my-docker-secret
```

Per immagini private:

```
kubectl create secret docker-registry my-secret \
  --docker-username=user \
  --docker-password=pass \
  --docker-email=email@example.com
```

Nel Pod:

```yaml
imagePullSecrets:
- name: my-secret
```

---
---

## Esercizi

1. Creare una ConfigMap da CLI.
2. Montare una ConfigMap come volume.
3. Creare un Secret e usarlo come variabile d'ambiente.
4. Definire requests e limits su un pod.
5. Creare una ResourceQuota che limiti a 5 pod in un namespace.
6. Creare un LimitRange con default CPU/memory.
7. Configurare un container che gira come non-root.
8. Usare un imagePullSecret per accedere a un registry privato.

---

