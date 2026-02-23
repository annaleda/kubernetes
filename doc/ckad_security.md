# CKAD Security

## Teoria

---

## Security in Kubernetes (CKAD Focus)

Nel CKAD la sicurezza è principalmente a livello di:

- Pod Security
- ServiceAccount
- RBAC
- SecurityContext
- Network Policies (base)

---

# ServiceAccount

Ogni Pod usa un ServiceAccount per autenticarsi verso l’API Server.

Default:
- Ogni namespace ha un `default` ServiceAccount.
- Se non specificato, il Pod usa quello.

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

Disabilitare automount token:

```yaml
automountServiceAccountToken: false
```

---

# RBAC (Role-Based Access Control)

Controlla chi può fare cosa nel cluster.

Componenti principali:

- Role (namespace)
- ClusterRole (cluster-wide)
- RoleBinding
- ClusterRoleBinding

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

---

# SecurityContext

Definisce privilegi a livello di:

- Pod
- Container

---

## Run as non-root

```yaml
securityContext:
  runAsUser: 1000
  runAsNonRoot: true
```

---

## Impostare privilegi container

```yaml
securityContext:
  allowPrivilegeEscalation: false
  readOnlyRootFilesystem: true
```

---

## Aggiungere capability Linux

```yaml
securityContext:
  capabilities:
    add: ["NET_ADMIN"]
    drop: ["ALL"]
```

---

# Pod-level vs Container-level SecurityContext

- Pod securityContext → applicato a tutti i container
- Container securityContext → specifico per container

Container-level sovrascrive quello del Pod.

---

# Image Pull Secrets

Per immagini private:

```
kubectl create secret docker-registry my-secret \
  --docker-username=user \
  --docker-password=pass \
  --docker-email=email@example.com
```

Nel Pod:

```yaml
imagePullSecrets:
- name: my-secret
```

---

# Network Policies (CKAD Base)

Controllano traffico tra Pod.

Senza NetworkPolicy:
- Tutto il traffico è permesso.

Con NetworkPolicy:
- Il traffico è negato per default se policy applicata.

Esempio base:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-nginx
spec:
  podSelector:
    matchLabels:
      app: nginx
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
```

---

# Verifica Permessi

Testare accesso:

```
kubectl auth can-i get pods
kubectl auth can-i create deployments --as=system:serviceaccount:default:my-sa
```

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


