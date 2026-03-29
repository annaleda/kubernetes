- [ Home ](../../readme.md)   | [ Teoria ](../../arguments.md)   | [ Info Exam ](../../doc/ckad_exam_strategy.md)    | [ Home Other Exercises ](../o_exercises.md)   
--- 
### Node / Taints / Affinity (15 esercizi)

---

## NA-1 — NodeSelector semplice

- Pod: `pod-ssd`
- Image: `nginx`
- NodeSelector:
  - `disktype=ssd`

- Validazione
  - Il Pod viene schedulato solo su un nodo con label `disktype=ssd`

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-ssd
spec:
  containers:
  - name: nginx
    image: nginx
  nodeSelector:
    disktype: ssd
```

</details>

---

## NA-2 — NodeAffinity Required con operator Exists

- Deployment: `red`
- Replicas: `2`
- Image: `nginx`

- NodeAffinity:
  - type: `requiredDuringSchedulingIgnoredDuringExecution`
  - key: `node-role.kubernetes.io/control-plane`
  - operator: `Exists`

- Validazione
  - I Pod vengono schedulati solo su nodi control-plane

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: red
spec:
  replicas: 2
  selector:
    matchLabels:
      app: red
  template:
    metadata:
      labels:
        app: red
    spec:
      containers:
      - name: nginx
        image: nginx
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: node-role.kubernetes.io/control-plane
                operator: Exists
```

</details>

---

## NA-3 — NodeAffinity Required con operator In

- Deployment: `blue-green`
- Replicas: `3`
- Image: `nginx`

- NodeAffinity:
  - type: `requiredDuringSchedulingIgnoredDuringExecution`
  - key: `color`
  - operator: `In`
  - values:
    - `blue`
    - `green`

- Validazione
  - I Pod vengono schedulati solo su nodi con `color=blue` oppure `color=green`

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: blue-green
spec:
  replicas: 3
  selector:
    matchLabels:
      app: blue-green
  template:
    metadata:
      labels:
        app: blue-green
    spec:
      containers:
      - name: nginx
        image: nginx
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: color
                operator: In
                values:
                - blue
                - green
```

</details>

---

## NA-4 — NodeAffinity Preferred

- Pod: `prefer-ssd`
- Image: `nginx`

- NodeAffinity:
  - type: `preferredDuringSchedulingIgnoredDuringExecution`
  - preferire nodi con `disktype=ssd`

- Validazione
  - Il Pod preferisce nodi SSD, ma può essere schedulato anche altrove

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: prefer-ssd
spec:
  containers:
  - name: nginx
    image: nginx
  affinity:
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1
        preference:
          matchExpressions:
          - key: disktype
            operator: In
            values:
            - ssd
```

</details>

---

## NA-5 — NodeAffinity DoesNotExist

- Pod: `no-maintenance`
- Image: `nginx`

- NodeAffinity:
  - type: `requiredDuringSchedulingIgnoredDuringExecution`
  - key: `maintenance`
  - operator: `DoesNotExist`

- Validazione
  - Il Pod non viene schedulato su nodi che hanno la label `maintenance`

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: no-maintenance
spec:
  containers:
  - name: nginx
    image: nginx
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: maintenance
            operator: DoesNotExist
```

</details>

---

## NA-6 — PodAffinity Required

- Pod: `affinity-app`
- Image: `nginx`

- Obiettivo
  - Schedulare il Pod sullo stesso nodo di altri Pod con label `app=frontend`

- Validazione
  - Il Pod viene schedulato vicino ai Pod frontend

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: affinity-app
  labels:
    app: affinity-app
spec:
  containers:
  - name: nginx
    image: nginx
  affinity:
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchLabels:
            app: frontend
        topologyKey: kubernetes.io/hostname
```

</details>

---

## NA-7 — PodAntiAffinity Required

- Pod: `anti-affinity-app`
- Image: `nginx`

- Obiettivo
  - Evitare nodi dove girano già Pod con label `app=frontend`

- Validazione
  - Il Pod non viene schedulato sullo stesso nodo dei Pod frontend

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: anti-affinity-app
  labels:
    app: anti-affinity-app
spec:
  containers:
  - name: nginx
    image: nginx
  affinity:
    podAntiAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchLabels:
            app: frontend
        topologyKey: kubernetes.io/hostname
```

</details>

---

## NA-8 — Taints e Tolerations con NoSchedule

- Pod: `toleration-app`
- Image: `nginx`

- Nodo target:
  - taint: `dedicated=blue:NoSchedule`

- Obiettivo
  - Permettere al Pod di essere schedulato su quel nodo

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: toleration-app
spec:
  containers:
  - name: nginx
    image: nginx
  tolerations:
  - key: dedicated
    operator: Equal
    value: blue
    effect: NoSchedule
```

</details>

---

## NA-9 — Toleration con PreferNoSchedule

- Pod: `soft-toleration`
- Image: `nginx`

- Nodo target:
  - taint: `gpu=exclusive:PreferNoSchedule`

- Obiettivo
  - Il Pod tollera il taint soft

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: soft-toleration
spec:
  containers:
  - name: nginx
    image: nginx
  tolerations:
  - key: gpu
    operator: Equal
    value: exclusive
    effect: PreferNoSchedule
```

</details>

---

## NA-10 — NodeAffinity + Toleration combinate

- Pod: `combo-app`
- Image: `nginx`

- Obiettivo
  - Schedulare il Pod su nodi:
    - con label `color=blue`
    - con taint `dedicated=blue:NoSchedule`

- Validazione
  - Il Pod richiede il nodo blu e tollera il taint corretto

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: combo-app
spec:
  containers:
  - name: nginx
    image: nginx
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: color
            operator: In
            values:
            - blue
  tolerations:
  - key: dedicated
    operator: Equal
    value: blue
    effect: NoSchedule
```

</details>

---

## NA-11 — Debug conflitto tra NodeSelector e NodeAffinity

- Pod: `conflict-app`
- Image: `nginx`

- Configurazione richiesta:
  - `nodeSelector: color=blue`
  - `nodeAffinity` richiede `color=green`

- Obiettivo
  - Identificare perché il Pod resta Pending
  - Correggere il conflitto

<details>
<summary>Soluzione</summary>

Problema: `nodeSelector` e `nodeAffinity` sono in conflitto.

Manifest di esempio con conflitto:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: conflict-app
spec:
  containers:
  - name: nginx
    image: nginx
  nodeSelector:
    color: blue
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: color
            operator: In
            values:
            - green
```

Fix possibile:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: conflict-app
spec:
  containers:
  - name: nginx
    image: nginx
  nodeSelector:
    color: blue
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: color
            operator: In
            values:
            - blue
```

</details>

---

## NA-12 — PodAntiAffinity Preferred

- Deployment: `spread-app`
- Replicas: `3`
- Image: `nginx`

- Obiettivo
  - Preferire la distribuzione dei Pod su nodi diversi

- Validazione
  - Lo scheduler tende a distribuire i Pod

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: spread-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: spread-app
  template:
    metadata:
      labels:
        app: spread-app
    spec:
      containers:
      - name: nginx
        image: nginx
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            podAffinityTerm:
              labelSelector:
                matchLabels:
                  app: spread-app
              topologyKey: kubernetes.io/hostname
```

</details>

---

## NA-13 — Toleration con NoExecute

- Pod: `noexecute-app`
- Image: `nginx`

- Nodo target:
  - taint con effect `NoExecute`
  - key: `critical`

- Obiettivo
  - Il Pod deve tollerare il taint `NoExecute`

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: noexecute-app
spec:
  containers:
  - name: nginx
    image: nginx
  tolerations:
  - key: critical
    operator: Exists
    effect: NoExecute
```

</details>

---

## NA-14 — NodeAffinity con più matchExpressions

- Pod: `multi-match-app`
- Image: `nginx`

- Obiettivo
  - Schedulare solo su nodi con:
    - `env=prod`
    - `tier=backend`

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-match-app
spec:
  containers:
  - name: nginx
    image: nginx
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: env
            operator: In
            values:
            - prod
          - key: tier
            operator: In
            values:
            - backend
```

</details>

---

## NA-15 — PodAffinity con topology per zona

- Pod: `zone-affinity-app`
- Image: `nginx`

- Obiettivo
  - Schedulare il Pod vicino a Pod `app=frontend`, ma a livello zona

- TopologyKey:
  - `topology.kubernetes.io/zone`

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: zone-affinity-app
spec:
  containers:
  - name: nginx
    image: nginx
  affinity:
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchLabels:
            app: frontend
        topologyKey: topology.kubernetes.io/zone
```

</details>

---

## Cheatsheet rapido

```sh
# vedere label nodi
kubectl get nodes --show-labels

# aggiungere label a un nodo
kubectl label node <node-name> disktype=ssd

# aggiungere taint a un nodo
kubectl taint nodes <node-name> dedicated=blue:NoSchedule

# rimuovere taint
kubectl taint nodes <node-name> dedicated=blue:NoSchedule-

# vedere perché un pod è Pending
kubectl describe pod <pod-name>
```

---

## Mini schema mentale

```text
nodeSelector   -> semplice match key=value
nodeAffinity   -> regole più potenti sui nodi
podAffinity    -> vicino ad altri pod
podAntiAffinity -> lontano da altri pod
taint          -> il nodo respinge
toleration     -> il pod può entrare
```
