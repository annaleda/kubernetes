# Kubernetes Exercises — Services and Networking

Questa raccolta contiene esercizi in stile CKAD focalizzati su servizi, DNS, network troubleshooting ed esposizione applicativa.

---

## Exercise 1

### SECTION: SERVICES AND NETWORKING

For this question, please set the context to cluster3 by running:

```bash
kubectl config use-context cluster3
```

We have an external webserver running on `student-node` which is exposed at port `9999`.

We have also created a service called `external-webserver-sn1` that should connect to our local webserver from within cluster3, but at the moment it is not working as expected.

Fix the issue so that other pods within cluster3 can use `external-webserver-sn1` service to access the webserver.

<details>
<summary>Soluzione</summary>

```bash
kubectl config use-context cluster3
kubectl get svc,endpoints external-webserver-sn1 -A -o wide

# Se il service deve puntare a un IP esterno, usa Service + Endpoints
kubectl edit svc external-webserver-sn1 -n default
# assicurati che la porta sia 9999 o quella richiesta dal service

kubectl apply -f - <<'YAML'
apiVersion: v1
kind: Endpoints
metadata:
  name: external-webserver-sn1
  namespace: default
subsets:
- addresses:
  - ip: <STUDENT_NODE_IP>
  ports:
  - port: 9999
YAML
```

> Nota: sostituisci `<STUDENT_NODE_IP>` con l'IP reale del `student-node`.

</details>

---

## Exercise 2

### SECTION: SERVICES AND NETWORKING

For this question, please set the context to cluster1 by running:

```bash
kubectl config use-context cluster1
```

Create a deployment named `nginx-sn2` in namespace `network-sn2` using image `nginx` with `2` replicas.

Expose the deployment with a service named `nginx-sn2-svc` using the following specifications:

- Type: `NodePort`
- Service port: `80`
- Target port: `80`
- Node port: `30080`

<details>
<summary>Soluzione</summary>

```bash
kubectl config use-context cluster1
kubectl create ns network-sn2
kubectl create deployment nginx-sn2 --image=nginx -n network-sn2 --replicas=2
kubectl expose deployment nginx-sn2 --name=nginx-sn2-svc --type=NodePort --port=80 --target-port=80 -n network-sn2
kubectl patch svc nginx-sn2-svc -n network-sn2 -p '{"spec":{"ports":[{"port":80,"targetPort":80,"nodePort":30080}]}}'
kubectl get svc -n network-sn2
```

</details>

---

## Exercise 3

### SECTION: SERVICES AND NETWORKING

For this question, please set the context to cluster2 by running:

```bash
kubectl config use-context cluster2
```

A pod named `tester-sn3` is running in namespace `network-sn3`.

A service named `api-sn3-svc` also exists in the same namespace, but the pod cannot resolve it correctly.

Investigate and fix the issue so that the DNS name `api-sn3-svc.network-sn3.svc.cluster.local` resolves successfully from inside the pod.

<details>
<summary>Soluzione</summary>

```bash
kubectl config use-context cluster2
kubectl get pod tester-sn3 -n network-sn3 -o yaml
kubectl get svc api-sn3-svc -n network-sn3 -o yaml
kubectl exec -it tester-sn3 -n network-sn3 -- nslookup api-sn3-svc.network-sn3.svc.cluster.local

# Verifica tipicamente:
# - namespace corretto
# - service name corretto
# - DNS policy del pod (di norma ClusterFirst)
# - CoreDNS funzionante
kubectl get pods -n kube-system | grep -i coredns
kubectl edit pod tester-sn3 -n network-sn3
# se necessario imposta dnsPolicy: ClusterFirst
```

</details>

---

## Exercise 4

### SECTION: SERVICES AND NETWORKING

For this question, please set the context to cluster3 by running:

```bash
kubectl config use-context cluster3
```

Create a pod named `backend-sn4` in namespace `svc-sn4` with image `httpd` and label `app=backend-sn4`.

Then create a service named `backend-sn4-svc` with the following requirements:

- Type: `ClusterIP`
- Port: `8080`
- Target port: `80`
- Selector: `app=backend-sn4`

Verify that the service has active endpoints.

<details>
<summary>Soluzione</summary>

```bash
kubectl config use-context cluster3
kubectl create ns svc-sn4
kubectl run backend-sn4 --image=httpd -n svc-sn4 --labels=app=backend-sn4
kubectl expose pod backend-sn4 --name=backend-sn4-svc --port=8080 --target-port=80 -n svc-sn4
kubectl get svc,endpoints -n svc-sn4
```

</details>

---

## Exercise 5

### SECTION: SERVICES AND NETWORKING

For this question, please set the context to cluster1 by running:

```bash
kubectl config use-context cluster1
```

A service named `broken-svc-sn5` exists in namespace `svc-sn5`, but it does not route traffic to the intended pods.

Fix the issue so that the service correctly forwards requests to pods with label:

```text
app=web-sn5
```

<details>
<summary>Soluzione</summary>

```bash
kubectl config use-context cluster1
kubectl get svc broken-svc-sn5 -n svc-sn5 -o yaml
kubectl get pods -n svc-sn5 --show-labels
kubectl edit svc broken-svc-sn5 -n svc-sn5
# correggi selector:
# selector:
#   app: web-sn5
kubectl get endpoints broken-svc-sn5 -n svc-sn5
```

</details>

---

## Exercise 6

### SECTION: SERVICES AND NETWORKING

For this question, please set the context to cluster2 by running:

```bash
kubectl config use-context cluster2
```

Create a headless service named `db-headless-sn6` in namespace `db-sn6` for pods labeled `app=db-sn6`.

Requirements:

- Service type: `ClusterIP`
- Cluster IP: `None`
- Port: `5432`
- Target port: `5432`

<details>
<summary>Soluzione</summary>

```bash
kubectl config use-context cluster2
kubectl create ns db-sn6
cat <<'YAML' | kubectl apply -f -
apiVersion: v1
kind: Service
metadata:
  name: db-headless-sn6
  namespace: db-sn6
spec:
  clusterIP: None
  selector:
    app: db-sn6
  ports:
  - port: 5432
    targetPort: 5432
YAML
kubectl get svc db-headless-sn6 -n db-sn6
```

</details>

---

## Exercise 7

### SECTION: SERVICES AND NETWORKING

For this question, please set the context to cluster3 by running:

```bash
kubectl config use-context cluster3
```

Create a service named `external-name-sn7` in namespace `ext-sn7` that points to the external DNS name `example.com`.

Requirements:

- Service type must be `ExternalName`
- The service name must be `external-name-sn7`

After creating it, verify the service definition.

<details>
<summary>Soluzione</summary>

```bash
kubectl config use-context cluster3
kubectl create ns ext-sn7
cat <<'YAML' | kubectl apply -f -
apiVersion: v1
kind: Service
metadata:
  name: external-name-sn7
  namespace: ext-sn7
spec:
  type: ExternalName
  externalName: example.com
YAML
kubectl get svc external-name-sn7 -n ext-sn7 -o yaml
```

</details>

---

## Exercise 8

### SECTION: SERVICES AND NETWORKING

For this question, please set the context to cluster1 by running:

```bash
kubectl config use-context cluster1
```

In namespace `ingress-sn8`, a deployment named `web-sn8` and a service named `web-sn8-svc` already exist.

Create an ingress named `web-sn8-ing` with the following requirements:

- Host: `ckad-sn8.local`
- Path: `/`
- PathType: `Prefix`
- Route traffic to service `web-sn8-svc` on port `80`

<details>
<summary>Soluzione</summary>

```bash
kubectl config use-context cluster1
cat <<'YAML' | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-sn8-ing
  namespace: ingress-sn8
spec:
  rules:
  - host: ckad-sn8.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-sn8-svc
            port:
              number: 80
YAML
kubectl get ingress -n ingress-sn8
```

</details>

---

## Exercise 9

### SECTION: SERVICES AND NETWORKING

For this question, please set the context to cluster2 by running:

```bash
kubectl config use-context cluster2
```

A service named `dns-check-sn9` exists in namespace `dns-sn9`.

Launch a temporary pod using image `busybox:1.36` and verify that the service is reachable using DNS.

Save the output of the successful DNS lookup to `/root/dns-sn9-output`.

<details>
<summary>Soluzione</summary>

```bash
kubectl config use-context cluster2
kubectl run dns-tmp-sn9 --rm -it --restart=Never --image=busybox:1.36 -n dns-sn9 -- nslookup dns-check-sn9 > /root/dns-sn9-output
cat /root/dns-sn9-output
```

</details>

---

## Exercise 10

### SECTION: SERVICES AND NETWORKING

For this question, please set the context to cluster3 by running:

```bash
kubectl config use-context cluster3
```

We need to expose an external IP address `8.8.8.8` on port `53` inside the cluster.

Create the required Kubernetes resources in namespace `external-ip-sn10` so that pods in the cluster can access the external endpoint through a service named `google-dns-sn10`.

The final service must listen on port `53`.

<details>
<summary>Soluzione</summary>

```bash
kubectl config use-context cluster3
kubectl create ns external-ip-sn10
cat <<'YAML' | kubectl apply -f -
apiVersion: v1
kind: Service
metadata:
  name: google-dns-sn10
  namespace: external-ip-sn10
spec:
  ports:
  - port: 53
    targetPort: 53
---
apiVersion: v1
kind: Endpoints
metadata:
  name: google-dns-sn10
  namespace: external-ip-sn10
subsets:
- addresses:
  - ip: 8.8.8.8
  ports:
  - port: 53
YAML
kubectl get svc,endpoints -n external-ip-sn10
```

</details>
