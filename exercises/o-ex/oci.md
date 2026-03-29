- [ Home ](../../readme.md)   | [ Teoria ](../../arguments.md)   | [ Info Exam ](../../doc/ckad_exam_strategy.md)    | [ Home Other Exercises ](../o_exercises.md)   
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
