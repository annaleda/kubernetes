### Security Context (6 esercizi)
---

## SEC-1 — RunAsUser

- Namespace: `secure-apps`
- Pod: `secure-pod`

Task:

Container deve girare come user ID 1000

Image: nginx

Validazione:

Controllare user nel container

<details> <summary>Soluzione</summary>
kubectl create ns secure-apps

kubectl run secure-pod \
--image=nginx \
-n secure-apps \
--dry-run=client -o yaml > secure-pod.yaml

Edit YAML:

spec:
  securityContext:
    runAsUser: 1000

  containers:
  - name: secure-pod
    image: nginx

Apply:

kubectl apply -f secure-pod.yaml
</details>

---

## SEC-2 — ReadOnlyRootFilesystem

- Namespace: `readonly-ns`
- Pod: `readonly-app`

Task:

Root filesystem deve essere read-only

Validazione:

Container non può scrivere su filesystem

<details> <summary>Soluzione</summary>
securityContext:
  readOnlyRootFilesystem: true

Container section:

containers:
- name: readonly-app
  image: nginx
  securityContext:
    readOnlyRootFilesystem: true
</details>

## SEC-3 — Privileged Container

- Namespace: `privileged-ns`
- Pod: `priv-app`

Task:

Container deve girare in privileged mode

Validazione:

Privileged flag attivo

<details> <summary>Soluzione</summary>
securityContext:
  privileged: true
</details>

---

## SEC-4 — Capabilities Drop

- Namespace: `cap-ns`
- Pod: `cap-app`

Task:

Rimuovere capability NET_RAW

Validazione:

Capability non presente

<details> <summary>Soluzione</summary>
securityContext:
  capabilities:
    drop:
    - NET_RAW
</details>

---

## SEC-5 — Pod + Container Security Context Combined

- Namespace: `mix-sec`
- Pod: `mix-sec-app`

runAsUser: 2000 (pod level)

Container:

runAsUser: 3000 (override container level)

Validazione:

Container level ha priorità

<details> <summary>Soluzione</summary>
spec:
  securityContext:
    runAsUser: 2000

  containers:
  - name: mix-app
    image: nginx
    securityContext:
      runAsUser: 3000
</details>

---

## SEC-6 — Allow Privilege Escalation

- Namespace: `escalation-ns`
- Pod: `escalate-app`

Task:

Consentire privilege escalation

Validazione:

allowPrivilegeEscalation = true

<details> <summary>Soluzione</summary>
securityContext:
  allowPrivilegeEscalation: true
</details>


---

