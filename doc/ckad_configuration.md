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

Permettono di controllare uso CPU e memoria.

### Requests
- Risorse minime garantite
- Usate dallo scheduler per decidere il nodo

### Limits
- Massimo utilizzabile dal container
- Se supera:
  - CPU → throttling
  - Memory → OOMKilled

### Esempio

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

## LimitRange

- Definisce limiti min/max in un namespace
- Può impostare default requests/limits per i container

---

## ResourceQuota

- Limita il consumo totale di risorse in un namespace
- Può limitare:
  - Numero massimo di Pod
  - CPU totale
  - Memoria totale
  - Numero di ConfigMap o Secret

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

