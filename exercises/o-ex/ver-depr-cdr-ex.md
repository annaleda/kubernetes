# Exercises: API Versions, Deprecations e CRD

---

###  Esercizio 1 – Fix API Version

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
<summary> Soluzione</summary>

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

###  Esercizio 2 – Fix Ingress

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

**Task:** aggiorna completamente.

<details>
<summary>✅ Soluzione</summary>

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


###  Esercizio 3 – CronJob Deprecato

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

**Task:** aggiorna la versione.

<details>
<summary> Soluzione</summary>

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

###  Esercizio 4 – kubectl explain

**Task:** trova apiVersion e campi obbligatori per Deployment.

<details>
<summary> Soluzione</summary>

```bash
kubectl explain deployment
kubectl explain deployment --api-version=apps/v1
kubectl explain deployment.spec
```

</details>

---


###  Esercizio 5 – Completa CRD

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

**Task:** completa correttamente.

<details>
<summary> Soluzione</summary>

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

###  Esercizio 6 – Usa la CRD

**Dato:**

* group: example.com
* version: v1
* kind: App

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


###  Esercizio 7 – Debug Deployment

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

**Task:** correggi completamente.

<details>
<summary> Soluzione</summary>

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

##  PATTERN CKAD

Quando qualcosa non funziona:

* apiVersion sbagliata
* selector mancante
* campi cambiati

---

