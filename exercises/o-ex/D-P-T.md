- [ Home ](../../readme.md)   | [ Teoria ](../../arguments.md)   | [ Info Exam ](../../doc/ckad_exam_strategy.md)    | [ Home Other Exercises ](../o_exercises.md) |  [ Cheatsheet Docker ](./Docker_CheatSheet_CKAD.md) |  [ Cheatsheet Podman ](./Podman_CheatSheet_CKAD.md)

> Esercizi pratici su immagini OCI, container, YAML, HTTP e Helm.

---

# Docker — 5 esercizi

---

## Docker-1 — Build e verifica immagine

Nella directory corrente sono presenti:
[index.html](https://github.com/user-attachments/files/30087454/index.html)

```
FROM nginx:1.27

COPY index.html /usr/share/nginx/html/index.html

EXPOSE 80
```


* `Dockerfile`
* `index
.html`



Creare l’immagine:

* `webapp:v1`

### Validazione

```sh
docker images
docker inspect webapp:v1
```

---

<details>
<summary>Soluzione</summary>

```sh
docker build -t webapp:v1 .
docker images
docker inspect webapp:v1
```

</details>

---

## Docker-2 — Avvio container con porta

Avviare un container:

* nome: `web`
* immagine: `nginx:1.27`
* modalità: detached
* porta host: `8080`
* porta container: `80`

### Validazione

```sh
docker ps
curl localhost:8080
```

---

<details>
<summary>Soluzione</summary>

```sh
docker run -d \
  --name web \
  -p 8080:80 \
  nginx:1.27

docker ps
curl localhost:8080
```

</details>

---

## Docker-3 — Variabili d’ambiente e inspect

Avviare un container:

* nome: `env-container`
* immagine: `alpine`
* variabile `APP_ENV=production`
* comando: `sleep 3600`

Verificare la variabile dall’interno del container.

---

<details>
<summary>Soluzione</summary>

```sh
docker run -d \
  --name env-container \
  -e APP_ENV=production \
  alpine \
  sleep 3600

docker exec env-container printenv APP_ENV
```

Alternativa:

```sh
docker inspect env-container | grep APP_ENV
```

</details>

---

## Docker-4 — Save, remove e load

Salvare l’immagine:

* `nginx:1.27`

nel file:

* `/tmp/nginx.tar`

Successivamente:

1. eliminare l’immagine;
2. ricaricarla dal file;
3. verificarne la presenza.

---

<details>
<summary>Soluzione</summary>

```sh
docker pull nginx:1.27

docker save -o /tmp/nginx.tar nginx:1.27

docker rmi nginx:1.27

docker load -i /tmp/nginx.tar

docker images
```

Se l’immagine è usata da un container:

```sh
docker rm -f web
docker rmi nginx:1.27
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

Nella directory corrente è presente un `Dockerfile`.

Creare l’immagine:

* `api:v1`

### Validazione

```sh
podman images
```

---

<details>
<summary>Soluzione</summary>

```sh
podman build -t api:v1 .
podman images
```

</details>

---

## Podman-2 — Avvio container

Avviare un container:

* nome: `podman-web`
* immagine: `nginx`
* modalità detached
* porta `8081:80`

### Validazione

```sh
podman ps
curl localhost:8081
```

---

<details>
<summary>Soluzione</summary>

```sh
podman run -d \
  --name podman-web \
  -p 8081:80 \
  nginx

podman ps
curl localhost:8081
```

</details>

---

## Podman-3 — Conteggio container

Contare tutti i container presenti sull’host:

* running;
* stopped;
* exited.

Mostrare anche solo quelli in esecuzione.

---

<details>
<summary>Soluzione</summary>

Tutti i container:

```sh
podman ps -aq | wc -l
```

Solo quelli in esecuzione:

```sh
podman ps -q | wc -l
```

Lista completa:

```sh
podman ps -a
```

</details>

---

## Podman-4 — Salvare immagine OCI

Salvare l’immagine:

* `api:v1`

nel file:

* `/tmp/api.tar`

usando il formato OCI archive.

---

<details>
<summary>Soluzione</summary>

```sh
podman save \
  --format oci-archive \
  -o /tmp/api.tar \
  api:v1
```

Verifica:

```sh
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

Dato il file `pod.yaml`, mostrare:

* nome del Pod;
* namespace;
* immagini di tutti i container.

---

<details>
<summary>Soluzione</summary>

```sh
yq '.metadata.name' pod.yaml

yq '.metadata.namespace' pod.yaml

yq '.spec.containers[].image' pod.yaml
```

</details>

---

## YQ-2 — Cambiare immagine

Dato il file `deployment.yaml`, modificare l’immagine del primo container in:

```text
nginx:1.27
```

La modifica deve essere effettuata direttamente nel file.

### Validazione

```sh
grep -n image deployment.yaml
```

---

<details>
<summary>Soluzione</summary>

```sh
yq -i \
  '.spec.template.spec.containers[0].image = "nginx:1.27"' \
  deployment.yaml
```

Verifica:

```sh
yq '.spec.template.spec.containers[0].image' deployment.yaml
```

</details>

---

## YQ-3 — Cambiare repliche e namespace

Nel file `deployment.yaml`:

* impostare `replicas` a `4`;
* impostare il namespace a `production`.

---

<details>
<summary>Soluzione</summary>

```sh
yq -i '.spec.replicas = 4' deployment.yaml

yq -i '.metadata.namespace = "production"' deployment.yaml
```

Verifica:

```sh
yq '.spec.replicas' deployment.yaml

yq '.metadata.namespace' deployment.yaml
```

</details>

---

## YQ-4 — Aggiungere una variabile d’ambiente

Nel primo container di `pod.yaml`, aggiungere:

* nome: `APP_ENV`
* valore: `production`

---

<details>
<summary>Soluzione</summary>

```sh
yq -i \
  '.spec.containers[0].env += [{"name":"APP_ENV","value":"production"}]' \
  pod.yaml
```

Verifica:

```sh
yq '.spec.containers[0].env' pod.yaml
```

</details>

---

## YQ-5 — Pulire un manifest esportato

Dato un file `pod-export.yaml`, eliminare:

* `.metadata.creationTimestamp`
* `.metadata.resourceVersion`
* `.metadata.uid`
* `.status`

---

<details>
<summary>Soluzione</summary>

```sh
yq -i '
  del(
    .metadata.creationTimestamp,
    .metadata.resourceVersion,
    .metadata.uid,
    .status
  )
' pod-export.yaml
```

Verifica:

```sh
yq pod-export.yaml
```

</details>

---

# curl — 5 esercizi

---

## CURL-1 — Verificare un Service

Nel namespace `frontend` esiste un Service chiamato:

* `web-service`

Avviare un Pod temporaneo e verificare la risposta HTTP del Service.

---

<details>
<summary>Soluzione</summary>

```sh
k run curl-test \
  -n frontend \
  --image=curlimages/curl \
  --rm -it \
  --restart=Never \
  -- curl http://web-service
```

Alternativa con BusyBox:

```sh
k run test \
  -n frontend \
  --image=busybox \
  --rm -it \
  --restart=Never \
  -- wget -qO- http://web-service
```

</details>

---

## CURL-2 — Mostrare status e header

Effettuare una richiesta verso:

```text
http://localhost:8080
```

Mostrare:

* status HTTP;
* header;
* body.

---

<details>
<summary>Soluzione</summary>

```sh
curl -i http://localhost:8080
```

Solo gli header:

```sh
curl -I http://localhost:8080
```

Solo lo status code:

```sh
curl -s -o /dev/null -w '%{http_code}\n' \
  http://localhost:8080
```

</details>

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

Nella directory:

```text
/opt/webapp
```

è presente un chart Helm.

Eseguire:

* validazione sintattica;
* rendering dei manifest senza installare il chart.

---

<details>
<summary>Soluzione</summary>

```sh
helm lint /opt/webapp

helm template webapp /opt/webapp
```

Con namespace:

```sh
helm template webapp /opt/webapp \
  -n frontend
```

</details>

---

## Helm-2 — Installare una release

Installare il chart:

```text
/opt/webapp
```

con:

* release: `webapp`
* namespace: `frontend`
* creazione automatica del namespace.

---

<details>
<summary>Soluzione</summary>

```sh
helm install webapp \
  /opt/webapp \
  -n frontend \
  --create-namespace
```

Verifica:

```sh
helm list -n frontend

helm status webapp -n frontend

k get all -n frontend
```

</details>

---

## Helm-3 — Installare con valori personalizzati

Installare il chart `/opt/webapp` usando:

* release: `webapp-prod`
* namespace: `production`
* file: `/opt/webapp/prod-values.yaml`
* replica count: `3`

Il valore delle repliche deve sovrascrivere quello presente nel file values.

---

<details>
<summary>Soluzione</summary>

```sh
helm install webapp-prod \
  /opt/webapp \
  -n production \
  --create-namespace \
  -f /opt/webapp/prod-values.yaml \
  --set replicaCount=3
```

Verifica:

```sh
helm get values webapp-prod -n production

k get deploy -n production
```

</details>

---

## Helm-4 — Upgrade e rollback

La release:

```text
webapp
```

è installata nel namespace:

```text
frontend
```

Aggiornare il tag dell’immagine a:

```text
1.27
```

Successivamente eseguire il rollback alla revisione precedente.

---

<details>
<summary>Soluzione</summary>

Upgrade:

```sh
helm upgrade webapp \
  /opt/webapp \
  -n frontend \
  --set image.tag=1.27
```

Controllare la cronologia:

```sh
helm history webapp -n frontend
```

Rollback:

```sh
helm rollback webapp 1 -n frontend
```

Verifica:

```sh
helm status webapp -n frontend
```

</details>

---

## Helm-5 — Debug chart non installabile

L’installazione restituisce:

```text
no matches for kind "Deployment" in version "v1"
```

Individuare e correggere il problema.

---

<details>
<summary>Soluzione</summary>

Cercare il Deployment:

```sh
grep -R -n \
  'kind: Deployment\|apiVersion:' \
  /opt/webapp/templates
```

Aprire il file interessato:

```sh
vi /opt/webapp/templates/deployment.yaml
```

Correggere:

```yaml
apiVersion: apps/v1
kind: Deployment
```

Verificare che il selector corrisponda alle label del Pod:

```yaml
spec:
  selector:
    matchLabels:
      app: webapp
  template:
    metadata:
      labels:
        app: webapp
```

Validare:

```sh
helm lint /opt/webapp

helm template webapp /opt/webapp
```

Installare:

```sh
helm install webapp \
  /opt/webapp \
  -n frontend \
  --create-namespace
```

</details>

---

# Simulazione finale — Tool misti

---

## MOCK-1 — Build, save e deploy

Nella directory `/opt/mock-app` è presente un Dockerfile.

Eseguire le seguenti operazioni:

1. costruire l’immagine `mock-app:v1`;
2. salvarla in `/tmp/mock-app.tar`;
3. verificare il file creato;
4. modificare `deployment.yaml` impostando l’immagine a `mock-app:v1`;
5. impostare le repliche a `2`;
6. applicare il Deployment;
7. verificare lo stato del rollout.

---

<details>
<summary>Soluzione Docker</summary>

```sh
cd /opt/mock-app

docker build -t mock-app:v1 .

docker save -o /tmp/mock-app.tar mock-app:v1

ls -lh /tmp/mock-app.tar

yq -i \
  '.spec.template.spec.containers[0].image = "mock-app:v1"' \
  deployment.yaml

yq -i \
  '.spec.replicas = 2' \
  deployment.yaml

k apply -f deployment.yaml

k rollout status deploy/mock-app
```

</details>

<details>
<summary>Soluzione Podman</summary>

```sh
cd /opt/mock-app

podman build -t mock-app:v1 .

podman save \
  --format oci-archive \
  -o /tmp/mock-app.tar \
  mock-app:v1

ls -lh /tmp/mock-app.tar

yq -i \
  '.spec.template.spec.containers[0].image = "mock-app:v1"' \
  deployment.yaml

yq -i \
  '.spec.replicas = 2' \
  deployment.yaml

k apply -f deployment.yaml

k rollout status deploy/mock-app
```

</details>

---

## MOCK-2 — Helm e curl

Nella directory `/opt/frontend-chart` è presente un chart Helm.

Eseguire le seguenti operazioni:

1. validare il chart;
2. installarlo come release `frontend`;
3. usare il namespace `ckad-tools`;
4. creare automaticamente il namespace;
5. impostare `service.type=NodePort`;
6. verificare le risorse create;
7. recuperare il NodePort;
8. testare l’applicazione con `curl`.

---

<details>
<summary>Soluzione</summary>

```sh
helm lint /opt/frontend-chart

helm template frontend \
  /opt/frontend-chart \
  -n ckad-tools \
  --set service.type=NodePort

helm install frontend \
  /opt/frontend-chart \
  -n ckad-tools \
  --create-namespace \
  --set service.type=NodePort

k get all -n ckad-tools

k get svc -n ckad-tools

NODE_PORT=$(k get svc \
  -n ckad-tools \
  -o jsonpath='{.items[0].spec.ports[0].nodePort}')

echo "$NODE_PORT"

curl http://localhost:$NODE_PORT
```

In alcuni ambienti il NodePort non è raggiungibile tramite `localhost`. In quel caso recuperare l’indirizzo del nodo:

```sh
k get nodes -o wide
```

e usare:

```sh
curl http://NODE_IP:$NODE_PORT
```

</details>

---
