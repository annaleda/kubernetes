- [ Home ](../../readme.md)   | [ Teoria ](../../arguments.md)   | [ Info Exam ](../../doc/ckad_exam_strategy.md)    | [ Home Other Exercises ](../o_exercises.md) |  [ Cheatsheet Docker ](./Docker_CheatSheet_CKAD.md) |  [ Cheatsheet Podman ](./Podman_CheatSheet_CKAD.md)

> Esercizi pratici su immagini OCI, container, YAML, HTTP e Helm.

---

# Docker — 5 esercizi

---

## Docker-1 — Build e verifica immagine

### Preparazione ambiente

```bash
mkdir -p /tmp/docker-1
cd /tmp/docker-1
```

Creare il `Dockerfile`:

```bash
cat > Dockerfile <<'EOF'
FROM nginx:1.27

COPY index.html /usr/share/nginx/html/index.html

EXPOSE 80
EOF
```

Creare `index.html`:

```bash
cat > index.html <<'EOF'
<!DOCTYPE html>
<html lang="it">
<head>
  <meta charset="UTF-8">
  <title>Docker CKAD Exercise</title>
</head>
<body>
  <h1>Welcome to the CKAD Docker Exercises!</h1>
  <p>The image was built successfully.</p>
</body>
</html>
EOF
```

Verificare:

```bash
ls -l
cat Dockerfile
```

### Task

Creare l’immagine:

- `webapp:v1`

### Validazione

```bash
docker images webapp:v1
docker inspect webapp:v1
```

Test facoltativo:

```bash
docker run --rm -d --name webapp-test -p 8080:80 webapp:v1
curl http://localhost:8080
docker rm -f webapp-test
```

---

<details>
<summary>Soluzione</summary>

```bash
cd /tmp/docker-1
docker build -t webapp:v1 .
docker images webapp:v1
docker inspect webapp:v1
```

</details>

---

## Docker-2 — Avvio container con porta

### Preparazione ambiente

Rimuovere un eventuale container omonimo e scaricare l’immagine necessaria:

```bash
docker rm -f web 2>/dev/null || true
docker pull nginx:1.27
```

### Task

Avviare un container:

- nome: `web`
- immagine: `nginx:1.27`
- modalità: detached
- porta host: `8080`
- porta container: `80`

### Validazione

```bash
docker ps --filter name=web
curl http://localhost:8080
```

---

<details>
<summary>Soluzione</summary>

```bash
docker run -d \
  --name web \
  -p 8080:80 \
  nginx:1.27

docker ps --filter name=web
curl http://localhost:8080
```

</details>

---

## Docker-3 — Variabili d’ambiente e inspect

### Preparazione ambiente

```bash
docker rm -f env-container 2>/dev/null || true
docker pull alpine
```

### Task

Avviare un container:

- nome: `env-container`
- immagine: `alpine`
- variabile: `APP_ENV=production`
- comando: `sleep 3600`

Verificare la variabile dall’interno del container e tramite `docker inspect`.

### Validazione

```bash
docker ps --filter name=env-container
docker exec env-container printenv APP_ENV
docker inspect -f '{{range .Config.Env}}{{println .}}{{end}}' env-container
```

---

<details>
<summary>Soluzione</summary>

```bash
docker run -d \
  --name env-container \
  -e APP_ENV=production \
  alpine \
  sleep 3600

docker exec env-container printenv APP_ENV

docker inspect -f \
  '{{range .Config.Env}}{{println .}}{{end}}' \
  env-container
```

</details>

---

## Docker-4 — Save, remove e load

### Preparazione ambiente

Assicurarsi che non esistano container basati sull’immagine e scaricarla:

```bash
docker rm -f web 2>/dev/null || true
docker pull nginx:1.27
rm -f /tmp/nginx.tar
```

### Task

1. Salvare `nginx:1.27` nel file `/tmp/nginx.tar`.
2. Verificare il file.
3. Eliminare l’immagine locale.
4. Ricaricarla dall’archivio.
5. Verificarne la presenza.

### Validazione

```bash
ls -lh /tmp/nginx.tar
docker images nginx:1.27
```

---

<details>
<summary>Soluzione</summary>

```bash
docker save -o /tmp/nginx.tar nginx:1.27

ls -lh /tmp/nginx.tar

docker rmi nginx:1.27

docker load -i /tmp/nginx.tar

docker images nginx:1.27
```

</details>

---

## Docker-5 — Cleanup completo

Eliminare:

* tutti i container, inclusi quelli arrestati;
* tutte le immagini;
* tutti i volumi inutilizzati;
* tutte le reti inutilizzate.

### Validazione

```sh
docker ps -a
docker images
docker volume ls
docker network ls
```

---

<details>
<summary>Soluzione</summary>

```sh
docker rm -f $(docker ps -aq)

docker rmi -f $(docker images -q)

docker volume prune -f

docker network prune -f
```

Alternativa:

```sh
docker system prune -a --volumes -f
```

</details>

---

# Podman — 5 esercizi

---

## Podman-1 — Build immagine

### Preparazione ambiente

```bash
mkdir -p /tmp/podman-1
cd /tmp/podman-1
```

Creare il `Dockerfile`:

```bash
cat > Dockerfile <<'EOF'
FROM nginx:1.27-alpine

COPY index.html /usr/share/nginx/html/index.html

EXPOSE 80
EOF
```

Creare `index.html`:

```bash
cat > index.html <<'EOF'
<!DOCTYPE html>
<html lang="it">
<head>
  <meta charset="UTF-8">
  <title>Podman CKAD Exercise</title>
</head>
<body>
  <h1>Welcome to the CKAD Podman Exercises!</h1>
</body>
</html>
EOF
```

### Task

Creare l’immagine:

- `api:v1`

### Validazione

```bash
podman images api:v1
podman inspect api:v1
```

Test facoltativo:

```bash
podman run --rm -d --name api-test -p 8081:80 api:v1
curl http://localhost:8081
podman rm -f api-test
```

---

<details>
<summary>Soluzione</summary>

```bash
cd /tmp/podman-1
podman build -t api:v1 .
podman images api:v1
podman inspect api:v1
```

</details>

---

## Podman-2 — Avvio container

### Preparazione ambiente

```bash
podman rm -f podman-web 2>/dev/null || true
podman pull nginx:1.27
```

### Task

Avviare un container:

- nome: `podman-web`
- immagine: `nginx:1.27`
- modalità detached
- porta `8081:80`

### Validazione

```bash
podman ps --filter name=podman-web
curl http://localhost:8081
```

---

<details>
<summary>Soluzione</summary>

```bash
podman run -d \
  --name podman-web \
  -p 8081:80 \
  nginx:1.27

podman ps --filter name=podman-web
curl http://localhost:8081
```

</details>

---

## Podman-3 — Conteggio container

### Preparazione ambiente

Creare container in stati differenti:

```bash
podman rm -f count-running count-stopped count-exited 2>/dev/null || true

podman run -d \
  --name count-running \
  alpine \
  sleep 3600

podman create \
  --name count-stopped \
  alpine \
  sleep 3600

podman run \
  --name count-exited \
  alpine \
  true
```

### Task

1. Contare tutti i container presenti sull’host.
2. Contare solo quelli in esecuzione.
3. Mostrare la lista completa con il relativo stato.

### Validazione

Nell’ambiente appena creato devono essere presenti tre container, di cui uno in esecuzione.

---

<details>
<summary>Soluzione</summary>

Tutti i container:

```bash
podman ps -aq | wc -l
```

Solo quelli in esecuzione:

```bash
podman ps -q | wc -l
```

Lista completa:

```bash
podman ps -a
```

</details>

---

## Podman-4 — Salvare immagine OCI

### Preparazione ambiente

Se `api:v1` non esiste, crearla rapidamente:

```bash
podman image exists api:v1 || \
  podman pull nginx:1.27-alpine

podman image exists api:v1 || \
  podman tag nginx:1.27-alpine api:v1

rm -f /tmp/api.tar
```

### Task

Salvare l’immagine `api:v1` nel file `/tmp/api.tar` usando il formato OCI archive.

### Validazione

```bash
ls -lh /tmp/api.tar
```

---

<details>
<summary>Soluzione</summary>

```bash
podman save \
  --format oci-archive \
  -o /tmp/api.tar \
  api:v1

ls -lh /tmp/api.tar
```

</details>

---

## Podman-5 — Eliminazione completa

Eliminare:

* tutti i container;
* tutte le immagini.

### Validazione

```sh
podman ps -a
podman images
```

---

<details>
<summary>Soluzione</summary>

```sh
podman rm -f -a

podman rmi -f -a
```

Pulizia generale:

```sh
podman system prune -a -f
```

</details>

---

# yq — 5 esercizi

---

## YQ-1 — Leggere nome e immagine

### Preparazione ambiente

```bash
mkdir -p /tmp/yq-exercises
cd /tmp/yq-exercises
```

Creare `pod.yaml`:

```bash
cat > pod.yaml <<'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: multi-container-pod
  namespace: ckad-tools
spec:
  containers:
  - name: web
    image: nginx:1.27
  - name: sidecar
    image: busybox:1.36
    command: ["sh", "-c", "sleep 3600"]
EOF
```

### Task

Mostrare con `yq`:

- nome del Pod;
- namespace;
- immagini di tutti i container.

### Validazione

Output atteso:

```text
multi-container-pod
ckad-tools
nginx:1.27
busybox:1.36
```

---

<details>
<summary>Soluzione</summary>

```bash
yq '.metadata.name' pod.yaml
yq '.metadata.namespace' pod.yaml
yq '.spec.containers[].image' pod.yaml
```

</details>

---

## YQ-2 — Cambiare immagine

### Preparazione ambiente

```bash
mkdir -p /tmp/yq-exercises
cd /tmp/yq-exercises

cat > deployment.yaml <<'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: nginx
        image: nginx:1.25
EOF
```

### Task

Modificare direttamente `deployment.yaml` impostando l’immagine del primo container a:

```text
nginx:1.27
```

### Validazione

```bash
yq '.spec.template.spec.containers[0].image' deployment.yaml
```

Output atteso:

```text
nginx:1.27
```

---

<details>
<summary>Soluzione</summary>

```bash
yq -i   '.spec.template.spec.containers[0].image = "nginx:1.27"'   deployment.yaml
```

</details>

---

## YQ-3 — Cambiare repliche e namespace

### Preparazione ambiente

```bash
mkdir -p /tmp/yq-exercises
cd /tmp/yq-exercises

cat > deployment.yaml <<'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: nginx
        image: nginx:1.25
EOF
```

### Task

Nel file `deployment.yaml`:

- impostare `replicas` a `4`;
- impostare il namespace a `production`.

### Validazione

```bash
yq '.spec.replicas' deployment.yaml
yq '.metadata.namespace' deployment.yaml
```

Output atteso:

```text
4
production
```

---

<details>
<summary>Soluzione</summary>

```bash
yq -i '.spec.replicas = 4' deployment.yaml
yq -i '.metadata.namespace = "production"' deployment.yaml
```

</details>

---

## YQ-4 — Aggiungere una variabile d’ambiente

### Preparazione ambiente

```bash
mkdir -p /tmp/yq-exercises
cd /tmp/yq-exercises

cat > pod.yaml <<'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: env-pod
spec:
  containers:
  - name: app
    image: alpine
    command: ["sh", "-c", "sleep 3600"]
EOF
```

### Task

Nel primo container di `pod.yaml`, aggiungere:

- nome: `APP_ENV`
- valore: `production`

### Validazione

```bash
yq '.spec.containers[0].env' pod.yaml
```

---

<details>
<summary>Soluzione</summary>

```bash
yq -i \
  '.spec.containers[0].env += [{"name":"APP_ENV","value":"production"}]' \
  pod.yaml
```

</details>

---

## YQ-5 — Pulire un manifest esportato

### Preparazione ambiente

```bash
mkdir -p /tmp/yq-exercises
cd /tmp/yq-exercises

cat > pod-export.yaml <<'EOF'
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2026-07-16T10:00:00Z"
  name: exported-pod
  namespace: default
  resourceVersion: "12345"
  uid: 11111111-2222-3333-4444-555555555555
spec:
  containers:
  - name: app
    image: nginx:1.27
status:
  phase: Running
  podIP: 10.244.0.10
EOF
```

### Task

Eliminare dal file:

- `.metadata.creationTimestamp`
- `.metadata.resourceVersion`
- `.metadata.uid`
- `.status`

### Validazione

```bash
yq pod-export.yaml
```

I campi indicati non devono più essere presenti.

---

<details>
<summary>Soluzione</summary>

```bash
yq -i '
  del(
    .metadata.creationTimestamp,
    .metadata.resourceVersion,
    .metadata.uid,
    .status
  )
' pod-export.yaml

yq pod-export.yaml
```

</details>

---

## CURL-1 — Verificare un Service

### Preparazione ambiente

Creare namespace, Deployment e Service:

```bash
kubectl create namespace frontend \
  --dry-run=client -o yaml | kubectl apply -f -

kubectl create deployment web \
  -n frontend \
  --image=nginx:1.27

kubectl expose deployment web \
  -n frontend \
  --name=web-service \
  --port=80 \
  --target-port=80

kubectl rollout status deployment/web -n frontend
```

### Task

Avviare un Pod temporaneo e verificare la risposta HTTP del Service `web-service`.

### Validazione

La risposta deve contenere la pagina predefinita di NGINX.

---

<details>
<summary>Soluzione</summary>

```bash
kubectl run curl-test \
  -n frontend \
  --image=curlimages/curl \
  --rm -it \
  --restart=Never \
  -- curl -s http://web-service
```

Alternativa con BusyBox:

```bash
kubectl run test \
  -n frontend \
  --image=busybox \
  --rm -it \
  --restart=Never \
  -- wget -qO- http://web-service
```

</details>

### Cleanup

```bash
kubectl delete namespace frontend
```

---

## CURL-2 — Mostrare status e header

### Preparazione ambiente

Creare un server HTTP locale sulla porta `8080`:

```bash
docker rm -f curl-web 2>/dev/null || true

docker run -d \
  --name curl-web \
  -p 8080:80 \
  nginx:1.27
```

### Task

Effettuare una richiesta verso:

```text
http://localhost:8080
```

Mostrare:

- status HTTP;
- header;
- body.

### Validazione

Lo status atteso è `200`.

---

<details>
<summary>Soluzione</summary>

Header e body:

```bash
curl -i http://localhost:8080
```

Solo gli header:

```bash
curl -I http://localhost:8080
```

Solo lo status code:

```bash
curl -s -o /dev/null -w '%{http_code}
' \
  http://localhost:8080
```

</details>

### Cleanup

```bash
docker rm -f curl-web
```

---

## CURL-3 — Test Ingress con Host header

Un Ingress risponde all’host:

```text
web.ckad.local
```

L’Ingress Controller è raggiungibile all’indirizzo:

```text
10.10.10.10
```

Verificare la risposta senza modificare `/etc/hosts`.

---

<details>
<summary>Soluzione</summary>

```sh
curl \
  -H 'Host: web.ckad.local' \
  http://10.10.10.10
```

In modalità verbose:

```sh
curl -v \
  -H 'Host: web.ckad.local' \
  http://10.10.10.10
```

</details>

---

## CURL-4 — POST JSON

Inviare una richiesta POST verso:

```text
http://localhost:8080/api/users
```

con il body:

```json
{
  "name": "ckad",
  "role": "developer"
}
```

---

<details>
<summary>Soluzione</summary>

```sh
curl -X POST \
  -H 'Content-Type: application/json' \
  -d '{"name":"ckad","role":"developer"}' \
  http://localhost:8080/api/users
```

</details>

---

## CURL-5 — Debug HTTPS

Verificare un endpoint HTTPS con certificato non attendibile:

```text
https://app.ckad.local
```

Mostrare anche i dettagli della connessione.

---

<details>
<summary>Soluzione</summary>

```sh
curl -k -v https://app.ckad.local
```

Dove:

* `-k` ignora la validazione del certificato;
* `-v` mostra i dettagli della connessione.

</details>

---

# Helm — 5 esercizi

---

## Helm-1 — Validare un chart

### Preparazione ambiente

```bash
rm -rf /opt/webapp
mkdir -p /opt/webapp/templates

cat > /opt/webapp/Chart.yaml <<'EOF'
apiVersion: v2
name: webapp
description: Minimal CKAD Helm chart
type: application
version: 0.1.0
appVersion: "1.27"
EOF

cat > /opt/webapp/values.yaml <<'EOF'
replicaCount: 1

image:
  repository: nginx
  tag: "1.27-alpine"
  pullPolicy: IfNotPresent

service:
  type: ClusterIP
  port: 80
EOF

cat > /opt/webapp/templates/deployment.yaml <<'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: {{ .Release.Name }}
  template:
    metadata:
      labels:
        app: {{ .Release.Name }}
    spec:
      containers:
      - name: nginx
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
        imagePullPolicy: {{ .Values.image.pullPolicy }}
        ports:
        - containerPort: 80
EOF

cat > /opt/webapp/templates/service.yaml <<'EOF'
apiVersion: v1
kind: Service
metadata:
  name: {{ .Release.Name }}
spec:
  type: {{ .Values.service.type }}
  selector:
    app: {{ .Release.Name }}
  ports:
  - port: {{ .Values.service.port }}
    targetPort: 80
EOF
```

### Task

Eseguire:

- validazione sintattica;
- rendering dei manifest senza installare il chart.

### Validazione

Entrambi i comandi devono terminare senza errori.

---

<details>
<summary>Soluzione</summary>

```bash
helm lint /opt/webapp

helm template webapp /opt/webapp   -n frontend
```

</details>

---

## Helm-2 — Installare una release

### Preparazione ambiente

```bash
rm -rf /opt/webapp
mkdir -p /opt/webapp/templates

cat > /opt/webapp/Chart.yaml <<'EOF'
apiVersion: v2
name: webapp
description: Minimal CKAD Helm chart
type: application
version: 0.1.0
appVersion: "1.27"
EOF

cat > /opt/webapp/values.yaml <<'EOF'
replicaCount: 1

image:
  repository: nginx
  tag: "1.27-alpine"
  pullPolicy: IfNotPresent

service:
  type: ClusterIP
  port: 80
EOF

cat > /opt/webapp/templates/deployment.yaml <<'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: {{ .Release.Name }}
  template:
    metadata:
      labels:
        app: {{ .Release.Name }}
    spec:
      containers:
      - name: nginx
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
        imagePullPolicy: {{ .Values.image.pullPolicy }}
        ports:
        - containerPort: 80
EOF

cat > /opt/webapp/templates/service.yaml <<'EOF'
apiVersion: v1
kind: Service
metadata:
  name: {{ .Release.Name }}
spec:
  type: {{ .Values.service.type }}
  selector:
    app: {{ .Release.Name }}
  ports:
  - port: {{ .Values.service.port }}
    targetPort: 80
EOF
```

Rimuovere un’eventuale release precedente:

```bash
helm uninstall webapp -n frontend 2>/dev/null || true
```

### Task

Installare il chart `/opt/webapp` con:

- release: `webapp`;
- namespace: `frontend`;
- creazione automatica del namespace.

### Validazione

```bash
helm list -n frontend
helm status webapp -n frontend
kubectl get all -n frontend
```

---

<details>
<summary>Soluzione</summary>

```bash
helm install webapp   /opt/webapp   -n frontend   --create-namespace
```

</details>

---

## Helm-3 — Installare con valori personalizzati

### Preparazione ambiente

```bash
rm -rf /opt/webapp
mkdir -p /opt/webapp/templates

cat > /opt/webapp/Chart.yaml <<'EOF'
apiVersion: v2
name: webapp
description: Minimal CKAD Helm chart
type: application
version: 0.1.0
appVersion: "1.27"
EOF

cat > /opt/webapp/values.yaml <<'EOF'
replicaCount: 1

image:
  repository: nginx
  tag: "1.27-alpine"
  pullPolicy: IfNotPresent

service:
  type: ClusterIP
  port: 80
EOF

cat > /opt/webapp/templates/deployment.yaml <<'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: {{ .Release.Name }}
  template:
    metadata:
      labels:
        app: {{ .Release.Name }}
    spec:
      containers:
      - name: nginx
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
        imagePullPolicy: {{ .Values.image.pullPolicy }}
        ports:
        - containerPort: 80
EOF

cat > /opt/webapp/templates/service.yaml <<'EOF'
apiVersion: v1
kind: Service
metadata:
  name: {{ .Release.Name }}
spec:
  type: {{ .Values.service.type }}
  selector:
    app: {{ .Release.Name }}
  ports:
  - port: {{ .Values.service.port }}
    targetPort: 80
EOF
```

Creare `/opt/webapp/prod-values.yaml`:

```bash
cat > /opt/webapp/prod-values.yaml <<'EOF'
replicaCount: 2

service:
  type: ClusterIP
  port: 80
EOF
```

Rimuovere un’eventuale release precedente:

```bash
helm uninstall webapp-prod -n production 2>/dev/null || true
```

### Task

Installare il chart usando:

- release: `webapp-prod`;
- namespace: `production`;
- file: `/opt/webapp/prod-values.yaml`;
- `replicaCount=3` tramite riga di comando.

Il valore passato con `--set` deve prevalere sul file values.

### Validazione

```bash
helm get values webapp-prod -n production
kubectl get deployment webapp-prod -n production
```

Il Deployment deve avere tre repliche desiderate.

---

<details>
<summary>Soluzione</summary>

```bash
helm install webapp-prod   /opt/webapp   -n production   --create-namespace   -f /opt/webapp/prod-values.yaml   --set replicaCount=3
```

</details>

---

## Helm-4 — Upgrade e rollback

### Preparazione ambiente

```bash
rm -rf /opt/webapp
mkdir -p /opt/webapp/templates

cat > /opt/webapp/Chart.yaml <<'EOF'
apiVersion: v2
name: webapp
description: Minimal CKAD Helm chart
type: application
version: 0.1.0
appVersion: "1.27"
EOF

cat > /opt/webapp/values.yaml <<'EOF'
replicaCount: 1

image:
  repository: nginx
  tag: "1.27-alpine"
  pullPolicy: IfNotPresent

service:
  type: ClusterIP
  port: 80
EOF

cat > /opt/webapp/templates/deployment.yaml <<'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: {{ .Release.Name }}
  template:
    metadata:
      labels:
        app: {{ .Release.Name }}
    spec:
      containers:
      - name: nginx
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
        imagePullPolicy: {{ .Values.image.pullPolicy }}
        ports:
        - containerPort: 80
EOF

cat > /opt/webapp/templates/service.yaml <<'EOF'
apiVersion: v1
kind: Service
metadata:
  name: {{ .Release.Name }}
spec:
  type: {{ .Values.service.type }}
  selector:
    app: {{ .Release.Name }}
  ports:
  - port: {{ .Values.service.port }}
    targetPort: 80
EOF
```

Installare la prima revisione:

```bash
helm uninstall webapp -n frontend 2>/dev/null || true

helm install webapp   /opt/webapp   -n frontend   --create-namespace   --set image.tag=1.26-alpine
```

### Task

1. Aggiornare il tag dell’immagine a `1.27-alpine`.
2. Verificare la cronologia.
3. Eseguire il rollback alla revisione precedente.
4. Verificare l’immagine finale del Deployment.

### Validazione

```bash
helm history webapp -n frontend

kubectl get deployment webapp   -n frontend   -o jsonpath='{.spec.template.spec.containers[0].image}{"
"}'
```

Dopo il rollback, l’immagine deve essere `nginx:1.26-alpine`.

---

<details>
<summary>Soluzione</summary>

Upgrade:

```bash
helm upgrade webapp   /opt/webapp   -n frontend   --set image.tag=1.27-alpine
```

Cronologia:

```bash
helm history webapp -n frontend
```

Rollback alla revisione 1:

```bash
helm rollback webapp 1 -n frontend
```

Verifica:

```bash
helm status webapp -n frontend

kubectl get deployment webapp   -n frontend   -o jsonpath='{.spec.template.spec.containers[0].image}{"
"}'
```

</details>

---

## Helm-5 — Debug chart non installabile

### Preparazione ambiente

Creare un chart intenzionalmente errato:

```bash
rm -rf /opt/webapp-broken
mkdir -p /opt/webapp-broken/templates

cat > /opt/webapp-broken/Chart.yaml <<'EOF'
apiVersion: v2
name: webapp-broken
description: Broken chart for troubleshooting
type: application
version: 0.1.0
appVersion: "1.27"
EOF

cat > /opt/webapp-broken/values.yaml <<'EOF'
replicaCount: 1

image:
  repository: nginx
  tag: "1.27-alpine"
EOF

cat > /opt/webapp-broken/templates/deployment.yaml <<'EOF'
apiVersion: v1
kind: Deployment
metadata:
  name: webapp-broken
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: webapp-broken
  template:
    metadata:
      labels:
        app: webapp-broken
    spec:
      containers:
      - name: nginx
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
EOF
```

Riprodurre l’errore:

```bash
helm install webapp-broken \
  /opt/webapp-broken \
  -n frontend \
  --create-namespace
```

L’installazione deve restituire un errore simile a:

```text
no matches for kind "Deployment" in version "v1"
```

### Task

1. Individuare il file errato.
2. Correggere l’API version del Deployment.
3. Validare il chart.
4. Installare la release.

### Validazione

```bash
helm lint /opt/webapp-broken
helm status webapp-broken -n frontend
kubectl get deployment webapp-broken -n frontend
```

---

<details>
<summary>Soluzione</summary>

Cercare il problema:

```bash
grep -R -n \
  'kind: Deployment\|apiVersion:' \
  /opt/webapp-broken/templates
```

Correggere con `sed`:

```bash
sed -i \
  's/^apiVersion: v1$/apiVersion: apps\/v1/' \
  /opt/webapp-broken/templates/deployment.yaml
```

Validare:

```bash
helm lint /opt/webapp-broken
helm template webapp-broken /opt/webapp-broken
```

Installare:

```bash
helm install webapp-broken \
  /opt/webapp-broken \
  -n frontend \
  --create-namespace
```

</details>

---

## MOCK-1 — Build, save e deploy

### Preparazione ambiente

Creare la directory di lavoro:

```bash
mkdir -p /opt/mock-app
cd /opt/mock-app
```

Creare il file `Dockerfile`:

```bash
cat > Dockerfile <<'EOF'
FROM nginx:1.27-alpine

COPY index.html /usr/share/nginx/html/index.html

EXPOSE 80
EOF
```

Creare il file `index.html`:

```bash
cat > index.html <<'EOF'
<!DOCTYPE html>
<html lang="it">
<head>
  <meta charset="UTF-8">
  <title>CKAD Mock App</title>
</head>
<body>
  <h1>Welcome to the CKAD mixed tools mock!</h1>
  <p>The OCI image was built successfully.</p>
</body>
</html>
EOF
```

Creare il file `deployment.yaml`:

```bash
cat > deployment.yaml <<'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mock-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mock-app
  template:
    metadata:
      labels:
        app: mock-app
    spec:
      containers:
      - name: mock-app
        image: IMAGE_TO_REPLACE
        imagePullPolicy: Never
        ports:
        - name: http
          containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: mock-app
spec:
  selector:
    app: mock-app
  ports:
  - name: http
    port: 80
    targetPort: http
EOF
```

Verificare i file creati:

```bash
ls -l /opt/mock-app
```

Output atteso:

```text
Dockerfile
deployment.yaml
index.html
```

---

### Task

Eseguire le seguenti operazioni:

1. Costruire l’immagine:

```text
mock-app:v1
```

2. Salvare l’immagine nel file:

```text
/tmp/mock-app.tar
```

3. Verificare che il file sia stato creato.

4. Modificare `deployment.yaml` usando `yq`:

   * impostare l’immagine del container a `mock-app:v1`;
   * impostare il numero di repliche a `2`.

5. Rendere l’immagine disponibile al cluster Kubernetes locale.

6. Applicare il manifest.

7. Verificare lo stato del rollout.

8. Verificare la pagina web usando `curl`.

---

### Validazione

Verificare l’immagine:

```bash
docker images mock-app:v1
```

Oppure, con Podman:

```bash
podman images mock-app:v1
```

Verificare l’archivio:

```bash
ls -lh /tmp/mock-app.tar
```

Verificare le modifiche YAML:

```bash
yq '.spec.template.spec.containers[0].image' deployment.yaml
```

Output atteso:

```text
mock-app:v1
```

```bash
yq '.spec.replicas' deployment.yaml
```

Output atteso:

```text
2
```

Verificare Kubernetes:

```bash
kubectl get deployment,pod,service
```

```bash
kubectl rollout status deployment/mock-app
```

Testare il Service:

```bash
kubectl port-forward service/mock-app 8080:80
```

In un secondo terminale:

```bash
curl http://localhost:8080
```

L’output deve contenere:

```text
Welcome to the CKAD mixed tools mock!
```

---

<details>
<summary>Soluzione Docker</summary>

Spostarsi nella directory:

```bash
cd /opt/mock-app
```

Costruire l’immagine:

```bash
docker build -t mock-app:v1 .
```

Verificare:

```bash
docker images mock-app:v1
```

Salvare l’immagine:

```bash
docker save -o /tmp/mock-app.tar mock-app:v1
```

Verificare l’archivio:

```bash
ls -lh /tmp/mock-app.tar
```

Modificare l’immagine nel Deployment:

```bash
yq -i \
  '.spec.template.spec.containers[0].image = "mock-app:v1"' \
  deployment.yaml
```

Impostare due repliche:

```bash
yq -i '.spec.replicas = 2' deployment.yaml
```

Verificare le modifiche:

```bash
yq '.spec.template.spec.containers[0].image' deployment.yaml
```

```bash
yq '.spec.replicas' deployment.yaml
```

Se il cluster è Kind, caricare l’immagine nel cluster:

```bash
kind load docker-image mock-app:v1
```

Se il cluster è Minikube:

```bash
minikube image load mock-app:v1
```

Applicare il manifest:

```bash
kubectl apply -f deployment.yaml
```

Verificare il rollout:

```bash
kubectl rollout status deployment/mock-app
```

Verificare le risorse:

```bash
kubectl get deployment,pod,service
```

Avviare il port-forward:

```bash
kubectl port-forward service/mock-app 8080:80
```

In un secondo terminale:

```bash
curl http://localhost:8080
```

</details>

---

<details>
<summary>Soluzione Podman</summary>

Spostarsi nella directory:

```bash
cd /opt/mock-app
```

Costruire l’immagine:

```bash
podman build -t mock-app:v1 .
```

Verificare:

```bash
podman images mock-app:v1
```

Salvare l’immagine in formato OCI:

```bash
podman save \
  --format oci-archive \
  -o /tmp/mock-app.tar \
  mock-app:v1
```

Verificare l’archivio:

```bash
ls -lh /tmp/mock-app.tar
```

Modificare l’immagine nel Deployment:

```bash
yq -i \
  '.spec.template.spec.containers[0].image = "mock-app:v1"' \
  deployment.yaml
```

Impostare due repliche:

```bash
yq -i '.spec.replicas = 2' deployment.yaml
```

Verificare:

```bash
yq '.spec.template.spec.containers[0].image' deployment.yaml
```

```bash
yq '.spec.replicas' deployment.yaml
```

Applicare il manifest se l’immagine è disponibile nel runtime Kubernetes:

```bash
kubectl apply -f deployment.yaml
```

Verificare:

```bash
kubectl rollout status deployment/mock-app
```

```bash
kubectl get deployment,pod,service
```

Testare tramite port-forward:

```bash
kubectl port-forward service/mock-app 8080:80
```

In un secondo terminale:

```bash
curl http://localhost:8080
```

> Se Kubernetes non riesce a trovare `mock-app:v1`, l’immagine deve essere importata nel runtime del nodo oppure pubblicata in un registry accessibile dal cluster.

</details>

---

### Cleanup

```bash
kubectl delete -f /opt/mock-app/deployment.yaml
```

Con Docker:

```bash
docker rmi -f mock-app:v1
```

Con Podman:

```bash
podman rmi -f mock-app:v1
```

Rimuovere i file:

```bash
rm -rf /opt/mock-app
rm -f /tmp/mock-app.tar
```

---

## MOCK-2 — Helm e curl

### Preparazione ambiente

Creare la struttura del chart:

```bash
mkdir -p /opt/frontend-chart/templates
cd /opt/frontend-chart
```

Creare il file `Chart.yaml`:

```bash
cat > Chart.yaml <<'EOF'
apiVersion: v2
name: frontend-chart
description: Chart Helm per simulazione CKAD
type: application
version: 0.1.0
appVersion: "1.27"
EOF
```

Creare il file `values.yaml`:

```bash
cat > values.yaml <<'EOF'
replicaCount: 1

image:
  repository: nginx
  tag: "1.27-alpine"
  pullPolicy: IfNotPresent

service:
  type: ClusterIP
  port: 80

resources: {}
EOF
```

Creare il file `templates/_helpers.tpl`:

```bash
cat > templates/_helpers.tpl <<'EOF'
{{- define "frontend-chart.name" -}}
{{- default .Chart.Name .Values.nameOverride | trunc 63 | trimSuffix "-" }}
{{- end }}

{{- define "frontend-chart.fullname" -}}
{{- printf "%s-%s" .Release.Name (include "frontend-chart.name" .) | trunc 63 | trimSuffix "-" }}
{{- end }}

{{- define "frontend-chart.labels" -}}
app.kubernetes.io/name: {{ include "frontend-chart.name" . }}
app.kubernetes.io/instance: {{ .Release.Name }}
app.kubernetes.io/managed-by: {{ .Release.Service }}
{{- end }}

{{- define "frontend-chart.selectorLabels" -}}
app.kubernetes.io/name: {{ include "frontend-chart.name" . }}
app.kubernetes.io/instance: {{ .Release.Name }}
{{- end }}
EOF
```

Creare il file `templates/deployment.yaml`:

```bash
cat > templates/deployment.yaml <<'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "frontend-chart.fullname" . }}
  labels:
    {{- include "frontend-chart.labels" . | nindent 4 }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      {{- include "frontend-chart.selectorLabels" . | nindent 6 }}
  template:
    metadata:
      labels:
        {{- include "frontend-chart.selectorLabels" . | nindent 8 }}
    spec:
      containers:
      - name: nginx
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
        imagePullPolicy: {{ .Values.image.pullPolicy }}
        ports:
        - name: http
          containerPort: 80
        readinessProbe:
          httpGet:
            path: /
            port: http
        resources:
          {{- toYaml .Values.resources | nindent 10 }}
EOF
```

Creare il file `templates/service.yaml`:

```bash
cat > templates/service.yaml <<'EOF'
apiVersion: v1
kind: Service
metadata:
  name: {{ include "frontend-chart.fullname" . }}
  labels:
    {{- include "frontend-chart.labels" . | nindent 4 }}
spec:
  type: {{ .Values.service.type }}
  selector:
    {{- include "frontend-chart.selectorLabels" . | nindent 4 }}
  ports:
  - name: http
    port: {{ .Values.service.port }}
    targetPort: http
    protocol: TCP
EOF
```

Verificare la struttura:

```bash
find /opt/frontend-chart -type f
```

Output atteso:

```text
/opt/frontend-chart/Chart.yaml
/opt/frontend-chart/values.yaml
/opt/frontend-chart/templates/_helpers.tpl
/opt/frontend-chart/templates/deployment.yaml
/opt/frontend-chart/templates/service.yaml
```

---

### Task

Eseguire le seguenti operazioni:

1. Validare il chart.

2. Eseguire il rendering locale dei manifest.

3. Installare il chart con:

   * release: `frontend`;
   * namespace: `ckad-tools`;
   * creazione automatica del namespace;
   * `service.type=NodePort`.

4. Verificare le risorse create.

5. Recuperare dinamicamente il nome del Service.

6. Recuperare dinamicamente il NodePort.

7. Testare l’applicazione usando `curl`.

8. Mostrare stato e cronologia della release.

---

### Validazione

Validare il chart:

```bash
helm lint /opt/frontend-chart
```

Elencare la release:

```bash
helm list -n ckad-tools
```

Verificare lo stato:

```bash
helm status frontend -n ckad-tools
```

Verificare la cronologia:

```bash
helm history frontend -n ckad-tools
```

Verificare le risorse:

```bash
kubectl get all -n ckad-tools
```

Verificare che il Service sia di tipo `NodePort`:

```bash
kubectl get service -n ckad-tools
```

---

<details>
<summary>Soluzione</summary>

Spostarsi nella directory:

```bash
cd /opt
```

Validare il chart:

```bash
helm lint frontend-chart
```

Eseguire il rendering locale:

```bash
helm template frontend frontend-chart \
  -n ckad-tools \
  --set service.type=NodePort
```

Installare il chart:

```bash
helm install frontend frontend-chart \
  -n ckad-tools \
  --create-namespace \
  --set service.type=NodePort
```

Verificare la release:

```bash
helm list -n ckad-tools
```

Verificare lo stato:

```bash
helm status frontend -n ckad-tools
```

Verificare la cronologia:

```bash
helm history frontend -n ckad-tools
```

Verificare le risorse Kubernetes:

```bash
kubectl get all -n ckad-tools
```

Recuperare dinamicamente il nome del Service:

```bash
SERVICE_NAME=$(kubectl get service \
  -n ckad-tools \
  -l app.kubernetes.io/instance=frontend \
  -o jsonpath='{.items[0].metadata.name}')
```

Visualizzare il nome:

```bash
echo "$SERVICE_NAME"
```

Recuperare il NodePort:

```bash
NODE_PORT=$(kubectl get service "$SERVICE_NAME" \
  -n ckad-tools \
  -o jsonpath='{.spec.ports[0].nodePort}')
```

Visualizzare il NodePort:

```bash
echo "$NODE_PORT"
```

Provare tramite `localhost`:

```bash
curl "http://localhost:${NODE_PORT}"
```

Se il NodePort non è raggiungibile tramite `localhost`, recuperare l’IP interno del nodo:

```bash
NODE_IP=$(kubectl get nodes \
  -o jsonpath='{.items[0].status.addresses[?(@.type=="InternalIP")].address}')
```

Visualizzare l’IP:

```bash
echo "$NODE_IP"
```

Testare:

```bash
curl "http://${NODE_IP}:${NODE_PORT}"
```

Alternativa con port-forward:

```bash
kubectl port-forward \
  -n ckad-tools \
  service/"$SERVICE_NAME" \
  8080:80
```

In un secondo terminale:

```bash
curl http://localhost:8080
```

</details>

---

### Cleanup

Disinstallare la release:

```bash
helm uninstall frontend -n ckad-tools
```

Eliminare il namespace:

```bash
kubectl delete namespace ckad-tools
```

Rimuovere il chart:

```bash
rm -rf /opt/frontend-chart
```

---


---
