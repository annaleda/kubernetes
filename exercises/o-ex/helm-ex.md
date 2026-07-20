* [ Home ](../../readme.md)   | [ Teoria ](../../arguments.md)   | [ Info Exam ](../../doc/ckad_exam_strategy.md)    | [ Home Other Exercises ](../o_exercises.md) | [ Helm_Cheatsheet ](../../doc/helm_cheatsheet.md)

---
### Helm (24 esercizi)

## HELM-1 — Installazione Chart

### Preparazione

```bash
kubectl create namespace helm-test --dry-run=client -o yaml | kubectl apply -f -
helm repo add bitnami https://charts.bitnami.com/bitnami 2>/dev/null || true
helm repo update
helm uninstall web-release -n helm-test 2>/dev/null || true
```

Namespace: `helm-test`

Installare chart nginx

- Specifiche
  - Release name: `web-release`
  - Chart: `bitnami/nginx`

- Validazione
  - helm list -n helm-test
---
<details>
<summary>Soluzione</summary>
  
```
 k create ns helm-test
 helm install web-release bitnami/nginx -n helm-test

helm list -n helm-test

```
</details>


### Cleanup

```bash
helm uninstall web-release -n helm-test 2>/dev/null || true
kubectl delete namespace helm-test --ignore-not-found
```


---

## HELM-2 — Override Values via CLI

### Preparazione

```bash
kubectl create namespace helm-test --dry-run=client -o yaml | kubectl apply -f -
helm repo add bitnami https://charts.bitnami.com/bitnami 2>/dev/null || true
helm repo update
helm uninstall custom-nginx -n helm-test 2>/dev/null || true
```

Installare release `custom-nginx`

- Specifiche
  - ReplicaCount: 3
  - Service type: ClusterIP
  - Usare flag --set

- Validazione
  - kubectl get deployment mostra 3 repliche

---
<details>
<summary>Soluzione</summary>
  
```
helm install custom-nginx bitnami/nginx --set replicaCount=3 --set service.type=ClusterIP -n helm-test

k get deploy custom-nginx -n helm-test
```
</details>


### Cleanup

```bash
helm uninstall custom-nginx -n helm-test 2>/dev/null || true
kubectl delete namespace helm-test --ignore-not-found
```


---

## HELM-3 — Override con values.yaml

### Preparazione

```bash
kubectl create namespace helm-test --dry-run=client -o yaml | kubectl apply -f -
helm repo add bitnami https://charts.bitnami.com/bitnami 2>/dev/null || true
helm repo update
helm uninstall file-nginx -n helm-test 2>/dev/null || true
rm -f custom-values.yaml
```

Creare file `custom-values.yaml`

- Specifiche
  - replicaCount: 2
  - service.type: NodePort
  - Installare release file-nginx usando file

- Validazione
  - Service è NodePort
---
<details>
<summary>Soluzione</summary>
  
```
vi custom-values.yaml

replicaCount: 2
service:
  type: NodePort

helm install file-nginx bitnami/nginx -f custom-values.yaml

k get svc file-nginx

NAME         TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)                      AGE
file-nginx   NodePort   10.96.83.236   <none>        80:31976/TCP,443:30153/TCP   68s


```
</details>


### Cleanup

```bash
helm uninstall file-nginx -n helm-test 2>/dev/null || true
kubectl delete namespace helm-test --ignore-not-found
rm -f custom-values.yaml
```


---

### HELM-4 — Upgrade Release

### Preparazione

```bash
kubectl create namespace helm-test --dry-run=client -o yaml | kubectl apply -f -
helm repo add bitnami https://charts.bitnami.com/bitnami 2>/dev/null || true
helm repo update
helm uninstall web-release -n helm-test 2>/dev/null || true
helm install web-release bitnami/nginx -n helm-test --set replicaCount=1
```

Aggiornare release `web-release`

- Specifiche
  - Cambiare replicaCount a 4

- Validazione
  - helm history web-release
  - Deployment aggiornato

---
<details>
<summary>Soluzione</summary>
  
```
helm install web-release bitnami/nginx -n helm-test --create-namespace

helm list -n helm-test

NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART          APP VERSION
custom-nginx    helm-test       1               2026-03-04 11:06:39.1978066 +0100 CET   deployed        nginx-22.5.4   1.29.5
web-release     helm-test       1               2026-03-04 11:15:35.4563521 +0100 CET   deployed        nginx-22.5.4   1.29.5


helm upgrade web-release bitnami/nginx --set replicaCount=4 -n helm-test
helm history web-release -n helm-test

REVISION        UPDATED                         STATUS          CHART           APP VERSION     DESCRIPTION
1               Wed Mar  4 11:15:35 2026        superseded      nginx-22.5.4    1.29.5          Install complete
2               Wed Mar  4 11:16:47 2026        deployed        nginx-22.5.4    1.29.5          Upgrade complete

```
</details>


### Cleanup

```bash
helm uninstall web-release -n helm-test 2>/dev/null || true
kubectl delete namespace helm-test --ignore-not-found
```


---

### HELM-5 — Rollback Release

### Preparazione

```bash
kubectl create namespace helm-test --dry-run=client -o yaml | kubectl apply -f -
helm repo add bitnami https://charts.bitnami.com/bitnami 2>/dev/null || true
helm repo update
helm uninstall web-release -n helm-test 2>/dev/null || true
helm install web-release bitnami/nginx -n helm-test --set replicaCount=1
helm upgrade web-release bitnami/nginx -n helm-test --set replicaCount=4
```

Eseguire rollback della release `web-release`

Alla revisione precedente

- Validazione
  - helm history mostra nuova revisione rollback
---
<details>
<summary>Soluzione</summary>
  
```
helm history web-release -n helm-test

REVISION        UPDATED                         STATUS          CHART           APP VERSION     DESCRIPTION
1               Wed Mar  4 11:15:35 2026        superseded      nginx-22.5.4    1.29.5          Install complete
2               Wed Mar  4 11:16:47 2026        deployed        nginx-22.5.4    1.29.5          Upgrade complete


helm rollback web-release 1 -n helm-test

helm history web-release -n helm-test

REVISION        UPDATED                         STATUS          CHART           APP VERSION     DESCRIPTION
1               Wed Mar  4 11:15:35 2026        superseded      nginx-22.5.4    1.29.5          Install complete
2               Wed Mar  4 11:16:47 2026        superseded      nginx-22.5.4    1.29.5          Upgrade complete
3               Wed Mar  4 11:19:15 2026        deployed        nginx-22.5.4    1.29.5          Rollback to 1

```
</details>


### Cleanup

```bash
helm uninstall web-release -n helm-test 2>/dev/null || true
kubectl delete namespace helm-test --ignore-not-found
```


---

### HELM-6 — Template Rendering

### Preparazione

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami 2>/dev/null || true
helm repo update
rm -f nginx-rendered.yaml
```

Eseguire render locale del chart `nginx`

- Specifiche
  - Non installare nel cluster
  - Output in file nginx-rendered.yaml

- Validazione
  - File contiene manifest Kubernetes
---
<details>
<summary>Soluzione</summary>
  
```
helm template nginx bitnami/nginx > nginx-rendered.yaml
cat nginx-rendered.yaml

apiVersion: apps/v1
kind: Deployment
...
---
apiVersion: v1
kind: Service
...
```
</details>


### Cleanup

```bash
rm -f nginx-rendered.yaml
```


---

### HELM-7

### Preparazione

```bash
rm -rf webapp-chart
```

Task  
SECTION: APPLICATION DEPLOYMENT (HELM)

Create a simple Helm chart named `webapp-chart`.

---

<details>
<summary>Soluzione</summary>

```bash

helm create webapp-chart
ls webapp-chart
tree webapp-chart
```

</details>


### Cleanup

```bash
rm -rf webapp-chart
```


---

### HELM-8

### Preparazione

```bash
rm -rf webapp-chart
helm create webapp-chart
kubectl create namespace helm-test --dry-run=client -o yaml | kubectl apply -f -
helm uninstall webapp-release -n helm-test 2>/dev/null || true
```

Task  
SECTION: APPLICATION DEPLOYMENT (HELM)


Install a Helm chart from local directory `./webapp-chart` with:

- Release name: `webapp-release`
- Namespace: `helm-test`

Create the namespace if it does not exist.

---

<details>
<summary>Soluzione</summary>

```bash

kubectl create namespace helm-test --dry-run=client -o yaml | kubectl apply -f -
helm install webapp-release ./webapp-chart -n helm-test
helm ls -n helm-test
```

</details>


### Cleanup

```bash
helm uninstall webapp-release -n helm-test 2>/dev/null || true
kubectl delete namespace helm-test --ignore-not-found
rm -rf webapp-chart
```


---

### HELM-9

### Preparazione

```bash
rm -rf webapp-chart
helm create webapp-chart
kubectl create namespace helm-one --dry-run=client -o yaml | kubectl apply -f -
kubectl create namespace helm-two --dry-run=client -o yaml | kubectl apply -f -
helm uninstall release-one -n helm-one 2>/dev/null || true
helm uninstall release-two -n helm-two 2>/dev/null || true
helm install release-one ./webapp-chart -n helm-one
helm install release-two ./webapp-chart -n helm-two
```

Task  
SECTION: APPLICATION DEPLOYMENT (HELM)

List all Helm releases in all namespaces.

---

<details>
<summary>Soluzione</summary>

```bash

helm ls -A
```

</details>


### Cleanup

```bash
helm uninstall release-one -n helm-one 2>/dev/null || true
helm uninstall release-two -n helm-two 2>/dev/null || true
kubectl delete namespace helm-one helm-two --ignore-not-found
rm -rf webapp-chart
```


---

### HELM-10

### Preparazione

```bash
rm -rf webapp-chart
helm create webapp-chart
kubectl create namespace helm-test --dry-run=client -o yaml | kubectl apply -f -
helm uninstall webapp-release -n helm-test 2>/dev/null || true
helm install webapp-release ./webapp-chart -n helm-test
```

Task  
SECTION: APPLICATION DEPLOYMENT (HELM)

Upgrade the release `webapp-release` to use:

- image.repository = nginx
- image.tag = 1.25

---

<details>
<summary>Soluzione</summary>

```bash


helm upgrade webapp-release ./webapp-chart \
  -n helm-test \
  --set image.repository=nginx \
  --set image.tag=1.25

helm get values webapp-release -n helm-test
```

</details>


### Cleanup

```bash
helm uninstall webapp-release -n helm-test 2>/dev/null || true
kubectl delete namespace helm-test --ignore-not-found
rm -rf webapp-chart
```


---

### HELM-11

### Preparazione

```bash
rm -rf webapp-chart
helm create webapp-chart
kubectl create namespace helm-test --dry-run=client -o yaml | kubectl apply -f -
helm uninstall webapp-release -n helm-test 2>/dev/null || true
helm install webapp-release ./webapp-chart -n helm-test --set replicaCount=1
helm upgrade webapp-release ./webapp-chart -n helm-test --set replicaCount=3
```

Task  
SECTION: APPLICATION DEPLOYMENT (HELM)

Rollback the release `webapp-release` to the previous revision.

---

<details>
<summary>Soluzione</summary>

```bash

helm history webapp-release -n helm-test
helm rollback webapp-release 1 -n helm-test
helm history webapp-release -n helm-test
```

</details>


### Cleanup

```bash
helm uninstall webapp-release -n helm-test 2>/dev/null || true
kubectl delete namespace helm-test --ignore-not-found
rm -rf webapp-chart
```


---

### HELM-12

### Preparazione

```bash
rm -rf webapp-chart
helm create webapp-chart
kubectl create namespace helm-test --dry-run=client -o yaml | kubectl apply -f -
helm uninstall webapp-release -n helm-test 2>/dev/null || true
helm install webapp-release ./webapp-chart -n helm-test
```

Task  
SECTION: APPLICATION DEPLOYMENT (HELM)

Uninstall the Helm release `webapp-release` from namespace `helm-test`.

---

<details>
<summary>Soluzione</summary>

```bash

helm uninstall webapp-release -n helm-test
helm ls -n helm-test
```

</details>


### Cleanup

```bash
kubectl delete namespace helm-test --ignore-not-found
rm -rf webapp-chart
```


---

### HELM-13

### Preparazione

```bash
rm -rf webapp-chart
helm create webapp-chart
```

Task  
SECTION: APPLICATION DEPLOYMENT (HELM)

Run lint checks against the Helm chart in `./webapp-chart`.

---

<details>
<summary>Soluzione</summary>

```bash

helm lint ./webapp-chart
```

</details>


### Cleanup

```bash
rm -rf webapp-chart
```


---

### HELM-14

### Preparazione

```bash
rm -rf webapp-chart
helm create webapp-chart
```

Task  
SECTION: APPLICATION DEPLOYMENT (HELM)

Render the chart templates without installing the chart.

---

<details>
<summary>Soluzione</summary>

```bash

helm template webapp-release ./webapp-chart
```

</details>


### Cleanup

```bash
rm -rf webapp-chart
```


---

### HELM-15

### Preparazione

```bash
rm -rf webapp-chart
helm create webapp-chart
kubectl delete namespace frontend-apd --ignore-not-found
```

Task  
SECTION: APPLICATION DEPLOYMENT (HELM)


Install the Helm chart `./webapp-chart` with:

- Release name: `custom-webapp`
- Namespace: `frontend-apd`
- replicaCount = 3
- service.type = NodePort

Create the namespace if needed.

---

<details>
<summary>Soluzione</summary>

```bash

kubectl create namespace frontend-apd --dry-run=client -o yaml | kubectl apply -f -

helm install custom-webapp ./webapp-chart \
  -n frontend-apd \
  --set replicaCount=3 \
  --set service.type=NodePort

helm get values custom-webapp -n frontend-apd
```

</details>


### Cleanup

```bash
helm uninstall custom-webapp -n frontend-apd 2>/dev/null || true
kubectl delete namespace frontend-apd --ignore-not-found
rm -rf webapp-chart
```


---

### HELM-16

### Preparazione

```bash
rm -rf webapp-chart
helm create webapp-chart
```

Task  
SECTION: APPLICATION DEPLOYMENT (HELM)


Find all configurable values in the Helm chart `./webapp-chart`.

---

<details>
<summary>Soluzione</summary>

```bash

cat ./webapp-chart/values.yaml
helm show values ./webapp-chart
```

</details>


### Cleanup

```bash
rm -rf webapp-chart
```


---

### HELM-17

### Preparazione

```bash
rm -rf webapp-chart custom-values.yaml
helm create webapp-chart
kubectl create namespace helm-test --dry-run=client -o yaml | kubectl apply -f -
helm uninstall webapp-custom-values -n helm-test 2>/dev/null || true
```

Task  
SECTION: APPLICATION DEPLOYMENT (HELM)

Create a custom values file named `custom-values.yaml` with the following values:

- replicaCount = 2
- service.type = ClusterIP
- image.repository = nginx
- image.tag = 1.27

Then install the chart as release `webapp-custom-values` in namespace `helm-test`.

---

<details>
<summary>Soluzione</summary>

```yaml
replicaCount: 2

image:
  repository: nginx
  tag: "1.27"

service:
  type: ClusterIP
```

```bash


kubectl create namespace helm-test --dry-run=client -o yaml | kubectl apply -f -

helm install webapp-custom-values ./webapp-chart \
  -n helm-test \
  -f custom-values.yaml
```

</details>


### Cleanup

```bash
helm uninstall webapp-custom-values -n helm-test 2>/dev/null || true
kubectl delete namespace helm-test --ignore-not-found
rm -rf webapp-chart custom-values.yaml
```


---

### HELM-18

### Preparazione

```bash
rm -rf webapp-chart upgrade-values.yaml
helm create webapp-chart
kubectl create namespace helm-test --dry-run=client -o yaml | kubectl apply -f -
helm uninstall webapp-custom-values -n helm-test 2>/dev/null || true
helm install webapp-custom-values ./webapp-chart -n helm-test --set replicaCount=1 --set service.type=ClusterIP
```

Task  
SECTION: APPLICATION DEPLOYMENT (HELM)

Upgrade the release `webapp-custom-values` using a new values file `upgrade-values.yaml` with:

- replicaCount = 4
- service.type = NodePort

---

<details>
<summary>Soluzione</summary>

```yaml
replicaCount: 4

service:
  type: NodePort
```

```bash


helm upgrade webapp-custom-values ./webapp-chart \
  -n helm-test \
  -f upgrade-values.yaml

helm get values webapp-custom-values -n helm-test
```

</details>


### Cleanup

```bash
helm uninstall webapp-custom-values -n helm-test 2>/dev/null || true
kubectl delete namespace helm-test --ignore-not-found
rm -rf webapp-chart upgrade-values.yaml
```


---

### HELM-19

### Preparazione

```bash
rm -rf webapp-chart
helm create webapp-chart
kubectl create namespace helm-test --dry-run=client -o yaml | kubectl apply -f -
helm uninstall webapp-custom-values -n helm-test 2>/dev/null || true
helm install webapp-custom-values ./webapp-chart -n helm-test --set replicaCount=1
helm upgrade webapp-custom-values ./webapp-chart -n helm-test --set replicaCount=2
```

Task  
SECTION: APPLICATION DEPLOYMENT (HELM)


Check the release history for `webapp-custom-values`.

---

<details>
<summary>Soluzione</summary>

```bash

helm history webapp-custom-values -n helm-test
```

</details>


### Cleanup

```bash
helm uninstall webapp-custom-values -n helm-test 2>/dev/null || true
kubectl delete namespace helm-test --ignore-not-found
rm -rf webapp-chart
```


---

### HELM-20

### Preparazione

```bash
rm -rf webapp-chart
helm create webapp-chart
kubectl create namespace helm-test --dry-run=client -o yaml | kubectl apply -f -
helm uninstall webapp-dryrun -n helm-test 2>/dev/null || true
```

Task  
SECTION: APPLICATION DEPLOYMENT (HELM)

Install the chart `./webapp-chart` in dry-run mode as release `webapp-dryrun` in namespace `helm-test`.

---

<details>
<summary>Soluzione</summary>

```bash

helm install webapp-dryrun ./webapp-chart -n helm-test --dry-run --debug
```

</details>


### Cleanup

```bash
kubectl delete namespace helm-test --ignore-not-found
rm -rf webapp-chart
```


---

### HELM-21

### Preparazione

```bash
rm -rf webapp-chart
helm create webapp-chart
```

Task  
SECTION: APPLICATION DEPLOYMENT (HELM)

Update the chart metadata so that:

- chart name remains `webapp-chart`
- appVersion becomes `2.0.0`
- version becomes `0.2.0`

---

<details>
<summary>Soluzione</summary>

Edit `Chart.yaml` so it looks like this:

```yaml
apiVersion: v2
name: webapp-chart
description: A Helm chart for Kubernetes
type: application
version: 0.2.0
appVersion: "2.0.0"
```

```bash

cat webapp-chart/Chart.yaml
```

</details>


### Cleanup

```bash
rm -rf webapp-chart
```


---

### HELM-22

### Preparazione

```bash
rm -rf broken-chart
helm create broken-chart
sed -i 's#apiVersion: apps/v1#apiVersion: apps/v1beta1#' broken-chart/templates/deployment.yaml
```

Task  
SECTION: APPLICATION DEPLOYMENT (HELM)


A chart in `./broken-chart` contains an incorrect deployment apiVersion. Fix the issue and then run lint.

Expected deployment apiVersion:

- `apps/v1`

---

<details>
<summary>Soluzione</summary>

Open the deployment template, usually:

```bash
vi ./broken-chart/templates/deployment.yaml
```

Find and correct:

```yaml
apiVersion: apps/v1
```

Then run:

```bash

helm lint ./broken-chart
```

</details>


### Cleanup

```bash
rm -rf broken-chart
```


---

### HELM-23

### Preparazione

```bash
rm -rf broken-chart
helm create broken-chart
cat >> broken-chart/values.yaml <<'EOF'

service:
  name: frontend-service
EOF
sed -i 's#name: {{ include "broken-chart.fullname" . }}#name: {{ .Values.service.wrongName }}#' broken-chart/templates/service.yaml
```

Task  
SECTION: APPLICATION DEPLOYMENT (HELM)

A service template in `./broken-chart` contains a broken Helm variable reference for the service name. Fix the template so that it correctly references:

- `.Values.service.name`

Then render the chart.

---

<details>
<summary>Soluzione</summary>

Open the service template:

```bash
vi ./broken-chart/templates/service.yaml
```

Correct the variable reference so that it contains:

```yaml
name: {{ .Values.service.name }}
```

Then render the chart:

```bash

helm template fixed-render ./broken-chart
```

</details>


### Cleanup

```bash
rm -rf broken-chart
```


---

### HELM-24

### Preparazione

```bash
rm -rf broken-chart
helm create broken-chart
sed -i 's#apiVersion: apps/v1#apiVersion: apps/v1beta1#' broken-chart/templates/deployment.yaml
```

Task  
SECTION: APPLICATION DEPLOYMENT (HELM)

Debug a failed Helm installation for the chart `./broken-chart` and identify the issue.

Use Helm commands to inspect the templates and validate the chart.

---

<details>
<summary>Soluzione</summary>

```bash


helm lint ./broken-chart
helm template debug-release ./broken-chart
helm install debug-release ./broken-chart -n helm-test --dry-run --debug
```

Typical issues to look for:

- invalid `apiVersion`
- wrong indentation in YAML
- broken template variables such as `{{ .Values... }}`
- missing required values in `values.yaml`


</details>


### Cleanup

```bash
rm -rf broken-chart
```


---
