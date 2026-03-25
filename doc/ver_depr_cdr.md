# CKAD Review: API Versions, Deprecations e CRD

---

##  1. API Versions in Kubernetes

Ogni risorsa Kubernetes definisce una versione API:

```yaml
apiVersion: <group>/<version>
kind: <Kind>
```

###  Esempi

Core API:

```yaml
apiVersion: v1
kind: Pod
```

Gruppi:

```yaml
apiVersion: apps/v1
kind: Deployment
```

---

##  Lifecycle delle API

| Livello     | Stabilità     | Note                    |
| ----------- | ------------- | ----------------------- |
| alpha       | instabile     | disabilitata di default |
| beta        | quasi stabile | può cambiare            |
| stable (v1) | stabile       | production-ready        |

---

##  Errori tipici CKAD

* Usare versioni deprecate
* Usare campi non più validi
* Non aggiornare `apiVersion`

---

##  2. Deprecations

###  Esempio classico

- Vecchio:

```yaml
apiVersion: extensions/v1beta1
kind: Deployment
```

- Nuovo:

```yaml
apiVersion: apps/v1
kind: Deployment
```

---

##  Altri esempi importanti

| Risorsa    | Vecchia            | Nuova                |
| ---------- | ------------------ | -------------------- |
| Deployment | extensions/v1beta1 | apps/v1              |
| Ingress    | extensions/v1beta1 | networking.k8s.io/v1 |
| CronJob    | batch/v1beta1      | batch/v1             |

---

##  Cambio importante (Deployment)

In `apps/v1`, il selector è obbligatorio:

```yaml
spec:
  selector:
    matchLabels:
      app: myapp
```

---

##  Strategia CKAD

Se trovi YAML vecchi:

1. Aggiorna `apiVersion`
2. Controlla campi obbligatori
3. Usa:

```bash
kubectl explain deployment --api-version=apps/v1
```

---

##  3. Verificare versioni disponibili

```bash
kubectl api-resources
```

```bash
kubectl api-versions
```

Per risorse specifiche:

```bash
kubectl explain ingress
```

---

##  4. CRD (Custom Resource Definitions)

###  Cos'è

Permette di creare nuovi tipi di risorse Kubernetes.

Esempio:

```yaml
kind: MyApp
```

---

##  Struttura CRD

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: myapps.example.com

spec:
  group: example.com
  names:
    kind: MyApp
    plural: myapps
  scope: Namespaced
  versions:
    - name: v1
      served: true
      storage: true
      schema:
        openAPIV3Schema:
          type: object
```

---

##  Uso della CRD

```yaml
apiVersion: example.com/v1
kind: MyApp
```

---

##  Errori tipici CRD

* Dimenticare `scope`
* Confondere `group` e `version`
* Nome plural errato
* Usare versioni deprecated

---

##  5. Versioning nelle CRD

```yaml
versions:
  - name: v1
    served: true
    storage: true
  - name: v2
    served: true
    storage: false
```

### Significato

* `served`: disponibile via API
* `storage`: versione salvata nel cluster

---

##  6. Conversion

Modalità:

* None (default)
* Webhook (avanzato)

---

##  7. Differenze chiave

| Concetto    | Significato            |
| ----------- | ---------------------- |
| API version | versione della risorsa |
| Deprecation | API rimossa            |
| CRD         | nuova risorsa custom   |

---

##  8. Checklist da esame

### YAML

* apiVersion corretta?
* Campi obbligatori?
* Risorsa valida?

### CRD

* Group corretto?
* Version definita?
* Nome plural?
* Schema presente?

---

##  9. Mini riassunto

* Deployment → `apps/v1`
* Ingress → `networking.k8s.io/v1`
* CronJob → `batch/v1`
* CRD → `apiextensions.k8s.io/v1`
* Selector obbligatorio nei Deployment

---

##  Consiglio finale

Nel CKAD:

 - Non memorizzare tutto
 - Usa velocemente:

```bash
kubectl explain
```

---

