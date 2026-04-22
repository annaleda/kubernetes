### Kubernetes Exercises — Application Deployment

---

## Exercise 1

### SECTION: APPLICATION DEPLOYMENT

For this question, please set the context to cluster2 by running:

```bash
kubectl config use-context cluster2
```

On the `student-node`, a Helm chart repository is given under the `/opt/app-color-ad1/` path. It contains the files that describe a set of Kubernetes resources that can be deployed as a single unit. The files have some issues.

Fix those issues and deploy them with the following specifications:

- The release name should be `webapp-color-ad1`
- All the resources should be deployed in the `frontend-ad1` namespace
- The service type should be `NodePort`
- Scale the deployment to `3`
- Application version should be `1.20.0`

### NOTE:

- Remember to make necessary changes in the `values.yaml` and `Chart.yaml` files according to the specifications
- To fix the issues, inspect the template files

<details>
<summary>Soluzione</summary>

```bash
kubectl config use-context cluster2
kubectl create ns frontend-ad1
cd /opt/app-color-ad1/

# Correggi values.yaml
# replicaCount: 3
# service:
#   type: NodePort
# image:
#   tag: "1.20.0"

# Correggi Chart.yaml
# appVersion: "1.20.0"

# Ispeziona e correggi eventuali errori nei template
helm lint .
helm template webapp-color-ad1 . -n frontend-ad1 > /tmp/rendered-ad1.yaml

helm install webapp-color-ad1 . -n frontend-ad1
kubectl get all -n frontend-ad1
```

</details>

---

## Exercise 2

### SECTION: APPLICATION DEPLOYMENT

For this question, please set the context to cluster1 by running:

```bash
kubectl config use-context cluster1
```

Create a deployment named `blue-app-ad2` in namespace `app-deploy-ad2` using image `nginx:1.19`.

The deployment must meet the following requirements:

- Deployment name: `blue-app-ad2`
- Namespace: `app-deploy-ad2`
- Replicas: `2`
- Container name: `blue-container-ad2`
- Expose the deployment internally with a service named `blue-app-svc-ad2`
- Service port: `80`
- Target port: `80`

After creating the deployment, scale it to `4` replicas.

<details>
<summary>Soluzione</summary>

```bash
kubectl config use-context cluster1
kubectl create ns app-deploy-ad2
kubectl create deployment blue-app-ad2 \
  --image=nginx:1.19 \
  -n app-deploy-ad2 \
  --dry-run=client -o yaml > blue-app-ad2.yaml

# Modifica il manifest:
# replicas: 2
# container name: blue-container-ad2
kubectl apply -f blue-app-ad2.yaml
kubectl expose deployment blue-app-ad2 \
  --name=blue-app-svc-ad2 \
  --port=80 --target-port=80 \
  -n app-deploy-ad2
kubectl scale deployment blue-app-ad2 --replicas=4 -n app-deploy-ad2
kubectl get deploy,svc -n app-deploy-ad2
```

</details>

---

## Exercise 3

### SECTION: APPLICATION DEPLOYMENT

For this question, please set the context to cluster3 by running:

```bash
kubectl config use-context cluster3
```

A deployment named `web-rollout-ad3` exists in namespace `rollout-ad3`.

Update the deployment with the following requirements:

- Change the container image from `nginx:1.18` to `nginx:1.21`
- Ensure the rollout completes successfully
- Record the change cause
- Save the rollout history output to `/root/web-rollout-history-ad3`

<details>
<summary>Soluzione</summary>

```bash
kubectl config use-context cluster3
kubectl set image deployment/web-rollout-ad3 *=nginx:1.21 -n rollout-ad3 --record
kubectl rollout status deployment/web-rollout-ad3 -n rollout-ad3
kubectl rollout history deployment/web-rollout-ad3 -n rollout-ad3 > /root/web-rollout-history-ad3
```

</details>

---

## Exercise 4

### SECTION: APPLICATION DEPLOYMENT

For this question, please set the context to cluster2 by running:

```bash
kubectl config use-context cluster2
```

Create a deployment named `api-app-ad4` in namespace `backend-ad4` with the following specifications:

- Image: `httpd:2.4`
- Replicas: `3`
- Container name: `api-container-ad4`
- Labels:
  - `app=api-app-ad4`
  - `tier=backend`

Also create a service named `api-app-svc-ad4` to expose the deployment.

Service requirements:

- Type: `ClusterIP`
- Port: `8080`
- Target Port: `80`
- Selector should match the deployment labels

<details>
<summary>Soluzione</summary>

```bash
kubectl config use-context cluster2
kubectl create ns backend-ad4
kubectl create deployment api-app-ad4 --image=httpd:2.4 -n backend-ad4 --dry-run=client -o yaml > api-app-ad4.yaml

# Modifica:
# replicas: 3
# container name: api-container-ad4
# labels app=api-app-ad4,tier=backend sia nel template sia nel selector
kubectl apply -f api-app-ad4.yaml
kubectl expose deployment api-app-ad4 \
  --name=api-app-svc-ad4 \
  --type=ClusterIP \
  --port=8080 --target-port=80 \
  -n backend-ad4
kubectl get deploy,svc -n backend-ad4
```

</details>

---

## Exercise 5

### SECTION: APPLICATION DEPLOYMENT

For this question, please set the context to cluster1 by running:

```bash
kubectl config use-context cluster1
```

On the `student-node`, a manifest file is available at `/opt/deployment-ad5.yaml`.

The file has configuration issues and does not deploy successfully.

Fix the manifest so that:

- The deployment name is `frontend-ad5`
- Namespace is `frontend-ad5-ns`
- Replicas are `3`
- The image used is `redis:7`
- The container port is `6379`

Apply the corrected file and verify that all pods are running.

<details>
<summary>Soluzione</summary>

```bash
kubectl config use-context cluster1
kubectl create ns frontend-ad5-ns
vi /opt/deployment-ad5.yaml

# Correggi name, namespace, replicas, image e containerPort
kubectl apply -f /opt/deployment-ad5.yaml
kubectl get pods -n frontend-ad5-ns
kubectl describe deploy frontend-ad5 -n frontend-ad5-ns
```

</details>

---

## Exercise 6

### SECTION: APPLICATION DEPLOYMENT

For this question, please set the context to cluster3 by running:

```bash
kubectl config use-context cluster3
```

Create a deployment named `canary-app-ad6` in namespace `canary-ad6` using image `nginx:1.20`.

Requirements:

- Replicas: `5`
- Container name: `nginx-ad6`
- Add label `release=stable`

Then patch the deployment so that all pods also contain the label:

- `env=staging`

<details>
<summary>Soluzione</summary>

```bash
kubectl config use-context cluster3
kubectl create ns canary-ad6
kubectl create deployment canary-app-ad6 --image=nginx:1.20 -n canary-ad6 --dry-run=client -o yaml > canary-app-ad6.yaml

# Modifica:
# replicas: 5
# container name: nginx-ad6
# labels release=stable nel template metadata.labels
kubectl apply -f canary-app-ad6.yaml
kubectl patch deployment canary-app-ad6 -n canary-ad6 \
  --type='merge' \
  -p '{"spec":{"template":{"metadata":{"labels":{"env":"staging"}}}}}'
kubectl get pods -n canary-ad6 --show-labels
```

</details>

---

## Exercise 7

### SECTION: APPLICATION DEPLOYMENT

For this question, please set the context to cluster2 by running:

```bash
kubectl config use-context cluster2
```

Create a namespace named `helm-ad7`.

Using the chart under `/opt/helm-web-ad7/`, deploy a Helm release with the following requirements:

- Release name: `web-ad7`
- Namespace: `helm-ad7`
- Replica count: `2`
- Service type: `ClusterIP`
- Image tag: `1.25`

The chart contains at least one template error. Fix it before deploying.

<details>
<summary>Soluzione</summary>

```bash
kubectl config use-context cluster2
kubectl create ns helm-ad7
cd /opt/helm-web-ad7/

# values.yaml
# replicaCount: 2
# service.type: ClusterIP
# image.tag: "1.25"

helm lint .
# correggi il/i template con errore
helm install web-ad7 . -n helm-ad7
kubectl get all -n helm-ad7
```

</details>

---

## Exercise 8

### SECTION: APPLICATION DEPLOYMENT

For this question, please set the context to cluster1 by running:

```bash
kubectl config use-context cluster1
```

A deployment named `audit-app-ad8` in namespace `audit-ad8` has recently been updated and is failing.

Perform the following actions:

- Inspect the rollout status
- Roll back the deployment to the previous working revision
- Confirm the pods become ready
- Save the final image name used by the deployment to `/root/audit-ad8-image`

<details>
<summary>Soluzione</summary>

```bash
kubectl config use-context cluster1
kubectl rollout status deployment/audit-app-ad8 -n audit-ad8
kubectl rollout undo deployment/audit-app-ad8 -n audit-ad8
kubectl rollout status deployment/audit-app-ad8 -n audit-ad8
kubectl get deployment audit-app-ad8 -n audit-ad8 -o jsonpath='{.spec.template.spec.containers[0].image}' > /root/audit-ad8-image
```

</details>

---

## Exercise 9

### SECTION: APPLICATION DEPLOYMENT

For this question, please set the context to cluster3 by running:

```bash
kubectl config use-context cluster3
```

Create a deployment named `multi-container-ad9` in namespace `multi-ad9`.

Deployment requirements:

- Replicas: `2`
- Pod must contain two containers:
  - `web-ad9` using image `nginx:1.25`
  - `sidecar-ad9` using image `busybox:1.36`
- The `sidecar-ad9` container must run the command:

```bash
sh -c 'while true; do echo sync; sleep 10; done'
```

<details>
<summary>Soluzione</summary>

```bash
kubectl config use-context cluster3
kubectl create ns multi-ad9
cat <<'YAML' | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: multi-container-ad9
  namespace: multi-ad9
spec:
  replicas: 2
  selector:
    matchLabels:
      app: multi-container-ad9
  template:
    metadata:
      labels:
        app: multi-container-ad9
    spec:
      containers:
      - name: web-ad9
        image: nginx:1.25
      - name: sidecar-ad9
        image: busybox:1.36
        command: ["sh","-c","while true; do echo sync; sleep 10; done"]
YAML
kubectl get deploy,pods -n multi-ad9
```

</details>

---

## Exercise 10

### SECTION: APPLICATION DEPLOYMENT

For this question, please set the context to cluster2 by running:

```bash
kubectl config use-context cluster2
```

Create a deployment named `frontend-probe-ad10` in namespace `probe-ad10` using image `nginx`.

Requirements:

- Replicas: `2`
- Container name: `frontend-ad10`
- Add a readiness probe using HTTP GET
- Probe path: `/`
- Probe port: `80`
- Initial delay seconds: `5`
- Period seconds: `10`

<details>
<summary>Soluzione</summary>

```bash
kubectl config use-context cluster2
kubectl create ns probe-ad10
cat <<'YAML' | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend-probe-ad10
  namespace: probe-ad10
spec:
  replicas: 2
  selector:
    matchLabels:
      app: frontend-probe-ad10
  template:
    metadata:
      labels:
        app: frontend-probe-ad10
    spec:
      containers:
      - name: frontend-ad10
        image: nginx
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 10
YAML
kubectl get deploy,pods -n probe-ad10
```

</details>
