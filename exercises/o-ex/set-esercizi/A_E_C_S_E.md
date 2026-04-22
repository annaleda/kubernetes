### Kubernetes Exercises — Application Environment, Configuration and Security

---

## Exercise 1

### SECTION: APPLICATION ENVIRONMENT, CONFIGURATION AND SECURITY

For this question, please set the context to cluster1 by running:

```bash
kubectl config use-context cluster1
```

Create a pod named `ckad17-qos-aecs-1` in namespace `ckad17-nqoss-aecs` with image `nginx` and container name `ckad17-qos-ctr-1-aecs`.

Define other fields such that the Pod is configured to use the Quality of Service (QoS) class of `Burstable`.

Also retrieve the name and QoS class of each pod in the namespace `ckad17-nqoss-aecs` in the below format and save the output to a file named `qos_status_aecs_1` in the `/root` directory.

Format:

```text
NAME    QOS
pod-1   qos_class
pod-2   qos_class
```

<details>
<summary>Soluzione</summary>

```bash
kubectl config use-context cluster1
cat <<'YAML' | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: ckad17-qos-aecs-1
  namespace: ckad17-nqoss-aecs
spec:
  containers:
  - name: ckad17-qos-ctr-1-aecs
    image: nginx
    resources:
      requests:
        cpu: "100m"
        memory: "128Mi"
      limits:
        cpu: "200m"
        memory: "256Mi"
YAML
kubectl get pods -n ckad17-nqoss-aecs -o custom-columns=NAME:.metadata.name,QOS:.status.qosClass > /root/qos_status_aecs_1
cat /root/qos_status_aecs_1
```

</details>

---

## Exercise 2

### SECTION: APPLICATION ENVIRONMENT, CONFIGURATION AND SECURITY

For this question, please set the context to cluster2 by running:

```bash
kubectl config use-context cluster2
```

Create a ConfigMap named `app-config-aecs2` in namespace `config-aecs2` with the following data:

- `APP_COLOR=blue`
- `APP_MODE=production`

Then create a pod named `app-config-pod-aecs2` using image `busybox:1.36` that prints environment variables and sleeps for 3600 seconds.

Load all ConfigMap entries as environment variables inside the container.

<details>
<summary>Soluzione</summary>

```bash
kubectl config use-context cluster2
kubectl create ns config-aecs2
kubectl create configmap app-config-aecs2 -n config-aecs2 \
  --from-literal=APP_COLOR=blue \
  --from-literal=APP_MODE=production

cat <<'YAML' | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: app-config-pod-aecs2
  namespace: config-aecs2
spec:
  containers:
  - name: app-config-pod-aecs2
    image: busybox:1.36
    command: ["sh","-c","env && sleep 3600"]
    envFrom:
    - configMapRef:
        name: app-config-aecs2
YAML
```

</details>

---

## Exercise 3

### SECTION: APPLICATION ENVIRONMENT, CONFIGURATION AND SECURITY

For this question, please set the context to cluster3 by running:

```bash
kubectl config use-context cluster3
```

Create a secret named `db-secret-aecs3` in namespace `secret-aecs3` with the following values:

- username: `admin`
- password: `redhat123`

Then create a pod named `db-client-aecs3` using image `nginx` and expose the secret values inside the container as environment variables named:

- `DB_USER`
- `DB_PASS`

<details>
<summary>Soluzione</summary>

```bash
kubectl config use-context cluster3
kubectl create ns secret-aecs3
kubectl create secret generic db-secret-aecs3 -n secret-aecs3 \
  --from-literal=username=admin \
  --from-literal=password=redhat123

cat <<'YAML' | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: db-client-aecs3
  namespace: secret-aecs3
spec:
  containers:
  - name: db-client-aecs3
    image: nginx
    env:
    - name: DB_USER
      valueFrom:
        secretKeyRef:
          name: db-secret-aecs3
          key: username
    - name: DB_PASS
      valueFrom:
        secretKeyRef:
          name: db-secret-aecs3
          key: password
YAML
```

</details>

---

## Exercise 4

### SECTION: APPLICATION ENVIRONMENT, CONFIGURATION AND SECURITY

For this question, please set the context to cluster1 by running:

```bash
kubectl config use-context cluster1
```

Create a pod named `config-volume-aecs4` in namespace `volume-aecs4` using image `nginx`.

Create a ConfigMap named `web-content-aecs4` with the following key/value:

- `index.html`: `Welcome to CKAD practice`

Mount the ConfigMap as a volume inside the container at path `/usr/share/nginx/html`.

<details>
<summary>Soluzione</summary>

```bash
kubectl config use-context cluster1
kubectl create ns volume-aecs4
kubectl create configmap web-content-aecs4 -n volume-aecs4 --from-literal=index.html='Welcome to CKAD practice'

cat <<'YAML' | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: config-volume-aecs4
  namespace: volume-aecs4
spec:
  containers:
  - name: config-volume-aecs4
    image: nginx
    volumeMounts:
    - name: web-content
      mountPath: /usr/share/nginx/html
  volumes:
  - name: web-content
    configMap:
      name: web-content-aecs4
YAML
```

</details>

---

## Exercise 5

### SECTION: APPLICATION ENVIRONMENT, CONFIGURATION AND SECURITY

For this question, please set the context to cluster2 by running:

```bash
kubectl config use-context cluster2
```

Create a service account named `sa-aecs5` in namespace `security-aecs5`.

Then create a pod named `pod-aecs5` using image `busybox` with command:

```bash
sh -c 'sleep 3600'
```

Configure the pod to use the service account `sa-aecs5`.

<details>
<summary>Soluzione</summary>

```bash
kubectl config use-context cluster2
kubectl create ns security-aecs5
kubectl create serviceaccount sa-aecs5 -n security-aecs5

cat <<'YAML' | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: pod-aecs5
  namespace: security-aecs5
spec:
  serviceAccountName: sa-aecs5
  containers:
  - name: pod-aecs5
    image: busybox
    command: ["sh","-c","sleep 3600"]
YAML
```

</details>

---

## Exercise 6

### SECTION: APPLICATION ENVIRONMENT, CONFIGURATION AND SECURITY

For this question, please set the context to cluster3 by running:

```bash
kubectl config use-context cluster3
```

Create a pod named `secure-pod-aecs6` in namespace `secure-aecs6` with image `nginx`.

Configure the container with the following security settings:

- `runAsUser: 1000`
- `allowPrivilegeEscalation: false`
- `readOnlyRootFilesystem: true`

Container name must be `secure-ctr-aecs6`.

<details>
<summary>Soluzione</summary>

```bash
kubectl config use-context cluster3
kubectl create ns secure-aecs6
cat <<'YAML' | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: secure-pod-aecs6
  namespace: secure-aecs6
spec:
  containers:
  - name: secure-ctr-aecs6
    image: nginx
    securityContext:
      runAsUser: 1000
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
YAML
```

</details>

---

## Exercise 7

### SECTION: APPLICATION ENVIRONMENT, CONFIGURATION AND SECURITY

For this question, please set the context to cluster1 by running:

```bash
kubectl config use-context cluster1
```

Create a pod named `resource-qos-aecs7` in namespace `qos-aecs7` using image `nginx`.

Container name: `resource-ctr-aecs7`

Configure the pod so that it gets QoS class `Guaranteed`.

<details>
<summary>Soluzione</summary>

```bash
kubectl config use-context cluster1
kubectl create ns qos-aecs7
cat <<'YAML' | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: resource-qos-aecs7
  namespace: qos-aecs7
spec:
  containers:
  - name: resource-ctr-aecs7
    image: nginx
    resources:
      requests:
        cpu: "200m"
        memory: "256Mi"
      limits:
        cpu: "200m"
        memory: "256Mi"
YAML
```

</details>

---

## Exercise 8

### SECTION: APPLICATION ENVIRONMENT, CONFIGURATION AND SECURITY

For this question, please set the context to cluster2 by running:

```bash
kubectl config use-context cluster2
```

Create a secret named `tls-aecs8` in namespace `tls-aecs8` from the following files already present on the student node:

- `/opt/certs/tls.crt`
- `/opt/certs/tls.key`

After creating the secret, create a pod named `tls-reader-aecs8` using image `busybox` and mount the secret at `/etc/tls`.

The pod should remain running for inspection.

<details>
<summary>Soluzione</summary>

```bash
kubectl config use-context cluster2
kubectl create ns tls-aecs8
kubectl create secret tls tls-aecs8 -n tls-aecs8 \
  --cert=/opt/certs/tls.crt \
  --key=/opt/certs/tls.key

cat <<'YAML' | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: tls-reader-aecs8
  namespace: tls-aecs8
spec:
  containers:
  - name: tls-reader-aecs8
    image: busybox
    command: ["sh","-c","sleep 3600"]
    volumeMounts:
    - name: tls-vol
      mountPath: /etc/tls
  volumes:
  - name: tls-vol
    secret:
      secretName: tls-aecs8
YAML
```

</details>

---

## Exercise 9

### SECTION: APPLICATION ENVIRONMENT, CONFIGURATION AND SECURITY

For this question, please set the context to cluster3 by running:

```bash
kubectl config use-context cluster3
```

Create a pod named `env-check-aecs9` in namespace `env-aecs9` using image `busybox:1.36`.

Requirements:

- Container name: `env-ctr-aecs9`
- Command: `['sh', '-c', 'env && sleep 3600']`
- Add environment variable `PLATFORM=ckad`
- Add environment variable `ENVIRONMENT=practice`

<details>
<summary>Soluzione</summary>

```bash
kubectl config use-context cluster3
kubectl create ns env-aecs9
cat <<'YAML' | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: env-check-aecs9
  namespace: env-aecs9
spec:
  containers:
  - name: env-ctr-aecs9
    image: busybox:1.36
    command: ["sh","-c","env && sleep 3600"]
    env:
    - name: PLATFORM
      value: ckad
    - name: ENVIRONMENT
      value: practice
YAML
```

</details>

---

## Exercise 10

### SECTION: APPLICATION ENVIRONMENT, CONFIGURATION AND SECURITY

For this question, please set the context to cluster1 by running:

```bash
kubectl config use-context cluster1
```

A pod manifest is available at `/opt/pod-aecs10.yaml`, but the pod does not start correctly.

Inspect and fix the manifest so that:

- Pod name is `fixed-pod-aecs10`
- Namespace is `fix-aecs10`
- Image is `nginx:1.25`
- Container name is `fixed-ctr-aecs10`
- An environment variable `MODE=prod` is present

Apply the fixed manifest and verify the pod reaches Running state.

<details>
<summary>Soluzione</summary>

```bash
kubectl config use-context cluster1
kubectl create ns fix-aecs10
vi /opt/pod-aecs10.yaml

# Correggi name, namespace, image, container name ed env MODE=prod
kubectl apply -f /opt/pod-aecs10.yaml
kubectl get pod fixed-pod-aecs10 -n fix-aecs10
```

</details>
