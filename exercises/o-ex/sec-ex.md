### Security Context (6 esercizi)
---

## SEC-1 — RunAsUser

- Namespace: `secure-apps`
- Pod: `secure-pod`

- Task:
  - Container deve girare come user ID 1000
  - Image: nginx

- Validazione:
  - Controllare user nel container

<details> <summary>Soluzione</summary>
  
```
k create ns secure-apps

k run secure-pod --image=nginx -n secure-apps --dry-run=client -o yaml > secure-pod.yaml

apiVersion: v1
kind: Pod
metadata:
  labels:
    run: secure-pod
  name: secure-pod
  namespace: secure-apps
spec:
  containers:
  - image: nginx
    name: secure-pod
  securityContext:
    runAsUser: 1000


k apply -f secure-pod.yaml

```
</details>

---

## SEC-2 — ReadOnlyRootFilesystem

- Namespace: `readonly-ns`
- Pod: `readonly-app`

- Task:
  - Root filesystem deve essere read-only

- Validazione:
  - Container non può scrivere su filesystem

<details> <summary>Soluzione</summary>
  
```
k create ns readonly-ns

k run readonly-app --image=nginx -n readonly-ns --dry-run=client -o yaml > rpod.yaml

apiVersion: v1
kind: Pod
metadata:
  labels:
    run: readonly-app
  name: readonly-app
  namespace: readonly-ns
spec:
  containers:
  - image: nginx
    name: readonly-app
    securityContext:
      readOnlyRootFilesystem: true

```
</details>

## SEC-3 — Privileged Container

- Namespace: `privileged-ns`
- Pod: `priv-app`

- Task:
  - Container deve girare in privileged mode

- Validazione:
  - Privileged flag attivo

<details> <summary>Soluzione</summary>

  ```
  k create ns privileged-ns

  k run priv-app --image=nginx -n privileged-ns --dry-run=client -o yaml > privpod.yaml


  apiVersion: v1
  kind: Pod
  metadata:
    labels:
      run: priv-app
    name: priv-app
    namespace: privileged-ns
  spec:
    containers:
    - image: nginx
      name: priv-app
      securityContext:
        privileged: true

  ```
</details>

---

## SEC-4 — Capabilities Drop

- Namespace: `cap-ns`
- Pod: `cap-app`

- Task:
  - Rimuovere capability NET_RAW
- Validazione:
  - Capability non presente

<details> <summary>Soluzione</summary>
 
```
  k create ns cap-ns

  k run cap-app --image=nginx -n cap-ns --dry-run=client -o yaml > cap-pod.yaml


  apiVersion: v1
  kind: Pod
  metadata:
    creationTimestamp: null
    labels:
      run: cap-app
    name: cap-app
    namespace: cap-ns
  spec:
    containers:
    - image: nginx
      name: cap-app
      securityContext:
        capabilities:
          drop:
          - NET_RAW
 ```
</details>

---

## SEC-5 — Pod + Container Security Context Combined

- Namespace: `mix-sec-ns`
- Pod: `mix-sec-app`

- runAsUser: 2000 (pod level)

- Container:
  - runAsUser: 3000 (override container level)

- Validazione:
  - Container level ha priorità

<details> <summary>Soluzione</summary>

```
k create ns mix-sec-ns

k run mix-sec-app --image=nginx -n mix-sec-ns --dry-run=client -o yaml > mix-sec-pod.yaml

apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: mix-sec-app
  name: mix-sec-app
  namespace: mix-sec-ns
spec:
  securityContext:
    runAsUser: 2000
  containers:
  - image: nginx
    name: mix-sec-app
    securityContext:
      runAsUser: 3000

```
  
</details>

---

## SEC-6 — Allow Privilege Escalation

- Namespace: `escalation-ns`
- Pod: `escalate-app`

- Task:
  - Consentire privilege escalation

- Validazione:
  - allowPrivilegeEscalation = true

<details> <summary>Soluzione</summary>

```
k create ns escalation-ns

k run escalation-app --image=nginx -n escalation-ns --dry-run=client -o yaml > escalation-pod.yaml

apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: escalation-app
  name: escalation-app
  namespace: escalation-ns
spec:
  containers:
  - image: nginx
    name: escalation-app
    securityContext:
      allowPrivilegeEscalation: true
```
</details>


---

