- [ Home ](../../readme.md)   | [ Teoria ](../../arguments.md)   | [ Info Exam ](../../doc/ckad_exam_strategy.md)    | [ Home Other Exercises ](../o_exercises.md) |  [ Cheatsheet Docker ](./Docker_CheatSheet_CKAD.md) |  [ Cheatsheet Podman ](./Podman_CheatSheet_CKAD.md)
--- 
### OCI Images (11 esercizi)
---

## OCI-1 — Specificare immagine e tag

- Pod: `nginx-pod`
- Image: `nginx:1.25`

- Obiettivo
  - Usare un tag specifico

- Validazione
  - kubectl describe pod mostra immagine corretta

---

<details>
<summary>Soluzione</summary>

```sh
k run nginx-pod --image=nginx:1.25
k describe pod nginx-pod | grep Image
```

</details>

---

## OCI-2 — imagePullPolicy Always

- Pod: `pull-always-pod`
- Image: `nginx`
- imagePullPolicy: Always

- Obiettivo
  - Forzare il pull dell’immagine

---

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pull-always-pod
spec:
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: Always
```

</details>

---

## OCI-3 — imagePullPolicy IfNotPresent

- Pod: `pull-ifnot-pod`
- Image: `nginx`
- imagePullPolicy: IfNotPresent

---

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pull-ifnot-pod
spec:
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
```

</details>

---

## OCI-4 — imagePullSecrets (Private Registry)

- Secret: `reg-secret`
- Pod: `private-pod`
- Image: `myrepo/private-image`

- Obiettivo
  - Usare credenziali per pull

---

<details>
<summary>Soluzione</summary>

```sh
k create secret docker-registry reg-secret \
  --docker-username=user \
  --docker-password=pass \
  --docker-server=myrepo \
  --dry-run=client -o yaml > secret.yaml
```

```yaml
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
```

</details>

---

## OCI-5 — Cambiare immagine Deployment

- Deployment: `web-deploy`
- Image iniziale: nginx:1.21

- Task
  - Aggiornare a nginx:1.25

---

<details>
<summary>Soluzione</summary>

```sh
k create deploy web-deploy --image=nginx:1.21
k set image deploy web-deploy nginx=nginx:1.25
k rollout status deploy web-deploy
```

</details>

---

## OCI-6 — Rollback immagine

- Deployment: `web-deploy`

- Task
  - Rollback alla versione precedente

---

<details>
<summary>Soluzione</summary>

```sh
k rollout undo deploy web-deploy
k rollout history deploy web-deploy
```

</details>

---

## OCI-7 — Debug immagine sbagliata

- Pod: `bad-image-pod`
- Image: `nginx:wrongtag`

- Obiettivo
  - Identificare errore

- Validazione
  - Stato: ImagePullBackOff

---

<details>
<summary>Soluzione</summary>

```sh
k run bad-image-pod --image=nginx:wrongtag
k describe pod bad-image-pod
```

Fix:

```sh
k delete pod bad-image-pod
k run bad-image-pod --image=nginx:1.25
```

</details>

---

## OCI-8 — Usare digest invece del tag

- Pod: `digest-pod`
- Image: nginx con digest

---

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: digest-pod
spec:
  containers:
  - name: nginx
    image: nginx@sha256:<digest>
```

</details>

---

## OCI-9 — Multi-container con immagini diverse

- Pod: `multi-image-pod`

- Container 1
  - nginx

- Container 2
  - busybox

---

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-image-pod
spec:
  containers:
  - name: nginx
    image: nginx
  - name: busybox
    image: busybox
    command: ["sh","-c","sleep 3600"]
```

</details>

---

## OCI-10 — Override command immagine

- Pod: `override-pod`
- Image: busybox

- Command:
  - echo hello world

---

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: override-pod
spec:
  containers:
  - name: app
    image: busybox
    command: ["sh","-c","echo hello world"]
```

</details>

---

## OCI-11 — CrashLoopBackOff debug

- Pod: `crash-pod`
- Image: busybox
- Command: exit 1

- Obiettivo
  - Analizzare crash

---

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: crash-pod
spec:
  containers:
  - name: app
    image: busybox
    command: ["sh","-c","exit 1"]
```

```sh
k logs crash-pod
k describe pod crash-pod
```

</details>

---

---

## Docker (CKAD) - 10 esercizi

> Esercizi pratici sull'utilizzo di Docker per la gestione delle immagini OCI.

---

## Docker-1 — Build Image

Nella directory corrente è presente un Dockerfile.

- Costruire l'immagine:
  - `myapp:v1`

- Validazione

```sh
docker images
```

---

<details>
<summary>Soluzione</summary>

```sh
docker build -t myapp:v1 .
```

</details>

---

## Docker-2 — Tag Image

Esiste già l'immagine:

- `myapp:v1`

Creare il tag:

- `registry.local/myapp:v1`

---

<details>
<summary>Soluzione</summary>

```sh
docker tag myapp:v1 registry.local/myapp:v1
```

</details>

---

## Docker-3 — Salvare immagine

Salvare l'immagine:

- `myapp:v1`

nel file

- `/tmp/myapp.tar`

---

<details>
<summary>Soluzione</summary>

```sh
docker save -o /tmp/myapp.tar myapp:v1
```

</details>

---

## Docker-4 — Caricare immagine

Importare

- `/tmp/myapp.tar`

e verificare che l'immagine sia presente.

---

<details>
<summary>Soluzione</summary>

```sh
docker load -i /tmp/myapp.tar
docker images
```

</details>

---

## Docker-5 — Avviare container

Avviare un container:

- nome: `web`
- immagine: `nginx`
- porta: `8080:80`
- modalità detached

---

<details>
<summary>Soluzione</summary>

```sh
docker run -d --name web -p 8080:80 nginx
```

</details>

---

## Docker-6 — Entrare nel container

Entrare nel container

- `web`

ed elencare il contenuto di

- `/usr/share/nginx/html`

---

<details>
<summary>Soluzione</summary>

```sh
docker exec -it web sh
ls /usr/share/nginx/html
```

</details>

---

## Docker-7 — Visualizzare logs

Mostrare i log del container:

- `web`

---

<details>
<summary>Soluzione</summary>

```sh
docker logs web
```

</details>

---

## Docker-8 — Esportare container

Esportare il container

- `web`

nel file

- `/tmp/web.tar`

---

<details>
<summary>Soluzione</summary>

```sh
docker export web > /tmp/web.tar
```

</details>

---

## Docker-9 — Commit

Creare una nuova immagine

- `web:v2`

partendo dal container

- `web`

---

<details>
<summary>Soluzione</summary>

```sh
docker commit web web:v2
```

</details>

---

## Docker-10 — Cleanup

Eliminare:

- tutti i container
- tutte le immagini

---

<details>
<summary>Soluzione</summary>

```sh
docker rm -f $(docker ps -aq)

docker rmi -f $(docker images -q)
```

</details>

---

---
