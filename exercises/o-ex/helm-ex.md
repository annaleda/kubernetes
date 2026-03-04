
### Helm (6 esercizi)

## HELM-1 — Installazione Chart

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

---

## HELM-2 — Override Values via CLI

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

---

## HELM-3 — Override con values.yaml

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

---

### HELM-4 — Upgrade Release

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

---

### HELM-5 — Rollback Release

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


helm rollback web-release 2 -n helm-test

helm history web-release -n helm-test

REVISION        UPDATED                         STATUS          CHART           APP VERSION     DESCRIPTION
1               Wed Mar  4 11:15:35 2026        superseded      nginx-22.5.4    1.29.5          Install complete
2               Wed Mar  4 11:16:47 2026        superseded      nginx-22.5.4    1.29.5          Upgrade complete
3               Wed Mar  4 11:19:15 2026        deployed        nginx-22.5.4    1.29.5          Rollback to 2

```
</details>

---

### HELM-6 — Template Rendering

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

---
