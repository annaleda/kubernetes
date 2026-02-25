# CKAD Security

## Teoria

---

## Security in Kubernetes (CKAD Focus)

Nel CKAD la sicurezza è principalmente a livello di:

- Pod Security
- ServiceAccount
- RBAC
- SecurityContext

---

# Pod Security

La Pod Security serve a limitare i privilegi dei container e ridurre i rischi di sicurezza.

Si controlla:
  - Esecuzione come root o non-root
  - Accesso al filesystem
  - Capability Linux
  - Escalation di privilegi

<img width="919" height="544" alt="Immagine 2026-02-24 132005" src="https://github.com/user-attachments/assets/d7dd8edb-119d-4303-93f0-d88b8779dde6" />


---

# ServiceAccount

Ogni Pod può autenticarsi verso l’API Server usando un ServiceAccount.
È l’identità del Pod dentro il cluster.

`Default ServiceAccount`

- Ogni namespace ha un ServiceAccount di default.
- Se non specificato, il Pod usa quello.

<img width="1191" height="658" alt="Immagine 2026-02-24 132633" src="https://github.com/user-attachments/assets/581af792-8828-4919-aac4-a3b7ecd51a43" />

Creazione:

```
kubectl create serviceaccount my-sa
```

Assegnazione a un Pod:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  serviceAccountName: my-sa
  containers:
  - name: nginx
    image: nginx
```
<img width="1164" height="721" alt="Immagine 2026-02-24 133108" src="https://github.com/user-attachments/assets/de5f77b3-b58f-40ea-866a-2803f133bec5" />

`Token Automation`

Per default Kubernetes può montare automaticamente il token del ServiceAccount nel Pod.

Disabilitare automount token:

```yaml
automountServiceAccountToken: false
```
<img width="625" height="417" alt="Immagine 2026-02-24 133509" src="https://github.com/user-attachments/assets/dc6fa975-673f-4297-84a0-c188f365dccb" />

---

> `ServiceAccount` = identità del Pod dentro Kubernetes

> `SecurityContext` = controllo dei privilegi del container

| Concetto | Serve a |
|---|---|
| ServiceAccount | Autenticazione |
| SecurityContext | Privilegi runtime |
| Pod Security | Policy di sicurezza |

In Kubernetes:

> `ServiceAccount` → identifica il Pod (autenticazione)

> `RBAC` → decide cosa può fare l’identità autenticata (autorizzazione)

| Concetto | Serve a | Livello |
|---|---|---|
| ServiceAccount | Identità del Pod | Autenticazione |
| RBAC | Permessi su risorse | Autorizzazione |
---

# RBAC (Role-Based Access Control)

Controlla chi può fare cosa nel cluster.

Componenti principali:

- Role (namespace)
- ClusterRole (cluster-wide)
- RoleBinding
- ClusterRoleBinding
  
<img width="1094" height="621" alt="Immagine 2026-02-24 234057" src="https://github.com/user-attachments/assets/579f16a2-14ac-407b-853e-ca1a22996055" />

---

## Role

Permessi limitati a un namespace.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
```

---

## RoleBinding

Associa Role a un utente o ServiceAccount.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods-binding
subjects:
- kind: ServiceAccount
  name: my-sa
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

---

## ClusterRole & ClusterRoleBinding

Permessi validi per tutto il cluster.

Esempio CLI:

```
kubectl create clusterrole pod-reader --verb=get,list,watch --resource=pods
kubectl create clusterrolebinding read-all --clusterrole=pod-reader --serviceaccount=default:my-sa
```
<img width="1080" height="630" alt="Immagine 2026-02-24 234616" src="https://github.com/user-attachments/assets/7be97a1a-de0d-4a2f-b0e0-b98bcb42b2c6" />

---

# SecurityContext

Definisce privilegi a livello di:

- Pod
- Container

Esempi di opzioni:
- `runAsUser`
- `runAsNonRoot`
- `readOnlyRootFilesystem`
- `allowPrivilegeEscalation`
  
Serve per controllare:

- Utente e gruppo di esecuzione
- Privilegi Linux
- Accesso al filesystem
- Capabilities del kernel
  
---

## Impostare privilegi container

```yaml
securityContext:
  allowPrivilegeEscalation: false
  readOnlyRootFilesystem: true
```

- `allowPrivilegeEscalation: false` → impedisce al processo di ottenere privilegi più elevati (es. tramite setuid)

- `readOnlyRootFilesystem`  →  Rende il filesystem root del container in sola lettura

---

## Run as non-root

Best practice: non eseguire container come root.

securityContext:
  runAsUser: 1000
  runAsNonRoot: true

- `runAsUser: 1000` → il container gira con UID 1000

- `runAsNonRoot: true` → impedisce l’avvio se l’immagine prova a usare root

---

## Aggiungere capability Linux

```yaml
securityContext:
  capabilities:
    add: ["NET_ADMIN"]
    drop: ["ALL"]
```
- `drop: ["ALL"]` → rimuove tutte le capability

- `add: ["NET_ADMIN"]` → aggiunge solo quella necessaria
---

# Pod-level vs Container-level SecurityContext

- Pod securityContext → applicato a tutti i container
- Container securityContext → specifico per container

Container-level sovrascrive quello del Pod.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secure-pod
spec:
  securityContext:
    runAsUser: 1000
    runAsNonRoot: true
  containers:
  - name: app
    image: nginx
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      capabilities:
        drop: ["ALL"]
```
---

# Verifica Permessi

Testare accesso:

```
kubectl auth can-i get pods
kubectl auth can-i create deployments --as=system:serviceaccount:default:my-sa
```
<img width="732" height="607" alt="Immagine 2026-02-24 234344" src="https://github.com/user-attachments/assets/5fe65618-af91-46e8-8397-2452f06710b8" />

---

# Best Practices CKAD

- Eseguire container come non-root
- Usare least privilege RBAC
- Disabilitare privilege escalation
- Usare readOnlyRootFilesystem quando possibile
- Non usare immagini latest in produzione

---

## Esercizi

1. Creare un Pod che gira come utente non-root.
2. Impostare readOnlyRootFilesystem.
3. Creare ServiceAccount personalizzato.
4. Associare ServiceAccount a un Pod.
5. Creare Role per leggere Pod.
6. Creare RoleBinding per ServiceAccount.
7. Verificare accesso con `kubectl auth can-i`.
8. Creare semplice NetworkPolicy che blocca traffico.

---


