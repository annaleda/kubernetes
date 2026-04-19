* [ Home ](../readme.md)   | [ Teoria ](../arguments.md)      | [ Home Other Exercises ](../exercises/o_exercises.md)

---

`helm` è il package manager di Kubernetes.

- `release` = istanza installata di un chart
- `chart` = pacchetto Helm
- `values` = configurazione del chart
- `namespace` = namespace dove installi la release

---

## Repository

Aggiungere repository:

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
```

Aggiornare indice repository:

```bash
helm repo update
```

Vedere repository configurati:

```bash
helm repo list
```

Cercare chart nei repository:

```bash
helm search repo nginx
helm search repo bitnami/nginx
```

Cercare una versione specifica:

```bash
helm search repo bitnami/nginx --versions
```

Rimuovere un repository:

```bash
helm repo remove bitnami
```

---

## Informazioni sui chart

Mostrare informazioni di base:

```bash
helm show chart bitnami/nginx
```

Mostrare README / descrizione:

```bash
helm show readme bitnami/nginx
```

Mostrare values disponibili:

```bash
helm show values bitnami/nginx
helm show values ./webapp-chart
```

Mostrare tutti i file del chart:

```bash
helm show all bitnami/nginx
```

Scaricare chart localmente:

```bash
helm pull bitnami/nginx
```

Scaricare ed estrarre chart:

```bash
helm pull bitnami/nginx --untar
```

Scaricare una versione specifica:

```bash
helm pull bitnami/nginx --version 22.5.4 --untar
```

---

## Creazione chart

Creare un chart base:

```bash
helm create webapp-chart
```

Struttura utile:

```bash
webapp-chart/
  Chart.yaml
  values.yaml
  charts/
  templates/
```

Lint chart locale:

```bash
helm lint ./webapp-chart
helm lint ./broken-chart
```

Pacchettizzare chart:

```bash
helm package ./webapp-chart
```

Pacchettizzare con versione specifica:

```bash
helm package ./webapp-chart --version 0.2.0
```

---

## Installazione

Installare un chart da repository:

```bash
helm install web-release bitnami/nginx -n helm-test --create-namespace
```

Installare chart locale:

```bash
helm install webapp-release ./webapp-chart -n helm-test
```

Installare una versione specifica del chart:

```bash
helm install web-release bitnami/nginx --version 22.5.4 -n helm-test
```

Installare aspettando che le risorse siano pronte:

```bash
helm install web-release bitnami/nginx -n helm-test --wait
```

Installare con timeout:

```bash
helm install web-release bitnami/nginx -n helm-test --wait --timeout 5m
```

Installare e creare output verboso di debug:

```bash
helm install web-release bitnami/nginx -n helm-test --debug
```

---

## Override dei values

Override con `--set`:

```bash
helm install custom-nginx bitnami/nginx \
  -n helm-test \
  --set replicaCount=3 \
  --set service.type=ClusterIP
```

Override multipli:

```bash
helm install custom-nginx bitnami/nginx \
  -n helm-test \
  --set image.repository=nginx \
  --set image.tag=1.25 \
  --set service.type=NodePort
```

Override da file:

```bash
helm install file-nginx bitnami/nginx -n helm-test -f custom-values.yaml
```

Più file values:

```bash
helm install file-nginx bitnami/nginx \
  -n helm-test \
  -f base-values.yaml \
  -f prod-values.yaml
```

Mescolare file + `--set`:

```bash
helm install file-nginx bitnami/nginx \
  -n helm-test \
  -f custom-values.yaml \
  --set replicaCount=4
```

Usare stringhe esplicite:

```bash
helm install web-release bitnami/nginx \
  -n helm-test \
  --set-string image.tag=1.25
```

Usare array / chiavi annidate:

```bash
helm install demo ./webapp-chart \
  --set image.repository=nginx \
  --set service.type=NodePort
```

---

## Upgrade

Aggiornare una release:

```bash
helm upgrade web-release bitnami/nginx -n helm-test
```

Upgrade con values:

```bash
helm upgrade web-release bitnami/nginx \
  -n helm-test \
  --set replicaCount=4
```

Upgrade con file:

```bash
helm upgrade webapp-custom-values ./webapp-chart \
  -n helm-test \
  -f upgrade-values.yaml
```

Upgrade mantenendo values già presenti:

```bash
helm upgrade web-release bitnami/nginx \
  -n helm-test \
  --reuse-values \
  --set replicaCount=5
```

Upgrade installando se non esiste:

```bash
helm upgrade --install web-release bitnami/nginx -n helm-test --create-namespace
```

Upgrade con attesa:

```bash
helm upgrade web-release bitnami/nginx -n helm-test --wait
```

Upgrade con rollback automatico se fallisce:

```bash
helm upgrade web-release bitnami/nginx -n helm-test --atomic
```

---

## Rollback

Vedere history:

```bash
helm history web-release -n helm-test
```

Rollback alla revisione precedente:

```bash
helm rollback web-release 1 -n helm-test
```

Rollback aspettando il completamento:

```bash
helm rollback web-release 1 -n helm-test --wait
```

Rollback con debug:

```bash
helm rollback web-release 1 -n helm-test --debug
```

---

## Rendering template

Render locale senza installare:

```bash
helm template web-release bitnami/nginx
```

Render chart locale:

```bash
helm template webapp-release ./webapp-chart
```

Render in file:

```bash
helm template nginx bitnami/nginx > nginx-rendered.yaml
```

Render con values file:

```bash
helm template webapp-release ./webapp-chart -f custom-values.yaml
```

Render con `--set`:

```bash
helm template webapp-release ./webapp-chart --set replicaCount=3
```

Render solo una parte del chart:

```bash
helm template webapp-release ./webapp-chart --show-only templates/service.yaml
```

---

## Dry-run e debug

Simulare installazione:

```bash
helm install webapp-dryrun ./webapp-chart -n helm-test --dry-run
```

Simulare installazione con debug:

```bash
helm install webapp-dryrun ./webapp-chart -n helm-test --dry-run --debug
```

Simulare upgrade:

```bash
helm upgrade web-release bitnami/nginx -n helm-test --dry-run
```

Debug chart rotto:

```bash
helm lint ./broken-chart
helm template debug-release ./broken-chart
helm install debug-release ./broken-chart -n helm-test --dry-run --debug
```

---

## Listing e ispezione release

Elencare release nel namespace:

```bash
helm list -n helm-test
helm ls -n helm-test
```

Elencare tutte le release:

```bash
helm ls -A
```

Vedere valori usati da una release:

```bash
helm get values web-release -n helm-test
```

Vedere tutti i values, inclusi default + override:

```bash
helm get values web-release -n helm-test -a
```

Vedere manifest generato da una release:

```bash
helm get manifest web-release -n helm-test
```

Vedere note post-install:

```bash
helm get notes web-release -n helm-test
```

Vedere metadata release:

```bash
helm get metadata web-release -n helm-test
```

Vedere tutto:

```bash
helm get all web-release -n helm-test
```

Controllare stato release:

```bash
helm status web-release -n helm-test
```

---

## Disinstallazione

Rimuovere una release:

```bash
helm uninstall webapp-release -n helm-test
```

Disinstallare mantenendo history:

```bash
helm uninstall webapp-release -n helm-test --keep-history
```

---

## Values file tipico

Esempio `custom-values.yaml`:

```yaml
replicaCount: 2

image:
  repository: nginx
  tag: "1.27"

service:
  type: ClusterIP
```

Installazione con file:

```bash
helm install webapp-custom-values ./webapp-chart \
  -n helm-test \
  -f custom-values.yaml
```

---

## File principali di un chart

`Chart.yaml`

```yaml
apiVersion: v2
name: webapp-chart
description: A Helm chart for Kubernetes
type: application
version: 0.2.0
appVersion: "2.0.0"
```

`values.yaml`

```yaml
replicaCount: 1

image:
  repository: nginx
  tag: "latest"

service:
  type: ClusterIP
  port: 80
```

---

## Comandi utili da ricordare

Creare namespace se non esiste:

```bash
kubectl create namespace helm-test --dry-run=client -o yaml | kubectl apply -f -
```

Controllare Deployment creato da Helm:

```bash
kubectl get deploy -n helm-test
```

Controllare Service creato da Helm:

```bash
kubectl get svc -n helm-test
```

Controllare Pod creati da Helm:

```bash
kubectl get pods -n helm-test
```

---

## Pattern veloci da esame

Installare chart da repo:

```bash
helm install web-release bitnami/nginx -n helm-test --create-namespace
```

Installare chart locale:

```bash
helm install webapp-release ./webapp-chart -n helm-test
```

Installare con `--set`:

```bash
helm install custom-webapp ./webapp-chart -n frontend-apd --set replicaCount=3 --set service.type=NodePort
```

Upgrade con immagine custom:

```bash
helm upgrade webapp-release ./webapp-chart \
  -n helm-test \
  --set image.repository=nginx \
  --set image.tag=1.25
```

Rollback:

```bash
helm rollback webapp-release 1 -n helm-test
```

Render senza installare:

```bash
helm template webapp-release ./webapp-chart
```

Lint chart:

```bash
helm lint ./webapp-chart
```

Dry-run:

```bash
helm install webapp-dryrun ./webapp-chart -n helm-test --dry-run --debug
```

Disinstallare:

```bash
helm uninstall webapp-release -n helm-test
```

---

## Errori comuni

- dimenticare `-n <namespace>`
- usare `helm install` invece di `helm upgrade --install`
- non controllare `values.yaml`
- modificare template senza fare `helm lint`
- non usare `--dry-run --debug` quando il chart fallisce
- confondere `chart version` con `appVersion`
- aspettarsi che `helm template` installi davvero risorse
- fare rollback alla revisione sbagliata

---

## Mini flusso mentale

1. Cercare chart
2. Mostrare values
3. Installare con release name
4. Verificare con `helm ls` e `kubectl get`
5. Fare upgrade
6. Controllare history
7. Fare rollback se serve
8. Usare lint / template / dry-run per debug
