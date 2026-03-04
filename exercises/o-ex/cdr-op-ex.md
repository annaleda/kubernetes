
### CRD and Operators (6 esercizi)

## CRD-1 — Creare CRD Base

- Definire CRD: `myapps.stable.example.com`
- Kind: `myApp`
- Version: v1
- Scope: Namespaced
- Validazione
  - `kubectl get crd`
---
<details>
<summary>Soluzione</summary>
  
```
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
    kind: myApp
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

k get crd

apiVersion: stable.example.com/v1
kind: myApp
metadata:
  name: test-app
spec:
  color: "blue"

k get ma
NAME       AGE
test-app   11s
```
</details>

---

## CRD-2 — Creare Custom Resource

- Creare oggetto `MyApp`
  - Nome: example-app
- Validazione
  - `kubectl get myapp`

---
<details>
<summary>Soluzione</summary>
  
```
apiVersion: stable.example.com/v1
kind: myApp
metadata:
  name: test-app
spec:
  color: "blue"

k get ma
NAME       AGE
test-app   11s
```
</details>

---

## CRD-3 — Versioning CRD

- Aggiungere versione v2 al CRD
- Validazione
  - CRD mostra multiple versioni
---
<details>
<summary>Soluzione</summary>
  
```  
```
</details>

---

### CRD-4 — Status Subresource

- Abilitare subresource status nel CRD
- Validazione
  - `kubectl get myapp -o yaml` mostra status

---
<details>
<summary>Soluzione</summary>
  
```
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
    kind: myApp
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
</details>

---

### CRD-5 — RBAC per CRD

- Creare Role che permette get su MyApp
- Validazione
  - `kubectl auth can-i get myapps`
---
<details>
<summary>Soluzione</summary>
  
```
k auth can-i get myapps
k create role myapp-reader --verb=get --resource=myapps --api-group=stable.example.com -n default
# notare --api-groups !

k create sa crd-user -n default
k create rolebinding myapp-reader-binding --role=myapp-reader --serviceaccount=default:crd-user -n default

k auth can-i get myapps --as=system:serviceaccount:default:crd-user -n default

```
</details>

---

### CRD-6 — Concept Operator

- Descrivere come un Operator controller osserva MyApp
- Simulare creando Deployment basato su campo spec
- Validazione
  - Deployment creato manualmente come simulazione
---
<details>
<summary>Soluzione</summary>
  
```  
```
</details>

---
