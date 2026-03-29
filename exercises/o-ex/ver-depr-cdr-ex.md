- [ Home ](../../readme.md)   | [ Teoria ](../../arguments.md)   | [ Info Exam ](../../doc/ckad_exam_strategy.md)    | [ Home Other Exercises ](../o_exercises.md)   
--- 
# Exercises: API Versions, Deprecations e CRD

---

## AP-1 — Fix Deployment API Version

Manifest iniziale:

```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: web
spec:
  replicas: 2
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: nginx
        image: nginx
```

**Task:** rendilo valido per Kubernetes moderno.

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: nginx
        image: nginx
```

</details>

---

## AP-2 — Fix Ingress deprecato

Manifest iniziale:

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: test
spec:
  rules:
  - http:
      paths:
      - path: /
        backend:
          serviceName: web
          servicePort: 80
```

**Task:** aggiorna completamente il manifest.

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: test
spec:
  rules:
  - http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web
            port:
              number: 80
```

</details>

---

## AP-3 — Fix CronJob deprecato

Manifest iniziale:

```yaml
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: job
spec:
  schedule: "* * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: busybox
            image: busybox
            command: ["echo", "hi"]
          restartPolicy: OnFailure
```

**Task:** aggiorna la versione API.

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: job
spec:
  schedule: "* * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: busybox
            image: busybox
            command: ["echo", "hi"]
          restartPolicy: OnFailure
```

</details>

---

## AP-4 — Usare kubectl explain

**Task:** trova `apiVersion` e campi importanti per un Deployment moderno.

<details>
<summary>Soluzione</summary>

```bash
kubectl explain deployment
kubectl explain deployment --api-version=apps/v1
kubectl explain deployment.spec
kubectl explain deployment.spec.selector
kubectl explain deployment.spec.template
```

</details>

---

## AP-5 — Completa una CRD base

Manifest iniziale:

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: apps.example.com
spec:
  names:
    kind: App
    plural: apps
  versions:
    - name: v1
```

**Task:** completa correttamente il manifest.

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: apps.example.com
spec:
  group: example.com
  scope: Namespaced
  names:
    kind: App
    plural: apps
    singular: app
    shortNames:
    - appx
  versions:
  - name: v1
    served: true
    storage: true
    schema:
      openAPIV3Schema:
        type: object
```

</details>

---

## AP-6 — Usa la CRD

**Dato:**
- group: `example.com`
- version: `v1`
- kind: `App`

**Task:** scrivi una risorsa valida.

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: example.com/v1
kind: App
metadata:
  name: my-app
```

</details>

---

## AP-7 — Debug Deployment incompleto

Manifest iniziale:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: broken
spec:
  replicas: 2
  template:
    metadata:
      labels:
        app: test
    spec:
      containers:
      - name: nginx
        image: nginx
```

**Task:** correggi completamente il manifest.

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: broken
spec:
  replicas: 2
  selector:
    matchLabels:
      app: test
  template:
    metadata:
      labels:
        app: test
    spec:
      containers:
      - name: nginx
        image: nginx
```

</details>

---

## AP-8 — Fix StatefulSet API Version

Manifest iniziale:

```yaml
apiVersion: apps/v1beta1
kind: StatefulSet
metadata:
  name: db
spec:
  serviceName: db
  replicas: 2
  template:
    metadata:
      labels:
        app: db
    spec:
      containers:
      - name: nginx
        image: nginx
```

**Task:** rendilo valido per Kubernetes moderno.

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: db
spec:
  serviceName: db
  replicas: 2
  selector:
    matchLabels:
      app: db
  template:
    metadata:
      labels:
        app: db
    spec:
      containers:
      - name: nginx
        image: nginx
```

</details>

---

## AP-9 — Fix ReplicaSet API Version

Manifest iniziale:

```yaml
apiVersion: extensions/v1beta1
kind: ReplicaSet
metadata:
  name: web-rs
spec:
  replicas: 2
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: nginx
        image: nginx
```

**Task:** aggiornalo per Kubernetes moderno.

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: web-rs
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: nginx
        image: nginx
```

</details>

---

## AP-10 — Ingress con host

**Task:** crea un Ingress moderno chiamato `web-ingress` che inoltri:
- host: `example.local`
- path: `/`
- service: `web`
- port: `80`

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-ingress
spec:
  rules:
  - host: example.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web
            port:
              number: 80
```

</details>

---

## AP-11 — CRD con campo spec richiesto

**Task:** definisci una CRD `widgets.demo.example.com` con:
- group: `demo.example.com`
- kind: `Widget`
- plural: `widgets`
- version: `v1`
- campo richiesto: `spec.size` di tipo integer

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: widgets.demo.example.com
spec:
  group: demo.example.com
  scope: Namespaced
  names:
    plural: widgets
    singular: widget
    kind: Widget
  versions:
  - name: v1
    served: true
    storage: true
    schema:
      openAPIV3Schema:
        type: object
        properties:
          spec:
            type: object
            required:
            - size
            properties:
              size:
                type: integer
```

</details>

---

## AP-12 — Crea una Custom Resource valida

**Dato:**
- group: `demo.example.com`
- version: `v1`
- kind: `Widget`
- campo richiesto: `spec.size`

**Task:** crea una risorsa valida chiamata `small-widget`.

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: demo.example.com/v1
kind: Widget
metadata:
  name: small-widget
spec:
  size: 1
```

</details>

---

## AP-13 — CRD con due versioni

**Task:** scrivi una CRD `gadgets.demo.example.com` con:
- kind: `Gadget`
- versions:
  - `v1` served true, storage false
  - `v2` served true, storage true

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: gadgets.demo.example.com
spec:
  group: demo.example.com
  scope: Namespaced
  names:
    plural: gadgets
    singular: gadget
    kind: Gadget
  versions:
  - name: v1
    served: true
    storage: false
    schema:
      openAPIV3Schema:
        type: object
  - name: v2
    served: true
    storage: true
    schema:
      openAPIV3Schema:
        type: object
```

</details>

---

## AP-14 — Kubectl explain su CRD

**Task:** dopo aver creato la CRD `myapps.stable.example.com`, usa `kubectl explain` per ispezionare:
- la risorsa custom
- il campo `spec`

<details>
<summary>Soluzione</summary>

```bash
kubectl explain myapps
kubectl explain myapps.spec
```

</details>

---

## AP-15 — Debug CRD non valida

Manifest iniziale:

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: books.demo.example.com
spec:
  group: demo.example.com
  scope: Namespaced
  names:
    kind: Book
    plural: books
  versions:
  - name: v1
    schema:
      openAPIV3Schema:
        type: object
```

**Task:** correggilo completamente.

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: books.demo.example.com
spec:
  group: demo.example.com
  scope: Namespaced
  names:
    kind: Book
    plural: books
    singular: book
  versions:
  - name: v1
    served: true
    storage: true
    schema:
      openAPIV3Schema:
        type: object
```

</details>

---

## Pattern da ricordare

Quando un manifest non funziona, controlla quasi sempre:

- `apiVersion` sbagliata
- `selector` mancante o incoerente
- campi rinominati
- struttura `backend` vecchia negli Ingress
- `served/storage` mancanti nelle CRD
- schema CRD incompleto

---

## Cheatsheet rapido

```bash
# capire quali API usare
kubectl api-resources

# ispezionare campi e struttura
kubectl explain deployment --api-version=apps/v1
kubectl explain ingress --api-version=networking.k8s.io/v1
kubectl explain cronjob --api-version=batch/v1

# vedere le CRD
kubectl get crd

# vedere le custom resources
kubectl get <plural-name>

# debug veloce
kubectl apply -f file.yaml
kubectl describe <kind> <name>
```
