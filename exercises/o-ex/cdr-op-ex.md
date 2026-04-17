- [ Home ](../../readme.md)   | [ Teoria ](../../arguments.md)   | [ Info Exam ](../../doc/ckad_exam_strategy.md)    | [ Home Other Exercises ](../o_exercises.md)   
--- 
### CRD and Operators (22 esercizi)

---

## CRD-1 — Creare CRD Base

- Definire CRD: `myapps.stable.example.com`
- Group: `stable.example.com`
- Kind: `MyApp`
- Version: `v1`
- Scope: `Namespaced`

- Campo richiesto:
  - `spec.color` di tipo stringa

- Validazione
  - `kubectl get crd`
  - `kubectl explain myapps`

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: myapps.stable.example.com
spec:
  group: stable.example.com
  scope: Namespaced
  names:
    plural: myapps
    singular: myapp
    kind: MyApp
    shortNames:
    - ma
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
            properties:
              color:
                type: string
```

```sh
kubectl apply -f crd.yaml
kubectl get crd
kubectl explain myapps
```

</details>

---

## CRD-2 — Creare una Custom Resource

- Creare un oggetto `MyApp`
- Nome: `example-app`

- Specifiche
  - `spec.color=blue`

- Validazione
  - `kubectl get myapps`
  - `kubectl get ma`

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: stable.example.com/v1
kind: MyApp
metadata:
  name: example-app
spec:
  color: blue
```

```sh
kubectl apply -f myapp.yaml
kubectl get myapps
kubectl get ma
```

</details>

---

## CRD-3 — Aggiungere una seconda versione al CRD

- Modificare il CRD `myapps.stable.example.com`
- Aggiungere versione `v2`

- Requisiti:
  - `v1` resta served
  - `v2` served
  - solo una versione deve avere `storage: true`

- Validazione
  - `kubectl get crd myapps.stable.example.com -o yaml`

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: myapps.stable.example.com
spec:
  group: stable.example.com
  scope: Namespaced
  names:
    plural: myapps
    singular: myapp
    kind: MyApp
    shortNames:
    - ma
  versions:
  - name: v1
    served: true
    storage: false
    schema:
      openAPIV3Schema:
        type: object
        properties:
          spec:
            type: object
            properties:
              color:
                type: string

  - name: v2
    served: true
    storage: true
    schema:
      openAPIV3Schema:
        type: object
        properties:
          spec:
            type: object
            properties:
              color:
                type: string
              size:
                type: integer
```

```sh
kubectl apply -f crd.yaml
kubectl get crd myapps.stable.example.com -o yaml
```

</details>

---

## CRD-4 — Creare una risorsa usando v2

- Creare una Custom Resource `MyApp`
- Versione API: `stable.example.com/v2`
- Nome: `advanced-app`

- Specifiche
  - `color=green`
  - `size=3`

- Validazione
  - `kubectl get myapps`
  - `kubectl get myapps advanced-app -o yaml`

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: stable.example.com/v2
kind: MyApp
metadata:
  name: advanced-app
spec:
  color: green
  size: 3
```

```sh
kubectl apply -f myapp-v2.yaml
kubectl get myapps
kubectl get myapps advanced-app -o yaml
```

</details>

---

## CRD-5 — Abilitare subresource status

- Modificare il CRD `myapps.stable.example.com`
- Abilitare subresource `status`

- Validazione
  - `kubectl get crd myapps.stable.example.com -o yaml`
  - la definizione contiene `subresources.status`

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: myapps.stable.example.com
spec:
  group: stable.example.com
  scope: Namespaced
  names:
    plural: myapps
    singular: myapp
    kind: MyApp
    shortNames:
    - ma
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
            properties:
              color:
                type: string
          status:
            type: object
            properties:
              phase:
                type: string
    subresources:
      status: {}
```

```sh
kubectl apply -f crd-status.yaml
kubectl get crd myapps.stable.example.com -o yaml
```

</details>

---

## CRD-6 — RBAC per CRD

- Namespace: `default`
- Creare Role: `myapp-reader`

- Permessi
  - `get`, `list`, `watch` su `myapps`
  - apiGroup: `stable.example.com`

- Creare ServiceAccount: `crd-user`
- Creare RoleBinding

- Validazione
  - `kubectl auth can-i get myapps --as=system:serviceaccount:default:crd-user -n default`

<details>
<summary>Soluzione</summary>

```sh
kubectl create sa crd-user -n default

kubectl create role myapp-reader \
  --verb=get,list,watch \
  --resource=myapps \
  --api-group=stable.example.com \
  -n default

kubectl create rolebinding myapp-reader-binding \
  --role=myapp-reader \
  --serviceaccount=default:crd-user \
  -n default

kubectl auth can-i get myapps \
  --as=system:serviceaccount:default:crd-user \
  -n default
```

</details>

---

## CRD-7 — Validazione schema: campo richiesto

- Modificare il CRD `myapps.stable.example.com`
- Rendere `spec.color` obbligatorio

- Validazione
  - una CR senza `spec.color` deve fallire

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: myapps.stable.example.com
spec:
  group: stable.example.com
  scope: Namespaced
  names:
    plural: myapps
    singular: myapp
    kind: MyApp
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
            - color
            properties:
              color:
                type: string
```

Esempio di CR non valida:

```yaml
apiVersion: stable.example.com/v1
kind: MyApp
metadata:
  name: invalid-app
spec: {}
```

```sh
kubectl apply -f invalid-app.yaml
```

</details>

---

## CRD-8 — Additional printer columns

- Modificare il CRD `myapps.stable.example.com`
- Aggiungere una printer column che mostri `spec.color`

- Validazione
  - `kubectl get myapps` mostra colonna `Color`

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: myapps.stable.example.com
spec:
  group: stable.example.com
  scope: Namespaced
  names:
    plural: myapps
    singular: myapp
    kind: MyApp
  versions:
  - name: v1
    served: true
    storage: true
    additionalPrinterColumns:
    - name: Color
      type: string
      jsonPath: .spec.color
    schema:
      openAPIV3Schema:
        type: object
        properties:
          spec:
            type: object
            properties:
              color:
                type: string
```

```sh
kubectl apply -f crd-columns.yaml
kubectl get myapps
```

</details>

---

## CRD-9 — Simulare il comportamento di un Operator

- Scenario
  - Esiste una Custom Resource `MyApp`
  - `spec.color=blue`
  - `spec.size=2`

- Obiettivo
  - Simulare manualmente ciò che farebbe un Operator:
    - creare un Deployment con:
      - nome uguale alla CR
      - replicas uguale a `spec.size`
      - label `color=<spec.color>`

- Validazione
  - Deployment creato coerentemente con la CR

<details>
<summary>Soluzione</summary>

Custom Resource:

```yaml
apiVersion: stable.example.com/v2
kind: MyApp
metadata:
  name: simulated-app
spec:
  color: blue
  size: 2
```

Deployment simulato “gestito” dall’operator:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: simulated-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: simulated-app
      color: blue
  template:
    metadata:
      labels:
        app: simulated-app
        color: blue
    spec:
      containers:
      - name: nginx
        image: nginx
```

```sh
kubectl apply -f myapp.yaml
kubectl apply -f simulated-deploy.yaml
kubectl get deploy
```

</details>

---

## CRD-10 — Aggiornare status manualmente (simulazione operator)

- Scenario
  - CRD con subresource `status` abilitata
  - Custom Resource: `example-app`

- Obiettivo
  - Aggiornare manualmente il campo:
    - `status.phase=Running`

- Validazione
  - `kubectl get myapps example-app -o yaml`

<details>
<summary>Soluzione</summary>

Esempio di patch:

```sh
kubectl patch myapp example-app \
  --subresource=status \
  --type=merge \
  -p '{"status":{"phase":"Running"}}'
```

Verifica:

```sh
kubectl get myapps example-app -o yaml
```

</details>

---

## Cheatsheet rapido

```sh
# vedere le CRD
kubectl get crd

# spiegare una CRD / risorsa custom
kubectl explain myapps

# vedere istanze della custom resource
kubectl get myapps
kubectl get ma

# vedere il manifest completo di una CRD
kubectl get crd myapps.stable.example.com -o yaml

# vedere se un SA può leggere una CRD
kubectl auth can-i get myapps \
  --as=system:serviceaccount:default:crd-user \
  -n default
```
## CRD-11

Task  
SECTION: APPLICATION EXTENSIBILITY (CRD)

Create a CustomResourceDefinition named `books.library.io` with the following properties:

- Group: `library.io`
- Version: `v1`
- Kind: `Book`
- Plural: `books`
- Scope: `Namespaced`

The CRD should define the following fields under `spec`:

- `title` as string
- `author` as string
- `pages` as integer

---

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: books.library.io
spec:
  group: library.io
  scope: Namespaced
  names:
    plural: books
    singular: book
    kind: Book
    shortNames:
    - bk
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
            properties:
              title:
                type: string
              author:
                type: string
              pages:
                type: integer
```

```bash

kubectl apply -f book-crd.yaml
kubectl get crd books.library.io
```

</details>

---

## Q. 12

Task  
SECTION: APPLICATION EXTENSIBILITY (CRD)


Create a custom resource of kind `Book` with the following specifications:

- Name: `my-book`
- title: `Kubernetes Guide`
- author: `John Doe`
- pages: `420`

---

<details>
<summary>Soluzione</summary>

> Prima assicurati che la CRD dell'esercizio precedente esista.

```yaml
apiVersion: library.io/v1
kind: Book
metadata:
  name: my-book
spec:
  title: "Kubernetes Guide"
  author: "John Doe"
  pages: 420
```

```bash

kubectl apply -f my-book.yaml
kubectl get books
kubectl get book my-book -o yaml
```

</details>

---

## Q. 13

Task  
SECTION: APPLICATION EXTENSIBILITY (CRD)


Inspect the CRD `books.library.io` and find all available fields under `spec`.

---

<details>
<summary>Soluzione</summary>

```bash


kubectl get crd books.library.io -o json | jq '.spec.versions[].schema.openAPIV3Schema.properties.spec.properties'

kubectl explain book.spec
```

Expected fields:

- `title`
- `author`
- `pages`
  

</details>

---

## Q. 14

Task  
SECTION: APPLICATION EXTENSIBILITY (CRD)

Create a CustomResourceDefinition named `movies.media.io` with the following properties:

- Group: `media.io`
- Version: `v1alpha1`
- Kind: `Movie`
- Plural: `movies`
- Scope: `Namespaced`

The CRD should define the following fields under `spec`:

- `movieName` as string
- `duration` as integer

---

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: movies.media.io
spec:
  group: media.io
  scope: Namespaced
  names:
    plural: movies
    singular: movie
    kind: Movie
    shortNames:
    - mv
  versions:
  - name: v1alpha1
    served: true
    storage: true
    schema:
      openAPIV3Schema:
        type: object
        properties:
          spec:
            type: object
            properties:
              movieName:
                type: string
              duration:
                type: integer
```

```bash

kubectl apply -f movie-crd.yaml
kubectl get crd movies.media.io
```

</details>

---

## Q. 15

Task  
SECTION: APPLICATION EXTENSIBILITY (CRD)

Create a custom resource of kind `Movie` with the following specifications:

- Name: `inception`
- movieName: `Inception`
- duration: `148`

---

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: media.io/v1alpha1
kind: Movie
metadata:
  name: inception
spec:
  movieName: "Inception"
  duration: 148
```

```bash

kubectl apply -f inception.yaml
kubectl get movies
kubectl get movie inception -o yaml
```

</details>

---

## Q. 16

Task  
SECTION: APPLICATION EXTENSIBILITY (CRD)

List all custom resources of kind `Movie`.

---

<details>
<summary>Soluzione</summary>

```bash

kubectl get movies
kubectl get movie
kubectl get movies -o wide
```

</details>

---

## Q. 17

Task  
SECTION: APPLICATION EXTENSIBILITY (CRD)

Delete the CRD `movies.media.io` and observe what happens to the custom resources created from it.

---

<details>
<summary>Soluzione</summary>

```bash


kubectl get movies
kubectl delete crd movies.media.io
kubectl get crd
kubectl get movies
```

Expected result:

- after deleting the CRD, the associated custom resources are removed and the resource type is no longer available.


</details>

---

## Q. 18

Task  
SECTION: APPLICATION EXTENSIBILITY (CRD)

Create a CustomResourceDefinition named `apps.company.io` with the following properties:

- Group: `company.io`
- Version: `v1`
- Kind: `App`
- Plural: `apps`
- Scope: `Cluster`

The CRD should define the following fields under `spec`:

- `appName` as string
- `team` as string

---

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: apps.company.io
spec:
  group: company.io
  scope: Cluster
  names:
    plural: apps
    singular: app
    kind: App
    shortNames:
    - capp
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
            properties:
              appName:
                type: string
              team:
                type: string
```

```bash

kubectl apply -f app-crd.yaml
kubectl get crd apps.company.io
```

</details>

---

## Q. 19

Task  
SECTION: APPLICATION EXTENSIBILITY (CRD)


Create a cluster-scoped custom resource of kind `App` with the following specifications:

- Name: `internal-portal`
- appName: `internal-portal`
- team: `platform`

---

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: company.io/v1
kind: App
metadata:
  name: internal-portal
spec:
  appName: "internal-portal"
  team: "platform"
```

```bash

kubectl apply -f internal-portal.yaml
kubectl get apps
kubectl get app internal-portal -o yaml
```

</details>

---

## Q. 20

Task  
SECTION: APPLICATION EXTENSIBILITY (CRD)


Update the CRD `books.library.io` to add a new field under `spec`:

- `version` as string

---

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: books.library.io
spec:
  group: library.io
  scope: Namespaced
  names:
    plural: books
    singular: book
    kind: Book
    shortNames:
    - bk
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
            properties:
              title:
                type: string
              author:
                type: string
              pages:
                type: integer
              version:
                type: string
```

```bash

kubectl apply -f book-crd.yaml
kubectl get crd books.library.io -o json | jq '.spec.versions[].schema.openAPIV3Schema.properties.spec.properties'
```

</details>

---

### Q. 21

Task  
SECTION: APPLICATION EXTENSIBILITY (CRD)

Create a namespace named `crd-lab` and then create a `Book` custom resource in that namespace:

- Name: `networking-book`
- title: `Kubernetes Networking`
- author: `Jane Roe`
- pages: `310`
- version: `second-edition`

---

<details>
<summary>Soluzione</summary>

```bash

kubectl create namespace crd-lab
```

```yaml
apiVersion: library.io/v1
kind: Book
metadata:
  name: networking-book
  namespace: crd-lab
spec:
  title: "Kubernetes Networking"
  author: "Jane Roe"
  pages: 310
  version: "second-edition"
```

```bash
kubectl apply -f networking-book.yaml
kubectl get books -n crd-lab
```

</details>

---

### Q. 22

Task  
SECTION: APPLICATION EXTENSIBILITY (CRD)


Export the custom resource `my-book` to YAML.

---

<details>
<summary>Soluzione</summary>

```bash

kubectl get book my-book -o yaml > my-book-export.yaml
cat my-book-export.yaml
```

</details>

---



---

## Mini schema mentale

```text
CRD = definizione di un nuovo tipo di risorsa
CR  = istanza concreta di quel tipo

CRD -> schema / versioni / scope / subresources
CR  -> spec / status

Operator = controller che osserva le CR
         e crea/aggiorna risorse Kubernetes in base a spec
```
