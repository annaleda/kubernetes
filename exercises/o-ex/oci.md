- [ Home ](../../readme.md)   | [ Teoria ](../../arguments.md)   | [ Info Exam ](../../doc/ckad_exam_strategy.md)    | [ Home Other Exercises ](../o_exercises.md) |  [ Cheatsheet Docker ](./Docker_CheatSheet_CKAD.md) |  [ Cheatsheet Podman ](./Podman_CheatSheet_CKAD.md)

---
# OCI Images — 11 esercizi

> Esercizi autonomi su immagini OCI in Kubernetes. Ogni esercizio contiene preparazione, task, validazione, soluzione e cleanup.

---

## OCI-1 — Specificare immagine e tag

### Preparazione ambiente

```bash
k delete pod nginx-pod --ignore-not-found
```

### Task

Creare un Pod con:

- nome: `nginx-pod`
- immagine: `nginx:1.25`

### Validazione

```bash
k get pod nginx-pod
k get pod nginx-pod \
  -o jsonpath='{.spec.containers[0].image}{"\n"}'
```

Output atteso:

```text
nginx:1.25
```

<details>
<summary>Soluzione</summary>

```bash
k run nginx-pod --image=nginx:1.25
```

</details>

### Cleanup

```bash
k delete pod nginx-pod
```

---

## OCI-2 — imagePullPolicy Always

### Preparazione ambiente

```bash
k delete pod pull-always-pod --ignore-not-found

cat > pull-always.yaml <<'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: pull-always-pod
spec:
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
EOF
```

### Task

Modificare `pull-always.yaml` affinché il container usi:

```text
imagePullPolicy: Always
```

Applicare il manifest.

### Validazione

```bash
k get pod pull-always-pod \
  -o jsonpath='{.spec.containers[0].imagePullPolicy}{"\n"}'
```

Output atteso:

```text
Always
```

<details>
<summary>Soluzione</summary>

```bash
sed -i 's/imagePullPolicy: IfNotPresent/imagePullPolicy: Always/' pull-always.yaml
k apply -f pull-always.yaml
```

</details>

### Cleanup

```bash
k delete -f pull-always.yaml
rm -f pull-always.yaml
```

---

## OCI-3 — imagePullPolicy IfNotPresent

### Preparazione ambiente

```bash
k delete pod pull-ifnot-pod --ignore-not-found

cat > pull-ifnot.yaml <<'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: pull-ifnot-pod
spec:
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: Always
EOF
```

### Task

Modificare il manifest per usare:

```text
imagePullPolicy: IfNotPresent
```

Applicare il manifest.

### Validazione

```bash
k get pod pull-ifnot-pod \
  -o jsonpath='{.spec.containers[0].imagePullPolicy}{"\n"}'
```

Output atteso:

```text
IfNotPresent
```

<details>
<summary>Soluzione</summary>

```bash
sed -i 's/imagePullPolicy: Always/imagePullPolicy: IfNotPresent/' pull-ifnot.yaml
k apply -f pull-ifnot.yaml
```

</details>

### Cleanup

```bash
k delete -f pull-ifnot.yaml
rm -f pull-ifnot.yaml
```

---

## OCI-4 — imagePullSecrets

### Preparazione ambiente

```bash
k delete pod private-pod --ignore-not-found
k delete secret reg-secret --ignore-not-found

cat > private-pod.yaml <<'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: private-pod
spec:
  containers:
  - name: app
    image: myrepo/private-image
EOF
```

### Task

1. Creare un Secret di tipo Docker Registry chiamato `reg-secret`.
2. Configurare `private-pod.yaml` affinché il Pod usi il Secret tramite `imagePullSecrets`.
3. Applicare le risorse.

Credenziali di esercizio:

- server: `myrepo`
- username: `user`
- password: `pass`

### Validazione

```bash
k get secret reg-secret
k get pod private-pod \
  -o jsonpath='{.spec.imagePullSecrets[0].name}{"\n"}'
```

Output atteso:

```text
reg-secret
```

> Il Pod può restare in `ImagePullBackOff` perché il registry è fittizio. L’obiettivo è verificare la configurazione.

<details>
<summary>Soluzione</summary>

```bash
k create secret docker-registry reg-secret \
  --docker-username=user \
  --docker-password=pass \
  --docker-server=myrepo

cat > private-pod.yaml <<'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: private-pod
spec:
  containers:
  - name: app
    image: myrepo/private-image
  imagePullSecrets:
  - name: reg-secret
EOF

k apply -f private-pod.yaml
```

</details>

### Cleanup

```bash
k delete pod private-pod --ignore-not-found
k delete secret reg-secret --ignore-not-found
rm -f private-pod.yaml
```

---

## OCI-5 — Cambiare immagine Deployment

### Preparazione ambiente

```bash
k delete deployment web-deploy --ignore-not-found

k create deployment web-deploy \
  --image=nginx:1.21

k rollout status deployment/web-deploy
```

### Task

Aggiornare l’immagine del Deployment `web-deploy` a:

```text
nginx:1.25
```

### Validazione

```bash
k get deployment web-deploy \
  -o jsonpath='{.spec.template.spec.containers[0].image}{"\n"}'

k rollout status deployment/web-deploy
```

Output atteso:

```text
nginx:1.25
```

<details>
<summary>Soluzione</summary>

```bash
k set image deployment/web-deploy \
  nginx=nginx:1.25

k rollout status deployment/web-deploy
```

</details>

### Cleanup

```bash
k delete deployment web-deploy
```

---

## OCI-6 — Rollback immagine

### Preparazione ambiente

```bash
k delete deployment web-deploy --ignore-not-found

k create deployment web-deploy \
  --image=nginx:1.21

k rollout status deployment/web-deploy

k set image deployment/web-deploy \
  nginx=nginx:1.25

k rollout status deployment/web-deploy
```

### Task

Eseguire il rollback del Deployment `web-deploy` alla revisione precedente.

### Validazione

```bash
k rollout history deployment/web-deploy

k get deployment web-deploy \
  -o jsonpath='{.spec.template.spec.containers[0].image}{"\n"}'
```

Output atteso dopo il rollback:

```text
nginx:1.21
```

<details>
<summary>Soluzione</summary>

```bash
k rollout undo deployment/web-deploy
k rollout status deployment/web-deploy
k rollout history deployment/web-deploy
```

</details>

### Cleanup

```bash
k delete deployment web-deploy
```

---

## OCI-7 — Debug immagine sbagliata

### Preparazione ambiente

```bash
k delete pod bad-image-pod --ignore-not-found

k run bad-image-pod \
  --image=nginx:wrongtag
```

Attendere qualche secondo:

```bash
k get pod bad-image-pod
```

### Task

1. Individuare il motivo per cui il Pod non parte.
2. Correggere il Pod mantenendo lo stesso nome.
3. Usare l’immagine `nginx:1.25`.

### Validazione

```bash
k get pod bad-image-pod
k get pod bad-image-pod \
  -o jsonpath='{.spec.containers[0].image}{"\n"}'
```

Output atteso:

```text
nginx:1.25
```

<details>
<summary>Soluzione</summary>

Analizzare:

```bash
k describe pod bad-image-pod
```

Correggere:

```bash
k delete pod bad-image-pod
k run bad-image-pod --image=nginx:1.25
```

</details>

### Cleanup

```bash
k delete pod bad-image-pod
```

---

## OCI-8 — Usare digest invece del tag

### Preparazione ambiente

Scaricare l’immagine con Docker o Podman per recuperare un digest reale:

```bash
docker pull nginx:1.25
docker inspect nginx:1.25 \
  --format '{{index .RepoDigests 0}}'
```

Oppure:

```bash
podman pull nginx:1.25
podman image inspect nginx:1.25 \
  --format '{{index .Digest}}'
```

Eliminare un eventuale Pod precedente:

```bash
k delete pod digest-pod --ignore-not-found
```

### Task

Creare `digest-pod.yaml` usando l’immagine NGINX tramite digest:

```text
nginx@sha256:<digest-reale>
```

Applicare il manifest.

### Validazione

```bash
k get pod digest-pod \
  -o jsonpath='{.spec.containers[0].image}{"\n"}'
```

L’output deve contenere `nginx@sha256:`.

<details>
<summary>Soluzione</summary>

Esempio:

```bash
IMAGE_DIGEST=$(docker inspect nginx:1.25 \
  --format '{{index .RepoDigests 0}}')

cat > digest-pod.yaml <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: digest-pod
spec:
  containers:
  - name: nginx
    image: ${IMAGE_DIGEST}
EOF

k apply -f digest-pod.yaml
```

</details>

### Cleanup

```bash
k delete pod digest-pod --ignore-not-found
rm -f digest-pod.yaml
```

---

## OCI-9 — Multi-container con immagini diverse

### Preparazione ambiente

```bash
k delete pod multi-image-pod --ignore-not-found
```

### Task

Creare un Pod chiamato `multi-image-pod` con due container:

1. `nginx`
   - immagine: `nginx:1.25`
2. `busybox`
   - immagine: `busybox:1.36`
   - comando: `sleep 3600`

### Validazione

```bash
k get pod multi-image-pod \
  -o jsonpath='{range .spec.containers[*]}{.name}{" -> "}{.image}{"\n"}{end}'
```

Output atteso:

```text
nginx -> nginx:1.25
busybox -> busybox:1.36
```

<details>
<summary>Soluzione</summary>

```bash
cat > multi-image-pod.yaml <<'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: multi-image-pod
spec:
  containers:
  - name: nginx
    image: nginx:1.25
  - name: busybox
    image: busybox:1.36
    command: ["sh", "-c", "sleep 3600"]
EOF

k apply -f multi-image-pod.yaml
```

</details>

### Cleanup

```bash
k delete -f multi-image-pod.yaml
rm -f multi-image-pod.yaml
```

---

## OCI-10 — Override command immagine

### Preparazione ambiente

```bash
k delete pod override-pod --ignore-not-found
```

### Task

Creare un Pod:

- nome: `override-pod`
- immagine: `busybox:1.36`
- comando: `echo hello world`

Il Pod deve terminare con stato `Completed`.

### Validazione

```bash
k get pod override-pod
k logs override-pod
```

Output atteso:

```text
hello world
```

<details>
<summary>Soluzione</summary>

```bash
k run override-pod \
  --image=busybox:1.36 \
  --restart=Never \
  --command -- \
  sh -c 'echo hello world'
```

</details>

### Cleanup

```bash
k delete pod override-pod
```

---

## OCI-11 — CrashLoopBackOff debug

### Preparazione ambiente

```bash
k delete pod crash-pod --ignore-not-found

cat > crash-pod.yaml <<'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: crash-pod
spec:
  containers:
  - name: app
    image: busybox:1.36
    command: ["sh", "-c", "echo starting; exit 1"]
EOF

k apply -f crash-pod.yaml
```

Attendere che il Pod inizi a riavviarsi:

```bash
k get pod crash-pod -w
```

Interrompere con `Ctrl+C`.

### Task

1. Individuare il motivo del `CrashLoopBackOff`.
2. Visualizzare i log dell’ultima esecuzione.
3. Correggere il Pod affinché resti in esecuzione con `sleep 3600`.

### Validazione

```bash
k get pod crash-pod
k get pod crash-pod \
  -o jsonpath='{.spec.containers[0].command}{"\n"}'
```

Il Pod deve risultare `Running`.

<details>
<summary>Soluzione</summary>

Analizzare:

```bash
k logs crash-pod
k logs crash-pod --previous
k describe pod crash-pod
```

Correggere:

```bash
k delete pod crash-pod

cat > crash-pod.yaml <<'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: crash-pod
spec:
  containers:
  - name: app
    image: busybox:1.36
    command: ["sh", "-c", "sleep 3600"]
EOF

k apply -f crash-pod.yaml
```

</details>

### Cleanup

```bash
k delete -f crash-pod.yaml
rm -f crash-pod.yaml
```

---
# Docker (CKAD) — 10 esercizi

> Esercizi autonomi sull’utilizzo di Docker per immagini OCI e container.

---

## Docker-1 — Build Image

### Preparazione ambiente

```bash
mkdir -p /tmp/docker-1
cd /tmp/docker-1
```

Creare `Dockerfile`:

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
  <title>Docker CKAD</title>
</head>
<body>
  <h1>Welcome to Docker CKAD!</h1>
</body>
</html>
EOF
```

### Task

Costruire l’immagine:

```text
myapp:v1
```

### Validazione

```bash
docker images myapp:v1
docker inspect myapp:v1
```

Test facoltativo:

```bash
docker run --rm -d \
  --name myapp-test \
  -p 8080:80 \
  myapp:v1

curl http://localhost:8080

docker rm -f myapp-test
```

<details>
<summary>Soluzione</summary>

```bash
cd /tmp/docker-1
docker build -t myapp:v1 .
```

</details>

### Cleanup

```bash
docker rmi -f myapp:v1
rm -rf /tmp/docker-1
```

---

## Docker-2 — Tag Image

### Preparazione ambiente

```bash
docker rmi -f registry.local/myapp:v1 2>/dev/null || true
docker rmi -f myapp:v1 2>/dev/null || true

docker pull alpine:3.20
docker tag alpine:3.20 myapp:v1
```

### Task

Creare il tag:

```text
registry.local/myapp:v1
```

partendo da:

```text
myapp:v1
```

### Validazione

```bash
docker images registry.local/myapp:v1
docker inspect registry.local/myapp:v1
```

<details>
<summary>Soluzione</summary>

```bash
docker tag myapp:v1 registry.local/myapp:v1
```

</details>

### Cleanup

```bash
docker rmi -f registry.local/myapp:v1 myapp:v1
```

---

## Docker-3 — Salvare immagine

### Preparazione ambiente

```bash
docker rmi -f myapp:v1 2>/dev/null || true
docker pull alpine:3.20
docker tag alpine:3.20 myapp:v1
rm -f /tmp/myapp.tar
```

### Task

Salvare l’immagine `myapp:v1` nel file:

```text
/tmp/myapp.tar
```

### Validazione

```bash
ls -lh /tmp/myapp.tar
tar -tf /tmp/myapp.tar | head
```

<details>
<summary>Soluzione</summary>

```bash
docker save -o /tmp/myapp.tar myapp:v1
```

</details>

### Cleanup

```bash
docker rmi -f myapp:v1
rm -f /tmp/myapp.tar
```

---

## Docker-4 — Caricare immagine

### Preparazione ambiente

```bash
docker rmi -f myapp:v1 2>/dev/null || true
docker pull alpine:3.20
docker tag alpine:3.20 myapp:v1
docker save -o /tmp/myapp.tar myapp:v1
docker rmi -f myapp:v1
```

### Task

Caricare l’immagine contenuta in:

```text
/tmp/myapp.tar
```

### Validazione

```bash
docker images myapp:v1
docker inspect myapp:v1
```

<details>
<summary>Soluzione</summary>

```bash
docker load -i /tmp/myapp.tar
```

</details>

### Cleanup

```bash
docker rmi -f myapp:v1
rm -f /tmp/myapp.tar
```

---

## Docker-5 — Avviare container

### Preparazione ambiente

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

<details>
<summary>Soluzione</summary>

```bash
docker run -d \
  --name web \
  -p 8080:80 \
  nginx:1.27
```

</details>

### Cleanup

```bash
docker rm -f web
```

---

## Docker-6 — Entrare nel container

### Preparazione ambiente

```bash
docker rm -f web 2>/dev/null || true

docker run -d \
  --name web \
  nginx:1.27
```

### Task

1. Entrare nel container `web`.
2. Elencare il contenuto di:

```text
/usr/share/nginx/html
```

3. Uscire dal container.

### Validazione

```bash
docker exec web ls -l /usr/share/nginx/html
```

<details>
<summary>Soluzione</summary>

Interattiva:

```bash
docker exec -it web sh
ls -l /usr/share/nginx/html
exit
```

Oppure direttamente:

```bash
docker exec web ls -l /usr/share/nginx/html
```

</details>

### Cleanup

```bash
docker rm -f web
```

---

## Docker-7 — Visualizzare logs

### Preparazione ambiente

```bash
docker rm -f web 2>/dev/null || true

docker run -d \
  --name web \
  -p 8080:80 \
  nginx:1.27

curl http://localhost:8080 >/dev/null
curl http://localhost:8080/not-found >/dev/null
```

### Task

1. Mostrare tutti i log del container `web`.
2. Mostrare solo le ultime cinque righe.
3. Seguire i log in tempo reale.

### Validazione

I log devono contenere richieste HTTP verso `/` e `/not-found`.

<details>
<summary>Soluzione</summary>

```bash
docker logs web

docker logs --tail 5 web

docker logs -f web
```

Interrompere `docker logs -f` con `Ctrl+C`.

</details>

### Cleanup

```bash
docker rm -f web
```

---

## Docker-8 — Esportare container

### Preparazione ambiente

```bash
docker rm -f web 2>/dev/null || true
rm -f /tmp/web.tar

docker run -d \
  --name web \
  nginx:1.27
```

### Task

Esportare il filesystem del container `web` nel file:

```text
/tmp/web.tar
```

### Validazione

```bash
ls -lh /tmp/web.tar
tar -tf /tmp/web.tar | head
```

<details>
<summary>Soluzione</summary>

```bash
docker export web > /tmp/web.tar
```

Oppure:

```bash
docker export -o /tmp/web.tar web
```

</details>

### Cleanup

```bash
docker rm -f web
rm -f /tmp/web.tar
```

---

## Docker-9 — Commit

### Preparazione ambiente

```bash
docker rm -f web 2>/dev/null || true
docker rmi -f web:v2 2>/dev/null || true

docker run -d \
  --name web \
  -p 8080:80 \
  nginx:1.27
```

Modificare la pagina nel container:

```bash
docker exec web \
  sh -c "printf '%s\n' 'CKAD committed image' > /usr/share/nginx/html/index.html"
```

### Task

Creare l’immagine:

```text
web:v2
```

partendo dal container:

```text
web
```

### Validazione

```bash
docker images web:v2
docker inspect web:v2
```

Verifica funzionale:

```bash
docker rm -f web

docker run --rm -d \
  --name web-v2-test \
  -p 8081:80 \
  web:v2

curl http://localhost:8081

docker rm -f web-v2-test
```

Output atteso:

```text
CKAD committed image
```

<details>
<summary>Soluzione</summary>

```bash
docker commit web web:v2
```

</details>

### Cleanup

```bash
docker rm -f web web-v2-test 2>/dev/null || true
docker rmi -f web:v2
```

---

## Docker-10 — Cleanup

> Attenzione: questo esercizio elimina tutti i container e tutte le immagini Docker presenti sull’host. Usarlo solo in un ambiente di laboratorio dedicato.

### Preparazione ambiente

Creare risorse di test:

```bash
docker rm -f cleanup-running cleanup-stopped 2>/dev/null || true

docker pull alpine:3.20
docker pull busybox:1.36

docker run -d \
  --name cleanup-running \
  alpine:3.20 \
  sleep 3600

docker create \
  --name cleanup-stopped \
  busybox:1.36 \
  sleep 3600
```

Verificare lo stato iniziale:

```bash
docker ps -a
docker images
```

### Task

Eliminare:

- tutti i container, inclusi quelli arrestati;
- tutte le immagini.

### Validazione

```bash
docker ps -a
docker images
```

Entrambi gli elenchi devono risultare vuoti.

<details>
<summary>Soluzione</summary>

```bash
CONTAINERS=$(docker ps -aq)
[ -z "$CONTAINERS" ] || docker rm -f $CONTAINERS

IMAGES=$(docker images -q)
[ -z "$IMAGES" ] || docker rmi -f $IMAGES
```

Alternativa:

```bash
docker system prune -a -f
```

> `docker system prune -a -f` non elimina i volumi senza l’opzione `--volumes`.

</details>

---

---
