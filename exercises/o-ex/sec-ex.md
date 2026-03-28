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

## SEC-7 — runAsNonRoot

- Namespace: `nonroot-ns`
- Pod: `nonroot-app`

- Task:
  - Impostare container per NON girare come root

- Validazione:
  - Pod fallisce se immagine usa root

---

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nonroot-app
  namespace: nonroot-ns
spec:
  containers:
  - name: app
    image: nginx
    securityContext:
      runAsNonRoot: true
```

 Se nginx gira come root → errore

```sh
k describe pod nonroot-app -n nonroot-ns
```

</details>

---

## SEC-8 — fsGroup (shared volume)

- Namespace: `fsgroup-ns`
- Pod: `fsgroup-app`

- Task:
  - Volume condiviso
  - fsGroup: 2000

- Obiettivo
  - Permessi corretti sui file

---

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: fsgroup-app
  namespace: fsgroup-ns
spec:
  securityContext:
    fsGroup: 2000

  volumes:
  - name: data
    emptyDir: {}

  containers:
  - name: app
    image: busybox
    command: ["sh","-c","sleep 3600"]
    volumeMounts:
    - name: data
      mountPath: /data
```

</details>

---

## SEC-9 — Capability Add

- Namespace: `cap-add-ns`
- Pod: `cap-add-app`

- Task:
  - Aggiungere capability NET_ADMIN

- Validazione:
  - Capability presente

---

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: cap-add-app
  namespace: cap-add-ns
spec:
  containers:
  - name: app
    image: nginx
    securityContext:
      capabilities:
        add:
        - NET_ADMIN
```

</details>

---

## SEC-10 — SeccompProfile

- Namespace: `seccomp-ns`
- Pod: `seccomp-app`

- Task:
  - Usare seccompProfile RuntimeDefault

- Validazione:
  - Profilo attivo

---

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: seccomp-app
  namespace: seccomp-ns
spec:
  securityContext:
    seccompProfile:
      type: RuntimeDefault

  containers:
  - name: app
    image: nginx
```

</details>

---

## SEC-11 — ReadOnly + Writable Volume (combo)

- Namespace: `mixed-fs`
- Pod: `mixed-app`

- Task:
  - Root filesystem readOnly
  - Volume /data scrivibile

- Validazione:
  - Scrittura possibile solo su volume

---

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mixed-app
  namespace: mixed-fs
spec:
  volumes:
  - name: data
    emptyDir: {}

  containers:
  - name: app
    image: busybox
    command: ["sh","-c","echo ok > /data/test && sleep 3600"]
    securityContext:
      readOnlyRootFilesystem: true
    volumeMounts:
    - name: data
      mountPath: /data
```

</details>

---

## SEC-12 — Debug Security Context failure

- Namespace: `debug-sec`
- Pod: `broken-sec`

- Problema:
  - runAsUser: 99999
  - Container non parte

- Obiettivo:
  - Identificare errore

---

<details>
<summary>Soluzione</summary>

```sh
k describe pod broken-sec -n debug-sec
```

 Possibili errori:
- user non valido
- permessi filesystem

Fix:

```yaml
securityContext:
  runAsUser: 1000
```

oppure:

```yaml
runAsNonRoot: true
```

</details>

---
