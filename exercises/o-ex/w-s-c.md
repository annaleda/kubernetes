
- [ Home ](../../readme.md)   | [ Teoria ](../../arguments.md)   | [ Info Exam ](../../doc/ckad_exam_strategy.md)    | [ Home Other Exercises ](../o_exercises.md)

---
# ConfigMap e Secret — esercizi completi

> Esercizi autonomi su tutte le principali modalità di creazione e utilizzo di ConfigMap e Secret.

---

## CM-1 — ConfigMap da directory

### Preparazione ambiente

```bash
mkdir -p /tmp/cm-folder/bar

cat > /tmp/cm-folder/bar/app.properties <<'EOF'
color=blue
mode=production
EOF

cat > /tmp/cm-folder/bar/message.txt <<'EOF'
Welcome to CKAD
EOF

k delete configmap my-config --ignore-not-found
```

### Task

Creare una ConfigMap chiamata `my-config` usando tutti i file presenti nella directory:

```text
/tmp/cm-folder/bar
```

### Validazione

```bash
k get configmap my-config
k get configmap my-config -o yaml
```

La ConfigMap deve contenere le chiavi:

```text
app.properties
message.txt
```

<details>
<summary>Soluzione</summary>

```bash
k create configmap my-config \
  --from-file=/tmp/cm-folder/bar
```

</details>

### Cleanup

```bash
k delete configmap my-config
rm -rf /tmp/cm-folder
```

---

## CM-2 — ConfigMap da file con chiavi personalizzate

### Preparazione ambiente

```bash
mkdir -p /tmp/cm-custom

cat > /tmp/cm-custom/file1.txt <<'EOF'
config1
EOF

cat > /tmp/cm-custom/file2.txt <<'EOF'
config2
EOF

k delete configmap my-config --ignore-not-found
```

### Task

Creare una ConfigMap chiamata `my-config` usando:

- chiave `key1` dal file `/tmp/cm-custom/file1.txt`
- chiave `key2` dal file `/tmp/cm-custom/file2.txt`

### Validazione

```bash
k get configmap my-config \
  -o jsonpath='{.data.key1}{"\n"}{.data.key2}{"\n"}'
```

Output atteso:

```text
config1
config2
```

<details>
<summary>Soluzione</summary>

```bash
k create configmap my-config \
  --from-file=key1=/tmp/cm-custom/file1.txt \
  --from-file=key2=/tmp/cm-custom/file2.txt
```

</details>

### Cleanup

```bash
k delete configmap my-config
rm -rf /tmp/cm-custom
```

---

## CM-3 — ConfigMap da valori letterali

### Preparazione ambiente

```bash
k delete configmap my-config --ignore-not-found
```

### Task

Creare una ConfigMap chiamata `my-config` contenente:

- `key1=config1`
- `key2=config2`

### Validazione

```bash
k get configmap my-config \
  -o jsonpath='{.data.key1}{"\n"}{.data.key2}{"\n"}'
```

<details>
<summary>Soluzione</summary>

```bash
k create configmap my-config \
  --from-literal=key1=config1 \
  --from-literal=key2=config2
```

</details>

### Cleanup

```bash
k delete configmap my-config
```

---

## CM-4 — ConfigMap da env file

### Preparazione ambiente

```bash
cat > /tmp/app.env <<'EOF'
APP_ENV=production
APP_PORT=8080
LOG_LEVEL=info
EOF

k delete configmap app-config --ignore-not-found
```

### Task

Creare una ConfigMap chiamata `app-config` usando il file:

```text
/tmp/app.env
```

### Validazione

```bash
k get configmap app-config -o yaml
```

La ConfigMap deve contenere:

```text
APP_ENV
APP_PORT
LOG_LEVEL
```

<details>
<summary>Soluzione</summary>

```bash
k create configmap app-config \
  --from-env-file=/tmp/app.env
```

</details>

### Cleanup

```bash
k delete configmap app-config
rm -f /tmp/app.env
```

---

## CM-5 — ConfigMap manifest con dry-run

### Preparazione ambiente

```bash
k delete configmap generated-config --ignore-not-found
rm -f /tmp/generated-config.yaml
```

### Task

Generare il manifest YAML di una ConfigMap chiamata `generated-config` contenente:

- `environment=dev`
- `version=v1`

Salvare il manifest in:

```text
/tmp/generated-config.yaml
```

Applicarlo successivamente.

### Validazione

```bash
cat /tmp/generated-config.yaml
k get configmap generated-config -o yaml
```

<details>
<summary>Soluzione</summary>

```bash
k create configmap generated-config \
  --from-literal=environment=dev \
  --from-literal=version=v1 \
  --dry-run=client -o yaml \
  > /tmp/generated-config.yaml

k apply -f /tmp/generated-config.yaml
```

</details>

### Cleanup

```bash
k delete configmap generated-config
rm -f /tmp/generated-config.yaml
```

---

## CM-6 — Usare ConfigMap con envFrom

### Preparazione ambiente

```bash
k delete pod cm-env-pod --ignore-not-found
k delete configmap app-config --ignore-not-found

k create configmap app-config \
  --from-literal=APP_ENV=production \
  --from-literal=APP_PORT=8080
```

### Task

Creare un Pod chiamato `cm-env-pod` con immagine `busybox:1.36` che:

- importi tutte le chiavi della ConfigMap `app-config` come variabili d’ambiente;
- esegua `sleep 3600`.

### Validazione

```bash
k exec cm-env-pod -- printenv APP_ENV
k exec cm-env-pod -- printenv APP_PORT
```

<details>
<summary>Soluzione</summary>

```bash
cat > /tmp/cm-env-pod.yaml <<'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: cm-env-pod
spec:
  containers:
  - name: app
    image: busybox:1.36
    command: ["sh", "-c", "sleep 3600"]
    envFrom:
    - configMapRef:
        name: app-config
EOF

k apply -f /tmp/cm-env-pod.yaml
```

</details>

### Cleanup

```bash
k delete pod cm-env-pod
k delete configmap app-config
rm -f /tmp/cm-env-pod.yaml
```

---

## CM-7 — Usare una singola chiave ConfigMap

### Preparazione ambiente

```bash
k delete pod cm-key-pod --ignore-not-found
k delete configmap app-config --ignore-not-found

k create configmap app-config \
  --from-literal=color=blue
```

### Task

Creare un Pod chiamato `cm-key-pod` che esponga la chiave `color` della ConfigMap `app-config` nella variabile:

```text
APP_COLOR
```

### Validazione

```bash
k exec cm-key-pod -- printenv APP_COLOR
```

Output atteso:

```text
blue
```

<details>
<summary>Soluzione</summary>

```bash
cat > /tmp/cm-key-pod.yaml <<'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: cm-key-pod
spec:
  containers:
  - name: app
    image: busybox:1.36
    command: ["sh", "-c", "sleep 3600"]
    env:
    - name: APP_COLOR
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: color
EOF

k apply -f /tmp/cm-key-pod.yaml
```

</details>

### Cleanup

```bash
k delete pod cm-key-pod
k delete configmap app-config
rm -f /tmp/cm-key-pod.yaml
```

---

## CM-8 — Montare ConfigMap come volume

### Preparazione ambiente

```bash
k delete pod cm-volume-pod --ignore-not-found
k delete configmap web-config --ignore-not-found

k create configmap web-config \
  --from-literal=index.html='Welcome from ConfigMap'
```

### Task

Creare un Pod chiamato `cm-volume-pod` con immagine `nginx:1.27` che monti la ConfigMap `web-config` in:

```text
/usr/share/nginx/html
```

### Validazione

```bash
k exec cm-volume-pod -- \
  cat /usr/share/nginx/html/index.html
```

Output atteso:

```text
Welcome from ConfigMap
```

<details>
<summary>Soluzione</summary>

```bash
cat > /tmp/cm-volume-pod.yaml <<'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: cm-volume-pod
spec:
  containers:
  - name: nginx
    image: nginx:1.27
    volumeMounts:
    - name: config
      mountPath: /usr/share/nginx/html
  volumes:
  - name: config
    configMap:
      name: web-config
EOF

k apply -f /tmp/cm-volume-pod.yaml
```

</details>

### Cleanup

```bash
k delete pod cm-volume-pod
k delete configmap web-config
rm -f /tmp/cm-volume-pod.yaml
```

---

## SEC-1 — Secret generic da literal

### Preparazione ambiente

```bash
k delete secret app-secret --ignore-not-found
```

### Task

Creare un Secret generico chiamato `app-secret` contenente:

- `username=admin`
- `password=ckad123`

### Validazione

```bash
k get secret app-secret
k get secret app-secret \
  -o jsonpath='{.data.username}' | base64 -d
echo
```

<details>
<summary>Soluzione</summary>

```bash
k create secret generic app-secret \
  --from-literal=username=admin \
  --from-literal=password=ckad123
```

</details>

### Cleanup

```bash
k delete secret app-secret
```

---

## SEC-2 — Secret generic da file

### Preparazione ambiente

```bash
mkdir -p /tmp/secret-files

printf '%s' 'admin' > /tmp/secret-files/username
printf '%s' 'ckad123' > /tmp/secret-files/password

k delete secret file-secret --ignore-not-found
```

### Task

Creare un Secret generico chiamato `file-secret` usando i file:

- `/tmp/secret-files/username`
- `/tmp/secret-files/password`

### Validazione

```bash
k get secret file-secret \
  -o jsonpath='{.data.username}' | base64 -d
echo

k get secret file-secret \
  -o jsonpath='{.data.password}' | base64 -d
echo
```

<details>
<summary>Soluzione</summary>

```bash
k create secret generic file-secret \
  --from-file=/tmp/secret-files/username \
  --from-file=/tmp/secret-files/password
```

</details>

### Cleanup

```bash
k delete secret file-secret
rm -rf /tmp/secret-files
```

---

## SEC-3 — Secret generic da directory

### Preparazione ambiente

```bash
mkdir -p /tmp/secret-dir

printf '%s' 'api-user' > /tmp/secret-dir/user
printf '%s' 'token-123' > /tmp/secret-dir/token

k delete secret dir-secret --ignore-not-found
```

### Task

Creare un Secret generico chiamato `dir-secret` usando tutti i file presenti nella directory:

```text
/tmp/secret-dir
```

### Validazione

```bash
k get secret dir-secret -o yaml
```

Il Secret deve contenere le chiavi:

```text
user
token
```

<details>
<summary>Soluzione</summary>

```bash
k create secret generic dir-secret \
  --from-file=/tmp/secret-dir
```

</details>

### Cleanup

```bash
k delete secret dir-secret
rm -rf /tmp/secret-dir
```

---

## SEC-4 — Secret generic da env file

### Preparazione ambiente

```bash
cat > /tmp/secret.env <<'EOF'
DB_USER=admin
DB_PASSWORD=supersecret
EOF

k delete secret env-secret --ignore-not-found
```

### Task

Creare un Secret generico chiamato `env-secret` usando:

```text
/tmp/secret.env
```

### Validazione

```bash
k get secret env-secret \
  -o jsonpath='{.data.DB_USER}' | base64 -d
echo
```

<details>
<summary>Soluzione</summary>

```bash
k create secret generic env-secret \
  --from-env-file=/tmp/secret.env
```

</details>

### Cleanup

```bash
k delete secret env-secret
rm -f /tmp/secret.env
```

---

## SEC-5 — Secret generic con chiavi personalizzate

### Preparazione ambiente

```bash
printf '%s' 'admin' > /tmp/db-user.txt
printf '%s' 'password123' > /tmp/db-pass.txt

k delete secret custom-secret --ignore-not-found
```

### Task

Creare un Secret chiamato `custom-secret` con:

- chiave `username` dal file `/tmp/db-user.txt`
- chiave `password` dal file `/tmp/db-pass.txt`

### Validazione

```bash
k get secret custom-secret \
  -o jsonpath='{.data.username}' | base64 -d
echo
```

<details>
<summary>Soluzione</summary>

```bash
k create secret generic custom-secret \
  --from-file=username=/tmp/db-user.txt \
  --from-file=password=/tmp/db-pass.txt
```

</details>

### Cleanup

```bash
k delete secret custom-secret
rm -f /tmp/db-user.txt /tmp/db-pass.txt
```

---

## SEC-6 — Secret docker-registry

### Preparazione ambiente

```bash
k delete secret reg-secret --ignore-not-found
```

### Task

Creare un Secret per registry Docker chiamato `reg-secret` con:

- server: `registry.example.com`
- username: `ckad-user`
- password: `ckad-pass`
- email: `ckad@example.com`

### Validazione

```bash
k get secret reg-secret \
  -o jsonpath='{.type}{"\n"}'
```

Output atteso:

```text
kubernetes.io/dockerconfigjson
```

<details>
<summary>Soluzione</summary>

```bash
k create secret docker-registry reg-secret \
  --docker-server=registry.example.com \
  --docker-username=ckad-user \
  --docker-password=ckad-pass \
  --docker-email=ckad@example.com
```

</details>

### Cleanup

```bash
k delete secret reg-secret
```

---

## SEC-7 — Secret TLS

### Preparazione ambiente

Generare certificato e chiave autofirmati:

```bash
openssl req -x509 -nodes \
  -newkey rsa:2048 \
  -keyout /tmp/tls.key \
  -out /tmp/tls.crt \
  -subj '/CN=ckad.local' \
  -days 1

k delete secret tls-secret --ignore-not-found
```

### Task

Creare un Secret TLS chiamato `tls-secret` usando:

- certificato: `/tmp/tls.crt`
- chiave: `/tmp/tls.key`

### Validazione

```bash
k get secret tls-secret \
  -o jsonpath='{.type}{"\n"}'
```

Output atteso:

```text
kubernetes.io/tls
```

<details>
<summary>Soluzione</summary>

```bash
k create secret tls tls-secret \
  --cert=/tmp/tls.crt \
  --key=/tmp/tls.key
```

</details>

### Cleanup

```bash
k delete secret tls-secret
rm -f /tmp/tls.crt /tmp/tls.key
```

---

## SEC-8 — Usare Secret con envFrom

### Preparazione ambiente

```bash
k delete pod secret-env-pod --ignore-not-found
k delete secret app-secret --ignore-not-found

k create secret generic app-secret \
  --from-literal=USERNAME=admin \
  --from-literal=PASSWORD=ckad123
```

### Task

Creare un Pod chiamato `secret-env-pod` che importi tutte le chiavi del Secret `app-secret` come variabili d’ambiente.

### Validazione

```bash
k exec secret-env-pod -- printenv USERNAME
k exec secret-env-pod -- printenv PASSWORD
```

<details>
<summary>Soluzione</summary>

```bash
cat > /tmp/secret-env-pod.yaml <<'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: secret-env-pod
spec:
  containers:
  - name: app
    image: busybox:1.36
    command: ["sh", "-c", "sleep 3600"]
    envFrom:
    - secretRef:
        name: app-secret
EOF

k apply -f /tmp/secret-env-pod.yaml
```

</details>

### Cleanup

```bash
k delete pod secret-env-pod
k delete secret app-secret
rm -f /tmp/secret-env-pod.yaml
```

---

## SEC-9 — Usare una singola chiave Secret

### Preparazione ambiente

```bash
k delete pod secret-key-pod --ignore-not-found
k delete secret db-secret --ignore-not-found

k create secret generic db-secret \
  --from-literal=password=supersecret
```

### Task

Creare un Pod chiamato `secret-key-pod` che esponga la chiave `password` del Secret `db-secret` nella variabile:

```text
DB_PASSWORD
```

### Validazione

```bash
k exec secret-key-pod -- printenv DB_PASSWORD
```

<details>
<summary>Soluzione</summary>

```bash
cat > /tmp/secret-key-pod.yaml <<'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: secret-key-pod
spec:
  containers:
  - name: app
    image: busybox:1.36
    command: ["sh", "-c", "sleep 3600"]
    env:
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: db-secret
          key: password
EOF

k apply -f /tmp/secret-key-pod.yaml
```

</details>

### Cleanup

```bash
k delete pod secret-key-pod
k delete secret db-secret
rm -f /tmp/secret-key-pod.yaml
```

---

## SEC-10 — Montare Secret come volume

### Preparazione ambiente

```bash
k delete pod secret-volume-pod --ignore-not-found
k delete secret volume-secret --ignore-not-found

k create secret generic volume-secret \
  --from-literal=username=admin \
  --from-literal=password=ckad123
```

### Task

Creare un Pod chiamato `secret-volume-pod` che monti il Secret `volume-secret` in:

```text
/etc/credentials
```

### Validazione

```bash
k exec secret-volume-pod -- \
  cat /etc/credentials/username

k exec secret-volume-pod -- \
  cat /etc/credentials/password
```

<details>
<summary>Soluzione</summary>

```bash
cat > /tmp/secret-volume-pod.yaml <<'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: secret-volume-pod
spec:
  containers:
  - name: app
    image: busybox:1.36
    command: ["sh", "-c", "sleep 3600"]
    volumeMounts:
    - name: credentials
      mountPath: /etc/credentials
      readOnly: true
  volumes:
  - name: credentials
    secret:
      secretName: volume-secret
EOF

k apply -f /tmp/secret-volume-pod.yaml
```

</details>

### Cleanup

```bash
k delete pod secret-volume-pod
k delete secret volume-secret
rm -f /tmp/secret-volume-pod.yaml
```

---

## SEC-11 — Generare manifest Secret con dry-run

### Preparazione ambiente

```bash
k delete secret generated-secret --ignore-not-found
rm -f /tmp/generated-secret.yaml
```

### Task

Generare il manifest YAML di un Secret generico chiamato `generated-secret` contenente:

- `token=abc123`

Salvare il manifest in:

```text
/tmp/generated-secret.yaml
```

Applicarlo successivamente.

### Validazione

```bash
cat /tmp/generated-secret.yaml
k get secret generated-secret
```

<details>
<summary>Soluzione</summary>

```bash
k create secret generic generated-secret \
  --from-literal=token=abc123 \
  --dry-run=client -o yaml \
  > /tmp/generated-secret.yaml

k apply -f /tmp/generated-secret.yaml
```

</details>

### Cleanup

```bash
k delete secret generated-secret
rm -f /tmp/generated-secret.yaml
```

---
