# Kubernetes Projected Volumes — esercizi con ambiente di preparazione
---

## PROJ-1 — Projected volume con ConfigMap

### Preparazione

```bash
kubectl delete namespace projected-test --ignore-not-found
kubectl create namespace projected-test

kubectl create configmap app-config \
  --from-literal=environment=production \
  --from-literal=log_level=info \
  -n projected-test
```

### Task

Crea il Pod `config-pod` nel namespace `projected-test` con:

- immagine `busybox`;
- comando `sleep 3600`;
- volume `app-data` di tipo `projected`;
- sorgente ConfigMap `app-config`;
- mount path `/projected`;
- sola lettura.

### Validazione

```bash
kubectl exec config-pod -n projected-test -- ls -l /projected
kubectl exec config-pod -n projected-test -- cat /projected/environment
kubectl exec config-pod -n projected-test -- cat /projected/log_level
```

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: config-pod
  namespace: projected-test
spec:
  containers:
  - name: app
    image: busybox
    command: ["sh", "-c", "sleep 3600"]
    volumeMounts:
    - name: app-data
      mountPath: /projected
      readOnly: true
  volumes:
  - name: app-data
    projected:
      sources:
      - configMap:
          name: app-config
```

```bash
kubectl apply -f config-pod.yaml
```

</details>

### Cleanup

```bash
kubectl delete namespace projected-test
rm -f config-pod.yaml
```

---

## PROJ-2 — Projected volume con Secret

### Preparazione

```bash
kubectl delete namespace projected-test --ignore-not-found
kubectl create namespace projected-test

kubectl create secret generic db-credentials \
  --from-literal=username=admin \
  --from-literal=password='s3cr3t' \
  -n projected-test
```

### Task

Crea il Pod `secret-pod` con:

- immagine `busybox`;
- comando `sleep 3600`;
- Secret `db-credentials` montato tramite volume `projected`;
- directory `/credentials`;
- sola lettura.

### Validazione

```bash
kubectl exec secret-pod -n projected-test -- ls -l /credentials
kubectl exec secret-pod -n projected-test -- cat /credentials/username
kubectl exec secret-pod -n projected-test -- cat /credentials/password
```

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-pod
  namespace: projected-test
spec:
  containers:
  - name: app
    image: busybox
    command: ["sh", "-c", "sleep 3600"]
    volumeMounts:
    - name: credentials
      mountPath: /credentials
      readOnly: true
  volumes:
  - name: credentials
    projected:
      sources:
      - secret:
          name: db-credentials
```

</details>

### Cleanup

```bash
kubectl delete namespace projected-test
rm -f secret-pod.yaml
```

---

## PROJ-3 — ConfigMap e Secret nella stessa directory

### Preparazione

```bash
kubectl delete namespace projected-test --ignore-not-found
kubectl create namespace projected-test

kubectl create configmap web-config \
  --from-literal=app.properties='port=8080' \
  -n projected-test

kubectl create secret generic web-secret \
  --from-literal=api-key='abc123' \
  -n projected-test
```

### Task

Crea il Pod `combined-pod` e proietta nello stesso volume:

- ConfigMap `web-config`;
- Secret `web-secret`.

Monta il volume in `/all-in-one`.

### Validazione

```bash
kubectl exec combined-pod -n projected-test -- find /all-in-one -maxdepth 1 -type f -print
kubectl exec combined-pod -n projected-test -- cat /all-in-one/app.properties
kubectl exec combined-pod -n projected-test -- cat /all-in-one/api-key
```

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: combined-pod
  namespace: projected-test
spec:
  containers:
  - name: app
    image: busybox
    command: ["sh", "-c", "sleep 3600"]
    volumeMounts:
    - name: all-in-one
      mountPath: /all-in-one
      readOnly: true
  volumes:
  - name: all-in-one
    projected:
      sources:
      - configMap:
          name: web-config
      - secret:
          name: web-secret
```

</details>

### Cleanup

```bash
kubectl delete namespace projected-test
rm -f combined-pod.yaml
```

---

## PROJ-4 — Percorsi personalizzati con `items`

### Preparazione

```bash
kubectl delete namespace projected-test --ignore-not-found
kubectl create namespace projected-test

kubectl create configmap app-config \
  --from-literal=environment=staging \
  --from-literal=timeout=30 \
  -n projected-test

kubectl create secret generic app-secret \
  --from-literal=username=developer \
  --from-literal=password=change-me \
  -n projected-test
```

### Task

Crea il Pod `paths-pod` e proietta:

- `environment` dalla ConfigMap nel file `config/env`;
- `timeout` dalla ConfigMap nel file `config/timeout`;
- `username` dal Secret nel file `credentials/user`;
- `password` dal Secret nel file `credentials/pass`.

Monta tutto in `/projected`.

### Validazione

```bash
kubectl exec paths-pod -n projected-test -- find /projected -type f -print
kubectl exec paths-pod -n projected-test -- cat /projected/config/env
kubectl exec paths-pod -n projected-test -- cat /projected/credentials/user
```

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: paths-pod
  namespace: projected-test
spec:
  containers:
  - name: app
    image: busybox
    command: ["sh", "-c", "sleep 3600"]
    volumeMounts:
    - name: projected-data
      mountPath: /projected
      readOnly: true
  volumes:
  - name: projected-data
    projected:
      sources:
      - configMap:
          name: app-config
          items:
          - key: environment
            path: config/env
          - key: timeout
            path: config/timeout
      - secret:
          name: app-secret
          items:
          - key: username
            path: credentials/user
          - key: password
            path: credentials/pass
```

</details>

### Cleanup

```bash
kubectl delete namespace projected-test
rm -f paths-pod.yaml
```

---

## PROJ-5 — Downward API in un projected volume

### Preparazione

```bash
kubectl delete namespace projected-test --ignore-not-found
kubectl create namespace projected-test
```

### Task

Crea il Pod `downward-pod` con:

- label `app=demo`;
- label `environment=test`;
- annotazione `owner=platform-team`;
- immagine `busybox`;
- richiesta CPU `100m`;
- limite CPU `250m`.

Proietta nella directory `/pod-info`:

- nome del Pod;
- namespace;
- UID;
- tutte le label;
- tutte le annotazioni;
- limite CPU del container.

### Validazione

```bash
kubectl exec downward-pod -n projected-test -- find /pod-info -type f -maxdepth 1 -print
kubectl exec downward-pod -n projected-test -- cat /pod-info/name
kubectl exec downward-pod -n projected-test -- cat /pod-info/namespace
kubectl exec downward-pod -n projected-test -- cat /pod-info/labels
kubectl exec downward-pod -n projected-test -- cat /pod-info/cpu_limit
```

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: downward-pod
  namespace: projected-test
  labels:
    app: demo
    environment: test
  annotations:
    owner: platform-team
spec:
  containers:
  - name: app
    image: busybox
    command: ["sh", "-c", "sleep 3600"]
    resources:
      requests:
        cpu: 100m
      limits:
        cpu: 250m
    volumeMounts:
    - name: pod-info
      mountPath: /pod-info
      readOnly: true
  volumes:
  - name: pod-info
    projected:
      sources:
      - downwardAPI:
          items:
          - path: name
            fieldRef:
              fieldPath: metadata.name
          - path: namespace
            fieldRef:
              fieldPath: metadata.namespace
          - path: uid
            fieldRef:
              fieldPath: metadata.uid
          - path: labels
            fieldRef:
              fieldPath: metadata.labels
          - path: annotations
            fieldRef:
              fieldPath: metadata.annotations
          - path: cpu_limit
            resourceFieldRef:
              containerName: app
              resource: limits.cpu
```

</details>

### Cleanup

```bash
kubectl delete namespace projected-test
rm -f downward-pod.yaml
```

---

## PROJ-6 — ServiceAccountToken personalizzato

### Preparazione

```bash
kubectl delete namespace projected-test --ignore-not-found
kubectl create namespace projected-test
kubectl create serviceaccount api-client -n projected-test
```

### Task

Crea il Pod `token-pod` con:

- ServiceAccount `api-client`;
- `automountServiceAccountToken: false`;
- volume projected contenente un `serviceAccountToken`;
- audience `api`;
- durata richiesta `3600` secondi;
- file `token`;
- mount path `/var/run/custom-token`.

### Validazione

```bash
kubectl get pod token-pod -n projected-test \
  -o jsonpath='{.spec.automountServiceAccountToken}{"\n"}'

kubectl exec token-pod -n projected-test -- \
  ls -l /var/run/custom-token

kubectl exec token-pod -n projected-test -- \
  sh -c 'test -s /var/run/custom-token/token && echo token-present'
```

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: token-pod
  namespace: projected-test
spec:
  serviceAccountName: api-client
  automountServiceAccountToken: false
  containers:
  - name: app
    image: busybox
    command: ["sh", "-c", "sleep 3600"]
    volumeMounts:
    - name: custom-token
      mountPath: /var/run/custom-token
      readOnly: true
  volumes:
  - name: custom-token
    projected:
      sources:
      - serviceAccountToken:
          audience: api
          expirationSeconds: 3600
          path: token
```

</details>

### Cleanup

```bash
kubectl delete namespace projected-test
rm -f token-pod.yaml
```

---

## PROJ-7 — Volume completo: ConfigMap, Secret, Downward API e token

### Preparazione

```bash
kubectl delete namespace projected-test --ignore-not-found
kubectl create namespace projected-test

kubectl create serviceaccount full-sa -n projected-test

kubectl create configmap full-config \
  --from-literal=application.yaml='server.port: 8080' \
  -n projected-test

kubectl create secret generic full-secret \
  --from-literal=database-password='db-pass-123' \
  -n projected-test
```

### Task

Crea il Pod `full-projection-pod` con:

- ServiceAccount `full-sa`;
- automount automatico del token disabilitato;
- label `tier=backend`;
- annotazione `team=payments`;
- mount path `/projected`;
- un solo volume projected contenente:
  - `application.yaml` dalla ConfigMap;
  - `database-password` dal Secret, salvata come `secrets/db-password`;
  - nome del Pod, salvato come `metadata/pod-name`;
  - namespace, salvato come `metadata/namespace`;
  - token della ServiceAccount, salvato come `tokens/api-token`;
- audience `kubernetes`;
- durata token `1800` secondi.

### Validazione

```bash
kubectl exec full-projection-pod -n projected-test -- \
  find /projected -type f -print

kubectl exec full-projection-pod -n projected-test -- \
  cat /projected/application.yaml

kubectl exec full-projection-pod -n projected-test -- \
  cat /projected/metadata/pod-name

kubectl exec full-projection-pod -n projected-test -- \
  sh -c 'test -s /projected/tokens/api-token && echo token-present'
```

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: full-projection-pod
  namespace: projected-test
  labels:
    tier: backend
  annotations:
    team: payments
spec:
  serviceAccountName: full-sa
  automountServiceAccountToken: false
  containers:
  - name: app
    image: busybox
    command: ["sh", "-c", "sleep 3600"]
    volumeMounts:
    - name: full-projection
      mountPath: /projected
      readOnly: true
  volumes:
  - name: full-projection
    projected:
      defaultMode: 0440
      sources:
      - configMap:
          name: full-config
          items:
          - key: application.yaml
            path: application.yaml
      - secret:
          name: full-secret
          items:
          - key: database-password
            path: secrets/db-password
      - downwardAPI:
          items:
          - path: metadata/pod-name
            fieldRef:
              fieldPath: metadata.name
          - path: metadata/namespace
            fieldRef:
              fieldPath: metadata.namespace
      - serviceAccountToken:
          audience: kubernetes
          expirationSeconds: 1800
          path: tokens/api-token
```

</details>

### Cleanup

```bash
kubectl delete namespace projected-test
rm -f full-projection-pod.yaml
```

---

## PROJ-8 — Permessi con `defaultMode` e `mode`

### Preparazione

```bash
kubectl delete namespace projected-test --ignore-not-found
kubectl create namespace projected-test

kubectl create configmap scripts \
  --from-literal=start.sh='#!/bin/sh
echo application-started' \
  -n projected-test

kubectl create secret generic credentials \
  --from-literal=password='top-secret' \
  -n projected-test
```

### Task

Crea il Pod `mode-pod` con un volume projected montato in `/data`.

Requisiti:

- `defaultMode: 0440`;
- `start.sh` dalla ConfigMap deve avere `mode: 0755`;
- `password` dal Secret deve usare il mode predefinito;
- salva il Secret come `private/password`.

### Validazione

```bash
kubectl exec mode-pod -n projected-test -- ls -l /data
kubectl exec mode-pod -n projected-test -- ls -l /data/private
kubectl exec mode-pod -n projected-test -- /data/start.sh
```

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mode-pod
  namespace: projected-test
spec:
  containers:
  - name: app
    image: busybox
    command: ["sh", "-c", "sleep 3600"]
    volumeMounts:
    - name: projected-data
      mountPath: /data
      readOnly: true
  volumes:
  - name: projected-data
    projected:
      defaultMode: 0440
      sources:
      - configMap:
          name: scripts
          items:
          - key: start.sh
            path: start.sh
            mode: 0755
      - secret:
          name: credentials
          items:
          - key: password
            path: private/password
```

</details>

### Cleanup

```bash
kubectl delete namespace projected-test
rm -f mode-pod.yaml
```

---

## PROJ-9 — Sorgenti opzionali

### Preparazione

```bash
kubectl delete namespace projected-test --ignore-not-found
kubectl create namespace projected-test
```

Non creare alcuna ConfigMap o Secret.

### Task

Crea il Pod `optional-pod` con un volume projected contenente:

- ConfigMap inesistente `optional-config`;
- Secret inesistente `optional-secret`;
- entrambe le sorgenti devono essere opzionali;
- mount path `/optional`.

Il Pod deve poter partire anche se le sorgenti non esistono.

### Validazione

```bash
kubectl get pod optional-pod -n projected-test
kubectl exec optional-pod -n projected-test -- ls -la /optional
```

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: optional-pod
  namespace: projected-test
spec:
  containers:
  - name: app
    image: busybox
    command: ["sh", "-c", "sleep 3600"]
    volumeMounts:
    - name: optional-data
      mountPath: /optional
      readOnly: true
  volumes:
  - name: optional-data
    projected:
      sources:
      - configMap:
          name: optional-config
          optional: true
      - secret:
          name: optional-secret
          optional: true
```

</details>

### Cleanup

```bash
kubectl delete namespace projected-test
rm -f optional-pod.yaml
```

---

## PROJ-10 — Debug: sorgente obbligatoria mancante

### Preparazione

```bash
kubectl delete namespace projected-test --ignore-not-found
kubectl create namespace projected-test

cat > broken-projected-pod.yaml <<'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: broken-projected-pod
  namespace: projected-test
spec:
  containers:
  - name: app
    image: busybox
    command: ["sh", "-c", "sleep 3600"]
    volumeMounts:
    - name: broken-data
      mountPath: /data
      readOnly: true
  volumes:
  - name: broken-data
    projected:
      sources:
      - configMap:
          name: missing-config
EOF

kubectl apply -f broken-projected-pod.yaml
```

### Task

Il Pod non parte.

- individua la causa;
- risolvi il problema senza modificare il manifest del Pod;
- la ConfigMap deve contenere `status=ready`.

### Validazione

```bash
kubectl get pod broken-projected-pod -n projected-test
kubectl exec broken-projected-pod -n projected-test -- cat /data/status
```

<details>
<summary>Soluzione</summary>

```bash
kubectl describe pod broken-projected-pod -n projected-test
kubectl get events -n projected-test --sort-by=.metadata.creationTimestamp

kubectl create configmap missing-config \
  --from-literal=status=ready \
  -n projected-test
```

</details>

### Cleanup

```bash
kubectl delete namespace projected-test
rm -f broken-projected-pod.yaml
```

---

## PROJ-11 — Aggiornamento automatico di ConfigMap e Secret

### Preparazione

```bash
kubectl delete namespace projected-test --ignore-not-found
kubectl create namespace projected-test

kubectl create configmap dynamic-config \
  --from-literal=version=v1 \
  -n projected-test

kubectl create secret generic dynamic-secret \
  --from-literal=password=old-password \
  -n projected-test
```

### Task

Crea il Pod `update-pod` con:

- ConfigMap `dynamic-config`;
- Secret `dynamic-secret`;
- entrambe proiettate in `/dynamic`;
- non usare `subPath`.

Dopo la creazione del Pod:

1. modifica `version` in `v2`;
2. modifica `password` in `new-password`;
3. verifica che i file montati vengano aggiornati.

### Validazione

```bash
kubectl exec update-pod -n projected-test -- cat /dynamic/version
kubectl exec update-pod -n projected-test -- cat /dynamic/password
```

Comandi per aggiornare le risorse:

```bash
kubectl create configmap dynamic-config \
  --from-literal=version=v2 \
  -n projected-test \
  --dry-run=client -o yaml | kubectl apply -f -

kubectl create secret generic dynamic-secret \
  --from-literal=password=new-password \
  -n projected-test \
  --dry-run=client -o yaml | kubectl apply -f -
```

L'aggiornamento del volume non è necessariamente immediato.

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: update-pod
  namespace: projected-test
spec:
  containers:
  - name: app
    image: busybox
    command: ["sh", "-c", "sleep 3600"]
    volumeMounts:
    - name: dynamic-data
      mountPath: /dynamic
      readOnly: true
  volumes:
  - name: dynamic-data
    projected:
      sources:
      - configMap:
          name: dynamic-config
      - secret:
          name: dynamic-secret
```

Dopo aver aggiornato ConfigMap e Secret:

```bash
kubectl exec update-pod -n projected-test -- \
  sh -c 'while true; do date; cat /dynamic/version; cat /dynamic/password; sleep 5; done'
```

</details>

### Cleanup

```bash
kubectl delete namespace projected-test
rm -f update-pod.yaml
```

---

## PROJ-12 — Analizzare il volume automatico `kube-api-access`

### Preparazione

```bash
kubectl delete namespace projected-test --ignore-not-found
kubectl create namespace projected-test

kubectl run inspect-token \
  --image=busybox \
  -n projected-test \
  --command -- sh -c 'sleep 3600'
```

### Task

Analizza il Pod `inspect-token` e individua il volume projected creato automaticamente da Kubernetes.

Identifica le tre sorgenti normalmente presenti:

- `serviceAccountToken`;
- ConfigMap `kube-root-ca.crt`;
- `downwardAPI` con il namespace del Pod.

### Validazione

```bash
kubectl get pod inspect-token -n projected-test -o yaml
kubectl describe pod inspect-token -n projected-test

kubectl exec inspect-token -n projected-test -- \
  find /var/run/secrets/kubernetes.io/serviceaccount -maxdepth 1 -type f -print
```

<details>
<summary>Soluzione</summary>

```bash
kubectl get pod inspect-token -n projected-test \
  -o jsonpath='{range .spec.volumes[*]}{.name}{"\n"}{.projected.sources}{"\n\n"}{end}'
```

Nel manifest del Pod cerca un volume con nome simile a:

```text
kube-api-access-xxxxx
```

Il volume contiene normalmente:

```yaml
projected:
  sources:
  - serviceAccountToken:
      path: token
  - configMap:
      name: kube-root-ca.crt
      items:
      - key: ca.crt
        path: ca.crt
  - downwardAPI:
      items:
      - path: namespace
        fieldRef:
          fieldPath: metadata.namespace
```

</details>

### Cleanup

```bash
kubectl delete namespace projected-test
```

---

# Esercizi avanzati dipendenti dal cluster

I due tipi seguenti fanno parte delle sorgenti ammesse per i projected volume, ma possono essere disabilitati nel cluster:

- `clusterTrustBundle`;
- `podCertificate`.

Prima di eseguirli, verifica la disponibilità delle API.

---

## PROJ-13 — ClusterTrustBundle *(avanzato e opzionale)*

### Preparazione

```bash
kubectl api-resources | grep -i clustertrustbundle || true
```

Se la risorsa non compare, il cluster non è configurato per questo esercizio.

Verifica anche:

```bash
kubectl api-versions | grep certificates.k8s.io
```

### Task

Supponendo che nel cluster esista un ClusterTrustBundle chiamato `example-roots`:

- crea il Pod `trust-bundle-pod`;
- monta il bundle in `/trust/example-roots.pem`;
- usa una proiezione `clusterTrustBundle`;
- rendi la sorgente opzionale.

### Validazione

```bash
kubectl get pod trust-bundle-pod
kubectl exec trust-bundle-pod -- ls -l /trust
```

<details>
<summary>Soluzione indicativa</summary>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: trust-bundle-pod
spec:
  containers:
  - name: app
    image: busybox
    command: ["sh", "-c", "sleep 3600"]
    volumeMounts:
    - name: trust
      mountPath: /trust
      readOnly: true
  volumes:
  - name: trust
    projected:
      sources:
      - clusterTrustBundle:
          name: example-roots
          path: example-roots.pem
          optional: true
```

</details>

### Cleanup

```bash
kubectl delete pod trust-bundle-pod --ignore-not-found
```

---

## PROJ-14 — PodCertificate *(avanzato e opzionale)*

### Preparazione

```bash
kubectl api-resources | grep -i podcertificaterequest || true
```

Se la risorsa non compare, il cluster non è configurato per questo esercizio.

Serve inoltre un signer realmente disponibile, ad esempio uno personalizzato come:

```text
example.com/workload-signer
```

### Task

Crea un Pod che richieda tramite `podCertificate`:

- chiave `ED25519`;
- signer `example.com/workload-signer`;
- durata massima `3600` secondi;
- bundle completo nel file `credential-bundle.pem`;
- mount path `/var/run/workload-certs`.

### Validazione

```bash
kubectl get pod certificate-pod
kubectl exec certificate-pod -- ls -l /var/run/workload-certs
kubectl get podcertificaterequests -A
```

<details>
<summary>Soluzione indicativa</summary>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: certificate-pod
spec:
  serviceAccountName: default
  containers:
  - name: app
    image: busybox
    command: ["sh", "-c", "sleep 3600"]
    volumeMounts:
    - name: workload-certificate
      mountPath: /var/run/workload-certs
      readOnly: true
  volumes:
  - name: workload-certificate
    projected:
      sources:
      - podCertificate:
          signerName: example.com/workload-signer
          keyType: ED25519
          maxExpirationSeconds: 3600
          credentialBundlePath: credential-bundle.pem
```

</details>

### Cleanup

```bash
kubectl delete pod certificate-pod --ignore-not-found
```

---

# Schema riassuntivo

```yaml
volumes:
- name: all-in-one
  projected:
    defaultMode: 0440
    sources:

    - configMap:
        name: example-config
        optional: false
        items:
        - key: config
          path: config/app.conf
          mode: 0444

    - secret:
        name: example-secret
        optional: false
        items:
        - key: password
          path: secrets/password
          mode: 0400

    - downwardAPI:
        items:
        - path: metadata/name
          fieldRef:
            fieldPath: metadata.name
        - path: resources/cpu-limit
          resourceFieldRef:
            containerName: app
            resource: limits.cpu

    - serviceAccountToken:
        audience: api
        expirationSeconds: 3600
        path: tokens/token

    # Solo se supportato e abilitato nel cluster:
    - clusterTrustBundle:
        name: example-roots
        path: trust/roots.pem
        optional: true

    # Solo se supportato, abilitato e con signer disponibile:
    - podCertificate:
        signerName: example.com/workload-signer
        keyType: ED25519
        maxExpirationSeconds: 3600
        credentialBundlePath: certificates/credential-bundle.pem
```
