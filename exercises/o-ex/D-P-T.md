- [ Home ](../../readme.md)   | [ Teoria ](../../arguments.md)   | [ Info Exam ](../../doc/ckad_exam_strategy.md)    | [ Home Other Exercises ](../o_exercises.md) |  [ Cheatsheet Docker ](./Docker_CheatSheet_CKAD.md) |  [ Cheatsheet Podman ](./Podman_CheatSheet_CKAD.md)

> Esercizi pratici su immagini OCI, container, YAML, HTTP e Helm.

---

### Docker — 5 esercizi

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

### Podman — 5 esercizi

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

### yq — 5 esercizi

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

### curl — 5 esercizi

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

### Helm — 5 esercizi

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

### Simulazione finale — Tool misti

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
