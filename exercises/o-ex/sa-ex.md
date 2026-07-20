- [ Home ](../../readme.md)   | [ Teoria ](../../arguments.md)   | [ Info Exam ](../../doc/ckad_exam_strategy.md)    | [ Home Other Exercises ](../o_exercises.md)   
--- 
### ServiceAccount (12 esercizi)

## SA-1 — Creazione ServiceAccount

### Preparazione

```bash
kubectl delete namespace sa-test --ignore-not-found
kubectl create namespace sa-test
```

### Task

Nel namespace `sa-test`:

- crea la ServiceAccount `app-sa`;
- crea il Pod `sa-pod` con immagine `nginx`;
- configura il Pod affinché usi `app-sa`.

### Validazione

```bash
kubectl get sa app-sa -n sa-test
kubectl get pod sa-pod -n sa-test \
  -o jsonpath='{.spec.serviceAccountName}{"\n"}'
```

<details>
<summary>Soluzione</summary>

```bash
kubectl create serviceaccount app-sa -n sa-test

kubectl run sa-pod \
  --image=nginx \
  -n sa-test \
  --dry-run=client -o yaml > sa-pod.yaml
```

Aggiungi sotto `spec`:

```yaml
serviceAccountName: app-sa
```

Poi:

```bash
kubectl apply -f sa-pod.yaml
```

</details>

### Cleanup

```bash
kubectl delete namespace sa-test
rm -f sa-pod.yaml
```

---

## SA-2 — ServiceAccount predefinita

### Preparazione

```bash
kubectl delete namespace sa-test --ignore-not-found
kubectl create namespace sa-test
```

### Task

Crea nel namespace `sa-test` un Pod chiamato `default-sa-pod` con immagine `nginx`, senza specificare alcuna ServiceAccount.

### Validazione

```bash
kubectl get pod default-sa-pod -n sa-test \
  -o jsonpath='{.spec.serviceAccountName}{"\n"}'
```

L'output deve essere:

```text
default
```

<details>
<summary>Soluzione</summary>

```bash
kubectl run default-sa-pod --image=nginx -n sa-test
```

</details>

### Cleanup

```bash
kubectl delete namespace sa-test
```

---

## SA-3 — Disabilitare il token automatico

### Preparazione

```bash
kubectl delete namespace sa-test --ignore-not-found
kubectl create namespace sa-test
```

### Task

Crea nel namespace `sa-test` il Pod `no-token-pod` con:

- immagine `nginx`;
- `automountServiceAccountToken: false`.

### Validazione

```bash
kubectl get pod no-token-pod -n sa-test \
  -o jsonpath='{.spec.automountServiceAccountToken}{"\n"}'

kubectl exec no-token-pod -n sa-test -- \
  sh -c 'ls /var/run/secrets/kubernetes.io/serviceaccount 2>/dev/null || echo token-not-mounted'
```

<details>
<summary>Soluzione</summary>

```bash
cat > no-token-pod.yaml <<'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: no-token-pod
  namespace: sa-test
spec:
  automountServiceAccountToken: false
  containers:
  - name: nginx
    image: nginx
EOF

kubectl apply -f no-token-pod.yaml
```

</details>

### Cleanup

```bash
kubectl delete namespace sa-test
rm -f no-token-pod.yaml
```

---

## SA-4 — ServiceAccount, Role e RoleBinding

### Preparazione

```bash
kubectl delete namespace sa-test --ignore-not-found
kubectl create namespace sa-test
```

### Task

Nel namespace `sa-test`:

- crea la ServiceAccount `sa-roling`;
- crea la Role `sa-role` con permessi `get` e `list` sui Pod;
- crea la RoleBinding `sa-roleb`;
- collega la Role alla ServiceAccount.

### Validazione

```bash
kubectl auth can-i get pods \
  --as=system:serviceaccount:sa-test:sa-roling \
  -n sa-test

kubectl auth can-i list pods \
  --as=system:serviceaccount:sa-test:sa-roling \
  -n sa-test

kubectl auth can-i delete pods \
  --as=system:serviceaccount:sa-test:sa-roling \
  -n sa-test
```

I primi due comandi devono restituire `yes`, il terzo `no`.

<details>
<summary>Soluzione</summary>

```bash
kubectl create serviceaccount sa-roling -n sa-test

kubectl create role sa-role \
  --verb=get,list \
  --resource=pods \
  -n sa-test

kubectl create rolebinding sa-roleb \
  --role=sa-role \
  --serviceaccount=sa-test:sa-roling \
  -n sa-test
```

</details>

### Cleanup

```bash
kubectl delete namespace sa-test
```

---

## SA-5 — ServiceAccount in un Deployment

### Preparazione

```bash
kubectl delete namespace sa-test --ignore-not-found
kubectl create namespace sa-test
```

### Task

Nel namespace `sa-test`:

- crea la ServiceAccount `custom-sa`;
- crea il Deployment `sa-deployment`;
- usa immagine `nginx`;
- imposta 2 repliche;
- configura `serviceAccountName: custom-sa`.

### Validazione

```bash
kubectl get deployment sa-deployment -n sa-test
kubectl get deployment sa-deployment -n sa-test \
  -o jsonpath='{.spec.template.spec.serviceAccountName}{"\n"}'

kubectl get pods -n sa-test \
  -l app=sa-deployment \
  -o jsonpath='{range .items[*]}{.metadata.name}{" -> "}{.spec.serviceAccountName}{"\n"}{end}'
```

<details>
<summary>Soluzione</summary>

```bash
kubectl create serviceaccount custom-sa -n sa-test

kubectl create deployment sa-deployment \
  --image=nginx \
  --replicas=2 \
  -n sa-test \
  --dry-run=client -o yaml > sa-deployment.yaml
```

Aggiungi in `spec.template.spec`:

```yaml
serviceAccountName: custom-sa
```

Poi:

```bash
kubectl apply -f sa-deployment.yaml
```

</details>

### Cleanup

```bash
kubectl delete namespace sa-test
rm -f sa-deployment.yaml
```

---

## SA-6 — Stesso nome in namespace diversi

### Preparazione

```bash
kubectl delete namespace one two --ignore-not-found
kubectl create namespace one
kubectl create namespace two
```

### Task

Crea una ServiceAccount chiamata `sa-duplicate` in entrambi i namespace:

- `one`;
- `two`.

### Validazione

```bash
kubectl get sa sa-duplicate -n one
kubectl get sa sa-duplicate -n two
kubectl get sa sa-duplicate -n one -o jsonpath='{.metadata.uid}{"\n"}'
kubectl get sa sa-duplicate -n two -o jsonpath='{.metadata.uid}{"\n"}'
```

Gli UID devono essere diversi.

<details>
<summary>Soluzione</summary>

```bash
kubectl create serviceaccount sa-duplicate -n one
kubectl create serviceaccount sa-duplicate -n two
```

</details>

### Cleanup

```bash
kubectl delete namespace one two
```

---

## SA-7 — Secret token manuale

### Preparazione

```bash
kubectl delete namespace sa-secret-ns --ignore-not-found
kubectl create namespace sa-secret-ns
kubectl create serviceaccount custom-sa -n sa-secret-ns
rm -f custom-sa-token.yaml
```

### Task

Nel namespace `sa-secret-ns`:

- crea un Secret chiamato `custom-sa-token`;
- usa tipo `kubernetes.io/service-account-token`;
- associa il Secret alla ServiceAccount `custom-sa` tramite annotazione.

### Validazione

```bash
kubectl get secret custom-sa-token -n sa-secret-ns \
  -o jsonpath='{.type}{"\n"}'

kubectl get secret custom-sa-token -n sa-secret-ns \
  -o jsonpath='{.metadata.annotations.kubernetes\.io/service-account\.name}{"\n"}'

kubectl describe serviceaccount custom-sa -n sa-secret-ns
```

<details>
<summary>Soluzione</summary>

```bash
cat > custom-sa-token.yaml <<'EOF'
apiVersion: v1
kind: Secret
metadata:
  name: custom-sa-token
  namespace: sa-secret-ns
  annotations:
    kubernetes.io/service-account.name: custom-sa
type: kubernetes.io/service-account-token
EOF

kubectl apply -f custom-sa-token.yaml
```

</details>

### Cleanup

```bash
kubectl delete namespace sa-secret-ns
rm -f custom-sa-token.yaml
```

---

## SA-8 — Disabilitare automount sulla ServiceAccount default

### Preparazione

```bash
kubectl delete namespace override-ns --ignore-not-found
kubectl create namespace override-ns
```

### Task

Nel namespace `override-ns`:

- modifica la ServiceAccount `default`;
- imposta `automountServiceAccountToken: false`;
- crea il Pod `test` con immagine `nginx`.

### Validazione

```bash
kubectl get sa default -n override-ns \
  -o jsonpath='{.automountServiceAccountToken}{"\n"}'

kubectl get pod test -n override-ns \
  -o jsonpath='{.spec.serviceAccountName}{"\n"}'

kubectl exec test -n override-ns -- \
  sh -c 'ls /var/run/secrets/kubernetes.io/serviceaccount 2>/dev/null || echo token-not-mounted'
```

<details>
<summary>Soluzione</summary>

```bash
kubectl patch serviceaccount default -n override-ns \
  --type=merge \
  -p '{"automountServiceAccountToken":false}'

kubectl run test --image=nginx -n override-ns
```

</details>

### Cleanup

```bash
kubectl delete namespace override-ns
```

---

## SA-9 — Debug di una ServiceAccount inesistente

### Preparazione

```bash
kubectl delete namespace sa-test --ignore-not-found
kubectl create namespace sa-test

cat > wrong-sa-pod.yaml <<'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: wrong-sa-pod
  namespace: sa-test
spec:
  serviceAccountName: missing-sa
  containers:
  - name: nginx
    image: nginx
EOF

kubectl apply -f wrong-sa-pod.yaml
```

### Task

Il Pod `wrong-sa-pod` non viene avviato.

- identifica la causa;
- correggi il problema senza cambiare il nome della ServiceAccount nel Pod.

### Validazione

```bash
kubectl get pod wrong-sa-pod -n sa-test
kubectl get sa missing-sa -n sa-test
```

Il Pod deve raggiungere lo stato `Running`.

<details>
<summary>Soluzione</summary>

```bash
kubectl describe pod wrong-sa-pod -n sa-test
kubectl get events -n sa-test --sort-by=.metadata.creationTimestamp

kubectl create serviceaccount missing-sa -n sa-test
```

</details>

### Cleanup

```bash
kubectl delete namespace sa-test
rm -f wrong-sa-pod.yaml
```

---

## SA-10 — Accesso limitato a un namespace

### Preparazione

```bash
kubectl delete namespace limited-sa other-ns --ignore-not-found
kubectl create namespace limited-sa
kubectl create namespace other-ns
```

### Task

Nel namespace `limited-sa`:

- crea la ServiceAccount `limited-sa`;
- crea una Role che consenta soltanto `get` sui Pod;
- crea una RoleBinding appropriata.

La ServiceAccount non deve ottenere lo stesso accesso in `other-ns`.

### Validazione

```bash
kubectl auth can-i get pods \
  --as=system:serviceaccount:limited-sa:limited-sa \
  -n limited-sa

kubectl auth can-i list pods \
  --as=system:serviceaccount:limited-sa:limited-sa \
  -n limited-sa

kubectl auth can-i get pods \
  --as=system:serviceaccount:limited-sa:limited-sa \
  -n other-ns
```

Output atteso: `yes`, `no`, `no`.

<details>
<summary>Soluzione</summary>

```bash
kubectl create serviceaccount limited-sa -n limited-sa

kubectl create role pod-reader \
  --verb=get \
  --resource=pods \
  -n limited-sa

kubectl create rolebinding rb \
  --role=pod-reader \
  --serviceaccount=limited-sa:limited-sa \
  -n limited-sa
```

</details>

### Cleanup

```bash
kubectl delete namespace limited-sa other-ns
```

---

## SA-11 — ServiceAccount e imagePullSecrets

### Preparazione

```bash
kubectl delete namespace registry-ns --ignore-not-found
kubectl create namespace registry-ns
rm -f reg-sa.yaml private-pod.yaml
```

### Task

Nel namespace `registry-ns`:

- crea un Secret docker-registry chiamato `reg-secret`;
- usa:
  - server: `myrepo.example.com`;
  - username: `user`;
  - password: `pass`;
- crea la ServiceAccount `reg-sa`;
- associa `reg-secret` tramite `imagePullSecrets`;
- crea il Pod `private-pod` che usa `reg-sa`.

Usa come immagine `myrepo.example.com/private-image:latest`.

> Il Pod può restare in `ImagePullBackOff`: l'obiettivo è verificare la configurazione, non l'accesso a un registry reale.

### Validazione

```bash
kubectl get sa reg-sa -n registry-ns \
  -o jsonpath='{.imagePullSecrets[*].name}{"\n"}'

kubectl get pod private-pod -n registry-ns \
  -o jsonpath='{.spec.serviceAccountName}{"\n"}'

kubectl get pod private-pod -n registry-ns \
  -o jsonpath='{.spec.imagePullSecrets[*].name}{"\n"}'
```

<details>
<summary>Soluzione</summary>

```bash
kubectl create secret docker-registry reg-secret \
  --docker-server=myrepo.example.com \
  --docker-username=user \
  --docker-password=pass \
  -n registry-ns

cat > reg-sa.yaml <<'EOF'
apiVersion: v1
kind: ServiceAccount
metadata:
  name: reg-sa
  namespace: registry-ns
imagePullSecrets:
- name: reg-secret
EOF

kubectl apply -f reg-sa.yaml

cat > private-pod.yaml <<'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: private-pod
  namespace: registry-ns
spec:
  serviceAccountName: reg-sa
  containers:
  - name: app
    image: myrepo.example.com/private-image:latest
EOF

kubectl apply -f private-pod.yaml
```

</details>

### Cleanup

```bash
kubectl delete namespace registry-ns
rm -f reg-sa.yaml private-pod.yaml
```

---

## SA-12 — Verifica del token nel container

### Preparazione

```bash
kubectl delete namespace sa-test --ignore-not-found
kubectl create namespace sa-test
```

### Task

Crea nel namespace `sa-test` il Pod `token-check` con:

- immagine `busybox`;
- comando che mantenga il Pod attivo per 3600 secondi.

Verifica la presenza dei file della ServiceAccount nel container.

### Validazione

```bash
kubectl exec token-check -n sa-test -- \
  ls -l /var/run/secrets/kubernetes.io/serviceaccount

kubectl exec token-check -n sa-test -- \
  sh -c 'test -f /var/run/secrets/kubernetes.io/serviceaccount/token && echo token-present'
```

<details>
<summary>Soluzione</summary>

```bash
kubectl run token-check \
  --image=busybox \
  -n sa-test \
  --command -- sh -c 'sleep 3600'

kubectl exec token-check -n sa-test -- \
  ls /var/run/secrets/kubernetes.io/serviceaccount
```

</details>

### Cleanup

```bash
kubectl delete namespace sa-test
```
