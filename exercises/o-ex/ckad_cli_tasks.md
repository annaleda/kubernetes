- [ Home ](../../readme.md)   | [ Teoria ](../../arguments.md)   | [ Info Exam ](../../doc/ckad_exam_strategy.md)    | [ Home Other Exercises ](../o_exercises.md)   
--- 

### CKAD CLI Tasks (10 esercizi)
---

## CLI-1 — api-resources by api-version

SECTION: APPLICATION OBSERVABILITY AND MAINTENANCE

- Obiettivo
  - Identificare tutte le risorse con apiVersion:
    - `apps/v1`
  - Salvare output in:
    - `/root/apps-api.txt`

---

<details>
<summary>Soluzione</summary>

```sh
kubectl api-resources --api-group=apps -o name > /root/apps-api.txt
```

</details>

---

## CLI-2 — Pod status export

SECTION: APPLICATION OBSERVABILITY AND MAINTENANCE

- Namespace: default

- Obiettivo
  - Stampare:
    - NAME
    - STATUS
  - Salvare in `/root/pod-status.txt`

---

<details>
<summary>Soluzione</summary>

```sh
kubectl get pods -o custom-columns="NAME:.metadata.name,STATUS:.status.phase" > /root/pod-status.txt
```

</details>

---

## CLI-3 — Node info

SECTION: APPLICATION OBSERVABILITY AND MAINTENANCE

- Obiettivo
  - Stampare:
    - NAME
    - OS-IMAGE
  - Salvare in `/root/node-info.txt`

---

<details>
<summary>Soluzione</summary>

```sh
kubectl get nodes -o custom-columns="NAME:.metadata.name,OS:.status.nodeInfo.osImage" > /root/node-info.txt
```

</details>

---

## CLI-4 — Services type filter

SECTION: APPLICATION OBSERVABILITY AND MAINTENANCE

- Obiettivo
  - Elencare solo Service di tipo `NodePort`
  - Salvare in `/root/nodeport-svc.txt`

---

<details>
<summary>Soluzione</summary>

```sh
kubectl get svc --field-selector spec.type=NodePort > /root/nodeport-svc.txt
```

</details>

---

## CLI-5 — PVC status

SECTION: APPLICATION OBSERVABILITY AND MAINTENANCE

- Obiettivo
  - Stampare:
    - NAME
    - STATUS
    - STORAGE
  - Salvare in `/root/pvc.txt`

---

<details>
<summary>Soluzione</summary>

```sh
kubectl get pvc -o custom-columns="NAME:.metadata.name,STATUS:.status.phase,STORAGE:.spec.resources.requests.storage" > /root/pvc.txt
```

</details>

---

## CLI-6 — Pods per namespace

SECTION: APPLICATION OBSERVABILITY AND MAINTENANCE

- Namespace: kube-system

- Obiettivo
  - Elencare Pod
  - Salvare output in `/root/system-pods.txt`

---

<details>
<summary>Soluzione</summary>

```sh
kubectl -n kube-system get pods > /root/system-pods.txt
```

</details>

---

## CLI-7 — Deployment replicas

SECTION: APPLICATION OBSERVABILITY AND MAINTENANCE

- Obiettivo
  - Stampare:
    - NAME
    - REPLICAS
  - Salvare in `/root/deploy.txt`

---

<details>
<summary>Soluzione</summary>

```sh
kubectl get deploy -o custom-columns="NAME:.metadata.name,REPLICAS:.spec.replicas" > /root/deploy.txt
```

</details>

---

## CLI-8 — Events sorted

SECTION: APPLICATION OBSERVABILITY AND MAINTENANCE

- Obiettivo
  - Elencare eventi ordinati per timestamp
  - Salvare in `/root/events.txt`

---

<details>
<summary>Soluzione</summary>

```sh
kubectl get events --sort-by=.metadata.creationTimestamp > /root/events.txt
```

</details>

---

## CLI-9 — Container images

SECTION: APPLICATION OBSERVABILITY AND MAINTENANCE

- Obiettivo
  - Stampare:
    - POD
    - IMAGE
  - Salvare in `/root/images.txt`

---

<details>
<summary>Soluzione</summary>

```sh
kubectl get pods -o custom-columns="NAME:.metadata.name,IMAGE:.spec.containers[*].image" > /root/images.txt
```

</details>

---

## CLI-10 — API groups

SECTION: APPLICATION OBSERVABILITY AND MAINTENANCE

- Obiettivo
  - Elencare tutti gli api groups disponibili
  - Salvare in `/root/api-groups.txt`

---

<details>
<summary>Soluzione</summary>

```sh
kubectl api-versions > /root/api-groups.txt
```

</details>

---
