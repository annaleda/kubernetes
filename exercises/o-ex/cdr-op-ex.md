- [ Home ](../../readme.md)   | [ Teoria ](../../arguments.md)   | [ Info Exam ](../../doc/ckad_exam_strategy.md)    | [ Home Other Exercises ](../o_exercises.md)   
--- 
### CRD and Operators (10 esercizi)

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
