
Mock Exercises – Node / Taints / Affinity (con soluzioni)
1. NodeSelector – Simple

Obiettivo: schedulare un pod solo su nodi con label disktype=ssd.

- Esercizio
  - Name: pod-ssd
  - NodeSelector: disktype=ssd
  - Image: nginx

Soluzione:
```
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
> Spiegazione: NodeSelector è il modo più semplice: match esatto label=value.

2. NodeAffinity – Required / Exists

Obiettivo: schedulare un deployment su tutti i controlplane nodes (label key presente, value vuoto).

- Esercizio
  - Name: red
  - Replicas: 2
  - Image: nginx
  - NodeAffinity: requiredDuringSchedulingIgnoredDuringExecution
  - Key: node-role.kubernetes.io/control-plane
  - Operator: Exists

Soluzione:
```
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
> Spiegazione: Key esiste ma value è vuoto → operator Exists.

3. NodeAffinity – Required / In

Obiettivo: schedulare un pod solo su nodi con label color=blue o color=green.

- Esercizio
  - Name: blue-green
  - Replicas: 3
  - Image: nginx
  - NodeAffinity: requiredDuringSchedulingIgnoredDuringExecution
  - Key: color
  - Operator: In
  - Values: [blue, green]

Soluzione:
```
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
> Spiegazione: Multiple valori → operator In.

4. NodeAffinity – Preferred / In

Obiettivo: preferire nodi con disktype=ssd ma non obbligatorio.

- Soluzione

```
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
> Spiegazione: PreferredAffinity indica solo preferenza, non obbligo.

5. NodeAffinity – Required / DoesNotExist

Obiettivo: schedulare pod solo su nodi che non hanno label maintenance=true.

- Soluzione
```
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
      - matchExpressions:
        - key: maintenance
          operator: DoesNotExist
```
> Spiegazione: operator DoesNotExist blocca i nodi con quella label.

6. PodAffinity – Required

Obiettivo: schedulare pod vicino ad altri pod con label app=frontend.

- Soluzione
```
affinity:
  podAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
    - labelSelector:
        matchLabels:
          app: frontend
      topologyKey: kubernetes.io/hostname
```
> Spiegazione: PodAffinity → schedula sullo stesso nodo dove girano pod matching.

7. PodAntiAffinity – Required

Obiettivo: schedulare pod lontano da altri pod con label app=frontend.

- Soluzione
```
affinity:
  podAntiAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
    - labelSelector:
        matchLabels:
          app: frontend
      topologyKey: kubernetes.io/hostname
```
> Spiegazione: PodAntiAffinity → evita nodi dove ci sono pod matching.

8. Taints & Tolerations – NoSchedule

Obiettivo: schedulare pod su nodo con taint dedicated=blue:NoSchedule.

- Soluzione
```
tolerations:
- key: dedicated
  operator: Equal
  value: blue
  effect: NoSchedule
```
> Spiegazione: Senza questa toleration, il pod non viene schedulato.

9. Taints & Tolerations – PreferNoSchedule

Obiettivo: pod preferisce non andare su nodi con taint gpu=exclusive:PreferNoSchedule.

- Soluzione
```
tolerations:
- key: gpu
  operator: Equal
  value: exclusive
  effect: PreferNoSchedule
```
> Spiegazione: Effetto “soft”: scheduler evita il nodo se possibile, ma non obbligatorio.

10. NodeAffinity + Taints & Tolerations combinato

Obiettivo: schedulare un pod su nodi color=blue e che tollerano la taint dedicated=blue:NoSchedule.

- Soluzione
```
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
> Spiegazione: Combinazione di obbligo (nodeAffinity) + tollerazione taint (NoSchedule).
