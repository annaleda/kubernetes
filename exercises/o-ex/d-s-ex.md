* [ Home ](../../readme.md)   | [ Teoria ](../../arguments.md)   | [ Info Exam ](../../doc/ckad_exam_strategy.md)    | [ Home Other Exercises ](../o_exercises.md)

---

### Deployment Strategies (30 esercizi)

---

## DS-1 — Update immagine senza downtime

Creare Deployment `web-app`

* nginx 1.21

* 3 istanze

* Task

  * Aggiornare a nginx 1.25 senza downtime

---

<details>
<summary>Soluzione</summary>

```
k create deploy web-app --image=nginx:1.21 --replicas=3
k set image deployment/web-app nginx=nginx:1.25
k rollout status deployment web-app
```

</details>

---

## DS-2 — Verifica rollout

Deployment: `web-app`

* Task

  * Verificare stato aggiornamento

---

<details>
<summary>Soluzione</summary>

```
k rollout status deployment web-app
```

</details>

---

## DS-3 — Storico rollout

Deployment: `web-app`

---

<details>
<summary>Soluzione</summary>

```
k rollout history deployment web-app
```

</details>

---

## DS-4 — Rollback

Deployment: `web-app`

---

<details>
<summary>Soluzione</summary>

```
k rollout undo deployment web-app
```

</details>

---

## DS-5 — Rollback revisione specifica

Deployment: `web-app`

---

<details>
<summary>Soluzione</summary>

```
k rollout undo deployment web-app --to-revision=1
```

</details>

---

## DS-6 — Strategia Recreate

Creare `recreate-app`

* nginx 1.24
* 3 istanze
* aggiornamento distruttivo

---

<details>
<summary>Soluzione</summary>

```
k create deploy recreate-app --image=nginx:1.24 --replicas=3 --dry-run=client -o yaml > rec.yaml
vi rec.yaml
k apply -f rec.yaml
```

</details>

---

## DS-7 — Update con Recreate

Deployment: `recreate-app`

---

<details>
<summary>Soluzione</summary>

```
k set image deployment/recreate-app nginx=nginx:1.25
```

</details>

---

## DS-8 — maxUnavailable

Creare `controlled-app`

* nginx
* 4 istanze

---

<details>
<summary>Soluzione</summary>

```
# yaml con maxUnavailable:1
k apply -f controlled.yaml
```

</details>

---

## DS-9 — maxSurge

Deployment: `controlled-app`

---

<details>
<summary>Soluzione</summary>

```
# yaml con maxSurge:2
k apply -f controlled.yaml
```

</details>

---

## DS-10 — Pause rollout

<details>
<summary>Soluzione</summary>

```
k rollout pause deployment controlled-app
```

</details>

---

## DS-11 — Resume rollout

<details>
<summary>Soluzione</summary>

```
k rollout resume deployment controlled-app
```

</details>

---

## DS-12 — Restart rollout

<details>
<summary>Soluzione</summary>

```
k rollout restart deployment controlled-app
```

</details>

---

## DS-13 — Modifica con apply

<details>
<summary>Soluzione</summary>

```
k get deploy web-app -o yaml > d.yaml
vi d.yaml
k apply -f d.yaml
```

</details>

---

## DS-14 — Set image

<details>
<summary>Soluzione</summary>

```
k set image deployment/web-app nginx=nginx:1.26
```

</details>

---

## DS-15 — Scale

<details>
<summary>Soluzione</summary>

```
k scale deployment web-app --replicas=6
```

</details>

---

## DS-16 — Debug rollout

<details>
<summary>Soluzione</summary>

```
k rollout status deployment web-app
k describe deployment web-app
```

</details>

---

## DS-17 — Verifica Pod

<details>
<summary>Soluzione</summary>

```
k get pods -o wide
```

</details>

---

## DS-18 — Restart senza modifica

<details>
<summary>Soluzione</summary>

```
k rollout restart deployment web-app
```

</details>

---

## DS-19 — Update rapido

<details>
<summary>Soluzione</summary>

```
k set image deployment/web-app nginx=nginx:latest
```

</details>

---

## DS-20 — ReplicaSet

<details>
<summary>Soluzione</summary>

```
k get rs
```

</details>

---

## DS-21 — Errore immagine

<details>
<summary>Soluzione</summary>

```
k set image deployment/web-app nginx=nginx:wrong
k rollout status deployment web-app
```

</details>

---

## DS-22 — Rollback errore

<details>
<summary>Soluzione</summary>

```
k rollout undo deployment web-app
```

</details>

---

## DS-23 — Eventi

<details>
<summary>Soluzione</summary>

```
k describe deployment web-app
```

</details>

---

## DS-24 — Full ciclo

<details>
<summary>Soluzione</summary>

```
k create deploy full-cycle-app --image=nginx:1.21
k set image deployment/full-cycle-app nginx=nginx:1.25
k rollout status deployment full-cycle-app
k rollout undo deployment full-cycle-app
```

</details>

---

### STRATEGIES

---

## DS-25 — Blue/Green base

Creare:

* app-blue (v1)
* app-green (v2)

Service → punta a blue

---

<details>
<summary>Soluzione</summary>

```
k create deploy app-blue --image=nginx:1.21 --replicas=3
k create deploy app-green --image=nginx:1.25 --replicas=3
k expose deploy app-blue --name app-service --port=80
```

</details>

---

## DS-26 — Switch traffico

<details>
<summary>Soluzione</summary>

```
k edit svc app-service
```

</details>

---

## DS-27 — Rollback traffico

<details>
<summary>Soluzione</summary>

```
k edit svc app-service
```

</details>

---

## DS-28 — Canary base

<details>
<summary>Soluzione</summary>

```
k create deploy app-stable --image=nginx:1.21 --replicas=4
k create deploy app-canary --image=nginx:1.25 --replicas=1
k expose deploy app-stable --name app-service --port=80
```

</details>

---

## DS-29 — Incremento canary

<details>
<summary>Soluzione</summary>

```
k scale deployment app-canary --replicas=3
k scale deployment app-stable --replicas=2
```

</details>

---

## DS-30 — Canary → full

<details>
<summary>Soluzione</summary>

```
k scale deployment app-canary --replicas=5
k delete deployment app-stable
```

</details>

---
