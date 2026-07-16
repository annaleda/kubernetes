# yq Cheat Sheet - CKAD

> Comandi più utili per leggere e modificare file YAML durante l'esame.

---

## Visualizzare un file

```bash
yq file.yaml
```

---

## Leggere un campo

```bash
yq '.metadata.name' pod.yaml

yq '.spec.containers[0].image' pod.yaml

yq '.spec.replicas' deploy.yaml
```

---

## Elencare un array

```bash
yq '.spec.containers[].name' pod.yaml

yq '.items[].metadata.name' pods.yaml
```

---

## Modificare un campo

```bash
yq -i '.spec.replicas=3' deploy.yaml

yq -i '.spec.containers[0].image="nginx:1.25"' deploy.yaml
```

---

## Aggiungere una label

```bash
yq -i '.metadata.labels.app="web"' pod.yaml
```

---

## Aggiungere una variabile d'ambiente

```bash
yq -i '.spec.containers[0].env += [{"name":"ENV","value":"prod"}]' pod.yaml
```

---

## Cambiare namespace

```bash
yq -i '.metadata.namespace="dev"' pod.yaml
```

---

## Eliminare un campo

```bash
yq -i 'del(.metadata.creationTimestamp)' pod.yaml

yq -i 'del(.status)' pod.yaml
```

---

## Convertire in JSON

```bash
yq -o=json pod.yaml
```

---

## I comandi da ricordare

```bash
yq '.metadata.name' file.yaml

yq '.spec.containers[].image' file.yaml

yq -i '.spec.replicas=3' deploy.yaml

yq -i 'del(.status)' pod.yaml

yq -o=json file.yaml
```

---

# curl Cheat Sheet - CKAD

> Comandi utili per verificare Pod, Service e Ingress.

---

## Richiesta HTTP

```bash
curl http://IP
```

---

## Visualizzare header

```bash
curl -I http://IP
```

---

## Header + body

```bash
curl -i http://IP
```

---

## Verbose

```bash
curl -v http://IP
```

---

## HTTPS ignorando il certificato

```bash
curl -k https://IP
```

---

## Salvare output

```bash
curl http://IP -o index.html
```

---

## Seguire redirect

```bash
curl -L http://IP
```

---

## POST

```bash
curl -X POST http://IP
```

---

## POST JSON

```bash
curl \
-H "Content-Type: application/json" \
-d '{"name":"ckad"}' \
http://IP
```

---

## Testare un Service

```bash
curl http://service-name
```

---

## Testare un Pod

```bash
curl http://POD_IP
```

---

## Testare un Ingress

```bash
curl http://HOSTNAME
```

---

## I comandi da ricordare

```bash
curl http://IP

curl -I http://IP

curl -v http://IP

curl -k https://IP

curl -L http://IP

curl -X POST http://IP
```

---

# Helm Cheat Sheet - CKAD

> Comandi Helm più comuni per l'esame CKAD.

---

## Informazioni

```bash
helm version

helm env
```

---

## Repository

```bash
helm repo list

helm repo add bitnami https://charts.bitnami.com/bitnami

helm repo update

helm repo remove bitnami
```

---

## Ricerca

```bash
helm search repo nginx

helm search hub nginx
```

---

## Installazione

```bash
helm install web bitnami/nginx

helm install web ./chart

helm install web ./chart -n dev
```

---

## Upgrade

```bash
helm upgrade web ./chart

helm upgrade web ./chart --set image.tag=1.25
```

---

## Rollback

```bash
helm rollback web 1
```

---

## Disinstallazione

```bash
helm uninstall web

helm uninstall web -n dev
```

---

## Elencare release

```bash
helm list

helm list -A
```

---

## Stato release

```bash
helm status web

helm history web
```

---

## Rendering template

```bash
helm template web ./chart

helm template web ./chart -f values.yaml
```

---

## Lint

```bash
helm lint ./chart
```

---

## Creare un chart

```bash
helm create mychart
```

---

## Package

```bash
helm package mychart
```

---

## Estrarre un chart

```bash
helm pull bitnami/nginx

helm pull bitnami/nginx --untar
```

---

## Values

```bash
helm show values bitnami/nginx

helm install web ./chart -f values.yaml

helm install web ./chart \
--set image.tag=1.25
```

---

## I 15 comandi da ricordare

```bash
helm repo add

helm repo update

helm search repo

helm install

helm upgrade

helm uninstall

helm rollback

helm list

helm status

helm history

helm template

helm lint

helm create

helm package

helm show values
```
