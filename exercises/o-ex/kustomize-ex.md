- [ Home ](../readme.md)   | [ Teoria ](../arguments.md)   | [ Info Exam ](../doc/ckad_exam_strategy.md)    | [ Home Other Exercises ](../o_exercises.md)   
--- 

###  Kustomize (12 esercizi)

## KUS-1 — Configurazione base

### Preparazione

```bash
rm -rf kus-1
mkdir -p kus-1/base

kubectl delete namespace kustomize --ignore-not-found
kubectl create namespace kustomize

kubectl create deployment nginx \
  --image=nginx:1.21 \
  -n kustomize \
  --dry-run=client -o yaml > kus-1/base/deployment.yaml

kubectl expose deployment nginx \
  --name=nginx \
  --port=80 \
  -n kustomize \
  --dry-run=client -o yaml > kus-1/base/service.yaml
```

### Task

Crea `kus-1/base/kustomization.yaml` includendo:

- `deployment.yaml`;
- `service.yaml`.

Applica la base al cluster.

### Validazione

```bash
kubectl kustomize kus-1/base
kubectl get deployment,service -n kustomize
```

<details>
<summary>Soluzione</summary>

```bash
cat > kus-1/base/kustomization.yaml <<'EOF'
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
- deployment.yaml
- service.yaml
EOF

kubectl apply -k kus-1/base
```

</details>

### Cleanup

```bash
kubectl delete -k kus-1/base --ignore-not-found
kubectl delete namespace kustomize --ignore-not-found
rm -rf kus-1
```

---

## KUS-2 — Overlay development

### Preparazione

```bash
rm -rf kus-2
mkdir -p kus-2/base kus-2/overlays/dev

kubectl delete namespace dev --ignore-not-found
kubectl create namespace dev

kubectl create deployment nginx \
  --image=nginx:1.21 \
  --dry-run=client -o yaml > kus-2/base/deployment.yaml

kubectl expose deployment nginx \
  --name=nginx \
  --port=80 \
  --dry-run=client -o yaml > kus-2/base/service.yaml

cat > kus-2/base/kustomization.yaml <<'EOF'
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
- deployment.yaml
- service.yaml
EOF
```

### Task

Crea l'overlay `kus-2/overlays/dev` con:

- base `../../base`;
- namespace `dev`;
- override dell'immagine da `nginx:1.21` a `nginx:1.23`.

### Validazione

```bash
kubectl kustomize kus-2/overlays/dev | grep -A1 'image:'
kubectl apply -k kus-2/overlays/dev
kubectl get deployment -n dev \
  -o jsonpath='{.items[0].spec.template.spec.containers[0].image}{"\n"}'
```

<details>
<summary>Soluzione</summary>

```bash
cat > kus-2/overlays/dev/kustomization.yaml <<'EOF'
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
namespace: dev
resources:
- ../../base
images:
- name: nginx
  newTag: "1.23"
EOF
```

</details>

### Cleanup

```bash
kubectl delete -k kus-2/overlays/dev --ignore-not-found
kubectl delete namespace dev --ignore-not-found
rm -rf kus-2
```

---

## KUS-3 — Name prefix

### Preparazione

```bash
rm -rf kus-3
mkdir -p kus-3/base kus-3/overlays/dev

kubectl delete namespace dev --ignore-not-found
kubectl create namespace dev

kubectl create deployment nginx \
  --image=nginx:1.23 \
  --dry-run=client -o yaml > kus-3/base/deployment.yaml

kubectl expose deployment nginx \
  --name=nginx \
  --port=80 \
  --dry-run=client -o yaml > kus-3/base/service.yaml

cat > kus-3/base/kustomization.yaml <<'EOF'
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
- deployment.yaml
- service.yaml
EOF
```

### Task

Crea un overlay in `kus-3/overlays/dev` che:

- usa `../../base`;
- imposta namespace `dev`;
- applica `namePrefix: dev-`.

### Validazione

```bash
kubectl kustomize kus-3/overlays/dev | grep '^  name:'
kubectl apply -k kus-3/overlays/dev
kubectl get deployment,service -n dev
```

<details>
<summary>Soluzione</summary>

```bash
cat > kus-3/overlays/dev/kustomization.yaml <<'EOF'
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
namespace: dev
namePrefix: dev-
resources:
- ../../base
EOF
```

</details>

### Cleanup

```bash
kubectl delete -k kus-3/overlays/dev --ignore-not-found
kubectl delete namespace dev --ignore-not-found
rm -rf kus-3
```

---

## KUS-4 — ConfigMap generator

### Preparazione

```bash
rm -rf kus-4
mkdir -p kus-4/base

kubectl delete namespace kustomize --ignore-not-found
kubectl create namespace kustomize

kubectl create deployment app \
  --image=nginx \
  --dry-run=client -o yaml > kus-4/base/deployment.yaml

cat > kus-4/base/app.properties <<'EOF'
key1=value1
key2=value2
EOF
```

### Task

Crea `kus-4/base/kustomization.yaml` con:

- namespace `kustomize`;
- `deployment.yaml` come risorsa;
- un `configMapGenerator` chiamato `app-config`;
- sorgente: file `app.properties`.

### Validazione

```bash
kubectl kustomize kus-4/base
kubectl apply -k kus-4/base
kubectl get configmap -n kustomize
```

Il nome della ConfigMap deve contenere un hash.

<details>
<summary>Soluzione</summary>

```bash
cat > kus-4/base/kustomization.yaml <<'EOF'
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
namespace: kustomize
resources:
- deployment.yaml
configMapGenerator:
- name: app-config
  files:
  - app.properties
EOF
```

</details>

### Cleanup

```bash
kubectl delete -k kus-4/base --ignore-not-found
kubectl delete namespace kustomize --ignore-not-found
rm -rf kus-4
```

---

## KUS-5 — Patch delle repliche

### Preparazione

```bash
rm -rf kus-5
mkdir -p kus-5/base kus-5/overlay

kubectl delete namespace kustomize --ignore-not-found
kubectl create namespace kustomize

kubectl create deployment deploy \
  --image=nginx:1.21 \
  --replicas=1 \
  --dry-run=client -o yaml > kus-5/base/deployment.yaml

cat > kus-5/base/kustomization.yaml <<'EOF'
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
- deployment.yaml
EOF
```

### Task

Nell'overlay `kus-5/overlay`:

- usa la base `../base`;
- imposta namespace `kustomize`;
- applica una patch che porti le repliche del Deployment `deploy` a `3`.

Usa la sintassi moderna `patches`.

### Validazione

```bash
kubectl kustomize kus-5/overlay | grep -A2 'replicas:'
kubectl apply -k kus-5/overlay
kubectl get deployment deploy -n kustomize
```

<details>
<summary>Soluzione</summary>

```bash
cat > kus-5/overlay/patch-replicas.yaml <<'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deploy
spec:
  replicas: 3
EOF

cat > kus-5/overlay/kustomization.yaml <<'EOF'
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
namespace: kustomize
resources:
- ../base
patches:
- path: patch-replicas.yaml
EOF
```

</details>

### Cleanup

```bash
kubectl delete -k kus-5/overlay --ignore-not-found
kubectl delete namespace kustomize --ignore-not-found
rm -rf kus-5
```

---

## KUS-6 — Image transformer

### Preparazione

```bash
rm -rf kus-6
mkdir -p kus-6/base

kubectl delete namespace dev --ignore-not-found
kubectl create namespace dev

kubectl create deployment deploy \
  --image=nginx:1.21 \
  --dry-run=client -o yaml > kus-6/base/deployment.yaml
```

### Task

Crea `kus-6/base/kustomization.yaml` che:

- usa namespace `dev`;
- include `deployment.yaml`;
- sostituisce l'immagine con `nginx:1.25`.

### Validazione

```bash
kubectl kustomize kus-6/base | grep 'image:'
kubectl apply -k kus-6/base
kubectl get deployment deploy -n dev \
  -o jsonpath='{.spec.template.spec.containers[0].image}{"\n"}'
```

<details>
<summary>Soluzione</summary>

```bash
cat > kus-6/base/kustomization.yaml <<'EOF'
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
namespace: dev
resources:
- deployment.yaml
images:
- name: nginx
  newTag: "1.25"
EOF
```

</details>

### Cleanup

```bash
kubectl delete -k kus-6/base --ignore-not-found
kubectl delete namespace dev --ignore-not-found
rm -rf kus-6
```

---

## KUS-7 — Label comuni

### Preparazione

```bash
rm -rf kus-7
mkdir -p kus-7/base

kubectl delete namespace kustomize --ignore-not-found
kubectl create namespace kustomize

kubectl create deployment app \
  --image=nginx \
  --dry-run=client -o yaml > kus-7/base/deployment.yaml

kubectl expose deployment app \
  --name=app \
  --port=80 \
  --dry-run=client -o yaml > kus-7/base/service.yaml
```

### Task

Crea `kus-7/base/kustomization.yaml` che:

- usa namespace `kustomize`;
- include Deployment e Service;
- applica la label comune `env=dev`.

Usa il campo `labels`.

### Validazione

```bash
kubectl kustomize kus-7/base | grep -A2 'labels:'
kubectl apply -k kus-7/base
kubectl get deployment,service -n kustomize --show-labels
```

<details>
<summary>Soluzione</summary>

```bash
cat > kus-7/base/kustomization.yaml <<'EOF'
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
namespace: kustomize
resources:
- deployment.yaml
- service.yaml
labels:
- pairs:
    env: dev
  includeSelectors: true
EOF
```

</details>

### Cleanup

```bash
kubectl delete -k kus-7/base --ignore-not-found
kubectl delete namespace kustomize --ignore-not-found
rm -rf kus-7
```

---

## KUS-8 — Namespace transformer

### Preparazione

```bash
rm -rf kus-8
mkdir -p kus-8/base

kubectl delete namespace kustom-ns --ignore-not-found
kubectl create namespace kustom-ns

kubectl create deployment app \
  --image=nginx \
  --dry-run=client -o yaml > kus-8/base/deployment.yaml

kubectl expose deployment app \
  --name=app \
  --port=80 \
  --dry-run=client -o yaml > kus-8/base/service.yaml
```

### Task

Crea `kus-8/base/kustomization.yaml` che:

- include Deployment e Service;
- assegna tutte le risorse al namespace `kustom-ns`.

### Validazione

```bash
kubectl kustomize kus-8/base | grep 'namespace:'
kubectl apply -k kus-8/base
kubectl get deployment,service -n kustom-ns
```

<details>
<summary>Soluzione</summary>

```bash
cat > kus-8/base/kustomization.yaml <<'EOF'
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
namespace: kustom-ns
resources:
- deployment.yaml
- service.yaml
EOF
```

</details>

### Cleanup

```bash
kubectl delete -k kus-8/base --ignore-not-found
kubectl delete namespace kustom-ns --ignore-not-found
rm -rf kus-8
```

---

## KUS-9 — Secret generator

### Preparazione

```bash
rm -rf kus-9
mkdir -p kus-9/base

kubectl delete namespace kustomize --ignore-not-found
kubectl create namespace kustomize
```

### Task

Crea `kus-9/base/kustomization.yaml` con:

- namespace `kustomize`;
- un `secretGenerator` chiamato `db-secret`;
- literal:
  - `username=admin`;
  - `password=secret`.

### Validazione

```bash
kubectl kustomize kus-9/base
kubectl apply -k kus-9/base
kubectl get secret -n kustomize
```

Decodifica i valori generati:

```bash
kubectl get secret -n kustomize \
  -l app.kubernetes.io/managed-by=kustomize \
  -o yaml
```

<details>
<summary>Soluzione</summary>

```bash
cat > kus-9/base/kustomization.yaml <<'EOF'
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
namespace: kustomize
secretGenerator:
- name: db-secret
  literals:
  - username=admin
  - password=secret
EOF
```

</details>

### Cleanup

```bash
kubectl delete -k kus-9/base --ignore-not-found
kubectl delete namespace kustomize --ignore-not-found
rm -rf kus-9
```

---

## KUS-10 — Name suffix

### Preparazione

```bash
rm -rf kus-10
mkdir -p kus-10/base

kubectl delete namespace kustomize --ignore-not-found
kubectl create namespace kustomize

kubectl create deployment app \
  --image=nginx \
  --dry-run=client -o yaml > kus-10/base/deployment.yaml

kubectl expose deployment app \
  --name=app \
  --port=80 \
  --dry-run=client -o yaml > kus-10/base/service.yaml
```

### Task

Crea `kus-10/base/kustomization.yaml` che:

- usa namespace `kustomize`;
- include Deployment e Service;
- applica `nameSuffix: -test`.

### Validazione

```bash
kubectl kustomize kus-10/base | grep '^  name:'
kubectl apply -k kus-10/base
kubectl get deployment,service -n kustomize
```

<details>
<summary>Soluzione</summary>

```bash
cat > kus-10/base/kustomization.yaml <<'EOF'
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
namespace: kustomize
nameSuffix: -test
resources:
- deployment.yaml
- service.yaml
EOF
```

</details>

### Cleanup

```bash
kubectl delete -k kus-10/base --ignore-not-found
kubectl delete namespace kustomize --ignore-not-found
rm -rf kus-10
```

---

## KUS-11 — Patch JSON 6902

### Preparazione

```bash
rm -rf kus-11
mkdir -p kus-11/base kus-11/overlay

kubectl delete namespace kustomize --ignore-not-found
kubectl create namespace kustomize

kubectl create deployment deploy \
  --image=nginx \
  --replicas=1 \
  --dry-run=client -o yaml > kus-11/base/deployment.yaml

cat > kus-11/base/kustomization.yaml <<'EOF'
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
- deployment.yaml
EOF
```

### Task

Nell'overlay `kus-11/overlay`:

- usa `../base`;
- imposta namespace `kustomize`;
- applica una patch JSON 6902 che sostituisce `/spec/replicas` con `5`;
- usa il campo moderno `patches`.

### Validazione

```bash
kubectl kustomize kus-11/overlay | grep -A2 'replicas:'
kubectl apply -k kus-11/overlay
kubectl get deployment deploy -n kustomize
```

<details>
<summary>Soluzione</summary>

```bash
cat > kus-11/overlay/patch.json <<'EOF'
[
  {
    "op": "replace",
    "path": "/spec/replicas",
    "value": 5
  }
]
EOF

cat > kus-11/overlay/kustomization.yaml <<'EOF'
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
namespace: kustomize
resources:
- ../base
patches:
- target:
    group: apps
    version: v1
    kind: Deployment
    name: deploy
  path: patch.json
EOF
```

</details>

### Cleanup

```bash
kubectl delete -k kus-11/overlay --ignore-not-found
kubectl delete namespace kustomize --ignore-not-found
rm -rf kus-11
```

---

## KUS-12 — Più risorse

### Preparazione

```bash
rm -rf kus-12
mkdir -p kus-12/base

kubectl delete namespace kustomize --ignore-not-found
kubectl create namespace kustomize

kubectl create deployment app \
  --image=nginx \
  --dry-run=client -o yaml > kus-12/base/deployment.yaml

kubectl expose deployment app \
  --name=app \
  --port=80 \
  --dry-run=client -o yaml > kus-12/base/service.yaml
```

### Task

Crea:

- `kus-12/base/configmap.yaml`, con ConfigMap `extra-config`;
- una chiave `key=value`;
- `kus-12/base/kustomization.yaml` che includa:
  - Deployment;
  - Service;
  - ConfigMap;
- namespace `kustomize`.

### Validazione

```bash
kubectl kustomize kus-12/base
kubectl apply -k kus-12/base
kubectl get deployment,service,configmap -n kustomize
```

<details>
<summary>Soluzione</summary>

```bash
cat > kus-12/base/configmap.yaml <<'EOF'
apiVersion: v1
kind: ConfigMap
metadata:
  name: extra-config
data:
  key: value
EOF

cat > kus-12/base/kustomization.yaml <<'EOF'
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
namespace: kustomize
resources:
- deployment.yaml
- service.yaml
- configmap.yaml
EOF
```

</details>

### Cleanup

```bash
kubectl delete -k kus-12/base --ignore-not-found
kubectl delete namespace kustomize --ignore-not-found
rm -rf kus-12
```

