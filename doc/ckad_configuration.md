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

## Security Context (Base)

Permette di controllare privilegi del container.

Esempi di opzioni:
- `runAsUser`
- `runAsNonRoot`
- `readOnlyRootFilesystem`
- `allowPrivilegeEscalation`

### Esempio

```yaml
securityContext:
  runAsNonRoot: true
  runAsUser: 1000
```

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

